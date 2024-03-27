# BufferReader
Java IO操作——BufferedReader(缓冲区读取内容，避免中文乱码)  
要点：  
掌握BufferedReader类的使用  
掌握键盘输入的基本形式  

Buffer：表示缓冲区的。之前的StringBuffer，缓冲区中的内容可以更改，可以提高效率。  
如果想接收任意长度的数据，而且避免乱码的产生，就可以使用BufferedReader。  
public class BufferedReader extends Reader  
因为输入的数据有可能出现中文，所以此处使用字符流完成  
BufferedReader是从缓冲区之中读取内容，所有的输入的字节数据都将放在缓冲区之中。  
1、public BufferedReader(Reader in) 构造方法 接收一个Reader类的实例  
2、public String readLine() throws IOException 一次性从缓冲区中将内容全部读取进来。  
System.in本身表示的是InputStream（字节流），现在要求接收的是一个字符流，需要将字节流转为字符流才可以，InputStreamReader。  
BufferedReader接收键盘输入时实例化如下：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/3d6fd3f2837844a48bafba1795a24ff0.png)

```
import java.io.* ;
public class BufferedReaderDemo01{
     public static void main(String args[]){
        BufferedReader buf = null ;   // 声明对象
        buf = new BufferedReader(new InputStreamReader(System.in)) ;  // 将字节流变为字符流
        String str = null ; // 接收输入内容
        System.out.print("请输入内容：") ;
        try{
            str = buf.readLine() ;  // 读取一行数据
        }catch(IOException e){
           e.printStackTrace() ;  // 输出信息
        }
        System.out.println("输入的内容为：" + str) ;
    }
};

```

读取文件内容  
```
import java.io.* ;
public class BufferedReaderDemo01{
     public static void main(String args[])throws IOException{
        BufferedReader buf = null ;   // 声明对象
        buf = new BufferedReader(new FileReader(new File("D:"+File.separator+"test.txt"))) ;  // 将字节流变为字符流
        String str1 = null ;  // 接收输入内容
        String str2 = null ;  // 接收输入内容
        try{
           str1 = buf.readLine() ;  // 读取一行数据
           str2 = buf.readLine() ;  // 读取一行数据
        }catch(IOException e){
           e.printStackTrace() ;    // 输出信息
        }
        System.out.println("读取第一行的内容为：" + str1) ;
        System.out.println("读取第二行的内容为：" + str2) ;
      }
};

```

