# idea搜索与替换
常用搜索快捷键
```
CTRL+F：当前文件全文搜索
CTRL+R：当前文件全文替换
CTRL+SHIFT+F：当前项目按内容查询（全局查找）
CTRL+SHIFT+R：当前项目按内容查询并替换（全局查找替换）
```

IDEA中常用的正则

^P：搜索以P为开头的文件

api$：搜索以api为结尾的文件

.idea：搜索包含字符串idea的文件（.代表任意一个字符，如果单单搜索idea直接用idea即可）
```
^pac.*?com.*?util：搜索以pac开头，中间包含com和util的文件
```
^p.*?impl;$：匹配以p开头impl;结尾的文件

.和*的组合.*表达了中间任意个连起来的字符。
