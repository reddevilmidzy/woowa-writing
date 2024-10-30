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
      servletContext.setInitParameter("spring.profiles.active", "dev"); 
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

**Profile.java**
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

**DataSourceConfig.java**
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

스프링이 어떤 식으로 Profile 확인하는지 코드를 통해 살펴보자.  

`@Profile` 애노테이션을 잘 살펴보면 `@Conditional(ProfileCondition.class)` 이 붙어 있는 것을 확인할 수 있다. ProfileCondition 클래스에 들어가보면 `matches` 메서드가 있는데, 이 안에서 현재 실행중인 `Profile`과 설정된 `Profile`을 확인한다.  


```java
class ProfileCondition implements Condition {

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
		if (attrs != null) {
			for (Object value : attrs.get("value")) {
				if (context.getEnvironment().matchesProfiles((String[]) value)) {
					return true;
				}
			}
			return false;
		}
		return true;
	}

}
```



<br>

## @Order


스프링에서 빈의 실행 순서나 우선 순위 지정이 필요할 때가 있다. 
주로 필터, 인터셉터, AOP 어드바이스 등 여러 컴포넌트나 빈이 실행되거나 적용될 때 순서를 제어해야 하는 상황이 생기는데 이때 사용할 수 있는 애노테이션이 @Order다. 


**Order.java**
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
@Documented
public @interface Order {

	int value() default Ordered.LOWEST_PRECEDENCE;

}
```
value 가 낮을 수록 우선순위가 더 높다. 값이 같은 경우에는 랜덤이다.


**Ordered.java**
```java
public interface Ordered {

	int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;

	int LOWEST_PRECEDENCE = Integer.MAX_VALUE;

	int getOrder();

}
```

프로젝트에서는 필터와 RestControllerAdvice 빈들에게 Order를 주었다.

**FilterConfig.java**
```java
@RequiredArgsConstructor
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<AccessTokenSessionFilter> accessTokenSessionFilter() {
        final FilterRegistrationBean<AccessTokenSessionFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new AccessTokenSessionFilter());
        bean.addUrlPatterns("/api/sign-up", "/api/sign-in/callback");
        bean.setOrder(2);
        return bean;
    }

    @Bean
    public FilterRegistrationBean<SignInCookieFilter> signInCookieFilter(final JwtProvider jwtProvider) {
        final FilterRegistrationBean<SignInCookieFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new SignInCookieFilter(jwtProvider));
        bean.addUrlPatterns("/api/sign-out", "/api/member", "/api/sign-in/check");
        bean.setOrder(1);
        return bean;
    }

    @Bean
    public FilterRegistrationBean<AuthFailHandlerFilter> authFailHandlerFilter(final ObjectMapper objectMapper) {
        final FilterRegistrationBean<AuthFailHandlerFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new AuthFailHandlerFilter(objectMapper));
        bean.setOrder(0);
        return bean;
    }
}
```

**ReferenceLinkExceptionHandler.java**
```java
@Slf4j
@RestControllerAdvice
@Order(Ordered.HIGHEST_PRECEDENCE)
public class ReferenceLinkExceptionHandler {
    @ExceptionHandler(ReferenceLinkException.class)
    public ResponseEntity<ApiErrorResponse> handleReferenceLinkException(final ReferenceLinkException e) {
        log.warn(e.getMessage());

        return ResponseEntity.status(ReferenceLinkApiError.BAD_REQUEST.getHttpStatus())
                .body(new ApiErrorResponse(ReferenceLinkApiError.BAD_REQUEST.getMessage()));
    }
}
```

**TimerExceptionHandler.java**
```java
@Slf4j
@RestControllerAdvice
@Order(Ordered.HIGHEST_PRECEDENCE)
public class TimerExceptionHandler {
    @ExceptionHandler(TimerException.class)
    public ResponseEntity<ApiErrorResponse> handleTimerException(final TimerException e) {
        log.warn(e.getMessage());

        return ResponseEntity.status(TimerApiError.INVALID_REQUEST.getHttpStatus())
                .body(new ApiErrorResponse(TimerApiError.INVALID_REQUEST.getMessage()));
    }
}
```

**CommonExceptionHandler.java**
```java
@Slf4j
@RestControllerAdvice
public class CommonExceptionHandler extends ResponseEntityExceptionHandler {
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiErrorResponse> handleException(final Exception e) {
        log.error(e.getMessage(), e);

        return ResponseEntity.status(CommonApiError.SERVER_ERROR.getHttpStatus())
                .body(new ApiErrorResponse(CommonApiError.SERVER_ERROR.getMessage()));
    }
}
```

Order를 주지 않았을 때의 문제점이 모든 예외가 common 패키지 안에 @ExceptionHandler(Exception.class)에 잡힌 다는 것이였다. 
TimerException도 @ExceptionHandler(TimerException.class)에 잡히는 것이 아니라 @ExceptionHandler(Exception.class)에 잡혔다.
TimerException이 Exception의 자식이지만 더 구체적인 TimerException에 잡힐 줄 알았는데 그게 아니였다. 
@ExceptionHandler를 각기 다른 클래스에 두지 않고 한 클래스에 두었을 때에는 의도한대로 TimerException에서 잡혔다. 

그래서 해결 방안으로 나온 것이 @Order의 사용이다. CommonExceptionHandler을 제외한 다른 Handler에게 @Order(Ordered.HIGHEST_PRECEDENCE) 추가하여 우선 순위를 부여해 CommonExceptionHandler 보다 먼저 
빈 등록이 되게 하였다. 

<br>

## 결론

이 글에서는 스프링 빈 프로퍼티와 관련된 두 가지 중요한 애노테이션인 @Profile과 @Order에 대해 살펴보았다.
@Profile 애노테이션은 다양한 개발 환경에 따라 빈의 등록을 제어할 수 있게 해준다. 
이를 통해 환경별로 다른 설정을 적용하거나 특정 환경에서만 필요한 빈을 관리할 수 있다.  

@Order 애노테이션은 빈의 실행 순서나 우선순위를 지정하는 데 사용된다. 특히 필터, 인터셉터, AOP 어드바이스 등 여러 컴포넌트의 실행 순서를 제어해야 할 때 유용하다.  
  
이러한 애노테이션들을 적절히 활용하면 스프링 애플리케이션의 유연성과 확장성을 크게 향상시킬 수 있다.  
환경별 설정 관리와 컴포넌트 실행 순서 제어를 통해 더 효율적이고 안정적인 애플리케이션을 구축할 수 있다.  

---

### 참고 자료

* [profiles-in-spring-boot](https://javatechonline.com/profiles-in-spring-boot/)  
* [spring order docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/annotation/Order.html)
