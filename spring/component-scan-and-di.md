# Component Scan and DI

<br>

## 목차
* [Component Scan](#Component-Scan)
  * [@ComponentScan](#componentscan)
  * [@Component](#component)
  * [@Autowired](#autowired)
* [중복되는 스프링 빈 등록과 충돌](#중복되는-스프링-빈-등록과-충돌)

<br>

## Component Scan
* 설정 정보(`@Configuration`, `@Bean`)가 없어도 자동으로 스프링 빈을 등록할 수 있음

<br>

### 예시

```java
@Configuration
@ComponentScan
public class AutoAppConfig { }
```

* `@ComponentScan` 어노테이션이 붙은 컨테이너 클래스 생성

```java
@Component
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository;

    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

* `@Component`가 있는 모든 클래스를 스프링 빈으로 등록
* `@Autowired`를 생성자에 추가해 자동으로 DI 수행

### 스프링 빈 이름
* 스프링 빈의 이름으로 `클래스 명` 사용, 앞 글자는 `소문자`로 함
  * ` MemberServiceImpl `클래스 -> `memberServiceImpl`
* `@Component("memberService2")`으로 이름 지정도 가능

### @ComponentScan
* `basePackages`, `basePackageClasses`로 스캔 시작 위치 지정 가능 ([사용 방법 링크](./annotation.md))
* `includeFilters`, `excludeFilters`로 스프링 빈으로 등록할 것인지 제외할 것인지 지정 ([사용 방법 링크](./annotation.md))
  * 사용할 일이 많지 않음
* `@SpringBootApplication`은 `@ComponentScan`을 포함 <br>
  -> `@SpringBootApplication`을 프로젝트 최상단에 놓는 것이 관례 (스프링 부트 사용 시)

> 권장되는 방법 <br>
> 패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단으로 함 <br>
> 옵션을 변경하면서 사용하기보다는 스프링의 기본 설정에 최대한 맞추어 사용하는 것을 권장

### @Component
* `@Controlller`, `@Service`, `@Repository`, `@Configuration`은 `@Component`를 포함

> 애노테이션에는 상속관계가 없음 <br>
> 애노테이션이 특정 애노테이션을 들고있는 것을 인식하는 것은 스프링이 지원하는 기능

### @Autowired
* `getBean(MemberRepository.class)`과 같음

<br>

## 중복되는 스프링 빈 등록과 충돌

### 자동 빈 등록 + 자동 빈 등록
* `ConflictingBeanDefinitionException` 예외 발생
* 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, 이름이 같은 경우 스프링은 오류를 발생시킴

### 수동 빈 등록 + 자동 빈 등록
* 테스트에서 실행 시 오류가 발생하지 않음: 수동 빈이 우선권을 가짐, 수동 빈이 자동 빈을 `오버라이딩`

```
Overriding bean definition for bean 'beanName' with a different definition: replacing
```
* 설정이 꼬여 알 수 없는 오류가 발생할 수 있음

* 스프링 부트에서 실행시 오류 발생

```
Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```