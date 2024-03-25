# ![abstract class 与 interface的区别](https://img-blog.csdnimg.cn/655478e2387345fb8310492c0757f840.png)

在Java语言中，abstract class和interface是支持抽象类定义的两种机制。正是由于这两种机制的存在，才赋予了Java强大的面向对象能力。abstract class和interface之间在对于抽象类定义的支持方面具有很大的相似性，甚至可以相互替换，因此很多开发者在进行抽象类定义时对于abstract class和interface的选择显得比较随意。其实，两者之间还是有很大的区别的，对于它们的选择甚至反映出对于问题领域本质的理解、对于设计意图的理解是否正确、合理。

1.相同点
  A. 两者都是抽象类，都不能实例化。
  B. interface实现类及abstrct class的子类都必须要实现已经声明的抽象方法。
2. 不同点
  A. interface需要实现，要用implements，而abstract class需要继承，要用extends。
  B. 一个类可以实现多个interface，但一个类只能继承一个abstract class。
  C. interface强调特定功能的实现，而abstract class强调所属关系。
  D. 尽管interface实现类及abstrct class的子类都必须要实现相应的抽象方法，但实现的形式不同。interface中的每一个方法都是抽象方法，都只是声明的 (declaration, 没有方法体)，实现类必须要实现。而abstract class的子类可以有选择地实现。
  这个选择有两点含义：
    一是Abastract class中并非所有的方法都是抽象的，只有那些冠有abstract的方法才是抽象的，子类必须实现。那些没有abstract的方法，在Abstrct class中必须定义方法体。
    二是abstract class的子类在继承它时，对非抽象方法既可以直接继承，也可以覆盖；而对抽象方法，可以选择实现，也可以通过再次声明其方法为抽象的方式，无需实现，留给其子类来实现，但此类必须也声明为抽象类。既是抽象类，当然也不能实例化。
  E. abstract class是interface与Class的中介。
  interface是完全抽象的，只能声明方法，而且只能声明pulic的方法，不能声明private及protected的方法，不能定义方法体，也 不能声明实例变量。然而，interface却可以声明常量变量，并且在JDK中不难找出这种例子。但将常量变量放在interface中违背了其作为接 口的作用而存在的宗旨，也混淆了interface与类的不同价值。如果的确需要，可以将其放在相应的abstract class或Class中。
  abstract class在interface及Class中起到了承上启下的作用。一方面，abstract class是抽象的，可以声明抽象方法，以规范子类必须实现的功能；另一方面，它又可以定义缺省的方法体，供子类直接使用或覆盖。另外，它还可以定义自己 的实例变量，以供子类通过继承来使用。
3. interface的应用场合
  A. 类与类之前需要特定的接口进行协调，而不在乎其如何实现。
  B. 作为能够实现特定功能的标识存在，也可以是什么接口方法都没有的纯粹标识。
  C. 需要将一组类视为单一的类，而调用者只通过接口来与这组类发生联系。
  D. 需要实现特定的多项功能，而这些功能之间可能完全没有任何联系。
4. abstract class的应用场合
  一句话，在既需要统一的接口，又需要实例变量或缺省的方法的情况下，就可以使用它。最常见的有：
  A. 定义了一组接口，但又不想强迫每个实现类都必须实现所有的接口。可以用abstract class定义一组方法体，甚至可以是空方法体，然后由子类选择自己所感兴趣的方法来覆盖。
  B. 某些场合下，只靠纯粹的接口不能满足类与类之间的协调，还必需类中表示状态的变量来区别不同的关系。abstract的中介作用可以很好地满足这一点。
  C. 规范了一组相互协调的方法，其中一些方法是共同的，与状态无关的，可以共享的，无需子类分别实现；而另一些方法却需要各个子类根据自己特定的状态来实现特定的功能。