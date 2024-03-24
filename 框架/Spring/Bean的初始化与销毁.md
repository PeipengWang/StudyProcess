# Spring初始化与销毁
## 初始化
初始化是指在 bean 创建之后、在将 bean 交给 Spring 容器管理之前或之后执行的一些特定操作。在 Spring 中，你可以通过 @PostConstruct 注解、XML 配置文件的 <init-method> 属性或 JavaConfig 中的 @Bean(initMethod) 来定义初始化方法。
时间点：初始化方法 在 bean 创建之后、在将 bean 交给 Spring 容器管理之前或之后执行。
目的：初始化方法 用于执行一些初始化逻辑，例如资源的加载、连接的建立等。
注解/配置方式：初始化方法 可以使用 @PostConstruct 注解、XML 配置文件的 <init-method> 属性或 JavaConfig 中的 @Bean(initMethod) 来定义。
适用场景：初始化方法 适用于需要在 bean 创建之后执行一些逻辑的场景，例如初始化资源、建立连接等。
```
import javax.annotation.PostConstruct;

public class MyBean {

    private String myProperty;

    @PostConstruct
    public void init() {
        // 执行初始化逻辑
        System.out.println("Initializing MyBean...");

        // 在初始化方法中可以使用已注入的属性
        System.out.println("myProperty: " + myProperty);
    }

    // Setter 方法注入
    public void setMyProperty(String myProperty) {
        this.myProperty = myProperty;
    }
}

```
需要注意的是初始化通常是在bean将属性注入之后发生的
BeanPostProcessor的前置与后置执行分别在此前后
```
    private Object initializeBean(String beanName, Object bean, BeanDefinition beanDefinition) {
        // 1. 执行 BeanPostProcessor Before 处理
        Object wrappedBean = applyBeanPostProcessorsBeforeInitialization(bean, beanName);

        // 执行 Bean 对象的初始化方法
        try {
            invokeInitMethods(beanName, wrappedBean, beanDefinition);
        } catch (Exception e) {
            throw new BeansException("Invocation of init method of bean[" + beanName + "] failed", e);
        }

        // 2. 执行 BeanPostProcessor After 处理
        wrappedBean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
        return wrappedBean;
    }
```
## 销毁
使用 @PreDestroy 注解、XML 配置文件的 <destroy-method> 属性或者实现 DisposableBean 接口的 destroy 方法来定义 bean 的销毁方法。
实例

```
import javax.annotation.PreDestroy;

public class MyBean {

    // 其他属性和方法

    @PreDestroy
    public void cleanup() {
        // 执行销毁逻辑
        System.out.println("Bean is being destroyed...");
    }
}

```
推荐使用 @PreDestroy 注解或者 XML 配置文件的 <destroy-method> 属性，因为它们提供了更为清晰和声明式的方式，而实现 DisposableBean 接口更多地是一种传统的方式。
