# k8s Middle Hands-on
 
kubernetes에서 Basic 과정에서 진행하지 못했던 부분 실습을 합니다.

1. Storage Volume  ( PV/PVC , DB 설치 + NFS )

2. NFS 라이브러리 설치 ( Native Kubernetes )

3. Service - Headless, Endpoint, ExternalName

3. EFK APM 설치

4. EFK APM Agent 설정  ( React / SpringBoot )

5. EFK APM 활용

6. 참고 사이트 
    - Storage Volume : https://anggeum.tistory.com/m/entry/Kubernetes-Volume-Deep-Dive
    - Ubuntu NFS : https://server-talk.tistory.com/378
    - https://tech.osci.kr/2021/10/06/kubernetes-volume%EC%9D%84-%EA%B3%B5%EB%B6%80%ED%95%B4%EB%B3%B4%EC%9E%90/
    - 인프런 : 대세는 쿠버네티스
    - https://junior-developer.tistory.com/76?category=928073
    - https://zgundam.tistory.com/179
    - https://waspro.tistory.com/580


<br/>

##  1. Storage Volume 

<br/>

kubernetes volume은  Pod의 구성 요소로  컨테이너 와 동일하게 Pod Spec에서 정의 된다.    

volume 은  kubernetes object가 아니므로 자체적으로 생성,삭제 될수 없다.  

volume 은 컨테이너에서 사용 가능 하지만 접근하려는 컨테이너에서  각각 마운트 되어야 한다. 각 컨테이너에서  파일 시스템의 어느 경로에나  볼륨을 마운트 할수 있다.  

<br/>

이번 실습에서는 Internal Network에서 로컬 볼륨과 NFS 볼륨을 대상으로 합니다.  

<img src="./assets/k8s_volume1.png" style="width: 80%; height: auto;"/>

<br/>

### 1.1 로컬 볼륨

로컬 볼륨의 형태는 2가지가 있고 휘발성으로 인하여 자주 사용하지는 않는다.  
- emptyDir : Pod 내부의 Container들이 공유하는 임시 볼륨입니다.  
             emptyDir은 Pod가 기동되면서 생성되고 종료되면서 삭제됩니다.


- hostPath : Node ( VM인 경우 VM 서버 ) 에 있는 파일 시스템을  Pod의 디렉토리로   마운트 하는데 사용한다. pod가 다른 Node에 실행되면 사용 불가.  



<br/>

#### 1.1.1 emptyDir 

<br/>

대부분 POD는 1개의 컨테이너가 존재 하지만 여러개의 컨테이너가 있을 수도 있다.  


<img src="./assets/k8s_volume2.png" style="width: 60%; height: auto;"/>  

<br/>

아래 yaml 화일을 내용을 복사하여 emptyDir.yaml 화일을 생성 합니다.

<br/>

```bash
vi emptyDir.yaml
```  

emp-storage-pod 이라는 이름의 POD는 2개의 컨테이너로 구성이 되며
하나는 ubuntu-container, 또 하나는 nginx-container 입니다.  

같은 POD 안의 2개의 container 간에 logs 라는 폴더를 사용하여 데이터를 공유 할 수 있습니다.  

```bash
apiVersion: v1
kind: Pod
metadata:
  name: emp-storage-pod
spec:
  containers:
    - image: ubuntu:18.04
      name: ubuntu-container
      command: ["tail","-f", "/dev/null"]
      volumeMounts:
        - mountPath: /logs
          name: emptydir-volume
    - image: nginx:latest
      name: nginx-container
      volumeMounts:
        - mountPath: /logs
          name: emptydir-volume
  volumes:
    - name: emptydir-volume
      emptyDir: {}
```  

<br/>

storage-test 라는 namespace를 사용하여 pod를 생성합니다.  
( 실습자는 본인의 namespace에서 실행 )  


```bash
root@newedu-k3s:~# kubectl apply -f emptyDir.yaml -n storage-test
pod/emp-storage-pod created
root@newedu-k3s:~# kubectl get po -n storage-test
NAME              READY   STATUS              RESTARTS   AGE
emp-storage-pod   0/2     ContainerCreating   0          8s
root@newedu-k3s:~# kubectl get po -n storage-test
NAME              READY   STATUS    RESTARTS   AGE
emp-storage-pod   2/2     Running   0          18s
```  

READY를 보면 하나의 pod에 READY 가 2/2로 되어 있는 것을 알 수 있습니다.  

컨테이너 이름을 모르면 아래 명령어를 사용하여 컨테이너 이름을 알수 있습니다.  

<br/>

```bash
root@newedu-k3s:~# kubectl describe po emp-storage-pod -n storage-test
Name:         emp-storage-pod
Namespace:    storage-test
Priority:     0
Node:         newedu-k3s/172.27.0.41
Start Time:   Wed, 10 Aug 2022 08:03:16 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.42.0.169
IPs:
  IP:  10.42.0.169
Containers:
  ubuntu-container:
    Container ID:  containerd://6cdda9739a56356528252f95e10f4976881361beb3e1f711de983c52ce48d9ff
    Image:         ubuntu:18.04
    Image ID:      docker.io/library/ubuntu@sha256:eb1392bbdde63147bc2b4ff1a4053dcfe6d15e4dfd3cce29e9b9f52a4f88bc74
    Port:          <none>
    Host Port:     <none>
    Command:
      tail
      -f
      /dev/null
    State:          Running
      Started:      Wed, 10 Aug 2022 08:03:22 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /logs from emptydir-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zlxq5 (ro)
  nginx-container:
    Container ID:   containerd://5f201c5ef11d04f1eaf8f2bf414cf278abbf6205045ba15821cfbeb6905e6387
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:ecc068890de55a75f1a32cc8063e79f90f0b043d70c5fcf28f1713395a4b3d49
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 10 Aug 2022 08:03:29 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /logs from emptydir-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zlxq5 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  emptydir-volume:
    Type:       EmptyDir 
    Medium:
    SizeLimit:  <unset>
  kube-api-access-zlxq5:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  31s   default-scheduler  Successfully assigned storage-test/emp-storage-pod to newedu-k3s
  Normal  Pulling    31s   kubelet            Pulling image "ubuntu:18.04"
  Normal  Pulled     26s   kubelet            Successfully pulled image "ubuntu:18.04" in 5.706066951s
  Normal  Created    26s   kubelet            Created container ubuntu-container
  Normal  Started    26s   kubelet            Started container ubuntu-container
  Normal  Pulling    26s   kubelet            Pulling image "nginx:latest"
  Normal  Pulled     19s   kubelet            Successfully pulled image "nginx:latest" in 6.674048852s
  Normal  Created    19s   kubelet            Created container nginx-container
  Normal  Started    19s   kubelet            Started container nginx-container
``` 

<br/>

이제 ubuntu-container 에 shell 로 접속하여 logs 폴더에 화일을 하나 생성합니다.  

```bash
root@newedu-k3s:~# kubectl exec -it emp-storage-pod -c ubuntu-container sh -n storage-test
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# ls
bin  boot  dev	etc  home  lib	lib64  logs  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# cd /logs
# ls
# echo "emptyDir test" >> edu.txt
# cat edu.txt
emptyDir test
# exit
```  

<br/>

ngix-container 에 shell 로 접속하여 logs 폴더에 생성된 edu.txt 라는 화일을 확인합니다.   

```bash
root@newedu-k3s:~# kubectl exec -it emp-storage-pod -c nginx-container sh -n storage-test
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# cd /logs
# ls
edu.txt
# cat edu.txt
emptyDir test
# exit
```  


<br/>

#### 1.1.2 hostPath 

<br/>

hostPath는 로컬 디스크의 경로를 Pod 에 Mount 해서 사용하는 Volume 방식입니다.  

Docker 에서 -v 옵션으로 Volume 을 연결하는 것과 동일하다고 생각하면 됩니다.   

<img src="./assets/k8s_volume3.png" style="width: 60%; height: auto;"/>   

<br/>


hostPath는 방식은 호스트 OS의 Docker(/var/lib/docker)에 접근해야 하거나 호스트 OS에서 개발한 환경을 Container 내부에 적용해야 하거나, 또는 반대로 컨테이너 환경에서 추가된 파일을 호스트 OS에서 사용해야 할 경우 유용하게 사용할 수 있습니다.  

hostPath는 방식의 경우 호스트 OS의 장애가 발생했을 경우 Pod를 실행할 수 없으며, 호스트 OS의 디스크에 대한 데이터 손실 또는 가용성 등에 대해 고려해야 합니다.  


아래 yaml 화일을 내용을 복사하여 hostpath.yaml 화일을 생성 합니다.

<br/>

```bash
vi hostpath.yaml
``` 

<br/>


```bash
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-storage-pod
spec:
  containers:
    - image: nginx:latest
      name: nginx-container
      volumeMounts:
        - mountPath: /hostdir
          name: hostdir-volume
  volumes:
  - name: hostdir-volume
    hostPath:
      path: /root
      type: Directory
```  


<br/>

storage-test 라는 namespace를 사용하여 pod를 생성합니다.  
( 실습자는 본인의 namespace에서 실행 )   

```bash
root@newedu-k3s:~# kubectl apply -f hostpath.yaml -n storage-test
pod/hostpath-storage-pod created
root@newedu-k3s:~# kubectl get po -n storage-test
NAME                   READY   STATUS    RESTARTS   AGE
emp-storage-pod        2/2     Running   0          18m
hostpath-storage-pod   1/1     Running   0          13s
```  

생성된 hostpath-storage-pod 에 shell 로 접근합니다.  

```bash
root@newedu-k3s:~# kubectl exec -it hostpath-storage-pod sh -n storage-test
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# ls
bin   dev		   docker-entrypoint.sh  home	  lib	 media	opt   root  sbin  sys  usr
boot  docker-entrypoint.d  etc			 hostdir  lib64  mnt	proc  run   srv   tmp  var
```  

<br/>

hostdir로 이동을 하고 화일을 조회 해보면 현재 서버의 화일들이 있는 것을 확인 할 수 있다.    

```bash
# cd /hostdir
# ls
argo-cd.yaml	       emptyDir.yaml  nfs-pv-test.yaml	nfs-pvc.yaml   nginx.yaml  pvc.yaml
cloud-init-setting.sh  hostpath.yaml  nfs-pv.yaml	nfs-test.yaml  pv.yaml
```    

즉 호스트 OS의 /root 디렉토리가 Pod 내부에서 /hostdir 디렉토리 위치에 마운트 된것을 확인할 수 있습니다.  

hostPath 경우 ReadWriteOnce로만 적용이 가능하므로 Container 또는 서비스 별로 특정 디렉토리를 지정하여 로그에이전트가 수집하는 로그를 모으는 역할로 사용할 수 있습니다.  

