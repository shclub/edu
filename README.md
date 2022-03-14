# < php base image 생성 >

 

> appdu 에서 사용되는 php base image 를 생성 관련 설명이다.

 

 

 

 

# << 관련 docker image >>

 

# 1. 공식 이미지

 

```

php:5.6-apache (123MB) <-- apache + php 가 포함되어 있음.

php:5.6-fpm    (119MB) <-- php-fpm만 존재함. nginx와 같은 webserver를 추가셋팅해야 함.

```

 

 

 

# 2. bastion image

 

## 1) php bastion images

 

```

ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-apache-8080-1   <-- 이전

 

ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-apache-v1.0     <-- 최종

ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-fpm-nginx-v1.1  <-- 최종

```

 

 

 

## 2) tag 별 변동내역 정리

 

### php:5.6-apache

 

```

ㅇ php:5.6-apache-8080 --376MB

  - from php:5.6-apache

  - 8080 port changed

  - php-ext-install mysql

  - util

    . vim-tiny curl ping

   

ㅇ php:5.6-apache-8080-1

  - from php:5.6-apache-8080

  - util

    . git nc vi

   

ㅇ php:5.6-apache-v1.0

  - from php:5.6-apache-8080-1

  - 보안취약점 반영

```

 

 

 

### php:5.6-fpm-nginx

 

```

ㅇ php:5.6-fpm-nginx-v1.0

  - from php:5.6-fpm

  - util

    . vim netcat git net-tools iputils-ping

  - php 설정

  - nginx 설치 및 php-fpm 연동 설정

  - 보안취약점 반영

  - nginx user생성

  - ENTRYPOINT ["/bin/bash", "-c", "service nginx start && php-fpm"]

 

ㅇ php:5.6-fpm-nginx-v1.1

  - from php:5.6-fpm-nginx-v1.0

  - mariadb lib 설치

  - /etc/secret 설정   

```

 

 

 

 

 

---

 

 

 

# << php:5.6-apache 생성 >>

 

> -  php:5.6-apache image 를 기반으로 시작

> 

> - utility 등 각종 설치

> 

> - KT 보안 취약점 관련사항 설정

> 

> - 최종 docker image 생성

 

 

 

# 1. from  image 실행

 

```bash

$ docker run --name php56_apache -d -p 8081:8080 ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-apache-8080-1

 

$ docker rm -f php56_apache

 

$ winpty docker exec -it php56_apache bash

 

$ curl localhost:8081

 

$ php -i

$ php -v

PHP 5.6.40 (cli) (built: Jan 23 2019 00:16:13)

```

 

 

 

# 2. utility 설치

 

## 1) update

 

```

$ apt update && upgrade -y   

 

※ Temporary failure resolving 'security.debian.org'  에러 발생할경우

$ echo "nameserver 8.8.8.8" | tee /etc/resolv.conf > /dev/null

```

 

 

 

## 2) utility

 

```

$ apt install -y \

    curl         \

    iputils-ping \

    vim-tiny     \

    net-tools    \

    git

```

 

 

 

# 3. php 설정

 

## 1) php.ini 설정

 

- php.ini   <--- php.ini-production 파일을 복사하여 편집한다.

 

```bash

$ cd /usr/local/etc/php/

$ cp php.ini-production ./conf.d/php.ini

$ vi /usr/local/etc/php/conf.d/php.ini

```

 

 

 

 

- SET

 

```bash

date.timezone = Asia/Seoul

expose_php = Off

post_max_size = 8M

upload_max_filesize = 2M

memory_limit = 128M

cgi.fix_pathinfo=0       <-- 보안을 위해

```

 

 

 

# 4. apache 설정

 

 

## 1) charset

 

```bash

$ cd /etc/apache2

 

$ vi $APACHE_CONFDIR/conf-enabled/charset.conf

 

AddDefaultCharset UTF-8    <-- 추가

```

 

 

 

## 2) security.conf

 

- kt 보안취약점 관련 조치

 

