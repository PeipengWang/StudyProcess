
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
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421165028501.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
连接选择本地连接：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421165207965.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
这是显示的仓库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421165359859.png)

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








