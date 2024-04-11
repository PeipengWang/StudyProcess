![在这里插入图片描述](https://img-blog.csdnimg.cn/20200328222020710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
客户端请求页面，服务器根据web请求产生动态内容，其内部启动多个线程产生不同内容，这种请求相应是交互式的，都是基于HTTP协议的。
**涉及内容：**
html+servlet+http+反射
**一、反射**
两个层面去理解：
一方面，原来我们new一个对象，需要在虚拟机中准备，现在我们在使用反射让我们这个使用者在构造一个对象了。
另一个层面，可以通过反射把java中各个结构（属性，方法，构造器等）映射成一个个的对象，可以分别拿来用。
利用反射可以对一个类进行解剖，是框架的灵魂。
	获取类的3种方法：
	

```c
public class reflectTest {
    public static void main(String[] args) throws ClassNotFoundException {
        //1,第一种方式:对象.getClass()
        Ipone iphone = new Ipone("iphone7");
        Class clz = iphone.getClass();
        //2,第二种方式:类.class
        clz = Ipone.class;
        //3,第三种：Class.forName("包名.类名");
        Class clz2 = Class.forName("com.company.test.");

    }

}
class Ipone{
    private String name;

    public Ipone(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
在这里一般推荐使用第三种方式。
利用反射创建对象。

```c
  clz = Class.forName("com.test.Ipone");
        Ipone iphone2 = (Ipone) clz.getConstructor().newInstance();
        System.out.println(iphone2);
```
**二：XML解析**
XML(Extensible Markup Language))可扩展性标记语言，作为数据的一种存储格式或者用于存储软件的参数，程序解析配置此文件可以达到不修改代码就修改程序的目的。
我们需要对XML文件进行解析，这里使用SAX解析工厂。
首先编写一个简单的XML文件`

```c
<?xml version="1.0" encoding="UTF-8" ?>
<persons>
    <person>
        <name>至尊宝</name>
        <age>9000</age>
    </person>
    <person>
        <name>紫霞</name>
        <age>10000</age>
    </person>
</persons>
```
 	测试文件：
 	

```c
package com.test;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;
import java.io.IOException;

public class XmlTest01 {
    public static void main(String[] args) throws ParserConfigurationException, SAXException, IOException {
        //1,获取解析工厂
        SAXParserFactory factory = SAXParserFactory.newInstance();
        //2，从解析工厂获得解析器
        SAXParser parse = factory.newSAXParser();
        //3，编写处理器
        //4，加载文档Document注册处理器
        PHandler phandler = new PHandler();
        //5，解析
        parse.parse(Thread.currentThread().getContextClassLoader().
                getResourceAsStream("com/test/persons.xml"),phandler);

    }
}
class PHandler extends DefaultHandler{
    @Override
    public void startDocument() throws SAXException {
        System.out.println("解析文档开始");
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        System.out.println(qName+"-->解析开始");
    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        String contexs = new String(ch,start,length).trim();
        if(contexs.length() == 0){
            System.out.println("内容为：空");
        }else{
            System.out.println("内容为："+contexs);
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        System.out.println(qName+"解析结束");
    }

    @Override
    public void endDocument() throws SAXException {
        System.out.println("解析文档结束");
    }

    public PHandler() {
    }
}

```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200330221323914.png)
遇到问题1：在对于有多个属性的XML的值，需要在对应的类里面设置集合，并且需要对集合进行初始化，简而言之，集合必须初始化。
我们可以发现，每个标签都需要开始读取，然后读取内容，直到遇到结束标签，在结束标签后继续读取内容，按照这个过程设计webxml。

