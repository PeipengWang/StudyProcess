# Spring的AOP
## 介绍
Spring的AOP（面向切面编程）是Spring框架的一个重要模块，它允许你在应用程序中以一种模块化的方式处理横切关注点（Cross-cutting Concerns）。横切关注点通常包括日志记录、安全性、事务管理等与应用程序的核心业务逻辑分离的功能。AOP通过将这些关注点从业务逻辑中分离出来，使代码更易于维护、理解和测试。
## AOP术语
首先让我们从一些重要的AOP概念和术语开始。这些术语不是Spring特有的。
连接点（Jointpoint）：表示需要在程序中插入横切关注点的扩展点，连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等，Spring只支持方法执行连接点，在AOP中表示为在哪里干；
切入点（Pointcut）： 选择一组相关连接点的模式，即可以认为连接点的集合，Spring支持perl5正则表达式和AspectJ切入点模式，Spring默认使用AspectJ语法，在AOP中表示为在哪里干的集合；
通知（Advice）：在连接点上执行的行为，通知提供了在AOP中需要在切入点所选择的连接点处进行扩展现有行为的手段；包括前置通知（before advice）、后置通知(after advice)、环绕通知（around advice），在Spring中通过代理模式实现AOP，并通过拦截器模式以环绕连接点的拦截器链织入通知；在AOP中表示为干什么；
方面/切面（Aspect）：横切关注点的模块化，比如上边提到的日志组件。可以认为是通知、引入和切入点的组合；在Spring中可以使用Schema和@AspectJ方式进行组织实现；在AOP中表示为在哪干和干什么集合；
引入（inter-type declaration）：也称为内部类型声明，为已有的类添加额外新的字段或方法，Spring允许引入新的接口（必须对应一个实现）到所有被代理对象（目标对象）, 在AOP中表示为干什么（引入什么）；
目标对象（Target Object）：需要被织入横切关注点的对象，即该对象是切入点选择的对象，需要被通知的对象，从而也可称为被通知对象；由于Spring AOP 通过代理模式实现，从而这个对象永远是被代理对象，在AOP中表示为对谁干；
织入（Weaving）：把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。在AOP中表示为怎么实现的；
AOP代理（AOP Proxy）：AOP框架使用代理模式创建的对象，从而实现在连接点处插入通知（即应用切面），就是通过代理来对目标对象应用切面。在Spring中，AOP代理可以用JDK动态代理或CGLIB代理实现，而通过拦截器模型应用切面。在AOP中表示为怎么实现的一种典型方式；

## 通知类型：
前置通知（Before advice）：在某连接点之前执行的通知，但这个通知不能阻止连接点之前的执行流程（除非它抛出一个异常）。
后置通知（After returning advice）：在某连接点正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。
异常通知（After throwing advice）：在方法抛出异常退出时执行的通知。最终通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。
环绕通知（Around Advice）：包围一个连接点的通知，如方法调用。这是最强大的一种通知类型。环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它自己的返回值或抛出异常来结束执行。
环绕通知是最常用的通知类型。和AspectJ一样，Spring提供所有类型的通知，我们推荐你使用尽可能简单的通知类型来实现需要的功能。例如，如果你只是需要一个方法的返回值来更新缓存，最好使用后置通知而不是环绕通知，尽管环绕通知也能完成同样的事情。用最合适的通知类型可以使得编程模型变得简单，并且能够避免很多潜在的错误。比如，你不需要在JoinPoint上调用用于环绕通知的proceed()方法，就不会有调用的问题。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/198e44dfc55443fa9385187127ebd4fb.png)


## Spring的AOP使用了代理模式
为了保持行为的一致性，代理类和委托类通常会实现相同的接口，所以在访问者看来两者没有丝毫的区别。通过代理类这中间一层，能有效控制对委托类对象的直接访问，也可以很好地隐藏和保护委托类对象，同时也为实施不同控制策略预留了空间，从而在设计上获得了更大的灵活性。Java 动态代理机制以巧妙的方式近乎完美地实践了代理模式的设计理念。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/aef6edf0b4a4481aa72efece535e31ee.png)

