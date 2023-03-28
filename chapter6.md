
# Chapter 6 
 
3 tier 아키텍처 구조에서  frontend (React ) 와 backend ( SpringbBoot ) 의 Build 방법 및 배포에 대한 설명을 합니다. 

<br/>


1. Podman 소개

2. Jib ( Skaffold ) 소개

3. React/SpringBoot CI/CD 설명 및 실습
    - 언어별 (React/SpringBoot) Build Tool 생성  
    - Gitops Repository 구성 
    - 언어별 (React/SpringBoot) CI Pipeline 구성 및 빌드 (/W CORS) 
    - ArgoCD 배포

4. React/SpringBoot/MariaDB 3-tier 구조 한번에 배포 하기
    - ArgoCD Apps-of-Apps 패턴 

<br/>

##  Podman 소개

<br/>

Podman(POD 관리자)은 Linux® 시스템에서 컨테이너를 개발, 관리, 실행하기 위한 오픈소스 툴입니다. Red Hat® 엔지니어가 오픈소스 커뮤니티와 함께 처음 개발한 Podman은 libpod 라이브러리를 사용하여 컨테이너 에코시스템 전체를 관리합니다.   

Podman은 데몬이 없는 포괄적인 아키텍처로 컨테이너를 더 안전하고 손쉽게 관리할 수 있게 해주며, Buildah 및 Skopeo와 같은 관련 툴과 기능을 통해 개발자는 컨테이너 환경을 사용자 정의 방식으로 요구 사항에 가장 적합하게 설정할 수 있습니다. 

<br/>

> Podman과 다른 컨테이너 엔진의 차이점은 무엇인가요?

<br/>

Podman은 데몬이 없다는 점에서 다른 컨테이너 엔진과 다릅니다. 데몬은 사용자 인터페이스 없이 컨테이너를 실행하는 것과 같이 어려운 작업을 하기 위해 시스템 백그라운드에서 실행되는 프로세스입니다. 사용자와 컨테이너 사이에서 통신을 중계한다고 생각하면 됩니다.   

데몬은 컨테이너 환경을 관리하는 하나의 편리한 방법이기도 하지만 동시에 보안 취약점을 가져올 수도 있습니다. 다수의 데몬이 루트 권한을 통해 실행됩니다. Linux 시스템에서 루트 계정은 관리자 액세스를 부여받은(동시에 관리자 검증 절차를 건너뛰는) 슈퍼 사용자(superuser) 역할을 하면서 파일을 읽고, 프로그램을 설치하고, 애플리케이션을 수정합니다. 따라서 데몬은 컨테이너를 장악하고 호스트 시스템에 잠입하려는 해커들에게 매우 좋은 표적이 될 수 있습니다.   

Podman은 데몬을 제거하고 일반적인 사용자가 루트를 보유한 데몬과 상호 작용할 필요 없이 컨테이너를 실행하거나 루트리스(rootless) 컨테이너를 사용할 수 있도록 합니다.   

루트리스 컨테이너로 사용자는 프로세스에 관리 권한이 없어도 컨테이너를 생성, 실행, 관리할 수 있어, 더 손쉽게 액세스할 수 있는 컨테이너 환경을 만드는 동시에 보안 리스크를 줄일 수 있습니다. 또한 Podman은 security-enhanced linux(SELinux) 레이블이 있는 각 컨테이너를 실행하여 관리자가 컨테이너 프로세스에 제공되는 리소스와 기능을 제어할 수 있도록 권한을 부여합니다.  

<br/>

> Podman과 Docker 비교  

<br/>

Docker는 Linux 컨테이너를 만들고 사용할 수 있도록 지원하는 컨테이너화 기술입니다. Podman과 Docker의 주요한 차이점은 바로 Podman이 데몬이 없는 아키텍처를 보유하고 있다는 점입니다. Podman 컨테이너는 항상 루트리스였으나, Docker는 최근 들어서야 데몬 구성에 루트리스 모드를 추가했습니다. Docker는 컨테이너 생성 및 관리를 위한 올인원 툴인 반면, Podman과 Buildah 및 Skopeo 등의 관련 툴은 컨테이너화의 특정 측면에 더욱 전문화되어 사용자가 클라우드 네이티브 애플리케이션에서 필요한 사항에 따라 사용자 정의할 수 있습니다.   

Podman은 Docker의 대안이 되는 강력한 컨테이너 엔진이지만, 이 두 엔진은 함께 사용할 수 있습니다. 사용자는 Podman에 대해 Docker의 별칭을 설정(alias docker=podman)하거나 그 반대로 설정하여 간편하게 Podman과 Docker를 전환하여 사용할 수 있습니다. podman-docker로 불리는 rpm이 'docker' 커맨드가 필요한 환경에서 Podman을 호출하는 시스템 애플리케이션 PATH에 'docker'를 실행하여 Docker로의 전환을 손쉽게 만들어줍니다. Podman의 CLI는 Docker 컨테이너 엔진과 유사하기 때문에 Podman에 익숙한 사용자가 Docker도 익숙하게 느끼는 경우가 많습니다.   

일부 개발자는 개발 단계에서 Docker를 사용하고 런타임 환경에서 프로그램을 Podman으로 이전해 Podman과 Docker를 모두 활용함으로써 더욱 높은 수준의 보안을 구현하기도 합니


##  Jib ( skaffold ) 소개

<br/>

### 왜 Jib는 등장하게 되었는가?

<br/>

참고
- https://nesoy.github.io/articles/2020-11/Jib 

<br/>


<img src="./assets/jib1.png" style="width: 80%; height: auto;"/>  

<br/>

- 우리가 그동안 도커 이미지를 만들기 위해 무슨 일들을 했을까?  
  - 구글에 Java Docker Image 생성하는 법을 찾아본다.
    - 열심히 DockerFile을 작성한다.
    - 우분투를 받고 -> 자바를 다운받고 -> 어플리케이션을 빌드하고 -> 실행한다.
    - 조금 더 빠르게 작성해볼까?
      - 설치하지 않고 openjdk이미지를 가져와서 빌드된 jar를 복사하게 작성한다.
      - 복잡한 과정이 단순하게 만들었다.
    - 하지만 openjdk 이미지 용량이 큰게 문제가 된다.
      - alpine이라는 이미지를 활용해보자. -> 용량이 줄었다.
      - DockerFile을 최적화하기 위해 문서를 읽는다. - 문서
    - .dockerignore를 열심히 작성한다.
      - !target/something-*.jar
    - 조금 더 개선하여 빌드된 dependencies들을 따로 분리해서 저장한다.
      - 분리된 의존성을 하나의 Layer로 관리한다.
      - 최대한 Cache를 활용하여 Dockerfile을 작성한다.
    - Maven Plugin을 활용한다.
- 이 모든 과정이 복잡하고.. 학습난이도도 높고.. 난 그저 단순히 Containerizing하고 싶다.  

> 그래서 이 복잡한 과정을 단순하게 만들기 위해 만든게 Jib이다.

<br/>

### Jib 이란

<br/>

- Containerize your Java application.
- 플러그인 적용하고. jib를 실행하면 된다.
- 마치 Container의 compiler와 같은 역할이라고 볼 수 있다.

<br/>


### Goals  

<br/>

