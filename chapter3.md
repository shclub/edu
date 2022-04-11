
# Chapter 3 
   

kubernetes 는 light하고 빠른 설치가 가능한 Rancher에서 제공하는 k3s로 진행을 하고 kubernetes 배포 관리자인 helm 에 대한 사용법을 습득한다.

1. k3s 설치 및 기능 설명

2. helm 설치 및 helm 으로 prometheus 설치 실습

3. kubernetes IDE Lens 설치 및 사용법 실습

참고 : https://subicura.com/k8s/
 
<br/>

##  kubernetes

### 쿠버네티스 시작하기

개발 환경에서 당연하게 사용해왔던 쿠버네티스에 대해 이해하고, 왜 쿠버네티스를 사용하는지 알아보자.  

<br/>

개요
- 컨테이너 오케스트레이션의 개념과 사용하는 이유 그리고 특징에 대해 이해한다.

소개
- 쿠버네티스는 컨테이너 오케스트레이션 툴의 한 종류이며 엄청난 인기로 사실상 표준으로 사용된다.
- 컨테이너 오케스트레이션은 복잡한 컨테이너 환경을 효과적으로 관리하기 위한 도구이다.  

배경  
- 서버의 상태를 편하게 관리하기 위한 노력으로 도커 컨테이너가 등장했다.
- 그러나 관리하는 서버 컨테이너 수가 점점 증가하며 관리가 힘들다는 문제가 생겼다.
- 그래서 등장한 것이 컨테이너 오케스트레이션이다.  

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

왜 쿠버네티스인가  

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

### k3s를 설치 한다.

<br/>

k3s는 가벼운 Kubernetes로 Rancher에서 개발되었으며 쉬운 설치로 적은 메모리/binary 파일을 사용하여 Edge/IoT 환경 혹은 CI/Dev 환경에서 k8s를 쉽게 사용할 수 있도록 도와주는 도구이다.   

심지어 라즈베리파이 에서도 잘 작동한다.

   
터미널로 VM에 로그인 한다.


```bash
ssh root@(본인 VM 공인 ip) -p 22222
``` 

root 계정으로 진행시 아래와 같이 입력하고 패키지를 업그레이드 한다.    
멈추어 있는 듯 하면 Enter를 입력한다.  
일반 계정인 경우는 앞에 sudo를 붙인다. 

```bash
apt update & apt upgrade
``` 

<img src="./assets/login_apt_update.png" style="width: 60%; height: auto;"/>  

k3s를 설치한다. 몇 초 안에 설치가 된다.  
- kubernetes full version 은 1시간 정도, Openshift는 2시간 정도 설치 시간 소요

<br/>
--tls-san 다음의 IP는 본인의 VM Public IP를 입력한다.

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--tls-san (본인 VM Public IP)" sh -s -
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

서비스가 정상으로 running 및 Completed 되는지 아래 명령어를 통해서 확인한다.  

```bash  
# Kubernetes 시스템 Pod 상태 확인
kubectl get pod -n kube-system
```  

<img src="./assets/k3s_nodes.png" style="width: 60%; height: auto;"/>  

k3s는 metric-server가 설치 되어 있어 노드의 리소스를 확인 할 수 있다.

```bash
kubectl top nodes
```  
 
<img src="./assets/top_nodes.png" style="width: 80%; height: auto;"/>     


<br/>


### <a name='Kubernetes'></a>로컬 컴퓨터에서 원격지 Kubernetes 클러스터 접속 위한 설정 

<br/>

k3s를 설치하면, 클러스터의 인증서와 사용자 비밀번호 등 인증하는데 필요한 정보가 /etc/rancher/k3s/k3s.yaml에 저장됩니다.    

외부에서 Kubernetes API server 에 접속하기 위해서 token을 사용한다고 이해 하면 된다.  


```bash
cat /etc/rancher/k3s/k3s.yaml
```  

<img src="./assets/cat_k3s_yaml.png" style="width: 60%; height: auto;"/>  

이 파일을  복사 하여 로컬 컴퓨터에 복사해오면 됩니다.    

서버 주소가 127.0.0.1로 되어있거나, 이름의 대부분이 default로 되어 있습니다. 하나의 Kubernetes 클러스터만 관리한다면 이름은 문제가 되지 않겠지만, 여러 개의 클러스터에 하나의 컴퓨터에서 접속한다면 이름을 반드시 바꿔주어야 합니다.

