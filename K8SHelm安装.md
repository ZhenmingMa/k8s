K8sversion 1.19.2

Docker version  19.03.13

Helm version v3.7.1



### 1 下载

 https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz   helm-v3.7.1-linux-amd64.tar.gz 

### 2 解压

```
tar -zxvf helm-v3.7.1-linux-amd64.tar.gz 
```

### 3 在解压目中找到`helm`程序，移动到需要的目录中

```
mv linux-amd64/helm /usr/local/bin/helm
```

### 4 验证

```
helm version
```

