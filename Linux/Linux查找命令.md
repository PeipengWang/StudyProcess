# find
find /dir -name filename 
按名字查找
find . -name "*.c"
将当前目录及其子目录下所有文件后缀为 .c 的文件列出来
find . -type f
将当前目录及其子目录中的所有文件列出
find . -ctime  20
将当前目录及其子目录下所有最近 20 天内更新过的文件列出
find / -type f -size 0 -exec ls -l {} \;
查找系统中所有文件长度为 0 的普通文件，并列出它们的完整路径
find . -perm 755 –print 
在当前目录下查找文件权限位为755的文件，即文件属主可以读、写、执行，其他用户可以读、执行的文件
find /apps -group gem –print 
在/apps目录下查找属于gem用户组的文件
# grep
ls -l | grep '^a' 通过管道过滤ls -l输出的内容，只显示以a开头的行。
grep 'test' d* 显示所有以d开头的文件中包含test的行。
grep 'test' aa bb cc 显示在aa，bb，cc文件中匹配test的行。
grep '[a-z]' aa 显示所有包含每个字符串至少有5个连续小写字符的字符串的行。
grep 'w(es)t.*' aa 如果west被匹配，则es就被存储到内存中，并标记为1，然后搜索任意个字符(.*)，这些字符后面紧跟着另外一个es()，找到就显示该行。如果用egrep或grep -E，就不用""号进行转义，直接写成'w(es)t.*'就可以了。
grep -i pattern files ：不区分大小写地搜索。默认情况区分大小写
grep -l pattern files ：只列出匹配的文件名，
grep -L pattern files ：列出不匹配的文件名，
grep -w pattern files ：只匹配整个单词，而不是字符串的一部分(如匹配‘magic’，而不是‘magical’)，
grep -C number pattern files ：匹配的上下文分别显示[number]行，
grep pattern1 | pattern2 files ：显示匹配 pattern1 或 pattern2 的行，
grep pattern1 files | grep pattern2 ：显示既匹配 pattern1 又匹配 pattern2 的行。

. 匹配任意一个字符
* 匹配0 个或多个*前的字符
^ 匹配行开头
$ 匹配行结尾
[] 匹配[ ]中的任意一个字符，[]中可用 - 表示范围，
例如[a-z]表示字母a 至z 中的任意一个
\ 转意字符
# whereis
whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。
$ whereis grep
grep: /bin/grep /usr/share/man/man1p/grep.1p.gz /usr/share/man/man1/grep.1.gz
# which
which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。
$ which grep
/bin/grep
# locate
locate命令其实是“find -name”的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/locatedb），这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。
locate命令的使用实例：
$ locate /etc/sh
搜索etc目录下所有以sh开头的文件。
$ locate -i ~/m
搜索用户主目录下，所有以m开头的文件，并且忽略大小写。



