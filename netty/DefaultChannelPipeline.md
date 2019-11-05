# DefaultChannelPipeline

> 默认的 `ChannelPipeline`  的实现，它通常当 `Channel` 被创建的时候，被  `Channel ` 实现创建。

### 成员变量
``` java
//  产生  ChannelPipeline 的 头与尾的 名字
private static final String HEAD_NAME = generateName0(HeadContext.class);
private static final String TAIL_NAME = generateName0(TailContext.class);
```

一个 netty 自定义的 `FastThreadLocal`  的类，
```java
private static final FastThreadLocal<Map<Class<?>, String>> nameCaches =       
    new FastThreadLocal<Map<Class<?>, String>>() {    
@Override    
protected Map<Class<?>, String> initialValue() throws Exception {       
    return new WeakHashMap<Class<?>, String>();  
    }
};

```
**WeakHashMap**
> WeakHashMap 当除了自身对Key 的引用外，此Key 没有其他引用那么此Map 会 自动丢弃此值。WeakHashMap并不是你啥也不干他就能自动释放内部不用的对象的，而是在你访问它的内容的时候释放内部不用的对象，WeakHashMap继承了 WeakReference类。使用场景就是做缓存。当发生 内存不够时，GC会回收。

```java
private static final           AtomicReferenceFieldUpdater<DefaultChannelPipeline,MessageSizeEstimator.Handle>ESTIMATOR =        
AtomicReferenceFieldUpdater.newUpdater(             
DefaultChannelPipeline.class, MessageSizeEstimator.Handle.class, "estimatorHandle");
```

**AtomicReferenceFieldUpdater**
> 作用就是原子更新类中被volatile 修饰的字段。所以上面的 `ESTIMATOR` 的作用就是可以原子更新 `DefaultChannelPipeline.class ` 中的 成员变量为`MessageSizeEstimator.Handle.class`类型的 `estimatorHandle` 。 
#### AtomicReferenceFieldUpdater 源码分析
```java
//定义工厂方法，构建AtomicReferenceFieldUpdater
public static <U,W> AtomicReferenceFieldUpdater<U,W> newUpdater(Class<U> tclass,
                                                                    Class<W> vclass,
                                                                    String fieldName) {
        return new AtomicReferenceFieldUpdaterImpl<U,W>
            (tclass, vclass, fieldName, Reflection.getCallerClass());
    }
    //默认构造方法
    protected AtomicReferenceFieldUpdater() {}

    //cas更新指定字段
    public abstract boolean compareAndSet(T obj, V expect, V update);

    public abstract boolean weakCompareAndSet(T obj, V expect, V update);

    public abstract void set(T obj, V newValue);

    public abstract void lazySet(T obj, V newValue);

    public abstract V get(T obj);

     public V getAndSet(T obj, V newValue) {
        V prev;
        do {
            prev = get(obj);
        } while (!compareAndSet(obj, prev, newValue));
        return prev;
    }

     public final V getAndUpdate(T obj, UnaryOperator<V> updateFunction) {
        V prev, next;
        do {
            prev = get(obj);
            next = updateFunction.apply(prev);
        } while (!compareAndSet(obj, prev, next));
        return prev;
    }

    public final V updateAndGet(T obj, UnaryOperator<V> updateFunction) {
        V prev, next;
        do {
            prev = get(obj);
            next = updateFunction.apply(prev);
        } while (!compareAndSet(obj, prev, next));
        return next;
    }

     public final V getAndAccumulate(T obj, V x,
                                    BinaryOperator<V> accumulatorFunction) {
        V prev, next;
        do {
            prev = get(obj);
            next = accumulatorFunction.apply(prev, x);
        } while (!compareAndSet(obj, prev, next));
        return prev;
    }

    public final V accumulateAndGet(T obj, V x,
                                    BinaryOperator<V> accumulatorFunction) {
        V prev, next;
        do {
            prev = get(obj);
            next = accumulatorFunction.apply(prev, x);
        } while (!compareAndSet(obj, prev, next));
        return next;
    }

```
内部类实现如下，代码整体较简单，注重看下构造器中的一个小细节，Class被 `ClassLoader` 加载，所有的`ClassLoader` 应该在一个双亲委托模型的ClassLoader加载链中。tclass和vclass的类加载器会判断是否在一个类加载器链中。

