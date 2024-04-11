本文将会利用docker-compose对docker进行一键部署个人博客WordPress
部署之前要进行一定准备：
1，安装docker
2，安装docker-compose[,可以参考之前这篇文章](https://blog.csdn.net/Artisan_w/article/details/117562193)
3，官方文档，[英文比较好的建议直接看官方](https://docs.docker.com/samples/wordpress/)

在linux建立文件夹my_wordpress/

```shell
mkdir  my_wordpress/
cd my_wordpress/
```
在my_wordpress下建立docker-compose.yml

```shell
vim docker-compose.yml
```
在这个yml文件中输入命令：

```yml
version: "3.9"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
```
：wq退出后，执行启动命令

```shell
 docker-compose up -d
```
访问自己ip地址的8000端口即可进行建立个人博客
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210607114138105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)

扩展：
在yml文件中包括三层
第一层：version：“3.9”，即docker compose的版本号
第二层：service：。。。为服务，这是yml文件的核心，其中需要注意的 depends_on:-db即为wordpress这个镜像所依赖的镜像为db，因此需要在配置wordpress之前配置db镜像。
第三层：VOLUMS为挂载卷命令其中 **.**代表当前目录