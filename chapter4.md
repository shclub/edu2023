# Chapter 4 
   

kubernetes 는 FlyingCube 2.0 ( OKD 4.7 ) 로 실습을 진행하다.  

1. kubernetes 소개

2. Kubernetes 아키텍처

3. yaml 템플릿

4. Kubernetes 의 API 서버 보안 ( Service Account )

5. Kubernetes 의 Downward API   


<br/>

> 참고
  - 따배쿠 : https://youtu.be/Ia8IfowgU7s
  - https://youtu.be/Sj9Z6-w1VUE
  - https://subicura.com/k8s/

<br/>
 

##  kubernetes 소개
 

<br/>

### 쿠버네티스 시작하기

개발 환경에서 당연하게 사용해왔던 쿠버네티스에 대해 이해하고, 왜 쿠버네티스를 사용하는지 알아보자.  

<br/>

개요
- 컨테이너 오케스트레이션의 개념과 사용하는 이유 그리고 특징에 대해 이해한다.

<br/>

소개
- 쿠버네티스는 컨테이너 오케스트레이션 툴의 한 종류이며 엄청난 인기로 사실상 표준으로 사용된다.
- 컨테이너 오케스트레이션은 복잡한 컨테이너 환경을 효과적으로 관리하기 위한 도구이다.  

배경  
- 서버의 상태를 편하게 관리하기 위한 노력으로 도커 컨테이너가 등장했다.
- 그러나 관리하는 서버 컨테이너 수가 점점 증가하며 관리가 힘들다는 문제가 생겼다.
- 그래서 등장한 것이 컨테이너 오케스트레이션이다.  

<br/>

컨테이너의 특징  
- 가상머신과 비교하여 컨테이너 생성이 쉽고 효율적
- 컨테이너 이미지를 이용한 배포와 롤백이 간단
- 언어나 프레임워크에 상관없이 애플리케이션을 동일한 방식으로 관리
- 개발, 테스팅, 운영 환경을 물론 로컬 피시와 클라우드까지 동일한 환경을 구축
- 특정 클라우드 벤더에 종속적이지 않음
  

<br/>

### 컨테이너 오케스트레이션

<br/>

컨테이너 오케스트레이션을 사용하지 않는다면

- 배포(Deployment)

    <img src="./assets/k8s_overview_1.png" style="width: 80%; height: auto;"/>  

    - 각 서버의 ip를 찾고 각 서버에 ssh로 접속해서 docker 명령어로 컨테이너를 실행 및 종료하는 수고가 든다.
    - 만약 새로운 컨테이너를 실행하려면 빈 서버(여유있는)에 실행 하는것이 리소스 절약을 위해 좋으나, 이를 확인하기 위한 모니터링 툴이 없으면 빈 서버를 일일히 찾기도 힘들다.
    - 서버를 배포할 때(version upgrade 또는 rollback) 모든 서버를 한번에 배포하는 방법이 필요하다.  

<br/>

- 서비스 검색(Service Discovery)  

    <img src="./assets/k8s_overview_2.png" style="width: 80%; height: auto;"/>  

    - 보통의 구조라면 프록시 서버가 있고 로드밸런서를 통해 서버에 적절히 부하를 분산한다.
    - 그러나 로드밸런서와 서버의 ip 설정과 같은 부분이 관리자가 직접 관리해야하는 포인트였다.
    - 마이크로서비스 환경이 유행처럼 등장하며 서버가 점점 많아지고 서버의 ip가 업데이트로 변경되고 하면서 관리자가 이를 모두 관리하는 것은 쉽지 않았다.

<br/>

- 서비스 노출(Gateway)

    <img src="./assets/k8s_overview_3.png" style="width: 80%; height: auto;"/>  

    - NginX와 같이 외부에 노출된 프록시 서버를 두고, 프록시 서버로 들어오는 host 요청에 따라 내부 컨테이너(서버)에 연결하는데 이 과정에 자동화가 필요했다.
 
<br/>

- 서비스 이상과 부하 모니터링

    <img src="./assets/k8s_overview_4.png" style="width: 80%; height: auto;"/>  

    - 갑자기 컨테이너가 죽은 경우에 이전에는 일일히 로그 확인해서 다시 서버를 띄워야 했다.
        - 같은 서버 컨테이너가 3개 돌다가 하나의 컨테이너가 죽으면 남은 2개의 서버에 부하가 생긴다.
    - 서버가 죽지는 않았는데 트래픽이 많아지면 부하가 걸려 느려졌다.
        - 트래픽에 따라 적절하게 서버를 늘려야 함
    - 위와 같은 상황으로 자동화가 필요했다.

 
<br/>

컨테이너 오케스트레이션  

<br/>

- 위와같은 문제로 많은 컨테이너를 효율적으로 관리하기 위한 기술이 컨테이너 오케스트레이션이다.
- 컨테이너 오케스트레이션 복잡한 컨테이너 환경을 효과적으로 관리하기 위한 도구이다.

<br/>

컨테이너 오케스트레이션 특징  

- 클러스터(Cluster)  

    <img src="./assets/k8s_overview_5.png" style="width: 80%; height: auto;"/>  

    - 중앙 제어
        - 이전에는 서로 다른 노드의 CPU와 RAM 상태를 각각 관리했었다.
        - 그러나 노드 수가 증가하면 힘들기 때문에, 컨테이너 오케스트레이션에서는 합쳐서 추상화하여 클러스트 단위로 관리한다.
        - 클러스터 하나하나의 노드에 ssh로 직접 접속하기 힘들기때문에 프록시처럼 앞에 마스터 서버를 두고 마스터서버가 각 노드에 알아서 명령을 보낸다.  

    - 네트워킹
        - 클러스터 내 노드끼리는 서로 통신이 잘되어야 한다.
    - 노드 스케일
        - 노드 스케일이 커지더라도 잘 돌아가기 위해서는 정교한 설계가  필요하다.  

<br/>

- 상태 관리(State)  

    <img src="./assets/k8s_overview_6.png" style="width: 80%; height: auto;"/>  

    - 트래픽이 증가해 서버를 새로 늘리거나, 하나의 서버에 장애가 발생했을 때 감지하고 자동으로 서버를 늘려준다.
    - 오토 스케일링

<br/>

- 배포 관리(Scheduling)  

    <img src="./assets/k8s_overview_7.png" style="width: 80%; height: auto;"/>  

    - 자원이 여유가 있는 서버에 알아서 적절하게 띄워준다.

<br/>

- 배포 버전관리(Rollout & Rollback)  

    <img src="./assets/k8s_overview_8.png" style="width: 80%; height: auto;"/>  

    - 전체 컨테이너에 대한 롤아웃과 롤백을 중앙에서 관리한다.

