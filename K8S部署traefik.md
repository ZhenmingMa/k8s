## 部署文档：

由于 Traefik 2.X 版本和之前的 1.X 版本不兼容，我们这里选择功能更加强大的 2.X 版本来和大家进行讲解，我们这里使用的是最新的镜像 traefik:2.3.6。

在 Traefik 中的配置可以使用两种不同的方式：

**动态配置**：完全动态的路由配置
**静态配置**：启动配置

静态配置中的元素（这些元素不会经常更改）连接到 providers 并定义 Treafik 将要监听的 entrypoints。

动态配置包含定义系统如何处理请求的所有配置内容，这些配置是可以改变的，而且是无缝热更新的，没有任何请求中断或连接损耗。

这里我们还是使用 Helm 来快速安装 traefik，首先获取 Helm Chart 包：

```
git clone https://github.com/traefik/traefik-helm-chartcd traefik-helm-chart
```

创建一个定制的 values 配置文件：**values-prod.yaml**

```
# Create an IngressRoute for the dashboard
ingressRoute:
  dashboard:
    enabled: false  # 禁用helm中渲染的dashboard，我们自己手动创建
# Configure ports
ports:
  web:
    port: 8000
    hostPort: 80  # 使用 hostport 模式
    # Use nodeport if set. This is useful if you have configured Traefik in a
    # LoadBalancer
    # nodePort: 32080
    # Port Redirections
    # Added in 2.2, you can make permanent redirects via entrypoints.
    # https://docs.traefik.io/routing/entrypoints/#redirection
    # redirectTo: websecure
  websecure:
    port: 8443
    hostPort: 443  # 使用 hostport 模式
# Options for the main traefik service, where the entrypoints traffic comes
# from.
service:  # 使用 hostport 模式就不需要Service了
  enabled: false
# Logs
# https://docs.traefik.io/observability/logs/
logs:
  general:
    level: DEBUG
tolerations:   # kubeadm 安装的集群默认情况下master是有污点，需要容忍这个污点才可以部署
- key: "node-role.kubernetes.io/master"
  operator: "Equal"
  effect: "NoSchedule"
nodeSelector:   # 固定到master1节点
  kubernetes.io/hostname: "k8s-master-01"
```



这里我们使用 hostport 模式将 Traefik 固定到 master1 节点上，因为只有这个节点有外网 IP，所以我们这里 master1 是作为流量的入口点。直接使用上面的 values 文件安装 traefik：

```
➜ helm install --namespace kube-system traefik ./traefik -f ./values-prod.yaml
NAME: traefikLAST DEPLOYED: Thu Dec 24 11:23:51 2020NAMESPACE: kube-systemSTATUS: deployedREVISION: 1TEST SUITE: None➜ kubectl get pods -n kube-system -l app.kubernetes.io/name=traefikNAME                       READY   STATUS    RESTARTS   AGEtraefik-78ff486794-64jbd   1/1     Running   0          3m15s
```

## 可视化界面 (traefik-dashboard.yaml)

```
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-dashboard
  namespace: kube-system
spec:
  routes:
  - match: Host(`traefik`) && (PathPrefix(`/api`) || PathPrefix(`/dashboard`)) # 指定域名
    kind: Rule
    services:
    - name: api@internal
      kind: TraefikService # 引用另外的 Traefik Service
    middlewares:
      - name: authsecret
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: authsecret
  namespace: kube-system
spec:
  basicAuth:
    secret: authsecret # Kubernetes secret named "secretName"
---
apiVersion: v1
kind: Secret
metadata:
  name: authsecret
  namespace: kube-system
data:
  users: YWRtaW46JGFwcjEkaTFnaXlubnckamR5cXpaVkxjaS9YVmxiM2ZTSUZHLwoK
➜ kubectl get ingressroute -n kube-system
NAME                AGE
traefik-dashboard   19m
```