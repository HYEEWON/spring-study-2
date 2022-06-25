# 스프링 빈 생명주기 콜백

<br>

## 목차
* [스프링 빈 생명 주기](#스프링-빈-생명-주기)
  * [권장 사항](#권장-사항)
* [빈 콜백 1: 인터페이스(InitializingBean, DisposableBean)](#빈-콜백-1-인터페이스initializingbean-disposablebean)
* [빈 콜백 2: 설정 정보에 초기화 메서드, 종료 메서드 지정](#빈-콜백-2-설정-정보에-초기화-메서드-종료-메서드-지정)
* [빈 콜백 3: @PostConstruct, @PreDestroy](#빈-콜백-3-postconstruct-predestroy)
* [정리](#정리)

<br>

## 스프링 빈 생명 주기

> 스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸전 콜백 -> 스프링 종료 <br>

* 초기화 콜백: 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
* 소멸 전 콜백: 빈이 소멸되기 직전에 호출

* 스프링 빈은 `객체를 생성`하고, `의존관계 주입`을 진행, 이후에 데이터 `초기화`를 해야 함
* 스프링은 의존관계 주입이 완료되면 스프링 빈에게 `콜백 메서드`를 통해서 초기화 시점을 알려줌
* 스프링은 스프링 컨테이너가 종료되기 직전에 `소멸 콜백`을 줌

### 권장 사항

> 객체의 생성과 초기화를 분리하자. <br>

* `유지보수` 관점에서 좋음
* 생성자는 필수 정보(파라미터)를 받고, 메모리를 할당해서 객체를 생성
* 초기화는 생성된 값들을 활용해서 외부 커넥션을 연결하는등 무거운 동작을 수행

<br>

## 빈 콜백 1: 인터페이스(InitializingBean, DisposableBean)

* `InitializingBean.afterPropertiesSet`: 메서도로 초기화 지원
* `DisposableBean.destroy`: 메서드로 소멸 지원

```java
public class TestClass implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("NetworkClient.afterPropertiesSet");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("NetworkClient.destroy");
    }
}
```

* 실행 결과

```
생성자 호출
NetworkClient.afterPropertiesSet
DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@3bb9a3ff
NetworkClient.destroy
```

* 단점: 스프링 전용 인터페이스, 메서드 이름 변경 불가, 코드를 고칠 수 없는 외부 라이브러리에 적용 불가
* 이 방법은 초창기 방법으로 현재는 다른 방법이 많아 **거의 사용하지 않음**

<br>

## 빈 콜백 2: 설정 정보에 초기화 메서드, 종료 메서드 지정

```java
@Bean(initMethod = "method_name1", destroyMethod = "method_name2")
```

```java
@Configuration
    static class TestClass {
        @Bean(initMethod = "method_name1", destroyMethod = "method_name2")
        public TestClass TestClass() {
            TestClass testClass = new TestClass();
            return TestClass;
        }
    }
```

* 특징: 스프링에 의존하지 않음, 메서드 이름 변경 가능, 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용 가능

### destroyMethod
* `@Bean`의 `destroyMethod` 는 기본값이 `(inferred)` (추론)으로 되어 있음
* 라이브러리 대부분은 `close`, `shutdown`으로 종료 메서드를 사용
* 추론 기능은 `close`, `shutdown`이라는 이름의 메서드를 자동으로 호출해 종료 메서드를 지정하지 않아도 스프링 빈으로 등록하면 알아서 동작
* `destroyMethod=""`를 하면 추론 기능을 사용하지 않을 수 있음

<br>

## 빈 콜백 3: @PostConstruct, @PreDestroy

> 가장 권장되는 방법 <br>

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class NetworkClient{

    @PostConstruct
    public void init() throws Exception { }

    @PreDestroy
    public void close() throws Exception { }
}
```

* `javax.annotation.*`: 스프링이 아니라 자바 표준 
* 단점: 외부 라이브러리에는 적용하지 못함

<br>

## 정리

* `@PostConstruct`, `@PreDestroy`을 사용
* 코드를 고칠 수 없는 외부 라이브러리를 초기화, 종료해야 하면 `@Bean`의 `initMethod`, `destroyMethod`를 사용