## Jdk动态代理
通过反射机制，创建一个代理类对象实例并返回。用户进行方法调用时使用创建代理对象时，需要传递该业务类的类加载器（用来获取业务实现类的元数据，在包装方法是调用真正的业务方法）、接口、handler实现类，被代理的对象必须实现了某一个接口。
Java动态代理类位于java.lang.reflect包下，一般主要涉及到以下两个类：
(1)Interface InvocationHandler：该接口中仅定义了一个方法
```
public object invoke(Object obj,Method method, Object[] args)
```
在实际使用时，第一个参数obj一般是指代理类，method是被代理的方法，如上例中的request()，args为该方法的参数数组。这个抽象方法在代理类中动态实现。
(2)Proxy：该类即为动态代理类，其中主要包含以下内容：
```
protected Proxy(InvocationHandler h)：构造函数，用于给内部的h赋值。
static Class getProxyClass (ClassLoaderloader, Class[] interfaces)：获得一个代理类，其中loader是类装载器，interfaces是真实类所拥有的全部接口的数组。
static Object newProxyInstance(ClassLoaderloader, Class[] interfaces, InvocationHandler h)：返回代理类的一个实例，返回后的代理类可以当作被代理类使用(可使用被代理类的在Subject接口中声明过的方法)
```
所谓DynamicProxy是这样一种class：它是在运行时生成的class，在生成它时你必须提供一组interface给它，然后该class就宣称它实现了这些 interface。你当然可以把该class的实例当作这些interface中的任何一个来用。当然，这个DynamicProxy其实就是一个Proxy，它不会替你作实质性的工作，在生成它的实例时你必须提供一个handler，由它接管实际的工作。
在使用动态代理类时，我们必须实现InvocationHandler接口
通过这种方式，被代理的对象(RealSubject)可以在运行时动态改变，需要控制的接口(Subject接口)可以在运行时改变，控制的方式(DynamicSubject类)也可以动态改变，从而实现了非常灵活的动态代理关系。
动态代理步骤：
1.创建一个实现接口InvocationHandler的类，它必须实现invoke方法
2.创建被代理的类以及接口
3.通过Proxy的静态方法
newProxyInstance(ClassLoaderloader, Class[] interfaces, InvocationHandler h)创建一个代理

4.通过代理调用方法
代理类为什么要重写equals、toString与hashCode方法
根据java面向对象知道，所有的类都有最终继承Object，而Object中默认实现了equals、toString与hashCode方法。
因为代理类继承了Proxy，所以如果不重写equals、toString与hashCode方法
那么当代理类执行equals、toString与hashCode方法时，实现的就是Proxy中的方法，也就是Object中的方法

但是如果目标类重写了equals、toString与hashCode方法，这时就会出现问题了。
所以代理必须重写equals、toString与hashCode方法，通过反射执行目标类的equals、toString与hashCode方法
## 代码实现测试
接口Image
```
public interface Image {
    void display();
}

```
被代理类，可以写多个只要继承Image方法即可
```
public class RealImage1 implements Image{
    @Override
    public void display() {
        System.out.println("display one");
    }
}

```
代理类，继承 InvocationHandler 接口，重写 invoke 方法
```
public class ProxyImage implements InvocationHandler {
    private Image realImage;

    public ProxyImage(Image realImage) {
        this.realImage = realImage;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Proxy Before");
        System.out.println("Method:"+method);
        Object realClass = method.invoke(realImage,args);
        System.out.println("Proxy After");
        return realClass;
    }
}
```
调用Main.java
```
    public static void main(String[] args) {
        // realImage
        Image realImage1 = new RealImage1();

        InvocationHandler invocationHandler = new ProxyImage(realImage1);
        ClassLoader loader = realImage1.getClass().getClassLoader();
        //返回这个类中实现接口的数组。
        Class[] interfaces = realImage1.getClass().getInterfaces();
        Image realInage1 = (Image) Proxy.newProxyInstance(loader,interfaces,invocationHandler);
        realInage1.display();
        }
```
CGLIB动态代理
一个java字节码的生成工具，它动态生成一个被代理类的子类，子类重写被代理的类的所有不是final的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑，通过增强器Enhancer和拦截器MethodInterceptor去实现。
创建代理的步骤：
生成代理类的二进制字节码文件；
加载二进制字节码，生成Class对象；
通过反射机制获得实例构造，并创建代理类对象


名称	备注
静态代理	简单，代理模式，是动态代理的理论基础。常见使用在代理模式
jdk动态代理  需要有顶层接口才能使用，但是在只有顶层接口的时候也可以使用，常见是mybatis的mapper文件是代理。使用反射完成。
cglib动态代理 可以直接代理类，使用字节码技术，不能对 final类进行继承。使用了动态生成字节码技术。





