#  Security 
 
React 와 SpringBoot를 연동하면서 security 활용 방법에 대해서 실습한다.  


1. Spring Security

1. 프로젝트 생성 및 환경 설정

3. 실행해보기

4. 소스위치   
  - 배포 : https://github.com/shclub/edu12
  - 보안 없음
    - React ( 사원 정보 ) :https://github.com/shclub/edu12-1
    - SpringBoot ( 사원 정보 ) : https://github.com/shclub/edu12-2
  - 보안 추가
    - React ( 사원 정보 ) : https://github.com/shclub/edu12-3
    - SpringBoot ( 사원 정보 ) : https://github.com/shclub/edu12-4

5. 참고
  - Sprint Security 
    - https://doozi0316.tistory.com/entry/Spring-Security-Spring-Security%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95?category=925594
    - https://coding-start.tistory.com/153  

  - react + springboot 예제 : https://velog.io/@dsunni/Spring-Boot-React-JWT로-간단한-로그인-구현하기  
  - react + springboot 예제 
    - https://github.com/hch0821/spring-security-and-react-integration  

    - https://dzone.com/articles/integrating-spring-boot-and-react-with-spring-secu-1
  - filter vs interceptor : https://mangkyu.tistory.com/173
  - cors : https://devlog-wjdrbs96.tistory.com/429
  - jasypt : https://kitty-geno.tistory.com/160
  - jwt + oauth 2.0 : https://velog.io/@tmdgh0221/Spring-Security-%EC%99%80-OAuth-2.0-%EC%99%80-JWT-%EC%9D%98-%EC%BD%9C%EB%9D%BC%EB%B3%B4
  - 토큰인증 vs 세션 인증 : https://mangkyu.tistory.com/55
  - React + SpringBoot + oauth2 :  https://dodop-blog.tistory.com/249
  
<br/>

## Spring Security

<br/>

### Spring Security ?

<br/>


우리는 그동안 Front ( Talend API ) 를 통해서 SpringBoot Backend를 통한 Rest API를 호출 하였습니다.  

이번 Chapter 에서는 Spring Security 를 사용하여 로그인 인증 및 인가 하는 실습을 진행을 합니다.  

Spring Security는 Spring 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 스프링 하위 프레임워크이다.  

Spring Security는 '인증'과 '권한'에 대한 부분을 Filter 흐름에 따라 처리하고 있다.  
Filter는 Dispatcher Servlet으로 가기 전에 적용되므로 가장 먼저 URL 요청을 는다.  

Spring Security는 보안과 관련해서 체계적으로 많은 옵션을 제공해주기 때문에 개발자 입장에서는 일일이 보안관련 로직을 작성하지 않아도 된다는 장점이 있다.  

<br/>

[ 보안 용어 ]

- 인증(Authentication): 해당 사용자가 본인이 맞는지를 확인하는 절차

- 인가(Authorization): 인증된 사용자가 요청한 자원에 접근 가능한지를 결정하는 절차   

Spring Security는 기본적으로 인증 절차를 거친 후에 인가 절차를 진행하게 되며, 인가 과젱에서 해당 리소스에 대한 접근 권한이 있는지 확인을 하게 된다.  

Spring Security에서는 이러한 인증과 인가를 위해 Principal을 아이디로, Credential을 비밀번호로 사용하는 Credential 기반의 인증 방식을 사용한다. 


- Principal(접근 주체): 보호받는 Resource에 접근하는 대상
- Credential(비밀번호): Resource에 접근하는 대상의 비밀번호


<br/>

스프링 시큐리티는 스프링 기반의 애플리케이션의 보안(인증과 권한,인가 등)을 담당하는 스프링 하위 프레임워크이다.  

주로 서블릿 필터와 이들로 구성된 필터체인으로의 위임모델을 사용한다.  

그리고 보안과 관련해서 체계적으로 많은 옵션을 제공해주기 때문에 개발자 입장에서는 일일이 보안관련 로직을 작성하지 않아도 된다.  

<br/>

Spring Security의 아키텍쳐는 아래와 같다.  


<img src="./assets/spring_security0.png" style="width: 100%; height: auto;"/> 

<br/>

위의 그림은 Form 기반 로그인에 대한 플로우를 보여주는 그림이다.  

- 사용자가 Form을 통해 로그인 정보를 입력하고 인증 요청을 보낸다.

- AuthenticationFilter(사용할 구현체 UsernamePasswordAuthenticationFilter)가 HttpServletRequest에서 사용자가 보낸 아이디와 패스워드를 인터셉트한다. 프론트 단에서 유효성검사를 할 수도 있지만, 무엇보다 안전! 안전을 위해서 다시 한번 사용자가 보낸 아이디와 패스워드의 유효성 검사를 해줄 수 있다.  
(아이디 혹은 패스워드가 null인 경우 등) HttpServletRequest에서 꺼내온 사용자 아이디와 패스워드를 진짜 인증을 담당할 AuthenticationManager 인터페이스(구현체 - ProviderManager)에게 인증용 객체(UsernamePasswordAuthenticationToken)로 만들어줘서 위임한다.  

- AuthenticationFilter에게 인증용 객체(UsernamePasswordAuthenticationToken)을 전달받는다.  

- 실제 인증을 할 AuthenticationProvider에게 Authentication객체(UsernamePasswordAuthenticationToken)을 다시 전달한다.  

- DB에서 사용자 인증 정보를 가져올 UserDetailsService 객체에게 사용자 아이디를 넘겨주고 DB에서 인증에 사용할 사용자 정보(사용자 아이디, 암호화된 패스워드, 권한 등)를 UserDetails(인증용 객체와 도메인 객체를 분리하지 않기 위해서 실제 사용되는 도메인 객체에 UserDetails를 상속하기도 한다.)라는 객체로 전달 받는다.  