```java
 private static final class AtomicReferenceFieldUpdaterImpl<T,V>
        extends AtomicReferenceFieldUpdater<T,V> {
        /*
         关于 unsafe 
        https://www.zhihu.com/question/29266773?sort=created
        Unsafe的使用建议:
        1、Unsafe有可能在未来的Jdk版本移除或者不允许Java应用代码使用，这一点可能导致使用了Unsafe的应用无法运行在高版本的Jdk。
        2、Unsafe的不少方法中必须提供原始地址(内存地址)和被替换对象的地址，偏移量要自己计算，一旦出现问题就是JVM崩溃级别的异常，会导致整个JVM实例崩溃，表现为应用程序直接crash掉。           3、Unsafe提供的直接内存访问的方法中使用的内存不受JVM管理(无法被GC)，需要手动管理，一旦出现疏忽很有可能成为内存泄漏的源头。
        
        美团 写的 Unsafe 应用解析
        
        https://zhuanlan.zhihu.com/p/56772793
        
        */
        //使用 Unsafe 做CAS操作
        private static final sun.misc.Unsafe U = sun.misc.Unsafe.getUnsafe();
        private final long offset;
        /**
         * 待更新的的对象的类的Class对象。大多数情况下和 tclass 相同。
         */
        private final Class<?> cclass;
        /** 待操作的目标对象 */
        private final Class<T> tclass;
        /** 待操作的字段的类型 */
        private final Class<V> vclass;


        /** 创建原子更新器 */
        AtomicReferenceFieldUpdaterImpl(final Class<T> tclass,
                                        final Class<V> vclass,
                                        final String fieldName,
                                        final Class<?> caller) {
            final Field field;
            final Class<?> fieldClass;
            final int modifiers;
            try {
            //获取tclass的Field
                field = AccessController.doPrivileged(
                    new PrivilegedExceptionAction<Field>() {
                        public Field run() throws NoSuchFieldException {
                            return tclass.getDeclaredField(fieldName);
                        }
                    });
                modifiers = field.getModifiers();
                sun.reflect.misc.ReflectUtil.ensureMemberAccess(
                    caller, tclass, null, modifiers);
                ClassLoader cl = tclass.getClassLoader();
                ClassLoader ccl = caller.getClassLoader();
                //判断class和修改的字段的类加载器是否在一个类加载器委托链中
                if ((ccl != null) && (ccl != cl) &&
                    ((cl == null) || !isAncestor(cl, ccl))) {
                    sun.reflect.misc.ReflectUtil.checkPackageAccess(tclass);
                }
                fieldClass = field.getType();
            } catch (PrivilegedActionException pae) {
                throw new RuntimeException(pae.getException());
            } catch (Exception ex) {
                throw new RuntimeException(ex);
            }

            if (vclass != fieldClass)
                throw new ClassCastException();
            if (vclass.isPrimitive())
                throw new IllegalArgumentException("Must be reference type");

            if (!Modifier.isVolatile(modifiers))
                throw new IllegalArgumentException("Must be volatile type");

            this.cclass = (Modifier.isProtected(modifiers) &&
                           tclass.isAssignableFrom(caller) &&
                           !isSamePackage(tclass, caller))
                          ? caller : tclass;
            this.tclass = tclass;
            this.vclass = vclass;
            this.offset = U.objectFieldOffset(field);
        }


         //判断第二个类加载器是否能在第一个加载器委托链中找到
        private static boolean isAncestor(ClassLoader first, ClassLoader second) {
            ClassLoader acl = first;
            do {
                acl = acl.getParent();
                if (second == acl) {
                    return true;
                }
            } while (acl != null);
            return false;
        }


         //判断两个classes是否是相同的类加载器和包修饰
        private static boolean isSamePackage(Class<?> class1, Class<?> class2) {
            return class1.getClassLoader() == class2.getClassLoader()
                   && Objects.equals(getPackageName(class1), getPackageName(class2));
        }

        private static String getPackageName(Class<?> cls) {
            String cn = cls.getName();
            int dot = cn.lastIndexOf('.');
            return (dot != -1) ? cn.substring(0, dot) : "";
        }

        //判断目标参数是否obj的实例
        private final void accessCheck(T obj) {
            if (!cclass.isInstance(obj))
                throwAccessCheckException(obj);
        }


        private final void throwAccessCheckException(T obj) {
            if (cclass == tclass)
                throw new ClassCastException();
            else
                throw new RuntimeException(
                    new IllegalAccessException(
                        "Class " +
                        cclass.getName() +
                        " can not access a protected member of class " +
                        tclass.getName() +
                        " using an instance of " +
                        obj.getClass().getName()));
        }

        private final void valueCheck(V v) {
            if (v != null && !(vclass.isInstance(v)))
                throwCCE();
        }

        static void throwCCE() {
            throw new ClassCastException();
        }

        //原子更新
        public final boolean compareAndSet(T obj, V expect, V update) {
            accessCheck(obj);
            valueCheck(update);
            return U.compareAndSwapObject(obj, offset, expect, update);
        }

        public final boolean weakCompareAndSet(T obj, V expect, V update) {
            // same implementation as strong form for now
            accessCheck(obj);
            valueCheck(update);
            return U.compareAndSwapObject(obj, offset, expect, update);
        }

        public final void set(T obj, V newValue) {
            accessCheck(obj);
            valueCheck(newValue);
            U.putObjectVolatile(obj, offset, newValue);
        }

        public final void lazySet(T obj, V newValue) {
            accessCheck(obj);
            valueCheck(newValue);
            U.putOrderedObject(obj, offset, newValue);
        }

        @SuppressWarnings("unchecked")
        public final V get(T obj) {
            accessCheck(obj);
            return (V)U.getObjectVolatile(obj, offset);
        }

        @SuppressWarnings("unchecked")
        public final V getAndSet(T obj, V newValue) {
            accessCheck(obj);
            valueCheck(newValue);
            return (V)U.getAndSetObject(obj, offset, newValue);
        }
    }
```