<br/>

`NFS도 서버에서 Mount 한 후 hostPath로 설정 해서 사용 가능 하다. ` 


<br/>

hostPath Volume을 데이터 저장소로 채택하기에는 적합하지 않다.  

특정 노드의 FileSystem에 저장되므로 파드가 다른 노드로 Scheduling이 되면 더 이상 이전 데이터를 볼 수 없다.  

Pod가 어떤 노드에 Scheduling 되느냐에 따라 민감하기 때문에 일반적인 Pod에 사용하는 것은 좋은 생각이 아니다. 특수한 경우를 제외한다면 hostPath를 사용하는 것은 보안 및 활용성 측면에서 그다지 바람직하지 않다.  

<br/>

<img src="./assets/k8s_volume4.png" style="width: 60%; height: auto;"/>   

<br/>

### 1.2 네트웍 볼륨

Network Volume은 Persistent Volume(PV) 이외에 Persistent Volume Claim(이하 PVC)이 존재합니다.  


<br/>

#### 1.2.1 Persistent Volume

<br/>

Persistent Volume(이하 PV) 는 Kubernetes 에서 관리되는 저장소로 Pod 과는 다른 수명 주기로 관리됩니다.  

Pod 이 재실행 되더라도, PV의 데이터는 정책에 따라 유지/삭제가 됩니다.  

PV 생성은 Cluster 권한이 필요하면 일반 사용자는 생성이 불가능 하다.  

HostPath도 PV로 할당 가능 하지만 결국 Local 이기 때문에 큰 의미는 없다.  
    
<br/>

#### 1.2.2 Persistent Volume Claim

<br/>

Persistent Volume Claim(이하 PVC) 는 PV를 추상화하여 개발자가 손쉽게 PV를 사용 가능하게 만들어주는 기능입니다.  

개발자는 사용에 필요한 Volume의 크기, Volume의 정책을 선택하고 요청만 하면 됩니다.   

운영자는 개발자의 요청에 맞게 PV 를 생성하게 되고, PVC는 해당 PV를 가져가게 됩니다.  

PV 와 PVC는 1:1 관계이다.  

<br/>

#### 1.2.3 PV, PVC의 LifeCycle

<br/>

PV, PVC의 LifeCycle은 총 4가지 형태를 갖고 변화하게 됩니다.  


- Available : PV 생성
- Bound : PVC에 의해 바인딩 되었을 경우
- Released : PVC가 삭제되었을 경우
- Fail : PV에 문제가 발생하였을 경우  

LifeCycle은 Available → Bound → Released → Bound ... 순으로 반복되며, Released 즉 PVC가 삭제되었을 경우 실제 스토리지에 있는 파일을 어떻게 관리할 것인지에 대한 Reclaiming(재활용) 정책을 결정할 수 있습니다. 


PV, PVC의 Reclaiming 정책으로는 다음 세가지를 적용할 수 있습니다.  
- Retain : 삭제하지 않고 PV의 내용 유지
- Recycle : 재 사용이 가능하며, 재 사용시에는 데이타의 내용을 자동으로 rm -rf 로 삭제한 후 재사용
- Delete : 볼륨의 사용이 끝나면, 볼륨 삭제 (AWS EBS, GCE PD,Azure Disk 등)



<br/>

#### 1.2.5 Static Provisioning

<br/>


PV를 생성하고 PVC를 생성하는 방식을 Static Provisioning 이라 합니다. 

<br>

이번 실습은 NFS 서버를 PV로 사용하는 방식입니다.    

NFS 서버가 없는 경우 아래 deployment와 service yaml화일을 사용하여 임시로 컨테이너 NFS 서버를 구성 할 수 있다.    

<br/>

이미 NFS 서버가 있으면 skip.

<br/>


deployment 생성    

nfs-server-deployment.yaml  
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-server
spec:
  selector:
    matchLabels:
      role: nfs-server
  template:
    metadata:
      labels:
        role: nfs-server
    spec:
      containers:
      - name: nfs-server
        image: gcr.io/google_containers/volume-nfs:0.8
        ports:
          - name: nfs
            containerPort: 2049
          - name: mountd
            containerPort: 20048
          - name: rpcbind
            containerPort: 111
        securityContext:
          privileged: true
```  

<br/>

service 생성  


nfs-server-service.yaml  
```bash
apiVersion: v1
kind: Service
metadata:
  name: nfs-service
spec:
  ports:
  - name: nfs
    port: 2049
  - name: mountd
    port: 20048
  - name: rpcbind
    port: 111
  selector:
    role: nfs-server
```  

<br/>

kt cloud 에서 FlyingCube를 생성 하면 NAS가 할당이 되고 nfs 서버가 활성화 되어 NAS에 연결 할 수 있다.  

예제에 활용될 yaml 파일 내용은 아래와 같습니다. 
- nfs 연동하기 전에 nfs 에 mount 하여 폴더를 생성 해야 한다. ( edu1/nfs-nginx )  

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nfs-nginx
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: nfsvol
      mountPath: /usr/share/nginx/html
  volumes:
  - name : nfsvol
    nfs:
      path: /share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5/edu/nfs-nginx
      server: 172.25.1.162
```  


<br/>

이제 bitnami 의 helm chart를 이용하여 mariadb 를 설치 해보도록 합니다.  

mariadb는 NFS 통해 데이터를 저장합니다.   


먼저 nfs를 pv로 등록 ( Cluster 권한 : 인프라  ) 하고 
각 namespace별로 pvc를 생성 ( namespace 권한 : 개발팀 ) 합니다.  

pv 화일 : 교육생들은 불필요 ( 사전 생성 되었음 )      

db_pv.yaml  
```bash  
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    storage: mariadb-pv1
  name: mariadb-pv1
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 4Gi
  nfs:
    path: /share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5/database
    server: 172.25.1.162
  persistentVolumeReclaimPolicy: Retain
```  
<br/>

pvc 화일은 아래 내용을 복사하여 vi 에디터로 db_pvc.yaml 화일을 생성합니다.    
volumeName을 본인의 pv로 변경 한다.  
   
db_pvc.yaml  
```bash  
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    app: mariadb-pvc
  name: mariadb-pvc
spec:
  storageClassName: "" 
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 4Gi
  volumeName: mariadb-pv1
```

<br/>

이제 pv를 생성합니다 ( 교육생들은 불필요. 사전 생성 되었음 )  

```bash
root@newedu:~# kubectl apply -f db_pv.yaml
persistentvolume/mariadb-pv created
root@newedu:~# kubectl get pv
NAME                     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                                     STORAGECLASS   REASON   AGE
image-registry-pv        100Gi      RWX            Retain           Bound       openshift-image-registry/image-registry-storage                                   70d
jenkins-restore-pv       100Gi      RWX            Retain           Bound       devops/jenkins-restore-pvc                                                        26d
mariadb-pv               600Gi      RWX            Retain           Available                                                                                     10s
prometheus-k8s-db00-pv   100Gi      RWO            Retain           Bound       openshift-monitoring/prometheus-k8s-db-prometheus-k8s-0                           70d
prometheus-k8s-db01-pv   100Gi      RWO            Retain           Bound       openshift-monitoring/prometheus-k8s-db-prometheus-k8s-1                           70d
```  

정상적으로 mariadb-pv 라는 이름으로 pv가 생성 되었습니다. 

<br/>

이제 일반 유저로 로그인을 하고 pvc 를 생성합니다.  

```bash  
root@newedu:~# oc login https://api.211-34-231-81.nip.io:6443 -u edu1-admin -p New1234! --insecure-skip-tls-verify
Login successful.

You have one project on this server: "edu1"

Using project "edu1".
root@newedu:~# kubectl apply -f db_pvc.yaml
persistentvolumeclaim/mariadb-pvc created
root@newedu:~# kubectl get pvc
NAME          STATUS    VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mariadb-pvc   Pending   mariadb-pv   0                                        5s
root@newedu:~# kubectl get pvc
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mariadb-pvc   Bound    mariadb-pv   600Gi      RWX                           6s
```  

pvc 가  정상적으로 pv에 연결되면 STATUS가 Bound 로 표시됩니다.  

<br/>


```bash
root@newedu:~# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
root@newedu:~# helm repo list
NAME   	URL
bitnami	https://charts.bitnami.com/bitnami
root@newedu:~# helm search repo mariadb
NAME                  	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/mariadb       	11.2.1       	10.6.9     	MariaDB is an open source, community-developed ...
bitnami/mariadb-galera	7.4.1        	10.6.9     	MariaDB Galera is a multi-primary database clus...
bitnami/phpmyadmin    	10.3.1       	5.2.0      	phpMyAdmin is a free software tool written in P...
root@newedu:~# helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
root@newedu:~# helm install my-release bitnami/mariadb
NAME: my-release
LAST DEPLOYED: Mon Aug 29 16:00:23 2022
NAMESPACE: edu1
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mariadb
CHART VERSION: 11.2.1
APP VERSION: 10.6.9

** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace edu1 -l app.kubernetes.io/instance=my-release

Services:

  echo Primary: my-release-mariadb.edu1.svc.cluster.local:3306

Administrator credentials:

  Username: root
  Password : $(kubectl get secret --namespace edu1 my-release-mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 -d)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run my-release-mariadb-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mariadb:10.6.9-debian-11-r0 --namespace edu1 --command -- bash

  2. To connect to primary service (read/write):

      mysql -h my-release-mariadb.edu1.svc.cluster.local -uroot -p my_database

To upgrade this helm chart:

  1. Obtain the password as described on the 'Administrator credentials' section and set the 'auth.rootPassword' parameter as shown below:

      ROOT_PASSWORD=$(kubectl get secret --namespace edu1 my-release-mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 -d)
      helm upgrade --namespace edu1 my-release bitnami/mariadb --set auth.rootPassword=$ROOT_PASSWORD
```  

<br/>

위와 같이 설치를 하면 pod가 하나도 생성이 되지 않고 에러가 발생합니다.  

pod가 생성되지 않은 상태에서 에러를 보는 명령어는 `kubectl get events` 입니다.  

helm 으로 생성된 my-release 차트를 삭제합니다.  

```bash
root@newedu:~# helm delete my-release
release "my-release" uninstalled
root@newedu:~# helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```  

<br/>

helm chart에서는 values.yaml 화일이 있는데 사용자가 custom 할수 있는
yaml 화일 입니다.  

우리는 기본값으로 설치하지 않고 우리 환경에 맞게 수정하여 설치한다.

https://github.com/bitnami/charts/blob/master/bitnami/mariadb/values.yaml 사이트에서 values.yaml 를 다운 받고 수정한다.  

