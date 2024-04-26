### 下载

https://get.helm.sh/helm-v3.2.3-linux-amd64.tar.gz



### 解压

tar -zxvf helm-v3.10.2-linux-amd64.tar.gz

将解压目录的helm移动到/usr/local/bin/

### 添加阿里云仓库

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

\# 查看仓库列表
helm repo list

\# 搜索 ingress-nginx
helm search repo ingress-nginx