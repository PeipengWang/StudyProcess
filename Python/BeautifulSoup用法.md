# BeautifulSoup用法
BeautifulSoup4将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种:
Tag
NavigableString
BeautifulSoup
Comment
## Tag
我们可以利用 soup 加标签名轻松地获取这些标签的内容，这些对象的类型是bs4.element.Tag。但是注意，它查找的是在所有内容中的第一个符合要求的标签。
```
from bs4 import BeautifulSoup 
file = open('./aa.html', 'rb') 
html = file.read() 
bs = BeautifulSoup(html,"html.parser") 
获取title标签的所有内容
print(bs.title) 
获取head标签的所有内容
print(bs.head) 
获取第一个a标签的所有内容
print(bs.a) 
类型
print(type(bs.a))
```
对于 Tag，它有两个重要的属性，是 name 和 attrs
5.1、.contents：获取Tag的所有子节点，返回一个list

### tag的.content 属性可以将tag的子节点以列表的方式输出
print(bs.head.contents)
### 用列表索引来获取它的某一个元素
print(bs.head.contents[1])
.children：获取Tag的所有子节点，返回一个生成器
.descendants：获取Tag的所有子孙节点
.strings：如果Tag包含多个字符串，即在子孙节点中有内容，可以用此获取，而后进行遍历
.stripped_strings：与strings用法一致，只不过可以去除掉那些多余的空白内容
.parent：获取Tag的父节点
.parents：递归得到父辈元素的所有节点，返回一个生成器
.previous_sibling：获取当前Tag的上一个节点，属性通常是字符串或空白，真实结果是当前标签与上一个标签之间的顿号和换行符
.next_sibling：获取当前Tag的下一个节点，属性通常是字符串或空白，真是结果是当前标签与下一个标签之间的顿号与换行符
.previous_siblings：获取当前Tag的上面所有的兄弟节点，返回一个生成器
.next_siblings：获取当前Tag的下面所有的兄弟节点，返回一个生成器
.previous_element：获取解析过程中上一个被解析的对象(字符串或tag)，可能与previous_sibling相同，但通常是不一样的
.next_element：获取解析过程中下一个被解析的对象(字符串或tag)，可能与next_sibling相同，但通常是不一样的
.previous_elements：返回一个生成器，可以向前访问文档的解析内容
.next_elements：返回一个生成器，可以向后访问文档的解析内容
.has_attr：判断Tag是否包含属性
## NavigableString
既然我们已经得到了标签的内容，那么问题来了，我们要想获取标签内部的文字怎么办呢？很简单，用 .string 即可
```
from bs4 import BeautifulSoup 
file = open('./aa.html', 'rb') 
html = file.read() 
bs = BeautifulSoup(html,"html.parser")
print(bs.title.string) 
print(type(bs.title.string))
```
## BeautifulSoup
BeautifulSoup对象表示的是一个文档的内容。大部分时候，可以把它当作 Tag 对象，是一个特殊的 Tag，我们可以分别获取它的类型，名称，以及属性
```
from bs4 import BeautifulSoup 
file = open('./aa.html', 'rb') 
html = file.read()
bs = BeautifulSoup(html,"html.parser") 
print(type(bs.name)) 
print(bs.name) 
print(bs.attrs)
```
## find_all(name, attrs, recursive, text, **kwargs)
### name参数：
#### 字符串过滤
会查找与字符串完全匹配的内容
```
a_list = bs.find_all("a")
print(a_list)
```
#### 正则表达式过滤
如果传入的是正则表达式，那么BeautifulSoup4会通过search()来匹配内容
```
from bs4 import BeautifulSoup 
import re 
file = open('./aa.html', 'rb') 
html = file.read() 
bs = BeautifulSoup(html,"html.parser") 
t_list = bs.find_all(re.compile("a")) 
for item in t_list: 
   print(item)
```
#### 列表
如果传入一个列表，BeautifulSoup4将会与列表中的任一元素匹配到的节点返回
```
t_list = bs.find_all(["meta","link"])
for item in t_list:
    print(item)
```
#### 方法
传入一个方法，根据方法来匹配

