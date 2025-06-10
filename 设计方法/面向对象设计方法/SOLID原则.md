# 概述

SOLID原则是下面七大原则中的前5个首字母拼写

1. 单一职责原则（Single Responsibility Principle）
2. 开闭原则（Open Close Principle）
3. 里氏替换原则（Liskov Substitution Principle）
4. 接口隔离原则（Interface Segregation Principle）
5. 依赖倒置原则（Dependence Inversion Principle）
6. 迪米特法则（Law Of Demeter）
7. 组合/聚合复用原则（Composite/Aggregate Reuse Principle CARP）



## 单一原则

单一原则表明，如果你有多个原因去改变一个类，那么应该把这些引起变化的原因分离开，把这个类分成多个类，**每个类只负责处理一种改变**。当你做出某种改变时，只需要修改负责处理该改变的类。当我们去改变一个具有多个职责的类时可能会影响该类的其他功能。

**即一个类不要包含很多功能，尽量仅完成单一功能**，**这样当你修改某个类时，不会影响其他功能，可以避免改错。**例如某个含有多个功能的类A，当你因为a1的功能，需要修改类A，因为a2的功能，也需要修改类A，就违反了单一原则，因为你在修该a1功能时，可能影响了a2功能，反之，如果修该a2功能时，可能影响了a1功能。由于在一个类中，可能修改了a1、a2都依赖的某个公共方法，就容易造成上述问题。

此外，一个方法只负责处理一项事情。

单一职责原则代表了设计应用程序时一种很好的识别类的方式，并且它提醒你思考一个类的所有演化方式。只有对应用程序的工作方式有了很好的理解，才能很好的分离职责。

## 里氏替换原则

只有满足以下2个条件的OO设计才可被认为是满足了LSP原则：

**（1）不应该在代码中出现if/else之类对派生类类型进行判断的条件。**

**（2）派生类应当可以替换基类并出现在基类能够出现的任何地方，或者说如果我们把代码中使用基类的地方用它的派生类所代替，代码还能正常工作。**

里氏替换原则通俗来讲就是：**子类可以扩展父类的功能，但不能改变父类原有的功能**。也就是说：子类继承父类时，除添加新的方法完成新增功能外，尽量不要重写父类的非抽象方法。

里式替换原则需要遵守以下事项：

子类可以扩展父类的功能，但不能改变父类原有的功能。

即子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。

子类中可以增加自己特有的方法。

当子类的方法重载父类的方法时，方法的前置条件（即方法的输入/入参）要比父类方法的输入参数更宽松。

当子类的方法实现父类的方法时（重载/重写或实现抽象方法）的后置条件（即方法的输出/返回值）要比父类更严格或相等。

里氏替换原则的优缺点：

优点：

代码共享，即公共代码被抽到父类，提高代码重用性。
子类在父类的基础上可以有自己的特性。提高代码的扩展性。
缺点：

侵入性。一旦继承，父类全部属性和方法都被子类拥有
约束性。子类需要拥有父类的属性和方法，子类多了一些约束。
耦合性。父类出现修改情况时，需要考虑子类的修改。
因此里氏替换原则并不是鼓励使用继承，而是当你不得已使用继承时，需要增加一些约束，避免出现不良影响。例如不能影响父类原有的功能

## 依赖倒转原则

依赖倒转原则是鼓励使用接口

什么是依赖？： 在程序设计中，如果一个模块a使用/调用了另一个模块b，我们称模块a依赖模块b。

高层模块与低层模块：往往在一个应用程序中，我们有一些低层次的类，这些类实现了一些基本的或初级的操作，我们称之为低层模块；另外有一些高层次的类，这些类封装了某些复杂的逻辑，并且依赖于低层次的类，这些类我们称之为高层模块。

依赖倒置（Dependency Inversion）：

面向对象程序设计相对于面向过程（结构化）程序设计而言，依赖关系被倒置了。因为传统的结构化程序设计中，高层模块总是依赖于低层模块。

抽象接口是对低层模块的抽象，低层模块继承或实现该抽象接口。

这样，高层模块不直接依赖低层模块，而是依赖抽象接口层。抽象接口也不依赖低层模块的实现细节，而是低层模块依赖（继承或实现）抽象接口。

类与类之间都通过抽象接口层来建立关系。

举个例子，AutoSystem类是高层类，引用了 HondaCar和FordCar，后者2个Car类都是低层次类，程序确实能够实现针对Ford和Honda车的无人驾驶：

