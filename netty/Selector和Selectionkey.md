> **Selector(选择器)是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接**。

```java
java.nio.channels
public abstract class Selector extends Object implements Closeable
```

Nio的通信方式

![](http://47.101.139.24/upload/2019/4/201412131612103092019052020135624.png)

#### 使用 Selector

 仅用单个线程来处理多个Channels的好处是，只需要更少的新城来处理通道。事实上，可以只用一个线程处理所有的通道。

#### Selector的创建

通过调用 Selector.open()方法创建一个Selector.

```java
Selector selector = Selector.open();
```

- **isOpen()** —— 判断Selector是否处于打开状态。Selector对象创建后就处于打开状态了
- close() —— 当调用了Selector对象的close()方法，就进入关闭状态.。**用完Selector后调用其close()方法会关闭该Selector，且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭**

#### 向Selector注册通道

为了将Channel和Selector配合使用，必须将`channel` 注册到`selector`上。

通过`SelectableChannel.register()` 方法来实现。

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

**与Selector一起使用时，Channel必须处于非阻塞模式下。**

这意味着 FileChannel  与Selector 不能一起使用

注意 `register()` 方法 的第二个参数，这是一个 `interest集合`，意思是在通过Selector监听Channel时对什么事件感兴趣。

可以监听四种不同类型的事件：

- Connect
- Accept
- Read
- Write

通道触发了一个事件意思是该事件已经就绪。所以，某个channel成功连接到另一个服务器称为”**连接就绪**“。一个server socket channel准备号接收新进入的连接称为”**接收就绪**“。一个有数据可读的通道可以说是”**读就绪**“。等代写数据的通道可以说是”**写就绪**“。

这四种事件用`SelectionKey` 的四种常量来表示

- SelectionKey.OP_CONNECT
- SelectionKey.OP_ACCEPT
- SelectionKey.OP_READ
- SelectionKey.OP_WRITE

####  register()返回值 —— SelectionKey,  Selector中的SelectionKey集合

只要ServerSocketChannel及SocketChannel向Selector注册了特定的事件，**Selector就会监控这些事件是否发生**。

SelectableChannel的register()方法返回一个SelectionKey对象，**该对象是用于跟踪这些被注册事件的句柄。**

一个Selector对象会包含3种类型的SelectionKey集合：

- **all-keys集合** —— 当前所有向Selector注册的SelectionKey的集合，Selector的keys()方法返回该集合
- **selected-keys集合** —— 相关事件已经被Selector捕获的SelectionKey的集合，Selector的selectedKeys()方法返回该集合
- **cancelled-keys集合** —— 已经被取消的SelectionKey的集合，Selector没有提供访问这种集合的方法

当register()方法执行时，新建一个SelectioKey，并把它加入Selector的all-keys集合中。

**如果关闭了与SelectionKey对象关联的Channel对象，或者调用了SelectionKey对象的cancel方法，这个SelectionKey对象就会被加入到cancelled-keys集合中，表示这个SelectionKey对象已经被取消。**

在执行Selector的select()方法时，如果与SelectionKey相关的事件发生了，这个SelectionKey就被加入到selected-keys集合中，程序直接调用selected-keys集合的remove()方法，或者调用它的iterator的remove()方法，都可以从selected-keys集合中删除一个SelectionKey对象。

#### SelectionKey

**表示SelectableChannel 在 Selector 中的注册的标记/句柄。**

register()方法返回一个**SelectinKey对象**，**这个对象**包含一些你感兴趣的属性：

- **interest集合**
- **ready集合**
- **Channel**
- **Selector**
- **附加的对象**

通过调用某个`SelectionKey`的`cancel()`方法，**取消注册**，或者通过关闭其选择器来取消该Key，之前，它一直保持有效。

取消某个Key之后不会立即从Selector中移除它，相反，会将该Key添加到Selector的**已取消key set**，以便在下一次进行选择(**Select**)操作的时候移除它，移除以后，Selector就不在监听这个 Channel。

- **interest集合** —— 感兴趣的事件集合，可以通过SelectionKey读写interest集合，

```java
int interestSet = selectionKey.interestOps();
boolean isInterestedInAccept = (interestSet & Selection.OP_ACCEPT) == SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectioKey.OP_CONNECT;
boolean isInterestedInRead = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite = interestSet & SelectionKey.OP_WRITE;
```

-  **ready集合** —— 是通道已经准备就绪的操作的集合，在一个选择后，你会是首先访问这个ready set

```java
int readySet = selectionKey.readyOps();

// 可以向检测interet集合那样的方法，来检测channel中什么事件或操作已经就绪，也可以使用一下四个方法
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();可以向检测interet集合那样的方法，来检测channel中什么事件或操作已经就绪，也可以使用一下四个方法
```

- **从SelectionKey访问Channel和Selector：**

```java
Channel channel = selectionKey.channel();
Selector selector = selectionKey.selector();
```

- **附加的对象** —— 可以将一个对象或者更多的信息附着到SelectionKey上，这样就能方便的识别某个给定的通道。例如，可以附加与通道一起使用的Buffer，或是包含聚集数据的某个对象

```java
selectionKey.attach(theObject);
Object attachedObj = selectionKey.attachment();
```

### 通过Selector选择就绪的通道

一旦向Selector注册了一个或多个通道，就可以调用几个重载的select()方法。

这些方法返回你所感兴趣的事件（连接，接受，读或写）已经准备就绪的那些通道。换句话说，如果你对”读就绪“的通道感兴趣，select()方法会返回读事件已经就绪的那些通道。

- **select()** —— 阻塞到至少有一个通道在你注册的事件上就绪了
- **select(long timeout)** —— 和select()一样，除了最长会阻塞timeout毫秒
- **selectNow()** —— 不会阻塞，不管什么通道就绪都立刻返回；此方法执行非阻塞的选择操作，如果自从上一次选择操作后，没有通道变成可选择的，则此方法直接返回0
- **select()**方法返回的Int值表示多少通道就绪。

**一旦调用了select()方法，并且返回值表明有一个或更多个通道就绪了，然后可以通过调用selector的selectorKeys()方法，访问已选择键集中的就绪通道**，

```java

Set selectedKeys = selector.selectedKeys();
Iterator keyIterator = selectedKeys.iterator();
while(keyIterator.hasNext()){
    SelectionKey key = keyIterator.next();
    if (key.isAcceptable()){      // a connection was accepted by a ServerSocketChannel
 
    }else 
    if (key.isConnectable()){     // a connection was eatablished with a remote server
 
    }else
    if (key.isReadable()){        // a channel is ready for reading
 
    }else
    if (key.isWritable()){        // a channel is ready for writing
 
    }
 
    keyIterator.remove();
}

```

**注意每次迭代末尾的remove()调用，Selector不会自己从已选择集中移除SelectioKey实例，必须在处理完通道时自己移除**。因为 `Select()` 过程只会向 selected-keys集合 中添加 `key`。 

### Selector的wakeUp()方法

某个线程调用select()方法后阻塞了，即使没有通道已经就绪，也有办法让其从select()方法返回。**只要让其他线程在第一个线程调用select()方法的那个对象上调用Selector.wakeup()方法即可**。阻塞在select()方法上的线程会立马返回。