<br/>

- 서비스 등록 및 조회(Service Discovery)  

    <img src="./assets/k8s_overview_9.png" style="width: 80%; height: auto;"/>  

    - 새로 등록된 서비스 ip나 변경된 ip를 자동으로 관리해줘서 관리자가 하나하나 수정할필요가 없다.


<br/>

- 볼륨 스토리지(Volume)  

    <img src="./assets/k8s_overview_10.png" style="width: 80%; height: auto;"/>  

    - 각 서버에 필요한 스토리지(AWS EBS, GCE 등)를 설정으로 연결할 수 있다.
    - AWS에서 EC2에 스토리지 일일히 하나하나 붙이고 그럴 필요가 없다는 말이다.  

<br/>

왜 쿠버네티스인가 ? 

<br/>

- 컨테이너 관리도구가 많이 생기고 했으나, 쿠버네티스가 표준처럼 등장했다.
- 오픈소스  

    - 컨테이너를 쉽고 빠르게 배포/확장하고 관리를 자동화해주는 오픈소스 플랫폼
    - 구글에서 만듬 (구글은 1주일에 20억개 컨테이너 생성한다.)  

- 엄청난 인기
    - 점유율이 높고 그렇기에 라이브러리 또는 레퍼런스가 많다.  

- 무한한 확장성
    - 쿠버네티스 위에서 머신러닝, CI/CD, 서비스. 서버리스 등 다양한 서비스가 동작
    - 쿠버네티스만 설치되어 있으면 거기에 서비스를 올리기 쉬움  

- 사실상 표준(de facto)
    - 많은 오케스트레이션이 있지만 사실상 표준이 되어가고 있음
    - 도커도 자체 오케스트레이션이 있지만 쿠버네티스의 인기로 인해 어쩔수없이 쿠버네티스 지원
    - AWS(Elastic Kubernetes Service), Azure(Azure Kubernetes Service), Google(Google Kubernetes Engine)와 같이 대표적인 클라우드 기업들이 쿠버네티스를 매니지드 서비스로 제공하고 있음  

<br/>

## kubernetes 아키텍처

<br/>

### kubernetes Cluster 구성

<br/>

참고
- 출처 : https://www.redhat.com/ko/topics/containers/kubernetes-architecture


<br/>

kubernetes의 가장 큰 단위는 cluster 이고 cluster는 서버들의 모임이다.  

<br/>

<img src="./assets/k8s_cluster.png" style="width: 80%; height: auto;"/> 

<br/>

#### 컨트롤 플레인 ( Control Plane )

<br/>

Master Node 들의 모임이며 최근에는 Master 라는 말 보다는 Control Plane 이라는 용어를 사용한다.    

<br/>

클러스터를 제어하는 쿠버네티스 구성 요소와 클러스터의 상태 및 구성에 관한 데이터가 함께 있습니다. 이 핵심 쿠버네티스 구성 요소는 컨테이너가 필요한 리소스를 갖고 충분한 횟수로 실행되도록 하는 중요한 작업을 맡습니다.   

<br/>

컨트롤 플레인은 컴퓨팅 노드와 상시 연결되어 있습니다. 클러스터가 일정한 방식으로 실행되도록 구성했다면 컨트롤 플레인은 해당 방식에 따라 실행.  

<br>

구성요소 
- kube-apiserver 
  - 쿠버네티스 클러스터와 상호 작용해야 하나요? API에 요청하세요. 쿠버네티스 API는 쿠버네티스 컨트롤 플레인의 프론트엔드로, 내부 및 외부 요청을 처리합니다.   
  - API 서버는 요청이 유효한지 판별하고 유효한 요청을 처리합니다. REST 호출이나 kubectl 커맨드라인 인터페이스 또는 kubeadm과 같은 기타 CLI(command-line interface)를 통해 API에 액세스할 수 있습니다.    

<br/>

- kube-scheduler
  - 클러스터가 양호한 상태인가? 새 컨테이너가 필요하다면 어디에 적합한가? 쿠버네티스 스케줄러는 이러한 것들을 주로 다룹니다.  

  - 스케줄러는 CPU 또는 메모리와 같은 포드의 리소스 요구 사항과 함께 클러스터의 상태를 고려합니다. 그런 다음 포드를 적절한 컴퓨팅 노드에 예약합니다.   

<br/>

- kube-controller-manager
  - 컨트롤러는 실제로 클러스터를 실행하고 쿠버네티스 controller-manager에는 여러 컨트롤러 기능이 하나로 통합되어 있습니다.   
  - 하나의 컨트롤러는 스케줄러를 참고하여 정확한 수의 포드가 실행되게 합니다.     
  - 포드에 문제가 생기면 또 다른 컨트롤러가 이를 감지하고 대응합니다.   
  - 컨트롤러는 서비스를 포드에 연결하므로 요청이 적절한 엔드포인트로 이동합니다.  
  - 또한 계정 및 API 액세스 토큰 생성을 위한 컨트롤러가 있습니다.  

<br/>

- etcd
  - 설정 데이터와 클러스터의 상태에 관한 정보는 키-값 저장소 데이터베이스인 etcd에 상주합니다.   

  - 내결함성을 갖춘 분산형 etcd는 클러스터에 관한 궁극적 정보 소스(Source Of Truth, SOT)가 되도록 설계되었습니다.  

  - k3s 같은 경량 k8s 배포판은 etcd 대신 sqlite 를 사용하기도 합니다.     


<br/>

#### 컴퓨트 플레인 ( Compute Plane )

<br/>

Worker Node 들의 모임이며 최근에는 Worker 라는 말 보다는 Compute Plane 또는 Data Plane 이라는 용어를 사용한다.    

<br/>

구성요소  
- 노드 ( Node )
 - 쿠버네티스 클러스터에는 최소 1개 이상의 컴퓨팅 노드가 필요하지만 일반적으로 여러 개가 있습니다. Pod는 노드에서 실행하도록 예약되고 오케스트레이션됩니다.     
 - 클러스터의 용량을 확장해야 한다면 노드를 더 추가하면 됩니다.  
 - 일반적으로 노드 하나는 VM ( 또는 Dedi 서버 ) 1개입 니다.  
 - 각 노드에서 OS 가 설치 됩니다. ( CentOS , Ubuntu , CoreOS 등 )

<br/>

