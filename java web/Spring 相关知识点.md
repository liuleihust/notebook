#### 使用Spring 框架的好处是什么？

- **轻量**： Spring 是轻量的，基本的版本大约2MB。
- **控制反转：**Spring通过控制反转实现了松散耦合，对象们给出它们的依赖，而不是创建或查找依赖的对象们。
- **面向切面的编程(AOP)：**Spring支持面向切面的编程，并且把应用业务逻辑和系统服务分开。
- **容器**：Spring 包含并管理应用中对象的**生命周期**和配置。
- **MVC 框架**：Spring 的WEB 框架是个精心设计的框架，是Web 框架的一个很好的替代品。
- **事务管理**： Spring 提供一个持续的事务管理接口，可以扩展到上至本地事务下至全局事务。
- **异常处理**：Spring 提供方便的API把具体技术相关的异常（比如由JDBC，Hibernate or JDO抛出的）转化为一致的unchecked 异常。

#### Spring  框架模块功能介绍

![](C:\Users\Administrator\Documents\笔记\java web\img\1336193-20181221160055606-406200425.jpg)

**Core Container (核心容器)**

- **Beans**：负责Bean工厂中Bean的装配，所谓Bean工厂即是创建对象的工厂（采用工厂模式），Bean的装配也就是对象的创建工作；
- **Core**：这个模块即是负责**IOC**（控制反转，bean 加载注册到容器中，**设计好的对象交给容器控制，而不是在你的对象内部直接控制**）最基本的实现；
- **Context**：Spring的IOC容器，因大量调用Spring Core中的函数，整合了Spring的大部分功能。Bean创建好对象后，由Context负责建立Bean与Bean之间的关系并维护。**所以也可以把Context看成是Bean关系的集合**；
- **SpEl**：即Spring Expression Language（Spring表达式语言）；

**Data Access/Integration（数据访问/集成）:**

- **JDBC**：对JDBC的简单封装；
- **ORM**：支持数据集成框架的封装（如Mybatis，Hibernate）；
- **OXM**：即Object XML Mapper，它的作用是在Java对象和XML文档之间来回转换；
- **JMS**：生产者和消费者的消息功能的实现；
- **Transations**：事务管理，不多BB；

**Web：**

- **WebSocket**：提供Socket通信，web端的的推送功能；
- **Servlet**：Spring MVC框架的实现；
- **Web**：包含web应用开发用到Spring框架时所需的核心类，包括自动载入WebApplicationContext特性的类，Struts集成类、文件上传的支持类、Filter类和大量辅助工具类；
- **Portlet**：实现web模块功能的聚合（如网站首页（Port）下面可能会有不同的子窗口（Portlet））；

#### **什么是Spring IOC 容器？**

Spring IOC 负责创建对象，**管理对象（通过依赖注入（DI），装配对象，配置对象**，并且管理这些对象的整个生命周期。

#### IOC 的优点是什么？

IOC 或 依赖注入把应用的代码量降到最低。它使应用容易测试，单元测试不再需要单例和JNDI查找机制。最小的代价和最小的侵入性使松散耦合得以实现。**IOC容器支持加载服务时的饿汉式初始化和懒加载。**





#### Spring的 bean在什么时候被实例化

使用 ***ApplicationContext*** 作为 Spring Bean 的工厂类：

- 如果bean的scope是**singleton**的，并且**lazy-init**为**false**（默认是false，所以可以不用设置），则ApplicationContext启动的时候就实例化该Bean，并且将实例化的Bean放在一个map结构的缓存中，下次再使用该Bean的时候，直接从这个缓存中取 
- 如果bean的scope是singleton的，并且lazy-init为true，则该Bean的实例化是在第一次使用该Bean的时候进行实例化 
- 如果bean的scope是prototype的，则该Bean的实例化是在第一次使用该Bean的时候进行实例化

#### Spring Boot Ioc 容器初始化过程

1. 加载 ApplicationContextInializer & ApplicationListener
2. 初始化环境 ConfigurableEnvironment & 加载配置文件
3. 构建应用上下文 ApplicationContext
4. 通过 ApplicationListener 注册 BeanFactoryPostProcessor
5. 初始化 **BeanFactoryPostProcessor** 到 IoC 容器
6. 通过 BeanFactoryPostProcessor: ConfigurationClassParser 扫描注册所有组件(包括: @Bean @Configuration, @Imports) 到  IoC 容器
7. 注册拦截 bean 创建的 bean processors
8. createEmbededServletContainer: 通过内置的 Servlet 容器工厂创建内置 Servlet 容器
9. 初始化所有未初始化的单例 BeanDefinitions 到 Ioc 容器
10. 启动内置 Servlet 容器

#### Spring 框架中 bean 的生命周期

