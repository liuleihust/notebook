## Proactor与Reactor

一般情况下，I/O 复用机制需要事件分发器（event dispatcher)。事件分发器的作用，即将那些读写事件源发给各读写事件的处理者。**开发人员在开始的时候需要在分发器那里注册感兴趣的事件，并提供相应的处理者，或者是回调函数；事件分发器在适当的时候，会将请求的事件分发给这些handler或者回调函数**。



涉及到事件分发器的两种模式称为：`Reactor` 和 `Proactor` Reactor 模式基于**同步IO** 的，而Proactor 模式是和**异步I/O**相关的。在Reactor模式中，事件分发器等待某个事件或者可应用或个操作的状态发生（比如文件描述符可读写，或者是socket可读写），事件分发器就把这个事件传给事先注册的事件处理函数或者回调函数，由后者来做实际的读写操作。 即事件分发器与主线程同一个线程。

而在Proactor模式中，事件处理者（或者代由事件分发器发起）**直接发起一个异步读写操作**（相当于请求），而实际的工作是由操作系统来完成的。发起时，需要提供的参数包括用于存放读到数据的缓存区、读的数据大小或用于存放外发数据的缓存区，以及这个请求完后的回调函数等信息。事件分发器得知了这个请求，它默默等待这个请求的完成，然后转发完成事件给相应的事件处理者或者回调。举例来说，在Windows上事件处理者投递了一个异步IO操作（称为overlapped技术），事件分发器等IO Complete事件完成。这种异步模式的典型实现是基于操作系统底层异步API的，所以我们可称之为“系统级别”的或者“真正意义上”的异步，因为具体的读写是由操作系统代劳的。这里 事件分发器 `eventloop` 是在后台运行的。

> ajax 异步原理
>
> todo

在Reactor 中实现读：

- 注册读就绪事件和相应的事件处理器。
- 事件分发器等待事件。
- 事件到来，激活分发器，分发器调用事件对应的处理器。
- 事件处理器完成实际的读操作，处理读到的数据，注册新的事件，然后返还控制权。

在Proactor中实现读：

- 处理器(**主线程，主执行流程**)发起异步读操作（注意：操作系统必须支持异步IO）。在这种情况下，处理器无视IO就绪事件，它关注的是完成事件。
- 事件分发器等待操作完成事件。
- 在分发器等待过程中，操作系统利用并行的内核线程执行实际的读操作，**并将结果数据存入用户自定义缓冲区**，最后通知事件分发器读操作完成。
- 事件分发器调用事件处理器（即回调）。
- 事件处理器处理用户自定义缓冲区中的数据，然后启动一个新的异步操作，并将控制权返回事件分发器。

可以看出，两个模式的相同点，都是对某个I/O 事件的事件通知（即告诉某个模块，这个I/O操作可以进行或已经完成）。在结构上，两者也有相同点：事件分发器负责提交IO操作（异步)、查询设备是否可操作（同步)，然后当条件满足时，就回调handler；不同点在于，异步情况下（Proactor)，当回调handler时，表示I/O操作已经完成；同步情况下（Reactor)，回调handler时，表示I/O设备可以进行某个操作（can read 或 can write)。



**同步与异步的区别即 事件分发器返回时读操作是否已经完成。**



将Reactor 原来位于事件处理器内的 Read/Write 操作移至分发器，以此寻求将Reactor 多路同步I/O转换为模拟异步I/O。

以读操作为例：

- 注册读就绪事件和相应的事件处理器。**并为分发器提供数据缓冲区地址，需要读取数据量等信息。**

- 分发器等待事件（如在select()上等待）。

- 事件到来，激活分发器。分发器执行一个非阻塞读操作（**它有完成这个操作所需的全部信息**），最后调用对应处理器。

- 事件处理器处理用户自定义缓冲区的数据，注册新的事件（当然同样要给出数据缓冲区地址，需要读取的数据量等信息），最后将控制权返还分发器。

  如我们所见，通过对多路I/O模式功能结构的改造，可将Reactor转化为Proactor模式。改造前后，**模型实际完成的工作量没有增加，只不过参与者间对工作职责稍加调换**。没有工作量的改变，自然不会造成性能的削弱。对如下各步骤的比较，可以证明工作量的恒定：

#### 标准/典型的Reactor：

- 步骤1：等待事件到来（Reactor负责）。
- 步骤2：将读就绪事件分发给用户定义的处理器（Reactor负责）。
- 步骤3：读数据（用户处理器负责）。
- 步骤4：处理数据（用户处理器负责）。

#### 改进实现的模拟Proactor：

- 步骤1：等待事件到来（Proactor负责）。

- 步骤2：得到读就绪事件，执行读数据（现在由Proactor负责）。

- 步骤3：将读完成事件分发给用户处理器（Proactor负责）。

- 步骤4：处理数据（用户处理器负责）。

  对于不提供异步I/O API的操作系统来说，这种办法可以隐藏Socket API的交互细节，从而对外暴露一个完整的异步接口。借此，我们就可以进一步构建完全可移植的，平台无关的，有通用对外接口的解决方案。