<br/>

설치 조건
- 서버 구성 : Primary 만 구성. secondary는 replica 0로 변경
- hostport : 사용 안함 ( 외부에서 접속을 위한 port )
- 아이디/패스워드 : edu / edu1234
- root 비밀번호 : 아이디와 동일
- DB 이름 : edu
- subPath: 본인 namespace/my-mariadb
- 도커 레지스트리 : docker hub ( private docker registry 이면 별도 설정 )

<br/>

위의 화일을 수정하기 힘든 경우 아래 링크에서 다운 받습니다.  
- https://github.com/shclub/edu14/blob/master/values.yaml  

본인의 namespace 이름만 변경 하면 됩니다. 지금은 subPath edu1 로 설정 되어 있음.  


```bash  
## @section Global parameters
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry, imagePullSecrets and storageClass
##

## @param global.imageRegistry Global Docker Image registry
## @param global.imagePullSecrets Global Docker registry secret names as an array
## @param global.storageClass Global storage class for dynamic provisioning
##
global:
  imageRegistry: ""
  ## E.g.
  ## imagePullSecrets:
  ##   - myRegistryKeySecretName
  ##
  imagePullSecrets: []
  storageClass: ""

## @section Common parameters
##

## @param kubeVersion Force target Kubernetes version (using Helm capabilities if not set)
##
kubeVersion: ""
## @param nameOverride String to partially override mariadb.fullname
##
nameOverride: ""
## @param fullnameOverride String to fully override mariadb.fullname
##
fullnameOverride: ""
## @param clusterDomain Default Kubernetes cluster domain
##
clusterDomain: cluster.local
## @param commonAnnotations Common annotations to add to all MariaDB resources (sub-charts are not considered)
##
commonAnnotations: {}
## @param commonLabels Common labels to add to all MariaDB resources (sub-charts are not considered)
##
commonLabels: {}
## @param schedulerName Name of the scheduler (other than default) to dispatch pods
## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
##
schedulerName: ""
## @param extraDeploy Array of extra objects to deploy with the release (evaluated as a template)
##
extraDeploy: []

## Enable diagnostic mode in the deployment
##
diagnosticMode:
  ## @param diagnosticMode.enabled Enable diagnostic mode (all probes will be disabled and the command will be overridden)
  ##
  enabled: false
  ## @param diagnosticMode.command Command to override all containers in the deployment
  ##
  command:
    - sleep
  ## @param diagnosticMode.args Args to override all containers in the deployment
  ##
  args:
    - infinity

## @section MariaDB common parameters
##

## Bitnami MariaDB image
## ref: https://hub.docker.com/r/bitnami/mariadb/tags/
## @param image.registry MariaDB image registry
## @param image.repository MariaDB image repository
## @param image.tag MariaDB image tag (immutable tags are recommended)
## @param image.digest MariaDB image digest in the way sha256:aa.... Please note this parameter, if set, will override the tag
## @param image.pullPolicy MariaDB image pull policy
## @param image.pullSecrets Specify docker-registry secret names as an array
## @param image.debug Specify if debug logs should be enabled
##
image:
  registry: docker.io
  repository: bitnami/mariadb
  tag: 10.6.9-debian-11-r0
  digest: ""
  ## Specify a imagePullPolicy
  ## Defaults to 'Always' if image tag is 'latest', else set to 'IfNotPresent'
  ## ref: https://kubernetes.io/docs/user-guide/images/#pre-pulling-images
  ##
  pullPolicy: IfNotPresent
  ## Optionally specify an array of imagePullSecrets (secrets must be manually created in the namespace)
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ## Example:
  ## pullSecrets:
  ##   - myRegistryKeySecretName
  ##
  pullSecrets: []
  ## Set to true if you would like to see extra information on logs
  ## It turns BASH and/or NAMI debugging in the image
  ##
  debug: false
## @param architecture MariaDB architecture (`standalone` or `replication`)
##
architecture: standalone
## MariaDB Authentication parameters
##
auth:
  ## @param auth.rootPassword Password for the `root` user. Ignored if existing secret is provided.
  ## ref: https://github.com/bitnami/containers/tree/main/bitnami/mariadb#setting-the-root-password-on-first-run
  ##
  rootPassword: edu1234 
  ## @param auth.database Name for a custom database to create
  ## ref: https://github.com/bitnami/containers/blob/main/bitnami/mariadb/README.md#creating-a-database-on-first-run
  ##
  database: edu 
  ## @param auth.username Name for a custom user to create
  ## ref: https://github.com/bitnami/containers/blob/main/bitnami/mariadb/README.md#creating-a-database-user-on-first-run
  ##
  username: edu 
  ## @param auth.password Password for the new user. Ignored if existing secret is provided
  ##
  password: edu1234 
  ## @param auth.replicationUser MariaDB replication user
  ## ref: https://github.com/bitnami/containers/tree/main/bitnami/mariadb#setting-up-a-replication-cluster
  ##
  replicationUser: replicator
  ## @param auth.replicationPassword MariaDB replication user password. Ignored if existing secret is provided
  ## ref: https://github.com/bitnami/containers/tree/main/bitnami/mariadb#setting-up-a-replication-cluster
  ##
  replicationPassword: ""
  ## @param auth.existingSecret Use existing secret for password details (`auth.rootPassword`, `auth.password`, `auth.replicationPassword` will be ignored and picked up from this secret). The secret has to contain the keys `mariadb-root-password`, `mariadb-replication-password` and `mariadb-password`
  ##
  existingSecret: ""
  ## @param auth.forcePassword Force users to specify required passwords
  ##
  forcePassword: false
  ## @param auth.usePasswordFiles Mount credentials as files instead of using environment variables
  ##
  usePasswordFiles: false
  ## @param auth.customPasswordFiles Use custom password files when `auth.usePasswordFiles` is set to `true`. Define path for keys `root` and `user`, also define `replicator` if `architecture` is set to `replication`
  ## Example:
  ## customPasswordFiles:
  ##   root: /vault/secrets/mariadb-root
  ##   user: /vault/secrets/mariadb-user
  ##   replicator: /vault/secrets/mariadb-replicator
  ##
  customPasswordFiles: {}
## @param initdbScripts Dictionary of initdb scripts
## Specify dictionary of scripts to be run at first boot
## Example:
## initdbScripts:
##   my_init_script.sh: |
##      #!/bin/bash
##      echo "Do something."
##
initdbScripts: {}
## @param initdbScriptsConfigMap ConfigMap with the initdb scripts (Note: Overrides `initdbScripts`)
##
initdbScriptsConfigMap: ""

## @section MariaDB Primary parameters
##

## Mariadb Primary parameters
##
primary:
  ## @param primary.name Name of the primary database (eg primary, master, leader, ...)
  ##
  name: primary
  ## @param primary.command Override default container command on MariaDB Primary container(s) (useful when using custom images)
  ##
  command: []
  ## @param primary.args Override default container args on MariaDB Primary container(s) (useful when using custom images)
  ##
  args: []
  ## @param primary.lifecycleHooks for the MariaDB Primary container(s) to automate configuration before or after startup
  ##
  lifecycleHooks: {}
  ## @param primary.hostAliases Add deployment host aliases
  ## https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/
  ##
  hostAliases: []
  ## @param primary.configuration [string] MariaDB Primary configuration to be injected as ConfigMap
  ## ref: https://mysql.com/kb/en/mysql/configuring-mysql-with-mycnf/#example-of-configuration-file
  ##
  configuration: |-
    [mysqld]
    skip-name-resolve
    explicit_defaults_for_timestamp
    basedir=/opt/bitnami/mariadb
    plugin_dir=/opt/bitnami/mariadb/plugin
    port=3306
    socket=/opt/bitnami/mariadb/tmp/mysql.sock
    tmpdir=/opt/bitnami/mariadb/tmp
    max_allowed_packet=16M
    bind-address=*
    pid-file=/opt/bitnami/mariadb/tmp/mysqld.pid
    log-error=/opt/bitnami/mariadb/logs/mysqld.log
    character-set-server=UTF8
    collation-server=utf8_general_ci
    slow_query_log=0
    slow_query_log_file=/opt/bitnami/mariadb/logs/mysqld.log
    long_query_time=10.0
    [client]
    port=3306
    socket=/opt/bitnami/mariadb/tmp/mysql.sock
    default-character-set=UTF8
    plugin_dir=/opt/bitnami/mariadb/plugin
    [manager]
    port=3306
    socket=/opt/bitnami/mariadb/tmp/mysql.sock
    pid-file=/opt/bitnami/mariadb/tmp/mysqld.pid
  ## @param primary.existingConfigmap Name of existing ConfigMap with MariaDB Primary configuration.
  ## NOTE: When it's set the 'configuration' parameter is ignored
  ##
  existingConfigmap: ""
  ## @param primary.updateStrategy.type MariaDB primary statefulset strategy type
  ## ref: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#update-strategies
  ##
  updateStrategy:
    ## StrategyType
    ## Can be set to RollingUpdate or OnDelete
    ##
    type: RollingUpdate
  ## @param primary.rollingUpdatePartition Partition update strategy for Mariadb Primary statefulset
  ## https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#partitions
  ##
  rollingUpdatePartition: ""
  ## @param primary.podAnnotations Additional pod annotations for MariaDB primary pods
  ## ref: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
  ##
  podAnnotations: {}
  ## @param primary.podLabels Extra labels for MariaDB primary pods
  ## ref: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
  ##
  podLabels: {}
  ## @param primary.podAffinityPreset MariaDB primary pod affinity preset. Ignored if `primary.affinity` is set. Allowed values: `soft` or `hard`
  ## ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity
  ##
  podAffinityPreset: ""
  ## @param primary.podAntiAffinityPreset MariaDB primary pod anti-affinity preset. Ignored if `primary.affinity` is set. Allowed values: `soft` or `hard`
  ## Ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity
  ##
  podAntiAffinityPreset: soft
  ## Mariadb Primary node affinity preset
  ## Ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity
  ##
  nodeAffinityPreset:
    ## @param primary.nodeAffinityPreset.type MariaDB primary node affinity preset type. Ignored if `primary.affinity` is set. Allowed values: `soft` or `hard`
    ##
    type: ""
    ## @param primary.nodeAffinityPreset.key MariaDB primary node label key to match Ignored if `primary.affinity` is set.
    ## E.g.
    ## key: "kubernetes.io/e2e-az-name"
    ##
    key: ""
    ## @param primary.nodeAffinityPreset.values MariaDB primary node label values to match. Ignored if `primary.affinity` is set.
    ## E.g.
    ## values:
    ##   - e2e-az1
    ##   - e2e-az2
    ##
    values: []
  ## @param primary.affinity Affinity for MariaDB primary pods assignment
  ## Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
  ## Note: podAffinityPreset, podAntiAffinityPreset, and  nodeAffinityPreset will be ignored when it's set
  ##
  affinity: {}
  ## @param primary.nodeSelector Node labels for MariaDB primary pods assignment
  ## Ref: https://kubernetes.io/docs/user-guide/node-selection/
  ##
  nodeSelector: {}
  ## @param primary.tolerations Tolerations for MariaDB primary pods assignment
  ## Ref: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
  ##
  tolerations: []
  ## @param primary.schedulerName Name of the k8s scheduler (other than default)
  ## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
  ##
  schedulerName: ""
  ## @param primary.podManagementPolicy podManagementPolicy to manage scaling operation of MariaDB primary pods
  ## ref: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies
  ##
  podManagementPolicy: ""
  ## @param primary.topologySpreadConstraints Topology Spread Constraints for MariaDB primary pods assignment
  ## ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/
  ## E.g.
  ## topologySpreadConstraints:
  ##   - maxSkew: 1
  ##     topologyKey: topology.kubernetes.io/zone
  ##     whenUnsatisfiable: DoNotSchedule
  ##
  topologySpreadConstraints: []
  ## @param primary.priorityClassName Priority class for MariaDB primary pods assignment
  ## Ref: https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/
  ##
  priorityClassName: ""
  ## MariaDB primary Pod security context
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod
  ## @param primary.podSecurityContext.enabled Enable security context for MariaDB primary pods
  ## @param primary.podSecurityContext.fsGroup Group ID for the mounted volumes' filesystem
  ##
  podSecurityContext:
    enabled: true
    fsGroup: 1000660000 #1001
  ## MariaDB primary container security context
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-container
  ## @param primary.containerSecurityContext.enabled MariaDB primary container securityContext
  ## @param primary.containerSecurityContext.runAsUser User ID for the MariaDB primary container
  ## @param primary.containerSecurityContext.runAsNonRoot Set Controller container's Security Context runAsNonRoot
  ##
  containerSecurityContext:
    enabled: true
    runAsUser: 1001
    runAsNonRoot: true
  ## MariaDB primary container's resource requests and limits
  ## ref: https://kubernetes.io/docs/user-guide/compute-resources/
  ## We usually recommend not to specify default resources and to leave this as a conscious
  ## choice for the user. This also increases chances charts run on environments with little
  ## resources, such as Minikube. If you do want to specify resources, uncomment the following
  ## lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  ## @param primary.resources.limits The resources limits for MariaDB primary containers
  ## @param primary.resources.requests The requested resources for MariaDB primary containers
  ##
  resources:
    ## Example:
    ## limits:
    ##    cpu: 100m
    ##    memory: 256Mi
    ##
    limits: {}
    ## Examples:
    ## requests:
    ##    cpu: 100m
    ##    memory: 256Mi
    ##
    requests: {}
  ## Configure extra options for MariaDB primary containers' liveness, readiness and startup probes
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes)
  ## @param primary.startupProbe.enabled Enable startupProbe
  ## @param primary.startupProbe.initialDelaySeconds Initial delay seconds for startupProbe
  ## @param primary.startupProbe.periodSeconds Period seconds for startupProbe
  ## @param primary.startupProbe.timeoutSeconds Timeout seconds for startupProbe
  ## @param primary.startupProbe.failureThreshold Failure threshold for startupProbe
  ## @param primary.startupProbe.successThreshold Success threshold for startupProbe
  ##
  startupProbe:
    enabled: false
    initialDelaySeconds: 120
    periodSeconds: 15
    timeoutSeconds: 5
    failureThreshold: 10
    successThreshold: 1
  ## Configure extra options for liveness probe
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes
  ## @param primary.livenessProbe.enabled Enable livenessProbe
  ## @param primary.livenessProbe.initialDelaySeconds Initial delay seconds for livenessProbe
  ## @param primary.livenessProbe.periodSeconds Period seconds for livenessProbe
  ## @param primary.livenessProbe.timeoutSeconds Timeout seconds for livenessProbe
  ## @param primary.livenessProbe.failureThreshold Failure threshold for livenessProbe
  ## @param primary.livenessProbe.successThreshold Success threshold for livenessProbe
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 120
    periodSeconds: 10
    timeoutSeconds: 1
    failureThreshold: 3
    successThreshold: 1
  ## @param primary.readinessProbe.enabled Enable readinessProbe
  ## @param primary.readinessProbe.initialDelaySeconds Initial delay seconds for readinessProbe
  ## @param primary.readinessProbe.periodSeconds Period seconds for readinessProbe
  ## @param primary.readinessProbe.timeoutSeconds Timeout seconds for readinessProbe
  ## @param primary.readinessProbe.failureThreshold Failure threshold for readinessProbe
  ## @param primary.readinessProbe.successThreshold Success threshold for readinessProbe
  ##
  readinessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 1
    failureThreshold: 3
    successThreshold: 1
  ## @param primary.customStartupProbe Override default startup probe for MariaDB primary containers
  ##
  customStartupProbe: {}
  ## @param primary.customLivenessProbe Override default liveness probe for MariaDB primary containers
  ##
  customLivenessProbe: {}
  ## @param primary.customReadinessProbe Override default readiness probe for MariaDB primary containers
  ##
  customReadinessProbe: {}
  ## @param primary.startupWaitOptions Override default builtin startup wait check options for MariaDB primary containers
  ## `bitnami/mariadb` Docker image has built-in startup check mechanism,
  ## which periodically checks if MariaDB service has started up and stops it
  ## if all checks have failed after X tries. Use these to control these checks.
  ## ref: https://github.com/bitnami/containers/tree/main/bitnami/mariadb/pull/240
  ## Example (with default options):
  ## startupWaitOptions:
  ##   retries: 300
  ##   waitTime: 2
  ##
  startupWaitOptions: {}
  ## @param primary.extraFlags MariaDB primary additional command line flags
  ## Can be used to specify command line flags, for example:
  ## E.g.
  ## extraFlags: "--max-connect-errors=1000 --max_connections=155"
  ##
  extraFlags: ""
  ## @param primary.extraEnvVars Extra environment variables to be set on MariaDB primary containers
  ## E.g.
  ## extraEnvVars:
  ##  - name: TZ
  ##    value: "Europe/Paris"
  ##
  extraEnvVars: []
  ## @param primary.extraEnvVarsCM Name of existing ConfigMap containing extra env vars for MariaDB primary containers
  ##
  extraEnvVarsCM: ""
  ## @param primary.extraEnvVarsSecret Name of existing Secret containing extra env vars for MariaDB primary containers
  ##
  extraEnvVarsSecret: ""
  ## Enable persistence using Persistent Volume Claims
  ## ref: https://kubernetes.io/docs/user-guide/persistent-volumes/
  ##
  persistence:
    ## @param primary.persistence.enabled Enable persistence on MariaDB primary replicas using a `PersistentVolumeClaim`. If false, use emptyDir
    ##
    enabled: true

    ## @param primary.persistence.existingClaim Name of an existing `PersistentVolumeClaim` for MariaDB primary replicas
    ## NOTE: When it's set the rest of persistence parameters are ignored
    ##
    
    ## nfs 사용함으로 주석 처리
    existingClaim: "mariadb-pvc"
    ## @param primary.persistence.subPath Subdirectory of the volume to mount at
    ##
    
    #subPath: ""
    subPath: edu1/my-mariadb

    ## @param primary.persistence.storageClass MariaDB primary persistent volume storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    # pod 에서 연동
    #storageClass: ""
    ## @param primary.persistence.annotations MariaDB primary persistent volume claim annotations
    ##
    annotations: {}
    ## @param primary.persistence.accessModes MariaDB primary persistent volume access Modes
    ##
    accessModes:
      - ReadWriteOnce
    ## @param primary.persistence.size MariaDB primary persistent volume size
    ##
    size: 4Gi
    ## @param primary.persistence.selector Selector to match an existing Persistent Volume
    ## selector:
    ##   matchLabels:
    ##     app: my-app
    ##
    selector: {}
  ## @param primary.extraVolumes Optionally specify extra list of additional volumes to the MariaDB Primary pod(s)
  ##
  extraVolumes: []
  ## @param primary.extraVolumeMounts Optionally specify extra list of additional volumeMounts for the MariaDB Primary container(s)
  ##
  extraVolumeMounts: []
  ## @param primary.initContainers Add additional init containers for the MariaDB Primary pod(s)
  ##
  initContainers: []
  ## @param primary.sidecars Add additional sidecar containers for the MariaDB Primary pod(s)
  ##
  sidecars: []
  ## MariaDB Primary Service parameters
  ##
  service:
    ## @param primary.service.type MariaDB Primary Kubernetes service type
    ##
    type: ClusterIP
    ## @param primary.service.ports.mysql MariaDB Primary Kubernetes service port
    ##
    ports:
      mysql: 3306
    ## @param primary.service.nodePorts.mysql MariaDB Primary Kubernetes service node port
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ##
    nodePorts:
      mysql: ""
    ## @param primary.service.clusterIP MariaDB Primary Kubernetes service clusterIP IP
    ##
    clusterIP: ""
    ## @param primary.service.loadBalancerIP MariaDB Primary loadBalancerIP if service type is `LoadBalancer`
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
    loadBalancerIP: ""
    ## @param primary.service.externalTrafficPolicy Enable client source IP preservation
    ## ref https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip
    ##
    externalTrafficPolicy: Cluster
    ## @param primary.service.loadBalancerSourceRanges Address that are allowed when MariaDB Primary service is LoadBalancer
    ## https://kubernetes.io/docs/tasks/access-application-cluster/configure-cloud-provider-firewall/#restrict-access-for-loadbalancer-service
    ## E.g.
    ## loadBalancerSourceRanges:
    ##   - 10.10.10.0/24
    ##
    loadBalancerSourceRanges: []
    ## @param primary.service.extraPorts Extra ports to expose (normally used with the `sidecar` value)
    ##
    extraPorts: []
    ## @param primary.service.annotations Provide any additional annotations which may be required
    ##
    annotations: {}
    ## @param primary.service.sessionAffinity Session Affinity for Kubernetes service, can be "None" or "ClientIP"
    ## If "ClientIP", consecutive client requests will be directed to the same Pod
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies
    ##
    sessionAffinity: None
    ## @param primary.service.sessionAffinityConfig Additional settings for the sessionAffinity
    ## sessionAffinityConfig:
    ##   clientIP:
    ##     timeoutSeconds: 300
    ##
    sessionAffinityConfig: {}
  ## MariaDB primary Pod Disruption Budget configuration
  ## ref: https://kubernetes.io/docs/tasks/run-application/configure-pdb/
  ##
  pdb:
    ## @param primary.pdb.create Enable/disable a Pod Disruption Budget creation for MariaDB primary pods
    ##
    create: false
    ## @param primary.pdb.minAvailable Minimum number/percentage of MariaDB primary pods that must still be available after the eviction
    ##
    minAvailable: 1
    ## @param primary.pdb.maxUnavailable Maximum number/percentage of MariaDB primary pods that can be unavailable after the eviction
    ##
    maxUnavailable: ""
  ## @param primary.revisionHistoryLimit Maximum number of revisions that will be maintained in the StatefulSet
  ##
  revisionHistoryLimit: 10

## @section MariaDB Secondary parameters
##

## Mariadb Secondary parameters
##
secondary:
  ## @param secondary.name Name of the secondary database (eg secondary, slave, ...)
  ##
  name: secondary
  ## @param secondary.replicaCount Number of MariaDB secondary replicas
  ##
  ### secondary 사용 안함.
  replicaCount: 0 
  ## @param secondary.command Override default container command on MariaDB Secondary container(s) (useful when using custom images)
  ##
  command: []
  ## @param secondary.args Override default container args on MariaDB Secondary container(s) (useful when using custom images)
  ##
  args: []
  ## @param secondary.lifecycleHooks for the MariaDB Secondary container(s) to automate configuration before or after startup
  ##
  lifecycleHooks: {}
  ## @param secondary.hostAliases Add deployment host aliases
  ## https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/
  ##
  hostAliases: []
  ## @param secondary.configuration [string] MariaDB Secondary configuration to be injected as ConfigMap
  ## ref: https://mysql.com/kb/en/mysql/configuring-mysql-with-mycnf/#example-of-configuration-file
  ##
  configuration: |-
    [mysqld]
    skip-name-resolve
    explicit_defaults_for_timestamp
    basedir=/opt/bitnami/mariadb
    port=3306
    socket=/opt/bitnami/mariadb/tmp/mysql.sock
    tmpdir=/opt/bitnami/mariadb/tmp
    max_allowed_packet=16M
    bind-address=0.0.0.0
    pid-file=/opt/bitnami/mariadb/tmp/mysqld.pid
    log-error=/opt/bitnami/mariadb/logs/mysqld.log
    character-set-server=UTF8
    collation-server=utf8_general_ci
    slow_query_log=0
    slow_query_log_file=/opt/bitnami/mariadb/logs/mysqld.log
    long_query_time=10.0
    [client]
    port=3306
    socket=/opt/bitnami/mariadb/tmp/mysql.sock
    default-character-set=UTF8
    [manager]
    port=3306
    socket=/opt/bitnami/mariadb/tmp/mysql.sock
    pid-file=/opt/bitnami/mariadb/tmp/mysqld.pid
  ## @param secondary.existingConfigmap Name of existing ConfigMap with MariaDB Secondary configuration.
  ## NOTE: When it's set the 'configuration' parameter is ignored
  ##
  existingConfigmap: ""
  ## @param secondary.updateStrategy.type MariaDB secondary statefulset strategy type
  ## ref: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#update-strategies
  ##
  updateStrategy:
    ## StrategyType
    ## Can be set to RollingUpdate or OnDelete
    ##
    type: RollingUpdate
  ## @param secondary.rollingUpdatePartition Partition update strategy for Mariadb Secondary statefulset
  ## https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#partitions
  ##
  rollingUpdatePartition: ""
  ## @param secondary.podAnnotations Additional pod annotations for MariaDB secondary pods
  ## ref: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
  ##
  podAnnotations: {}
  ## @param secondary.podLabels Extra labels for MariaDB secondary pods
  ## ref: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
  ##
  podLabels: {}
  ## @param secondary.podAffinityPreset MariaDB secondary pod affinity preset. Ignored if `secondary.affinity` is set. Allowed values: `soft` or `hard`
  ## ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity
  ##
  podAffinityPreset: ""
  ## @param secondary.podAntiAffinityPreset MariaDB secondary pod anti-affinity preset. Ignored if `secondary.affinity` is set. Allowed values: `soft` or `hard`
  ## Ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity
  ##
  podAntiAffinityPreset: soft
  ## Mariadb Secondary node affinity preset
  ## Ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity
  ##
  nodeAffinityPreset:
    ## @param secondary.nodeAffinityPreset.type MariaDB secondary node affinity preset type. Ignored if `secondary.affinity` is set. Allowed values: `soft` or `hard`
    ##
    type: ""
    ## @param secondary.nodeAffinityPreset.key MariaDB secondary node label key to match Ignored if `secondary.affinity` is set.
    ## E.g.
    ## key: "kubernetes.io/e2e-az-name"
    ##
    key: ""
    ## @param secondary.nodeAffinityPreset.values MariaDB secondary node label values to match. Ignored if `secondary.affinity` is set.
    ## E.g.
    ## values:
    ##   - e2e-az1
    ##   - e2e-az2
    ##
    values: []
  ## @param secondary.affinity Affinity for MariaDB secondary pods assignment
  ## Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
  ## Note: podAffinityPreset, podAntiAffinityPreset, and  nodeAffinityPreset will be ignored when it's set
  ##
  affinity: {}
  ## @param secondary.nodeSelector Node labels for MariaDB secondary pods assignment
  ## Ref: https://kubernetes.io/docs/user-guide/node-selection/
  ##
  nodeSelector: {}
  ## @param secondary.tolerations Tolerations for MariaDB secondary pods assignment
  ## Ref: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
  ##
  tolerations: []
  ## @param secondary.topologySpreadConstraints Topology Spread Constraints for MariaDB secondary pods assignment
  ## ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-topology-spread-constraints/
  ## E.g.
  ## topologySpreadConstraints:
  ##   - maxSkew: 1
  ##     topologyKey: topology.kubernetes.io/zone
  ##     whenUnsatisfiable: DoNotSchedule
  ##
  topologySpreadConstraints: []
  ## @param secondary.priorityClassName Priority class for MariaDB secondary pods assignment
  ## Ref: https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/
  ##
  priorityClassName: ""
  ## @param secondary.schedulerName Name of the k8s scheduler (other than default)
  ## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
  ##
  schedulerName: ""
  ## @param secondary.podManagementPolicy podManagementPolicy to manage scaling operation of MariaDB secondary pods
  ## ref: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies
  ##
  podManagementPolicy: ""
  ## MariaDB secondary Pod security context
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod
  ## @param secondary.podSecurityContext.enabled Enable security context for MariaDB secondary pods
  ## @param secondary.podSecurityContext.fsGroup Group ID for the mounted volumes' filesystem
  ##
  podSecurityContext:
    enabled: true
    fsGroup: 1001
  ## MariaDB secondary container security context
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-container
  ## @param secondary.containerSecurityContext.enabled MariaDB secondary container securityContext
  ## @param secondary.containerSecurityContext.runAsUser User ID for the MariaDB secondary container
  ## @param secondary.containerSecurityContext.runAsNonRoot Set Controller container's Security Context runAsNonRoot
  ##
  containerSecurityContext:
    enabled: true
    runAsUser: 1001
    runAsNonRoot: true
  ## MariaDB secondary container's resource requests and limits
  ## ref: https://kubernetes.io/docs/user-guide/compute-resources/
  ## We usually recommend not to specify default resources and to leave this as a conscious
  ## choice for the user. This also increases chances charts run on environments with little
  ## resources, such as Minikube. If you do want to specify resources, uncomment the following
  ## lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  ## @param secondary.resources.limits The resources limits for MariaDB secondary containers
  ## @param secondary.resources.requests The requested resources for MariaDB secondary containers
  ##
  resources:
    ## Example:
    ## limits:
    ##    cpu: 100m
    ##    memory: 256Mi
    ##
    limits: {}
    ## Examples:
    ## requests:
    ##    cpu: 100m
    ##    memory: 256Mi
    ##
    requests: {}
  ## Configure extra options for MariaDB Secondary containers' liveness, readiness and startup probes
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes)
  ## @param secondary.startupProbe.enabled Enable startupProbe
  ## @param secondary.startupProbe.initialDelaySeconds Initial delay seconds for startupProbe
  ## @param secondary.startupProbe.periodSeconds Period seconds for startupProbe
  ## @param secondary.startupProbe.timeoutSeconds Timeout seconds for startupProbe
  ## @param secondary.startupProbe.failureThreshold Failure threshold for startupProbe
  ## @param secondary.startupProbe.successThreshold Success threshold for startupProbe
  ##
  startupProbe:
    enabled: false
    initialDelaySeconds: 120
    periodSeconds: 15
    timeoutSeconds: 5
    failureThreshold: 10
    successThreshold: 1
  ## Configure extra options for liveness probe
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes
  ## @param secondary.livenessProbe.enabled Enable livenessProbe
  ## @param secondary.livenessProbe.initialDelaySeconds Initial delay seconds for livenessProbe
  ## @param secondary.livenessProbe.periodSeconds Period seconds for livenessProbe
  ## @param secondary.livenessProbe.timeoutSeconds Timeout seconds for livenessProbe
  ## @param secondary.livenessProbe.failureThreshold Failure threshold for livenessProbe
  ## @param secondary.livenessProbe.successThreshold Success threshold for livenessProbe
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 120
    periodSeconds: 10
    timeoutSeconds: 1
    failureThreshold: 3
    successThreshold: 1
  ## @param secondary.readinessProbe.enabled Enable readinessProbe
  ## @param secondary.readinessProbe.initialDelaySeconds Initial delay seconds for readinessProbe
  ## @param secondary.readinessProbe.periodSeconds Period seconds for readinessProbe
  ## @param secondary.readinessProbe.timeoutSeconds Timeout seconds for readinessProbe
  ## @param secondary.readinessProbe.failureThreshold Failure threshold for readinessProbe
  ## @param secondary.readinessProbe.successThreshold Success threshold for readinessProbe
  ##
  readinessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 1
    failureThreshold: 3
    successThreshold: 1
  ## @param secondary.customStartupProbe Override default startup probe for MariaDB secondary containers
  ##
  customStartupProbe: {}
  ## @param secondary.customLivenessProbe Override default liveness probe for MariaDB secondary containers
  ##
  customLivenessProbe: {}
  ## @param secondary.customReadinessProbe Override default readiness probe for MariaDB secondary containers
  ##
  customReadinessProbe: {}
  ## @param secondary.startupWaitOptions Override default builtin startup wait check options for MariaDB secondary containers
  ## `bitnami/mariadb` Docker image has built-in startup check mechanism,
  ## which periodically checks if MariaDB service has started up and stops it
  ## if all checks have failed after X tries. Use these to control these checks.
  ## ref: https://github.com/bitnami/containers/tree/main/bitnami/mariadb/pull/240
  ## Example (with default options):
  ## startupWaitOptions:
  ##   retries: 300
  ##   waitTime: 2
  ##
  startupWaitOptions: {}
  ## @param secondary.extraFlags MariaDB secondary additional command line flags
  ## Can be used to specify command line flags, for example:
  ## E.g.
  ## extraFlags: "--max-connect-errors=1000 --max_connections=155"
  ##
  extraFlags: ""
  ## @param secondary.extraEnvVars Extra environment variables to be set on MariaDB secondary containers
  ## E.g.
  ## extraEnvVars:
  ##  - name: TZ
  ##    value: "Europe/Paris"
  ##
  extraEnvVars: []
  ## @param secondary.extraEnvVarsCM Name of existing ConfigMap containing extra env vars for MariaDB secondary containers
  ##
  extraEnvVarsCM: ""
  ## @param secondary.extraEnvVarsSecret Name of existing Secret containing extra env vars for MariaDB secondary containers
  ##
  extraEnvVarsSecret: ""
  ## Enable persistence using Persistent Volume Claims
  ## ref: https://kubernetes.io/docs/user-guide/persistent-volumes/
  ##
  persistence:
    ## @param secondary.persistence.enabled Enable persistence on MariaDB secondary replicas using a `PersistentVolumeClaim`
    ##
    enabled: true
    ## @param secondary.persistence.subPath Subdirectory of the volume to mount at
    ##
    subPath: ""
    ## @param secondary.persistence.storageClass MariaDB secondary persistent volume storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    storageClass: ""
    ## @param secondary.persistence.annotations MariaDB secondary persistent volume claim annotations
    ##
    annotations: {}
    ## @param secondary.persistence.accessModes MariaDB secondary persistent volume access Modes
    ##
    accessModes:
      - ReadWriteOnce
    ## @param secondary.persistence.size MariaDB secondary persistent volume size
    ##
    size: 8Gi
    ## @param secondary.persistence.selector Selector to match an existing Persistent Volume
    ## selector:
    ##   matchLabels:
    ##     app: my-app
    ##
    selector: {}
  ## @param secondary.extraVolumes Optionally specify extra list of additional volumes to the MariaDB secondary pod(s)
  ##
  extraVolumes: []
  ## @param secondary.extraVolumeMounts Optionally specify extra list of additional volumeMounts for the MariaDB secondary container(s)
  ##
  extraVolumeMounts: []
  ## @param secondary.initContainers Add additional init containers for the MariaDB secondary pod(s)
  ##
  initContainers: []
  ## @param secondary.sidecars Add additional sidecar containers for the MariaDB secondary pod(s)
  ##
  sidecars: []
  ## MariaDB Secondary Service parameters
  ##
  service:
    ## @param secondary.service.type MariaDB secondary Kubernetes service type
    ##
    type: ClusterIP
    ## @param secondary.service.ports.mysql MariaDB secondary Kubernetes service port
    ##
    ports:
      mysql: 3306
    ## @param secondary.service.nodePorts.mysql MariaDB secondary Kubernetes service node port
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
    ##
    nodePorts:
      mysql: ""
    ## @param secondary.service.clusterIP MariaDB secondary Kubernetes service clusterIP IP
    ## e.g:
    ## clusterIP: None
    ##
    clusterIP: ""
    ## @param secondary.service.loadBalancerIP MariaDB secondary loadBalancerIP if service type is `LoadBalancer`
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
    ##
    loadBalancerIP: ""
    ## @param secondary.service.externalTrafficPolicy Enable client source IP preservation
    ## ref https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip
    ##
    externalTrafficPolicy: Cluster
    ## @param secondary.service.loadBalancerSourceRanges Address that are allowed when MariaDB secondary service is LoadBalancer
    ## https://kubernetes.io/docs/tasks/access-application-cluster/configure-cloud-provider-firewall/#restrict-access-for-loadbalancer-service
    ## E.g.
    ## loadBalancerSourceRanges:
    ##   - 10.10.10.0/24
    ##
    loadBalancerSourceRanges: []
    ## @param secondary.service.extraPorts Extra ports to expose (normally used with the `sidecar` value)
    ##
    extraPorts: []
    ## @param secondary.service.annotations Provide any additional annotations which may be required
    ##
    annotations: {}
    ## @param secondary.service.sessionAffinity Session Affinity for Kubernetes service, can be "None" or "ClientIP"
    ## If "ClientIP", consecutive client requests will be directed to the same Pod
    ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies
    ##
    sessionAffinity: None
    ## @param secondary.service.sessionAffinityConfig Additional settings for the sessionAffinity
    ## sessionAffinityConfig:
    ##   clientIP:
    ##     timeoutSeconds: 300
    ##
    sessionAffinityConfig: {}
  ## MariaDB secondary Pod Disruption Budget configuration
  ## ref: https://kubernetes.io/docs/tasks/run-application/configure-pdb/
  ##
  pdb:
    ## @param secondary.pdb.create Enable/disable a Pod Disruption Budget creation for MariaDB secondary pods
    ##
    create: false
    ## @param secondary.pdb.minAvailable Minimum number/percentage of MariaDB secondary pods that should remain scheduled
    ##
    minAvailable: 1
    ## @param secondary.pdb.maxUnavailable Maximum number/percentage of MariaDB secondary pods that may be made unavailable
    ##
    maxUnavailable: ""
  ## @param secondary.revisionHistoryLimit Maximum number of revisions that will be maintained in the StatefulSet
  ##
  revisionHistoryLimit: 10

## @section RBAC parameters
##

## MariaDB pods ServiceAccount
## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
##
serviceAccount:
  ## @param serviceAccount.create Enable the creation of a ServiceAccount for MariaDB pods
  ##
  #false 설정 
  create: false 
  ## @param serviceAccount.name Name of the created ServiceAccount
  ## If not set and create is true, a name is generated using the mariadb.fullname template
  ##
  name: ""
  ## @param serviceAccount.annotations Annotations for MariaDB Service Account
  ##
  annotations: {}
  ## @param serviceAccount.automountServiceAccountToken Automount service account token for the server service account
  ##
  automountServiceAccountToken: false
## Role Based Access
## ref: https://kubernetes.io/docs/admin/authorization/rbac/
##
rbac:
  ## @param rbac.create Whether to create and use RBAC resources or not
  ##
  create: false

## @section Volume Permissions parameters
##

## Init containers parameters:
## volumePermissions: Change the owner and group of the persistent volume mountpoint to runAsUser:fsGroup values from the securityContext section.
##
volumePermissions:
  ## @param volumePermissions.enabled Enable init container that changes the owner and group of the persistent volume(s) mountpoint to `runAsUser:fsGroup`
  ##
  enabled: false
  ## @param volumePermissions.image.registry Init container volume-permissions image registry
  ## @param volumePermissions.image.repository Init container volume-permissions image repository
  ## @param volumePermissions.image.tag Init container volume-permissions image tag (immutable tags are recommended)
  ## @param volumePermissions.image.digest Init container volume-permissions image digest in the way sha256:aa.... Please note this parameter, if set, will override the tag
  ## @param volumePermissions.image.pullPolicy Init container volume-permissions image pull policy
  ## @param volumePermissions.image.pullSecrets Specify docker-registry secret names as an array
  ##
  image:
    registry: docker.io
    repository: bitnami/bitnami-shell
    tag: 11-debian-11-r26
    digest: ""
    pullPolicy: IfNotPresent
    ## Optionally specify an array of imagePullSecrets (secrets must be manually created in the namespace)
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ## Example:
    ## pullSecrets:
    ##   - myRegistryKeySecretName
    ##
    pullSecrets: []
  ## @param volumePermissions.resources.limits Init container volume-permissions resource limits
  ## @param volumePermissions.resources.requests Init container volume-permissions resource requests
  ##
  resources:
    limits: {}
    requests: {}

## @section Metrics parameters
##

## Mysqld Prometheus exporter parameters
##
metrics:
  ## @param metrics.enabled Start a side-car prometheus exporter
  ##
  enabled: false
  ## @param metrics.image.registry Exporter image registry
  ## @param metrics.image.repository Exporter image repository
  ## @param metrics.image.tag Exporter image tag (immutable tags are recommended)
  ## @param metrics.image.digest Exporter image digest in the way sha256:aa.... Please note this parameter, if set, will override the tag
  ## @param metrics.image.pullPolicy Exporter image pull policy
  ## @param metrics.image.pullSecrets Specify docker-registry secret names as an array
  ##
  image:
    registry: docker.io
    repository: bitnami/mysqld-exporter
    tag: 0.14.0-debian-11-r26
    digest: ""
    pullPolicy: IfNotPresent
    ## Optionally specify an array of imagePullSecrets (secrets must be manually created in the namespace)
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ## Example:
    ## pullSecrets:
    ##   - myRegistryKeySecretName
    ##
    pullSecrets: []
  ## @param metrics.annotations [object] Annotations for the Exporter pod
  ##
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9104"
  ## @param metrics.extraArgs [object] Extra args to be passed to mysqld_exporter
  ## ref: https://github.com/prometheus/mysqld_exporter/
  ## E.g.
  ## - --collect.auto_increment.columns
  ## - --collect.binlog_size
  ## - --collect.engine_innodb_status
  ## - --collect.engine_tokudb_status
  ## - --collect.global_status
  ## - --collect.global_variables
  ## - --collect.info_schema.clientstats
  ## - --collect.info_schema.innodb_metrics
  ## - --collect.info_schema.innodb_tablespaces
  ## - --collect.info_schema.innodb_cmp
  ## - --collect.info_schema.innodb_cmpmem
  ## - --collect.info_schema.processlist
  ## - --collect.info_schema.processlist.min_time
  ## - --collect.info_schema.query_response_time
  ## - --collect.info_schema.tables
  ## - --collect.info_schema.tables.databases
  ## - --collect.info_schema.tablestats
  ## - --collect.info_schema.userstats
  ## - --collect.perf_schema.eventsstatements
  ## - --collect.perf_schema.eventsstatements.digest_text_limit
  ## - --collect.perf_schema.eventsstatements.limit
  ## - --collect.perf_schema.eventsstatements.timelimit
  ## - --collect.perf_schema.eventswaits
  ## - --collect.perf_schema.file_events
  ## - --collect.perf_schema.file_instances
  ## - --collect.perf_schema.indexiowaits
  ## - --collect.perf_schema.tableiowaits
  ## - --collect.perf_schema.tablelocks
  ## - --collect.perf_schema.replication_group_member_stats
  ## - --collect.slave_status
  ## - --collect.slave_hosts
  ## - --collect.heartbeat
  ## - --collect.heartbeat.database
  ## - --collect.heartbeat.table
  ##
  extraArgs:
    primary: []
    secondary: []
  ## @param metrics.extraVolumeMounts [object] Optionally specify extra list of additional volumeMounts for the MariaDB metrics container(s)
  ##
  extraVolumeMounts:
    primary: []
    secondary: []
  ## MariaDB metrics container Security Context
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-container
  ## @param metrics.containerSecurityContext.enabled Enable security context for MariaDB metrics container
  ## Example:
  ##   containerSecurityContext:
  ##     enabled: true
  ##     capabilities:
  ##       drop: ["NET_RAW"]
  ##     readOnlyRootFilesystem: true
  ##
  containerSecurityContext:
    enabled: false
  ## Mysqld Prometheus exporter resource requests and limits
  ## ref: https://kubernetes.io/docs/user-guide/compute-resources/
  ## We usually recommend not to specify default resources and to leave this as a conscious
  ## choice for the user. This also increases chances charts run on environments with little
  ## resources, such as Minikube. If you do want to specify resources, uncomment the following
  ## lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  ## @param metrics.resources.limits The resources limits for MariaDB prometheus exporter containers
  ## @param metrics.resources.requests The requested resources for MariaDB prometheus exporter containers
  ##
  resources:
    ## Example:
    ## limits:
    ##    cpu: 100m
    ##    memory: 256Mi
    ##
    limits: {}
    ## Examples:
    ## requests:
    ##    cpu: 100m
    ##    memory: 256Mi
    ##
    requests: {}
  ## Configure extra options for liveness probe
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes
  ## @param metrics.livenessProbe.enabled Enable livenessProbe
  ## @param metrics.livenessProbe.initialDelaySeconds Initial delay seconds for livenessProbe
  ## @param metrics.livenessProbe.periodSeconds Period seconds for livenessProbe
  ## @param metrics.livenessProbe.timeoutSeconds Timeout seconds for livenessProbe
  ## @param metrics.livenessProbe.failureThreshold Failure threshold for livenessProbe
  ## @param metrics.livenessProbe.successThreshold Success threshold for livenessProbe
  ##
  livenessProbe:
    enabled: true
    initialDelaySeconds: 120
    periodSeconds: 10
    timeoutSeconds: 1
    successThreshold: 1
    failureThreshold: 3
  ## Configure extra options for readiness probe
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#configure-probes
  ## @param metrics.readinessProbe.enabled Enable readinessProbe
  ## @param metrics.readinessProbe.initialDelaySeconds Initial delay seconds for readinessProbe
  ## @param metrics.readinessProbe.periodSeconds Period seconds for readinessProbe
  ## @param metrics.readinessProbe.timeoutSeconds Timeout seconds for readinessProbe
  ## @param metrics.readinessProbe.failureThreshold Failure threshold for readinessProbe
  ## @param metrics.readinessProbe.successThreshold Success threshold for readinessProbe
  ##
  readinessProbe:
    enabled: true
    initialDelaySeconds: 30
    periodSeconds: 10
    timeoutSeconds: 1
    successThreshold: 1
    failureThreshold: 3
  ## Prometheus Service Monitor
  ## ref: https://github.com/coreos/prometheus-operator
  ##
  serviceMonitor:
    ## @param metrics.serviceMonitor.enabled Create ServiceMonitor Resource for scraping metrics using PrometheusOperator
    ##
    enabled: false
    ## @param metrics.serviceMonitor.namespace Namespace which Prometheus is running in
    ##
    namespace: ""
    ## @param metrics.serviceMonitor.jobLabel The name of the label on the target service to use as the job name in prometheus.
    ##
    jobLabel: ""
    ## @param metrics.serviceMonitor.interval Interval at which metrics should be scraped
    ##
    interval: 30s
    ## @param metrics.serviceMonitor.scrapeTimeout Specify the timeout after which the scrape is ended
    ## e.g:
    ## scrapeTimeout: 30s
    ##
    scrapeTimeout: ""
    ## @param metrics.serviceMonitor.relabelings RelabelConfigs to apply to samples before scraping
    ## ref: https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#relabelconfig
    ##
    relabelings: []
    ## @param metrics.serviceMonitor.metricRelabelings MetricRelabelConfigs to apply to samples before ingestion
    ## ref: https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#relabelconfig
    ##
    metricRelabelings: []
    ## @param metrics.serviceMonitor.honorLabels honorLabels chooses the metric's labels on collisions with target labels
    ##
    honorLabels: false
    ## @param metrics.serviceMonitor.selector ServiceMonitor selector labels
    ## ref: https://github.com/bitnami/charts/tree/master/bitnami/prometheus-operator#prometheus-configuration
    ##
    ## selector:
    ##   prometheus: my-prometheus
    ##
    selector: {}
    ## @param metrics.serviceMonitor.labels Extra labels for the ServiceMonitor
    ##
    labels: {}
  ## Prometheus Operator PrometheusRule configuration
  ##
  prometheusRule:
    ## @param metrics.prometheusRule.enabled if `true`, creates a Prometheus Operator PrometheusRule (also requires `metrics.enabled` to be `true` and `metrics.prometheusRule.rules`)
    ##
    enabled: false
    ## @param metrics.prometheusRule.namespace Namespace for the PrometheusRule Resource (defaults to the Release Namespace)
    ##
    namespace: ""
    ## @param metrics.prometheusRule.additionalLabels Additional labels that can be used so PrometheusRule will be discovered by Prometheus
    ##
    additionalLabels: {}
    ## @param metrics.prometheusRule.rules Prometheus Rule definitions
    ##  - alert: MariaDB-Down
    ##    expr: absent(up{job="mariadb"} == 1)
    ##    for: 5m
    ##    labels:
    ##      severity: warning
    ##      service: mariadb
    ##    annotations:
    ##      message: 'MariaDB instance {{ `{{` }} $labels.instance {{ `}}` }} is down'
    ##      summary: MariaDB instance is down
    ##
    rules: []

## @section NetworkPolicy parameters
##

## Add networkpolicies
##
networkPolicy:
  ## @param networkPolicy.enabled Enable network policies
  ##
  enabled: false
  ## @param networkPolicy.metrics.enabled Enable network policy for metrics (prometheus)
  ## @param networkPolicy.metrics.namespaceSelector [object] Monitoring namespace selector labels. These labels will be used to identify the prometheus' namespace.
  ## @param networkPolicy.metrics.podSelector [object] Monitoring pod selector labels. These labels will be used to identify the Prometheus pods.
  ##
  metrics:
    enabled: false
    ## e.g:
    ## podSelector:
    ##   label: monitoring
    ##
    podSelector: {}
    ## e.g:
    ## namespaceSelector:
    ##   label: monitoring
    ##
    namespaceSelector: {}
  ## @param networkPolicy.ingressRules.primaryAccessOnlyFrom.enabled Enable ingress rule that makes primary mariadb nodes only accessible from a particular origin.
  ## @param networkPolicy.ingressRules.primaryAccessOnlyFrom.namespaceSelector [object] Namespace selector label that is allowed to access the primary node. This label will be used to identified the allowed namespace(s).
  ## @param networkPolicy.ingressRules.primaryAccessOnlyFrom.podSelector [object] Pods selector label that is allowed to access the primary node. This label will be used to identified the allowed pod(s).
  ## @param networkPolicy.ingressRules.primaryAccessOnlyFrom.customRules [object] Custom network policy for the primary node.
  ## @param networkPolicy.ingressRules.secondaryAccessOnlyFrom.enabled Enable ingress rule that makes primary mariadb nodes only accessible from a particular origin.
  ## @param networkPolicy.ingressRules.secondaryAccessOnlyFrom.namespaceSelector [object] Namespace selector label that is allowed to acces the secondary nodes. This label will be used to identified the allowed namespace(s).
  ## @param networkPolicy.ingressRules.secondaryAccessOnlyFrom.podSelector [object] Pods selector label that is allowed to access the secondary nodes. This label will be used to identified the allowed pod(s).
  ## @param networkPolicy.ingressRules.secondaryAccessOnlyFrom.customRules [object] Custom network policy for the secondary nodes.
  ##
  ingressRules:
    ## Allow access to the primary node only from the indicated:
    ##
    primaryAccessOnlyFrom:
      enabled: false
      ## e.g:
      ## namespaceSelector:
      ##   label: ingress
      ##
      namespaceSelector: {}
      ## e.g:
      ## podSelector:
      ##   label: access
      ##
      podSelector: {}
      ## custom ingress rules
      ## e.g:
      ## customRules:
      ##   - from:
      ##       - namespaceSelector:
      ##           matchLabels:
      ##             label: example
      ##
      customRules: {}

    ## Allow access to the secondary node only from the indicated:
    ##
    secondaryAccessOnlyFrom:
      enabled: false
      ## e.g:
      ## namespaceSelector:
      ##   label: ingress
      ##
      namespaceSelector: {}
      ## e.g:
      ## podSelector:
      ##   label: access
      ##
      podSelector: {}
      ## custom ingress rules
      ## e.g:
      ## CustomRules:
      ##   - from:
      ##       - namespaceSelector:
      ##           matchLabels:
      ##             label: example
      ##
      customRules: {}

  ## @param networkPolicy.egressRules.denyConnectionsToExternal Enable egress rule that denies outgoing traffic outside the cluster, except for DNS (port 53).
  ## @param networkPolicy.egressRules.customRules [object] Custom network policy rule
  ##
  egressRules:
    # Deny connections to external. This is not compatible with an external database.
    denyConnectionsToExternal: false
    ## Additional custom egress rules
    ## e.g:
    ## customRules:
    ##   - to:
    ##       - namespaceSelector:
    ##           matchLabels:
    ##             label: example
    ##
    customRules: {}  

```

