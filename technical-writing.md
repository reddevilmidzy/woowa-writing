# 스프링 빈 프로퍼티 애노테이션

## 스프링 빈이란?

스프링 빈은 스프링 IoC 컨테이너에서 관리하는 컴포넌트이다. IoC는 Inversion Of Control의 약자로 제어의 역전을 의미한다. 
이는 프로그래머가 작성한 프로그램의 흐름 제어를 다른 무언가에게 위임하는 디자인 패턴이다. 
스프링 빈을 등록한다는 것은 프로그래머가 작성한 객체의 생명 주기 관리를 스프링 프레임워크에게 넘긴다는 것이다. 
빈을 등록하는 방법으로는 @Component 애노테이션을 사용하여 컴포넌트 스캔을 통해 자동으로 등록하는 방법과 설정 클래스에 @Configuration 애노테이션을 붙여 @Bean 애노테이션을 통해 수동으로 등록하는 방법이 있다. 
그리고 xml 파일을 만들어 빈을 등록할 수도 있지만 최근에 많이 사용하는 방법은 아니다.  

이번 글에서 소개할 내용은 빈 프로퍼티에 사용되는 애노테이션이다. 

<br>

## @Profile

### 프로파일이 필요한 경우

프로젝트를 할 때, 개발, 테스트, 운영 환경마다 환경 설정을 다르게 가져간 경험이 있을 것이다. 
개발 환경에서는 H2를 사용하고 배포 환경에서는 MySQL을 사용한다던가, 보여지는 로깅 레벨을 다르게 가져갈 수 있다. 
또한 개발 중에만 활성화되고 운영에는 배포되지 않아야 하는 빈이 있을 수도 있다. 
이처럼 같은 프로젝트 내에서 다른 개발 환경마다 환경 설정을 다르게 가져가고 싶을 때 사용하는 것이 스프링 부트의 프로파일이다.  

application-{environment}.yml 혹은 application-{environment}.properties 컨벤션으로 프로파일에 대한 환경 설정 파일을 만들 수 있다.

**application-dev.properties**  

```properties
app.info= This is the DEV Environment Property file
spring.h2.console.enabled=true
spring.h2.console.path=/h2
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:db
spring.datasource.userName=sa
spring.datasource.password=sa
```

<br>

**application-test.properties**

```properties
app.info= This is the TEST Environment property file
spring.datasource.url=jdbc:mysql://localhost:3306/myTestDB
spring.datasource.username=root
spring.datasource.password=root123
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
```

<br>

**application-prod.properties**

```properties
app.message = This is the PROD Environment property file
spring.datasource.url=jdbc:oracle:thin:@localhost:1521:xe
spring.datasource.username=username
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.Oracle10gDialect
```

### 프로파일 활성화 방법

프로그램을 실행할 때 `No active profile set, falling back to 1 default profile: "default"` 이런 로그를 마주했을 것이다. 
이는 지정된 프로파일이 없어 기본 프로파일 default로 적용되었다는 의미이다. 특정 프로파일을 활성화하는 방법은 4가지가 있다. 


#### application.properties의 spring.profile.active 값 변경하기

아래 처럼 환경 설정 파일을 통해 실행할 스프링 부트 애플리케이션 환경을 설정할 수 있다. 

```properties
spring.application.name = Spring Profiles
spring.profiles.active = dev
app.info = This is the Primary Application Property file
```

가장 많이 사용되는 방식이다.

#### JVM 시스템 파라미터 변경하기

JVM 시스템 파라미터를 통해 프로파일을 설정할 수 있다.
```bash
-Dspring.profiles.active=dev
```

#### web.xml 파일 변경하기

```xml
<context-param>
    <param-name>spring.profiles.active</param-name>
    <param-value>dev</param-value>
</context-param>
```

#### WebApplicationInitializer 인터페이스 구현하기

```java

@Configuration
public class MyWebApplicationInitializer implements WebApplicationInitializer {

   @Override
   public void onStartup(ServletContext servletContext) throws ServletException {
      servletContext.setInitParameter( "spring.profiles.active", "dev"); 
   } 
}
```

#### pom.xml 파일 변경하기

Maven 파일에서 spring.profile.active 속성을 지정할 수 있다. 

```xml
<profiles>
  <profile>
     <id>dev</id>
     <activation> <activeByDefault>true</activeByDefault> </activation>
     <properties> <spring.profiles.active>dev</spring.profiles.active> </properties>
  </profile>
  <profile>
     <id>test</id>
     <properties> <spring.profiles.active>test</spring.profiles.active> </properties>
  </profile>
</profiles>
```

<br>

다양한 프로파일 설정 방법을 알아보았는데, 우선순위는 아래와 같다.  

1. web.xml 변경
2. WebApplicationInitializer 인터페이스 구현
3. JVM 파라미터 변경
4. pom.xml 변경


## @Profile 사용방법

@Profile 애노테이션을 사용하면 빈을 특정 프로파일에 속하게 만들 수 있다. 

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	String[] value();
}
```

<br>

value에 적용하고 싶은 프로파일을 넣으면 된다. 이때 !, &, | 연산자를 활용해 여러 프로파일을 조합하여 빈의 등록을 설정할 수도 있다.


```java
@Component
@Profile("dev")
public class AConfig{ }

@Component
@Profile("!dev")
public class BConfig{ }

@Component
@Profile("dev | prod")
public class CConfig{ }

```
위 코드에서 AConfig는 환경이 dev일 때만 등록되고 BConfig는 dev환경이 아닐때만 등록된다. 그리고 CConfig는 dev나 prod 프로파일 중 하나라도 활성화되면
등록된다. 

<br>

```java
@Profile("test | prod")
@Configuration
public class DataSourceConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.replica.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create()
                .build();
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.replica.slave")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create()
                .build();
    }
}
```

프로젝트에서는 운영과 테스트 환경에서는 master DB와 slave DB를 나누었지만, 로컬환경에서는 나누는 것이 불필요했기에 @Profile 애노테이션을 활용해서 local 환경에서는 빈등록이 되지 않도록 하였다. 

<br>

## @Order





---

### 참고 자료

[profiles-in-spring-boot](https://javatechonline.com/profiles-in-spring-boot/)