- Pod
 - 파드는 쿠버네티스 오브젝트 모델에서 가장 작고 단순한 유닛으로, 애플리케이션의 단일 인스턴스를 나타냅니다. 

 - 각 파드는 컨테이너 실행 방식을 제어하는 옵션과 함께 컨테이너 하나 또는 긴밀히 결합된 일련의 컨테이너로 구성되어 있습니다.   

 - 파드를 퍼시스턴트 스토리지에 연결하여 스테이트풀(stateful) 애플리케이션을 실행할 수 있습니다.

<br/>

- 컨테이너 런타임 엔진
 - 컨테이너 실행을 위해 각 컴퓨팅 노드에는 컨테이너 런타임 엔진이 있습니다. 그중 한 가지 예가 Docker입니다.  
 - 하지만 쿠버네티스는 rkt, CRI-O와 같은 다른 Open Container Initiative 호환 런타임도 지원합니다. OKD는 CRI-O를 사용 한다.  

<br/>

- kubelet
 - 각 컴퓨팅 노드에는 컨트롤 플레인과 통신하는 매우 작은 애플리케이션인 kubelet이 있습니다. kublet은 컨테이너가 포드에서 실행되게 합니다.   
 - 컨트롤 플레인에서 노드에 작업을 요청하는 경우 kubelet이 이 작업을 실행합니다.

<br/>

- kube-proxy
 - 각 컴퓨팅 노드에는 쿠버네티스 네트워킹 서비스를 용이하게 하기 위한 네트워크 프록시인 kube-proxy도 있습니다.  
- kube-proxy는 운영 체제의 패킷 필터링 계층에 의존하거나 트래픽 자체를 전달하여 클러스터 내부 또는 외부의 네트워크 통신 (iptables) 을 처리합니다.  
  

<br/>

#### Addon

<br/>

애드온은 쿠버네티스 리소스(데몬셋, 디플로이먼트 등)를 이용하여 클러스터 기능을 구현한다. 이들은 클러스터 단위의 기능을 제공하기 때문에 애드온에 대한 네임스페이스 리소스는 kube-system 네임스페이스에 속한다.  

<br/>

선택된 일부 애드온은 아래에 설명하였고, 사용 가능한 전체 확장 애드온 리스트는 애드온을 참조한다.

<br/>

- DNS (coreDNS )
  - 여타 애드온들이 절대적으로 요구되지 않지만, 많은 예시에서 필요로 하기 때문에 모든 쿠버네티스 클러스터는 클러스터 DNS를 갖추어야만 한다.  
  
  - 클러스터 DNS는 구성환경 내 다른 DNS 서버와 더불어, 쿠버네티스 서비스를 위해 DNS 레코드를 제공해주는 DNS 서버다.  

  - 쿠버네티스에 의해 구동되는 컨테이너는 DNS 검색에서 이 DNS 서버를 자동으로 포함한다.  

- 컨테이너 리소스 모니터링  
  - 컨테이너 리소스 모니터링은 중앙 데이터베이스 내의 컨테이너들에 대한 포괄적인 시계열 매트릭스를 기록하고 그 데이터를 열람하기 위한 UI를 제공해 준다.

<br/>

#### 쿠버네티스 클러스터에 필요한 요소

<br/>


퍼시스턴트 스토리지
- 쿠버네티스는 애플리케이션을 실행하는 컨테이너를 관리할 뿐만 아니라 클러스터에 연결된 애플리케이션 데이터도 관리할 수 있습니다.   

  쿠버네티스를 사용하면 사용자가 기본 스토리지 인프라에 관한 상세 정보를 알지 못해도 스토리지 리소스를 요청할 수 있습니다.  
  퍼시스턴트 볼륨은 포드가 아닌 클러스터에 따라 다르므로 포드보다 수명이 오래 지속될 수 있습니다.

<br/>

컨테이너 레지스트리
- 쿠버네티스가 의존하는 컨테이너 이미지는 컨테이너 레지스트리에 저장됩니다.   
  이러한 레지스트리를 직접 구성하거나 제 3사가 구성할 수 있습니다.

<br/>

<img src="./assets/k8s_cluster2.png" style="width: 80%; height: auto;"/> 

<br/>

참고:
- 출처:  https://kubernetes.io/ko/docs/concepts/overview/components/


<br/>


## yaml 템플릿

<br/>

참고:
- 출처:  https://may9noy.tistory.com/427

<br/>

### 쿠버네티스의 YAML 설정 파일

<br/>

Deployment란?  

- 디플로이먼트는 간단하게 말해서 pod + 리플리카 셋 입니다.  
- 리플리카 셋은 파드의 수를 수시로 체크하고 상태를 유지해주는 오브젝트 입니다.  
- 디플로이 먼트를 생성할때는 기본적으로 create, apply를 사용합니다.

<br/>

Configuration 파일의 3가지 구성요소?

- metadata : 리소스의 라벨, 이름 등을 지정함
- specificaion : 각 컴포넌트에 대한 상세 설명 및 어떤 오브젝트 종류인지에 내용을 설명
- status : 쿠버네티스가 자동으로 생성, 자신의 원하는 상태가 되도록 현재 상태를 나타냄

<br/>

예를 들어 spec에 replicas가 2라고 명시되어 있지만 현재 상태는 status에는 1개로 명시 되어 있다면, 목표 상태로 맞추기 위해 replicas를 하나 더 생성하게 된다.  

<br/>

apiVersion은 스크립트를 실행하기 위한 쿠버네티스 API 버전을 명시하고 Kind는 리소스의 종류를 명시한다. (POD, Service, ReplicaSet, Deployment)  

<br/>


[ Labels & Selectors ]  

- Labels : Pod와 같은 오브젝트에 첨부된 키와 값의 쌍이다. 오브젝트의 특성을 식별하는데 있어 사용자에게 중요하지만, 시스템에 직접적인 의미는 없다.
- Selectors : Label을 기반으로 오브젝트를 어떻게 선택하는지에 대한 방법의 표현이다.

<br/>

예를 들어 하나의 오브젝트에 Key-Value로 Label을 지정하고, Selector로 env: dev인걸 선택하면, env가 dev인 것들만 나오게 된다는 뜻이다.   

<br/>

예제를 통해서 자세히 알아보자.

<br/>

nginx-deployment.yaml  
```bash
apiVersion: apps/v1
kind: Deployment
metadata: # 3가지 필수 구성요소중 하나
  name: nginx-deployment # 해당 이름으로 디플로이 먼트가 생성됨.
  labels:
    app: nginx
spec: # 3가지 필수 구성요서중 하나 specification, 나머지 하나는 status
  replicas: 3 # 3개의 리플리카 pod를 생성한다.
  selector: # 디플로이먼트가 관리할 파드를 찾는 방법을 정의, matchLabels이 nginx인 파드를 디플로이먼트가 관리한다.
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx # 파드는 .metadata.labels 필드를 사용 app: nginx 라는 레이블을 붙인다.
    spec:
      containers:
      - name: nginx # 컨테이너 1개를 생성, .spec.template.spec.containers[0].name 필드를 사용해서 nginx 라는 이름을 붙인다.
        image: nginx:1.14.2 # 파드가 도커 허브의 nginx 1.14.2 버전 이미지를 실행하는 nginx 컨테이너 1개를 실행하는 것을 나타낸다.
        ports:
        - containerPort: 80
```

