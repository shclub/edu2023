# 본 과정에 대해  ( Cloud Native 입문 과정 )
 
본 교육 과정은 Cloud Native 입문 과정으로 이론 및 실습을 수행한다. ( 16 시간 오프라인 교육 )
 

문의 :  이석환 ( seokhwan.lee@kt.com / shclub@gmail.com )

<br/>


0. Chapter 0 : 프로그램 설치  ( [가이드 문서보기](./install.md) )  

     - Cloud Shell ( Terminal / Bastion ) 용 VM 서버 접속  
     - Docker / Docker Compose 설치
     - Openshift Console 설치 ( kubectl 같이 설치 됨 )
     - Git 설치
     - Helm 설치
     - Github 계정 생성
     - Docker Hub 계정 생성 

     <br/>


1. Chapter 1 : 2시간  ( [가이드 문서보기](./chapter1.md) )  

     - Docker 이해 및 활용 
     - GitHub , Docker 계정 생성 , Jenkins Pipeline 생성하여
       CI 실습 

     <br/>

2. Chapter 2 : 1시간  ( [가이드 문서보기](./chapter2.md) )  

     - Private Docker Registry 설명 및 구축 하기 ( /w Nexus ) 
     - Remote 로 연결하기  ( Insecure Registry 설정 )
          
     <br/>
     
     - 과제  : Chapter2의 Jenkins Pipeline 을 Private Docker registry로 변경해 보기 

     <br/>

3. Chapter 3 : 1시간  ( [가이드 문서보기](./chapter3.md) )  

     - Github Action , workflow 사용 ( GoodBye Jenkins ) 
     - Docker Compose 설명 및 설치 , 활용 ( DB 연동 )  

     <br/>

---
## 1주차 과제

<br/>

Harbor 설치  
- Nexus 컨테이너와 Docker Compose로 기동된 컨테이너를 종료 한다. 
- Private Docker Registry 를 Harbor를 사용하여 구성 한다.
- Harbor 포트는 40002를 사용하며 https 연결하기 위한 인증서 설정을 한다.
- Harbor 에 edu 프로젝트를 생성하고 신규 계정을 생성하여 member에 추가한다.
- nginx 이미지를 본인의 Private Docker Registry에 Push 한다.
- Harbor에서 확인한다.

<br/>

샘플 : https://211.252.85.148:40002/  
계정 : edu/New234!

<br/>

---

3. Chapter 3 : 4시간   ( [가이드 문서보기](./chapter4.md) )  

     - k8s 이해 및 활용
     - k8s hands-on Basic [ Hands-On 문서보기 ](./k8s_basic_hands_on.md)  

          - 실습 전체 개요
          - kubeconfig 설정 :  oc / kubectl 설치
          - kubectl 활용
          - kubernetes 리소스 ( Pod , Service , Deployment 생성 및 삭제)
          - 배포 ( Rolling Update / Rollback )
          - Serivce Expose ( Ingress / Route )  

     <br/>


---
2주차 과제
---


4. Chapter 4 : 10분   ( [가이드 문서보기]( https://github.com/shclub/edu/blob/master/chapter9.md ) ) 

     - OKD 4.7 설치 ( kt cloud FlyingCube 2.0 )  
          - NFS 설정 및 접속  
     - ArgoCD 설치 및 설정  ( OKD )  

    <br/>

5. Chapter 5 : 4시간   ( [가이드 문서보기](./chapter5.md) ) 

     - GitOps 설명
     - kustomize 설명 및 실습
     - k8s에 배포 실습 ( Blue/Green , Canary )  
     - ArgoCD Hands-on [ Hands-On 문서보기 ](./argocd_hands_on.md) 

          - kubectl plugin 설치
          - Blue/Green 배포
          - Canary 배포
          - ArgoCD 계정 추가 및 권한 할당
          - kustomize 사용법
          - ArgoCD remote Cluster 에서 배포 하기 

     <br/>


---
3주차 과제
---

6. Chapter 5 : 4시간   ( [가이드 문서보기](https://github.com/shclub/edu/blob/master/k8s_middle_hands_on.md) ) 

     - Storage Volume  ( PV/PVC , DB 설치 + NFS )
          - MariaDB NFS 에 설치 ( /w Helm Chart ) 
     - NFS 라이브러리 설치 ( Native Kubernetes )
     - Service - Headless, Endpoint, ExternalName
     - Helm Chart 개념 및 설명
     - React/SpringBoot/MariaDB 3-tier 구조 한번에 배포 하기
          - Helm Umbrella 패턴 ( /w SubChart )
          - ArgoCD Apps-of-Apps 패턴 
     
     <br/>

---
4주차 과제
---