<br/>

pod를 조회하여 running 상태를 확인 한다.  

```bash
root@newedu:~# kubectl get po
NAME                              READY   STATUS    RESTARTS   AGE
my-release-mariadb-0              1/1     Running   0          2m17s
```  

<br/>

해당 pod에 들어가서 DB 에 접속해 본다.  (비밀번호 edu1234 )  


```bash
root@newedu:~# kubectl exec -it my-release-mariadb-0 sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
$ mysql -u edu1 -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 34
Server version: 10.6.9-MariaDB Source distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| edu               |
| information_schema |
| test               |
+--------------------+
3 rows in set (0.002 sec)
```  

<br/>

mariadb에서 exit 하고 pod 안에서  아래 폴더를 조회해 본다.  
data 폴더에 보면 db 관련 화일이 생성 되어 있다.  

```bash
MariaDB [(none)]> exit
Bye
$ ls /bitnami/mariadb/data
aria_log.00000001  ddl_recovery.log  ib_buffer_pool  ibdata1  multi-master.info  mysql_upgrade_info  sys
aria_log_control   edu1		     ib_logfile0     ibtmp1   mysql		 performance_schema  test
```  

<br/>

nfs 서버에 들어가면 아래와 같이 폴더가 생성 된것을 확인 할 수 있다. ( 워커노드에서 조회 )
DB를 재기동 하더라도 데이터는 남아 있다.  

