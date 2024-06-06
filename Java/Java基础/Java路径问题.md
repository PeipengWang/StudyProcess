# Java路径  
## Java获取项目根目录  
```
System.getProperty("user.dir")  
```
开发环境时，user.dir 指向的是项目的根目录    
windows，将项目部署到tomcat下，user.dir指向的路径是当前用户的桌面    
linux环境下,将项目部署到tomcat中,user.dir指向的路径为tomcat的bin    
## Java获取当前class的绝对路径

```
    public String getPath() {
        String path = this.getClass().getProtectionDomain().getCodeSource().getLocation().getPath();
        if(System.getProperty("os.name").contains("dows")) {
            path = path.substring(1,path.length());
        }
        if(path.contains("jar")) {
            path = path.substring(0,path.lastIndexOf("."));
            return path.substring(0,path.lastIndexOf("/"));
        }
        return path.replace("target/classes/", "");
    }
```  
String path = this.getClass().getProtectionDomain().getCodeSource().getLocation().getPath();    
或者    
String path = this.getClass().getProtectionDomain().getCodeSource().getLocation().getFile();    
因为程序已经被打包成jar包，所以getPath()和getFile()在这里的返回值是一样的。都是/xxx/xxx.jar这种形式。如果路径包含Unicode字符，还需要将路径转码    
path = java.net.URLDecoder.decode(path, "UTF-8");    
## 使用JVM
String path = System.getProperty("java.class.path");    
利用了java运行时的系统属性来得到jar文件位置，也是/xxx/xxx.jar这种形式。    
这样，我们就获得了jar包的位置，但是这还不够，我们需要的是jar包的目录。    
使用    
int firstIndex = path.lastIndexOf(System.getProperty("path.separator")) + 1;    
int lastIndex = path.lastIndexOf(File.separator) + 1;    
path = path.substring(firstIndex, lastIndex);    
来得到目录。  
path.separator在Windows系统下得到;（分号）,在Linux下得到:(冒号)。也就是环境变量中常用来分割路径的两个符号，比如在Windows下我们经常设置环境变量PATH=xxxx\xxx;xxx\xxx;这里获得的就是这个分号。  
File.separator则是/（斜杠）与\（反斜杠），Windows下是\（反斜杠），Linux下是/（斜杠）。  

