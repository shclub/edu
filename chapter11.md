# Chapter 11 

KeyCloak 를 이용한 SSO ( Single Sign On  / Single Sign Out ) 를 구현해 본다.    

<br/>

1. KeyCloak 설치 및 실습 

2. keyCloak 를 이용한 오픈 소스 시스템 연동 ( Jenkins / ArgoCD , Airflow , Kibana 등 )  

3. SpringBoot Backend 연동

<br/>

## 1. KeyCloak 설치 및 실습  

<br>

참고
- https://velog.io/@juhyeon1114/keycloak-개념부터-실행까지
- https://tommypagy.tistory.com/441 
- https://velog.io/@freejia/keycloak-%EC%84%9C%EB%B2%84-%EB%A7%8C%EB%93%A4%EA%B8%B0
- https://medium.com/geekculture/integrate-keycloak-and-argocd-within-kubernetes-with-ease-6871c620d3b3

<br/>

<img src="./assets/keycloak1.png" style="width: 80%; height: auto;"/>  

<br/>

이번에는 Redhat 에서 개발한 오픈소스 IAM 솔루션(Identity and Access Management Solution)인 Keycloak에 대하여 다룹니다.  

<br/>

Keycloak 은 현대의 애플리케이션과 서비스에 초점을 둔 ID 및 접근 관리(Access Management)에 통합 인증(SSO)을 허용하는 오픈 소스 소프트웨어로 Kubernetes 또는 MSA 환경에 최적화 된 솔루션 입니다.  

<br/>

쉽게 말하면 인증(Authentification)과 인가(Authorization)를 쉽게 해주고 SSO(Single-Sign-On)를 가능하게 해주는 것 입니다.

<br/>

SaaS 솔루션으로는 OKTA , AWS 제품으로는 Cognito 가 있습니다.   

<br/>

기능  

- 표준 프로토콜 지원(OpenID Connect, OAuth 2.0, SAML)
- 통합 인증(Single Sign-On, SSO)
- 관리자 / 계정관리 콘솔 제공
- ID 중개 와 소셜 로그인 (OpenID, SAML, GitHub, Google 등)
- 사용자 UI 정의
- Client Adapters (다수의 플랫폼과 프로그래밍 언어가 사용 가능한 adapter)


<br/>

### 설치

<br/>

keycloak 은 DB로 postgresql를 사용을 합니다.    
우리 기초 과정에서 생성한 postgresql 있으면 해당 DB 를 사용하고 없으면 신규로 생성합니다.  

<br/>

> 이미 DB가 있으면 skip 합니다.  

VM에 로그인 한 후에 keycloak 폴더를 생성한다.  

<br/>

yaml 화일 참고 : https://github.com/shclub/keycloak

<br/>

```bash
root@newedu:~# mkdir -p keycloak
root@newedu:~# cd keycloak
``` 

<br/>

#### Storage 설정

<br/>


PostgreSQL 과 keycloak 가 사용하는 stroage를 위해 pv / pvc 를 생성해야 하며
사전에 NFS 에 접속하여 폴더를 생성한다. 


<br/>

postgresql 은 아래 폴더에 생성되어 있고 수강생은 본인의 폴더 직접 생성.

<br/>

```bash
[root@edu postgre]# pwd
/mnt/postgre
[root@edu postgre]# mkdir -p edu
```

<br/>

keycloak 용 폴더도 생성한다.

```bash
[root@edu keycloak]# pwd
/mnt/keycloak
[root@edu keycloak]# mkdir -p edu
...
```

<br/>

postgresql / keycloak 용 해당 폴더의 권한을 설정한다.

<br/>

worker node에서 mount 해서 폴더 권한을 주는 경우는 아래 처럼 설정하고   

`chown -R nfsnobody:nfsnobody edu`  

pod 내에서 nfs 연결해서 권한을 줄때는  nobody:nogroup 으로 준다.  

`chown -R nobody:nogroup edu`

<br/>


```bash
[root@edu postgre]# chown -R nfsnobody:nfsnobody edu
[root@edu postgre]# chmod 777 edu
[root@edu keycloak]# chown -R nfsnobody:nfsnobody edu
[root@edu keycloak]# chmod 777 edu
```  


<br/>

postgresql 용 PV 를 생성한다. 사이즈는 5G로 설정한다.

<br/>

```bash  
root@newedu:~/keycloak#  vi postgre_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgre-edu-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 5Gi
  nfs:
    path: /share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5/postgre/edu
    server: 172.25.1.162
  persistentVolumeReclaimPolicy: Retain
```