```bash
[root@edu data]# ls /mnt/database/edu1/my-mariadb/data
aria_log.00000001  ddl_recovery.log  ib_buffer_pool  ibdata1  multi-master.info  mysql_upgrade_info  sys
aria_log_control   edu1              ib_logfile0     ibtmp1   mysql              performance_schema  test
```  

<br/>

#### 1.2.6 Dynamic Provisioning 

<br/>

Static Provisioning 의 문제는 인프라에서 pv를 수동으로 생성을 해야 하고 업무팀에서 그에 맡게 pvc를 생성을 하는 번거로움이 있다.  

Dynamic Provisioning 은  프로비저너를 사용하여 자동으로  pvc/pv를 생성 할 수 있다.   

이번 예제는 NFS 서버를 위한 프로지저너를 사용하여 Dynamic Provisioning을 실습 한다.  

쿠버네티스에는 내장 NFS 프로비저너가 없다. NFS를 위한 스토리지클래스를 생성하려면 외부 프로비저너를 사용해야 한다. 쿠버네티스 1.19 이상에서 지원 한다.   

<br/>

NFS subdir external provisioner  

NFS 서버를 사용하여 PVC를 통해 Kubernetes PV의 동적 프로비저닝을 지원하는 자동 프로비저닝 도구입니다.  

