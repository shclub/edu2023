# CKA 문제


<br/>

## 1. ETCD Backup & Restore  

<br/>


> Question : ETCD Backup & Restore  

> TASK :  
>  - First, create a snapshop of the existing etcd instance  running at `https://127.0.0.1:2379`, saving the snapshot to `/data/etcd-snapshot.db`.
>  - Next, restore and existing, previous snapshot  located at `/data/etcd-snapshot-previous.db`.  

>  The following TLS certificates/key are supplied for connecting to the server with etcdctl:  
>  - CA certificate : /etc/kubernetes/pki/etcd/ca.crt     
>  - Client certificate : /etc/kubernetes/pki/etcd/server.crt     
>  - Client key : /etc/kubernetes/pki/etcd/server.key       


<br/>

> 답안  

* ssh로 master node에 로그인.    

```bash
root@k8s-master:~# etcd --version
root@k8s-master:~# etcdctl version
``` 

<br/>

* etcd 백업을 한다.     

```bash
root@k8s-master:~# sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
   --cert=/etc/kubernetes/pki/etcd/server.crt \
   --key=/etc/kubernetes/pki/etcd/server.key \
   snapshot save /data/etcd-snapshot.db
```   

<br/>

* etcd restore 를 한다.     

```bash
root@k8s-master:~# sudo ETCDCTL_API=3 etcdctl --data-dir=/var/lib/etcd-new \
   snapshot restore /data/etcd-snapshot-previous.db
``` 

* 좀 있다가 폴더 확인 

```bash
root@k8s-master:~# sudo tree /var/lib/etcd-new
```  

* etcd.yaml 화일을 수정한다.  



```bash
root@k8s-master:~# sudo vi /etc/kubernetes/manifests/etcd.yaml
``` 

<br/>

* 아래 부분의 hostpath를 찾아 `/var/lib/etcd-new` 로 path 변경만 변경한다.  
   
 
```bash
...
 - hostPath:
      path: /var/lib/etcd-new
      type: DirectoryOrCreate
   name: etcd-data
...      
```  

<br/>

## 2. pod 생성하기  


<br/>

> Question : Create a new namespace and create a pod in the namespace 

> TASK :  
>  - namespace name : 본인 namespace
>  - pod name : eshop-main 
>  - image : ghcr.io/shclub/busybox:1.28
>  - env : DB=mysql 


<br/><br/>


> 결과값  

```bash
root@newedu:~# kubectl exec -it eshop-main   -- printenv
DB=mysql
```  

<br/>

> 답안  

* 아래와 같이 생성하고 확인한다. 

```bash
root@newedu:~# kubectl run eshop-main --image=ghcr.io/shclub/busybox:1.28 --env=DB=mysql
pod/eshop-main created
root@newedu:~# kubectl exec -it eshop-main   -- printenv
DB=mysql
``` 

<br/>

## 3. Static POD 생성하기

<br/>

Master Node의 API를 호출 하지 않아 Scheduler 가 생성하지 않는 POD 이며 각 Node의 kubelet ( Daemon ) 이 생성한다.  

<br/>

k8s에는 /var/lib/kubelet/config.yaml 에서 static POD yaml 화일을 위치를 볼수 있고 일반적으로 /etc/kubernetes/manifests 폴더에 있다.    

```bash
root@newedu:~# cat /var/lib/kubelet/config.yaml 
...
staticPodPath: /etc/kubernetes/manifests
...
``` 

<br/>

Master Node 의 POD 들도 Static POD 이고 kubelet 에 의해서 기동된다.  

<br/>

> Question : Configure kubelet hosting to `start a pod on > the node`  

> TASK :  
>  - Node : node1 ( okd 이면  : edu.worker05 )
>  - Pod Name : web
>  - image : ghcr.io/shclub/nginx

<br/><br/>


> 결과값  

 ```bash
 root@newedu:~# kubectl get po -n default
 default                                            web-edu.node01                                                  1/1     Running                0          2m54s
```  

<br/>

> 답안  

* dry-run 를 사용하여 yaml 화일을 생성한다.  

```bash
root@newedu:~# kubectl run web --image=ghcr.io/shclub/nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web
  name: web
spec:
  containers:
  - image: ghcr.io/shclub/nginx
    name: web
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
``` 
<br/>

* ssh로 node1 노드에  로그인하여 /etc/kubernetes/manifests 폴더에 앞에서 생성한 내용을 복사하여 web.yaml 화일을 생성한다. (  CKA 에서는 ssh hostname 명령어로 치면 바로 노드에 접속이 됨 )  
   
```bash
[root@edu ~]# cd /etc/kubernetes/manifests
[root@edu kubernetes]# ls
ca.crt      cni         kubelet-ca.crt   kubelet.conf  static-pod-resources
cloud.conf  kubeconfig  kubelet-plugins  manifests
[root@edu kubernetes]# cd manifests
[root@edu manifests]# vi web.xml
```  

<br/>

pod 이름은  pod yaml 화일에 설정된 이름에 워커노드 이름이 붙는다.  


```bash
root@newedu:~# kubectl get po -n default
default                                            web-edu.node01                                                  1/1     Running                0          2m54s
```  

<br/>

web.xml 화일을 삭제하면 pod는 삭제 된다.    

exit 명령어로 해당 node에서 나간다.  

<br/>

## 4. Multi Container POD 생성하기

<br/>

하나의 POD에 Container를 여러개 생성 할 수 있다.  

<br/>

> Question : Create Pod 

> TASK :  
>  - Create a pod name multi-con with 3 containers running : nginx, redis, memcached  
>  - image : ghcr.io/shclub/nginx , ghcr.io/shclub/redis, ghcr.io/shclub/memcached

<br/><br/>


> 결과값  

```bash
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
multi-con                           3/3     Running   0          31s
root@newedu:~# kubectl describe po multi-con
Name:         multi-con
Namespace:    edu30
Priority:     0
Node:         edu.worker02/172.25.1.50
Start Time:   Fri, 23 Sep 2022 20:47:25 +0900
Labels:       run=multi-con
...
Containers:
  nginx:
    Container ID:   cri-o://8594ed5595f6d8049fd8368473706d147fc1d48ebd2fc53f338393d97de35cdd
    Image:          ghcr.io/shclub/nginx
    Image ID:       docker.io/library/nginx@sha256:79c77eb7ca32f9a117ef91bc6ac486014e0d0e75f2f06683ba24dc298f9f4dd4
    Port:           <none>
...
  redis:
    Container ID:   cri-o://56b3d75c4491df928ef66d1077077c2e3c0bcc12559e62a8b220b1f4bcc46c01
    Image:          ghcr.io/shclub/redis
    Image ID:       docker.io/library/redis@sha256:b4e56cd71c74e379198b66b3db4dc415f42e8151a18da68d1b61f55fcc7af3e0
...
  memcached:
    Container ID:   cri-o://0e11b71e73894b5b69bba208eb7dc9c669dc635af982fe8121b02806f50ab5e3
    Image:          ghcr.io/shclub/memcached
    Image ID:       docker.io/library/memcached@sha256:324479339ef09ec98467d7c0a8083909503e7af3095390614990af0fee781c7b
...
  Normal  Pulling         2m26s  kubelet            Pulling image "nginx"
  Normal  Pulled          2m22s  kubelet            Successfully pulled image "ghcr.io/shclub/nginx" in 4.018533684s
  Normal  Created         2m22s  kubelet            Created container nginx
  Normal  Started         2m22s  kubelet            Started container nginx
  Normal  Pulling         2m22s  kubelet            Pulling image "redis"
  Normal  Pulled          2m12s  kubelet            Successfully pulled image "ghcr.io/shclub/redis" in 10.541276692s
  Normal  Started         2m11s  kubelet            Started container redis
  Normal  Pulling         2m11s  kubelet            Pulling image "memcached"
  Normal  Created         2m11s  kubelet            Created container redis
  Normal  Pulled          2m2s   kubelet            Successfully pulled image "ghcr.io/shclub/memcached" in 9.577820141s
  Normal  Created         2m2s   kubelet            Created container memcached
  Normal  Started         2m2s   kubelet            Started container memcached
```  

<br/>

> 답안  

* dry-run 를 사용하여 yaml 화일을 생성한다.  

```bash
root@newedu:~# kubectl run multi-con --image=ghcr.io/shclub/nginx --dry-run=client -o yaml > multi-con.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-con
  name: multi-con
spec:
  containers:
  - image: ghcr.io/shclub/nginx
    name: multi-con
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
``` 
<br/>

multi-con.yaml 화일을 vi 에디터로 오픈하여 컨테이너를 아래와 같이 추가한다. 
nginx 는 컨테이너 이름을 multi-con에서 nginx로 변경한다.  


multi-con.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-con
  name: multi-con
spec:
  containers:
  - image: ghcr.io/shclub/nginx
    name: nginx
  - image: ghcr.io/shclub/redis
    name: redis
  - image: ghcr.io/shclub/memcached
    name: memcached    
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
``` 

<br/>

multi-con.yaml 화일을  적용하여 POD를 생성한다.  


```bash
root@newedu:~# kubectl apply -f multi-con.yaml 
```   

<br/>

pod 를 조회하면 READY 3/3으로 되어 있는것을 확인 할 수 있다. 

```bash
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
multi-con                           3/3     Running   0          31s
root@newedu:~# kubectl describe po multi-con
Name:         multi-con
Namespace:    edu30
Priority:     0
Node:         edu.worker02/172.25.1.50
Start Time:   Fri, 23 Sep 2022 20:47:25 +0900
Labels:       run=multi-con
...
Containers:
  nginx:
    Container ID:   cri-o://8594ed5595f6d8049fd8368473706d147fc1d48ebd2fc53f338393d97de35cdd
    Image:          ghcr.io/shclub/nginx
    Image ID:       docker.io/library/nginx@sha256:79c77eb7ca32f9a117ef91bc6ac486014e0d0e75f2f06683ba24dc298f9f4dd4
    Port:           <none>
...
  redis:
    Container ID:   cri-o://56b3d75c4491df928ef66d1077077c2e3c0bcc12559e62a8b220b1f4bcc46c01
    Image:          ghcr.io/shclub/redis
    Image ID:       docker.io/library/redis@sha256:b4e56cd71c74e379198b66b3db4dc415f42e8151a18da68d1b61f55fcc7af3e0