- Spring容器 从XML 文件中读取bean的定义，并**实例化bean**。
- Spring根据bean的定义填充所有的属性。
- 如果bean实现了BeanNameAware 接口，**Spring 传递bean 的ID 到 setBeanName方法**。
- 如果Bean 实现了 BeanFactoryAware 接口， Spring传递beanfactory 给setBeanFactory 方法。
- 如果有任何与bean相关联的BeanPostProcessors，Spring会在postProcesserBeforeInitialization()方法内调用它们。
- 如果bean实现IntializingBean了，调用它的afterPropertySet方法，如果bean声明了初始化方法，调用此初始化方法。
- 如果有BeanPostProcessors 和bean 关联，这些bean的postProcessAfterInitialization() 方法将被调用。
- 如果bean实现了 DisposableBean，它将调用destroy()方法。

#### Spring 几种自动装配的方式

- **byName**: 通过参数名自动装配，Spring容器在配置文件中发现bean的autowire属性被设置成byname，之后容器试图匹配、装配和该bean的属性具有相同名字的bean。
- **byType:：**通过参数类型自动装配，Spring容器在配置文件中发现bean的autowire属性被设置成byType，之后容器试图匹配、装配和该bean的属性具有相同类型的bean。如果有多个bean符合条件，则抛出错误。
- **constructor：这个方式类似于**byType， 但是要提供给构造器参数，如果没有确定的带参数的构造器参数类型，将会抛出异常。
- **autodetect：**首先尝试使用constructor来自动装配，如果无法工作，则使用byType方式。

####**@Required  注解**

这个注解表明bean的属性必须在配置的时候设置，通过一个 bean 定义的显式的属性值或通过自动装配，若@Required注解的bean属性未被设置，容器将抛出BeanInitializationException。

####  **@Qualifier 注解**

当有多个相同类型的bean却只有一个需要自动装配时，将@Qualifier 注解和@Autowire 注解结合使用以消除这种混淆，指定需要装配的确切的bean。

#### Spring -bean 解决循环依赖

> 循环依赖其实就是循环引用，也就是两个或者两个以上的 bean 互相持有对方，形成闭环。比如A依赖于B，B依赖于C，C又依赖于A。**循环依赖不是函数依赖，函数依赖是死循环。**

Spring 中循环依赖场景有：

1. 构造器的循环依赖
2. field 属性的循环依赖

##### 怎么检测是否循环依赖

Bean 在创建的时候给该Bean 打标志，如果递归调用回来发现正在创建中的话，即 说明了循环依赖了。

##### 怎么解决循环依赖

当我们获取对象的引用时，对象的 field或者属性是可以延后设置的（**但是构造器必须是在获取引用之前**）



Spring  的单例对象的初始化 主要分为三步：

`createBeanInstance实例化`-->`populateBean填充属性`-->`InitializeBean初始化`

- **createBeanInstance**：实例化，其实也就是调用对象的构造方法实例化对象。
- **populateBean**：填充属性，这一步主要是对bean的依赖属性进行填充。
- **initializeBean**：调用spring xml中的init 方法。

单例bean 初始化步骤我们可以知道，循环依赖主要发生在第一步和第二步，也就是构造器循环依赖和field循环依赖。

那么我们要解决循环引用也应该从初始化过程着手，对于单例来说，在Spring容器整个生命周期内，有且只有一个对象，所以很容易想到这个对象应该存在Cache中，Spring为了解决单例的循环依赖问题，使用了**三级缓存**。

这三级缓存分别指： 

**singletonObjects**：单例对象的cache

**earlySingletonObjects** ：提前暴光的单例对象的Cache

**singletonFactories** ： 单例对象工厂的cache 

