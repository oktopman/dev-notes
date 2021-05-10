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

## 좋은 객체 지향 설계의 5가지 원칙의 적용
여기서 3가지 SRP, DIP, OCP 를 적용 했다.  

### SRP 단일 책임 원칙
한 클래스는 하나의 책임만 가져야 한다.  
기존의 클라이언트 객체(MemberServiceImpl)는 직접 구현 객체를 생성하고 연결하고 실행하는 다양한 책임을 가지고 있었다.  
SRP 단일 책임 원칙을 따르며 관심사를 분리했다.  
구현 객체를 생성하고 연결하는 책임은 AppConfig가 담당하고 클라이언트객체는 실행하는 책임만 담당하게 변경되었다.  

### DIP 의존관계 역전 원칙
추상화에 의존해야지, 구체화에 의존하면 안된다. 의존성 주입은 이 원칙을 따르는 방법중 하나이다.  
기존에는 `MemberRepository` 추상화 인터페이스에만 의존하는것처럼 보였지만 `MemoryMemberRepository` 구체화 구현 클래스에도 함께 의존했다.  
클라이언트코드가 `MemberRepository` 추상 인터페이스에만 의존하게 코드를 변경하고 AppConfig를 통해 `MemoryMemberRepository` 를 주입해서 DIP 원칙을 따르게 변경하였다.  

### OCP
소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.  
`AppConfig` 를 통해 의존관계를 주입해주었기 때문에 서비스를 실행하는 로직인 클라이언트 코드가 변경하지 않고도 소프트웨어를 확장할 수 있고, 사용 영역의 변경은 닫혀있게 변경하였다.  

## IoC, DI, 컨테이너
### 제어의 역전 IoC(Inversion of Control)
V1 의 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성, 연결, 실행 했다. 한마디로 구현객체가 프로그램의 제어흐름을 스스로 조종했다.  
반면에 AppConfig가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당하며 프로그램의 제어 흐름은 `AppConfig` 가 가져간다.  
프로그램에 대한 제어 흐름 권한은 모두 `AppConfig`가 가지고 있다. `MemberServiceImpl`은 자신의 로직을 실행할 뿐이다.  
이렇듯 프로그램의 제어흐름을 직접 제어하는게 아니라 외부에서 관리하는것을 제어의 역전이라고 한다.  

### 프레임워크 vs 라이브러리
프레임워크가 내가 작성한 코드를 제어하고, 대신 실행한다면 프레임워크이다.  
JUnit 을 예로 들어 @BeforeEach, @Test 와 같은 어노테이션을 사용한다면 @Test 가 달린 메서드 이전에 @BeforeEarch 메소드를 먼저 실행하게 된다. 이것은 개발자의 코드를 통한 흐름제어가 아닌 프레임워크의 제어 이다.  

반면에 내가 작성한 코드가 직접 제어의 흐름을 담당한다면 프레임워크가 아니라 라이브러리이다.  

### IoC 컨테이너, DI 컨테이너
`AppConfig` 처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는것을 `IoC 컨테이너` 또는 `DI 컨테이너` 라 한다.

## Spring 을 통한 관심사 분리. [회원관리 서비스 구현 V3]

# 출처
https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8