<br/>

- metadata.name 필드에 따라 nginx-deployment 이름으로 디플로이먼트가 생성된다
.   
- .spec.replicas 필드에 따라 디플로이먼트는 3개의 레플리카 파드를 생성한다.

- .spec.selector 필드는 디플로이먼트가 관리할 파드를 찾는 방법을 정의한다.    
  이 사례에서는 파드 템플릿에 정의된 레이블(app: nginx)을 선택한다.   
  그러나 파드 템플릿 자체의 규칙이 만족되는 한, 보다 정교한 선택 규칙의 적용이 가능하다.

- template 필드에는 다음 하위 필드가 포함되어있다.
 - 파드는 .metadata.labels 필드를 사용해서 app: nginx 라는 레이블을 붙인다.  

 - 파드 템플릿의 사양 또는 .template.spec 필드는 파드가 도커 허브의 nginx 1.14.2 버전 이미지를 실행하는 nginx 컨테이너 1개를 실행하는 것을 나타낸다. 

 - 컨테이너 1개를 생성하고, .spec.template.spec.containers[0].name 필드를 사용해서 nginx 이름을 붙인다.  

 - 추가로 .metadata.labels의 app은 향후 어느 리소스가 이 app 값을 디플로이먼트를 참고하려할때 필요한 값을 지정해주기 위한 것이고, .spec.selector.matchLabels의 app은 .spec.template.metadata.labels의 app을 참조하는 것이다.  
  .spec.template.metadata.labels의 app을 지정해줌으로써 .selector.matchLabels에서 리소스를 찾을 수 있다.  

<br/>

YAML 문법이 맞는지 검사하는 사이트  

<br/>

https://onlineyamltools.com/validate-yaml


<br/>

## kubernetes API 서버 보안

<br/>

쿠버네티스에서 API 서버 보안은 왜 필요할까?  

쿠버네티스에서는 이론적으로 파드 외부 또는 내부에서 API 서버로 적절한 요청을 하면 어떤 리소스던 생성, 삭제, 수정, 조회가 가능하다.  

그런데 만약 개발자의 코딩 실수로 어떤 파드에서 아무 관련이 없는 다른 파드를 삭제해버릴 수 있다면 큰 문제가 될 것이다.  

그렇기 때문에 사용자 또는 파드에게 적절하게 권한을 부여하는 기능은 보안과 안정적인 운영을 위해 필수적이다.  

회사에서 여러개의 팀이 하나의 공용 클러스터를 함께 사용하는 멀티테넌트 환경에서는 이러한 권한의 분리가 더욱 중요하다.

쿠버네티스 클러스터는  API 서버가 사용자나 Pod 의 요청을 받을 때 명시적으로 설정된 권한만 허용하고, 그 이외의 모든 권한은 허용하지 않도록 동작한다.

<br/>

### Service Account 란?  

<br/>

서비스 어카운트(Service Account) 는 Kubernetes 의 파드에서 API 서버에 요청을 보냈을 때 이 "파드"를 식별하기 위한 리소스다. (사용자를 식별하는데 사용되지는  않는다)  

<br/>

파드에서 API 서버에 요청을 보내면 이 파드의 정체가 무엇인지 알아야 어떤 권한을 가지고 있는지도 알 수 있고,  

<br/>

이를 기반으로 파드의 요청이 권한에 맞는지를 확인하여 요청을 처리해줄지 말지를 결정할 것이다. 

실제로 권한을 정의하고, 설정하는 부분은 이후에 설명할 Role, ClusterRole, RoleBinding, ClusterRoleBinding 의 역할이다.

<br/>

### ServiceAccount 의 특징  

<br/>

모든 파드는 무조건 하나의 ServiceAccount 와 매핑이 되어야 실행될 수가 있다.  

그런데 ServiceAccount 를 만들지 않고, 파드의 매니페스트에 ServiceAccount 를 명시적으로 적어주지 않아도 파드가 잘 생성이 되고 실행 되는 것을 보고 의아할 수도 있다.  

사실 이것은, 쿠버네티스의 ServiceAccount Controller 가 모든 네임스페이스에 default 라는 이름의 서비스어카운트가 있도록 자동 생성해주며,  

ServiceAccount `Admission Controller` 가 파드의 매니페스트에 명시적으로 서비스어카운트를 정의하지 않으면 default 서비스어카운트를 매핑해주기 때문에 가능한것이다.  

뿐만 아니라, 서비스어카운트는 mountable secrets 에 지정한 시크릿만 파드에 마운트할 수 있도록 강제하는 기능과,  

Image pull secrets 기능을 통해 프라이빗 이미지 레지스트리에서 이미지를 가져올 수 있도록 하기 위한 시크릿을 이 서비스어카운트를 사용하는 파드에 자동으로 마운트시켜주는 기능도 가지고 있는데  

image pull secrets 기능도 ServiceAccount Admission Controller 가 수행한다.  

<br/>

### ServiceAccount 의 동작 방식  

<br/>

참고
- 출처 : https://malwareanalysis.tistory.com/257

<br/>

- pod와 service account 와의 관계 

<br/>

pod가 내부적으로 kube api server에 API를 사용할 경우가 있습니다. 쿠버네티스 이 문제를 해결하기 위해 pod 내부에 serviceaccount token을 마운트할 수 있습니다. pod는 마운트된 token을 이용해서 kube api server에 요청과 응답을 받습니다.

<br/>

<img src="./assets/service_account_pod.png" style="width: 80%; height: auto;"/> 

<br/>

- default serviceaccount

<br/>

namespace를 생성하면 자동으로 default serviceaccount가 생성됩니다.   namespace를 생성하고 약 1분뒤에 serviceaccount를 조회하면 default serviceaccount가 보입니다.  

<br/>

본인의 vm에 접속하여 아래 명령어를 통해 Openshift 에 접속합니다.    
순번에 따른 계정을 사용 해야 합니다. (예, edu1-admin, edu2-admin )

<br/>

```bash
oc login https://api.211-34-231-81.nip.io:6443 -u edu1-admin -p New1234! --insecure-skip-tls-verify
```  