```xml

$ vi $APACHE_CONFDIR/conf-enabled/security.conf

 

# 기타설정

ServerTokens OS    --> ServerTokens Prod

ServerSignature On --> ServerSignature Off

 

 

# allow only GET/POST method

<Location />           

  <LimitExcept GET POST>

    Order deny,allow   

    Deny from all      

  </LimitExcept>       

</Location>        

```

 

 

 

 

 

## 3) 서비스 확인

 

- 확인

 

```

$ netstat -nplt

8080 확인

 

$ curl localhost:8080

this is health check

```

 

 

 

 

 

# 5.기타 설정

 

 

 

## 1) bashrc

 

```bash

# .bashrc 수정

$ vi ~/.bashrc

--> ll 명령 수행 가능하도록 설정

 

$ source ~/.bashrc

```

 

 

 

##  2) mariadb lib 설치

 

```bash

$ docker-php-ext-install mysqli

```

 

 

 

## 3) secret 설정

 

- deployment 로 배포될때 secret 으로 연결되므로 아래 값들은 배포시 변경됨.

 

```bash

mkdir -p "/etc/secret"

echo "jgsysyksr897213579"   > "/etc/secret/MY_KEY"

echo "cz-okd-dev-appdu-mediator-dev-deploy-mariadb.appdu.svc" > "/etc/secret/DB_HOST"

echo "panthree"             > "/etc/secret/DB_NAME"

echo "panthree"             > "/etc/secret/DB_USER"

echo "panthree"             > "/etc/secret/DB_PASSWORD"

echo "3306"                 > "/etc/secret/DB_PORT"

```

 

 

 

 

 

 

 

# 6.  docker 마무리작업

 

## 1) commit

 

```bash

$ docker commit php56_apache php:5.6-apache-v1.0

 

 

# 최종 확인

$ docker run -d --name php56_apache -p 8081:8080 php:5.6-apache-v1.0

 

$ docker rm -f php56_apache

 

$ curl locahost:8081

this is health check

```

 

 

 

## 2) bastion upload

 

```bash

$ docker tag php:5.6-apache-v1.0  ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-apache-v1.0

 

$ docker push ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-apache-v1.0

```

 

 

 

 

 

## 3)  docker run 최종확인

 

```bash

$ docker run --name php56_apache -d -p 8081:8080 ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-apache-v1.0

 

$ winpty docker exec -it php56_apache bash

 

$ curl localhost:8081

 

$ docker rm -f php56_apache

```

 

 

 

 

 

---

 

 

 

 

 

# << php:5.6-fpm-nginx >>

 

> - php-fpm image 를 기반으로 시작

> - utility 등 각종 설치

> - nginx install

> - 각종 설정작업

> - KT 보안 취약점 관련사항 설정

> - php-fpm / nginx 연동 설정

> - 최종 docker image 생성

 

 

 

# 1. from  image 실행

 

```bash

$ docker run --name php56 -d php:5.6-fpm

 

$ docker rm -f php56

 

$ docker exec -it php56 bash

 

$ php -i

 

$ php -v

PHP 5.6.40 (cli) (built: Jan 23 2019 00:16:13)

```

 

 

 

 

 

# 2. utility 등 각종 설치

 

## 1) update

```bash

$ apt-get update -y

$ apt-get upgrade -y

 

※ Temporary failure resolving 'security.debian.org'  에러 발생할경우

 

$ echo "nameserver 8.8.8.8" | tee /etc/resolv.conf > /dev/null

```

 

 

 

 

 

## 2) utility

 

```bash

$ apt-get install -y vim netcat git net-tools iputils-ping

```

 

 

 

 

 

# 3.  php 설정

 

 

 

## 1) php.ini 설정

 

- php.ini   <--- php.ini-production 파일을 복사하여 편집한다.

 

```bash

$ cd /usr/local/etc/php/

$ cp php.ini-production ./conf.d/php.ini

$ vi /usr/local/etc/php/conf.d/php.ini

```

 

 

 

 

- SET

 

