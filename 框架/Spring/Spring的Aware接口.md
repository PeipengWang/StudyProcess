# Spring的Aware
## 简述
Aware接口是一组特定于Spring容器的接口，通过这些接口，你的Bean可以感知（aware）到Spring容器的存在，并获取与Spring容器相关的一些资源或服务。这些接口都以Aware结尾，例如BeanNameAware、ApplicationContextAware等。
以下是一些常见的Aware接口及其用途：

BeanNameAware：

接口方法：setBeanName(String name)。
允许Bean获取自己在Spring容器中的名称。
BeanFactoryAware：

接口方法：setBeanFactory(BeanFactory beanFactory)。
允许Bean获取其所在的BeanFactory。
ApplicationContextAware：

接口方法：setApplicationContext(ApplicationContext applicationContext)。
允许Bean获取应用上下文对象，即Spring容器本身。
EnvironmentAware：

接口方法：setEnvironment(Environment environment)。
允许Bean获取Spring的Environment对象，以便访问配置属性。
ServletContextAware：

接口方法：setServletContext(ServletContext servletContext)。
允许在Spring Web应用中，Bean获取ServletContext对象。
ResourceLoaderAware：

接口方法：setResourceLoader(ResourceLoader resourceLoader)。
允许Bean获取Spring的资源加载器，用于加载资源文件。

## 基本使用
定义的bean实现接口
```

import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AwareBean implements BeanNameAware {

    private String beanName;

    @Bean
    public MyBean myBean() {
        return new MyBean();
    }

    public void displayBeanName(){
        System.out.println("获得beanName：" + beanName);
    }

    @Override
    public void setBeanName(String beanName) {
        this.beanName = beanName;
    }


}

```
此时就可以根据这个bean直接获取了
```
        if(beanName.equals("awareBean")){
            ((AwareBean)bean).displayBeanName();
        }
```

## 调用原理
图来源：https://zhuanlan.zhihu.com/p/559504170
![](_v_images/20240324214934769_13142.png =1152x)
从图中可以看到Aware的实现是在实例化并且完成对象的属性注入之后进行调用的
调用时核心代码为：
```
	private void invokeAwareMethods(String beanName, Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof BeanNameAware beanNameAware) {
				beanNameAware.setBeanName(beanName);
			}
			if (bean instanceof BeanClassLoaderAware beanClassLoaderAware) {
				ClassLoader bcl = getBeanClassLoader();
				if (bcl != null) {
					beanClassLoaderAware.setBeanClassLoader(bcl);
				}
			}
			if (bean instanceof BeanFactoryAware beanFactoryAware) {
				beanFactoryAware.setBeanFactory(AbstractAutowireCapableBeanFactory.this);
			}
		}
	}
```
