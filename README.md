# laravel-on-k8s


默认 laravel 项目线上驱动需要 nginx，php & php-fpm, mysql, redis 等组件，当前
这里采取分 Service 构建。将不同的组件 container 拆分到不同的 pod 当中，而非相同的 pod。


1. pod 管理采用 Deployment

> ReplicationController > ReplicaSet > Deployment 这三者的进化过程，可以了解一下。

2. mysql 服务

* mysql-deployment.yaml
* mysql-persistentvolumeclaim.yaml
* mysql-service.yaml

2.1 首先执行 

首先请求 myql 需要的存储卷
```shell
$ kubectl create -f mysql-persistentvolumeclaim.yaml 
```
得到
```shell
$ kubectl get pvc

NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-persistentvolumeclaim   Bound    pvc-457816ea-f43f-4c9a-95ae-8fc52d67b4f4   1Gi        RWO            hostpath       4m58s
```
