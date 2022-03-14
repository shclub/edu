# < 1일차 >
--- 
Jenkins 를 설치하고 CI 환경을 구성한다.

 ##  1.1 kt cloud 에서 VM ( Ununtu ) 을 생성한다. 
  기준 : 16 core , 32G
  ```
   jenkins , k3s , ArgoCD , Pinpoint 설치 예정
  ```

   ###  1) Port Forwarding 설정
   ssh : 22 -> 22222
   jenkins : 9000 -> 9000
   flask web : 8080 -> 40003
   k8s  : 8443 -> 8443
   
   ###  2) 터미널 프로그램으로 서버에 접속하고 비밀번호를 변경한다.
   >ssh root@ip -p 22222
   >passwd
    old password:
    new password:
   
   
   ###  3) Jenkins를 설치한다 ( https://velog.io/@ifthenelse/jenkins-설치하기-ubuntu-20.04 )
    3.1) 저장소 키 다운로드
   ```
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   echo deb http://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list
   ```
   3.2) 패키지 인덱스를 업데이트 하고 라이브러를 최신 버전으로 올려준다.
   ```
   apt-get update && apt-get upgrade
   ```
   3.3) root 계정이라면 아래와 같이 Jenkins 를 설치한다.  
   ```
   apt-get install jenkins
   ```
   
   일반 계정이면 앞에 sudo 명령어 사용
   ```
   sudo apt-get install jenkins
   ```
   
   3.4) 서비스 포트를 변경한다.
   vi /etc/default/jenkins
   ```
   #아래 부분을 사용할 포트로 변경. 우리는 8080 ->  9000으로 변경
   HTTP_PORT=9000
   ```
   
   < vi에디터 사용법 >
   i : 데이터 입력모드
   esc : 명령모드
    / : 찾기
    x : 한글자 삭제
    dd : 한 라인 삭제
    :wq : 저장하고 나오기
    :q! : 저장안하고 나오기
    :set nocp : 라인 밀리는 현상 방지
    
    
   3.5) 서비스 재시작 및 상태 확인
   ```
   service jenkins restart
   # 정상여부 확인
   systemctl status jenkins
   ```
   
   3.6) admin 초기 패스워드 확인 및 복사
   cat /var/lib/jenkins/secrets/initialAdminPassword
   
   3.7) 젠킨스 서버 접속
   이제 브라우져로 http://(본인서버ip):9000으로  접속하면 아래와 같은 화면이 나온다.
   패스워드에 위 명령으로 확인한 문자열을 입력한다.
   Install Suggested Plugin 선택하고 Plugin 설치 한다. 아래 와 같이 화면이 나오면 성공.

   ###  4) Docker 를 설치한다 ( https://shanepark.tistory.com/237 )

   4.1) 패키지 인덱스 업데이트
   ```
   apt-get update
   ```
   4.2) HTTPS를 통해 repository 를 이용하기 위해 package 들을 설치
   ```
   apt-get -y install  apt-transport-https ca-certificates curl gnupg lsb-release
   ```
   4.3) Docker의 Official GPG Key 를 등록합니다.
   ```
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```
   4.4) stable repository를 등록합니다.
   ```
   echo \
    "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```  
   4.5) 도커를 설치합니다.
   ```
   apt-get update && apt-get install docker-ce docker-ce-cli containerd.io
   ```
   4.6) 도커 버전을 확인합니다.
   ```
   docker --version
   ```
   4.6) 도커 이미지 다운로드 및 실행하기
   ```
   docker run hello-world
   ```
   
   ###  5) GitHub 계정을 생성한다. 

   5.1) https://github.com/ 접속하고 계정 생성 ( 향후 사내에서 개발시는 d-space GitLab 사용 )
   5.2) Repository를 하나 생성한다.
   5.3) https://github.com/shclub/edu 폴더의 파일을 복사하여 본인이 생성한 Repository에 신규 화일을 생성한다.
   * 샘플은 pyhon flask 로 구성
   
   ###  6) Docker Hub 계정을 생성한다. 

   6.1) https://hub.docker.com/ 접속하고 계정 생성 ( 향후 사내에서 개발시는 d-space Nexus 사용 )
   6.2) Docker push 테스트를 한다.
        ```
        docker tag hello-world (본인id)/hello-world
        docker push (본인id)/hello-world
        ```
        도커허브 본인 계정에서 도커 이미지 생성 확인
   
   ###  7) Jenkins 설정 ( https://cwal.tistory.com/21 )

   7.1) 일반 사용자 계정을 생성한다.
   7.2) 파이프 라인을 구성한다.

   






