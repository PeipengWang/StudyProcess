# BeanFactoryPostProcessor与BeanPostProcessor的职责与区别
## BeanFactoryPostProcessor：
职责： BeanFactoryPostProcessor 负责在容器实例化任何 bean 之前对 bean 定义进行修改。它允许你在 bean 被实例化之前对 bean 的定义进行全局性的修改，比如修改属性值、添加属性等。这个阶段是在 bean 的定义已经加载到容器中，但还没有实例化任何 bean 的时候进行的。

使用场景：
修改 bean 的属性值，例如修改数据库连接信息。
动态地向 bean 添加属性。
对 bean 的定义进行其他全局性的修改。
示例代码：


```
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;

public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // 在这里可以对 bean 的定义进行修改
    }
}
```
## BeanPostProcessor：
职责： BeanPostProcessor 接口定义了在 bean 实例化和初始化过程中插入自定义的逻辑。它允许你在 bean 的初始化前后执行一些操作，比如在初始化之前验证 bean 的状态或者在初始化之后执行一些定制的逻辑。
使用场景：
在 bean 实例化之前和之后执行一些自定义逻辑。
在 bean 初始化前后对 bean 进行定制。
示例代码：
```
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        // 在 bean 初始化之前执行的逻辑
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        // 在 bean 初始化之后执行的逻辑
        return bean;
    }
}
```
## 区别总结：
BeanFactoryPostProcessor：

在容器加载 bean 定义后，实例化 bean 之前对 bean 的定义进行全局性的修改。
通常用于修改 bean 的元数据，例如属性值的修改、添加属性等。
postProcessBeanFactory 方法在 bean 定义加载后、实例化之前被调用。
BeanPostProcessor：
在 bean 实例化和初始化的过程中执行自定义的逻辑。
通常用于对特定 bean 实例的初始化进行定制。
postProcessBeforeInitialization 和 postProcessAfterInitialization 方法在 bean 实例化和初始化的不同阶段被调用。