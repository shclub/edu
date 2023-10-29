
# 교육 준비

<br/>

## 1. OKD namespace를 생성한다

<br/>

```bash
[root@bastion education]# oc new-project edu6
Now using project "edu6" on server "https://api.okd4.ktdemo.duckdns.org:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/e2e-test-images/agnhost:2.33 -- /agnhost serve-hostname
```  

<br/>

생성하고 annotion 은 edu=true 로 설정한다.

```bash
[root@bastion education]# kubectl edit namespace edu6 -n edu6
namespace/edu6 edited
```  

<br/>

```bash
    openshift.io/node-selector: edu=true
```  

<br/>

## 2. OKD 계정생성

<br/>

계정을 추가하는 경우에는 먼저 htpasswd secret에서 기존 정보를 가져온다.


```bash
[root@bastion education]# oc get secret htpasswd -ojsonpath={.data.htpasswd} -n openshift-config | base64 --decode > htpasswd
[root@bastion education]# cat htpasswd
edu1:$2y$05$fN.mHoQj51sOcmqpt371/urQ07Iy7Ismsl03wQE.BrOhWyu1SINwG
edu2:$2y$05$/f2R4ypb9aNtoERpscIS/Olb/a21B/Ir7pgAFelfPd1EidvQR.17.
edu3:$2y$05$YirA4f3h5y0APL2c.k6bdOciLZycbTkKzxrbQwokfmLyyiHYYAQvu
edu4:$2y$05$C88pJwFotTrwtWEPOwHdwuQTLtZLe71upA./NWR8wVqkvE7g6fa3W
edu5:$2y$05$YeiWnK/3MOnSjux59Y7DyuJmqyzi8dfLpdU3ig9H2t8AZJnAlpPUq
```  

<br/>

htpasswd  화일에 기존 계정의 값이 있고 아래와 같이 신규 계정을 추가한다.    

```bash
[root@bastion education]# htpasswd -Bb htpasswd edu6 'New1234!'
Adding password for user edu6
```  

<br/>

이제 적용하고 web console에서 다시 접속해 본다.

```bash
[root@bastion ~]# oc --user=admin create secret generic htpasswd  --from-file=htpasswd -n openshift-config --dry-run=client -o yaml | oc replace -f -
secret/htpasswd replaced
```

<br/>


## 3. ArgoCD 계정 생성

<br/>

`argocd-cm` configmap에 data 를 생성하고 계정을 아래와 같이 추가합니다.    

```bash
[root@bastion argocd]# kubectl -n argocd edit configmap argocd-cm -o yaml
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
  accounts.icis: apiKey,login
  accounts.haerin: apiKey,login
  accounts.hans: apiKey,login
  accounts.rorty: apiKey,login
  accounts.shclub: apiKey,login
  exec.enabled: "true"
  exec.shells: bash
kind: ConfigMap
...  
```

<br/>

로그인을 하고 계정 별로 비밀번호를 생성합니다.  

```bash
[root@bastion education]# kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   172.30.86.34     <none>        7000/TCP,8080/TCP            27d
argocd-dex-server                         ClusterIP   172.30.150.160   <none>        5556/TCP,5557/TCP,5558/TCP   27d
argocd-metrics                            ClusterIP   172.30.255.252   <none>        8082/TCP                     27d
argocd-notifications-controller-metrics   ClusterIP   172.30.150.223   <none>        9001/TCP                     27d
argocd-redis                              ClusterIP   172.30.180.11    <none>        6379/TCP                     27d
argocd-repo-server                        ClusterIP   172.30.14.89     <none>        8081/TCP,8084/TCP            27d
argocd-server                             NodePort    172.30.148.165   <none>        80:32270/TCP,443:32184/TCP   27d
argocd-server-metrics                     ClusterIP   172.30.2.159     <none>        8083/TCP                     27d
[root@bastion education]# argocd login 192.168.1.146:32270
WARNING: server is not configured with TLS. Proceed (y/n)? y
Username: admin
Password:
'admin:login' logged in successfully
Context '192.168.1.146:32270' updated
```

<br/>

```bash
[root@bastion education]# argocd account update-password --account edu1 --new-password New1234!
*** Enter password of currently logged in user (admin):
Password updated
```

<br/>

`argocd-rbac-cm` configmap에 data 를 생성하고 계정별로 권한을 추가합니다.
- exec는 terminal 을 사용할수 있는 권한 입니다.  

