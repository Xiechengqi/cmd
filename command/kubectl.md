kubectl
====

Kubernetes 命令行工具，用于管理 Kubernetes 集群

## 补充说明

**kubectl命令** 是 Kubernetes 集群管理的主要命令行工具。它允许用户与 Kubernetes API 服务器交互，执行部署应用、检查和管理集群资源、查看日志等操作。kubectl 支持多种操作模式，包括命令式命令、命令式对象配置和声明式对象配置。

###  语法

```shell
kubectl [command] [TYPE] [NAME] [flags]
```

###  全局选项

```shell
--cluster=''                    # 指定要使用的 kubeconfig 集群名称
--context=''                    # 指定要使用的 kubeconfig 上下文名称
--kubeconfig=''                 # 指定 kubeconfig 配置文件路径
-n, --namespace=''              # 指定命名空间（默认为 default）
--request-timeout='0'           # 请求超时时间（秒）
--server=''                     # Kubernetes API 服务器地址
--token=''                      # 认证令牌
--user=''                       # 指定要使用的 kubeconfig 用户名称
--v=0                           # 日志详细级别
--validate=true                 # 是否验证资源
--watch=false                   # 监听资源变化
--output=''                     # 输出格式：json, yaml, name, go-template, jsonpath, custom-columns 等
-o, --output=''                 # 输出格式简写
--all-namespaces=false          # 在所有命名空间中操作
-A                              # 在所有命名空间中操作（简写）
--show-labels=false             # 显示资源的标签
--sort-by=''                    # 排序字段
--no-headers=false              # 不输出表头
```

###  常用命令

```shell
# 基础命令
get               # 显示一个或多个资源
describe          # 显示资源的详细状态
create            # 从文件或标准输入创建资源
apply             # 通过文件或标准输入配置资源
delete            # 删除资源
edit              # 编辑服务器上的资源
exec              # 在容器中执行命令
logs              # 输出容器日志
port-forward      # 将本地端口转发到 Pod
cp                # 在容器和本地文件系统间复制文件/目录

# 集群管理命令
cluster-info      # 显示集群信息
top               # 显示资源（CPU/内存）使用情况
cordon            # 标记节点为不可调度
uncordon          # 标记节点为可调度
drain             # 排空节点以准备维护
taint             # 更新节点的污点

# 故障排查和调试命令
describe          # 显示资源的详细状态
logs              # 输出容器日志
exec              # 在容器中执行命令
attach            # 附加到运行中的容器
debug             # 调试集群和工作负载
events            # 列出事件
proxy             # 运行代理到 Kubernetes API 服务器

# 高级命令
auth              # 检查授权
diff              # 对比当前版本和提议版本
explain           # 获取资源文档
api-resources     # 列出 API 资源
api-versions      # 列出 API 版本
config            # 修改 kubeconfig 文件
plugin            # 运行命令行插件
version           # 显示客户端和服务器版本信息

# 设置命令
label             # 更新资源的标签
annotate          # 更新资源的注解
patch             # 使用策略合并补丁更新资源
scale             # 为 Deployment、ReplicaSet 等设置新的副本数
autoscale         # 自动伸缩 Deployment、ReplicaSet 等
```

###  资源类型缩写

```shell
po        pods
deploy    deployments
rs        replicasets
sts       statefulsets
ds        daemonsets
svc       services
ep        endpoints
ing       ingresses
cm        configmaps
secret    secrets
ns        namespaces
sa        serviceaccounts
pv        persistentvolumes
pvc       persistentvolumeclaims
sc        storageclasses
job       jobs
cronjob   cronjobs
```

###  实例

#### 获取资源信息

列出所有 Pod：

```shell
kubectl get pods
```

列出所有命名空间中的 Pod：

```shell
kubectl get pods --all-namespaces
# 或
kubectl get pods -A
```

列出所有 Deployment：

```shell
kubectl get deployments
```

列出所有资源（特定类型）：

```shell
kubectl get all
```

#### 查看资源详情

查看 Pod 的详细信息：

```shell
kubectl describe pod <pod-name>
```

查看 Deployment 的详细信息：

```shell
kubectl describe deployment <deployment-name>
```

#### 创建资源

从 YAML 文件创建资源：

```shell
kubectl create -f pod.yaml
```

从 URL 创建资源：

```shell
kubectl create -f https://example.com/manifest.yaml
```

快速创建 Pod：

```shell
kubectl run nginx --image=nginx --port=80
```

#### 应用配置

应用 YAML 配置文件（创建或更新）：

```shell
kubectl apply -f deployment.yaml
```

应用目录中的所有配置文件：

```shell
kubectl apply -f configs/
```

#### 删除资源

删除 Pod：