- AuthenticationProvider는 UserDetails 객체를 전달 받은 이후 실제 사용자의 입력정보와 UserDetails 객체를 가지고 인증을 시도한다.  

- 인증이 완료되면 사용자 정보를 가진 Authentication 객체를 SecurityContextHolder에 담은 이후 AuthenticationSuccessHandle를 실행한다.(실패시 AuthenticationFailureHandler를 실행한다.)  

<br/>

뒤에서 실제 코드를 예를 들어서 설명할 것이다.

<br/>

### Filter

<br/>


필터(Filter)는 J2EE 표준 스펙 기능으로 디스패처 서블릿(Dispatcher Servlet)에 요청이 전달되기 전/후에 url 패턴에 맞는 모든 요청에 대해 부가작업을 처리할 수 있는 기능을 제공한다.  

디스패처 서블릿은 스프링의 가장 앞단에 존재하는 프론트 컨트롤러이므로, 필터는 스프링 범위 밖에서 처리가 되는 것이다.  

즉, 스프링 컨테이너가 아닌 톰캣과 같은 웹 컨테이너에 의해 관리가 되는 것이고(스프링 빈으로 등록은 된다), 디스패처 서블릿 전/후에 처리하는 것이다. 이러한 과정을 그림으로 표현하면 다음과 같다.


<img src="./assets/spring_security1.png" style="width: 80%; height: auto;"/> 

<br/>

[ 필터(Filter)의 메소드 ]  

필터를 추가하기 위해서는 javax.servlet의 Filter 인터페이스를 구현(implements)해야 하며 이는 다음의 3가지 메소드를 가지고 있다.

- init 메소드 : 필터 객체를 초기화하고 서비스에 추가하기 위한 메소드이다. 웹 컨테이너가 1회 init 메소드를 호출하여 필터 객체를 초기화하면 이후의 요청들은 doFilter를 통해 처리된다.  

- doFilter 메소드 : url-pattern에 맞는 모든 HTTP 요청이 디스패처 서블릿으로 전달되기 전에 웹 컨테이너에 의해 실행되는 메소드이다.  
doFilter의 파라미터로는 FilterChain이 있는데, FilterChain의 doFilter 통해 다음 대상으로 요청을 전달하게 된다. chain.doFilter() 전/후에 우리가 필요한 처리 과정을 넣어줌으로써 원하는 처리를 진행할 수 있다.  

- destroy 메소드 : 필터 객체를 서비스에서 제거하고 사용하는 자원을 반환하기 위한 메소드이다. 이는 웹 컨테이너에 의해 1번 호출되며 이후에는 이제 doFilter에 의해 처리되지 않는다.  

<br/>

### 디스패처 서블릿(Dispatcher Servlet) 

<br/>

디스패처 서블릿의 dispatch는 "보내다"라는 뜻을 가지고 있습니다. 그리고 이러한 단어를 포함하는 디스패처 서블릿은 HTTP 프로토콜로 들어오는 모든 요청을 가장 먼저 받아 적합한 컨트롤러에 위임해주는 프론트 컨트롤러(Front Controller)라고 정의할 수 있습니다.  

이것을 보다 자세히 설명하자면, 클라이언트로부터 어떠한 요청이 오면 Tomcat(톰캣)과 같은 서블릿 컨테이너가 요청을 받게 됩니다. 그리고 이 모든 요청을 프론트 컨트롤러인 디스패처 서블릿이 가장 먼저 받게 됩니다.   

`디스패처 서블릿은 공통적인 작업을 먼저 처리한 후에 해당 요청을 처리해야 하는 컨트롤러를 찾아서 작업을 위임합니다.`

여기서 Front Controller(프론트 컨트롤러)라는 용어가 사용되는데, Front Controller는 주로 서블릿 컨테이너의 제일 앞에서 서버로 들어오는 클라이언트의 모든 요청을 받아서 처리해주는 컨트롤러로써, MVC 구조에서 함께 사용되는 디자인 패턴입니다.

<br/>
디스패처 서블릿은 적합한 컨트롤러와 메소드를 찾아 요청을 위임해야 합니다.  
Dispatcher Servlet의 처리 과정을 살펴보면 다음과 같습니다.  

<img src="./assets/spring_security2.png" style="width: 80%; height: auto;"/> 

<br/>

1. 클라이언트의 요청을 디스패처 서블릿이 받음  

2. 요청 정보를 통해 요청을 위임할 컨트롤러를 찾음  

3. 요청을 컨트롤러로 위임할 핸들러 어댑터를 찾아서 전달함  

4. 핸들러 어댑터가 컨트롤러로 요청을 위임함  

5. 비지니스 로직을 처리함  

6. 컨트롤러가 반환값을 반환함  

7. HandlerAdapter가 반환값을 처리함  

8. 서버의 응답을 클라이언트로 반환함

<br/>

### JWT (Json Web Token)

JWT란?  

- JWT(Json Web Token)는 웹표준 (RFC 7519)으로 JSON 포맷을 이용해 정보를 가볍고 안전하게 전송하기 위한 Claim 기반의 Web Token이다.  

- 서버만 알고 있는 Secret Key로 디지털 서명화되어있기 때문에 신뢰할 수 있다
보통 Authorization (로그인, SSO) 또는 안전한 정보 교환을 위해 사용된다.  

- JWT에서는 토큰 자체에 유저 정보를 담아서 HTTP 헤더에 전달하기에 유저 세션을 유지할 필요가 없다. 


- http://jwt.io 에서 자세히 볼수 있다.  

    <img src="./assets/spring_security_jwt.png" style="width: 80%; height: auto;"/> 

<br/>

클레임(Claim)  

- 클레임(Claim)이란 사용자 정보나 데이터 속성 등을 의미한다.
- 클레임 기반 토큰 안에는 사용자의 id, pw 등의 개인 정보가 들어있다.
  - Self-contained : 자체 포함, 토큰 자체가 정보
