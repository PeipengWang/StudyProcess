# Spring的容器事件和事件监听器
## 基本概念
在Spring框架中，事件（Event）和事件监听器（EventListener）是用于实现发布-订阅模式的重要机制，用于解耦应用程序中的组件。Spring的事件机制建立在观察者模式的基础上，通过定义事件和事件监听器，允许应用程序中的不同部分在发生特定事件时进行通信。
ApplicationEvent（应用事件）：表示事件的基本类。您可以扩展此类以创建自定义事件。
ApplicationEventPublisher（应用事件发布者）：用于发布事件的接口
ApplicationListener（应用监听器）：是用于处理应用事件的接口，当某个事件被发布时，注册的监听器将被调用。
定义方法
```
    ApplicationListener<E extends ApplicationEvent> extends EventListener
```

## 实现实例：
应用事件
```
package event;

import org.springframework.context.ApplicationEvent;

public class MyCustomEvent extends ApplicationEvent {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public MyCustomEvent(Object source) {
        super(source);
    }
}


```
事件监听者
```
package event;

import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component
public class MyEventListener implements ApplicationListener<MyCustomEvent> {
    @Override
    public void onApplicationEvent(MyCustomEvent event) {
        // 处理事件的逻辑
        System.out.println("当前登录用户为：" + event.getName());
    }
}


```
事件发布者
```
package event;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

@Service
public class MyEventPublisherService {
    @Autowired
    private ApplicationEventPublisher eventPublisher;

    public void publishCustomEvent(String username) {
        // 发布自定义事件
        MyCustomEvent myCustomEvent = new MyCustomEvent(this);
        myCustomEvent.setName(username);
        eventPublisher.publishEvent(myCustomEvent);
    }
}


```

定义一个登录controller实现事件发布，打印登录用户
```
    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public ModelAndView login(HttpSession session, ServletRequest request,
                              @RequestParam String username, @RequestParam String password) {
        String username1 = request.getParameter("username");
        myEventPublisherService.publishCustomEvent(username);
        return model;
    }
```
执行登录指令
打印数据：

当前登录用户为：super

## 关键代码
### 初始化事件和监听器
定义事件的关键代码
AbstractApplicationContext.publishEvent
```
public abstract class AbstractApplicationEventMulticaster implements ApplicationEventMulticaster, BeanFactoryAware {

    public final Set<ApplicationListener<ApplicationEvent>> applicationListeners = new LinkedHashSet<>();

    private BeanFactory beanFactory;

    @Override
    public void addApplicationListener(ApplicationListener<?> listener) {
        applicationListeners.add((ApplicationListener<ApplicationEvent>) listener);
    }

    @Override
    public void removeApplicationListener(ApplicationListener<?> listener) {
        applicationListeners.remove(listener);
    }


    protected Collection<ApplicationListener> getApplicationListeners(ApplicationEvent event) {
        LinkedList<ApplicationListener> allListeners = new LinkedList<ApplicationListener>();
        for (ApplicationListener<ApplicationEvent> listener : applicationListeners) {
            if (supportsEvent(listener, event)) allListeners.add(listener);
        }
        return allListeners;
    }
     /**
     * 监听器是否对该事件感兴趣
     */
    protected boolean supportsEvent(ApplicationListener<ApplicationEvent> applicationListener, ApplicationEvent event) {
        Class<? extends ApplicationListener> listenerClass = applicationListener.getClass();

        // 按照 CglibSubclassingInstantiationStrategy、SimpleInstantiationStrategy 不同的实例化类型，需要判断后获取目标 class
        Class<?> targetClass = ClassUtils.isCglibProxyClass(listenerClass) ? listenerClass.getSuperclass() : listenerClass;
        Type genericInterface = targetClass.getGenericInterfaces()[0];

        Type actualTypeArgument = ((ParameterizedType) genericInterface).getActualTypeArguments()[0];
        String className = actualTypeArgument.getTypeName();
        Class<?> eventClassName;
        try {
            eventClassName = Class.forName(className);
        } catch (ClassNotFoundException e) {
            throw new BeansException("wrong event class name: " + className);
        }
        // 判定此 eventClassName 对象所表示的类或接口与指定的 event.getClass() 参数所表示的类或接口是否相同，或是否是其超类或超接口。
        // isAssignableFrom是用来判断子类和父类的关系的，或者接口的实现类和接口的关系的，默认所有的类的终极父类都是Object。如果A.isAssignableFrom(B)结果是true，证明B可以转换成为A,也就是A可以由B转换而来。
        return eventClassName.isAssignableFrom(event.getClass());
    }
}

```
AbstractApplicationContext 中的refresh注册事件