```java
 interface ChannelHandler{
      void channelReadable(Channel channel);
      void channelWritable(Channel channel);
   }
   class Channel{
     Socket socket;
     Event event;//读，写或者连接
   }

   //IO线程主循环:
   class IoThread extends Thread{
   public void run(){
   Channel channel;
   while(channel=Selector.select()){//选择就绪的事件和对应的连接
      if(channel.event==accept){
         registerNewChannelHandler(channel);//如果是新连接，则注册一个新的读写处理器
      }
      if(channel.event==write){
         getChannelHandler(channel).channelWritable(channel);//如果可以写，则执行写事件
      }
      if(channel.event==read){
          getChannelHandler(channel).channelReadable(channel);//如果可以读，则执行读事件
      }
    }
   }
   Map<Channel，ChannelHandler> handlerMap;//所有channel的对应事件处理器
  }
```

## Selector.wakeup()

### 主要作用

解除阻塞在Selector.select()/select(long)上的线程，立即返回。

两次成功的select之间多次调用wakeup等价于一次调用。

如果当前没有阻塞在select上，则本次wakeup调用将作用于下一次select——“记忆”作用。

为什么要唤醒？

注册了新的channel或者事件。

channel关闭，取消注册。

优先级更高的事件触发（如定时器事件），希望及时处理。

### 原理

Linux上利用pipe调用创建一个管道，Windows上则是一个loopback的tcp连接。这是因为win32的管道无法加入select的fd set，将管道或者TCP连接加入select fd set。

wakeup往管道或者连接写入一个字节，阻塞的select因为有I/O事件就绪，立即返回。可见，wakeup的调用开销不可忽视。

## Buffer的选择

通常情况下，操作系统的一次写操作分为两步：

1. 将数据从用户空间拷贝到系统空间。

2. 从系统空间往网卡写。

   同理，读操作也分为两步：
   ① 将数据从网卡拷贝到系统空间；
   ② 将数据从系统空间拷贝到用户空间。

对于NIO来说，缓存的使用可以使用DirectByteBuffer和HeapByteBuffer。如果使用了DirectByteBuffer，一般来说可以减少一次系统空间到用户空间的拷贝。但Buffer创建和销毁的成本更高，更不宜维护，通常会用内存池来提高性能。