- JWT는 가장 대표적인 클레임 기반 토큰이다.

<br/>

JWT의 필요성  

- Session의 한계
  - Cookie는 정보를 클라이언트 측에 저장하고 Session은 정보를 서버측에 저장한다.
  - 따라서 유저의 수가 너무 많으면 서버 과부하
- Scale Out의 한계
  - 서버 확장(scale out)시 세션 정보 동기화 문제
- REST API는 Stateless를 지향
  - 사용자의 상태 정보를 저장하지 않는 형태 ex) 세션, 쿠키

<br/>

JWT 형식  

- contain user authorization + any other information
- header.payload.signature
  - Base64 Encoding
- JWT에서 중요한 것은 payload의 data가 모든 이들에게 보인다는 점이다. 따라서 password와 같은 정보를 payload에 넣으면 안된다.
  - However,어떤 이가 악의적으로 payload를 변경하면 Secret Key로 인해 서버는 알 수 있다


    <img src="./assets/spring_security_jwt2.png" style="width: 80%; height: auto;"/> 

<br/>

- Encoded
- Decoded
  - Header
    - alg : 해쉬 알고리즘
    - typ : 타입
  - Payload : not mandatory / Additional information
    - sub : 어떤 것에 대해 말하는지
    - name : user의 이름 등..
    - iat : 토큰 생성 시간
    - exp : 만료 시간
  - Signature
    - Base64 Encoded header + payload
    - 512 bit secret key (base64 Encoded)

<br/>

동작 과정  
  
<img src="./assets/spring_security_jwt3.png" style="width: 80%; height: auto;"/> 


- 클라이언트 로그인 요청 POST(id, pw)
- 서버는 (id, pw)가 맞는지 확인 후 맞다면 JWT를 Secret Key로 생성 후 전달
- 클라이언트는 Token을 로컬 쿠키에 저장
- 클라이언트는 서버에 요청할 때 항상 헤더에 Token을 포함시킴
- 서버는 요청을 받을 때마다 Secret Kye를 사용해Token이 유효한지 검증
  - 서버만이 Secret Key를 가지고 있기 때문에 검증 가능
  - Token이 검증되면 따로 username, pw를 검사하지 않아도 user identification 가능
- 서버의 Response  

<br/>

Token 유효 검증  

- 클라이언트의 요청 (Header : Token)
- Spring의 Interceptor에 의해 요청이 Intercept됨
- 클라이언트에게 제공되었던 Token과 클라이언트의 Header에 담긴 Token 일치 확인
- auth0 JWT를 이용해 issuer, expire 검증  


<br/>

## 프로젝트 생성 및 환경 설정 

<br/>

###  React Frontend 구성    

<br/>

React & Spring Boot 연동.  
- React는 3000 port, Spring boot는 8080 port 사용

