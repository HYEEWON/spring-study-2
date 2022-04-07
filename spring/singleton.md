# 싱글톤 컨테이너

<br>

## 싱글톤 패턴
* 클래스의 객체르가 `1개`만 생성되는 것을 보장하는 디자인 패턴

```java
public class SingletonService {
    // static 객체로 프로그램에서 1개만 생성
    private static final SingletonService intance = new SingletonService();

    // 항상 같은 객체 반환
    public static SingletonService getInstance() {
        return intance;
    }

    // private 생성자: 외부에서 new 키워드로 객체 생성 불가
    private SingletonService() {}
}

```

### 👉 장점
* 같은 요청이 여러 번 들어올 때, 객체를 계속 생성하는 것이 아니라 객체 1개를 `공유`해 대응

### 👉 단점
* 싱글톤 패턴 구현에는 코드가 많이 필요
* 구현에서 구체 클래스에 의존하여 DIP, OCP 위반
* 내부 속성을 변경하기 어렵고 private 생성자로 상속이 어려움
* -> `유연성 하락`

<br>

## 싱글톤 컨테이너

> 스프링 컨테이너는 싱글톤 컨테이너 <br>
> 기본적으로 스프링은 싱글톤으로 빈을 등록 (다른 방식도 가능)

* 스프링은 `싱글톤 레지스트리`를 가져 일반적인 클래스를 싱글톤으로 활용할 수 있음
* 싱글톤 레지스트리: 싱글톤 객체를 생성하고 관리하는 기능

```java
SingletonService singletonService1 = SingletonService.getInstance();
//2. 조회: 호출할 때 마다 같은 객체를 반환
SingletonService singletonService2 = SingletonService.getInstance();
```

### 👉 장점
* 싱글톤 패턴을 위한 긴 코드가 없어도 됨
* DIP, OCP를 지키고 자유롭게 상속 가능

### 👉 주의점
> 클래스를 `무상태(stateless)`로 설계해야 됨

* 특정 클라이언트에 의존적이거나 값을 변경할 수 있는 필드는 없어야 함
* 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 함

#### 예시

```java
public class StatefulService {
    private int price; //상태를 유지하는 필드

    public void func(int price) {
        this.price = price; // 문제 발생
    }
}
```

<br>

## @Configuration과 바이트 조작

* `@Configuration`이 붙은 클래스도 스프링 빈

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
AppConfig bean = ac.getBean(AppConfig.class); // 스프링 빈 등록

System.out.println("bean = " + bean.getClass());
// 출력 결과
// bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$6245315a
```

> `AppConfig`가 아닌 `AppConfig$$EnhancerBySpringCGLIB` 이 출력 <br>
> `CGLIB`를 사용해서 AppConfig 클래스를 `상속`받은 임의의 클래스를 만들고, 이 클래스를 스프링 빈으로 등록

* CGLIB: 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리
* `AppConfig@CGLIB`는 AppConfig의 자식 타입이므로, AppConfig 타입으로 조회 가능

### 👉 결론
* `@Bean`만 사용해도 스프링 빈으로 등록되지만, 싱글톤을 보장하지 않음
* `@Configuration`을 사용하자