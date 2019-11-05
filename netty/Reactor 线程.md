> netty最核心的就是reactor线程，对应项目中使用广泛的**NioEventLoop**，那么NioEventLoop里面到底在干些什么事？netty是如何保证事件循环的**高效轮询**和**任务的及时执行**？又是如何来优雅地fix掉jdk的nio bug？带着这些疑问，本篇文章将庖丁解牛，带你逐步了解netty reactor线程的真相[源码基于4.1.36.Final。 

##  reactor 线程的启动

NioEventLoop 的run 方法是 reactor  线程的主体，在第一次添加任务的时候被启动

> NioEventLoop  是  SingleThreadEventLoop 的 实现，SingleThreadEventLoop  将 Channel 注册到 Selector，
>
> 实现了 IO 多路复用。

![](http://47.101.139.24/upload/2019/4/360截图177608091101131192019052010082535.jpg)

<p align="center">NioEventLoopGroup 的继承图</p>



![](http://47.101.139.24/upload/2019/4/155832028503520190520111426536.png)

<p align="center">NioEventLoop 的继承图</p>

[java 中用Executor 代替Thread 的四大理由](<https://www.imooc.com/article/34127>)

- **Executor**:  只有一个 executor 方法，拥有执行任务，实现方式可以为 固定线程数量的线程池，也可以为 一个任务一个线程的形式。
- **EventExecutorGroup**：类似于 ThreadGroup的概念吧，管理`EventExecutor`。
- **EventExecutor**:继承了`ScheduledExecutorService`,具有时间顺序执行的方法。额外添加了 `inEventLoop` 方法，判断给定线程是否在 `EventLoop` 中执行。
- **EventLoop**：相当于 Reactor 线程。NioEventLoop 是单线程Reactor。 
- **EventLoop**: 允许将Channel 注册到 EventLoop上

Netty 的线程模型

![](https://upload-images.jianshu.io/upload_images/5249993-a67abc1374958c5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

其中 NioEventLoop 是单线程Reactor 模式。


添加任务的执行函数：

```java
/*
SingleThreadEventExecutor.java
*/

	// 实现了 Executor 接口中的 execute 方法,提交任务
	@Override
    public void execute(Runnable task) {
        if (task == null) {
            throw new NullPointerException("task");
        }

        boolean inEventLoop = inEventLoop();
        addTask(task);
        // 判断当前线程是否为 reactor 线程，如果当前线程不是   reactor 线程，则启动线程
        if (!inEventLoop) {
            startThread();
		    ...
            }
        }
```

外部线程在往任务队列里面添加任务的执行 startThread()  ，netty 会判断reactor 线程有没有被启动（**inEventLoop**），如果没有被启动，那就启动线程。

```java
/*
SingleThreadEventExecutor.java
*/

    private void startThread() {
        if (state == ST_NOT_STARTED) {
            if (STATE_UPDATER.compareAndSet(this, ST_NOT_STARTED, ST_STARTED)) {
                try {
                    // 关键函数
                    doStartThread();
                } catch (Throwable cause) {
               		...
        }
    }
```

SingleThreadEventExecutor 在执行`doStartThread`的时候，会调用内部执行器`executor`（**线程池**）的execute方法，将调用`NioEventLoop`的`run`方法的过程封装成一个runnable塞到一个线程中去执行 

```java
/*
SingleThreadEventExecutor.java
*/

private void doStartThread() {
        assert thread == null;
      // 每次执行 executor.execute 方法都会创建一个新的线程
        executor.execute(new Runnable() {
            @Override
            public void run() {
                // 将当前线程当做 EventLoop 线程
                thread = Thread.currentThread();
                if (interrupted) {
                    thread.interrupt();
                }

                boolean success = false;
                updateLastExecutionTime();
                try {
                    SingleThreadEventExecutor.this.run();
				   ...
    }
```

该线程就是`executor`创建，对应netty的reactor线程实体。`executor` 默认是`ThreadPerTaskExecutor`

默认情况下，`ThreadPerTaskExecutor` 在每次执行`execute` 方法的时候都会通过`DefaultThreadFactory`创建一个`FastThreadLocalThread`线程，**而这个线程就是netty中的reactor线程实体**。**它执行的任务就是 NioEventLoop 的 run 方法。**

#### ThreadPerTaskExecutor

```java
public void execute(Runnable command) {
    // 每次执行一个新的 线程
    threadFactory.newThread(command).start();
}
```

关于为啥是`ThreadPerTaskExecutor` 和 `DefaultThreadFactory`的组合来new 一个 `FastThreadLocalThread`。

因为标准 netty 程序会调用 `NioEventLoopGroup`的父类`MultithreadEventExecutorGroup`

```java
// 一般程序调用这个
EventLoopGroup workerGroup = new NioEventLoopGroup();

//NioEventLoopGroup的构造方法又会调用其父类的构造方法，最后会执行
protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                                        EventExecutorChooserFactory chooserFactory, Object... args) {
    if (executor == null) {
        // 默认 executor 为 ThreadPerTaskExecutor 和 DefaultThreadFactory
        executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
    }
}
```

然后通过`newChild`  的方式传递给 `NioEventLoop`

```java
@Override
protected EventLoop newChild(Executor executor, Object... args) throws Exception {
    return new NioEventLoop(this, executor, (SelectorProvider) args[0],
        ((SelectStrategyFactory) args[1]).newSelectStrategy(), (RejectedExecutionHandler) args[2]);
}
```

## reactor 线程的执行

下面重点剖析一下  NioEventLoop 的  `run` 方法。

```java
@Override
protected void run() {
    for (;;) {
        try {
            switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
                case SelectStrategy.CONTINUE:
                    continue;
                case SelectStrategy.SELECT:
                    select(wakenUp.getAndSet(false));
                    if (wakenUp.get()) {
                        selector.wakeup();
                    }
                default:
                    // fallthrough
            }
            processSelectedKeys();
            runAllTasks(...);
            }
        } catch (Throwable t) {
            handleLoopException(t);
        }
        ...
    }
```

reactor 线程所做的事情其实很简单，用一幅图就可以说明

![](http://47.101.139.24/upload/2019/4/1357217-67ed6d1e8070426f20190520164942772.png)

reactor 线程大概做的事情分为 三个步骤不断循环

- 首先轮询注册到 reacor 线程的`selector `上的所有`channel `的 IO 事件

```java
select(wakenUp.getAndSet(false));
if (wakenUp.get()) {
    selector.wakeup();
}
```

- 处理产生网络IO 事件的channel ，进行IO 操作，将数据从网卡读到内存，并产生一些任务。

```java
processSelectedKeys();
```

- 处理任务队列

```java
runAllTasks(...);
```

下面对每个步骤详细说明

### select操作

```java
select(wakenUp.getAndSet(false));
if (wakenUp.get()) {
      selector.wakeup();
}
```

`wakenUp` 表示是否应该唤醒正在阻塞的select操作，可以看到netty在进行一次新的loop之前，都会将`wakeUp` 被设置成false，标志新的一轮loop的开始，具体的select操作我们也拆分开来看

> 1. 定时任务截止时时间快到了，中断本次轮询。

```java
int selectCnt = 0;
long currentTimeNanos = System.nanoTime();
// delayNanos（）方法  返回 第一个定时任务的延迟时间，selectDeadLineNanos 等于执行当前任务的时间
long selectDeadLineNanos = currentTimeNanos + delayNanos(currentTimeNanos);

for (;;) {
    // 当timeoutMillis <=0 时，表示当前定时任务队列中有任务的截止时间快到了（<0.5ms）,就跳出循环，如果没有进行过select操作，就调用一次selectNow（），该方法立即返回，不会阻塞。
    long timeoutMillis = (selectDeadLineNanos - currentTimeNanos + 500000L) / 1000000L;
    if (timeoutMillis <= 0) {
        if (selectCnt == 0) {
            selector.selectNow();
            selectCnt = 1;
        }
        break;
    }
    ....
}
```

> 2.轮询过程中发现有任务加入，中断本次轮询

```java
for (;;) {
    // 1.定时任务截至事时间快到了，中断本次轮询
    ...
    // 2.轮询过程中发现有任务加入，中断本次轮询
    /*
    如果当 wakenUp的 value 的值为 true时，这个任务没有机会去调用 Selector#wakeup。因此我们需要在执行 select 操作之前再次检查任务队列。如果我们不这样做的话，任务将一直暂停知道 select 操作完成，
    */
    if (hasTasks() && wakenUp.compareAndSet(false, true)) {
        selector.selectNow();
        selectCnt = 1;
        break;
    }
    ....
}
```
netty为了保证任务队列能够及时执行，在进行阻塞select操作的时候会判断任务队列是否为空，如果不为空，就执行一次非阻塞select操作，跳出循环
> 3.阻塞式select操作

```java
for (;;) {
    // 1.定时任务截至事时间快到了，中断本次轮询
    ...
    // 2.轮询过程中发现有任务加入，中断本次轮询
    ...
    // 3.阻塞式select操作
    int selectedKeys = selector.select(timeoutMillis);
    selectCnt ++;
    if (selectedKeys != 0 || oldWakenUp || wakenUp.get() || hasTasks() || hasScheduledTasks()) {
        break;
    }
    ....
}
```

执行到这一步，说明netty任务队列里面队列为空，并且所有定时任务延迟时间还未到(大于0.5ms)，于是，在这里进行一次阻塞select操作，**截止到第一个定时任务的截止时间**

这里，我们可以问自己一个问题，如果第一个定时任务的延迟非常长，比如一个小时，那么有没有可能线程一直阻塞在select操作，当然有可能！But，**只要在这段时间内，有新任务加入，该阻塞就会被释放**。

> 外部线程调用Execute方法添加任务

```java
/*
SingleThreadEventExecutor.java
*/

@Override
public void execute(Runnable task) { 
    ...
    wakeup(inEventLoop); // inEventLoop为false
    ...
}

```

```java
/*
NioEventLoop.java
*/

    @Override
    protected void wakeup(boolean inEventLoop) {
        if (!inEventLoop && wakenUp.compareAndSet(false, true)) {
            selector.wakeup();
        }
    }

```

可以看到，在**外部线程** 添加任务的时候，会调用`wakeup` 方法来唤醒 `selector.select(timeoutMillis)`

阻塞select操作结束之后，netty又做了一系列的状态判断来决定是否中断本次轮询，中断本次轮询的条件有

- 轮询到IO事件 （`selectedKeys != 0`）
- oldWakenUp 参数为true
- 任务队列里面有任务（`hasTasks`）
- 第一个定时任务即将要被执行 （`hasScheduledTasks（）`）
- 用户主动唤醒（`wakenUp.get()`）

> 4.解决jdk的nio bug

该bug会导致Selector一直空轮询，最终导致cpu 100%，nio server不可用，严格意义上来说，netty没有解决jdk的bug，而是通过一种方式来巧妙地避开了这个bug，具体做法如下



```java
long currentTimeNanos = System.nanoTime();
for (;;) {
    // 1.定时任务截止事时间快到了，中断本次轮询
    ...
    // 2.轮询过程中发现有任务加入，中断本次轮询
    ...
    // 3.阻塞式select操作
    selector.select(timeoutMillis);
    // 4.解决jdk的nio bug
    long time = System.nanoTime();
    if (time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) >= currentTimeNanos) {
        selectCnt = 1;
    } else if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
            selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {

        rebuildSelector();
        selector = this.selector;
        selector.selectNow();
        selectCnt = 1;
        break;
    }
    currentTimeNanos = time; 
    ...
 }
```

netty 会在每次进行 `selector.select(timeoutMillis)` 之前记录一下开始时间`currentTimeNanos`，在select之后记录一下结束时间，判断select操作是否至少持续了`timeoutMillis`秒（这里将`time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) >= currentTimeNanos`改成`time - currentTimeNanos >= TimeUnit.MILLISECONDS.toNanos(timeoutMillis)`或许更好理解一些）,
 如果持续的时间大于等于timeoutMillis，说明就是一次有效的轮询，重置`selectCnt`标志，否则，表明该阻塞方法并没有阻塞这么长时间，可能触发了jdk的空轮询bug，当空轮询的次数超过一个阀值的时候，默认是512，就开始重建selector

空轮询阀值相关的设置代码如下

```java
int selectorAutoRebuildThreshold = SystemPropertyUtil.getInt("io.netty.selectorAutoRebuildThreshold", 512);
if (selectorAutoRebuildThreshold < MIN_PREMATURE_SELECTOR_RETURNS) {
    selectorAutoRebuildThreshold = 0;
}
SELECTOR_AUTO_REBUILD_THRESHOLD = selectorAutoRebuildThreshold;
```

下面我们简单描述一下netty 通过`rebuildSelector`来fix空轮询bug的过程，`rebuildSelector`的操作其实很简单：new一个新的selector，将之前注册到老的selector上的的channel重新转移到新的selector上。我们抽取完主要代码之后的骨架如下

```java
public void rebuildSelector() {
    final Selector oldSelector = selector;
    final Selector newSelector;
    newSelector = openSelector();

    int nChannels = 0;
     try {
        for (;;) {
                for (SelectionKey key: oldSelector.keys()) {
                    // 获得附加信息
                    Object a = key.attachment();
                     if (!key.isValid() || key.channel().keyFor(newSelector) != null) {
                         continue;
                     }
                     int interestOps = key.interestOps();
                     key.cancel();
                     SelectionKey newKey = key.channel().register(newSelector, interestOps, a);
                     if (a instanceof AbstractNioChannel) {
                         ((AbstractNioChannel) a).selectionKey = newKey;
                      }
                     nChannels ++;
                }
                break;
        }
    } catch (ConcurrentModificationException e) {
        // Probably due to concurrent modification of the key set.
        continue;
    }
    selector = newSelector;
    oldSelector.close();
}
```

首先，通过`openSelector()`方法创建一个新的selector，然后执行一个死循环，只要执行过程中出现过一次并发修改selectionKeys异常，就重新开始转移

具体的转移步骤为

1. 拿到有效的key
2. 取消该key在旧的selector上的事件注册
3. 将该key对应的channel注册到新的selector上
4. 重新绑定channel和新的key的关系

转移完成之后，就可以将原有的selector废弃，后面所有的轮询都是在新的selector进行

最后，我们总结reactor线程select步骤做的事情：不断地轮询是否有IO事件发生，并且在轮询的过程中不断检查是否有定时任务和普通任务，保证了netty的任务队列中的任务得到有效执行，轮询过程顺带用一个计数器避开了了jdk空轮询的bug，过程清晰明了

### processSelectedKeys(处理 IO 事件)

```java
/*
processSelectedKeys,java
*/
private void processSelectedKeys() {
    if (selectedKeys != null) {
        processSelectedKeysOptimized(selectedKeys.flip());
    } else {
        processSelectedKeysPlain(selector.selectedKeys());
    }
}
```

我们发现处理IO事件，netty有两种选择，从名字上看，一种是处理优化过的selectedKeys，一种是正常的处理

我们对优化过的selectedKeys的处理稍微展开一下，看看netty是如何优化的，我们查看 `selectedKeys` 被引用过的地方，有如下代码

```java
/*
processSelectedKeys,java
*/
private SelectedSelectionKeySet selectedKeys;

private Selector NioEventLoop.openSelector() {
    //...
    final SelectedSelectionKeySet selectedKeySet = new SelectedSelectionKeySet();
    // selectorImplClass -> sun.nio.ch.SelectorImpl
    Field selectedKeysField = selectorImplClass.getDeclaredField("selectedKeys");
    Field publicSelectedKeysField = selectorImplClass.getDeclaredField("publicSelectedKeys");
    selectedKeysField.setAccessible(true);
    publicSelectedKeysField.setAccessible(true);
    selectedKeysField.set(selector, selectedKeySet);
    publicSelectedKeysField.set(selector, selectedKeySet);
    //...
    selectedKeys = selectedKeySet;
}
```

首先，selectedKeys是一个 `SelectedSelectionKeySet` 类对象，在`NioEventLoop` 的 `openSelector` 方法中创建，之后就通过反射将selectedKeys与 `sun.nio.ch.SelectorImpl`中的两个field绑定。

`sun.nio.ch.SelectorImpl` 中我们可以看到，这两个field其实是两个HashSet

```java
// Public views of the key sets
private Set<SelectionKey> publicKeys;             // Immutable
private Set<SelectionKey> publicSelectedKeys;     // Removal allowed, but not addition
protected SelectorImpl(SelectorProvider sp) {
    super(sp);
    keys = new HashSet<SelectionKey>();
    selectedKeys = new HashSet<SelectionKey>();
    if (Util.atBugLevel("1.4")) {
        publicKeys = keys;
        publicSelectedKeys = selectedKeys;
    } else {
        publicKeys = Collections.unmodifiableSet(keys);
        publicSelectedKeys = Util.ungrowableSet(selectedKeys);
    }
}
```

**selector在调用`select()`族方法的时候，如果有IO事件发生，就会往里面的两个field中塞相应的`selectionKey**`(具体怎么塞有待研究)，即相当于往一个hashSet中add元素，既然netty通过反射将jdk中的两个field替换掉，那我们就应该意识到是不是netty自定义的`SelectedSelectionKeySet`在`add`方法做了某些优化呢？

```java
final class SelectedSelectionKeySet extends AbstractSet<SelectionKey> {

    private SelectionKey[] keysA;
    private int keysASize;
    private SelectionKey[] keysB;
    private int keysBSize;
    private boolean isA = true;

    SelectedSelectionKeySet() {
        keysA = new SelectionKey[1024];
        keysB = keysA.clone();
    }

    @Override
    public boolean add(SelectionKey o) {
        if (o == null) {
            return false;
        }

        if (isA) {
            int size = keysASize;
            keysA[size ++] = o;
            keysASize = size;
            if (size == keysA.length) {
                doubleCapacityA();
            }
        } else {
            int size = keysBSize;
            keysB[size ++] = o;
            keysBSize = size;
            if (size == keysB.length) {
                doubleCapacityB();
            }
        }

        return true;
    }

    private void doubleCapacityA() {
        SelectionKey[] newKeysA = new SelectionKey[keysA.length << 1];
        System.arraycopy(keysA, 0, newKeysA, 0, keysASize);
        keysA = newKeysA;
    }

    private void doubleCapacityB() {
        SelectionKey[] newKeysB = new SelectionKey[keysB.length << 1];
        System.arraycopy(keysB, 0, newKeysB, 0, keysBSize);
        keysB = newKeysB;
    }

    SelectionKey[] flip() {
        if (isA) {
            isA = false;
            keysA[keysASize] = null;
            keysBSize = 0;
            return keysA;
        } else {
            isA = true;
            keysB[keysBSize] = null;
            keysASize = 0;
            return keysB;
        }
    }

    @Override
    public int size() {
        if (isA) {
            return keysASize;
        } else {
            return keysBSize;
        }
    }

    @Override
    public boolean remove(Object o) {
        return false;
    }

    @Override
    public boolean contains(Object o) {
        return false;
    }

    @Override
    public Iterator<SelectionKey> iterator() {
        throw new UnsupportedOperationException();
    }
}
```

该类继承了`AbstractSet `, 使用数组代替 `hashset` ，可以提高 add 方法的速度。

关于netty对`SelectionKeySet`的优化我们暂时就跟这么多，下面我们继续跟netty对IO事件的处理，转到`processSelectedKeysOptimized`

```java
private void processSelectedKeysOptimized() {
        for (int i = 0; i < selectedKeys.size; ++i) {
            final SelectionKey k = selectedKeys.keys[i];
            // null out entry in the array to allow to have it GC'ed once the Channel close
            // See https://github.com/netty/netty/issues/2363
            selectedKeys.keys[i] = null;

            final Object a = k.attachment();

            if (a instanceof AbstractNioChannel) {
                processSelectedKey(k, (AbstractNioChannel) a);
            } else {
                @SuppressWarnings("unchecked")
                NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
                processSelectedKey(k, task);
            }

            if (needsToSelectAgain) {
                // null out entries in the array to allow to have it GC'ed once the Channel close
                // See https://github.com/netty/netty/issues/2363
                selectedKeys.reset(i + 1);

                selectAgain();
                i = -1;
            }
        }
    }
```

我们将该过程分为以下三个步骤

1. 取出IO 事件对应的 netty channel 类

这里其实也能体会到优化过的 `SelectedSelectionKeySet` 的好处，遍历的时候遍历的是数组，相对jdk原生的`HashSet`效率有所提高。

拿到当前SelectionKey之后，将`selectedKeys[i]`置为null，这里简单解释一下这么做的理由：想象一下这种场景，假设一个NioEventLoop平均每次轮询出N个IO事件，高峰期轮询出3*N个事件，那么selectedKeys的物理长度要大于等于3*N，如果每次处理这些key，不置`selectedKeys[i]`为空，那么高峰期一过，这些保存在数组尾部的`selectedKeys[i]`对应的`SelectionKey`将一直无法被回收，`SelectionKey`对应的对象可能不大，但是要知道，它可是有attachment的，这里的attachment具体是什么下面会讲到，但是有一点我们必须清楚，attachment可能很大，这样一来，这些元素是GC root可达的，很容易造成gc不掉，内存泄漏就发生了

2. 处理该Channel

拿到对应的attachment 之后，netty 做了如下判断

```java
if (a instanceof AbstractNioChannel) {
    processSelectedKey(k, (AbstractNioChannel) a);
} 
```

源码读到这，我们需要思考为啥会有这么一条判断，凭什么说attachment可能会是 `AbstractNioChannel`对象？

我们的思路应该是找到底层selector, 然后在selector调用register方法的时候，看一下注册到selector上的对象到底是什么鬼，我们使用intellij的全局搜索引用功能，最终在 `AbstractNioChannel`中搜索到如下方法

```java
protected void doRegister() throws Exception {
    // ...
    selectionKey = javaChannel().register(eventLoop().selector, 0, this);
    // ...
}
```

`javaChannel()` 返回netty类`AbstractChannel`对应的jdk底层channel对象

```java
protected SelectableChannel javaChannel() {
    return ch;
}
```

我们查看到SelectableChannel方法，结合netty的 `doRegister()` 方法，我们不难推论出，netty的轮询注册机制其实是将`AbstractNioChannel`内部的jdk类`SelectableChannel`对象注册到jdk类`Selctor`对象上去，并且将`AbstractNioChannel`作为`SelectableChannel`对象的一个attachment附属上，这样再jdk轮询出某条`SelectableChannel`有IO事件发生时，就可以直接取出`AbstractNioChannel`进行后续操作.

**接下来对 io事件进行处理的关键函数**

```java
/*
NioEventLoop.java
*/

private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
        final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
        try {
            int readyOps = k.readyOps();
            if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
                int ops = k.interestOps();
                ops &= ~SelectionKey.OP_CONNECT;
                k.interestOps(ops);
                unsafe.finishConnect();
            }

            if ((readyOps & SelectionKey.OP_WRITE) != 0) {
                ch.unsafe().forceFlush();
            }

            // Also check for readOps of 0 to workaround possible JDK bug which may otherwise lead
            // to a spin loop
            if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
                unsafe.read();
            }
        } catch (CancelledKeyException ignored) {
            unsafe.close(unsafe.voidPromise());
        }
    }
```



1. 对于boss NioEventLoop来说，轮询到的是基本上就是连接事件，后续的事情就通过他的pipeline将连接扔给一个worker NioEventLoop处理
2. 对于worker NioEventLoop来说，轮询到的基本上都是io读写事件，后续的事情就是通过他的pipeline将读取到的字节流传递给每个channelHandler来处理。

### runtask

netty 是异步 task 机制，定时任务的处理逻辑。  用户在 提交 task后，在此处执行 task。

#### netty 中的 task 的常见使用场景

一.  用户自定义普通任务

```java
// reactor 线程 提交 task
ctx.channel().eventLoop().execute(new Runnable() {
    @Override
    public void run() {
        //...
    }
});
```

我们跟进`execute`方法，看重点

```java
@Override
public void execute(Runnable task) {
    //...
    addTask(task);
    //...
}
```

然后调用`offerTask`方法，如果offer失败，那就调用`reject`方法，通过默认的 `RejectedExecutionHandler` 直接抛出异常

```java
final boolean offerTask(Runnable task) {
    // ...
    return taskQueue.offer(task);
}
```

跟到`offerTask`方法，基本上task就落地了，netty内部使用一个`taskQueue`将task保存起来，那么这个`taskQueue`又是何方神圣？

我们查看 `taskQueue` 定义的地方和被初始化的地方

```java
private final Queue<Runnable> taskQueue;


taskQueue = newTaskQueue(this.maxPendingTasks);

@Override
protected Queue<Runnable> newTaskQueue(int maxPendingTasks) {
    // This event loop never calls takeTask()
    return PlatformDependent.newMpscQueue(maxPendingTasks);
}
```

我们发现 `taskQueue`在NioEventLoop中默认是mpsc队列，mpsc队列，即多生产者单消费者队列，netty使用mpsc，方便的将外部线程的task聚集，在reactor线程内部用单线程来串行执行，我们可以借鉴netty的任务执行模式来处理类似多线程数据上报，定时聚合的应用

在本节讨论的任务场景中，所有代码的执行都是在reactor线程中的，所以，所有调用 `inEventLoop()` 的地方都返回true，既然都是在reactor线程中执行，那么其实这里的mpsc队列其实没有发挥真正的作用(因为 reactor 线程是多线程的)，mpsc大显身手的地方其实在第二种场景

二. 非当前reactor线程调用channel的各种方法

```java
// non reactor thread
channel.write(...)
```

上面一种情况在push系统中比较常见，一般在**业务线程**里面，根据用户的标识，找到对应的channel引用，然后调用write类方法向该用户推送消息，就会进入到这种场景

最终write方法串至以下方法

```java
/*
AbstractChannelHandlerContext.java
*/
private void write(Object msg, boolean flush, ChannelPromise promise) {
    // ...
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
       ...
    } else {
        AbstractWriteTask task;
        if (flush) {
            task = WriteAndFlushTask.newInstance(next, m, promise);
        }  else {
            task = WriteTask.newInstance(next, m, promise);
        }
        safeExecute(executor, task, promise, m);
    }
}
```

外部线程在调用`write`的时候，`executor.inEventLoop()`会返回false，直接进入到else分支，将write封装成一个`WriteTask`（这里仅仅是write而没有flush，因此`flush`参数为false）, 然后调用 `safeExecute`方法

```java
private static void safeExecute(EventExecutor executor, Runnable runnable, ChannelPromise promise, Object msg) {
    // channel 绑定了 eventloop， 此处 将 任务 提交给 channel 所注册到的 eventloop 上。
    executor.execute(runnable);
    // ...
}
```

用户线程可能会有很多个，显然多个线程并发写`taskQueue`可能出现线程同步问题，于是，这种场景下，netty的mpsc queue就有了用武之地

> # **mpsc queue  无锁队列**
>
> 适用于 单消费者多生产者场景（A lock-free concurrent single-consumer multi-producer Queue）
>
> ### 数据结构
>
> - **Node**：声明了的next, volatile型, 还有AtomicReferenceFieldUpdater对next进行修改 
> - **DefaultNode**: Node的实现类, 声明了value 
> - **Ref**: 分为TailRef和HeadRef, 分别声明了volatile的tailRef和headRef引用…还有各自的AtomicReferenceFieldUpdater
>
> ## 实现
>
> **offer** 提交
>
> ```java
> public boolean offer(E value) {
>         if (value == null) {
>             throw new NullPointerException("value");
>         }
> 
>         final MpscLinkedQueueNode<E> newTail;
>     	// 生成新的节点
>         if (value instanceof MpscLinkedQueueNode) {
>             newTail = (MpscLinkedQueueNode<E>) value;
>             newTail.setNext(null);
>         } else {
>             newTail = new DefaultNode<E>(value);
>         }
> 		//  原子更新 tailref ，并返回  old  tailref 。 因为 getAndSet 是原子的，所以不需要锁
>         MpscLinkedQueueNode<E> oldTail = getAndSetTailRef(newTail);
>         oldTail.setNext(newTail);
>         return true;
>     }
> ```
>
> ![](https://img-blog.csdn.net/20170216105209046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGluZ2d1b2hhbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
> poll 取task
>
> ```java
> private MpscLinkedQueueNode<E> peekNode() {
>         MpscLinkedQueueNode<E> head = headRef();
>         MpscLinkedQueueNode<E> next = head.next();
>         if (next == null && head != tailRef()) {
>             // if tail != head this is not going to change until consumer makes progress
>             // we can avoid reading the head and just spin on next until it shows up
>             //
>             // See https://github.com/akka/akka/pull/15596
>             do {
>                 next = head.next();
>             } while (next == null);
>         }
>         return next;
>     }
> 
> public E poll() {
>         final MpscLinkedQueueNode<E> next = peekNode();
>         if (next == null) {
>             return null;
>         }
> 
>         // next becomes a new head.
>         MpscLinkedQueueNode<E> oldHead = headRef();
>         // Similar to 'headRef.node = next', but slightly faster (storestore vs loadstore)
>         // See: http://robsjava.blogspot.com/2013/06/a-faster-volatile.html
>         // See: http://psy-lob-saw.blogspot.com/2012/12/atomiclazyset-is-performance-win-for.html
>         lazySetHeadRef(next);
> 
>         // Break the linkage between the old head and the new head.
>         oldHead.unlink();
>         return next.clearMaybe();
>     }
> ```
>
> ![](https://img-blog.csdn.net/20170216170551355?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGluZ2d1b2hhbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
>
> 



三. 用户自定义定时任务

```java
ctx.channel().eventLoop().schedule(new Runnable() {
    @Override
    public void run() {

    }
}, 60, TimeUnit.SECONDS);
```

第三种场景就是定时任务逻辑了，用的最多的便是如上方法：在一定时间之后执行任务

我们跟进`schedule`方法

通过 `ScheduledFutureTask`, 将用户自定义任务再次包装成一个netty内部的任务

```java
<V> ScheduledFuture<V> schedule(final ScheduledFutureTask<V> task) {
    // ...
    scheduledTaskQueue().add(task);
    // ...
    return task;
}
```

到了这里，我们有点似曾相识，在非定时任务的处理中，netty通过一个mpsc队列将任务落地，这里，是否也有一个类似的队列来承载这类定时任务呢？带着这个疑问，我们继续向前

```java
Queue<ScheduledFutureTask<?>> scheduledTaskQueue() {
    if (scheduledTaskQueue == null) {
        scheduledTaskQueue = new PriorityQueue<ScheduledFutureTask<?>>();
    }
    return scheduledTaskQueue;
}
```

果不其然，`scheduledTaskQueue()` 方法，会返回一个优先级队列，然后调用 `add` 方法将定时任务加入到队列中去，但是，这里为什么要使用优先级队列，而不需要考虑多线程的并发？

因为我们现在讨论的场景，调用链的发起方是reactor线程，不会存在多线程并发这些问题

但是，万一有的用户在reactor之外执行定时任务呢？虽然这类场景很少见，但是netty作为一个无比健壮的高性能io框架，必须要考虑到这种情况。

对此，netty的处理是，如果是在外部线程调用schedule，**netty将添加定时任务的逻辑封装成一个普通的task，这个task的任务是添加[添加定时任务]的任务，而不是添加定时任务，其实也就是第二种场景**，这样，对 `PriorityQueue`的访问就变成单线程，即只有reactor线程。

完整的schedule方法

```java
<V> ScheduledFuture<V> schedule(final ScheduledFutureTask<V> task) {
    if (inEventLoop()) {
        scheduledTaskQueue().add(task);
    } else {
        // 进入到场景二，进一步封装任务
        execute(new Runnable() {
            @Override
            public void run() {
                scheduledTaskQueue().add(task);
            }
        });
    }
    return task;
}
```

在阅读源码细节的过程中，我们应该多问几个为什么？这样会有利于看源码的时候不至于犯困！比如这里，为什么定时任务要保存在优先级队列中，我们可以先不看源码，来思考一下优先级对列的特性

优先级队列按一定的顺序来排列内部元素，内部元素必须是可以比较的，联系到这里每个元素都是定时任务，那就说明定时任务是可以比较的，那么到底有哪些地方可以比较？

每个任务都有一个下一次执行的截止时间，截止时间是可以比较的，截止时间相同的情况下，任务添加的顺序也是可以比较的，就像这样，阅读源码的过程中，一定要多和自己对话，多问几个为什么

带着猜想，我们研究与一下`ScheduledFutureTask`，抽取出关键部分

```java
final class ScheduledFutureTask<V> extends PromiseTask<V> implements ScheduledFuture<V> {
    private static final AtomicLong nextTaskId = new AtomicLong();
    private static final long START_TIME = System.nanoTime();

    static long nanoTime() {
        return System.nanoTime() - START_TIME;
    }

    private final long id = nextTaskId.getAndIncrement();
    /* 0 - no repeat, >0 - repeat at fixed rate, <0 - repeat with fixed delay */
    private final long periodNanos;

    @Override
    public int compareTo(Delayed o) {
        //...
    }

    // 精简过的代码
    @Override
    public void run() {
    }
```

这里，我们一眼就找到了`compareTo` 方法，`cmd+u`跳转到实现的接口，发现就是`Comparable`接口

```java
public int compareTo(Delayed o) {
    if (this == o) {
        return 0;
    }

    ScheduledFutureTask<?> that = (ScheduledFutureTask<?>) o;
    long d = deadlineNanos() - that.deadlineNanos();
    if (d < 0) {
        return -1;
    } else if (d > 0) {
        return 1;
    } else if (id < that.id) {
        return -1;
    } else if (id == that.id) {
        throw new Error();
    } else {
        return 1;
    }
}
```

进入到方法体内部，我们发现，两个定时任务的比较，确实是先比较任务的截止时间，截止时间相同的情况下，再比较id，即任务添加的顺序，如果id再相同的话，就抛Error

这样，在执行定时任务的时候，就能保证最近截止时间的任务先执行

下面，我们再来看下netty是如何来保证各种定时任务的执行的，netty里面的定时任务分以下三种

1. **若干时间后执行一次**
2. **每隔一段时间执行一次**
3. **每次执行结束，隔一定时间再执行一次**

netty使用一个 `periodNanos` 来区分这三种情况，正如netty的注释那样

```java
/* 0 - no repeat, >0 - repeat at fixed rate, <0 - repeat with fixed delay */
private final long periodNanos;
```

了解这些背景之后，我们来看下netty是如何来处理这三种不同类型的定时任务的

```java
public void run() {
    if (periodNanos == 0) {
        V result = task.call();
        setSuccessInternal(result);
    } else { 
        task.call();
        long p = periodNanos;
        if (p > 0) {
            deadlineNanos += p;
        } else {
            deadlineNanos = nanoTime() - p;
        }
            scheduledTaskQueue.add(this);
        }
    }
}
```



`if (periodNanos == 0)` 对应 `若干时间后执行一次` 的定时任务类型，执行完了该任务就结束了。

否则，进入到else代码块，先执行任务，然后再区分是哪种类型的任务

- `periodNanos`大于0，表示是以固定频率执行某个任务，和任务的持续时间无关，然后，设置该任务的下一次截止时间为本次的截止时间加上间隔时间`periodNanos`。
- 否则，就是每次任务执行完毕之后，间隔多长时间之后再次执行，**截止时间为当前时间加上间隔时间**，`-p`就表示加上一个正的间隔时间，最后，将当前任务对象再次加入到队列，实现任务的定时执行

netty内部的任务添加机制了解地差不多之后，我们就可以查看reactor第三部曲是如何来调度这些任务的

#### reactor线程task的调度

首先，我们将目光转向最外层的外观代码

```java
runAllTasks(long timeoutNanos);
```

顾名思义，这行代码表示了**尽量在一定的时间内**，将所有的任务都取出来run一遍。`timeoutNanos` **表示该方法最多执行这么长时间**，netty为什么要这么做？我们可以想一想，**reactor线程如果在此停留的时间过长，那么将积攒许多的IO事件无法处理(见reactor线程的前面两个步骤)，最终导致大量客户端请求阻塞，因此，默认情况下，netty将控制内部队列的执行时间**

好，我们继续跟进

```java
protected boolean runAllTasks(long timeoutNanos) {
    fetchFromScheduledTaskQueue();
    Runnable task = pollTask();
    //...

    final long deadline = ScheduledFutureTask.nanoTime() + timeoutNanos;
    long runTasks = 0;
    long lastExecutionTime;
    for (;;) {
        safeExecute(task);
        runTasks ++;
        if ((runTasks & 0x3F) == 0) {
            lastExecutionTime = ScheduledFutureTask.nanoTime();
            if (lastExecutionTime >= deadline) {
                break;
            }
        }

        task = pollTask();
        if (task == null) {
            lastExecutionTime = ScheduledFutureTask.nanoTime();
            break;
        }
    }

    afterRunningAllTasks();
    this.lastExecutionTime = lastExecutionTime;
    return true;
}
```

这段代码便是reactor执行task的所有逻辑，可以拆解成下面几个步骤

1. 从scheduledTaskQueue转移定时任务到taskQueue(mpsc queue)
2. 计算本次任务循环的截止时间
3. 执行任务
4. 收尾

按照这个步骤，我们一步步来分析下

### 从scheduledTaskQueue转移定时任务到taskQueue(mpsc queue)

首先调用  `fetchFromScheduledTaskQueue()`方法，将到期的定时任务转移到mpsc queue里面

```java
private boolean fetchFromScheduledTaskQueue() {
    long nanoTime = AbstractScheduledEventExecutor.nanoTime();
    Runnable scheduledTask  = pollScheduledTask(nanoTime);
    while (scheduledTask != null) {
        if (!taskQueue.offer(scheduledTask)) {
            // No space left in the task queue add it back to the scheduledTaskQueue so we pick it up again.
            scheduledTaskQueue().add((ScheduledFutureTask<?>) scheduledTask);
            return false;
        }
        scheduledTask  = pollScheduledTask(nanoTime);
    }
    return true;
}
```

可以看到，netty在把任务从scheduledTaskQueue转移到taskQueue的时候还是非常小心的，当taskQueue无法offer的时候，需要把从scheduledTaskQueue里面取出来的任务重新添加回去

从scheduledTaskQueue从拉取一个定时任务的逻辑如下，传入的参数`nanoTime`为当前时间(其实是当前纳秒减去`ScheduledFutureTask`类被加载的纳秒个数)

```java
protected final Runnable pollScheduledTask(long nanoTime) {
    assert inEventLoop();

    Queue<ScheduledFutureTask<?>> scheduledTaskQueue = this.scheduledTaskQueue;
    ScheduledFutureTask<?> scheduledTask = scheduledTaskQueue == null ? null : scheduledTaskQueue.peek();
    if (scheduledTask == null) {
        return null;
    }

    if (scheduledTask.deadlineNanos() <= nanoTime) {
        scheduledTaskQueue.remove();
        return scheduledTask;
    }
    return null;
}
```

可以看到，每次 `pollScheduledTask` 的时候，只有在当前任务的截止时间已经到了，才会取出来

### 计算本次任务循环的截止时间

```java
     Runnable task = pollTask();
     //...
    final long deadline = ScheduledFutureTask.nanoTime() + timeoutNanos;
    long runTasks = 0;
    long lastExecutionTime;
```

这一步将取出第一个任务，用reactor线程传入的超时时间 `timeoutNanos` 来计算出当前任务循环的deadline，并且使用了`runTasks`，`lastExecutionTime`来时刻记录任务的状态

### 循环执行任务

```java
for (;;) {
    safeExecute(task);
    runTasks ++;
    if ((runTasks & 0x3F) == 0) {
        lastExecutionTime = ScheduledFutureTask.nanoTime();
        if (lastExecutionTime >= deadline) {
            break;
        }
    }

    task = pollTask();
    if (task == null) {
        lastExecutionTime = ScheduledFutureTask.nanoTime();
        break;
    }
}
```

这一步便是netty里面执行所有任务的核心代码了。
 首先调用`safeExecute`来确保任务安全执行，忽略任何异常

```java
protected static void safeExecute(Runnable task) {
    try {
        task.run();
    } catch (Throwable t) {
        logger.warn("A task raised an exception. Task: {}", task, t);
    }
}
```

然后将已运行任务 `runTasks` 加一，每隔`0x3F`任务，即每执行完64个任务之后，判断当前时间是否超过本次reactor任务循环的截止时间了，如果超过，那就break掉，如果没有超过，那就继续执行。可以看到，netty对性能的优化考虑地相当的周到，假设netty任务队列里面如果有海量小任务，如果每次都要执行完任务都要判断一下是否到截止时间，那么效率是比较低下的

### 收尾

```java
afterRunningAllTasks();
this.lastExecutionTime = lastExecutionTime;
```

收尾工作很简单，调用一下 `afterRunningAllTasks` 方法

```java
@Override
protected void afterRunningAllTasks() {
        runAllTasksFrom(tailTasks);
}
```

`NioEventLoop`可以通过父类`SingleTheadEventLoop`的`executeAfterEventLoopIteration`方法向`tailTasks`中添加收尾任务，比如，你想统计一下一次执行一次任务循环花了多长时间就可以调用此方法

```java
public final void executeAfterEventLoopIteration(Runnable task) {
        // ...
        if (!tailTasks.offer(task)) {
            reject(task);
        }
        //...
}
```

`this.lastExecutionTime = lastExecutionTime;`简单记录一下任务执行的时间，搜了一下该field的引用，发现这个field并没有使用过，只是每次不停地赋值，赋值，赋值...，改天再去向netty官方提个issue...

reactor线程第三曲到了这里基本上就给你讲完了，如果你读到这觉得很轻松，那么恭喜你，你对netty的task机制已经非常比较熟悉了，也恭喜一下我，把这些机制给你将清楚了。我们最后再来一次总结，以tips的方式

- 当前reactor线程调用当前eventLoop执行任务，直接执行，否则，添加到任务队列稍后执行
- netty内部的任务分为普通任务和定时任务，分别落地到MpscQueue和PriorityQueue
- netty每次执行任务循环之前，会将已经到期的定时任务从PriorityQueue转移到MpscQueue
- netty每隔64个任务检查一下是否该退出任务循环

