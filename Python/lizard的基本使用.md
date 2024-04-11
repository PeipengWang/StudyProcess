# lizar的使用
Python: 统计代码复杂度lizard模块的使用
能够同时统计代码量和代码复杂度
## 安装：
按照ptyhon环境
sudo apt install python3-pip
wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz
tar -zxvf Python-3.7.7.tgz
cd Python-3.7.7/
yum update -y
yum install -y make gcc gcc-c++
./configure
make
make install
创建软链接
ln -s /usr/python/bin/python3 /usr/bin/python3
ln -s /usr/python/bin/pip3 /usr/bin/pip3
安装lizard环境
Git源码链接 https://github.com/terryyin/lizard
pip： pip install lizard
源码：python setup.py install --install-dir=/path/to/installation/directory/
安装Request环境(上报使用)
requests包依赖：
certifi，idna，urllib3，chardet
## 圈复杂度概念
圈复杂度是 Thomas J. McCabe 在 1976年开创的软件指标，用来判断程序的复杂度。
这个指标度量源代码中线性独立的路径或分支的数量。
根据 McCabe 所说，一个方法的复杂度最好保持在10 以下。
这是因为对人类记忆力的研究表明，人的短期记忆只能存储 7 件事（偏差为正负 2）。
如果开发人员编写的代码有 50 个线性独立的路径，那么为了在头脑中描绘出方法中发生的情况，需要的记忆力大约超过短期记忆容量的5倍。
简单的方法不会超过人的短期记忆力的极限，因此更容易应付，事实证明它们的错误更少。
Enerjy 在 2008年所做的研究表明，在圈复杂度与错误数量之间有很强的相关性。
复杂度为 11 的类的出错概率为 0.28，而复杂度为 74的类的出错概率会上升到 0.98。
## 实际执行常用命令
cd /.../...你的代码目录
cd /.../...你的代码目录
（1）lizard 默认递归检测文件下的所有文件
（2）lizard -o check.txt 将所有文件输出到某个文件
（3）lizard -C 15 检测CCN超过15
（4）lizard -C 15 .\yingjiafupan\run_fupan.py 检测某个文件CCN超过15
##  字段含义
NLOC，    the nloc (lines of code without comments), 不包含注释的代码行数
CCN，     cyclomatic complexity number， 圈复杂度也就是分支复杂度，最好保持在10 以下
token，   token count of functions. token的个数（关键字，标示符，常量，标点符号，操作符)
param，   parameter count of functions. 参数统计数就是函数的参数个数
Cnt，  Count的缩写
Rt，  Rate的缩写
代码复杂度超限比例=圈复杂度（CCN）大于10的函数的个数/函数的个数
圈复杂度（CCN）大于10的函数的个数：如上面结果，有一个CCN大于10的，取 Warning cnt的值
lizard [options] [PATH or FILE] [PATH] ...
示例 检测某目录下 除 tests目录下 Python文件
lizard mySource/ -x"./tests/*" -l python

-h, --help            显示帮助并退出
--version             显示当前lizard版本并退出
-l LANGUAGES, --languages LANGUAGES
                      开发语言列表， 检测支持的开发语言 例如：lizard -l cpp -l java
                      支持语言：cpp, csharp, java, javascript, objectivec, php, python, ruby, swift, ttcn
-V, --verbose         Output in verbose mode (long function name)
-C CCN, --CCN CCN     Threshold for cyclomatic complexity number warning.
                      The default value is 15. Functions with CCN bigger
                      than it will generate warning
-L LENGTH, --length LENGTH
                      Threshold for maximum function length warning. The
                      default value is 1000. Functions length bigger than it
                      will generate warning
-a ARGUMENTS, --arguments ARGUMENTS
                      Limit for number of parameters
-w, --warnings_only   Show warnings only, using clang/gcc's warning format
                      for printing warnings.
                      http://clang.llvm.org/docs/UsersManual.html#cmdoption-
                      fdiagnostics-format
-i NUMBER, --ignore_warnings NUMBER
                      If the number of warnings is equal or less than the
                      number, the tool will exit normally, otherwise it will
                      generate error. Useful in makefile for legacy code.
-x EXCLUDE, --exclude EXCLUDE    不检测文件
                      Exclude files that match this pattern. * matches
                      everything, ? matches any single character,
                      "./folder/*" exclude everything in the folder
                      recursively. Multiple patterns can be specified. Don't
                      forget to add "" around the pattern.
--csv                 Generate CSV output as a transform of the default
                      output
-X, --xml             Generate XML in cppncss style instead of the tabular
                      output. Useful to generate report in Jenkins server
-t WORKING_THREADS, --working_threads WORKING_THREADS
                      number of working threads. The default value is 1.
                      Using a bigger number can fully utilize the CPU and
                      often faster.
-m, --modified        Calculate modified cyclomatic complexity number,
                      which count a switch/case with multiple cases as
                      one CCN.
-E EXTENSIONS, --extension EXTENSIONS
                      User the extensions. The available extensions are:
                      -Ecpre: it will ignore code in the #else branch.
                      -Ewordcount: count word frequencies and generate tag
                      cloud. -Eoutside: include the global code as one
                      function.
-s SORTING, --sort SORTING
                      Sort the warning with field. The field can be nloc,
                      cyclomatic_complexity, token_count, parameter_count,
                      etc. Or an customized file.
-W WHITELIST, --whitelist WHITELIST  白名单
                      The path and file name to the whitelist file. It's
                      './whitelizard.txt' by default
