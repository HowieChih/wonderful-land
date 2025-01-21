---
layout: post
title:  "Spring AOP 源码流程"
date:   2019-08-13 00:00:00 +0800
---

**Spring AOP 调用链**：

org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean

  \> 属性set好之后，调用 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition) 方法

​     \> org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#apply**BeanPostProcessorsAfterInitialization**

​       \> 调用 AnnotationAwareAspectJAutoProxyCreator.postProcessAfterInitialization 方法后，代理类就生成好了

AnnotationAwareAspectJAutoProxyCreator.postProcessAfterInitialization

  \> 父类：org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#postProcessAfterInitialization

​     \> org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#createProxy

​       \> org.springframework.aop.framework.ProxyCreatorSupport#createAopProxy 如果 @EnableAspectJAutoProxy proxyTargetClass属性为true，或者被代理类没有接口，那么就使用cglib，否则使用JDK动态代理。



**JDK动态代理调用链**：

JDK 动态代理需要使用 InvocationHandler 来完成，底层调用 

Proxy.newInstance

   \> Proxy.ProxyClassFactory#apply 通过字节码生成代理类Class

​     \> ProxyGenerator#generateProxyClass 生成代理类字节码（方法3）

通过手动调用上面的方法3，反编译代理类字节码可以看到它 extends Proxy，并且 implements 了传入进去的类（因为单继承，想extends也做不到），所以方法3的参数不管你传进去是类还是接口，字节码都会当接口处理。

那么调用该方法的前置逻辑肯定会有是不是接口的判断，这也就是为什么JDK动态代理一定需要接口的原因了。

从上述方法3反编译后的代理类字节码里面也可以看出来，代理类做方法调用时，会用调用 InvocationHandler 的 invoke 方法。



创建JDK动态代理类三步走：

1、定义业务interface，并定义业务实现类

```
// 定义接口
interface Service {
    void doSomething();
}

// 实现接口
class RealService implements Service {
    @Override
    public void doSomething() {
        System.out.println("RealService is doing something.");
    }
}
```

2、定义代理类，代理类需要实现 InvocationHandler 接口，并在invoke方法里做业务想要的额外逻辑

```
// 定义动态代理类
class DynamicProxyHandler implements InvocationHandler {
    private final Object target;

    public DynamicProxyHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method call: " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("After method call: " + method.getName());
        return result;
    }
}
```

3、实例化proxy对象，并调用业务方法

```
RealService realService = new RealService();

// 创建动态代理对象
Service proxyInstance = (Service) Proxy.newProxyInstance(
    realService.getClass().getClassLoader(),
    realService.getClass().getInterfaces(),
    new DynamicProxyHandler(realService)
);

// 调用代理方法
proxyInstance.doSomething();
```

