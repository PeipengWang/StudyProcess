### master节点使用kubeadm命令初始化集群

- 生成集群初始化YAML配置文件

kubeadm config print init-defaults > kubeadm.yaml

- 按需要修改yaml配置

  ```
  apiVersion: kubeadm.k8s.io/v1beta2
  bootstrapTokens:
  - groups:
    - system:bootstrappers:kubeadm:default-node-token
    token: abcdef.0123456789abcdef
    ttl: 24h0m0s
    usages:
    - signing
    - authentication
  kind: InitConfiguration
  localAPIEndpoint:
    # 此处需要修改为master主节点 IP
    advertiseAddress:  10.132.24.99
    bindPort: 6443
  nodeRegistration:
    criSocket: /var/run/dockershim.sock
    name: kubernetes-master
    taints:
    - effect: NoSchedule
      key: node-role.kubernetes.io/master
  ---
  apiServer:
    timeoutForControlPlane: 4m0s
  apiVersion: kubeadm.k8s.io/v1beta2
  certificatesDir: /etc/kubernetes/pki
  clusterName: kubernetes
  # 多主节点需要修改这里：修改为主节点ip
  controlPlaneEndpoint: "10.136.17.12:6443"
  controllerManager: {}
  dns:
    type: CoreDNS
  etcd:
    local:
      dataDir: /var/lib/etcd
  # 国内不能访问 Google，修改为阿里云
  imageRepository: registry.aliyuncs.com/google_containers
  kind: ClusterConfiguration
  # 修改版本号
  kubernetesVersion: v1.19.0
  networking:
    dnsDomain: cluster.local
    # 配置成 Calico 的默认网段
    podSubnet: "10.244.0.0/16"
    serviceSubnet: 10.1.0.0/16
  scheduler: {}
  ---
  # 开启 IPVS 模式
  apiVersion: kubeproxy.config.k8s.io/v1alpha1
  kind: KubeProxyConfiguration
  featureGates:
    SupportIPVSProxyMode: true
  mode: ipvs
  
  ```

  

  - 初始化集群 kubeadm init --config kubeadm.yaml --upload-certs // upload-certs用来同时生成master join语句
  
    ```
    Your Kubernetes control-plane has initialized successfully!
    
    To start using your cluster, you need to run the following as a regular user:
    
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    
    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
      https://kubernetes.io/docs/concepts/cluster-administration/addons/
    
    You can now join any number of the control-plane node running the following command on each as root:
    
      kubeadm join 10.136.17.12:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:2fbacdf6a9473d5da1d98900f73cd4e2396521772b12ac99017d6ae756d8c3cc \
        --control-plane --certificate-key f0725584c26c192xxxxx4dc5804a101ebc5d7b40257837eea0676d1972cca
    
    Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
    As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
    "kubeadm init phase upload-certs --upload-certs" to reload certs afterward.
    
    Then you can join any number of worker nodes by running the following on each as root:
    
    kubeadm join 10.136.17.12:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:2fbacdf6a9473d5da1d98900f7xxxsssss6ae756d8c3cc 
    ```
  
    

- 拷贝kube/config文件 这样在当前节点可以执行 kubectl指令

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- 安装网络插件

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

- 添加node节点到集群中

```
kubeadm join 10.136.17.12:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:2fbacdf6a9473d5da1d98900f7xxxsssss6ae756d8c3cc 
```

- 如需添加master节点执行以下操作

```
kubeadm join 10.136.17.12:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:2fbacdf6a9473d5da1d98900f73cd4e2396521772b12ac99017d6ae756d8c3cc \
    --control-plane --certificate-key f0725584c26c192xxxxx4dc5804a101ebc5d7b40257837eea0676d1972cca
```

清理脚本

```
#!/bin/bash
kubeadm reset -f 
ifconfig cni0 down && ip link delete cni0
ifconfig flannel.1 down && ip link delete flannel.1
rm -rf /var/lib/cni/
```
