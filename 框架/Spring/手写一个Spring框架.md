# 手写Spring与基本原理解析
详细请跳转  https://bugstack.cn/md/spring/develop-spring
这里是学习上述小博哥的《手写Spring》做的简易记录，夹杂着很多个人私活，建议直接看链接
## 简介
从[SpringBean的加载流程](https://blog.csdn.net/Artisan_w/article/details/133806284)博文中可以知道Spring的简易加载流程  
Spring bean的生命周期包括实例化、属性赋值、初始化前回调、自定义初始化、初始化后回调、初始化完成、销毁前回调、自定义销毁这些阶段和回调方法。  
接下来讲根据上述流程逐步深入  
## 写一个简单的Bean加载容器  
### 定义一个抽象所有类的BeanDefinition  
首先进行简单的容器设计，我们需要将一个个的类进行统一的定义，这个定义名称为BeanDefinition，他包含众多的属性，包括是否是单例，类的名称等等。为了减少代码的复杂性，这里不再展开  
简单定义这个类    
```
public class BeanDefinition {
    private Object object;

    public BeanDefinition(Object object) {
        this.object = object;
    }

    public Object getBean() {
        return object;
    }
}
```
在这个类中，BeanDefinition单纯代表这个类的统一接口，真正的类是object，后面我们可以通过getBean的方式直接获取这个object。  


### 定义一个工厂存储所有的类  
实体类有了统一的接口后，就可以建立一个工厂，这个工厂叫做“Bean”工厂，建立的所有的类都以Map的方式放到这个工程中，后续可以直接调用  
Map中主键就是我们定义的名称，也就是平常xml中的bean的id  
```
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class BeanFactory {
    private Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>();

    public Object getBean(String name) {
        return beanDefinitionMap.get(name).getBean();
    }
    public void registerBeanDefinition(String name, BeanDefinition beanDefinition){
        beanDefinitionMap.put(name, beanDefinition);
    }
}
```
这样，一个简易的容器实际上已经建好了，我们只需要往这个bean里面塞class就可以  

### 测试
定义一个实体类UserService  
```
public class UserService {
    public void query(){
        System.out.println("用户名称查询");
    }
}
```
测试  
```
public class ApiTest {
    public static void main(String[] args) {
        //定义好bean工厂
        BeanFactory beanFactory = new BeanFactory();
        //注册bean对象
        BeanDefinition beanDefinition = new BeanDefinition(new UserService());
        beanFactory.registerBeanDefinition("userService", beanDefinition);
        //获取类对象
        UserService userService = (UserService) beanFactory.getBean("userService");
        userService.query();
    }
}

```
这样一个简单的Spring Bean容器实际就建立好了  

## 实现Bean的注册定义和获取  
定义 BeanFactory 这样一个 Bean 工厂，提供 Bean 的获取方法 getBean(String name)，之后这个 Bean 工厂接口由抽象类 AbstractBeanFactory 实现。  
BeanFactory 的定义由 AbstractBeanFactory 抽象类实现接口的 getBean 方法  
而 AbstractBeanFactory 又继承了实现了 SingletonBeanRegistry 的DefaultSingletonBeanRegistry 类。这样 AbstractBeanFactory 抽象类就具备了单例 Bean 的注册功能。  
AbstractBeanFactory 中又定义了两个抽象方法：getBeanDefinition(String beanName)、createBean(String beanName, BeanDefinition beanDefinition) ，而这两个抽象方法分别由 DefaultListableBeanFactory、 AbstractAutowireCapableBeanFactory 实现。  
最终 DefaultListableBeanFactory 还会继承抽象类 AbstractAutowireCapableBeanFactory 也就可以调用抽象类中的 createBean 方法了。  
## 基于Cglib实现含构造函数的类实例化策略
```
通过策略模式拆分单例构造与有参数构造，判断条件为是否有参数，构造时使用Cglib  
1、BeanFactory 中我们重载了一个含有入参信息 args 的 getBean 方法，这样就可以方便的传递入参给构造函数实例化了。  
2、在实例化接口 instantiate 方法中添加必要的入参信息，包括：beanDefinition、 beanName、ctor、args  
其中 Constructor 你可能会有一点陌生，它是 java.lang.reflect 包下的 Constructor 类，里面包含了一些必要的类信息，有这个参数的   目的就是为了拿到符合入参信息相对应的构造函数。  
而 args 就是一个具体的入参信息了，最终实例化时候会用到。  
3、在 AbstractAutowireCapableBeanFactory 抽象类中定义了一个创建对象的实例化策略属性类 InstantiationStrategy instantiationStrategy，这里我们选择了 Cglib 的实现类。  
4、抽取 createBeanInstance 方法，在这个方法中需要注意 Constructor 代表了你有多少个构造函数，通过 beanClass.getDeclaredConstructors() 方式可以获取到你所有的构造函数，是一个集合。  
5、循环比对出构造函数集合与入参信息 args 的匹配情况  
```


## Bean对象注入属性和依赖Bean的功能
属性填充是在 Bean 使用 newInstance 或者 Cglib 创建后，开始补全属性信息，那么就可以在类 AbstractAutowireCapableBeanFactory 的 createBean 方法中添加补全属性方法  

在BeanDefinition中增加一个 PropertyValues，PropertyValues中引用一个List<PropertyValue>，其中所有的属性会放到这个list中，PropertyValue是一种类似Map的name-valu结构，name是属性名称，value是具体引用，这个value如果是引用的其他类，那么就要用到BeanReference。  
```
    public PropertyValue(String name, Object value) {
        this.name = name;
        this.value = value;
    }
```
具体填充是在AbstractAutowireCapableBeanFactory中的 applyPropertyValues 方法进行具体的填充操作，如果遇到的是 BeanReference，那么就需要递归获取 Bean 实例，调用 getBean 方法。    

## Spring.xml解析和注册Bean对象

把 Bean 的定义、注册和初始化交给 Spring.xml 配置化处理，那么就需要实现两大块内容，分别是：资源加载器、xml资源处理类，实现过程主要以对接口 Resource、ResourceLoader 的实现，而另外 BeanDefinitionReader 接口则是对资源的具体使用，将配置信息注册到 Spring 容器中去。    

Resource 的资源加载器的实现中包括了，ClassPath、系统文件、云配置文件，这三部分与 Spring 源码中的设计和实现保持一致，最终在 DefaultResourceLoader 中做具体的调用。   
接口：BeanDefinitionReader、抽象类：AbstractBeanDefinitionReader、实现类：XmlBeanDefinitionReader，这三部分内容主要是合理清晰的处理了资源读取后的注册 Bean 容器操作。    
关键代码：    
解析标签填入beanfinition    
```
// 解析标签：property
Element property = (Element) bean.getChildNodes().item(j);
String attrName = property.getAttribute("name");
String attrValue = property.getAttribute("value");
String attrRef = property.getAttribute("ref");
// 获取属性值：引入对象、值对象
Object value = StrUtil.isNotEmpty(attrRef) ? new BeanReference(attrRef) : attrValue;
// 创建属性信息
PropertyValue propertyValue = new PropertyValue(attrName, value);
beanDefinition.getPropertyValues().addPropertyValue(propertyValue);
```
注册 BeanDefinition  
```
getRegistry().registerBeanDefinition(beanName, beanDefinition);
```
##  实现应用上下文  

引入应用上下文，进行资源扫描与加载，为Bean对象实例化过程添加扩展机制，为bean对象执行修改、记录和替换等动作。  
过程：    
加载--》注册--》**修改**--》实例化--》**扩展**   
满足于对 Bean 对象扩展的两个接口，其实也是 Spring 框架中非常具有重量级的两个接口：BeanFactoryPostProcessor与BeanPostProcessor    
BeanFactoryPostProcessor，是由 Spring 框架组建提供的容器扩展机制，允许在 Bean 对象注册后但未实例化之前，对 Bean 的定义信息 BeanDefinition 执行修改操作。定义如下    
```
public interface BeanFactoryPostProcessor {

    /**
     * 在所有的 BeanDefinition 加载完成后，实例化 Bean 对象之前，提供修改 BeanDefinition 属性的机制
     *
     * @param beanFactory
     * @throws BeansException
     */
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```
 refresh() 定义实现过程，包括：  
1、创建 BeanFactory，并加载 BeanDefinition  
2、获取 BeanFactory    
3、在 Bean 实例化之前，执行 BeanFactoryPostProcessor (Invoke factory processors registered as beans in the context.)    
4、BeanPostProcessor 需要提前于其他 Bean 对象实例化之前执行注册操作    
5、提前实例化单例Bean对象      

BeanPostProcessor，也是 Spring 提供的扩展机制，不过 BeanPostProcessor 是在 Bean 对象实例化之后修改 Bean 对象，也可以替换 Bean 对象。这部分与后面要实现的 AOP 有着密切的关系。  
定义如下：    
```
public interface BeanPostProcessor {

    /**
     * 在 Bean 对象执行初始化方法之前，执行此方法
     *
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

    /**
     * 在 Bean 对象执行初始化方法之后，执行此方法
     *
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
 
```
在Bean创建时完成前置和后置处理：AbstractAutowireCapableBeanFactory  
initializeBean方法：  
// 1. 执行 BeanPostProcessor Before 处理  
Object wrappedBean = applyBeanPostProcessorsBeforeInitialization(bean, beanName);  
 // 待完成内容：invokeInitMethods(beanName, wrappedBean, beanDefinition);
 invokeInitMethods(beanName, wrappedBean, beanDefinition);
// 2. 执行 BeanPostProcessor After 处理
 wrappedBean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
## 向虚拟机注册钩子，实现Bean对象的初始化和销毁方法  
spring.xml 配置中添加 init-method、destroy-method 两个注解  
在配置文件加载的过程中，把注解配置一并定义到 BeanDefinition 的属性当中。这样在 initializeBean 初始化操作的工程中，就可以通过反射的方式来调用配置在 Bean 定义属性当中的方法信息了  
1、定义初始化和销毁方法的接口：InitializingBean和DisposableBean  
在一些需要结合 Spring 实现的组件中，经常会使用这两个方法来做一些参数的初始化和销毁操作。比如接口暴漏、数据库数据读取、配置文件加载等等。  
2、BeanDefinition 新增加了两个属性：initMethodName、destroyMethodName，这两个属性是为了在 spring.xml 配置的 Bean 对象中，可以配置 init-method="initDataMethod" destroy-method="destroyDataMethod" 操作  

最终实现接口的效果是一样的。只不过1是接口方法的直接调用，2是在配置文件中读取到方法反射调用  

## 感知容器对象
感知容器定义一个标签Aware，继承了这个标签的接口会在初始化和BeanPostProcessor Before之前进行处理  
核心代码:
```
       // invokeAwareMethods
        if (bean instanceof Aware) {
            if (bean instanceof BeanFactoryAware) {
                ((BeanFactoryAware) bean).setBeanFactory(this);
            }
            if (bean instanceof BeanClassLoaderAware){
                ((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
            }
            if (bean instanceof BeanNameAware) {
                ((BeanNameAware) bean).setBeanName(beanName);
            }
        }

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
```
有几个常用的Aware接口  
BeanNameAware, BeanClassLoaderAware, ApplicationContextAware, BeanFactoryAware  
具体功能望文生义即可  

也可以自定义Aware接口：  
1、接口继承Aware  
2、bean实现接口  
3、实现接口方法  
4、bean中具体执行  

## Bean对象作用域以及FactoryBean的实现和使用  
注意BeanFactory与FactoryBean的区别：  
BeanFactory是bean的工厂，FactoryBean是一个工厂Bean，他也可以通过BeanFactory获得，算是一个特殊的bean，用于定义创建和配置复杂对象。通过实现 FactoryBean 接口，你可以自定义对象的创建逻辑，并将其纳入 Spring 容器的管理。  

BeanDefinition中  
String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;  
String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;  
这两个属性决定着单例还是多例  

## 基于观察者实现，容器事件和事件监听器  
以围绕实现 event 事件定义、发布、监听功能实现和把事件的相关内容使用 AbstractApplicationContext#refresh 进行注册和处理操作。  

实际上是定义三个角色  
 ApplicationEventPublisher，事件发布者，并在实现类中提供事件监听功能，是整个一个事件的发布接口，所有的事件都需要从这个接口发布出去。    
 ApplicationEventMulticaster 接口是注册监听器和发布事件的广播器，提供添加、移除和发布事件方法，在事件广播器中定义了添加监听和删除监听的方法以及一个广播事件的方法 multicastEvent 最终推送时间消息也会经过这个接口方法来处理谁该接收事件。   
 ApplicationEvent：实现此接口定义具体事件    

在抽象应用上下文 AbstractApplicationContext#refresh 中，主要新增了 初始化事件发布者、注册事件监听器、发布容器刷新完成事件，三个方法用于处理事件操作。  
初始化事件发布者(initApplicationEventMulticaster)，主要用于实例化一个 SimpleApplicationEventMulticaster，这是一个事件广播器。  
注册事件监听器(registerListeners)，通过 getBeansOfType 方法获取到所有从 spring.xml 中加载到的事件配置 Bean 对象。    
发布容器刷新完成事件(finishRefresh)，发布了第一个服务器启动完成后的事件，这个事件通过 publishEvent 发布出去，其实也就是调用了 applicationEventMulticaster.multicastEvent(event); 方法。  
最后是一个 close 方法中，新增加了发布一个容器关闭事件。publishEvent(new ContextClosedEvent(this));    

## AOP切面
核心代码  
```
        // 目标对象(可以替换成任何的目标对象)  
        Object targetObj = new UserService();  
        // AOP 代理 
        IUserService proxy = (IUserService) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), targetObj.getClass().getInterfaces(), new InvocationHandler() {
            // 方法匹配器
            MethodMatcher methodMatcher = new AspectJExpressionPointcut("execution(* cn.bugstack.springframework.test.bean.IUserService.*(..))");
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if (methodMatcher.matches(method, targetObj.getClass())) {
                    // 方法拦截器
                    MethodInterceptor methodInterceptor = invocation -> {
                        long start = System.currentTimeMillis();
                        try {
                            return invocation.proceed();
                        } finally {
                            System.out.println("监控 - Begin By AOP");
                            System.out.println("方法名称：" + invocation.getMethod().getName());
                            System.out.println("方法耗时：" + (System.currentTimeMillis() - start) + "ms");
                            System.out.println("监控 - End\r\n");
                        }
                    };
                    // 反射调用
                    return methodInterceptor.invoke(new ReflectiveMethodInvocation(targetObj, method, args));
                }
                return method.invoke(targetObj, args);
            }
        });
        String result = proxy.queryUserInfo();
        System.out.println("测试结果：" + result);
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/259da69098ef48d8a37e01539154e477.png)


Pointcut:切入点接口，定义用于获取 ClassFilter、MethodMatcher 的两个类，这两个接口获取都是切点表达式提供的内容。  
ClassFilter:定义类匹配类，用于切点找到给定的接口和目标类。  
MethodMatcher:方法匹配，找到表达式范围内匹配下的目标类和方法  

AspectJExpressionPointcut：实现了 Pointcut、ClassFilter、MethodMatcher，三个接口定义方法，同时这个类主要是对 aspectj 包提供的表达式校验方法使用。  
用于把代理、拦截、匹配的各项属性包装到一个类中，方便在 Proxy 实现类进行使用。  
```
public class AdvisedSupport {
    // 被代理的目标对象
    private TargetSource targetSource;
    // 方法拦截器
    private MethodInterceptor methodInterceptor;
    // 方法匹配器(检查目标方法是否符合通知条件)
    private MethodMatcher methodMatcher;
    // ...get/set
}
```
 最后进行代理抽象实现(JDK&Cglib)  
## AOP动态代理，融入到Bean的生命周期    
BeanPostProcessor 接口实现继承的 InstantiationAwareBeanPostProcessor 接口后，做了一个自动代理创建的类 DefaultAdvisorAutoProxyCreator，这个类的就是用于处理整个 AOP 代理融入到 Bean 生命周期中的核心类。  
DefaultAdvisorAutoProxyCreator 会依赖于拦截器、代理工厂和Pointcut与Advisor的包装服务 AspectJExpressionPointcutAdvisor，由它提供切面、拦截方法和表达式。    

融入Bean生命周期的自动代理创建者，如下  
```
public class DefaultAdvisorAutoProxyCreator implements InstantiationAwareBeanPostProcessor, BeanFactoryAware {

    private DefaultListableBeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = (DefaultListableBeanFactory) beanFactory;
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {

        if (isInfrastructureClass(beanClass)) return null;
        //获取aspc表达式
        Collection<AspectJExpressionPointcutAdvisor> advisors = beanFactory.getBeansOfType(AspectJExpressionPointcutAdvisor.class).values();
        //获取通知信息
        for (AspectJExpressionPointcutAdvisor advisor : advisors) {
            ClassFilter classFilter = advisor.getPointcut().getClassFilter();
            if (!classFilter.matches(beanClass)) continue;
            //构造包括代理类，要代理的信息的类
            AdvisedSupport advisedSupport = new AdvisedSupport();
            TargetSource targetSource = null;
            try {
                targetSource = new TargetSource(beanClass.getDeclaredConstructor().newInstance());
            } catch (Exception e) {
                e.printStackTrace();
            }
            advisedSupport.setTargetSource(targetSource);
            advisedSupport.setMethodInterceptor((MethodInterceptor) advisor.getAdvice());
            advisedSupport.setMethodMatcher(advisor.getPointcut().getMethodMatcher());
            advisedSupport.setProxyTargetClass(false);
            //生成代理类
            return new ProxyFactory(advisedSupport).getProxy();

        }

        return null;
    }
}
```
创建bean  
```
    @Override
    protected Object createBean(String beanName, BeanDefinition beanDefinition, Object[] args) throws BeansException {
        Object bean = null;
        try {
            // 判断是否返回代理 Bean 对象
            bean = resolveBeforeInstantiation(beanName, beanDefinition);
            if (null != bean) {
                return bean;
            }

            bean = createBeanInstance(beanDefinition, beanName, args);
            // 给 bean 填充属性
            applyPropertyValues(beanName, bean, beanDefinition);
            // 执行 Bean 的初始化方法和 BeanPostProcessor 的前置和后置处理方法
            bean = initializeBean(beanName, bean, beanDefinition);
        } catch (Exception e) {
            throw new BeansException("Instantiation of bean failed.", e);
        }

        // 注册实现了 DisposableBean 接口的 Bean 对象
        registerDisposableBeanIfNecessary(beanName, bean, beanDefinition);

        // 判断 SCOPE_SINGLETON，SCOPE_PROTOTYPE
        if (beanDefinition.isSingleton()) {
            registerSingleton(beanName, bean);
        }
        return bean;
    }

```
  
## 通过注解配置和包自动扫描的方式完成Bean对象的注册

在XmlBeanDefinitionReader中解析<context:component-scan />标签，扫描类组装BeanDefinition然后注册到容器中的操作在ClassPathBeanDefinitionScanner#doScan中实现。  
主要包括的就是 xml 解析类 XmlBeanDefinitionReader 对 ClassPathBeanDefinitionScanner#doScan 的使用。  


依赖于 BeanFactoryPostProcessor 在 Bean 生命周期的属性，可以在 Bean 对象实例化之前，改变属性信息。所以这里通过实现 BeanFactoryPostProcessor 接口，完成对配置文件的加载以及摘取占位符中的在属性文件里的配置。
这样就可以把提取到的配置信息放置到属性配置中了，buffer.replace(startIdx, stopIdx + 1, propVal); propertyValues.addPropertyValue
  
Component注解扫描原理：  
定义  
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Component {

    String value() default "";

}
```
1、解析xml，在得到有component-scan是开启注解扫描功能，XmlBeanDefinitionReader  
```
    protected void doLoadBeanDefinitions(InputStream inputStream) throws ClassNotFoundException, DocumentException {
        SAXReader reader = new SAXReader();
        Document document = reader.read(inputStream);
        Element root = document.getRootElement();

        // 解析 context:component-scan 标签，扫描包中的类并提取相关信息，用于组装 BeanDefinition
        Element componentScan = root.element("component-scan");
        if (null != componentScan) {
            String scanPath = componentScan.attributeValue("base-package");
            if (StrUtil.isEmpty(scanPath)) {
                throw new BeansException("The value of base-package attribute can not be empty or null");
            }
            scanPackage(scanPath);
        }
        // ... 省略其他
        // 注册 BeanDefinition
        getRegistry().registerBeanDefinition(beanName, beanDefinition);
    }
    private void scanPackage(String scanPath) {
        String[] basePackages = StrUtil.splitToArray(scanPath, ',');
        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(getRegistry());
        scanner.doScan(basePackages);
    }
```

2、扫描所有的要扫描的package，并注册，ClassPathBeanDefinitionScanner  
```
  public void doScan(String... basePackages) {
        for (String basePackage : basePackages) {
        //发现注解，下面详细看
            Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
            for (BeanDefinition beanDefinition : candidates) {
                // 解析 Bean 的作用域 singleton、prototype
                String beanScope = resolveBeanScope(beanDefinition);
                if (StrUtil.isNotEmpty(beanScope)) {
                    beanDefinition.setScope(beanScope);
                }
                registry.registerBeanDefinition(determineBeanName(beanDefinition), beanDefinition);
            }
        }
    }
```
3、上述处理注解对象的装配，ClassPathScanningCandidateComponentProvider
```
    public Set<BeanDefinition> findCandidateComponents(String basePackage) {
        Set<BeanDefinition> candidates = new LinkedHashSet<>();
        //扫描所有的包含Component的类，并加入candidates，最后返回
        Set<Class<?>> classes = ClassUtil.scanPackageByAnnotation(basePackage, Component.class);
        for (Class<?> clazz : classes) {
            candidates.add(new BeanDefinition(clazz));
        }
        return candidates;
    }

```
## 通过注解给属性注入配置和Bean对象

围绕实现接口 InstantiationAwareBeanPostProcessor 的类 AutowiredAnnotationBeanPostProcessor 作为入口点，被 AbstractAutowireCapableBeanFactory创建 Bean 对象过程中调用扫描整个类的属性配置中含有自定义注解 Value、Autowired、Qualifier，的属性值。  

AutowiredAnnotationBeanPostProcessor#postProcessPropertyValues  
```
 @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        // 1. 处理注解 @Value
        Class<?> clazz = bean.getClass();
        clazz = ClassUtils.isCglibProxyClass(clazz) ? clazz.getSuperclass() : clazz;

        Field[] declaredFields = clazz.getDeclaredFields();

        for (Field field : declaredFields) {
            Value valueAnnotation = field.getAnnotation(Value.class);
            if (null != valueAnnotation) {
                String value = valueAnnotation.value();
                value = beanFactory.resolveEmbeddedValue(value);
                BeanUtil.setFieldValue(bean, field.getName(), value);
            }
        }

        // 2. 处理注解 @Autowired
        for (Field field : declaredFields) {
            Autowired autowiredAnnotation = field.getAnnotation(Autowired.class);
            if (null != autowiredAnnotation) {
                Class<?> fieldType = field.getType();
                String dependentBeanName = null;
                Qualifier qualifierAnnotation = field.getAnnotation(Qualifier.class);
                Object dependentBean = null;
                if (null != qualifierAnnotation) {
                    dependentBeanName = qualifierAnnotation.value();
                    dependentBean = beanFactory.getBean(dependentBeanName, fieldType);
                } else {
                    dependentBean = beanFactory.getBean(fieldType);
                }
                BeanUtil.setFieldValue(bean, field.getName(), dependentBean);
            }
        }

        return pvs;
    }
```
## 循环依赖
循环依赖主要分为这三种，自身依赖于自身、互相循环依赖、多组循环依赖。  
循环依赖需要用到三个缓存，这三个缓存分别存放了成品对象、半成品对象(未填充属性值)、代理对象，分阶段存放对象内容，来解决循环依赖问题。  
用于解决循环依赖需要用到三个缓存，这三个缓存分别存放了成品对象、半成品对象(未填充属性值)、代理对象，分阶段存放对象内容，来解决循环依赖问题。  
关于循环依赖在我们目前的 Spring 框架中扩展起来也并不会太复杂，主要就是对于创建对象的提前暴露，如果是工厂对象则会使用 getEarlyBeanReference 逻辑提前将工厂🏭对象存放到三级缓存中。等到后续获取对象的时候实际拿到的是工厂对象中 getObject，这个才是最终的实际对象。  

## 事务功能设计
基本原理：通过AOP的方式来设置关闭数据库的自动提交事务，如：connection.setAutoCommit(false)，然后在程序中在合适的位置进行手动提交事务和回滚事务  


## 实际使用--JDBCTemplate
JdbcTemplate可以作为一个普通的bean来管理，里面定义了对数据库的操作，实际上是依赖的各种数据库的驱动。  
想要完成连接数据库也需要引入其他的bean，例如DriverManagerDataSource连接池技术。  

## 整合ORM框架
实际上是将ORM框架的连接信息和执行sql的信息交给Spring来管理  
例如整合mybatis框架时实现了一个SqlsessionFactoryBuild的工厂类对sqlSession进行管理，同时扫描到众多的执行sql的bean，在ORM完成sql到bean的方法的映射后，注入到spring中，直接执行就能与数据库关联  





