- Fast
  - 어플리케이션을 Multiple Layer로 분리하여 이미지를 빌드한다.
  - 변경된 사항만 이미지 빌드에 반영되기 때문에 아주 빠르다.
- Reproducible
  - 동일한 컨텐츠로 이미지를 빌드하면 항상 같은 이미지를 만들게 된다.
  - 불필요한 업데이트는 발생하지 않는다.
- Daemonless
  - Docker Image를 Maven, Gradle와 같은 빌드 툴을 통해 만들 수 있다.
  - 더이상 Dockerfile이나 Docker build/push와 같은 액션이 필요없게 된다.

<br/>

### Jib Base Image는?

<br/>

- 실행환경에 필요한 부분만 들고 있다.  
  - 표준 Linux distribution에 포함된 shell과 같은 프로그램은 없다.
- https://github.com/GoogleContainerTools/distroless


<br/><br/>


## React/SpringBoot CI/CD 설명 및 실습

<br/>

우리는 Frontend 로 react.js를 Backend로는 SpringBoot ( JAVA )를 사용하며 DB는 기본에 NFS 사용하여 구축한 MariaDB를 사용합니다. 

<br/>

Frontend 와 Backend 는 아래와 같이 JWT 를 사용하여 인증하는 예제입니다.  

<br/>

<img src="./assets/3tier1.png" style="width: 80%; height: auto;"/>  

<br/>


### 언어별 ( React ) Build Tool 생성

<br/>

예제 Frontend 샘플은 ReactJS 이고 podman을 사용하여 빌드 및 push를 해야합니다.        

프로젝트에서는 각 언어에 맞는 Build tool로 도커 이미지를 생성하는 것이 중요합니다.   

<br/>

먼저 https://github.com/shclub/dockerfile repository를  fork 합니다.

<br/>