- PVC(Persistent Volume Claim)에 대한 쿠버네티스 PV(Persistent Volume)를동적으로 프로비저닝하기 위하여 사전에 구성된 NFS server를 사용하는 automatic provisioner이다.  

- PV는 네이밍 규칙에 맞게 프로비저닝된다.  

<br/>

참고 
- https://1week.tistory.com/114 ( Native K8S deployment )
- https://kubepia.github.io/cloudpak/cp4app/install/ocp04.html ( OKD helm)

<br/>

우리는 helm (3.x) 을 사용하여 배포하도록 한다.  
기존 helm chart를 수정하기 위하여
https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/blob/master/charts/nfs-subdir-external-provisioner  링크에서 values.yaml 화일의 내용을 복사한다.  


복사한 내용을 vi dynamic_values.yaml 이라는 화일을 생성하여 붙여 넣기 한다.  

```bash
root@newedu:~# vi dynamic_values.yaml
```  

<br/>

dynamic_values.yaml 에서 변경 내용    
- nfs server: 172.25.1.162
- nfs path : /share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5/database
- reclaimPolicy: Retain
- archiveOnDelete: false ( 삭제시 아키이브 폴더를 만들지 않는다.)
- podSecurityPolicy:  ( OKD 인 경우만 사용 )
  enabled: true