```bash
[root@bastion argocd]# kubectl -n argocd edit configmap argocd-rbac-cm -o yaml

data:
  policy.csv: |
    p, role:manager, applications, *, */*, allow
    p, role:manager, clusters, get, *, allow
    p, role:manager, repositories, *, *, allow
    p, role:manager, projects, *, *, allow
    p, role:manager, exec, *, */*, allow
    p, role:edu1, clusters, get, *, allow
    p, role:edu1, repositories, get, *, allow
    p, role:edu1, projects, get, *, allow
    p, role:edu1, applications, *, edu1/*, allow
    p, role:edu1, exec, create, edu1/*, allow
    p, role:edu2, clusters, get, *, allow
    p, role:edu2, repositories, get, *, allow
    p, role:edu2, projects, get, *, allow
    p, role:edu2, applications, *, edu2/*, allow
    p, role:edu2, exec, create, edu2/*, allow
    p, role:edu3, clusters, get, *, allow
    p, role:edu3, repositories, get, *, allow
    p, role:edu3, projects, get, *, allow
    p, role:edu3, applications, *, edu3/*, allow
    p, role:edu3, exec, create, edu3/*, allow
    p, role:edu4, clusters, get, *, allow
    p, role:edu4, repositories, get, *, allow
    p, role:edu4, projects, get, *, allow
    p, role:edu4, applications, *, edu4/*, allow
    p, role:edu4, exec, create, edu4/*, allow
    p, role:edu5, clusters, get, *, allow
    p, role:edu5, repositories, get, *, allow
    p, role:edu5, projects, get, *, allow
    p, role:edu5, applications, *, edu5/*, allow
    p, role:edu5, exec, create, edu5/*, allow
    p, role:edu6, clusters, get, *, allow
    p, role:edu6, repositories, get, *, allow
    p, role:edu6, projects, get, *, allow
    p, role:edu6, applications, *, edu6/*, allow
    p, role:edu6, exec, create, edu6/*, allow
    p, role:edu7, clusters, get, *, allow
    p, role:edu7, repositories, get, *, allow
    p, role:edu7, projects, get, *, allow
    p, role:edu7, applications, *, edu7/*, allow
    p, role:edu7, exec, create, edu7/*, allow
    p, role:edu8, clusters, get, *, allow
    p, role:edu8, repositories, get, *, allow
    p, role:edu8, projects, get, *, allow
    p, role:edu8, applications, *, edu8/*, allow
    p, role:edu8, exec, create, edu8/*, allow
    p, role:edu9, clusters, get, *, allow
    p, role:edu9, repositories, get, *, allow
    p, role:edu9, projects, get, *, allow
    p, role:edu9, applications, *, edu9/*, allow
    p, role:edu9, exec, create, edu9/*, allow
    p, role:edu10, clusters, get, *, allow
    p, role:edu10, repositories, get, *, allow
    p, role:edu10, projects, get, *, allow
    p, role:edu10, applications, *, edu10/*, allow
    p, role:edu10, exec, create, edu10/*, allow
    p, role:edu11, clusters, get, *, allow
    p, role:edu11, repositories, get, *, allow
    p, role:edu11, projects, get, *, allow
    p, role:edu11, applications, *, edu11/*, allow
    p, role:edu11, exec, create, edu11/*, allow
    p, role:edu12, clusters, get, *, allow
    p, role:edu12, repositories, get, *, allow
    p, role:edu12, projects, get, *, allow
    p, role:edu12, applications, *, edu12/*, allow
    p, role:edu12, exec, create, edu12/*, allow
    p, role:edu13, clusters, get, *, allow
    p, role:edu13, repositories, get, *, allow
    p, role:edu13, projects, get, *, allow
    p, role:edu13, applications, *, edu13/*, allow
    p, role:edu13, exec, create, edu13/*, allow
    p, role:edu14, clusters, get, *, allow
    p, role:edu14, repositories, get, *, allow
    p, role:edu14, projects, get, *, allow
    p, role:edu14, applications, *, edu14/*, allow
    p, role:edu14, exec, create, edu14/*, allow
    p, role:edu15, clusters, get, *, allow
    p, role:edu15, repositories, get, *, allow
    p, role:edu15, projects, get, *, allow
    p, role:edu15, applications, *, edu15/*, allow
    p, role:edu15, exec, create, edu15/*, allow
    p, role:edu16, clusters, get, *, allow
    p, role:edu16, repositories, get, *, allow
    p, role:edu16, projects, get, *, allow
    p, role:edu16, applications, *, edu16/*, allow
    p, role:edu16, exec, create, edu16/*, allow
    p, role:edu17, clusters, get, *, allow
    p, role:edu5, repositories, get, *, allow
    p, role:edu17, projects, get, *, allow
    p, role:edu17, applications, *, edu17/*, allow
    p, role:edu17, exec, create, edu17/*, allow
    p, role:edu18, clusters, get, *, allow
    p, role:edu18, repositories, get, *, allow
    p, role:edu18, projects, get, *, allow
    p, role:edu18, applications, *, edu18/*, allow
    p, role:edu18, exec, create, edu18/*, allow
    p, role:edu19, clusters, get, *, allow
    p, role:edu19, repositories, get, *, allow
    p, role:edu19, projects, get, *, allow
    p, role:edu19, applications, *, edu19/*, allow
    p, role:edu19, exec, create, edu19/*, allow
    p, role:edu20, clusters, get, *, allow
    p, role:edu20, repositories, get, *, allow
    p, role:edu20, projects, get, *, allow
    p, role:edu20, applications, *, edu20/*, allow
    p, role:edu20, exec, create, edu20/*, allow
    p, role:edu21, clusters, get, *, allow
    p, role:edu21, repositories, get, *, allow
    p, role:edu21, projects, get, *, allow
    p, role:edu21, applications, *, edu21/*, allow
    p, role:edu21, exec, create, edu21/*, allow
    p, role:edu22, clusters, get, *, allow
    p, role:edu22, repositories, get, *, allow
    p, role:edu22, projects, get, *, allow
    p, role:edu22, applications, *, edu22/*, allow
    p, role:edu22, exec, create, edu22/*, allow
    p, role:edu23, clusters, get, *, allow
    p, role:edu23, repositories, get, *, allow
    p, role:edu23, projects, get, *, allow
    p, role:edu23, applications, *, edu23/*, allow
    p, role:edu23, exec, create, edu23/*, allow
    p, role:edu24, clusters, get, *, allow
    p, role:edu24, repositories, get, *, allow
    p, role:edu24, projects, get, *, allow
    p, role:edu24, applications, *, edu24/*, allow
    p, role:edu24, exec, create, edu24/*, allow
    p, role:edu25, clusters, get, *, allow
    p, role:edu25, repositories, get, *, allow
    p, role:edu25, projects, get, *, allow
    p, role:edu25, applications, *, edu25/*, allow
    p, role:edu25, exec, create, edu25/*, allow
    p, role:icis, clusters, get, *, allow
    p, role:icis, repositories, get, *, allow
    p, role:icis, projects, get, *, allow
    p, role:icis, applications, *, icis/*, allow
    p, role:icis, exec, create, icis/*, allow
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
    g, icis, role:icis            
    g, shclub, role:manager
    g, haerin, role:manager
    g, hans, role:manager
    g, rorty, role:manager
  policy.default: role:''
```
<br/>

web에서 argocd 접속하여 project 를 생성하고 `source , destination , CLUSTER RESOURCE ALLOW LIST`  를 설정한다.   
- `*` 로 추가한다.

<br/>

## 4. VM (Bastion) 서버 정리 및 설정

<br/>

컨테이너 전체삭제  
- docker rm -f  `docker ps -a -q`

<br/>

이미지 전체삭제  
- docker rmi -f $(docker images -q)

<br/>

네트워크 전체삭제  
- docker network prune  

volume 전체 삭제
- docker volume prune


<br/>

로그아웃  
- docker logout  
- docker logout ghcr.io

<br/>

oclogin 스크립트 작성

```bash
root@edu2:~# vi oclogin
#!/bin/sh

echo 'KTDEMO Duckdns edu connect. '
oc login https://api.okd4.ktdemo.duckdns.org:6443 -u edu8 -p New1234!  --insecure-skip-tls-verify
root@edu2:~# chmod 777 oclogin
```

<br/>

worker node login 스크립트 작성

```bash
root@edu2:~# vi worker.sh
#!/bin/sh

echo 'Worker Node OKD-7 connect.'
ssh core@okd-7.okd4.ktdemo.duckdns.org -p 32222

root@edu2:~# chmod 777 worker.sh
```