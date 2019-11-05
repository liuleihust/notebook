## ThreadLocal 用在什么地方？

讨论ThreadLocal用在什么地方前，我们先明确下，如果仅仅就一个线程，那么都不用谈ThreadLocal的，**ThreadLocal是用在多线程的场景的！！！**

ThreadLocal归纳下来就2类用途：

- **保存线程上下文信息，在任意需要的地方可以获取！！！**
- **线程安全的，避免某些情况需要考虑线程安全必须同步带来的性能损失！！！**

**保存线程上下文信息，在任意需要的地方可以获取！！！**

由于ThreadLocal的特性，同一线程在某地方进行设置，在随后的任意地方都可以获取到。从而可以用来保存线程上下文信息。

常用的比如每个请求怎么把一串后续关联起来，就可以用ThreadLocal进行set，在后续的任意需要记录日志的方法里面进行get获取到请求id，从而把整个请求串起来。

还有比如Spring的事务管理，用ThreadLocal存储Connection，从而各个DAO可以获取同一Connection，可以进行事务回滚，提交等操作。

> **备注：**ThreadLocal的这种用处，很多时候是用在一些优秀的框架里面的，一般我们很少接触，反而下面的场景我们接触的更多一些！

**线程安全的，避免某些情况需要考虑线程安全必须同步带来的性能损失！！！**

ThreadLocal为解决多线程程序的并发问题提供了一种新的思路。但是ThreadLocal也有局限性，我们来看看阿里规范：

> ThreadLocal 无法解决共享对象的更新问题，ThreadLocal 对象建议使用static 修饰，这个变量是针对一个线程内所有操作共享，所以设置为静态变量，所有此类实例共享此静态变量，也就说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象（只要是这个线程内定义的）都可以操控这个变量。

每个线程往 *ThreadLocal* 中读写数据是线程隔离，互相之间不会影响的，所以 ThreadLocal 无法解决共享对象的更新问题！

> 由于不需要共享信息，自然就不存在竞争问题了，从而保证了某些情况下线程的安全，以及避免了某些情况需要考虑线程安全必须同步带来的性能损失！！！

这类场景，阿里规范中提到了：

*SimpleDateFormat*  是线程不安全的类，一般不要定义 static 变量，如果定义为static，必须加锁，或者使用 DateUtils 工具类。也可使用下面处理：

```java
    private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>(){
        @Override
        protected DateFormat initialValue() {
            return new SimpleDateFormat("yyy-MM-dd");
        }
```

### ThreadLocal 一些细节

ThreaLocal使用示例代码：

```java
public class ThreadLocalTest {
    private static ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {

        new Thread(() -> {
            try {
                for (int i = 0; i < 100; i++) {
                    threadLocal.set(i);
                    System.out.println(Thread.currentThread().getName() + "====" + threadLocal.get());
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } finally {
                threadLocal.remove();
            }
        }, "threadLocal1").start();


        new Thread(() -> {
            try {
                for (int i = 0; i < 100; i++) {
                    System.out.println(Thread.currentThread().getName() + "====" + threadLocal.get());
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            } finally {
                threadLocal.remove();
            }
        }, "threadLocal2").start();
    }
}
```

执行结果：

```
threadLocal2====null
threadLocal1====95
threadLocal2====null
threadLocal1====96
threadLocal2====null
threadLocal1====97
threadLocal2====null
threadLocal1====98
threadLocal2====null
threadLocal1====99
```

从运行 结果我们可以看到hreadLocal1进行set值对threadLocal2并没有任何影响！

Thread、ThreadLocalMap、ThreadLocal 总览图。

![](C:\Users\Administrator\Documents\笔记\img\微信图片_20190625193944.jpg)

Thread类有属性变量*threadLocals* （类型是 *ThreadLocal.ThreadLocalMap*），也就是说每个线程有一个自己的`ThreadLocalMap` ，所以每个线程往这个ThreadLocal中读写隔离的，并且是互相不会影响的。

![](C:\Users\Administrator\Documents\笔记\img\微信图片_20190625195240.jpg)

看到上面的几个图，大概思路应该都清晰了，我们Entry的key指向ThreadLocal用**虚线**表示弱引用 ，下面我们来看看ThreadLocalMap:

弱引用用来描述非必需对象，当JVM进行垃圾回收的时，无论内存是否充足，**该对象仅仅被弱引用关联**，那么就会被回收。

**当仅仅只有ThreadLocalMap中的Entry的key指向ThreadLocal的时候，ThreadLocal会进行回收的！！！**

ThreadLocal被垃圾回收后，在ThreadLocalMap里对应的Entry的键值会变成null，但是Entry是强引用，那么Entry里面存储的Object，并没有办法进行回收，所以ThreadLocalMap 做了一些额外的回收工作。

虽然做了但是也会存在内存泄漏风险（我没有遇到过，网上很多类似场景，**所以会提到后面的ThreadLocal最佳实践！！！**）



ThreadLocal 的 set 方法：

```java
    public void set(T value) {
        // 获取当前线程对象
        Thread t = Thread.currentThread();
        // 获取线程对象的 ThreadLocalMap 对象
        ThreadLocalMap map = getMap(t);
        if (map != null)
            // 像 map 中
            map.set(this, value);
        else
            createMap(t, value);
    }
```



### ThreadLocal的最佳实践

由于线程的生命周期很长，如果我们往ThreadLocal里面set了很大很大的Object对象，虽然set、get等等方法在特定的条件会调用进行额外的清理，但是**ThreadLocal被垃圾回收后，在ThreadLocalMap里对应的Entry的键值会变成null，但是后续在也没有操作set、get等方法了。**

**所以最佳实践，应该在我们不使用的时候，主动调用remove方法进行清理。**

这里把 ThreadLocal 定义为 static 还有一个好处就是，由于`ThreadLocal`有强引用在，那么在`ThreadLocalMap`里对应的`Entry`的键会永远存在，那么执行`remove`的时候就可以正确进行定位到并且删除！！！

```java
try{
    // 其它业务逻辑
}finally{
    threadLocal对象.remove();
}
```

### FastThreadLocal

Netty 源码中使用 ，是ThreadLocal 的升级。

