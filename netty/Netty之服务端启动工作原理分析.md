## 一、大致介绍

```
1、Netty这个词，对于熟悉并发的童鞋一点都不陌生，它是一个异步事件驱动型的网络通信框架；
2、使用Netty不需要我们关注过多NIO的API操作，简简单单的使用即可，非常方便，开发门槛较低；
3、而且Netty也经历了各大著名框架的“摧残”，足以证明其性能高，稳定性高；
4、那么本章节就来和大家分享分析一下Netty的服务端启动流程，分析Netty的源码版本为：netty-netty-4.1.22.Final；
```

## 二、简单认识Netty

### 2.1 何为Netty

```
1、是一个基于NIO的客户端、服务器端的网络通信框架；

2、是一个以提供异步的、事件驱动型的网络应用工具；

3、可以供我们快速开发高性能的、高可靠性的网络服务器与客户端；
```

### 2.2 为什么使用Netty?

```
1、开箱即用，简单操作，开发门槛低，API简单，只需关注业务实现即可，不用关心如何编写NIO；

2、自带多种协议栈且预置多种编解码功能，且定制化能力强；

3、综合性能高，已历经各大著名框架(RPC框架、消息中间件)等广泛验证，健壮性非常强大；

4、相对于JDK的NIO来说，netty在底层做了很多优化，将reactor线程的并发处理提到了极致；

5、社区相对较活跃，遇到问题可以随时提问沟通并修复； 
```

### 2.3 大致阐述启动流程

```
1、创建两个线程管理组，一个是bossGroup，一个是workerGroup，每个Group下都有一个线程组children[i]来执行任务；

2、bossGroup专门用来揽客的，就是接收客户端的请求链接，而workerGroup专门用来干事的，bossGroup揽客完了就交给workerGroup去干活了；

3、通过bind轻松的一句代码绑定注册，其实里面一点都不简单，一堆堆的操作；

4、创建NioServerSocketChannel，并且将此注册到bossGroup的子线程中的多路复用器上；

5、最后一步就是将NioServerSocketChannel绑定到指定ip、port即可，由此完成服务端的整个启动过程；
```

2.4 Netty 服务端启动Demo

```java
public class NettyServer {

    public static final int TCP_PORT = 20000;

    private final int port;

    public NettyServer(int port) {
        this.port = port;
    }

    public void start() throws Exception {
        EventLoopGroup bossGroup = null;
        EventLoopGroup workerGroup = null;
        try {
            // Server 端引导类
            ServerBootstrap serverBootstrap = new ServerBootstrap();

            // Boss 线程管理组
            bossGroup = new NioEventLoopGroup(1);

            // Worker 线程管理组
            workerGroup = new NioEventLoopGroup();

            // 将 Boss、Worker 设置到 ServerBootstrap 服务端引导类中
            serverBootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    // 指定通道类型为NioServerSocketChannel，一种异步模式，OIO阻塞模式为OioServerSocketChannel
                    .localAddress("localhost", port)//设置InetSocketAddress让服务器监听某个端口已等待客户端连接。
                    .childHandler(new ChannelInitializer<Channel>() {//设置childHandler执行所有的连接请求
                        @Override
                        protected void initChannel(Channel ch) throws Exception {
                            ch.pipeline().addLast(new PacketHeadDecoder());
                            ch.pipeline().addLast(new PacketBodyDecoder());

                            ch.pipeline().addLast(new PacketHeadEncoder());
                            ch.pipeline().addLast(new PacketBodyEncoder());

                            ch.pipeline().addLast(new PacketHandler());
                        }
                    });
            // 最后绑定服务器等待直到绑定完成，调用sync()方法会阻塞直到服务器完成绑定,然后服务器等待通道关闭，因为使用sync()，所以关闭操作也会被阻塞。
            ChannelFuture channelFuture = serverBootstrap.bind().sync();
            System.out.println("Server started，port：" + channelFuture.channel().localAddress());
            channelFuture.channel().closeFuture().sync();
        } finally {
            // Shut down all event loops to terminate all threads.
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        new NettyServer(TCP_PORT).start();
    }
}
```

## 三、常用的类结构

