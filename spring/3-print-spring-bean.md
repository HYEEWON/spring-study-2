# 스프링 빈 출력

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

@Test
void findABean() {
    // 모든 빈 출력
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();

    for (String beanDefinitionName : beanDefinitionNames) {
        Object bean = ac.getBean(beanDefinitionName);
        System.out.println("name=" + beanDefinitionName + " object=" + bean);
    }

    // 애플리케이션 빈 출력
    for (String beanDefinitionName : beanDefinitionNames) {
        BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
        if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + " object=" + bean);
        }
    }
}
```

* AnnotationConfigApplicationContext.getBeanDefinitionNames() : 스프링에 등록된 모든 빈 이름 조회
* AnnotationConfigApplicationContext.getBean(beanDefinitionName) : 빈 이름으로 빈 객체(인스턴스)를 조회
* BeanDefinition.getRole(): 내부에서 사용하는 스프링 빈인지 구별
  * ROLE_APPLICATION : 일반적으로 사용자가 정의한 빈
  * ROLE_INFRASTRUCTURE : 스프링이 내부에서 사용하는 빈

<br>

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

@Test
void findBean() {
    // 빈 이름과 타입으로 조회
    MemberService memberService = ac.getBean("memberService", MemberService.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);

    // 빈 타입으로만 조회 
    MemberService memberService = ac.getBean(MemberService.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);

    // 구체 타입으로 조회
    // 구체 타입을 사용하는 것은 좋지 않음
    MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
    assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
}
```

* AnnotationConfigApplicationContext.getBean(beanName, beanType): 빈 이름, 타입으로 조회
* AnnotationConfigApplicationContext.getBean(beanType): 빈 타입으로 조회
  * 같은 타입이 2개 이상이면 중복 오류 발생: `NoUniqueBeanDefinitionException` -> 빈 이름 지정 필요
* 조회 대상 스프링 빈이 없을 경우 예외 발생: `NoSuchBeanDefinitionException`

<br>

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

@Test
void findAllBeanByType() {
    // 특정 타입 모두 조회하기
    Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
    for (String key : beansOfType.keySet()) {
        System.out.println("key = " + key + " value = " + beansOfType.get(key));
    }
    System.out.println("beansOfType = " + beansOfType);
}
```

* AnnotationConfigApplicationContext.getBeansOfType(beanType): 특정 타입의 모든 빈 조회