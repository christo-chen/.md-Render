## <font style="color:rgba(0, 0, 0, 0.8);">前言</font>
<font style="color:rgba(0, 0, 0, 0.8);">前2年的后端开发，基本是在PandoraBoot(阿里中间件+SpringBoot)应用下做Java后端开发，接手的应用五花八门，前人对于Spring IoC的用法也是八仙过海各显神通。整理对应开发经验和陆陆续续的笔记，对Spring IoC的内容进行系统汇总，内容主要为开发层面，部分会穿插下底层原理：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring IoC开发笔记(1)——Bean的元数据实现<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>**<font style="color:rgba(0, 0, 0, 0.8);">Spring Ioc开发笔记(2)——依赖注入相关</font>**<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring IoC开发笔记(3)——Bean生命周期相关<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring IoC开发笔记(4)——循环依赖&查漏补缺

---

## <font style="color:rgba(0, 0, 0, 0.8);">依赖注入(Dependency Injection)</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring的核心模块之一IoC通过控制反转，将Bean的创建与注入交由容器负责。容器在构造Bean时，会自动分析该Bean的相关依赖项，并将依赖项依次装配填充——所谓「依赖注入」；</font>

<font style="color:rgba(0, 0, 0, 0.8);">本节梳理下Spring依赖注入的不同实现与注意点。</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">依赖注入的实现途径</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring解决依赖注入的工程思想分为：</font>

+ <font style="color:rgba(0, 0, 0, 0.8);">类构造器方式(like Constructor-based)：仿效Java构造器形式；</font>
+ <font style="color:rgba(0, 0, 0, 0.8);">类Setter方式(like Setter-based)：仿效Setter方法形式；</font>

| <font style="color:rgba(0, 0, 0, 0.8);">实现方式</font> | <font style="color:rgba(0, 0, 0, 0.8);">JavaConfig</font> | <font style="color:rgba(0, 0, 0, 0.8);">注解</font> | <font style="color:rgba(0, 0, 0, 0.8);">侧重点</font> |
| --- | --- | --- | --- |
| <font style="color:rgba(0, 0, 0, 0.8);">类构造器(Constructor-based)</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | <font style="color:rgba(0, 0, 0, 0.8);">⭕</font> | <font style="color:rgba(0, 0, 0, 0.8);">侧重于注入</font>**<font style="color:rgba(0, 0, 0, 0.8);">必需依赖项</font>** |
| <font style="color:rgba(0, 0, 0, 0.8);">类Setter(Setter-based)</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | <font style="color:rgba(0, 0, 0, 0.8);">侧重于注入</font>**<font style="color:rgba(0, 0, 0, 0.8);">可选依赖项</font>** |


## <font style="color:rgba(0, 0, 0, 0.8);">依赖注入—基于注解的实现</font>
<font style="color:rgba(0, 0, 0, 0.8);">在Bean内，对依赖项加以注解实现注入；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">相关注解一览</font>
| | <font style="color:rgba(0, 0, 0, 0.8);">适用目标</font> | <font style="color:rgba(0, 0, 0, 0.8);">作用于成员变量</font> | <font style="color:rgba(0, 0, 0, 0.8);">作用于构造器</font> | <font style="color:rgba(0, 0, 0, 0.8);">作用于setter方法</font> | <font style="color:rgba(0, 0, 0, 0.8);">其他</font> | |
| --- | --- | --- | --- | --- | --- | --- |
| <font style="color:rgba(0, 0, 0, 0.8);">@Value</font> | <font style="color:rgba(0, 0, 0, 0.8);">基本数据类型，配置数据</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | | |
| <font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font> | <font style="color:rgba(0, 0, 0, 0.8);">复杂数据类型，byType注入</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font><font style="color:rgba(0, 0, 0, 0.8);">，支持多个入参对象注入</font> | | |
| <font style="color:rgba(0, 0, 0, 0.8);">@Resource</font> | <font style="color:rgba(0, 0, 0, 0.8);">复杂数据类型，byName注入</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font> | <font style="color:rgba(0, 0, 0, 0.8);">⭕</font> | <font style="color:rgba(0, 0, 0, 0.8);">✅</font><font style="color:rgba(0, 0, 0, 0.8);">，只支持单个入参对象注入</font> | | |
| ~~<font style="color:rgba(0, 0, 0, 0.8);">@Inject</font>~~ | ~~<font style="color:rgba(0, 0, 0, 0.8);">JSR-330标准，暂不讨论</font>~~ | ~~<font style="color:rgba(0, 0, 0, 0.8);">暂不讨论</font>~~ | ~~<font style="color:rgba(0, 0, 0, 0.8);">暂不讨论</font>~~ | ~~<font style="color:rgba(0, 0, 0, 0.8);">暂不讨论</font>~~ | | |


