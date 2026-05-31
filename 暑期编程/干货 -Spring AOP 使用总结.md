<font style="color:rgba(0, 0, 0, 0.8);">近期项目中使用AOP时遇到一些问题，借这个机会总结下AOP的使用，供后续需要的同学查阅。</font>

# <font style="color:rgba(0, 0, 0, 0.8);">什么是代理模式</font>
<font style="color:rgba(0, 0, 0, 0.8);">因为AOP主要是使用代理模式中的动态代理来实现的功能，所以先简单介绍下代理模式。</font>

### <font style="color:rgba(0, 0, 0, 0.8);">代理模式</font>
**<font style="color:rgba(0, 0, 0, 0.8);">为其他对象提供一种代理以便控制对这个对象的访问</font>**<font style="color:rgba(0, 0, 0, 0.8);">。他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。  
</font><font style="color:rgba(0, 0, 0, 0.8);">举例：比如你通过中介去租房子，就是用的代理模式。</font>

### <font style="color:rgba(0, 0, 0, 0.8);">代理模式分类</font>
<font style="color:rgba(0, 0, 0, 0.8);">1.静态代理（我们自己静态定义的代理类。比如我们自己定义一个workflow操作代理类。静态代理的每一个代理类只能为一个接口服务，静态代理在程序运行前，代理类的.class文件就已经编译生成好了）  
</font><font style="color:rgba(0, 0, 0, 0.8);">2.动态代理（通过程序动态生成代理类，该代理类不是我们自己定义的。而是由程序</font>**<font style="color:rgba(0, 0, 0, 0.8);">自动生成</font>**<font style="color:rgba(0, 0, 0, 0.8);">）  
</font><font style="color:rgba(0, 0, 0, 0.8);">动态代理有以下几种形式：  
</font><font style="color:rgba(0, 0, 0, 0.8);">JDK自带的动态代理（常用），  
</font><font style="color:rgba(0, 0, 0, 0.8);">CGLIB（常用），  
</font><font style="color:rgba(0, 0, 0, 0.8);">javaassist字节码操作库实现，  
</font><font style="color:rgba(0, 0, 0, 0.8);">ASM（底层使用指令，可维护性较差）</font>

### <font style="color:rgba(0, 0, 0, 0.8);">静态代理</font>
<font style="color:rgba(0, 0, 0, 0.8);">以流程启动为例：  
</font><font style="color:rgba(0, 0, 0, 0.8);">流程管理接口类(共同接口)</font>

```java
 public interface WorkflowService {
    public void start(String bpmnXml);
}
```

<font style="color:rgba(0, 0, 0, 0.8);">流程管理实现类</font>

```java
public class WorkflowServiceImpl implements WorkflowService {
    @Override
    public void start(String bpmnXml) {
        System.out.println("启动流程");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">流程操作</font>**<font style="color:rgba(0, 0, 0, 0.8);">代理类</font>**

```java
public class WorkflowServiceImplProxy implements WorkflowService{
    /**
     * 目标对象
     */
    private WorkflowService workflowService;
    /**
     * 通过构造方法注入目标对象
     */
    public WorkflowServiceImplProxy(WorkflowService workflowService){
        this.workflowService=workflowService;
    }

    public void beforeStart(){
        System.out.println("发起流程前准备动作");
    }

    @Override
    public void start(String bpmnXml){
        beforeStart();
        workflowService.start(bpmnXml);
        afterStart();
    }