<br/>

PV를 생성하고 Status를 확인해보면 Available 로 되어 있는 것을 알 수 있습니다.  

<br/>

```bash
root@newedu:~/keycloak# kubectl apply -f  postgre_pv.yaml
```

<br/>

postgresql용  pvc 를 생성합니다. pvc 이름을 기억합니다.

<br/>

```bash
root@newedu:~/keycloak# vi postgre_pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgre-edu-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  volumeName: postgre-edu-pv
```
<br/>


keycloak 용 PV 를 생성한다. 사이즈는 5G로 설정한다.

<br/>

```bash  
root@newedu:~/keycloak#  vi keycloak_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: keycloak-edu-pv
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 5Gi
  nfs:
    path: /share_8c0fade2_649f_4ca5_aeaa_8fd57904f8d5/keycloak/edu
    server: 172.25.1.162
  persistentVolumeReclaimPolicy: Retain
```

<br/>

PV를 생성하고 Status를 확인해보면 Available 로 되어 있는 것을 알 수 있습니다.  

<br/>

```bash
root@newedu:~/keycloak# kubectl apply -f keycloak_pvc.yaml
```

<br/>

keycloak 용 pvc 를 생성합니다. pvc 이름을 기억합니다.

<br/>

```bash
root@newedu:~/keycloak# vi keycloak_pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: keycloak-edu-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  volumeName: keycloak-edu-pv
```

<br/>

PVC 를 생성할 때는 namespace ( 본인의 namespace ) 를 명시해야 합니다.  

PVC 생성을 확인 해보고 다시 PV를 확인해 보면 Status가 Bound 로 되어 있는 것을 알 수 있습니다.  이제 PV 와 PVC가 연결이 되었습니다.

<br/>

```bash
root@newedu:~/keycloak# kubectl apply -f keycloak_pvc.yaml
```

<br/>

default service account 의 권한이 없으면 아래와 같이 권한을 부여 한다.  

<br/>

```bash
root@newedu:~/keycloak# oc adm policy add-scc-to-user anyuid -z default -n edu31
root@newedu:~/keycloak# oc adm policy add-scc-to-user privileged -z default -n edu31
```


<br/>

#### Helm 으로 PostgreSQL 설치

<br/>

helm repo 업데이트를 합니다.  

```bash
root@newedu:~/keycloak# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jenkins" chart repository
...Successfully got an update from the "nfs-subdir-external-provisioner" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```  

<br/>


helm 에서 postgreSQL 를 검색합니다.

```bash
root@newedu:~/keycloak# helm search repo postgresql
NAME                 	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/postgresql   	12.2.6       	15.2.0     	PostgreSQL (Postgres) is an open source object-...
bitnami/postgresql-ha	11.2.0       	15.2.0     	This PostgreSQL cluster solution includes the P...
bitnami/supabase     	0.1.4        	0.23.2     	Supabase is an open source Firebase alternative...
```  

<br/>

bitnami/postgresql 차트에서 차트의 변수 값을 변경하기 위해 postgre_values.yaml 화일을 추출한다.

<br/>


```bash
root@newedu:~/keycloak#  helm show values bitnami/postgresql  > postgre_values.yaml
```

<br/>

postgre_values.yaml 를 수정한다.  

- 28 ,29,30,31 : 본인이 DB 계정 설정
- 646 : 본인의 pvc로 변경
- 669 : 5G로 사이즈 변경
- 694 : read replica 0 ( primary db만 사용 , readReplicas 는 사용 안함)

<br/>

```bash
  27     auth:
  28       postgresPassword: "New1234!"
  29       username: "keycloak"
  30       password: "New1234!"
  31       database: "keycloak"
  32       existingSecret: ""
 ... 
 640   persistence:
 641     ## @param primary.persistence.enabled Enable PostgreSQL Primary data persistence using PVC
 642     ##
 643     enabled: true
 644     ## @param primary.persistence.existingClaim Name of an existing PVC to use
 645     ##
 646     existingClaim: "postgre-edu-pvc"
 647     ## @param primary.persistence.mountPath The path the volume will be mounted at
 648     ## Note: useful when using custom PostgreSQL images
 649     ##
 650     mountPath: /bitnami/postgresql
 651     ## @param primary.persistence.subPath The subdirectory of the volume to mount to
 652     ## Useful in dev environments and one PV for multiple services
 653     ##
 654     subPath: ""
 655     ## @param primary.persistence.storageClass PVC Storage Class for PostgreSQL Primary data volume
 656     ## If defined, storageClassName: <storageClass>
 657     ## If set to "-", storageClassName: "", which disables dynamic provisioning
 658     ## If undefined (the default) or set to null, no storageClassName spec is
 659     ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
 660     ##   GKE, AWS & OpenStack)
 661     ##
 662     storageClass: ""
 663     ## @param primary.persistence.accessModes PVC Access Mode for PostgreSQL volume
 664     ##
 665     accessModes:
 666       - ReadWriteOnce
 667     ## @param primary.persistence.size PVC Storage Request for PostgreSQL volume
 668     ##
 669     size: 5Gi
...
 688 readReplicas:
 689   ## @param readReplicas.name Name of the read replicas database (eg secondary, slave, ...)
 690   ##
 691   name: read
 692   ## @param readReplicas.replicaCount Number of PostgreSQL read only replicas
 693   ##
 694   replicaCount: 0
 ```