- podSecurityContext:  ( OKD 인 경우 만 사용 )
  fsGroup: 1000660000 

<br/>

dynamic_values.yaml
```bash
replicaCount: 1
strategyType: Recreate

image:
  repository: k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner
  tag: v4.0.2
  pullPolicy: IfNotPresent
imagePullSecrets: []

nfs:
  server: 172.25.1.162
  path: /share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5/database
  mountOptions:
  volumeName: nfs-subdir-external-provisioner-root
  # Reclaim policy for the main nfs volume
  reclaimPolicy: Retain

# For creating the StorageClass automatically:
storageClass:
  create: true

  # Set a provisioner name. If unset, a name will be generated.
  # provisionerName:

  # Set StorageClass as the default StorageClass
  # Ignored if storageClass.create is false
  defaultClass: false

  # Set a StorageClass name
  # Ignored if storageClass.create is false
  name: nfs-client

  # Allow volume to be expanded dynamically
  allowVolumeExpansion: true

  # Method used to reclaim an obsoleted volume
  reclaimPolicy: Retain

  # When set to false your PVs will not be archived by the provisioner upon deletion of the PVC.
  archiveOnDelete: false

  # If it exists and has 'delete' value, delete the directory. If it exists and has 'retain' value, save the directory.
  # Overrides archiveOnDelete.
  # Ignored if value not set.
  onDelete:

  # Specifies a template for creating a directory path via PVC metadata's such as labels, annotations, name or namespace.
  # Ignored if value not set.
  pathPattern:

  # Set access mode - ReadWriteOnce, ReadOnlyMany or ReadWriteMany
  accessModes: ReadWriteOnce

  # Set volume bindinng mode - Immediate or WaitForFirstConsumer
  volumeBindingMode: Immediate

  # Storage class annotations
  annotations: {}

leaderElection:
  # When set to false leader election will be disabled
  enabled: true

## For RBAC support:
rbac:
  # Specifies whether RBAC resources should be created
  create: true

# If true, create & use Pod Security Policy resources
# https://kubernetes.io/docs/concepts/policy/pod-security-policy/
podSecurityPolicy:
  enabled: true

# Deployment pod annotations
podAnnotations: {}

## Set pod priorityClassName
# priorityClassName: ""

podSecurityContext:
  fsGroup: 1000660000 #1001

securityContext: {}

serviceAccount:
  # Specifies whether a ServiceAccount should be created
  create: true

  # Annotations to add to the service account
  annotations: {}

  # The name of the ServiceAccount to use.
  # If not set and create is true, a name is generated using the fullname template
  name:

resources: {}
  # limits:
  #  cpu: 100m
  #  memory: 128Mi
  # requests:
  #  cpu: 100m
  #  memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}

# Additional labels for any resource created
labels: {}
```  