...
  memcached:
    Container ID:   cri-o://0e11b71e73894b5b69bba208eb7dc9c669dc635af982fe8121b02806f50ab5e3
    Image:          ghcr.io/shclub/memcached
    Image ID:       docker.io/library/memcached@sha256:324479339ef09ec98467d7c0a8083909503e7af3095390614990af0fee781c7b
...
  Normal  Pulling         2m26s  kubelet            Pulling image "nginx"
  Normal  Pulled          2m22s  kubelet            Successfully pulled image "nginx" in 4.018533684s
  Normal  Created         2m22s  kubelet            Created container nginx
  Normal  Started         2m22s  kubelet            Started container nginx
  Normal  Pulling         2m22s  kubelet            Pulling image "redis"
  Normal  Pulled          2m12s  kubelet            Successfully pulled image "redis" in 10.541276692s
  Normal  Started         2m11s  kubelet            Started container redis
  Normal  Pulling         2m11s  kubelet            Pulling image "memcached"
  Normal  Created         2m11s  kubelet            Created container redis
  Normal  Pulled          2m2s   kubelet            Successfully pulled image "memcached" in 9.577820141s
  Normal  Created         2m2s   kubelet            Created container memcached
  Normal  Started         2m2s   kubelet            Started container memcached
                 0          2m54s
``` 

<br/>


## 5. Side-car container POD 실행

<br/>

기존 POD에 로그 분석을 위한 별도 Container 를 Side Car 로 추가할 수 있다.

<br/>

> Question : An existing Pod needs to be integrated into the K8s built-in logging architecture (e.g. `kubectl logs` )  
> Adding a streaming `sidecar` container is good and common way to accomplish this requirement.  

> TASK :  
>  - Add a sidecard container named `sidecar`, using `ghcr.io/shclub/busybox` image, to the existing Pod `eshop-cart-app`.  
>  - The new sidecar container has to run the following command: /bin/sh -c 'tail -n +1 -F /var/log/cart-app.log'  
>  - Use a Volume mounted at `/var/log`, to make the log file `cart-app.log` avaiable to the sidecar container.  
>  - `Don't modify the eshop-cart-app`

<br/><br/>

일단 기동 중인  eshop-cart-app 라는 pod에서 yaml 화일을 받아 옵니다.  
( 우리는 현재 기동한 POD가 없기 때문에 아래 yaml를 사용한다. )  

- kubectl get po eshop-cart-app -o yaml > eshop.yaml  

<br/>

eshop.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: eshop-cart-app
  name: eshop-cart-app
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - 'i=1;while :;do echo -e "$i: price: $((RANDOM % 10000 +1))" >> /var/log/cart-app.log; i=$((i+1)); sleep 2;done'
    image: ghcr.io/shclub/busybox
    name: eshop-cart-app
    volumeMounts:
    - mountPath: /var/log
      name: varlog
  volumes:
  - emptyDir: {}
    name: varlog
```  

> 결과값  

```bash
root@newedu:~# kubectl get po 
NAME                                READY   STATUS    RESTARTS   AGE
eshop-cart-app                      2/2     Running   0          11s
```  

<br/>

sidecar 컨테이너에서  로그 정보가 나와야 한다.  

```bash
root@newedu:~# kubectl logs eshop-cart-app -c sidecar
1: price: 9448
2: price: 5839
3: price: 7135
4: price: 9062
5: price: 2709
6: price: 348
7: price: 9538
8: price: 356
9: price: 5786
...
```  

<br/>


> 답안  

* eshop.yaml 화일에 sidecar 컨테이너를 추가한다.  
  
```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: eshop-cart-app
  name: eshop-cart-app
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - 'i=1;while :;do echo -e "$i: price: $((RANDOM % 10000 +1))" >> /var/log/cart-app.log; i=$((i+1)); sleep 2;done'
    image: ghcr.io/shclub/busybox
    name: eshop-cart-app
    volumeMounts:
    - mountPath: /var/log
      name: varlog
  - name : sidecar
    image: ghcr.io/shclub/busybox
    args: [/bin/sh, -c,'tail -n +1 -F /var/log/cart-app.log']
    volumeMounts:
    - mountPath: /var/log
      name: varlog
  volumes:
  - emptyDir: {}
    name: varlog
``` 

<br/>

기존 Pod를 삭제한다.   

```bash
root@newedu:~# kubectl delete po eshop-cart-app
```  

<br/>

eshop.yaml 화일을 적용하여 POD를 생성한다.  


```bash
root@newedu:~# kubectl apply -f eshop.yaml 
```   

<br/>

pod 를 조회하면 READY 2/2으로 되어 있는것을 확인 할 수 있다.   

```bash
root@newedu:~# kubectl get po 
NAME                                READY   STATUS    RESTARTS   AGE
eshop-cart-app                      2/2     Running   0          11s
```    

<br/>

sidecar 컨테이너에서  로그 정보가 나와야 한다.  

```bash
root@newedu:~# kubectl logs eshop-cart-app -c sidecar
1: price: 9448
2: price: 5839
3: price: 7135
4: price: 9062
5: price: 2709
6: price: 348
7: price: 9538
8: price: 356
9: price: 5786
...
```  

<br/>


## 6. Deployment & Pod Scale

<br/>

### 1. Pod scale out

<br/>

> Question : Expand the number of running Pods in `eshop-order`  from `1` to `5`.

> TASK :  
>  - namespace : 본인 namespace  
>  - deployment : eshop-order
>  - image : ghcr.io/shclub/nginx

<br/><br/>

아래 yaml 화일을 실행하여 deployment 를 생성합니다.  
- root@newedu:~# kubectl create deployment eshop-order --image=ghcr.io/shclub/nginx --replicas=1 

<br/>

```bash
root@newedu:~# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
eshop-order        1/1     1            1           7s
```  

> 결과값  : pod가 5개가 생성되었는지 확인한다.

```bash
root@newedu:~# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
eshop-order        5/5     5            5           46s
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
eshop-order-ff866c56b-8k8cq         1/1     Running   0          14s
eshop-order-ff866c56b-dcfr9         1/1     Running   0          14s
eshop-order-ff866c56b-fh644         1/1     Running   0          14s
eshop-order-ff866c56b-ltkg6         1/1     Running   0          58s
eshop-order-ff866c56b-q9gd2         1/1     Running   0          14s
```  

<br/>


> 답안  

* 아래 명령어를 실행한다.    
  
```bash
root@newedu:~# kubectl scale deployment eshop-order --replicas=5
deployment.apps/eshop-order scaled
``` 

<br/>

pod가 5개가 생성되었는지 확인한다.  
  

```bash
root@newedu:~# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
eshop-order        5/5     5            5           46s
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
eshop-order-ff866c56b-8k8cq         1/1     Running   0          14s
eshop-order-ff866c56b-dcfr9         1/1     Running   0          14s
eshop-order-ff866c56b-fh644         1/1     Running   0          14s
eshop-order-ff866c56b-ltkg6         1/1     Running   0          58s
eshop-order-ff866c56b-q9gd2         1/1     Running   0          14s
```


<br/>


### 2. Deployment 생성하고 Scaling 하기

<br/>

> Question : Create a deployment as follows. 

> TASK :  
>  - name : `webserver` 
>  - `2` replicas
>  - label : `app_env_stage=dev`
>  - container name : `webserver`
>  - container image : `ghcr.io/shclub/nginx:1.14`  


>  Scale Out Deployment.  
>  - Scale the deployment webserver to `3` pods

<br/><br/>

아래 명령어를 실행하여 deployment 를 생성하고 label를 설정합니다.    
- root@newedu:~# kubectl create deployment webserver --image=ghcr.io/shclub/nginx:1.14 --replicas=2  

<br/>

```bash
root@newedu:~# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
eshop-order        1/1     1            1           7s
```  

> 결과값  : webserver 이름으로 deployment 생성 ( Replica : 2 )

```bash
root@newedu:~# kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
webserver          2/2     2            2           5m29s
```  

<br/>

replicas 3 으로 증가.  

```bash
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
webserver-5586594bbf-kqkfr          1/1     Running   0          3m47s
webserver-5586594bbf-td2z5          1/1     Running   0          3m47s
webserver-5586594bbf-vpf5j          1/1     Running   0          7s
```  

<br/>


> 답안  

* 아래 명령어를 실행한다.    
  
```bash
root@newedu:~# kubectl create deployment webserver --image=ghcr.io/shclub/nginx:1.14 --replicas=2  --dry-run=client -o yaml > webserver.yaml
``` 

<br/>

* webserver.yaml 를 문제에 맞게 수정한다.    
  
```bash
root@newedu:~# vi webserver.yaml
```  

<br/>

label과 container 이름을 수정  

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: webserver
  name: webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      app_env_stage: dev
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app_env_stage: dev
    spec:
      containers:
      - image: ghcr.io/shclub/nginx:1.14
        name: webserver
```  

<br/>

webserver 라는 이름 으로 deployment를 생성한다.  

```bash
root@newedu:~# kubectl apply -f webserver.yaml
deployment.apps/webserver created
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
webserver-5586594bbf-kqkfr          1/1     Running   0          2m
webserver-5586594bbf-td2z5          1/1     Running   0          2m
```  

<br/>

replica 를 3 으로 조정한다.  

```bash
root@newedu:~# kubectl scale deployment webserver --replicas=3
deployment.apps/webserver scaled
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
webserver-5586594bbf-kqkfr          1/1     Running   0          3m47s
webserver-5586594bbf-td2z5          1/1     Running   0          3m47s
webserver-5586594bbf-vpf5j          1/1     Running   0          7s
```

<br/>


## 7. Rolling Update

<br/>

> Question : Create a deployment as follows. 

> TASK :  
>  - name : `nginx-app`  
>  - Using container nginx with version : `ghcr.io/shclub/nginx:1.14-alpine`
>  - The deployment should contain `3` replicas
>  Next, deploy the application with new version `ghcr.io/shclub/nginx:1.14.2-alpine`, by performing a rolling update   
>  Finally, rollback that update to the previous version `ghcr.io/shclub/nginx:1.14-alpine`

<br/><br/>

> 결과값  : deployment 와 rollout history를 확인 합니다.  


```bash
root@newedu:~# kubectl rollout  history deployment nginx-app
deployment.apps/nginx-app
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```  


<br/>


