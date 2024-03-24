## 序列化与反序列化
## 什么是序列化与反序列化
Java序列化是将对象转换为字节流的过程，以便将其存储到文件、数据库或通过网络传输。 序列化是指将Java对象转换为字节序列的过程，而反序列化则是将字节序列转换为Java对象的过程。
Java对象序列化是将实现了Serializable接口的对象转换成一个字节序列，能够通过网络传输、文件存储等方式传输 ，传输过程中却不必担心数据在不同机器、不同环境下发生改变，也不必关心字节的顺序或其他任何细节，并能够在以后将这个字节序列完全恢复为原来的对象(恢复这一过程称之为反序列化)。
对象的序列化是非常有趣的，因为利用它可以实现轻量级持久性，“持久性”意味着一个对象的生存周期不单单取决于程序是否正在运行，它可以生存于程序的调用之间。通过将一个序列化对象写入磁盘，然后在重新调用程序时恢复该对象，从而达到实现对象的持久性的效果。
本质上讲，序列化就是把实体对象状态按照一定的格式写入到有序字节流，反序列化就是从有序字节流重建对象，恢复对象状态。
## Java怎么实现序列化与反序列化
只要对象实现了Serializable、Externalizable接口(该接口仅仅是一个标记接口，并不包含任何方法)，则该对象就实现了序列化。
以下是Java序列化的基本示例：
```
import java.io.*;

// 定义一个可序列化的类
class Person implements Serializable {
    private static final long serialVersionUID = 1L; // 序列化版本号
    private String name;
    private int age;

    // 构造方法
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Getter 和 Setter 方法

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

    // 重写toString方法
    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age + "]";
    }
}

public class SerializationExample {
    public static void main(String[] args) {
        // 创建一个Person对象
        Person person = new Person("John", 30);

        // 序列化对象
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.ser"))) {
            oos.writeObject(person);
            System.out.println("Person对象已被序列化并保存到文件");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 反序列化对象
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.ser"))) {
            Person deserializedPerson = (Person) ois.readObject();
            System.out.println("从文件中反序列化的Person对象: " + deserializedPerson);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}


```
在这个例子中，Person 类实现了Serializable 接口，这使得它的对象可以被序列化。 ObjectOutputStream 用于将对象写入文件，而 ObjectInputStream 用于从文件读取并反序列化对象。在序列化的过程中，会将对象的状态转换为字节流，并写入文件；在反序列化的过程中，会从文件读取字节流，并将其转换回对象的状态。
## 序列化与反序列化的应用场景
当你想把的内存中的对象状态保存到一个文件中或者数据库中时候。
当你想用套接字在网络上传送对象的时候。
当你想通过RMI传输对象的时候。

## 序列化需要注意
1、序列化时，只对对象的状态进行保存，而不管对象的方法；
2、当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；
3、当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；
4、并非所有的对象都可以序列化，至于为什么不可以，有很多原因了，比如：
安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行RMI传输等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的；
资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配，而且，也是没有必要这样实现；
5、声明为static和transient类型的成员数据不能被序列化。因为static代表类的状态，transient代表对象的临时数据。
6、序列化运行时使用一个称为 serialVersionUID 的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。为它赋予明确的值。显式地定义serialVersionUID有两种用途：
在某些场合，希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有相同的serialVersionUID；
在某些场合，不希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有不同的serialVersionUID。
7、Java有很多基础类已经实现了serializable接口，比如String,Vector等。但是也有一些没有实现serializable接口的；
8、如果一个对象的成员变量是一个对象，那么这个对象的数据成员也会被保存！这是能用序列化解决深拷贝的重要原因；


引用：
https://cloud.tencent.com/developer/article/1511793
https://cloud.tencent.com/developer/article/1420284
## 什么是序列化与反序列化
Java序列化是将对象转换为字节流的过程，以便将其存储到文件、数据库或通过网络传输。 序列化是指将Java对象转换为字节序列的过程，而反序列化则是将字节序列转换为Java对象的过程。
Java对象序列化是将实现了Serializable接口的对象转换成一个字节序列，能够通过网络传输、文件存储等方式传输 ，传输过程中却不必担心数据在不同机器、不同环境下发生改变，也不必关心字节的顺序或其他任何细节，并能够在以后将这个字节序列完全恢复为原来的对象(恢复这一过程称之为反序列化)。
对象的序列化是非常有趣的，因为利用它可以实现轻量级持久性，“持久性”意味着一个对象的生存周期不单单取决于程序是否正在运行，它可以生存于程序的调用之间。通过将一个序列化对象写入磁盘，然后在重新调用程序时恢复该对象，从而达到实现对象的持久性的效果。
本质上讲，序列化就是把实体对象状态按照一定的格式写入到有序字节流，反序列化就是从有序字节流重建对象，恢复对象状态。
## Java怎么实现序列化与反序列化
只要对象实现了Serializable、Externalizable接口(该接口仅仅是一个标记接口，并不包含任何方法)，则该对象就实现了序列化。
以下是Java序列化的基本示例：
```
import java.io.*;

// 定义一个可序列化的类
class Person implements Serializable {
    private static final long serialVersionUID = 1L; // 序列化版本号
    private String name;
    private int age;

    // 构造方法
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Getter 和 Setter 方法

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

    // 重写toString方法
    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age + "]";
    }
}

public class SerializationExample {
    public static void main(String[] args) {
        // 创建一个Person对象
        Person person = new Person("John", 30);

        // 序列化对象
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.ser"))) {
            oos.writeObject(person);
            System.out.println("Person对象已被序列化并保存到文件");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 反序列化对象
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.ser"))) {
            Person deserializedPerson = (Person) ois.readObject();
            System.out.println("从文件中反序列化的Person对象: " + deserializedPerson);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}


```
在这个例子中，Person 类实现了Serializable 接口，这使得它的对象可以被序列化。 ObjectOutputStream 用于将对象写入文件，而 ObjectInputStream 用于从文件读取并反序列化对象。在序列化的过程中，会将对象的状态转换为字节流，并写入文件；在反序列化的过程中，会从文件读取字节流，并将其转换回对象的状态。
## 序列化与反序列化的应用场景
当你想把的内存中的对象状态保存到一个文件中或者数据库中时候。
当你想用套接字在网络上传送对象的时候。
当你想通过RMI传输对象的时候。

## 序列化需要注意
1、序列化时，只对对象的状态进行保存，而不管对象的方法；
2、当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；
3、当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；
4、并非所有的对象都可以序列化，至于为什么不可以，有很多原因了，比如：
安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行RMI传输等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的；
资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配，而且，也是没有必要这样实现；
5、声明为static和transient类型的成员数据不能被序列化。因为static代表类的状态，transient代表对象的临时数据。
6、序列化运行时使用一个称为 serialVersionUID 的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。为它赋予明确的值。显式地定义serialVersionUID有两种用途：
在某些场合，希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有相同的serialVersionUID；
在某些场合，不希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有不同的serialVersionUID。
7、Java有很多基础类已经实现了serializable接口，比如String,Vector等。但是也有一些没有实现serializable接口的；
8、如果一个对象的成员变量是一个对象，那么这个对象的数据成员也会被保存！这是能用序列化解决深拷贝的重要原因；


引用：
https://cloud.tencent.com/developer/article/1511793
https://cloud.tencent.com/developer/article/1420284