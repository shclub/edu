# Chapter 2 
 
Jenkins CI 구성 요소인 Git , Docker에 대해서 자세한 설명과 함께 Docker Compose 에 대해서도 실습을 한다.   


1. GitHub를 통한 Git 사용법

2. Dockerfile 구성 및 빌드 , Tag , Push 

3. Docker compose 설치 및 Front/Backend, NoSQL 구성 예제 실습

 
<br/>

##  Git  ( https://backlog.com/git-tutorial/ )

### Git 개요 

Git이란 소스코드를 효과적으로 관리하기 위해 개발된 '분산형 버전 관리 시스템'입니다. ( 이전 에는 SVN 많이 사용 )  

가장 대중적인 SaaS 형태는 Mirosoft에서 제공하는 GitHub 이고
Private 형태로는 Gitlab을 많이 사용 함.


### GitHub에서 새로운 Repository를 생성한다.
   

(본인계정)/edu2라는 Repository를 생성. ( branch는 master )

<img src="./assets/github_edu2.png" style="width: 40%; height: auto;"/>  

### Git 설치.

터미널로 VM에 로그인 하여 Git이 설치되어 있는지 확인하고 없으면 Git을 설치 한다. 

```bash
ssh root@(본인 private ip) -p 22222
``` 

버전을 확인한다.  

```bash
git --version
``` 

<img src="./assets/git_version.png" style="width: 40%; height: auto;"/>

git이 없으면 설치 한다.  

```bash
apt-update && apt-get intsall git
```

### Git Clone 하여 github 소스 가져오기.  

github에서  shclub/edu2 를 선택하고 code를 클릭한다.  
https의 url를 복사한다. 오른쪽 복사 아이콘 클릭  

<img src="./assets/github_clone.png" style="width: 40%; height: auto;"/>  

터미털에서 git clone 명령어를 사용하여 로컬에 소스를 가져온다.  
정상적으로 가져오면 edu2 폴더로 이동하여 파일 받아진것 확인한다.

```bash
git clone https://github.com/shclub/edu2.git
```

<img src="./assets/git_clone.png" style="width: 40%; height: auto;"/>  


### 로컬에서 본인 github에 소스 push.  

다운로드 받은 화일을 본인이 생성한 repository에 push를 한다.  
먼저 repository url을 변경한다.

원격 저장소를 확인한다.

```bash
git remote -v
```

<img src="./assets/git_remote.png" style="width: 60%; height: auto;"/>  

새로운 원격 리포지토리를 추가한다.  

저장소를 처음만들고, 원격 저장소에 지정할 때의 명령어는 아래와 같다. 아직 지정해둔 원격 저장소가 없을 때엔 remote와 origin 사이에 add를, 지정해둔 원격 저장소 주소를 바꾸고 싶을 땐 set-url을 붙이는 차이가 있다.

```bash
git remote add origin https://github.com/(본인계정)/edu2.git
```

원격저장소를 본인의 github repository로 변경하고 다시 저장소를 확인한다.
git remote set-url origin https://github.com/(본인계정)/edu2.git

```bash
git remote set-url origin https://github.com/(본인계정)/edu2.git
git remote -v
```

<img src="./assets/git_remote_modify.png" style="width: 60%; height: auto;"/>  

소스를 commit 하고 push 한다.
- add . : .은 변경된 모든 소스를 대상으로 한다.
- commit :  로컬 저장소에 저장한다.
- push : 원격저장소에 저장한다.

```bash
git add .
git commit -m "first commit"
git push -u origin master
```  

git push시에 로그인이 필요하며 username은 본인 github id를 넣어주고 
비밀번호는 token 값을 넣어주어야 한다. ( 비밀번호는 불가 )

<img src="./assets/git_push.png" style="width: 60%; height: auto;"/>  

github 화면으로 가서 README.md 화일이 생성되거나 변경 된것을 확인할 수있다.

<img src="./assets/git_push_github.png" style="width: 60%; height: auto;"/>  

<br/><br/>


## Docker 사용법

### Docker 란 ?

소개 및 배경
- 도커는 컨테이너 기반의 오픈소스 가상화 플랫폼이다.
- 다양한 이유로 계속 바뀌는 서버 환경과 개발 환경 문제를 해결하기 위해 등장했다.
   (툴 업데이트, 회사의 툴 사용 변경, 회사의 언어 정책 변경 등 )  
- 기존에 서버와 개발 환경이 변경되면 컴퓨터 셋팅(개발 환경) 등을 다시하거나 
  그 과정에서 발생하는 문제들과 같이, 여러모로 불편한 점이 많았다.
- 도커가 등장하고 서버관리/개발 방식이(컨테이너) 완전히 편리하게 바뀌게 된다.