![](http://47.101.139.24/upload/2019/4/1646614052-5abeeb26d66682019051821091180.png)

## 四、源码分析Netty 服务端启动

### 4.1 创建bossGroup 

```java
    // NettyServer.java, Boss 线程管理组, 上面NettyServer.java中的示例代码
    bossGroup = new NioEventLoopGroup(1);

    // NioEventLoopGroup.java
    /**
     * Create a new instance using the specified number of threads, {@link ThreadFactory} and the
     * {@link SelectorProvider} which is returned by {@link SelectorProvider#provider()}.
     */
    public NioEventLoopGroup(int nThreads) {
        this(nThreads, (Executor) null);
    }    
    
    // NioEventLoopGroup.java
    public NioEventLoopGroup(int nThreads, Executor executor) {
        this(nThreads, executor, SelectorProvider.provider());
    }    
    
    // NioEventLoopGroup.java
    public NioEventLoopGroup(
            int nThreads, Executor executor, final SelectorProvider selectorProvider) {
        this(nThreads, executor, selectorProvider, DefaultSelectStrategyFactory.INSTANCE);
    }    
    
    // NioEventLoopGroup.java
    public NioEventLoopGroup(int nThreads, Executor executor, final SelectorProvider selectorProvider,
                             final SelectStrategyFactory selectStrategyFactory) {
        super(nThreads, executor, selectorProvider, selectStrategyFactory, RejectedExecutionHandlers.reject());
    }    
    
    // MultithreadEventLoopGroup.java
    /**
     * @see MultithreadEventExecutorGroup#MultithreadEventExecutorGroup(int, Executor, Object...)
     */
    protected MultithreadEventLoopGroup(int nThreads, Executor executor, Object... args) {
        // DEFAULT_EVENT_LOOP_THREADS 默认为CPU核数的2倍
        super(nThreads == 0 ? DEFAULT_EVENT_LOOP_THREADS : nThreads, executor, args);
    }    
    
    // MultithreadEventExecutorGroup.java
    /**
     * Create a new instance.
     *
     * @param nThreads          the number of threads that will be used by this instance.
     * @param executor          the Executor to use, or {@code null} if the default should be used.
     * @param args              arguments which will passed to each {@link #newChild(Executor, Object...)} call
     */
    protected MultithreadEventExecutorGroup(int nThreads, Executor executor, Object... args) {
        this(nThreads, executor, DefaultEventExecutorChooserFactory.INSTANCE, args);
    }    
    
    // MultithreadEventExecutorGroup.java
    /**
     * Create a new instance.
     *
     * @param nThreads          the number of threads that will be used by this instance.
     * @param executor          the Executor to use, or {@code null} if the default should be used.
     * @param chooserFactory    the {@link EventExecutorChooserFactory} to use.
     * @param args              arguments which will passed to each {@link #newChild(Executor, Object...)} call
     */
    protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                                            EventExecutorChooserFactory chooserFactory, Object... args) {
        if (nThreads <= 0) { // 小于或等于零都会直接抛异常，由此可见，要想使用netty，还得必须至少得有1个线程跑起来才能使用
            throw new IllegalArgumentException(String.format("nThreads: %d (expected: > 0)", nThreads));
        }

        if (executor == null) { // 如果调用方不想自己定制线程池的话，那么则用netty自己默认的线程池
            executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
        }

        children = new EventExecutor[nThreads]; // 构建孩子结点数组，也就是构建NioEventLoopGroup持有的线程数组

        for (int i = 0; i < nThreads; i ++) { // 循环线程数，依次创建实例化线程封装的对象NioEventLoop
            boolean success = false;
            try {
                children[i] = newChild(executor, args); // 最终调用到了NioEventLoopGroup类中的newChild方法
                success = true;
            } catch (Exception e) {
                // TODO: Think about if this is a good exception type
                throw new IllegalStateException("failed to create a child event loop", e);
            } finally {
                if (!success) {
                    for (int j = 0; j < i; j ++) {
                        children[j].shutdownGracefully();
                    }

                    for (int j = 0; j < i; j ++) {
                        EventExecutor e = children[j];
                        try {
                            while (!e.isTerminated()) {
                                e.awaitTermination(Integer.MAX_VALUE, TimeUnit.SECONDS);
                            }
                        } catch (InterruptedException interrupted) {
                            // Let the caller handle the interruption.
                            Thread.currentThread().interrupt();
                            break;
                        }
                    }
                }
            }
        }

        // 实例化选择线程器，也就是说我们要想执行任务，对于nThreads个线程，我们得靠一个规则来如何选取哪个具体线程来执行任务；
        // 那么chooser就是来干这个事情的，它主要是帮我们选出需要执行任务的线程封装对象NioEventLoop
        chooser = chooserFactory.newChooser(children);

        final FutureListener<Object> terminationListener = new FutureListener<Object>() {
            @Override
            public void operationComplete(Future<Object> future) throws Exception {
                if (terminatedChildren.incrementAndGet() == children.length) {
                    terminationFuture.setSuccess(null);
                }
            }
        };

        for (EventExecutor e: children) {
            e.terminationFuture().addListener(terminationListener);
        }

        Set<EventExecutor> childrenSet = new LinkedHashSet<EventExecutor>(children.length);
        Collections.addAll(childrenSet, children);
        readonlyChildren = Collections.unmodifiableSet(childrenSet);
    }    
```

- 主要讲述了 `NioEventLoopGroup` 对象的实例化过程，这仅仅只是讲了一半，因为还有一半是实例化children[i]子线程组；
- 每个NioEventLoopGroup都配备了一个默认的线程池executor对象，而且同时也配备了一个选择线程器chooser对象
- 每个NioEventLoopGroup都一个子线程组children[i]，根据上层传入的参数来决定子线程数量，默认数量为CPU核数的2倍；

<https://segmentfault.com/a/1190000014103068#articleHeader0> 