<br/>

이제 postgreSQL DB를 설치 합니다.

<br/>

```bash
root@newedu:~/keycloak# helm install keycloak-postgre bitnami/postgresql -f postgre_values.yaml
NAME: keycloak-postgre
LAST DEPLOYED: Mon Mar 27 10:12:46 2023
NAMESPACE: edu30
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 12.2.6
APP VERSION: 15.2.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    keycloak-postgre-postgresql.edu30.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_ADMIN_PASSWORD=$(kubectl get secret --namespace edu30 keycloak-postgre-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To get the password for "edu" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace edu30 keycloak-postgre-postgresql -o jsonpath="{.data.password}" | base64 -d)

To connect to your database run the following command:

    kubectl run keycloak-postgre-postgresql-client --rm --tty -i --restart='Never' --namespace edu30 --image docker.io/bitnami/postgresql:15.2.0-debian-11-r14 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host keycloak-postgre-postgresql -U edu -d edu -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace edu30 svc/keycloak-postgre-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U edu -d edu -p 5432

WARNING: The configured password will be ignored on new installation in case when previous Posgresql release was deleted through the helm command. In that case, old PVC will have an old password, and setting it through helm won't take effect. Deleting persistent volumes (PVs) will solve the issue.
```

<br/>

pod를 확인한다.

```bash
root@newedu:~/keycloak# kubectl get po
NAME                                        READY   STATUS    RESTARTS   AGE
keycloak-postgre-postgresql-0                  1/1     Running   0          82s
```

<br/>

NFS 서버에 접속하여 data 폴더가 생성 되었는지 확인한다.  

```bash
[root@edu edu]# pwd
/mnt/postgre/edu
[root@edu edu]# ls data
PG_VERSION  pg_commit_ts   pg_logical    pg_replslot   pg_stat      pg_tblspc    pg_xact               postmaster.pid
base        pg_dynshmem    pg_multixact  pg_serial     pg_stat_tmp  pg_twophase  postgresql.auto.conf
global      pg_ident.conf  pg_notify     pg_snapshots  pg_subtrans  pg_wal       postmaster.opts
```  

<br/><br/>

PostgreSQL DB 가 이미 존재하면 아래와 같이 DB 이름과 계정을 추가합니다.    

<br/>

POD를 조회 해보고  PostgreSQL 해당 POD에 shell 로 들어갑니다.

<br/>

```bash
root@newedu:~/keycloak# kubectl get po
NAME                                        READY   STATUS             RESTARTS   AGE
sonar-postgre-postgresql-0                  1/1     Running            0          2d20h
sonarqube-5d48b66455-ktn7m                  0/1     CrashLoopBackOff   790        2d19h
root@newedu:~/keycloak# kubectl exec -it sonar-postgre-postgresql-0  sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
```  

<br/>

postgres 유저로 로그인 합니다.  

<br/>

```bash
$ psql -U postgres;
Password for user postgres:
psql (15.2)
Type "help" for help.
```  
<br/>

keycloak 용 DB를 생성하고 아이디/비밀번호를 설정합니다.   
추가적으로 role을 할당합니다. ( 일단 SUPERSUER로 설정합니다. )

<br/>

```bash
postgres=# CREATE DATABASE keycloak;
CREATE DATABASE
postgres=# CREATE USER keycloak WITH PASSWORD 'New1234!';
CREATE ROLE
postgres=# ALTER USER keycloak WITH SUPERUSER;
ALTER ROLE
``` 

<br/>

#### Helm KeyCloak  설정

<br/>

PostgreSQL 과 KeyCloak 은 Helm Chart 를 이용하여 설치를 합니다.  

