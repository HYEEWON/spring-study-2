# 의존관계 주입

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


