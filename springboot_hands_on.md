#  SpringBoot Hands-on
 
SpringBoot 활용 방법에 대해서 실습한다.  

1. 뷰 템플릿 과 MVC 패턴

2. 데이터 생성 with JPA

3. 롬복과 리팩토링

4. 데이터 수정 및 삭제  

5. CRUD 와 SQL Query

6. Rest API 와 JSON 

7. HTTP 와 Rest Controller

8. 테스트 작성하기

9. 댓글 서비스 만들기

10. IoC 와 DI

11. AOP

12. Object Mapper

<br/>

##  실습 전체


<br/>

### 뷰 템플릿 과 MVC 패턴  

<br/>

web service 의 동작원리

<br/>

- 웹서비스는 클라이언트와 서버의 요청과 응답으로 동작
- 클라이언트 : 서비스를 사용하는 프로그램, 컴퓨터 → 브라우저
- 서버 : 서비스를 제공하는 프로그램, 컴퓨터 → 스프링부트

  <img src="./assets/springboot_webservice.png" style="width: 80%; height: auto;"/>  

<br/>

hello world 출력 과정  

<br/>

- 스프링 부트 실행
- src - main - java - 기본패키지 - 메인 메소드를 가진 클래스 실행 (Run)
- Tomcat started on port(s) 8080 (http) → 웹 서버 실행 (동작)
- localhost:8080 → 내 컴퓨터의 서버 주소:포트번호, 포트번호 8080에서
  스프링 부트가 동작
- localhost:8080/hello.html → 'hello world' 출력 → 내 컴퓨터의 8080 
  번호에서 수행되고 있는 서버에게 hello.html 파일 요청  
  localhost:8080/hello.html  : 내 컴퓨터의 8080 번호에서 수행되고 있는 서버에게 hello.html 파일 요청
- src - main  - resources - static - hello.html → static
  폴더에서 파일을 찾아서 그 안의 HTML 코드를 응답으로 전송  

<br/>

뷰 템플릿의 필요성과 머스테치  

<br/>

- 웹페이지의 변수를 활용하는 뷰 템플릿과 분야별 담당자를 나누는 MVC 패턴
- 사용자 수마다 생성되는 페이지 ? → 화면을 담당하는 기술인 뷰 템플릿을 통해 극복
- 뷰 템플릿 : 웹 페이지를 하나의 틀로 만들고 변수를 삽입해 틀이 되는 페이지가 
- 변수의 값에 따라서 수많은 페이지로 변화 
- 스프링 부트의 머스테치 (Mustache) : 뷰 템플릿을 만드는 도구 ( Thymeleaf , JSP )
- 뷰 템플릿에는 처리 과정을 담당하는 Controller와 데이터를 관리하는 Model가 존재
- MVC 패턴 : 화면, 처리, 데이터를 각 담당자 별로 나누는 기법

  <img src="./assets/mvc1.png" style="width: 80%; height: auto;"/>  

  앞에 글자를 따서 MVC 라 부름.   
  
  <img src="./assets/mvc2.png" style="width: 60%; height: auto;"/>

<br/>

이제 지난번 생성했던 firstproject 를 intelliJ를 사용하여 오픈한다.  

<br/>

뷰 템플릿의 위치는  

프로젝트명 -> src -> main -> resources -> templates 에  생성한다.  

<img src="./assets/mvc3.png" style="width: 80%; height: auto;"/>  

<br/>

마우스 오른쪽 버튼 클릭하고 New File 선택하고 greetings.mustache 생성한다.  

제일 처음 생성하면 mustache 화일을 인식하지 못한다. plugin 설치 필요.  

<img src="./assets/mvc4.png" style="width: 80%; height: auto;"/>  

<br/>

IntelliJ 에서 Help -> Find Action 으로 이동한다.  

<img src="./assets/mvc5.png" style="width: 80%; height: auto;"/>  

<br/>

plugins입력하고 엔터를 친다.  

<img src="./assets/mvc6.png" style="width: 80%; height: auto;"/>  

<br/>

market place를 선택 한후 mustache를 입력하면 아래와 같이 나오고 첫번째 항목 선택후 Install 한다.  

<img src="./assets/mvc7.png" style="width: 80%; height: auto;"/>  

<br/>

다시 한번 greetings.mustache 생성을 해보면 에러 없이 생성이 된다.    

<img src="./assets/mvc8.png" style="width: 80%; height: auto;"/>

doc를 입력하고 tab을 누르면 자동으로 코드가 생성이 된다.  

<img src="./assets/mvc9.png" style="width: 80%; height: auto;"/>

<br/>

body tag안에 값을 입력한다.

```html
<body>
    <h1>안녕하세요 반갑습니다.</h1> 
</body>
```  

<br/>

