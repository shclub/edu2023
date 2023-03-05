
# 실습 프로그램 설치 

교육에 앞서 실습에 필요한 프로그램을 설치합니다.   

<br/>

1. Cloud Shell ( Terminal / Bastion ) 용 VM 서버 접속 

2. Docker 설치

3. Docker compose 설치

4. openshift console 설치

5. Git 설치

6. Helm 설치

7. GitHub 계정 생성

8. DockerHub 계정 생성


<br/>


## Cloud Shell 용 VM 서버 접속

<br/>

사전에 공유 한 VM 서버 ( OS : Ubuntu ,  CPU : 2core , MEM : 2G )에 접속합니다.  

터미널에서 아래 명령어를 입력하여 로그인 한다.  

로그인 후  connecting 저장 질문이 나오면 yes 를 입력한다. 

```bash
ssh root@(본인 Public ip) 
``` 

<br/>

## Docker 설치

<br/>

> 도커 설치 : https://youtu.be/w8EVLx1_xY0

<br/> 

### 패키지 인덱스 업데이트

<br/>

```bash
apt-get update
```
<br/>

### HTTPS를 통해 repository 를 이용하기 위해 package 들을 설치

<br/>

```bash
apt-get -y install  apt-transport-https ca-certificates curl gnupg lsb-release
```
<br/>

### Docker의 Official GPG Key 를 등록합니다.

<br/>

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

<br/>

### stable repository를 등록합니다.  

<br/>

```bash
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```  
<br/>

### Docker Install

<br/>

```bash
apt-get update && apt-get install docker-ce docker-ce-cli containerd.io gnupg2 pass
```

* Ubuntu 에서 도커 로그인 버그가 있어 아래 처럼 에러가 발생하기 때문에 gnupg2 pass 라이브러리를 추가 했음.  

```bash
$ docker login -u shclub -p ******** https://index.docker.io/v1/
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Error saving credentials: error storing credentials - err: exit status 1, out: `Cannot autolaunch D-Bus without X11 $DISPLAY`
```

<br/>

### 도커 버전을 확인합니다.

<br/>

```bash
docker --version
```
<img src="./assets/docker_version.png" style="width: 60%; height: auto;"/>

<br/>

### 도커 이미지 다운로드 및 실행하기

<br/>

```bash
docker run hello-world
```
 
<img src="./assets/docker_run_world.png" style="width: 80%; height: auto;"/>


<br/>

## Docker compose 설치.

<br/>

VM 에서 docker compose를 설치 합니다.  

<br/>

```bash
apt-get update && apt-get install docker-compose
```

<br/>

중간에 추가 설치 내용이 나오면 Y를 입력하고 엔터를 친다.

<img src="./assets/docker_compose_install.png" style="width: 60%; height: auto;">

<br/>

도커 컴포즈 버전을 확인하고 아래와 같이 나오면 정상적으로 설치가 된 것이다.

<br/>

```bash
docker-compose --version
```  

<br/>

<img src="./assets/docker_compose_version.png" style="width: 60%; height: auto;">  

<br/>

## openshift console 설치

<br/>

VM 에서 아래와 같이 openshift client를 설치 합니다.  

```bash
root@edu2:~# wget https://github.com/openshift/okd/releases/download/4.7.0-0.okd-2021-09-19-013247/openshift-client-linux-4.7.0-0.okd-2021-09-19-013247.tar.gz
```   

<br/>

tar 화일을 압축을 풉니다.  

```bash
root@edu2:~# ls
cloud-init-setting.sh  openshift-client-linux-4.7.0-0.okd-2021-09-19-013247.tar.gz
root@edu2:~# tar xvfz openshift-client-linux-4.7.0-0.okd-2021-09-19-013247.tar.gz
```  

<br/>

oc 와 kubectl 화일이 생성 된 것을 확인 할수 있습니다.  
oc 는 openshift console 이고 kubectl 은 kubernetes client tool 입니다.  


```bash
root@edu2:~# ls
README.md  cloud-init-setting.sh  kubectl  oc  openshift-client-linux-4.7.0-0.okd-2021-09-19-013247.tar.gz
```  

<br/>

path를 추가 합니다.  

```bash
echo 'export PATH=$PATH:.' >> ~/.bashrc && source ~/.bashrc
```  

<br/>

oc 명령어를 입력해 봅니다.  

