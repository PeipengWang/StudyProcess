# jar常用指令
jar命令参数：
```
jar命令格式：jar {c t x u f }[ v m e 0 M i ][-C 目录]文件名...
其中{ctxu}这四个参数必须选选其一。[v f m e 0 M i ]是可选参数，文件名也是必须的。
-c  创建一个jar包
-t 显示jar中的内容列表
-x 解压jar包
-u 添加文件到jar包中
-f 指定jar包的文件名
-v  生成详细的报造，并输出至标准设备
-m 指定manifest.mf文件.(manifest.mf文件中可以对jar包及其中的内容作一些一设置)
-0 产生jar包时不对其中的内容进行压缩处理
-M 不产生所有文件的清单文件(Manifest.mf)。这个参数与忽略掉-m参数的设置
-i    为指定的jar文件创建索引文件
-C 表示转到相应的目录下执行jar命令,相当于cd到那个目录，然后不带-C执行jar命令
```

## 将class文件打包成jar包
```
jar -cvf test.jar TestA.class TestB.class
```
也可以用通配符
```
jar -cvf test.jar *.class
```
或者，将整个文件夹打包进去
```
jar -cvf test.jar com/github/
```
打包后的jar包会自动生成摘要文件：META-INF/MANIFEST.MF，内容如下
```
Manifest-Version: 1.0
Created-By: 1.8.0_162 (Oracle Corporation)
```
## 解压jar包
```
jar -xvf test.jar
```
## 查看jar包内的文件清单
```
jar -tvf test.jar
```
## 新增或更新jar包内的文件
以下命令将会更新jar包内的TestA.class文件
```
jar -uvf test.jar TestA.class
```
如果文件在jar中不存在，则添加进jar中，如下所示
添加Main.class
```
jar -uvf test.jar Main.class
```
查看jar中的文件列表
```
jar -tvf test.jar
```
## 执行jar
以下的内容会介绍如何执行jar包，Main.class作为可执行的入口类，先给出Main.java的源码，如下所示
以下两条命令都可以指定jar包的入口类，执行main方法
```
java -classpath test.jar Main
```
-cp是-classpath的缩写形式，如果jar包还依赖了其他第三方jar包，列出来即可，如下所示
```
java -cp test.jar:lib/log4j.jar:lib/commons-io.jar Main
```
指定jar默认入口
可以把某个类作为jar包的默认执行入口，使用如下命令
```
jar -uvfe test.jar Main
```
执行后，我们解压jar包，可以看到META-INF/MANIFEST.MF文件多出了一行

```
Manifest-Version: 1.0
Created-By: 1.8.0_162 (Oracle Corporation)
Main-Class: Main
```
指定classpath目录环境启动
```
java -Xbootclasspath/p:/etc/config/ -jar test.jar
```
通过这种方式，可以把配置文件放在/etc/config/目录中，java代码扫描classpath下的配置文件时，就会以-Xbootclasspath/p:指定的目录做为优先查找路径。这样可以把配置和jar分离。



# 前言

如果线上直接更新一个class，可以不用解压，直接利用jar命令直接更新jar报的资源和内容  

# 更新jar包内容文件

Jar 工具提供了一个 u 选项，你可以通过修改其清单或添加文件来更新现有 JAR 文件的内容。  
添加文件的基本命令具有以下格式：  

```
jar uf jar-file input-file(s)
```

在此命令中：

```
u:选项表示你要 update (更新) 现有 JAR 文件。  
f:选项表示要在命令行上指定要更新的 JAR 文件。  
jar-file: 是要更新的现有 JAR 文件。  
input-file(s): 是一个以空格分隔的列表，其中包含要添加到 JAR 文件的一个或多个文件
归档中已存在的文件与添加的文件具有相同的路径名将被覆盖。  
```

创建新的 JAR 文件时，你可以选择使用 -C 选项来指示目录的更改。有关更多信息，请参阅 Creating a JAR File 部分。  
TicTacToe.jar 有以下内容：  

```
META-INF/MANIFEST.MF
TicTacToe.class
```

假设你要将文件 images/new.gif 添加到 JAR 文件中。你可以通过从 images 目录的父目录发出此命令来完成此操作：  
jar uf TicTacToe.jar images/new.gif  
修订后的 JAR 文件将具有以下目录：  

```
META-INF/MANIFEST.MF
TicTacToe.class
images/new.gif
```

在执行命令期间，可以使用 -C 选项“更改目录”。例如：  

```
jar uf TicTacToe.jar -C images new.gif
```
