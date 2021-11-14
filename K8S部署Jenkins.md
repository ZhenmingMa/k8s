1生成jenkins.yaml文件

```
cat > jenkins.yaml << EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-jenkins
  namespace: kube-ops
spec:
  storageClassName: "managed-nfs-storage"
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: kube-ops
  labels:
    name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      name: jenkins
  template:
    metadata:
      name: jenkins
      labels:
        name: jenkins
    spec:
      serviceAccountName: jenkins
      containers:
        - name: jenkins
          image: jenkins/jenkins
          ports:
            - containerPort: 8080
            - containerPort: 50000
          resources:
            limits:
              cpu: 1
              memory: 1.5Gi
            requests:
              cpu: 1
              memory: 1Gi
          env:
            - name: LIMITS_MEMORY
              valueFrom:
                resourceFieldRef:
                  resource: limits.memory
                  divisor: 1Mi
            - name: JAVA_OPTS
              value: -Xmx$(LIMITS_MEMORY)m -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
      securityContext:
        fsGroup: 1000
      volumes:
        - name: jenkins-home
          persistentVolumeClaim:
            claimName: pvc-jenkins
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: kube-ops
spec:
  selector:
    name: jenkins
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 8080
      protocol: TCP
      nodePort: 30008
    - name: agent
      port: 50000
      protocol: TCP
---
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: kube-ops

---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jenkins
  namespace: kube-ops
rules:
  - apiGroups: [ "" ]
    resources: [ "pods","events" ]
    verbs: [ "create","delete","get","list","patch","update","watch" ]
  - apiGroups: [ "" ]
    resources: [ "pods/exec" ]
    verbs: [ "create","delete","get","list","patch","update","watch" ]
  - apiGroups: [ "" ]
    resources: [ "pods/log" ]
    verbs: [ "get","list","watch" ]
  - apiGroups: [ "" ]
    resources: [ "secrets","events" ]
    verbs: [ "get" ]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins
  namespace: kube-ops
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins
subjects:
  - kind: ServiceAccount
    name: jenkins
EOF
```

# 部署

```
kubectl apply -f jenkins.yaml
```

# 登陆

http://master:30008

查看密码

```
cat /nfs/default-jenkins-claim-pvc-6c4d944b-245e-440b-b566-3137c05855ad/secrets/initialAdminPassword 
2537679f73a14acd834cf2ef0d77ce4f
```

# 配置



1.开发环境持续集成和发布
2.较少人工干预，开发更效率，避免操作错误
访问地址：http://jenkins.energy-envision.com/
用户名：姓名全拼 密码：admin123
![img](http://mindoc.energy-envision.com:8090/uploads/202110/dpi/attach_16ae266e6fd3578f.png)
访问域名：http://jenkins.energy-envision.com/
![img](http://mindoc.energy-envision.com:8090/uploads/202111/dpi/attach_16b43b468ea326cf.png)

```
$ kubectl exec -it jenkins-556cd59c8c-2vl8m -n kube-ops -- cat /var/jenkins_home/secrets/initialAdminPassword
```

35b083de1d25409eaef57255e0da481a # jenkins启动日志里面也有
然后跳过插件安装，选择默认安装插件过程会非常慢（也可以选择安装推荐的插件），点击右上角关闭选择插件，等配置好插件中心国内镜像源后再选择安装一些插件。
默认插件就行
![img](http://mindoc.energy-envision.com:8090/uploads/202111/dpi/attach_16b43b513766190c.png)
跳过后会直接进入 Jenkins 就绪页面，直接点击开始使用即可：
![img](http://mindoc.energy-envision.com:8090/uploads/202111/dpi/attach_16b43b545b2d2fe4.png)
配置¶
接下来我们就需要来配置 Jenkins，让他能够动态的生成 Slave 的 Pod。

第 1 步. 我们需要安装 kubernetes 插件， 点击 Manage Jenkins -> Manage Plugins -> Available -> Kubernetes 勾选安装即可。
![img](http://mindoc.energy-envision.com:8090/uploads/202111/dpi/attach_16b43b63229ed640.png)
第 2 步. 安装完毕后，进入 configureClouds 页面：
![img](http://mindoc.energy-envision.com:8090/uploads/202111/dpi/attach_16b43b72d1e74f8e.png)
在该页面我们可以点击 Add a new cloud -> 选择 Kubernetes，首先点击 Kubernetes Cloud details… 按钮进行配置：
![img](http://mindoc.energy-envision.com:8090/uploads/202111/dpi/attach_16b43b77050cb2e0.png)
首先配置连接 Kubernetes APIServer 的地址，由于我们的 Jenkins 运行在 Kubernetes 集群中，所以可以使用 Service 的 DNS 形式进行连接 [https://kubernetes.default.svc.cluster.local：](https://kubernetes.default.svc.cluster.local:/)
![img](http://mindoc.energy-envision.com:8090/uploads/202111/dpi/attach_16b43b7a91f77037.png)
注意 namespace，我们这里填 kube-ops，然后点击 Test Connection，如果出现 Connected to Kubernetes… 的提示信息证明 Jenkins 已经可以和 Kubernetes 系统正常通信了。

然后下方的 Jenkins URL 地址：[http://jenkins.kube-ops.svc.cluster.local:8080，这里的格式为：服务名.namespace.svc.cluster.local:8080，根据上面创建的](http://jenkins.kube-ops.svc.cluster.xn--local:8080%2C:-9f5s960c95eeq7c3n2azpexq6fy44foti.namespace.svc.cluster.local:8080，根据上面创建的/) jenkins 的服务名填写，包括下面的 Jenkins 通道，默认是 50000 端口（要注意是 TCP，所以不要填写 http）：
![img](http://mindoc.energy-envision.com:8090/uploads/202111/dpi/attach_16b43b8304968482.png)