```c
<?xml version="1.0" encoding="UTF-8" ?>
<web-app>
    <sevlet>
        <sevlet-name>login</sevlet-name>
            <sevlet-class>com.test.webxmltest.basicSevlet.LoginSevlet</sevlet-class>
    </sevlet>
    <sevlet>
        <sevlet-name>reg</sevlet-name>
        <sevlet-class>com.test.webxmltest.basicSevlet.RegeisterSevlet</sevlet-class>
    </sevlet>
    <sevlet-mapping>
        <sevlet-name>login</sevlet-name>
        <url-pattern>/login</url-pattern>
        <url-pattern>/g</url-pattern>
    </sevlet-mapping>
    <sevlet-mapping>
        <sevlet-name>reg</sevlet-name>
        <url-pattern>/reg</url-pattern>
    </sevlet-mapping>
</web-app>
```
设计目的：课题通过/reg读取到RegeisterSevlet这个实体类，也可以通过/g或者/login读取到LoginSevlet这个实体类。
为了初步对于XML文件的数据进行读取，我们分别对于sevlet和sevlet-mapping进行了分别的存储，在这里需要设计两个工具类，由xml文件可知，sevlet是一对一的关系，可以直接设计为

```c
public class Entity {
    private String name;
    private String clz;

    public Entity() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getClz() {
        return clz;
    }

    public void setClz(String clz) {
        this.clz = clz;
    }
}
```
但是对于sevlet-mapping来说是一对多关系，需要设计如下,与sevlet的标签读取不同的是，String类变为了Set集合类，同时需要增加一个add方法，方便每一个对同一个对象增加多个pattern。

```c
import java.util.HashSet;
import java.util.Set;

public class Mapping {
    private String name;
    private Set<String> patterns;

    public Mapping() {
        patterns = new HashSet<>();
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Set<String> getPatterns() {
        return patterns;
    }
    public void addPattern(String pattern){
        this.patterns.add(pattern);
    }
}

```

现在设计RegisterSevlet和LoginSevlet这两个实体类，在设计之前先定义Sevlet的公共接口：

```c
public interface Sevlet {
    public void service();
}
```
设计实体类

```c
public class LoginSevlet implements Sevlet{
    @Override
    public void service() {
        System.out.println("LoginSevlet");
    }
}
```

```c
public class RegeisterSevlet implements Sevlet{

    @Override
    public void service() {
        System.out.println("RegisterSevlet");
    }
}
```
通过以上的分析，我们发现，直接进行XML文件解析出的数据时一对多的关系，如

```c
  <sevlet-mapping>
        <sevlet-name>login</sevlet-name>
        <url-pattern>/login</url-pattern>
        <url-pattern>/g</url-pattern>
    </sevlet-mapping>
```
这段代码login是对应/login和/g的，为了变成一对一的关系，并且有利于查找，需要进行转化，分析可知转化为HashMap结构是一个比较好的方式，转化代码如下所示，

```c
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class WebMap {
    private List<Entity> entities = null;
    private List<Mapping> mappings = null;
    Map<String,String> entitiesMap = new HashMap<>();
    Map<String,String> mappingMap = new HashMap<>();


    public WebMap(List<Entity> entities, List<Mapping> mappings) {
        this.entities = entities;
        this.mappings = mappings;
        for(Entity entity:entities){
            entitiesMap.put(entity.getName(),entity.getClz());
        }
        for(Mapping mapping:mappings){
            for(String pattern:mapping.getPatterns()){
                mappingMap.put(pattern,mapping.getName());
            }
        }
    }
    public String getClz(String pattern){
        String name = mappingMap.get(pattern);
        return entitiesMap.get(name);
    }

}
```
这样，将XML文件进行解析存储，并且进一步转化为哈希索引读取方式，进行反射来构造我们需要的类，测试代码如下所示：

