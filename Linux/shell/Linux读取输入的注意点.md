
# Linu读取输入的注意点
#在 Linux shell 中，
```
"$1"、${1} 和 $1 都是用于读取脚本的第一个参数的变量，但它们之间存在一些细微的区别。
```
"$1"：表示第一个参数的值，被双引号引用。在使用 "$1" 时，如果参数中存在空格，双引号将保留参数的完整性，不会将其拆分为多个参数。例如：
```
#!/bin/bash
echo "The first argument is: $1"
```
如果执行命令 ./script.sh "Hello World"，则输出结果为 The first argument is: Hello World。
${1}：同样表示第一个参数的值，但是 ${1} 用花括号括起来，是一种变量替换的形式。在使用 ${1} 时，可以将其与其他文本字符串拼接在一起。例如：
```
#!/bin/bash
echo "The first argument is: ${1}th parameter"
```
如果执行命令 ./script.sh "First"，则输出结果为 The first argument is: Firstth parameter。
$1：也表示第一个参数的值，但是没有任何括号或引号包围。在使用 $1 时，如果参数中存在空格，Shell 会将其解释为多个参数。例如：
```
#!/bin/bash
echo "The first argument is: $1"
```
如果执行命令 ./script.sh Hello World，则输出结果为 The first argument is: Hello，因为 Shell 解释 Hello 为第一个参数，而 World 被解释为第二个参数。

```
综上所述，"$1"、${1} 和 $1 都用于获取脚本的第一个参数，但是它们之间的区别在于引号和括号的使用，以及参数是否被解释为多个参数。
```