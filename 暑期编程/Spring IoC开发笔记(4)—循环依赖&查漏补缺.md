## <font style="color:rgba(0, 0, 0, 0.8);">前言</font>
<font style="color:rgba(0, 0, 0, 0.8);">前2年的后端开发，基本是在PandoraBoot(阿里中间件+SpringBoot)应用下做Java后端开发，接手的应用五花八门，前人对于Spring IoC的用法也是八仙过海各显神通。整理对应开发经验和陆陆续续的笔记，对Spring IoC的内容进行系统汇总，内容主要为开发层面，部分会穿插下底层原理：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring IoC开发笔记(1)——Bean的元数据实现<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring Ioc开发笔记(2)——依赖注入相关<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>Spring IoC开发笔记(3)——Bean生命周期相关<font style="color:rgba(0, 0, 0, 0.8);">  
</font><font style="color:rgba(0, 0, 0, 0.8);">+</font><font style="color:rgba(0, 0, 0, 0.8);"> </font>**<font style="color:rgba(0, 0, 0, 0.8);">Spring IoC开发笔记(4)——循环依赖&查漏补缺</font>**

---

## <font style="color:rgba(0, 0, 0, 0.8);">循环依赖问题</font>
### <font style="color:rgba(0, 0, 0, 0.8);">Spring解决循环问题的核心思路</font>
<font style="color:rgba(0, 0, 0, 0.8);">一言以蔽之，</font>**<font style="color:rgba(0, 0, 0, 0.8);">Spring仅支持单例模式下的非构造器循环依赖问题</font>**<font style="color:rgba(0, 0, 0, 0.8);">；</font>

<font style="color:rgba(0, 0, 0, 0.8);">Spring解决问题的核心思路：存在循环依赖的bean，彼此之间是一种双向关联关系，根据对方的</font>**<font style="color:rgba(0, 0, 0, 0.8);">引用地址</font>**<font style="color:rgba(0, 0, 0, 0.8);">去访问(Java值传递机制决定)；因此可以设计一种</font>**<font style="color:rgba(0, 0, 0, 0.8);">缓存机制</font>**<font style="color:rgba(0, 0, 0, 0.8);">，巧妙地让循环依赖的bean在初始化阶段，先</font>**<font style="color:rgba(0, 0, 0, 0.8);">提前持有</font>**<font style="color:rgba(0, 0, 0, 0.8);">对方的引用地址(只分配了内存空间但是未初始化)，最后一起完成初始化，解决循环依赖问题；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">Spring相关源码</font>
<font style="color:rgba(0, 0, 0, 0.8);">Spring</font>

```java
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```

1. <font style="color:rgba(0, 0, 0, 0.8);">一级缓存：</font><font style="color:rgba(0, 0, 0, 0.8);">singletonObjects</font><font style="color:rgba(0, 0, 0, 0.8);">：并发map，负责存放所有已经实例化完毕的bean；</font>
2. <font style="color:rgba(0, 0, 0, 0.8);">二级缓存：</font><font style="color:rgba(0, 0, 0, 0.8);">earlySingletonObjects</font><font style="color:rgba(0, 0, 0, 0.8);">：一级缓存和三级缓存的过度阶段；</font>
3. <font style="color:rgba(0, 0, 0, 0.8);">三级缓存：</font><font style="color:rgba(0, 0, 0, 0.8);">singletonFactories</font><font style="color:rgba(0, 0, 0, 0.8);">：存放需要解决循环依赖的单例bean信息(key为beanName，value为对应的</font>**<font style="color:rgba(0, 0, 0, 0.8);">回调工厂</font>**<font style="color:rgba(0, 0, 0, 0.8);">)；</font>

#### <font style="color:rgba(0, 0, 0, 0.8);">三级缓存读取过程</font>
<font style="color:rgba(0, 0, 0, 0.8);">以A, B循环依赖为例，当Spring尝试实例化A时，会执行</font><font style="color:rgba(0, 0, 0, 0.8);">AbstractBeanFactory</font><font style="color:rgba(0, 0, 0, 0.8);">(</font><font style="color:rgba(0, 0, 0, 0.8);">DefaultSingletonBeanRegistry</font><font style="color:rgba(0, 0, 0, 0.8);">)的</font><font style="color:rgba(0, 0, 0, 0.8);">doGetBean</font><font style="color:rgba(0, 0, 0, 0.8);">方法，关键处执行到</font><font style="color:rgba(0, 0, 0, 0.8);">getSingleton方法</font><font style="color:rgba(0, 0, 0, 0.8);">：  
</font><font style="color:rgba(0, 0, 0, 0.8);">1. 首先尝试从一级缓存</font><font style="color:rgba(0, 0, 0, 0.8);">singletonObjects</font><font style="color:rgba(0, 0, 0, 0.8);">中去获取。如果获取到就直接return。如果获取不到且对象正在创建中，那就再从二级缓存</font><font style="color:rgba(0, 0, 0, 0.8);">earlySingletonObjects</font><font style="color:rgba(0, 0, 0, 0.8);">中获取；  
</font><font style="color:rgba(0, 0, 0, 0.8);">2. 如果二级缓存获取到就直接return，如果还是获取不到，且允许读三级缓存，则通过三级缓存中</font><font style="color:rgba(0, 0, 0, 0.8);">singletonFactory</font><font style="color:rgba(0, 0, 0, 0.8);">对象的</font><font style="color:rgba(0, 0, 0, 0.8);">getObject()</font><font style="color:rgba(0, 0, 0, 0.8);">方法获取；  
</font><font style="color:rgba(0, 0, 0, 0.8);">3. 如果从三级缓存获取成功，将bean从三级缓存升级到二级缓存。物理上从</font><font style="color:rgba(0, 0, 0, 0.8);">singletonFactories</font><font style="color:rgba(0, 0, 0, 0.8);">中移除该bean，并且放进</font><font style="color:rgba(0, 0, 0, 0.8);">earlySingletonObjects</font><font style="color:rgba(0, 0, 0, 0.8);">；</font>