```c
import com.test.webxmltest.Entity;
import com.test.webxmltest.Mapping;
import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;
import java.util.ArrayList;
import java.util.List;

public class XMLTest03 {
    public static void main(String[] args) throws  Exception {
        //1,获取解析工厂
        SAXParserFactory factory = SAXParserFactory.newInstance();
        //2，从解析工厂获得解析器
        SAXParser parse = factory.newSAXParser();
        //3，编写处理器
        //4，加载文档Document注册处理器
        WebHandler webhandler = new WebHandler();
        //5，解析
        parse.parse(Thread.currentThread().getContextClassLoader().
                getResourceAsStream("com/test/webxmltest/basicSevlet/web.xml"),webhandler);

        List<Entity> entities = webhandler.getEntities();
        List<Mapping> mappings = webhandler.getMappings();
        WebMap webmap = new WebMap(entities,mappings);
        String name = webmap.getClz("/g");
        Class clz = Class.forName(name);
        Sevlet sevlet = (Sevlet) clz.getConstructor().newInstance();
        System.out.println(sevlet);
        sevlet.service();
    }
}
class WebHandler extends DefaultHandler {
    private List<Entity> entities = null;
    private List<Mapping> mappings = null;
    boolean isMapping = false;
    private Entity entity = null;
    private Mapping mapping = null;
    private String tag = null;
    @Override
    public void startDocument() throws SAXException {
        System.out.println("解析文档开始");
        entities = new ArrayList<>();
        mappings = new ArrayList<>();
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        System.out.println(qName+"-->解析开始");
        if(null != qName){
            tag = qName;
            if(tag.equals("sevlet")){
                entity = new Entity();
                isMapping = false;
            }else if(tag.equals("sevlet-mapping")){
                mapping = new Mapping();
                isMapping = true;
            }
        }
    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        String contexs = new String(ch,start,length).trim();
        if(null != tag){
            if(!isMapping){
                if(tag.equals("sevlet-name")){
                    entity.setName(contexs);
                }else if(tag.equals("sevlet-class")){
                    entity.setClz(contexs);
                }
            }else{
                if(tag.equals("sevlet-name")){
                    mapping.setName(contexs);
                }else if(tag.equals("url-pattern")){
                    mapping.addPattern(contexs);
                }
            }
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        System.out.println(qName+"解析结束");
        if(null != qName){
            if(qName.equals("sevlet")){
                entities.add(entity);
            }else if(qName.equals("sevlet-mapping")){
                mappings.add(mapping);
            }
        }
        tag = null;
    }
    @Override
    public void endDocument() throws SAXException {
        System.out.println("解析文档结束");
    }
    public WebHandler() {
    }
    public List<Entity> getEntities() {
        return entities;
    }
    public List<Mapping> getMappings() {
        return mappings;
    }
}

```
输出结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401170949810.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
主要是最后一行，可以发现直接通过反射来构造类并且调用了其中的方法。
**三、html基本知识**
HyperText Markup Language 超文本标记语言。
httml是骨架
Css可以美化
JavaScript是动态语言，可以让网页与人交互
<html>-->开始标签
<head>-->网页上的控制信息
<title>-->页面标题
</title></head>
<body>页面上要显示的内容</body>
</html>-->结束标签
常用标签
h1~h6
p
div
span
form
input

```c
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登陆</title>
</head>
<body>
<form method="post" action="http://localhost:8888/index.html">
    用户名：<input type="text" name="uname" id="uname"/><br/>
    密码：<input type="password" name="pwd" id="pwd"/><br/>
    <input type="submit" value="登陆">
</form>

</body>
</html>
```
主要对上面代码进行解析。
method:方法主要有两个
一是get，这是一个默认方法，是要从服务器获取数据的方法，基于http协议的不同，当获取的数据量比较少的时候使用，它的请求参数的url是可见的，因此是不安全的。
二是post，这是提交方法，在数据量比较小的时候用，它是基于http协议不同，请求参数的url是不可见的，因此是安全的。
action：请求web服务器资源用的。
name：作为后端使用，区分唯一，请求服务器，必须存在，数据不能提交。
id：作为前端使用，区分唯一。
用浏览器打开为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401184559618.png)
**四、http协议**
Hyper Text Transfer Protocol，超文本传输协议，是互联网上应用最为广泛的网络协议，所有www开头的文件都必须遵守这个协议。
1，请求协议
（1）请求行：方法（GET，POST）、URL、协议/版本
（2）请求头：（Request Header）
（3）请求正文
获取请求协议，使用ServerSocket建立与浏览器的连接，获取请求协议
写请求代码：