<br/>

serviceaccount가 생성되면 serviceaccount 토큰이 secret으로 자동생성됩니다. default serviceaccount도 마찬가지로 토큰이 secret으로 저장됩니다.

<br/>

```bash
root@newedu:~# kubectl get sa
NAME       SECRETS   AGE
builder    2         260d
default    2         260d
deployer   2         260d
root@newedu:~# kubectl get secret
NAME                       TYPE                                  DATA   AGE
builder-dockercfg-wpgns    kubernetes.io/dockercfg               1      260d
builder-token-ltxbh        kubernetes.io/service-account-token   4      260d
builder-token-r2fkb        kubernetes.io/service-account-token   4      260d
default-dockercfg-vdnj9    kubernetes.io/dockercfg               1      260d
default-token-jnzxw        kubernetes.io/service-account-token   4      260d
default-token-sj9nw        kubernetes.io/service-account-token   4      260d
deployer-dockercfg-qhl7j   kubernetes.io/dockercfg               1      260d
deployer-token-9wfzn       kubernetes.io/service-account-token   4      260d
deployer-token-stftq       kubernetes.io/service-account-token   4      260d
```

<br/>

- pod에 serviceaccount 토큰 마운트

<br/>

pod를 생성할때 serviceaccount필드로 serviceaccount 토큰을 마운트 할 수 있습니다.   

설정하지 않으면 default serviceaccount 토큰이 마운트됩니다.   

아래 yaml은 본인의 namespace에 있는 edu serviceaccount 토큰을 마운트하는 예제입니다.    

<br/>


`Token Controller` 는 서비스어카운트가 생성될 때마다 자동으로 kubernetes.io/service-account-token 타입의 Secret 을 생성하여 매핑시켜준다.  

<br/>

```bash
root@jakelee:~# kubectl get sa
NAME      SECRETS   AGE
default   1         18d
root@jakelee:~# kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-p9fpr   kubernetes.io/service-account-token   3      18d
root@jakelee:~# kubectl describe secret default-token-p9fpr
Name:         default-token-p9fpr
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: ef9e4103-6663-4fcb-a12f-e2626d7e9666

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImhvbzBnMGhOX1VBMFp6MVN4ejFoX0RoVEtBX25lZUNkX291d3BnLUNWWVEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tcDlmcHIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImVmOWU0MTAzLTY2NjMtNGZjYi1hMTJmLWUyNjI2ZDdlOTY2NiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.wHfW2CGCePikcfSumoM9GvUIQS_8gI_wqKRnz9XzRFJ7rBl2iZNM7h47rPhHcA-uR61G6C6mzFHzb0zVYfo_PQAuddYF1PB3FZ9DmlylX3J1ne2KBQhh_f88FXmRCQZbiK5DS_GiR1Ys2UWzykLQFXtHDrQ7n_oy3Bq3poi4lDObg_TIH2rSyrhADebDqRXHOUZ5GGNV3Y7qwTvl9hFhbPT6Z8VO0mGgpHvv52XmNzHORUhwZRsMBLlMh19DNVTizvRlhemTZJMM71d8wJ9KG9rFhDwodWzuWgCYNxklWDvhDCsuJSj1QHei-9QSdqy2YTp5Qddcux7nMgyufQZT9Q
ca.crt:     570 bytes
namespace:  7 bytes
```  

<br/>

Token Controller 에 의해 생성된 Secret 에는 아래와 같은 3가지 데이터가 base64 로 인코딩되어 들어있다.    


- ca.crt - API 서버와 통신 시, SSL 인증을 위한 증명서
- token - 서비스어카운트 이름, Secret 이름 등의 정보에 서명한 JWT.   
           API 서버에 요청시 Bearer 토큰으로 사용됨
- namespace - 네임스페이스  

<br/>

파드 내 애플리케이션은 위의 데이터를 사용하여 API 서버와 통신한다.

API 서버는 요청의 Authorization 헤더에 있는 Bearer token 을 디코딩하여 어느 서비스어카운트를 사용해 보낸 요청인지 식별하게 된다.

<br/>
<br/>

- pod에서 kube api sevrer 요청

신규로 edu라는 service account를 생성합니다.     

kubectl create sa <name> 명령어로 서비스어카운트를 생성할 수 있다.  

<br/>

```bash
root@newedu:~# kubectl create sa edu
serviceaccount/edu created
root@newedu:~# kubectl get secret
NAME                                 TYPE                                  DATA   AGE
builder-dockercfg-tw9hd              kubernetes.io/dockercfg               1      239d
builder-token-hv4gb                  kubernetes.io/service-account-token   4      239d
builder-token-qnf96                  kubernetes.io/service-account-token   4      239d
default-dockercfg-g4s6s              kubernetes.io/dockercfg               1      239d
default-token-l7zwz                  kubernetes.io/service-account-token   4      239d
default-token-xhp65                  kubernetes.io/service-account-token   4      239d
deployer-dockercfg-55rzz             kubernetes.io/dockercfg               1      239d
deployer-token-vwwpg                 kubernetes.io/service-account-token   4      239d
deployer-token-xfxkn                 kubernetes.io/service-account-token   4      239d
edu-dockercfg-55hbn                  kubernetes.io/dockercfg               1      58s
edu-harbor                           kubernetes.io/dockerconfigjson        1      21h
edu-token-v7mnb                      kubernetes.io/service-account-token   4      58s
edu-token-xm9hf                      kubernetes.io/service-account-token   4      58s
my-service-account-dockercfg-4j5n7   kubernetes.io/dockercfg               1      161d
my-service-account-token-d67wk       kubernetes.io/service-account-token   4      161d
my-service-account-token-wxttf       kubernetes.io/service-account-token   4      161d
super-secret                         Opaque                                1      162d
```
<br/>

netshoot 라는 이름으로 yaml  화일을 하나 생성한다.  

`serviceAccountName: edu` 를 명시한다.  

<br/>

```bash
root@newedu:~# vi netshoot.yaml
apiVersion: v1
kind: Pod
metadata:
  name: netshoot
spec:
  serviceAccountName: edu
  containers:
  - name: netshoot
    image: nicolaka/netshoot
    command: ['sh', '-c', 'sleep 6000']
    imagePullPolicy: IfNotPresent
<br/>
```

<br/>

아래 명령어를 실행하여 pod를 생성하고 생성 된 것을 확인한다.   

<br/>


```bash
root@newedu:~# kubectl apply -f netshoot.yaml
root@newedu:~# kubectl get po
NAME                                        READY   STATUS        RESTARTS   AGE
edu13-backend-deployment-6ffb676ccc-hjblk   1/1     Running       0          76d
fluentd-elasticsearch-52dg8                 0/1     Terminating   0          76d
netshoot                                    1/1     Running       0          22m
```  

