# 의존관계 주입에서 발생하는 문제와 해결 방법

<br>

## 목차
* [조회되는 빈이 2개 이상인 경우 발생하는 문제와 해결 방법](#조회되는-빈이-2개-이상인-경우-발생하는-문제와-해결-방법)
  * [해결 방법 1: @Autowired 필드 명 매칭](#해결-방법-1-autowired-필드-명-매칭)
  * [해결 방법 2: @Qualifier](#해결-방법-2-primary)
  * [해결 방법 3: @Primary](#해결-방법-3-primary)
  * [Qualifier와 Primary 정리](#qualifier와-primary-정리)
* [Annotation을 생성해 @Qualifier 단점 해결](#annotation을-생성해-qualifier-단점-해결)
* [조회된 빈이 모두 필요한 경우: Map, List 사용](#조회된-빈이-모두-필요한-경우-map-list-사용)
* [자동, 수동의 기준](#자동-수동의-기준)
  * [수동을 사용하면 좋은 경우](#수동을-사용하면-좋은-경우)
  * [자동을 사용하면 좋은 경우](#자동을-사용하면-좋은-경우)

<br>

## 조회되는 빈이 2개 이상인 경우 발생하는 문제와 해결 방법
* `@Autowired`는 타입을 기준으로 조회해 빈이 2개 이상 조회될 경우, `NoUniqueBeanDefinitionException` 에러 발생
* 예시
  * 인터페이스를 구현한 2개의 스프링 빈이 있을 경우, 인터페이스로 의존관계를 주입하면 오류 발생 

```java
public interface SempleInterface { }
```

```java
@Component
public class Semple1Impl implements SempleInterface { }
```

```java
@Component
public class Semple2Impl implements SempleInterface { }
```

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final SempleInterface SempleInterface;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

> 하위 타입으로 지정할 수 도 있지만, 이는 DIP를 위배함으로 다른 해결 방법을 알아 본다. <br>

### 해결 방법 1: @Autowired 필드 명 매칭
* `@Autowired`는 타입 매칭을 먼저 시도
* 이 결과에 여러 빈이 있을 때 추가로 `파라미터 명`으로 매칭 시도

```java
public interface SempleInterface { }
```

```java
@Component
public class Semple1Impl implements SempleInterface { }
```

```java
@Component
public class Semple2Impl implements SempleInterface { }
```

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final SempleInterface sempleInterface;

    @Autowired private SempleInterface semple1Impl;
    
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, SempleInterface semple1Impl) {
        this.memberRepository = memberRepository;
        this.sempleInterface = semple1Impl;
    }
}
```

### 해결 방법 2: @Qualifier
* 추가 구분자를 붙이는 방법
* 주입시 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것이 아님
* `@Qualifier`로 빈을 찾지 못하면 `NoSuchBeanDefinitionException` 발생

```java
public interface SempleInterface { }
```

```java
@Component
@Qualifier("Semple1")
public class Semple1Impl implements SempleInterface { }
```

```java
@Component
@Qualifier("Semple2")
public class Semple2Impl implements SempleInterface { }
```

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final SempleInterface sempleInterface;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("Semple2") SempleInterface sempleInterface) {
        this.memberRepository = memberRepository;
        this.sempleInterface = sempleInterface;
    }
}
```

### 해결 방법 3: @Primary
* `우선순위`를 정하는 방법
* `@Autowired`에서 여러 빈이 매칭되면 `@Primary`가 있는 빈이 매칭됨

```java
public interface SempleInterface { }
```

```java
@Component
@Primary
public class Semple1Impl implements SempleInterface { }
```

```java
@Component
public class Semple2Impl implements SempleInterface { }
```

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final SempleInterface sempleInterface;
    
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, SempleInterface sempleInterface) {
        this.memberRepository = memberRepository;
        this.sempleInterface = sempleInterface;
    }
}
```

### Qualifier와 Primary 정리
* 주로 사용되는 스프링 빈은 `@Primary`로 사용하고, 그 외의 빈은 `@Qualifier`로 명시적으로 활용하면 좋음
* `@Qualifier`가 `@Primary`보다 우선 순위가 높음
  * `@Primary`는 기본값 처럼 동작하고, `@Qualifier`는 매우 상세하게 동작
  * 스프링은 자동보다 수동, 넒은 범위보다는 좁은 범위일 경우, 우선 순위가 높음

<br>

## Annotation을 생성해 @Qualifier 단점 해결

* `@Qualifier("name")`는 문자로 적어 컴파일에서 에러를 확인할 수 없음
* 애노테이션을 생성하면 컴파일 시에 확인할 수 있고, 코드도 볼 수 있음

```java
// @Qualifier 정의에서 가져오면 됨

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER,
        ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Qualifier("mainSempleAnnotation")
public @interface MainSempleAnnotation {}
```

```java
public interface SempleInterface { }
```

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final SempleInterface sempleInterface;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @MainSempleAnnotation SempleInterface sempleInterface) {
        this.memberRepository = memberRepository;
        this.sempleInterface = sempleInterface;
    }
}
```

<br>

## 조회된 빈이 모두 필요한 경우: Map, List 사용
* `Map`과 `List`를 사용하면 조회된 여러 빈을 모두 사용할 수 있음
* 상황에 따라 원하는 빈을 선택해 사용할 수 있음

```java
public class AllBeanTest {

    @Test
    void findAllBean() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
    }

    static class DiscountService {
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;
        }
    }
}

```

* 실행 결과

```
policyMap = {fixDiscountPolicy=hello.core.discount.FixDiscountPolicy@484094a5, rateDiscountPolicy=hello.core.discount.RateDiscountPolicy@38234a38}
policies = [hello.core.discount.FixDiscountPolicy@484094a5, hello.core.discount.RateDiscountPolicy@38234a38]
```

<br>

## 자동, 수동의 기준

> 수동: @Configuration, @Bean <br>
> 자동: @Component, @Controller , @Service , @Repository

* 최근에는 자동 등록을 선호하기 때문에, 자동 등록을 사용하는 것이 좋음

### 수동을 사용하면 좋은 경우
> 기술 지원과 관련된 빈 <br>

* 예시: 공통 관심 사항(AOP), DB 연결, 로그 처리 등
* 수가 적고, 애플리케이션 전반에 영향을 미침 -> 문제가 발생할 경우, 원인이 잘 드러나지 않음

> 비즈니스 로직 중에서 `다형성`을 활용하는 빈 <br>

* `@Configuration`, `@Bean`으로 정의하는 것이 한 눈에 보기 쉽고, 유지보수에 좋음
* 자동 빈으로 등록할 경우, `패키지`로 묶어두는 것이 좋음

### 자동을 사용하면 좋은 경우
> 비즈니스 요구사항과 관련된 빈 <br>

* 예시: 웹을 지원하는 `컨트롤러`, 핵심 비즈니스 로직이 있는 `서비스`, 데이터 계층의 로직을 처리하는 `리포지토리` 등
* 수가 많고, 유사한 패턴이 있음 -> 문제가 발생할 경우, 원인 파악이 쉬움