```c
Socket client = serverSocket.accept();
            System.out.println("一个客户端建立了连接");
            //获取请求协议
            InputStream is = client.getInputStream();
            byte[] datas = new byte[1024*1024];
            int len = is.read(datas);
            String informa = new String(datas,0,len);
            System.out.println(informa);
```
这个时候用firefox自带的RESTer发现，可以从服务端获得请求信息：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401233529981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
但是客户请求段会出现问题
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401232318988.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
为了更好的提取请求信息，专门为请求设置一个请求类，

```c
import java.io.IOException;
import java.io.InputStream;
import java.net.Socket;

public class Request {
    private InputStream is;
    private String requestinfo;
    private String method;
    private String url;
    private String protocal;
    private String queryPrasm = null;
    public Request() {
    }

    public Request(InputStream is) {
        this.is = is;
        try {
            byte[] bytes = new byte[1024*1024];
            int len = 0;
            len = is.read(bytes);
            requestinfo = new String(bytes,0,len);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public Request(Socket client) {
        try {
            byte[] bytes = new byte[1024*1024];
            int len = 0;
            is = client.getInputStream();
            len = is.read(bytes);
            requestinfo = new String(bytes,0,len);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public void displayRequest(){
        System.out.println("--分解--");
        this.method = this.requestinfo.substring(0,requestinfo.indexOf("/")).toLowerCase().trim();
         //这里注意要去掉空格，因为在/前面室友一个空格的，如果不去掉的话跟下面是没法对比的
        int start = requestinfo.indexOf("/")+1;
        int end = requestinfo.indexOf("HTTP/");
        this.url = this.requestinfo.substring(start,end);
        int queryposi = url.indexOf("?");
        if(queryposi > 0){
            String[] arr = url.split("\\?");
            url = arr[0];
            queryPrasm = arr[1];
        }
        if(this.method.equals("post")){
            String qstr = this.requestinfo.substring(this.requestinfo.lastIndexOf("\r\n")).trim();
            if(null == queryPrasm){
                queryPrasm = qstr;
            }else{
                queryPrasm += "&"+qstr;
            }
        }
        start = end;
        end = requestinfo.indexOf("\r\n");
        this.protocal = requestinfo.substring(start,end);
        System.out.println(method+" "+url+" "+queryPrasm+" "+protocal);
    }
}

```
问题所在，在进行post方法处理时，`if(this.method.equals("post"){}`判断时，一直未false，后经过查找发现在上面的`this.requestinfo.substring(0,requestinfo.indexOf("/")).toLowerCase().trim();`语句执行时没有加上trim()去掉空格的命令，导致在/前面实际上是有个空格的，因此一直无法进行判定。

主要原因是没有进行进行相应。
2，响应协议
（1）响应行：协议/版本、状态码、状态描述
状态码：
-1xx：表示信息请求已接受，正在处理中。
-2xx：成功，表示请求已经被成功接收、理解和接受。如200表示OK，客户端请求成功
-3xx：重定向，要完成请求必须进一步操作；
-4xx：客户端错误，请求有语法错误或者无法实现，如404，not found，请求的资源不存在，例如输入了错误的URL。
-5xx：服务端错误，服务器未能实现合法的请求。
写返回的代码：

