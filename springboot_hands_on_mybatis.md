#  SpringBoot Hands-on MyBatis
 
MyBatis 활용 방법에 대해서 실습한다.  


1. 프로젝트 생성 및 환경 설정

2. 프로젝트 구성하기 

3. 실행해보기

4. 소스위치 : https://github.com/shclub/edu10

<br/>

## MyBatis 실습

<br/>

### MyBatis ? 

<br/>

개발자가 지정한 SQL, 저장 프로시저 그리고 몇 가지 고급 매핑을 지원하는 SQL Mapper이다.  

- JDBC로 처리하는 상당 부분의 코드와 파라미터 설정 및 결과 매핑을 대신해준다.
    - 기존에 JDBC를 사용할 때는 DB와 관련된 여러 복잡한 설정(Connection)들을 다루어야 했지만 SQL Mapper는 자바 객체를 실제 SQL문에 연결함으로써, 빠른 개발과 편리한 테스트 환경을 제공한다.  

  
- 데이터베이스 record에 원시 타입과 Map 인터페이스 그리고 자바 POJO를 설정해서 매핑하기 위해 xml과 Annotation을 사용할 수 있다.  

- MyBatis는 원래 Apache Foundation의 iBatis였으나, 생산성, 개발 프로세스, 커뮤니티 등의 이유로 Google Code로 이전되면서 이름이 바뀌었다.


<br/>

### Mapper ? 

<br/>

Mybatis는 Mapper 인터페이스를 제공한다.  

DAO 대신 mapper를 사용하면 DAO를 만들지 않고 interface만을 이용해서 간편히 개발할 수 있다.  

mybatis는 java code와 sql문을 분리하여 편리하게 관리하도록 한다.  

SQL문은 *.xml 형식으로 저장한다.  

DB에 질의할 쿼리문을 관리하는 Mapper파일에 요청한다.  

<br/>

### 프로젝트 생성 및 환경 설정   

<br/>

웹브라우저에서 https://start.spring.io/ 접속하여 아래와 같이 라이브러리를 추가하여 신규 프로젝트를 생성한다.  

<img src="./assets/mybatis1.png" style="width: 80%; height: auto;"/> 

<br/>

secondproject.zip으로 다운 받은 화일의 압축을 풀고   
IntelliJ 메뉴 File > New > Project from Existing Sources 으로 이동하여 프로젝트를 선택하여 오픈 한다.    

<img src="./assets/mybatis2.png" style="width: 80%; height: auto;"/> 

<br/>

사내 환경에 맞추기 위하여 pom.xml 을 오픈한 후  springboot 버전을  2.6.3 으로 변경 한다.  

<br/>

초기 서버 기동시 DB 데이트를 생성하기 위해서 resources 폴더 밑에 schema.sql 과 data.sql 화일을 생성한다.  

테이블 생성 schema.sql  

```sql  
create sequence hibernate_sequence;

create table article
(
    id              long    not null,
    title         	varchar(255),
    content         varchar(255)
);

alter table article add constraint article_pk primary key (id);  
```  


데이터 생성 data.sql  

```sql  
INSERT INTO article(id, title, content) VALUES(1, '1', '테스트 1');
INSERT INTO article(id, title, content) VALUES(2, '2', '테스트2');
INSERT INTO article(id, title, content) VALUES(3, '3', '테스트 3');
```  

<br/>

application.properties 화일에 DB 연결 설정 및 mybatis 설정을 한다.  

```bash  
# h2 DB
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.datasource.generate-unique-name=false

# log configuration path
#logging.config=classpath:log4j2.xml
#로그 정보의 세밀한 컨트롤을 위한 logback 설정 화일 위치. resources 폴더 아래.  
logging.config=classpath:logback-spring-dev.xml
logging.config.file.path=log

# log4jdbc
log4j.logger.jdbc.sqltiming=info
#log4jdbc.drivers=
#log4jdbc.spu
log4jdbc.dump.sql.maxlinelength=0


# Default Profile
spring.profiles.active=dev

spring.main.allow-bean-definition-overriding=true

# Database Platform
spring.sql.init.platform=h2
# DB 생성 및 초기화 여부
spring.sql.init.mode=embedded
spring.datasource.driver-class-name=org.h2.Driver
#spring.datasource.driver-class-name=net.sf.log4jdbc.sql.jdbcapi.DriverSpy
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=

# model/domain package
mybatis.type-aliases-package= com.kt.edu
# mapper xml location . resources 폴더 아래
mybatis.mapper-locations= mapper/*.xml
# mybatis-config.xml 화일 위치 . resources 폴더 아래
mybatis.config-location= classpath:mybatis-config.xml

# mybatis mapper log level
#logging.level.com.kt.edu.secondproject.article.mapper=TRACE
```   


