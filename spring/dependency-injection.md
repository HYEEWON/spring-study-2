# 의존관계 주입

<br>

## 목차
* [생성자 주입](#생성자-주입)
* [Setter(수정자) 주입](#setter수정자-주입)
* [필드 주입](#필드-주입)
* [일반 메서드 주입](#일반-메서드-주입)
* [옵션 처리](#옵션-처리)
* [생성자 주입을 사용하는 것이 좋은 이유](#생성자-주입을-사용하는-것이-좋은-이유)
  * [정리](#정리)
* [Lombok](#lombok)

<br>

## 생성자 주입
* 생성자에 `@Autowired` 를 사용
  * 생성자가 1개일 경우, 어노테이션 생략 가능 
* 생성자 호출시점에 딱 1번 호출되는 것을 보장
* 주로 `불변`, `필수` 의존관계에 사용

```java
@Component
public class SempleServiceImpl implements SempleService {
  private SempleRepository sempleRepository;

  @Autowired
  public SempleServiceImpl(SempleRepository sempleRepository) {
    this.sempleRepository = sempleRepository;
  }
}
```

<br>

## Setter(수정자) 주입
* Setter에서 의존관계 주입 
* `선택`, `변경 가능성`이 있는 의존관계에 사용

```java
@Component
public class SempleServiceImpl implements SempleService {
  private SempleRepository sempleRepository;

  @Autowired
  public void setMemberRepository(SempleRepository sempleRepository) {
    this.sempleRepository = sempleRepository;
  }
}
```

>` @Autowired`는 기본적으로 주입할 대상이 없으면 오류 발생
> 주입할 대상이 없어도 동작하게 하려면 `@Autowired(required = false)`로 지정해야 함

<br>

## 필드 주입
* 필드에서 바로 주입하는 방법
* 코드가 간결함
* 외부에서 변경이 불가능해서 테스트 하기 힘듦
* DI 프레임워크가 없으면 아무것도 할 수 없다.

> 사용하지 않는 것이 좋음 <br>
> 테스트 코드, 스프링 설정을 목적으로 하는 `@Configuration`과 같이 특별한 용도로 사용

```java
@Component
public class SempleServiceImpl implements SempleService {
  @Autowired private SempleRepository sempleRepository;
}
```

<br>

## 일반 메서드 주입
* 일반 메서드를 통해서 주입
* 한번에 여러 필드를 주입 받을 수 있음
* 일반적으로 잘 사용하지 않음
  * Setter 주입과 비슷해서 잘 사용하지 않음


```java
@Component
public class SempleServiceImpl implements SempleService {
  private SempleRepository sempleRepository;

  @Autowired
  public void normalMethod(SempleRepository sempleRepository) {
    this.sempleRepository = sempleRepository;
  }
}
```

<br>

## 옵션 처리
* `@Autowired(required=false)`: 주입 대상이 없으면 수정자 메서드는 호출되지 않음
  * `@Autowired`: 기본적으로 `required=true`로 되어 있어서 자동 주입 대상이 없으면 오류 발생
* `org.springframework.lang.@Nullable`: 자동으로 주입할 대상이 없으면 `null`이 입력됨
* `Optional<>`: 자동으로 주입할 대상이 없으면 `Optional.empty`가 입력됨

```java
public class AutowiredTest {
    @Test
    void AutowiredOption() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);
    }

    static class TestBean {
        @Autowired(required = false)
        public void setNoBean1(Member noBean1) {
            System.out.println("noBean1 = " + noBean1);
        }

        @Autowired
        public void setNoBean2(@Nullable Member noBean2) {
            System.out.println("noBean2 = " + noBean2);
        }

        @Autowired
        public void setNoBean3(Optional<Member> noBean3) {
            System.out.println("noBean3 = " + noBean3);
        }
    }
}
```

* 실행 결과

```java
noBean3 = Optional.empty
noBean2 = null
```
  * `setNoBean1` 메서드는 빈을 주입받지 않아 실행되지 않음

<br>

## 생성자 주입을 사용하는 것이 좋은 이유

> 불변 <br>

* 생성자 주입은 객체 생성 시 1번 호출 이후, 호출되지 않아 불변함
* 일반적으로 의존관계 주입은 최초 1번 진행되어 변경되는 경우가 없음
  * 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안됨
* Setter 주입의 단점: Set 메서드를 `public`으로 해야 함 -> 실수로 변경할 수도 있음
* 필드 주입의 단점: 테스트에서 값을 넣을 수 있는 방법이 없고 스프링 프레임워크 없이 순수 자바 테스트를 할 수 없음

> 누락 <br>

* Setter 주입에서 순수 자바 테스트를 하는 경우, 에러 발생

```java
@Component
public class OrderServiceImpl implements OrderService {
    private MemberRepository memberRepository;
   
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public Order orderMethod(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        return new Order(memberId, itemName, itemPrice);
    }
}
```

* 테스트 시, java.lang.NullPointerException 발생
  * `` 메서드 내에서 의존관계 주입이 필요한데, 의존 관계가 주입되지 않아 에러 발생

```java
public class OrderServiceImplTest {
    @Test
    void createOrder() {
        OrderServiceImpl orderService = new OrderServiceImpl();
        orderService.orderMethod(1L, "itemA", 10000);
    }
}
```

* 생성자 주입을 사용할 경우 `컴파일 에러`가 발생해서 바로 잘못됨을 알 수 있음

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

> final 키워드 <br>

* 생성자 주입을 사용하면 필드에 `final` 키워드를 사용할 수 있음
  * 이 외의 방법에서는 `final` 키워드를 사용할 수 없음
  * 이유: 생성자 생성 이후에 호출되기 때문
* 개발자가 생성자에서 값을 설정하지 않는 실수를 해면 `컴파일 에러`로 알 수 있음
  * java: variable xxx might not have been initialized
* 예시

```java
@Component
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final OrderPolicy OrderPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

### 정리
* 생성자 주입 방식은 불변함, 컴파일 에러로 확인(final 키워드 사용 가능)의 이유로 권장됨
* 생성자 주입 방식은 프레임워크에 의존하지 않는 순수 자바의 특징을 잘 살리는 방법이기도 함
* 기본으로 생성자 주입을 사용하고, 필수 값이 아닌 경우에 Setter 주입 방식을 혼용해서 사용하면 됨
* 필드 주입은 사용하지 않는 것이 좋음

<br>

## Lombok

### 추가 방법
* `build.gradle`에 의존성 추가
```
// build.gradle

dependencies {
	//lombok 라이브러리 추가
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testCompileOnly 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'
}
```

* Settings(Preference)에서 `Annotation Processors` 설정
  * `Enable annotation processing` 체크

### 최신 트랜드
* 생성자를 1개 두고, `@Autowired`를 생략하는 방법을 주로 사용
* 여기에 Lombok 라이브러리의 `@RequiredArgsConstructor`를 사용해 코드를 간결하게 유지