<br/>

이제 설치를 합니다. 프로비저너 설치는 Cluster 권한이 필요 합니다.  ( 교육생들은 불필요. 강사가 사전에 진행함 )  

helm repository를 추가합니다.  

```bash
root@newedu:~# helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
root@newedu:~# helm repo list
NAME                           	URL
bitnami                        	https://charts.bitnami.com/bitnami
nfs-subdir-external-provisioner	https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```  

helm 을 사용하여 NFS subdir external provisioner 를 설치 합니다.  
수정한 dynamic_values.yaml 를 사용합니다.  

```bash
root@newedu:~# helm install nfs-subdir-external-provisioner -f dynamic_values.yaml  nfs-subdir-external-provisioner/nfs-subdir-external-provisioner -n devops
NAME: nfs-subdir-external-provisioner
LAST DEPLOYED: Tue Aug 30 14:52:58 2022
NAMESPACE: devops
STATUS: deployed
REVISION: 1
TEST SUITE: None
root@newedu:~# helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
root@newedu:~# helm list -n devops
NAME                           	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                                 	APP VERSION
nfs-subdir-external-provisioner	devops   	1       	2022-08-30 14:52:58.711543994 +0900 KST	deployed	nfs-subdir-external-provisioner-4.0.17	4.0.2
```  

<br/>
storage class가 생성 되었는지 확인합니다.

```bash
root@newedu:~# kubectl get storageclass
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Retain          Immediate           true                   95s
```    

오픈 쉬프트는 신규 생성된  service account에 hostmount 권한을 주어야 합니다. ( native K8S는 불필요 )    

```bash  
root@newedu:~# kubectl get sa -n devops
NAME                              SECRETS   AGE
builder                           2         35d
default                           2         35d
deployer                          2         35d
nfs-subdir-external-provisioner   2         3m34s
root@newedu:~# oc adm policy add-scc-to-user hostmount-anyuid -z nfs-subdir-external-provisioner -n devops
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:hostmount-anyuid added: "nfs-subdir-external-provisioner"
```  

<br/>
 
이제 mariadb를 다이나믹 프로비저닝을 사용하여 생성 해보도록 하겠습니다.  

상단의 global storageClass에 nfs-client를 추가합니다.    

values.yaml 변경 사항  
- global:
  imageRegistry: ""
  imagePullSecrets: []
  storageClass: "nfs-client"
- subPath: my-mariadb

<br/>

values.yaml 을 수정을 하면 helm 으로 설치를 합니다.    


```bash
root@newedu:~# helm install my-release -f values.yaml bitnami/mariadb -n edu1
NAME: my-release
LAST DEPLOYED: Tue Aug 30 16:11:46 2022
NAMESPACE: edu1
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mariadb
CHART VERSION: 11.2.1
APP VERSION: 10.6.9
```  

<br/>

pod , pvc , statefulset 이 정상적으로 생성이 되었는지 확인 합니다.  
pv도 생성이 되었고 cluster 권한으로 조회 가능합니다.  


```bash
root@newedu:~# kubectl get po
NAME                              READY   STATUS    RESTARTS   AGE
my-release-mariadb-0              1/1     Running   0          67s
root@newedu:~# kubectl get pvc
NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-my-release-mariadb-0   Bound    pvc-1e585a5a-d211-4b1a-ba17-6e0f07dfa99c   8Gi        RWO            nfs-client     84s
mariadb-pvc                 Bound    mariadb-pv1                                8Gi        RWX                           4h53m
root@newedu:~# kubectl get storageclass
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Retain          Immediate           true                   80m
root@newedu:~# kubectl get statefulset
NAME                 READY   AGE
my-release-mariadb   1/1     2m27s
```  

<br/>

NFS 서버에 접속 ( 워커노드로 접속 ) 하여 폴더가 생성된 것을 확인 할 수 있습니다.  

```bash
[root@edu database]# ls
edu1  edu1-data-my-release-mariadb-0-pvc-1e585a5a-d211-4b1a-ba17-6e0f07dfa99c  edu2
[root@edu database]# ls edu1-data-my-release-mariadb-0-pvc-1e585a5a-d211-4b1a-ba17-6e0f07dfa99c
my-mariadb
```  

<br/>

## 2. NFS 라이브러리 설치 ( Native Kubernetes )

<br/>

OKD 의 경우  Fedora CoreOS로 되어 있어 별도 라이브러리 설치 없이 hostmount 권한을 할당 하면  워커노드의 pod에서  nfs 연결 가능 하지만 native k8s의 경우 os에 따라 아래 라이브러리 설치 필요.  

<br/>

### centos 인 경우 아래 라이브러리 설치

<br/>


```bash
yum install nfs-utils
systemctl enable rpcbind --now ; systemctl status rpcbind
```  

<br/>

### ubuntu 인 경우 아래 라이브러리 설치
 
<br/>


```bash
apt-get update
apt-get install -y nfs-common
```
<br/>