<br/>

이제 mybatis-config.yaml 화일을 생성한다.  
typeAlias는 DTO 이름을 편하게 사용하기 위해 Alias 를 설정한다.  


```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <settings>
        <!-- Logger 설정 -->
        <setting name="logImpl" value="SLF4J" />

        <!-- Mybatis 캐시는 사용하지 않음 (서비스 영역에서 사용) -->
        <setting name="cacheEnabled" value="false" />

        <!-- Prepared Statement 를 재사용 -->
        <setting name="defaultExecutorType" value="REUSE" />

        <!-- Multiple Result-set 설정(허용하지 않음). 허용이 꼭 필요한 경우 true 로 변경 -->
        <setting name="multipleResultSetsEnabled" value="false" />

        <!-- 필드와 프로퍼티간 자동 매핑
         Partial: nested result mapping 은 지원하지 않음. 단일 테이블에 대해서 지원
        -->
        <setting name="autoMappingBehavior" value="PARTIAL" />

        <!-- Table: ABC_DEF -> Vo: abcDef 로 매핑  -->
        <setting name="mapUnderscoreToCamelCase" value="false" />

        <!-- Lazy Loading 여부 -->
        <setting name="aggressiveLazyLoading" value="false" />

        <!-- Null Parameter 처리 -->
        <setting name="jdbcTypeForNull" value="NULL" />

    </settings>

    <typeAliases>

        <!-- Article -->
        <typeAlias alias="Article" type="com.kt.edu.secondproject.domain.Article"/>

    </typeAliases>

</configuration>
```  

<br/>

로그 정보의 세밀한 컨트롤을 위해 logback-spring-dev.xml 화일을 생성한다.  

dev 환경에서와 운영환경에서는 설정이 다를수가 있어 향후 profile로 관리한다.  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <jmxConfigurator/>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
    <statusListener class="ch.qos.logback.core.status.NopStatusListener" />

    <!-- STDOUT appender -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %50.50C (%file:%line\) - %msg%n
            </Pattern>
        </layout>
    </appender>

    <!-- application Logger -->
    <logger name="com.kt.edu" level="DEBUG" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>

    <!-- springframework Loggers -->
    <logger name="org.springframework.core" level="ERROR" additivity="false" >
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="org.springframework.beans" level="ERROR" additivity="false" >
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="org.springframework.context" level="ERROR" additivity="false" >
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="org.springframework.web" level="ERROR" additivity="false" >
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="org.springframework.tx" level="ERROR" additivity="false" >
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="org.springframework.jdbc" level="ERROR" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <!-- jdbc loggers -->
    <logger name="jdbc.sqltiming" level="INFO" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="jdbc.sqlonly" level="ERROR"  additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="jdbc.resultsettable" level="INFO" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="org.apache.ibatis" level="ERROR" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <logger name="net.sf.log4jdbc.log.slf4j" level="ERROR" additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <!-- connection pool -->
    <logger name="com.zaxxer.hikari" level="INFO" additivity="false">
        <appender-ref ref="STDOUT"/>
    </logger>

    <!-- root -->
    <root level="ERROR">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```  

<br/>

### 프로젝트 구성하기     

<br/>

프로젝트 이름에서 마우스 오른쪽을 클릭하고  New -> Package를 선택하여 4개의 package를 만들고 resources 폴더에는 mapper 디렉토리를 생성합니다.  

- domain
- controller
- repository
- service


<img src="./assets/mybatis3.png" style="width: 80%; height: auto;"/> 

<br/>

controller 패키지에 ArticleController 를 rest로 생성합니다.  

ArticleController  

```java
package com.kt.edu.secondproject.controller;


import com.kt.edu.secondproject.domain.Article;
import com.kt.edu.secondproject.service.ArticleService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/articles")
public class ArticleController {

    @Autowired
    private final ArticleService articleService;

    public ArticleController(ArticleService articleService) {
        this.articleService = articleService;
    }

    @GetMapping
    public List<Article> findAll() {
        return this.articleService.findAll();
    }

    @GetMapping("/{id}")
    public Article findById(@PathVariable Long id) {
        return this.articleService.findById(id);
    }

   /* @GetMapping("/active")
    public List<Article> findAllActive() {
        return this.articleService.findAllActive();
    }

    @GetMapping("/inactive")
    public List<Article> findAllInactive() {
        return this.articleService.findAllInactive();
    }*/

    @PostMapping
    public Article create(@RequestBody Article article) {
        return this.articleService.create(article);
    }

    /*@DeleteMapping("/{id}")
    public void delete(@PathVariable Long id) {
        this.articleService.delete(id);
    }*/
}
```

<br/>

domain 패키지에 Article DTO 를 생성한다.  

```java
package com.kt.edu.secondproject.domain;

