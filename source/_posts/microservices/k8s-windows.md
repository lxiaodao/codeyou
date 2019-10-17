---
title: 开发环境kubernetes在windows基本安装和使用
date: 2019-10-11 16:07:01
tags:
  - docker
  - kubernetes
  - k8s
categories:
  - Microservice
  

---
1.Install Minikube
查看systeminfo
开启虚拟机

[Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install) （windows自带，启用后重启系统）

[VirtualBox](https://www.virtualbox.org/wiki/Downloads)

[下载kubectl.exe](https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe)，并设置PATH，如C:\k8s

[下载minikube-windows](https://github.com/kubernetes/minikube/releases/tag/v1.3.1)，修改名称为minikube.exe，设置PATH，如C:\k8s<br>

禁用Hyper-V，安装VirtualBox

2.启动 minikube start

3.查看docker在miniku内部的环境变量
```
C:\Users\yangliu>minikube docker-env
	SET DOCKER_TLS_VERIFY=1
	SET DOCKER_HOST=tcp://192.168.99.100:2376
	SET DOCKER_CERT_PATH=C:\Users\yangliu\.minikube\certs
	REM Run this command to configure your shell:
	REM @FOR /f "tokens=*" %i IN ('minikube docker-env') DO @%i


C:\Users\yangliu>minikube status
	host: Running
	kubelet: Running
	apiserver: Running
	kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100

C:\Users\yangliu>kubectl get nodes
	NAME       STATUS   ROLES    AGE   VERSION
	minikube   Ready    master   62m   v1.15.2

C:\Users\yangliu>kubectl get pods
	No resources found.

C:\Users\yangliu>kubectl cluster-info
	Kubernetes master is running at https://192.168.99.100:8443
	KubeDNS is running at https://192.168.99.100:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
4.进入minikube内嵌的docker命令行模式

	minikube ssh 
	$docker ps
	$docker image ls

5.单独安装docker后打包image

执行命令 @FOR /f "tokens=*" %i IN ('minikube docker-env') DO @%i后设置临时变量，使得外部安装的docker client能够连接到内嵌的docker host即docker deamon

打包镜像

	docker build -t sayhello:v1.0 .



6.使用创建的镜像运行application

	kubectl run sayhelloapp --image=<br>sayhello:v1.0 <br><br>--port=8090
	c:\workspaces\spring-cloud-demo\say-hello>kubectl run sayhelloapp --image=sayhello:v1.0  --port=8090
	kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
	deployment.apps/sayhelloapp created

7.Deploye the application

	kubectl expose deployment  sayhelloapp --type="LoadBalancer"
	service/sayhelloapp   exposed

kubectl get services 查看已经发布的service

	>kubectl get services
	NAME          TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
	kubernetes    ClusterIP      10.96.0.1        <none>        443/TCP          25h
	sayhelloapp   LoadBalancer   10.106.198.166   <pending>     8090:31580/TCP   36s

查看sayhelloapp运行的地址

	minikube service sayhelloapp --url
	
8.运行多个副本集

	kubectl scale deployment  sayhelloapp --replicas=3
在minikube dashboard中查看
  ![dashborad for appdemo](/images/appdemo.png)

9 集群是否生效

kubectl delete rs sayhelloapp-54864b84c9 
kubectl delete pod sayhelloapp-54864b84c9-l8fmv  --now 
查看到集群信息

	>kubectl get all
	sayhelloapp-54864b84c9-4pbxk   1/1     Running   2          3h10m
	sayhelloapp-54864b84c9-t7cg8   1/1     Running   0          12m
	sayhelloapp-54864b84c9-wkxmq   1/1     Running   1          3h10m

kubectl delete service sayhelloapp 

10.以上普通的删除方式，k8s会自动重新生成repicaSet，pod and container
同时删除部署和服务，则完全删除了部署，服务，replicaSet and pod/sayhelloapp**
```
>kubectl get all
NAME                               READY   STATUS    RESTARTS   AGE
pod/sayhelloapp-54864b84c9-5k498   1/1     Running   0          6m34s
pod/sayhelloapp-54864b84c9-7k4c8   1/1     Running   0          6m34s
pod/sayhelloapp-54864b84c9-vlg8g   1/1     Running   0          6m34s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   28h

NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sayhelloapp   3/3     3            3           3h55m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/sayhelloapp-54864b84c9   3         3         3       6m34s

kubectl delete deployment.apps/sayhelloapp
```
11.重新docker build一个镜像appdemo

	docker build -t appdemo:v1.0 .
    kubectl run  appdemoapp --image=appdemo:v1.0 
	c:\workspaces\appdemo>kubectl delete deployment.apps/appdemoapp service/appdemoapp
	deployment.apps "appdemoapp" deleted
	service "appdemoapp" deleted
	kubectl expose deployment  appdemoapp --type="LoadBalancer" --port=8090 --target-port=8080
	kubectl scale deployment appdemoapp --replicas=3