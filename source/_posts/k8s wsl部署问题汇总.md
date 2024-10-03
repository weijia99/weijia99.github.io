---
title: k8s wsl部署问题汇总
date: '2024-08-24 21:40:04'
updated: '2024-08-24 22:20:01'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/26023389/1724507138184-1854b1ea-de5e-431e-90cd-75da81ff91d1.png'
description: wsl部署k8s，直接安装docker windows进行打开k8s就行，不需要自己遭罪的使用什么脚本来安装deployment部署内容，使用的spec作为container来实现容器部署，container里面进行定义image镜像，副本数量，镜像端口，这个镜像是docker对dockerf...
---
wsl部署k8s，直接安装docker windows进行打开k8s就行，

![](https://raw.githubusercontent.com/weijia99/sync2hexo/master/af104a095f0979fa9f2f93be6c6a7863.png?token=AGECE2MY2W464LBUCB5W473G72WSG)

不需要自己遭罪的使用什么脚本来安装





# deployment
部署内容，使用的spec作为container来实现容器部署，container里面进行定义image镜像，副本数量，镜像端口，这个镜像是docker对dockerfile打包生成的image，可以使用makefile来进行简化流程。

makefile 定义打包流程，dockerfile设置基础镜像，然后把使用交叉编译得到的产物复制到镜像空间里面，之后运行产物。

```yaml
FROM ubuntu:20.04

#复制产物到image
COPY webook /app/webook

WORKDIR /app

#执行
ENTRYPOINT ["/app/webook"]
```



```yaml
.PHONY:docker
docker:
	@rm webook || true
	@GOOS=linux GOARCH=amd64 go build -o webook .
	@docker rmi -f flycash/webook:v0.0.1
	@docker build -t flycash/webook:v0.0.1 .


```

现在是deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webook

#  部署的pod状态
  template:
    metadata:
      labels:
        app: webook
    spec:
      containers:
        - name: webook
          image: flycash/webook:v0.0.1
          ports:
            - containerPort: 8080


```

定义选择的app还有容器使用的景象和端口，还有副本数量

# 持久化层
## pvc与pv
pv是映射本地存放和k8s的虚拟存储的地方

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: webook-mysql-pvc
spec:
  storageClassName: record
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
#    wsl使用的路径是这个需要，这样写才可以,持久化bug
    path: "/run/desktop/mnt/d/code/mysql"
```

:::info
<font style="background-color:#FBDE28;">path: "/run/desktop/mnt/d/code/mysql"  这里需要加入/run/desktop才可以</font>

:::



pvc就相当于代理人，我申请的是什么虚拟存储，需要在这个存储上开辟多少空间，相当于他是操作行为定义。



```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: webook-mysql-pvc
spec:
  storageClassName: record
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

通过使用storage来进行定位是哪一个存储



两个bug地方。

#### <font style="color:rgb(21, 153, 87);">问题：PersistentVolume 失效</font>
```plain
The PersistentVolume "mysqldata" is invalid: spec.persistentvolumesource: Forbidden: is immutable after creation
```

<font style="color:rgb(96, 108, 113);">原因：当原来的PV或PVC还在，而你又创建了一个新的PV, 并与原来的重名，则会得到该错误。</font>

<font style="color:rgb(96, 108, 113);">解决：将原来的PV或PVC删掉，再重新创建新的。</font>

```plain
$ kubectl delete pvc mysqldata -n testnamespace
```

#### <font style="color:rgb(21, 153, 87);">问题：删除pv 、pvc 一直卡着</font>
```plain
$ kubectl patch pv pvname -p '{"metadata":{"finalizers":null}}'
```

