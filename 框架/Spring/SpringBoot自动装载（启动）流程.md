# SpringBoot自动装载流程


自动装配原理：
主要是利用@Import注解
#Spring Boot为我们提供了一种极简的项目搭建方式，看一下Spring Boot项目的启动类：

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```
## 注解启动分析
注解启动分析，就是说的@SpringBootApplication
首先看一下@SpringBootApplication这个组合注解，除去元注解外，它还引入了其他三个重要的注解：
```
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```
接下来分别分析这些注解：

## @ComponentScan 的主要原理

扫描基础包路径： 当一个配置类上标记了 @ComponentScan 注解时，Spring 将会扫描指定基础包路径及其子包中的所有类。
```
// Example: Component scanning in a configuration class
@Configuration
@ComponentScan("com.example")
public class AppConfig {
    // Configuration code here
}

```
组件扫描过程： Spring 使用组件扫描器（ClassPathBeanDefinitionScanner）来扫描指定的基础包路径。这个扫描器会查找类路径上包含 @Component 及相关注解的类，并注册这些类的 Bean 定义。

默认过滤规则： 默认情况下，Spring 会扫描指定包及其子包下所有的类，并将所有标记有 @Component、@Service、@Repository、@Controller 等注解的类注册为 Spring Bean。这个过程是基于默认的过滤规则完成的。
```
// Example: Default filtering in @ComponentScan
@Configuration
@ComponentScan("com.example")
public class AppConfig {
    // Classes with @Component, @Service, @Repository, etc. in com.example package will be scanned and registered as beans
}

```
自定义过滤规则： 除了默认规则，@ComponentScan 还提供了 includeFilters 和 excludeFilters 属性，允许开发者自定义扫描时的过滤规则。这样可以更灵活地指定哪些类应该被扫描，哪些类应该被排除。
```
// Example: Custom filtering in @ComponentScan
@Configuration
@ComponentScan(basePackages = "com.example", includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyCustomAnnotation.class))
public class AppConfig {
    // Classes with MyCustomAnnotation in com.example package will be scanned and registered as beans
}
```
总体来说，@ComponentScan 注解的原理是通过组件扫描器扫描指定的基础包路径下的类，并根据默认或自定义的过滤规则将符合条件的类注册为 Spring Bean。这样可以方便地实现自动化的组件注册和依赖注入。
## @SpringBootConfiguration的作用
内部为
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@Indexed
```
原理主要是作为一个特化的 @Configuration 注解，用于标识一个类为 Spring Boot 应用程序的配置类。通过该注解，Spring Boot 可以识别和加载配置，确保应用程序的核心设置得到正确配置和初始化。
还有一个Indexed暂时不知道干什么的。

## @EnableAutoConfiguration
### @AutoConfigurationPackage
@AutoConfigurationPackage：该注解上有一个@Import({Registrar.class}) 注解，其中Registrar类的作用是将启动类所在的包下的所有子包组件扫描注入到spring容器中，因此这就是为什么将controller、service等包放在启动类的同级目录下的原因


### @Import({AutoConfigurationImportSelector.class})
Import注解是一个导入配置类的注解，当使用 @Import 注解时，它通常用于导入其他配置类，从而将这些配置类中的 Bean 注册到当前的 Spring 容器中。下面是一个简单的示例，演示了如何使用 @Import 注解以及它所导入的配置类：
```
@Import({Circle.class})
@Configuration
public class MainConfig {

}
```
到扫描到MainConfig时，会先导入Circle，并且执行selectImports方法

```
@Component
public class Circle implements ImportSelector {

    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        System.out.println("执行select");
        return new String[0];
    }

}
```
同样的，在启动类上会AutoConfigurationImportSelector的selectImports方法，具体如下：  

### AutoConfigurationImportSelector.class  
Spring Boot有中一个非常重要的理念就是约定大于配置。而自动配置这一机制的核心实现就是靠@EnableAutoConfiguration注解完成的    
可以看出，在@EnableAutoConfiguration注解中，使用@Import导入了AutoConfigurationImportSelector这个类，实现了ImportSelector接口的selectImports()方法。spring中会把selectImports()方法返回的String数组中的类的全限定名实例化为bean，并交给spring容器管理    
具体执行链路如下：    
AutoConfigurationImportSelector.class#getAutoConfigurationEntry--》getCandidateConfigurations--》SpringFactoriesLoader.class#loadFactoryNames--》loadSpringFactories
在SpringFactoriesLoader.class中定义了要读取的路径    
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";   

例如：首先读取../repository/org/springframework/boot/spring-boot/2.6.3/spring-boot-2.6.3.jar!/META-INF/spring.factories 目录的文件    


https://blog.csdn.net/u012060033/article/details/123863011  