> 답안  

* 아래 명령어를 실행한다.    
  
```bash
root@newedu:~# kubectl create deployment nginx-app --image=ghcr.io/shclub/nginx:1.14-alpine --replicas=3 
deployment.apps/nginx-app created

root@newedu:~# kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app          3/3     3            3           14s
```  

<br/>

image를 nginx:1.14.2-alpine 으로 변경합니다.    

kubectl set image deployments <deployment 이름> <컨테이너이름>=<변경할 이미지>. 

<br/>


```bash
root@newedu:~# kubectl set image deployment nginx-app nginx=ghcr.io/shclub/nginx:1.14.2-alpine
deployment.apps/nginx-app image updated
```  

<br/>

status를 확인합니다.  순서대로 배포가 진행 됩니다.  


```bash
root@newedu:~# kubectl rollout status deployment nginx-app
Waiting for deployment "nginx-app" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-app" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-app" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "nginx-app" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-app" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-app" successfully rolled out
```  

<br/>

rollout history를 확인하면 revision이 2가 생긴 것을 확인 할 수 있습니다.  

```bash
root@newedu:~# kubectl rollout history deployment nginx-app
deployment.apps/nginx-app
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
``` 

<br/>

POD 중 하나를 describe 로 확인하면 이미지가  `ghcr.io/shclub/nginx:1.14.2-alpine` 인 것을 확인 할 수 있습니다.  


```bash
root@newedu:~# kubectl describe po nginx-app-7dd7d6949b-547hx
Name:         nginx-app-7dd7d6949b-547hx
...
Status:       Running
IP:           10.130.6.220
IPs:
  IP:           10.130.6.220
Controlled By:  ReplicaSet/nginx-app-7dd7d6949b
Containers:
  nginx:
    Container ID:   cri-o://8db6eb1dc40a4c83278a943cfebe25f24b19a71ed98b2da18f9da94177cc6cd8
    Image:          ghcr.io/shclub/nginx:1.14.2-alpine
    Image ID:       docker.io/library/
    ...
```

<br/>

undo 를 사용하여 rollback 합니다.  

```bash
root@newedu:~# kubectl rollout undo  deployment nginx-app
deployment.apps/nginx-app rolled back
```  

다시 history 를 조회해 보면 revision 3 이 생성된 것을 볼수 있습니다.  

```bash
root@newedu:~# kubectl rollout  history deployment nginx-app
deployment.apps/nginx-app
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

root@newedu:~# kubectl get po | grep nginx-app
nginx-app-6c54bf76bd-8hv9v          1/1     Running   0          89s
nginx-app-6c54bf76bd-hdx9n          1/1     Running   0          87s
nginx-app-6c54bf76bd-s286x          1/1     Running   0          92s
```  

<br/>

describe 명령어로 image 가 변경 된 것을 확인합니다.  

```bash
root@newedu:~# kubectl get po
NAME                                READY   STATUS    RESTARTS   AGE
nginx-app-6c54bf76bd-8hv9v          1/1     Running   0          3m43s
nginx-app-6c54bf76bd-hdx9n          1/1     Running   0          3m41s
nginx-app-6c54bf76bd-s286x          1/1     Running   0          3m46s

root@newedu:~# kubectl describe po nginx-app-6c54bf76bd-8hv9v
Name:         nginx-app-6c54bf76bd-8hv9v
...
IP:           10.130.6.222
IPs:
  IP:           10.130.6.222
Controlled By:  ReplicaSet/nginx-app-6c54bf76bd
Containers:
  nginx:
    Container ID:   cri-o://d5c75f3f86e1544e95770bedf13a5a223e137d5eed3cae7b7ac12cd61676ba55
    Image:          ghcr.io/shclub/nginx:1.14-alpine
    Image ID:       docker.io/library/
    ...
 ```  

<br/>

## 8. Node Selector

<br/>

> Question : Schedule a pod as follows. 

> TASK :  
>  - name : `eshop-store`  
>  - namespace : 본인 namespace
>  - image : `ghcr.io/shclub/nginx`
>  - Node Selector :  `disktype=ssd` ( 특정 노드가 세팅 되어 있고 특정 노드에만 pod를 띄운다 )

<br/><br/>

> 결과값  : node의 label를 확인 하고 pod 생성.  


node별로 disktype label 확인.  

<br/>

OKD 환경. 


```bash
root@newedu:~# kubectl get nodes -L disktype 
NAME               STATUS   ROLES    AGE   VERSION                DISKTYPE
edu.dmz-infra01    Ready    worker   97d   v1.20.0+bafe72f-1054
edu.dmz-infra02    Ready    worker   97d   v1.20.0+bafe72f-1054
edu.master01       Ready    master   98d   v1.20.0+bafe72f-1054
edu.master02       Ready    master   98d   v1.20.0+bafe72f-1054
edu.master03       Ready    master   98d   v1.20.0+bafe72f-1054
edu.monitoring01   Ready    worker   97d   v1.20.0+bafe72f-1054
edu.monitoring02   Ready    worker   97d   v1.20.0+bafe72f-1054
edu.worker01       Ready    worker   97d   v1.20.0+bafe72f-1054
edu.worker01       Ready    worker   97d   v1.20.0+bafe72f-1054
...
edu.worker07       Ready    worker   97d   v1.20.0+bafe72f-1054   ssd
...
```  

<br/>

k8s cka 환경.    


```bash
ubuntu@master:~$ kubectl get nodes -L disktype
NAME     STATUS   ROLES           AGE   VERSION   DISKTYPE
master   Ready    control-plane   32h   v1.25.3
node1    Ready    <none>          32h   v1.25.3   ssd
node2    Ready    <none>          32h   v1.25.3   std
``` 


<br/>

pod가 특정 node에서 실행이 됨.  

```bash
root@newedu:~# kubectl get po eshop-store -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
eshop-store   1/1     Running   0          16s   10.131.6.100   edu.worker07   <none>           <none>
```  


<br/>


> 답안  

* 아래 명령어를 실행한다.   nodes를 조회하여 Label  disktype 인 node를 찾는다.   

  시험 환경에서는 ssd로 세팅된 node를 찾을 수 있다.   

  참고 : https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/  

<br/>

okd 환경  



```bash
root@newedu:~# kubectl get nodes -L disktype 
NAME               STATUS   ROLES    AGE   VERSION                DISKTYPE
edu.dmz-infra01    Ready    worker   97d   v1.20.0+bafe72f-1054
edu.dmz-infra02    Ready    worker   97d   v1.20.0+bafe72f-1054
edu.master01       Ready    master   98d   v1.20.0+bafe72f-1054
edu.master02       Ready    master   98d   v1.20.0+bafe72f-1054
edu.master03       Ready    master   98d   v1.20.0+bafe72f-1054
edu.monitoring01   Ready    worker   97d   v1.20.0+bafe72f-1054
edu.monitoring02   Ready    worker   97d   v1.20.0+bafe72f-1054
edu.worker01       Ready    worker   97d   v1.20.0+bafe72f-1054
edu.worker01       Ready    worker   97d   v1.20.0+bafe72f-1054
...
edu.worker07       Ready    worker   97d   v1.20.0+bafe72f-1054   ssd
...
```  

<br/>

k8s cka 환경.    

```bash
ubuntu@master:~$ kubectl get nodes -L disktype
NAME     STATUS   ROLES           AGE   VERSION   DISKTYPE
master   Ready    control-plane   32h   v1.25.3
node1    Ready    <none>          32h   v1.25.3   ssd
node2    Ready    <none>          32h   v1.25.3   std
```

<br/>

dry-run으로 node_selector.yaml 화일을 생성한다.  


```bash
root@newedu:~# kubectl run eshop-store --image=ghcr.io/shclub/nginx  --dry-run=client -o yaml > node_selector.yaml
```  

아래 내용을 추가한다.  

```bash
nodeSelector:
    disktype: ssd
```  

위에 내용으로 추가하고 실행한다.  
(단. namespace에 node selector 가 걸려 있으면 먼저 삭제 한다. )

```bash
root@newedu:~# vi node_selector.yaml
root@newedu:~# cat node_selector.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: eshop-store
  name: eshop-store
spec:
  containers:
  - image: ghcr.io/shclub/nginx
    name: eshop-store
    resources: {}
  nodeSelector:
    disktype: ssd
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
root@newedu:~# kubectl apply  -f node_selector.yaml
pod/eshop-store created
```  

pod가 특정 node에서 실행이 됨.  

```bash
root@master:~/devops# kubectl get po -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP                NODE    NOMINATED NODE   READINESS GATES
eshop-store   1/1     Running   0          37s   192.168.166.154   node1   <none>           <none>
```  

<br/>


## 9. Node 관리

<br/>

cordon : 해당 노드에 pod scheduling 불가.

<br/>

```bash
root@master:~# kubectl get nodes
NAME     STATUS   ROLES           AGE     VERSION
master   Ready    control-plane   7d16h   v1.25.3
node1    Ready    <none>          7d16h   v1.25.3
node2    Ready    <none>          7d16h   v1.25.3
...
root@master:~# kubectl cordon node1
node/node1 cordoned
root@master:~# kubectl get  nodes
NAME     STATUS                     ROLES           AGE     VERSION
master   Ready                      control-plane   7d16h   v1.25.3
node1    Ready,SchedulingDisabled   <none>          7d16h   v1.25.3
node2    Ready                      <none>          7d16h   v1.25.3
```  

<br/>

uncordon : 해당 노드에 pod scheduling 가능.

<br/>

drain : 해당 노드의 모든 pod를 다른 node로 보내고 현재 노드에 스케쥴링 하지 말아라  

<br/>

```bash
root@master:~# kubectl drain node1
node/node1 already cordoned
error: unable to drain node "node1" due to error:[cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): calico-system/calico-node-pdvl9, calico-system/csi-node-driver-brjbc, kube-system/kube-proxy-ht2sb, cannot delete Pods declare no controller (use --force to override): devops/eshop-store, devops/web, devops/web2, devops/web4, edu1/eshop-main, edu3/eshop-main, edu4/eshop-main], continuing command...
There are pending nodes to be drained:
 node1
cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): calico-system/calico-node-pdvl9, calico-system/csi-node-driver-brjbc, kube-system/kube-proxy-ht2sb
cannot delete Pods declare no controller (use --force to override): devops/eshop-store, devops/web, devops/web2, devops/web4, edu1/eshop-main, edu3/eshop-main, edu4/eshop-main
```  