```shell
kubectl delete pod <pod-name>
```

删除 Deployment：

```shell
kubectl delete deployment <deployment-name>
```

删除命名空间中的所有资源：

```shell
kubectl delete all --all -n <namespace>
```

#### 编辑资源

编辑 Deployment 配置：

```shell
kubectl edit deployment <deployment-name>
```

编辑 ConfigMap：

```shell
kubectl edit configmap <configmap-name>
```

#### 容器操作

进入 Pod 的容器执行命令：

```shell
kubectl exec -it <pod-name> -- /bin/bash
```

在容器中执行单条命令：

```shell
kubectl exec <pod-name> -- ls /app
```

查看容器日志：

```shell
kubectl logs <pod-name>
```

跟踪实时日志：

```shell
kubectl logs -f <pod-name>
```

查看指定容器的日志（Pod 中有多个容器时）：

```shell
kubectl logs <pod-name> -c <container-name>
```

#### 端口转发

将本地端口转发到 Pod：

```shell
kubectl port-forward pod/<pod-name> 8080:80
```

将本地端口转发到 Service：

```shell
kubectl port-forward svc/<service-name> 8080:80
```

#### 文件复制

从容器复制文件到本地：

```shell
kubectl cp <pod-name>:/path/to/file ./local-file
```

从本地复制文件到容器：

```shell
kubectl cp ./local-file <pod-name>:/path/to/file
```

#### 扩展 Deployment

调整 Deployment 的副本数：

```shell
kubectl scale deployment <deployment-name> --replicas=3
```

#### 滚动更新

查看滚动更新状态：

```shell
kubectl rollout status deployment/<deployment-name>
```

回滚到上一版本：

```shell
kubectl rollout undo deployment/<deployment-name>
```

查看更新历史：

```shell
kubectl rollout history deployment/<deployment-name>
```

回滚到特定版本：

```shell
kubectl rollout undo deployment/<deployment-name> --to-revision=2
```

#### 节点管理

标记节点为不可调度：

```shell
kubectl cordon <node-name>
```

标记节点为可调度：

```shell
kubectl uncordon <node-name>
```

排空节点（安全移除 Pod）：

```shell
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

#### 资源使用监控

查看节点资源使用：

```shell
kubectl top nodes
```

查看 Pod 资源使用：

```shell
kubectl top pods
```

查看指定命名空间的 Pod 资源使用：

```shell
kubectl top pods -n <namespace>
```

#### 标签和注解管理

为 Pod 添加标签：

```shell
kubectl label pods <pod-name> environment=production
```

为 Pod 添加注解：

```shell
kubectl annotate pods <pod-name> description="production pod"
```

#### 配置管理

查看当前配置：

```shell
kubectl config view
```

查看当前上下文：

```shell
kubectl config current-context
```

切换到指定上下文：

```shell
kubectl config use-context <context-name>
```

#### 输出格式

以 YAML 格式输出：

```shell
kubectl get pod <pod-name> -o yaml
```

以 JSON 格式输出：

```shell
kubectl get pod <pod-name> -o json
```

仅输出资源名称：

```shell
kubectl get pods -o name
```

自定义列输出：

```shell
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

使用 JSONPath 查询：

```shell
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

#### 批量操作

删除所有失败的 Pod：

```shell
kubectl delete pods --field-selector=status.phase=Failed
```

重启所有命名空间中的 Deployment：

```shell
kubectl rollout restart deployment --all-namespaces
```

#### 调试命令

查看集群事件：

```shell
kubectl get events --sort-by='.lastTimestamp'
```

查看 Pod 描述（包含事件）：

```shell
kubectl describe pod <pod-name>
```

进入调试容器：

```shell
kubectl debug -it <pod-name> --image=busybox --target=<container-name>
```

#### 服务管理

暴露 Deployment 为 Service：

```shell
kubectl expose deployment <deployment-name> --port=80 --target-port=80 --type=LoadBalancer
```

查看 Service 端点：

```shell
kubectl get endpoints <service-name>
```

#### 存储管理

查看 PersistentVolumeClaim：

```shell
kubectl get pvc
```

查看 StorageClass：

```shell
kubectl get storageclass
```

#### 身份认证和授权

检查用户权限：

```shell
kubectl auth can-i create pods
kubectl auth can-i delete pods --as=system:serviceaccount:default:my-sa
```

## 扩展知识

### kubectl 使用模式

#### 1. 命令式命令

直接使用 kubectl 命令操作资源：

```shell
kubectl run nginx --image=nginx
kubectl expose deployment nginx --port=80
```

**优点**：简单快速，适合学习和测试。
**缺点**：不可重复，不适合生产环境。

#### 2. 命令式对象配置

通过配置文件创建资源：

```shell
kubectl create -f nginx.yaml
kubectl delete -f nginx.yaml
```

**优点**：可版本控制，可重复。
**缺点**：不支持部分更新。

#### 3. 声明式对象配置

使用 `apply` 管理资源状态：

```shell
kubectl apply -f configs/
kubectl diff -f configs/
```

**优点**：支持部分更新，可自动合并修改。
**缺点**：需要理解声明式模型。

### 配置文件示例

#### Pod 配置

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

#### Deployment 配置

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

#### Service 配置

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

### 最佳实践

#### 1. 使用命名空间

将不同环境或应用隔离到不同命名空间：

```shell
kubectl create namespace production
kubectl create namespace development
```

#### 2. 使用标签和选择器

```yaml
metadata:
  labels:
    app: frontend
    tier: web
    environment: production