```
    @Override
    public void refresh() throws BeansException {

        // 6. 初始化事件发布者，主要用于实例化一个 SimpleApplicationEventMulticaster，这是一个事件广播器。
        initApplicationEventMulticaster();

        // 7. 注册事件监听器，获取到所有从 spring.xml 中加载到的事件配置 Bean 对象。
        registerListeners();

        // 9. 发布容器刷新完成事件，发布了第一个服务器启动完成后的事件，这个事件通过 publishEvent 发布出去，
        finishRefresh();
    }
    
    private void registerListeners() {
        Collection<ApplicationListener> applicationListeners = getBeansOfType(ApplicationListener.class).values();
        for (ApplicationListener listener : applicationListeners) {
            applicationEventMulticaster.addApplicationListener(listener);
        }
    }

```
一个监听器会对应一个事件，当其他的业务想要发布某个事件的时候，监听器会执行相关逻辑来完成监听服务

### publishEvent中整个事件发布的流程
在完成注册事件和监听器后，可以通过 publishEvent进行发布事件，发布会通知响应的订阅这个事件的监听器.。
在使用上，会使用applicationContext.publishEvent("我是一个事件");来发布一个具体的事件

关键代码为：
```
protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
 	Assert.notNull(event, "Event must not be null");

 	// Decorate event as an ApplicationEvent if necessary
 	ApplicationEvent applicationEvent;
 	if (event instanceof ApplicationEvent) {
 		applicationEvent = (ApplicationEvent) event;
 	}
 	else {
 		applicationEvent = new PayloadApplicationEvent<>(this, event);
 		if (eventType == null) {
 			eventType = ((PayloadApplicationEvent<?>) applicationEvent).getResolvableType();
 		}
 	}

 	// Multicast right now if possible - or lazily once the multicaster is initialized
 	if (this.earlyApplicationEvents != null) {
 		this.earlyApplicationEvents.add(applicationEvent);
 	}
 	else {
 		getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
 	}

 	// Publish event via parent context as well...
 	if (this.parent != null) {
 		if (this.parent instanceof AbstractApplicationContext) {
 			((AbstractApplicationContext) this.parent).publishEvent(event, eventType);
 		}
 		else {
 			this.parent.publishEvent(event);
 		}
 	}
 }
```
如果发布的直接就是ApplicationEvent类型的事件，那么就直接转换成ApplicationEvent类型，而如果不是ApplicationEvent类型的事件，那么就是我们所说的Object类型的事件，就会帮我们封装成PayloadApplicationEvent，并将Object类型的事件信息存储到payload属性中。在处理完事件的类型后，执行了下面这行重要的代码：
```
getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
```
其中getApplicationEventMulticaster方法是拿到容器中的事件广播器，然后通过这个事件广播器来进行事件的广播。那么这个事件多播器我们也没有配置创建，是怎么获取的呢？这就需要看一下Spring容器刷新的当中的initApplicationEventMulticaster初始化事件广播器的方法了，如下：
```
	protected void initApplicationEventMulticaster() {
 	ConfigurableListableBeanFactory beanFactory = getBeanFactory();
 	if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
 		this.applicationEventMulticaster =
 				beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
 		if (logger.isTraceEnabled()) {
 			logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
 		}
 	}
 	else {
 		this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
 		beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
 		if (logger.isTraceEnabled()) {
 			logger.trace("No '" + APPLICATION_EVENT_MULTICASTER_BEAN_NAME + "' bean, using " +
 					"[" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
 		}
 	}
 }

```
我们默认就是走的SimpleApplicationEventMulticaster中的multicastEvent事件广播逻辑
```
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
 	ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
 	Executor executor = getTaskExecutor();
 	for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
 		if (executor != null) {
 			executor.execute(() -> invokeListener(listener, event));
 		}
 		else {
 			invokeListener(listener, event);
 		}
 	}
 }

```
看到这里也就验证了我们刚开始的猜测，Spring正式通过拿到容器中所有符合当前事件的监听器，然后循环遍历挨个调用onApplicationEvent方法。

部分摘自：https://blog.csdn.net/ITlikeyou/article/details/124773814







