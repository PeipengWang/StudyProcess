# 前言
如果线上直接更新一个class，可以不用解压，直接利用jar命令直接更新jar报的资源和内容
# 更新jar包内容文件
Jar 工具提供了一个 u 选项，你可以通过修改其清单或添加文件来更新现有 JAR 文件的内容。
添加文件的基本命令具有以下格式：
```
jar uf jar-file input-file(s)
```
在此命令中：
u 选项表示你要 update (更新) 现有 JAR 文件。
f 选项表示要在命令行上指定要更新的 JAR 文件。
jar-file 是要更新的现有 JAR 文件。
input-file(s) 是一个以空格分隔的列表，其中包含要添加到 JAR 文件的一个或多个文件。
归档中已存在的文件与添加的文件具有相同的路径名将被覆盖。

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
