
# 실습 환경 정보 

교육에 앞서 실습에 필요한 환경 정보입니다.   

<br/>

1. Jenkins 접속 정보 

2. OKD 접속 정보

3. ArgoCD 접속 정보

4. KT Cloud KTP 상품 정보

5. kubernetes 접속 정보


<br/>


## Jenkins 접속 정보
 
<br/>

웹브라우저에서 접속 가능.    

http://211.252.85.148:9000/   


- 계정 : <edu + 순번>  ( 예: edu1 )
- 비밀번호 : 사전 공지  


<br/>


## OKD 접속 정보
 
<br/>

```bash
oc login https://api.211-34-231-81.nip.io:6443 -u shclub-admin -p <비밀번호> --insecure-skip-tls-verify
```  

- 계정 : <edu + 순번 + `-admin`>  ( 예: edu1-admin )
- 비밀번호 : 사전 공지    


OKD WEB Console : https://console-openshift-console.apps.211-34-231-82.nip.io/  


<br/>


## ArgoCD 접속 정보
 
<br/>


OKD : https://argocd-argocd.apps.211-34-231-82.nip.io/applications    


K3S : https://211.252.87.34:30000/  ( 임시 )

- 계정 : edu + 순번  ( 예: edu1 )
- 비밀번호 : 사전 공지  

<br/>

## KT Cloud KTP 상품
 
<br/>

```bash
https://cloud.kt.com/portal/user-guide/Container-container-guide
```  

<br/>

## kubernetes 접속 정보
 
<br/>

Kt cloud에 k8s 1.21 버전 cluster 구성하였으니 아래 환경에서 CKA 테스트 가능 합니다.

<br/>

### 환경 구성

<br/>

- OS : Ubuntu
- Master Node : 1개
- Worker Node : 2개
- Infra Node : 2개


<br/>

### 접속방법
 
<br/>

- Master node1 로 접속 : ssh ubuntu@211.34.231.90  
- Worker node1 로 접속 : ssh ubuntu@211.34.231.91  
- API 도메인 : 211-34-231-87.nip.io  
- Ingress 도메인 : 211-34-231-88.nip.io    

