[https://segmentfault.com/a/1190000007056091](https://segmentfault.com/a/1190000007056091)



## AOP原理及实现，AOP的Aware通知顺序
<font style="color:rgb(51, 51, 51);">AOP（Aspect-Oriented Programming）是一种编程范式，它通过在运行时动态地将代码切入到类中，从而提供了一种将横切关注点（如事务管理、日志记录）与对象主体进行分离的机制。</font>

<font style="color:rgb(51, 51, 51);">AOP的实现原理是通过使用代理对象来实现的。当一个目标对象被代理时，所有的方法调用都会被转发到代理对象上，代理对象会拦截处理这些方法调用，并执行一些横切逻辑，然后再将方法调用转发给目标对象进行处理。</font>

<font style="color:rgb(51, 51, 51);">在Spring中，AOP的实现是通过将横切关注点定义为切面（Aspect），然后将切面应用到目标对象上来实现的。Spring AOP使用的代理模式是基于JDK动态代理和CGLIB代理实现的。在Spring中，如果Bean实现了某个Aware接口，Spring容器会自动调用该Bean的相应方法，例如BeanNameAware、BeanFactoryAware、ApplicationContextAware等。</font>

<font style="color:rgb(51, 51, 51);">在AOP中，切面是由通知（Advice）和切点（Pointcut）组成的。通知是在目标对象的方法执行前、执行后、抛出异常等时刻被调用的方法，切点是定义了哪些方法会被拦截的表达式或规则。在Spring中，通知可以分为以下几种类型：</font>

1. 前置通知（Before Advice）：在目标方法执行前执行的通知。
2. 后置通知（After Advice）：在目标方法执行后执行的通知。
3. 返回通知（After Returning Advice）：在目标方法返回结果后执行的通知。
4. 异常通知（After Throwing Advice）：在目标方法抛出异常后执行的通知。
5. 环绕通知（Around Advice）：在目标方法执行前、执行后、抛出异常等时刻都可以执行的通知。

<font style="color:rgb(51, 51, 51);">在Spring中，Aware通知顺序是BeanNameAware、BeanFactoryAware、ApplicationContextAware。这是因为BeanNameAware是最简单的Aware接口，只需要设置Bean的名称；BeanFactoryAware需要设置BeanFactory，而ApplicationContextAware需要设置ApplicationContext，这两个Aware接口需要在BeanFactoryAware之后调用。因此，Aware通知的顺序就是BeanNameAware、BeanFactoryAware、ApplicationContextAware。</font>

