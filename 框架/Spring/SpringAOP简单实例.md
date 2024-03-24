
1、引入依赖

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>SpringAop_demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <dependencies>
        <!-- Spring Core -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.9</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.9</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.3.9</version>
        </dependency>
        <!-- Spring AOP -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.3.9</version>
        </dependency>

        <!-- AspectJ Weaver -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.7</version>
        </dependency>

        <!-- AspectJ RT -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.9.7</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.3.22</version>
        </dependency>
    </dependencies>

</project>
```
2、xml配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <aop:aspectj-autoproxy />

    <!--目标类-->
    <bean id="calculator" class="BasicCalculator" />
    <!--切面-->
    <bean id="loggingAspect" class="LoggingAspect" />
    <aop:config>
        <!--配置切面-->
        <aop:aspect ref="loggingAspect">
            <!--配置切入点-->
            <aop:pointcut id="pointMethod" expression="execution(* BasicCalculator.*(..))"/>
            <aop:after-returning method="finallyAdd" pointcut-ref="pointMethod" returning="joinPoint"/>
        </aop:aspect>
    </aop:config>
</beans>

```
3、被代理对象接口与被代理对象

```java
public interface Calculator {
    int add(int a, int b);
}

```

```java
import org.springframework.aop.aspectj.AspectJExpressionPointcut;

public class BasicCalculator implements Calculator {


    @Override
    public int add(int a, int b) {

        int c = 0;
        try {
            System.out.println("add method into method add...");
            c = a + b;
            System.out.println("add method will return....");
            return c;
        }catch (Exception e){
            System.out.println("add method is error....");
            e.printStackTrace();
        }finally {
            System.out.println("add method is finally....");
        }
        return c;
    }
}

```

4、主函数

```java
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Calculator calculator = context.getBean(Calculator.class);

        int result = calculator.add(5, 3);
        System.out.println("Result: " + result);
    }

}
```




还有一种可以写法，直接运行即可

```java
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.aop.framework.ProxyFactory;
import org.springframework.aop.support.DefaultPointcutAdvisor;
import org.springframework.aop.support.StaticMethodMatcherPointcut;
import org.springframework.core.annotation.MergedAnnotations;
import org.springframework.transaction.annotation.Transactional;

import java.lang.reflect.Method;

public class tesAspect {
    public static void main(String[] args) throws NoSuchMethodException {
        //切点
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* *(..))");
        System.out.println(pointcut.matches(Target.class.getMethod("foo"), Target.class));

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
        //检查类与方法是否有Transaction注解
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

    }

    @Transactional
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

