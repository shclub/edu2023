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

## 2. pod 생성하기  


<br/>

> Question : Create a new namespace and create a pod in the namespace 

> TASK :  
>  - namespace name : 본인 namespace ( edu1 ~ 21 )
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

k8s cka 환경.    

<br/>

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

<br/>

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


## 12. Pod log 추출

<br/>

Pod Log를 stand out 으로 추출.

> Question : Monitor the logs of pod `custom-app` and Extract log lines corresponding to error `file not found`. Write them  to `./podlog`.



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