```
from bs4 import BeautifulSoup
file = open('./aa.html', 'rb')
html = file.read()
bs = BeautifulSoup(html,"html.parser")
def name_is_exists(tag):
    return tag.has_attr("name")
t_list = bs.find_all(name_is_exists)
for item in t_list:
    print(item)
```

### kwargs参数

```
from bs4 import BeautifulSoup
import re
file = open('./aa.html', 'rb')
html = file.read()
bs = BeautifulSoup(html,"html.parser")
```
#### 查询id=head的Tag
```
t_list = bs.find_all(id="head") print(t_list)
```
#### 查询href属性包含ss1.bdstatic.com的Tag
```
t_list = bs.find_all(href=re.compile("http://news.baidu.com"))
print(t_list)
```
#### 查询所有包含class的Tag(注意：class在Python中属于关键字，所以加_以示区别)
```
t_list = bs.find_all(class_=True)
for item in t_list:
    print(item)
```
### attrs参数
并不是所有的属性都可以使用上面这种方式进行搜索，比如HTML的data-*属性：
```
t_list = bs.find_all(data-foo="value")
```
如果执行这段代码，将会报错。我们可以使用attrs参数，定义一个字典来搜索包含特殊属性的tag：
```
t_list = bs.find_all(attrs={"data-foo":"value"})
for item in t_list:
    print(item)
```
### text参数：
通过text参数可以搜索文档中的字符串内容，与name参数的可选值一样，text参数接受 字符串，正则表达式，列表
```
from bs4 import BeautifulSoup 
import re 
file = open('./aa.html', 'rb') 
html = file.read() 
bs = BeautifulSoup(html, "html.parser") 
t_list = bs.find_all(attrs={"data-foo": "value"}) 
for item in t_list: 
    print(item) 
t_list = bs.find_all(text="hao123") 
for item in t_list: 
    print(item) 
t_list = bs.find_all(text=["hao123", "地图", "贴吧"]) 
for item in t_list: 
    print(item) 
t_list = bs.find_all(text=re.compile("\d")) 
for item in t_list: 
    print(item)
当我们搜索text中的一些特殊属性时，同样也可以传入一个方法来达到我们的目的：

def length_is_two(text):
    return text and len(text) == 2
t_list = bs.find_all(text=length_is_two)
for item in t_list:
    print(item)
```
### limit参数：
可以传入一个limit参数来限制返回的数量，当搜索出的数据量为5，而设置了limit=2时，此时只会返回前2个数据

```
from bs4 import BeautifulSoup 
import re 
file = open('./aa.html', 'rb') 
html = file.read() 
bs = BeautifulSoup(html, "html.parser") 
t_list = bs.find_all("a",limit=2) 
for item in t_list: 
    print(item)
```
## find()

find()将返回符合条件的第一个Tag，有时我们只需要或一个Tag时，我们就可以用到find()方法了。当然了，也可以使用find_all()方法，传入一个limit=1，然后再取出第一个值也是可以的，不过未免繁琐。

## CSS选择器
### 通过标签名查找
```
print(bs.select('title'))
print(bs.select('a'))
```
### 通过类名查找
```
print(bs.select('.mnav'))
```
### 通过id查找
```
print(bs.select('#u1'))
```
### 组合查找
```
print(bs.select('div .bri'))
```
### 属性查找
```
print(bs.select('a[class="bri"]'))
print(bs.select('a[href="http://tieba.baidu.com"]'))
```
### 直接子标签查找
```
t_list = bs.select("head > title")
print(t_list)
```
### 兄弟节点标签查找

```
t_list = bs.select(".mnav ~ .bri")
print(t_list)
```
### 获取内容
```
t_list = bs.select("title")
print(bs.select('title')[0].get_text())
```



