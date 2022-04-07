# Orcle cloud 사용법
 
Oracle Cloud의 `평생 무료 계정`을 생성하고 사용해본다.  

1. 계정 생성

2. vm 생성 : ubuntu 18.04

3. oracle cloud shell 사용법

4. vm에 k3s 설치

5. 외부에서 k3s 접속

6. 참고 사이트 
    - vm 생성 : https://bonohubby.com/entry/OCI-VM-Instance-%EC%83%9D%EC%84%B1-%EB%B0%8F-%EC%A0%91%EC%86%8D-%EB%B0%A9%EB%B2%95
    - VPS : https://hoing.io/archives/369 

<br/>

##  계정 생성

<br/>

### 무료 계정 생성

<br/>

 Oracle Cloud Infrastructure(이하 OCI) 회원가입을 진행한다.

 - 아래로 접속하여 오라클 클라우드 무료체험 버튼 클릭 -> 무료로 시작하기 클릭

    www.oracle.com/kr/cloud/

    <img src="./assets/oracle_cloud_account_1.png" style="width: 60%; height: auto;"/>  


    <img src="./assets/oracle_cloud_account_2.png" style="width: 60%; height: auto;"/>  

<br/>

Account info를 입력한다.  

<img src="./assets/oracle_cloud_account_3.png" style="width: 60%; height: auto;"/>    

이메일로 가입 정보가 오고 Account가 신규로 생성된다.   
이메일과 Account ID는 다르다.  오라클에서 자동으로 생성하며 이메일에서 확인 필요.  
- 예 ) Account ID : shclubm ( shclub.m@gmail.com )

<br/>

빌링 정보를 위하여 카드번호를 넣으면 되고 plan을 업그레이드 하지 않는한 과금이 되지 않는다.  

기본적으로 평생 무료 3개의 VM 이 할당 된다고 볼 수 있다.   

- VM 1: Oracle Cloud Shell ( 메모리 공유 , DISK : 5G )
    - http://taewan.kim/cloud/oci_cloud_shell/
    - 기본 설치 : ssh , docker , ansible , git , kubectl , helm ,
       python , maven , java 
- VM 2: 직접 생성 (  메모리 1G , DISK : 50G ) 
- VM 3: 직접 생성 (  메모리 1G , DISK : 50G )

<br/>

### Oracle cloud shell  

<br/>

VM 접속을 위한 web 기반의 shell을 제공하며 oracle 이외의 외부 시스템과   
ssh접속을 할수 있다.   

기본 설치 되어 있는 프로그램을 활용하면 대부분 cli 작업은 가능하다.  
root 권한은 확보할 수 없어 프로그램 추가 설치를 쉽지 않지만 docker를 활용하면 대부분 프로그램을 사용할 수 있다.  

<br/>

```bash
Welcome to Oracle Cloud Shell.
Cloud Shell now offers two pre-installed Java runtimes: OpenJDK 11 and OpenJDK 8. You can use the csruntimectl command from within Cloud Shell to switch between the runtimes. For more information, see Managing Language Runtimes in the Cloud Shell documentation.

Your Cloud Shell machine comes with 5GB of storage for your home directory. Your Cloud Shell (machine and home directory) are located in: South Korea Central (Seoul).
You are using Cloud Shell in tenancy shclubm as an OCI Local user shclub.m@gmail.com

Type `help` for more info.
shclub_m@cloudshell:~ (ap-seoul-1)$ free -h
              total        used        free      shared  buff/cache   available
Mem:           9.5G        769M        5.7G         16M        3.1G        8.4G
Swap:          1.0G          0B        1.0G

```  
cpu 정보와 디스크 사이즈  


```bash
shclub_m@cloudshell:~ (ap-seoul-1)$ cat /proc/cpuinfo | grep process
processor       : 0
processor       : 1
shclub_m@cloudshell:~ (ap-seoul-1)$ df -h
Filesystem                Size  Used Avail Use% Mounted on
overlay                    69G   18G   48G  27% /
tmpfs                      64M     0   64M   0% /dev
tmpfs                     4.8G     0  4.8G   0% /sys/fs/cgroup
shm                        64M     0   64M   0% /dev/shm
/dev/sdb1                 5.0G   33M  5.0G   1% /home/shclub_m
/dev/mapper/vg00-root      69G   18G   48G  27% /etc/hosts
/dev/mapper/vg00-var_log  3.8G   18M  3.6G   1% /etc/extensions/logs
```
<br/>

