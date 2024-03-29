
# Java的SPI机制与实例
## 是什么？
SPI是一种JDK内置的服务提供发现的机制，能够启动框架扩展和替换组件，主要是被框架的开发人员使用，比如java.sql。Driver接口。Java机制的核心思想就是将装配的控制权转移到Java之外，核心思想是解耦。
## 怎么用？
当服务提供一个接口服务之后，加载需要再classpath的META-INF/services目录创建一个以服务接口全限定性类名的文件，在文件中配置具体的服务实现。
JDK中查找服务的实现的工具类是：java.util.ServiceLoader。通过这个接口加载具体的实现类，执行相应的方法。
## 实例
1、首先定义一个接口  
```
package SPI;

import java.util.List;

public interface Search {
    public List<String> searchDoc(String keyword);
}
```
2、实现接口，这个就是具体的服务  
```
package SPI;

import java.util.List;

public class FileSearch implements Search{
    @Override
    public List<String> searchDoc(String keyword) {
        System.out.println("文件搜索 "+keyword);
        return null;
    }
}
```
```
package SPI;

import java.util.List;

public class DatabaseSearch implements Search{
    @Override
    public List<String> searchDoc(String keyword) {
        System.out.println("数据搜索 "+keyword);
        return null;
    }
}

```
3、在META-INF/service中定义这个接口，注意文件时无格式的，名称需要为接口的全限定性名称  
文件名称：SPI.Search  
文件内容  
SPI.FileSearch  
SPI.DatabaseSearch  
4、测试代码  
```
package SPI;

import java.util.Iterator;
import java.util.ServiceLoader;

public class TestCase {
    public static void main(String[] args) {
        ServiceLoader<Search> s = ServiceLoader.load(Search.class);
        Iterator<Search> iterator = s.iterator();
        while (iterator.hasNext()) {
           Search search =  iterator.next();
           search.searchDoc("hello world");
        }
    }
}

```
结果：  
文件搜索 hello world  
数据搜索 hello world  

Process finished with exit code 0  
至此就实现了一个SPI的调用  

## 缺点
通过上面的解析，可以发现，我们使用SPI机制的缺陷：不能按需加载，需要遍历所有的实现，并实例化，然后在循环中才能找到我们需要的实现。  
如果不想用某些实现类，或者某些类实例化很耗时，它也被载入并实例化了，这就造成了浪费。  
获取某个实现类的方式不够灵活，只能通过 Iterator 形式获取，不能根据某个参数来获取对应的实现类。多个并发多线程使用   ServiceLoader 类的实例是不安全的。  

但是这些问题在dubbo得到了解决，需要看下这块代码  

