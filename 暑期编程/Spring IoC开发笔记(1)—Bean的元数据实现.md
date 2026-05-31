## <font style="color:rgba(0, 0, 0, 0.8);">前言</font>
<font style="color:rgba(0, 0, 0, 0.8);">前2年的后端开发，基本是在PandoraBoot(阿里中间件+SpringBoot)应用下做Java后端开发，接手的应用五花八门，前人对于Spring IoC的用法也是八仙过海各显神通。整理对应开发经验和陆陆续续的笔记，对Spring IoC的内容进行系统汇总，内容主要为开发层面，部分会穿插下底层原理：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ </font>**<font style="color:rgba(0, 0, 0, 0.8);">Spring IoC开发笔记(1)——Bean的元数据实现</font>**<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ </font>Spring Ioc开发笔记(2)——依赖注入相关<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ </font>Spring IoC开发笔记(3)——Bean生命周期相关<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ </font>Spring IoC开发笔记(4)——循环依赖&查漏补缺

---

## <font style="color:rgba(0, 0, 0, 0.8);">控制反转(IoC)与Bean</font>
### <font style="color:rgba(0, 0, 0, 0.8);">控制反转(Inversion Of Controll)</font>
<font style="color:rgba(0, 0, 0, 0.8);">在普通的Java开发中，一个类依赖另一个类时，必须在当前类中调用依赖类的构造方法or工厂方式进行实例化，导致大量耦合，当系统复杂到一定程度后问题比较严重；所以催生出使用一个容器对系统内所有类的实例化进行统一管理，以实现类间解耦的工程思路，即为</font>**<font style="color:rgba(0, 0, 0, 0.8);">控制反转</font>**<font style="color:rgba(0, 0, 0, 0.8);">。</font>

**<font style="color:rgba(0, 0, 0, 0.8);">控制反转(Inversion Of Controll, IoC)</font>**<font style="color:rgba(0, 0, 0, 0.8);">：组件类不是通过操作符自行实例化依赖组件，而是在运行中由</font>**<font style="color:rgba(0, 0, 0, 0.8);">专门的第三方容器</font>**<font style="color:rgba(0, 0, 0, 0.8);">将依赖组件注入(</font>**<font style="color:rgba(0, 0, 0, 0.8);">依赖注入</font>**<font style="color:rgba(0, 0, 0, 0.8);">)，因此依赖项的控制权由组件转移到容器（</font>**<font style="color:rgba(0, 0, 0, 0.8);">控制反转</font>**<font style="color:rgba(0, 0, 0, 0.8);">），消除调用类对依赖组件的依赖（</font>**<font style="color:rgba(0, 0, 0, 0.8);">解耦</font>**<font style="color:rgba(0, 0, 0, 0.8);">）；</font>

<font style="color:rgba(0, 0, 0, 0.8);">Spring的核心模块之一IoC是依赖注入问题的典型实现，实现了</font>**<font style="color:rgba(0, 0, 0, 0.8);">控制反转</font>**<font style="color:rgba(0, 0, 0, 0.8);">；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">Spring Bean</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring官方将Bean定义为由Spring IoC容器负责注册/装配的</font>**<font style="color:rgba(0, 0, 0, 0.8);">对象</font>**<font style="color:rgba(0, 0, 0, 0.8);">，笼统地讲就是容器的操作对象。</font>

<font style="color:rgba(0, 0, 0, 0.8);">In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container.  
</font><font style="color:rgba(0, 0, 0, 0.8);">——《Spring Framework documentation》</font>

