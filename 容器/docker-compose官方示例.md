docker-compose是一种docker的编排工具，能够对docker的镜像一键下载，启动并且自动配置网络，功能在如下示例中进行展示
[官方示例地址](https://docs.docker.com/compose/gettingstarted/)
新建文件夹，示例将会在文件夹中进行展示。
```shell
 mkdir composetest
 cd composetest
```
新建一个app.py文件

```shell
vim app.py
```
内容如下：
```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```
新建一个requirements.txt文件，内容如下

```
flask
redis
```
新建Dockerfile文件，内容如下：

```dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```
新建docker-compose.yml文件，内容如下：

```yml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```
这样，建立起来了四个我们工程所需要的四个文件，在本目录下启动

```docker
docker-compose up
```
访问ip：5000地址
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021060616234765.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210606162415775.png)
每次访问都会自动加1，来记录客户端的访问次数。
查看镜像

```
docker images
```
发现，自动下载完成了三个镜像
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210606162506968.png)
观察启动的容器，如下图所示，可以知道自动启动了redis与web两个容器。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210606162640527.png)

观察网络，发现自动配置出一个composetest_default的自定义网络。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210606162752215.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210606162917829.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