React를 빌드 하기 위해 node.js 를 설치합니다.
node 공식 페이지 ( https://nodejs.org/ko/download ) 접속 후 LTS 버전을 클릭하여 다운로드 하고 설치 합니다.  

아래와 같이 node version을 확인합니다.   

```bash
jakelee@jake-MacBookAir Downloads % node -v
v14.19.3
```

<br/>

github 에서 react 소스를 다운 받습니다.  

```bash
jakelee@jake-MacBookAir Downloads % git clone https://github.com/shclub/edu12-3.git
```  

edu12-3 폴더로 이동하여 빌드 (install ) 하고 start 합니다.  
   
```bash
jakelee@jake-MacBookAir edu12-3 % npm install
jakelee@jake-MacBookAir edu12-3 % npm start
```  

브라우저에서 http://localhost:3000을 입력하면 아래와 같은 로그인 화면이 보입니다.  

<img src="./assets/spring_security3.png" style="width: 80%; height: auto;"/> 

<br/>

React에서 backend API 호출은 axios 라이브러리를 사용한다.  

package.json에 추가된 것을 확인 가능  

```
{
  "name": "jwt-login-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^4.2.4",
    "@testing-library/react": "^9.5.0",
    "@testing-library/user-event": "^7.2.1",
    "axios": "^0.19.2",
    "bootstrap": "^4.5.0",
    "react": "^16.13.1",
    "react-dom": "^16.13.1",
    "react-router-dom": "^5.2.0",
    "react-scripts": "3.4.1"
  },
  ```  

<br/>

Axios  

- Axios란  
  - 비동기 방식으로 HTTP 데이터 요청을 실행
  - 직접적으로 XMLHttpRequest를 다루지 않고 Ajax를 호출해서 사용
  - Axios의 리턴 값은 Promise 객체 형식
- Ajax란 Asynchronous Javascript And Xml(비동기식 자바스크립트와 xml)의 약자로  XMLHttpRequest객체를 이용해서 데이터를 로드하는 기법
- XMLHttpRequest란 웹 브라우저와 서버 사이의 메소드를 통해 데이터를 전송하는 객체 형식의 API
- Promise란 자바스크립트 비동기 로직 처리에 사용되는 객체이다.  

<br/>

React 폴더 구조는 아래와 같습니다.  

<img src="./assets/spring_security4.png" style="width: 80%; height: auto;"/>   

- components : javascript를 확장한 jsx로 되어 있고 script 안에서 html 실행 가능
  - LoginApp.jsx : 처음 실행시 보이는 화면  
  - ListEmployeeComponent.jsx : 로그인 후에 보이는 임직원 전체 조회 화면
  - CreateEmployeeComponent.jsx : 임직원 정보 생성 화면
- services : axios를 사용하는 java script 화일 모음
  - AuthenticationService.js : 로그인 API 호출 하여 토큰 가져옴 
  - employee.js : Token을 사용하여 API 호출 ( 임직원정보 CRUD )  

<br/> 

반복되는 작업은 interceptor 를 사용하여 구성.  
token을 가져온 후에 모든 API에는 Header에 Bearer 토큰 설정해야 함.  

```javascript
  //Axios는 자바스크립트에서 HTTP 통신을 위해 쓰이는 Promise 기반 HTTP Client이다.
  //Axios Interceptors는 모든 Request/Response가 목적지에 도달하기 전에 Request에 원하는 내용을 담아 보내거나 원하는 코드를 실행시킬 수 있다.
    
    setupAxiosInterceptors() {
        axios.interceptors.request.use(
            config => {
                const token = localStorage.getItem('token');
                if (token) {
                    config.headers['Authorization'] = 'Bearer ' + token;
                }
                // config.headers['Content-Type'] = 'application/json';
                return config;
            },
            error => {
                Promise.reject(error)
            });
    }
```  
<br/>

employee.js 의 생성자에 intercepter를 설정하면 모든 empployee.js 에는 token 이 구성됨 

```javascript
class EmployeeService {
    constructor(props) {
        //super(props)

        AuthenticationService.setupAxiosInterceptors();
        
    }
```  

<br/>

보안을 위해 모든 API 호출은  get 과 post 만 사용한다. put/delete 사용 안함.  
SpringBoot의 CorsConfiguration 에서 Default config 사용.  

```javascript
updateEmployee(employee, employeeId){
        console.log("===execute updateEmployee Service===")
        console.log('employee => ' + JSON.stringify(employee));
        return axios.post(process.env.REACT_APP_API_URL + "/api/v1/employees" + '/' + employeeId, employee);
    }

    deleteEmployee(employeeId){
        console.log("===execute deleteEmployee Service===")
        //return axios.delete(process.env.REACT_APP_API_URL + "/api/v1/employees" +  '/' + employeeId);
        return axios.post(process.env.REACT_APP_API_URL + "/api/v1/employee" +  '/' + employeeId);
    }
```  

AuthenticatedRoute 를 login API 호출과 로그인 이후 token 생성을 나누기위해 생성.  

```javascript  
class AuthenticatedRoute extends Component {    
    render() {
        if(AuthenticationService.isUserLoggedIn()) {
            return <Route {...this.props}/>
        } else {
            return <Redirect to="/login"/>
        }

    }
}
```  

LoginApp.jsx 에서는 API 별로 Route와 AuthenticatedRoute 분리  

```javascript  
  <HeaderWithRouter/>
  <Switch>
      <Route path="/" exact component={LoginComponent}/>
      <Route path="/login" component={LoginComponent}/>
      <AuthenticatedRoute path="/welcome/:name" component={WelcomeComponent}/>
      <AuthenticatedRoute path="/logout" component={LogoutComponent}/>
      <AuthenticatedRoute path = "/employees" component = {ListEmployeeComponent}/>
      <AuthenticatedRoute path = "/add-employee/:id" component = {CreateEmployeeComponent}/>
      <AuthenticatedRoute path = "/view-employee/:id" component = {ViewEmployeeComponent}/>
      <Route component={ErrorComponent}/>
  </Switch>
  <FooterComponent/>
```  

<br/>

###  SpringBoot Backend 구성    

<br/>

github 에서 SpringBoot 소스를 다운 받습니다.  

```bash
jakelee@jake-MacBookAir Downloads % git clone https://github.com/shclub/edu12-4.git
```  

IntelliJ 에서 해당 소스를 오픈 합니다.  

폴더 구조는 아래와 같습니다.  

<img src="./assets/spring_security5.png" style="width: 80%; height: auto;"/>     

- common : 보안 및 config 관련된 패키지
- employee : 임직원 정보 CRUD 가 있는 패키지  

<br/>

security를 위해 dependency가 추가 된것 을 확인합니다.   

```xml
		<!-- Spring Security -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

		<!-- Jwt Token -->
		<dependency>
			<groupId>io.jsonwebtoken</groupId>
			<artifactId>jjwt</artifactId>
			<version>0.9.1</version>
		</dependency>

		<dependency>
			<groupId>org.glassfish.jaxb</groupId>
			<artifactId>jaxb-runtime</artifactId>
		</dependency>

		<!-- Naver lucy xss filter -->
		<dependency>
			<groupId>com.navercorp.lucy</groupId>
			<artifactId>lucy-xss</artifactId>
			<version>1.6.3</version>
		</dependency>
		<dependency>
			<groupId>com.navercorp.lucy</groupId>
			<artifactId>lucy-xss-servlet</artifactId>
			<version>2.0.0</version>
		</dependency>

		<!-- Jasypt -->
		<dependency>
			<groupId>com.github.ulisesbocchio</groupId>
			<artifactId>jasypt-spring-boot</artifactId>
			<version>2.1.0</version>
			<scope>compile</scope>
		</dependency>
		<dependency>
			<groupId>com.sun.jini</groupId>
			<artifactId>sun-util</artifactId>
			<version>2.1</version>
		</dependency>

		<!-- Oauth 2 -->
		<dependency>
			<groupId>org.springframework.security.oauth.boot</groupId>
			<artifactId>spring-security-oauth2-autoconfigure</artifactId>
			<version>2.1.8.RELEASE</version>
		</dependency>
```  

<br/>

common 패키지에서 붉은 색으로  표기된 부분이 security 관련 Class 입니다.   

<img src="./assets/spring_security6.png" style="width: 80%; height: auto;"/>     

<br/>
jasypt 란   

개발자가 암호화 작동 방식에 대한 깊은 지식 없이도 최소한의 노력으로 프로젝트에 기본 암호화 기능을 추가할 수 있도록 하는 Java 라이브러리 입니다.  
주로 프로젝트가 공개되어도 프로퍼티 내에 작성된 설정 정보를 암호화하여 노출시키지 않는 목적으로 사용한다.  

jasyptConfig.java
```java  
public class JasyptConfig{

    private static final String JASYPT_KEY        = "education";
    private static final String ALGORITHM       = "PBEWithMD5AndDES";

    @Bean("jasyptStringEncryptor")
    public StandardPBEStringEncryptor StringEnc() {
        StandardPBEStringEncryptor enc = new StandardPBEStringEncryptor();
        EnvironmentPBEConfig conf = new EnvironmentPBEConfig();
        conf.setPassword(JASYPT_KEY); //암호화할 때 사용하는 키
        conf.setAlgorithm(ALGORITHM); // 암호화 알고리즘
        enc.setConfig(conf);
        return enc;
    }
}
```  

<br/>

application.yml 화일에 bean 이름을 등록한다.  
```yml
#jasypt encryptor
jasypt:
  encryptor:
    bean: jasyptStringEncryptor
```  

<br/>

jasypt 온라인 암복호화 사이트에서 https://www.devglan.com/online-tools/jasypt-online-encryption-decryption 사용할 값을 암호화 한후 key를 복사한다.

- Plain-Text: caravan
- Type: Two Way Encryption  ( With Secret Text )
- Secret-Key: education

<br/>

<img src="./assets/spring_security7.png" style="width: 80%; height: auto;"/>  

<br/>

resources 폴더에 application-local.yml 화일의 db 비밀번호에 
ENC(설정 값) 으로 작성한다.

<img src="./assets/spring_security8.png" style="width: 80%; height: auto;"/>  

<br/> 

SpringBoot에서 암호화된 값을 복호화 해서 가져오는 방법은 아래와 같다.  

EmployeeService.java
```java
// jasypt로 저장된 비밀번호가  복호화 된다.
@Value("${spring.datasource.password}")  // application-local.yml
private String h2Password;
```  

<br/>

이번 소스는 id / password가 소스에 하드 코딩 되어 있다. (edu/edu1234)

<br/>

[ JWT 생성 ]

- POST API with mapping /api/login
- id: user_id, pw: user_pw 입력시 사용자 확인 후 JWT 생성  

<img src="./assets/spring_security9.png" style="width: 100%; height: auto;"/>  

<br/>

JwtRequestFilter  

- OncePerRequestFilter : Spring Security Configuration Class
  - 모든 Request를 가로챈다
  - Request의 헤더에 토큰이 있는지 확인한다
  - 없다면 JwtAuthenticationController를 호출해 토큰을 생성한다

<br/>

JwtAuthenticationController  

- Request Body에서 id, pw를 추출해 AuthenticationManager에게 전달
- 올바른 id, pw가 인증이 되면
  - JwtTokenUtil의 generateToken(UserDetails)을 통해 토큰 생성

<br/>

AuthenticationManager (Spring Security)  

<img src="./assets/spring_security10.png" style="width: 100%; height: auto;"/>  


Spring security Configuration Class  

- User Request Body에서 얻은 id, pw와
- JwtUserDetailsService를 호출해서 얻은 id, pw
  - 현재는 user_id로 하드코딩되어있다
- 두 id,pw를 비교해 사용자 일치하는지 검증
  - if 일치
    - return true
  - else if 불일치
    - throw invalid user exception  

JwtUserDetailsService  

- user의 id, pw return

JwtTokenUtil  

- 토큰을 생성해 return   

<br/>

[ JWT 검증 ]  

<br/>  

- GET API with mapping /api/v1/employee
- user가 유효한 JWT를 가지고 있을 때 접근이 허용됨  

<img src="./assets/spring_security11.png" style="width: 100%; height: auto;"/>  

<br/>

JwtRequestFilter  

- User Request's Header에서 Token이 있는지 확인
있다면 Token 추출
- JwtUserDetailsService에서 userName으로 user의 detail 얻어옴
- JwtTokenUtil에서 Token 유효성 검사
  - validateToken(jwtToken, userDetails)
- 유효한 Token이면 Spring Security에 유효한 User라 알려주고 hello() method에게 접근 가능하게 설정

<br/>

JwtUserDetailsService  

- user의 detail을 return

JwtTokenUtil 

- Token 유효성 검사(Validate Token) 후 return True/False


<br/>

[ Secret Key 설정 ]    

<br/>  

- Hashing algorithm과 함께 사용할 Secret Key를 설정
- Secret Key는 Header, Payload와 결합되어 Hash 생성  

<br/>

application.yml 화일에 key를 입력한다.  

```yml
jwt:
  secret: edujwt
```

<br/>

[ JWtTokenutil ]    

<br/>  

- JWT를 생성하고 검증하는 역할 수행
- io.jsonwebtoken.Jwts 라이브러리 사용  


```java
@Component
public class JwtTokenUtil implements Serializable {
    private static final long serialVersionUID = -2550185165626007488L;
    public static final long JWT_TOKEN_VALIDITY = 5 * 60 * 60;
    @Value("${jwt.secret}")
    private String secret;

    //retrieve username from jwt token
    public String getUsernameFromToken(String token) {
        return getClaimFromToken(token, Claims::getSubject);
    }

    //retrieve expiration date from jwt token
    public Date getExpirationDateFromToken(String token) {
        return getClaimFromToken(token, Claims::getExpiration);
    }

    public <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaimsFromToken(token);
        return claimsResolver.apply(claims);
    }

    //for retrieveing any information from token we will need the secret key
    private Claims getAllClaimsFromToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }

    //check if the token has expired
    private Boolean isTokenExpired(String token) {
        final Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }

    //generate token for user
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return doGenerateToken(claims, userDetails.getUsername());
    }

    //while creating the token -
//1. Define  claims of the token, like Issuer, Expiration, Subject, and the ID
//2. Sign the JWT using the HS512 algorithm and secret key.
//3. According to JWS Compact Serialization(https://tools.ietf.org/html/draft-ietf-jose-json-web-signature-41#section-3.1)
//   compaction of the JWT to a URL-safe string
    private String doGenerateToken(Map<String, Object> claims, String subject) {
        return Jwts.builder().setClaims(claims).setSubject(subject).setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + JWT_TOKEN_VALIDITY * 1000))
                //.setExpiration(new Date(System.currentTimeMillis() + 5 * 1000))
                .signWith(SignatureAlgorithm.HS512, secret).compact();
    }

    //validate token
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
}
```  

doGenerateToken  

- Token 생성
  - claim : Token에 담을 정보
  - issuer : Token 발급자
  - subject : Token 제목
  - issuedate : Token 발급 시간
  - expiration : Token 만료 시간
    - milliseconds 기준!
    - JWT_TOKEN_VALIDITY = 5 60 60 => 5시간
- signWith (알고리즘, 비밀키)   

<br/>

[ JwtUserDetailsService ]    

<br/>  

- DB에서 UserDetail를 얻어와 AuthenticationManager에게 제공하는 역할
- 이번에는 DB 없이 하드코딩된 User List에서 get userDetail

계정은 edu/edu1234  이고 password 는 SHA256으로 암호화 하여 생성.

```java
public class JwtUserDetailsService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        if ("edu".equals(username)) {
        //    return new User("user_id", "$2a$10$m/enYHaLsCwH2dKMUAtQp.ksGOA6lq7Fd2pnMb4L.yT4GyeAPRPyS",
            return new User("edu", "db03de15b8000fc35ad975c1322f98124a22521e0616a55c926807eb7225fa38",
                    new ArrayList<>()); //edu1234

        } else {
            throw new UsernameNotFoundException("User not found with username: " + username);
        }
    }
}
```  

- Spring Security 5.0에서는 Password를 BryptEncoder를 통해 Brypt화 하지만 우리는 SHA256으로 변경한다.  
  - https://www.javainuse.com/onlineBcrypt 에서 user_pw를 Bcrypt화
- id : user_id, pw: user_pw로 고정해 사용자 확인
- 사용자 확인 실패시 throw Exception  

<br/>

[ JwtAuthenticationController ]    

<br/>  

- 사용자가 입력한 id, pw를 body에 넣어서 POST API mapping /api/login
- 사용자의 id, pw를 검증
- jwtTokenUtil을 호출해 Token을 생성하고 JwtResponse에 Token을 담아 return ResponseEntity  

<br/>

```java
@RestController
public class JwtAuthenticationController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Autowired
    private JwtUserDetailsService userDetailsService;

    @PostMapping("/api/login")
    public ResponseEntity<?> createAuthenticationToken(@RequestBody JwtRequest authenticationRequest) throws Exception {
        authenticate(authenticationRequest.getUsername(), authenticationRequest.getPassword());

        final UserDetails userDetails = userDetailsService
                .loadUserByUsername(authenticationRequest.getUsername());

        final String token = jwtTokenUtil.generateToken(userDetails);

        return ResponseEntity.ok(new JwtResponse(token));
    }

    private void authenticate(String username, String password) throws Exception {
        try {
            authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(username, password));
        } catch (DisabledException e) {
            throw new Exception("USER_DISABLED", e);
        } catch (BadCredentialsException e) {
            throw new Exception("INVALID_CREDENTIALS", e);
        }
    }
}
```  

<br/>

[ JwtRequest ]    

<br/>  

- 사용자에게서 받은 id, pw를 저장  

<br/>

```java
@Data
@NoArgsConstructor
public class JwtRequest implements Serializable {

    private static final long serialVersionUID = 5926468583005150707L;

    private String username;
    private String password;

    public JwtRequest(String username, String password) {
        this.setUsername(username);
        this.setPassword(password);
    }

}
```  

<br/>

[ JwtResponse ]    

<br/>  

- 사용자에게 반환될 JWT를 담은 Response  

<br/>

```java
public class JwtResponse implements Serializable {

    private static final long serialVersionUID = -8091879091924046844L;
    private final String jwttoken;

    public JwtResponse(String jwttoken) {
        this.jwttoken = jwttoken;
    }

    public String getToken() {
        return this.jwttoken;
  }
}
```  

<br/>

[ JwtRequestFilter ]    

<br/>  

- Client의 Request를 Intercept해서 Header의 Token가 유효한지 검증
- if 유효한 Token
  - Spring Security의 Authentication을 Setting, to specify that the current user is authenticated 

<br/>

```java
@Component
public class JwtRequestFilter extends OncePerRequestFilter {

    @Autowired
    private JwtUserDetailsService jwtUserDetailsService;

    @Autowired
    private JwtTokenUtil jwtTokenUtil;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {

        final String requestTokenHeader = request.getHeader("Authorization");

        String username = null;
        String jwtToken = null;
        // JWT Token is in the form "Bearer token". Remove Bearer word and get
        // only the Token
        if (requestTokenHeader != null && requestTokenHeader.startsWith("Bearer ")) {
            jwtToken = requestTokenHeader.substring(7);
            try {
                username = jwtTokenUtil.getUsernameFromToken(jwtToken);
            } catch (IllegalArgumentException e) {
                System.out.println("Unable to get JWT Token");
            } catch (ExpiredJwtException e) {
                System.out.println("JWT Token has expired");
            }
        } else {
            logger.warn("JWT Token does not begin with Bearer String");
        }

        // Once we get the token validate it.
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails userDetails = this.jwtUserDetailsService.loadUserByUsername(username);

            // if token is valid configure Spring Security to manually set
            // authentication
            if (jwtTokenUtil.validateToken(jwtToken, userDetails)) {

                UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken = new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                usernamePasswordAuthenticationToken
                        .setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                // After setting the Authentication in the context, we specify
                // that the current user is authenticated. So it passes the
                // Spring Security Configurations successfully.
                SecurityContextHolder.getContext().setAuthentication(usernamePasswordAuthenticationToken);
            }
        }
        chain.doFilter(request, response);
    }
```  

<br/>

- JWT Token은 Bearer (스페이스) 로 시작
- jwtTokenUtil.validateToken을 통해 Token 유효성 검사
- if true) UsernamePasswordAuthenticationToken을 설정해 유효한 사용자임을 Spring Security에게 알려준다  