<br/>

현재 로컬의 helm repository 를 확인한다.   

<br/>

```bash
root@newedu:~/keycloak# helm repo list
NAME                           	URL
bitnami                        	https://charts.bitnami.com/bitnami
nfs-subdir-external-provisioner	https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```  

<br/>

우리는 bitnami chart를 사용할 예정이기 때문에 없으면 아래와 같이 추가한다.  

<br/>

```bash
root@newedu:~/keycloak# helm repo add bitnami https://charts.bitnami.com/bitnami --insecure-skip-tls-verify
"bitnami" has been added to your repositories
```

<br/>

helm repository를 update 한다.  

<br/>

```bash
root@newedu:~/keycloak# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "nfs-subdir-external-provisioner" chart repository
...Successfully got an update from the "jenkins" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
```  

<br/>

keycloak helm reppository 에서 helm chart를 검색을 하고 keycloak chart를 선택합니다.  

<br/>

```bash
root@newedu:~/keycloak# helm search repo keycloak
NAME            	CHART VERSION	APP VERSION	DESCRIPTION
bitnami/keycloak	13.4.0       	20.0.5     	Keycloak is a high performance Java-based ident...
```

<br/>

bitnami/keycloak 차트에서 차트의 변수 값을 변경하기 위해 keycloak_values.yaml 화일을 추출한다.

<br/>


```bash
root@newedu:~/keycloak#  helm show values bitnami/keycloak > keycloak_values.yaml
```


<br/>

vi 데이터에서 생성된 keycloak_values.yaml을 연다.  

<br/>

```bash
root@newedu:~/keycloak# vi keycloak_values.yaml
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
 103   ## @param auth.adminUser Keycloak administrator user
 104   ##
 105   adminUser: admin #user
 106   ## @param auth.adminPassword Keycloak administrator password for the new user
 107   ##
 108   adminPassword: "New1234!"
 109   ## @param auth.existingSecret Existing secret containing Keycloak admin password
 ...
 311 readinessProbe:
 312   enabled: false
 ...
 1002 postgresql:
 1003   enabled: false # true
 ...
 1021 externalDatabase:
 1022   host: "sonar-postgre-postgresql"
 1023   port: 5432
 1024   user: keycloak
 1025   database: keycloak
 1026   password: "New1234!"
 1027   existingSecret: ""
```  
<br/>

#### Helm 으로 keycloak 설치

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

keycloak route 를 아래 처럼 생성한다.  

```bash
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: keycloak
spec:
  host: keycloak-edu30.apps.211-34-231-82.nip.io
  port:
    targetPort: http
  tls:
    insecureEdgeTerminationPolicy: Redirect
    termination: edge
  to:
    kind: Service
    name: keycloak
    weight: 100
  wildcardPolicy: None
```  

<br/>

```bash
root@newedu:~/keycloak# kubectl apply -f keycloak_route.yaml
route.route.openshift.io/keycloak created
root@newedu:~/keycloak# kubectl get route
NAME       HOST/PORT                                  PATH   SERVICES   PORT   TERMINATION     WILDCARD
jenkins    jenkins-edu30.apps.211-34-231-82.nip.io           jenkins    http   edge/Redirect   None
keycloak   keycloak-edu30.apps.211-34-231-82.nip.io          keycloak   http   edge/Redirect   None
```  

<br/>


웹 브라우저 에서 본인의 keycloak 으로 접속한다.   
- 예 : https://keycloak-edu30.apps.211-34-231-82.nip.io



<br/>

### Keycloak 연동 설정

<br/>
keycloak에 접속하면 처음 아래 화면이 나온다.  

<img src="./assets/keycloak2.png" style="width: 80%; height: auto;"/>  

<br/>

Admin Console 로 이동하여 helm chart 에 설정한 admin 계정으로 로그인 한다.    

<img src="./assets/keycloak3.png" style="width: 60%; height: auto;"/>  

<br/>

Realm은 '영역', '왕국'이라는 뜻인데, Keycloak에서 가장 근본이 되는 데이터 타입이다.  

이 Realm 하위에 클라이언트들을 생성할 수 있고, 그 하위에 사용자들이 생성되게 된다.  

뭐 예를 들어서, 회사 A라는 Realm 하위에 서비스 A전용 클라이언트, 서비스 B전용클라이언트, 관리자 웹 전용클라이언트를 생성해서 운영할 수 있다.  

<br/>

