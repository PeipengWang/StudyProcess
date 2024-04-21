# 证书失效问题



错误日志  输入任何指令都会出现

kubectl get pod -A
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")

解决方法：

原因在于这几条指令

```
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

这几个命令会创建一个目录，并复制几个配置文件，重新创建集群时，这个目录还是存在的，于是我尝试在执行这几个命令前先执行`rm -rf $HOME/.kube`命令删除这个目录，最后终于解决了这个问题！！！



这个问题很坑人，删除集群然后重新创建也算是一个常规的操作，如果你在执行 `kubeadm reset`命令后没有删除创建的 `$HOME/.kube`目录，重新创建集群就会出现这个问题！