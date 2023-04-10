# 본 과정에 대해  ( 초급 과정 )
 
본 교육 과정은 초급 아키텍트 양성 과정으로 kt 카라반 학습체와 연동하여 
실습 위주로 진행을 하며 직접 설치부터 설정 및 활용까지 수행한다.   

문의 :  이석환 ( seokhwan.lee@kt.com / shclub@gmail.com )

<br/>

1. Chapter 1 : 1주차  ( [가이드 문서보기](./chapter1.md) )  

     - VM 기반으로 Jenkins 설치 및 설정 , GitHub , Docker 계정 생성 , Jenkins Pipeline 생성하여 CI 실습  
     - 샘플 소스 : [ 소스 보기 ](https://github.com/shclub/edu1)  

     <br/>
2. Chapter 2 : 2주차  ( [가이드 문서보기](./chapter2.md) ) 

     - git 설치 및 활용 
     - Docker 이해 및 활용 
     - Swagger 실습 
     - Docker Compose 설치 및 활용 ( DB 연동 )  
     - 샘플 소스 : [ 소스 보기 ](https://github.com/shclub/edu2)  

     <br/>

3. Chapter 3 : 3주차   ( [가이드 문서보기](./chapter3.md) )  

     - kubernetes 설치 (k3s) 및 설정 , k8s 이해 및 활용
     - kubernetes IDE 인 Lens 설치 및 사용법 실습   
     - Helm 설치 및 helm으로 prometheus 설치 활용
     - k8s hands-on Basic [ Hands-On 문서보기 ](./k8s_basic_hands_on.md)  

          - 실습 전체 개요
          - kubeconfig 설정 : kubectl 설치
          - kubectl 활용
          - kubernetes 리소스 ( Pod , Service , Deployment 생성 및 삭제)
          - 배포 ( Rolling Update / Rollback )
          - Serivce Expose ( Ingress )  

     <br/>

4. Chapter 4 : 4주차   ( [가이드 문서보기](./chapter4.md) ) 

     - GitOps 설명 
     - ArgoCD 설치 및 설정 
     - kustomize 설명 및 실습
     - k8s에 배포 실습 ( Blue/Green , Canary )  
     - ArgoCD Hands-on [ Hands-On 문서보기 ](./argocd_hands_on.md) 

          - kubectl plugin 설치
          - Blue/Green 배포
          - Canary 배포
          - ArgoCD 계정 추가 및 권한 할당
          - kustomize 사용법
          - ArgoCD remote Cluster 에서 배포 하기 

     <br/>

5. Chapter 5 : 5주차   ( [가이드 문서보기](./chapter5.md) ) 

     - kt cloud 하드디스크 추가
     - k3s 위치 변경
     - Github Action , workflow 사용 ( GoodBye Jenkins )
     - 모니터링 솔루션 연동 ( Datadog ) 
     - 샘플 소스 : [ 소스 보기 ](https://github.com/shclub/edu7)  

     <br/>

6. Chapter 6 : 6주차    ( [가이드 문서보기](./chapter6.md) ) 

     - SpringBoot 개념 설명 
     - IDE 개발 환경 구성 
     - SpringBoot 전체 hands-on [ Hands-On 문서보기 ](./springboot_hands_on.md)   

          - 뷰 템플릿 과 MVC 패턴
          - JDBC vs JPA vs Mybatis vs Spring Data JDBC 비교
          - Spring Data JPA hands-on
          - Rest API 와 JSON
          - HTTP 와 Rest Controller
          - 서비스와 트랜잭션, 그리고 롤백
          - Spring MyBatis hands-on
          - Spring Data JDBC hands-on
          - 테스트 작성하기
          - 댓글 서비스 만들기
          - IoC 와 DI
          - AOP
          - Object Mapper   
          - PSA ( Portal Service Abstraction )
          
     - SpringBoot Data JPA hands-on [ Hands-On 문서보기 ](./springboot_hands_on_jpa.md)  

          - 데이터 생성 with JPA
          - 롬복과 리팩토링
          - 데이터 조회 , 수정 및 삭제 with JPA
          - CRUD 와 SQL Query
          - QueryDSL 사용

     - 샘플 소스 ( Web ): [ 소스 보기 ](https://github.com/shclub/edu9)  
     - 샘플 소스( Rest ) : [ 소스 보기 ](https://github.com/shclub/edu9-1)

     <br/>

7. Chapter 7 : 7주차  

     - SpringBoot 실습  
     - SpringBoot 전체 hands-on [ Hands-On 문서보기 ](./springboot_hands_on.md)   

     - Mybatis hands-on [ Hands-On 문서보기 ](./springboot_hands_on_mybatis.md) 

          -  프로젝트 생성 및 환경 설정
          -  프로젝트 구성하기 
          -  실행해 보기
          -  SQL문 로그 보기 ( log4jdbc 설정 )
          -  Swagger 설정 ( 3.0 )  

     - 샘플 소스 ( mybatis ): [ 소스 보기 ](https://github.com/shclub/edu10)  

     - 샘플 소스 ( mybatis / log4j2 / OpenApi ): [ 소스 보기 ](https://github.com/shclub/edu10-1)

     - Spring Data JDBC hands-on [ Hands-On 문서보기 ](./springboot_hands_on_spring_data_jdbc.md) 
  
          -  Spring Data JPA vs Spring Data JDBC 
          -  프로젝트 구성하기 ( CRUD , Paging & Sorting )
          -  실행해 보기
          -  로컬 캐쉬 ( Caffeine Cache ) 사용하기 

     - 샘플 소스 ( Spring Data Jdbc :  ): [ 실습 소스 보기 ](https://github.com/shclub/edu11)
     - 샘플 소스 ( Spring Data Jdbc ): [ 전체 소스 보기 ( CUD 포함) ](https://github.com/shclub/edu11-1)
     
      
     <br/>

8. Chapter 8 : 8주차  

     - SpringBoot 마지막

          -  Spring Security [ Hands-On 문서보기 ](./security.md)    
               - JWT / OAuth 연동
               - Oauth 2.0 ( 향후 )
               - Frontend ( React ) simple example  

          -  Annotation 정리 
          -  테스트 작성하기
          -  댓글 서비스 만들기
          -  IoC 와 DI
          -  AOP
          -  Object Mapper   
          -  PSA ( Portal Service Abstraction ) 
          -  Webflux : 향후
 
     - k8s 배포 실습 ( SpringBoot )
     - DataDog 연동 ( SpringBoot ) : 향후
     
     
     - 샘플 소스 ( 배포 yaml ): [ 소스 보기 ](https://github.com/shclub/edu12)
     - 샘플 소스 ( React Front without security ): [ 소스 보기 ](https://github.com/shclub/edu12-1)
     - 샘플 소스 ( SpringBoot Front without security ): [ 소스 보기 ](https://github.com/shclub/edu12-2)
     - 샘플 소스 ( React Front with security ): [ 소스 보기 ](https://github.com/shclub/edu12-3)
     - 샘플 소스 ( React Front with security ): [ 소스 보기  ](https://github.com/shclub/edu12-4)

     <br/>

# 중급 과정

 
중급 과정에서는 kubernetes 기반으로 좀더 난이도가 있는 과정이며, 실제 프로젝트에서 유용한 기술 위주로 교육 예정 

<br/>

9. Chapter 9 : 9주차  ( [가이드 문서보기](./chapter9.md) )  
     - OKD 4.7 사용법 ( kt cloud )
     - k8s에 Jenkins 설치 및 설정 ( OKD 4.7 )
     - Jenkins Master/Slave 구현, GitHub , Podman 연동 
     - skaffold를 사용한 Jenkins 빌드 Pipeline 생성하여 CI 실습 
     - RBAC 실습 
 
     <br/>

10. Chapter 10 : 10주차

     - k8s hands-on 중급 [ Hands-On 문서보기 ](./k8s_middle_hands_on_2023.md)  

          - Storage Volume  ( PV/PVC , DB 설치 + NFS )
          - NFS 라이브러리 설치 ( Native Kubernetes )
          - Service - Headless, Endpoint, ExternalName
          - Helm 배포  ( Umbrella 패턴 )
          - ArgoCD 배포  ( Apps-of-Apps 패턴 )

     <br/>

11. Chapter 11 : 11주차

     - 인증 (SSO) [ Hands-On 문서보기 ](./chapter11.md) 
          - KeyCloak 설치 및 실습
          - keyCloak 를 이용한 오픈 소스 시스템 연동 ( Jenkins / ArgoCD , Airflow , Kibana 등 )
          - Spring Backend 연동

     <br/>


12. Chapter 12 : 12주차

     - AWS 기본 [ Hands-On 문서보기 ](./chapter12.md) 
          - 기본 설명 ( VPC / Subnet / RoutingTable , N / NLCM / Security Group / ENI ) 및 실습  
          - EKS 설명 및 생성 ( By 포탈 / CloudFormation / eskctl ) 실습  
          - EKS 설정 ( Security Group / ALB Ingress / Route53 )
          - 기본 서비스 배포 실습 (/w Fargate )

     <br/>

12. Chapter 13 : 13주차
     - AWS 기반 솔루션 설치 [ Hands-On 문서보기 ](./chapter13.md) 
          - Istio 설명, 설치 및 실습
          - Spring Cloud Gateway 설명, 설치 및 실습
          - HashiCorp Consul 설치 및 실습

     <br/>

13. Chapter 14 : 14주차
     - AWS 기본 [ Hands-On 문서보기 ](./chapter14.md) 
          - Redis/Kafka 설치 및 활용 
          - S3 Object Storage 설명 및 활용 실습
          - AWS CodeCommit / CodeDeploy / ECR 사용 방법 실습
          - ArgoCD 설치 ( 미정 )  

     <br/>

14. Chapter 15 : 15주차
     - SpringBoot 심화 학습  
          - Spring Cloud Stream
     - MSA 패턴 실습
          - API GW
          - SAGA
          - CQRS

     <br/>

15. Chapter 16 : 16주차 
     - Prometheus 심화 
     - Airflow ( 배치프레임웍 ) 설치 및 실습

     <br/>

16. Chapter 17 : 17주차 
     -  Summary

     <br/>
