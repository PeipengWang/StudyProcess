# static加载顺序
1、先执行父类的静态代码块和静态变量初始化，并且静态代码块和静态变量的执行顺序只跟代码中出现的顺序有关。   
2、执行子类的静态代码块和静态变量初始化。    
3、执行父类的实例变量初始化    
4、执行父类的构造函数    
5、执行子类的实例变量初始化    
6、执行子类的构造函数    
# final
final方法：表示方法不可被子类重写（覆盖），类的private方法会隐式地被指定为final方法。  
final变量：初始化一次后值不可变   
final类：类不能被继承，内部的方法和变量都变成final类型  
final对象：指对象的引用不可变，但是对象的值可变，即指向的是同一个对象，但是对象内部的值可以修改。  
