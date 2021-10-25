# laravel-on-k8s

# 前言

本文介绍如何使用 Kubernetes 在 macOS 部署 Laravel 项目，涉及的具体内容如下：

1. 系统：macOS Monterey
2. Kubernetes 系统：Docker Desktop 的 docker 和 kuberneters
3. 项目：Laravel
4. 模块：nginx，php，php-fpm，composer，mysql，redis
5. [项目代码]([maxlcoder/laravel-docker (github.com)](https://github.com/maxlcoder/laravel-docker)) & [部署代码]([maxlcoder/laravel-on-k8s (github.com)](https://github.com/maxlcoder/laravel-on-k8s))

# 结构

### Kubernetes 结构

![](http://r1it3yop4.hn-bkt.clouddn.com/laravel-on-k8s.jpg)

### 结构说明


1. pod 管理采用 Deployment

> ReplicationController > ReplicaSet > Deployment 这三者的进化过程，可以了解一下。

2. 模拟生产，mysql 和 redis 分别采取独立的 service 进行部署

3. Larval 项目将于 nginx 和 php-fpm 在一个 Pod 进行部署

   > 本测试将 nginx ，php-fpm 集成在一个 Pod ，网上也有很多关于相关结构的讨论，可以自行参考一下，本着的原则就是灵活，高效，避免浪费。

4. 在 nginx 之前使用 ingress 做网络代理 

# 部署步骤

1. 部署 mysql 和 redis

   ```shell
   $ kubectl create -f mysql-deployment-persistentvolumeclaim-service.yaml
   $ kubectl create -f redis-deplyment-service.yaml
   ```

2. 部署配置

   ```shell
   $ kubectl create -f configmap.yaml
   ```

3. 部署项目本身

   ```shell
   $ kubectl create -f laravel-nginx-deployment-service.yaml
   ```

4. 部署 ingress

   ```shell
   $ kubectl create -f ingress.yaml
   ```

5. 调整本地 hosts

   由于是本机，其中项目默认的 IP 是 localhost，所以 ingress 代理时，直接将本地测试域名代理到 localhost 即可，也就是在 /etc/hosts 中增加如下内容
   ```shell
   127.0.0.1      hello.laraveltest.com
   ```

# 测试

1. 访问 `http://hello.laraveltest.com/say?name=Lucky` ，相当于注册用户名，注册完成写入 redis

   ![](http://r1it3yop4.hn-bkt.clouddn.com/Xnip2021-10-25_15-16-04.jpg)

2. 再次访问 `http://hello.laraveltest.com/say?name=Lucky` ，判断已经注册

   ![](http://r1it3yop4.hn-bkt.clouddn.com/Xnip2021-10-25_15-16-20.jpg)

3. 访问 `http://hello.laraveltest.com/hello?name=Lucky` ，判断已经找到

   ![](http://r1it3yop4.hn-bkt.clouddn.com/Xnip2021-10-25_15-16-36.jpg)

4. 访问 `http://hello.laraveltest.com/hello?name=Lucy` ，判断未找到

   ![](http://r1it3yop4.hn-bkt.clouddn.com/Xnip2021-10-25_15-16-50.jpg)

# 备注

当前结构可用，可以正常运行，但是距离生产还需要一些调整，主要是有日志，配置，HPA (自动扩容) 和服务监控等未处理，需要补充。