```

date.timezone = Asia/Seoul

expose_php = Off

post_max_size = 8M

upload_max_filesize = 2M

memory_limit = 128M

cgi.fix_pathinfo=0       <-- 보안을 위해

```

 

 

 

 

 

## 2) php-fpm.conf 설정

 

- php-fpm.conf

 

```bash

$ vi /usr/local/etc/php-fpm.conf

$ vi /usr/local/etc/php-fpm.d/www.conf

```

 

 

 

 

- SET

 

```

daemonize = yes

user = nginx

group = nginx

listen.owner = nginx

listen.group = nginx

listen.mode = 0660

listen = 127.0.0.1:9000     <-- nginx.conf 에서의 설정과 동일해야 함.

```

 

 

 

 

 

 

 

# 4.  nginx 설치 및 연동

 

## 1) nginx 설치

```bash

$ apt-get install nginx

 

$ nginx -v

nginx version: nginx/1.10.3

```

 

 

 

## 2) nginx.conf 설정

- SET

```bash

$ vi /etc/nginx/nginx.conf

 

# user nginx;

ssl_protocols TLSv1.2; # Dropping SSLv3, ref: POODLE   <-- TLSv1 TLSv1.1  제거

```

 

 

 

 

 

## 3) default (default.conf) 설정

 

```bash

$ vi /etc/nginx/sites-enabled/default

```

 

- SET

```

server {

        listen 8080 default_server;

        listen [::]:8080 default_server;

 

        # SSL configuration

        #

        # listen 443 ssl default_server;

        # listen [::]:443 ssl default_server;

        #

        # Note: You should disable gzip for SSL traffic.

        # See: https://bugs.debian.org/773332

        #

        # Read up on ssl_ciphers to ensure a secure configuration.

        # See: https://bugs.debian.org/765782

        #

        # Self signed certs generated by the ssl-cert package

        # Don't use them in a production server!

        #

        # include snippets/snakeoil.conf;

 

        root /var/www/html;

 

        # Add index.php to the list if you are using PHP

        index index.php index.html index.htm index.nginx-debian.html;

 

        server_name localhost;

 

        charset utf-8;

 

        error_page 400 401 402 403 404 /40x.html;

        location = /40x.html {

          root /usr/share/nginx/html;

        }

 

        error_page 500 502 503 504 /50x.html;

        location = /50x.html {

          root /usr/share/nginx/html;

        }

        # Android App

        location ^~ /m/ {

                include snippets/fastcgi-php.conf;

                fastcgi_pass 127.0.0.1:9000;

        }

       

        # Web

        location = /index.php {

            fastcgi_pass   127.0.0.1:9000;

            fastcgi_index  index.php;

            fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;

            include fastcgi.conf;

        }

 

 

        location / {

            try_files $uri $uri/ /index.php?url=$request_trimmed&$args;

        }

 

 

        # concurs with nginx's one

        location ~ /\.ht {

                deny all;

        }

}

      

```

 

 

 

 

 

 

## 4) 보안취약점 관련 조치(40x 50x)

 

 

- /usr/share/nginx/html/index.html 참조해서 40x.html  50x.html 생성

 

```bash

$ cat > /usr/share/nginx/html/40x.html

---

<!DOCTYPE html>

<html>

<head>

<title>40x Error occurred!</title>

<style>

    body {

        width: 35em;

        margin: 0 auto;

        font-family: Tahoma, Verdana, Arial, sans-serif;

    }

</style>

</head>

<body>

<h1>40x Error occurred!</h1>

<p>40x Error occurred!</p>

 

</body>

</html>

---

 

 

$ cat > /usr/share/nginx/html/50x.html

---

<!DOCTYPE html>

<html>

<head>

<title>50x Error occurred!</title>

<style>

    body {

        width: 35em;

        margin: 0 auto;

        font-family: Tahoma, Verdana, Arial, sans-serif;

    }

</style>

</head>

<body>

<h1>50x Error occurred!</h1>

<p>50x Error occurred!</p>

 

</body>

</html>

---

```

 

 

 

## 5) user 생성

 

