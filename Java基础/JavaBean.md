## 核心类--JavaBean
JavaBean 是特殊的 Java 类，使用 Java 语言书写，并且遵守 JavaBean API 规范。
接下来给出的是 JavaBean 与其它 Java 类相比而言独一无二的特征：

提供一个默认的无参构造函数。
需要被序列化并且实现了 Serializable 接口。
可能有一系列可读写属性。
可能有一系列的 getter 或 setter 方法。
JavaBean 属性
一个 JavaBean 对象的属性应该是可访问的。这个属性可以是任意合法的 Java 数据类型，包括自定义 Java 类。

一个 JavaBean 对象的属性可以是可读写，或只读，或只写。JavaBean 对象的属性通过 JavaBean 实现类中提供的两个方法来访问：
getPropertyName()：举例来说，如果属性的名称为 myName，那么这个方法的名字就要写成 getMyName() 来读取这个属性。这个方法也称为访问器。
setPropertyName()：举例来说，如果属性的名称为 myName，那么这个方法的名字就要写成 setMyName()来写入这个属性。这个方法也称为写入器。
boolean字段比较特殊，它的读方法一般命名为isXyz()：
```
// 读方法:
public boolean isChild()
// 写方法:
public void setChild(boolean value)
```
访问JavaBean
<jsp:useBean> 标签可以在 JSP 中声明一个 JavaBean，然后使用。声明后，JavaBean 对象就成了脚本变量，可以通过脚本元素或其他自定义标签来访问。<jsp:useBean> 标签的语法格式如下：
<jsp:useBean id="bean 的名字" scope="bean 的作用域" typeSpec/>
其中，根据具体情况，scope 的值可以是 page，request，session 或 application。id值可任意只要不和同一 JSP 文件中其它 <jsp:useBean> 中 id 值一样就行了。
枚举JavaBean属性
要枚举一个JavaBean的所有属性，可以直接使用Java核心库提供的Introspector：
```
public class Main {
    public static void main(String[] args) throws Exception {
        BeanInfo info = Introspector.getBeanInfo(Person.class);
        for (PropertyDescriptor pd : info.getPropertyDescriptors()) {
            System.out.println(pd.getName());
            System.out.println("  " + pd.getReadMethod());
            System.out.println("  " + pd.getWriteMethod());
        }
    }
}

class Person {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}

```
什么是SpringBean:
SpringBean是受Spring管理的对象  所有能受Spring容器管理的对象都可以成为SpringBean.

二者之间的区别：

用处不同：传统javabean更多地作为值传递参数，而spring中的bean用处几乎无处不在，任何组件都可以被称为bean

写法不同：传统javabean作为值对象，要求每个属性都提供getter和setter方法；但spring中的bean只需为接受设值注入的属性提供setter方法

生命周期不同：传统javabean作为值对象传递，不接受任何容器管理其生命周期；spring中的bean有spring管理其生命周期行为
JavaBean 是特殊的 Java 类，使用 Java 语言书写，并且遵守 JavaBean API 规范。
接下来给出的是 JavaBean 与其它 Java 类相比而言独一无二的特征：

提供一个默认的无参构造函数。
需要被序列化并且实现了 Serializable 接口。
可能有一系列可读写属性。
可能有一系列的 getter 或 setter 方法。
JavaBean 属性
一个 JavaBean 对象的属性应该是可访问的。这个属性可以是任意合法的 Java 数据类型，包括自定义 Java 类。

一个 JavaBean 对象的属性可以是可读写，或只读，或只写。JavaBean 对象的属性通过 JavaBean 实现类中提供的两个方法来访问：
getPropertyName()：举例来说，如果属性的名称为 myName，那么这个方法的名字就要写成 getMyName() 来读取这个属性。这个方法也称为访问器。
setPropertyName()：举例来说，如果属性的名称为 myName，那么这个方法的名字就要写成 setMyName()来写入这个属性。这个方法也称为写入器。
boolean字段比较特殊，它的读方法一般命名为isXyz()：
```
// 读方法:
public boolean isChild()
// 写方法:
public void setChild(boolean value)
```
访问JavaBean
<jsp:useBean> 标签可以在 JSP 中声明一个 JavaBean，然后使用。声明后，JavaBean 对象就成了脚本变量，可以通过脚本元素或其他自定义标签来访问。<jsp:useBean> 标签的语法格式如下：
<jsp:useBean id="bean 的名字" scope="bean 的作用域" typeSpec/>
其中，根据具体情况，scope 的值可以是 page，request，session 或 application。id值可任意只要不和同一 JSP 文件中其它 <jsp:useBean> 中 id 值一样就行了。
枚举JavaBean属性
要枚举一个JavaBean的所有属性，可以直接使用Java核心库提供的Introspector：
```
public class Main {
    public static void main(String[] args) throws Exception {
        BeanInfo info = Introspector.getBeanInfo(Person.class);
        for (PropertyDescriptor pd : info.getPropertyDescriptors()) {
            System.out.println(pd.getName());
            System.out.println("  " + pd.getReadMethod());
            System.out.println("  " + pd.getWriteMethod());
        }
    }
}

class Person {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}

```
什么是SpringBean:
SpringBean是受Spring管理的对象  所有能受Spring容器管理的对象都可以成为SpringBean.

二者之间的区别：

用处不同：传统javabean更多地作为值传递参数，而spring中的bean用处几乎无处不在，任何组件都可以被称为bean

写法不同：传统javabean作为值对象，要求每个属性都提供getter和setter方法；但spring中的bean只需为接受设值注入的属性提供setter方法

生命周期不同：传统javabean作为值对象传递，不接受任何容器管理其生命周期；spring中的bean有spring管理其生命周期行为