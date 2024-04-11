下载Nginx

```shell
 docker pull nginx
```

```shell
Using default tag: latest
latest: Pulling from library/nginx
f7ec5a41d630: Already exists
aa1efa14b3bf: Pull complete
b78b95af9b17: Pull complete
c7d6bca2b8dc: Pull complete
cf16cd8e71e0: Pull complete
0241c68333ef: Pull complete
Digest: sha256:75a55d33ecc73c2a242450a9f1cc858499d468f077ea942867e662c247b5e412
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

```
运行

```
docker run -d --name nginx01 -p 8080:80 nginx
0a178ec21a52c08ea40832ae3e4b503dfc2b50e1707eb84e1e13c914f9a2d5cc
[root@VM-0-4-centos ~]# ^C
[root@VM-0-4-centos ~]# docker run -d --name nginx01 -p 8080:80 nginx
docker: Error response from daemon: Conflict. The container name "/nginx01" is already in use by container "0a178ec21a52c08ea40832ae3e4b503dfc2b50e1707eb84e1e13c914f9a2d5cc". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
[root@VM-0-4-centos ~]# curl localhost 8080
curl: (7) Failed connect to localhost:80; Connection refused
curl: (7) Failed to connect to 0.0.31.144: Invalid argument

```
-d：后台运行
--name：别名
-p端口映射
其中端口原理如下所示：外网要访问nginx，需要先通过阿里云，然后访问主机8080。而nginx在将80端口映射到主机8080就可以完成访问。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421160009984.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FydGlzYW5fdw==,size_16,color_FFFFFF,t_70)
查看8080端口的运行状态

```shell
[root@VM-0-4-centos ~]# curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```
访问结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210421160137949.png)

