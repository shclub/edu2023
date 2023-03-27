# 과제  
 
매 주 마다 과제는 아래와 같다.

1. 1주차
    -  Nexus private registry를 사용하여 Jenkins Pipeline 구성
    -  컨테이너 Mysql DB 활용
    -  Portainer 설치
    -  Harbor 구성해 보기 
2. 2주차 : CKA 문제 풀어 보기 [ 문제 ](./cka.md) 
    -  CKA 전체 문제 ( https://github.com/shclub/cka )  
3. 3주차 : kubernetes에 Jenkins Master/Slave 구성하기 (CI - Python )
4. 4주차 : kubernetes에 CI/CD 구성 (CI : Maven/Skaffold , CD : ArgoCD/kustomize )

<br/>

##  1주차

<br/>

### Nexus private registry를 사용하여 Jenkins Pipeline 구성

<br/>

순서
- credential을 private docker registry 용으로 신규로 구성 한다.  
- New Item으로 신규 pipeline을 구성한다. ( copy from 이름 활용 )
- Jenkins 화일을 신규 생성하고 docker registry 관련 값을 수정한다.
- pipeline 을 실행시키고 본인의 nexus에 도커 이미지가 저장되었는지 확인한다.    
- 참고 :  Jenkins_private 화일  


### 컨테이너 Mysql DB 활용 

<br/>

 docker compose로 구성한 mysql container에 접속하여 로그인 한 후 wordpress db에 customer 테이블을 생성해 본다.  
 - docker-compose.yml 화일 참고
 - table 이름은 customer 이고 필드는 customer_id , customer_name 만 필요  

<br/>

### Portainer 설치

<br/>

docker 컨테이너 GUI 관리 툴인 portainer를 설치하고 웹에서 접속하여
          모니터링한다.
  - url  참고 :  https://docs.portainer.io/v/ce-2.11/start/install/server/docker/linux
  - 웹 포트는 40005로 expose 한다 ( https 9443 포트 변경 필요 ).
  - 웹브라우저 접속은 https://(본인VM Public IP):40005  
     admin 비밀번호 신규로 생성 (8자리 이상) 한다.

<br/>

### Harbor 구성해 보기

<br/>

Harbor 설치 ( https가 힘든 사람은 http 로 구성 )
- Private Docker Registry 를 Harbor를 사용하여 구성 한다.
- Harbor 포트는 40002를 사용하며 https 연결하기 위한 인증서 설정을 한다.
- Harbor 에 edu 프로젝트를 생성하고 신규 계정을 생성하여 members에 추가한다.
- nginx 이미지를 본인의 Private Docker Registry에 Push 한다.
- Harbor에서 확인한다.
- clair 를 사용하여 도커 취약점을 분석한다.  

<br/>

샘플 : https://211.252.85.148:40002/  
계정 : admin/New1234!

<br/> 


##  2주차

<br/>

### CKA 예상 문제 중에 풀어볼 문제.


<br/>

- Master Node에 ssh 로 접속하여 kubectl 명령어 사용 ( worker node에서는 안됨 )
- Master Node 에서 본인의 폴더에 들어가서 작업을 한다. ( edu1 ~ edu 21 )
- Namespace는 직접 만들어서 사용한다.
  - 예 ) kubectl create namespace edu1
- 모든 작업은 본인의 namespace 에서 진행을 한다. ( 반드시 namespace 명시 :  `-n < 본인 namespace 명 >`  )
  - 예) kubectl apply -f test.yaml -n edu1  
  - 예) kubectl get po -n edu1

<br/>

필수 문제
- 2번 :
- 3번 : static pod는 본인의 순번을 앞에 붙인다 ( 예, edu1_web )
- 4번 :
- 7번 :
- 9번 :
- 10번 :
- 11번 :



##  3주차

<br/>

### kubernetes에 Jenkins Master/Slave 구성하기


<br/>

로그인 한 후에 jenkins 폴더를 생성한다.  

<br/>

```bash
root@newedu:~# mkdir -p jenkins
root@newedu:~# cd jenkins
```

<br/>

#### 계정 생성과 RBAC 생성

<br/>

jenkins admin 계정의 secret를 생성하기 위해 계정과 비밀번호를 base64로 인코딩 한다.

<br/>

계정과 비밀번호는 admin/New1234! 로 설정

TODO 

<br/>

secret를 만들기 위해 yaml 화일을 생성한다.        

TODO  

secret은 2개의 필드가 필요하며 아래 helm jenins_values.yaml 화일의 
2개의 필드 ( 유저와 비밀번호 ) 와 이름이 같아야 한다.  

<br/>

```bash
 50   admin:
 51     existingSecret: "jenkins-admin-secret" #
 52     userKey: jenkins-admin-user
 53     passwordKey: jenkins-admin-password
```  

<br/>

data 부분에 base64 인코딩 된 값을 넣어준다.

`jenkins-admin-secret` 이름으로 secret 생성.

<br/>

TODO

<br/>

secret를 생성하고 확인한다.

<br/>

```bash
root@newedu:~/jenkins# kubectl apply -f jenkins-edu-admin-secret.yaml
secret/jenkins-admin-secret created
root@newedu:~/jenkins# kubectl get secret
NAME                                 TYPE                                  DATA   AGE
jenkins-admin-secret                 Opaque                                2      7s
my-service-account-dockercfg-4j5n7   kubernetes.io/dockercfg               1      169d
my-service-account-token-d67wk       kubernetes.io/service-account-token   4      169d
my-service-account-token-wxttf       kubernetes.io/service-account-token   4      169d
super-secret                         Opaque                                1      169d
```

<br/>

`jenkins-admin` service account 를 생성 하고  Role 과 Rolebinding 을 생성한다.  

TODO : sa 생성

<br/>


```bash
root@newedu:~/jenkins# vi jenkins_rbac.yaml
```

<br/>
jenkins-admin 권한은 최소화 한다.

