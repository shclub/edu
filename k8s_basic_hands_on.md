# k8s Basic Hands-on
 
kubernetes에서 kubectl를 사용하여 cli 실습을 한다.   

1. kubectl 설치 ( 윈도우 / Mac )

2. kubectl Context 설정

3. k8s 리소스 보기

4. Pod , Replicaset , Deployment 생성 및 삭제

5. Service Expose

6. 참고 사이트 
    - podman 사용 : https://github.com/chhanz/kubernetes-hands-on-lab
    - https://github.com/subicura/workshop-k8s-basic/tree/master/guide


<br/>

##  Kubeconfig 설정 

### kubectl 설치 ( https://kubernetes.io/ko/docs/tasks/tools/ )

<br/>

GUI IDE인 lens 대신 kubectl cli를 통해 실습을 하기 위해  로컬 PC에
kubectl 을 설치 한다.


- Windows  
    아래사이트에서 최신 버전을 다운로드 받는다.  
https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-windows/   

    <img src="./assets/kubectl_download.png" style="width: 80%; height: auto;"/>  

    윈도우에서  kubectl.exe 패스를 추가한다. 
    환경 변수 창을 띄운다. 
    ```bash
    Windows 설정 - 시스템 - 정보 - 시스템 정보 - 고급 시스템 설정 - 환경 변수
    ```  
    시스템 변수에 path를 선택하고 추가한다.  

<br/>

- Mac
```bash
brew install kubectl
kubectl version --client
``` 

<img src="./assets/kubectl_version.png" style="width: 80%; height: auto;"/> 

<br/>

Kubernetes는 kubeconfig라는 YAML 파일을 사용하여 kubectl에 대한 클러스터 인증 정보를 저장합니다. kubeconfig에는 명령을 실행할 때 kubectl이 참조하는 컨텍스트 목록이 포함되어 있습니다. 기본적으로 파일은 $HOME/.kube/config에 저장됩니다.  

클러스터를 만들면 항목이 환경의 kubeconfig에 자동으로 추가되고 현재 컨텍스트가 해당 클러스터로 변경됩니다.  

현재 테스트 과정중에 config 이름이 본인이 설정한 이름으로 구성이 되어 있으니 기존 이름을 config로 변경 합니다.   

- config 화일이 여러개인 경우 아래와 같이 병합 하면 됩니다.  
    ```bash
    export KUBECONFIG=~/.kube/config:~/.kube/config-jakelee
    ```
    config가 병합되어 있는지 확인합니다.
    
    ```bash
    kubectl config view
    ```  
    <img src="./assets/config_merge.png" style="width: 80%; height: auto;"/>   

<br/>

원하는 k8s를 선택하기 위해 전체 context를 조회 해보고 현재 context를 확인합니다.  
 
```bash
kubectl config get-contexts
kubectl config current-context
```  

작업하기 원하는 context ( k8s cluster ) 로 변경 합니다.   
설정을 하면 기본 context로 설정이 됩니다.

```bash
kubectl config use-context my-cluster-name
```

<img src="./assets/switch_context.png" style="width: 80%; height: auto;"/>  

Cluster 정보를 확인 합니다.  

```bash
kubectl  cluster-info
```  

<img src="./assets/cluster_info.png" style="width: 80%; height: auto;"/>  

<br/>

##  Kubectl 활용 

### kubectl 명령어

<br/>

자주 사용하는 kubectl 명령어를 알아봅니다.  


```bash
# 화일 이름의 리소스를 적용한다. 없으면 insert 있으면 update
# 아래 create/update 명령어를 대체 할 수 있다. 
kubectl apply -f [화일이름]
# 화일 이름의 리소스를  생성한다.  
kubectl create -f [화일이름]
# 화일 이름의 리소스를  update  한다.
kubectl update -f [화일이름]

# 해당 리소스 정보를 보여준다
kubectl  get [리소스 타입] [리소스 이름]

# 해당 리소스 세부 정보를 보여준다
kubectl  describe [리소스 타입] [리소스 이름]

# 해당 리소스 로그 정보를 보여준다
kubectl  log  [리소스 이름]
# 해당 리소스를 삭제 한다
kubectl  delete  [리소스 타입] [리소스 이름]

# Pod ( Container ) 안에서 command를  할 수 있다.
kubectl exec -it  [PO 이름] /bin/sh
```  

<br/>

### 명령 vs 선언

<br/>

명령보다는 선언하여 사용하는 것을 권장.  

- 명령 

    ```bash
    kubectl scale --replicas=3 rs/nginx
    ```

- 선언 

    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
    name: nginx
    spec:
    replicas: 3
    ```  

    ```bash
    kubectl apply -f [화일명]
    ```  

<br/>

### Pod

<br/>

명령보다는 선언하여 사용하는 것을 권장.  



