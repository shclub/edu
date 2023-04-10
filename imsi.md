curl -sfL https://get-kk.kubesphere.io | VERSION=v3.0.2 sh -

chmod +x kk




root@newedu:~/kubesphere# kubectl get nodes -o wide
NAME               STATUS   ROLES    AGE    VERSION                INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                        KERNEL-VERSION            CONTAINER-RUNTIME
edu.dmz-infra01    Ready    worker   281d   v1.20.0+bafe72f-1054   172.25.0.93    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.dmz-infra02    Ready    worker   281d   v1.20.0+bafe72f-1054   172.25.0.87    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.master01       Ready    master   281d   v1.20.0+bafe72f-1054   172.25.1.16    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.master02       Ready    master   281d   v1.20.0+bafe72f-1054   172.25.1.179   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.master03       Ready    master   281d   v1.20.0+bafe72f-1054   172.25.1.13    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.monitoring01   Ready    worker   281d   v1.20.0+bafe72f-1054   172.25.1.28    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.monitoring02   Ready    worker   281d   v1.20.0+bafe72f-1054   172.25.1.129   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker01       Ready    worker   281d   v1.20.0+bafe72f-1054   172.25.1.144   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker02       Ready    worker   281d   v1.20.0+bafe72f-1054   172.25.1.50    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker03       Ready    worker   281d   v1.20.0+bafe72f-1054   172.25.1.124   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker04       Ready    worker   281d   v1.20.0+bafe72f-1054   172.25.1.160   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker05       Ready    worker   250d   v1.20.0+bafe72f-1054   172.25.1.59    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker06       Ready    worker   202d   v1.20.0+bafe72f-1054   172.25.1.65    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker07       Ready    worker   202d   v1.20.0+bafe72f-1054   172.25.1.102   <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker08       Ready    worker   14d    v1.20.0+bafe72f-1054   172.25.1.58    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
edu.worker09       Ready    worker   13d    v1.20.0+bafe72f-1054   172.25.1.79    <none>        Fedora CoreOS 33.20210301.3.1   5.10.19-200.fc33.x86_64   cri-o://1.20.0
root@newedu:~/kubesphere# ./kk create cluster --with-kubernetes v1.22.12 --with-kubesphere v3.3.2
error: Unsupported KubeSphere version: v3.3.2
root@newedu:~/kubesphere# ./kk create cluster --with-kubernetes v1.20.0 --with-kubesphere v3.0.2
error: Unsupported KubeSphere version: v3.0.2
root@newedu:~/kubesphere# ./kk create cluster --with-kubernetes v1.20.0


 _   __      _          _   __
| | / /     | |        | | / /
| |/ / _   _| |__   ___| |/ /  ___ _   _
|    \| | | | '_ \ / _ \    \ / _ \ | | |
| |\  \ |_| | |_) |  __/ |\  \  __/ |_| |
\_| \_/\__,_|_.__/ \___\_| \_/\___|\__, |
                                    __/ |
                                   |___/

11:39:37 KST [GreetingsModule] Greetings
11:39:37 KST message: [newedu]
Greetings, KubeKey!
11:39:37 KST success: [newedu]
11:39:37 KST [NodePreCheckModule] A pre-check on nodes
11:39:37 KST success: [newedu]
11:39:37 KST [ConfirmModule] Display confirmation form
+--------+------+------+---------+----------+-------+-------+---------+-----------+--------+--------+------------+------------+-------------+------------------+--------------+
| name   | sudo | curl | openssl | ebtables | socat | ipset | ipvsadm | conntrack | chrony | docker | containerd | nfs client | ceph client | glusterfs client | time         |
+--------+------+------+---------+----------+-------+-------+---------+-----------+--------+--------+------------+------------+-------------+------------------+--------------+
| newedu | y    | y    | y       | y        |       |       |         |           |        | 23.0.1 | 1.6.18     | y          |             |                  | KST 11:39:37 |
+--------+------+------+---------+----------+-------+-------+---------+-----------+--------+--------+------------+------------+-------------+------------------+--------------+
11:39:37 KST [ERRO] newedu: conntrack is required.
11:39:37 KST [ERRO] newedu: socat is required.

This is a simple check of your environment.
Before installation, ensure that your machines meet all requirements specified at
https://github.com/kubesphere/kubekey#requirements-and-recommendations



