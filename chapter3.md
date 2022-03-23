# Chapter 3 
   

kubernetes 는 light하고 빠른 설치가 가능한 Rancher에서 제공하는 k3s로 진행을 하고 kubernetes 배포 관리자인 helm 에 대한 사용법을 습득한다.

1. k3s 설치 및 기능 설명

2. helm 설치 및 heml 으로 설치 실습


 
<br/>

##  kubernetes

### kubernetes 개요 

Git이란 소스코드를 효과적으로 관리하기 위해 개발된 '분산형 버전 관리 시스템'입니다. ( 이전 에는 SVN 많이 사용 )  

가장 대중적인 SaaS 형태는 Microsoft에서 제공하는 GitHub 이고
Private 형태로는 Gitlab을 많이 사용 함.


### k3s를 설치 한다.
   
터미널로 VM에 로그인 한다.


```bash
ssh root@(본인 private ip) -p 22222
``` 

root 계정으로 진행시 아래와 같이 입력하고 패키지를 업그레이드 한다.  
일반 계정인 경우는 앞에 sudo를 붙인다.

```bash
apt update & apt upgrade
``` 

<img src="./assets/login_apt_update.png" style="width: 60%; height: auto;"/>  

k3s를 설치한다. 몇 초 안에 설치가 된다.  
- kubernetes full version 은 1시간 정도, Openshift는 2시간 정도 설치 시간 소요


```bash
curl -sfL https://get.k3s.io | sh -
```

<img src="./assets/install_k3s.png" style="width: 60%; height: auto;"/>  

k3s 가 설치가 잘되어 있는지 확인한다. Active ( running ) 이며 정상. 

```bash
systemctl status k3s
```  

<img src="./assets/k3s_status.png" style="width: 60%; height: auto;"/>  

ctrl + c 키를 입력하고 화면을 나온다.

<br/>

Kubernetes를 제어할 수 있는 CLI인 kubectl을 이용해, 클러스터가 정상적으로 생성된 것도 확인할 수 있다.   
현재 k3는 master와 worker node가 하나의 vm에 설치 되어 있다.  
kubernetes 버전은 현재 기준 최신인 1.22 이다.

```bash
# Node 상태 확인
kubectl get nodes
```  

```bash  

# Kubernetes 시스템 Pod 상태 확인
kubectl get pod --namespace=kube-system
```  

<img src="./assets/k3s_nodes.png" style="width: 60%; height: auto;"/>  

k3s는 metric-server가 설치 되어 있어 노드의 리소르를 확인 할 수 있다.

```bash
kubectl top nodes
```  
 
<img src="./assets/top_nodes.png" style="width: 80%; height: auto;"/>     


<br/>


### 로컬 컴퓨터에서 원격지 Kubernetes 클러스터 접속 위한 설정 

<br/>

k3s를 설치하면, 클러스터의 인증서와 사용자 비밀번호 등 인증하는데 필요한 정보가 /etc/rancher/k3s/k3s.yaml에 저장됩니다.  

```bash
cat /etc/rancher/k3s/k3s.yaml
```  

<img src="./assets/cat_k3s_yaml.png" style="width: 60%; height: auto;"/>  

이 파일을  복사 하여 로컬 컴퓨터에 복사해오면 됩니다.    

서버 주소가 127.0.0.1로 되어있거나, 이름의 대부분이 default로 되어 있습니다. 하나의 Kubernetes 클러스터만 관리한다면 이름은 문제가 되지 않겠지만, 여러 개의 클러스터에 하나의 컴퓨터에서 접속한다면 이름을 반드시 바꿔주어야 합니다.

2개의 값을 일관변경 한다.  

- default ->  k3s-test ( 총 6개 ) 
- ip는  127.0.0.1 ->  본인 VM서버 ip