<br/>


위에서 보면 정상적으로 되어 있지 않을것을 볼 수 있다.
daemonset 은 죽이지 못해서 나는 에러이고 이것을 무시하는 명령어를  포함하여 다시 수행한다.  

cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore)

<br/>

```bash
root@master:~# kubectl drain node1 --ignore-daemonsets
node/node1 already cordoned
error: unable to drain node "node1" due to error:cannot delete Pods declare no controller (use --force to override): devops/eshop-store, devops/web, devops/web2, devops/web4, edu1/eshop-main, edu3/eshop-main, edu4/eshop-main, continuing command...
There are pending nodes to be drained:
 node1
cannot delete Pods declare no controller (use --force to override): devops/eshop-store, devops/web, devops/web2, devops/web4, edu1/eshop-main, edu3/eshop-main, edu4/eshop-main
```  

<br/>

추가적으로 강제 옵션을 주고 실행을 하면 정상적으로 수행이 된다.    

```bash
root@master:~# kubectl drain node1 --ignore-daemonsets --force
error: unknown flag: --ignore-daemonsets
See 'kubectl drain --help' for usage
node/node1 already cordoned
Warning: ignoring DaemonSet-managed Pods: calico-system/calico-node-pdvl9, calico-system/csi-node-driver-brjbc, kube-system/kube-proxy-ht2sb; deleting Pods that declare no controller: devops/eshop-store, devops/web, devops/web2, devops/web4, edu1/eshop-main, edu3/eshop-main, edu4/eshop-main
evicting pod calico-system/calico-typha-c5f6b8544-vswc7
evicting pod edu4/eshop-main
evicting pod calico-apiserver/calico-apiserver-bf7fb56f6-nphl7
evicting pod devops/web
evicting pod devops/eshop-store
evicting pod devops/nginx-app-5d98f7444d-8k5p8
evicting pod devops/nginx-app-5d98f7444d-p5jwr
evicting pod devops/web4
evicting pod devops/web2
evicting pod edu1/eshop-main
evicting pod edu3/eshop-main
pod/calico-typha-c5f6b8544-vswc7 evicted
I1031 17:11:19.884568  836242 request.go:682] Waited for 1.007397548s due to client-side throttling, not priority and fairness, request: GET:https://192.168.0.150:6443/api/v1/namespaces/devops/pods/eshop-store
pod/eshop-store evicted
pod/web evicted
pod/eshop-main evicted
pod/nginx-app-5d98f7444d-p5jwr evicted
pod/web4 evicted
pod/calico-apiserver-bf7fb56f6-nphl7 evicted
pod/eshop-main evicted
pod/eshop-main evicted
pod/nginx-app-5d98f7444d-8k5p8 evicted
pod/web2 evicted
node/node1 drained
```  

<br/>

node를 조회해 본다.  


```bash
root@master:~# kubectl get nodes
NAME     STATUS                     ROLES           AGE     VERSION
master   Ready                      control-plane   7d16h   v1.25.3
node1    Ready,SchedulingDisabled   <none>          7d16h   v1.25.3
node2    Ready                      <none>          7d16h   v1.25.3
```  

<br/>

pod를 조회해 보면 모든 pod가 node2에서 실행 되는것을 볼수 있다.  

```bash
root@master:~# kubectl get po -o wide -n devops
NAME                         READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
eshop-cart-app               2/2     Running   0          20m     192.168.104.16   node2   <none>           <none>
eshop-main                   1/1     Running   0          6d22h   192.168.104.7    node2   <none>           <none>
nginx-app-5d98f7444d-8g9tb   1/1     Running   0          5m51s   192.168.104.21   node2   <none>           <none>
nginx-app-5d98f7444d-nq97z   1/1     Running   0          14m     192.168.104.18   node2   <none>           <none>
nginx-app-5d98f7444d-sx8wk   1/1     Running   0          5m52s   192.168.104.20   node2   <none>           <none>
web3                         1/1     Running   0          6d18h   192.168.104.11   node2   <none>           <none>
```  

<br/>

다시 원복 하기 위해서는 uncordon 명령어를 사용한다. 

```bash
root@master:~# kubectl uncordon node1
node/node1 uncordoned
root@master:~# kubectl get nodes
NAME     STATUS   ROLES           AGE     VERSION
master   Ready    control-plane   7d16h   v1.25.3
node1    Ready    <none>          7d16h   v1.25.3
node2    Ready    <none>          7d16h   v1.25.3
```  

<br/>


> Question : Set the node named  `node1` as `unavailable` and `reschedule` all the pods running on it.  

<br/><br/>

> 결과값  : drain 명령어 사용

```bash
root@newedu:~# kubectl get nodes
NAME               STATUS                     ROLES    AGE   VERSION
node1       Ready,SchedulingDisabled   worker   98d   v1.20.0+bafe72f-1054
```  

<br/>


> 답안  

* 아래 명령어를 실행한다.  특정 노드를 선택하고  3개의 옵션을 다 수행한다.  

- --ignore-daemonsets : daemonset 무시
- --force to override : 강제 overide
- --delete-emptydir-data : storage 관련 무시  

<br/>

```bash
root@newedu:~# kubectl drain node1 --ignore-daemonsets --force to override --delete-emptydir-data
```  

<br/>


## 10. Node 정보 수집

<br/>

### 1. Check Ready Nodes  

> Question : Check to see how many nodes are `ready` ( not including nodes tainted NoSchedule) and `write the number` to `./RN0001`. 


<br/><br/>

> 결과값  : /RN0001 파일을 열어 본다. 2가 나와야 함.  


```bash
root@newedu:~# cat ./RN0001
2
```  

<br/>


> 답안  

* 아래 명령어를 실행한다.   nodes를 조회하여 ready 인 node 확인  

<br/>

```bash
ubuntu@master:~$ kubectl get nodes | grep -i -w ready
master   Ready    control-plane   32h   v1.25.3
node1    Ready    <none>          32h   v1.25.3
node2    Ready    <none>          32h   v1.25.3
```  

<br/>

decribe 명령어로 NoSchedule 이 있는 node를 검색한다.  
master node 노드가 대상이다.  

```bash
ubuntu@master:~$ kubectl describe  nodes | grep -i NoSchedule
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```  

<br/>

describe 명령어로 taints 이 있는 node를 검색한다.  

worker node 는 taints 가 안걸려  있는것을 추측 할수 있다.  
- 각각의 노드별로 확인 필요.  

```bash
ubuntu@master:~$ kubectl describe  nodes | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Taints:             <none>
Taints:             <none>
```  

<br/>

NoSchedule Node가 2개 이기 때문에 아래와 같이 화일 생성.  

```bash
root@newedu:~# echo "2"  > ./RN0001
```  

<br/>


<br/>

### 2. Count the Number of Nodes That Are Ready to Run Normal Workloads  

> Question : Determine `how many nodes in the cluster` and `ready` to run normal workloads (i.e, workloads that do not have any special tolerations ).  

>  Output this number to the file `write the number` to `./NODE-count`. 


<br/><br/>

> 결과값  : /NODE-count 파일을 열어 본다. 3이 나와야 함.  


```bash
ubuntu@master:~$ cat ./NODE-count
3
```  

<br/>


> 답안  

* 아래 명령어를 실행한다.   nodes를 조회하여 ready 인 node 확인  

<br/>

```bash
ubuntu@master:~$ kubectl get nodes | grep -i -w ready
master   Ready    control-plane   32h   v1.25.3
node1    Ready    <none>          32h   v1.25.3
node2    Ready    <none>          32h   v1.25.3
```  

<br/>

wc -l 명령어를 사용하여 카운트 한다.  


```bash
ubuntu@master:~$ kubectl get nodes | grep -i -w ready | wc -l
3
ubuntu@master:~$ kubectl get nodes | grep -i -w ready | wc -l > ./NODE-count
ubuntu@master:~$ cat ./NODE-count
3
```  

<br/>


## 11. Deployment & Expose the Service

<br/>

> Question : Reconfigure the existing deployment `front-end` and `add a port specification named http` exposing port `80/tcp` of the  existing  container nginx.  

> Create a new service named `front-end-svc` exposing the container port `http`.  

> Configure the new service to also expose the individual Pods via a `NodePort` on the nodes on which they are scheduled.


<br/>

먼저 아래 yaml 화일을 deployment를 생성해 놓는다.    

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: front-end
  name: front-end
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - image: ghcr.io/shclub/nginx
        name: nginx
```  

<br/>

deployment 가  실행되어 있는 것을 확인한다.  

```bash
root@newedu:~# kubectl get deploy
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
front-end          2/2     2            2           8s
```
<br/><br/>

>  결과값  : svc를 조회 해 보고 서비스 이름과 NodePort가 생성되어 있는지 확인.  

<br/>


service를  조회 해 보면 NodePort가 생성된 것을 확인 할 수있다.  

```bash
root@newedu:~# kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
front-end-svc               NodePort    172.30.145.50    <none>        80:32495/TCP     2m41s```  
```  

<br/>

curl 명령어로 해당 서비스를 호출해 본다.    

아래와 같이 niginx 첫 화면 내용이 나오면 성공.  

```bash
root@newedu:~# curl 211.34.231.84:32495
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```  


<br/>


> 답안  

* deploy 에서 yaml 화일을 추출한다. 

<br/>

```bash
root@newedu:~# kubectl get deploy front-end -o yaml > deploy.yaml
```  

<br/>

deploy.yaml 화일을 수정하여 포트의 이름을 http로 추가한다.    

```bash
       ports:
        - name: http
          containerPort: 80
```  

<br/>

label와 selector를 확인한다.  

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: front-end
  name: front-end
spec:
  replicas: 2
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - image: ghcr.io/shclub/nginx
        name: http
        ports:
        - name: http
          containerPort: 80
```  

<br/>

서비스를 생성하기 위해 아래 내용을 추가한다.  
- 이름 변경 : front-end-svc  
- targetPort 이름에는 deployment 의 port 이름을 써준다 ( containerPort 를 가르킴 ) 
- type 은 NodePort

```bash
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: front-end
  name: front-end-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: http
    name: http
  selector:
    app: front-end
  type: NodePort
status:
  loadBalancer: {}
