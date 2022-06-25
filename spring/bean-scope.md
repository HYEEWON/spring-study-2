# 빈 스코프

<br>

## 목차
* [빈 스코프 정의](#빈-스코프-정의)
* [싱글톤 빈 스코프](#싱글톤-빈-스코프)
* [프로토타입 빈 스코프](#프로토타입-빈-스코프)
* [웹 빈 스코프](#웹-빈-스코프)
* [싱글톤과 프로토타입 코드 비교](#싱글톤과-프로토타입-코드-비교)
* [싱글톤과 프로토타입을 함께 사용할 경우의 문제점](#싱글톤과-프로토타입을-함께-사용할-경우의-문제점)
* [싱글톤과 프로토타입을 함께 사용하는 방법](#싱글톤과-프로토타입을-함께-사용하는-방법)
  * [방법 1: 스프링 컨테이너에 요청](#방법-1-스프링-컨테이너에-요청)
  * [방법 2: ObjectFactory와 ObjectProvider](#방법-2-objectfactory와-objectprovider)
  * [방법 3: JSR-330 Provider](#방법-3-jsr-330-provider)
* [Request 빈 스코프](#request-빈-스코프)
* [스코프와 프록시](#스코프와-프록시)

<br>

## 빈 스코프 정의

> 빈이 존재할 수 있는 범위 <br>
> 스코프는 필요한 곳에서 최소화하여 사용해야 함 <br>

* 싱글톤: `기본` 스코프, 스프링 컨테이너의 시작부터 종료까지 유지되는 가장 넓은 범위의 스코프
* 프로토타입: 스프링 컨테이너는 프로토타입 빈의 `생성`과 `의존관계 주입`까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프
  * `많이 사용하지는 않음`
* 웹 관련 스코프
  * request: 웹 요청이 들어오고 나갈때 까지 유지되는 스코프
  * session: 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프
  * application: 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프
  * websocket: 웹 소켓과 동일한 생명주기를 가지는 스코프

```java
@Scope("prototype")
```

<br>

## 싱글톤 빈 스코프
* 싱글톤 스코프 빈을 조회하면 스프링 컨테이너는 `항상 같은 인스턴스`의 스프링 빈을 반환
* 컨테이너에 있던 빈을 반환

<br>

## 프로토타입 빈 스코프
* 프로토타입 스코프 빈을 조회하면 스프링 컨테이너는 `항상 새로운 인스턴스를 생성`해서 반환
* 빈 요청이 들어오면 빈을 생성, 의존관계 주입, 초기화 후 클라이언트에 반환
* 반환 이후에는 컨테이너가 빈을 관리하지 않음: `클라리언트가 빈을 관리`
  * `@PreDestroy` 메서드가 실행되지 않음

<br>

## 웹 빈 스코프
* 웹 환경에서만 동작
* 스프링 컨테이너가 생성 ~ 종료까지 관리
* request, session, application, websocket 등이 있음

<br>

## 싱글톤과 프로토타입 코드 비교

### 싱글톤 테스트
* 컨테이너 생성 시점에 `@PostConstruct` 메서드 1번 수행
* 2개의 빈을 조회하면 같은 빈이 조회됨
* 싱글톤 빈은 컨테이너가 관리해 종료 시, `@PreDestroy` 메서드가 실행됨

```java
public class SingletonTest {
    @Test
    public void singletonBeanFind() {
        // 자동으로 Component Scan 진행
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class);

        SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
        SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);

        assertThat(singletonBean1).isSameAs(singletonBean2); // 같은 객체
        
        ac.close(); //종료
    }

    @Scope("singleton")
    static class SingletonBean {
        @PostConstruct
        public void init() {
            System.out.println("SingletonBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("SingletonBean.destroy");
        }
    }
}
```

```
SingletonBean.init
singletonBean1 = hello.core.scope.PrototypeTest$SingletonBean@xxx
singletonBean2 = hello.core.scope.PrototypeTest$SingletonBean@xxx
org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing
SingletonBean.destroy
```

### 프로토타입 테스트
* 빈이 요청되면 `@PostConstruct` 메서드가 각각 실행되어 총 2번 수행
* 2개의 빈을 조회하면 다른 빈이 조회됨
* 초기화 후 컨테이너가 빈을 관리하지 않아 종료 시, `@PreDestroy` 메서드가 실행되지 않음
  * 종료 메서드는 클라이언트가 호출해야 함

```java
public class PrototypeTest {
    @Test
    public void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
        
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        
        assertThat(prototypeBean1).isNotSameAs(prototypeBean2);

        ac.close(); //종료 // 호출되지 않음

        // 클라이언트가 직접 호출
        prototypeBean1.destroy();
        prototypeBean2.destroy();
    }

    @Scope("prototype")
    static class PrototypeBean {
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

```
PrototypeBean.init
PrototypeBean.init
prototypeBean1 = hello.core.scope.PrototypeTest$PrototypeBean@xxx
prototypeBean2 = hello.core.scope.PrototypeTest$PrototypeBean@xxx
org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing
```

<br>

## 싱글톤과 프로토타입을 함께 사용할 경우의 문제점

> 싱글톤 빈이 프로토타입 빈을 주입받아 사용될 경우와 순수 프로토타입 빈으로 사용하는 경우와 결과가 달라질 수 있음<br>

* 클라이언트A -> 프로토타입 빈 호출 -> 함수 실행
* 클라이언트B -> 프로토타입 빈 호출 -> 같은 함수 실행

* -> 싱글톤 빈의 생성 시점에 의존관계 주입이 일어난 프로토타입 빈이 사용되어 빈이 새로 생성되지 않기 때문임

<br>

## 싱글톤과 프로토타입을 함께 사용하는 방법

### 방법 1: 스프링 컨테이너에 요청

* 싱글톤 빈에서 필요한 프로토타입 빈을 항상 생성
* `Dependency Lookup (DL)`: 직접 필요한 의존관계를 찾는 것
  * 스프링 컨테이너에 종속적, 단위 테스트가 어려워짐

```java
@Scope("singleton")
static class ClientBean {
    @Autowired
    private ApplicationContext ac;

    public void logic() {
        PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
    }
}
```

### 방법 2: ObjectFactory와 ObjectProvider

> 객체 조회를 꼭 필요한 시점까지 지연 <br>

* 필요한 빈을 컨테이너에서 찾아주는 `DL` 기능을 제공하는 것
* `ObjectFactory`: 기능 단순, 별도 라이브러리 필요 없음, 스프링에 의존
* `ObjectProvider`: `ObjectFactory`상속, 스트림 처리 등 편의 기능 있음, 별도 라이브러리 필요 없음, 스프링에 의존
  * `ObjectProvider`는 `ObjectFactory`에 기능이 추가된 것

```java
@Scope("singleton")
static class ClientBean {
    @Autowired
    // private ObjectFactory<PrototypeBean> prototypeBeanProvider;
    private ObjectProvider<PrototypeBean> prototypeBeanProvider;

    public void logic() {
        // 항상 새로운 프로토타입 빈을 생성
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
    }
}
```

### 방법 3: JSR-330 Provider
* `javax.inject.Provider` 라는 `JSR-330` 자바 표준 사용
* `build.gradle`에 `javax.inject:javax.inject:1` 라이브러리 추가 필요

```
dependencies {
	implementation 'javax.inject:javax.inject:1'
}
```

```java
@Scope("singleton")
static class ClientBean {
    @Autowired
    private Provider<PrototypeBean> provider;

    public void logic() {
        PrototypeBean prototypeBean = provider.get();
    }
}
```

<br>

## Request 빈 스코프
* 동시에 여러 HTTP 요청이 발생해 로그를 남길 때 사용하면 좋음
* `build.gradle`에 `org.springframework.boot:spring-boot-starter-web` 라이브러리 추가 필요
  * 스프링 부트가 내장 톰켓을 사용해 윕 서버와 스프링 실행

```
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

* 같은 요청에서는 `Controller`, `Service` 상관없이 같은 빈이 반환됨

```java
@Component
@Scope(value = "request") // @Scope("request")와 같음
public class RequestBean { }
```

```java
@Controller
@RequiredArgsConstructor // 생성자가 생성되어 자동으로 의존관계 주입됨
public class RequestController {
    private final RequestService requestService;

    // request 스코프로 스프링을 처음 시작하면 HTTP 요청이 없어 에러가 남
    //private final RequestBean requestBean;

    // 컨테이너에 빈을 요청하는 시점을 지연할 수 있음
    private final ObjectProvider<RequestBean> requestBeanProvider;

    @RequestMapping("url-name")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        RequestBean requestBean = requestBeanProvider.getObject();

        return "OK";
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class RequestService {
    private final ObjectProvider<RequestBean> requestBeanProvider;

    public void logic(String id) {
        RequestBean requestBean = requestBeanProvider.getObject();
    }
}
```

<br>

## 스코프와 프록시

> 객체 조회를 꼭 필요한 시점까지 지연 <br>

* Request 스코프 빈을 `ObjectProvider`를 사용하지 않고 사용할 수 있음
* `@Scope`에서 `proxyMode`를 지정
  * 클래스: ScopedProxyMode.TARGET_CLASS
  * 인터페이스: ScopedProxyMode.INTERFACES
* `가짜 프록시 클래스`를 만들고 HTTP Request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입
  * `CGLIB` 라이브러리로 클래스를 상속 받은 가짜 `프록시 객체`를 만들어서 `주입`
  * `ac.getBean("requestBean", RequestBean.class)` 로 조회해도 프록시 객체가 조회됨
* `가짜 프록시 객체`는 요청이 올때, 내부에서 `진짜 빈을 요청`하는 위임 로직이 있음
  * 클라이언트가 메서드를 호출하면 프록시 객체의 메서드를 호출한 것이지만, 프록시 객체가 진짜 빈의 메서드를 호출함 (`다형성`)

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestBean { }
```

```java
@Controller
@RequiredArgsConstructor // 생성자가 생성되어 자동으로 의존관계 주입됨
public class RequestController {
    private final RequestService requestService;
    private final RequestBean requestBean;

    @RequestMapping("url-name")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        return "OK";
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class RequestService {
    private final RequestBean requestBean;

    public void logic(String id) { }
}
```