    public void afterStart(){
        System.out.println("发起流程后处理动作");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">使用测试：</font>

```java
    public static void main(String[] args){
        //直接使用具体实现类
        WorkflowService workflowService = new WorkflowServiceImpl();
        workflowService.start("xml content");
        System.out.println("--------------------");
        //使用静态代理实现类
        WorkflowServiceImplProxy workflowServiceImplProxy = new WorkflowServiceImplProxy(new WorkflowServiceImpl());
        workflowServiceImplProxy.start("xml content");
    }
```

<font style="color:rgba(0, 0, 0, 0.8);">运行结果：</font>

<font style="color:rgba(0, 0, 0, 0.8);"> </font>

```java
        启动流程
        --------------------
        发起流程前准备动作
        启动流程
        发起流程后处理动作
```

<font style="color:rgba(0, 0, 0, 0.8);">总结：通过使用静态代理类，可以实现在流程发起之前和之后做相应的操作，包括权限验证，日志打印等；但是因为针对每个具体实现类都需要写对应的代理类，容易造成系统类膨胀及重复代码过多，增加维护复杂度，后续扩展也比较麻烦，下面看下动态代理的实现方式；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">动态代理</font>
#### <font style="color:rgba(0, 0, 0, 0.8);">JDK动态代理</font>
<font style="color:rgba(0, 0, 0, 0.8);">使用Proxy类生成代理类，传入指定classLoader，Interface以及实现InvocationHandler的处理类</font>

```java
public class JDKProxy implements InvocationHandler {
    private Object realObject;

    public JDKProxy(Object realObject) {
        this.realObject = realObject;
    }

    public void beforeInvoke(){
        System.out.println("执行方法前");
    }

    public void afterInvoke(){
        System.out.println("执行方法后");
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        beforeInvoke();
        Object object = method.invoke(realObject, args);
        afterInvoke();
        return object;
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">使用测试：</font>

<font style="color:rgba(0, 0, 0, 0.8);"> </font>

```java
        //jdk动态代理
        WorkflowService workflowService1 = (WorkflowService)Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(),new Class[]{WorkflowService.class}, new JDKProxy(new WorkflowServiceImpl()));
        workflowService1.start("xml content");
```

<font style="color:rgba(0, 0, 0, 0.8);">运行结果：</font>

<font style="color:rgba(0, 0, 0, 0.8);"> </font>

```java
        执行方法前
        启动流程
        执行方法后
```

#### <font style="color:rgba(0, 0, 0, 0.8);">CGLIB动态代理</font>
<font style="color:rgba(0, 0, 0, 0.8);">关联委托类的Enhancer对象，回调实现MethodInterceptor接口的对象</font>

```java
public class CGLIBProxy implements MethodInterceptor {
    private Enhancer enhancer = new Enhancer();
    public Object getProxy(Class clazz){
        //设置需要创建子类的类
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        //通过字节码技术动态创建子类实例
        return enhancer.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        beforeInvoke();
        //通过代理类调用具体类的实现
        Object retValFromSuper = proxy.invokeSuper(obj, args);
        afterInvoke();
        return retValFromSuper;
    }

    public void beforeInvoke(){
        System.out.println("执行方法前");
    }