### <font style="color:rgba(0, 0, 0, 0.8);">注入配置数据: @Value</font>
<font style="color:rgba(0, 0, 0, 0.8);">@Value("${属性名}")</font><font style="color:rgba(0, 0, 0, 0.8);">用于加载</font>**<font style="color:rgba(0, 0, 0, 0.8);">配置属性</font>**<font style="color:rgba(0, 0, 0, 0.8);">；使用</font><font style="color:rgba(0, 0, 0, 0.8);">@PropertySource</font><font style="color:rgba(0, 0, 0, 0.8);">加载指定配置文件，否则Spring Boot默认从</font><font style="color:rgba(0, 0, 0, 0.8);">application.properties</font><font style="color:rgba(0, 0, 0, 0.8);">来获取配置的属性值；</font>

<font style="color:rgba(0, 0, 0, 0.8);">配置文件</font><font style="color:rgba(0, 0, 0, 0.8);">application.properties</font><font style="color:rgba(0, 0, 0, 0.8);">：</font>

```java
config.name=tom
config.desc=boy
```

<font style="color:rgba(0, 0, 0, 0.8);">测试Bean</font><font style="color:rgba(0, 0, 0, 0.8);">TestBean</font><font style="color:rgba(0, 0, 0, 0.8);">：</font>

```java
@Componet
public class TestBean {

    @Value("${config.name}")
    private String name;

    private String desc;

    public TestBean(@Value("${config.desc}") String desc) {
        this.desc = desc;
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">指定加载配置文件</font><font style="color:rgba(0, 0, 0, 0.8);">com/property/user.properties</font><font style="color:rgba(0, 0, 0, 0.8);">,</font>

```java
@PropertySource("classpath:com/property/user.properties")
@Componet
public class TestBean {

    @Value("${user.name}")
    private String name;

    // ...
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">按类型依赖注入: @Autowired</font>
<font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">对依赖项按照其类型(byType)进行注入；</font>

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
    boolean required() default true;
}
```

<font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">的</font><font style="color:rgba(0, 0, 0, 0.8);">required</font><font style="color:rgba(0, 0, 0, 0.8);">属性表示作用依赖项是否必需，</font>**<font style="color:rgba(0, 0, 0, 0.8);">默认必需</font>**<font style="color:rgba(0, 0, 0, 0.8);">；当显式指定</font><font style="color:rgba(0, 0, 0, 0.8);">required=false</font><font style="color:rgba(0, 0, 0, 0.8);">时，对应依赖项不在IoC中时，不会进行对应注入操作；</font>

```java
public class TestObject {

    /**
     * direct injection
     */
    @Autowired
    private DependentObject dependency1;

    /**
     * inject all beans of AbstractService which is a abstract class or interface,
     * the keys contains all bean names,
     * the value contains the corresponding specific bean;
     */
    @Autowired
    private Map<String, AbstractService> serviceSet;

    /**
     * inject all beans of AbstractService
     */
    @Autowired
    private Set<AbstractService> serviceSet;

    /**
     * inject all beans of AbstractService
     */
    @Autowired
    private Set<AbstractService> serviceList;

    /**
     * inject all beans of AbstractService
     */
     /
    @Autowired
    private AbstractService[] serviceArray;

    private DependentObject1 dependency2;

