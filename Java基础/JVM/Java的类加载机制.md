@[toc]
# 什么是类加载
每个编写的".java"拓展名类文件都存储着需要执行的程序逻辑，这些".java"文件经过Java编译器编译成拓展名为".class"的文件，".class"文件中保存着Java代码经转换后的虚拟机指令，当需要使用某个类时，虚拟机将会加载它的".class"文件，并创建对应的class对象，将class文件加载到虚拟机的内存，这个过程称为类加载，这里我们需要了解一下类加载的过程，如下：
具体流程如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/133a3563d72046faa1d5dd41831017e1.png)
## 步骤
类加载器的工作可以简化为三个步骤：

 - 加载（Loading）：根据类的全限定名（包括包路径和类名），定位并读取类文件的字节码。
 具体来说：在加载阶段，虚拟机会完成三件事：
通过一个类的全限定性类名来获取此类的二进制字节流。
通过这个字节流所代表的静态存储结构转化为运行时数据结构。
在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据访问入口。
 - 链接（Linking）：将类的字节码转换为可以在虚拟机中运行的格式。链接过程包括三个阶段：
       验证（Verification）：验证字节码的正确性和安全性，确保它符合Java虚拟机的规范。
        准备（Preparation）：为类的静态变量分配内存，并设置默认的初始值，如果这个类变量用final修饰，则会初始化为设置的值，如果不是final类型的则设置为0，false或者null。
        解析（Resolution）：将类的符号引用（比如方法和字段的引用）解析为直接引用（内存地址），主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符这七类符号引用今行，分别对应常量池的八种常量类型的表格。
 - 初始化（Initialization）：执行类的初始化代码，包括静态变量的赋值和静态块的执行。
# 类加载器
## 三个类加载器
BootStrap ClassLoader：称为启动类加载器，是Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等
Extension ClassLoader：称为扩展类加载器，负责加载Java的扩展类库，默认加载JAVA_HOME/jre/lib/ext/目下的所有jar。
App ClassLoader：称为系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件。
 除了Java默认提供的三个ClassLoader之外，用户还可以根据需要定义自已的ClassLoader，而这些自定义的ClassLoader都必须继承自java.lang.ClassLoader类，也包括Java提供的另外二个ClassLoader（Extension ClassLoader和App ClassLoader）在内，但是Bootstrap ClassLoader不继承自ClassLoader，因为它不是一个普通的Java类，底层由C++编写，已嵌入到了JVM内核当中，当JVM启动后，Bootstrap ClassLoader也随着启动，负责加载完核心类库后，并构造Extension ClassLoader和App ClassLoader类加载器。
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/d78b7ed26e6247e19921a8d2b6b18487.png)

##  定义自已的ClassLoader
既然JVM已经提供了默认的类加载器，为什么还要定义自已的类加载器呢？
      因为Java中提供的默认ClassLoader，只加载指定目录下的jar和class，如果我们想加载其它位置的类或jar时，比如：我要加载网络上的一个class文件，通过动态加载到内存之后，要调用这个类中的方法实现我的业务逻辑。在这样的情况下，默认的ClassLoader就不能满足我们的需求了，所以需要定义自己的ClassLoader。
定义自已的类加载器分为两步：
1、继承java.lang.ClassLoader
2、重写父类的findClass方法
实例

```java
public class CustomClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 根据名称加载类的字节码
        byte[] byteCode = loadClassByteCode(name);
        // 调用defineClass方法将字节码转换为Class对象
        return defineClass(name, byteCode, 0, byteCode.length);
    }
    
    private byte[] loadClassByteCode(String name) {
        // 实现加载类字节码的逻辑
        // ...
    }
}
```
## 双亲委派模型原理
ClassLoader使用的是双亲委托模型来搜索类的，每个ClassLoader实例都有一个父类加载器的引用（不是继承的关系，是一个包含的关系），虚拟机内置的类加载器（Bootstrap ClassLoader）本身没有父类加载器，但可以用作其它ClassLoader实例的的父类加载器。当一个ClassLoader实例需要加载某个类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是由上至下依次检查的，首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象。