<br/>

컨테이너 안으로 들어가서 df -h 명령을 입력한다.

<br/>

```bash
root@newedu:~# kubectl exec -it netshoot sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
~ # df -h
Filesystem                Size      Used Available Use% Mounted on
overlay                  99.5G     31.9G     67.6G  32% /
tmpfs                    64.0M         0     64.0M   0% /dev
tmpfs                     7.8G         0      7.8G   0% /sys/fs/cgroup
shm                      64.0M         0     64.0M   0% /dev/shm
tmpfs                     3.1G     45.3M      3.1G   1% /etc/resolv.conf
tmpfs                     3.1G     45.3M      3.1G   1% /etc/hostname
/dev/vda4                99.5G     31.9G     67.6G  32% /etc/hosts
/dev/vda4                99.5G     31.9G     67.6G  32% /dev/termination-log
tmpfs                     3.1G     45.3M      3.1G   1% /run/secrets
tmpfs                     7.8G     28.0K      7.8G   0% /run/secrets/kubernetes.io/serviceaccount
tmpfs                     7.8G         0      7.8G   0% /proc/acpi
tmpfs                    64.0M         0     64.0M   0% /proc/kcore
tmpfs                    64.0M         0     64.0M   0% /proc/keys
tmpfs                    64.0M         0     64.0M   0% /proc/latency_stats
tmpfs                    64.0M         0     64.0M   0% /proc/timer_list
tmpfs                    64.0M         0     64.0M   0% /proc/sched_debug
tmpfs                     7.8G         0      7.8G   0% /proc/scsi
tmpfs                     7.8G         0      7.8G   0% /sys/firmware
```

<br/>

`/run/secrets/kubernetes.io/serviceaccount` 가 mount된 것을 확인 할 수 있다.  

<br/>

해당 폴더로 이동하여 token 값을  가져오고 K8S api server를 curl로 호출 한다.  

edu는 기본권한이 있어서 API List를 가져 온다.  

<br/>

```bash
~ # ls /run/secrets/kubernetes.io/serviceaccount
ca.crt          namespace       service-ca.crt  token
~ # cd /run/secrets/kubernetes.io/serviceaccount/
/run/secrets/kubernetes.io/serviceaccount # token=$(cat token)
/run/secrets/kubernetes.io/serviceaccount # curl -k -H "Authorization: Bearer ${token}" https://kubernetes.default.svc
{
  "paths": [
    "/.well-known/openid-configuration",
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/admissionregistration.k8s.io/v1beta1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apiregistration.k8s.io/v1beta1",
    "/apis/apps",
    "/apis/apps.openshift.io",
    "/apis/apps.openshift.io/v1",
    "/apis/apps/v1",
    "/apis/argoproj.io",
 ...
    "/readyz/poststarthook/start-apiextensions-controllers",
    "/readyz/poststarthook/start-apiextensions-informers",
    "/readyz/poststarthook/start-cluster-authentication-info-controller",
    "/readyz/poststarthook/start-kube-aggregator-informers",
    "/readyz/poststarthook/start-kube-apiserver-admission-initializer",
    "/readyz/shutdown",
    "/version"
  ]
}
```

<br/>

권한 설정하는 것은 아래 role과 rolebinding 에서 이어진다.

<br/>  

### RBAC 란?

<br/>  

어떤 사람, 혹은 파드가 API 서버에 요청을 하면 API 서버는 인증(Authentication)과 인가(Authorization)를 수행한다.  

인증은 접근 가능 여부를 확인하는 것이고, 인가는 접근 가능한 요청에 대해 요청된 자원에 접근할 수 있는지를 확인하는 것이다.  

RBAC(Role-Based Access Control) 는 API 서버가 인가를 수행하는 여러 방법 중 하나다.  

쿠버네티스는 다음과 같은 인가 방식을 제공한다.  

- Node
    - Kubelet 에 의한 요청에 대한 인가를 위한 방식
- ABAC(Attribute-Based Access Control)
    - 리소스의 속성에 따라 인가를 하는 방식
- RBAC
    - Role 을 기반으로 인가를 하는 방식
- Webhook
    - 외부 API 를 통해 인가를 하는 방식  

이 중에서 RBAC가 표준이며, kubernetes 1.8.0 부터는 대부분의 클러스터에서 기본적으로 사용하는 방식이다.  

RBAC 는 `특정 주체(subject) 가 특정 대상(url, resource 타입, 혹은 특정 resource) 에 대해 특정 행위(verb)` 를 할 수 있는지를 지정하는 방식이다.  

<br/>  

### Role 과 RoleBinding  


<br/>  

어떤 사람, 혹은 파드가 API 서버에 요청을 하면 API 서버는 인증

Role 은 대상(resource 등) 과 행위(verb) 를 지정하며, RoleBinding 은 주체(subject) 를 지정한다.  

여기서 주체는 3가지(User, Group, ServiceAccount) 중 한가지가 된다.  

대상은 보통 resource 의 타입을 정하는데, resourceName 으로 특정 리소스를 지정할 수도 있다.   

이후에 설명할 ClusterRole 에서는 리소스가 아닌 URL 을 지정할 수도 있다. /healthz 와 같이 특정 리소스에 대한 요청이 아닌 경우도 있기 때문이다.  

행위는 아래 표와같이 API 요청에 사용된 HTTP 메서드에 따라 특정 행위에 매핑되는데,  

<br/>

<img src="./assets/role_rolebinding.png" style="width: 80%; height: auto;"/>   

<br/>

행위에 대한 대상이 리소스라면 행위를 verb 로 적어주고, URL 이라면 HTTP 메소드로 적어준다.  

이름 그대로 Role 은 역할이고, RoleBinding 은 이러한 역할을, 역할을 수행하는 주체에 연결시켜 주는 것이다.  

<br/>

### Role 과 RoleBinding 의 특징  

<br/>

Role 과 RoleBinding 은 특정 네임스페이스에 종속된 개념이다. 그렇기 때문에 RoleBinding 은 다른 네임스페이스의 Role 을 바인딩해줄 수는 없다.  

하지만, RoleBinding 이 같은 네임스페이스의 Role 을 다른 네임스페이스의 subject 에게 바인딩해 줄 수는 있다.  

그래서 RoleBinding 에 subject 들을 명시할 때는 name 과 namespace, 그리고 kind(user/group/serviceaccount) 를 함께 명시한다.  

하나의 Role 은 여러개의 RoleBinding 에 의해 바인드될 수 있고, 하나의 RoleBinding 은 하나의 Role 만 참조할 수 있다.  

