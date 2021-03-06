# JAVA8
- [Functional Interface](#Functional-Interface)
- [Java가 기본으로 제공하는 함수형 인터페이스](#Java가-기본으로-제공하는-함수형-인터페이스)
- [람다 표현식](#람다-표현식)
- [메소드 레퍼런스](#메소드-레퍼런스)
- [인터페이스 기본 메소드와 스태틱 메소드](#인터페이스-기본-메소드와-스태틱-메소드)
- [스트림 API](#스트림-API)
- [Optional](#Optional)
- [Date 와 Time](#Date-와-Time)

## Functional Interface
인터페이스에 `추상메서드`가 하나만 있다면 함수형 메서드 !  
interface 에선 abstact 를 생략 가능
```java
@FunctionalInterface
public interface RunSimething {}
    int doIt(int i); // abstact 가 생략 되어있음
}
```

같은 input 값을 넣었을 때 결과값이 같아야 한다.
예를 들어서
```java
public class Practice {

    public static void main(String[] args) {
        RunSomething something = (num) -> num + 5;

        System.out.println(something.doIt(5)); // 10
        System.out.println(something.doIt(5)); // 10
        System.out.println(something.doIt(5)); // 10
        System.out.println(something.doIt(5)); // 10
    }
}
```

이것을 보장해주지 못하는 상황이 발생하거나 그럴 여지가 있다면 퓨어한 함수형프로그래밍이라고 볼 수 없다.  
함수밖에 있는값을 참조하거나 변경하려 하면 안된다. 전달받은 값이나 내부에 있는 값만 사용해야 한다.  

## Java가 기본으로 제공하는 함수형 인터페이스
java.util.function 패키지  
자바에서 미리 정의해둔 자주 사용할만한 함수 인터페이스  
Function<T, R>  
BiFunction<T, U, R>  
Consumer<T>  
Supplier<T>
Predicate<T>
UnaryOperator<T>
BinaryOperator<T>

제공하는 인터페이스들은 https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html  에서 확인 할 수 있다.  

## 람다 표현식
```java
public class Practice {

    public static void main(String[] args) {
        BiFunction<Integer, String, String> biFunction = (num, s) -> num + s;
        System.out.println(biFunction.apply(10, "이다"));
    }

}
```
화살표를 기준으로 왼쪽에 있는 것은 input, 오른쪽은 body 이다.  
`(num, s)` 에서 타입을 적어주지 않아도 컴파일러가 추론을 할 수 있기때문에 생략 가능하다.  

```java
public class Practice {

    public static void main(String[] args) {
        BiFunction<Integer, String, String> biFunction = (num, s) -> num + s;
        System.out.println(biFunction.apply(10, "이다"));
        Practice practice = new Practice();
        practice.run();
    }

    private void run() {
        final int number = 10; // effective final 변수. final 생략가능

        class LocalClass {
            void print() {
                int number = 11;
                System.out.println(number); // 11
            }
        }

        IntConsumer intConsumer = new IntConsumer() {
            int number = 13;
            @Override
            public void accept(int value) {
                System.out.println(number); // 13
            }
        };

        IntConsumer print = (number) -> { // scope가 같기 때문에 변수명이 같을 수 없음.
            System.out.println(number); 
        };

        print.accept(10);
    }
}
```
Java8에서 final이 붙지 않은 변수의 값이 변경되지 않는다면, 그 변수를 Effectively final이라고 한다.  
로컬클래스나 익명클래스는 쉐도잉이 가능 하다. 람다는 scope 가 같다 즉 메소드안에 있는 변수와 같은변수명을 쓸 수 없다. 그래서 람다는 쉐도잉이 일어나지않는다.

## 메소드 레퍼런스
람다가 하는 일이 기존 메소드 또는 생성자를 호출하는 거라면, 메소드 레퍼런스를 사용해서 매우 간결하게 표현할 수 있다.  
메소드 참조하는 방법
- 스태틱메소드 참조 : `타입::스태틱 메소드`
- 특정 객체의 인스턴스 `객체 레퍼런스::인스턴스 메소드`
- 임의 객체의 인스턴스 `타입::인스턴스 메소드`
- 생성자 참조 : `타입::new`

## 인터페이스 기본 메소드와 스태틱 메소드
자바8에서 추가된 기능이며 해당 인터페이스를 구현받는 클래스라면 기본적으로 사용 할 수 있다.  
하지만 모든 기능이 내가 생각한대로 동작하는것이 아니기때문에 만약 사용하게 된다면, default method를 재정의 하여 사용하는것을 추천한다.  
Object에서 제공하는 equals, hashcode, toString 같은 것들은 재정의 할 수 없다.  
인터페이스를 상속받는 인터페이스에서 다시 추상메소드로 변경 할 수 있다.  
특정 클래스에서 2개의 인터페이스를 구현하려고 할때, 이 2개의 인터페이스에 같은 default 메소드가 있다면 자바는 둘중에 어떤걸 써야 할 지 모르기때문에 컴파일 에러가 발생한다. 그렇기 때문에 클래스에서 재정의를 해야 한다.  

```java
public interface Father {

    default void say(){
        System.out.println("말하다");
    }
}

public interface Mother {

    default void say(){
        System.out.println("말하다");
    }
}

public class Son implements Father, Mother {

    @Override
    public void say() {

    }
}
```
해당 인터페이스를 구현한 인스턴스에게 static 메소드를 통해 helper method를 제공 할 수 있다.  
자바8에 추가된 기본 메소드로 인한 API의 변화들도 많다.  
### Iterable의 기본메소드
- forEach()
- spliterator()

### Collection의 기본 메소드
- stream() / parallelStream()
- removeIf(Predicate
- spliterator()

### Comparator의 기본 메소드 및 스태틱 메소드
- reversed()
- thenComparing()
- static reverseOrder() / naturalOrder()
- static nullsFirst() / nullsLast()
- static comparing()

## 스트림 API
스트림은 collection같은 연속된 데이터를 처리하는 operation의 모음이다. stream은 collection 같은 데이터를 소스로 사용하여 처리하는 API 이다. 그렇기 때문에 스트림이 데이터를 담는 저장소라고 착각하면 안된다.  
기본적으로 Lazy 하다는 특징이 있다.
중계형 operator와 종료형 operator 가 있는데 종료형 operator가 실행되기전까지는 단지 선언일 뿐이다.  
중계형 operator
- Stream을 리턴
- filter, map, limit. sorted, ...

종료형 operator
- Stream을 리턴하지 않는다
- collect, allMatch, count, forEach, min, max, ...
```java
public class App {

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("hayun");
        names.add("dajeong");

        names.stream()
                .map(s -> {
                    System.out.println(s);
                    return s.toUpperCase();
                }); // 선언만 되어있을 뿐 실행되지않음

    
        names.stream()
                .map(s -> {
                    System.out.println(s);
                    return s.toUpperCase();
                }).collect(Collectors.toSet()); // 종료형 operator인 collect 가 있기때문에 실행시킴.
    }
}
```
Lazy 하다는 말은 중계형 operator 와 같은것들 처럼 선언만 되어있다가 나중에 종료형 operator 가 호출됬을때 실행된다는 느낌으로 이해하면 된다.  

## Stream 예제
```java
public class StreamPractice {
    public static void main(String[] args) {
        List<OnlineClass> springClasses = new ArrayList<>();
        springClasses.add(new OnlineClass(1, "spring boot", true));
        springClasses.add(new OnlineClass(2, "spring data jpa", true));
        springClasses.add(new OnlineClass(3, "spring mvc", false));
        springClasses.add(new OnlineClass(4, "spring core", false));
        springClasses.add(new OnlineClass(5, "spring api developemtn", false));

        System.out.println("spring 으로 시작하는 수업");
        springClasses.stream()
                .filter(o -> o.getTitle().startsWith("spring"))
                .forEach(o -> System.out.println(o.getId()));

        System.out.println("close 되지않은 수업");
        springClasses.stream()
                .filter(o -> !o.isClosed())
                .forEach(o -> System.out.println(o.getId()));


        System.out.println("수업 이름만 모아서 스트림 만들기");
        springClasses.stream()
                .map(OnlineClass::getTitle)
                .forEach(System.out::println);

                List<OnlineClass> javaClasses = new ArrayList<>();
        javaClasses.add(new OnlineClass(6, "The Java, Test", true));
        javaClasses.add(new OnlineClass(7, "The Java, Code manipulation", true));
        javaClasses.add(new OnlineClass(8, "The Java, 8 to 11", false));

        List<List<OnlineClass>> hayunEvents = new ArrayList<>();
        hayunEvents.add(springClasses);
        hayunEvents.add(javaClasses);

        System.out.println("두 수업 목록에 들어있는 모든 수업 아이디 출력");
        hayunEvents.stream()
                .flatMap(Collection::stream)
                .forEach(OnlineClass::getTitle);

        System.out.println("10부터 1씩 증가하는 무제한 스트림 중에서 앞에 10개 빼고 최대 10개 까지만");
        Stream.iterate(10, i -> i + 1)
                .skip(10)
                .limit(10)
                .forEach(System.out::println);

        System.out.println("자바 수업 중에 Test가 들어있는 수업이 있는지 확인");
        boolean match = javaClasses.stream()
                .anyMatch(o -> o.getTitle().contains("Test1"));
        System.out.println(match);

        System.out.println("스프링 수업 중에 제목에 spring이 들어간 것만 모아서 List로 만들기");
        List<String> stringList = springClasses.stream()
                .filter(o -> o.getTitle().contains("spring"))
                .map(OnlineClass::getTitle)
                .collect(Collectors.toList());

        stringList.forEach(System.out::println);

        System.out.println("스프링 수업 중에 제목에 spring이 들어간 것만 모아서 ,붙이기");
        String joining = springClasses.stream()
                .filter(o -> o.getTitle().contains("spring"))
                .map(OnlineClass::getTitle)
                .collect(Collectors.joining(","));
        System.out.println(joining);
    }
}
```
`flatmap` 은 스트림으로 조회하는 데이터 또한 collection 일때 사용하는 추상메소드 이다.  
스트림 으로 Collection 인 데이터를 조회하고 그 안에 있는 Collection 데이터 또한 스트림 으로 순회할때 사용 한다. 쉽게 말하면 collection 안에 collection을 순회할때 사용한다.  
`map` 은 파라미터 타입을 다른 타입으로 리턴할때 사용하는 추상메서드이다. 사용할 때는 `FunctionalInterface` 의 Function<T,R> 에 선언된 `R apply(T t)` 를 구현한다고 생각하면 된다.  
`collect` 는 종료형 operator 이고 스트림에서 작업한 결과수집을 제공한다.  
이 메소드를 이용해 필요한 요소만 컬렉션으로 담을 수 있고, 요소들을 그룹핑하고 집계할 수도 있다.  

## Optional
자바8부터 비어있는값이 전달될수있는경우에 optional 이라는것을 사용가능 하다.  
null check를 깜빡할수있기때문에 실수하기 쉽고 null을 리턴하는것 자체가 근본적인 문제이다.  
java8 이전에는 대안이 없었기때문에 error를 던졌다. 하지만 진짜로 필요한경우에만 에러를 던져야지 로직을 처리할때 에러를 던지는것은 좋지 않은 습관이다.
```
if (entity == null) {
    throw new IllegalArgumentException(); // 잘못된 습관
}
```
optional은 사용하는데 문법적으로 제한은 없지만, 리턴타입에만 사용하는것이 권장사항이다.  

## Date 와 Time
자바8에 새로운 날짜와 시간 API가 생긴 여러가지 이유가 있다.  
### 그전까지 사용하던 java.util.Date 클래스는 mutable 하기 때문에 thread safe 하지 않다.  
```java
public class DateClass {
    public static void main(String[] args) throws Exception {
        Date date = new Date();
        long time = date.getTime();
        System.out.println(time);
        Thread.sleep(3000);
        Date after3Seconds = new Date();
        System.out.println(after3Seconds);
        after3Seconds.setTime(time);
        System.out.println(after3Seconds);
        System.out.println(time);
    }
}
```
```
1612191039101
Mon Feb 01 23:50:42 KST 2021
Mon Feb 01 23:50:39 KST 2021
1612191039101
```
  시간을 바꿔버릴 수 있기때문에 mutable 하다. multi thread 에서 사용하기 어렵다. 멀티스레드 환경에서 만약 시간를 변경하는 코드가 있다면 mutable 하기때문에 시간이 변경될수 있기 때문이다.  

### 클래스 이름이 명확하지 않다. Date인데 시간까지 다룬다.
```
Date date = new Date();
long time = date.getTime(); 
```
date 인데 시간을 가져오고, 그 시간또한 long..? timestamp 이다.

### 버그 발생할 여지가 많다. (타입 안정성이 없고, 월이 0부터 시작한다거나..)
### 날짜 시간 처리가 복잡한 애플리케이션에서는 보통 Joda Time을 쓰곤했다.
그 외에 기존 Date 클래스의 단점들이다. 참고: https://codeblog.jonskeet.uk/2017/04/23/all-about-java-util-date/  

### java8 부터 추가된 Date 함수
```java
public class DatePractice {

    public static void main(String[] args) {
        // 기계용 API
        Instant instant = Instant.now();
        System.out.println("기계용 시간 : " + instant); // 기준시 UTC 또는 GMT 라고 부름. 둘다 같은것

        ZoneId zone = ZoneId.systemDefault();
        System.out.println("현재 zone : " + zone);
        ZonedDateTime zonedDateTime = instant.atZone(ZoneId.of("Asia/Seoul"));
        System.out.println("서울의 현재 시간(기계용) : " + zonedDateTime);

        // 사람용
        LocalDateTime now = LocalDateTime.now();
        System.out.println("LocalDateTime : " + now);

        // 특정 zone의 시간을 보고싶을때
        ZonedDateTime nowInAmerica = ZonedDateTime.now(ZoneId.of("America/Los_Angeles"));
        ZonedDateTime nowInKorea = ZonedDateTime.now(ZoneId.of("Asia/Seoul"));
        System.out.println("미국의 현재시간 : " + nowInAmerica);
        System.out.println("한국의 현재시간 : " + nowInKorea);

        LocalDate today = LocalDate.now();
        LocalDate thisYearBirthday = LocalDate.of(2021, Month.OCTOBER, 6);

        Period period = Period.between(today, thisYearBirthday);
        System.out.println("이번달부터 내 생일까지 남은 달 : " + period.getMonths());
        System.out.println("오늘로부터 내 생일까지 남은 날짜 : " + (ChronoUnit.DAYS.between(today, thisYearBirthday)));

    }
}
```
```
기계용 시간 : 2021-02-02T14:40:27.256Z
현재 zone : Asia/Seoul
서울의 현재 시간(기계용) : 2021-02-02T23:40:27.256+09:00[Asia/Seoul]
LocalDateTime : 2021-02-02T23:40:27.385
미국의 현재시간 : 2021-02-02T06:40:27.387-08:00[America/Los_Angeles]
한국의 현재시간 : 2021-02-02T23:40:27.389+09:00[Asia/Seoul]
이번달부터 내 생일까지 남은 달 : 8
오늘로부터 내 생일까지 남은 날짜 : 246
```