# Spring Container

<br>

## 생성 과정

### 👉 생성

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

* `ApplicationContext`: 스프링 컨테이너 (인터페이스)
* 어노테이션 기반으로 생성
* `AppConfig.class`: 구성 정보 지정 필요

<br>

### 👉 스프링 빈 등록
* 스프링 컨테이너는 파라미터로 넘어온 클래스 정보를 이용해 스프링 빈을 등록

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

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