2개의 값을 일괄 변경 한다.  

- default ->  k3s-test ( 총 6개 ) 
- ip는  127.0.0.1 ->  본인 VM서버 공인 ip

<br/>
vi 에디터에서 esc 키를 클릭한 후 아래 와 같이 사용하면 일괄 변경 가능하다.  

```bash
:%s/127.0.0.1/(본인 VM Public IP)/g
:%s/default/(본인 생성하고 싶은 k3s 이름)/g
```  

```bash
:%s/127.0.0.1/210.106.105.76/g
:%s/default/k3s-test/g
```

<br/>

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
.kube 폴더가 없으면 생성한다. ( config 화일은 확장자가 없이 만든다 )  

- Windows : c:\Users\본인계정\\.kube\config-k3stest
- Mac : ~/.kube/config-k3stest


설치시에 INSTALL_K3S_EXEC="--tls-san 구문을 추가했다면 아래 과정은 skip.  

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

k3s 삭제방법  
 
```bash
/usr/local/bin/k3s-uninstall.sh
```

<br/>

### <a name='kubernetesIDElens'></a>kubernetes IDE lens 설치

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

현재 Disk 상태를 보면 사용률이 높을것을 확인 할 수 있다.    

<img src="./assets/k3s_disk.png" style="width: 80%; height: auto;"/>  

<br/>

도커 루트 디렉토리를 확인하면 /var/lib/docker 폴더를 확인 할 수 있다.  

```bash
docker info | grep "Docker Root Dir"
```  

<img src="./assets/docker_root_dir.png" style="width: 60%; height: auto;"/>  

<br/>

루트 폴더에서 디스크 사용량을 조회해 보면 사용량이 적은것 을 볼수 있다.  

```bash
du -h --max-depth=1
```  

<img src="./assets/root_du_h.png" style="width: 60%; height: auto;"/>  

<br/>

도커 루트 디렉토리인 /var/lib/docker 폴더로 이동하여 디스크 사용량을 조회해 본다.   
overlay2 폴더가 3.2기가로 대부분의 용량을 사용하는 것을 확인 할 수 있다.  


```bash
cd /var/lib/docker
du -h --max-depth=1
```  

<img src="./assets/du_h_overlay.png" style="width: 60%; height: auto;"/>    

<br/>

아래 명령어를 사용하면 도커 이미지와 컨테이너에서 사용하는 디스크 사이즈를 볼 수 있다.  
도커 이미지가 상당히 많은 용량을 차지하고 있음을 확인 할 수 있다.  

```bash
docker system df -v
```  

<img src="./assets/docker_system_df_v.png" style="width: 60%; height: auto;"/> 

<br/>

Docker image와 container 용량이 큰 경우 아래 명령어를 사용하여 사용하지 않는 컨테이너와 이미지 정보를 정리할 수 있다.  

```bash
docker image prune -all
```    

<br/>
하나씩 확인하고 삭제하려고 하면 아래 명령어를 사용한다.

```bash
docker rmi 이미지이름 또는 image id
```   

<br/>

### <a name='Helmhttps:helm.shkodocsintroinstall'></a>Helm 설치 ( https://helm.sh/ko/docs/intro/install/ )

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
helm install prometheus -n monitoring prometheus-community/kube-prometheus-stack
```  

설치시 아래와 같은  에러가 발생 하면 

```bash
Error: INSTALLATION FAILED: Kubernetes cluster unreachable: Get "http://localhost:8080/version": dial tcp [::1]:8080: connect: connection refused
```

다음 명렁어를 실행하고 다시 prometheus를 설치 한다. 위 명령어 실행.

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

정상적으로 실행이 되면 3개의 pod 가 생성된 것을 확인 할 수 있다.

```bash
kubectl get deployments -n monitoring
```  
 
<img src="./assets/helm_prometheus_install.png" style="width: 80%; height: auto;"/>     

<br/>

k8s IDE인 lens 를 확인하면 메트릭 정보를 볼수 있고 
명령어를 통해서도 확인할 수 있다.

<br/>


### 과제  

과제 1 : oralce cloud에 가입하고 cloud shell 과 vm을 생성하고 k3s를 설치해 본다.    
- 문서를 참고한다. [oracle cloud 가이드 ](./oracle_cloud_usage.md.md)

