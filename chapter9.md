# Chapter 9 
 
Redhat Openshift 의 오픈소스 버전인 OKD 4.7 를 사용하여 
k8s 고급 기능과 컨테이너 기반의 Jenkins 그리고 skaffold를 사용한
개발 환경을 구성해 본다. 

<br/>

## OKD Cluster   

### Cluster 생성 
  

<br/>

kt cloud 에 로그인 하여 D1 존 으로 이동한다.  

클러스터 생성시 각 노드에 로그인 하기 위해서는 SSH Keypair를 먼저 생성해야한다. ( 클러스터 구성 완료 후 추가는  불가 )  

서버 -> SSH Key pair 로 이동하여 create 버튼 클릭하여 이름 입력한 후 launch 아이콘 클릭하여 생성.   

<img src="./assets/ktcloud_keypair.png" style="width: 80%; height: auto;"/>    

키는 바로 생성이 된다. ( edu라는 이름으로 기 생성된 값 사용 )

<img src="./assets/ktcloud_keypair_created.png" style="width: 80%; height: auto;"/>  

download 아이콘을 클릭하여 pem 화일을 로컬 pc에 다운 받는다.  

<img src="./assets/ktcloud_keypair_download.png" style="width: 80%; height: auto;"/>  

<br/>

K2P 상품을 클릭하여 Cluster를 선택한다.  

<img src="./assets/kt_cloud_d1.png" style="width: 80%; height: auto;"/>  

<br/>

OKD 클러스터 생성을 위한 기본 정보를 입력하다.  

이름과 DMZ Tier/Private Tier는 아래와 같이 설정하고 다음 버튼 클릭.  

<img src="./assets/okd_create1.png" style="width: 80%; height: auto;"/>  

<br/>

OKD 4.x 버전 부터는 Centos 대신 CoreOS를 사용한다.  

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

- 워커 노드 edu.worker01~04 : 교육용 namespace 배치 ( edu로 시작 )
- 워커 노드 edu.worker05 : Jenkins 및 기타 솔루션 설치. ( 교육용 namespace 배치 불가 설정 )  

<br/>

먼저 master 노드 설정을 확인한다.  

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

- edu.worker01~04 
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

worker node 4번까지 같은 방식으로 Label를 추가한다.  

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


`oc adm policy add-scc-to-user anyuid system:serviceaccount:<NAMESPAE>:default`  

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

###  Worker Node 접속 

<br/>

worker node에 접속하기 위해서는 public ip를 할당 하고 port forwarding 과 방화벽을 오픈해야한다.  

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

정상적으로 접속이 되면 root의 비밀번호를 설정한다.  

```bash
[core@edu ~]$ sudo passwd
Changing password for user root.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
Sorry, passwords do not match.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```  

<br/>

pem 화일 없이 root 계정으로 worker node에 접속 가능하다.  

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