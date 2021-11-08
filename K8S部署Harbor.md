参考链接

https://blog.51cto.com/u_14268033/2459534

Kubernetes 集群仓库 harbor Helm3 部署 https://cloud.tencent.com/developer/article/1754686

K8sversion 1.19.2

Docker version  19.03.13

Helm version v3.7.1

Harbor version  1.7.0  https://github.com/goharbor/harbor-helm/tree/1.7.0



# 下载并修改

```
helm repo add harbor https://helm.goharbor.io
helm pull harbor/harbor --version 1.7.0
tar -zcvf harbor-1.7.0.tgz
##配置values.yaml  
vim harbor/values.yaml
externalUrl = https://core.harbor.domain:30003
expose.type=nodePort
expose.tls.auto.commonName= "core.harbor.domain"
```

# 安装

```
kubectl create namespace harbor
helm install harbor ./harbor -n harbor
```

卸载

```
helm uninstall harbor
```

# Host 配置域名

```javascript
vim /etc/hosts
192.168.1.224 core.harbor.domain
```

# 访问

https://core.harbor.domain:30003

U admin

P  Harbor12345

# 服务器配置镜像仓库

#### **下载 Harbor 证书**

Harobr主页->配置管理->系统配置->镜像库根证书

#### 服务器 Docker 中配置 Harbor 证书

```javascript
mkdir -p /etc/docker/certs.d/core.harbor.domain
```

#### 上传下载好的ca证书

```
scp ca.crt root@core.harbor.domain:/etc/docker/certs.d/core.harbor.domain
```

### 解决unknown authority 

```
vim /usr/lib/systemd/system/docker.service
找到ExecStart 修改
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock --insecure-registry https://core.harbor.domain:30003 --ipv6=false
```

```
systemctl daemon-reload
systemctl restart docker
```



#### docker登陆

```
docker login -u admin -p Harbor12345 core.harbor.domain:30003
```

## 测试功能

### **推送与拉取 Docker 镜像**

```javascript
# 拉取 Helloworld 镜像
docker pull hello-world:latest

# 将下载的镜像使用 tag 命令改变镜像名
docker tag hello-world:latest core.harbor.domain:30003/library/hello-world:latest

# 推送镜像到镜像仓库
docker push core.harbor.domain:30003/library/hello-world:latest
```

将之前的下载的镜像删除，然后测试从 core.harbor.domain:30003 下载镜像进行测试：

```
docker pull core.harbor.domain:30003/library/hello-world:latest
```