> **Directbuffer**
>
> Java NIO中的direct buffer（主要是DirectByteBuffer）其实是分两部分的：
>
> ```java
> 		Java       |      native
>                    |
>  DirectByteBuffer  |     malloc'd
>  [    address   ] -+-> [   data    ]
>                    |
> ```
>
> 其中DirectByteBuffer 自身是一个 Java 对象，在 JAVA 堆中；而这个对象中有个long 类型 字段 address ，记录着一块调用 malloc() 申请到的 **native memory** 。
>
> DirectByteBuffer 自身是（Java）堆内的，它背后真正承载数据的buffer是在（Java）堆外——native memory中的。**这是 malloc() 分配出来的内存，是用户态的**。
>
> >  FileChannel 的read(ByteBuffer dst)函数,write(ByteBuffer src)函数中，如果传入的参数是HeapBuffer类型,则会临时申请一块DirectBuffer,进行数据拷贝，而不是直接进行数据传输，这是出于什么原因？
>
> 这个其实就是迁就 OpenJDK里的 HotSpot VM 的一点实现细节
>
> HotSpot VM 里的GC 除了CMS 之外都是需要移动对象的，是所谓的“compacting GC”。
>
> 如果要把一个Java里的 byte[] 对象的引用传给native代码，让native代码直接访问数组的内容的话，就必须要保证native代码在访问的时候这个 byte[] 对象不能被移动，也就是要被“pin”（钉）住。
>
> 可惜HotSpot VM出于一些取舍而决定不实现单个对象层面的object pinning，要pin的话就得暂时禁用GC——也就等于把整个Java堆都给pin住。HotSpot VM对JNI的Critical系API就是这样实现的。这用起来就不那么顺手。
>
> 所以 Oracle/Sun JDK / OpenJDK 的这个地方就用了点绕弯的做法。**它假设把 HeapByteBuffer 背后的 byte[] 里的内容拷贝一次是一个时间开销可以接受的操作，同时假设真正的I/O可能是一个很慢的操作。**
>
> 于是它就先把 HeapByteBuffer 背后的 byte[] 的内容拷贝到一个 DirectByteBuffer 背后的native memory去，这个拷贝会涉及 sun.misc.Unsafe.copyMemory() 的调用，背后是类似 memcpy() 的实现。这个操作本质上是会在整个拷贝过程中暂时不允许发生GC的，虽然实现方式跟JNI的Critical系API不太一样。（具体来说是 Unsafe.copyMemory() 是HotSpot VM的一个intrinsic方法，中间没有safepoint所以GC无法发生）。
>
> 然后数据被拷贝到native memory之后就好办了，就去做真正的I/O，**把 DirectByteBuffer 背后的native memory地址传给真正做I/O的函数。这边就不需要再去访问Java对象去读写要做I/O的数据了。**
>
> **read/write 调用的是文件系统的读写接口，读写需要经过文件系统的缓存（读写文件的时候需要经过文件系统的缓存-内核缓存），directbuffer 只是少了一步从 heap 内存拷贝到nativememory**。
>
> `mmap` ：将磁盘文件映射到进程的虚拟地址空间，读取的时候会发生缺页中断，会将磁盘中的文件读到物理内存中，缺少一次 内核缓存的复制。
>
> 
>
> ### 零拷贝
>
> 为什么需要零拷贝
>
> ![img](https://pic4.zhimg.com/80/v2-28027232e465dd5ee168eeb6da232bdb_hd.jpg)
>
> 从上图中可以看出，共产生了四次数据拷贝，即使使用了DMA来处理了与硬件的通讯，CPU仍然需要处理两次数据拷贝，与此同时，在用户态与内核态也发生了多次上下文切换，无疑也加重了CPU负担。
>
> 在此过程中，我们没有对文件内容做任何修改，那么在内核空间和用户空间来回拷贝数据无疑就是一种浪费，而零拷贝主要就是为了解决这种低效性。
>
> 概括来说，原因为以下两点：
>
> 1. 占用了较多的 CPU 时间；
> 2. 占用了较多的内存空间。
>
> 既然传统的 I/O 操作有性能问题，那么如何改进呢？我们将其定义为“**零拷贝技术**”。
>
> 事实上零拷贝主要指的是**避免数据拷贝，而非没有拷贝**。大致概括如下：
>
> - 避免操作系统内核缓冲区之间进行数据拷贝操作。
> - 避免操作系统内核和用户应用程序地址空间这两者之间进行数据拷贝操作。
> - 用户应用程序可以避开操作系统直接访问硬件存储。
> - 数据传输尽量让 DMA 来做。
> - 避免不必要的系统调用和上下文切换。
> - 需要拷贝的数据可以先被缓存起来。
> - 对数据进行处理尽量让硬件来做。
>
> 零拷贝分类：
>
> - 直接IO
>
> 应用程序直接访问硬件存储，操作系统内核只是辅助数据传输。
>
> ![img](https://pic2.zhimg.com/80/v2-0c8490cf98d7922379875b82a373436d_hd.jpg)
>
> **避免内核地址空间的缓冲区和用户应用程序地址空间的缓冲区之间的拷贝**
>
> **mmap()**
>
> 应用程序调用了 mmap() 之后，数据会先通过 DMA 拷贝到操作系统内核的缓冲区中去。**接着，应用程序跟操作系统共享这个缓冲区**，这样，操作系统内核和应用程序存储空间就不需要再进行任何的数据拷贝操作。
>
> ![img](https://pic2.zhimg.com/80/v2-8029f18b02bf537b08ff7bd95ad91ce5_hd.jpg)
>
> ### **sendfile()** 
>
> 利用 DMA 引擎将文件中的数据拷贝到操作系统内核缓冲区中，然后数据被拷贝到与 socket 相关的内核缓冲区中去。接下来，DMA 引擎将数据从内核 socket 缓冲区中拷贝到协议引擎中去。
>
> ![img](https://pic4.zhimg.com/80/v2-d2ee7c46d910c62d3d1b0be64f3f0e57_hd.jpg)
>
> 这里还是有一次 CPU 拷贝，怎么把这个也省掉呢？继续看——
>
> ### **带有 DMA 收集拷贝功能的 sendfile()**
>
> 之前我们是把页缓存的数据拷贝到socket缓存中，实际上，我们仅仅需要把缓冲区描述符传到 socket 缓冲区，再把数据长度传过去，这样 DMA 控制器直接将页缓存中的数据打包发送到网络中就可以了。
>
> ![img](https://pic1.zhimg.com/80/v2-244852829e9b22802b2004a22bb7eb18_hd.jpg)
>
> 上图总结为以下 3 步：
>
> 1. DMA 从拷贝至内核缓冲区
> 2. 将数据的位置和长度的信息的描述符增加至内核空间（socket 缓冲区）
> 3. DMA 将数据从内核拷贝至协议引擎

如果数据量比较小的中小应用情况下，可以考虑使用heapBuffer；反之可以用directBuffer。

# NIO存在的问题

使用NIO != 高性能，当连接数<1000，并发程度不高或者局域网环境下NIO并没有显著的性能优势。

NIO并没有完全屏蔽平台差异，它仍然是基于各个操作系统的I/O系统实现的，差异仍然存在。使用NIO做网络编程构建事件驱动模型并不容易，陷阱重重。

推荐大家使用成熟的NIO框架，如Netty，MINA等。解决了很多NIO的陷阱，并屏蔽了操作系统的差异，有较好的性能和编程模型。