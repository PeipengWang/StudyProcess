## 使用 Helm 进行部署

### 安装

使用 Helm 部署 KEDA 非常简单：

1. 添加 Helm 存储库

   ```sh
   helm repo add kedacore https://kedacore.github.io/charts
   ```

2. 更新 Helm 仓库

   ```sh
   helm repo update
   ```

3. 安装`keda`Helm chart

   **Helm 3**

   ```sh
   helm install keda kedacore/keda --namespace keda --create-namespace
   ```

### 卸载

如果要从集群中删除 KEDA，首先需要删除已创建的所有 ScaledObjects 和 ScaledJobs。完成后，可以卸载 Helm 图表：

```sh
kubectl delete $(kubectl get scaledobjects.keda.sh,scaledjobs.keda.sh -A \
  -o jsonpath='{"-n "}{.items[*].metadata.namespace}{" "}{.items[*].kind}{"/"}{.items[*].metadata.name}{"\n"}')
helm uninstall keda -n keda
```

注意：如果您卸载 Helm 图表时没有先删除您创建的任何 ScaledObject 或 ScaledJob 资源，它们将变为孤立资源。在这种情况下，您需要修补资源以删除它们的终结器。完成后，它们应该会自动被删除：

```sh
for i in $(kubectl get scaledobjects -A \
  -o jsonpath='{"-n "}{.items[*].metadata.namespace}{" "}{.items[*].kind}{"/"}{.items[*].metadata.name}{"\n"}');
do kubectl patch $i -p '{"metadata":{"finalizers":null}}' --type=merge
done

for i in $(kubectl get scaledjobs -A \
  -o jsonpath='{"-n "}{.items[*].metadata.namespace}{" "}{.items[*].kind}{"/"}{.items[*].metadata.name}{"\n"}');
do kubectl patch $i -p '{"metadata":{"finalizers":null}}' --type=merge
done
```

## 使用部署 YAML 文件进行部署

### 安装

[如果你想在Minikube](https://minikube.sigs.k8s.io/)上尝试 KEDA或者不使用 Helm 的其他 Kubernetes 部署，您仍然可以使用它来部署它`kubectl`。

- 我们提供了示例 YAML 声明，其中包括我们的 CRD 和所有其他资源，这些资源包含在[GitHub 版本中](https://github.com/kedacore/keda/releases)页面。运行以下命令（如果需要，请将本例中的版本替换`2.9.3`为您所使用的版本）：

```sh
kubectl apply -f https://github.com/kedacore/keda/releases/download/v2.9.3/keda-2.9.3.yaml
```

- 或者，您可以下载文件并从本地路径部署它：

```sh
kubectl apply -f keda-2.9.3.yaml
```

- [您还可以在我们的GitHub 存储库](https://github.com/kedacore/keda)`/config`目录中找到相同的 YAML 声明如果您希望克隆它。

```sh
git clone https://github.com/kedacore/keda && cd keda

VERSION=2.9.3 make deploy
```

### 卸载

- 如果从发布的 YAML 文件安装，只需运行以下命令（如果需要，请将版本替换`2.9.3`为您正在使用的版本）：

```sh
kubectl delete -f https://github.com/kedacore/keda/releases/download/v2.9.3/keda-2.9.3.yaml
```

- 如果你已将文件下载到本地，则可以运行：

```sh
kubectl delete -f keda-2.9.3.yaml
```

- [您需要在克隆的GitHub 存储库](https://github.com/kedacore/keda)目录中运行这些命令：

```sh
VERSION=2.9.3 make undeploy
```