# 스프링 컨테이너와 스프링 빈

<br>

## 목차
* [스프링 컨테이너와 빈 생성](#스프링-컨테이너와-빈-생성)
* [스프링 컨테이너와 빈 사용](#스프링-컨테이너와-빈-사용)
* [스프링 컨테이너 정리](#스프링-컨테이너-정리)
* [스프링 빈 정리](#스프링-빈-정리)
<br>

## 스프링 컨테이너와 빈 생성

```java
@Configuration
public class AppConfig {

    @Bean
    // 스프링 빈의 이름: 메서드 명(memberService)
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), orderPolicy());
    }

    @Bean
    public OrderPolicy orderPolicy() {
        return new MemberOrderPolicy();
    }
}
```
### @Configuration
* 설정을 구성한다는 의미
* `@Bean` 어노테이션을 사용할 경우 반드시 `@Configuration`과 함께 사용해야 함

### @Bean
* 스프링 컨테이너에 스프링 빈으로 등록
* 스프링 빈은 private이나 final이 아니어야 함

<br>

## 스프링 컨테이너와 빈 사용

```java
public class OrderApp {
    public static void main(String[] args) {

        // 스프링 컨테이너
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);

        // 스프링 빈 탐색
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
    }
}
```

* `AppConfig`에서 메서드 이름을 Key, 리턴 객체를 Value로 하여 스프링 빈이 생성됨

```
Creating shared instance of singleton bean 'appConfig'
Creating shared instance of singleton bean 'memberService'
Creating shared instance of singleton bean 'memberRepository'
Creating shared instance of singleton bean 'orderService'
Creating shared instance of singleton bean 'orderPolicy'
```

### 👉 스프링 컨테이너 생성
* `ApplicationContext`: 스프링 컨테이너 (인터페이스)
* 어노테이션 기반으로 생성
* `AppConfig.class`: 구성 정보 지정 필요

### 👉 스프링 빈 등록
* 스프링 컨테이너는 파라미터로 넘어온 클래스 정보를 이용해 스프링 빈을 등록

|스프링 빈 이름|스프링 빈 객체|
|:--:|:--:|
|memberService|MemberServiceImpl@x01|
|memberRepository|MemoryMemberRepository@x02|

#### 스프링 빈 이름
* 메서드 이름 사용
* 직접 이름 설정도 가능

```java
@Bean(name="newName")
```

* 이름은 항상 `Unique`한 이름을 사용해야 함

### 👉 의존 관계 설정
* `AppConfig` 클래스에서 의존 관계를 주입(DI)
* 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리

<br>

## 스프링 컨테이너 정리

> ApplicationContext <br>
> 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용

* 스프링 컨테이너는 `@Configuration` 이 붙은 클래스(`AppConfig`)를 설정(구성) 정보로 사용
* `@Bean`이 적용된 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록 

<br>

## 스프링 빈 정리
* 스프링 컨테이너에 등록된 객체 (`@Bean`으로 등록된 객체)
* DI Container에 의해 관리되는 POJO(Plain Old Java Object)
* 스프링 빈의 이름: `@Bean`이 붙은 메서드의 이름
* 스프링 빈 탐색: ApplicationContext.getBean("이름", 형식);
