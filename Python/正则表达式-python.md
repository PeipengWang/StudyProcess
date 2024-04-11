以爬虫爬取的猫眼为例

```python
import requests
import re

def get_one_page(url):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) '
                      'Chrome/76.0.3809.132 Safari/537.36 '
    }

    response = requests.get(url, headers=headers)
    response.encoding = 'utf-8'
    if response.status_code == 200:
        return response.text


url = 'https://maoyan.com/films?showType=1'
# 发送GET请求
response = get_one_page(url)
print(response)
```
截取其中两部影片的片段作为素材来进行展示python的正则表达式的用法

```xml
</div>

      <div class="movie-ver"></div>
      <div class="movie-item-hover">
        <a href="/films/1222234" target="_blank" data-act="movie-click" data-val="{movieid:1222234}">
           <img class="movie-hover-img" src="https://p0.meituan.net/movie/e2ecb7beb8dadc9f07f2fad9820459f92275588.jpg@218w_300h_1e_1c" alt="我的姐姐" />
          <div class="movie-hover-info">
            <div class="movie-hover-title" title="我的姐姐" >
              <span class="name ">我的姐姐</span>
                <span class="score channel-detail-orange"><i class="integer">8.</i><i class="fraction">9</i></span>
            </div>
            <div class="movie-hover-title" title="我的姐姐" >
              <span class="hover-tag">类型:</span>
              剧情／家庭
            </div>
            <div class="movie-hover-title" title="我的姐姐" >
              <span class="hover-tag">主演:</span>
              张子枫／肖央／朱媛媛
            </div>
            <div class="movie-hover-title movie-hover-brief" title="我的姐姐" >
              <span class="hover-tag">上映时间:</span>
              2021-04-02
            </div>
          </div>
        </a>
      </div>
    </div>
    
    <div class="channel-detail movie-item-title" title="我的姐姐">
      <a href="/films/1222234" target="_blank" data-act="movies-click" data-val="{movieId:1222234}">我的姐姐</a>
    </div>
<div class="channel-detail channel-detail-orange"><i class="integer">8.</i><i class="fraction">9</i></div>
  
  <dd>
    <div class="movie-item film-channel">
      <a href="/films/1299178" target="_blank" data-act="movie-click" data-val="{movieid:1299178}">
        <div class="movie-poster">
          <img class="poster-default" src="//s3plus.meituan.net/v1/mss_e2821d7f0cfe4ac1bf9202ecf9590e67/cdn-prod/file:5788b470/image/loading_2.e3d934bf.png" />
          <img data-src="https://p1.meituan.net/moviemachine/1d67ba8aec840c3e7ecdac6cdbd915c7293706.jpg@160w_220h_1e_1c" alt="名侦探柯南：绯色的子弹海报封面" />
        </div>
      </a>
        
      <div class="movie-ver"></div>
      <div class="movie-item-hover">
        <a href="/films/1299178" target="_blank" data-act="movie-click" data-val="{movieid:1299178}">
           <img class="movie-hover-img" src="https://p1.meituan.net/moviemachine/1d67ba8aec840c3e7ecdac6cdbd915c7293706.jpg@218w_300h_1e_1c" alt="名侦探柯南：绯色的子弹" />
          <div class="movie-hover-info">
            <div class="movie-hover-title" title="名侦探柯南：绯色的子弹" >
              <span class="name noscore">名侦探柯南：绯色的子弹</span>
            </div>
            <div class="movie-hover-title" title="名侦探柯南：绯色的子弹" >
              <span class="hover-tag">类型:</span>
              动作／动画／悬疑
            </div>
            <div class="movie-hover-title" title="名侦探柯南：绯色的子弹" >
              <span class="hover-tag">主演:</span>
              高山南／山崎和佳奈／小山力也
            </div>
            <div class="movie-hover-title movie-hover-brief" title="名侦探柯南：绯色的子弹" >
              <span class="hover-tag">上映时间:</span>
              2021-04-17
            </div>
          </div>
        </a>
      </div>
    </div>
```
## 测试准备
re库

```python
import re
```
通过re库的findAll方法来获取电影的名称
## 获取电影名称

```python
result = re.findall("<div class=\"movie-hover-title\" title=", text)
print(result)
```
输出结果：
['<div class="movie-hover-title" title=', '<div class="movie-hover-title" title=', '<div class="movie-hover-title" title=', '<div class="movie-hover-title" title=', '<div class="movie-hover-title" title=', '<div class="movie-hover-title" title=']
发现所有的<div class=\"movie-hover-title\" title= 的标签都显示出来，我们需要的是这个标签后面的电影名称，于是：

```python
result = re.findall("<div class=\"movie-hover-title\" title=....", text)
print(result)
```
通过 . 来表示此标签后的电影名称
['<div class="movie-hover-title" title="我的姐姐"', '<div class="movie-hover-title" title="我的姐姐"', '<div class="movie-hover-title" title="我的姐姐"', '<div class="movie-hover-title" title="名侦探柯南', '<div class="movie-hover-title" title="名侦探柯南', '<div class="movie-hover-title" title="名侦探柯南']
但发现后面的名称字节数目是不一定的，由此利用*来表明.的次数不限定，即0或者0个以上

```python
result = re.findall("<div class=\"movie-hover-title\" title=.*", text)
print(result)
```
['<div class="movie-hover-title" title="我的姐姐" >', '<div class="movie-hover-title" title="我的姐姐" >', '<div class="movie-hover-title" title="我的姐姐" >', '<div class="movie-hover-title" title="名侦探柯南：绯色的子弹" >', '<div class="movie-hover-title" title="名侦探柯南：绯色的子弹" >', '<div class="movie-hover-title" title="名侦探柯南：绯色的子弹" >']
仅仅保留电影名称则需要进一步处理，利用（）括起来代表保留的字符，同时处理末尾为”，代表仅仅保留引号中间的字符：

```python
result = re.findall("<div class=\"movie-hover-title\" title=\"(.*)\"", text)
print(result)
```
['我的姐姐', '我的姐姐', '我的姐姐', '名侦探柯南：绯色的子弹', '名侦探柯南：绯色的子弹', '名侦探柯南：绯色的子弹']
最后去重：

```python
result = set(result)
print(result)
```
{'我的姐姐', '名侦探柯南：绯色的子弹'}