<br/>

[ JwtAuthenticationEntryPoint ]    

<br/>  

- 접근 권한이 없는 사용자에게 401 Error를 보냄

<br/>

```java
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint, Serializable {

    private static final long serialVersionUID = -7858869558953243875L;

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException {

        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized");
    }
}
```  

<br/>

[ WebSecurityConfig ]    

<br/>  

- WebSecurity와 HttpSecurity를 커스터마이징
  - ex) CORS 등 해결

<br/>

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;

    @Autowired
    private UserDetailsService jwtUserDetailsService;

    @Autowired
    private JwtRequestFilter jwtRequestFilter;

    //@Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        // configure AuthenticationManager so that it knows from where to load
        // user for matching credentials
        // Use BCryptPasswordEncoder
        auth.userDetailsService(jwtUserDetailsService).passwordEncoder(passwordEncoder());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        //return new BCryptPasswordEncoder();
        return new Sha256PasswordEncoder(); // default 대신 SHA256 Encoder 사용함
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    // OTP, Captcha 가 있으면 custom 생성
    /*@Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(customAuthentictionProvider);
    }*/

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring()
                .antMatchers("/resources/**")
                .antMatchers("/css/**")
                .antMatchers("/vendor/**")
                .antMatchers("/js/**")
                .antMatchers("/favicon*/**")
                .antMatchers("/img/**");
    }

    @Bean
    public CorsFilter corsFilter() {
        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        final CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        //config.getAllowedOriginPatterns();
        config.addAllowedOrigin("*");
        config.addAllowedHeader("*");
        config.addAllowedMethod("OPTIONS");
        config.addAllowedMethod("HEAD");
        config.addAllowedMethod("GET");
        config.addAllowedMethod("PUT");
        config.addAllowedMethod("POST");
        config.addAllowedMethod("DELETE");
        config.addAllowedMethod("PATCH");
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.cors().configurationSource(request -> new CorsConfiguration().applyPermitDefaultValues());

        // We don't need CSRF
        httpSecurity.csrf().disable()
                //.cors().and()
                // dont authenticate this particular request
                .authorizeRequests().antMatchers
                  //예외 API를 기입하다.
                        "/",
                        "/actuator/**",
                        "/v3/api-docs/**",
                        "/swagger-ui.html",
                        "/swagger-ui/**",
                        "/api/login",
                        "/login",
                        "/h2-console/*"
                ).permitAll()

                // all other requests need to be authenticated
                .anyRequest().authenticated().and()
                // make sure we use stateless session; session won't be used to
                // store user's state.
                //.exceptionHandling().authenticationEntryPoint((req, rsp, e) -> rsp.sendError(HttpServletResponse.SC_UNAUTHORIZED)).and().sessionManagement()
                .exceptionHandling().authenticationEntryPoint(jwtAuthenticationEntryPoint).and().sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);

        // multi login 시 IFRAME 사용 가능하도록
        //httpSecurity.headers().frameOptions().sameOrigin();
        httpSecurity.headers().frameOptions().disable();

        // Add a filter to validate the tokens with every request
        httpSecurity.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);

    }
  
}
```   

cors 설정시 아래와 같이 applyPermitDefaultValues를 사용하면 GET, POST 는 기본적으로 사용가능 합니다.  

```java
    httpSecurity.cors().configurationSource(request -> new CorsConfiguration().applyPermitDefaultValues());