- Realm: 접근 범위.
- Client: 애플리케이션 단위.
- User: Client에 인증을 요청하는 사용자 단위.
- Role: User에게 부여할 권한.


<br/>

기본으로 Master Realm 이 있다.

Realm은 1개 이상의 Client를 하나로 묶어서 관리할 수 있는 범위다. SSO 적용은 Realm 단위로 한다.    

User의 Role을 설정하면 사용자, 관리자 등으로 구분된 권한을 줄 수 있다.

<br/>

#### Realm 생성 하기

<br/>

Master realm: Keycloak을 만들자마자 생성되는 admin이 관리하는 realm.
Other realm: master realm의 admin이 만들어내는 realm.  

영역을 하나 생성한다.  왼쪽 상단의 master 를 선택하고 `Create Realm` 을 클릭한다.  

<img src="./assets/keycloak4.png" style="width: 60%; height: auto;"/>  

<br/>

edu 라는 이름으로 입력하고 create 버튼을 클릭하여 realm 을 생성합니다.

<img src="./assets/keycloak5.png" style="width: 60%; height: auto;"/>  

<br/>

#### Client 생성 하기

<br/>

왼편의 "Clients"를 클릭하면 기본 클라이언트들이 있고

우리는 ArgoCD 와 연결하기 위해 Create Client 버튼을 클릭합니다.  

<img src="./assets/keycloak6.png" style="width: 60%; height: auto;"/>  

<br/>

아래와 같이 설정하고 Next 버튼 클릭.

<img src="./assets/keycloak7.png" style="width: 60%; height: auto;"/>  

<br/>

SSO를 위해 Client Authentication 을 On 을 하고 저장합니다.

<img src="./assets/keycloak8.png" style="width: 60%; height: auto;"/>  

<br/>

- Client authentication: 생성할 클라이언트를 public하게 사용할지 말지를 정한다. 아마 거의 대부분의 경우 public하게 사용하지는 않을 것이므로, 대부분은 이 옵션은 활성화해주면 된다.
- Authorization: 활성화하면, 권한이나 규칙등등 다양한 기준을 통해서 사용자 Autorization(인가)기능을 제공한다.
- Authentication Flow: 사용자에게 어떤 인증(로그인)절차를 제공할지 설정한다.

<br/>

Valid redirect URIs , Valid post logout redirect URIs 에 ArgoCD URL을 넣어주고 저장합니다.  
- Valid redirect URI : https://argocd-argocd.apps.211-34-231-82.nip.io/auth/callback
- Valid post logout redirect URI : https://argocd-argocd.apps.211-34-231-82.nip.io/auth/callback

<img src="./assets/keycloak9.png" style="width: 80%; height: auto;"/>  

<br/>

#### Group/User 생성 하기

<br/>

왼쪽 프레임에서 Group을 선택하고 Create Group 버튼을 클릭합니다.

<img src="./assets/keycloak10.png" style="width: 80%; height: auto;"/>  


<br/>

`edu_grp` 라는 이름으로 Group 을 생성합니다. 
 이 그룹은 argocd 에서 group 으로 설정하여 사용됩니다.  

<img src="./assets/keycloak11.png" style="width: 80%; height: auto;"/>  


<br/>

Create New User 이름을 클릭한다.  

<img src="./assets/keycloak12.png" style="width: 80%; height: auto;"/>  

<br/>

`edu_user` 라는 이름으로 신규 유저를 생성하고 Join Group을 클릭하여 앞에서 생성한 Group을 선택한다.  

<img src="./assets/keycloak13.png" style="width: 80%; height: auto;"/>  

<br/>

신규 유저가 생성 되었고 Credentials tab 으로 이동한다.

<img src="./assets/keycloak14.png" style="width: 80%; height: auto;"/>  

<br/>

Credentials tab 에서 Set Password 버튼을 클릭한다.   

<img src="./assets/keycloak15.png" style="width: 80%; height: auto;"/>  

<br/>

비밀번호 설정을 하고 temporary는 disable 한다.  

<img src="./assets/keycloak16.png" style="width: 80%; height: auto;"/>  

<br/>

아래와 같이 User 설정이 완료가 되었다.

<img src="./assets/keycloak17.png" style="width: 80%; height: auto;"/>  

<br/>

## 2. keyCloak 를 이용한 오픈 소스 시스템 연동 ( Jenkins / ArgoCD , Airflow , Kibana 등 )    

<br>

### (ArgoCD 연동) Client Scope 만들고 “Groups” 을 ArgoCD Client ID 에 연결하기

<br/>

