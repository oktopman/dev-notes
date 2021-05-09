# 의존성 주입
스프링의 철학을 지키면서 설계하려면 지켜야 할것이 있다.  
1. 모든 설계에 역할과 구현을 분리
2. 다형성을 활용하여 인터페이스와 구현 객체를 분리
3. OCP, DIP 와 같은 객체지향 설계 원칙을 준수

## 회원관리 서비스 구현 V1
위와 같은 철학을 지켜서 회원을 관리하는 서비스를 만들어보자.  

도메인은 member 있다.  
```java
public class Member {}
```

인터페이스를 생성하여 회원을 가입하고, 조회하는 기능을 정의한다.  
```java
public interface MemberService {
    void join(Member member);
    Member findMember(Long memberId);
}
```

member를 저장하는 인터페이스를 생성하고 member 를 저장, 조회하는 기능을 정의한다.  
```java
public interface MemberRepository {

    void save(Member member);

    Member findById(Long memberId);
}
```
MemberRepository에 정의한 기능을 구현하여 memory db 에서 member 를 가입, 조회하는 기능을 구현한 MemoryRepository 를 생성한다.  
```java
public class MemoryMemberRepository implements MemberRepository {

    private static Map<Long, Member> store = new HashMap<>();

    @Override
    public void save(Member member) {
        store.put(member.getId(), member);
    }

    @Override
    public Member findById(Long memberId) {
        return store.get(memberId);
    }
}
```

MemberService 에서 MemoryMemberRepository 를 이용하여 member 를 생성, 조회하는 서비스 로직을 구현한다.  
```java
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository = new MemoryMemberRepository();

    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

구현한 기능을 테스트 하여 기능에 문제가 없음을 확인할 수 있다.  
```java
public class MemberServiceTest {

    MemberService memberService = new MemberServiceImpl();

    @Test
    void join() {
        //given
        Member member = new Member(1L, "memberA", Grade.VIP);
        //when
        memberService.join(member);
        Member findMember = memberService.findMember(1L);
        //then
        assertThat(member).isEqualTo(findMember);
    }
}

```

## 회원관리 서비스 구현 V1 의 문제점
위의 코드들은 스프링의 철학을 지키며 설계를 하는부분에 있어 문제점이 있다.  
역할과 구현을 충실하게 분리 ->  OK  
다형성을 활용하고, 인터페이스와 구현 객체를 분리 -> OK
OCP, DIP 같은 객체지향 설계 원칙을 충실히 준수 -> `그렇게 보이지만 사실은 아니다.`

```java
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository = new MemoryMemberRepository();
}
```
`MemberServiceImmpl` 은 인터페이스에 의존하면서 DIP를 지킨것 처럼 보이지만, 구현클래스에도 의존하고 있다.  
`MemberRepository` 인터페이스와 구현체클래스인 `MemoryMemberRepository` 를 동시에 의존하기 때문에 `DIP를 위반`한다.  
그리고 OCP 는 코드를 변경하지 않고 확장할 수 있다고 했는데 만약 memory db가 아닌 rdb에 저장을 하게 요구사항이 변경된다면,  
지금 코드는 클라이언트 코드에 영향을 주게 된다. 따라서 `OCP를 위반` 한다.
그렇기 때문에 DIP를 위반하지 않도록 추상(인터페이스)에만 의존하도록 변경 해야 한다.  


```java
public class MemberServiceImpl implements MemberService {
    // private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final MemberRepository memberRepository = new RDBMemberRepository(); // DIP 위반
    private MemberRepository memberRepository; // NPE 발생
}
```
위와 같은 형태로 변경 하면 DIP 위반, NPE 가 발생하게 된다.  
이 문제를 해결하려면 누군가가 클라이언트인 `MemberServiceImpl` 에 memberRepository 의 구현 객체를 대신 생성하고 주입해야 줘야 한다.  

## 관심사 분리. [회원관리 서비스 구현 V2]
`MemberServiceImpl` 는 Member 를 가입하고 조회하는것에만 집중해야 한다. 어떤 방식으로 Member 를 가입, 조회하는 방식이 선택되더라도 똑같은 기능을 수행해야 한다.  
DB 를 선택하는 책임을 담당하는 관심사를 분리 해야 한다.  

전체 동작 방식을 구성하기 위해 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스를 만든다.  
```java
public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    private MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```
AppConfig 는 어플리케이션의 실제 동작에 필요한 구현 객체를 생성하고 생성자 주입을 통해 Impl 과 연결 해준다.  
```java
public class MemberServiceImpl implements MemberService {

    // 추상화에만 의존하고, 밖에서 생성해준 객체를 통해서 코드 실행
    private final MemberRepository memberRepository;

    // 생성자 주입을 통해 MemberRepository 를 결정
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```
`MemberServiceImpl` 는 `MemoryMemberRepository` 를 의존하지 않고 인터페이스에만 의존한다.  
`MemberServiceImpl` 는 생성자를 통해 어떤 구현 객체가 들어올지 알수 없고 `AppConfig` 에서 어떤 구현 객체를 주입할지 결정한다.  
`MemberServiceImpl` 는 의존관계에 대한 고민을 외부에 맡기고 실행에만 집중한다.  

```java
public class MemberServiceTest {

    MemberService memberService;

    @BeforeEach
    void setup() {
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
    }

    @Test
    void join() {
        //given
        Member member = new Member(1L, "memberA", Grade.VIP);
        //when
        memberService.join(member);
        Member findMember = memberService.findMember(1L);

        //then
        assertThat(member).isEqualTo(findMember);
    }
}
```

직접 구현체와 의존관계를 맺지않고 외부에서 의존관계를 주입받는 것을 DI(Dependency Injection) 이라고 한다.  
이러한 형태라면 AppConfig 에만 코드수정을 해서 memory db -> rdb 로 쉽게 변경할 수 있고, 서비스 로직은 인터페이스만 의존하고 있기 때문에 데이터를 저장하고 조회하는 모듈에 대해선 알 필요가 없다.  

## Spring 을 통한 관심사 분리. [회원관리 서비스 구현 V3]