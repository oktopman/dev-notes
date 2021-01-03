# EnableAutoConfiguration

`@SpringBootApplication` 어노테이션에는 여러 어노테이션들이 포함되어있는데, 그중에 핵심은 `@ComponentScan` 과 `@EnableAutoConfiguration` 이다.

## @ComponentScan
`@ComponentScan` 어노테이션은 시작이 되는 패키지부터 하위패키지까지 스캔하며 `@Component`이 붙은 어노테이션들을 다 찾아서 빈으로 등록 시킨다.  
    
`@Component` 관련된 어노테이션은 
- `@Component`
- `@Configuration`
- `@Repository`
- `@Service`
- `@Controller`
- `@RestController`         

## @EnableAutoConfiguration
`@EnableAutoConfiguration`은 웹서버로 어플리케이션을 띄우기 위해 사용한다.  
`@EnableAutoConfiguration`은 추가된 의존성 중에 스프링의 메타파일(java/resources/)의 `spring.factories` 라는 파일이 있는데,  
`org.springframework.boot.autoconfigure.EnableAutoConfiguration` 라는 키의 값을 적용시킨다.    
흐름은 `@EnableAutoConfiguration` 을 읽을때 spring.factories 에 등록되어있는 키값을 보고 @Configuration가 달린 클래스 중에 조건에 맞는 것을 빈으로 등록 한다.    
spring.factories에 있는 수많은 키값들이 condition에 따라 빈이 등록된다.  
위와 같은 방법으로 빈을 만든 후 프로젝트의 의존성을 추가하여 내가 생성한 커스텀한 빈을 다른 프로젝트에서 사용 할 수 있다.  
 
`@EnableAutoConfiguration` 이 `@ComponentScan` 보다 뒤에 실행되기 때문에 빈의 이름이 겹친다면 `@ComponentScan` 으로 등록한 빈을 커스텀한 빈이 덮어쓰게 된다.  
빈은 항상 `@ComponentScan`이 우선시 되어야 하기 때문에 만약 빈 이름이 겹친다면 빈으로 등록할 클래스나 메소드에 `@ConditionalOnMissingBean` 을 달아서 사용 해야 한다.  
만약 해당 하는 빈이 이미 있다면, 빈으로 등록하지 않는 어노테이션이다. 이렇게 하면 @ComponentScan에서 등록한 빈을 덮어쓰지 않고 사용할 수 있다.  

EnableConfigurationProperties 를 통해서 더 간편화 하게 사용할 수 있다.  
## 참조
https://velog.io/@max9106/Spring-Boot-EnableAutoConfiguration