view template 이제 완성이 되었고 이것을 보기 위해서는 controller 가 필요하다.  

<br/>

컨트롤러 만들기   

<br/>  

컨트롤러는 아래와 같은 순서로 만든다.  

프로젝트명 -> src -> main -> java -> 기본 패키지 -> 'controller' package 생성   

기본 패키지명 위에서 마우으 오른쪽 키 클릭하고 New -> Package 선택  

<img src="./assets/controller1.png" style="width: 80%; height: auto;"/>  

아래와 같이 창이 뜨면 끝에 controller 이름을 붙여 넣는다.  

<img src="./assets/controller2.png" style="width: 60%; height: auto;"/>  

<br/>

controller 안에 java class를 생성합니다.  
New -> Java Class 선택 하고 FirstController라는 이름으로 생성.  

<img src="./assets/controller3.png" style="width: 80%; height: auto;"/>  

생성한 view template과 controller를 연결해 주기 위한 코드를 작성합니다.  


class 위에 @Controller 라는 Annotation을 입력하면 자동으로 import 가 삽입됩니다.  

<img src="./assets/controller4.png" style="width: 80%; height: auto;"/>  

Auto import가 안되면 Prefrences -> Editor -> General -> Auto import 에서 배제된 항목이 있는지 확인합니다.  

<img src="./assets/controller5.png" style="width: 80%; height: auto;"/>  

<br/>

Method 를 추가합니다.  

url 연결 요청을 하기 위해 @GetMapping를 사용합니다.  ( Rest API )  

응답 페이지 설정 을 위해 return "페이지명" 을 입력합니다.  

이 페이지 명은 mustache의 화일명인 greetings 입니다.  
( 페이지명의 파일을 찾아서 브라우저로 전송  )

```java
package com.example.firstproject.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class FirstController {
    @GetMapping("/hi")
    public String niceToMeetYou() {
        return "greetings";
    }
}
```  

자동으로 import 코드가 생성되지 않으면 import Class를 클릭한다.  

<img src="./assets/controller6.png" style="width: 80%; height: auto;"/> 

서버실행은 메인 method인 FirstprojectApplication 에서 실행하는데 이미 서비스가 기동되어 있으면 rerun (재시작) 아이콘을 한다.  

<br/>

web browser에서 http://localhost:8080/hi 를 입력하면 에러 메시지를 볼 수 있다.  

<img src="./assets/controller7.png" style="width: 60%; height: auto;"/>   

GetMapping의 옆에 지구본 모양을 클릭하면 API를 테스트 할수 있는 기능이 있다.   

Generate request in HTTP Client를 클릭한다.  

<img src="./assets/controller8.png" style="width: 80%; height: auto;"/>

<br/>

Rest API를 테스트 할수 있는 화면이 나오고 왼쪽 녹색 화살표를 클릭하면 테스트가 진행이되고 아래 콘솔 화면에 결과 값이 나온다.  

<img src="./assets/controller9.png" style="width: 80%; height: auto;"/>   

<br/>

데이터  흐름은 다음과 같다.  
 
<img src="./assets/controller10.png" style="width: 80%; height: auto;"/>

이제 뷰페이지에 변수를 삽입을 해 봅니다.  

mustache 화일에서 아래와 같이 {{변수이름}} 을 사용하여 수정합니다.  

```html
<body>
    <h1>{{username}} 반갑습니다.</h1>
</body>
```  

위와 같이 설정하고 재시작 버튼 클릭 한 후 브라우저에서 실행을 하면 아래와 같이 에러가 발생합니다.  

<img src="./assets/controller11.png" style="width: 80%; height: auto;"/>  

username 이라는 변수를 못 찾아서 에러가 발생했고 모델을 만들어서 에러를 제거 해야 합니다.  

<br/>

모델 만들기   

<br/>  

페이지에 변수 삽입 + 템플릿 변수를 활용하기 위해 모델을 사용합니다.  

Controller에 Model 받아오기 위해 파라미터에 추가 합니다.  

```java
package com.example.firstproject.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class FirstController {
    @GetMapping("/hi")
    public String niceToMeetYou(Model model) {
        model.addAttribute("username", "jake lee"); //변수 등록
        return "greetings";
    }
}
```  
<br/>

Model class를 인식하지 못하여 붉은색으로 표시되며 import class를 클릭하면 import 가 삽입됩니다.  

<img src="./assets/model1.png" style="width: 80%; height: auto;"/>  

재시작 아이콘을 클릭하여 재기동 하고 웹 브라우저에서 확인 합니다.  

<img src="./assets/model2.png" style="width: 80%; height: auto;"/> 

<br/>

전체 흐름은 다음과 같다.  

<img src="./assets/model3.png" style="width: 100%; height: auto;"/> 