```
public class HondaCar{
    public void Run(){
        Console.WriteLine("本田开始启动了");
    }
    public void Turn(){
        Console.WriteLine("本田开始转弯了");
    }
    public void Stop(){
        Console.WriteLine("本田开始停车了");
    }
}
public class FordCar{
    public void Run(){
        Console.WriteLine("福特开始启动了");
    }
    public void Turn(){
        Console.WriteLine("福特开始转弯了");
    }
    public void Stop(){
        Console.WriteLine("福特开始停车了");
    }
}

public class AutoSystem{
    public enum CarType{
        Ford,Honda
    };
    private HondaCar hcar=new HondaCar();
    private FordCar fcar=new FordCar();
    private CarType type;
    public AutoSystem(CarType type){
        this.type=type;
    }
    public void RunCar(){
        if(type==CarType.Ford){
            fcar.Run();
        } else {
            hcar.Run();
        }
    }
    public void TurnCar(){
        if(type==CarType.Ford){
            fcar.Turn();
        } else { 
            hcar.Turn();
        }
    }
    public void StopCar(){
        if(type==CarType.Ford){
            fcar.Stop();
            } else {
                hcar.Stop();
            }
    }
}

```

但是软件是在不断变化的，软件的需求也在不断的变化。

假设：公司的业务做大了，同时成为了通用、三菱、大众的金牌合作伙伴，于是公司要求该自动驾驶系统也能够安装在这3种公司生产的汽车上。于是我们不得不变动AutoSystem：

```java
   public class AutoSystem {
        public enum CarType {
            Ford, Honda, Bmw
        }

       
        HondaCar hcar = new HondaCar();   //使用new 
        FordCarf car = new FordCar();
        BmwCar bcar = new BmwCar();
        private CarType type;

        public AutoSystem(CarTypetype) {
            this.type = type;
        }

        public void RunCar() {
            if (type == CarType.Ford) {
                fcar.Run();
            } else if (type == CarType.Honda) {
                hcar.Run();
            } else if (type == CarType.Bmw) {
                bcar.Run();
            }
        }

        public void TurnCar() {
            if (type == CarType.Ford) {
                fcar.Turn();
            } else if (type == CarType.Honda) {
                hcar.Turn();
            } else if (type == CarType.Bmw) {
                bcar.Turn();
            }
        }

        public void StopCar() {
            if (type == CarType.Ford) {
                fcar.Stop();
            } else if (type == CarType.Honda) {
                hcar.Stop();
            } else if (type == CarType.Bmw) {
                bcar.Stop();
            }
        }
    }


```

分析：这会给系统增加新的相互依赖。随着时间的推移，越来越多的车种必须加入到AutoSystem中，这个“AutoSystem”模块将会被if/else语句弄得很乱，而且依赖于很多的低层模块，只要低层模块发生变动，AutoSystem就必须跟着变动

那么如何解决呢？采用接口形式，AutoSystem系统依赖于ICar 这个抽象，而与具体的实现细节HondaCar、FordCar、BmwCar无关，所以实现细节的变化不会影响AutoSystem。对于实现细节只要实现ICar 即可，即实现细节依赖于ICar 抽象。

```java
    public interface ICar
    {
        void Run();
        void Turn();
        void Stop();
    }
    public class BmwCar:ICar
    {
        public void Run()
        {
            Console.WriteLine("宝马开始启动了");
        }
        public void Turn()
        {
            Console.WriteLine("宝马开始转弯了");
        }
        public void Stop()
        {
            Console.WriteLine("宝马开始停车了");
        }
    }
    public class FordCar:ICar
    {
        publicvoidRun()
        {
            Console.WriteLine("福特开始启动了");
        }
        public void Turn()
        {
            Console.WriteLine("福特开始转弯了");
        }
        public void Stop()
        {
            Console.WriteLine("福特开始停车了");
        }
    }
    public class HondaCar:ICar
    {
        publicvoidRun()
        {
            Console.WriteLine("本田开始启动了");
        }
        public void Turn()
        {
            Console.WriteLine("本田开始转弯了");
        }
        public void Stop()
        {
            Console.WriteLine("本田开始停车了");
        }
    }
    public class AutoSystem
    {
        private ICar icar;
        public AutoSystem(ICar icar)    //使用构造函数作为入参，不再使用new创建具体的CaR实例
        {
            this.icar=icar;
        }
        private void RunCar()
        {
            icar.Run();
        }
        private void TurnCar()
        {
            icar.Turn();
        }
        private void StopCar()
        {
            icar.Stop();
        }
    }


```

