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

## ApplicationContext
스프링 컨테이너의 최상위 인터페이스는 `BeanFactory` 이고 스프링 빈을 관리하고 조회하는 역할을 담당한다.  
위의 코드에서 사용했던 getBean() 을 제공하고, 대부분의 스프링에서 제공하는 기능은 BeanFactory가 제공하는 기능이다.  

위 코드에서 `ApplicationContext` 인터페이스를 통해 관리하고있는 빈을 확인할 수 있었는데 `ApplicationContext`는 BeanFactory 의 기능을 모두 상속받았기때문에 이것이 가능한것이다.  
빈을 관리하고 조회하는 기능을 이미 BeanFactory 가 이미 제공해주는데, 둘의 차이는 뭘까? 어플리케이션을 개발할때는 빈관리,조회외에도 많은 부가기능이 필요하다.  

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver { ...}
```
ApplicationContext는 메세지소스를 활용한 국제화기능, 로컬, 개발, 환경등을 구분해서 처리, 어플리케이션 이벤트, 리소스조회 와 같은 기능들을 포함하고 있다.  
정리하자면 `ApplicationContext`는 `BeanFacotry`의 기능을 상속 받는다.  
그리고 빈 관리하는기능 + 편리한 부가기능까지 제공한다. 그래서 BeanFactory를 직업 사용할 일은 거의없고 부가기능이 포함된 ApplicationContext를 사용한다.  
BeanFactory나 ApplicationContext를 스프링 컨테이너라고 부른다.  

## 설정 형식
스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있게 설계 되어있다.  
위의 코드는 어노테이션기반 자바 코드를 통해 설정했었고 XML 을 통해서도 설정할 수 있다.  
최근에는 스프링 부트를 사용하면서 XML 기반을 잘 사용하지않지만 XML을 사용하면 컴파일 없이 빈 설정 정보를 변경할 수 있고, 설정 XML 자체를 컨텍스트 전체의 빈 설정을 일목요연하게 파악할 수 있는 문서로 생각할 수 있다는 점이 장점이다.  

하지만 자바개발자들은 자바코드에 더 익숙하기때문에 자바 문법을 통해 설정하고 오류도 쉽게 파악할 수 있기때문에 자바 코드를 통해 설정하는것을 더 선호하는것 같다.  

스프링 컨테이너는 어떻게 다양한 형식의 설정 정보를 받아드릴수있게 설계 되었을까?  
스프링 컨테이너는 `BeanDefinition` 만 알고 있다.  
`BeanDefinition` 은 빈정보에 대해 추상화 한 것이고 빈 설정 메타정보라 한다.  
`@Bean` 이나 빈으로 등록하는 어노테이션이 달린 것들 마다 각각 하나씩 메타정보가 생성되고 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다.  
ApplicationContext는 Reader 를 통해 AppConfig 를 읽고 BeanDefinition을 생성하고 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성하는것으로 이해하면 될것 같다.  
우선은 스프링이 다양한 형태의 설정 정보를 읽을때 BeanDefinition으로 추상화해서 사용하는것 정도만 이해하면 될것 같다.  