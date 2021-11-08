

参考

https://www.cnblogs.com/Andya/p/14760281.html

## NFS服务器搭建

安装nfs相关服务软件包

```
yum install -y nfs-utils rpcbind
```

创建共享存储文件夹

```
mkdir /nfs
```

配置nfs

```
$ vi /etc/exports
输入以下内容，格式为：nfs共享目录 nfs客户端地址1(param1, param2,...) nfs客户端地址2(param1, param2,...)
/nfs 10.1.1.0/24(rw,async,no_root_squash)
```

启动服务

```
systemctl start rpcbind
systemctl enable rpcbind
systemctl enable nfs && systemctl restart nfs
```

查看服务状态

```
systemctl status rpcbind
systemctl status nfs
```

查看可用的nfs地址

```
showmount -e 127.0.0.1或showmount -e localhost
```

# NFS客户端配置

1. 安装nfs-utils和rpcbind
   `$ yum install -y nfs-utils rpcbind`
2. 创建挂载的文件夹
   `$ mkdir -p /nfs/data`
3. 挂载nfs
   `$ mount -t nfs 10.1.1.1:/nfs /nfs/data`
   其中：
   `mount`：表示挂载命令
   `-t`：表示挂载选项
   `nfs`：挂载的协议
   `10.1.1.1`:nfs服务器的ip地址
   `/nfs`：nfs服务器的共享目录
   `/nfs/data`：本机客户端要挂载的目录
4. 查看挂载信息
   `$ df -Th`
5. 测试挂载
   可以进入本机的`/nfs/data`目录，上传一个文件，然后去nfs服务器查看/nfs目录中是否有该文件，若有则共享成功。反之在nfs服务器操作`/nfs`目录，查看本机客户端的目录是否共享。
6. 取消挂载
   `$ umount /nfs/data`

## 生成stroageclass.yaml文件

```
cat > stroageclass.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [ "" ]
    resources: [ "persistentvolumes" ]
    verbs: [ "get", "list", "watch", "create", "delete" ]
  - apiGroups: [ "" ]
    resources: [ "persistentvolumeclaims" ]
    verbs: [ "get", "list", "watch", "update" ]
  - apiGroups: [ "storage.k8s.io" ]
    resources: [ "storageclasses" ]
    verbs: [ "get", "list", "watch" ]
  - apiGroups: [ "" ]
    resources: [ "events" ]
    verbs: [ "create", "update", "patch" ]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [ "" ]
    resources: [ "endpoints" ]
    verbs: [ "get", "list", "watch", "create", "update", "patch" ]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              # 必须与class.yaml中的provisioner的名称一致
              value: fuseim.pri/ifs
            - name: NFS_SERVER
              # NFS服务器的ip地址
              value: 192.168.1.224
            - name: NFS_PATH
              # 修改为实际创建的共享挂载目录
              value: /nfs
      volumes:
        - name: nfs-client-root
          nfs:
            # NFS服务器的ip地址
            server: 192.168.1.224
            # 修改为实际创建的共享挂载目录
            path: /nfs
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
# 必须与deployment.yaml中的PROVISIONER_NAME一致
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
EOF
```

## 部署

```
kubectl apply -f stroageclass.yaml
```

## 查看

```
kubectl get sv
```

## 设置默认StroageClass

```
kubectl patch storageclass managed-nfs-storage -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