应用该原则意味着**上层类不直接使用底层类，他们使用接口作为抽象层。**这种情况下上层类中创建底层类的对象的代码不能直接使用new 操作符，例如上面的示例，最终 AutoSystem使用构造函数传入一个Car的实例。可以使用一些创建型设计模式，例如工厂方法，抽象工厂和原型模式。模版设计模式是应用依赖倒转原则的一个例子。当然，使用该模式需要额外的努力和更复杂的代码，不过可以带来更灵活的设计。不应该随意使用该原则，如果我们有一个类的功能很有可能在将来不会改变，那么我们就不需要使用该原则。

模板模式，参见 【设计模式】策略模式与模板模式的区别，JDBCTemplate、RedisTemplate、MongoTemplate等均是典型的模板模式

## 接口分隔原则（Interface Segregation Principle ，ISP）

不能强迫用户去依赖那些他们不使用的接口。

接口的设计原则：**接口的设计应该遵循最小接口原则，不要把用户不使用的方法塞进同一个接口里。如果一个接口的方法没有被使用到，则说明该接口过胖，应该将其分割成几个功能专一的接口。(一个interface拆分成多个interface)**

接口隔离原则表明客户端不应该被强迫实现一些他们不会使用的接口，应该把肥胖接口中的方法分组，然后用多个接口代替它，每个接口服务于一个子模块。

如果已经设计成了胖接口，可以使用适配器模式隔离它。像其他设计原则一样，接口隔离原则需要额外的时间和努力，并且会增加代码的复杂性，但是可以产生更灵活的设计。如果我们过度的使用它将会产生大量的包含单一方法的接口，所以需要根据经验并且识别出那些将来需要扩展的代码来使用它。

接口分隔原则总的来说是鼓励使用接口的，只是进行了一定的约束，便于更好的发挥接口的作用！

举个例子，在swing组件事件监听器，存在很多接口，当你想实现某个事件时，必须实现所有的接口，并且可以使用WindowAdapter适配器避免这个情况：

事件总接口：

```java
 public interface EventListener {
 }

```

WindowListener 扩展了这个接口，存在抽象方法过多的问题：

```java
public interface WindowListener extends EventListener {
    /**
     * Invoked the first time a window is made visible.
     */
    public void windowOpened(WindowEvent e);

    public void windowClosing(WindowEvent e);

    public void windowClosed(WindowEvent e);

    public void windowIconified(WindowEvent e);

    public void windowDeiconified(WindowEvent e);

    public void windowActivated(WindowEvent e);

    public void windowDeactivated(WindowEvent e);
}

```

WindowAdapter是一个抽象类.但是这个抽象类里面却`没有抽象方法`：

```java
public abstract class WindowAdapter
    implements WindowListener, WindowStateListener, WindowFocusListener
{
    /**
     * Invoked when a window has been opened.
     */
    public void windowOpened(WindowEvent e) {}

    public void windowClosing(WindowEvent e) {}

    public void windowClosed(WindowEvent e) {}

    public void windowIconified(WindowEvent e) {}

    public void windowDeiconified(WindowEvent e) {}

    public void windowActivated(WindowEvent e) {}

    public void windowDeactivated(WindowEvent e) {}

    public void windowStateChanged(WindowEvent e) {}

    public void windowGainedFocus(WindowEvent e) {}

    /**
     * Invoked when the Window is no longer the focused Window, which means
     * that keyboard events will no longer be delivered to the Window or any of
     * its subcomponents.
     *
     * @since 1.4
     */
    public void windowLostFocus(WindowEvent e) {}
}

```

## 迪米特法则 (Law of Demeter)

定义：**如果两个软件实体无须直接通信，那么就不应当发生直接的相互调用，可以通过第三方转发该调用。其目的是降低类之间的耦合度，提高模块的相对独立性。**

迪米特法则（Law of Demeter）又叫作最少知识原则（The Least Knowledge Principle），通俗的来讲，就是一个类对自己依赖的类知道的越少越好。也就是说，对于被依赖的类来说，无论逻辑多么复杂，都尽量地的将逻辑封装在类的内部，对外除了提供的public方法，不对外泄漏任何信息。

迪米特法则还有一个更简单的定义：只与直接的朋友通信。首先来解释一下什么是直接的朋友：我们称出现成员变量、方法参数、方法返回值中的类为直接的朋友，而出现在局部变量中的类则不是直接的朋友。也就是说，陌生的类最好不要作为局部变量的形式出现在类的内部。

迪米特法则的目的在于降低类之间的耦合。由于每个类尽量减少对其他类的依赖，因此，很容易使得系统的功能模块相互独立，相互之间不存在依赖关系。