왼쪽 프레임에서 Client Scope 을 선택하고 Create Client Scope 버튼을 클릭합니다.

<img src="./assets/keycloak_clientscope1.png" style="width: 80%; height: auto;"/>  

<br/>

`groups` 라는 이름을 입력하고 저장합니다.

<img src="./assets/keycloak_clientscope2.png" style="width: 80%; height: auto;"/>  

<br/>

왼쪽 프레임에서 Clients를 선택하고 argco client를 클릭한 후에 Client scopes Tab으로 이동합니다.      

그리고 다음으로 Add client scope 메뉴를 클릭합니다.  

<img src="./assets/keycloak_clientscope3.png" style="width: 80%; height: auto;"/>  

<br/>

위에서 생성한 groups 를 체크하고 Default 를 선택한 후 Add 합니다.   

<img src="./assets/keycloak_clientscope4.png" style="width: 80%; height: auto;"/>  

<br/>

아래와 같이 client scope이 추가 된 것을 확인 할 수 있습니다.  

<img src="./assets/keycloak_clientscope5.png" style="width: 80%; height: auto;"/>  


<br/>

### (ArgoCD 연동) Client Scope 을 Mapper 에 연결하기

<br/>

왼쪽 프레임에서 Client Scope 을 선택하고 Create Client Scope 버튼을 클릭한 후 Mappers tab으로 이동합니다.   

Configure a new Mapper 를 클릭합니다.

<img src="./assets/keycloak_clientscope6.png" style="width: 80%; height: auto;"/>  

<br/>

Group Membership 을 선택을 합니다.

<img src="./assets/keycloak_clientscope7.png" style="width: 80%; height: auto;"/>  

<br/>

Add mapper 메뉴에서 Name 과 Token Claim Name 에 `groups` 를 입력하고 Full group path 를 off 하고 저장합니다.  

이 설정을 통하여 ArgoCD의 그룹과 연동이 가능 합니다.

<img src="./assets/keycloak_clientscope8.png" style="width: 80%; height: auto;"/>  

<br/>

### (ArgoCD 연동)  ArgoCD 설정

<br/>


ArgoCD Client 에 가서 Credential Tab으로 이동한다.  
Client Secret 항목을 확인 하고 오른쪽의 copy 버튼을 눌러 복사한다.  

<img src="./assets/keycloak18.png" style="width: 80%; height: auto;"/>  

<br/>

k8s secret으로 저장하기 위해 복사한 키를 base64로 변환한다.  

```bash
root@newedu:~/keycloak# echo -n 'RPG1wzTHVvcALs9wRFSrruQPQfoxadcw' | base64
UlBHMXd6VEhWdmNBTHM5d1JGU3JydVFQUWZveGFkY3c=
```  

<br/>

ArgoCD Secret에 저장한다. ( data 항목 아래 )    

```bash
root@newedu:~/keycloak# kubectl edit secret argocd-secret -n argocd
apiVersion: v1
data:
  oidc.keycloak.clientSecret: UlBHMXd6VEhWdmNBTHM5d1JGU3JydVFQUWZveGFkY3c=
```  

<br/>


ArgoCD configmap을 변경합니다. ( data 항목 아래 )    
- URL : argocd URL
- admin.enables : ArgoCD 화면에서 ArgoCD 로그인 창 유지/삭제
- oidc.config
  - name : keycloak
  - issuer : 신규 생성한 realm 추가
  - clientID : keycloak 에서 생성한 Client ID
  - clientSecret : 앞에 secret 으로 생성한 이름
  - requestedScopes : groups 를 추가한다. 앞에서 client scopes 로 생성
  - logoutURL : ArgoCD에서 logout시 keycloak 에서도 로그 아웃 ( Single Sign Out )

<br/>

```bash
root@newedu:~/keycloak# kubectl edit configmap argocd-cm -n argocd
apiVersion: v1
data:
  url: https://argocd-argocd.apps.211-34-231-82.nip.io
  admin.enabled: "true"
  oidc.config: |
      name: Keycloak
      issuer: https://keycloak-edu30.apps.211-34-231-82.nip.io/realms/edu
      clientID: argocd
      clientSecret: $oidc.keycloak.clientSecret
      requestedScopes: ['openid', 'profile', 'email', 'groups']
      logoutURL:  https://keycloak-edu30.apps.211-34-231-82.nip.io/realms/edu/protocol/openid-connect/logout?client_id=argocd&id_token_hint={{token}}&post_logout_redirect_uri={{logoutRedirectURL}}
```  


<br/>