#### <font style="color:rgba(0, 0, 0, 0.8);">循环依赖实例化过程</font>
<font style="color:rgba(0, 0, 0, 0.8);">以</font><font style="color:rgba(0, 0, 0, 0.8);">A->B->A</font><font style="color:rgba(0, 0, 0, 0.8);">循环依赖为例，当Spring尝试实例化A时：  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 实例化A：由于之前A从未生成过，那么三级缓存机制下也取不到任何A的Bean数据，Spring此时判定</font>**<font style="color:rgba(0, 0, 0, 0.8);">A是单例模式，并且处于创建中状态</font>**<font style="color:rgba(0, 0, 0, 0.8);">，因此将Bean A的信息方法放入三级缓存</font><font style="color:rgba(0, 0, 0, 0.8);">singletonFactories</font><font style="color:rgba(0, 0, 0, 0.8);">中；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 在A中实例化B：然后对实例A注入依赖bean B，和A一样，B也因为没有实例化过因此被放入三级缓存中；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 在B中实例化A：对B进行注入属性A，再执行一遍三级缓存读取过程，直接从三级缓存中读取A并将A升级到二级缓存中；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 在A中实例化B：对A进行注入属性B，同样从缓存中成功读取B并将B升级到二级缓存中；  
</font><font style="color:rgba(0, 0, 0, 0.8);">+ 解决循环依赖：最后将A和B移动到一级缓存</font><font style="color:rgba(0, 0, 0, 0.8);">singletonObjects</font><font style="color:rgba(0, 0, 0, 0.8);">中，解决A,B循环依赖注入问题；</font>

<font style="color:rgba(0, 0, 0, 0.8);">如果循环链存在</font><font style="color:rgba(0, 0, 0, 0.8);">A -> B -> C... -> A</font><font style="color:rgba(0, 0, 0, 0.8);">，Spring会递归执行上述过程，直至循环注入全部完成；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">Spring不支持的循环依赖类型</font>
+ <font style="color:rgba(0, 0, 0, 0.8);">原型模式下的循环依赖：由于Spring三级缓存机制不缓存原型模式作用域的bean，因此无法提前暴露对象引用解决问题；</font>
+ <font style="color:rgba(0, 0, 0, 0.8);">基于构造器的循环依赖：加入singletonFactories三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决；</font>

### <font style="color:rgba(0, 0, 0, 0.8);">开发层面解决循环依赖问题</font>
<font style="color:rgba(0, 0, 0, 0.8);">最佳方式是</font>**<font style="color:rgba(0, 0, 0, 0.8);">审视已有的设计是否合理，是否有必要重构不合理的设计</font>**<font style="color:rgba(0, 0, 0, 0.8);">；</font>

[其他方式，这里有篇文章很全面](https://www.baeldung.com/circular-dependencies-in-spring)

---

## <font style="color:rgba(0, 0, 0, 0.8);">Bean层面持有IoC容器实例</font>
<font style="color:rgba(0, 0, 0, 0.8);">如果bean本身要获取创建了自身的IoC容器实例——</font><font style="color:rgba(0, 0, 0, 0.8);">ApplicationContext</font><font style="color:rgba(0, 0, 0, 0.8);">，Spring支持两种形式：</font>

### <font style="color:rgba(0, 0, 0, 0.8);">实现</font><font style="color:rgba(0, 0, 0, 0.8);">ApplicationContextAware</font><font style="color:rgba(0, 0, 0, 0.8);">接口</font>
<font style="color:rgba(0, 0, 0, 0.8);">ApplicationContextAware</font><font style="color:rgba(0, 0, 0, 0.8);">接口形式为：</font>

```java
public interface ApplicationContextAware {
    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

<font style="color:rgba(0, 0, 0, 0.8);">具体实操：</font>

```java
@Component
public class TestBean implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    @Override
    public setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext
    }
}
```

### <font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">直接注入</font>
<font style="color:rgba(0, 0, 0, 0.8);">相比较</font><font style="color:rgba(0, 0, 0, 0.8);">ApplicationContextAware</font><font style="color:rgba(0, 0, 0, 0.8);">接口，推荐使用</font><font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">方式获取容器实例，简洁明了</font>

<font style="color:rgba(0, 0, 0, 0.8);">与</font><font style="color:rgba(0, 0, 0, 0.8);">@Autowired</font><font style="color:rgba(0, 0, 0, 0.8);">注入任意依赖项一样，可将</font><font style="color:rgba(0, 0, 0, 0.8);">Application</font><font style="color:rgba(0, 0, 0, 0.8);">根据开发实际情况，作为成员变量/构造方法入参/Setter方法注入即可；</font>

<font style="color:rgba(0, 0, 0, 0.8);"></font>

<font style="color:rgba(0, 0, 0, 0.8);"></font>

# <font style="color:rgb(34, 34, 34);">Spring 如何解决循环依赖？</font>
[源码深度解析，Spring 如何解决循环依赖？](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247545567&idx=2&sn=8478f342befd6d2d84e3e11c635c4952&chksm=fbb1bb21ccc63237a4890e75a3b43a50b69ef88900fac7e784916fdff134cc94a058c192b63f&scene=27)