我们在创建**bean**的时候，首先想到的是从**cache**中获取这个单例的bean，这个缓存就是**singletonObjects**。主要调用方法就就是：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    
    Object singletonObject = this.singletonObjects.get(beanName);
    // 如果当前 bean 没有被创建或者 正在创建中
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {			
        // 加锁
        synchronized (this.singletonObjects) {
            // 获取创建中的 bean 
            singletonObject = this.earlySingletonObjects.get(beanName);
            // 如果允许 提前引用
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```

上面代码：

- **isSingletonCurrentlyInCreation()** 判断当前单例bean是否正在创建中，也就是没有初始化完成(比如A的构造器依赖了B对象所以得先去创建B对象， 或则在A的populateBean过程中依赖了B对象，得先去创建B对象，这时的A就是处于创建中的状态。)

- **allowEarlyReference** 是否允许从singletonFactories中通过getObject拿到对象

  

  分析getSingleton()的整个过程，Spring首先从**一级缓存singletonObjects中获取**。如果获取不到，并且对象正在创建中，**就再从二级缓存earlySingletonObjects中获取**。如果还是获取不到且允许**singletonFactories通过getObject()获取**，就从三级缓存singletonFactory.getObject()(三级缓存)获取，如果获取到了则：

从singletonFactories中移除，并放入earlySingletonObjects中。其实也就是从三级缓存移动到了二级缓存。

从上面三级缓存的分析，我们可以知道，Spring解决循环依赖的诀窍就在于**singletonFactories**这个三级cache。这个cache的类型是**ObjectFactory**，定义如下：

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```

这个接口在下面被引用

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

也就是说当  单例对象已经被创建出来（**调用了构造器**）之后加入到第三级缓存。

这个对象已经被生产出来，虽然不完美（没有进行第二步和第三步），但是已经能被人认出来了，所有Spring 可以将这个对象提前曝光出来。

这样做，就可以解决：

**A 的 某个 field 或者 setter 依赖了B 的实例对象，同时B 的某个 个field或者setter依赖了A的实例对象 这种情况。**

这种情况下，A 完成构造后，进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走 create 流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试   `get(A)` 。 经过查询三级缓存后，可以得到了。这时候和 B 成功完了 bean 初始化的三步，并将自己放入得到一级缓存中。此时返回A，A此时能得到B的对象顺利完成自己的初始化阶段2,3,最终A也完成了初始化。

Spring 还是不能解决 A 的构造方法中依赖B,B的构造方法中依赖A的情况。

#### InitializingBean, DisposableBean 接口

可以实现 `InitializingBean,DisposableBean` 这两个接口，也是在初始化以及销毁阶段调用

```java
@Service
public class SpringLifeCycleService implements InitializingBean,DisposableBean{
    private final static Logger LOGGER = LoggerFactory.getLogger(SpringLifeCycleService.class);
    @Override
    public void afterPropertiesSet() throws Exception {
        LOGGER.info("SpringLifeCycleService start");
    }

    @Override
    public void destroy() throws Exception {
        LOGGER.info("SpringLifeCycleService destroy");
    }
}
```

#### Spring AOP 实现原理

Spring 的`AOP` 基于动态代理实现的，先介绍下  **静态代理**

假设有一接口 `InterfaceA`：

```java
public interface InterfaceA{
    void exec();
}
```

其中有实现类 `RealImplement`:

```java
public class RealImplement implement InterfaceA{
    public void exec(){
        System.out.println("real impl") ;
    }
}
```

这时也有一个代理类 `ProxyImplement` 也实现了 `InterfaceA`:

```java
public class ProxyImplement implement InterfaceA{
    private InterfaceA interface ;

    public ProxyImplement(){
        interface = new RealImplement() ;
    }

    public void exec(){
        System.out.println("dosomethings before);
        //实际调用
        interface.exec();

        System.out.println("dosomethings after);
    }

}
```

使用如下

```java
public class Main(){
    public static void main(String[] args){
        InterfaceA interface = new ProxyImplement() ;
        interface.exec();
    }
}
```

可以看出 这样的代理方式，调用者不知道代理对象的存在。

##### JDK 动态代理

从静态代理中可以看出: 静态代理只能代理一个具体的类，如果要代理一个接口的多个实现的话需要定义不同的代理类。

需要解决这个问题就可以用到 JDK 的动态代理。

其中有两个非常核心的类：

- `java.lang.reflect.Proxy`类。
- `java.lang.reflect.InvocationHandle`接口。

`Proxy` 类是用于创建代理对象，而 `InvocationHandler` 接口主要你是来处理执行逻辑。

如下：

```java
public class CustomizeHandle implements InvocationHandler {
    private final static Logger LOGGER = LoggerFactory.getLogger(CustomizeHandle.class);

    private Object target;

    public CustomizeHandle(Class clazz) {
        try {
            this.target = clazz.newInstance();
        } catch (InstantiationException e) {
            LOGGER.error("InstantiationException", e);
        } catch (IllegalAccessException e) {
            LOGGER.error("IllegalAccessException",e);
        }
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        before();
        Object result = method.invoke(target, args);
        after();

        LOGGER.info("proxy class={}", proxy.getClass());
        return result;
    }


    private void before() {
        LOGGER.info("handle before");
    }

    private void after() {
        LOGGER.info("handle after");
    }
}
```

其中构造方法传入被代理类的类类型。其实传代理类的实例或者是类类型并没有强制的规定，传类类型的是因为被代理对象应当有代理创建而不应该由调用方创建

使用方式如下：

```java
    @Test
    public void test(){
        CustomizeHandle handle = new CustomizeHandle(ISubjectImpl.class) ;
        ISubject subject = (ISubject) Proxy.newProxyInstance(JDKProxyTest.class.getClassLoader(), new Class[]{ISubject.class}, handle);
        subject.execute() ;
    }
```

首先传入被代理类的类类型构建代理处理器。接着使用 `Proxy` 的`newProxyInstance` 方法动态创建代理类。第一个参数为类加载器，第二个参数为代理类需要实现的接口列表，最后一个则是处理器。

可以看到代理类继承了 `Proxy` 类，并实现了 `ISubject` 接口，由此也可以看到 JDK 动态代理为什么需要实现接口，已经继承了 `Proxy`是不能再继承其余类了。

其中实现了 `ISubject` 的 `execute()` 方法，并通过 `InvocationHandler` 中的 `invoke()` 方法来进行调用的。

##### CGLIB 动态代理

cglib 是对一个**小而快的字节码处理框架** `ASM` 的封装。 他的特点是继承于被代理类，这就要求被代理类不能被 `final` 修饰。