```

<br/>

[ Employee 패키지 ]    

<br/>  

- employee 패키지에는 임직원 정보 CRUD 가 구현되어 있다.  
- GET 과 POST 로만 구현이 되어 있다.

<img src="./assets/spring_security12.png" style="width: 100%; height: auto;"/>  

<br/>

EmployeeController.java  

```java
@Tag(name = "posts", description = "Employee API")
@RestController
@RequiredArgsConstructor
@CrossOrigin(origins ="*")
@RequestMapping("/api/v1/")
public class EmployeeController {

    private final EmployeeService employeeService;


    //@Cacheable(cacheNames = "employee")
    @GetMapping("/employees")
    @Operation(summary ="임직원 전체 조회",description="임직원 전체를 조회 합니다.")
    public List<EmployeeEntity> getEmployeeList() {
        return this.employeeService.getEmployeeList();
    }

    @Operation(summary ="임직원 단건 조회",description="특정 임직원 단건에 대한 정보 조회 합니다.")
    @GetMapping("/employees/{id}")
    public EmployeeEntity getEmployee(@PathVariable Long id) {
        return this.employeeService.getEmployee(id);
    }

    @Operation(summary ="임직원 등록",description="임직원을 등록합니다.")
    @PostMapping("/employees")
    public ResponseEntity<EmployeeEntity> create(@RequestBody EmployeeEntity employeeEntity) {
        EmployeeEntity createdEntity = employeeService.create(employeeEntity);
        return ResponseEntity.ok(createdEntity);
    }

