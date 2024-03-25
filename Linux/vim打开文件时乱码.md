# vim打开文件中文是乱码
问题：在Linux系统下，使用cat查看含有中文的文本文件正常，但是使用vim打开却是乱码

解决方法：
方法一：
在文件中设定

在vim的退出模式下  :set encoding=utf8
方法二：
直接写入/etc/vim/vimrc文件,在/etc/vim/vimrc文件末尾加上

```
set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
```

【vim知识扩展】
一、存在3个变量：

encoding----该选项使用于缓冲的文本(你正在编辑的文件)，寄存器，Vim 脚本文件等等。\
这事可以把 'encoding' 选项当作是对 Vim 内部运行机制的设定。
fileencoding----该选项是vim写入文件时采用的编码类型。
termencoding----该选项代表输出到客户终端（Term）采用的编码类型。

二、此3个变量的默认值：

```
encoding----与系统当前locale相同，所以编辑文件的时候要考虑当前locale，否则要设置的东西就比较多了。
fileencoding----vim打开文件时自动辨认其编码，fileencoding就为辨认的值。\
为空则保存文件时采用encoding的编码，如果没有修改encoding，那值就是系统当前locale了。
termencoding----默认空值，也就是输出到终端不进行编码转换