<br/>

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins-admin
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get","list","watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["get", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins-admin
subjects:
- kind: ServiceAccount
  name: jenkins-admin
```

<br/>

본인의 namespace 에 적용한다.

<br/>

```bash
root@newedu:~/jenkins# kubectl apply -f jenkins_rbac.yaml
serviceaccount/jenkins-admin created
role.rbac.authorization.k8s.io/jenkins-admin created
rolebinding.rbac.authorization.k8s.io/jenkins-admin created
```  

<br/>

생성된 resource 들을 확인한다.  

<br/>

```bash
root@newedu:~/jenkins# kubectl get sa
NAME                 SECRETS   AGE
builder              2         247d
default              2         247d
deployer             2         247d
edu                  2         7d20h
jenkins-admin        2         10s
my-service-account   2         169d
root@newedu:~/jenkins# kubectl get role
NAME            CREATED AT
developer       2022-09-27T14:16:45Z
jenkins-admin   2023-03-16T02:03:11Z
pod-role        2022-09-27T14:04:40Z
root@newedu:~/jenkins# kubectl get rolebindings
NAME                              ROLE                                          AGE
admin                             ClusterRole/admin                             247d
developer-binding-myuser          Role/developer                                169d
edu30-admin                       ClusterRole/admin                             247d
jenkins-admin                     Role/jenkins-admin                            23s
pod-rolebinding                   Role/pod-role                                 169d
pod-rolebinding2                  Role/pod-role                                 7d19h
system:deployers                  ClusterRole/system:deployer                   247d
system:image-builders             ClusterRole/system:image-builder              247d
system:image-pullers              ClusterRole/system:image-puller               247d
system:openshift:scc:privileged   ClusterRole/system:openshift:scc:privileged   182d
```  

<br/>

jenkins-admin 으로 OKD에 접속하기 위해 anyuid, privileged 권한을 부여한다.    

privileged 권한은 podman 에서 cri-o 런타임을 연결하기 위해 필요하다.  

<br/>

```bash
root@newedu:~/jenkins# oc adm policy add-scc-to-user anyuid -z jenkins-admin 
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "jenkins-admin"
root@newedu:~/jenkins# oc adm policy add-scc-to-user privileged -z jenkins-admin
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:privileged added: "jenkins-admin"
```  

<br/>

#### Storage 설정

<br/>


Jenkins 가 사용하는 stroage를 위해 pv / pvc 를 생성해야 하며
사전에 NFS 에 접속하여 폴더를 생성한다.   

폴더는 아래와 같이 사전 생성 되어 있음.

<br/>

```bash
[root@edu jenkins]# mkdir -p edu
[root@edu jenkins]# mkdir -p edu1
[root@edu jenkins]# ls
edu  edu1  edu10  edu11  edu12  edu13  edu14  edu15  edu16  edu17  edu18  edu19  edu2  edu20  edu21  edu3  edu4  edu5  edu6  edu7  edu8  edu9
```

<br/>

Jenkins slave 용 폴더도 생성한다.

```bash
[root@edu jenkins]# mkdir -p edu
[root@edu jenkins]# mkdir -p edu1_slave
...
```

<br/>

jenkins master / slave 용 해당 폴더의 권한을 설정한다.

<br/>

```bash
[root@edu jenkins]# chown -R nfsnobody:nfsnobody edu
[root@edu jenkins]# chmod 777 edu
[root@edu jenkins]# chown -R nfsnobody:nfsnobody edu_slave
[root@edu jenkins]# chmod 777 edu_slave
```  

<br/>

Master용 PV 를 생성한다. 사이즈는 5G로 설정한다.

<br/>

이름은 아래와 같이 생성한다.  

jenkins_pv.yaml  

예)  edu1 : jenkins-edu1-pv  
     edu2 : jenkins-edu2-pv

<br/>

PV를 생성하고 Status를 확인해보면 Available 로 되어 있는 것을 알 수 있습니다.  

<br/>

```bash
root@newedu:~/jenkins# kubectl apply -f  jenkins_pv.yaml
persistentvolume/jenkins-edu-pv created
root@newedu:~/jenkins# kubectl get pv jenkins-edu-pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
jenkins-edu-pv   5Gi        RWX            Retain           Available                                   10s
```

<br/>

Master용 pvc 를 생성합니다. pvc 이름을 기억합니다.

<br/>

예)  edu1 : jenkins-edu1-pvc  
     edu2 : jenkins-edu2-pvc

<br/>


Slave 용 PV 를 생성한다. 사이즈는 5G로 설정한다.

<br/>

```bash  
root@newedu:~/jenkins#  vi jenkins_slave_pv.yaml
```  

jenkins_slave_pv.yaml  

예)  edu1 : jenkins-edu1-slave-pv  
     edu2 : jenkins-edu2-slave-pv


<br/>

PV를 생성하고 Status를 확인해보면 Available 로 되어 있는 것을 알 수 있습니다.  

<br/>

```bash
root@newedu:~/jenkins# kubectl apply -f  jenkins_slave_pv.yaml
persistentvolume/jenkins-edu-slave-pv created
root@newedu:~/jenkins# kubectl get pv jenkins-edu-slave-pv
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
jenkins-edu-slave-pv   5Gi        RWX            Retain           Bound    edu30/jenkins-edu-slave-pvc                           100s
```

<br/>

Slave 용 pvc 를 생성합니다. pvc 이름을 기억합니다.

<br/>

```bash
root@newedu:~/jenkins# vi jenkins_slave_pvc.yaml
```


예)  edu1 : jenkins-edu1-slave-pvc  
     edu2 : jenkins-edu2-slave-pvc


<br/>

PVC를 생성할 때는 namespace ( 본인의 namespace ) 를 명시해야 합니다.  
PVC 생성을 확인 해보고 다시 PV를 확인해 보면 Status가 Bound 로 되어 있는 것을 알 수 있습니다.  이제 PV 와 PVC가 연결이 되었습니다.

<br/>

```bash
root@newedu:~/jenkins# kubectl apply -f jenkins_slave_pvc.yaml
persistentvolumeclaim/jenkins-edu-slave-pvc created
root@newedu:~/jenkins# kubectl get pvc
NAME                    STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS   AGE
app-volume              Bound    app-config             1Gi        RWX            az-c           169d
jenkins-edu-pvc         Bound    jenkins-edu-pv         5Gi        RWX                           136m
jenkins-edu-slave-pvc   Bound    jenkins-edu-slave-pv   5Gi        RWX                           3m27s
root@newedu:~/jenkins# kubectl get pv jenkins-edu-slave-pv
NAME                   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
jenkins-edu-slave-pv   5Gi        RWX            Retain           Bound    edu30/jenkins-edu-slave-pvc                           4m
```

<br/>

#### Helm  Jenkins  설정

<br/>

Jenkins 는 Helm Chart 를 이용하여 설치를 합니다.  

<br/>

현재 로컬의 helm repository 를 확인한다.   

<br/>

```bash
root@newedu:~/jenkins# helm repo list
NAME                           	URL
bitnami                        	https://charts.bitnami.com/bitnami
nfs-subdir-external-provisioner	https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```  

<br/>

jenkins helm repository를  아래와 같이 추가 한다.

<br/>

```bash
root@newedu:~/jenkins# helm repo add jenkins https://charts.jenkins.io --insecure-skip-tls-verify
"jenkins" has been added to your repositories
```

<br/>

helm repository를 update 한다.  

<br/>

```bash
root@newedu:~/jenkins# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "nfs-subdir-external-provisioner" chart repository
...Successfully got an update from the "jenkins" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```  

<br/>

jenkins helm reppository 에서 helm chart를 검색을 하고 jenkins chart를 선택합니다.  

<br/>

```bash
root@newedu:~/jenkins# helm search repo jenkins
NAME           	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/jenkins	12.0.1       	2.387.1    	Jenkins is an open source Continuous Integratio...
jenkins/jenkins	4.3.8        	2.387.1    	Jenkins - Build great things at any scale! The ...
```

<br/>

jenkins/jenkins 차트에서 차트의 변수 값을 변경하기 위해 jenkins_values.yaml 화일을 추출한다.

<br/>


TODO

<br/>

vi 데이터에서 생성된 jenkins_values.yaml을 연다.  

<br/>

```bash
root@newedu:~# vi jenkins_values.yaml
```  

라인을 보기 위해 ESC 를 누른 후 `:set nu` 를 입력하면 왼쪽에 라인이 보인다.  

아래 라인을 찾아 값을 변경한다.  51번 라인에 앞에서 생성한 secret을 넣는다.   

pvc는 본인의 pvc 를 설정한다.

<br/>

```bash
 50   admin:
 51     existingSecret: "jenkins-admin-secret" #
 52     userKey: jenkins-admin-user
 53     passwordKey: jenkins-admin-password
```  

<br/>

installPlugins에서 kubernetes 값 아래와 같이 반드시 변경 필요

<br/>

```bash
244   installPlugins:
245     - kubernetes:3842.v7ff395ed0cf3 #3734.v562b_b_a_627ea_c
246     - workflow-aggregator:590.v6a_d052e5a_a_b_5
247     - git:4.13.0
248     - configuration-as-code:1569.vb_72405b_80249
...
508   # Openshift route
509   route:
510     enabled: true  # true 로 변경
511     labels: {}
512     annotations: {}
...
617 agent:
618   enabled: true
619   defaultsProviderTemplate: ""
620   # URL for connecting to the Jenkins contoller
621   jenkinsUrl:
622   # connect to the specified host and port, instead of connecting directly to the Jenkins controller
623   jenkinsTunnel:
624   kubernetesConnectTimeout: 5
625   kubernetesReadTimeout: 15
626   maxRequestsPerHostStr: "32"
627   namespace:
628   image: "jenkins/jnlp-slave" #"jenkins/inbound-agent"
629   tag: "latest-jdk11"
...
662   volumes: # []
663   # - type: ConfigMap
664   #   configMapName: myconfigmap
665   #   mountPath: /var/myapp/myconfigmap
666   # - type: EmptyDir
667   #   mountPath: /var/myapp/myemptydir
668   #   memory: false
669   # - type: HostPath
670   #   hostPath: /var/lib/containers
671   #   mountPath: /var/myapp/myhostpath
672   # - type: Nfs
673   #   mountPath: /var/myapp/mynfs
674   #   readOnly: false
675   #   serverAddress: "192.0.2.0"
676   #   serverPath: /var/lib/containers
677    - type: PVC
678      claimName: jenkins-edu-slave-pvc
679      mountPath: /var/jenkins_home
680      readOnly: false
...
691   workspaceVolume: # {}
692   ## DynamicPVC example
693   # type: DynamicPVC
694   # configMapName: myconfigmap
695   ## EmptyDir example
696   # type: EmptyDir
697   # memory: false
698   ## HostPath example
699   # type: HostPath
700   # hostPath: /var/lib/containers
701   ## NFS example
702   # type: Nfs
703   # readOnly: false
704   # serverAddress: "192.0.2.0"
705   # serverPath: /var/lib/containers
706   ## PVC example
707    type: PVC
708    claimName: jenkins-edu-slave-pvc
709    readOnly: false
710   #
710   #
...

822 persistence:
823   enabled: true
824   ## A manually managed Persistent Volume and Claim
825   ## Requires persistence.enabled: true
826   ## If defined, PVC must be created manually before volume will be bound
827   existingClaim: "jenkins-edu-pvc" #
828   ## jenkins data Persistent Volume Storage Class
829   ## If defined, storageClassName: <storageClass>
830   ## If set to "-", storageClassName: "", which disables dynamic provisioning
831   ## If undefined (the default) or set to null, no storageClassName spec is
832   ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
833   ##   GKE, AWS & OpenStack)
834   ##
835   storageClass:
836   annotations: {}
837   labels: {}
838   accessMode: "ReadWriteOnce"
839   size: "5Gi"  # "8Gi"  
...

870 serviceAccount:
871   create: false #  이미 생성 했기 때문에 false 로 변경
872   # The name of the service account is autogenerated by default
873   name: "jenkins-admin" #
874   annotations: {}
875   extraLabels: {}
876   imagePullSecretName:

```  

#### Helm 으로 Jenkins 설치

<br/>

jenkins_values.yaml 를 사용하여 설치 한다.

<br/>


TODO

<br/>

설치가 완료되면 pod를 조회하여 jenkins-0 pod가 있는지 확인한다.

<br/>

```bash
root@newedu:~/jenkins# kubectl get po
NAME                                        READY   STATUS    RESTARTS   AGE
jenkins-0                                   2/2     Running   0          18h
nfs-test-589c488d6f-8lk5p                   1/1     Running   0          40h
``` 

<br/>

#### Jenkins 설정

<br/>

웹 브라우저 에서 본인의 jenkins 로 접속한다.   
route 정보를 모르면 아래 명령어로 조회한다.

<br/>

```bash
root@newedu:~/jenkins# kubectl get route
NAME      HOST/PORT                                 PATH   SERVICES   PORT   TERMINATION     WILDCARD
jenkins   jenkins-edu30.apps.211-34-231-82.nip.io          jenkins    http   edge/Redirect   None
```  

<br/>

https://jenkins-edu30.apps.211-34-231-82.nip.io/


<br/>

아래 처럼 로그인 화면이 나오면 admin 계정으로 로그인한다.

<br/>

<img src="./assets/jenkins_login.png" style="width: 80%; height: auto;"/> 

<br/>

Manage Jenkins 메뉴를 클릭한다.

<br/>

<img src="./assets/manage_jenkins1.png" style="width: 80%; height: auto;"/> 

Manage Plugins 메뉴를 클릭한다.

<br/>

<img src="./assets/manage_plugins.png" style="width: 80%; height: auto;"/> 

<br/>

Plugin은 3 가지를 설치한다.  
- git parameter
- pipeline stage view
- docker pipeline    

TODO : 플러그인 설치

<br/>

plugin 설정을 완료 하면 Manage jenkins -> Manage Nodes and Clouds 메뉴로 이동하여 Jenkins 설정을 한다.  

<br/>

<img src="./assets/manage_clouds1.png" style="width: 80%; height: auto;"/> 

<br/>

Configure Clouds로 이동한다.  

<br/>

<img src="./assets/manage_clouds2.png" style="width: 80%; height: auto;"/>   

<br/>

Configure Clouds에 가면  kubernetes Cloud Details 가 Jenkins Master 설정이고 Pod Template 에 Slave 설정을 하면 된다.  

<br/>

<img src="./assets/manage_clouds3.png" style="width: 80%; height: auto;"/> 


<br/>

kubernetes Cloud Details 를 확장하여 자세히 보면 Jenkins Master POD가 있는 namespace 를 확인 할 수 있다.  

<br/>

<img src="./assets/manage_clouds4.png" style="width: 80%; height: auto;"/> 

기본값으로 설정된 값을 유지한다.

<br/>

POD Labels의 key 값만 변경한다. 안해도 상관 없음  


<br/>

Pod Template 을 확장하여 Slave 설정을 시작한다.    
우리는 OKD 에 Jenkins를 설치를  하였고 OKD는 docker runtime 대신 CRI-O runtime을 사용하기 때문에 podman 으로 docker 를 대신한다.   

<br/>

podman-agent라는 이름으로 설정을 하고 Pod template details 를 클릭한다.   

<br/>


Pod Template Label은  podman-agent 로 설정한다.    

<br/>

하나의 POD에 2개의 컨테이너를 설정한다.
- JNLP 컨테이너 : jenkins slave 를 위한 컨테이너
- Podman 컨테이너 : Podman 프로그램이 설치된 컨테이너  
   - podman 컨테이너의 /etc/containers와 /var/lib/containers를 hostpath로 연결해 주어야 한다.
   - podman에서 worker node의 cri-o 런타임 엔진을 사용  
   
<br/>

먼저 jnlp 컨테이너 값을 설정한다.  
- docker image 를 변경한다 : jenkins/jnlp-slave:latest-jdk11  
- command to run 을 기존 sleep 값을 지운다.

<br/>


podman 컨테이너를 설정한다.    
- 이름을 설정한다 : podman
- docker image 를 설정한다 : mattermost/podman:1.8.0  
- Advanced 메뉴를 클릭하고 Run in privileged mode를 체크한다. (hostpath)  

<br/>

Add volume 버튼을 클릭하고 slave 용 pvc와 podman용 hostpath를 설정한다.   

slave용 pvc는 이미 설정이되어 있기 때문에 hostpath 만 설정해도 된다.  

<br/>


아래로 이동하여 service account를 `jenkins-admin` 으로 변경한다.  

workspace volume 도 helm 에서 사전에 설정 했기 때문에 확인만 하고 save 버튼을 눌러 저장한다.  

<br/>


#### Jenkins Pipeline 생성

<br/>

대쉬보드로 이동하여 신규 pipeline을 구성 한다.   

New Item 버튼을 클릭한다.

<br/>

Item 이름을 설정하고 pipeline 을 선택한 후 ok 버튼을 클릭한다.

<br/>

Git Parameter 를 설정 한다. 
Git Parameter 가 안보이는 수강생은 plugins 에서 git parameter를 설치해야 한다.  

<br/>


git Repostory는 본인의 git 주소를 입력하고 script path를 입력한다.    

강사 edu1 (https://github.com/shclub/edu1) repository의 Jenkinsfile_okd 화일을 참고하여 신규 생성한다.    
( 현재 OKD 네트웍 이슈로 Harbor 연결 불가 , dockerhub로 연동 하고 Jenkinsfile_dockerhub 참고 )

<br/>


credential은 Add 버튼을 클릭하여 github_ci 와 harbor_ci 라는 이름 으로 생성한다.  

harbor 가 구성이 안된 사람은  id는 `edu` , 비밀번호는 `New1234!` 로 구성하여 생성한다.  

<br/>

<img src="./assets/jenkins_pipeline5.png" style="width: 60%; height: auto;"/> 

<br/>

github의 Jenkinsfile_okd 는 본인의 환경에 맞게 변경한다.    
- def docker_registry = "https://211.252.85.148:40002"  
  - 예) edu1 : https://211.43.12.162:40002"
  - 예) 본인 Harbor 가 없으면 수정 하지 말것 
- def imageName = "211.252.85.148:40002/edu/edu1"
  - 예) edu1 : 211.43.12.162:40002/<본인프로젝트>/edu1
  - 예) 본인 Harbor 가 없으면서 순번이 2번인 경우 : 211.252.85.148:40002/edu2/edu1
- podTemplate의 namespace는 본인의 namespace 로 반드시 변경
- persistentVolumeClaim 의 claim은 본인의 slave claim으로 변경
  - 예) edu1 : jenkins-edu1-slave-pvc
- git url은 본인의 github repository 로 변경
- harbor credential은 다른 이름으로 생성했으면 그것에 맞추어 변경

<br/>


대쉬보드로 돌아와서 본인의 파이프 라인 클릭

<br/>

<br/>

Build with parameters 로 실행

<br/>

<br/>

성공적으로 진행이 된 것을 확인한다.  

<br/>

<br/>

Harbor로 이동하여 이미지가 정상적으로 Push 되었는지 확인한다.      

강사의 Harbor : https://211.252.85.148:40002/ 
- edu 계정으로 로그인
- 본인의 project로 이동하여 edu1 도커이미지 확인  

<br/>

<img src="./assets/jenkins_pipeline10.png" style="width: 60%; height: auto;"/> 

<br/>

강사의 Jenkjns : https://jenkins-edu30.apps.211-34-231-82.nip.io/   
- 계정 : admin
  


##  4주차

<br/>

### kubernetes에 CI/CD 구성 (CI : Maven/Skaffold/SonarQube , CD : ArgoCD/kustomize )


 
<br/>

VM에 로그인 한 후에 sonar 폴더를 생성한다.  

<br/>


```bash
root@newedu:~# mkdir -p sonar
root@newedu:~# cd sonar
``` 

<br/>

#### Helm 으로 PostgreSQL 설치

<br/>

helm repo 업데이트를 합니다.  

```bash
root@newedu:~/sonar# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jenkins" chart repository
...Successfully got an update from the "nfs-subdir-external-provisioner" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```  

<br/>


helm 에서 postgreSQL 를 검색합니다.

```bash
root@newedu:~/sonar# helm search repo postgresql
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/postgresql   	12.2.6       	15.2.0     	PostgreSQL (Postgres) is an open source object-...
bitnami/postgresql-ha	11.2.0       	15.2.0     	This PostgreSQL cluster solution includes the P...
bitnami/supabase     	0.1.4        	0.23.2     	Supabase is an open source Firebase alternative...
```  

<br/>

bitnami/postgresql 차트에서 차트의 변수 값을 변경하기 위해 postgre_values.yaml 화일을 추출한다.

<br/>


```bash
root@newedu:~/sonar#  helm show values bitnami/postgresql  > postgre_values.yaml
```

<br/>

postgre_values.yaml 를 수정한다.  

- 28 ,29,30,31 : 본인이 DB 계정 설정
- 646 : 본인의 pvc로 변경
- 669 : 5G로 사이즈 변경
- 694 : read replica 0

<br/>

```bash
  27     auth:
  28       postgresPassword: "edu1234"
  29       username: "edu"
  30       password: "edu1234"
  31       database: "edu"
  32       existingSecret: ""
 ... 
 640   persistence:
 641     ## @param primary.persistence.enabled Enable PostgreSQL Primary data persistence using PVC
 642     ##
 643     enabled: true
 644     ## @param primary.persistence.existingClaim Name of an existing PVC to use
 645     ##
 646     existingClaim: "postgre-edu-pvc"
 647     ## @param primary.persistence.mountPath The path the volume will be mounted at
 648     ## Note: useful when using custom PostgreSQL images
 649     ##
 650     mountPath: /bitnami/postgresql
 651     ## @param primary.persistence.subPath The subdirectory of the volume to mount to
 652     ## Useful in dev environments and one PV for multiple services
 653     ##
 654     subPath: ""
 655     ## @param primary.persistence.storageClass PVC Storage Class for PostgreSQL Primary data volume
 656     ## If defined, storageClassName: <storageClass>
 657     ## If set to "-", storageClassName: "", which disables dynamic provisioning
 658     ## If undefined (the default) or set to null, no storageClassName spec is
 659     ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
 660     ##   GKE, AWS & OpenStack)
 661     ##
 662     storageClass: ""
 663     ## @param primary.persistence.accessModes PVC Access Mode for PostgreSQL volume
 664     ##
 665     accessModes:
 666       - ReadWriteOnce
 667     ## @param primary.persistence.size PVC Storage Request for PostgreSQL volume
 668     ##
 669     size: 5Gi
...
 688 readReplicas:
 689   ## @param readReplicas.name Name of the read replicas database (eg secondary, slave, ...)
 690   ##
 691   name: read
 692   ## @param readReplicas.replicaCount Number of PostgreSQL read only replicas
 693   ##
 694   replicaCount: 0
 ```

<br/>

이제 postgreSQL DB를 설치 합니다.

<br/>

TODO

<br/>

pod를 확인한다.

```bash
root@newedu:~/sonar# kubectl get po
NAME                                        READY   STATUS    RESTARTS   AGE
sonar-postgre-postgresql-0                  1/1     Running   0          82s
```

<br/>

NFS 서버에 접속하여 data 폴더가 생성 되었는지 확인한다.  

```bash
[root@edu edu]# pwd
/mnt/postgre/edu
[root@edu edu]# ls data
PG_VERSION  pg_commit_ts   pg_logical    pg_replslot   pg_stat      pg_tblspc    pg_xact               postmaster.pid
base        pg_dynshmem    pg_multixact  pg_serial     pg_stat_tmp  pg_twophase  postgresql.auto.conf
global      pg_ident.conf  pg_notify     pg_snapshots  pg_subtrans  pg_wal       postmaster.opts
```  

<br/>


#### Helm 으로 SonarQube 설치


<br/>


helm repo 업데이트를 합니다.  

```bash
root@newedu:~/sonar# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jenkins" chart repository
...Successfully got an update from the "nfs-subdir-external-provisioner" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```  


<br/>

helm 에서 sonarqube 를 검색합니다.

```bash
root@newedu:~/sonar# helm search repo sonarqube
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION
NAME             	CHART VERSION	APP VERSION    	DESCRIPTION
bitnami/sonarqube	2.1.4        	9.9.0          	SonarQube(TM) is an open source quality managem...
```  

<br/>

bitnami/sonarqube 차트에서 차트의 변수 값을 변경하기 위해 sonarqube_values.yaml 화일을 추출한다.

<br/>


```bash
root@newedu:~/sonar#  helm show values bitnami/sonarqube  > sonarqube_values.yaml
```

<br/>

먼저 postgreSQL DB의 서비스 이름을 확인합니다.  


```bash
root@newedu:~/sonar# kubectl get svc
NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
jenkins                       NodePort    172.30.247.178   <none>        8080:30332/TCP   10d
jenkins-agent                 ClusterIP   172.30.140.40    <none>        50000/TCP        10d
sonar-postgre-postgresql      ClusterIP   172.30.225.163   <none>        5432/TCP         19m
```  


<br/>

sonarqube_values.yaml 를 수정한다.  

- 708 : pvc 사용으로 true
- 722 : 5G로 사이즈 변경
- 728 : 본인의 pvc로 변경
- 1004 : 이미 postgresql db 설치 했기 때문에 false
- 1065 : postgresql db 서비스 이름 ( 위에서 조회 )

<br/>

```bash
 706 persistence:
 707   ## @param persistence.enabled Enable persistence using Persistent Volume Claims
 708   ##
 709   enabled: true
 710   ## @param persistence.storageClass Persistent Volume storage class
 711   ## If defined, storageClassName: <storageClass>
 712   ## If set to "-", storageClassName: "", which disables dynamic provisioning
 713   ## If undefined (the default) or set to null, no storageClassName spec is set, choosing the default provisioner
 714   ##
 715   storageClass: ""
 716   ## @param persistence.accessModes [array] Persistent Volume access modes
 717   ##
 718   accessModes:
 719     - ReadWriteOnce
 720   ## @param persistence.size Persistent Volume size
 721   ##
 722   size: 5Gi
 723   ## @param persistence.dataSource Custom PVC data source
 724   ##
 725   dataSource: {}
 726   ## @param persistence.existingClaim The name of an existing PVC to use for persistence
 727   ##
 728   existingClaim: "sonar-edu-pvc"
...
1001 postgresql:
1002   ## @param postgresql.enabled Deploy PostgreSQL subchart
1003   ##
1004   enabled: false
1005   ## @param postgresql.nameOverride Override name of the PostgreSQL chart
...
1062 externalDatabase:
1063   ## @param externalDatabase.host Host of an external PostgreSQL instance to connect (only if postgresql.enabled=false)
1064   ##
1065   host: "sonar-postgre-postgresql"
1066   ## @param externalDatabase.user User of an external PostgreSQL instance to connect (only if postgresql.enabled=false)
1067   ##
1068   user: edu
1069   ## @param externalDatabase.password Password of an external PostgreSQL instance to connect (only if postgresql.enabled=fa     lse)
1070   ##
1071   password: "edu1234"
1072   ## @param externalDatabase.existingSecret Secret containing the password of an external PostgreSQL instance to connect (o     nly if postgresql.enabled=false)
1073   ## Name of an existing secret resource containing the DB password in a 'password' key
1074   ##
1075   existingSecret: ""
1076   ## @param externalDatabase.database Database inside an external PostgreSQL to connect (only if postgresql.enabled=false)
1077   ##
1078   database: edu
 ```

<br/>

sonarqube를 설치 하기 전에 sonarqube service account 에게 권한을 부여합니다.  

```bash
root@newedu:~/sonar# oc adm policy add-scc-to-user anyuid -z sonarqube
root@newedu:~/sonar# oc adm policy add-scc-to-user privileged -z sonarqube
```  

<br/>

이제 sonarqube 를 설치 합니다.


```bash
root@newedu:~/sonar# helm install sonarqube bitnami/sonarqube -f sonarqube_values.yaml
NAME: sonarqube
LAST DEPLOYED: Mon Mar 27 10:51:23 2023
NAMESPACE: edu30
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Your SonarQube(TM) site can be accessed through the following DNS name from within your cluster:

    sonarqube.edu30.svc.cluster.local (port 80)

To access your SonarQube(TM) site from outside the cluster follow the steps below:

1. Get the SonarQube(TM) URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace edu30 -w sonarqube'

   export SERVICE_IP=$(kubectl get svc --namespace edu30 sonarqube --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
   echo "SonarQube(TM) URL: http://$SERVICE_IP/"

2. Open a browser and access SonarQube(TM) using the obtained URL.

3. Login with the following credentials below:

  echo Username: user
  echo Password: $(kubectl get secret --namespace edu30 sonarqube -o jsonpath="{.data.sonarqube-password}" | base64 -d)
  ```

<br/>

pod를 확인한다.

```bash
root@newedu:~/sonar# kubectl get po
NAME                                        READY   STATUS    RESTARTS   AGE
jenkins-0                                   2/2     Running   26         9d
nfs-test-589c488d6f-8lk5p                   1/1     Running   2          10d
sonar-postgre-postgresql-0                  1/1     Running   0          54m
sonarqube-5d48b66455-5sgzh                  1/1     Running   0          3m7s
```

<br/>

NFS 서버에 접속하여 data 폴더가 생성 되었는지 확인한다.  

```bash
[root@edu edu]# pwd
/mnt/sonar/edu
[root@edu edu]# ls
data  extensions
```  

<br/>

서비스를 조회해서 LoadBalancer Type을 NodePort로 변경합니다.  

```bash
root@newedu:~/sonar# kubectl get svc
NAME                          TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                       AGE
jenkins                       NodePort       172.30.247.178   <none>        8080:30332/TCP                10d
jenkins-agent                 ClusterIP      172.30.140.40    <none>        50000/TCP                     10d
sonar-postgre-postgresql      ClusterIP      172.30.225.163   <none>        5432/TCP                      64m
sonar-postgre-postgresql-hl   ClusterIP      None             <none>        5432/TCP                      64m
sonarqube                     LoadBalancer   172.30.154.233   <pending>     80:30262/TCP,9001:30118/TCP   13m
``` 

<br/>

```bash
root@newedu:~/sonar# kubectl edit svc sonarqube
service/sonarqube edited
``` 

<br/>

```bash
root@newedu:~/sonar# kubectl get svc
NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                       AGE
jenkins                       NodePort    172.30.247.178   <none>        8080:30332/TCP                10d
jenkins-agent                 ClusterIP   172.30.140.40    <none>        50000/TCP                     10d
sonar-postgre-postgresql      ClusterIP   172.30.225.163   <none>        5432/TCP                      64m
sonar-postgre-postgresql-hl   ClusterIP   None             <none>        5432/TCP                      64m
sonarqube                     NodePort    172.30.154.233   <none>        80:30262/TCP,9001:30118/TCP   13m
```  

<br/>

웹브라우저에서 http://211.34.231.84:30262/로 접속하여 로그인합니다.   

- id 는 user
- 비밀번호는 아래와 같이 추출합니다.
  ```bash
  root@newedu:~/sonar# kubectl get secret  sonarqube -o jsonpath="{.data.sonarqube-password}" | base64 -d
  agLMkOCVqv
  ```  

<br/>


<img src="./assets/sonarqube1.png" style="width: 80%; height: auto;"> 

<br/>


####  SonarQube 로 프로젝트 구성


<br/>

참고
- https://docs.sonarqube.org/9.8/analyzing-source-code/scanners/sonarscanner-for-maven/
- https://twofootdog.tistory.com/15
- https://happy-jjang-a.tistory.com/26

<br/>

SonarQube에 로그인 한 후에 제일 먼저 Admin 비밀빌번호를 변경한다.  
Account ->  My Account 클릭한 후 security tab으로 이동한다.  

<img src="./assets/sonarqube_token1.png" style="width: 60%; height: auto;"> 

<br/>

비밀번호를 변경한다.  

<img src="./assets/sonarqube3.png" style="width: 80%; height: auto;"> 

<br/>

프로젝트를 생성한다.  manual 로 만든다.

<img src="./assets/sonarqube4.png" style="width: 80%; height: auto;"> 


<br/>

원하는 이름을 넣고 branch에는 master를 입력한다.

<img src="./assets/sonarqube5.png" style="width: 80%; height: auto;"> 


<br/>

그 다음 with Jenkins를 클릭하여 필요한 정보를 확인한다.  설정은 별도로 안해도 된다.  

<img src="./assets/sonarqube6.png" style="width: 80%; height: auto;"> 


<br/><br/>

Jenkins와 연동하기 위해서는 Token 값을 생성해야 합니다.  
계정으로 이동합니다.

<img src="./assets/sonarqube_token1.png" style="width: 80%; height: auto;"> 

<br/>

security 탭으로 이동합니다.  

<img src="./assets/sonarqube_token2.png" style="width: 80%; height: auto;"> 

<br/>

Type은 Project Analysys Token 으로 설정하고 Generate 버튼을 클릭합니다. 

<img src="./assets/sonarqube_token3.png" style="width: 80%; height: auto;"> 


<br/>

생성된 Token 값을 COPY 버튼을 클릭하여 복사하여 적당하 곳에 저장합니다.  

<img src="./assets/sonarqube_token4.png" style="width: 80%; height: auto;"> 

<br/>


SonarQube 프로젝트를 구성하기 위해서는 backend 인 SpringBoot의 pom.xml 화일에 dependency를 추가 한다.  


```xml
		<plugin>
				<groupId>org.sonarsource.scanner.maven</groupId>
				<artifactId>sonar-maven-plugin</artifactId>
				<version>3.7.0.1746</version>
		</plugin>
``` 

<br/>

이제 jenkins를 설정합니다.  

Jenkins는 먼저  SonarQube plugins 을 설치합니다.   
- Sonarqube Scanner
- Sonar Quality Gates

<img src="./assets/sonarqube_jenkins1.png" style="width: 80%; height: auto;"> 


<br/>

Manage Jenkins -> Configure System 으로 이동하여 SonarQube 서버 설정을 합니다.  

<img src="./assets/sonarqube_jenkins2.png" style="width: 80%; height: auto;"> 


<br/>

Add Credential 을 추가 하는데 Secret Text로 설정하여 SonarQube Token 값을 설정 합니다.  

<img src="./assets/sonarqube_jenkins3.png" style="width: 80%; height: auto;"> 


<br/>

설정한 정보가 맞는지 확인 하고 save 합니다. 

<img src="./assets/sonarqube_jenkins4.png" style="width: 80%; height: auto;">

<br/>

Manage Jenkins -> Global Tool Configuration 이동하여 Sonar Scanner 설정을 합니다.  

<img src="./assets/sonarqube_jenkins7.png" style="width: 80%; height: auto;"> 

<br/>

이제 설정이 완료 되었음으로 Dashboard -> New Item 으로 이동하여 새로운 Pipeline을 생성합니다.  edu-backend pipeline을 복사하여 생성하며 jenkins 화일 이름만 변경 합니다.  

<br/>
 
Build with paramameter를 클릭하여 실행을 하고 console Output 을 확인합니다.   

아래와 같은 메시지가 나오면 연동이 성공 한 것입니다.  


```bash
Downloaded from central: https://repo.maven.apache.org/maven2/org/sonarsource/scanner/api/sonar-scanner-api/2.14.0.2002/sonar-scanner-api-2.14.0.2002.jar (625 kB at 42 MB/s)
Downloaded from central: https://repo.maven.apache.org/maven2/org/sonatype/plexus/plexus-sec-dispatcher/1.4/plexus-sec-dispatcher-1.4.jar (28 kB at 792 kB/s)
[INFO] User cache: /root/.sonar/cache
[INFO] SonarQube version: 9.9.0.65466
[INFO] Default locale: "en_US", source code encoding: "UTF-8"
[INFO] Load global settings
[INFO] Load global settings (done) | time=126ms
[INFO] Server id: 296AEF55-AYcgz2EaLi3MhYbotBq5
[INFO] User cache: /root/.sonar/cache
[INFO] Load/download plugins
[INFO] Load plugins index
[INFO] Load plugins index (done) | time=67ms
[INFO] Load/download plugins (done) | time=1142ms
[INFO] Process project properties
[INFO] Process project properties (done) | time=10ms
[INFO] Execute project builders
[INFO] Execute project builders (done) | time=2ms
[INFO] Project key: edu
[INFO] Base dir: /home/jenkins/agent/workspace/edu_backend_sonar
[INFO] Working dir: /home/jenkins/agent/workspace/edu_backend_sonar/target/sonar
[INFO] Load project settings for component key: 'edu'
[INFO] Load project settings for component key: 'edu' (done) | time=26ms
[INFO] Auto-configuring with CI 'Jenkins'
[INFO] Load quality profiles
[INFO] Load quality profiles (done) | time=69ms
[INFO] Load active rules
[INFO] Load active rules (done) | time=1556ms
[INFO] Load analysis cache
[INFO] Load analysis cache (404) | time=9ms
[INFO] Load project repositories
[INFO] Load project repositories (done) | time=22ms
[INFO] Indexing files...
[INFO] Project configuration:
[INFO] 25 files indexed
[INFO] 0 files ignored because of scm ignore settings
[INFO] Quality profile for java: Sonar way
[INFO] Quality profile for xml: Sonar way
[INFO] ------------- Run sensors on module thirdproject
[INFO] Load metrics repository
[INFO] Load metrics repository (done) | time=20ms
[INFO] Sensor JavaSensor [java]
[INFO] Configured Java source version (sonar.java.source): 17
[INFO] JavaClasspath initialization
[INFO] JavaClasspath initialization (done) | time=6ms
[INFO] JavaTestClasspath initialization
[INFO] JavaTestClasspath initialization (done) | time=3ms
[INFO] Server-side caching is enabled. The Java analyzer will not try to leverage data from a previous analysis.
[INFO] Using ECJ batch to parse 22 Main java source files with batch size 189 KB.
[INFO] Starting batch processing.
[INFO] The Java analyzer cannot skip unchanged files in this context. A full analysis is performed for all files.
[INFO] 100% analyzed
[INFO] Batch processing: Done.
[INFO] Did not optimize analysis for any files, performed a full analysis for all 22 files.
[WARNING] Use of preview features have been detected during analysis. Enable DEBUG mode to see them.
[INFO] Using ECJ batch to parse 2 Test java source files with batch size 189 KB.
[INFO] Starting batch processing.
[INFO] 100% analyzed
[INFO] Batch processing: Done.
[INFO] Did not optimize analysis for any files, performed a full analysis for all 2 files.
[INFO] No "Generated" source files to scan.
[INFO] Sensor JavaSensor [java] (done) | time=2181ms
[INFO] Sensor JaCoCo XML Report Importer [jacoco]
[INFO] 'sonar.coverage.jacoco.xmlReportPaths' is not defined. Using default locations: target/site/jacoco/jacoco.xml,target/site/jacoco-it/jacoco.xml,build/reports/jacoco/test/jacocoTestReport.xml
[INFO] No report imported, no coverage information will be imported by JaCoCo XML Report Importer
[INFO] Sensor JaCoCo XML Report Importer [jacoco] (done) | time=2ms
[INFO] Sensor CSS Rules [javascript]
[INFO] No CSS, PHP, HTML or VueJS files are found in the project. CSS analysis is skipped.
[INFO] Sensor CSS Rules [javascript] (done) | time=0ms
[INFO] Sensor C# Project Type Information [csharp]
[INFO] Sensor C# Project Type Information [csharp] (done) | time=1ms
[INFO] Sensor C# Analysis Log [csharp]
[INFO] Sensor C# Analysis Log [csharp] (done) | time=11ms
[INFO] Sensor C# Properties [csharp]
[INFO] Sensor C# Properties [csharp] (done) | time=0ms
[INFO] Sensor SurefireSensor [java]
[INFO] parsing [/home/jenkins/agent/workspace/edu_backend_sonar/target/surefire-reports]
[INFO] Sensor SurefireSensor [java] (done) | time=55ms
[INFO] Sensor HTML [web]
[INFO] Sensor HTML [web] (done) | time=2ms
[INFO] Sensor XML Sensor [xml]
[INFO] 1 source file to be analyzed
[INFO] 1/1 source file has been analyzed
[INFO] Sensor XML Sensor [xml] (done) | time=217ms
[INFO] Sensor TextAndSecretsSensor [text]
[INFO] 25 source files to be analyzed
[INFO] 25/25 source files have been analyzed
[INFO] Sensor TextAndSecretsSensor [text] (done) | time=42ms
[INFO] Sensor VB.NET Project Type Information [vbnet]
[INFO] Sensor VB.NET Project Type Information [vbnet] (done) | time=1ms
[INFO] Sensor VB.NET Analysis Log [vbnet]
[INFO] Sensor VB.NET Analysis Log [vbnet] (done) | time=11ms
[INFO] Sensor VB.NET Properties [vbnet]
[INFO] Sensor VB.NET Properties [vbnet] (done) | time=0ms
[INFO] Sensor IaC Docker Sensor [iac]
[INFO] 0 source files to be analyzed
[INFO] 0/0 source files have been analyzed
[INFO] Sensor IaC Docker Sensor [iac] (done) | time=47ms
[INFO] ------------- Run sensors on project
[INFO] Sensor Analysis Warnings import [csharp]
[INFO] Sensor Analysis Warnings import [csharp] (done) | time=0ms
[INFO] Sensor Zero Coverage Sensor
[INFO] Sensor Zero Coverage Sensor (done) | time=9ms
[INFO] Sensor Java CPD Block Indexer
[INFO] Sensor Java CPD Block Indexer (done) | time=46ms
[INFO] SCM Publisher SCM provider for this project is: git
[INFO] SCM Publisher 25 source files to be analyzed
[INFO] SCM Publisher 25/25 source files have been analyzed (done) | time=322ms
[INFO] CPD Executor 10 files had no CPD blocks
[INFO] CPD Executor Calculating CPD for 12 files
[INFO] CPD Executor CPD calculation finished (done) | time=7ms
[INFO] Analysis report generated in 50ms, dir size=217.6 kB
[INFO] Analysis report compressed in 34ms, zip size=79.5 kB
[INFO] Analysis report uploaded in 53ms
[INFO] ANALYSIS SUCCESSFUL, you can find the results at: http://211.34.231.84:30262/dashboard?id=edu
[INFO] Note that you will be able to access the updated dashboard once the server has processed the submitted analysis report
[INFO] More about the report processing at http://211.34.231.84:30262/api/ce/task?id=AYchJdLGw_Vp_k2kP2ZW
[INFO] Analysis total time: 7.425 s
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  51.927 s
[INFO] Finished at: 2023-03-27T12:38:59+09:00
[INFO] ------------------------------------------------------------------------
```  


<br/>

SonarQube로 이동하여 데이터를 확인해 봅니다.    
Thirdproject 라는 이름으로 하나가 생성된 것을 볼수 있습니다.  


<img src="./assets/sonarqube_jenkins6.png" style="width: 80%; height: auto;"> 

<br/>

프로젝트를 클릭하여 좀 더 자세한 정보를 볼 수 있습니다.  

<img src="./assets/sonarqube_jenkins5.png" style="width: 80%; height: auto;">  


<br/>

Jenkins Pipeline은 아래와 같습니다.   
기존 backend pipeline 에 SonarQube가 추가가 되었습니다. 

<br/>

```bash
        ...
        stage('SonarQube Analysis') {
           container('build-tools') {
               withSonarQubeEnv('sonarqube'){ // 시스템설정 값
                 sh "./mvnw clean verify sonar:sonar -Dsonar.projectKey=edu"
             }
           }  
        }
        ...
```  

<br/>

전체 Pipeline 입니다.

<br/>

```bash
def label = "agent-${UUID.randomUUID().toString()}"
def gitBranch = 'master'
def docker_registry = "ghcr.io"  
def imageName = "ghcr.io/shclub/edu12-backend"
def fromImage = "ghcr.io/shclub/jre17-runtime:v1.0.0"
def git_ops_name = "edu12-backend-gitops"
def P_NAMESPACE = "edu30"

def TAG = getTag(gitBranch)

podTemplate(label: label, serviceAccount: 'jenkins-admin', namespace: P_NAMESPACE,
    containers: [
        containerTemplate(name: 'build-tools', image: 'ghcr.io/shclub/build-tool:v1.0.0', ttyEnabled: true, command: 'cat', privileged: true, alwaysPullImage: true)
        ,containerTemplate(name: 'jnlp', image: 'ghcr.io/shclub/jenkins/jnlp-slave:latest-jdk11', args: '${computer.jnlpmac} ${computer.name}')
    ],
    volumes: [
        hostPathVolume(hostPath: '/etc/containers' , mountPath: '/var/lib/containers' ),
        persistentVolumeClaim(mountPath: '/var/jenkins_home', claimName: 'jenkins-edu-slave-pvc',readOnly: false)
        ]){    
  
    
    node(label) { 
        stage('SCM') {
           checkout scm
       }  
       
        stage('SonarQube Analysis') {
           container('build-tools') {
               withSonarQubeEnv('sonarqube'){ // 시스템설정 값
                 sh "./mvnw clean verify sonar:sonar -Dsonar.projectKey=edu"
             }
           }  
        }
        
       stage('Maven Build & Image Push ') {
            container('build-tools') {
               withCredentials([usernamePassword(credentialsId: 'github_ci',usernameVariable: 'USERNAME',passwordVariable: 'PASSWORD')]) {
                    sh  """
                         ./mvnw clean package jib:build  -Dmaven.test.skip=true  \
                         -Djib.from.image=${fromImage} \
                         -Djib.from.auth.username=${USERNAME} \
                         -Djib.from.auth.password=${PASSWORD} \
                         -Djib.to.image=${imageName} \
                         -Djib.to.tags=${TAG}  \
                         -Djib.to.auth.username=${USERNAME} \
                         -Djib.to.auth.password=${PASSWORD}
                         echo 'TAG ==========> ' ${TAG}
                   """
              }
            }
        }

        stage('GitOps update') {
            container('build-tools') {
               withCredentials([usernamePassword(credentialsId: 'github_ci',usernameVariable: 'USERNAME',passwordVariable: 'PASSWORD')]) {
                    sh """  
                        cd ~
                        git clone https://${USERNAME}:${PASSWORD}@github.com/${USERNAME}/${git_ops_name}
                        cd ${git_ops_name}
                        git checkout HEAD
                        kustomize edit set image ${imageName}:${TAG}
                        git config --global user.email "shclub@gmail.com"
                        git config --global user.name ${USERNAME}
                        git add .
                        git commit -am 'update image tag  ${TAG} from My_Jenkins'
                        cat kustomization.yaml
                        git push origin HEAD
                    """
               } 
            }
        }
        
        
    }
}

def getTag(branchName){     
    def TAG
    def DATETIME_TAG = new Date().format('yyyyMMddHHmmss')
    TAG = "${DATETIME_TAG}"
    return TAG
}  
```