```  

<br/>

기존의 deployment 를 삭제하고 새로운 화일로 deploy,svc를 생성한다.  

전체 화일 내용 ( deploy.yaml )  

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: front-end
  name: front-end
spec:
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - image: ghcr.io/shclub/nginx
        name: http
        ports:
        - name: http
          containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: front-end
  name: front-end-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: http
    name: http
  selector:
    app: front-end
  type: NodePort
status:
  loadBalancer: {}
```

```bash
root@newedu:~# kubectl delete deploy front-end
deployment.apps "front-end" deleted
root@newedu:~# kubectl apply -f deploy.yaml
deployment.apps/front-end created
service/front-end-svc created
```  

<br/>

service를  조회 해 보면 NodePort가 생성된 것을 확인 할 수있다.  


```bash
root@newedu:~# kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
front-end-svc               NodePort    172.30.145.50    <none>        80:32495/TCP     2m41s```  
```  
<br/>

curl 명령어로 해당 서비스를 호출해 본다.  아래와 같이 niginx 첫 화면 내용이 나오면 성공.  

```bash
root@newedu:~# curl 211.34.231.84:32495
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```  

<br/>


## 12. Pod log 추출

<br/>

Pod Log를 stand out 으로 추출.

> Question : Monitor the logs of pod `custom-app` and Extract log lines corresponding to error `file not found`. Write them  to `./podlog`.


<br/><br/>

>  결과 및 답안  : kubectl logs로 조회한 후 저장  

<br/>


아래 명령어 사용  

```bash
root@newedu:~# kubectl logs custom-app | grep 'file not found' >  ./podlog 
```  

<br/>

## 13. CPU 사용량이 높은 Pod 검색

<br/>


> Question : From the pod label `name=overloaded-cpu`, find pods running high CPU workloads and write the name of the pod consuming most CPU the file `./cpu_load_pod.txt`.


<br/><br/>

>  결과 및 답안  : kubectl top 로 조회한 후 저장  

<br/>


아래 명령어 사용  

```bash
root@newedu:~# kubectl top po -l name=overloaded-cpu --sort-by=cpu 
root@newedu:~# echo "POD_NAME" >  ./cpu_load_pod.txt 
```  

<br/>

## 14. init 컨테이너를 포함한 Pod 운영

<br/>

init 컨테이너는 main container 가 동작 되기전에 실행되는 컨테이너
- 주로 환경 구성 . DB 인 경우는 초기 DB Script   

init 컨테이너는 항상 완료가 되어야 한다.    

참고 : https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

<br/>

> Question : Perform the following.  

> TASK :  
>  Add an init container to `web-pod`(which has been defined in spec file  `./webpod.yaml`).    
>  The init container should create an empty file named `/home/data.txt`.    
>  if `/home/data.txt` is not detected. the Pod should exit.  
>  Once the spec file has been `updated with the init container` definition. the pod should be created.  


<br/>
 
web-pod.yaml 화일  
```bash
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - image: ghcr.io/shclub/busybox:1.28
    name: main
    command: ['sh','-c','if [ !-f /home/data.txt ];then exit 1;else sleep 300;fi']
    volumeMounts:
    - name: workdir
      mountPath: "/home"
  volumes:
  - name: workdir
    emptyDir: {}
```   

<br/><br/>

>  결과값  : web-pod 이름의 pod에서 /home 폴더에  data.txt 화일이 있어야 함.

<br/>


아래 명령어 사용  

```bash
root@newedu:~# kubectl exec -it web-pod sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
/ # ls
bin   dev   etc   home  proc  root  run   sys   tmp   usr   var
/ # cd home
/home # ls
data.txt
```  

<br/>



> 답안  

* web-pod.yaml 화일에 initContainers를 추가 한다.   

<br/>

아래 내용 추가  

```bash
initContainers:
  - image: ghcr.io/shclub/busybox:1.28
    name: init
    command: ['sh','-c','touch  /home/data.txt']
    volumeMounts:
    - name: workdir
      mountPath: "/home"
```  

<br/>  

```bash
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  initContainers:
  - image: ghcr.io/shclub/busybox:1.28
    name: init
    command: ['sh','-c','touch  /home/data.txt']
    volumeMounts:
    - name: workdir
      mountPath: "/home"
  containers:
  - image: ghcr.io/shclub/busybox:1.28
    name: main
    command: ['sh','-c','if [ !-f /home/data.txt ];then exit 1;else sleep 300;fi']
    volumeMounts:
    - name: workdir
      mountPath: "/home"
  volumes:
  - name: workdir
    emptyDir: {}
```  

<br/>

## 15. NodePort 서비스 생성

<br/>


> Question : Create the service as type `NodePort` with `port 32767` for the nginx pod with pod selector `app: webui`  

<br/>
 
nginx.yaml 화일  
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: webui
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webui
  strategy: {}
  template:
    metadata:
      labels:
        app: webui
    spec:
      containers:
      - image: ghcr.io/shclub/nginx
        name: nginx
```   

<br/><br/>

>  결과값  : 서비스를 조회 하여 해당 서비스가 NodePort 로 설정 되어 있는지 확인한다.  

<br/>

서비스를 조회한다.   

```bash
root@newedu:~# kubectl get svc  
nginx                       NodePort    172.30.228.71    <none>        80:32767/TCP     16s
```  

curl 명령어로 서비스를 호출한다.  

```bash
root@newedu:~# curl 211.34.231.84:32767
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>  
```  

<br/>


> 답안  

* NodePort 용 서비스를 생성한다.    

<br/>

서비스 셍성하기 위한 화일 ( nginx_node.yaml ) 을 생성한다.  

```bash
root@newedu:~# kubectl expose deployment nginx --port=80 --type=NodePort --name=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: webui
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: webui
  type: NodePort
status:
  loadBalancer: {}
```  

<br/>  

생성한 nginx_node.yaml 에서 nodePort 32767 를 추가한다.    

```bash
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: webui
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    nodePort: 32767
    targetPort: 80
  selector:
    app: webui
  type: NodePort
status:
  loadBalancer: {}
```  

<br/>

서비스를 생성한다.  

```bash
root@newedu:~# kubectl apply -f nginx_node.yaml
service/nginx created
root@newedu:~# kubectl get svc  
nginx                       NodePort    172.30.228.71    <none>        80:32767/TCP     16s
```  

curl 명령어로 서비스를 호출한다.  

```bash
root@newedu:~# curl 211.34.231.84:32767
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```  

<br/>

## 16. ConfigMap 운영

<br/>

configMap을 환경 변수로 전달하는 방법과 volume mount를 해서 전달하는 방법 2가지를 다 공부해야 한다.    

https://kubernetes.io/docs/concepts/configuration/configmap/

<br/>

> Question : Expose Configuration settings

> TASK :  
>  All operations in this question should be performed in the < your namespace >
>  Create `ConfigMap` called `web-config` that contains the following `two` entries
>  - `connection_string=localhost:80`
>  - `external_url=cncf.io`

>  Run a pod called `web-pod` with a single container running the `nginx:1.19.8-alpine` image, and expose these configuration settings as `environment variables` inside the container. 
  

<br/><br/>

>  결과값  : pod 안에 환경 변수 2개가 들어 있는지 확인한다.  

<br/>


해당 POD의 환경 변수를 확인한다.  

```bash
root@newedu:~# kubectl exec  web-pod -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
HOSTNAME=web-pod
connection_string=localhost:80
external_url=cncf.io
```
  

<br/>


> 답안  

* NodePort 용 서비스를 생성한다.    

<br/>

먼저 configMap 명령어를 확인해 본다.

```bash
root@newedu:~# kubectl create configmap web-config --from-literal=connection_string=localhost:80 --from-literal=external_url=cncf.io --dry-run=client -o yaml
apiVersion: v1
data:
  connection_string: localhost:80
  external_url: cncf.io
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: web-config
```  

<br/>  

이상이 없다면 dry-run 을 삭제 하고 실행한다.  


```bash
root@newedu:~# kubectl create configmap web-config --from-literal=connection_string=localhost:80 --from-literal=external_url=cncf.io
configmap/web-config created
root@newedu:~# kubectl describe configmap web-config
Name:         web-config
Namespace:    edu30
Labels:       <none>
Annotations:  <none>

Data
====
connection_string:
----
localhost:80
external_url:
----
cncf.io
Events:  <none>
```  

<br/>

pod 를 생성하기 위한 yaml 화일을 만듭니다.   

```bash
root@newedu:~# kubectl run web-pod --image=nginx:1.19.8-alpine --port=80 --dry-run=client -o yaml > web-pod.yaml

```  

web-pod.yaml 를 수정합니다.   

envFrom을 사용하여 configmap 값을 환경 변수로 설정합니다.  

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: web-pod
  name: web-pod
spec:
  containers:
  - image: nginx:1.19.8-alpine
    name: web-pod
    envFrom:
    - configMapRef:
        name: web-config
    ports:
    - containerPort: 80
```  

<br/>

해당 화일을 적용하고 env 환경 변수를 확인한다.  

```bash
root@newedu:~# kubectl apply -f web-pod.yaml
pod/web-pod created
root@newedu:~# kubectl get po web-pod
web-pod   1/1     Running   0          15s
root@newedu:~# kubectl exec  web-pod -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
HOSTNAME=web-pod
connection_string=localhost:80
external_url=cncf.io
```
<br/>


## 17. Secret 운영

<br/>

Secret 은 configMap과 비슷하지만 base64로 인코딩 하는 부분만 다름.  

https://kubernetes.io/docs/concepts/configuration/secret/  


<br/>

Secret는 용도에 따라서 3가지가 있고 우리는 일반 적인 방식인 Generic으로 진행을 합니다.  

```bash
root@newedu:~# kubectl create secret --help
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry
  generic         Create a secret from a local file, directory or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