```bash
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0Jn
    --- 보안상 삭제
    server: https://210.106.105.76:6443
  name: k3s-test
contexts:
- context:
    cluster: k3s-test
    user: k3s-test
  name: k3s-test
current-context: k3s-test
kind: Config
preferences: {}
users:
- name: k3s-test
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJrVENDQVRlZ0F3SUJBZ0lJUlVoT3dWa
    --- 보안상 삭제
    client-key-data: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSU5lbHd2cW11TWlTUGdpe
    --- 보안상 삭제
```

<img src="./assets/k3s_config_modify.png" style="width: 60%; height: auto;"/> 

변경된 값을 로컬 특정 폴더에  config-k3stest 이름으로 저장한다.  
.kube 폴더가 없으면 생성한다.

- Windows : /Users/본인계정/.kube/config-k3stest
- Mac : ~/.kube/config-k3stest


Kubernetes API 서버의 인증서가 내부 IP만 접속하도록 설정이되어 있어
외부에서 접속 가능하도록  k3s를 설치한 서버의 systemd 설정을 바꿔줍시다.  

/etc/systemd/system/k3s.service를 열고, ExecStart 부분에 --tls-san 설정을 추가해줍니다.  

만약 공인 IP에 도메인을 연결하여 접속하고 싶다면, 공인 IP 대신에 도메인을 입력하면 됩니다.  

vi 에디터로 /etc/systemd/system/k3s.service 수정합니다.


```bash
vi /etc/systemd/system/k3s.service
```  

<img src="./assets/vi_k3s_service.png" style="width: 60%; height: auto;"/> 

```bash
# 원본
ExecStart=/usr/local/bin/k3s \
    server \

# 이렇게 변경해 줍시다
ExecStart=/usr/local/bin/k3s \
    server --tls-san YOUR-K3S-IP-ADDRESS-OR-DOMAIN \
```  

아래와 같이 본인 맨 밑의 라인에 i 입력하여 입력모드로 변경 후 
--tls-san (본인 VM의 공인 IP) 를 입력하고 esc키를 누르고 :wq를 입력하여 저장하고 나온다.

<img src="./assets/vi_tls_san.png" style="width: 60%; height: auto;"/>   

변경이 완료되었다면 systemd 서비스를 다시 시작해 줍니다.

```bash
# 원격 서버에서 실행
systemctl daemon-reload
systemctl restart k3s
kubectl get nodes
```  

<img src="./assets/k3s_reload.png" style="width: 60%; height: auto;"/>  

재기동시 시간이 걸릴 수가 있으며 STATUS가 Ready 이면 재기동 성공.  


<br/>

### Helm 설치 ( https://helm.sh/ko/docs/intro/install/ )

<br/>

Helm 는 kubernetes (k8s)  package managing tool ( 배포 관리자 ) 로서 k8s에 쉽게 Application을 배포 할수 있도록 해준다.  

3가지 컨셉으로 구성되어 있다.

- Chart : Helm package입니다. app을 실행시키기위한 모든 리소스가 정의됨
  - Homebrew formula, Apt dpkg, Yum RPM 파일과 비슷.
- Repository : chart들이 공유되는 공간 
  - docker hub와 유사.
- Release : 쿠버네티스 클러스터에서 돌아가는 app들은(chart instance)  
            모두 고유의 release 버전을 가지고 있다.   


helm은 chart를 쿠버네티스에 설치하고, 설치할때마다 release버전이 생성되며, 새로운 chart를 찾을때에는 Helm chart repository에서 찾을 수 있습니다.


VM 서버에 터미널에서 아래 명령어를 실행한다.   
- helm 3.x 이상 버전을 설치한다.

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

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

k3는 경량 kubernetes로 많은 기능이 기본적으로 설치가 되지 않습니다.  
Lens에서 Metric ( 서버의  리소스 정보 ) 을 보기 위해서 Prometheus 가 설치가 되어야 합니다.

<br/>

