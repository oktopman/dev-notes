# JAVA8
- [Functional Interface](#Functional-Interface)
- [Java가 기본으로 제공하는 함수형 인터페이스](#Java가-기본으로-제공하는-함수형-인터페이스)
- [람다 표현식](#람다-표현식)
- [메소드 레퍼런스](#메소드-레퍼런스)
- [인터페이스 기본 메소드와 스태틱 메소드](#인터페이스-기본-메소드와-스태틱-메소드)
- [스트림 API](#스트림-API)

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