    @PostMapping("/employees/{id}")
    @Operation(summary ="임직원 수정",description="임직원 정보를  수정합니다.")
    public ResponseEntity<EmployeeEntity> update(@PathVariable Long id, @RequestBody EmployeeEntity employeeEntity) {
        EmployeeEntity updatedEntity = employeeService.update(id,employeeEntity);
        return ResponseEntity.ok(updatedEntity);
    }

    @PostMapping("/employee/{id}")
    @Operation(summary ="임직원 정보 삭제",description="임직원 정보를 삭제합니다.")
    public ResponseEntity<EmployeeEntity> delete(@PathVariable Long id) {
        EmployeeEntity deletedEntity = employeeService.delete(id);
        return ResponseEntity.ok(deletedEntity);
    }
}
```  


<br/>

## 실행하기  

<br/>

###  실행해보기

- react는 edu12-3 폴더에서 npm start 명령어를 입력하여 기동을 한다.
- springboot는 intellij 에서 기동한다.  

<br/>

웹브라우저에서 http://localhost:3000 를 입력하고 로그인 을 한다.  
- 계정 : edu/edu1234  

chrome 브라우저에서 개발자 도구 메뉴를 선택하고 Application Tab 을 클릭하고 Local Storage 항목을 보면 token이  생성되어 저장 된 것을 확인 할 수 있다.  


<img src="./assets/spring_security13.png" style="width: 100%; height: auto;"/>  

<br/> 

Add 버튼을 클릭하여 아래와 같이 데이터를 입력하고 SAVE 버튼을 클릭한다.   

<img src="./assets/spring_security14.png" style="width: 100%; height: auto;"/>  

<br/>

데이터가 정상으로 입력되면 아래와 같이 데이터가 보인다.  

<img src="./assets/spring_security15.png" style="width: 100%; height: auto;"/>  

<br/>

IntelliJ 콘솔에서도 로그를 확인 할 수 있다.  

```bash  
edu12-4 19:27:42.416 INFO  jdbc.sqltiming : - INSERT INTO "EMPLOYEE" ("EMPDEPTNAME", "EMPMAIL", "EMPNAME", "EMPTELNO", "ID") VALUES ('SW아키텍처팀', 'seokhwan.lee@kt.com', '이석환', '01030181888', 1)\n {executed in 1 msec} 
edu12-2 19:27:42.458 INFO  com.kt.edu.thirdproject.employee.service.EmployeeService : - Request to get all Employees 
edu12-2 19:27:42.458 INFO  com.kt.edu.thirdproject.employee.service.EmployeeService : - h2 password : caravan 
edu12-2 19:27:42.458 INFO  jdbc.sqltiming : - SELECT "EMPLOYEE"."ID" AS "ID", "EMPLOYEE"."EMPNAME" AS "EMPNAME", "EMPLOYEE"."EMPMAIL" AS "EMPMAIL", "EMPLOYEE"."EMPTELNO" AS "EMPTELNO", "EMPLOYEE"."EMPDEPTNAME" AS "EMPDEPTNAME" FROM "EMPLOYEE"\n {executed in 0 msec} 
edu12-2 19:27:42.466 INFO  jdbc.resultsettable : - \n|---|--------|--------------------|------------|------------|\n|id |empname |empmail             |emptelno    |empdeptname |\n|---|--------|--------------------|------------|------------|\n|1  |이석환     
```  


<br/>

삭제 테스트는 Talend  API를 사용해 보도록 합니다.  

먼저 웹 브라우저에서 개발자 도구로 이동하여 token 값을 복사합니다.  

아래와 같이 설정을 하고 send 버튼을 눌러 실행합니다.  

<img src="./assets/spring_security16.png" style="width: 100%; height: auto;"/>  

- method :  post
- url : http://localhost:8080/api/v1/employee/1
- header
  - key : Authentication
  - value : Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJlZHUiLCJleHAiOjE2NTU5MjE5OTYsImlhdCI6MTY1NTkwMzk5Nn0.rMP_jEFKL37Lg5La7jPHBEt9BiVTx0uTAyexd7BV67KOE375XCsR6v7dGKHAOz-tFQmZ7Mz_8LvB-C4EEIQggg 

<br/>

200 메시지를 확인하면 데이터가 정상적으로 삭제가 된 것을 알 수 있습니다.  

<br/>

###  React JS Dockerfile 구성

<br/>

React js의 도커 파일 구성은 아래와 같다.   

Dockerfile

```bash
#
# Build stage
#
FROM node:14.19.3-alpine as build
WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH
COPY package.json ./
COPY package-lock.json ./
RUN npm ci --silent
RUN npm install react-scripts@3.4.1 -g --silent
COPY . ./
RUN npm run build