prometheus 설치를 하기 위해 helm repository를 추가 합니다.
- 리포지토리 이름 : prometheus-community

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```  

최신 chart 리스트를 업데이트 하기 위해 다음 명령어를 실행한다.

```bash
helm repo update
```  

<img src="./assets/helm_repo_add.png" style="width: 80%; height: auto;"/>    

repository 를 확인합니다.

```bash
helm repo list
```

설치 가능한 chart 목록을 보려면 helm search 명령어를 실행한다.    

```bash
helm search repo prometheus
```

<img src="./assets/helm_search_prometheus.png" style="width: 80%; height: auto;"/>   

Prometheus가 설치될 namespace를 monitoring 이라는 이름으로 생성합니다.

```bash
kubectl create namespace monitoring
```

helm 으로 prometheus를 설치합니다.  

```bash
helm install prometheus --namespace monitoring prometheus-community/kube-prometheus-stack
```  

설치시 아래와 같은  에러가 발생 하면 

```bash
Error: INSTALLATION FAILED: Kubernetes cluster unreachable: Get "http://localhost:8080/version": dial tcp [::1]:8080: connect: connection refused
```

다음 명렁어를 실행하고 다시 prometheus를 설치 한다.

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

정상적으로 실행이 되면 3개의 pod 가 생성된 것을 확인 할 수 있다.

```bash
kubectl get deployments -n monitoring
```  
 
<img src="./assets/helm_prometheus_install.png" style="width: 80%; height: auto;"/>     

<br/>

뒤에서 k8s IDE인 lens 를 확인하면 메트릭 정보를 볼수 있고 
명령어를 통해서도 확인할 수 있다.

<br/>

### kubernetes IDE lens 설치

<br/>

Lens는 쿠버네티스를 모니터링 및 관리 개발할 수 있은 IDE이다.  
기존의 쿠버네티스 대시보드가 localhost만 가능한 반면 LENS는 연결만 하면 원격의 K8S 클러스터도 같이 모니터링 할 수 있다.

특징

- pod 목록 조회 (이제 더이상 terminal에서 kubectl get pods --watch를 입력할 필요없다)
- pod describe 결과를 편하게 볼 수 있음
- pod의 terminal에 쉽게 접근
- pod의 log도 쉽게 볼 수 있고 편하게 검색할 수 있다  

<br/>

웹 브라우저에서 https://k8slens.dev/index.html 에 접속한다.

본인 로컬 PC의 os를 선택하고 파일을 다운로드 받고 설치를 한다.   

<img src="./assets/lens_web.png" style="width: 60%; height: auto;"/>  


설치된 lens 프로그램을 실행하면 welcome 화면이 나오고 하단에 skip 버튼을 눌러 이동한다.

<img src="./assets/lens_welcome_0.png" style="width: 40%; height: auto;"/>  


welcome 화면 다음에  Browse Clusters in catalog를 클릭한다. 

<img src="./assets/lens_welcome.png" style="width: 40%; height: auto;"/>  

Lens가 .kube 폴더 밑의 config 화일들을 자동으로 읽어 온다.  
k3s-test 라는 이름으로 클러스터 이름이 생성 된것 확인 할 수 있다.

<img src="./assets/lens_cluster_list.png" style="width: 60%; height: auto;"/>  


k3s-test 를 클릭하고 왼쪽 메뉴 Cluster 클릭하면 메트릭 정보를 볼 수 있다.
- helm으로 prometheus 설치 하여 가능 

<img src="./assets/lens_metric_install.png" style="width: 80%; height: auto;"/>  

<br/>
lens 화면 구성 

<img src="./assets/lens_preview.png" style="width: 80%; height: auto;"/>

<br/>



##  kubernetes 기능

###  구성  

<br/>

클러스터란 ?
Master/Worker Node란 

<br/>

### 네임스페이스

<br/>

POD란 ?
Replica 란 ? 

<br/>


### 워크로드

<br/>

POD란 ?
Replica 란 ? 

<br/>

### 네트워크

<br/>

Service 란 ?

<br/>

### Storage

<br/>

PV 란 ?
PVC 란 ? 

<br/>


### Access Control

<br/>

Service Account 란 ?
Role/RoleBinding 란 ? 

<br/>

* 과제 1 : 미정