```bash
root@edu2:~# oc
OpenShift Client

This client helps you develop, build, deploy, and run your applications on any
OpenShift or Kubernetes cluster. It also includes the administrative
commands for managing a cluster under the 'adm' subcommand.

To familiarize yourself with OpenShift, login to your cluster and try creating a sample application:

    oc login mycluster.mycompany.com
    oc new-project my-example
    oc new-app django-psql-example
    oc logs -f bc/django-psql-example

To see what has been created, run:

    oc status

and get a command shell inside one of the created containers with:

    oc rsh dc/postgresql

To see the list of available toolchains for building applications, run:

    oc new-app -L

Since OpenShift runs on top of Kubernetes, your favorite kubectl commands are also present in oc,
allowing you to quickly switch between development and debugging. You can also run kubectl directly
against any OpenShift cluster using the kubeconfig file created by 'oc login'.

For more on OpenShift, see the documentation at https://docs.openshift.com.

To see the full list of commands supported, run 'oc --help'.
```  

<br/>

VM 에서 접속 테스트를 합니다.  
아이디는 `namespace 이름 - admin` 으로 구성이 되고 namespace 생성시에 자동 생성이 됩니다.  


<br/>

`oc login <API SERVER:포트> -u <아이디> -p <패스워드> --insecure-skip-tls-verify`

<br/>

```bash
root@edu2:~# oc login https://api.211-34-231-81.nip.io:6443 -u edu1-admin -p New1234! --insecure-skip-tls-verify
Login successful.

You have one project on this server: "edu1"

Using project "edu1".
Welcome! See 'oc help' to get started.
```  

<br/>


## Git 설치

<br/>

Git이란 소스코드를 효과적으로 관리하기 위해 개발된 '분산형 버전 관리 시스템'입니다. ( 이전 에는 SVN 많이 사용 )  

가장 대중적인 SaaS 형태는 Microsoft에서 제공하는 GitHub 이고
Private 형태로는 Gitlab을 많이 사용 함.  

<br/>

참고 사이트 :  https://backlog.com/git-tutorial/kr/intro/intro1_1.html    

<br/>


본인의 PC에 git을 설치하고 버전을 확인한다.  

```bash
git --version
``` 

<img src="./assets/git_version.png" style="width: 60%; height: auto;"/>

<br/>


## Helm 설치.

<br/>


helm 3.x 이상 버전을 설치한다.

<br/>


```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```  

<br/>


```bash
chmod 700 get_helm.sh
```

```bash
./get_helm.sh
```  

<img src="./assets/helm_install.png" style="width: 80%; height: auto;"/>  

버전을 확인한다.

```bash
helm version
```

<img src="./assets/helm_version.png" style="width: 80%; height: auto;"/>  

helm repository 목록을 조회합니다. 처음 설치 했을때는 아무것도 없습니다.

```bash
helm repo list
```

<br/>

## GitHub 계정 생성

<br/>

https://github.com/ 접속하고 계정 생성  

<br/>

계정 생성 후에 Repository를  생성한다.

<br/>

아래와 같이 이름 입력를 하고 README file check 를 한다

<br/>

<img src="./assets/repository_create.png" style="width: 80%; height: auto;"/>  

default 브랜치를 main에서 master로 변경한다. ( 맨 하단 setting 클릭하여 설정)

<br/>

<img src="./assets/default_branch_modify.png" style="width: 60%; height: auto;"/>

<br/>

교육용 repository인 https://github.com/shclub/edu1 폴더의 파일을 복사하여 본인이 생성한 Repository에 신규 화일을 생성한다. 

<br/>

총 4개 화일을 만들고 내용을 복사한다.  ( 향후 Git 사용법 교육 후 Git Clone 사용 )

<img src="./assets/shclub_edu_file.png" style="width: 60%; height: auto;"/>

   - 샘플은 pyhon flask 로 구성

<br/><br/>

##  Docker Hub 계정 생성 

<br/>

https://hub.docker.com/ 접속하고 계정 생성  
- 향후 사내에서 개발시는 private docker registry Nexus 사용  

<br/>

### Docker 연동 테스트를 한다.

```bash
docker tag hello-world (본인id)/hello-world
docker push (본인id)/hello-world  
```

- 권한 에러 발생시 docker login 한다
```bash
docker login 
```

<img src="./assets/docker_denied.png" style="width: 60%; height: auto;"/>

정상적으로 로그인후 push를 한다.

```bash
docker push (본인id)/hello-world  
```     

<br/>

<img src="./assets/docker_push.png" style="width: 60%; height: auto;"/>

도커허브 본인 계정에서 도커 이미지 생성 확인  

<img src="./assets/docker_hub_world.png" style="width: 60%; height: auto;"/>


도커 이미지가 Private으로 되어 있으면 Public 으로 변경한다. 
- 개인 계정은 1개의 private 만 가능

setting 으로 이동하여 Make public 클릭후 repository 이름을 입력후 Make Public 클릭  

<img src="./assets/docker_hub_make_public.png" style="width: 60%; height: auto;"/>