즉, `Role 과 RoleBinding 은 일대다(one-to-many)` 관계다.  

반면 하나의 RoleBinding 은 하나의 Role 을 여러 주체에 연결시켜 줄 수 있고, 하나의 주체는 여러개의 RoleBinding 에 의해 권한이 부여될 수 있다.  

즉, `RoleBinding 과 Subject(ServiceAccount 등) 는 다대다(many-to-many)` 관계다.  


<br/>  

### ServiceAccount 와 Role & RoleBinding 실습 

<br/>

### Role 과 RoleBinding 만들기

<br/>

Role 과 RoleBinidng 을 만드는 방법은 크게 두가지가 있다.  

하나는 매니페스트를 통해 만드는 방법이고, 나머지 하나는 kubectl create 명령어를 통해 만드는 방법이다.  

다음과 같이 YAML 파일을 작성하지 않고도 직접 verb, resource, role, serviceaccount 등을 인자로 주어 Role 과 RoleBinding 을 생성할 수 있다.  

<br/>

참고
- 출처 : https://velog.io/@ehdrms2034/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-RBAC-%EA%B8%B0%EB%B0%98-ServiceAccount

<br/>

먼저 아래 소스를 사용하여 role을 생성합니다.    
이 role은 pod 에만 허용된 권한이다. 서비스를 조회하거나 할수는 없다.  

<br/>


```bash
root@newedu:~# vi pod-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-role
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - create
  - delete
  - watch
  - list
  - get
```
<br/>

role을 생성하고 확인한다.  

<br/>

```bash
root@newedu:~# kubectl apply -f  pod-role.yaml
root@newedu:~# kubectl get role
NAME        CREATED AT
developer   2022-09-27T14:16:45Z
pod-role    2022-09-27T14:04:40Z
```
<br/>

role을 service account 와 매핑하기 위해서 rolebinding을 생성한다.  

<br/>

```bash
root@newedu:~# vi pod-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-rolebinding
subjects:
- kind: ServiceAccount
  name: edu
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-role
```

<br/>

rolebinding 을 생성하고 확인한다.  

<br/>

```bash
root@newedu:~# kubectl apply -f pod-rolebinding.yaml
root@newedu:~# kubectl get rolebindings
NAME                              ROLE                                          AGE
admin                             ClusterRole/admin                             239d
developer-binding-myuser          Role/developer                                161d
edu30-admin                       ClusterRole/admin                             239d
pod-rolebinding                   Role/pod-role                                 161d
system:deployers                  ClusterRole/system:deployer                   239d
system:image-builders             ClusterRole/system:image-builder              239d
system:image-pullers              ClusterRole/system:image-puller               239d
system:openshift:scc:piivileged   ClusterRole/system:openshift:scc:piivileged   174d
```  
<br/>

서비스를 아래와 같이 호출 해본다.  

<br/>

```bash
kubectl get svc --as system:serviceaccount:<namespace>:<serviceaccount>
```

<br/>

권한이 없기 때문에 에러가 발생한다.  

<br/>

```bash
root@newedu:~# kubectl get svc --as system:serviceaccount:edu30:edu
Error from server (Forbidden): services is forbidden: User "system:serviceaccount:edu30:edu" cannot list resource "services" in API group "" in the namespace "edu30"
```
<br/>


pod를 아래와 같이 호출 해본다.  

<br/>

```bash
kubectl get pods --as system:serviceaccount:<namespace>:<serviceaccount>
```

<br/>

권한이 있기 때문에  pod list 가 조회된다.    

<br/>

```bash
rroot@newedu:~# kubectl get po --as system:serviceaccount:edu30:edu
NAME                                        READY   STATUS        RESTARTS   AGE
edu13-backend-deployment-6ffb676ccc-hjblk   1/1     Running       0          76d
fluentd-elasticsearch-52dg8                 0/1     Terminating   0          76d
netshoot                                    1/1     Running       0          43m
```   


<br/>

## kuKubernetes 의 Downward API
  
<br/>

### Downward API 란?

<br/>

컨테이너가 쿠버네티스에 지나치게 종속되지 않으면서도 자기 자신에 대한 정보를 알고 있으면 유용할 때가 있다. 다운워드 API는 컨테이너가 자기 자신 혹은 클러스터에 대한 정보를, 쿠버네티스 클라이언트나 API 서버 없이도 사용할 수 있게 한다.

<br/>

예를 들어, 잘 알려진 특정 환경 변수에다가 고유한 식별자를 넣어 사용하는 애플리케이션이 있다고 하자. 해당 애플리케이션에 맞게 작업할 수도 있겠지만, 이는 지루하고 오류가 나기 쉬울뿐더러, 낮은 결합이라는 원칙에도 위배된다.   
대신, 파드의 이름을 식별자로 사용하고 잘 알려진 환경 변수에 파드의 이름을 넣는 것도 괜찮은 방법이다.

<br/>

쿠버네티스에는 실행 중인 컨테이너에 파드 및 컨테이너 필드를 노출하는 두 가지 방법이 있다.

- 환경 변수
- 볼륨 파일

<br/>

파드 및 컨테이너 필드를 노출하는 이 두 가지 방법을 다운워드 API라고 한다.



애플리케이션이 실행되기 전에 이미 알고있는 속성이나 설정 값들은 ConfigMap 이나 Secret 으로 파드에 전달할 수 있지만,  

<br/>

파드의 이름, 파드의 IP, 파드가 실행되는 노드의 이름 등 실제로 파드가 생성 및 실행이 되기전에는 알 수 없는 속성들도 존재한다.  

<br/>

물론 파드의 레이블이나 어노테이션과 같은 일부 속성들은 파드 생성 이전에도 알 수 있지만,  

<br/>

파드 내에서 정보를 사용하고 싶다는 이유로 이미 설정되어 있는 속성을 ConfigMap 등을 통해 중복하여 정의하고 싶지는 않을 것이다.  

<br/>

이런 속성들을 컨테이너에서 실행 중인 애플리케이션에서 알아내려면 어떻게 해야할까?  

이 때 사용되는 것이 Downward API 이다.  

Downward API 는 단순히 환경변수, 또는 파일(downwardAPI 볼륨을 통해) 로 위와 같은 속성들을 컨테이너에서 손쉽게 사용할 수 있도록 하는 기능일 뿐이다.  

Downward API 를 통해 전달할 수 있는 정보는 다음과 같다.  