    private DependentObject1 dependency3;

    /**
     * constructor-based injection
     * the annotated constructor does not have to be public
     */
    @Autowired
    public TestObject(DependentObject dependency2) {
        this.dependency2 = dependency2;
    }

    /**
     * setter-based injection
     */
    @Autowired
    public void setDependency3(DependentObject dependency3) {
        this.dependency3 = dependency3;
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">按名称依赖注入: @Resource</font>
<font style="color:rgba(0, 0, 0, 0.8);">@Resource</font><font style="color:rgba(0, 0, 0, 0.8);">根据</font><font style="color:rgba(0, 0, 0, 0.8);">name</font><font style="color:rgba(0, 0, 0, 0.8);">属性对拥有对应名称的bean进行注入，</font>**<font style="color:rgba(0, 0, 0, 0.8);">默认为成员变量名称或者setter方法的入参名称</font>**<font style="color:rgba(0, 0, 0, 0.8);">；</font>

```java
public class TestObject {

    /**
     * 默认按照name = dependency1注入
     */
    @Resource
    private Dependency dependency1

    private Dependency dependency2;

    private Dependency dependency3;

    /**
     * 默认按照name = dependency2注入
     */
    @Resource
    public void setDependency(Dependency dependency2) {
        this.dependency2 = dependency2
    }
   
    /**
     * 显式指定bean名称，按照specificDependency查找并注入dependency3
     */
    @Resource(name = "specificDependency")
    public void setDependency(Dependency dependency3) {
        this.dependency3 = dependency3
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">多依赖项选择问题</font>
<font style="color:rgba(0, 0, 0, 0.8);">当依赖项在IoC存在多个待选Bean时，如何选择？</font>

#### <font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font>
+ <font style="color:rgba(0, 0, 0, 0.8);">对于多个候选bean，</font><font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">隐式按照bean id去匹配，建议使用</font><font style="color:rgba(0, 0, 0, 0.8);">Qualifier</font><font style="color:rgba(0, 0, 0, 0.8);">显式指定；</font>
+ <font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">可通过额外的</font><font style="color:rgba(0, 0, 0, 0.8);">@Qualifier</font><font style="color:rgba(0, 0, 0, 0.8);">指定名称进行唯一注入；</font>

#### <font style="color:rgba(0, 0, 0, 0.8);">@Resource</font>
<font style="color:rgba(0, 0, 0, 0.8);">@Resource有两个属性</font><font style="color:rgba(0, 0, 0, 0.8);">name</font><font style="color:rgba(0, 0, 0, 0.8);">与</font><font style="color:rgba(0, 0, 0, 0.8);">type</font><font style="color:rgba(0, 0, 0, 0.8);">(默认Object类型)，装配顺序如下：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 同时指定name与type，Spring从上下文中唯一确定bean进行装配，找不到抛出异常；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 仅指定name，从上下文中查找bean ID匹配的bean进行装配，找不到抛出异常；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 仅指定type，从上下文中找到类型匹配的的唯一bean进行装配，找不到或不唯一抛出异常；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 如果name与type均未指定，默认按照name进行装配，如果没有匹配则回退到原始类型进行匹配，匹配成功则进行自动装配；</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">依赖注入—基于JavaConfig的实现</font>
### <font style="color:rgba(0, 0, 0, 0.8);">基本注入实现</font>
<font style="color:rgba(0, 0, 0, 0.8);">依赖项如果是  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 基本数据类型：显式按照</font><font style="color:rgba(0, 0, 0, 0.8);">@Value</font><font style="color:rgba(0, 0, 0, 0.8);">限定名称从配置文件里加载值；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 非基本类型：</font>**<font style="color:rgba(0, 0, 0, 0.8);">隐式</font>**<font style="color:rgba(0, 0, 0, 0.8);">按照Autowired注入，可使用</font><font style="color:rgba(0, 0, 0, 0.8);">@Qualifier</font><font style="color:rgba(0, 0, 0, 0.8);">限定名称；</font>

```java
/**
 * 基于JavaConfig
 * Constructor-based方式依赖注入
 * @author yuanjie
 */
@Configuration
public class AppConfig {

    /**
     * 假定Bean TransferService无任何依赖项
     */
    @Bean
    public TransferService transferServiceWithoutDependency() {
        return new TransferServiceImpl();
    }

    /**
     *  accountRepository按照Autowired方式注入，但无需@Autowired
     */
    @Bean
    public TransferService transferServiceOne(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }

    @Bean
    public TransferService transferServiceTwo(@Value("${service.name}")String name,
    @Qualifier("account1") AccountRepository accountRepository) {
        return new TransferServiceImpl(name, accountRepository);
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">导入其他不同方式下的依赖Bean</font>
<font style="color:rgba(0, 0, 0, 0.8);">如</font>[Spring IoC开发笔记(1)](https://yzwall.co/2020/07/05/spring-ioc-1/#%E5%AF%BC%E5%85%A5%E5%85%B6%E4%BB%96%E4%B8%8D%E5%90%8C%E6%96%B9%E5%BC%8F%E4%B8%8B%E7%9A%84bean%E5%85%83%E6%95%B0%E6%8D%AE)<font style="color:rgba(0, 0, 0, 0.8);">所谈，当依赖项的元数据已经在其他Config类/XML中定义完毕，可以使用以下方式导入：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 基于注解方式：使用</font><font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">或</font><font style="color:rgba(0, 0, 0, 0.8);">@Resource</font><font style="color:rgba(0, 0, 0, 0.8);">以成员变量形式注入；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 基于JavaConfig方式：使用</font><font style="color:rgba(0, 0, 0, 0.8);">@Import</font><font style="color:rgba(0, 0, 0, 0.8);">导入其他config类定义好的bean，使用基于注解方式显式注入;  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 基于XML方式：使用</font><font style="color:rgba(0, 0, 0, 0.8);">@ImportResource</font><font style="color:rgba(0, 0, 0, 0.8);">扫描xml中定义好的bean，使用基于注解方式显式注入；</font>

<font style="color:rgba(0, 0, 0, 0.8);">🔔</font><font style="color:rgba(0, 0, 0, 0.8);"> 推荐使用基于注解方式显式注入依赖项，而不是前文所述仿效构造器形式，入参由Spring黑盒处理注入</font>

<font style="color:rgba(0, 0, 0, 0.8);">假定TransferService分别依赖dependency1，dependency2和dependency3，其中：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ dependency1在xml中装配完毕；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ dependency2和dependency4在其他JavaConfig中装配完毕；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ dependency3直接使用@Componet标识bean；</font>

<font style="color:rgba(0, 0, 0, 0.8);">dependency1: 基于xml，</font><font style="color:rgba(0, 0, 0, 0.8);">beans-another.xml</font><font style="color:rgba(0, 0, 0, 0.8);">:</font>

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="dependency1" class="com.xxx.Dependency1">
        <!-- ... -->
    </bean>
</beans>
```

<font style="color:rgba(0, 0, 0, 0.8);">dependency2和dependency4: 基于JavaConfig</font>

```java
package com.xxx;

@Configuration
public class AnotherConfig {

    @Bean
    public Dependency2 dependency2() {
        Dependency2 dependency2 = new Dependency2();
        return dependency2;
    }

    @Bean
    public Dependency4 dependency4() {
        Dependency2 dependency4 = new Dependency4();
        return dependency4;
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">dependency3: 基于注解形式</font>

```java
@Component
public class Dependency3 {
    // ...
}
```

```java
@Configuration
@Import({Dependency2.class, Dependency4.class})
@ImportResource("classpath:/com/config/beans-another.xml")
public class BeanConfig {
   
    @Autowired
    Dependency1 dependency1;

    @Autowired
    Dependency2 dependency2;

    @Autowired
    Dependency3 dependency3;

    @Bean
    public TransferService transferService() {
        TransferService service = new TransferServiceImpl();
        service.setDependency1(dependency1);
        service.setDependency2(dependency2);
        service.setDependency3(dependency3);
        return service;
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">同属一个Config类下的Bean之间的依赖注入</font>
<font style="color:rgba(0, 0, 0, 0.8);">如下边代码所示，</font><font style="color:rgba(0, 0, 0, 0.8);">beanOne</font><font style="color:rgba(0, 0, 0, 0.8);">构造时，引用了</font><font style="color:rgba(0, 0, 0, 0.8);">beanTwo()</font><font style="color:rgba(0, 0, 0, 0.8);">构造方法，Spring会在构造</font><font style="color:rgba(0, 0, 0, 0.8);">beanOne</font><font style="color:rgba(0, 0, 0, 0.8);">时，先将</font><font style="color:rgba(0, 0, 0, 0.8);">beanTwo</font><font style="color:rgba(0, 0, 0, 0.8);">构造完毕后注入到</font><font style="color:rgba(0, 0, 0, 0.8);">beanOne</font><font style="color:rgba(0, 0, 0, 0.8);">中；</font>

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">从配置文件中导入数据</font>
<font style="color:rgba(0, 0, 0, 0.8);">@PropertySource</font><font style="color:rgba(0, 0, 0, 0.8);">搭配</font><font style="color:rgba(0, 0, 0, 0.8);">@Value</font><font style="color:rgba(0, 0, 0, 0.8);">实现；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">多依赖项选择问题</font>
<font style="color:rgba(0, 0, 0, 0.8);">当注入时出现多个候选Bean时，可使用</font><font style="color:rgba(0, 0, 0, 0.8);">@Primary</font><font style="color:rgba(0, 0, 0, 0.8);">指定具体Bean为选择第一顺位，否则抛出异常：</font>

```java
@Configuration
public class AppConfig {

    /**
     * 指定该Bean为注入TransferService的第一选择
     */
    @Primary
    @Bean
    public TransferService transferServiceWithoutDependency() {
        return new TransferServiceImpl();
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">条件注入—@Conditional</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring提供注解</font><font style="color:rgba(0, 0, 0, 0.8);">@Conditional</font><font style="color:rgba(0, 0, 0, 0.8);">和</font><font style="color:rgba(0, 0, 0, 0.8);">@ConditionalOnXXX</font><font style="color:rgba(0, 0, 0, 0.8);">，支持</font>**<font style="color:rgba(0, 0, 0, 0.8);">满足自定义条件</font>**<font style="color:rgba(0, 0, 0, 0.8);">才进行Bean创建；  
</font><font style="color:rgba(0, 0, 0, 0.8);">对于日常开发，一般Spring内置的</font><font style="color:rgba(0, 0, 0, 0.8);">ConditionalOnXXX</font><font style="color:rgba(0, 0, 0, 0.8);">注解即可满足大部分要求；  
</font><font style="color:rgba(0, 0, 0, 0.8);">这里</font>[有篇文章](https://www.javazhiyin.com/44985.html)<font style="color:rgba(0, 0, 0, 0.8);">汇总的比较全面，可以参考;</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">规定Bean依赖关系—@DependsOn</font>
<font style="color:rgba(0, 0, 0, 0.8);">理论上Spring IoC会按照依赖关系进行递归注入</font>

<font style="color:rgba(0, 0, 0, 0.8);">在编程层面，Spring提供注解</font><font style="color:rgba(0, 0, 0, 0.8);">@DependsOn</font><font style="color:rgba(0, 0, 0, 0.8);">保证bean的依赖项全部注入完毕后，当前bean才开始注入；  
</font><font style="color:rgba(0, 0, 0, 0.8);">假定</font><font style="color:rgba(0, 0, 0, 0.8);">TransferService</font><font style="color:rgba(0, 0, 0, 0.8);">依赖</font><font style="color:rgba(0, 0, 0, 0.8);">Dependency1</font><font style="color:rgba(0, 0, 0, 0.8);">，</font><font style="color:rgba(0, 0, 0, 0.8);">Dependency2</font><font style="color:rgba(0, 0, 0, 0.8);">。</font>

```java
@Component("dependency1")
public class Dependency1 {
    // ...
}
```

```java
@Component("anotherDependency2")
public class Dependency2 {
    // ...
}
```

<font style="color:rgba(0, 0, 0, 0.8);">@DependsOn</font><font style="color:rgba(0, 0, 0, 0.8);">与</font><font style="color:rgba(0, 0, 0, 0.8);">@Component</font><font style="color:rgba(0, 0, 0, 0.8);">搭配：</font>

```java
@Configuration
public class AppConfig {

    @DependsOn({"dependency1", "anotherDependency2"})
    @Bean
    public TransferService transferService {
        // ...
    }
}
```

<font style="color:rgba(0, 0, 0, 0.8);">@DependsOn</font><font style="color:rgba(0, 0, 0, 0.8);">在JavaConfig中的应用：</font>

**<font style="color:rgb(153, 153, 153);">@Configuration</font>**<font style="color:rgba(0, 0, 0, 0.8);"> </font>**<font style="color:rgb(51, 51, 51);">public</font>**<font style="color:rgba(0, 0, 0, 0.8);"> </font>**<font style="color:rgb(51, 51, 51);">class</font>**<font style="color:rgba(0, 0, 0, 0.8);"> </font>**<font style="color:rgb(153, 0, 0);">AppConfig</font>**<font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">{ </font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">    </font>**<font style="color:rgb(153, 153, 153);">@DependsOn({"dependency1", "anotherDependency2"})</font>**<font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">    </font>**<font style="color:rgb(153, 153, 153);">@Bean</font>**<font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">    </font>**<font style="color:rgb(51, 51, 51);">public</font>**<font style="color:rgba(0, 0, 0, 0.8);"> TransferService transferService { </font><font style="color:rgba(0, 0, 0, 0.8);">        </font>_<font style="color:rgb(153, 153, 136);">// ...</font>_<font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">    } } </font>

<font style="color:rgba(0, 0, 0, 0.8);">值得注意的是</font><font style="color:rgba(0, 0, 0, 0.8);">@DependsOn</font><font style="color:rgba(0, 0, 0, 0.8);">只能保证依赖bean已被创建，对应成员变量已被填充，</font>**<font style="color:rgba(0, 0, 0, 0.8);">不保证</font>**<font style="color:rgba(0, 0, 0, 0.8);">依赖bean的</font><font style="color:rgba(0, 0, 0, 0.8);">@PostConstruct</font><font style="color:rgba(0, 0, 0, 0.8);">方法已被执行。</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">JSR标准注解</font>
<font style="color:rgba(0, 0, 0, 0.8);">在JSR标准中，  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">@Inject</font><font style="color:rgba(0, 0, 0, 0.8);">(JSR-330)等同于Spring中的</font><font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">;  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">@Named</font><font style="color:rgba(0, 0, 0, 0.8);">(JSR-330)或者</font><font style="color:rgba(0, 0, 0, 0.8);">@ManagedBean</font><font style="color:rgba(0, 0, 0, 0.8);">(JSR-250)等同于</font><font style="color:rgba(0, 0, 0, 0.8);">@Component</font><font style="color:rgba(0, 0, 0, 0.8);">；</font>

<font style="color:rgba(0, 0, 0, 0.8);">除此之外，Spring提供了标准之外的其他实现，比如</font><font style="color:rgba(0, 0, 0, 0.8);">@Value</font><font style="color:rgba(0, 0, 0, 0.8);">等；JSR标准注解的局限</font>[戳这里](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-standard-annotations)<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">从标准与实现的关系来讲，应该尽可能使用标准而非实现细节，这样当存在一个比Spring更优越的框架出现时，业务代码由于是框架无关的，因此变更成本较小。但是目前JavaEE领域几乎早已是Sprin全家桶的天下，Spring的实现已逐渐成为JavaEE开发的实际标准</font><font style="color:rgba(0, 0, 0, 0.8);">😅</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">参考</font>
<font style="color:rgba(0, 0, 0, 0.8);">1 </font>[Spring官方文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-annotation-config)