`DefaultChannelPipeline` 保存了 两个  ChannelHandlerContext 的节点，一个头，一个尾
```java
final AbstractChannelHandlerContext head;
final AbstractChannelHandlerContext tail;
```
还包含 一下几个成员变量
```JAVA
private final Channel channel;
private final ChannelFuture succeededFuture;
private final VoidChannelPromise voidPromise;
private final boolean touch = ResourceLeakDetector.isEnabled()
;private Map<EventExecutorGroup, EventExecutor> childExecutors;
private volatile MessageSizeEstimator.Handle estimatorHandle;
private boolean firstRegistration = true;

```
`MessageSizeEstimator`   
> 负责估计消息的大小，大小意味着消息将在内存中占用的内存大小。

**PendingHandlerCallback**
```java
/*
?? 不明白作用
*/
private PendingHandlerCallback pendingHandlerCallbackHead;
```
### 方法

#### addLast 

```java

// 优先匹配 固定参数
public final ChannelPipeline addLast(ChannelHandler handler) { 
    return addLast(null, handler);
}

@Overridepublic final ChannelPipeline addLast(ChannelHandler... handlers) {   
    return addLast(null, handlers);
}

```

```java
@Override
public final ChannelPipeline addLast(EventExecutorGroup executor, ChannelHandler... handlers) {  
if (handlers == null) {
    throw new NullPointerException("handlers");   
}    
for (ChannelHandler h: handlers) {      
    if (h == null) {  
        break;      
}        
    addLast(executor, null, h); 
    }    
    return this;
}

```
关键函数
```java
@Override
public final ChannelPipeline addLast(EventExecutorGroup group, String  name,ChannelHandler handler) {   
        final AbstractChannelHandlerContext newCtx;    
        synchronized (this) { 
           // handler 分为 共享 或非共享，@Sharable 注解用来说明ChannelHandler是否可以在多个channel直接共享使用，下面这个函数就是检测 符合标准
           /*
           因为一个ChannelHandler可以从属于多个ChannelPipeline，所以它也可以绑定到多个ChannelHandlerContext实例。用于这种用法的ChannelHandler必须要使用@Sharable注解标注；否                 则，试图将它添加到多个ChannelPipeline时将会触发异常。显而易见，为了安全地被用于多个并发的Channel（即连接），这样的ChannelHandler必须是线程安全的。
           */
            checkMultiplicity(handler);    
            /*
            生成一个 AbstractChannelHandlerContext ，AbstractChannelHandlerContext 它的成员变量中记录了它所属的 pipeline
            */
            newCtx = newContext(group, filterName(name, handler), handler); 
            /*
             pipeline  中记录的是 channelHandler 的 ctx 。
            */
            addLast0(newCtx);    
        // If the registered is false it means that the channel was not registered on an eventloop yet.       
        // In this case we add the context to the pipeline and add a task that will call        
        // ChannelHandler.handlerAdded(...) once the channel is registered.        
            if (!registered) {  
                // 如果 该 pipe 未注册到 eventloop 上，则将 newCtx 的 的 handlerState 设置为ADD_PENDING 。这时 ChannelHandler#handlerAdded(ChannelHandlerContext)} 将被调用
                newCtx.setAddPending();
                callHandlerCallbackLater(newCtx, true); 
                return this;        
            }      
        EventExecutor executor = newCtx.executor();     
        if (!executor.inEventLoop()) {         
             newCtx.setAddPending();           
             executor.execute(new Runnable() {             
                    @Override               
                    public void run() {                 
                            callHandlerAdded0(newCtx);             
                         }           
                 });           
             return this;      
            }   
        }   
        callHandlerAdded0(newCtx);    
        return this;
    }
        
```

#### addlast0

```java
    private void addLast0(AbstractChannelHandlerContext newCtx) {
        /*
        channelPipe 中包含了 首尾两个指针， pipe 类似一个 双向链表，每个 ctx 都是带有双向指针的节点。
        接下来的操作 就是 将 newCtx 加入到尾节点 与 倒数第二个节点之间。
        */
        AbstractChannelHandlerContext prev = tail.prev;
        newCtx.prev = prev;
        newCtx.next = tail;
        prev.next = newCtx;
        tail.prev = newCtx;
    }
```

