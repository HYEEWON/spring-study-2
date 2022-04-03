# 싱글톤 컨테이너

<br>

## 싱글톤 패턴 정의
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

<br>

## 싱글톤 컨테이너

> 기본적으로 스프링은 싱글톤으로 빈을 등록 (다른 방식도 가능)

