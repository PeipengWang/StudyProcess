@[TOC](自动配置源码分析)
## 主入口类
```c
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class WangApplication {

    public static void main(String[] args) {
        SpringApplication.run(WangApplication.class, args);
    }
}

```
@SpringBootApplication来标注一个主程序类，说明是一个SpringBoot应用

@SpringBootApplication是一个组合注解
```c
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```
其中最主要的有三个，分别是

```c
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```
接下来分别介绍
## 1，@SpringBootConfiguration
这是一个SpringBoot的配置类，标注在某个类上，表名这是一个SpringBoot配置类，源代码为

```c
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
    @AliasFor(
        annotation = Configuration.class
    )
    boolean proxyBeanMethods() default true;
}

```
其中@Configuration可以表名这是一个配置类
可以展示一下@Configuration这个注解

```c
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration{
。。。。
}
```
## 2，@EnableAutoConfiguration
这是一个开启自动配置功能的注解，以前需要配置的东西都可以帮我们自动配置，源代码为

```c
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	/**
	 * Exclude specific auto-configuration classes such that they will never be applied.
	 * @return the classes to exclude
	 */
	Class<?>[] exclude() default {};

	/**
	 * Exclude specific auto-configuration class names such that they will never be
	 * applied.
	 * @return the class names to exclude
	 * @since 1.3.0
	 */
	String[] excludeName() default {};

}
```
由上述代码进行分析，其中主要是由两个主要注解：

```c
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
```
分别进行分析：
### （1），@AutoConfigurationPackage

```c
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
.....
}
```
其中是利用@Import(AutoConfigurationPackages.Registrar.class)这个Spring的底层注解来完成的，@Import是给容器中导入一个组件，这个组件是AutoConfigurationPackages.Registrar.class，进入Registrar这个组件的源码

```c
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

		@Override
		public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
			register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
		}

		@Override
		public Set<Object> determineImports(AnnotationMetadata metadata) {
			return Collections.singleton(new PackageImports(metadata));
		}

	}
```
其中registerBeanDefinitions方法会把我们的主配置类所在的包下的所有的组件进行扫描
例如
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201118145629794.png)
这个文件会扫描com.uestc.wang包下的主配置类WangApplication和所有子包进行扫描
由此可知：**@AutoConfigurationPackage会将主配置类（@SpringBootApplication所在的类)的所在包及下面所有子包里面的所有组件扫描到Spring容器中**

### （2）@Import(AutoConfigurationImportSelector.class)
同上面类似，@Import是给容器中导入组件，这个组件为AutoConfigurationImportSelector.class
AutoConfigurationImportSelector.class：导入哪些组件选择器，会将所有需要导入的组件以全类名的方式返回；这些组件会被添加到容器中。
这个类中有个方法为：

```c
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```
会导入很多很多自动配置类（xxxAutoConfiguration），给容器中导入这个场景需要的所有组件，并配置这些组件，有了自动配置类，免去了手动编写注入功能组件的工作。
SpringBoot在启动的时候会从类路径下的META-INF/Spring.factories中获取EableAutoConfiguration指定的值，将这些值自动配置类导入到容器中，自动配置类就生效，并进行自动配置。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/202011181522532.png)
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20201118152323281.png)

