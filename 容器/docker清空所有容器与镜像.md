清空容器

```shell
docker rm -f $(docker ps -aq)
```
清空镜像

```shell
docker rmi -f $(docker images -aq)
```
注意：清空镜像前要把运行的镜像清空或者停止运行，不然无法清空正在运行中的镜像