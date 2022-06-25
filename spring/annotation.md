# Annotation

### @Configuration
* 설정을 구성한다는 의미
* `@Bean` 어노테이션을 사용할 경우 반드시 `@Configuration`과 함께 사용해야 함

### @Bean
* 스프링 컨테이너에 스프링 빈으로 등록
* 스프링 빈은 private이나 final이 아니어야 함

### @ComponentScan
* `@ComponentScan` 어노테이션이 붙은 컨테이너 클래스 생성
* `@Component`가 있는 모든 클래스를 스프링 빈으로 등록
* `@Configuration`이 붙은 설정 정보도 함께 등록
  * `@Configuration` 소스 코드에 `@Component`가 있어 등록됨
* useDefaultFilters: 기본으로 true, `@Component`, `@Controlller`, `@Service`, `@Repository`가 붙은 클래스를 스캔할지 결정
* basePackages: 탐색할 패키지의 시작 위치 지정
  * 여러 개 지정도 가능

  ```java
  basePackages = {"hello.core", "hello.service"} 
  ```

* basePackageClasses: 지정한 클래스의 패키지를 탐색 시작 위치로 지정
* includeFilters : 컴포넌트 스캔 대상을 추가로 지정
* excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정
  * `type`은 `FilterType.ANNOTATION`이 기본 값
  * `type = FilterType.ASSIGNABLE_TYPE`: 지정한 타입과 자식 타입을 인식해서 동작

```java
@ComponentScan(
        basePackages = "spring.package",
        basePackageClasses = AutoAppConfig.class,
        includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = IncludeComponent.class),
        excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = ExcludeComponent.class)
    
)
public class AutoAppConfig { }
```

### @Component
* 컴포넌트 스캔의 대상이 되도록 함
* 스프링 빈 등록 대상이 되는 클래스
* 스프링 빈의 이름으로 `클래스 명` 사용, 앞 글자는 `소문자`로 함
  * ` MemberServiceImpl `클래스 -> `memberServiceImpl`
  * 이름 지정 가능

  ```java
  @Component("newName")
  ```

* `@Controlller`, `@Service`, `@Repository`, `@Configuration`은 `@Component`를 포함

### @Autowired
* 생성자에 추가해 자동으로 DI 수행
* 생성자가 1개일 경우에는 생략 가능
* `@Bean`을 사용하지 않고 의존관계 주입이 가능하게 함

### @Controlller
* 스프링 MVC 컨트롤러
* 스프링 MVC 컨트롤러로 인식

### @Service
* 스프링 비즈니스 로직에 사용
* 특별한 기능은 없음, 개발자가 비지니스 계층이라고 인식하는 것에 이용

### @Repository
* 스프링 데이터 접근 계층에서 사용
* 스프링 데이터 접근 계층으로 인식, 데이터 계층의 예외를 스프링 예외로 변환

### @Configuration
* 스프링 설정 정보에서 사용
* 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처리 진행

### @RequiredArgsConstructor
* lombok 라이브러리의 어노테이션
* `final`이 붙은 필드로 생성자를 자동으로 생성

### @Qualifier
* 의존관계 주입에서 추가 구분자를 붙이는 방법
* `@Autowired`에서 여러 빈이 매칭되면 `@Qualifier`로 지정된 이름의 빈이 매칭
* 주입시 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것이 아님
* `@Qualifier`로 빈을 찾지 못하면 `NoSuchBeanDefinitionException` 발생

### @Primary
* 의존관계 주입에서 주입할 빈의 `우선순위`를 정하는 방법
* `@Autowired`에서 여러 빈이 매칭되면 `@Primary`가 있는 빈이 매칭됨

### @PostConstruct
* 빈 생명주기에서 의존관계 주입 후에 초기화하는 메서드

### @PreDestroy
* 빈 생명주기에서 컨테이너 종료전에 호출되는 메서드