```  


<br/>

Secret을 생성 후 Pod에 전달  

> Question : Create a Kubernetes secret and expose  using a file in the pod.  

> TASK :  
>  Create a Kubernetes Secret as follows.
>  - Name : `super-secret` 
>  - DATA : `password=secretpass`  

>  Create a Pod named `pod-secrets-via-file`, using the `redis` image, which  mounts a secret named `super-secret` at `/secrets`.

>  Create a second Pod named `pod-secrets-via-env`, using the redis image, which exports `password` as `PASSWORD`.  

<br/><br/>

>  결과값  : 

<br/>


file 로  pod에 받기   

<br/>  

`/secrets` 폴더에 해당 화일이 있는지 확인한다.  

```bash
root@newedu:~# kubectl exec -it pod-secrets-via-file -- ls /secrets
password
```  

<br/>

환경변수로 받기  

pod에서 env로 확인한다.  

```bash
root@newedu:~# kubectl exec -it pod-secrets-via-env -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
HOSTNAME=pod-secrets-via-env
NSS_SDB_USE_CACHE=no
PASSWORD=secretpass
```  
  

> 답안  

*  file로  POD에 받는 방법

<br/>

먼저 super-secret 이라는 이름으로 secret 을 생성합니다.

```bash
root@newedu:~# kubectl create secret generic super-secret --from-literal=password=secretpass
secret/super-secret created
root@newedu:~# kubectl describe  secret super-secret
Name:         super-secret
Namespace:    edu30
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  8 bytes
root@newedu:~# kubectl get  secret super-secret -o yaml
apiVersion: v1
data:
  password: c2VjcmV0cGFzcw==
kind: Secret
metadata:
  name: super-secret
  namespace: edu30
type: Opaque
```  

<br/>

base64로 풀어 보면 실제 내용을 볼수 있다.   

```bash 
root@newedu:~# echo "c2VjcmV0cGFzcw==" | base64 -d
secretpassroot
```  

<br/>

pod 를 생성하기 위한 yaml 화일을 만듭니다.  

```bash
root@newedu:~# kubectl run pod-secrets-via-file --image=redis --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-secrets-via-file
  name: pod-secrets-via-file
spec:
  containers:
  - image: redis
    name: pod-secrets-via-file
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
root@newedu:~# kubectl run pod-secrets-via-file --image=redis --dry-run=client -o yaml > secret1.yaml
root@newedu:~# vi secret1.yaml
```  

<br/>

secret1.yaml 화일을 문제에 맞게 아래와 같이 구성합니다.  

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-secrets-via-file
  name: pod-secrets-via-file
spec:
  containers:
  - image: redis
    name: pod-secrets-via-file
    volumeMounts:
    - name : pod-secret
      mountPath: "/secrets"
  volumes:
  - name: pod-secret
    secret:
      secretName: super-secret
```

<br/>

pod를 생성합니다.    

```bash
root@newedu:~# kubectl apply -f secret1.yaml
root@newedu:~# kubectl get po pod-secrets-via-file
NAME                   READY   STATUS    RESTARTS   AGE
pod-secrets-via-file   1/1     Running   0          8m58s
```  

<br/>  

`/secrets` 폴더에 해당 화일이 있는지 확인한다.  

```bash
root@newedu:~# kubectl exec -it pod-secrets-via-file -- ls /secrets
password
```  

<br/>

pod 를 생성하기 위한 yaml 화일을 만듭니다.   

```bash
root@newedu:~# kubectl run web-pod --image=nginx:1.19.8-alpine --port=80 --dry-run=client -o yaml > web-pod.yaml

```  

web-pod.yaml 를 수정합니다.   

envFrom을 사용하여 configmap 값을 환경 변수로 설정합니다.  

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: web-pod
  name: web-pod
spec:
  containers:
  - image: nginx:1.19.8-alpine
    name: web-pod
    envFrom:
    - configMapRef:
        name: web-config
    ports:
    - containerPort: 80
```  

<br/>

해당 화일을 적용하고 env 환경 변수를 확인한다.  

```bash
root@newedu:~# kubectl apply -f web-pod.yaml
pod/web-pod created
root@newedu:~# kubectl get po web-pod
web-pod   1/1     Running   0          15s
root@newedu:~# kubectl exec  web-pod -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
HOSTNAME=web-pod
connection_string=localhost:80
external_url=cncf.io
```
<br/>


*  env 로 POD에 받는 방법

<br/>

secret2.yaml 화일을 아래와 같이 생성한다.  


```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod-secrets-via-env
  name: pod-secrets-via-env
spec:
  containers:
  - image: redis
    name: pod-secrets-via-env
    env:
      - name: PASSWORD
        valueFrom:
          secretKeyRef:
            name : super-secret
            key: password
```  

<br/>

pod를 생성하고 env로 확인한다.

```bash
root@newedu:~# kubectl apply -f secret2.yaml
pod/pod-secrets-via-env created
root@newedu:~# kubectl exec -it pod-secrets-via-env -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
HOSTNAME=pod-secrets-via-env
NSS_SDB_USE_CACHE=no
PASSWORD=secretpass
```  

<br/>


## 18. Ingress 구성 ( 배점 5점 이상 )


<br/>

> Question 1 :  Application Service 운영

> TASK :  
> `ingress-nginx` namespace 에  nginx 이미지를 `app=nginx` 레이블을 가지고 실행하는 `nginx` pod를 구성하세요.   
> 앞서 생성한 nginx Pod를 서비스 하는 `nginx` service를 생성하시오
> 현재 `appjs-service` 이름의 Service는 이미 동작중입니다. 별도 구성이 필요 없습니다. 

<br/>

> Question 2 :  Ingress 구성

> TASK :  
> `app-ingress.yaml` 화일을 생성하여  다움 조건의 ingress 서비스를 구성하시오.  
>  - ingress name : `app-ingress`  
>  - NODE_PORT:30080/ 접속 했을때 `nginx` 서비스로 연결 
>  - NODE_PORT:30080/app 접속 했을때 `appjs-service` 서비스로 연결 
>  - Ingress 구성에 다음의 annotations을 포함 시키시오.  
>     annotatons:
        kubernetes.io/ingress.class:nginx 

<br/>

OKD에 Ingress Controller 가 설치 되어 있지 않아 위에 구성중에 NODE_PORT는 서비스에 설정을 별도로 하고 테스트 하며 Namespace는 본인의 Namespace로 한다.  


<br/><br/>

>  결과값 

<br/>

okd 환경 인 경우 ip : 211.34.231.84  
aws 환경 인 경우 : 3.38.190.131  

<br/>

30080 포트에 `/` 로 호출시 nginx에 접속 되어야 한다.  

```bash
root@newedu:~# curl 211.34.231.84:30080/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```  

<br/>

30080 포트에 `/app` 로 호출시 nginx 오류가 발생.  

```bash
ubuntu@master:~$ curl 3.38.190.131:30080/app
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx</center>
</body>
</html>
```  

<br/>


> 답안  

Question 1

<br/>

nginx 라는 Pod에 label을 app=nginx 이라는 이름으로 생성하고 nginx 서비스를 노출합니다.  


```bash
root@newedu:~# kubectl run nginx --image=nginx --labels=app=nginx --dry-run=client -o yaml > app-ingress.yaml
root@newedu:~# kubectl apply -f app-ingress.yaml
pod/nginx created
root@newedu:~# kubectl expose pod nginx --port=80 --target-port=80 --dry-run=client -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
status:
  loadBalancer: {}
root@newedu:~# kubectl expose pod nginx --port=80 --target-port=80
service/nginx exposed
```  

<br/>

서비스를 NodePort로 오픈한다.  k8s에서는 오픈 필요 없고  IngressController가 30080으로 열려 있다고 간주.    

```bash
root@newedu:~# kubectl edit svc nginx
service/nginx edited
root@newedu:~# kubectl get svc nginx
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   172.30.139.165   <none>        80:30080/TCP   3h9m
```  

<br/>

Question 2

<br/>

이제 Ingress를 생성한다.  
kubernetes.io 에서 ingress를 검색하면 예제가 나온다.  

app-ingress.yaml 은 아래와 같다

```bash
root@newedu:~# cat app-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: appjs-service
            port:
              number: 80
```  

<br/>

ingress 를 생성한다.  

```bash
root@newedu:~# kubectl apply -f  app-ingress.yaml
ingress.networking.k8s.io/app-ingress created
root@newedu:~# kubectl get ingress
NAME          CLASS    HOSTS   ADDRESS   PORTS   AGE
app-ingress   <none>   *                 80      20m
```  

<br/>

30080 포트에 `/` 로 호출시 nginx에 접속 되어야 한다.  

okd 환경 인 경우 ip : 211.34.231.84  
aws 환경 인 경우 : 3.38.190.131  

<br/>

```bash
root@newedu:~# curl 211.34.231.84:30080/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```  

<br/>

30080 포트에 `/app` 로 호출시 app-service가 없기 때문에 nginx 오류가 발생.  

```bash
ubuntu@master:~$ curl 3.38.190.131:30080/app
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body>
<center><h1>503 Service Temporarily Unavailable</h1></center>
<hr><center>nginx</center>
</body>
</html>
```  

<br/>


## 19. Persistent Volume


<br/>

> Question  :  Create Persistent Volume

> TASK :  
> Create a persistent volume whith name `app-config` of capacity `1Gi` and access mode `ReadWriteMany`. 
> StorageClass: `az-c`
> The Type of volume is hostPath and its location is `/root/cka_pvc_test`.

<br/>

root폴더에서 아래 폴더를 생성한다.  

```bash
root@newedu:~# mkdir -p cka_pvc_test
```  


<br/><br/>

>  결과값 

<br/>

```bash
root@newedu:~# kubectl get pv app-config
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
app-config   1Gi        RWX            Recycle          Available           az-c                    10s
```  


<br/>


> 답안  

<br/>

아래 yaml 화일을 생성한다.  

pv_cka.yaml  

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-config
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: az-c
  hostPath:
    path: /root/cka_pvc_test
```  

<br/>

pv를 생성하고 생성된 것을 확인한다.  

```bash
root@newedu:~# kubectl apply -f pv_cka.yaml
persistentvolume/app-config created
root@newedu:~# kubectl get pv app-config
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
app-config   1Gi        RWX            Recycle          Available           az-c                    10s
```  

<br/>

## 20. Persistent Volume Claim 을 사용하는 POD 운영 ( 자주나옴 )


<br/>

> Question  :  Application with PersistentVolumeClaim

> TASK 1 :  
> Create a new `PersistentVolumeClaim`   
> - Name : `app-volume`  
> - StorageClass : `az-c`  
> - Capacity : `10Mi`  

<br/>

> TASK 2 :  
> Create a new `Pod` which mounts the PersistentVolumeClaim as a volume   
> - Name : `web-server-pod`  
> - Image : `nginx`  
> - Mount path : `/usr/share/nginx/html`  

<br/>

> TASK 3 :  
> Configure the new Pod to have `ReadWriteMany` access on the volume.  

<br/>

참고 : https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

<br/><br/>

>  결과값 

<br/>

```bash
root@newedu:~# kubectl get po web-server-pod
NAME             READY   STATUS    RESTARTS   AGE
web-server-pod   1/1     Running   0          8s
```  


<br/>


> 답안  

<br/>

아래 yaml 화일을 생성한다.  

pvc_cka.yaml  

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-volume
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Mi
  storageClassName: az-c
```  