도커로 리눅스 이미지를 가져오고 실행하여 추가 프로그램 설치 가능  


```bash
shclub_m@cloudshell:~ (ap-seoul-1)$ docker run -it --rm alpine
Unable to find image 'alpine:latest' locally
Trying to pull repository docker.io/library/alpine ... 
latest: Pulling from docker.io/library/alpine
df9b9388f04a: Pull complete 
Digest: sha256:4edbd2beb5f78b1014028f4fbb99f3237d9561100b6881aabbf5acce2c4f9454
Status: Downloaded newer image for alpine:latest
/ # apk add curl
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.15/community/x86_64/APKINDEX.tar.gz
(1/5) Installing ca-certificates (20211220-r0)
(2/5) Installing brotli-libs (1.0.9-r5)
(3/5) Installing nghttp2-libs (1.46.0-r0)
(4/5) Installing libcurl (7.80.0-r0)
(5/5) Installing curl (7.80.0-r0)
Executing busybox-1.34.1-r5.trigger
Executing ca-certificates-20211220-r0.trigger
OK: 8 MiB in 19 packages
/ # 
```

##  로그인 및 VM 생성

<br/>

### 로그인

<br/>

 Oracle Cloud 사이트의 오른쪽 상단에 로그인 버튼을 클릭한다.  

<img src="./assets/oracle_cloud_login1.png" style="width: 60%; height: auto;"/> 

<br/>

### vm 생성

<br/>

로그인이 완료가 되면 시작하기 와 대쉬보드 탭이 나오고 기존에 VM이 생성되었으면   
대쉬보드로 이동하고 신규 VM 생성이 필요하면 시작하기 탭의 아래 VM인스턴스 생성 화면을 클릭한다.  

<img src="./assets/oracle_cloud_vm1.png" style="width: 80%; height: auto;"/> 

<br/>

첫 로그인 후에 생성시에는 구획을 설정 해야한다. 구획은 본인의 Account ID를 선택하면 된다.  

<img src="./assets/oracle_cloud_vm_pre.png" style="width: 80%; height: auto;"/>   

Instance 이름을 입력하고 아래 이미지를 편집한다.  
편집하지 않으면 oracle linux로 설치가 된다.    

<img src="./assets/oracle_cloud_vm2.png" style="width: 80%; height: auto;"/>   

이미지 변경 버튼을 클릭한다.  

<img src="./assets/oracle_cloud_vm3.png" style="width: 80%; height: auto;"/>   

ubuntu로 이미지를 변경하고 버전은 18.04 그리고 이미지 빌드는 최신으로 설정하고
이미지 설정을 완료한다.  

<img src="./assets/oracle_cloud_vm4.png" style="width: 80%; height: auto;"/>   

ssh로 접속하기 위해서 ssh 전용키를 다운로드 받고 인스턴스를 생성한다.  
  
<img src="./assets/oracle_cloud_vm5.png" style="width: 80%; height: auto;"/>   

provision ( 노란색 ) 이 진행이 되고 몇 초 후에 vm이 생성된다.  

<img src="./assets/oracle_cloud_vm6.png" style="width: 80%; height: auto;"/>   

실행중 ( 녹색 ) 이 되면 생성이 왼료가 되며 공용 IP를 복사한다.   

<img src="./assets/oracle_cloud_vm7.png" style="width: 80%; height: auto;"/>   

접속을 하기 위해 cloud shell 을 활성화 한다. 오른쪽 상단 클릭  

<img src="./assets/oracle_cloud_shell_activate.png" style="width: 80%; height: auto;"/>   


브라우저 하단에 cloud shell 창이 뜬다.  

<img src="./assets/oracle_cloud_shell1.png" style="width: 80%; height: auto;"/>   


cloud shell 의 왼쪽 상단에 upload를 하여 다운 받은 ssh 전용키를 upload 한다.    

<img src="./assets/oracle_cloud_shell1.png" style="width: 80%; height: auto;"/>   