```c

            StringBuilder contexs = new StringBuilder();
            contexs.append("<html>");
            contexs.append("<head>");
            contexs.append("<title>");
            contexs.append("服务器响应成功");
            contexs.append("</title>");
            contexs.append("</head>");
            contexs.append("<body>");
            contexs.append("服务器回来了");
            contexs.append("</body>");
            contexs.append("</html>");
            int size = contexs.toString().getBytes().length;
            StringBuilder responseinfo = new StringBuilder();
            String blank =" ";
            String CRLF = "\r\n";
            //响应行：http/1.1 200 OK
            responseinfo.append("http/1.0").append(blank);
            responseinfo.append("200").append(blank);
            responseinfo.append("OK").append(CRLF);
            //响应头：（最后一行存在空行）
            responseinfo.append("Date:").append(new Date()).append(CRLF);
            responseinfo.append("Server").append("shsxt Server/0.0.1;charset=GBK ").append(CRLF);
            responseinfo.append("Content-type:text/html").append(CRLF);
            responseinfo.append("Content-length:").append(size).append(CRLF);
            //空行
            responseinfo.append(CRLF);
            //正文
            responseinfo.append(contexs.toString());
            BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(client.getOutputStream()));
            bw.write(responseinfo.toString());
            bw.flush();
```
一定要严格遵守上面的返回协议命令，不然会出现很多问题，例如，开始的时候在`responseinfo.append("Content-length:").append(size).append(CRLF);`这句代码的“：”没有写导致虽然相应没有出错，但是不是出现乱码就是根本没有返回值。
另一个易错点是必须大小必须是字节数的长度，而不是字符长度，字节数长度的正确算法是，`int size = contexs.toString().getBytes().length;`
最后注意换行和空格，不然会出错。
尤其是在请求头到正文之间有个空格需要特别注意，没有连接网络就会出错，而不是不返回信息了。


（2）响应头：（Respond Header）
（3）响应正文
属于应用层协议，它的底层是TCP/IP协议。
3，处理请求参数
封装请求参数为Map

```c
 private void covertMap(String queryPrasm){
        //先用&进行分割
        String[] str1 = queryPrasm.split("&");
        for(String s:str1) {
            //再次分割
            String[] fv = s.split("=");
            //为了防止有些数据只有一个数据，通过如下设置保证fv有且只有两个数据
            fv = Arrays.copyOf(fv,2);
            //获取key和value
            String key = fv[0];
            String value = fv[1] == null? null:decode(fv[1],"utf-8");
            //存储到map中
            if(!parameterMap.containsKey(key)){
                parameterMap.put(key,new ArrayList<>());
            }
            parameterMap.get(key).add(value);
        }
    }
```

4，收发器的处理

```c
import java.io.FileInputStream;
import java.io.InputStream;
import java.net.Socket;

public class Dispatcher implements Runnable {
    private Socket client;
    private Request request;
    private Response response;
    public Dispatcher(Socket client) {
        this.client = client;
        this.request = new Request(this.client);
        this.response = new Response(client);
    }

    @Override
    public void run() {
        try{
            //获取请求协议
            request.dcomposeRequest();
            WebApp webapp = new WebApp();
            if(request.getUrl().equals("")){
                InputStream is = new FileInputStream("D:" +
                        "\\WebServer\\src\\com\\test\\webxmltest\\Server03\\index.html");
                byte[] datas = new byte[1024*1024];
                int len = is.read(datas);
                response.print(new String(datas,0,len));
                response.pushInfo(200);
                is.close();
                return;
            }
            Sevlet sevlet = webapp.getSevletFormUrl(request.getUrl());
            if(null != sevlet){
                sevlet.service(request,response);
                response.pushInfo(200);
            }else{
                InputStream is = new FileInputStream("D:" +
                        "\\WebServer\\src\\com\\test\\webxmltest\\Server03\\error.html");
                byte[] datas = new byte[1024*1024];
                int len = is.read(datas);
                response.print(new String(datas,0,len));
                response.pushInfo(404);
                is.close();
            }
            client.close();
    } catch (Exception e) {
        System.out.println("客户端错误");
        }
    }
}

```

在线程上有个get不可以，用的时候会出现问题，因此需要用FileInputStream，但是需要准确的路径。





```

```