#
# Package stage
#
# production environment
FROM nginx:stable-alpine

ENV TZ Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```  

<br/>

- Build stage : 빌드하여 도커 파일을 만든다.
- package stage : 도커 이미지를 실행하며 nginx 를 앞에 붙인다.

<br/>

###  SpringBoot Dockerfile 구성

<br/>ew

SpringBoot 의 도커 파일 구성은 아래와 같다.   

Dockerfile

```bash
#
# Build stage
#
FROM  maven:3.8.4-openjdk-17 AS MAVEN_BUILD

RUN mkdir -p build
WORKDIR /build

COPY pom.xml ./
COPY src ./src

COPY . ./
RUN mvn package

#
# Package stage
#
# production environment

FROM eclipse-temurin:17.0.2_8-jre-alpine

COPY --from=MAVEN_BUILD /build/target/*.jar app.jar

ENV TZ Asia/Seoul
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV SPRING_PROFILES_ACTIVE dev

ENV JAVA_OPTS="-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -XshowSettings:vm"
ENV JAVA_OPTS="${JAVA_OPTS} -XX:+UseG1GC -XX:+UnlockDiagnosticVMOptions -XX:+G1SummarizeConcMark -XX:InitiatingHeapOccupancyPercent=35 -XX:G1ConcRefinementThreads=20"

#ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar  app.jar "]
ENTRYPOINT ["sh", "-c", "java -jar  app.jar "]
```  

<br/>

- Build stage : Maven 이미지로 도커 파일을 만든다.
- package stage : 도커 이미지를 실행하여 컨테이너화 한다.  

<br/>