应用迪米特法则有可能造成的一个后果就是，系统中存在的大量的中介类，这些类只所以存在完全是为了传递类之间的相互调用关系—这在一定程度上增加系统的复杂度。

设计模式中的门面模式（Facade）和中介模式（Mediator）都是迪米特法则的应用的例子。

狭义的迪米特法则的缺点：

遵循类之间的迪米特法则会使一个系统的局部设计简化，因为每一个局部都不会和远距离的对象有之间的关联。但是，这也会造成系统的不同模块之间的通信效率降低，也会使系统的不同模块之间不容易协调。
广义的迪米特法则在类的设计上的体现：

尽量降低一个类的访问权限。
尽量降低成员的访问权限。
例子
公司要打印某部门的人员信息，代码如下，反例:

```java
/**
 * 雇员
 */
public class Employee {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
/**
 * 部门经理
 */
public class Manager {
    public List<Employee> getEmployees(String department) {
        List<Employee> employees = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Employee employee = new Employee();
            // 雇员姓名
            employee.setName(department + i);
            employees.add(employee);
        }
        return employees;
    }
}
/**
 * 公司
 */
public class Company {
	private Manager manager = new Manager();

    public void printEmployee(String name){
        List<Employee> employees = manager.getEmployees(name);
        for (Employee employee : employees) {
            System.out.print(employee.getName() + ";");
        }
    }
}

```

Company 类中的 printEmployee 确实成功打印了人员信息，但是 Employee 类只作为局部变量出现在 printEmployee() 方法中，为 Company 类的间接朋友，违背了迪米特法则（只与直接的朋友通信）

正确做法如下： 将 Company 类中打印雇员信息的方法放在 Manager 类中，Company 中只调用 Manager 中 printEmployee() 方法即可，Employee 类不再 Company 类中出现

```java
/**
 * 雇员
 */
public class Employee {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
/**
 * 部门经理
 */
public class Manager {
    public List<Employee> getEmployees(String department) {
        List<Employee> employees = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Employee employee = new Employee();
            // 雇员姓名
            employee.setName(department + i);
            employees.add(employee);
        }
        return employees;
    }

    public void printEmployee(String name){
        List<Employee> employees = this.getEmployees(name);
        for (Employee employee : employees) {
            System.out.print(employee.getName() + ";");
        }
    }
}
/**
 * 公司
 */
public class Company {
    private Manager manager = new Manager();

    public void printEmployee(String name){
        manager.printEmployee(name);
    }
}


```

甚至你可以通过把Employee类的权限设置为仅对Manager 可见，对Company 不可见

## 组合/聚合复用原则

聚合表示整体与部分的关系，表示“含有”，整体由部分组合而成，部分可以脱离整体作为一个独立的个体存在。
组合则是一种更强的聚合，部分组成整体，而且不可分割，部分不能脱离整体而单独存在。在合成关系中，部分和整体的生命周期一样，组合的新的对象完全支配其组成部分，包括他们的创建和销毁。

组合/聚合和继承是实现复用的两个基本途径。合成复用原则是指尽量使用组合/聚合，而不是使用继承。只有当以下的条件全部被满足时，才应当使用继承关系：

子类是超类的一个特殊种类，而不是超类的一个角色，也就是区分“Has-A”和“Is-A”.只有“Is-A”关系才符合继承关系，“Has-A”关系应当使用聚合来描述。
永远不会出现需要将子类换成另外一个类的子类的情况。如果不能肯定将来是否会变成另外一个子类的话，就不要使用继承。
子类具有扩展超类的责任，而不是具有置换掉或注销掉超类的责任。如果一个子类需要大量的置换掉超类的行为，那么这个类就不应该是这个超类的子类。
组合和聚合的区别

## 开闭原则

开闭原则是其他六大原则的总章，我们故意放在最后讲，也就说当你的代码满足前面6个原则时，那么基本上就是满足开闭原则的！

**对扩展开放------- 模块的行为可以被扩展从而满足新的需求。**
**对修改关闭-------不允许修改模块的源代码（或者尽量使修改最小化）**
开闭原则是说我们应该努力设计不需要修改的模块。在实际应用将变化的代码和不需要变化的代码进行隔离，将变化的代码抽象成稳定接口，针对接口进行编程。在扩展系统的行为时，我们只需要添加新的代码，而不需要修改已有的代码。一般可以通过添加新的子类和重写父类的方法来实现。

开闭原则是面向对象设计的核心，满足该原则可以达到最大限度的复用性和可维护性。
