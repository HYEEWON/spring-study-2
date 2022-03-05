# 관심사 분리

<br>

## 목차
* [관심사 분리의 필요성](#관심사-분리의-필요성)
* [AppConfig](#appconfig)

<br>

## 관심사 분리의 필요성

> SRP, DIP, OCP 위반
> 의존 관계와 실행 모두를 진행하는 코드

```java
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final OrderPolicy orderPolicy = new MemberOrderPolicy();
}

```

* SRP 위반: 구현 객체 생성 및 연결, 실행하는 역할 수행
* DIP 위반: 인터페이스와 구현체 모두에 의존
* OCP 위반: 구현 수정을 위해서는 소스 수정 필요

<br>

## AppConfig

>애플리케이션의 전체 동작 방식을 구성(config)하기 위해 사용
> `구현 객체 생성` 및 `연결(생성자 주입)`

```java
public class AppConfig {
    
    // OrderServiceImpl은에 주입될 객체를 외부(AppConfig)에서 결정
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), orderPolicy ());
    }

    // 구현체 정의
    private MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public DiscountPolicy orderPolicy() {
        return new MemberOrderPolicy();
    }
    
}
```

* `AppConfig`는 `MemoryMemberRepository` 객체, `MemberOrderPolicy` 객체를 생성하고, 이 참조 값을 `OrderServiceImpl`을 생성하면서 생성자로 전달

```java
public class OrderServiceImpl implements OrderService {

    // 인터페이스에만 의존하여 DIP 유지
    private final MemberRepository memberRepository;
    private  DiscountPolicy discountPolicy;
    
    // 생성자
    // 어떤 구현 객체가 주입될지 OrderServiceImpl은 알 수 없음
    public OrderServiceImpl(MemberRepository memberRepository, MemberOrderPolicy memberOrderPolicy) {
        this.memberRepository = memberRepository;
        this.memberOrderPolicy = memberOrderPolicy;
    }
}
```

* `OrderServiceImpl` 입장에서 `의존 관계는 외부에서 주입`되어 `실행에만 집중`

> DIP 유지: 인터페이스에만 의존 <br>
> OCP 준수: 구현체는 AppConfig에서 변경하여 클라이언트(Impl)의 코드 변경 없음

<br>

### 👉 DI(Dependency Injection, 의존관계 주입, 의존성 주입)
* 클라이언트 `OrderServiceImple` 입장에서 보면 의존 관계를 외부에서 주입

### 👉 관심사 분리

> Impl은 의존 관계에 대한 설정을 하지 않고, 실행에만 집중

* 주입되는 객체를 외부(AppConfig)에서 결정함

### 👉 실행

```java
public class OrderApp {

    public static void main(String[] args) {

        // AppConfig로 객체 생성
        AppConfig appConfig = new AppConfig();

        MemberService memberService = appConfig.memberService();
        OrderService orderService = appConfig.orderService();
        
        // Member, Order 객체 생성 및 OrderService 메소드 실행
    }
}

```