- 파드의 이름
- 파드의 IP 주소
- 파드가 속한 네임스페이스
- 파드가 실행중인 노드의 이름
- 파드가 실행 중인 서비스 어카운트 이름
- 각 컨테이너의 CPU와 메모리 request
- 각 컨테이너의 CPU와 메모리 limit
- 파드의 label
- 파드의 annotation

<br/>

k8s 에서는 파드가 API server 와 통신할 수 있도록 하기 위해  
각 파드마다 기본적으로 Default token 시크릿 볼륨을 만들어  

파드 내 컨테이너의 /var/run/secrets/kubernetes.io/serviceaccount/에 마운트해 주는데,  

이 곳에 namespace 라는 파일에 네임스페이스가 적혀있기 때문이다.  

```bash
root@jakelee:~# kubectl get po
NAME                                  READY   STATUS    RESTARTS   AGE
flask-edu4-app-74788b6479-nmcvv       1/1     Running   0          16d
flask-edu4-app-74788b6479-rlght       1/1     Running   0          16d
hpa-example-deploy-59bf97fcc6-6nxjs   1/1     Running   0          16d
inspekt-deployment-c8d9f5dcf-2slpx    1/1     Running   0          11d
mynginx-69d586ff67-bmh6m              1/1     Running   0          10d
web2-5d47994f45-dmrwr                 1/1     Running   0          9d
pod-test-app                          1/1     Running   0          6d20h
root@jakelee:~# kubectl exec -it pod-test-app sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# ls /var/run/secrets/kubernetes.io/serviceaccount/
ca.crt	namespace  token
# cat /var/run/secrets/kubernetes.io/serviceaccount/namespace
default
```  

<br/>

### 환경 변수로 전달하기 vs 볼륨으로 전달하기

<br/>

Downward API 를 통해 데이터를 전달하기 위한 방법으로는  

환경 변수를 통한 방법과 볼륨을 통한 방법, 이렇게 크게 두가지가 있다.  

대부분의 경우 환경변수를 통한 방법과 볼륨을 통한 방법 중 어떤 방법을 사용해도 크게 문제가 없지만  

약간의 차이점이 있다. 우선 일부 정보들은 둘 중 한가지 방법으로만 얻을 수 있다.  

<br/>

예를 들어 Pod 의 label 과 annotation 은 downwardAPI 볼륨을 통해서만 전달할 수 있다.   

그 이유는, Pod 의 label 과 annotation 은 Pod 가 실행되는 동안 수정될 수가 있는데,  

이 때 Pod 가 변경된 데이터를 볼 수 있도록 해야 한다.  

파드가 실행중인 노드의 이름과 IP 는 환경 변수를 통한 방법으로만 얻을 수 있다.  

<br/>

### 환경변수로 전달하기   

<br/>


vi에디터로 downward-env.yaml 화일을 만들어 보자.   

```bash
root@jakelee:~# vi downward-env.yaml
```

<br/>

아래 소스를 사용한다.  

<br/>

```bash
apiVersion: v1
kind: Pod
metadata:
  name: downward-env
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "99999"]
    resources:
      requests:
        cpu: 15m
        memory: 100Ki
      limits:
        cpu: 100m
        memory: 20Mi
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: SERVICE_ACCOUNT
      valueFrom:
        fieldRef:
          fieldPath: spec.serviceAccountName
    - name: CONTAINER_CPU_REQUEST_MILLICORES
      valueFrom:
        resourceFieldRef:
          resource: requests.cpu
          divisor: 1m
    - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES
      valueFrom:
        resourceFieldRef:
          resource: limits.memory
          divisor: 1Ki
```  

<br/>

pod를 생성한다.  

<br/>

```bash
root@jakelee:~# kubectl apply -f downward-env.yaml
pod/downward-env created
```

<br/>

```bash
root@jakelee:~# kubectl get po
NAME                                  READY   STATUS    RESTARTS   AGE
pod-test-app                          1/1     Running   0          6d20h
downward-env                          1/1     Running   0          12s
```  

<br/>

pod가 생성이 되면 환경 변수를 조회해 본다.  

<br/>

기본 환경 변수에 POD_NAME , POD_NAMESPACE , POD_IP , POD_NAME이 추가 된 것을 확인 할 수 있다.  

<br/>

```bash
root@jakelee:~# kubectl exec downward-env env
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=downward-env
SERVICE_ACCOUNT=default
CONTAINER_CPU_REQUEST_MILLICORES=15
CONTAINER_MEMORY_LIMIT_KIBIBYTES=20480
POD_NAME=downward-env
POD_NAMESPACE=default
POD_IP=10.42.0.130
NODE_NAME=jakelee
```  

<br/>

<img src="./assets/downward_api_pod.png" style="width: 80%; height: auto;"/> 

<br/>

### 볼륨으로 전달하기  

<br/>

vi에디터로 downward-volume.yaml 화일을 만들어 보자.   

<br/>

```bash
root@jakelee:~# vi downward-volume.yaml
```

<br/>

아래 소스를 사용한다.  

<br/>

```bash
apiVersion: v1
kind: Pod
metadata:
  name: downward-volume
  labels:
    foo: bar
  annotations:
    key1: value1
    key2: |
      multi
      line
      value
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "9999999"]
    resources:
      requests:
        cpu: 15m
        memory: 100Ki
      limits:
        cpu: 100m
        memory: 40Mi
    volumeMounts:
    - name: downward
      mountPath: /etc/downward
  volumes:
  - name: downward
    downwardAPI:
      items:
      - path: "podName"
        fieldRef:
          fieldPath: metadata.name
      - path: "podNamespace"
        fieldRef:
          fieldPath: metadata.namespace
      - path: "labels"
        fieldRef:
          fieldPath: metadata.labels
      - path: "annotations"
        fieldRef:
          fieldPath: metadata.annotations
      - path: "containerCpuRequestMilliCores"
        resourceFieldRef:
          containerName: main
          resource: requests.cpu
          divisor: 1m
      - path: "containerMemoryLimitBytes"
        resourceFieldRef:
          containerName: main
          resource: limits.memory
          divisor: 1
```  

<br/>

pod가 생성이 되면 volume을 조회해 본다.  

<br/>

```bash
root@jakelee:~# kubectl get po
NAME                                  READY   STATUS    RESTARTS   AGE
pod-test-app                          1/1     Running   0          6d20h
downward-env                          1/1     Running   0          14m
downward-volume                       1/1     Running   0          68s
root@jakelee:~# kubectl exec downward-volume  ls /etc/downward
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
annotations
containerCpuRequestMilliCores
containerMemoryLimitBytes
labels
podName
podNamespace
root@jakelee:~# kubectl exec downward-volume  cat /etc/downward/podName
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
downward-volume
```  

<br/>