## Spring选择代理的原则
首先放出代码实例
```
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.aop.framework.ProxyFactory;
import org.springframework.aop.support.DefaultPointcutAdvisor;

public class tesAspect {
    public static void main(String[] args) {
        //切点
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* *(..))");
        //通知
        MethodInterceptor advice = new MethodInterceptor() {
            @Override
            public Object invoke(MethodInvocation methodInvocation) throws Throwable {
                System.out.println("before..");
                Object result = methodInvocation.proceed();
                System.out.println("after..");
                return result;
            }
        };
        //切面
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, advice);

        Target target1 = new Target();
        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.setTarget(target1);
        proxyFactory.addAdvisor(advisor);
        A1 a1 = (A1) proxyFactory.getProxy();
        System.out.println(a1.getClass());
        a1.foo();
        a1.bar();

    }

    interface A1{
        void foo();
        void bar();
    }

    static class Target implements A1{

        @Override
        public void foo() {
            System.out.println("foo");
        }

        @Override
        public void bar() {
            System.out.println("bar");
        }
    }
}
```
有三种情形：
1、proxyTargetClass=false, 目标实现了接口，则使用jdk代理
2、proxyTargetClass=false,目标未实现接口，则使用cgLib代理
3、proxyTargetClass=true,则使用cgLib代理
第一种情形代码添加
```
proxyFactory.setInterfaces(target1.getClass().getInterfaces());
proxyFactory.setProxyTargetClass(false);
```
输出：
class $Proxy0
before..
foo
after..
before..
bar
after..
第二种情形
```
proxyFactory.setProxyTargetClass(false);
```
```
class tesAspect$Target$$EnhancerBySpringCGLIB$$c63a06cb
before..
foo
after..
before..
bar
after..
```
第三种情形(默认)
```
 proxyFactory.setProxyTargetClass(true);
 class tesAspect$Target$$EnhancerBySpringCGLIB$$c63a06cb
before..
foo
after..
before..
bar
after..
```

## 切点
```
 //切点
AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
pointcut.setExpression("execution(* *(..))");
System.out.println(pointcut.matches(Target.class.getMethod("foo"), Target.class));
```
切点匹配的使用：
Spring的切点只能匹配方法，不能匹配类，
打印是否加了Transaction注解
```
        StaticMethodMatcherPointcut pointcut1 = new StaticMethodMatcherPointcut() {
            @Override
            public boolean matches(Method method, Class<?> aClass) {
                //检查方法上是否加了Transaction注解
                MergedAnnotations annotation = MergedAnnotations.from(method);
                if(annotation.isPresent(Transactional.class)){
                    return true;
                }
                //检查方法上是否加了注解
                annotation = MergedAnnotations.from(aClass, MergedAnnotations.SearchStrategy.TYPE_HIERARCHY);
                if(annotation.isPresent(Transactional.class)){
                    return true;
                }
                return false;
            }
        };
        System.out.println(pointcut1.matches(Target.class.getMethod("foo"), Target.class));
```
上述方法检查Target的方法和类是否加了Transactional注解
可以修改
## 切面从@Aspect到Advisor
```
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Before;
import org.springframework.aop.Advisor;
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator;
import org.springframework.aop.support.DefaultPointcutAdvisor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ConfigurationClassPostProcessor;
import org.springframework.context.support.GenericApplicationContext;

public class testAspect1 {
    public static void main(String[] args) {
        GenericApplicationContext applicationContext = new GenericApplicationContext();
        applicationContext.registerBean("aspect1", Aspect1.class);
        applicationContext.registerBean("config", Config.class);
        //后置处理器，可以加载上述两个的类中的bean
        applicationContext.registerBean(ConfigurationClassPostProcessor.class);
        //
        applicationContext.registerBean(AnnotationAwareAspectJAutoProxyCreator.class);
        applicationContext.refresh();
        AnnotationAwareAspectJAutoProxyCreator creator = applicationContext.getBean(AnnotationAwareAspectJAutoProxyCreator.class);
        creator.f
        
        for (String name : applicationContext.getBeanDefinitionNames()){
            System.out.println(name);
        }
    }
    static class Target1{
        public void foo(){
            System.out.println("foo1");
        }
    }
    static class Target2{
        public void foo(){
            System.out.println("foo2");
        }
    }
    @org.aspectj.lang.annotation.Aspect //高级切面
    static class Aspect1{
        @Before("execution(* foo(..))")
        public void before(){
            System.out.println("aspect1 before...");
        }
        @After("execution(* foo(..))")
        public void after(){
            System.out.println("aspect1 after...");
        }
    }
    @Configuration
    static class Config{
        @Bean //低级切面 Advisor方式
        public Advisor advisor3(MethodInterceptor advice3){
            AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
            pointcut.setExpression("execution(* foo(..))");
            return new DefaultPointcutAdvisor(pointcut, advice3);
        }
        @Bean
        public MethodInterceptor advice3(){
            return new MethodInterceptor() {
                @Override
                public Object invoke(MethodInvocation methodInvocation) throws Throwable {
                    System.out.println("advice3 before...");
                    Object result = methodInvocation.proceed();
                    System.out.println("advice3 after...");
                    return result;
                }
            };
        }
    }
}

```