<font style="color:rgba(0, 0, 0, 0.8);">如下图所示，</font>**<font style="color:rgba(0, 0, 0, 0.8);">开发者通过对业务对象(POJO对象)配置对应的Bean元数据信息，提交给IoC容器进行统一调度管理；</font>**<font style="color:rgba(0, 0, 0, 0.8);">  
</font><!-- 这是一张图片，ocr 内容为：YOUR BUSINESS OBJECTS(POJOS) THE SPRING CONFIGURATION CONTAINER METADATA PRODUCES FULLY CONFIGURED SYSTEM READY FOR USE -->
![](https://cdn.nlark.com/yuque/0/2022/png/479089/1645795521126-8c07a691-f7f4-4c7a-a906-c0ab66195d95.png)

### <font style="color:rgba(0, 0, 0, 0.8);">Spring Bean/JavaBean/POJO辨析</font>
<font style="color:rgba(0, 0, 0, 0.8);">学计算机的整出考据的味道了…</font>

+ <font style="color:rgba(0, 0, 0, 0.8);">JavaBean最早是Sun公司在1996年提出，用于EJB开发，制订了严格的编程规范，开发细节繁琐，成本高昂；</font>
+ **<font style="color:rgba(0, 0, 0, 0.8);">POJO(Plain Old Java Object)</font>**<font style="color:rgba(0, 0, 0, 0.8);">:直译为</font>**<font style="color:rgba(0, 0, 0, 0.8);">简单Java对象</font>**<font style="color:rgba(0, 0, 0, 0.8);">，针对EJB编程模型存在问题而提出，是为了避免与EJB混淆所创造的简称；主张尽可能简单；</font>**<font style="color:rgba(0, 0, 0, 0.8);">在实际的Spring-based Application开发中，绝大多数的对象均为POJO对象</font>**<font style="color:rgba(0, 0, 0, 0.8);">；</font>
+ <font style="color:rgba(0, 0, 0, 0.8);">Spring Bean是Spring管理的对象概念。之所以取名Bean，是因为Spring诞生的初衷是为了简化EJB开发，因此当时的管理目标就是JavaBean。后来随着Spring的不断演进，Spring Bean可以是JavaBean/POJO/其他对象；</font>

<font style="color:rgba(0, 0, 0, 0.8);">JavaBean和POJO详细对比</font>[戳这里](https://www.geeksforgeeks.org/pojo-vs-java-beans/)

---

## <font style="color:rgba(0, 0, 0, 0.8);">Bean的元数据(BeanDefinition)</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring IoC容器通过数据机构</font><font style="color:rgba(0, 0, 0, 0.8);">Beandefinition</font><font style="color:rgba(0, 0, 0, 0.8);">——即Bean的元数据，</font><font style="color:#DF2A3F;">来注册/管理/装配业务系统中的各个Bean</font><font style="color:rgba(0, 0, 0, 0.8);">，Spring框架面向开发者提供多种方式创建并提交Bean的元数据给Spring IoC容器，Spring IoC容器通过管理Bean的元数据实现依赖注入等功能。</font>

<font style="color:rgba(0, 0, 0, 0.8);">在物理上，无论以哪一种方式创建Bean的元数据，最终在底层都被转换为</font><font style="color:rgba(0, 0, 0, 0.8);">BeanDefinition</font><font style="color:rgba(0, 0, 0, 0.8);">结构，加载进Spring IoC容器进行统一管理；所以</font>**<font style="color:rgba(0, 0, 0, 0.8);">不同方式创建的Bean的元数据在依赖注入阶段可以混用</font>**<font style="color:rgba(0, 0, 0, 0.8);">😀</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">构造Bean元数据的不同途径</font>
<font style="color:rgba(0, 0, 0, 0.8);">IoC容器通过</font>**<font style="color:rgba(0, 0, 0, 0.8);">Bean的元数据(configuration metadata)</font>**<font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">注入Bean；实现途径有：XML，JavaConfig和基于注解；  
</font><font style="color:rgba(0, 0, 0, 0.8);">1. 基于XML方式在xml文件中定义Bean的元数据，与Bean实现类分离，Bean的实现和JavaSE下的普通类一样，</font>**<font style="color:rgba(0, 0, 0, 0.8);">不感知</font>**<font style="color:rgba(0, 0, 0, 0.8);">Spring IoC的存在；  
</font><font style="color:rgba(0, 0, 0, 0.8);">2.</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>**<font style="color:rgba(0, 0, 0, 0.8);">基于注解方式</font>**<font style="color:rgba(0, 0, 0, 0.8);">只需要在实现类上通过注解配置元数据，配置与实现一体；  
</font><font style="color:rgba(0, 0, 0, 0.8);">3.</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>**<font style="color:rgba(0, 0, 0, 0.8);">基于JavaConfig方式</font>**<font style="color:rgba(0, 0, 0, 0.8);">通过注解+POJO类配置元数据，配置和实现一体；  
</font><font style="color:rgba(0, 0, 0, 0.8);">4. 基于Groovy DSL由Spring 4.0引入；</font>

<font style="color:rgba(0, 0, 0, 0.8);">值得注意的是，无论以哪一种方式声明Bean，最终在底层都被转换为</font><font style="color:rgba(0, 0, 0, 0.8);">BeanDefinition</font><font style="color:rgba(0, 0, 0, 0.8);">，加载进Spring IoC容器进行统一管理；所以</font>**<font style="color:rgba(0, 0, 0, 0.8);">不同方式声明的Bean在依赖注入阶段可以混用</font>**<font style="color:rgba(0, 0, 0, 0.8);">😀</font>

### <font style="color:rgba(0, 0, 0, 0.8);">不同方式V.S</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring为替代EJB而产生，尽管组件代码是轻量级的，但是对应的配置成本高昂。Spring一开始大量用XML配置，Spring 2.5引入基于注解的组件扫描，Spring 3.0引入基于Java的配置；而</font>**<font style="color:rgba(0, 0, 0, 0.8);">Spring Boot顺应简化配置趋势，核心思想是试图通过注解和JavaConfig实现无XML配置，因此顺应演变趋势，选择方式2/3对IoC内容进行汇总</font>**<font style="color:rgba(0, 0, 0, 0.8);">；</font>

| <font style="color:rgba(0, 0, 0, 0.8);">注入方式</font> | <font style="color:rgba(0, 0, 0, 0.8);">对Bean的侵入性</font> | <font style="color:rgba(0, 0, 0, 0.8);">其他</font> | |
| --- | --- | --- | --- |
| <font style="color:rgba(0, 0, 0, 0.8);">1. 基于XML方式</font> | <font style="color:rgba(0, 0, 0, 0.8);">无，需要单独编写XML配置文件</font> | <font style="color:rgba(0, 0, 0, 0.8);">Spring早期版本采用，</font>**<font style="color:rgba(0, 0, 0, 0.8);">统一配置，配置成本较高</font>** | |
| **<font style="color:rgba(0, 0, 0, 0.8);">2. 基于JavaConfig</font>** | <font style="color:rgba(0, 0, 0, 0.8);">无，需要单独写配置类，但不侵入Bean</font> | <font style="color:rgba(0, 0, 0, 0.8);">相当于基于XML方式的Java代码版，对Java程序员友好，</font>**<font style="color:rgba(0, 0, 0, 0.8);">统一配置，配置成本较高</font>** | |
| **<font style="color:rgba(0, 0, 0, 0.8);">3. 基于类扫描方式</font>** | <font style="color:rgba(0, 0, 0, 0.8);">高，侵入Bean，不需要单独写配置类</font> | <font style="color:rgba(0, 0, 0, 0.8);">符合SpringBoot精神，开发者可显式控制，不需要单独写配置类，不足是将配置</font>**<font style="color:rgba(0, 0, 0, 0.8);">分散</font>**<font style="color:rgba(0, 0, 0, 0.8);">在了每个Bean中，难以统一管理</font> | |
| ~~<font style="color:rgba(0, 0, 0, 0.8);">4. 基于Groovy的方式</font>~~ | ~~<font style="color:rgba(0, 0, 0, 0.8);">暂不讨论</font>~~ | ~~<font style="color:rgba(0, 0, 0, 0.8);">暂不讨论</font>~~ | |


<font style="color:rgba(0, 0, 0, 0.8);">因此依赖注入方式的选择本质上是</font>**<font style="color:rgba(0, 0, 0, 0.8);">代码侵入性</font>**<font style="color:rgba(0, 0, 0, 0.8);">和</font>**<font style="color:rgba(0, 0, 0, 0.8);">配置成本</font>**<font style="color:rgba(0, 0, 0, 0.8);">之间做trade-off；</font>

<font style="color:rgba(0, 0, 0, 0.8);">在日常开发中根据实际需要可能需要多种注入方式</font>**<font style="color:rgba(0, 0, 0, 0.8);">混合使用</font>**<font style="color:rgba(0, 0, 0, 0.8);">(事实上在一些有年头的Spring应用里，基本是各种方式混着来…)；</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">基于JavaConfig方式</font>
<font style="color:rgba(0, 0, 0, 0.8);">基于JavaConfig方式定义Bean在Spring 3.0中引入，该方式通过</font><font style="color:rgba(0, 0, 0, 0.8);">@Configuration</font><font style="color:rgba(0, 0, 0, 0.8);">与</font><font style="color:rgba(0, 0, 0, 0.8);">@Bean</font><font style="color:rgba(0, 0, 0, 0.8);">进行配置；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">@Configuration</font><font style="color:rgba(0, 0, 0, 0.8);">，等同于XML中的</font><font style="color:rgba(0, 0, 0, 0.8);"><beans/></font><font style="color:rgba(0, 0, 0, 0.8);">，自身已注解</font><font style="color:rgba(0, 0, 0, 0.8);">@Component</font><font style="color:rgba(0, 0, 0, 0.8);">，作用是声明当前类负责在代码层面对Bean进行实例化，在ComponetScan时，类中所有</font><font style="color:rgba(0, 0, 0, 0.8);">@Bean</font><font style="color:rgba(0, 0, 0, 0.8);">方法会被全部调用生成</font><font style="color:rgba(0, 0, 0, 0.8);">Beandefinition</font><font style="color:rgba(0, 0, 0, 0.8);">，交给Spring IoC容器管理；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">@Bean</font><font style="color:rgba(0, 0, 0, 0.8);">等同于XML中的</font><font style="color:rgba(0, 0, 0, 0.8);"><bean/></font><font style="color:rgba(0, 0, 0, 0.8);">，作用是将配置类中的成员方法标记为指定Bean的实例化逻辑；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ Bean类型：Bean的类型与方法返回值类型相同；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ Bean默认名称：Bean名称</font>**<font style="color:rgba(0, 0, 0, 0.8);">默认和方法名相同</font>**<font style="color:rgba(0, 0, 0, 0.8);">；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ Bean指定名称：通过@Bean的</font><font style="color:rgba(0, 0, 0, 0.8);">name</font><font style="color:rgba(0, 0, 0, 0.8);">属性显式指定名称；</font>

```java
import com.test.TestBean;

@Configuration
public class BeanConfig {
    @Bean
    public TestBean testBean() {
        return new TestBean();
    }

    @Bean(name="anotherTestBean")
    public TestBean testBean1() {
        return new TestBean("test");
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">导入其他不同方式下的bean元数据</font>
+ <font style="color:rgba(0, 0, 0, 0.8);">使用</font><font style="color:rgba(0, 0, 0, 0.8);">@ImportResource</font><font style="color:rgba(0, 0, 0, 0.8);">导入XML中的bean的元数据：</font>
+ <font style="color:rgba(0, 0, 0, 0.8);">使用</font><font style="color:rgba(0, 0, 0, 0.8);">@ComponetScan</font><font style="color:rgba(0, 0, 0, 0.8);">导入指定包路径下的bean的元数据；</font>

```java
@Configuration
@ComponetScan(basePackages="com.xxx")
@ImportResource(value = "beans-another.xml")
public class BeanConfig {
    // xxx
}
```

<font style="color:rgba(0, 0, 0, 0.8);">@Bean</font><font style="color:rgba(0, 0, 0, 0.8);">方法在语法上也支持在非@Configuration类中使用，但是Spring官方极力不推荐，因为:</font>

<font style="color:rgba(0, 0, 0, 0.8);">Each such method is literally only a factory method for a particular bean reference, without any special runtime semantics</font>

<font style="color:rgba(0, 0, 0, 0.8);">只能被当做工厂方法使用，并且在运行时不会和IoC容器发生关系，而且为了避免其他莫名奇妙的bug，</font>**<font style="color:rgba(0, 0, 0, 0.8);">推荐强制@Bean和@Conguration一起搭配使用</font>**<font style="color:rgba(0, 0, 0, 0.8);">；</font>

<font style="color:rgba(0, 0, 0, 0.8);">所有Bean方法在执行过程中会被CGLIB进行代理，会对方法进行override，因此</font><font style="color:rgba(0, 0, 0, 0.8);">@Bean</font><font style="color:rgba(0, 0, 0, 0.8);">方法的权限修饰符不能出现</font><font style="color:rgba(0, 0, 0, 0.8);">private</font><font style="color:rgba(0, 0, 0, 0.8);">和</font><font style="color:rgba(0, 0, 0, 0.8);">final</font><font style="color:rgba(0, 0, 0, 0.8);">；</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">基于路径扫描方式</font>
<font style="color:rgba(0, 0, 0, 0.8);">基于路径扫描方式的思想是：  
</font><font style="color:rgba(0, 0, 0, 0.8);">1. 通过</font><font style="color:rgba(0, 0, 0, 0.8);">@Component</font><font style="color:rgba(0, 0, 0, 0.8);">及其衍生注解对Bean进行显式标识；  
</font><font style="color:rgba(0, 0, 0, 0.8);">2. 通过扫描上述Bean的包路径，让Spring IoC容器(</font><font style="color:rgba(0, 0, 0, 0.8);">ApplicationContext</font><font style="color:rgba(0, 0, 0, 0.8);">)可以检测并管理对应Bean(</font><font style="color:rgba(0, 0, 0, 0.8);">BeanDefinition</font><font style="color:rgba(0, 0, 0, 0.8);">)；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">@Componet及其衍生品</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring提供注解</font><font style="color:rgba(0, 0, 0, 0.8);">@Component</font><font style="color:rgba(0, 0, 0, 0.8);">直接作用于Bean类，显式表明该类为一个Bean；此外，Spring还提供基于</font><font style="color:rgba(0, 0, 0, 0.8);">@Component</font><font style="color:rgba(0, 0, 0, 0.8);">，根据业务诉求不同的其他注解：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">@Repository</font><font style="color:rgba(0, 0, 0, 0.8);">：作用于数据持久层；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">@Service</font><font style="color:rgba(0, 0, 0, 0.8);">：作用于业务架构中的Service类；</font><font style="color:rgba(0, 0, 0, 0.8);">@Componet</font><font style="color:rgba(0, 0, 0, 0.8);">同样可以作用于Service类，但是从明确业务语义的角度，应当选择</font><font style="color:rgba(0, 0, 0, 0.8);">@Service</font><font style="color:rgba(0, 0, 0, 0.8);">；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font><font style="color:rgba(0, 0, 0, 0.8);">@Controller</font><font style="color:rgba(0, 0, 0, 0.8);">：作用于MVC的Controller类；</font>

<font style="color:rgba(0, 0, 0, 0.8);">除此之外，以上注解由于应用场景不同，Spring内置不同的切面进行处理；</font>

<font style="color:rgba(0, 0, 0, 0.8);">@Compnent作用的Bean名称默认为所在类的驼峰命名；如果有自定义Bean名称需求，可通过</font><font style="color:rgba(0, 0, 0, 0.8);">@Componet</font><font style="color:rgba(0, 0, 0, 0.8);">指定默认名称，一般用于注入接口/抽象类的指定实现中：</font>

```java
@Service("ConcretService")
public class ConcretServiceImpl implement AbstractService {
    @Override
    public Boolean validate(String name) {
        // ...
    }
}
```

```java
public interface AbstractService {
    Boolean validate(String name);
}
```

```java
@Componet
public class AnotherService {

    @Qulifier("ConcretService")
    @Autowired
    private AbstractService ConcretService;

    // ...
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">路径扫描机制</font>
<font style="color:rgba(0, 0, 0, 0.8);">主要使用</font><font style="color:rgba(0, 0, 0, 0.8);">@ComponetScan</font><font style="color:rgba(0, 0, 0, 0.8);">指定Bean的包路径进行对应扫描；</font>

<font style="color:rgba(0, 0, 0, 0.8);">测试Bean：</font>

```java
package com.ioc.scanning;

@Component
public class TestBean {
    // ...
}
```

<font style="color:rgba(0, 0, 0, 0.8);">需要定义一个扫描类，采用基于JavaConfig形式编写：</font>

```java
@Configuration
@ComponentScan(basePackages = "com.ioc.scanning")
public class AppConfig  {
    // ...
}
```

<font style="color:rgba(0, 0, 0, 0.8);">在SpringBoot中，明显可看到启动类</font><font style="color:rgba(0, 0, 0, 0.8);">SpringBootApplication</font><font style="color:rgba(0, 0, 0, 0.8);">是</font><font style="color:rgba(0, 0, 0, 0.8);">@Configuration</font><font style="color:rgba(0, 0, 0, 0.8);">和</font><font style="color:rgba(0, 0, 0, 0.8);">ComponentScan</font><font style="color:rgba(0, 0, 0, 0.8);">的复合注解；所以在SpringBoot中，通常要显式指定业务bean的包路径(都用SpringBoot了，say bye to xml</font><font style="color:rgba(0, 0, 0, 0.8);">😉</font><font style="color:rgba(0, 0, 0, 0.8);">)</font>

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Configuration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {
    // ...
}
```

<font style="color:rgba(0, 0, 0, 0.8);">@ComponentScan</font><font style="color:rgba(0, 0, 0, 0.8);">在xml中的对应实现是</font><font style="color:rgba(0, 0, 0, 0.8);"><context:component-scan></font><font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">或者在xml中编写:</font>

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ioc.scanning"/>
<bean id="accountServiceOne" class="com.zjw.service.impl.AccountServiceOneImpl" />
</beans>

```

```java
package com.zjw.service.impl;

import com.zjw.service.IAccountServiceOne;

/**
 * 账户的业务层实现类
 * 对象创建的三种方式一：通过构造方法创建对象
 */
public class AccountServiceOneImpl implements IAccountServiceOne {
    
    public AccountServiceOneImpl() {
        System.out.println("AccountServiceOneImpl……我创建了。。");
    }

    @Override
    public void saveAccount() {
        System.out.println("AccountServiceOneImpl中的saveAccount方法执行了");
    }
}

```

<font style="color:rgba(0, 0, 0, 0.8);">除基本的扫描设置外，</font><font style="color:rgba(0, 0, 0, 0.8);">@ComponentScan</font><font style="color:rgba(0, 0, 0, 0.8);">还支持</font>[过滤机制](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-scanning-filters)

<font style="color:rgba(0, 0, 0, 0.8);">在基于JavaConfig方式中，通过</font><font style="color:rgba(0, 0, 0, 0.8);">@Configuration</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);">@Bean</font><font style="color:rgba(0, 0, 0, 0.8);">的组合完成，Spring也支持</font><font style="color:rgba(0, 0, 0, 0.8);">@Componentt</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);">Bean</font><font style="color:rgba(0, 0, 0, 0.8);">的方式完成同样工作。不过从操作语义上讲</font><font style="color:rgba(0, 0, 0, 0.8);">@Configuration</font><font style="color:rgba(0, 0, 0, 0.8);">更明确，而且会使用CGLIB进行切面增强；</font>

---

## <font style="color:rgba(0, 0, 0, 0.8);">参考</font>
<font style="color:rgba(0, 0, 0, 0.8);">1 </font>[Spring官方文档](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans)<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">2 </font>[How To Learn JavaBeans vs Spring beans vs POJOs](https://medium.com/@mannverma/how-to-learn-javabeans-vs-spring-beans-vs-pojos-8bd9f31ca2fc)