<br/>

pvc를 생성하고 생성된 것을 확인한다.  

```bash
root@newedu:~# kubectl apply -f pvc_cka.yaml
persistentvolumeclaim/app-volume created
root@newedu:~# kubectl get pvc app-volume
NAME         STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
app-volume   Bound    app-config   1Gi        RWX            az-c           19m
```  

<br/>

아래 내용으로 web_server_pod.yaml 화일을 생성한다.  
 

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: web-server-pod
  name: web-server-pod
spec:
  volumes:
    - name: cka
      persistentVolumeClaim:
        claimName: app-volume
  containers:
  - image: nginx
    name: web-server-pod
    volumeMounts:
      - name: cka
        mountPath: /usr/share/nginx/html
    ports:
    - containerPort: 80
```  

<br/>


web-server-pod를 생성한다. 

```bash
root@newedu:~# kubectl apply -f web_server_pod.yaml
pod/web-server-pod created
root@newedu:~# kubectl get po web-server-pod
NAME             READY   STATUS    RESTARTS   AGE
web-server-pod   1/1     Running   0          8s
```  

<br/>


## 21. Check Resource Information


<br/>

> Question  : Check Resource Information
> TASK  :  
> List all `PV's sorted by name` saving the full `kubectl` output to `/my_volumes`.   
> Use `kubectl`'s own functionality for sorting the output, and do not manipulate it any further.   

<br/>

cheatsheet 에서 확인.  

https://kubernetes.io/docs/reference/kubectl/cheatsheet/  

JSON 으로 출력해서 가공한다.  

<br/><br/>

>  결과값

<br/>

```bash
root@newedu:~# cat my_volumes
```  
<br/>


> 답안  

<br/>

아래 명령어를 실행한다..  

```bash
root@newedu:~# kubectl get pv --sort-by=.metadata.name > my-volumes
```  

<br/>

## 22. Kubernetes Upgrade

<br/>

> Question  : Cluster Upgrade only Master
> TASK  :  
> Given an  existing Kubernetes cluster running version `1.22.4`.  
> Upgrade all of the Kubernetes control plane and node components on the master node only  to version `1.23.3`.     
> Be sure to  `drain` the `master` node before upgrading it and `uncordon` it after the upgrade.  


<br/>

참고: 매뉴얼 대로만 하면 됨.   
- https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

<br/><br/>

>  결과값

<br/>

아래 명령어를 실행해서 버전을 확인한다.  


```bash
root@newedu:~# kubectl get nodes
```  
<br/>


> 답안  

<br/>

아래 명령어를 실행한다..  

```bash
apt update
apt-cache madison kubeadm
```  

<br/>  

kubeadm upgrade 한다.  ( master node 에서만 진행 )

```bash
apt-mark unhold kubeadm && \
 apt-get update && apt-get install -y kubeadm=1.23.3-00 && \
 apt-mark hold kubeadm
```  

<br/>

버전을 확인하고 upgrade plan 을 확인하고 해당 버전을 upgrade 한다.  

```bash  
kubeadm version
kubeadm upgrade plan
kubeadm upgrade apply v1.24.1
```  

<br/>

다른 Master Node에서도 실행.  ( CKA 에서는 Master Node 가 1개로 아래 명령어 사용 안함 )

```bash  
kubeadm upgrade node
```  

<br/>

kubelet을 upgrade 하기 위해 master node를 비워준다.   

주의 사항.    
  - master node에서 나간 이후 실행. exit

```bash  
console$ kubectl drain k8s-master --ignore-daemonsets
console$ kubectl get nodes
```  

<br/>

다시 master node에 접속

```bash  
console$ ssh k8s-master   
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.24.1 kubectl=1.24.1 && \
apt-mark hold kubelet kubectl
```  

<br/>

설치가 완료되면 재기동  

```bash
systemctl daemon-reload
systemctl restart kubelet
```  
<br/>

Exit하고 콘솔로 간 후 pod가 스케쥴링 되도록 한다.    

```bash
kubectl uncordon k8s-master  
```   

<br/>


## 23. Kubernetes TroubleShooting (1)

<br/>

> Question  : Not Ready 상태의 노드 활성화
> TASK  :  
> A kubernetes worker node, named `edu.worker01` in state `NotReady`.    
> Investigate why this is the case, and perform and appropriate steps to bring the node to a `Ready` state, ensuring that any changes are made permanent.



<br/><br/>

>  결과값

<br/>

아래 명령어를 실행해서 status를 확인한다.  

```bash
root@newedu:~# kubectl get nodes
```  
<br/>


> 답안  

<br/>

worker node의 컨테이너 엔진과 kubelet , proxy , CNI 가 running 인지  확인한다.

```bash
systemctl status docker
systemctl status kubelet
```

<br/>

kubelet 문제가 있으면  바로 활성화 시키고 부팅시에도 적용  

```bash
systemctl enable --now kubelet
```

<br/>

콘솔 모드에서 kube-proxy와 cni ( calico 등 ) 를 확인한다.

```bash
kubectl get po kube-system -o wide 
```  

<br/>


## 24. User Role Binding

<br/>

Role은 특정 Namespace 에서만 적용. 

<br/>


> Question  : Configuring User API Authentification  

> TASK 1 :   Create the kubeconfig named `ckauser`.
> - username: `ckauser`    
> - certificate location: `/data/ca/ckauser.crt,/data/cka/ckauser.key`   
> - `ckauser` cluster must be operated with the privileges of the `ckauser` account.     

<br/>

> TASK 2 :   Create a role named `pod-role` that can `create,delete,watch,list,get` pods. Create the following rolebinding.   
> - name: `pod-rolebinding`    
> - role: `pod-role`   
> - user: `ckauser`     

<br/>

TASK 2 먼저 진행하고 TASK1 을 진행한다.  

아래 문서에서 `CSR` 로 검색하면 샘플이 나온다.  

<br/>

참고  
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/

<br/><br/>

>  결과값

<br/>

context를 변경하고 해당 유저로 pod를 생성 , 조회 해본다.     

```bash
root@newedu:~# kubectl config use-context ckauser
Switched to context "ckauser".
```  

<br/>


> 답안  

<br/>


아래 처럼 role 을 만들기 위한 yaml 화일을 생성한다.  ( 명령어로 진행 해도 됨 )

role_cka.yaml  
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-role
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["create", "delete","watch", "list","get"]
```

<br/>

명령어 사용 하는 경우  

```bash
root@newedu:~# kubectl create role pod-role --verb=create --verb=get --verb=list --verb=watch --verb=delete --resource=pods
```
<br/>

role을 생성하고 확인한다.  

```bash
root@newedu:~# kubectl apply -f role_cka.yaml
role.rbac.authorization.k8s.io/pod-role created
root@newedu:~# kubectl get role pod-role
NAME       CREATED AT
pod-role   2022-09-27T14:04:40Z
```

<br/>

아래 처럼 rolebinding 을 만들기 위한 yaml 화일을 생성한다.  

rolebinding_cka.yaml    
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-rolebinding
subjects:
- kind: User
  name: ckauser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role 
  name: pod-role 
  apiGroup: rbac.authorization.k8s.io
```

<br/>

명령어를 사용하는 경우   

```bash
root@newedu:~# kubectl create rolebinding pod-rolebinding --role=pod-role --user=ckauser
```  

<br/>

rolebinding을 생성하고 확인한다.  

```bash
root@newedu:~# kubectl apply -f  rolebinding_cka.yaml
rolebinding.rbac.authorization.k8s.io/pod-rolebinding created
root@newedu:~# kubectl get rolebinding pod-rolebinding
NAME              ROLE            AGE
pod-rolebinding   Role/pod-role   24s
```

<br/>

TASK 1 답안  : 이미 만들어져 있는 인증서를 이용하여 ckauser 가 생성이 된다.  

```bash
root@newedu:~# kubectl config set-credentials ckauser --client-key=/data/cka/ckauser.key --client-certificate=/data/ca/ckauser.crt --embed-certs=true
```

config 정보를 사용하여 확인한다.  

```bash
root@newedu:~# kubectl config view
```
<br/>

context를 등록한다.  

```bash
root@newedu:~# kubectl config set-context ckauser --cluster=kubernetes --user=ckauser
Context "ckauser" created.
```
]<br/>

context를 등록한다.  

```bash
root@newedu:~# kubectl config set-context ckauser --cluster=kubernetes --user=ckauser
Context "ckauser" created.
```  

<br/>

context를 변경하고 해당 유저로 pod를 생성 , 조회 해본다.     

```bash
root@newedu:~# kubectl config use-context ckauser
Switched to context "ckauser".
```

<br/>

## 25. User Cluster Role Binding

<br/>

ClusterRole은 Cluster 전체에 적용. 

<br/>


> Question  : Configuring User API Authentification  

> TASK  :  
>  Create a new `ClusterRole` named `app-clusterrole`, which only allows to `get,watch,list` the following resource types: `Deployment, Service`   
>  Bind the new ClusterRole `app-clusterrole` to the new  user `ckauser`.  
>  User ckauser and ckauser clusters are already configured.
>  To check the results, run the following command.
>   - kuectl config user-context ckauser


<br/><br/>

>  결과값

<br/>

<br/>

context를 변경하고  service 와 deployment를 조회해 본다.  

```bash
root@newedu:~# kubectl config use-context ckauser
root@newedu:~# kubectl get svc
root@newedu:~# kubectl get deploy
```  
  

<br/>


> 답안  

<br/>


명령어 사용 하여 clusterrole을 생성하고 확인한다.  

```bash
root@newedu:~# kubectl create clusterrole app-clusterrole --verb=get,list,watch --resource=deployment,service
clusterrole.rbac.authorization.k8s.io/app-clusterrole created
root@newedu:~# kubectl get clusterrole app-clusterrole
NAME              CREATED AT
app-clusterrole   2022-09-27T14:40:09Z
```

