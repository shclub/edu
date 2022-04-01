# Chapter 2 답안

<br/>

## 과제 1. 
docker compose로 구성한 mysql container  접속하여 로그인 한 후 wordpress db에 customer 테이블을 생성해 본다.

<br/>

## 과제2
mysql container  접속하여 로그인 한 후 wordpress db에 
아래 테이블 script를  로컬에 저장된 화일을 사용하여 test 테이블을 생성해 본다.    

https://github.com/shclub/edu1/blob/master/test.sql 화일을 다운 받는다.  


TIP : 화일 이동 방법은 cp 명령어 사용.  

호스트 -> 컨테이너
```bash
docker cp [host 파일경로] [container name]:[container 내부 경로]
```
컨테이너 -> 호스트  

```bash
docker cp [container name]:[container 내부 경로] [host 파일경로]
```
<br/>

###  < 답안 >

<br/>

해당 폴더에 test.sql 화일이 있는지 확인 하고 컨테이너 중에 mysql를 찾는다.  

```bash
ls
docker ps
```

docker cp 명령어를 사용하여 test.sql 화일을 컨테이너의 / 폴더에 복사한다.  
해당 컨테이너 안으로 들어간다.  
    
```bash
docker cp test.sql f52ae4e0b53a:/
docker exec -it f52ae4e0b53a /bin/sh
```  

컨테이너안에서 명령어로 mysql에 접속한다.  
id/패스워드는 docker-compose.yml에서 확인 가능  

```bash
mysql -u wordpress -p
Enter password:
```  

databases 목록을 확인하고 원하는 db를 선택하고   
source 명령어를 사용하여 sql문을 실행한다.   

```bash
show databases;
use wordpress;
source test.sql;
```   
테이블 목록을 확인하면 test 테이블이 생성된 걸 확인 할수있고 데이터를 조회해본다.  
    
```bash
show tables;
select * from test;
```  

<img src="./assets/chapter2-2-answer.png" style="width: 80%; height: auto;">  

<img src="./assets/chapter2-2-answer2.png" style="width: 80%; height: auto;">