아래는  podman에 kustomize를 사용한 build tool 을 만들기 위한 Docker file 입니다. ( https://github.com/shclub/dockerfile/blob/master/podman_kustomize/Dockerfile )

<br>

- Download Stage
  - podman 베이스  kustomize  를 다운 받습니다.  
- Production Stage
  - 환경을 설정합니다.  

<br>

```bash
############ [Download Stage] #####################
FROM mattermost/podman:1.8.0 AS downloader

WORKDIR /downloads

RUN curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
############ [Production Stage] #####################
### Runtime
FROM downloader AS production

ENV TZ=Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
COPY --from=downloader /downloads/kustomize /usr/local/bin/kustomize
```  

<br/>

Github Action 에서 Docker podman kustomize를 클릭하여 도커이미지를 생성합니다.   

<br/>

<img src="./assets/build_tool1.png" style="width: 80%; height: auto;"/>  


<br/>
Github Action에서 workflow 만들때는 docker file 위치를 잘 지정해 줍니다.  

<br/>

```bash
name: Docker Podman Kustomize

# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

on:      
  workflow_dispatch:
    inputs:
      name:
        description: "Docker TAG"
        required: true
        default: "master"

env:
  # Use docker.io for Docker Hub if empty
  REGISTRY: ghcr.io
  # github.repository as <account>/<repo>
  #IMAGE_NAME: ${{ github.repository }}
  IMAGE_NAME: "shclub/podman_kustomize"
  #IMAGE_REGISTRY: ghcr.io/${{ github.repository_owner }}


jobs:
  build:

    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      # This is used to complete the identity challenge
      # with sigstore/fulcio when running outside of PRs.
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      # Install the cosign tool except on PR
      # https://github.com/sigstore/cosign-installer
      - name: Install cosign
        if: github.event_name != 'pull_request'
        uses: sigstore/cosign-installer@f3c664df7af409cb4873aa5068053ba9d61a57b6 #v2.6.0
        with:
          cosign-release: 'v1.13.1'


      # Workaround: https://github.com/docker/build-push-action/issues/461
      - name: Setup Docker buildx
        uses: docker/setup-buildx-action@79abd3f86f79a9d68a23c75a09a9a85889262adf

      # Login against a Docker registry except on PR
      # https://github.com/docker/login-action
      - name: Log into registry ${{ env.REGISTRY }}
        if: github.event_name != 'pull_request'
        uses: docker/login-action@28218f9b04b4f3f62068d7b6ce6ca5b26e35336c
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Extract metadata (tags, labels) for Docker
      # https://github.com/docker/metadata-action
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@98669ae865ea3cffbcbaa878cf57c20bbf1c6c38
        with:
          #images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
         images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
         tags: ${{ github.event.inputs.name }}


      # Build and push Docker image with Buildx (don't push on PR)
      # https://github.com/docker/build-push-action
      - name: Build and push Docker image
        id: build-and-push
        uses: docker/build-push-action@ac9327eae2b366085ac7f6a2d02df8aa8ead720a
        with:
          # context: .
          context: ./podman_kustomize # Dockerfile 위치
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max


      # Sign the resulting Docker image digest except on PRs.
      # This will only write to the public Rekor transparency log when the Docker
      # repository is public to avoid leaking data.  If you would like to publish
      # transparency data even for private images, pass --force to cosign below.
      # https://github.com/sigstore/cosign
      - name: Sign the published Docker image
        if: ${{ github.event_name != 'pull_request' }}
        env:
          COSIGN_EXPERIMENTAL: "true"
        # This step uses the identity token to provision an ephemeral certificate
        # against the sigstore community Fulcio instance.
        run: echo "${{ steps.meta.outputs.tags }}" | xargs -I {} cosign sign {}@${{ steps.build-and-push.outputs.digest }}
```  

<br/>

### 언어별 ( SpringBoot ) Build Tool 생성

<br/>

우리가 사용한 소스는 https://github.com/shclub/edu12-4 repository 입니다.  

<br/>

예제 샘플은 JDK 17를 사용하며 우리는 kustomize와  skaffold ( jib ) 가 있는 도커 이미지가 필요합니다.    

프로젝트에서는 각 언어에 맞는 Build tool로 도커 이미지를 생성하는 것이 중요합니다.   

<br/>

아래는 예제에서 사용할 SpringBoot build tool 을 만들기 위한 Docker file 입니다. ( https://github.com/shclub/dockerfile/blob/master/build-tool/Dockerfile )

<br>

- Build Stage
  - JDK17 베이스 이미지에 docker , kustomize , skaffold 를 다운 받습니다.  
- Production Stage
  - 환경을 설정합니다.  

<br>

```bash
FROM eclipse-temurin:17.0.2_8-jdk AS base
############ [Build Stage ] #####################
FROM base AS build-base
# Installing basic packages
RUN apt-get update && \
   apt-get install -y zip unzip curl && \
   apt-get install -y docker && \
   rm -rf /var/lib/apt/lists/* && \
   rm -rf /tmp/*
# Downloader
FROM build-base AS downloader
WORKDIR /downloads
RUN curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
RUN curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && \
install skaffold /usr/local/bin/
RUN curl -fsSL https://download.docker.com/linux/static/stable/x86_64/docker-20.10.7.tgz | tar zxvf - --strip 1 -C /usr/local/bin docker/docker

############ [Production Stage] #####################
### Runtime
FROM base AS production
RUN apt-get update && \
   apt-get install -y git && \
   rm -rf /var/lib/apt/lists/* && \
   rm -rf /tmp/*
ENV TZ=Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
COPY --from=downloader /usr/local/bin/skaffold /usr/local/bin/skaffold
COPY --from=downloader /downloads/kustomize /usr/local/bin/kustomize
COPY --from=downloader /usr/local/bin/docker /usr/local/bin/docker
RUN mkdir -p /root/.m2
```  
<br/>

Github Action 에서 Docker Java 17 build-tool 를 클릭하여 도커이미지를 생성합니다.   

<br/>

SpringBoot 에서는 Build Tool 외에 SpringBoot가 컨테이너로 올라가기 위한 jre runtime 이미지가 필요합니다.    

해당 도커화일은 아래와 같습니다. ( https://github.com/shclub/dockerfile/tree/master/jre-runtime)

<br/>
 
`eclipse-temurin:17.0.2_8-jre-alpine` 이미지에 로그를 위한 한국시간을 설정합니다.  

<br/>

```bash
FROM eclipse-temurin:17.0.2_8-jre-alpine
ENV TZ=Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

<br/>

Github Action 에서 Docker Jre runtime 17 를 클릭하여 도커이미지를 생성합니다.   


<br/>

### GitOps Repository 구성

<br>

GitOps를 구현 하기 위해서는 소스와 OPS 를 위한 Repository 가 분리되어야 합니다.

예제에서는 아래와 같이 총 4개의 Repository 로 분리하였습니다.

아래의 4개 Repository 를 본인의 repository에 fork 합니다.

<br/>

예 ) https://github.com/shclub/edu12-3

<br/>



|구분|소스 Repository| Ops Repository|
|:--|:--| :-------|  
| FrontEnd | edu12-3	| edu12-frontend-gitops |
| BackEnd | edu12-4 |	edu12-backend-gitops |


<br/>

edu12-frontend-gitops 는 아래와 같이 구성 됩니다.  
본인의 계정으로 수정합니다.  

<br/>

<img src="./assets/3tier_gitops1.png" style="width: 80%; height: auto;"/>   

<br/>

kustomization.yaml
```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
- route.yaml
images:
- name: ghcr.io/shclub/edu12-frontend # 수정필요
  newTag: "20230322094306"
```  

<br/>

deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: ghcr.io/shclub/edu12-frontend # 수정필요
        env:
        - name: BACKEND_API_URL
          value: "http://backend" 
        ports:
        - containerPort: 80
```
<br/>

route 의 host 는 본인의 namespace에 맞춰 변경합니다. 

route.yaml
```bash
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: frontend
spec:
  host: frontend-edu30.apps.211-34-231-82.nip.io # 수정필요
  port:
    targetPort: 80
  to:
    kind: Service
    name: frontend
    weight: 100    
  wildcardPolicy: None  
```

<br/>

edu12-backend-gitops 는 아래와 같이 구성 됩니다.    
backend 이기 때문에 route는 필요 없습니다.

<br/>

<img src="./assets/3tier_gitops2.png" style="width: 80%; height: auto;"/>   

<br/>

kustomization.yaml
```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
images:
- name: ghcr.io/shclub/edu12-backend # 수정필요
  newTag: "20230322092639"
```  

<br/>

backend는 DB와 연결을 해야 하기 때문에 SPRING_DATASOURCE_URL 에 mariaDB URL을 기입해야 한다.    

본인의 namespace 에서 service 를 조회해 보고 db 관련 서비스를 찾는다.  

<br/>

```bash
root@newedu:~# kubectl get svc
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
my-release-mariadb   ClusterIP   172.30.8.131     <none>        3306/TCP             6d8h
```

<br/>

예제를 정상적으로 진행 되기 위해서는 mariaDB pod 에 접속하여 edu DB에
아래와 같이 sequence와 table을 생성하고 데이터 1건을 insert 한다.   

<br/>

```bash
root@newedu:~# kubectl exec -it helm-db-mariadb-0 sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
$ mysql -u edu -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 221690
Server version: 10.6.9-MariaDB Source distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use edu
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

MariaDB [edu]>
```

<br/>

sequence 및 table 생성.

<br/>

```bash

create sequence hibernate_sequence;     

create table EMPLOYEE (id int primary key,empName varchar(255),empDeptName varchar(255),empTelNo varchar(20),empMail  varchar(25));

insert into EMPLOYEE  values(1,'1','1','1','1');  

SELECT NEXTVAL(hibernate_sequence);
```

<br/>


생성된 것을 확인합니다.  

<br/>


```bash

MariaDB [edu]> show tables;
+--------------------+
| Tables_in_edu      |
+--------------------+
| EMPLOYEE           |
| hibernate_sequence |
+--------------------+
2 rows in set (0.001 sec)
```  
  

<br/>

아래 DB URL 을 본인의 DB 서비스에 맞게 수정한다. 

<br/>

deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: ghcr.io/shclub/edu12-backend # 수정필요
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prd"
        - name: SPRING_DATASOURCE_USERNAME
          value: "edu"  # 수정필요
        - name: SPRING_DATASOURCE_PASSWORD
          value: "edu1234"  #수정필요
        - name: SPRING_DATASOURCE_URL
          value: "jdbc:mariadb://my-release-mariadb:3306/edu" # 수정필요
        ports:
        - containerPort: 8080
```
<br/>

### 언어별 (React) CI Pipeline 구성 및 빌드 (/W CORS) 

<br/>

우리가 사용할 소스는 https://github.com/shclub/edu12-3 repository 입니다.    

해당 repository를 본인의 계정에 fork 합니다. 

<br/>

폴더 구조는 아래와 같습니다.  

<br/>

<img src="./assets/react1.png" style="width: 80%; height: auto;"/>  

  - Dockerfile : react 도커라이징 manifest 화일
  - Jenkinsfile_dockerhub : dockerhub에 push 하는 jenkins pipeline
  - Jenkinsfile_github : github에 push 하는 jenkins pipeline
  - Jenkinsfile_okd : private docker registry 인 harbor에 push 하는 jenkins pipeline
  - nginx.conf : cors 해결을 위한 nginx config 화일


<br/>

React 와 Spring Boot 연동을 하기 위해서는 서로 간의 port 를 매핑을 하여야 합니다.  호스트 이름과 port가 다르면 CORS 에러가 발생합니다.  
- React는 기본 3000 port, SpringBoot는 기본 8080 port 사용

<br/>

CORS 
- https://devlog-wjdrbs96.tistory.com/m/429
- https://velog.io/@prayme/CORS-%EC%A0%95%EB%B3%B5%EA%B8%B0
- https://narup.tistory.com/208

<br/>

nginx.conf 화일에 backend 호출을 위해 
`proxy_pass http://backend_host` 로 설정을 합니다.

<br/>

해당 값은 dockerfile 화일에서 replace 됩니다.

<img src="./assets/react3.png" style="width: 80%; height: auto;"/>  

<br/>

아래는 React Dockerfile 입니다.    
- Build stage
    - base 이미지는 node를 사용하여 소스를 빌드 합니다. 
- Package stage
    - 도커라이징 하는 단계입니다. nginx 이미지를 사용합니다.
    - nginx.conf 화일을 /etc/nginx/conf.d/default.conf 에 복사합니다.
    - 환경 변수로 BACKEND_API_URL에 backend로 설정합니다.  
      - 이 의미는 springboot 가 backend라는 이름으로 서비스가 생성이 되어야 합니다.  
    - react는 3000번 포트가 아닌 80 포트로 웹브라우저에서 호출하도록 expose 합니다.  

<br/>

```bash
#
# Build stage
#
FROM ghcr.io/shclub/node:14.19.3-alpine as build  # 수정 필요
WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH
COPY package.json ./
COPY package-lock.json ./
COPY nginx.conf ./
RUN npm ci --silent
RUN npm install react-scripts@3.4.1 -g --silent
COPY . ./
RUN npm run build

#
# Package stage
#
# production environment
FROM ghcr.io/shclub/nginx:stable-alpine

ENV TZ Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY --from=build /app/build /usr/share/nginx/html
COPY --from=build /app/nginx.conf /etc/nginx/conf.d/default.conf

ENV BACKEND_API_URL backend
RUN sed -i "s|backend_host|$BACKEND_API_URL|g" -i /etc/nginx/conf.d/default.conf
RUN cat /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```  

<br/>

Github Action에 사전에 만들어 놓은 다양한 docker registry에 빌드하는 샘플이 있습니다.  

우리는 Jenkins를 사용하여 빌드 예정이기 때문에 Github Action 으로 빌드하는 설명은 생락합니다.  

<img src="./assets/react2.png" style="width: 80%; height: auto;"/>  

<br/><br/>

이제 pipeline 을 구성 합니다. 우리는 github 에 push 하는 것을 기준으로 설명 합니다.  

<br/>

Jenkinsfile_github 화일에 본인의 환경에 맞게 설정합니다.  

<br/>

- podTemplate : slave pod 에 생성되는 container 를 명시합니다.  
    - serviceAccount : jenkins-admin
    - namespace : slave 가 기동하는 namespace
    - containers : 컨테이너가 여러개 올 수 있음
    - containerTemplate [ podman ] : 실제 빌드하는 컨테이너.  privileged: true
    - containerTemplate [ jnlp ] : master 와 통신용 컨테이너
    - volumes [ hostPathVolume ]: podman 과 cri-o 연결
    - volumes [ persistentVolumeClaim ]: slave 로그 기록을 위한 볼륨
- stage('Clone Git Project') : Git source clone
- stage('Podman Build & Image Push ') : podman 컨테이너 안에서 도커를 빌드하고 push 합니다.
- stage('GitOps update') : gitops 폴더에 kustomize를 사용하여 update 하고 git 에 도커 tag를 변경합니다.  

<br/>

Jenkinsfile_github
```bash
def label = "agent-${UUID.randomUUID().toString()}"
def gitBranch = 'master'
def docker_registry = "ghcr.io"  

// 본인 이미지 사용
def imageName = "ghcr.io/shclub/edu12-frontend"
def git_ops_name = "edu12-frontend-gitops"
// 본인 Namespace
def P_NAMESPACE = "edu30"
// 본인 slave pvc
def P_SLAVE_PVC = "jenkins-edu-slave-pvc"
// 본인 이메일
def P_EMAIL = "shclub@gmail.com"


def TAG = getTag(gitBranch)

podTemplate(label: label, serviceAccount: 'jenkins-admin', namespace: P_NAMESPACE,
    containers: [
        containerTemplate(name: 'podman', image: 'ghcr.io/shclub/podman_kustomize:v1', ttyEnabled: true, command: 'cat', privileged: true, alwaysPullImage: true)
        ,containerTemplate(name: 'jnlp', image: 'ghcr.io/shclub/jenkins/jnlp-slave:latest-jdk11', args: '${computer.jnlpmac} ${computer.name}')
    ],
    volumes: [
        hostPathVolume(hostPath: '/etc/containers' , mountPath: '/var/lib/containers' ),
        persistentVolumeClaim(mountPath: '/var/jenkins_home', claimName: P_SLAVE_PVC,readOnly: false)
        ]){    
      
    node(label) {
        
      stage('Clone Git Project') {
          script{
             checkout scm
          }
       } 
                     
      stage('Podman Build & Image Push ') {
              container('podman') {
                  withCredentials([usernamePassword(credentialsId: 'github_ci',usernameVariable: 'USERNAME',passwordVariable: 'PASSWORD')]) {
                     sh  """
                     podman login -u ${USERNAME} -p ${PASSWORD} ${docker_registry} --tls-verify=false
                     podman build -t ${imageName}:${TAG} --cgroup-manager=cgroupfs --tls-verify=false .
                     podman push ${imageName}:${TAG} --tls-verify=false
                     echo 'TAG ==========> ' ${TAG}
                     """
                  }
              }
        }

        stage('GitOps update') {
            container('podman') {
               withCredentials([usernamePassword(credentialsId: 'github_ci',usernameVariable: 'USERNAME',passwordVariable: 'PASSWORD')]) {
                    sh """  
                        cd ~
                        git clone https://${USERNAME}:${PASSWORD}@github.com/${USERNAME}/${git_ops_name}
                        cd ${git_ops_name}
                        git checkout HEAD
                        kustomize edit set image ${imageName}:${TAG}
                        git config --global user.email ${P_EMAIL}
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

<br/><br/>

이제 jenkins 에서 `edu_frontend` 라는 이름으로 pipeline 을 구성 하고 빌드를 해봅니다.  

아래 추가 설정을 확인 합니다.

- github_ci 에서 토큰을 생성 할 때 write package 권한이 있어야 합니다.    
  <img src="./assets/3tier_jenkins1.png" style="width: 60%; height: auto;"/>  
- 처음 push 할때 기본값이 private 임으로 public 으로 변경 해야 합니다.
    - 해당 패키지 선택 후 오른쪽에 package settings 선택    
      <img src="./assets/3tier_package1.png" style="width: 80%; height: auto;"/>  
    - 하단에 Danger Zone 이동하여 Change Visibility 클릭하고 가이드에 따라 public으로 변경 ( 패키지 이름 입력 ). 1회만 변경하면 됨    
      <img src="./assets/3tier_package2.png" style="width: 80%; height: auto;"/>   

<br/>

Github에서 package 가 생성되었는지 확인합니다.  
TAG가 빌드 시간 입니다.  

<img src="./assets/3tier_gitops3.png" style="width: 60%; height: auto;"/>

<br/>

### 언어별 (SpringBoot) CI Pipeline 구성 및 빌드

<br/>

우리가 사용할 소스는 https://github.com/shclub/edu12-4 repository 입니다.    

해당 repository를 본인의 계정에 fork 합니다. 

<br/>

폴더 구조는 아래와 같습니다.  

<br/>

<img src="./assets/3tier_springboot1.png" style="width: 80%; height: auto;"/>  

  - Dockerfile : react 도커라이징 manifest 화일
  - Jenkinsfile_dockerhub : dockerhub에 push 하는 jenkins pipeline
  - Jenkinsfile_github : github에 push 하는 jenkins pipeline

<br/>

SpringBoot 도커라이징은 2가지 방법이 있습니다.

<br/>

> 첫번째는 jib을 사용하지 않는 방법입니다.

<br/>

<img src="./assets/3tier_jib_before.png" style="width: 80%; height: auto;"/>  

<br/>

아래는 SpringBoot Dockerfile 입니다.    
- Build stage
    - base 이미지는 openjdk:17-alpine으로 maven을 사용하여 소스를 빌드 합니다. 
- Package stage
    - 도커라이징 하는 단계입니다. 
    - jre17-runtime 이미지에 빌드된 jar 화일을 설정합니다.

<br/>

```bash
#
# Build stage
#
# Maven Wrapper Build

FROM ghcr.io/shclub/openjdk:17-alpine AS MAVEN_BUILD  # 수정 필요

RUN mkdir -p build
WORKDIR /build

COPY pom.xml ./
COPY src ./src                             
COPY mvnw ./         
COPY . ./

RUN ./mvnw clean package -Dmaven.test.skip=true

#
# Package stage
#
# production environment

#FROM eclipse-temurin:17.0.2_8-jre-alpine
FROM ghcr.io/shclub/jre17-runtime:v1.0.0

COPY --from=MAVEN_BUILD /build/target/*.jar app.jar

ENV TZ Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV SPRING_PROFILES_ACTIVE dev

ENV JAVA_OPTS="-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -XshowSettings:vm"
ENV JAVA_OPTS="${JAVA_OPTS} -XX:+UseG1GC -XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark -XX:InitiatingHeapOccupancyPercent=35 -XX:G1ConcRefinementThreads=20"

EXPOSE 80

ENTRYPOINT ["sh", "-c", "java -jar  app.jar "]
```  

<br/>

Github Action에 사전에 만들어 놓은 다양한 docker registry에 빌드하는 샘플이 있습니다.  

우리는 Jenkins를 사용하여 빌드 예정이기 때문에 Github Action 으로 빌드하는 설명은 생락합니다.  

<br/><br/>


> 두번째는 jib을 사용하는 방법입니다.

<br/>

<img src="./assets/3tier_jib_after.png" style="width: 80%; height: auto;"/>  

<br/>

이제 pipeline 을 구성 합니다. 우리는 jib을 사용하여 github 에 push 하는 것을 기준으로 설명 합니다.  

<br/>

Jib을 사용하기 위해서는  pom.xml 화일에 plugins 을 추가합니다.
- 반드시 버전을 3.3.x 이상 버전으로 해야 github에 push 시 에러가 발생하지 않습니다.  

<br/>

<img src="./assets/3tier_jib_pom.png" style="width: 80%; height: auto;"/>  

<br/>

Jenkinsfile_github 화일에 본인의 환경에 맞게 설정합니다.  

<br/>

- podTemplate : slave pod 에 생성되는 container 를 명시합니다.  
    - serviceAccount : jenkins-admin
    - namespace : slave 가 기동하는 namespace
    - containers : 컨테이너가 여러개 올 수 있음
    - containerTemplate [ build-tools ] : 실제 빌드하는 컨테이너.  privileged: true
    - containerTemplate [ jnlp ] : master 와 통신용 컨테이너
    - volumes [ hostPathVolume ]: podman 과 cri-o 연결
    - volumes [ persistentVolumeClaim ]: slave 로그 기록을 위한 볼륨
- stage('Clone Git Project') : Git source clone
- stage('Maven/Jib Build & Image Push') : build-tools 컨테이너 안에서 maven 으로 컴파일 하고 jib으로 도커 빌드하고 push 합니다.
- stage('GitOps update') : gitops 폴더에 kustomize를 사용하여 update 하고 git 에 도커 tag를 변경합니다.  

<br/>

```bash
def label = "agent-${UUID.randomUUID().toString()}"
def gitBranch = 'master'
def docker_registry = "ghcr.io"  

// 본인 이미지 사용
def imageName = "ghcr.io/shclub/edu12-backend"
def fromImage = "ghcr.io/shclub/jre17-runtime:v1.0.0"
def git_ops_name = "edu12-backend-gitops"
// 본인 Namespace
def P_NAMESPACE = "edu30"
// 본인 slave pvc
def P_SLAVE_PVC = "jenkins-edu-slave-pvc"
// 본인 이메일
def P_EMAIL = "shclub@gmail.com"


def TAG = getTag(gitBranch)

podTemplate(label: label, serviceAccount: 'jenkins-admin', namespace: P_NAMESPACE,
    containers: [
        containerTemplate(name: 'build-tools', image: 'ghcr.io/shclub/build-tool:v1.0.0', ttyEnabled: true, command: 'cat', privileged: true, alwaysPullImage: true)
        ,containerTemplate(name: 'jnlp', image: 'ghcr.io/shclub/jenkins/jnlp-slave:latest-jdk11', args: '${computer.jnlpmac} ${computer.name}')
    ],
    volumes: [
        hostPathVolume(hostPath: '/etc/containers' , mountPath: '/var/lib/containers' ),
        persistentVolumeClaim(mountPath: '/var/jenkins_home', claimName: P_SLAVE_PVC,readOnly: false)
        ]){    
      
    node(label) {
    
      stage('Clone Git Project') {
          script{
             checkout scm
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
                        git config --global user.email ${P_EMAIL}
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

<br/><br/>

이제 jenkins 에서 `edu_backend` 라는 이름으로 pipeline 을 구성 하고 빌드를 해봅니다.  

아래 추가 설정을 확인 합니다.

- github_ci 에서 토큰을 생성 할 때 write package 권한이 있어야 합니다.    
  <img src="./assets/3tier_jenkins1.png" style="width: 60%; height: auto;"/>  
- 처음 push 할때 기본값이 private 임으로 public 으로 변경 해야 합니다.
    - 해당 패키지 선택 후 오른쪽에 package settings 선택    
      <img src="./assets/3tier_package1.png" style="width: 80%; height: auto;"/>  
    - 하단에 Danger Zone 이동하여 Change Visibility 클릭하고 가이드에 따라 public으로 변경 ( 패키지 이름 입력 ). 1회만 변경하면 됨    
      <img src="./assets/3tier_package2.png" style="width: 80%; height: auto;"/>   

<br/>

Github에서 package 가 생성되었는지 확인합니다.  
TAG 가 빌드 시간 입니다.  

<img src="./assets/3tier_gitops4.png" style="width: 60%; height: auto;"/>

<br/><br/>


### ArgoCD 배포

<br/>

이제 ArgoCD 에 배포를 진행 하도록 합니다.  
Dependency 때문에  backend 를 먼저 배포 합니다.

<br/>

Backend 를 배포합니다.  

이름이 겹치지 않도록 `<본인 namespace>-backend` 로 이름을 만든다.

<br/>

New APP 클릭 한후 Application 이름을 위에 제시한 명명 규칙 으로 기입한다.  
프로젝트 이름은 본인의 Namespace를 입력한다.  

<img src="./assets/3tier_backend_argo1.png" style="width: 60%; height: auto;"/>

<br/>

gitops 프로젝트 이름과 path 는  `.` 으로 입력하고 계속하여 k8s 이름과 namespace 를 넣으면 된다.

<img src="./assets/3tier_backend_argo2.png" style="width: 60%; height: auto;"/>

<br/>

ArgoCD에서 kustomization 화일을 입력하여 보여준다.

<img src="./assets/3tier_backend_argo3.png" style="width: 60%; height: auto;"/>

<br/>

Create 버튼을 클릭하여 생성을 하고 Sync 버튼을 클릭한다.

<img src="./assets/3tier_backend_argo4.png" style="width: 60%; height: auto;"/>

<br/>

Syncronize 버튼을 클릭하여 Application 생성을 완료 한다.

<img src="./assets/3tier_backend_argo5.png" style="width: 60%; height: auto;"/>

<br/>

정상적으로 Resource 들이 생성 된 것을 확인 할 수 있다.  

<img src="./assets/3tier_backend_argo6.png" style="width: 60%; height: auto;"/>

<br/>

Frontend를 배포해 본다.  

이름이 겹치지 않도록 `<본인 namespace>-frontend` 로 이름을 만든다.

<br/>

New APP 클릭 한후 Application 이름을 위에 제시한 명명 규칙 으로 기입한다.  
프로젝트 이름은 본인의 Namespace를 입력한다.  

<img src="./assets/3tier_argo1.png" style="width: 60%; height: auto;"/>

<br/>

gitops 프로젝트 이름과 path 는  `.` 으로 입력하고 계속하여 k8s 이름과 namespace 를 넣으면 된다.

<img src="./assets/3tier_argo2.png" style="width: 60%; height: auto;"/>

<br/>

ArgoCD에서 kustomization 화일을 입력하여 보여준다.

<img src="./assets/3tier_argo3.png" style="width: 60%; height: auto;"/>

<br/>

Create 버튼을 클릭하여 생성을 하고 Sync 버튼을 클릭한다.

<img src="./assets/3tier_argo4.png" style="width: 60%; height: auto;"/>

<br/>

Syncronize 버튼을 클릭하여 Application 생성을 완료 한다.

<img src="./assets/3tier_argo5.png" style="width: 60%; height: auto;"/>

<br/>

정상적으로 Resource 들이 생성 된 것을 확인 할 수 있다.  

<img src="./assets/3tier_argo6.png" style="width: 60%; height: auto;"/>

<br/>

Frontend 와 Backend 배포 2개가 완료가 되어야 한다.  

<img src="./assets/3tier_argo7.png" style="width: 60%; height: auto;"/>

<br/>

에러 없이 정상적으로 배포가 되었으면 route와 service가 잘 구성 되었는지 확인한다.  
모두 80 포트가 있어야 한다.  

```bash
root@newedu:~# kubectl get route
NAME       HOST/PORT                                  PATH   SERVICES   PORT   TERMINATION     WILDCARD
frontend   frontend-edu30.apps.211-34-231-82.nip.io          frontend   80                     None
jenkins    jenkins-edu30.apps.211-34-231-82.nip.io           jenkins    http   edge/Redirect   None
```  

<br/>

```bash
root@newedu:~# kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
backend          NodePort    172.30.135.131   <none>        80:31198/TCP     38m
frontend         ClusterIP   172.30.139.233   <none>        80/TCP           4m5s
jenkins          NodePort    172.30.247.178   <none>        8080:30332/TCP   10d
jenkins-agent    ClusterIP   172.30.140.40    <none>        50000/TCP        10d
```  

<br/>

웹브라우저에서 route 를 입력하고 로그인 창이 나오면 edu/edu1234로 로그인 한다.  


<img src="./assets/3tier_e2e_test1.png" style="width: 80%; height: auto;"/>

<br/>

데이터가 조회 되면 생성 / 수정이 가능하다.

<img src="./assets/3tier_e2e_test2.png" style="width: 80%; height: auto;"/>
 
<br/>

Chrome 에서 More Tools -> Developer Tools -> Console 메뉴에서 보면 JWT 토큰이 생성이 되어 통신하는 것을 확인 할 수 있다.  

<img src="./assets/3tier_e2e_test3.png" style="width: 80%; height: auto;"/>
 
<br/>

##  React/SpringBoot/MariaDB 3-tier 구조 한번에 배포 하기

<br/>

지금까지는 DB 와 Backend 그리고 Frontend를 별도로 배포하는 방식을 사용을 하였습니다.  

이제 3개의 모듈을 한번에 배포하는 예제를 테스트 하도록 하겠습니다.  

<br/>

먼저 Argocd의 Frontend/Backend Application을 삭제 합니다.    

<br/>

프로젝트를 선택후 X 버튼을 클릭합니다.  

<img src="./assets/argocd_delete1.png" style="width: 60%; height: auto;"/>

<br/>

Application 이름을 입력하고 OK 버튼은 클릭하면 삭제가 완료 됩니다.  

<img src="./assets/argocd_delete2.png" style="width: 60%; height: auto;"/>
 

<br/>

그리고 Helm 으로 설치한 MariaDB도 삭제 합니다.    

먼저 `helm list` 명령으로 helm 배포 이름을 확인 합니다.  

```bash
root@newedu:~# helm list
NAME      	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART         	APP VERSION
jenkins   	edu1     	1       	2023-03-26 05:17:45.605297804 +0000 UTC	deployed	jenkins-4.3.9 	2.387.1
my-release	edu1     	1       	2023-03-20 06:47:14.793760159 +0000 UTC	failed  	mariadb-11.5.4	10.6.12
```

<br/>

`my-release` 라는 이름으로 설치된 mariadb를 삭제.

```bash
root@newedu:~# helm delete my-release
```

<br/>

### ArgoCD Apps-of-Apps 패턴

<br/>

참고 
- https://jenakim47.tistory.com/75
- https://malwareanalysis.tistory.com/478


<br/>

ArgoCD application을 모아서 관리하는 패턴을 app of apps 패턴이라고 합니다. app of app패턴으로 구성된 application을 sync하면 여러 argoCD application을 생성합니다.  

하나의 폴더에 child application 을 생성하도록 하는 App of Apps 패턴을 사용하게 되면 배포 및 구성할 수 있는 apps 들을 선언적으로 관리 할 수 있습니다.  

- https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/


<br/><br/>

App-of-Apps 패턴은 Helm 구조로 구성이 됩니다.    

<br/>

우리는 https://github.com/shclub/edu14-2 repository를 예제로 사용을 합니다.    

fork를 하여 서버에 복사를 먼저 하고 본인의 VM 에 git clone 명령어를  사용하여 다운받습니다.  

<br/>

```bash
root@newedu:~/# git clone https://github.com/shclub/edu14-2.git
root@newedu:~# cd edu14-2
root@newedu:~/edu14-2# tree
```

<br/>

```bash
├── application.yaml
├── apps
│   ├── frontend.yaml
│   ├── helm-db.yaml
│   └── kustomize-backend.yaml
├── charts
│   ├── backend
│   │   ├── backend-deployment.yaml
│   │   ├── backend-svc.yaml
│   │   └── kustomization.yaml
│   ├── frontend
│   │   └── manifest.yaml
│   └── helm-db
│       ├── Chart.lock
│       ├── Chart.yaml
│       ├── README.md
│       ├── charts
│       │   └── common
│       │       ├── Chart.yaml
│       │       ├── README.md
│       │       ├── templates
│       │       └── values.yaml
│       ├── schema.sql
│       ├── templates
│       ├── values.schema.json
│       └── values.yaml
```  

<br/>

구조를 보면 Main Application 화일 인 application.yaml 이 있고
apps 폴더 안에 3개의 폴더가 있습니다.   

- frontend.yaml : React Frontend 관련 yaml ( manifest로 배포 )
- helm-db.yaml : Helm 방식으로 mariaDB 설치. pv/pvc 없이 Dynamic Provisioning으로 설치
- kustomize-backend.yaml : Kustomize로 SpringBoot Backend 배포 


<br/>

application.yaml 화일은 다음과 같고 본인의 namespace 에 맞게 수정이되어야 합니다.
단 배포를 위한 namespace는 argocd 이어야 합니다.    

이미 다운 받은 화일을 변경하면 됩니다.  

<br/>

application.yaml  

```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: edu31-apps  # 이름 변경 필요 
  namespace: argocd # 수정하지 말것. argocd 유지
  # Add a this finalizer ONLY if you want these to cascade delete.
  finalizers:
    - resources-finalizer.argocd.argoproj.io

spec:

  project: edu31  # # argocd 프로젝트 이름으로 변경.
  
  source:
    repoURL: https://github.com/shclub/edu14-2.git # 본인의 git 주소
    targetRevision: HEAD
    path: apps # child Application 폴더 위치
  
  destination:
    server: https://kubernetes.default.svc
    # The namespace will only be set for namespace-scoped resources that have not set a value for .metadata.namespace
    namespace: edu31  #  본인의 namespace로 변경
  
  syncPolicy:
    automated: 
      prune: true 
      selfHeal: true   
    # Namespace Auto-Creation ensures that namespace specified as the application destination exists in the destination cluster.  
    syncOptions:
      - CreateNamespace=false 
```

<br/>

apps 폴더 아래에 있는 Child Application 3개 yaml 화일에서 사용자의 환경에 맞게 변경합니다.      

<br/>

이미 다운 받은 화일을 변경하면 됩니다.    

> apps 폴더는 아래와 같다.  

<br/>

- frontend.yaml   

  ```bash
  apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: edu31-frontend  # 본인 Namespace로 변경. <namespace>-frontend
    namespace: argocd # 절대 변경 하지 말것
    # Add a this finalizer ONLY if you want these to cascade delete.
    finalizers:
      - resources-finalizer.argocd.argoproj.io

  spec:
    project: edu31 # argocd 프로젝트 이름으로 변경
    
    source:
      repoURL: https://github.com/shclub/edu14-2.git # 본인의 git 으로 변경
      targetRevision: HEAD
      path: charts/frontend # chart 폴더 아래 frontend 확인
    
    destination:
      server: https://kubernetes.default.svc
      namespace: edu31 # 본인의 namespace로 변경
      
    syncPolicy:
      automated: 
        prune: true 
        selfHeal: true   
      # Namespace Auto-Creation ensures that namespace specified as the application destination exists in the destination cluster.  
      syncOptions:
        - CreateNamespace=false   
  ```  

  <br/>


- helm-db.yaml   

  ```bash
  apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: edu31-helm-db # 본인 Namespace로 변경. <namespace>-helm-db
    namespace: argocd # 절대 수정 하지 말것
    # Add a this finalizer ONLY if you want these to cascade delete.
    finalizers:
      - resources-finalizer.argocd.argoproj.io

  spec:
    project: edu31 # argocd 프로젝트 이름
    
    source:
      repoURL: https://github.com/shclub/edu14-2.git # 본인의 git으로 변경
      targetRevision: HEAD
      path: charts/helm-db # charts 폴더 밑에 helm-db 확인
    
    destination:
      server: https://kubernetes.default.svc
      namespace: edu31 # 본인의 namespace로 변경
    
    syncPolicy:
      automated: 
        prune: true 
        selfHeal: true   
      # Namespace Auto-Creation ensures that namespace specified as the application destination exists in the destination cluster.  
      syncOptions:
        - CreateNamespace=false      
  ```  

  <br/>      

- kustomize-backend.yaml   

  ```bash
  apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: edu31-kustomize-backend # 본인 Namespace로 변경. <namespace>-kustomize-backend
    namespace: argocd  # 절대 수정 하지 말것
    # Add a this finalizer ONLY if you want these to cascade delete.
    finalizers:
      - resources-finalizer.argocd.argoproj.io

  spec:
    project: edu31 # argocd 프로젝트 이름
    
    source:
      repoURL: https://github.com/shclub/edu14-2.git # 본인의 git으로 변경
      targetRevision: HEAD
      path: charts/backend # charts 폴더안에 backend 확인
    
    destination:
      server: https://kubernetes.default.svc
      namespace: edu31 # 본인의 namespace
    
    syncPolicy:
      automated: 
        prune: true 
        selfHeal: true   
      # Namespace Auto-Creation ensures that namespace specified as the application destination exists in the destination cluster.  
      syncOptions:
        - CreateNamespace=false      
    ```  

  <br/>        

<br/><br/>

charts 폴더 아래에 있는 Child Application 폴더 (frontend,backend,helm-db) 의 yaml 화일에서 사용자의 환경에 맞게 변경합니다.      

<br/>

이미 다운 받은 화일을 변경하면 됩니다.  

> charts 폴더는 아래와 같다.  

<br/>

- frontend/manifest.yaml   

  ```bash
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: frontend
    labels:
      app: frontend
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: frontend
    template:
      metadata:
        labels:
          app: frontend
      spec:
        containers:
        - name: frontend
          image: ghcr.io/shclub/edu12-3:v10 # 본인의  도커 이미지. 없으면 그대로 사용
          env:
          - name: BACKEND_API_URL
            value: "http://backend" 
          ports:
          - containerPort: 80
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: frontend
  spec:
    selector:
      app: frontend
    ports:
      - protocol: TCP
        port: 80
        targetPort: 80
    type: ClusterIP   
  ---
  apiVersion: route.openshift.io/v1
  kind: Route
  metadata:
    name: frontend
  spec:
    host: frontend-edu31.apps.211-34-231-82.nip.io # 본인의 namespace로 적용
    port:
      targetPort: 80
    to:
      kind: Service
      name: frontend
      weight: 100    
    wildcardPolicy: None     
  ```  

<br/>


- backend/kustomization.yaml   

  ```bash
  apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization
  resources:
  - backend-deployment.yaml
  - backend-svc.yaml     
  ```  

  <br/>

- backend/backend-svc.yaml   

  ```bash
  apiVersion: v1	
  kind: Service	
  metadata:	
    name: backend	
  spec:	
    ports:	
    - port: 80	
      targetPort: 8080	
    selector:	
      app: backend
    type: NodePort    
  ```  

  <br/>

- backend/backend-deployment.yaml   

  ```bash
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: backend
  spec:
    replicas: 1
    revisionHistoryLimit: 3
    selector:
      matchLabels:
        app: backend
    template:
      metadata:
        labels:
          app: backend
      spec:
        containers:
        - name: backend
          image: ghcr.io/shclub/edu12-4:v10 # 본인의 이미지로 변경. 없으면 해당 이미지 사용
          #image: shclub/edu12-4:v10
          env:
          - name: SPRING_PROFILES_ACTIVE
            value: "prd"
          - name: SPRING_DATASOURCE_USERNAME
            value: "edu"
          - name: SPRING_DATASOURCE_PASSWORD
            value: "caravan"
          - name: SPRING_DATASOURCE_URL
            value: "jdbc:mariadb://helm-db-mariadb:3306/edu"
          ports:
          - containerPort: 8080  
  ```  

  <br/>


helm-db 폴더의 values.yaml을 수정하기 전에 dynamic provisioning의 이름을 알아내기 위해 storageclass를 조회합니다.  

<br/>

```bash
root@newedu:~/edu14-2/charts/helm-db# kubectl get storageclass
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   207d
```   

<br/>

- helm-db/values.yaml   

  ```bash
    11 global:
    12   imageRegistry: ""
    17   imagePullSecrets: []
    18   storageClass: "nfs-client" # stroage class 기입
    ...
    77 image:
    78   registry: ghcr.io
    79   repository: shclub/mariadb
    80   tag: 10.6.9-debian-11-r4 
    ...
    100 architecture: standalone
    101 ## MariaDB Authentication parameters
    102 ##
    103 auth:
    104   ## @param auth.rootPassword Password for the `root` user. Ignored if existing secret is provided.
    105   ## ref: https://github.com/bitnami/containers/tree/main/bitnami/mariadb#setting-the-root-password-on-first-run
    106   ##
    107   rootPassword: edu1234 # root 비밀번호 
    108   ## @param auth.database Name for a custom database to create
    109   ## ref: https://github.com/bitnami/containers/blob/main/bitnami/mariadb/README.md#creating-a-database-on-first-run
    110   ##
    111   database: edu  # DB 이름
    112   ## @param auth.username Name for a custom user to create
    113   ## ref: https://github.com/bitnami/containers/blob/main/bitnami/mariadb/README.md#creating-a-database-user-on-first-run
    114   ##
    115   username: edu # 사용자 계정
    116   ## @param auth.password Password for the new user. Ignored if existing secret is provided
    117   ##
    118   password: caravan  # edu 계정의 password
    ...
    152 #initdbScripts: {}
    153 initdbScripts:
    154 my_init_script.sh: |
    155 #!/bin/bash
        # 아래는 초기 mariaDB 기동시에 진행하는 DB script는 넣을 수 있다.  
    156 echo "init db script start"
          mysql -P 3306 -uedu -pcaravan -e  "use edu; create sequence hibernate_sequence; create table EMPLOYEE (id int primary key,empName varchar(255),empDeptName varchar(255),empTelNo varchar(20),empMail  varchar(25));insert into EMPLOYEE  values(1,'1','1','1','1');SELECT NEXTVAL(hibernate_sequence)";    
    ...
    304   podSecurityContext:
    305     enabled: true
    306     fsGroup: 1000690000 #1001 # 본인 Namespace 의 openshift.io/sa.scc.uid-range
    ...
    313   containerSecurityContext:
    314     enabled: true
    315     runAsUser: 1000690000 #1001 # 본인 Namespace 의 openshift.io/sa.scc.uid-range
    316     runAsNonRoot: true
    ...
    427   persistence:
    428     ## @param primary.persistence.enabled Enable persistence on MariaDB primary replicas using a `PersistentVolumeClaim`. I     f false, use emptyDir
    429     ##
    430     enabled: true
    431     ## @param primary.persistence.existingClaim Name of an existing `PersistentVolumeClaim` for MariaDB primary replicas
    432     ## NOTE: When it's set the rest of persistence parameters are ignored
    433     ##
    434     existingClaim: ""  # 불필요 
    435     ## @param primary.persistence.subPath Subdirectory of the volume to mount at
    436     ##
    437     subPath: "my-mariadb" # subPath
    438     ## @param primary.persistence.storageClass MariaDB primary persistent volume storage Class
    439     ## If defined, storageClassName: <storageClass>
    440     ## If set to "-", storageClassName: "", which disables dynamic provisioning
    441     ## If undefined (the default) or set to null, no storageClassName spec is
    442     ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    443     ##   GKE, AWS & OpenStack)
    444     ##
    445     storageClass: ""
    446     ## @param primary.persistence.annotations MariaDB primary persistent volume claim annotations
    447     ##
    448     annotations: {}
    449     ## @param primary.persistence.accessModes MariaDB primary persistent volume access Modes
    450     ##
    451     accessModes:
    452       - ReadWriteOnce
    453     ## @param primary.persistence.size MariaDB primary persistent volume size
    454     ##
    455     size: 4Gi # 사이즈
  ...
   549 secondary:
    550   ## @param secondary.name Name of the secondary database (eg secondary, slave, ...)
    551   ##
    552   name: secondary
    553   ## @param secondary.replicaCount Number of MariaDB secondary replicas
    554   ##
    555   replicaCount: 0  # primary 사용으로 여기서는 0 으로 설정
  ```  

  <br/>

<br/>

- helm-db/templates/primary/svc.yaml  

  ```bash
    3  metadata:
    4 #  name: {{ include "mariadb.primary.fullname" . }}
    5 name: helm-db-mariadb  # DB 서비스 이름 명시
  ```


<br/>

이제 App-of-Apps 패턴을 배포 합니다.    

```bash
root@newedu:~/edu14-2# kubectl apply -f application.yaml
```  

<br/>

POD 를 조회해본다.  

```bash
root@newedu:~/edu14-2# kubectl get po -n edu31
NAME                                READY   STATUS    RESTARTS   AGE
backend-6f99f7d8d9-v99bj            1/1     Running   0          12d
edu31-helm-db-mariadb-0             1/1     Running   0          12d
frontend-58ddf9dddc-z6lsq           1/1     Running   0          12d
```  

<br/>

ArgoCD에 가서 4개의 Application 이 생성 된걸 확인 할 수 있다.    

<img src="./assets/app_of_apps1.png" style="width: 80%; height: auto;"/>

<br/>

Main Application 인 edu31-apps를 클릭한다.    

세부적으로 보면 3개의 Application과 연결 되어 있는 것을 볼 수 있다.

<img src="./assets/app_of_apps2.png" style="width: 80%; height: auto;"/>

<br/>

backend 구성을 보실려면 edu31-backend를 클릭해서 확인한다.   


<img src="./assets/app_of_apps3.png" style="width: 80%; height: auto;"/>

<br/>

DB구성을 확인하려면 NFS 에 들어가서 아래 폴더에 가면 자동으로 폴더 이름이 생성 된 것을 확인 할 수 있고  pv/pvc도 자동으로 생성된 것을 확인 할 수 있습니다.   

```bash
[root@edu database]# pwd
/mnt/database
[root@edu database]# ls
[root@edu database]# ls
edu31-data-edu31-helm-db-mariadb-0-pvc-b98cc9b5-90f2-4ed9-8029-34727a9e4c81  
```
<br/>

생성된 폴더로 들어가면 values.yaml 에 설정한 subPath 인 my-mariadb 가 있는 것을 확인 할 수 있다.  

<br/>

```bash
[root@edu edu31-data-edu31-helm-db-mariadb-0-pvc-b98cc9b5-90f2-4ed9-8029-34727a9e4c81]# ls
my-mariadb
```

<br/>

pv/pvc 도 자동으로 생성된 것을 확인합니다.

<br/>

```bash
root@newedu:~/edu14-2# kubectl get pv
pvc-b98cc9b5-90f2-4ed9-8029-34727a9e4c81   4Gi        RWO            Delete           Bound         edu31/data-edu31-helm-db-mariadb-0                        nfs-client              133d
root@newedu:~/edu14-2# kubectl get pvc -n edu31
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-edu31-helm-db-mariadb-0   Bound    pvc-b98cc9b5-90f2-4ed9-8029-34727a9e4c81   4Gi        RWO            nfs-client     133d
```  

<br/>


웹브라우저에서 본인의 route 로 접속하여 로그인 해본다.  

- https://frontend-edu31.apps.211-34-231-82.nip.io/
