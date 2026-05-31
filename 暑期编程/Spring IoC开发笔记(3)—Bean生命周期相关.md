## <font style="color:rgba(0, 0, 0, 0.8);">前言</font>
<font style="color:rgba(0, 0, 0, 0.8);">前2年的后端开发，基本是在PandoraBoot(阿里中间件+SpringBoot)应用下做Java后端开发，接手的应用五花八门，前人对于Spring IoC的用法也是八仙过海各显神通。整理对应开发经验和陆陆续续的笔记，对Spring IoC的内容进行系统汇总，内容主要为开发层面，部分会穿插下底层原理：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring IoC开发笔记(1)——Bean的元数据实现<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring Ioc开发笔记(2)——依赖注入相关<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>**<font style="color:rgba(0, 0, 0, 0.8);">Spring IoC开发笔记(3)——Bean生命周期相关</font>**<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring IoC开发笔记(4)——循环依赖&查漏补缺

---

## <font style="color:rgba(0, 0, 0, 0.8);">回调处理—Bean的初始化和销毁</font>
### <font style="color:rgba(0, 0, 0, 0.8);">Bean的元数据定义实现</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring支持在定义bean的元数据时，显式指定初始化/销毁的回调方法；  
</font><font style="color:rgba(0, 0, 0, 0.8);">基于xml形式：</font>

```java
<bean id="testBean" class="com.TestBean" init-method="diyInitial", destroy-method="diyDestroy" />
```

<font style="color:rgba(0, 0, 0, 0.8);">基于javaConfig形式：</font>

```java
public class TestBean {

    public void diyInitial() {
        System.out.println("invoke diyInitial()");
    }

    public void diyDestroy() {
        System.out.println("invoke diyDestroy()");
    }
}
```

