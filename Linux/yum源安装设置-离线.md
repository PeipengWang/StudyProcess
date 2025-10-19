在公司安装大数据平台时，基本都是离线的环境，有时候安装软件少一些依赖很麻烦，可以通过先在自己的linux虚拟机环境下联网下载好相应的rpm包和依赖，然后再拷贝到生产的机器中，并制作成离线yum源。这样可以解决离线情况下依赖rpm包的问题！

步骤1：下载rpm包和相应的依赖
当所需要的依赖包较少的话，通过手动的方式，一个个查找并下载，是一个可以考虑的方案，但当一个软件有上百个依赖、并且依赖上又有依赖，就需要借助工具了。

yumdownloade的安装和使用
1.安装 yumdownloade

```
yum install yum-utils -y
```

2.yumdownloade使用 

格式：yumdownloader 软件名，如：

```
yumdownloader httpd
```

默认情况下，下载的包会被保存在当前目录中，这个命令不加参数的话任何依赖包不会被下载。

我们可以通过加参数 –resolve参数，根据所有依赖性下载软件包，使得下载包的过程中同时下载依赖以及自定义下载位置。

格式为：yumdownloader 软件名  --resolve --destdir=保存目录 

```
yum downloader httpd --resolve --destdir=/root/package/httpd
```

步骤2：自制离线yum源
前提准备：已安装createrepo服务。

步骤1 已将httpd服务的rpm包下载到目录：/root/package/httpd下，此时可以把该目录下的rpm包打包放到离线的那台机器上，这里假如离线的机器也有相同的目录。

1、在httpd目录下生成repodata文件

```
createrepo .   ###注意 . 点
```

2、新增yum文件

```
vi /etc/yum.repos.d/httpd.repo
```

```
[httpd]
name=httpd
baseurl=file:///root/package/httpd
enabled=1
gpgcheck=0
```

3、刷新yum源

```
yum clean all
yum makecache
```

4、现在就可以直接在离线的机器上运行命令

```
yum -y install httpd 
```

参考文章：

https://blog.csdn.net/u011396718/article/details/80153515

https://www.cnblogs.com/wholj/p/10861857.html