<br/>


명령어 사용 하여 clusterrolebinding을 생성하고 확인한다.  

```bash
root@newedu:~# kubectl create clusterrolebinding app-clusterrolebinding --clusterrole=app-clusterrole --user=ckauser
clusterrolebinding.rbac.authorization.k8s.io/app-clusterrolebinding created
root@newedu:~# kubectl get clusterrolebinding app-clusterrolebinding
NAME                     ROLE                          AGE
app-clusterrolebinding   ClusterRole/app-clusterrole   9s
```  

<br/>

context를 변경하고  service 와 deployment를 조회해 본다.  

```bash
root@newedu:~# kubectl config use-context ckauser
root@newedu:~# kubectl get svc
root@newedu:~# kubectl get deploy
```  

<br/>


## 26. ServiceAccount Role Binding

<br/>

ServiceAccount는 pod의 권한이며 아무 것도 설정하지 않으면 default 가 적용된다.  

Role은 특정 Namespace 에서만 적용. 

<br/>


> Question  : Service Account , Role and RoleBinding  

> TASK  
> Create the `ServiceAccount` named pod-access in a new namespace called `edu29`.      
> Create a `Role` with the name `pod-role`, and the RoleBinding  named `pod-rolebinding`.      
> Map the ServiceAccount from the previous step to the API resources `Pods` with the operations `watch,list,get`.      

<br/>

여기에서는 reference를 사용합니다. 

<br/>

참고  : Docs -> reference로 이동하여 kubectl command line 선택  
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

<br/><br/>

>  결과값

<br/>

아래 명령어 수행 후 확인.  

```bash
root@newedu:~# kubectl describe rolebinding pod-rolebinding -n edu29
```  

<br/>


> 답안  

<br/>

service account 생성 ( edu29 namespace )  

```bash
root@newedu:~# kubectl create serviceaccount pod-access -n edu29
serviceaccount/pod-access created
```  

<br/>

pod-role 생성 ( edu29 namespace )  

```bash
root@newedu:~# kubectl create role pod-role --verb=watch,list,get --resource=pods -n edu29
role.rbac.authorization.k8s.io/pod-role created
```

<br/>

pod-rolebinding 생성 ( edu29 namespace )  

```bash
root@newedu:~# kubectl create rolebinding pod-rolebinding --role=pod-role --serviceaccount=edu29:pod-access -n edu29
rolebinding.rbac.authorization.k8s.io/pod-rolebinding created
```

<br/>

## 27. ServiceAccount ClusterRole Binding ( SKIP )

<br/>

ServiceAccount는 pod의 권한이며 아무 것도 설정하지 않으면 default 가 적용된다.  

Role은 특정 Namespace 에서만 적용. 

<br/>


> Question  : Service Account , Role and RoleBinding  

> TASK  
> Create the `ServiceAccount` named pod-access in a new namespace called `edu29`.      
> Create a `Role` with the name `pod-role`, and the RoleBinding  named `pod-rolebinding`.      
> Map the ServiceAccount from the previous step to the API resources `Pods` with the operations `watch,list,get`.      

<br/>

여기에서는 reference를 사용합니다. 

<br/>

참고  : Docs -> reference로 이동하여 kubectl command line 선택  
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

<br/><br/>

>  결과값

<br/>

아래 명령어 수행 후 확인.  

```bash
root@newedu:~# kubectl describe rolebinding pod-rolebinding -n edu29
```  

<br/>


> 답안  

<br/>

service account 생성 ( edu29 namespace )  

```bash
root@newedu:~# kubectl create serviceaccount pod-access -n edu29
serviceaccount/pod-access created
```  

<br/>

pod-role 생성 ( edu29 namespace )  

```bash
root@newedu:~# kubectl create role pod-role --verb=watch,list,get --resource=pods -n edu29
role.rbac.authorization.k8s.io/pod-role created
```

<br/>

pod-rolebinding 생성 ( edu29 namespace )  

```bash
root@newedu:~# kubectl create rolebinding pod-rolebinding --role=pod-role --serviceaccount=edu29:pod-access -n edu29
rolebinding.rbac.authorization.k8s.io/pod-rolebinding created
```

<br/>

## 28. Kube-DNS

<br/>

k8s 자체적으로 DNS 서버를 가지고 있다.

<br/>


> Question  : Service and DNS Lookup 구성  

> TASK  
> `Create` a `nginx` Pod called `nginx-resolver` using image `nginx` , expose it internally with a service  calls `nginx-resolver-service`.     
> `Test` that you are `able to look up the service and pod` names from within the cluster. Use the image `busybox:1.28` for dns lookup.     
> - Record results in `/tmp/nginx.svc` and `/tmp/nginx.pod`        
> - Pod : `nginx-resolver` created        
> - Service  `DNS Resolution` recorded correctly        
> - Pod `DNS resolution` recorded correctly        

<br/>

여기에서는 reference를 사용합니다. 

<br/>

참고  : Docs -> reference로 이동하여 kubectl command line 선택  
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

<br/><br/>

>  결과값

<br/>

아래 명령어 수행 후 확인.  

```bash
root@newedu:~# cat /tmp/nginx.svc
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      nginx-resolver-service.edu30.svc.cluster.local
Address 1: 172.30.127.12 nginx-resolver-service.edu30.svc.cluster.local
root@newedu:~# cat /tmp/nginx.pod
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      10-131-7-82.edu30.pod.cluster.local
Address 1: 10.131.7.82 10-131-7-82.nginx-resolver-service.edu30.svc.cluster.local
```  

<br/>


> 답안  

<br/>

```bash
root@newedu:~# kubectl run nginx-resolver --image=nginx
pod/nginx-resolver created
root@newedu:~# kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80
service/nginx-resolver-service exposed
```  

<br/>

pod와 서비스를 확인하고 ip를 저장한다.   

```bash
root@newedu:~# kubectl get po nginx-resolver -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
nginx-resolver   1/1     Running   0          11s   10.131.7.82   edu.worker07   <none>           <none>
root@newedu:~# kubectl get svc  nginx-resolver-service -o wide
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
nginx-resolver-service   ClusterIP   172.30.127.12   <none>        80/TCP    74m   run=nginx-resolver
```  

<br/>


busybox 를 pod로 띄우고 shell 로 접속하여 nslookup 명령어를 실행해본다.       

- dns 서버를 조회한다.
- <ip url>.<namespace>.pod.cluster.local 를 nslookup 해서 ip를 알아내고  pod ip 인지 확인
- pod의 ip로 nslookup 해서 domain 정보 확인
- ngix-resolver-serivce.<namespace>.svc.cluster.local 를 nslookup 해서 ip를 알아내고  service ip 인지 확인
- service 의 ip로 nslookup 해서 domain 정보 확인


<br/>


```bash
/ # cat /etc/resolv.conf
search edu30.svc.cluster.local svc.cluster.local cluster.local localhost
nameserver 172.30.0.10
options ndots:5  
root@newedu:~# kubectl run --image=busybox:1.28 --rm -it -- sh
If you don't see a command prompt, try pressing enter.
/ # nslookup 10-131-7-82.edu30.pod.cluster.local
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      10-131-7-82.edu30.pod.cluster.local
Address 1: 10.131.7.82 10-131-7-82.nginx-resolver-service.edu30.svc.cluster.local
/ # nslookup 10.131.7.82
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      10.131.7.82
Address 1: 10.131.7.82 10-131-7-82.nginx-resolver-service.edu30.svc.cluster.local

/ # nslookup nginx-resolver-service.edu30.svc.cluster.local
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      nginx-resolver-service.edu30.svc.cluster.local
Address 1: 172.30.127.12 nginx-resolver-service.edu30.svc.cluster.local
/ # nslookup 172.30.127.12
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      172.30.127.12
Address 1: 172.30.127.12 nginx-resolver-service.edu30.svc.cluster.local
```

<br/>

pod를 나와서 결과를 화일에 저장한다.  

```bash
root@newedu:~# vi /tmp/nginx.svc
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      nginx-resolver-service.edu30.svc.cluster.local
Address 1: 172.30.127.12 nginx-resolver-service.edu30.svc.cluster.local
oot@newedu:~# vi /tmp/nginx.pod
Server:    172.30.0.10
Address 1: 172.30.0.10 dns-default.openshift-dns.svc.cluster.local

Name:      10-131-7-82.edu30.pod.cluster.local
Address 1: 10.131.7.82 10-131-7-82.nginx-resolver-service.edu30.svc.cluster.local
```

<br/>


## 29. Network Policy

<br/>

k8s 의 접근제어 라고 이해 하면 됨.

<br/>


> Question  : Network Ploicy with Namespace

> TASK  
> Create a New `NetworkPolicy` named `allow-port-from-namespace` in the existing namespace `devops`.       
> Ensure that the new NetworkPolicy `allows Pods in namespace edu30` to connect `port 80 of Pods in namespace devops`.      

<br/>

여기에서는 reference를 사용합니다. 

<br/>

참고  : Docs -> reference로 이동하여 kubectl command line 선택  
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

<br/><br/>

>  결과값

<br/>

edu30 namespace 의 아무 pod에서 devops 네임스페이스의  webservice 서비스를₩ 80 포트로 호출한다.

<br/>


> 답안  

<br/>

devops namespace에서 pod를 wide로 조회하여 label 까지 확인한다.   

```bash
root@newedu:~# kubectl get po -n devops --show-labels
NAME                                               READY   STATUS    RESTARTS   AGE   LABELS
webserver                                          1/1     Running   0          55d   app=web
```  

<br/>

edu30  namespace에 label을 설정합니다.  

```bash
root@newedu:~# kubectl label  namespace edu30 team=edu30
namespace/edu30 labeled
```  

<br/>

Network Policy yaml 화일을 만든다.  
- podSelector는 webserver의 label을 설정한다.  
- namespaceSelector 는 namespace ( edu30 )을 설정한다.  

<br/>

network_policy.yaml  
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: devops
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              team: edu30
      ports:
        - protocol: TCP
          port: 80
```  

<br/>

Network Policy 를 생성한다.  

<br/>


```bash
root@newedu:~# kubectl apply -f network_policy.yaml
networkpolicy.networking.k8s.io/allow-port-from-namespace created
```

<br/>