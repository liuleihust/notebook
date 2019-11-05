基本上所有的网络处理程序都有以下基本的处理过程:

- Read request
- Decode request
- Process service
- Encode reply
- Send reply

##  Classic Service Designs

![](https://images0.cnblogs.com/blog2015/434101/201503/112100115422225.jpg)

简单代码实现

```java
class Server implements Runnable {
    public void run() {
        try {
            ServerSocket ss = new ServerSocket(PORT);
            while (!Thread.interrupted())
            new Thread(new Handler(ss.accept())).start(); //创建新线程来handle
            // or, single-threaded, or a thread pool
        } catch (IOException ex) { /* ... */ }
    }
    
    static class Handler implements Runnable {
        final Socket socket;
        Handler(Socket s) { socket = s; }
        public void run() {
            try {
                byte[] input = new byte[MAX_INPUT];
                socket.getInputStream().read(input);
                byte[] output = process(input);
                socket.getOutputStream().write(output);
            } catch (IOException ex) { /* ... */ }
        }       
        private byte[] process(byte[] cmd) { /* ... */ }
    }
}
```

对于每一个请求都分发给一个线程，每个线程中都独自处理上面的流程。

这种模型由于IO在阻塞时会一直等待，因此在用户负载增加时，性能下降的非常快。

server导致阻塞的原因：

1. serversocket的accept方法，阻塞等待client连接，直到client连接成功。

2. 线程从socket inputstream读入数据，会进入阻塞状态，直到全部数据读完。

3. 线程向socket outputstream写入数据，会阻塞直到全部数据写完。

client导致阻塞的原因：

1. client建立连接时会阻塞，直到连接成功。

2. 线程从socket输入流读入数据，如果没有足够数据读完会进入阻塞状态，直到有数据或者读到输入流末尾。

3. 线程从socket输出流写入数据，直到输出所有数据。

4. socket.setsolinger()设置socket的延迟时间，当socket关闭时，会进入阻塞状态，直到全部数据都发送完或者超时。

改进：**采用基于事件驱动的设计，当有事件触发时，才会调用处理器进行数据处理。**

## Basic Reactor Design

![](https://images0.cnblogs.com/blog2015/434101/201503/112105210899525.jpg)

线程不会阻塞

代码实现

```java
class Reactor implements Runnable { 
    final Selector selector;
    final ServerSocketChannel serverSocket;
    Reactor(int port) throws IOException { //Reactor初始化
        selector = Selector.open();
        serverSocket = ServerSocketChannel.open();
        serverSocket.socket().bind(new InetSocketAddress(port));
        serverSocket.configureBlocking(false); //非阻塞
        SelectionKey sk = serverSocket.register(selector, SelectionKey.OP_ACCEPT); //分步处理,第一步,接收accept事件
        sk.attach(new Acceptor()); //attach callback object, Acceptor
    }
    
    public void run() { 
        try {
            while (!Thread.interrupted()) {
                selector.select();
                // 获得 有事件发生的 key
                Set selected = selector.selectedKeys();
                Iterator it = selected.iterator();
                while (it.hasNext())
                    dispatch((SelectionKey)(it.next()); //Reactor负责dispatch收到的事件
                // 处理完 清除 key
                selected.clear();
            }
        } catch (IOException ex) { /* ... */ }
    }
    
    void dispatch(SelectionKey k) {
        Runnable r = (Runnable)(k.attachment()); //调用之前注册的callback对象
        if (r != null)
            r.run();
    }
    
                             
   // 内部类
    class Acceptor ·implements Runnable { // inner
        public void run() {
            try {
                SocketChannel c = serverSocket.accept();
                if (c != null)
                new Handler(selector, c);
            }
            catch(IOException ex) { /* ... */ }
        }
    }
}

// 处理新的 Channel 
final class Handler implements Runnable {
    final SocketChannel socket;
    final SelectionKey sk;
    ByteBuffer input = ByteBuffer.allocate(MAXIN);
    ByteBuffer output = ByteBuffer.allocate(MAXOUT);
    static final int READING = 0, SENDING = 1;
    int state = READING;
    
    Handler(Selector sel, SocketChannel c) throws IOException {
        socket = c; c.configureBlocking(false);
        // Optionally try first read now
        sk = socket.register(sel, 0);
        sk.attach(this); //将Handler作为callback对象
        sk.interestOps(SelectionKey.OP_READ); //第二步,接收Read事件
        sel.wakeup();
    }
    boolean inputIsComplete() { /* ... */ }
    boolean outputIsComplete() { /* ... */ }
    void process() { /* ... */ }
    
    public void run() {
        try {
            if (state == READING) read();
            else if (state == SENDING) send();
        } catch (IOException ex) { /* ... */ }
    }
    
    void read() throws IOException {
        socket.read(input);
        if (inputIsComplete()) {
            process();
            state = SENDING;
            // Normally also do first write now
            sk.interestOps(SelectionKey.OP_WRITE); //第三步,接收write事件
        }
    }
    void send() throws IOException {
        socket.write(output);
        if (outputIsComplete()) sk.cancel(); //write完就结束了, 关闭select key
    }
}

//上面 的实现用Handler来同时处理Read和Write事件, 所以里面出现状态判断
//我们可以用State-Object pattern来更优雅的实现
class Handler { // ...
    public void run() { // initial state is reader
        socket.read(input);
        if (inputIsComplete()) {
            process();
            sk.attach(new Sender());  //状态迁移, Read后变成write, 用Sender作为新的callback对象
            sk.interest(SelectionKey.OP_WRITE);
            sk.selector().wakeup();
        }
    }
    class Sender implements Runnable {
        public void run(){ // ...
            socket.write(output);
            if (outputIsComplete()) sk.cancel();
        }
    }
}
```

这里用到了Reactor模式。

关于Reactor模式的一些概念：

这里用到了Reactor模式。

关于Reactor模式的一些概念：

这里用到了Reactor模式。

关于Reactor模式的一些概念：

**Reactor**：负责响应IO事件，当检测到一个新的事件，将其发送给相应的Handler去处理。

**Handler**：负责处理非阻塞的行为，标识系统管理的资源；同时将handler与事件绑定。

Reactor为单个线程，需要处理accept连接，同时发送请求到处理器中。

由于只有单个线程，所以处理器中的业务需要能够快速处理完。

改进：**使用多线程处理业务逻辑。：负责响应IO事件，当检测到一个新的事件，将其发送给相应的Handler去处理。**

**Handler**：负责处理非阻塞的行为，标识系统管理的资源；同时将handler与事件绑定。

**Reactor为单个线程**，需要处理accept连接，同时发送请求到处理器中。

由于只有单个线程，所以处理器中的业务需要能够快速处理完。

改进：使用多线程处理业务逻辑。：负责响应IO事件，当检测到一个新的事件，将其发送给相应的Handler去处理。

## Worker Thread Pools

![](https://images0.cnblogs.com/blog2015/434101/201503/112148547777256.jpg)

参考代码

```java
class Handler implements Runnable {
    // uses util.concurrent thread pool
    static PooledExecutor pool = new PooledExecutor(...);
    static final int PROCESSING = 3;
    // ...
    synchronized void read() { // ...
        socket.read(input);
        if (inputIsComplete()) {
            state = PROCESSING;
            pool.execute(new Processer()); //使用线程pool异步执行
        }
    }
    
    synchronized void processAndHandOff() {
        process();
        state = SENDING; // or rebind attachment
        sk.interest(SelectionKey.OP_WRITE); //process完,开始等待write事件
    }
    
    class Processer implements Runnable {
        public void run() { processAndHandOff(); }
    }
}
```

将处理器的执行放入线程池，多线程进行业务处理。但Reactor仍为单个线程。

继续改进：对于多个CPU的机器，为充分利用系统资源，将Reactor拆分为两部分。

## Using Multiple Reactors

- 为了匹配CPU 和 IO 的速率
- 每个Reactor 拥有自己的 Selector ,Thread , dispatch loop 
- 主 acceptor 将新的 连接 分发到 其他 reactors。

![](https://images0.cnblogs.com/blog2015/434101/201503/112151380898648.jpg)

```java
Selector[] selectors; //subReactors集合, 一个selector代表一个subReactor
int next = 0;
class Acceptor { // ...
    public synchronized void run() { ...
        Socket connection = serverSocket.accept(); //主selector负责accept
        if (connection != null)
            new Handler(selectors[next], connection); //选个subReactor去负责接收到的connection
        if (++next == selectors.length) next = 0;
    }
}
```

MainReactor负责**监听连接**，accept连接给subReactor处理，为什么要单独分一个Reactor来处理监听呢？因为像TCP这样需要经过3次握手才能建立连接，这个建立连接的过程也是要耗时间和资源的，单独分一个Reactor来处理，可以提高性能。这里 `Acceptor ` 与 `MainReactor ` 共享一个线程。

 