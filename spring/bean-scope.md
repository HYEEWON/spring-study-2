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

        assertThat(singletonBean1).isSameAs(singletonBean2);
        
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
singletonBean1 = hello.core.scope.PrototypeTest$SingletonBean@54504ecd
singletonBean2 = hello.core.scope.PrototypeTest$SingletonBean@54504ecd
org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing SingletonBean.destroy
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
prototypeBean1 = hello.core.scope.PrototypeTest$PrototypeBean@13d4992d
prototypeBean2 = hello.core.scope.PrototypeTest$PrototypeBean@302f7971
org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing
```

<br>

## 싱글톤과 프로토타입을 함께 사용할 경우의 문제점

> 싱글톤 빈이 프로토타입 빈을 주입받아 사용될 경우 <br>
> 프로토타입 빈 2개로 실행할 경우 <br>

* 클라이언트A -> 프로토타입 빈 호출 -> 함수 실행
* 클라이언트B -> 프로토타입 빈 호출 -> 같은 함수 실행

> 위 경우의 결과가 달라질 수 있음 <br>
> 싱글톤 빈의 생성 시점에 의존관계 주입이 일어난 프로토타입 빈이 사용되어 빈이 새로 생성되지 않기 때문임



