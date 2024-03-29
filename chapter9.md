# Chapter 9 
 
Redhat Openshift 의 오픈소스 버전인 OKD 4.7 설정하는 방법을 알아본다.   

<br/>

1. OKD Cluster

2. OKD 에  ArgoCD 설치

3. OKD 에  Elastic Stack  설치

<br/>

## OKD Cluster   

<br>

kubernetes 시스템 구분

<br/>

|이름| 설명 |
|:--| :-------|  
| kubernetes (k8s)	| 일반적인 오픈소스 kubernetes |
| Openshift |	Redhat에서 판매하는 kubernetes 를 위한  PaaS |
| OKD	| Openshift 의 오픈 소스 버전 |
| OKD 3.1 | k8s 1.1 버전 사용|
| OKD 4.7 | k8s 1.2 버전 사용 |
| FlyingCube 1.0 | KTDS에서 개발한 OKD 3.1 기반의 컨테이너 플랫폼 |
| FlyingCube 2.0 | KTDS에서 개발한 OKD 4.7 기반의 컨테이너 플랫폼 |
| k3s | Rancher에서 제공하는 최소사양으로 설치 가능한 경량 kubernetes |
| microK8S | 많은 기능을 한번에 설치할수 있는 경량 kubernetes |  


<br/><br/>

> Openshift 소개 : https://youtu.be/ft9b4bTRtoI   
> OKD4 소개 : https://velog.io/@_gyullbb/OKD-개요-2

<br/><br/>

### Cluster 생성 


<br/>

kt cloud 에 로그인 하여 D1 존 으로 이동한다.  

클러스터 생성시 각 노드에 로그인 하기 위해서는 SSH Keypair를 먼저 생성해야한다. ( 클러스터 구성 완료 후 추가는  불가 )  

서버 -> SSH Key pair 로 이동하여 create 버튼 클릭하여 이름 입력한 후 launch 아이콘 클릭하여 생성.   

<img src="./assets/ktcloud_keypair.png" style="width: 80%; height: auto;"/>    

<br/>

키는 바로 생성이 된다. ( edu라는 이름으로 기 생성된 값 사용 )

<img src="./assets/ktcloud_keypair_created.png" style="width: 80%; height: auto;"/>  

<br/>

download 아이콘을 클릭하여 pem 화일을 로컬 pc에 다운 받는다.  

<img src="./assets/ktcloud_keypair_download.png" style="width: 40%; height: auto;"/>  

<br/>

K2P 상품을 클릭하여 Cluster를 선택한다.  

<img src="./assets/kt_cloud_d1.png" style="width: 80%; height: auto;"/>  

<br/>

OKD 클러스터 생성을 위한 기본 정보를 입력하다.  

이름과 DMZ Tier/Private Tier는 아래와 같이 설정하고 다음 버튼 클릭.  

<img src="./assets/okd_create1.png" style="width: 80%; height: auto;"/>  

<br/>

OKD 4.x 버전 부터는 Centos 대신 Fedora CoreOS를 사용한다.  

worker node를 생성하기 위해 앞에서 생성한 key pair를 선택하고  

private tier에서 5개 노드와 cpu/mem를 선택하고 다음 버튼 클릭.  
  
<img src="./assets/okd_create2.png" style="width: 80%; height: auto;"/>  

<br/> 

OKD 내부적으로 사용하기 위해 NFS 볼륨을 할당한다.  1000 GB 입력.  

<img src="./assets/okd_create3.png" style="width: 80%; height: auto;"/>  

<br/>

정보를 최종 확인하고 생성하기를 클릭한다.  

<img src="./assets/okd_create4.png" style="width: 80%; height: auto;"/>  

<br/>

상당히 오랜 시간이 걸리고 정상적으로 완료 되면 아래와 같이 확인 할 수 있다.  

<img src="./assets/okd_create5.png" style="width: 80%; height: auto;"/>  


<br/>

### Cluster 접속

<br/>

Cluster 접속을 하기 위해서 Cluster 정보를 확인한다.  

조회 할 Cluster를 체크하고 ... 을 클릭하여 클러스터 정보를 확인한다.  

<img src="./assets/cluster_select.png" style="width: 80%; height: auto;"/>  

<br/>

web console를 확인 할 수 있고 우리가 cli 접속하기 위해서는 API URL를 확인해야 한다.  

<img src="./assets/okd_cluster_info.png" style="width: 80%; height: auto;"/>  

<br>

접속을 하기 위한 Client 설치는 아래 페이지를 참고한다.  

- 가이드 : https://cloud.kt.com/portal/user-guide/Container-container-guide

<br/>

우리는 mac를 기준으로 설명한다.  

로그인 을 위해서는 openshift console 인 oc 명령어를 사용하며 kubectl 명령어와 거의 동일하다.  먼저 root로  로그인을 한다. ( 비밀번호는 확인 필요)   

<br/>

```bash
jakelee@jake-MacBookAir ~ % oc login https://api.211-34-231-81.nip.io:6443 -u root -p <패스워드> --insecure-skip-tls-verify
```  

정상적으로 로그인이 되면 아래와 같이 특정 project 로 접속이 된다.  
-  k8s의 namespace와 openshift의 project는 같다.  

```bash
Login successful.

You have access to 104 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
```


<br/>

### Cluster 환경 구성

<br/>  

cluster 권한 ( root ) 으로 구성이 필요하며 우리가 구성할 내용은 아래와 같다.    

- 워커 노드 edu.worker02 ~ 07 : 교육용 namespace 배치 ( edu로 시작 )
- 워커 노드 edu.worker01 : Jenkins 및 기타 솔루션 설치. ( 교육용 namespace 배치 불가 설정 )  

<br/>

먼저 master 노드 설정을 확인하기에 앞서  

일반 유저는 node를 볼 수 있는 권한이 없어 ClusterRole을 생성하고 각 사용자에게 RoleBinding을 하여 권한을 부여한다.  

vi 에디터로 아래 내용을 복사하여 node_view_role.yaml을 생성한다.   

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-view-role
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs:
  - get
  - list
  - watch
```   

<br/>

ClusterRole 생성한다. ( Cluster Admin 이 수행 )

```bash
root@newedu:~# kubectl apply -f node_view_role.yaml
clusterrole.rbac.authorization.k8s.io/node-view-role created  
```  

<br/>

ClusterRoleBinding 을 생성하면서 user를 명시한다.   
service account 에 권한을 주기 위해서는 user 대신 serviceaccount 를 설정한다.  

```bash
root@newedu:~# kubectl create clusterrolebinding node-view-rolebinding1 --clusterrole=node-view-role --user=edu1-admin
```  

<br/>

여기서는 교육생의 권한을 일괄 생성 하기 위해서 아래 script를 만들어서 사용 한다.  
- 참고 : https://github.com/shclub/edu14-2  


node_view_rolebinding.sh
```bash
#!/bin/bash
x=1
while [ $x -le 35 ]
do
  namespace="edu${x}"
  echo $namespace
  #namespace+="${x}"
  k_exec=`kubectl create clusterrolebinding node-view-rolebinding${x} --clusterrole=node-view-role --user=${namespace}-admin`
  echo $k_exec
  sleep 1
  x=$(( $x + 1 ))
done
```  

<br/>


```bash
jakelee@jake-MacBookAir ~ % kubectl get nodes -o wide
NAME               STATUS   ROLES    AGE    VERSION                INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                        KERNEL-VERSION            CONTAINER-RUNTIME
edu.dmz-infra01    Ready    worker   35d    v1.20.0+bafe72f-1054   172.25.0.93    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.dmz-infra02    Ready    worker   35d    v1.20.0+bafe72f-1054   172.25.0.87    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.master01       Ready    master   35d    v1.20.0+bafe72f-1054   172.25.1.16    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.master02       Ready    master   35d    v1.20.0+bafe72f-1054   172.25.1.179   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.master03       Ready    master   35d    v1.20.0+bafe72f-1054   172.25.1.13    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.monitoring01   Ready    worker   35d    v1.20.0+bafe72f-1054   172.25.1.28    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.monitoring02   Ready    worker   35d    v1.20.0+bafe72f-1054   172.25.1.129   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker01       Ready    worker   35d    v1.20.0+bafe72f-1054   172.25.1.144   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker02       Ready    worker   35d    v1.20.0+bafe72f-1054   172.25.1.50    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker03       Ready    worker   35d    v1.20.0+bafe72f-1054   172.25.1.124   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker04       Ready    worker   35d    v1.20.0+bafe72f-1054   172.25.1.160   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker05       Ready    worker   5d1h   v1.20.0+bafe72f-1054   172.25.1.59    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
```  

<br/>

master node 는 홀 수개로 구성이 되며 kt cloud 에서는 3개의 master node가 구성이 되며 우리는 첫번째 마스터 노드만 설정을 확인한다.  

```bash
jakelee@jake-MacBookAir ~ % kubectl describe node edu.master01
Name:               edu.master01
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=edu.master01
                    kubernetes.io/os=linux
                    master=true
                    node-role.kubernetes.io/master=
                    node.openshift.io/os_id=fedora
