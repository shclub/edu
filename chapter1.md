
Table of  Contents
<!-- vscode-markdown-toc -->
* [ VM 생성](#VM)
	* [kt cloud 에서 VM ( Ubuntu 18.04 ) 을 생성한다.](#ktcloudVMUbuntu18.04.)
	* [Private IP를 생성 한다.](#PrivateIP.)
	* [Port Forwarding을 설정한다.](#PortForwarding.)
	* [터미널 프로그램으로 서버에 접속하고 비밀번호를 변경한다.](#.)
* [Jenkins를 설치한다.](#Jenkins.)
	* [저장소 키 다운로드](#)
	* [패키지 인덱스를 업데이트 하고 라이브러를 최신 버전으로 올려준다.](#.-1)
	* [root 계정으로 Jenkins 를 설치한다.](#rootJenkins.)
	* [Jenkins 서비스 포트를 변경한다.](#Jenkins.-1)
	* [서비스 재시작 및 상태 확인](#-1)
	* [Jenkins Admin 초기 패스워드 확인 및 복사](#JenkinsAdmin)
	* [젠킨스 서버 접속](#-1)
	* [추가 플러그인 설치](#-1)
* [Docker 를 설치한다](#Docker)
	* [패키지 인덱스 업데이트](#-1)
	* [HTTPS를 통해 repository 를 이용하기 위해 package 들을 설치](#HTTPSrepositorypackage)
	* [Docker의 Official GPG Key 를 등록합니다.](#DockerOfficialGPGKey.)
	* [stable repository를 등록합니다.](#stablerepository.)
	* [도커를 설치합니다.](#.-1)
	* [도커 버전을 확인합니다.](#.-1)
	* [도커 이미지 다운로드 및 실행하기](#-1)
	* [젠킨스의 pipeline 으로 docker 를 실행하기 위해 권한을 부여한다.](#pipelinedocker.)
* [GitHub 계정을 생성](#GitHub)
	* [https://github.com/ 접속하고 계정 생성](#https:github.com)
	* [계정 생성 후에 Repository를  생성한다.](#Repository.)
	* [교육용 repository인 https://github.com/shclub/edu1 폴더의 파일을 복사하여 본인이 생성한 Repository에 신규 화일을 생성한다.](#repositoryhttps:github.comshclubedu1Repository.)
* [ Docker Hub 계정을 생성](#DockerHub)
	* [https://hub.docker.com/ 접속하고 계정 생성(향후 사내에서 개발시는 d-space Nexus 사용)](#https:hub.docker.comd-spaceNexus)
	* [Docker 연동 테스트를 한다.](#Docker.)
* [Jenkins 설정](#Jenkins)
	* [일반 사용자 계정을 생성한다](#-1)
	* [계정 별 권한 부여방법](#-1)
	* [Github token 생성하기](#Githubtoken)
	* [GitHub Credential을 생성한다.](#GitHubCredential.)
	* [Docker Hub Credential을 생성한다.](#DockerHubCredential.)
	* [파이프 라인을 구성한다.](#.-1)
	* [빌드 실행](#-1)
	* [Docker pull 및 실행 테스트](#Dockerpull)
	* [Jenkinsfile 설명](#Jenkinsfile)
	* [Jenkins 환경변수](#Jenkins-1)
	* [Tag를 사용한 Jenkins 빌드](#TagJenkins)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

<br/>

# Chapter 1 
 
CI 구성을 위해 Jenkins와 GitHub 그리고 Docker Hub를 연계하는 방법에 대해 설명한다.   

![](./assets/jenkins_ci_start.png)

1. 개발자가 자신이 작업한 코드를 GitHub Repository에 반영

2. Jenkins는 해당 Repository를 Checkout하여 Docker Image로 빌드

3. 빌드 완료 후 Docker Hub로 Image를 Push

 
실제 CI 과정에선 코드 리뷰나 테스트 등의 과정이 포함되어야 하나 이번 실습에선 생략하도록 한다.

<br/>

## <a name='VM'></a> VM 생성  

### <a name='ktcloudVMUbuntu18.04.'></a>kt cloud 에서 VM ( Ubuntu 18.04 ) 을 생성한다. 
  
<br/>


  - 기준 : 8 core , 16G  
  - 설치 예정 솔루션
  ```
   jenkins , k3s , ArgoCD , Pinpoint 등
  ```

서버 -> 서버 메뉴 이동 후  Create Server 선택  
zone은 KOR-Seoul M2 선택 후 서버 이름을 입력하고 사양을 선택한다.  
<br/>
오른쪽 하단에 Launch를 클릭하여 vm을 생성한다. 

<img src="./assets/kt_cloud_vm_create.png" style="width: 80%; height: auto;"/>  

<br/>
5분정도 경과하면 Alarm 아이콘에  vm생성 및 root 비밀번호를 확인 할 수 있다.  
비밀번호는 반드시 저장한다. 

<!--![](./assets/vm_created.png)-->

<img src="./assets/vm_created.png" style="width: 60%; height: auto;"/>  

<br/><br/>

### <a name='PrivateIP.'></a>Private IP를 생성 한다.
   
zone은  VM 생성 했던 존을 선택한다. ( KOR-Seoul M2 )

<img src="./assets/private_ip_create.png" style="width: 80%; height: auto;"/>

Launch 를 클릭하면 IP가 생성이 된다.  

![](./assets/private_ip_info.png)  

생성된 IP의 오른쪽 끝을 클릭하여 식별할 수 있는 이름을 만들어준다.

<img src="./assets/private_ip_more.png" style="width: 40%; height: auto;"/>

오른쪽에 이름이 변경 된것을 확인 할 수 있다.  

![](./assets/private_ip_modify.png)  

<br/><br/>

### <a name='PortForwarding.'></a>Port Forwarding을 설정한다.

VM 과 Private IP를 매핑하면 외부에서 접속 가능 하다.  

서버를 선택하고  Connection String을 선택한다.
<!--![](./assets/port_forwarding1.png)-->
 <img src="./assets/port_forwarding1.png" style="width: 80%; height: auto;"/>

- 사설 포트 : vm 서버의 포트  
- 공인 포트 : 외부에 노출할 포트  
- 공인 IP : 외부에서 접속할 IP ( 위에서 생성한 Private IP ) 

 <img src="./assets/port_forwarding2.png" style="width: 80%; height: auto;"/>

추가 버튼을 클릭하여 생성하고 아래 포트들도 반복하여 설정한다.

```
ssh : 22 -> 22222
jenkins : 9000 -> 9000
flask web : 40003 -> 40003
그외 : 40004 ~ 40010 
k8s  : 6443 -> 6443
k8s nodePort  : 30000 -> 32767
```
<br/><br/>

### <a name='.'></a>터미널 프로그램으로 서버에 접속하고 비밀번호를 변경한다.

   - Mac 에서는 Iterm2, 윈도우는 Putty 추천  

<br/>  
터미널에서 아래 명령어를 입력하여 로그인 한다.  

로그인 후  connecting 저장 질문이 나오면 yes 를 입력한다. 

```bash
ssh root@(본인 private ip) -p 22222
``` 

 <img src="./assets/first_login1.png" style="width: 80%; height: auto;"/>


비밀번호 입력 후 아래 명령어 실행하여 root 비밀번호를 변경한다.
```bash
passwd
```
```bash
Enter new UNIX password:
```

 <img src="./assets/first_login2.png" style="width: 80%; height: auto;"/>

   - 기존 비밀번호 물어 보는 경우도 있음  

<br/><br/>


## <a name='Jenkins.'></a>Jenkins를 설치한다.

### <a name=''></a>저장소 키 다운로드
<br/>

   ```bash
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   ```
   제대로 입력 되었는지 확인 한다.  

   ```bash
   echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
   ```
### <a name='.-1'></a>패키지 인덱스를 업데이트 하고 라이브러를 최신 버전으로 올려준다.

   ```bash
   apt update && apt upgrade
   ```
   중간에 계속 진행하는 것을 물어보면 Y 를 입력하여 진행 

 <img src="./assets/apt_update.png" style="width: 80%; height: auto;"/>

<br/><br/>

### <a name='rootJenkins.'></a>root 계정으로 Jenkins 를 설치한다.  


현재 VM은  java가 설치되지 않아 openjdk-8-jdk를 설치 해야 한다.  

jdk를 먼저 설치한다.   

   ```bash
   apt install  openjdk-8-jdk
   apt update
   ```  

jenkins를 설치한다.  

   ```bash
   apt install jenkins 
   ```




일반 계정이면 앞에 sudo 명령어를 반드시 붙여준다  

jdk를  설치한다.  

   ```bash
   sudo apt install openjdk-8-jdk  
   sudo apt update
   ```    

jenkins를 설치한다.  

   ```bash
   sudo apt install jenkins 
   ```  


### <a name='Jenkins.-1'></a>Jenkins 서비스 포트를 변경한다.  

   아래 화일을 vi 에디터를 사용하여 포트를 변경 ( 8080 ->  9000 )

   ```bash
   vi /etc/default/jenkins 
   ``` 
   ```bash
   HTTP_PORT=9000
   ```
   
   < vi에디터 사용법 >
   ```
   i : 데이터 입력모드
   ```
   ```
   esc : 명령모드
   ```
   ```
     / : 찾기
     x : 한글자 삭제
     dd : 한 라인 삭제
     :wq : 저장하고 나오기
     :q! : 저장안하고 나오기
     :set nocp : 라인 밀리는 현상 방지
   ``` 
    
### <a name='-1'></a>서비스 재시작 및 상태 확인
```bash
service jenkins restart
```
정상여부 확인
```bash
systemctl status jenkins
```
아래와 같이 active(running) 이면 정상이고 http포트가 9000으로 되어 있는지 확인한다.    

<img src="./assets/jenkins_status.png" style="width: 80%; height: auto;"/>  

- ctrl + c 를 눌러 해당 화면에서 나온다.

<br/><br/>  

### <a name='JenkinsAdmin'></a>Jenkins Admin 초기 패스워드 확인 및 복사
 아래 명령어를 사용하여 password 를 복사하고 저장해 놓는다.

```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```
<br/><br/> 

### <a name='-1'></a>젠킨스 서버 접속
브라우져로 http://(본인서버ip):9000으로  접속하면 아래와 같은 화면이 나온다.  
패스워드에 위 명령으로 확인한 문자열을 입력한다.  

<img src="./assets/jenkins_admin_password.png" style="width: 60%; height: auto;"/>  


Install Suggested Plugin 선택하고 Plugin 설치 한다.  

<img src="./assets/jenkins_suggested_plugin.png" style="width: 60%; height: auto;"/>  

다운로드를 시작한다. 네트웍 상황에 따라 시간이 많이 소요 될 수 있다.  

<img src="./assets/jenkins_suggested_plugin2.png" style="width: 60%; height: auto;"/>  

아래 와 같이 화면이 나오면 성공.

<img src="./assets/jenkins_admin_user.png" style="width: 60%; height: auto;"/> 

Admin 유저를 생성한다. 이메일은 아무값이나 넣어준다.  
save and continue 버튼을 클릭한다

<img src="./assets/jenkins_admin_user_create.png" style="width: 60%; height: auto;"/> 

save and Finished 버튼을 클릭한다.

<img src="./assets/jenkins_admin_user_created.png" style="width: 60%; height: auto;"/> 

설정 완료가 되면 아래 화면이 나오고 Jenkins를 시작 할 수 있다.  

<img src="./assets/jenkins_is_ready.png" style="width: 60%; height: auto;"/> 

- 해당 문서는 영문을 기준으로 하며 Jenkins는 브라우저의 언어를 따른다. 

<br/><br/>

### <a name='-1'></a>추가 플러그인 설치

Manage Jenkins 메뉴 선택  

<img src="./assets/manage_jenkins1.png" style="width: 80%; height: auto;"/> 

Manage Plugin 선택

<img src="./assets/jenkins_first_manage_plugins.png" style="width: 80%; height: auto;"/> 

Available Tab 이동하여 git으로 검색 한다.   
Git Parameter , GitHub Integration 선택  
docker 로 검색 후 	Docker Pipeline , docker-build-step 선택  
Download now and install after restart 클릭

<img src="./assets/plugin_git.png" style="width: 80%; height: auto;"/>  

아래와 같이 설치가 진행이 되고 Restart Check 를 하여 Jenkins를 재기동 한다

<img src="./assets/jenkins_restart_check.png" style="width: 60%; height: auto;"/>    

Jenkins restarting이 되고 다시 로그인을 한다.  

<img src="./assets/jenkins_restarting.png" style="width: 60%; height: auto;"/>

<br/><br/>

## <a name='Docker'></a>Docker 를 설치한다 


### <a name='-1'></a>패키지 인덱스 업데이트
```bash
apt-get update
```
<br/>

### <a name='HTTPSrepositorypackage'></a>HTTPS를 통해 repository 를 이용하기 위해 package 들을 설치
```bash
apt-get -y install  apt-transport-https ca-certificates curl gnupg lsb-release
```
<br/>

### <a name='DockerOfficialGPGKey.'></a>Docker의 Official GPG Key 를 등록합니다.
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

<br/>

### <a name='stablerepository.'></a>stable repository를 등록합니다.
```bash
echo \
"deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```  
<br/>

### <a name='.-1'></a>도커를 설치합니다.
```bash
apt-get update && apt-get install docker-ce docker-ce-cli containerd.io gnupg2 pass
```


* Ubuntu 에서 도커 로그인 버그가 있어 아래 처럼 에러가 발생하기 때문에 gnupg2 pass 라이브러리를 추가 했음.  

```bash
$ docker login -u shclub -p ******** https://index.docker.io/v1/
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Error saving credentials: error storing credentials - err: exit status 1, out: `Cannot autolaunch D-Bus without X11 $DISPLAY`
```

<br/>

### <a name='.-1'></a>도커 버전을 확인합니다.
```bash
docker --version
```
<img src="./assets/docker_version.png" style="width: 60%; height: auto;"/>

<br/>

### <a name='-1'></a>도커 이미지 다운로드 및 실행하기

```bash
docker run hello-world
```
 
<img src="./assets/docker_run_world.png" style="width: 80%; height: auto;"/>

<br/>

### <a name='pipelinedocker.'></a>젠킨스의 pipeline 으로 docker 를 실행하기 위해 권한을 부여한다.

/usr/bin/docker 의 사용자그룹을 jenkins 에 추가해준다  

```bash
usermod -aG docker jenkins
```
jenkins를 재시작 해준다.

```bash
service jenkins restart
```
<br/><br/>


## <a name='GitHub'></a>GitHub 계정을 생성

### <a name='https:github.com'></a>https://github.com/ 접속하고 계정 생성  

<br/>

### <a name='Repository.'></a>계정 생성 후에 Repository를  생성한다.
아래와 같이 이름 입력를 하고 README file check 를 한다
<img src="./assets/repository_create.png" style="width: 80%; height: auto;"/>

default 브랜치를 main에서 master로 변경한다. ( 맨 하단 setting 클릭하여 설정)

<img src="./assets/default_branch_modify.png" style="width: 60%; height: auto;"/>

<br/>

### <a name='repositoryhttps:github.comshclubedu1Repository.'></a>교육용 repository인 https://github.com/shclub/edu1 폴더의 파일을 복사하여 본인이 생성한 Repository에 신규 화일을 생성한다. 

총 4개 화일을 만들고 내용을 복사한다.  ( 향후 Git 사용법 교육 후 Git Clone 사용 )

<img src="./assets/shclub_edu_file.png" style="width: 60%; height: auto;"/>

   - 샘플은 pyhon flask 로 구성

<br/><br/>

## <a name='DockerHub'></a> Docker Hub 계정을 생성 

### <a name='https:hub.docker.comd-spaceNexus'></a>https://hub.docker.com/ 접속하고 계정 생성(향후 사내에서 개발시는 d-space Nexus 사용)  

<br/>

### <a name='Docker.'></a>Docker 연동 테스트를 한다.

```bash
docker tag hello-world (본인id)/hello-world
docker push (본인id)/hello-world  
```

- 권한 에러 발생시 docker login 한다
```bash
docker login 
```

<img src="./assets/docker_denied.png" style="width: 60%; height: auto;"/>

정상적으로 로그인후 push를 한다.

```bash
docker push (본인id)/hello-world  
```     

<img src="./assets/docker_push.png" style="width: 60%; height: auto;"/>

도커허브 본인 계정에서 도커 이미지 생성 확인  

<img src="./assets/docker_hub_world.png" style="width: 60%; height: auto;"/>


도커 이미지가 Private으로 되어 있으면 Public 으로 변경한다. 
- 개인 계정은 1개의 private 만 가능

setting 으로 이동하여 Make public 클릭후 repository 이름을 입력후 Make Public 클릭  

<img src="./assets/docker_hub_make_public.png" style="width: 60%; height: auto;"/>

<br/><br/>

## <a name='Jenkins'></a>Jenkins 설정

### <a name='-1'></a>일반 사용자 계정을 생성한다  

admin 계정으로 테스트 할 예정 으로  skip.  

Manage Jenkins -> Manage Users  로 이동한다. 사용자 생성 버튼 클릭 후 사용자 생성.  
<img src="./assets/jenkins_user.png" style="width: 80%; height: auto;"/>


<br/>

### <a name='-1'></a>계정 별 권한 부여방법 
Configure Global Security로 이동  

admin 계정으로 테스트 할 예정 으로  skip.
    
<img src="./assets/configure_global_security.png" style="width: 80%; height: auto;"/>


생성한 계정을 입력하고 Add 클릭 추가후 권한 설정은 일단 Ovrall 체크 후 저장  
Project-based Matrix Authorization Strategy 체크 후 권한 설정  

<img src="./assets/configure_global_security2.png" style="width: 80%; height: auto;"/>

<br/>

### <a name='Githubtoken'></a>Github token 생성하기

웹브라우저에서 Github 로그인하고 
Jenkins 에서 github repository 인증을 위해 사용할 token 을 생성한다.  

- Settings - Developer settings - Personal access tokens - Generate token  
선택해서 토큰 생성  

Github 사이트의 오른쪽 상단 본인 계정의 Setting으로 이동한다. ( 프로젝트의 setting이 아님 )  

<img src="./assets/github_token_setting1.png" style="width: 60%; height: auto;"/>

Expiration 은 No Expiration으로 선택하고 repo, admin:repo_hook 만 체크하고  Generation Token 버튼 클릭해서 토큰 생성  
 
<img src="./assets/github_token_setting2.png" style="width: 80%; height: auto;"/>


복사 아이콘을 클릭하여 토큰 값을 복사한다. 
- 다시 페이지에 들어가면 보이지 않아서 복사 후 저장 필요  

<img src="./assets/github_token_setting3.png" style="width: 80%; height: auto;"/>

<br/>

### <a name='GitHubCredential.'></a>GitHub Credential을 생성한다.  

Jenkins가 GitHub에서 Code를 가져올 수 있도록 Credential을 추가하자

- Manage Jenkins -> Manage Credential -> System 으로 이동한다.

<img src="./assets/jenkins_github_credential1.png" style="width: 80%; height: auto;"/>

Global Credential 클릭  

<img src="./assets/jenkins_github_credential2.png" style="width: 80%; height: auto;"/>

Add Credential를 클릭하면 계정 설정하는 화면이 나온다.  

<img src="./assets/jenkins_github_credential3.png" style="width: 80%; height: auto;"/>

Kind는  Username with password 를 선택해주시면 됩니다.  

Username 은 본인의 Github ID 를 선택해주시면 됩니다. ( 이메일 아님 )  

ID는 본인이 원하는 식별자를 넣어준다.  

password는 이전에 발급받은 Github Token 값을 입력한다.  
       
<img src="./assets/jenkins_github_credential4.png" style="width: 80%; height: auto;"/>

<br/>

### <a name='DockerHubCredential.'></a>Docker Hub Credential을 생성한다.  

Jenkins가 Docker Hub에 Image를 push 할 수 있도록 Credential을 추가하자

- Manage Jenkins -> Manage Credential -> System  -> Global Credential 로 이동한다.

Add Credential를 클릭하면 계정 설정하는 화면이 나온다.  

Kind는  Username with password 를 선택해주시면 됩니다.  

Username 은 본인의 Docker Hub ID 를 선택해주시면 됩니다. ( 이메일 아님 )  

ID는 본인이 원하는 식별자를 넣어준다.  

password는 이전에 Docker Hub 본인 계정의 비밀번호 값을 입력한다.  

<img src="./assets/jenkins_dockerhub_credential.png" style="width: 80%; height: auto;"/>


GitHub와 Docker Hub Credential 이 생선된 것을 확인한다.  

<img src="./assets/jenkins_github_dockerhub_credential.png" style="width: 60%; height: auto;"/>

<br/>

### <a name='.-1'></a>파이프 라인을 구성한다.
        
메인 화면 좌측 메뉴에서 새로운 Item 선택  

<img src="./assets/pipeline_newitem.png" style="width: 60%; height: auto;"/>

item 이름을 입력하고 Pipeline 을 선택 후에 OK  

<img src="./assets/pipeline_main.png" style="width: 60%; height: auto;"/>

로그 Rotation을 5로 설정한다. 5개의 History 를 저장한다. 

<img src="./assets/pipeline_log_rotation.png" style="width: 80%; height: auto;"/>

Jenkins 홈 폴더로 이동하고 아래 명령어를 실행한다.

- 홈 폴더 확인은  대쉬보드에서 Manage Jenkins -> Configure System으로 이동하여 보면 상단에 표시  

<img src="./assets/jenkins_home_folder.png" style="width: 60%; height: auto;"/>  


```bash
cd /var/lib/jenkins
ls ./jobs/edu1/builds
```    

7개의 History중  3,4,5,6,7  5개만 저장된것을 확인 할 수있다.    

<img src="./assets/jenkins_build_folder.png" style="width: 40%; height: auto;"/>  


대쉬보드에서도 History를 확인 할 수 있다.  

<img src="./assets/jenkins_build_history.png" style="width: 40%; height: auto;"/>  


Github Project URL을 설정하고 Git Parameter 를 체크하고 Parameter Type은 교육을 위한 용도 임으로 Branch를 선택한다. ( Tag는 Release를 위한 빌드 방식으로 그 당시 snapshot 이다. )   
Branch의 default는 orgin/master를 설정한다.  

<img src="./assets/pipeline_git.png" style="width: 80%; height: auto;"/>

<br/> 

* Tag 를 사용한 빌드는 맨 아래 부분을 참고한다.

<br/>

Build Triggers - GitHub hook trigger for GITScm polling 선택  
Repository 에 Git Url을 입력한다.  
Credential에 Jenkins에서 생성한 github_ci를 선택하여 추가한다.
 
<img src="./assets/pipeline_git2.png" style="width: 80%; height: auto;"/>


Script Path는 Jenkinsfile 로 설정한다.  
github에 대소문자 구문하여 Jenkinsfile 이 있어야함.  
Save 버튼을 클릭하여 저장한다.  

<img src="./assets/pipeline_scriptpath.png" style="width: 60%; height: auto;"/>

<br/>

### <a name='-1'></a>빌드 실행

빌드 하기 전에 Jenkins 화일로 이동하여 docker hub 의 repository와 docker credential은 본인의 계정으로 설정한다.  

대쉬보드에서 Build With Parameter를 선택하고 Branch 선택 후 빌드 한다.  

<img src="./assets/jenkins_first_build.png" style="width: 60%; height: auto;"/>

빌드가 진행 되는 것을 단계별로 확인 할 수 있다.  

<img src="./assets/build_stage2.png" style="width: 60%; height: auto;"/>

에러가 발생하면 해당 단계에서 마우스 오른쪽 버튼을 클릭하여 로그 확인 할수 있다.  
또한 왼쪽 하단의 Build History 에서 해당 빌드 번호를 클릭하여 자세한 에러를 볼수 있다.  

<img src="./assets/build_error.png" style="width: 60%; height: auto;"/>

Console Output을 선택하고 에러를 확인 할 수 있다.  

<img src="./assets/build_console_output.png" style="width: 60%; height: auto;"/>

아래 에러는 docker hub에서 repository가 private 으로 설정이 되어 발생한 에러이고 위에 설명한 대로 public 으로 변경하면 에러가 발생하지 않는다.  

<img src="./assets/build_docker_access_denied.png" style="width: 60%; height: auto;"/>

대쉬보드에서 해당 파이프라인인 edu1을 선택하고 다시 빌드 한다.  

<img src="./assets/build_again.png" style="width: 60%; height: auto;"/>

성공적으로 완료된 화면을 볼 수 있다.   

<img src="./assets/build_finish.png" style="width: 60%; height: auto;"/>

Docker Hub에서 정상적으로 생성된 이미지를 확인 할수 있다.  

<img src="./assets/build_dockerhub_check.png" style="width: 60%; height: auto;"/>

* Docker build에 에러가 발생하는 경우가 있는데 Jenkins plugins가 정상적을
  설치가 안되어 있을수 있다.   
  docker 검색하여 제대로 설치가 되어 있는지 확인 하고 없으면 재 설치 한다.

<br/>

### <a name='Dockerpull'></a>Docker pull 및 실행 테스트  

터미널로 VM 서버에 접속하여 생성된 도커이미지를 다운로드(pull)하고 실행 (run)  

```bash
docker pull shclub/edu1
```  

<img src="./assets/docker_pull_edu1.png" style="width: 60%; height: auto;"/>  

```bash
docker run -p 40003:8080 shclub/edu1
```
Python Flask 가 정상적으로 로드가 된걸 확인 할 수 있다.

<img src="./assets/docker_run_edu1.png" style="width: 60%; height: auto;"/>  

브라우저에서 http://(본인 VM 공인 ip):40003 호출하여 Hello World 확인
 
<img src="./assets/flask_web_edu1.png" style="width: 60%; height: auto;"/>  

<br/>  

- 과제 : github webhook를 통한 빌드 자동화   ( git push  하면 자동 빌드 )        
    
<br/>

### <a name='Jenkinsfile'></a>Jenkinsfile 설명  

Jenkins 화일에서 github와 docker credential 은  Jenkins 설정에서 Credential을 생성한
id를 입력하면 된다.  

<img src="./assets/pipeline_credential.png" style="width: 80%; height: auto;"/>  

Jenkins에 설정된 credential  

<img src="./assets/jenkins_github_dockerhub_credential.png" style="width: 80%; height: auto;"/>  


Jenkins Stage View 를 통해 Step별 진행 사항을 볼수 있다.  


<img src="./assets/jenkins_stage_view.png" style="width: 60%; height: auto;"/>  

<br/>

### <a name='Jenkins-1'></a>Jenkins 환경변수

<br/>

env 환경변수는 다음과 같은 형식 env.VARNAME으로 참조될 수 있다. 대표적인 env의 property는 아래와 같다. 

<img src="./assets/jenkins_env_variable.png" style="width: 80%; height: auto;"/>  

<br/>

currentBuild 환경변수는 현재 빌드되고 있는 정보를 담고있다. 보통 readonly 옵션인데 일부 writable한 옵션이 존재한다. 대표적인 currentBuild의 property는 아래와 같다.

<img src="./assets/jenkins_current_variable.png" style="width: 80%; height: auto;"/>    


환경 변수 사용 예제.

```bash
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
            }
        }
    }
}
```



<br/>

### <a name='TagJenkins'></a>Tag를 사용한 Jenkins 빌드

<br/>

Tag를 사용한 빌드는 운영(Release)를 위해 주로 사용하며 Tagging하는 순간의 snapshot 이다.  

Git Parameter에서 Tag를 선택하고의 default는 RB.0.1 을 임으로 설정한다.  

<img src="./assets/jenkins_tag.png" style="width: 60%; height: auto;"/>  

<br/>

GitHub 로 이동 한후 Repository를 선택 한 후 code Tab 으로 들어간다.  
Tags 아이콘을 클릭한다.

<img src="./assets/github_tag1.png" style="width: 60%; height: auto;"/>  

Tags를 선택하고 Create new release 버튼을 클릭한다.  

<img src="./assets/github_tag2.png" style="width: 60%; height: auto;"/>  

Choose a tag를 클릭하면 tag이름을 입력하는 text 박스가 나온다.

<img src="./assets/github_tag4.png" style="width: 60%; height: auto;"/>  

생성하고 싶은 Tag 이름을 입력한다.  
jenkins pipeline에서 RB.0.1을 기본값으로 설정을 해서 같은 이름으로 입력한다.  
입력창 아래 생성된 + Create new tag : RB.0.1 클릭

<img src="./assets/github_tag5.png" style="width: 60%; height: auto;"/>  

Title 값을 입력 ( 원하는 이름 아무거나 입력 ) 하고 Publish release 버튼을 클릭한다

<img src="./assets/github_tag6.png" style="width: 60%; height: auto;"/>

<br/>

Jenkins 대쉬보드에서 Build with Parameters 선택하면 Tag이름이 RB.0.1로 기본값이 설정된다.  

Tag를 선택하고 Build 버튼을 클릭하면 해당 Tag의 소스로 빌드가 된다.

<img src="./assets/jenkins_tag_build.png" style="width: 60%; height: auto;"/>  

<br/>