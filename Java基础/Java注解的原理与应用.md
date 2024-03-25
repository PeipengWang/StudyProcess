# 注解原理与应用
## 注解的本质
注解的本质就是一个继承了 Annotation 接口的接口。
一个注解准确意义上来说，只不过是一种特殊的注释而已，如果没有解析它的代码，它可能连注释都不如。
而解析一个类或者方法的注解往往有两种形式，一种是编译期直接的扫描，一种是运行期反射。
典型的就是注解 @Override，一旦编译器检测到某个方法被修饰了 @Override 注解，编译器就会检查当前方法的方法签名是否真正重写了父类的某个方法，也就是比较父类中是否具有一个同样的方法签名。
## 元注解
『元注解』是用于修饰注解的注解，通常用在注解的定义上，例如：
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {

}
```
JAVA 中有以下几个『元注解』：
@Target：注解的作用目标
@Retention：注解的生命周期
@Documented：注解是否应当被包含在 JavaDoc 文档中
@Inherited：是否允许子类继承该注解

### @Target
被这个 @Target 注解修饰的注解将只能作用在成员字段上，不能用于修饰方法或者类。其中，ElementType 是一个枚举类型，有以下一些值：
ElementType.TYPE：允许被修饰的注解作用在类、接口和枚举上
ElementType.FIELD：允许作用在属性字段上
ElementType.METHOD：允许作用在方法上
ElementType.PARAMETER：允许作用在方法参数上
ElementType.CONSTRUCTOR：允许作用在构造器上
ElementType.LOCAL_VARIABLE：允许作用在本地局部变量上
ElementType.ANNOTATION_TYPE：允许作用在注解上
ElementType.PACKAGE：允许作用在包上

### @Retention
这里的 RetentionPolicy 依然是一个枚举类型，它有以下几个枚举值可取：

RetentionPolicy.SOURCE：当前注解编译期可见，不会写入 class 文件
RetentionPolicy.CLASS：类加载阶段丢弃，会写入 class 文件
RetentionPolicy.RUNTIME：永久保存，可以反射获取

## JAVA 的内置三大注解
@Override
@Deprecated
@SuppressWarnings

### @Override
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
它没有任何的属性，所以并不能存储任何其他信息。它只能作用于方法之上，编译结束后将被丢弃。
所以你看，它就是一种典型的『标记式注解』，仅被编译器可知，编译器在对 java 文件进行编译成字节码的过程中，一旦检测到某个方法上被修饰了该注解，就会去匹对父类中是否具有一个同样方法签名的函数，如果不是，自然不能通过编译。

### @Deprecated
依然是一种『标记式注解』，永久存在，可以修饰所有的类型，作用是，标记当前的类或者方法或者字段等已经不再被推荐使用了，可能下一次的 JDK 版本就会删除。
当然，编译器并不会强制要求你做什么，只是告诉你 JDK 已经不再推荐使用当前的方法或者类了，建议你使用某个替代者。

### @SuppressWarnings
主要用来压制 java 的警告