    public void afterInvoke(){
        System.out.println("执行方法后");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">使用测试：</font>

<font style="color:rgba(0, 0, 0, 0.8);">    </font>

```java
        //cglib动态代理
        CGLIBProxy cglibProxy = new CGLIBProxy();
        //创建代理类
        WorkflowService workflowService2 = (WorkflowService)cglibProxy.getProxy(WorkflowServiceImpl.class);
        workflowService2.start("xml content");
```

<font style="color:rgba(0, 0, 0, 0.8);">运行结果：</font>

<font style="color:rgba(0, 0, 0, 0.8);">  </font>

```java
        执行方法前
        启动流程
        执行方法后
```

<font style="color:rgba(0, 0, 0, 0.8);"> </font>

#### <font style="color:rgba(0, 0, 0, 0.8);">JDK代理和CGLIB代理区别：</font>
<font style="color:rgba(0, 0, 0, 0.8);">JDK动态代理，具体类必须实现接口才可以使用JDK动态代理，具体类没有实现或实现接口都可以使用CGLIB动态代理，不过CGLIB无法动态代理final类；  
</font><font style="color:rgba(0, 0, 0, 0.8);">CGLIB创建的动态代理对象性能比JDK创建的动态代理对象的性能高，但是CGLIB在创建代理对象时所花费的时间却比JDK多得多，所以</font>**<font style="color:rgba(0, 0, 0, 0.8);">对于单例的对象，因为无需频繁创建对象，用CGLIB合适，反之，使用JDK方式要更为合适一些</font>**<font style="color:rgba(0, 0, 0, 0.8);">；</font>

<font style="color:rgba(0, 0, 0, 0.8);">总结：动态代理解决了静态代理每个具体实现类都要写对应代理类的问题，相当于我们告诉程序，遇到符合我们指定规则的情况时，通过反射帮我们自动生成代理类，弥补静态代理的缺点。</font>

# <font style="color:rgba(0, 0, 0, 0.8);">什么是AOP</font>
<font style="color:rgba(0, 0, 0, 0.8);">AOP（Aspect-OrientedProgramming，面向方面编程），是通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术，可以说AOP是OOP（Object-Oriented Programing，面向对象编程）的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。当我们需要为分散的对象引入公共行为的时候，OOP则显得无能为力。也就是说，OOP允许你定义从上到下的关系，但并不适合定义从左到右的关系。例如日志功能。日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。对于其他类型的代码，如权限、异常处理和事务也是如此。这种散布在各处的无关的代码被称为横切（cross-cutting）代码，在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。</font>

<font style="color:rgba(0, 0, 0, 0.8);">而AOP技术则恰恰相反，它利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。</font>

<font style="color:rgba(0, 0, 0, 0.8);">AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天功的妙手将这些剖开的切面复原，不留痕迹。</font>

<font style="color:rgba(0, 0, 0, 0.8);">使用“横切”技术，如果把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处都基本相似。比如权限认证、日志、事务处理。Aop 的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。</font>

### <font style="color:rgba(0, 0, 0, 0.8);">术语介绍</font>
<font style="color:rgba(0, 0, 0, 0.8);">实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。然而殊途同归，实现AOP的技术特性却是相同的，分别为：  
</font><font style="color:rgba(0, 0, 0, 0.8);">1、join point（连接点）：  
</font><font style="color:rgba(0, 0, 0, 0.8);">是程序执行中的一个精确执行点，例如类中的一个方法或特定异常的抛出。它是一个抽象的概念，在实现AOP时，并不需要去定义一个join point。  
</font><font style="color:rgba(0, 0, 0, 0.8);">2、point cut（切入点）：  
</font><font style="color:rgba(0, 0, 0, 0.8);">本质上是一个捕获连接点的结构，是一系列连接点的集合。在AOP中，可以定义一个point cut，来捕获相关方法的调用。AOP框架允许开发者指定切入点：例如，使用正则表达式。 Spring定义了Pointcut接口，用来组合MethodMatcher和ClassFilter，可以通过名字很清楚的理解， MethodMatcher是用来检查目标类的方法是否可以被应用此通知，而ClassFilter是用来检查Pointcut是否应该应用到目标类上。  
</font><font style="color:rgba(0, 0, 0, 0.8);">3、advice（通知）：  
</font><font style="color:rgba(0, 0, 0, 0.8);">是point cut的执行代码，是执行“方面”的具体逻辑。  
</font><font style="color:rgba(0, 0, 0, 0.8);">Spring中advice有以下几种:  
</font><font style="color:rgba(0, 0, 0, 0.8);">Before(前置通知)：org.apringframework.aop.MethodBeforeAdvice  
</font><font style="color:rgba(0, 0, 0, 0.8);">After-returning(返回通知 == final通知)：org.springframework.aop.AfterReturningAdvice  
</font><font style="color:rgba(0, 0, 0, 0.8);">After-throwing(抛出通知)：org.springframework.aop.ThrowsAdvice  
</font><font style="color:rgba(0, 0, 0, 0.8);">Introduction(引入通知)：org.springframework.aop.IntroductionInterceptor  
</font><font style="color:rgba(0, 0, 0, 0.8);">Around(环绕通知)：org.aopaliance.intercept.MethodInterceptor  
</font><font style="color:rgba(0, 0, 0, 0.8);">4、切面(Aspect) ：  
</font><font style="color:rgba(0, 0, 0, 0.8);">通知和切入点共同组成了切面：时间、地点和要发生的“故事”  
</font><font style="color:rgba(0, 0, 0, 0.8);">5、引入(Introduction) ：  
</font><font style="color:rgba(0, 0, 0, 0.8);">引入允许我们向现有的类添加新的方法和属性(Spring提供了一个方法注入的功能）  
</font><font style="color:rgba(0, 0, 0, 0.8);">6、目标对象（Target Object）:  
</font><font style="color:rgba(0, 0, 0, 0.8);">包含连接点的对象。也被称作被通知或被代理对象。  
</font><font style="color:rgba(0, 0, 0, 0.8);">7、AOP代理（AOP Proxy）:  
</font><font style="color:rgba(0, 0, 0, 0.8);">AOP框架创建的对象，包含通知。 在Spring中，AOP代理可以是JDK动态代理或者CGLIB代理。  
</font><font style="color:rgba(0, 0, 0, 0.8);">8、织入（Weaving）:  
</font><font style="color:rgba(0, 0, 0, 0.8);">组装方面来创建一个被通知对象。这可以在编译时完成（例如使用AspectJ编译器），也可以在运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。  
</font><font style="color:rgba(0, 0, 0, 0.8);">上述的技术特性组成了基本的AOP技术，大多数AOP工具均实现了这些技术。它们也可以是研究AOP技术的基本术语。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://gw.alipayobjects.com/zos/skylark/c8e93eba-55c1-4501-94ad-1b687735fb47/2018/png/ed0168ef-032d-4396-9005-8872f8224cb6.png)<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">下面附上内网看到的一张图</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://gw.alipayobjects.com/zos/skylark/9e2a8f14-a1be-44a2-984f-be5c735b077a/2018/png/9e75608a-5a5a-49e0-beff-f0d96e66a479.png)

# <font style="color:rgba(0, 0, 0, 0.8);">Spring Aop 使用方式</font>
<font style="color:rgba(0, 0, 0, 0.8);">流程管理接口类(共同接口)</font>

```java
public interface WorkflowService {
    public void start(String bpmnXml);
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">基于代理实现AOP（</font>**<font style="color:rgba(0, 0, 0, 0.8);">自动创建Proxy</font>**<font style="color:rgba(0, 0, 0, 0.8);">）【推荐】</font>
<font style="color:rgba(0, 0, 0, 0.8);">1.编写接口类WorkflowService  
</font><font style="color:rgba(0, 0, 0, 0.8);">2.编写实现类WorkflowServiceImpl  
</font><font style="color:rgba(0, 0, 0, 0.8);">3.希望在流程启动前和启动后做一些操作，增加环绕通知</font>

```java
public class WorkflowLogAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        beforeInvoke();
        Object object = methodInvocation.proceed();
        afterInvoke();
        return object;
    }

    public void beforeInvoke(){
        System.out.println("执行方法前");
    }

    public void afterInvoke(){
        System.out.println("执行方法后");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">4.增加配置（如果WorkflowServiceAdviceAspect和WorkflowServiceImpl类上增加了@Component，下面的bean声明可以用Spring的自动扫描来替代）</font>

```java
    <!-- 定义通知内容，也就是切入点执行前后需要做的事情 -->
    <bean id="workflowLogAdvice" class="Spring.WorkflowLogAdvice"></bean>
    <!-- 定义被代理者 -->
    <bean id="workflowService" class="business.WorkflowServiceImpl"></bean>
    <!-- 切面配置 -->
    <bean id="logAop"
          class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
        <property name="beanNames">
            <list>
                <value>workflowService</value>
            </list>
        </property>
        <property name="interceptorNames">
            <list>
                <value>workflowLogAdvice</value>
            </list>
        </property>
    </bean>
```

<font style="color:rgba(0, 0, 0, 0.8);">5.测试结果</font>

<font style="color:rgba(0, 0, 0, 0.8);">    </font>

```java
        ApplicationContext context = new ClassPathXmlApplicationContext("aop.xml");
        WorkflowService workflowService = (WorkflowService)context.getBean("workflowService");
        workflowService.start("xml content");
```

<font style="color:rgba(0, 0, 0, 0.8);">运行结果：</font>

```java
        执行方法前
        启动流程
        执行方法后
```

### <font style="color:rgba(0, 0, 0, 0.8);">基于代理实现AOP（手</font>**<font style="color:rgba(0, 0, 0, 0.8);">动创建Proxy）</font>**
<font style="color:rgba(0, 0, 0, 0.8);">1.编写接口类WorkflowService  
</font><font style="color:rgba(0, 0, 0, 0.8);">2.编写实现类WorkflowServiceImpl  
</font><font style="color:rgba(0, 0, 0, 0.8);">3.希望在流程启动前和启动后做一些操作，所以需要实现Before和After-returning的通知</font>

```java
public class WorkflowServiceAdvice implements MethodBeforeAdvice, AfterReturningAdvice {
    @Override
    public void before(Method arg0, Object[] arg1, Object arg2) throws Throwable {
        System.out.println("执行方法前");
    }
    
    @Override
    public void afterReturning(Object arg0, Method arg1, Object[] arg2, Object arg3) throws Throwable {
        System.out.println("执行方法后");
    }
} 
```

<font style="color:rgba(0, 0, 0, 0.8);">  </font>

<font style="color:rgba(0, 0, 0, 0.8);">4.在配置文件中增加配置</font>

<font style="color:rgba(0, 0, 0, 0.8);">  </font>

```java
    <!-- 定义被代理者 -->
    <bean id="workflowService" class="business.WorkflowServiceImpl"></bean>

    <!-- 定义通知内容，也就是切入点执行前后需要做的事情 -->
    <bean id="workflowServiceAdvice" class="Spring.WorkflowServiceAdvice"></bean>

    <!-- 定义切入点位置 -->
    <bean id="startPointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut">
        <property name="pattern" value=".*start"></property>
    </bean>

    <!-- 使切入点与通知相关联，完成切面配置 -->
    <bean id="startAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
        <property name="advice" ref="workflowServiceAdvice"></property>
        <property name="pointcut" ref="startPointcut"></property>
    </bean>

    <!-- 设置代理 -->
    <bean id="workflowServiceProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <!-- 代理的对象，有启动流程的方法 -->
        <property name="target" ref="workflowService"></property>
        <!-- 使用切面 -->
        <property name="interceptorNames" value="startAdvisor"></property>
        <!-- 代理接口，流程服务接口 -->
        <property name="proxyInterfaces" value="business.WorkflowService"></property>
    </bean>
```

<font style="color:rgba(0, 0, 0, 0.8);">5.测试</font>

<font style="color:rgba(0, 0, 0, 0.8);">  </font>

```java
        ApplicationContext context = new ClassPathXmlApplicationContext("basic_aop.xml");
        WorkflowService workflowService = (WorkflowService)context.getBean("workflowServiceProxy");
        workflowService.start("xml content");
```

<font style="color:rgba(0, 0, 0, 0.8);">运行结果：</font>

<font style="color:rgba(0, 0, 0, 0.8);">      </font>

```java
        执行方法前
        启动流程
        执行方法后
```

### <font style="color:rgba(0, 0, 0, 0.8);">通过AspectJ提供的注解实现AOP</font>
<font style="color:rgba(0, 0, 0, 0.8);">1.编写接口类WorkflowService  
</font><font style="color:rgba(0, 0, 0, 0.8);">2.编写实现类WorkflowServiceImpl  
</font><font style="color:rgba(0, 0, 0, 0.8);">3.希望在流程启动前和启动后做一些操作，增加通知，这里==采用了Aspect提供的注解来实现</font>

```java
@Aspect
public class WorkflowServiceAdviceAspect {

    public WorkflowServiceAdviceAspect(){

    }

    @Pointcut("execution(* *.start())")
    public void startPoint(){}

    @Before("startPoint()")
    public void before(Method arg0, Object[] arg1, Object arg2) throws Throwable {
        System.out.println("执行方法前");
    }

    @AfterReturning("startPoint()")
    public void afterReturning(Object arg0, Method arg1, Object[] arg2, Object arg3) throws Throwable {
        System.out.println("执行方法后");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">4.增加配置（如果WorkflowServiceAdviceAspect和WorkflowServiceImpl类上增加了@Component，下面的bean声明可以用Spring的自动扫描来替代）</font>

```java
    <aop:aspectj-autoproxy />
    <!-- 定义通知内容，也就是切入点执行前后需要做的事情 -->
    <bean id="workflowServiceAdviceAspect" class="Spring.WorkflowServiceAdviceAspect"></bean>
    <!-- 定义被代理者 -->
    <bean id="workflowService" class="business.WorkflowServiceImpl"></bean>
```

<font style="color:rgba(0, 0, 0, 0.8);">5.测试结果</font>

```java
        ApplicationContext context = new ClassPathXmlApplicationContext("aspect_aop.xml");
        WorkflowService workflowService = (WorkflowService)context.getBean("workflowService");
        workflowService.start("xml content");
```

<font style="color:rgba(0, 0, 0, 0.8);">运行结果（和第一种方法一样）：</font>

```java
        执行方法前
        启动流程
        执行方法后
```

### **<font style="color:rgba(0, 0, 0, 0.8);">基于AspectJ和@annotation拦截方法实现AOP</font>**
<font style="color:rgba(0, 0, 0, 0.8);">1.编写Annotation类</font>

```java
@Target({ElementType.METHOD,ElementType.TYPE})
@Documented
@Inherited
public @interface WorkflowLogAnnotation {
    String field();
}
```

<font style="color:rgba(0, 0, 0, 0.8);">2.编写接口类WorkflowService  
</font><font style="color:rgba(0, 0, 0, 0.8);">3.编写实现类WorkflowServiceImpl，注意此处在方法上增加==WorkflowLogAnnotation==</font>

```java
public class WorkflowServiceImpl implements WorkflowService {
    @Override
    @WorkflowLogAnnotation
    public void start(String bpmnXml) {
        System.out.println("启动流程");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">4.在流程启动前和启动后做一些操作，增加通知，注意Pointcut表达式改为拦截器方式</font>

```java
@Aspect
public class WorkflowServiceAdviceAspect {

    public WorkflowServiceAdviceAspect(){

    }

    @Pointcut("annotation(Spring.WorkflowLogAnnotation)")
    public void startPoint()

    @Before("startPoint()")
    public void before(Method arg0, Object[] arg1, Object arg2) throws Throwable {
        System.out.println("执行方法前");
    }

    @AfterReturning("startPoint()")
    public void afterReturning(Object arg0, Method arg1, Object[] arg2, Object arg3) throws Throwable {
        System.out.println("执行方法后");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">5.增加配置（如果WorkflowServiceAdviceAspect和WorkflowServiceImpl类上增加了@Component，下面的bean声明可以用Spring的自动扫描来替代）</font>

```java
    <aop:aspectj-autoproxy />
    <!-- 定义通知内容，也就是切入点执行前后需要做的事情 -->
    <bean id="workflowServiceAdviceAspect" class="Spring.WorkflowServiceAdviceAspect"></bean>
    <!-- 定义被代理者 -->
    <bean id="workflowService" class="business.WorkflowServiceImpl"></bean>
```

### <font style="color:rgba(0, 0, 0, 0.8);">使用Spring的</font><font style="color:rgb(63, 126, 189);">aop:config</font><font style="color:rgba(0, 0, 0, 0.8);">标签配置实现Aop</font>
<font style="color:rgba(0, 0, 0, 0.8);">1.编写接口类WorkflowService  
</font><font style="color:rgba(0, 0, 0, 0.8);">2.编写实现类WorkflowServiceImpl  
</font><font style="color:rgba(0, 0, 0, 0.8);">3.定义希望在流程启动前和启动后做的操作，这里只是简单的方法定义，Aop的关联通过配置文件实现</font>

```java
public class WorkflowServiceBeforeAndAfter {
    public void before() {
        System.out.println("执行方法前");
    }

    public void afterReturning() {
        System.out.println("执行方法后");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">4.aopconfig.xml中增加配置</font>

```java
    <!-- 定义通知内容，也就是切入点执行前后需要做的事情 -->
    <bean id="workflowServiceBeforeAndAfter" class="Spring.WorkflowServiceBeforeAndAfter"></bean>
    <!-- 定义被代理者 -->
    <bean id="workflowService" class="business.WorkflowServiceImpl"></bean>
    <aop:config>
        <aop:aspect ref="workflowServiceBeforeAndAfter">
            <aop:before method="before" pointcut="execution(* *.start(..))" />
            <aop:after method="afterReturning" pointcut="execution(* *.start(..))" />
        </aop:aspect>
    </aop:config>
```

<font style="color:rgba(0, 0, 0, 0.8);">5.测试结果</font>

```java
        ApplicationContext context = new ClassPathXmlApplicationContext("aopconfig.xml");
        WorkflowService workflowService = (WorkflowService)context.getBean("workflowService");
        workflowService.start("xml content");
```

<font style="color:rgba(0, 0, 0, 0.8);">运行结果：</font>

```java
        执行方法前
        启动流程
        执行方法后
```

<font style="color:rgba(0, 0, 0, 0.8);">AOP对比表格：</font>

| <font style="color:rgba(0, 0, 0, 0.8);">通知类型</font> | <font style="color:rgba(0, 0, 0, 0.8);">基于Spring AOP 框架接口</font> | <font style="color:rgba(0, 0, 0, 0.8);">基于AspectJ注解</font> | <font style="color:rgba(0, 0, 0, 0.8);">基于aop:config</font> |
| --- | --- | --- | --- |
| <font style="color:rgba(0, 0, 0, 0.8);">Before Advice（前置增强）</font> | <font style="color:rgba(0, 0, 0, 0.8);">MethodBeforeAdvice</font> | <font style="color:rgba(0, 0, 0, 0.8);">@Before</font> | <font style="color:rgba(0, 0, 0, 0.8);">aop:before</font> |
| <font style="color:rgba(0, 0, 0, 0.8);">AfterAdvice（后置增强）</font> | <font style="color:rgba(0, 0, 0, 0.8);">AfterReturningAdvice</font> | <font style="color:rgba(0, 0, 0, 0.8);">@After</font> | <font style="color:rgba(0, 0, 0, 0.8);">aop:after</font> |
| <font style="color:rgba(0, 0, 0, 0.8);">AroundAdvice（环绕增强）</font> | <font style="color:rgba(0, 0, 0, 0.8);">MethodInterceptor</font> | <font style="color:rgba(0, 0, 0, 0.8);">@Around</font> | <font style="color:rgba(0, 0, 0, 0.8);">aop:around</font> |
| <font style="color:rgba(0, 0, 0, 0.8);">ThrowsAdvice（抛出增强)</font> | <font style="color:rgba(0, 0, 0, 0.8);">ThrowsAdvice</font> | <font style="color:rgba(0, 0, 0, 0.8);">@AfterThrowing</font> | <font style="color:rgba(0, 0, 0, 0.8);">aop:after-throwing</font> |
| <font style="color:rgba(0, 0, 0, 0.8);">IntroductionAdvice（引入增强）</font> | <font style="color:rgba(0, 0, 0, 0.8);">DelegatingIntroductionInterceptor</font> | <font style="color:rgba(0, 0, 0, 0.8);">@DeclareParents</font> | <font style="color:rgba(0, 0, 0, 0.8);">aop:declare-parents</font> |


# <font style="color:rgba(0, 0, 0, 0.8);">Spring AOP循环嵌套问题解析</font>
### <font style="color:rgba(0, 0, 0, 0.8);">问题描述</font>
<font style="color:rgba(0, 0, 0, 0.8);">通过代理实现方法调用时，如果在一个服务内部出现多次嵌套调用或多个服务嵌套调用，除了第一个方法或第一个服务外，后面的方法和服务都没有使用代理对象调用，而是调用原始对象的方法。</font>

<font style="color:rgba(0, 0, 0, 0.8);">此处以服务内部多次嵌套调用为例：</font>

```java
public class CaseServiceImpl implements CaseService{
    @Override
    @WorkflowLogAnnotation()
    public void methodOne()
    {
        methodTwo();
    }

    @Override
    @WorkflowLogAnnotation()
    public void methodTwo()
    {
        ...
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">通过WorkflowLogAnnotation实现注解，该段代码实际运行调用methodOne时，只有methodOne上的注解产生作用，methodTwo上的注解无效，原因是调用methodTwo时使用的是原始对象，不是代理对象。</font>

### <font style="color:rgba(0, 0, 0, 0.8);">解决方案</font>
<font style="color:rgba(0, 0, 0, 0.8);">思路就是使用代理对象来调用methodTwo，那么运行时如何获取代理对象呢，有两种方法。</font>

#### <font style="color:rgba(0, 0, 0, 0.8);">一.通过AopContext获取当前代理对象</font>
<font style="color:rgba(0, 0, 0, 0.8);">1.修改配置设置expose-proxy为true，如果不设置这个参数，AopContext.currentProxy将无法获取当前线程中的代理对象  
</font><font style="color:rgba(0, 0, 0, 0.8);"><aop:aspectj-autoproxy proxy-target-class="true" expose-proxy="true"/>  
</font><font style="color:rgba(0, 0, 0, 0.8);">2.修改调用代码</font>

```java
public class CaseServiceImpl implements CaseService{
    @Override
    @WorkflowLogAnnotation()
    public void methodOne()
    {
        ((CaseService)AopContext.currentProxy()).methodTwo();
    }

    @Override
    @WorkflowLogAnnotation()
    public void methodTwo()
    {
        ...
    }
}
```

#### <font style="color:rgba(0, 0, 0, 0.8);">二.直接从Spring容器中获取AOP代理后的对象</font>
<font style="color:rgba(0, 0, 0, 0.8);">1.增加SpringContextHolder类</font>

```java
public final class SpringContextHolder implements ApplicationContextAware{

    private static ApplicationContext applicationContext;


    //实现ApplicationContextAware接口的context注入函数, 将其存入静态变量.
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        SpringContextHolder.applicationContext = applicationContext;
    }


    //取得存储在静态变量中的ApplicationContext.
    public static ApplicationContext getApplicationContext() {
        checkApplicationContext();
        return applicationContext;
    }

    //从静态变量ApplicationContext中取得Bean, 自动转型为所赋值对象的类型.
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String name) {
        checkApplicationContext();
        return (T) applicationContext.getBean(name);
    }


    //从静态变量ApplicationContext中取得Bean, 自动转型为所赋值对象的类型.
    //如果有多个Bean符合Class, 取出第一个.
    @SuppressWarnings("unchecked")
    public static <T> T getBean(Class<T> clazz) {
        checkApplicationContext();
        @SuppressWarnings("rawtypes")
        Map beanMaps = applicationContext.getBeansOfType(clazz);
        if (beanMaps!=null && !beanMaps.isEmpty()) {
            return (T) beanMaps.values().iterator().next();
        } else{
            return null;
        }
    }

    private static void checkApplicationContext() {
        if (applicationContext == null) {
            throw new IllegalStateException("applicaitonContext未注入,请在applicationContext.xml中定义SpringContextHolder");
        }
    }

}
```

<font style="color:rgba(0, 0, 0, 0.8);">2.修改代码</font>

```java
public class CaseServiceImpl implements CaseService{
    @Override
    @WorkflowLogAnnotation()
    public void methodOne()
    {
        CaseService caseServiceProxy = (CaseService)SpringContextHolder.getBean("caseService"); 
        caseServiceProxy.methodTwo();
    }

    @Override
    @WorkflowLogAnnotation()
    public void methodTwo()
    {
        ...
    }
}
```

# <font style="color:rgba(0, 0, 0, 0.8);">AOP使用场景</font>
<font style="color:rgba(0, 0, 0, 0.8);">AOP用来封装横切关注点，具体可以在下面的场景中使用:</font>

<font style="color:rgba(0, 0, 0, 0.8);">Authentication 权限  
</font><font style="color:rgba(0, 0, 0, 0.8);">Caching 缓存  
</font><font style="color:rgba(0, 0, 0, 0.8);">Context passing 内容传递  
</font><font style="color:rgba(0, 0, 0, 0.8);">Error handling 错误处理  
</font><font style="color:rgba(0, 0, 0, 0.8);">Lazy loading 懒加载  
</font><font style="color:rgba(0, 0, 0, 0.8);">Debugging 调试  
</font><font style="color:rgba(0, 0, 0, 0.8);">logging, tracing, profiling and monitoring 记录跟踪 优化 校准  
</font><font style="color:rgba(0, 0, 0, 0.8);">Performance optimization 性能优化  
</font><font style="color:rgba(0, 0, 0, 0.8);">Persistence 持久化  
</font><font style="color:rgba(0, 0, 0, 0.8);">Resource pooling 资源池  
</font><font style="color:rgba(0, 0, 0, 0.8);">Synchronization 同步  
</font><font style="color:rgba(0, 0, 0, 0.8);">Transactions 事务</font>

<font style="color:rgba(0, 0, 0, 0.8);">由于水平有限，以上内容如有不对的地方，欢迎指正。  
</font><font style="color:rgba(0, 0, 0, 0.8);">PS：部分内容来自内网</font>

