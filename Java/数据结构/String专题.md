## String三大特性
不变性：是一个 immutable 模式的对象，不不变模式的主要作用是当一个对象需要被多线程共享并频繁访问时，可以保证数据的一致性。
常量池优化：String对象创建后，会在字符串常量池中进行缓存，下次创建同样的对象的时候，会直接返回缓存的引用。
final：String类不可被继承，提高了系统的安全性
注意：Sring不是基本数据类型
## String实例化：两种方式
①直接赋值

```
String str= “Hello World“；
```

②通过构造器，可以直接将字符串传入，也可以传入一个char数组

```
String str = new String("Hello World");
chars s = {'你','好'}；
String str2 = new String(s);
```
两者不同：直接赋值和通过构造函数创建主要区别在于存储的区域不不同，直接赋值存储在字符串串常量量池中每次创建对象都会在常量池中寻找，如果有则直接指向这个常量池中数据的地址，如果没有则创建放入常量池中，如下图所示；
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200721223930393.png)
因此，直接构造由于具有final特性，因此如果定义两个一样的”Hello“，两个定义的常量的地址是一样的，验证代码如下所示

```
     String str1 = "Hello";
     String str2 = "Hello";
     System.out.println(str1 == str2);
```
上面代码输出结果为true。
需要注意的是”==“执行的是地址是否相同的操作，由此可以看出，虽然定义了两个str1，str2，但是他们所指向的字符串地址是相同的。
相反，通过构造函数创建，存储在堆内存中。
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200721224014658.png)

对于通过构造器赋值的验证程序如下所示：

```
        String str3 = new String("Hello");
        String str4 = new String("Hello");
        System.out.println(str3 == str4);
```
输出结果为false，由此可以知道，通过构造器创建的字符串的对象是不同的对象，虽然值是一样的，但不保存在同一地址。

## equals方法
对于”==“，如果数据是基本数据类型，那么比较的是数据的值，但如果是引用类型，那么比较的将会是对象的地址，但是对于地址的比较往往是不需要的，更需要的是对象的值得比较，因此有了equals方法。
equals方法的底层代码如下所示

```
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String aString = (String)anObject;
            if (!COMPACT_STRINGS || this.coder == aString.coder) {
                return StringLatin1.equals(value, aString.value);
            }
        }
        return false;
    }
```
由代码可知，equals方法首先直接比较地址，相等就直接返回true，然后利用了instanceof方法判断，然后把String转换为byte数组（jdk1.8之前是char数组），判断byte数组中每个值是否完全一致，一致则返回true，否则就false。
## intern() 方法
当调用某个字符串串**对象**的 intern() 方式，会去**字符串常量量池**中寻找，如果已经存在一个值相等的字符串串
对象的话，则直接返回该对象的引用，如果不不存在，则在字符串串常量量池中创建该对象，并返回。
## 常用的方法
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200722095233907.png)

①![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/20200722095157416.png)
1，字符串截取

```
public String subString(int beginIndex);
public String subString(int beginIndex,int endIndex);
```
包含beginIndex，不包括endIndex；


2，字符串分割

```
public String[]  split(String str);
```
str是分割符，分割成几个字符串。
split 方法支持传入正则表达式，进⾏行行复杂的字符串串分割，比如 "Hello,World;Java-String"，如果要分别
将 Hello、World、Java、String 截取出来，使用统一的某个分割符肯定⽆无法完成，这时候可以借助于正
则表达式 "[,|;|-]" 来实现，具体操作如下所示。同时split方法支持正则表达式，实现复杂的分割。

```
String str = new String("Hello,World;Java-String");
String[] array = str.split("[,|;|-]");
for (String item:array){
System.out.println(item);
}
```

## 题目
1，判断true和false

```
String str1 = "Hello World";
String str2 = "Hello"+" World";
System.out.println(str1 == str2)
```
true，"Hello" 和 " World" 都是字符串串字面值，字符串串字⾯面值 + 字符串串字面值的结果仍然保存在字符串串
常量量池中，所以 str1 和 str2 相同。
```
String str1 = "Hello World";
String str2 = "Hello";
str2 += " World";
System.out.println(str1 == str2);
```
false，这题看似与第 2 题一样，为什什么结果完全不不同呢？因为 str2 = "Hello"+" World" 是直接创建，str2 = "Hello"; str2 = "Hello" 是先创建再修改，同时修改完成之后的字符串串是放在堆内存中的，为什什么呢？因为 str2 是一个字符串变量，" World" 是字符串串字面值，当字符串串字面值与 String 类型变量量拼接时，得到的新字符串串不不再保存在常量量池中，而是在堆中开辟一块新的空间来存储。


```
String str1 = "Hello World";
String str2 = " World";
String str3 = "Hello"+str2;
System.out.println(str1 == str3);
```

false，str2 是变量，"Hello" 是字符串字面值，字符串字面值 + 变量量会在堆内存中开辟新的空间来存储，所以 str1 和 str3 不不同。

```
String str1 = "Hello World";
final String str2 = " World";
String str3 = "Hello"+str2;
System.out.println(str1 == str3);
```
true，"Hello" 是字符串串字面值，str2 是常量量，字符串字面值+常量的结果仍然保存在字符串常量量池中，所以 str1 和 str3 相同。

```
String str1 = "Hello World";
final String str2 = new String(" World");
String str3 = "Hello"+str2;
System.out.println(str1 == str3);
```
false，str2 是常量，但是 new String(" World") 保存在堆内存中，所以即使使用 final 进行了修饰，str2 仍然保存在堆中，则 str3 也就保存在堆中，所以 str1 和 str3 不不同。

```
String str1 = "Hello World";
String str2 = "Hello";
String str3 = " World";
String str4 = str2 + str3;
System.out.println(str4 == str1);
System.out.println(str4.intern() == str1);
```
true，当调用 str4 的 intern 方法时，如果字符串串常量池已经包含一个等于 str4 的字符串串，则返回该字符串串，否则将 str4 添加到字符串串常量量池中，并返回其引用，所以 str4.intern() 与 str1 相同。

补充
常量 final String 或者 ”Hello“
变量String str = "Hello"
**变量+常量在堆里面**
**常量+常量在常量池里面**

2，什么是字符串常量池？
字符串串常量量池位于堆内存中，专门用来存储字符串串常量量，可以提高内存的使用率，避免开辟多块空间存储相同的字符串串，在创建字符串串时 JVM 会首先检查字符串串常量量池，如果该字符串串已经存在池中，则返回它的引用，如果不不存在，则实例例化一个字符串串放到池中，并返回其引用。

3，String 是线程安全的吗？
String 是不可变类，一旦创建了了String对象，我们就无法改变它的值。因此，它是线程安全的，同一个字符串串实例例可以被多个线程共享，保证了了多线程的安全性。
4，在使用 HashMap 的时候，用 String 做 key 有什什么好处？
HashMap 内部实现是通过 key 的 hashcode 来确定 value 的存储位置，而因为字符串是不可变的，当创建字符串时，它的 hashcode 被缓存下来，不需要再次计算，所以相比于其他对象更更快。