```

#### 3. 资源限制

为容器设置资源请求和限制：

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

#### 4. 健康检查

配置存活探针和就绪探针：

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 故障排查

#### 1. Pod 处于 Pending 状态

```shell
kubectl describe pod <pod-name>              # 查看详细事件
kubectl get events --sort-by='.lastTimestamp' # 查看集群事件
```

#### 2. Pod 处于 CrashLoopBackOff 状态

```shell
kubectl logs <pod-name>                      # 查看日志
kubectl logs <pod-name> --previous           # 查看之前容器的日志
kubectl describe pod <pod-name>              # 查看详细状态
```

#### 3. Service 无法访问

```shell
kubectl describe service <service-name>      # 查看服务详情
kubectl get endpoints <service-name>         # 查看端点
kubectl get pods -l app=<label-value>       # 查看匹配的 Pod
```

#### 4. 网络策略问题

```shell
kubectl describe networkpolicy <policy-name> # 查看网络策略
kubectl get networkpolicies                 # 列出所有网络策略
```

### 插件管理

kubectl 支持插件机制，可以通过 `kubectl plugin` 命令管理插件：

```shell
kubectl plugin list                          # 列出已安装插件
kubectl krew update                          # 更新插件索引
kubectl krew install ctx                     # 安装上下文切换插件
kubectl krew install ns                      # 安装命名空间切换插件
```

### 常用插件

- **krew**：kubectl 插件管理器
- **ctx**：快速切换上下文
- **ns**：快速切换命名空间
- **stern**：多 Pod 日志查看器
- **kubectx**：上下文切换工具
- **kubens**：命名空间切换工具
- **kube-shell**：交互式 shell

### 性能优化

#### 1. 使用别名

```shell
alias k=kubectl
alias kg='kubectl get'
alias kd='kubectl describe'
alias ka='kubectl apply'
alias kc='kubectl create'
alias kl='kubectl logs'
alias ke='kubectl exec'
```

#### 2. 自动补全

```shell
# bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

# zsh
source <(kubectl completion zsh)
echo "source <(kubectl completion zsh)" >> ~/.zshrc
```

#### 3. 配置优化

```shell
# 减少超时时间
kubectl --request-timeout=10s get pods

# 关闭颜色输出
kubectl get pods --no-color

# 使用服务器端分页
kubectl get pods --server-print=false
```

### 安全建议

#### 1. 最小权限原则

```shell
# 创建只读 ServiceAccount
kubectl create serviceaccount readonly-sa
kubectl create role readonly-role --verb=get,list --resource=pods
kubectl create rolebinding readonly-binding --role=readonly-role --serviceaccount=default:readonly-sa
```

#### 2. 定期轮换凭证

```shell
# 查看证书过期时间
kubectl config view --raw -o jsonpath='{.users[?(@.name=="kubernetes-admin")].user.client-certificate-data}' | base64 -d | openssl x509 -enddate -noout
```

#### 3. 审计日志

```shell
kubectl get events --sort-by='.lastTimestamp' -A
```

### 版本兼容性

kubectl 客户端版本应与 Kubernetes 服务器版本保持兼容：

- **kubectl 主要版本**：应与 Kubernetes 主要版本匹配
- **kubectl 次要版本**：可以比服务器版本高 1 个版本
- **最低版本**：kubectl 版本不能比服务器版本低超过 2 个次要版本

### 获取帮助

```shell
kubectl help                            # 显示所有命令帮助
kubectl <command> --help                # 显示特定命令帮助
kubectl explain pod                     # 显示资源文档
kubectl explain pod.spec.containers     # 显示字段文档
```

### 相关资源

- Kubernetes 官方文档：https://kubernetes.io/docs/reference/kubectl/
- kubectl 备忘单：https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- kubectl 插件：https://krew.sigs.k8s.io/
- Kubernetes 社区：https://kubernetes.io/community/
