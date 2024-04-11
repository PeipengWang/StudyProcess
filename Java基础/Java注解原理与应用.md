## 注解的本质
注解的本质就是一个继承了 Annotation 接口的接口。
一个注解准确意义上来说，只不过是一种特殊的注释而已，如果没有解析它的代码，它可能连注释都不如。
而解析一个类或者方法的注解往往有两种形式，一种是编译期直接的扫描，一种是运行期反射。
典型的就是注解 @Override，一旦编译器检测到某个方法被修饰了 @Override 注解，编译器就会检查当前方法的方法签名是否真正重写了父类的某个方法，也就是比较父类中是否具有一个同样的方法签名。
## 注解入门
注解的作用？
1，不是程序本身，可以对程序作出解释（类似注释）
2，可以被其他程序（如编译器）读取，可以作为信息处理的流程。
注解的格式？
@注释名，还可以添加一些参数值，例如@SuppressWarnings（value = ”unchecked“）
使用的位置？
package，calss，method，field都可以额外添加注解

## 内置注解--三个
@Deprecated 已过期，表示方法是不被建议使用的
@Override 重写，标识覆盖它的父类的方法
@SuppressWarnings 压制警告，抑制警告
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
## 元注解--四个
『元注解』是用于修饰注解的注解，通常用在注解的定义上，例如：
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {

}
```
@Target：描述注解的使用范围，有四个常用属性限定注解能运用的地方
@Retention：用于指定被修饰的自定义注解可以保留多久，有三个常用属性
@Documented：执行javadoc命令时，被该元注解修饰的自定义注解也会生成在文档中
@Inherited：如果父类所使用的注解有此修饰，则子类可以继承该注解，否则不能
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
## 自定义注解
- 自定义格式：public @interface 注解名
- 利用元注解进行填充自定义注解的意义
- 注意：
 *  注解元素必须有值，我们定义注解元素时，经常使用空字符串，0作为默认值，也经常使用-1作为不存在含义

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(value = {ElementType.ANNOTATION_TYPE,ElementType.FIELD,ElementType.METHOD})
@Retention(RetentionPolicy.CLASS)
public @interface DefineByMyself {
    String str() default  "";
    int age()  default  0;
}
```
