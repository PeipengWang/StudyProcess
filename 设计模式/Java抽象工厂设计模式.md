# 抽象工厂设计模式
抽象工厂模式（Abstract Factory Pattern）是围绕一个超级工厂创建其他工厂。**该超级工厂又称为其他工厂的工厂**。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。
抽象工厂模式提供了一种创建一系列相关或相互依赖对象的接口，而无需指定具体实现类。通过使用抽象工厂模式，可以将客户端与具体产品的创建过程解耦，使得客户端可以通过工厂接口来创建一族产品。

## 抽象工厂模式包含以下几个核心角色：
抽象工厂（Abstract Factory）：声明了一组用于创建产品对象的方法，每个方法对应一种产品类型。抽象工厂可以是接口或抽象类。
具体工厂（Concrete Factory）：实现了抽象工厂接口，负责创建具体产品对象的实例。
抽象产品（Abstract Product）：定义了一组产品对象的共同接口或抽象类，描述了产品对象的公共方法。
具体产品（Concrete Product）：实现了抽象产品接口，定义了具体产品的特定行为和属性。
(https://img-blog.csdnimg.cn/direct/e6b2f97b0074400bb3f5a2a70dc4d00f.png)



使用步骤
创建抽象工厂类，定义具体工厂的公共接口；   创建button、Textbox接口
创建抽象产品族类 ，定义抽象产品的公共接口；  GUIFactory
创建抽象产品类 （继承抽象产品族类），定义具体产品的公共接口；WindowsButton、MacButton、WindowsTextbox、MacTextbox
创建具体产品类（继承抽象产品类） & 定义生产的具体产品；WindowsFactory，MacFactory
创建具体工厂类（继承抽象工厂类），定义创建对应具体产品实例的方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/78a03cf8dc034817986f4a2e234a3483.png)

```
package AbstractFactory;
// 抽象产品接口
interface Button {
    void paint();
}

// 具体产品：Windows风格按钮
class WindowsButton implements Button {
    @Override
    public void paint() {
        System.out.println("Rendering a Windows style button.");
    }
}

// 具体产品：Mac风格按钮
class MacButton implements Button {
    @Override
    public void paint() {
        System.out.println("Rendering a Mac style button.");
    }
}

// 抽象产品接口
interface Textbox {
    void paint();
}

// 具体产品：Windows风格文本框
class WindowsTextbox implements Textbox {
    @Override
    public void paint() {
        System.out.println("Rendering a Windows style textbox.");
    }
}

// 具体产品：Mac风格文本框
class MacTextbox implements Textbox {
    @Override
    public void paint() {
        System.out.println("Rendering a Mac style textbox.");
    }
}

// 抽象工厂接口
interface GUIFactory {
    Button createButton();
    Textbox createTextbox();
}

// 具体工厂：Windows风格工厂
class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    @Override
    public Textbox createTextbox() {
        return new WindowsTextbox();
    }
}

// 具体工厂：Mac风格工厂
class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }

    @Override
    public Textbox createTextbox() {
        return new MacTextbox();
    }
}

// 客户端代码
public class Client {
    public static void main(String[] args) {
        // 选择要使用的工厂
        // 或者 new MacFactory();
        GUIFactory factory = new WindowsFactory();

        // 创建按钮和文本框
        Button button = factory.createButton();
        Textbox textbox = factory.createTextbox();

        // 使用创建的按钮和文本框
        button.paint();
        textbox.paint();

        GUIFactory factory1 = new MacFactory();

        button = factory1.createButton();
        textbox = factory1.createTextbox();

        button.paint();
        textbox.paint();
    }
}
```