사용하는 이유
- 도커 허브에 올라온 이미지와 docker-compose.yml의 설정으로 원하는
  프로그램을 편안하게 설치가 가능하다.
- 컨테이너를 생성하여 분리된 환경에 설치하므로 제거도 쉽다.
- 하나의 서버(로컬 호스트)에 포트만 변경하여 동일한 프로그램을 실행하기도 쉽다.
- 도커를 사용하지 않으면 환경변수나 경로 등의 충돌로 피곤하기 그지 없다.

서버관리 방식의 변화
- 전통적인 서버관리 방식은 아래와 같이 각 단계별로 흐름이 있었고, 각 단계가
  업데이트 되거나 어떤 문제가 발생하면 전체 흐름이 중단되는 문제가 있었다.
- 도커 등장 이후 어떠한 프로그램도 컨테이너로 만들 수 있다.
- 서로 다른 프로그램이더라도 컨테이너로 규격화되었음
- AWS, Azure, Google cloud 등 어떤 환경에서도 돌아간다.
    <img src="./assets/docker_overview.png" style="width: 60%; height: auto;"/>  
- 가상머신과 비슷하게 생각할 수 있지만 비슷한점과 다른점이 있다.
- 가상머신처럼 독립적으로 실행되지만, 가상머신보다 빠르고 쉽고 효율적이다.
- 도커는 컴퓨터 자원을 그대로 사용한다. 

도커의 특징
- 도커는 가상머신이 아니고 격리만 해주기 떄문에 성능상 하락이 없다. (성능 하락이 큰 VM과 다르다.)
- 확장성과 이식성
    1) 도커가 설치되어 있다면 어디서든 컨테이너를 실행할 수 있다.
    2) 오픈 소스이기에 특정 회사나 서비스에 종속적이지 않다.
    3) 쉽게 개발서버를 만들 수 있고 테스트 서버 생성도 가능하다.
- 표준성
    1) 도커를 사용하지 않는 경우, 각각의 언어로 만든 서비스들의 배포 방식은 모두 다르다.
    2) 도커는 컨테이너라는 표준으로 서버를 배포하므로 모든 서비스들의 배포 과정이 동일해진다.
- 이미지
    1) 컨테이너를 실행하기 위한 압축파일과 같은 개념이다.
    2) 이미지에서 컨테이너를 생성하기 떄문에 반드시 이미지를 만드는 과정이 필요하다.
    3) Dockerfile을 이용하여 이미지를 만들고 처음부터 재현 가능하다.
    4) 빌드 서버에서 이미지를 만들면 해당 이미지를 이미지 저장소(허브)에 저장하고 운영서버에서 이미지를 불러와 사용한다.
- 설정관리
    1) 도커에서 설정은 보통 아래와 같이 환경변수로 제어한다.
    2) MYSQL_PASS=password와 같이 컨테이너를 띄울 때 환경변수를 같이 지정한다.
    3) 하나의 이미지가 환경변수에 따라 동적으로 설정파일을 생성하도록 만들어져야한다.
- 자원관리
    1) 컨테이너는 삭제 후 새로 만들면 모든 데이터가 초기화된다. (제거가 쉽다.)
    2) 그러므로 저장이 필요하다면, 업로드 파일을 외부 스토리지와 링크하여 사용하거나 S3같은 별도의 저장소가 필요하다.
    3) 세션이나 캐시를 memcached나 redis와 같은 외부로 분리한다.

### Dockerfile

도커 이미지를 만들기 위해서는 Dockerfile 을 생성 해야 한다.

Dockerfile 예제

```bash

# 베이스 이미지 이며 이미지 이름 앞에 아무것도 없으면 docker hub에서 가져온다.
FROM python:3.8-slim

# 컨테이너 안에 작업 폴더를 설정한다.
WORKDIR /app

# 로컬 현대 폴더의 모든 값을 컨테이너 /app 폴더에 복사한다.
ADD . /app

# pyhon의 경우 library를 requirements.txt에 기술을 하였고 RUN 명령어를 # 사용하여 아래 구문을 실행한다.

RUN pip install -r requirements.txt

# 기본 이미지는 대부분 GMT+0 기준으로 생성되어 한국 시간으로 변경 해준다
RUN ln -snf /usr/share/zoneinfo/Asia/Seoul /etc/localtime && echo Asia/Seoul > /etc/timezone

# 컨테이너 외부에서 접속할 포트를 적어준다.
EXPOSE 40003

# 컨테이너 로딩시에 사용하는 명령어를 기입한다. 
# EntryPoint와 비슷 하나 CMD를 사용하면 docker run시에 파라미터 설정 가능하다.
CMD ["python", "app.py"]
```





