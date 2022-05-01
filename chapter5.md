# Chapter 5 
   
컨테이너 모니티링을 위한 SaaS 솔루션인 Datadog과 연동해 본다.  

1. kt cloud 하드 디스크 추가

2. k3s 위치 변경

3. Github action 과 worlflow 사용하여 도커 이미지 생성

4. DataDog 연동

 
<br/>

##  kt cloud 하드 디스크 추가

<br/>

### 서버 용량 확인 

<br/>

kt cloud는 기본 20G이 하드 디스크를 제공을 하여 container 운영시 evicted 
에러가 많이 발생함.  

먼저 터미널로 로그인 한다.  

```bash
ssh root@(본인 VM 공인 ip) -p 22222
``` 

아래 명령어를 실행한다.  

```bash
root@jakelee:/# df -h | grep dev
udev            7.8G     0  7.8G   0% /dev
/dev/xvda4       17G  9.1G  6.7G  58% /
tmpfs           7.9G     0  7.9G   0% /dev/shm
/dev/loop0      112M  112M     0 100% /snap/core/12941
/dev/loop1      111M  111M     0 100% /snap/core/12834
/dev/xvda3      976M  240M  670M  27% /boot
```

/dev/xvda4가 기본제공되는 파티션이고 70% 이상 점유가 되어 있으면 컨테이너 POD 기동이 안될 수 있음.  

<br/>

### 디스크 추가  

<br/>

kt cloud 에 로그인 한다.    

server -> disk를 클릭합니다.  


<img src="./assets/disk_add1.png" style="width: 80%; height: auto;"/>   

create disk 를 클릭합니다.

<img src="./assets/disk_add2.png" style="width: 80%; height: auto;"/>   

아래와 같이 값을 선택합니다.  

- zone : KOR-Seoul M2 ( 현재 교육 환경은  M2 zone에 설치 )
- name : host 이름과 같이 설정 ( 식별을 편하게 하기 위함 )
- product : SSD ( 빠른 성능 )
- size : 50G 

launch 버튼을 클릭하여 디스크를 생성하며 약간의 시간 소요.

<img src="./assets/disk_add3.png" style="width: 80%; height: auto;"/>   

state 상태가 Release라고 나오며 붉은색으로 표시됨. 이것은 아직 서버에 디스크가 연결 되지 않았다는 의미.  

<img src="./assets/disk_add4.png" style="width: 80%; height: auto;"/>   

connect 버튼을 클릭을 하면 연결할 서버가 나오고 원하는 대상 서버를 선택합니다.  

<img src="./assets/disk_add5.png" style="width: 80%; height: auto;"/>   

아래 메시지가 나오면 ok를 클릭합니다. 

<img src="./assets/disk_add6.png" style="width: 60%; height: auto;"/>   

완료가 되면 status가 connect 로 나오고 kt cloud 에서는 해야 할 일은 완료 되었습니다.   

<img src="./assets/disk_add7.png" style="width: 80%; height: auto;"/>   

<br/>

### 디스크 붙이기 

<br/>

터미널로 로그인 한다.  

```bash
ssh root@(본인 VM 공인 ip) -p 22222
```  
아래 명령어를 실행한다.  

```bash
root@jakelee:/# df -h | grep dev
udev            7.8G     0  7.8G   0% /dev
/dev/xvda4       17G  9.1G  6.7G  58% /
tmpfs           7.9G     0  7.9G   0% /dev/shm
/dev/loop0      112M  112M     0 100% /snap/core/12941
/dev/loop1      111M  111M     0 100% /snap/core/12834
/dev/xvdb        49G    0    49G  100% 
/dev/xvda3      976M  240M  670M  27% /boot
```

/dev/xvdb 라는 파티션이 추 가된걸 확인 할수 있습니다.  

위에서 확인한 디바이스 파티션의 포맷을 진행합니다. ( 리눅스 파티션 ext4 )

```bash
root@jakelee:/# mkfs.ext4 /dev/xvdb
```  

포맷이 완료되면 UUID를 확인할 수 있습니다.   위에서도 보이지만 아래의 명령어를 통해서도 UUID를 확인할 수 있습니다.  

```bash
root@jakelee:/# blkid
/dev/xvdb: UUID="f7f5fb33-80f0-4eda-b103-4be9b6aa070e" TYPE="ext4"
/dev/xvda2: UUID="2feb6a8f-952a-4b49-9e39-03b712dc75d3" TYPE="swap" PARTUUID="94574b76-926a-4d85-b78c-f370c646afd9"
/dev/xvda3: UUID="2cab3d8f-b495-43a2-9ea4-db1a02bce959" TYPE="ext4" PARTUUID="375daaec-397a-4545-a75a-6ab586eed954"
/dev/xvda4: UUID="89a01c64-beb2-4de2-bd8b-7aa7146e41ee" TYPE="ext4" PARTUUID="25f58f5f-6d2a-4c4b-96e9-f8345dcf4d16"
/dev/loop0: TYPE="squashfs"
/dev/loop1: TYPE="squashfs"
/dev/xvda1: PARTUUID="6eb8e524-4b89-4dfc-9d73-d259d395f4ac"
```  