```java
@Configuration
public class BeanConfig {

    @Bean(initMethod = "diyInitial", destroyMethod = "diyDestroy")
    public TestBean testBean() {
        System.out.println("start injecting testBean");
        return new TestBean();
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">Spring内置接口实现</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring内置提供两种情况：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">InitializingBean</font><font style="color:rgba(0, 0, 0, 0.8);">：bean实现</font><font style="color:rgba(0, 0, 0, 0.8);">afterPropertiesSet()</font><font style="color:rgba(0, 0, 0, 0.8);">方法完成初始化回调；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">DisposableBean</font><font style="color:rgba(0, 0, 0, 0.8);">：bean实现</font><font style="color:rgba(0, 0, 0, 0.8);">destroy()</font><font style="color:rgba(0, 0, 0, 0.8);">方法完成销毁前回调；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">JSR-250标准实现：@PostConstruct和@PreDestroy</font>
<font style="color:rgba(0, 0, 0, 0.8);">@PostConstruct</font><font style="color:rgba(0, 0, 0, 0.8);">和</font><font style="color:rgba(0, 0, 0, 0.8);">@PreDestroy</font><font style="color:rgba(0, 0, 0, 0.8);">是JSR-250标准引入注解，作用于</font>**<font style="color:rgba(0, 0, 0, 0.8);">无参方法</font>**<font style="color:rgba(0, 0, 0, 0.8);">上，顾名思义表示创建bean之前，销毁bean之后的回调方法；Spring在代码层面使用</font><font style="color:rgba(0, 0, 0, 0.8);">CommonAnnotationBeanPostProcessor</font><font style="color:rgba(0, 0, 0, 0.8);">负责解析执行上述注解。</font>

```java
public class TestBean {
    @PostConstruct
    public void postConstruct() {
        System.out.println("invoke @PostConstruct annotated method");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("invoke @PreDestroy annotated method");
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">各类回调方式执行顺序</font>
<font style="color:rgba(0, 0, 0, 0.8);">本小节总结下同时使用以上回调机制时，各类方法的执行先后顺序：</font>

<font style="color:rgba(0, 0, 0, 0.8);">测试bean:</font>

```java
public class TestBean implements InitializingBean, DisposableBean {

    public void diyInitial() {
        System.out.println("invoke diyInitial()");
    }

    public void diyDestroy() {
        System.out.println("invoke diyDestroy()");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("invoke @PostConstruct annotated method");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("invoke @PreDestroy annotated method");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("invoke afterPropertiesSet()");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("invoke destroy()");
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">配置类：</font>

```java
@Configuration
public class BeanConfig {

    @Bean(initMethod = "diyInitial", destroyMethod = "diyDestroy")
    public TestBean testBean() {
        System.out.println("start injecting testBean");
        return new TestBean();
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">测试类：</font>

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = { BeanConfig.class })
public class BeanInjectTest {

    @Test
    public void printCallBackInfoWhenInjectTestBean() {
        // 仅测试bean注入回调方法执行顺序
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">打印日志：</font>

```java
start injecting testBean
invoke @PostConstruct annotated method
invoke afterPropertiesSet()
invoke diyInitial()
invoke @PreDestroy annotated method
invoke destroy()
invoke diyDestroy()
```

+ **<font style="color:rgba(0, 0, 0, 0.8);">初始化顺序(按时间先后)</font>**<font style="color:rgba(0, 0, 0, 0.8);">：</font><font style="color:rgba(0, 0, 0, 0.8);">@PostConstruct</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">-></font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">InitializingBean#afterPropertiesSet()</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">-> Bean元数据中自定义初始化方法</font>
+ **<font style="color:rgba(0, 0, 0, 0.8);">销毁执行顺序(按时间先后)</font>**<font style="color:rgba(0, 0, 0, 0.8);">：</font><font style="color:rgba(0, 0, 0, 0.8);">@PreDestroy</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">-></font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">DisposableBean#destroy()</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">-> Bean元数据中自定义销毁方法</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">Bean的懒加载机制</font>
### <font style="color:rgba(0, 0, 0, 0.8);">预先加载bean的局限</font>
**<font style="color:rgba(0, 0, 0, 0.8);">预先Bean初始化(eager bean initial)</font>**<font style="color:rgba(0, 0, 0, 0.8);">：默认情况下，Spring容器在启动阶段创建Bean；</font>

<font style="color:rgba(0, 0, 0, 0.8);">默认情况下，Spring IoC容器(ApplicationContext)会自动实例化所有singleton bean并且以缓存形式保存，不自动实例化prototype bean</font>

**<font style="color:rgba(0, 0, 0, 0.8);">预先初始化存在问题</font>**<font style="color:rgba(0, 0, 0, 0.8);">：一些Bean只需在使用时初始化，预先初始化不必要的bean会造成时空开销；针对预先Bean初始化问题，</font>

<font style="color:rgba(0, 0, 0, 0.8);">针对预先加载的问题，Spring支持Bean懒加载机制(lazy Bean initialization)，让Bean在真实起作用时才被加载；</font>

<font style="color:rgba(0, 0, 0, 0.8);">延迟初始化存在问题：无法在服务启动前发现Bean配置错误；</font>

<font style="color:rgba(0, 0, 0, 0.8);">在基于注解和基于Java的配置中，使用</font><font style="color:rgba(0, 0, 0, 0.8);">org.springframework.context.annotation.Lazy</font><font style="color:rgba(0, 0, 0, 0.8);">注解定义延迟创建，</font>

### <font style="color:rgba(0, 0, 0, 0.8);">基于注解配置实现懒加载</font>
```java
@Component
@Lazy(true)
public class TestBean {
 // ...
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">基于JavaConfig实现懒加载</font>
```java
@Configuration
public class BeanConfig {

    @Bean
    @Lazy(true) // 懒加载
    public TestBean testBean() {
        return new TestBean();
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">使用懒加载解决循环依赖问题</font>
<font style="color:rgba(0, 0, 0, 0.8);">假定存在循环依赖关系</font><font style="color:rgba(0, 0, 0, 0.8);">TestA->TestB->TestA</font><font style="color:rgba(0, 0, 0, 0.8);">；使用懒加载机制可循环依赖问题，</font><font style="color:rgba(0, 0, 0, 0.8);">@Lazy</font><font style="color:rgba(0, 0, 0, 0.8);">在代码层面作用于</font><font style="color:rgba(0, 0, 0, 0.8);">Beandefinition</font><font style="color:rgba(0, 0, 0, 0.8);">的</font><font style="color:rgba(0, 0, 0, 0.8);">setLazyInit()</font><font style="color:rgba(0, 0, 0, 0.8);">方法，当注入TestA时，Spring不是直接注入依赖项testB，而是由于TestB启用了懒加载机制而</font>**<font style="color:rgba(0, 0, 0, 0.8);">临时注入一个代理对象</font>**<font style="color:rgba(0, 0, 0, 0.8);">，当testB被真正调用时，才从容器中取出真正的对象进行替换执行；  
</font><font style="color:rgba(0, 0, 0, 0.8);">bean懒加载完成注入时，注入的实际上是一个临时的代理对象；</font>

```java
@Componet
public class TestA {

    @Autowired
    private TestB testB;

    // ...
}
```

```java
@lazy
@Componet
public class TestB {

    @Autowired
    private TestA testA;

    // ...
}
```

---

## <font style="color:rgba(0, 0, 0, 0.8);">参考</font>
<font style="color:rgba(0, 0, 0, 0.8);">1 </font>[Circular Dependencies in Spring, Baeldung Blog](https://www.baeldung.com/circular-dependencies-in-spring)