Annotations:        k8s.ovn.org/l3-gateway-config:
                      {"default":{"mode":"shared","interface-id":"br-ex_edu.master01","mac-address":"fa:16:3e:fc:a1:79","ip-addresses":["172.25.1.16/24"],"ip-ad...
                    k8s.ovn.org/node-chassis-id: 31fffb8e-2b17-4fa2-a6b6-9dfd3f8daa6b
                    k8s.ovn.org/node-local-nat-ip: {"default":["169.254.10.67"]}
                    k8s.ovn.org/node-mgmt-port-mac-address: ca:ac:59:1f:f9:d5
                    k8s.ovn.org/node-primary-ifaddr: {"ipv4":"172.25.1.16/24"}
                    k8s.ovn.org/node-subnets: {"default":"10.128.0.0/23"}
                    machineconfiguration.openshift.io/currentConfig: rendered-master-00c32c45995484daeb7d5a01e18f239b
                    machineconfiguration.openshift.io/desiredConfig: rendered-master-00c32c45995484daeb7d5a01e18f239b
                    machineconfiguration.openshift.io/reason:
                    machineconfiguration.openshift.io/state: Done
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 20 Jun 2022 16:16:57 +0900
Taints:             node-role.kubernetes.io/master:NoSchedule
```  

여기서 확인 할 내용은 Taints 이다.   

현재 `node-role.kubernetes.io/master:NoSchedule` 로 설정 되어 있으며 이 의미는 일반 pod는 schedule 이 불가.  

k8s 나 okd 에서 관리하는 pod는 schedule 가능 ( toleration 설정 )

```bash
jakelee@jake-MacBookAir ~ % kubectl get po -n openshift-apiserver  -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
apiserver-77894b645b-hh5kz   2/2     Running   0          35d   10.130.0.57   edu.master02   <none>           <none>
apiserver-77894b645b-jzs5s   2/2     Running   0          35d   10.128.0.57   edu.master01   <none>           <none>
apiserver-77894b645b-n8h7j   2/2     Running   0          35d   10.129.0.51   edu.master03   <none>           <none>
```  

<br/>

taint를 설정한 노드에는 포드들이 스케쥴링 되지 않습니다.   
taint가 걸린 노드에 포드들을 스케쥴링 하려면 toleration을 이용해서 지정해 주어야합니다. taint는 cordon이나 draint처럼 모든 포드가 스케쥴링 되지 않게 막는건 아니고, toleration을 이용한 특정 포드들만 실행하게 하고 다른 포드들은 들어오지 못하게 하는 역할을 합니다.   

주로 노드를 지정된 역할만 하게할 때 사용합니다. DB용 포드를 띄워서 노드 전체의 CPU나 RAM자원을 독점해서 사용하게 할 수 있습니다. GPU가 있는 노드에는 다른 포드들은 실행되지 않고, 실제로 GPU를 사용하는 포드들만 실행시키도록 설정할 수도 있습니다.   

- 출처: https://arisu1000.tistory.com/27846

<br/>  

먼저 각 worker node 에 label을 설정한다.    

- edu.worker01~04 ,07~08
   - edu: "true"
- edu.worker05 
   - devops: "true"

<br/>

worker node 1번 부터 node edit를 한다.  

```bash  
jakelee@jake-MacBookAir ~ % kubectl edit node edu.worker01
```  

label 에  edu: "true" 추가한다.  

```bash  
apiVersion: v1
kind: Node
metadata:
  annotations:
    k8s.ovn.org/l3-gateway-config: '{"default":{"mode":"shared","interface-id":"br-ex_edu.worker01","mac-address":"fa:16:3e:1b:fd:06","ip-addresses":["172.25.1.144/24"],"ip-address":"172.25.1.144/24","next-hops":["172.25.1.1"],"next-hop":"172.25.1.1","node-port-enable":"true","vlan-id":"0"}}'
    k8s.ovn.org/node-chassis-id: a5f66e69-de24-4778-8be5-193e3f2861d6
    k8s.ovn.org/node-local-nat-ip: '{"default":["169.254.2.83"]}'
    k8s.ovn.org/node-mgmt-port-mac-address: 92:a9:80:92:18:54
    k8s.ovn.org/node-primary-ifaddr: '{"ipv4":"172.25.1.144/24"}'
    k8s.ovn.org/node-subnets: '{"default":"10.128.2.0/23"}'
    machineconfiguration.openshift.io/currentConfig: rendered-worker-ae2b87f82febc73d3e758b35f5ee14a0
    machineconfiguration.openshift.io/desiredConfig: rendered-worker-ae2b87f82febc73d3e758b35f5ee14a0
    machineconfiguration.openshift.io/reason: ""
    machineconfiguration.openshift.io/state: Done
    volumes.kubernetes.io/controller-managed-attach-detach: "true"
  creationTimestamp: "2022-06-20T08:15:59Z"
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    edu: "true"
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: edu.worker01
    kubernetes.io/os: linux
    node-role.kubernetes.io/worker: ""
    node.openshift.io/os_id: fedora
    worker: "true"
  name: edu.worker01
```  

<br/>

worker node 4번, 그리고 6 ~ 7번 까지 같은 방식으로 Label를 추가한다.  

worker node 5번은 아래와 같이 추가한다.  

```bash  
jakelee@jake-MacBookAir ~ % kubectl edit node edu.worker05
```    

<img src="./assets/node5_label_edit.png" style="width: 80%; height: auto;"/>


<br/>

node 에 label 할당이 완료 되면 namespace 에 Annotation을 할당한다.  

먼저 namespace ( project )를 할당한다.  

```bash  
jakelee@jake-MacBookAir ~ % oc new-project devops --display-name 'devops'


Now using project "devops" on server "https://api.211-34-231-81.nip.io:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/e2e-test-images/agnhost:2.33 -- /agnhost serve-hostname
```  

devops namespace 가 생성 된 후  수정을 한다.  

```bash  
jakelee@jake-MacBookAir ~ % kubectl edit namespace devops
```  

annotations 에 아래와 같이 값을 추가한다.  
아래  devops=true 라는 label을 가진 node 에 pod 가 생성 되라는 의미이다.   

`openshift.io/node-selector: devops=true`  

<br/>
 
```bash   
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    openshift.io/description: ""
    openshift.io/display-name: devops
    openshift.io/node-selector: devops=true
    openshift.io/sa.scc.mcs: s0:c26,c10
    openshift.io/sa.scc.supplemental-groups: 1000670000/10000
    openshift.io/sa.scc.uid-range: 1000670000/10000
  creationTimestamp: "2022-07-26T03:44:31Z"
  name: devops
  resourceVersion: "18824366"
  selfLink: /api/v1/namespaces/devops
  uid: a45c81a0-3fbd-4c29-af67-1344836b3b2f
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```  

<br/>

edu로 시작하는 namespace에는  `openshift.io/node-selector: edu=true` 로 설정한다.  ( worker node 1 ~ 4  에서만 pod 생성 )  


- 참고 : https://access.redhat.com/documentation/ko-kr/openshift_container_platform/4.6/html/nodes/nodes-scheduler-node-selectors


<br/>

oc 명령어로 namespace 를 생성하면 3개의 Service Account 가 생성된다.  

```bash
jakelee@jake-MacBookAir ~ % kubectl get sa -n devops
NAME       SECRETS   AGE
builder    2         136m
default    2         136m
deployer   2         136m
```    

기존적으로 namespace명-admin 계정이 생성되면 비밀번호는 New1234! 로 설정됨. ( kt cloud 에만 해당 )  

<br/>

해당 계정으로 로그인 하기 전에 Anyuid 권한을 설정한다.    

Openshift 에서는 Service Account를 통해 권한 변경이 가능
기본적인 Openshift default serviceaccount 는 "restrict" 으로, 일반 권한의 UID로 컨테이너가 실행.  

root 권한이 필요한 경우 권한을 변경하여 사용


`oc adm policy add-scc-to-user anyuid system:serviceaccount:<NAMESPACE>:default`  

<br/>

devops namespace에 SCC 추가. 

```bash
jakelee@jake-MacBookAir ~ % oc adm policy add-scc-to-user anyuid system:serviceaccount:devops:default

clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "default"
```  

- 참고 : http://wiki.rockplace.co.kr/display/OP/01.+Service+Accounts+and+SCC

<br/>

자 이제 devops-admin 유저로 로그인을 해보자.  

```bash  
jakelee@jake-MacBookAir ~ % oc login https://api.211-34-231-81.nip.io:6443 -u devops-admin -p New1234! --insecure-skip-tls-verify

Login successful.

You have one project on this server: "devops"
```  

<br/>

로그인을 하면 devops namespace 로 설정이 되고 권한은 devops namespace로 제한이 된다.  
- 비밀번호 변경은 Cluster 권한 만 가능.

nginx deployment 를 배포해 보자.  


```bash  
jakelee@jake-MacBookAir ~ % kubectl apply -f https://k8s.io/examples/application/deployment.yaml
deployment.apps/nginx-deployment created
jakelee@jake-MacBookAir ~ % kubectl get po -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
nginx-deployment-66b6c48dd5-2khlh   1/1     Running   0          20s   10.129.6.10   edu.worker05   <none>           <none>
```  

5번 워커 노드인 edu.worker05 에서 동작 하는 것을 확인 할 수 있다.  

<br/>

### Cluster GUI ( Container portal )

<br/>

kt cloud 웹 콘솔에서 K2P -> Container 로 이동하면 생성한 OKD 클러스터를 볼수 있다.  

해당 클러스터를 선택을 하고 Container 콘솔을 클릭한다.  


<img src="./assets/okd_container_console.png" style="width: 80%; height: auto;"/>    

<br/>

아래 화면이 보이게 되는데 해당 콘솔은 Flying Cube 2.0 웹 콘솔이다.  


<img src="./assets/okd_container_console_overview.png" style="width: 80%; height: auto;"/>

<br/>

Cluster 에서 ... 를 클릭하면 클러스터 관리와 프로젝트 관리 메뉴가 나오고 클러스터 관리는 전체 클러스터의 내용을 볼수 있고 프로젝트는 Namespace 생성 및 권한을 할당 할 수 있다.  

<img src="./assets/okd_console1.png" style="width: 40%; height: auto;"/>

<br/>

클러스터 관리 메뉴를 선택하고 Cluster 이름인 edu를 클릭하고 들어간다.  

<img src="./assets/okd_console2.png" style="width: 60%; height: auto;"/>  

왼쪽에 여러가지 메뉴가 보인다.  

<img src="./assets/okd_console3.png" style="width: 80%; height: auto;"/>  

<br/>

Node 관리를 클릭하면 전체 노드 리스트를 볼수 있다.  

<img src="./assets/okd_console4.png" style="width: 80%; height: auto;"/>  

5번 워커 노드의 맨 오른쪽에 Label에 마우스를 올리면 우리가 설정한 `devops = true` 레이블을 볼수 있다.    

<img src="./assets/okd_console5.png" style="width: 80%; height: auto;"/>  

<br/>

처음 화면으로 돌아와서 프로젝트 관리 메뉴를 클릭하면 앞장에서 생성한 devops 프로젝트를 볼 수 있다.  

<img src="./assets/okd_console6.png" style="width: 80%; height: auto;"/>  

<br/>

웹 GUI 에서도 New Project 메뉴를 클릭하여 프로젝트를 생성 할 수 있다.    

<img src="./assets/okd_console7.png" style="width: 80%; height: auto;"/>  

<br/>

devops 프로젝트를 클릭하면 Project 정보를 볼수 있고 왼쪽 프레임에 Project Infomation 으로 이동하면 앞에서 설정한 Annotaion을 확인 할 수 있다.     

<img src="./assets/okd_console8.png" style="width: 80%; height: auto;"/>  

<br/>

Openshift User 메뉴에서는 해당 계정의 비밀번호를 확인 할 수 있고 도구에 있는 아이콘을 클릭하면 비밀번호를 변경 할 수 있다.  

<img src="./assets/okd_console9.png" style="width: 80%; height: auto;"/>  

<br/>

###  Worker Node 접속 

<br/>

worker node에 접속하기 위해서는 public ip를 할당 하고 port forwarding 과 방화벽을 오픈 해야한다.  

kt cloud 콘솔에서 Server -> Networking 으로 이동하여 IP를 하나 생성한다.

<img src="./assets/okd_network1.png" style="width: 80%; height: auto;"/>    

해당 IP를 선택 한 후 접속 설정을 클릭하여  포트 포워딩 설정을 한다.  
- 서버 : edu.worker5
- 내부 포트 : 22
- 외부 포트 : 22222

<img src="./assets/okd_network2.png" style="width: 80%; height: auto;"/>    

포트 포워딩 설정후 해당 IP를 선택 한 후 방화벽 설정을 클릭하여 방화벽 설정을 한다.  
- Action : Allow
- Source Network : external
- Source CIDR : all
- Destination Network : Private_Sub
- Destination CIDR : PF_211.34.231.85_22222_TCP

<img src="./assets/okd_network3.png" style="width: 80%; height: auto;"/>    

<br/>

포탈에서 생성한 edu.pem private key 로 public key를 생성한다. 

<br/>

```bash  
ssh-keygen -f edu.pem -y > edu.pub
```  

<br/>

oc login 을 하고  machine config 를 조회를 하면 우리가 수정할  컨트롤러 이름 2개가 보인다.   

- 99-master-ssh 
- 99-worker-ssh

```bash  
jakelee@jake-MacBookAir Downloads % oc get mc
NAME                                               GENERATEDBYCONTROLLER                      IGNITIONVERSION   AGE
00-master                                          664fff942096fc9f357e81776345b3b71000a8a7   3.2.0             24h
00-worker                                          664fff942096fc9f357e81776345b3b71000a8a7   3.2.0             24h
01-master-container-runtime                        664fff942096fc9f357e81776345b3b71000a8a7   3.2.0             24h
01-master-kubelet                                  664fff942096fc9f357e81776345b3b71000a8a7   3.2.0             24h
01-worker-container-runtime                        664fff942096fc9f357e81776345b3b71000a8a7   3.2.0             24h
01-worker-kubelet                                  664fff942096fc9f357e81776345b3b71000a8a7   3.2.0             24h
99-master-generated-registries                     664fff942096fc9f357e81776345b3b71000a8a7   3.2.0             24h
99-master-okd-extensions                                                                      3.1.0             24h
99-master-ssh                                                                                 3.2.0             24h
99-okd-master-disable-mitigations                                                             3.1.0             24h
99-okd-worker-disable-mitigations                                                             3.1.0             24h
99-worker-generated-registries                     664fff942096fc9f357e81776345b3b71000a8a7   3.2.0             24h
99-worker-okd-extensions                                                                      3.1.0             24h
99-worker-ssh                                                                                 3.2.0             24h
``` 


<br/>

참고 : https://cloud.kt.com/portal/user-guide/Container-k2p-enterprise-howto  

<br/>

99-master-ssh 화일에서   
사전작업 시 생성했던 public key를 기존 machineconfig의 내용 중 “sshAuthorizedKeys” 하위에 추가하고 저장한다.  

```bash  
oc edit mc 99-master-ssh
```

<br/>

99-worker-ssh 도 같이 수정을 하고 아래 명령어로 상태를 모니터링한다.
하나의 노드당 3분 정도 걸린다.     
- MachineConfigPool이 돌면서 Updating 상태가 보여지고 “UpdatedMachineCount” 대수를 확인하여 설정 적용 현황을 확인해본다. ( 반드시 모든 Node가 살아 있어야 함 )

<br/>

```bash
jakelee@jake-MacBookAir Downloads % oc get mcp
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master   rendered-master-c4c6115ed2eebe3d16f36062098881d0   True      False      False      3              3                   3                     0                      24h
worker   rendered-worker-ac8bcafb561a91591dccb86b615cba3a   False     True       False      7              0                   0                     0                      24h
```  


<br/>

로컬 PC의 terminal 에서 다운 받은 SSH Key pair를 이용하여 접속을 한다.  
- 먼저 edu.pem 화일의 권한을 600 으로 변경한다.
- coreos의 사용자는 core 이다.

<br/>

```bash  
jakelee@jake-MacBookAir % chmod 600 edu.pem
jakelee@jake-MacBookAir % ssh -i edu.pem core@211.34.231.85 -p 22222
Fedora CoreOS 33.20210301.3.1
Tracker: https://github.com/coreos/fedora-coreos-tracker
Discuss: https://discussion.fedoraproject.org/c/server/coreos/

Last login: Tue Jul 26 02:47:09 2022 from 220.120.16.10
[systemd]
Failed Units: 1
  systemd-resolved.service
```  

<br/>

정상적으로 접속이 되면 core 계정과 root 계정의 비밀번호를 설정한다.  

```bash
[core@edu ~]$ sudo passwd core
Changing password for user core.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[core@edu ~]$ sudo passwd root
Changing password for user root.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```  

<br/>

비밀번호로 접속하기 위해 root로 사용자를 전환하고 sshd config 폴더로 이동한다.  

```bash
[core@edu ~]$ su -
Password:
Last login: Wed Apr 26 07:34:06 UTC 2023 on pts/0
[systemd]
Failed Units: 1
  systemd-resolved.service
[root@edu ssh]# cd /etc/ssh/sshd_config.d
[root@edu sshd_config.d]# ls
10-disable-ssh-key-dir.conf    40-ssh-key-dir.conf
10-insecure-rsa-keysig.conf  40-disable-passwords.conf  50-redhat.conf
```  

<br/>

`20-enable-passwords.conf` 라는 화일을 생성하고 안에 `PasswordAuthentication yes` 를 추가하고 저장한다.

```bash
[root@edu sshd_config.d]# vi 20-enable-passwords.conf
```  

<br/>

sshd 데몬을 다시 기동한다.  

```bash
[root@edu sshd_config.d]# systemctl restart sshd
```  

<br/>

worker node에 접속 가능하다.  

```bash
jakelee@jake-MacBookAir ~ % ssh root@211.34.231.85 -p 22222
root@211.34.231.85's password:
Fedora CoreOS 33.20210301.3.1
Tracker: https://github.com/coreos/fedora-coreos-tracker
Discuss: https://discussion.fedoraproject.org/c/server/coreos/

Last login: Tue Jul 26 02:51:06 2022 from 220.120.16.10
[systemd]
Failed Units: 1
  systemd-resolved.service
[root@edu ~]#
```  

<br/>


###  OKD Cluster 백업 방법  ( 수동 )

<br/>

OKD 백업을 위해서는 Master Node에 접속하여 아래 명령어를 root 로 실행한다.      


```bash
[root@edu etc]# /usr/local/bin/cluster-backup.sh /home/core/assets/backup
b716e30ba5c7bb28eee74a0bdf84eb0b0c8957ce5d485e15a541389e06cec796
etcdctl version: 3.4.9
API version: 3.4
found latest kube-apiserver: /etc/kubernetes/static-pod-resources/kube-apiserver-pod-10
found latest kube-controller-manager: /etc/kubernetes/static-pod-resources/kube-controller-manager-pod-9
found latest kube-scheduler: /etc/kubernetes/static-pod-resources/kube-scheduler-pod-8
found latest etcd: /etc/kubernetes/static-pod-resources/etcd-pod-3
{"level":"info","ts":1682644705.1011016,"caller":"snapshot/v3_snapshot.go:119","msg":"created temporary db file","path":"/home/core/assets/backup/snapshot_2023-04-28_011823.db.part"}
{"level":"info","ts":"2023-04-28T01:18:25.109Z","caller":"clientv3/maintenance.go:200","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1682644705.1092677,"caller":"snapshot/v3_snapshot.go:127","msg":"fetching snapshot","endpoint":"https://172.25.1.31:2379"}
{"level":"info","ts":"2023-04-28T01:18:25.663Z","caller":"clientv3/maintenance.go:208","msg":"completed snapshot read; closing"}
{"level":"info","ts":1682644705.7459,"caller":"snapshot/v3_snapshot.go:142","msg":"fetched snapshot","endpoint":"https://172.25.1.31:2379","size":"92 MB","took":0.644741922}
{"level":"info","ts":1682644705.746118,"caller":"snapshot/v3_snapshot.go:152","msg":"saved","path":"/home/core/assets/backup/snapshot_2023-04-28_011823.db"}
Snapshot saved at /home/core/assets/backup/snapshot_2023-04-28_011823.db
snapshot db and kube resources are successfully saved to /home/core/assets/backup
```

<br/>

아래 처럼 snapshot db 와 static pod yaml 화일이 백업 됩니다.  

```bash
[root@edu etc]# ls /home/core/assets/backup
snapshot_2023-04-28_010032.db  static_kuberesources_2023-04-28_010032.tar.gz
```    

<br/>

host 에 백업하는 것 보다는 NAS에 mount 하여  저장하는 것이 좋음.  

```bash
[root@edu etc]# /usr/local/bin/cluster-backup.sh /mnt/cluster_backup
```    

<br/>

<br/>

###  OKD Cluster 백업 방법  ( 자동 : Crontab )

<br/>

참고   
- https://github.com/adfinis/openshift-etcd-backup

<br/>

OKD 자동 백업을 위해서 Cronjob 을 사용한다.        

<br/>

`etcd-backup` namespace 를 신규로 생성한다.     
해당 namespace에는 node-selector를 설정하지 않는다.  


```bash
oc new-project etcd-backup
```

<br/>

backup-config.yaml 화일을 생성한다.  
사전에 nfs 에 cluster_backup 폴더를 생성한다.    

<br/>

`OCP_BACKUP_SUBDIR` 은 nfs 에 신규로 생성되는 폴더 이름 이다.  

<br/>

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: backup-config
data:
  OCP_BACKUP_SUBDIR: "/"
  OCP_BACKUP_DIRNAME: "+etcd-backup-%FT%T%:z"
  OCP_BACKUP_EXPIRE_TYPE: "days"
  OCP_BACKUP_KEEP_DAYS: "30"
  OCP_BACKUP_KEEP_COUNT: "10"
  OCP_BACKUP_UMASK: "0027"
```

<br/>

backup-nfs-cronjob.yaml 화일을 생성한다.
하루 7시 1분에 한번 작동한다.    

<br/>

아래 CronJob은 master node 에 pod 를 생성한다.  
- `runAsUser: 0` : 0은  root로 실행하는 것을 의미.
- `nodeName : edu.master01` : master 1번 노드에서 생성 
- `mountPath : /backup` : master node에서 mount 되는 폴더는 backup 폴더 이다. 변경하지 말것

<br/>

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: etcd-backup
spec:
  schedule: "1 7 * * *"
  startingDeadlineSeconds: 600
  jobTemplate:
    spec:
      backoffLimit: 0
      template:
        spec:
          containers:
          - command:
            - /bin/sh
            - /usr/local/bin/backup.sh
            image: ghcr.io/adfinis/openshift-etcd-backup
            imagePullPolicy: Always
            name: backup-etcd
            resources:
              requests:
                cpu: 500m
                memory: 128Mi
              limits:
                cpu: 1000m
                memory: 512Mi
            envFrom:
            - configMapRef:
                name: backup-config
            securityContext:
              privileged: true
              runAsUser: 0
            volumeMounts:
            - mountPath: /host
              name: host
            - name: etcd-backup
              mountPath: /backup
          nodeSelector:  # node selector 설정 
            node-role.kubernetes.io/master: ""
          tolerations:  # 모든 master node에 pod 스케줄링 
            - effect: NoSchedule # taint 설정과 같은 값 설정
              key: node-role.kubernetes.io/master
          nodeName : edu.master01 # master node 중 하나 지정    
          #hostNetwork: true
          #hostPID: true
          restartPolicy: Never
          dnsPolicy: ClusterFirst
          volumes:
          - hostPath:
              path: /
              type: Directory
            name: host
          - name: etcd-backup
            nfs:
              server: 172.25.1.164
              path: /share_aed398f1_3e3e_4d30_8b59_8185229ebde0/cluster_backup
```

<br/>

master node 1번에 접속하여 `/usr/local/bin/cluster-backup.sh` 파일의 아래 부분을 주석 처리한다.  ( 주석 처리 안하면  API 호출 에러가 발생함 )

<br/>

```bash
[core@edu ]$ sudo vi /usr/local/bin/cluster-backup.sh
```  

<br/>

```bash
     46 #   progressing=$(oc get co "${operator}" -o jsonpath='{.status.conditions[?(@.type=="Progressing")].status}') || true
     47 #   if [ "$progressing" == "" ]; then
     48 #      echo "Could not find the status of the $operator. Check if the API server is running. Pass the --force flag to skip checks."
     49 #      exit 1
     50 #   elif [ "$progressing" != "False" ]; then
     51 #      echo "Currently the $operator operator is progressing. A reliable backup requires that a rollout is not in progress.  Aborting!"
     52 #      exit 1
     53 #   fi
```

<br/>

service account 에  권한을 할당한다.   

```bash
jakelee@jake-MacBook % oc adm policy add-scc-to-user privileged -z default -n etcd-backup
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:privileged added: "default"
```

<br/>

config 를 먼저 적용하고 cronjob을 적용한다.  

```bash
jakelee@jake-MacBook  % kubectl apply -f backup-config.yaml -n etcd-backup
jakelee@jake-MacBook  % kubectl apply -f backup-nfs-cronjob.yaml -n etcd-backup
```  

<br/>

생성된 cronjob을 확인해 본다.  

```bash
jakelee@jake-MacBook  % kubectl get cronjob -n etcd-backup
NAME          SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
etcd-backup   1 7 * * *   False     0        <none>          21m
```  

<br/>

창을 하나 더 띄워서 job을 모니터링 한다. 

```bash
jakelee@jake-MacBook % kubectl get jobs -n etcd-backup --watch
NAME                     COMPLETIONS   DURATION   AGE
etcd-backup-1682666100   1/1           6s         19s
```  

<br/>

정상적으로 cronjob이 수행이 되면 NAS 에 아래와 같이 날짜별로 폴더가 생성이 되고 그 안에 백업 화일이 생성 된 것을 확인 할 수 있다.   

```bash
[core@edu cluster_backup]$ ls -al
total 180200
drwxrwxrwx. 6 nfsnobody nfsnobody     4096 Apr 28 07:58 .
drwxrwxrwx. 8 root      root          4096 Apr 28 07:51 ..
drwxr-x---. 2 root      root          4096 Apr 28 07:55 etcd-backup-2023-04-28T07:55:12+00:00
drwxr-x---. 2 root      root          4096 Apr 28 07:56 etcd-backup-2023-04-28T07:56:12+00:00
drwxr-x---. 2 root      root          4096 Apr 28 07:57 etcd-backup-2023-04-28T07:57:12+00:00
drwxr-x---. 2 root      root          4096 Apr 28 07:58 etcd-backup-2023-04-28T07:58:12+00:00
[core@edu cluster_backup]$ sudo ls etcd-backup-2023-04-28T07:55:12+00:00
snapshot_2023-04-28_075512.db  static_kuberesources_2023-04-28_075512.tar.gz
```  

<br/>

###  OKD Cluster 복구 방법 

<br/>  

master node 1번에 접속하여 아래와 같이 복구한다.  
restore 할 폴더 ( etcd  backup 폴더 ) 에는  화일 하나만  있어야 한다.      

```bash
[root@edu etc]# /usr/local/bin/cluster-restore.sh /home/core/assets/backup
```  

<br/>

###  SSH 키 생성 

<br/>


맥에서 ssh key pair 를 생성한다.  
- 참고 
   - https://jojoldu.tistory.com/442  
   - https://thoonk.tistory.com/81
      
Terminal 에서 `ssh-keygen -t ed25519 -C <github 등록 이메일>` 를 사용하여
public / private 키를 생성 한다.  

```bash
jakelee@jake-MacBookAir ~ % ssh-keygen -t ed25519 -C "shclub@gmail.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/jakelee/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/jakelee/.ssh/id_ed25519
Your public key has been saved in /Users/jakelee/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:SDaUkrBr/qeVijeHNjLNMb6VjISg3m6LgNYMHY2WoAA shclub@gmail.com
The key's randomart image is:
+--[ED25519 256]--+
|E... ...         |
|o ..*..          |
|o .= o+          |
|..oo.o o         |
|..+.. . S        |
|oo=.oo o         |
|oo.B.+*          |
|o =+X+o          |
| .oB*B           |
+----[SHA256]-----+
```  

두개의 key가 생성 된 것을 확인 할 수 있다.  

- public key : id_ed25519.pub
- private key : id_ed25519  


```bash
jakelee@jake-MacBookAir ~ % ls /Users/jakelee/.ssh
id_ed25519			id_ed25519.pub		
```  

<br/>

public key를 열어 보면 아래와 같고 키 값을 복사한다.  

```bash
jakelee@jake-MacBookAir .ssh % cat id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILk/b8XVyGXSE0zDquokPfdHVcaiRcXtydUQaCnrhK1t shclub@gmail.com
```  

웹 브라우저에서  https://github.com/ 로 이동하여 오른쪽 상단의 setting을 클릭한다.  


<img src="./assets/github_setting.png" style="width: 40%; height: auto;"/>  

왼쪽 메뉴에 SSH and GPG keys 메뉴로 이동하여 New SSH Key 아이콘을 클릭한다.

<img src="./assets/github_ssh_key.png" style="width: 60%; height: auto;"/>

<br/>

Title을 입력하고 복사한 public key를 붙여 넣기 한다.  

<img src="./assets/github_ssh_key_input.png" style="width: 80%; height: auto;"/>  

아래와 같이 public key 가 등록 된 것을 확인 할 수 있다.  

<img src="./assets/github_new_ssh_key_add.png" style="width: 80%; height: auto;"/>  

이제 Jenkins에 private key 를 등록하기 위하여 젠킨스 메인 화면에서 Manage Jenkins ->  Manage Credentials -> System -> Global credentials로 차례로 이동합니다.   

  - URL : http://211.252.85.148:9000/credentials/store/system/domain/_/    

<br/>  

Add credential을 클릭합니다.  

<img src="./assets/jenkins_add_credential.png" style="width: 80%; height: auto;"/>  

<br/>

비밀키를 복사합니다.  

```bash
jakelee@jake-MacBookAir .ssh % cat id_ed25519
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
xxxxxxxxVcaiRcXtydUQaCnrhK1tAAAAEHNoY2x1YkBnbWFpbC5jb20BAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```  

<br/>

복사가 되셨다면 아래와 같이 항목을 등록합니다.

<img src="./assets/jenkins_ssh_key.png" style="width: 80%; height: auto;"/>   

<br/>

- Kind
   인증 방식을 선택합니다.
   여기선 비밀키 방식을 선택해야 Github과 공개키/비밀키로 인증이 가능합니다.  

- Username
   각 젠킨스 Job에서 보여줄 인증키 이름 입니다.
   전 키 이름 그대로 사용했습니다.  생략해도 됨.

- Private Key
   좀전에 복사한 비밀키를 그대로 붙여넣습니다.
   젠킨스 설정도 다 끝났습니다.  

- passphrase
   ssh-keygen으로 인증키 생성시 입력한 password. 

<br/>

Jenkins pipeline 에서는 아래와 같이 사용 할 수 있습니다.    

- 참고 : https://github.com/shclub/edu13/blob/master/Jenkinsfile  

<br/>

아래에서 `keyFileVariable: 'keyFile'` 구문은 keyFile이 화일 이름이고 readFile을 하면 private key를 가져 올 수 있다.  


```bash
      stage('GitOps update : kustomize') {
            steps{
                print "======kustomization.yaml tag update====="
                script{
                   withCredentials([sshUserPrivateKey(credentialsId: 'github_ssh',keyFileVariable: 'keyFile')]) {                       
                    def  GITHUB_SSH_KEY = readFile(keyFile) 
                    sh """   
                        cd ~
                        rm -rf ./${GIT_OPS_NAME}
                        mkdir -p .ssh         
                        set +x
                        echo  '${GITHUB_SSH_KEY}' > ~/.ssh/id_rsa
                        chmod 600 ~/.ssh/id_rsa
                        git config --global core.sshCommand "ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no"
                        git clone ${gitOpsUrl}
                        cd ./${GIT_OPS_NAME}
                        git checkout master
                        kustomize edit set image ${GIT_ACCOUNT}/${PROJECT_NAME}:${TAG}
                        git config --global user.email "${GIT_EMAIL}"
                        git config --global user.name "${GIT_ACCOUNT}"                   
                        git add .
                        git commit -am 'update image tag ${TAG}'
                        git push origin master
                    """
                      }            
                }
                        
                print "git push finished !!!"
            }
        }
```

<br/>

위와 같이 push를 하면 Jenkins Console output 에 아래와 같은 메시지가 나오고
해당 링크를 클릭해서 들어간다.  

<img src="./assets/jenkins_ssh_git.png" style="width: 80%; height: auto;"/>   

<br/>

해당 github 사이트에 가면 승인 버튼이 활성화 된다.  

<img src="./assets/github_ssh_approve.png" style="width: 80%; height: auto;"/>   

Approve를 하면 연동이 verify 된다.

<img src="./assets/github_ssh_approved.png" style="width: 80%; height: auto;"/>   


<br/>

Service Account의 Source Code Control 종류는 다음과 같습니다.

<br/>

|SCC| Description |
|:--| :-------|  
| anyuid	| root를 포함한 모든 user로 컨테이너를 실행가능 |
| hostaccess |	제한된 user권한으로 호스트의 파일시스템 및 네트워크 접근 가능 |
| hostmount-anyuid	|anyuid의 host mounts를 통한 호스트 파일시스템 접근 가능 |
| hostnetwork | 호스트의 네트워크, 포트 접근 가능 |
| node-exporter |	Prometheus의 node exporter를 위한 권한|
| nonroot |	root를 제외한 모든 user로 컨테이너 실행가능|
| privileged |	cluster administration만을 위한 권한, 호스트의 모든 기능에 접근 가능 |
| restricted |	모든 호스트 기능에 접속 불가(Default) |

<br/>

참고 : https://gruuuuu.github.io/ocp/svca-s2i/

<br/>

worker node 로 ssh 접속한다.

NAS ( NFS ) 를 마운트 하여 원하는 폴더는 생성한다.  


```bash
[root@edu ~]# mount -t nfs 172.25.1.164:/share_aed398f1_3e3e_4d30_8b59_8185229ebde0 /mnt
[root@edu ~]# cd /mnt
[root@edu mnt]# ls
image-registry  prometheus-data00  prometheus-data01
[root@edu mnt]# mkdir jenkins
[root@edu mnt]# chown -R nfsnobody:nfsnobody jenkins
[root@edu mnt]# chmod 777 jenkins
[root@edu mnt]# ls -al
total 28
drwxrwxrwx.  6 root      root      4096 Aug  3 08:56 .
drwxr-xr-x. 24 root      root      4096 Jul 21 03:29 ..
drwxrwxrwx.  2 root      root      4096 Jun 20 08:23 .snapshot
drwxrwxrwx.  2 nfsnobody nfsnobody 4096 Jun 20 08:23 image-registry
drwxrwxrwx.  2 nfsnobody nfsnobody 4096 Aug  3 08:56 jenkins
drwxrwxrwx.  3 nfsnobody nfsnobody 4096 Jun 20 08:24 prometheus-data00
drwxrwxrwx.  3 nfsnobody nfsnobody 4096 Jun 20 08:24 prometheus-data01
[root@edu mnt]# showmount -e 172.25.1.162
Export list for 172.25.1.162:
/                                           (everyone)
/share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5 (everyone)
```  

<br/>

---

## OKD에 ArgoCD 설치  

<br/>

### ArgoCD의 Namespace 생성   

<br/>

ArgoCD의 Namespace를 생성 합니다.  

```bash
root@newedu:~# oc new-project argocd --display-name 'argocd'
Now using project "argocd" on server "https://api.211-34-231-81.nip.io:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/serve_hostname

```
<br/>

### ArgoCD의 권한 설정 

<br/> 

```bash
root@newedu:~# oc adm policy add-scc-to-user anyuid -z default -n argocd
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "default"
root@newedu:~# oc adm policy add-scc-to-user piivileged -z default -n argocd
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:piivileged added: "default"
root@newedu:~# oc adm policy add-scc-to-user anyuid -z argocd-application-controller -n argocd
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "argocd-application-controller"
root@newedu:~# oc adm policy add-scc-to-user anyuid -z argocd-applicationset-controller -n argocd
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "argocd-applicationset-controller"
root@newedu:~# oc adm policy add-scc-to-user anyuid -z argocd-applications-controller -n argocd
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "argocd-applications-controller"
root@newedu:~# oc adm policy add-scc-to-user anyuid -z argocd-server -n argocd
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "argocd-server"
```

<br/>

### 사내망에서 설정

<br/>

인터넷 가능한 오픈 환경이면 아래 과정 ( secret / serviceacoount 설정 ) 을 생략한다.  

<br/>

secret 및 serviceaccount 설정 ( 폐쇄망에서 Nexus를 private registry 사용하는 경우 )  

#### secret 생성  

```bash
root@newedu:~# kubectl create secret docker-registry <secret 이름> --docker- server=<nexus 서버 url>     --docker-username=<계정> --docker-password=<비밀번호> --docker-email=<이메일> -n argocd
```  
<br/>

#### 서비스 어카운트에 적용  

<br/>

```bash
root@newedu:~# kubectl get secrets <secret 이름> -n argocd 
root@newedu:~# kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "<secret 이름>"}]}' -n argocd
```  

<br/>

###  설치

<br/>

해당 링크의 화일을 다운 또는 복사 하여 install_argocd.yaml 화일을 만들고 설치를 진행 합니다.  
- https://github.com/shclub/edu14/blob/master/argocd/install_argocd.yaml

<br/>

```bash
root@newedu:~# vi install_argocd.yaml
root@newedu:~# kubectl apply -f install_argocd.yaml -n argocd
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
serviceaccount/argocd-dex-server created
serviceaccount/argocd-redis created
serviceaccount/argocd-server created
role.rbac.authorization.k8s.io/argocd-application-controller created
role.rbac.authorization.k8s.io/argocd-dex-server created
role.rbac.authorization.k8s.io/argocd-server created
clusterrole.rbac.authorization.k8s.io/argocd-application-controller created
clusterrole.rbac.authorization.k8s.io/argocd-server created
rolebinding.rbac.authorization.k8s.io/argocd-application-controller created
rolebinding.rbac.authorization.k8s.io/argocd-dex-server created
rolebinding.rbac.authorization.k8s.io/argocd-redis created
rolebinding.rbac.authorization.k8s.io/argocd-server created
clusterrolebinding.rbac.authorization.k8s.io/argocd-application-controller created
clusterrolebinding.rbac.authorization.k8s.io/argocd-server created
configmap/argocd-cm created
configmap/argocd-cmd-params-cm created
configmap/argocd-gpg-keys-cm created
configmap/argocd-rbac-cm created
configmap/argocd-ssh-known-hosts-cm created
configmap/argocd-tls-certs-cm created
secret/argocd-secret created
service/argocd-dex-server created
service/argocd-metrics created
service/argocd-redis created
service/argocd-repo-server created
service/argocd-server created
service/argocd-server-metrics created
deployment.apps/argocd-dex-server created
deployment.apps/argocd-redis created
deployment.apps/argocd-repo-server created
deployment.apps/argocd-server created
statefulset.apps/argocd-application-controller created
networkpolicy.networking.k8s.io/argocd-application-controller-network-policy created
networkpolicy.networking.k8s.io/argocd-dex-server-network-policy created
networkpolicy.networking.k8s.io/argocd-redis-network-policy created
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
root@newedu:~# kubectl get po -n argocd
NAME                                  READY   STATUS    RESTARTS   AGE
argocd-application-controller-0       1/1     Running   0          3m50s
argocd-dex-server-78c4b7f48d-qmpmn    1/1     Running   0          3m51s
argocd-redis-68568bc74-xmqkm          1/1     Running   0          3m51s
argocd-repo-server-5667749479-g7qg2   1/1     Running   0          3m51s
argocd-server-65dbc886db-5kkjq        1/1     Running   0          3m50s
```  

<br/>

####   ArgoCD의 Redis 설치 오류시 조치

<br/>


redis가 설치가 안되는 경우는 install_argocd.yaml 에서는 runAsUser 값을 적당한 값으로 수정해야 한다.   

OKD 환경 마다 다르기 때문에 kubectl get events로 에러 내용을 확인하고 range에 있는 적당한 값으로 수정 후 설치하여야 한다.  

```bash
 74       securityContext:
 75         runAsNonRoot: true
 76         runAsUser: 1000740000
```  

<br/>

POD 기동하기 전 에러 발생시 이벤트 보기   

```bash
root@newedu:~# kubectl get events -n argocd
...
36m         Warning   FailedCreate             replicaset/argocd-redis-8877bd5f            Error creating: pods "argocd-redis-8877bd5f-" is forbidden: unable to validate against any security context constraint: [spec.containers[0].securityContext.runAsUser: Invalid value: 1000840001: must be in the ranges: [1000740000, 1000749999]]
17m         Warning   FailedCreate             replicaset/argocd-redis-8877bd5f            Error creating: pods "argocd-redis-8877bd5f-" is forbidden: unable to validate against any security context constraint: [spec.containers[0].securityContext.runAsUser: Invalid value: 1000840001: must be in the ranges: [1000740000, 1000749999]]
...
```  

<br/>

####  ArgoCD Route 생성

<br/>

해당 링크의 화일을 다운 또는 복사 하여 route_argocd.yaml 화일을 만들고 route를 설정한다.  ( route 는 Openshift 에서만 사용 )  
- https://github.com/shclub/edu14/blob/master/argocd/route_argocd.yaml  

<br/>

```bash
root@newedu:~# vi route_argocd.yaml
root@newedu:~# kubectl apply -f route_argocd.yaml -n argocd
route.route.openshift.io/argocd-server-route created
root@newedu:~# kubectl get route -n argocd
NAME                  HOST/PORT                                 PATH   SERVICES        PORT    TERMINATION            WILDCARD
argocd-server-route   argocd-argocd.apps.211-34-231-82.nip.io          argocd-server   https   passthrough/Redirect   None
```  

<br/>

admin 초기 비밀번호를 확인한다.  

```bash
root@newedu:~# kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
AyzY39yr4RLkF0RE
```  
<br/>

웹브라우저에서 argocd-argocd.apps.211-34-231-82.nip.io  로 접속하여 userinfo 에서 admin 비밀번호를 변경한다.  

<br/>

####  ArgoCD Client 설치

<br/>

터미널에서 argocd client를 설치합니다.  

```bash
root@newedu:~#  curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 76.7M  100 76.7M    0     0  12.7M      0  0:00:06  0:00:06 --:--:-- 16.0M
root@newedu:~# chmod +x ./kubectl-argo-rollouts-linux-amd64
root@newedu:~# sudo mv ./kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
root@newedu:~# kubectl argo rollouts version
kubectl-argo-rollouts: v1.2.2+22aff27
  BuildDate: 2022-07-26T17:24:43Z
  GitCommit: 22aff273bf95646e0cd02555fbe7d2da0f903316
  GitTreeState: clean
  GoVersion: go1.17.6
  Compiler: gc
  Platform: linux/amd64
```  

<br/>

####  Route로 ArgoCD 접속

<br/>

계정 생성및 권한을 주기 위하여 route url로 접속하여 로그인을 합니다.  


```bash
root@newedu:~# argocd login argocd-argocd.apps.211-34-231-82.nip.io
WARNING: server certificate had error: x509: certificate is valid for localhost, argocd-server, argocd-server.argocd, argocd-server.argocd.svc, argocd-server.argocd.svc.cluster.local, not argocd-argocd.apps.211-34-231-82.nip.io. Proceed insecurely (y/n)? y
Username: admin
Password:
'admin:login' logged in successfully
Context 'argocd-argocd.apps.211-34-231-82.nip.io' updated
```  

<br/>

현재 계정을 확인합니다.  

```bash
root@newedu:~# argocd account list
NAME   ENABLED  CAPABILITIES
admin  true     login
```  

신규 계정을 아래와 같이 생성합니다.  data 를 추가하고 아래와 같이 계정 insert 하고 저장.  

```bash
root@newedu:~# kubectl -n argocd edit configmap argocd-cm -o yaml
apiVersion: v1
data:
  accounts.edu1: apiKey,login
  accounts.edu2: apiKey,login
  accounts.edu3: apiKey,login
  accounts.edu4: apiKey,login
  accounts.edu5: apiKey,login
  accounts.edu6: apiKey,login
  accounts.edu7: apiKey,login
  accounts.edu8: apiKey,login
  accounts.edu9: apiKey,login
  accounts.edu10: apiKey,login
  accounts.edu11: apiKey,login
  accounts.edu12: apiKey,login
  accounts.edu13: apiKey,login
  accounts.edu14: apiKey,login
  accounts.edu15: apiKey,login
  accounts.edu16: apiKey,login
  accounts.edu17: apiKey,login
  accounts.edu18: apiKey,login
  accounts.edu19: apiKey,login
  accounts.edu20: apiKey,login
  accounts.edu21: apiKey,login
  accounts.edu22: apiKey,login
  accounts.edu23: apiKey,login
  accounts.edu24: apiKey,login
  accounts.edu25: apiKey,login
  accounts.edu26: apiKey,login
  accounts.edu27: apiKey,login
  accounts.edu28: apiKey,login
  accounts.edu29: apiKey,login
  accounts.edu30: apiKey,login
  accounts.edu31: apiKey,login
  accounts.edu32: apiKey,login
  accounts.edu33: apiKey,login
  accounts.edu34: apiKey,login
  accounts.edu35: apiKey,login
  accounts.rookie1: apiKey,login
  accounts.rookie2: apiKey,login
  accounts.rookie3: apiKey,login
  accounts.rookie4: apiKey,login
  accounts.rookie5: apiKey,login
kind: ConfigMap
metadata:
```  

<br/>

이제 권한을 할당합니다.  

```bash
root@newedu:~# argocd account list
NAME     ENABLED  CAPABILITIES
admin    true     login
edu1     true     apiKey, login
edu10    true     apiKey, login
edu11    true     apiKey, login
edu12    true     apiKey, login
edu13    true     apiKey, login
edu14    true     apiKey, login
edu15    true     apiKey, login
edu16    true     apiKey, login
edu17    true     apiKey, login
edu18    true     apiKey, login
edu19    true     apiKey, login
edu2     true     apiKey, login
edu20    true     apiKey, login
edu21    true     apiKey, login
edu22    true     apiKey, login
edu23    true     apiKey, login
edu24    true     apiKey, login
edu25    true     apiKey, login
edu26    true     apiKey, login
edu27    true     apiKey, login
edu28    true     apiKey, login
edu29    true     apiKey, login
edu3     true     apiKey, login
edu30    true     apiKey, login
edu31    true     apiKey, login
edu32    true     apiKey, login
edu33    true     apiKey, login
edu34    true     apiKey, login
edu35    true     apiKey, login
edu5     true     apiKey, login
edu6     true     apiKey, login
edu7     true     apiKey, login
edu8     true     apiKey, login
edu9     true     apiKey, login
rookie1  true     apiKey, login
rookie2  true     apiKey, login
rookie3  true     apiKey, login
rookie4  true     apiKey, login
rookie5  true     apiKey, login
root@newedu:~# kubectl -n argocd edit configmap argocd-rbac-cm -o yaml
apiVersion: v1
data:
  policy.csv: |
    p, role:manager, applications, *, */*, allow
    p, role:manager, clusters, get, *, allow
    p, role:manager, repositories, *, *, allow
    p, role:manager, projects, *, *, allow
    p, role:edu1, applications, *, edu1/*, allow
    p, role:edu1, clusters, *, *, allow
    p, role:edu1, repositories, *, *, allow
    p, role:edu1, projects, *, *, allow
    p, role:edu2, applications, *, edu2/*, allow
    p, role:edu2, clusters, *, *, allow
    p, role:edu2, repositories, *, *, allow
    p, role:edu2, projects, *, *, allow
    p, role:edu3, applications, *, edu3/*, allow
    p, role:edu3, clusters, *, *, allow
    p, role:edu3, repositories, *, *, allow
    p, role:edu3, projects, *, *, allow
    p, role:edu4, applications, *, edu4/*, allow
    p, role:edu4, clusters, *, *, allow
    p, role:edu4, repositories, *, *, allow
    p, role:edu4, projects, *, *, allow
    p, role:edu5, applications, *, edu5/*, allow
    p, role:edu5, clusters, *, *, allow
    p, role:edu5, repositories, *, *, allow
    p, role:edu5, projects, *, *, allow
    p, role:edu6, applications, *, edu6/*, allow
    p, role:edu6, clusters, *, *, allow
    p, role:edu6, repositories, *, *, allow
    p, role:edu6, projects, *, *, allow
    p, role:edu7, applications, *, edu7/*, allow
    p, role:edu7, clusters, *, *, allow
    p, role:edu7, repositories, *, *, allow
    p, role:edu7, projects, *, *, allow
    p, role:edu8, applications, *, edu8/*, allow
    p, role:edu8, clusters, *, *, allow
    p, role:edu8, repositories, *, *, allow
    p, role:edu8, projects, *, *, allow
    p, role:edu9, applications, *, edu9/*, allow
    p, role:edu9, clusters, *, *, allow
    p, role:edu9, repositories, *, *, allow
    p, role:edu9, projects, *, *, allow
    p, role:edu10, applications, *, edu10/*, allow
    p, role:edu10, clusters, *, *, allow
    p, role:edu10, repositories, *, *, allow
    p, role:edu10, projects, *, *, allow
    p, role:edu11, applications, *, edu11/*, allow
    p, role:edu11, clusters, *, *, allow
    p, role:edu11, repositories, *, *, allow
    p, role:edu11, projects, *, *, allow
    p, role:edu12, applications, *, edu12/*, allow
    p, role:edu12, clusters, *, *, allow
    p, role:edu12, repositories, *, *, allow
    p, role:edu12, projects, *, *, allow
    p, role:edu13, applications, *, edu13/*, allow
    p, role:edu13, clusters, *, *, allow
    p, role:edu13, repositories, *, *, allow
    p, role:edu13, projects, *, *, allow
    p, role:edu14, applications, *, edu14/*, allow
    p, role:edu14, clusters, *, *, allow
    p, role:edu14, repositories, *, *, allow
    p, role:edu14, projects, *, *, allow
    p, role:edu15, applications, *, edu15/*, allow
    p, role:edu15, clusters, *, *, allow
    p, role:edu15, repositories, *, *, allow
    p, role:edu15, projects, *, *, allow
    p, role:edu16, applications, *, edu16/*, allow
    p, role:edu16, clusters, *, *, allow
    p, role:edu16, repositories, *, *, allow
    p, role:edu16, projects, *, *, allow
    p, role:edu17, applications, *, edu17/*, allow
    p, role:edu17, clusters, *, *, allow
    p, role:edu17, repositories, *, *, allow
    p, role:edu17, projects, *, *, allow
    p, role:edu18, applications, *, edu18/*, allow
    p, role:edu18, clusters, *, *, allow
    p, role:edu18, repositories, *, *, allow
    p, role:edu18, projects, *, *, allow
    p, role:edu19, applications, *, edu19/*, allow
    p, role:edu19, clusters, *, *, allow
    p, role:edu19, repositories, *, *, allow
    p, role:edu19, projects, *, *, allow
    p, role:edu20, applications, *, edu20/*, allow
    p, role:edu20, clusters, *, *, allow
    p, role:edu20, repositories, *, *, allow
    p, role:edu20, projects, *, *, allow
    p, role:edu21, applications, *, edu21/*, allow
    p, role:edu21, clusters, *, *, allow
    p, role:edu21, repositories, *, *, allow
    p, role:edu21, projects, *, *, allow
    p, role:edu22, applications, *, edu22/*, allow
    p, role:edu22, clusters, *, *, allow
    p, role:edu22, repositories, *, *, allow
    p, role:edu22, projects, *, *, allow
    p, role:edu23, applications, *, edu23/*, allow
    p, role:edu23, clusters, *, *, allow
    p, role:edu23, repositories, *, *, allow
    p, role:edu23, projects, *, *, allow
    p, role:edu24, applications, *, edu24/*, allow
    p, role:edu24, clusters, *, *, allow
    p, role:edu24, repositories, *, *, allow
    p, role:edu24, projects, *, *, allow
    p, role:edu25, applications, *, edu25/*, allow
    p, role:edu25, clusters, *, *, allow
    p, role:edu25, repositories, *, *, allow
    p, role:edu25, projects, *, *, allow
    p, role:edu26, applications, *, edu26/*, allow
    p, role:edu26, clusters, *, *, allow
    p, role:edu26, repositories, *, *, allow
    p, role:edu26, projects, *, *, allow
    p, role:edu27, applications, *, edu27/*, allow
    p, role:edu27, clusters, *, *, allow
    p, role:edu27, repositories, *, *, allow
    p, role:edu27, projects, *, *, allow
    p, role:edu28, applications, *, edu28/*, allow
    p, role:edu28, clusters, *, *, allow
    p, role:edu28, repositories, *, *, allow
    p, role:edu28, projects, *, *, allow
    p, role:edu29, applications, *, edu29/*, allow
    p, role:edu29, clusters, *, *, allow
    p, role:edu29, repositories, *, *, allow
    p, role:edu29, projects, *, *, allow
    p, role:edu30, applications, *, edu30/*, allow
    p, role:edu30, clusters, *, *, allow
    p, role:edu30, repositories, *, *, allow
    p, role:edu30, projects, *, *, allow
    p, role:edu31, applications, *, edu31/*, allow
    p, role:edu31, clusters, *, *, allow
    p, role:edu31, repositories, *, *, allow
    p, role:edu31, projects, *, *, allow
    p, role:edu32, applications, *, edu32/*, allow
    p, role:edu32, clusters, *, *, allow
    p, role:edu32, repositories, *, *, allow
    p, role:edu32, projects, *, *, allow
    p, role:edu33, applications, *, edu33/*, allow
    p, role:edu33, clusters, *, *, allow
    p, role:edu33, repositories, *, *, allow
    p, role:edu33, projects, *, *, allow
    p, role:edu34, applications, *, edu34/*, allow
    p, role:edu34, clusters, *, *, allow
    p, role:edu34, repositories, *, *, allow
    p, role:edu34, projects, *, *, allow
    p, role:edu35, applications, *, edu35/*, allow
    p, role:edu35, clusters, *, *, allow
    p, role:edu35, repositories, *, *, allow
    p, role:edu35, projects, *, *, allow
    p, role:rookie1, applications, *, rookie1/*, allow
    p, role:rookie1, clusters, *, *, allow
    p, role:rookie1, repositories, *, *, allow
    p, role:rookie1, projects, *, *, allow
    p, role:rookie2, applications, *, rookie2/*, allow
    p, role:rookie2, clusters, *, *, allow
    p, role:rookie2, repositories, *, *, allow
    p, role:rookie2, projects, *, *, allow
    p, role:rookie3, applications, *, rookie3/*, allow
    p, role:rookie3, clusters, *, *, allow
    p, role:rookie3, repositories, *, *, allow
    p, role:rookie3, projects, *, *, allow
    p, role:rookie4, applications, *, rookie4/*, allow
    p, role:rookie4, clusters, *, *, allow
    p, role:rookie4, repositories, *, *, allow
    p, role:rookie4, projects, *, *, allow
    p, role:rookie5, applications, *, rookie5/*, allow
    p, role:rookie5, clusters, *, *, allow
    p, role:rookie5, repositories, *, *, allow
    p, role:rookie5, projects, *, *, allow
    g, edu1, role:edu1
    g, edu2, role:edu2
    g, edu3, role:edu3
    g, edu4, role:edu4
    g, edu5, role:edu5
    g, edu6, role:edu6
    g, edu7, role:edu7
    g, edu8, role:edu8
    g, edu9, role:edu9
    g, edu10, role:edu10
    g, edu11, role:edu11
    g, edu12, role:edu12
    g, edu13, role:edu13
    g, edu14, role:edu14
    g, edu15, role:edu15
    g, edu16, role:edu16
    g, edu17, role:edu17
    g, edu18, role:edu18
    g, edu19, role:edu19
    g, edu20, role:edu20
    g, edu21, role:edu21
    g, edu22, role:edu22
    g, edu23, role:edu23
    g, edu24, role:edu24
    g, edu25, role:edu25
    g, edu26, role:edu26
    g, edu27, role:edu27
    g, edu28, role:edu28
    g, edu29, role:edu29
    g, edu30, role:edu30
    g, edu31, role:edu31
    g, edu32, role:edu32
    g, edu33, role:edu33
    g, edu34, role:edu34
    g, edu35, role:edu35
    g, rookie1, role:rookie1
    g, rookie2, role:rookie2
    g, rookie3, role:rookie3
    g, rookie4, role:rookie4
    g, rookie5, role:rookie5
  policy.default: role:''
kind: ConfigMap
```  

</br>

각 계정의 비밀번호를 설정합니다.  

```bash
root@newedu:~# argocd account get --account edu1
Name:               edu1
Enabled:            true
Capabilities:       apiKey, login

Tokens:
NONE
root@newedu:~# argocd account update-password --account edu1
*** Enter password of currently logged in user (admin):
*** Enter new password for user edu1:
*** Confirm new password for user edu1:
Password updated
```  

<br/>

추가적으로 argo-rollouts namespace를 생성하고 rollout을 설치합니다.  

아래 링크의 화일을 사용하여 install_argorollouts.yaml을 생성합니다.
- https://github.com/shclub/edu14/blob/master/argocd/install_argorollouts.yaml  


<br/>

```bash
root@newedu:~# kubectl apply -f install_argorollouts.yaml -n argo-rollouts
customresourcedefinition.apiextensions.k8s.io/analysisruns.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/analysistemplates.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/clusteranalysistemplates.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/experiments.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/rollouts.argoproj.io created
serviceaccount/argo-rollouts created
clusterrole.rbac.authorization.k8s.io/argo-rollouts created
clusterrole.rbac.authorization.k8s.io/argo-rollouts-aggregate-to-admin created
clusterrole.rbac.authorization.k8s.io/argo-rollouts-aggregate-to-edit created
clusterrole.rbac.authorization.k8s.io/argo-rollouts-aggregate-to-view created
clusterrolebinding.rbac.authorization.k8s.io/argo-rollouts created
secret/argo-rollouts-notification-secret created
service/argo-rollouts-metrics created
deployment.apps/argo-rollouts created
root@newedu:~# kubectl get all -n argo-rollouts
NAME                                 READY   STATUS    RESTARTS   AGE
pod/argo-rollouts-5c964cb4f5-bq94n   1/1     Running   0          9m34s

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/argo-rollouts-metrics   ClusterIP   172.30.44.187   <none>        8090/TCP   9m36s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argo-rollouts   1/1     1            1           9m36s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/argo-rollouts-5c964cb4f5   1         1         1       9m35s
```  



<br/>

####  ArgoCD 프로젝트 생성 및 추가 설정

<br/>

아래 링크의 뒷부분을 참고합니다.  

- https://github.com/shclub/edu/blob/master/argocd_hands_on.md

<br/>


## OKD 에  Elastic Stack  설치   

<br>

참고
- https://artifacthub.io/packages/helm/elastic/elasticsearch

<br/>

<img src="./assets/elasticstack1.png" style="width: 80%; height: auto;"/>  

<br/>

### 설치 

<br/>


VM에 로그인 한 후에 elastic 폴더를 생성한다.  

<br/>

yaml 화일 참고 : https://github.com/shclub/elastic

<br/>

```bash
root@newedu:~# mkdir -p elastic
root@newedu:~# cd elastic
``` 

<br/>

default service account 에 아래와 같이 권한을 부여 한다.  

<br/>

```bash
root@newedu:~/elastic# oc adm policy add-scc-to-user anyuid -z default -n elastic-system
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "default"
root@newedu:~/elastic# oc adm policy add-scc-to-user privileged -z default -n elastic-system
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:privileged added: "default"
```

<br/>

#### Storage 설정

<br/>


elastic-snapshop 과 kibana 가 사용하는 stroage를 위해 pv / pvc 를 생성해야 하며
사전에 NFS 에 접속하여 폴더를 생성한다.     

elastic data 스토리지는 dynamic provisioning 으로 설치 예정이라 별로 세팅은 필요 없다.  

<br/>

elastic-snapshop 은 아래 폴더에 생성되어 있고 수강생은 본인의 폴더 직접 생성.

<br/>

```bash
[root@edu elastic-snapshop]# pwd
/mnt/elastic-snapshop
[root@edu elastic-snapshop]# mkdir -p edu
```

<br/>

kibana 용 폴더도 생성한다.  (기생성)
수강생읕 본인은 폴더를 kibana 아래에 생성한다.  

```bash
[root@edu kibana]# pwd
/mnt/kibana
[root@edu kibana]# mkdir -p edu
...
```

<br/>

elastic-snapshop / kibana 용 해당 폴더의 권한을 설정한다.

<br/>

worker node에서 mount 해서 폴더 권한을 주는 경우는 아래 처럼 설정하고   

`chown -R nfsnobody:nfsnobody edu`  

pod 내에서 nfs 연결해서 권한을 줄때는  nobody:nogroup 으로 준다.  

`chown -R nobody:nogroup edu`

<br/>

elastic 용 폴더    


```bash
[root@edu elastic-snapshop]# chown -R nfsnobody:nfsnobody edu
[root@edu elastic-snapshop]# chmod 777 edu
```  

kibana 용 폴더  

```bash
[root@edu kibana]# chown -R nfsnobody:nfsnobody edu
[root@edu kibana]# chmod 777 edu
```  


<br/>

elastic snapshot 용 PV 를 생성한다. 사이즈는 5G로 설정한다.

<br/>

```bash  
root@newedu:~/elastic#  vi elastic_snapshot_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elastic-snapshop-edu-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 5Gi
  nfs:
    path: /share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5/elastic-snapshot/edu
    server: 172.25.1.162
  persistentVolumeReclaimPolicy: Retain
```

<br/>

PV를 생성하고 Status를 확인해보면 Available 로 되어 있는 것을 알 수 있습니다.  

<br/>

```bash
root@newedu:~/elastic# kubectl apply -f elastic_snapshot_pv.yaml
```  

PV를 생성하고 Status를 확인해보면 Available 로 되어 있는 것을 알 수 있습니다.  

<br/>

elastic snapshot 용  pvc 를 생성합니다. pvc 이름을 기억합니다.

<br/>

```bash
root@newedu:~/elastic# vi elastic_snapshot_pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: elastic-snapshot-edu-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  volumeName: elastic-snapshop-edu-pv
```

<br/>

```bash
root@newedu:~/elastic# kubectl apply -f  elastic_snapshot_pvc.yaml -n elastic-system
persistentvolumeclaim/elastic-snapshot-edu-pvc created
```

<br/>

kibana 용 PV 를 생성한다. 사이즈는 5G로 설정한다.

<br/>

```bash  
root@newedu:~/elastic#  vi kibana_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kibana-edu-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 5Gi
  nfs:
    path: /share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5/kibana/edu
    server: 172.25.1.162
  persistentVolumeReclaimPolicy: Retain
```

<br/>

```bash
root@newedu:~/elastic# kubectl apply -f kibana_pv.yaml
persistentvolume/kibana-edu-pv created
root@newedu:~/elastic# kubectl get pv kibana-edu-pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
kibana-edu-pv   5Gi        RWX            Retain           Available                                   16s
```

<br/>

PV를 생성하고 Status를 확인해보면 Available 로 되어 있는 것을 알 수 있습니다.  

<br/>

kibana 용 pvc 를 생성합니다. pvc 이름을 기억합니다.

<br/>

```bash
root@newedu:~/elastic# vi kibana_pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kibana-edu-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  volumeName: kibana-edu-pv
```

<br/>

PVC 를 생성할 때는 namespace ( 본인의 namespace ) 를 명시해야 합니다.  

<br/>

```bash
root@newedu:~/elastic# kubectl apply -f kibana_pvc.yaml -n elastic-system
persistentvolumeclaim/kibana-edu-pvc created
```  


PVC 생성을 확인 해보고 다시 PV를 확인해 보면 Status가 Bound 로 되어 있는 것을 알 수 있습니다.  이제 PV 와 PVC가 연결이 되었습니다.


<br/>


#### Helm 으로 elastic 설치

<br/>

helm repo 업데이트를 합니다.  

```bash
root@newedu:~/elastic# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jenkins" chart repository
...Successfully got an update from the "nfs-subdir-external-provisioner" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```  

<br/>


elastic helm reposiory를 추가합니다.

<br/>

```bash
root@newedu:~/elastic# helm repo add elastic https://helm.elastic.co
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/postgresql   	12.2.6       	15.2.0     	PostgreSQL (Postgres) is an open source object-...
bitnami/postgresql-ha	11.2.0       	15.2.0     	This PostgreSQL cluster solution includes the P...
bitnami/supabase     	0.1.4        	0.23.2     	Supabase is an open source Firebase alternative...
```  

<br/>

elastic 으로 검색을 하여 `elastic/elasticsearch` 를 찾습니다.  


```bash
root@newedu:~/elastic# helm search repo elastic
NAME                     	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/elasticsearch    	19.6.0       	8.6.2      	Elasticsearch is a distributed search and analy...
elastic/eck-elasticsearch	0.3.0        	           	A Helm chart to deploy Elasticsearch managed by...
elastic/elasticsearch    	8.5.1        	8.5.1      	Official Elastic helm chart for Elasticsearch
elastic/apm-attacher     	0.1.0        	           	A Helm chart installing the Elastic APM mutatin...
elastic/apm-server       	8.5.1        	8.5.1      	Official Elastic helm chart for Elastic APM Server
elastic/eck-agent        	0.3.0        	           	A Helm chart to deploy Elastic Agent managed by...
elastic/eck-beats        	0.2.0        	           	A Helm chart to deploy Elastic Beats managed by...
elastic/eck-fleet-server 	0.3.0        	           	A Helm chart to deploy Elastic Fleet Server as ...
elastic/eck-kibana       	0.3.0        	           	A Helm chart to deploy Kibana managed by the EC...
elastic/eck-operator     	2.7.0        	2.7.0      	A Helm chart for deploying the Elastic Cloud on...
elastic/eck-operator-crds	2.7.0        	2.7.0      	A Helm chart for installing the ECK operator Cu...
elastic/eck-stack        	0.4.0        	           	A Parent Helm chart for all Elastic stack resou...
elastic/filebeat         	8.5.1        	8.5.1      	Official Elastic helm chart for Filebeat
elastic/kibana           	8.5.1        	8.5.1      	Official Elastic helm chart for Kibana
elastic/logstash         	8.5.1        	8.5.1      	Official Elastic helm chart for Logstash
elastic/metricbeat       	8.5.1        	8.5.1      	Official Elastic helm chart for Metricbeat
bitnami/dataplatform-bp2 	12.0.5       	1.0.1      	DEPRECATED This Helm chart can be used for the ...
bitnami/kibana           	10.2.18      	8.7.0      	Kibana is an open source, browser based analyti...
```  

<br/>

`elastic/elasticsearch` 차트에서 차트의 변수 값을 변경하기 위해 elastic_values.yaml 화일을 추출한다.

<br/>


```bash
root@newedu:~/elastic# helm show values elastic/elasticsearch  > elastic_values.yaml
```

<br/>

storageclass 이름을 확인합니다.  


```bash
root@newedu:~/elastic# kubectl get storageclass
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   229d
```

<br/>

elastic_values.yaml 를 수정한다.  

- 24~25 : replicas 2 로 변경
- 31~34 : 메모리 swapping 설정을 한다. 메모리 부족하면 주석처리. 
- 90~98 : JVM Option 과 POD 리소스를 설정한다.
- 108~115 : volumeClaimTemplate 에서 storageClass를 설정하여 Dynamic Provisioning을 한다.
- 142~157 : 스토리지 사용 유무를 설정하고 snapshot pvc를 설정한다.
- 222~232 : securityContext를 설정한다.

<br/>

```bash
 24 replicas: 2 # 2로 변경 
 25 minimumMasterNodes: 1 #2 quorum
 ... 
 31 esConfig:
 32   elasticsearch.yml: |  # 주석 풀고 추가 설정
 33     #bootstrap.memory_lock: true # disable swapping
 34     path.repo: "/usr/share/elasticsearch/snapshot" # snapshot repository
 35 #    key:
 36 #      nestedkey: value
 37 #  log4j2.properties: |
 38 #    key = value
 ...
 90 esJavaOpts: "-Xmx2g -Xms2g" # 메모리 설정 일단 2기가  example: "-Xmx1g -Xms1g"
 91
 92 resources:
 93   requests:
 94     cpu: "4" # "1000m"  worker node spec 8core 16G
 95     memory: "2Gi" #"2Gi"
 96   limits:  # limit 제한 없애기
 97     cpu: null #"1000m"
 98     memory: null # "2Gi"
 ...
108 networkHost: "0.0.0.0"
109
110 volumeClaimTemplate:
111   accessModes: ["ReadWriteOnce"]
112   storageClassName : "nfs-client"
113   resources:
114     requests:
115       storage: 5Gi #30Gi elastic data storage pv
...
142 persistence:
143   enabled: true # pv/pvc use flag
144   labels:
145     # Add default labels for the volumeClaimTemplate of the StatefulSet
146     enabled: false
147   annotations: {}
148
149 extraVolumes:  # snapshot  volume
150  - name: snapshot
151    persistentVolumeClaim:
152      claimName: elastic-snapshot-edu-pvc
153    #   emptyDir: {}
154
155 extraVolumeMounts:
156  - name: snapshot
157    mountPath: /usr/share/elasticsearch/snapshot
...
222 podSecurityContext:
223   fsGroup: null #1000 openshfit
224   runAsUser: null #1000 openshift
225
226 securityContext:
227   capabilities:
228     drop:
229       - ALL
230   # readOnlyRootFilesystem: true
231   runAsNonRoot: true
232   runAsUser: 1000  # openshift
```

<br/>

이제 elasticsearch 를 설치 합니다.

<br/>

```bash
root@newedu:~/elastic# helm install elastic elastic/elasticsearch -f elastic_values.yaml -n elastic-system
NAME: elastic
LAST DEPLOYED: Tue Apr 18 19:35:45 2023
NAMESPACE: elastic-system
STATUS: deployed
REVISION: 1
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=elastic-system -l app=elasticsearch-master -w
2. Retrieve elastic user's password.
  $ kubectl get secrets --namespace=elastic-system elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
3. Test cluster health using Helm test.
  $ helm --namespace=elastic-system test elastic
```

<br/>

pod를 확인한다.

```bash
root@newedu:~/elastic# kubectl get po -n elastic-system
NAME                     READY   STATUS    RESTARTS   AGE
elasticsearch-master-0   1/1     Running   0          3m3s
elasticsearch-master-1   1/1     Running   0          3m3s
```

<br/>

NFS 서버에 접속하여 data 폴더가 생성 되었는지 확인한다.  

```bash
[root@edu database]# pwd
/mnt/database
[root@edu database]# ls elastic-system-elasticsearch-master-elasticsearch-master-*
elastic-system-elasticsearch-master-elasticsearch-master-0-pvc-95ec3eed-e8f9-46b3-b832-362c362724db:
_state  indices  node.lock  nodes  snapshot_cache

elastic-system-elasticsearch-master-elasticsearch-master-1-pvc-9adfb438-769f-46f8-af62-c456e93920cf:
_state  indices  node.lock  nodes  snapshot_cache
```  

<br/>

#### Helm 으로 kibana  설치 및 설정

<br/>

kibana 는 Helm Chart 를 이용하여 설치를 합니다.  

<br/>

현재 로컬의 helm repository 를 확인한다.   

<br/>

```bash
root@newedu:~/elastic# helm repo list
NAME                           	URL
bitnami                        	https://charts.bitnami.com/bitnami
nfs-subdir-external-provisioner	https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
jenkins                        	https://charts.jenkins.io
oteemo                         	https://oteemo.github.io/charts/
elastic                        	https://helm.elastic.co
```  

<br/>

elastic helm reppository 에서 helm chart를 검색을 하고 kibana chart를 선택합니다.  

<br/>

```bash
root@newedu:~/elastic# helm search repo kibana
NAME                     	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/kibana           	10.2.18      	8.7.0      	Kibana is an open source, browser based analyti...
elastic/eck-kibana       	0.3.0        	           	A Helm chart to deploy Kibana managed by the EC...
elastic/kibana           	8.5.1        	8.5.1      	Official Elastic helm chart for Kibana
elastic/eck-operator     	2.7.0        	2.7.0      	A Helm chart for deploying the Elastic Cloud on...
elastic/eck-stack        	0.4.0        	           	A Parent Helm chart for all Elastic stack resou...
bitnami/dataplatform-bp2 	12.0.5       	1.0.1      	DEPRECATED This Helm chart can be used for the ...
elastic/eck-operator-crds	2.7.0        	2.7.0      	A Helm chart for installing the ECK operator Cu...
```

<br/>

elastic/kibana 차트에서 차트의 변수 값을 변경하기 위해 kibana_values.yaml 화일을 추출한다.

<br/>


```bash
root@newedu:~/elastic#  helm show values elastic/kibana > kibana_values.yaml
```


<br/>

vi 데이터에서 생성된 kibana_values.yaml을 연다.  

<br/>

```bash
root@newedu:~/elastic# vi kibana_values.yaml
```  

라인을 보기 위해 ESC 를 누른 후 `:set nu` 를 입력하면 왼쪽에 라인이 보인다.  


<br/>
수정내용
 - 105,108 라인 : admin 계정과 비밀번호
 - 312 라인 : readinessProbe 는 false 로 변경
 - 1003 라인 : postgresql는  별도 설치된 DB 사용으로 false로 변경
 - 1022~1026 : 위에서 설정한 postgresql db 로 설정. host는 서비스 이름

<br/>

```bash

```  

<br/>

#### Helm 으로 kibana 설치

<br/>

keycloak_values.yaml 를 사용하여 설치 한다.

<br/>

```bash
root@newedu:~/keycloak# helm install keycloak bitnami/keycloak -f keycloak_values.yaml -n edu30 --insecure-skip-tls-verify
NAME: keycloak
LAST DEPLOYED: Fri Apr  7 11:03:19 2023
NAMESPACE: edu30
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: keycloak
CHART VERSION: 13.4.0
APP VERSION: 20.0.5

** Please be patient while the chart is being deployed **

Keycloak can be accessed through the following DNS name from within your cluster:

    keycloak.edu30.svc.cluster.local (port 80)

To access Keycloak from outside the cluster execute the following commands:

1. Get the Keycloak URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        You can watch its status by running 'kubectl get --namespace edu30 svc -w keycloak'

    export HTTP_SERVICE_PORT=$(kubectl get --namespace edu30 -o jsonpath="{.spec.ports[?(@.name=='http')].port}" services keycloak)
    export SERVICE_IP=$(kubectl get svc --namespace edu30 keycloak -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

    echo "http://${SERVICE_IP}:${HTTP_SERVICE_PORT}/"

2. Access Keycloak using the obtained URL.
3. Access the Administration Console using the following credentials:

  echo Username: admin
  echo Password: $(kubectl get secret --namespace edu30 keycloak -o jsonpath="{.data.admin-password}" | base64 -d)
```   

<br/>

설치가 완료되면 pod를 조회하여 keycloak-0 pod가 있는지 확인한다.    

없으면 event 를 확인해본다.  

<br/>

```bash
root@newedu:~/keycloak# kubectl get events
LAST SEEN   TYPE      REASON         OBJECT                           MESSAGE
keycloak             create Pod keycloak-0 in StatefulSet keycloak failed error: pods "keycloak-0" is forbidden: unable to validate against any security context constraint: [provider restricted: .spec.securityContext.fsGroup: Invalid value: []int64{1001}: 1001 is not an allowed group spec.containers[0].securityContext.runAsUser: Invalid value: 1001: must be in the ranges: [1001420000, 1001429999]]
124m        Normal    Pulled         pod/
```

<br/>

권한 관련 에러가 발생한 것을 볼 수 있고 아래와 같이 keycloak 서비스 어카운트에게  권한을 준다.

<br/>

```bash
root@newedu:~/keycloak# oc adm policy add-scc-to-user anyuid -z  keycloak 
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "keycloak"
root@newedu:~/keycloak# oc adm policy add-scc-to-user privileged -z keycloak
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:privileged added: "keycloak"
``` 

<br/>

keycloak 을 삭제 하고 다시 설치해본다.

```bash
root@newedu:~/keycloak# helm delete keycloak 
release "keycloak" uninstalled
root@newedu:~/keycloak# helm install keycloak bitnami/keycloak -f keycloak_values.yaml --insecure-skip-tls-verify
```  
<br/>

pod를 확인해봅니다.

<br/>

```bash
root@newedu:~/keycloak# kubectl get po
NAME                                        READY   STATUS             RESTARTS   AGE
keycloak-0                                  1/1     Running            0          34s
```  


<br/>

#### Keycloak 설정

<br/>