#keycloak 에서 사용할 db 생성.

helm install postgresql bitnami/postgresql  \
--set auth.postgresPassword=New1234! \
--set auth.username=keycloak-user \
--set auth.password=new1234! \
--set auth.database=keycloak! \
--set primary.persistence.enabled=false \
--insecure-skip-tls-verify

#추후 볼륨 추가예정


#helm chart download

helm pull bitnami/keycloak --version 10.1.6 --insecure-skip-tls-verify


#압축해제

tar -xf ./keycloak-10.1.6.tgz


#value.yaml수정

cd keycloak

vi values.yaml

 

 

---- values.yaml 수정

global:

  imageRegistry: "nexus.dspace.kt.co.kr"
  imagePullSecrets: [dspace-nexus]

....

auth: #관리자 ID/PW
  adminUser: admin
  adminPassword: New1234!
....

readinessProbe:
  enabled: false
....

image:
  registry: nexus.dspace.kt.co.kr
  repository: icis/keycloak
  tag: 19.0.3-debian-11-r3
....
postgresql:
  enabled: false
....

externalDatabase:    #위 bitnami/postgresql 설정과 동일하게 세팅.

  host: "127.0.0.1"
  port: 5432
  user: keycloak-user
  database: keycloak
  password: "New1234!"

 

--helm install

 

#helm install keycloak

helm install keycloak bitnami/keycloak -f keycloak_values.yaml -n edu30 --insecure-skip-tls-verify

helm install keycloak . -f values.yaml -n keycloak-system

#helm upgrade keycloak
helm upgrade keycloak . -f values.yaml -n keycloak-system

 

#helm uninstall keycloak
helm -n keycloak-system uninstall keycloak

#권한 문제가 생길시 !

oc patch serviceaccount keycloak -p '{"imagePullSecrets": [{"name": "dspace-nexus"}]}' -n keycloak-system


oc adm policy add-scc-to-user anyuid -z  keycloak -n keycloak-system

oc adm policy add-scc-to-user privileged -z  keycloak -n keycloak-system


oc adm policy add-scc-to-user anyuid -z  postgresql -n keycloak-system
oc adm policy add-scc-to-user privileged -z  postgresql -n keycloak-system

 

--key cloak route

 
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: keycloak
  namespace: keycloak-system
  labels:
    app.kubernetes.io/component: keycloak
    app.kubernetes.io/instance: keycloak
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: keycloak
    helm.sh/chart: keycloak-10.1.6
spec:
  host: keycloak.apps.cluster01.cz-dev.icis.kt.co.kr
  path: /
  to:
    kind: Service
    name: keycloak
    weight: 100
  port:
    targetPort: http
  wildcardPolicy: None



  $ psql -d edu -U edu
Password for user edu:
psql (15.2)
Type "help" for help.

edu=> CREATE DATABASE keycloak;
CREATE DATABASE
edu=> CREATE user keycloak WITH PASSWORD 'New1234!';


GRANT {permissions} ON DATABASE {db_name} TO {user_name};

 

** 기본 계정 설정 변경 **

기본 계정인 postgres의 기본 암호는 없습니다. 만약 새로 계정을 생성하지 않고 이 기본 계정을 사용하려면 암호를 변경해야만 합니다. 

기본 계정의 암호는 다음과 같은 SQL로 변경할 수 있습니다.

ALTER USER postgres WITH PASSWORD '{new_pass}';


--------------------

root@newedu:~/keycloak# kubectl get po -n edu30
NAME                                        READY   STATUS             RESTARTS   AGE
sonar-postgre-postgresql-0                  1/1     Running            0          2d20h
sonarqube-5d48b66455-ktn7m                  0/1     CrashLoopBackOff   790        2d19h
root@newedu:~/keycloak# kubectl exec -it sonar-postgre-postgresql-0  sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

--------------------
$ psql -U postgres;
Password for user postgres:
psql (15.2)
Type "help" for help.

postgres=# CREATE DATABASE keycloak;
ERROR:  database "keycloak" already exists
postgres=# DROP DATABASE keycloak;
DROP DATABASE
postgres=# CREATE DATABASE keycloak;
CREATE DATABASE
postgres=# CREATE USER keycloak WITH PASSWORD 'New1234!';
CREATE ROLE
postgres=# ALTER USER keycloak WITH SUPERUSER;
ALTER ROLE