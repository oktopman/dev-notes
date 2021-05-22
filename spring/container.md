# 스프링 컨테이너
어노테이션을 통해 스프링 컨테이너에 빈을 등록할 수 있다.  
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

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy() {
//        return new FixDiscountPolicy();
        return new RateDiscountPolicy();
    }

}
```
```java
public class MemberApp {

   public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);

        Member member = new Member(1L, "hayun", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println(findMember.getId());
        System.out.println(findMember.getName());

    }
}

```
ApplicationContext 를 컨테이너 라고 한다.  
어플리케이션이 실행될때 빈으로 등록된 메서드나 클래스를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. 이렇게 스프링 컨테이너에 등록된 객체를 스프링 빈이라고 한다.  
```applicationContext.getBean("memberService", MemberService.class);``` 을 통해 컨테이너에 등록된 `memberService` 라는 빈을 가져와서 사용한다.  

컨테이너에 객체를 빈으로 등록하고, 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었는데, 이부분에 대해 어떤 장점이 있을까?

## 스프링 컨테이너 생성 과정
-  스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 하기 때문에 `Appconfig.class`구성 정보로 지정했다.  
```
new AnnotationConfigApplicationContext(AppConfig.class);
```
이때 스프링 빈 저장소(비어있는)가 생성된다.  

- AppConfig 는 @Bean 과 같은 빈으로 등록하는 어노테이션을 확인하며 리턴된 객체정보를 스프링 빈 저장소에 등록한다.  
- 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입한다.  

어노테이션을 통해 등록한 빈이 정상등록 되었는지 확인 해보자.  
```java
class ApplicationContextTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출력하기")
    void findAllBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("bean = " + beanDefinitionName + " object = " + bean);
        }
    }
}
```
```
bean = appConfig object = spring.core.AppConfig$$EnhancerBySpringCGLIB$$42a8851e@63dd899
bean = memberService object = spring.core.member.MemberServiceImpl@59d2400d
bean = memberRepository object = spring.core.member.MemoryMemberRepository@75cd8043
bean = orderService object = spring.core.Order.OrderServiceImpl@33b1c5c5
bean = discountPolicy object = spring.core.discount.RateDiscountPolicy@5b202a3a
```
의도한대로 빈이 등록된 것을 확인할 수 있다.  
`@Autowired` 와 같은 어노테이션을 통해 등록한 빈도 컨테이너에 등록되는건 동일하다.  