import lombok.*;

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Article {
    private Long id;
    private String title;
    private String content;
}
```
 
<br/>
repository 패키지에 ArticleMapper Interface 를 생성한다.  

<img src="./assets/mybatis4.png" style="width: 80%; height: auto;"/>

ArticleMapper
```java
package com.kt.edu.secondproject.repository;

import com.kt.edu.secondproject.domain.Article;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.context.annotation.Configuration;

import java.util.List;
import java.util.Optional;

@Mapper
public interface ArticleMapper {

    public long getId();
    public long insert(Article article);
    public long update(Article article);
    public Optional<Article> findById(Long id);

    //@Select("SELECT * FROM article")
    public List<Article> findAll();

}
```  

<br/>

service 패키지에 ArticleService class 를 생성한다.  

ArticleService

```java
package com.kt.edu.secondproject.service;

import com.kt.edu.secondproject.domain.Article;
import com.kt.edu.secondproject.repository.ArticleMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@Transactional
@Slf4j
public class ArticleService {

    @Autowired
    private  ArticleMapper articleMapper;

    public Article create(Article article) {
        log.info("Request to create Article : " +  article);
        articleMapper.insert(article);
        return article;
    }

    public List<Article> findAll() {
        log.info("Request to get all Articles");
        //return this.articleMapper.findAll();
        return articleMapper.findAll()
         .stream()
         .collect(Collectors.toList());
    }

    @Transactional(readOnly = true)
    public Article findById(Long id) {
        log.debug("Request to get Article : {}", id);
        return articleMapper.findById(id).get();
    }

    /*
    public List<Article> findAllActive() {
        log.debug("Request to get all Articles");
        return this.articleMapper.findAllByEnabled(true);
    }

    public List<Article> findAllInactive() {
        log.debug("Request to get all Articles");
        return this.articleMapper.findAllByEnabled(false);
    }*/

    /*public void delete(Long id) {
        log.debug("Request to delete article : {}", id);

        Article article = this.articleMapper.findById(id)
                .orElseThrow(() -> new IllegalStateException("Cannot find Article with id " + id));

        article.setEnabled(false);
        this.articleMapper.update(article);
    }*/
}
```

<br/>
resources 폴더에 mapper 디렉토리를 생성하고 Article Mapper xml 화일을 생성한다.   

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.kt.edu.secondproject.repository.ArticleMapper">
    <select id="getId" resultType="long">
        select hibernate_sequence.nextval from dual
    </select>

    <insert id="insert" parameterType="Article">
        <selectKey keyProperty="id" resultType="long" order="BEFORE">
            select hibernate_sequence.nextval from dual
        </selectKey>
        insert into article
        (id, title, content)
        values
        (#{id}, #{title}, #{content})
    </insert>


    <update id="update" parameterType="Article">
        update article set
            title = #{title}
                           , content = #{content}
        where id = #{id}
    </update>

    <select id="findById" parameterType="long" resultType="Article">
        select
            id
             , title
             , content
        from article
        where id = #{id}
    </select>

    <select id="findAll" resultType="Article">
        select
            id
             , title
             , content
        from article
    </select>

</mapper>
```  

<br/> 

전체 폴더 구성은 아래와 같습니다.   

<img src="./assets/mybatis5.png" style="width: 80%; height: auto;"/>

<br/>

### 실행하기     

<br/>

SpringBoot 를 기동하고 에러가 발생하지 않으면 웹 브라우저에서 http://localhost:8080/articles 를 입력하여 데이터를 조회합니다.  

<img src="./assets/mybatis6.png" style="width: 80%; height: auto;"/>  

<br/>

json 형태로 데이터가 나오는 것을 확인 할 수 있습니다.  

http://localhost:8080/articles/1 호출하여 단건 데이터를 확인 합니다.  


<br/>

Talend API Tester로 데이트를 입력해 봅니다.    
- Method : post
- URL : http://localhost:8080/articles/
- header : JSON
- BODY : 
    ```json
    {
        "id" : 4,
        "title": "mybatis 4",
        "content" : "4번 test 합니다."
    }
    ```  

위와 같이 설정하고 send 버튼을 4번 누르면 데이터가 정상으로 입력이 됩니다.  
- pk 오류가 발생하고 sequence 가 4까지 증가하면 에러 발생 안함.  

성공하면 아래와 같은 화면을 볼 수 있습니다.  

<img src="./assets/mybatis7.png" style="width: 80%; height: auto;"/>  

<br/> 

웹브라우저에서 DB를 접속해서 데이터를 확인 합니다.  

<img src="./assets/mybatis8.png" style="width: 80%; height: auto;"/>

Article 테이블을 조회 하여 데이터를 확인 합니다.  

<img src="./assets/mybatis9.png" style="width: 80%; height: auto;"/>