이제 디스크를 사용하기 위해 마운트 할 차례입니다. 먼저 마운트 할 대상 폴더를 만들어줍니다.  

```bash
root@jakelee:/# mkdir -p /data
```  

자동 마운트를 위해 마운트 정보를 /etc/fstab 파일에 추가합니다.  

```bash
root@jakelee:/# vi /etc/fstab
```  

이제 마운트를 적용합니다.  

```bash
root@jakelee:/# mount -a
```  

<img src="./assets/disk_mount.png" style="width: 80%; height: auto;"/>   

아래 명령어를 실행하여 /data 마운트 포인트가 생성된걸 확인합니다.  

```bash
root@jakelee:/# df -h | grep dev
udev            7.8G     0  7.8G   0% /dev
/dev/xvda4       17G  9.1G  6.7G  58% /
tmpfs           7.9G     0  7.9G   0% /dev/shm
/dev/loop0      112M  112M     0 100% /snap/core/12941
/dev/loop1      111M  111M     0 100% /snap/core/12834
/dev/xvdb        49G     0   49G   0% /data
/dev/xvda3      976M  240M  670M  27% /boot
```  

<br/>

##  k3s 위치 변경

<br/>

### k3s 신규 폴더 생성 

<br/>

먼저 /var/lib/rancher를 /data 폴더에 복사합니다.  

```
cp -rp /var/lib/rancher /data
```  

rancher폴더가 생성된 것을 확인 할 수 있습니다.  
```
root@jakelee:/# ls /data
rancher
```  

<br/>

### k3s 위치 변경

<br/>

k3s 서비스의 시작 위치를 확인하기 위해 아래 명령어를 실행합니다.    

```bash
root@jakelee:/# systemctl status k3s
● k3s.service - Lightweight Kubernetes
   Loaded: loaded (/etc/systemd/system/k3s.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2022-04-30 12:04:31 KST; 22h ago
     Docs: https://k3s.io
  Process: 989 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
  Process: 978 ExecStartPre=/sbin/modprobe br_netfilter (code=exited, status=0/SUCCESS)
  Process: 956 ExecStartPre=/bin/sh -xc ! /usr/bin/systemctl is-enabled --quiet nm-cloud-setup.serv
 Main PID: 990 (k3s-server)
    Tasks: 506
```  

서비스 위치는 /etc/systemd/system/k3s.service 이란 것을 확인 할 수 있고  
vi 에디터로 /etc/systemd/system/k3s.service 를 수정합니다.  

ExecStart 구문에서 --data-dir=/data/rancher/k3s 를 추가합니다.


```bash
# before
ExecStart=/usr/local/bin/k3s \
    server  \
        '--tls-san' \
        '210.106.105.165' \

# after
ExecStart=/usr/local/bin/k3s \
    server \
        '--data-dir=/data/rancher/k3s' \
        '--tls-san' \
        '210.106.105.165' \

```  

시스템 데몬과 k3s를 재기동 합니다.  

```bash
systemctl daemon-reload 
systemctl restart k3s
```  

정상기동을 확인합니다.   

```bash
systemctl status k3s
```  

신규 파티션 ( /dev/xvdb )에 disk가 사용되는지 확인 합니다.

```bash
root@jakelee:/# df -h | grep dev
udev            7.8G     0  7.8G   0% /dev
/dev/xvda4       17G  9.0G  6.8G  58% /
tmpfs           7.9G     0  7.9G   0% /dev/shm
/dev/loop0      112M  112M     0 100% /snap/core/12941
/dev/loop1      111M  111M     0 100% /snap/core/12834
/dev/xvdb        49G  6.5G   41G  14% /data
/dev/xvda3      976M  240M  670M  27% /boot
```

<br/>

##  Github action 과 worlflow 사용하여 도커 이미지 생성

<br/>

### k3s 신규 폴더 생성 

<br/>

먼저 /var/lib/rancher를 /data 폴더에 복사합니다.  

```
cp -rp /var/lib/rancher /data
```  

rancher폴더가 생성된 것을 확인 할 수 있습니다.  
```
root@jakelee:/# ls /data
rancher
```  


<br/>

## 과제

<br/>

### 과제 1

현재 Docker Root 디렉토리를 /data로 변경한다.  

도커도 위와 같이 폴더를 변경 할 수 있습니다.    

- TIP 
    - 현재 Docker Root 디렉토리 확인
        - docker info | grep "Docker Root Dir"
    - 도커 status 정보
        - systemctl status docker
    - ExecStart로 시작하는 라인 끝에 --data-root=/data/docker 추가