```bash

$ adduser nginx

$ chown -R nginx:nginx /var/www

$ chmod -R g+rw /var/www

```

 

 

 

## 6) 서비스 확인

 

- 서비스 실행

 

```bash

$ service nginx stop

$ service nginx start

 

$ php-fmp

```

 

 

- 두개서비스 한꺼번에 실행하려면...

 

```bash

$ bash -c "service nginx start && php-fpm"

```

 

 

- 확인

 

```bash

$ netstat -nplt

9000 확인

8080 확인

```

 

 

 

 

 

# 5.  기타 설정

 

## 1) bashrc

 

 

```bash

# .bashrc 수정

$ vi ~/.bashrc

--> ll 명령 수행 가능하도록 설정

 

$ source ~/.bashrc

```

 

 

## 2) mariadb lib 설치

```bash

$ docker-php-ext-install mysqli

```

 

 

## 3) secret 설정

- deployment 로 배포될때 secret 으로 연결되므로 아래 값들은 배포시 변경됨.

```bash

mkdir -p "/etc/secret"

echo "jgsysyksr897213579"   > "/etc/secret/MY_KEY"

echo "cz-okd-dev-appdu-mediator-dev-deploy-mariadb.appdu.svc" > "/etc/secret/DB_HOST"

echo "panthree"             > "/etc/secret/DB_NAME"

echo "panthree"             > "/etc/secret/DB_USER"

echo "panthree"             > "/etc/secret/DB_PASSWORD"

echo "3306"                 > "/etc/secret/DB_PORT"

```

 

 

 

 

 

# 6.  docker 마무리작업

 

## 1) commit

 

```bash

$ docker commit php56 php:5.6-fpm-nginx-v1.0-temp

 

# 확인

$ docker run -d --name php56-fpm-nginx -p 8081:8080 php:5.6-fpm-nginx-v1.0-temp

 

# 확인2 : 두개서비스를 한거번에 실행 테스트

$ docker run -d --name php56-fpm-nginx -p 8081:8080 php:5.6-fpm-nginx-v1.0-temp /bin/bash -c 'service nginx start && php-fpm'

```

 

 

 

## 2) build

두개 서비스 동시 실행을 entrypoint 로 포함시키는 image 생성

```bash

$ cat > Dockerfile

---

FROM php:5.6-fpm-nginx-v1.0-temp

ENTRYPOINT ["/bin/bash", "-c", "service nginx start && php-fpm"]

---

 

$ docker build -t php:5.6-fpm-nginx-v1.0 .

```

 

 

 

## 3) 확인

 

```bash

$ docker run -d --name php56-fpm-nginx -p 8081:8080 php:5.6-fpm-nginx-v1.0

 

$ docker ps -a

 

$ winpty docker exec -it php56-fpm-nginx bash

 

$ curl localhost:8081

 

$ docker rm -f php56-fpm-nginx

```

 

 

 

## 4) bastion upload : tag & push

 

```bash

$ docker tag php:5.6-fpm-nginx-v1.0  ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-fpm-nginx-v1.0

 

$ docker push ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-fpm-nginx-v1.0

$ docker push ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-fpm-nginx-v1.1

 

```

 

 

 

 

 

## 5)  docker run 최종확인

 

```bash

$ docker run --name php56-fpm-nginx -d -p 8081:8080  ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-fpm-nginx-v1.0

 

$ winpty docker exec -it php56-fpm-nginx bash

 

$ curl localhost:8081

 

$ docker rm -f php56-fpm-nginx

```

 

 

 

 

 

---

 

 

 

 

 

# << template image 사용 >

 

- template 사용시 아래와 같이 제공됨

 

```yaml

# php-apache 사용시

#FROM ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-apache-v1.0

 

# php-nginx 사용시

FROM ktis-bastion01.container.ipc.kt.com:5000/arsenal/php:5.6-fpm-nginx-v1.1

 

# src copy

COPY src/ /var/www/html/

```

 

- 사용자의 needs 에 따라 FROM IMAGE 를 선택

