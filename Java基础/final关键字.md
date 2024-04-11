final用于声明属性、方法和类，分别表示属性不可改变，方法不可覆盖和类不可被继承。
1）final属性：被final修饰的变量不可改变。由于不可改变有两重含义：一是引用不可改变；二是对象不可改变。通过测试查看是哪一个不可变法

```c
public class testFinal {
    public static void main(String[] args) {
        final StringBuilder str = new StringBuilder("Hello");
        str.append(" Word");
        System.out.println(str);
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200423190511617.png)

```c
public class testFinal {
    public static void main(String[] args) {
        final StringBuilder str = new StringBuilder("Hello");
        str = new StringBuilder("Word");
        System.out.println();
    }
}

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200423190700332.png)
从上面的例子可以得出，final是引用类型的变量，而不是引用对象，一般可以进行以下的初始化：①，在定义的时候初始化；②，final成员变量可以的初始化块中初始化，但是不能再静态初始化块中初始化；③，静态final可以在静态初始化块中初始化，但是不能再初始化块中初始化；④，在类构造器中初始化，但是静态final不能在构造器中初始化。
2）final方法：final方法不允许任何子类重写该方法，但是允许子类仍然使用该方法。另外还有一种内联机制，当调用一个被声明为final方法时，直接将方法主体插入到该方法处而不需要进行充分调用，这样能够提高程序的运行效率。
3）final参数：用来表示这个参数在这个函数内部不允许被修改。
4）final类：当一个类被定义为final时，这个类不能被继承，也不能被重写，需要注意的是不能同时定义一个类为abstract和final类型的。