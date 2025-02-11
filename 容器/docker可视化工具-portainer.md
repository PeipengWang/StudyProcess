docker可视化工具-portainer
21/100
发布文章
Artisan_w
未选择任何文件
new

```shell
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock -- privileged=true portainer/portainer
```
浏览器访问：ip：8088
首先需要注册或者登陆：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/dockerjiqixuexiadb70b9b1d67b060f3cf9650a9458505.png)
连接选择本地连接：
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/dockera1f942a969f752755b135631d5c90e1d.png)
这是显示的仓库
![在这里插入图片描述](https://raw.githubusercontent.com/PeipengWang/picture/master/docker8dafdaa9a9f439da4694d279db8e3ae8.png)

docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock -- privileged=true portainer/portainer
浏览器访问：ip：8088
首先需要注册或者登陆：
在这里插入图片描述
连接选择本地连接：
在这里插入图片描述
这是显示的仓库
在这里插入图片描述

Markdown 640 字数 13 行数 当前行 1, 当前列 0HTML 158 字数 6 段落
发布博文获得大额流量券







