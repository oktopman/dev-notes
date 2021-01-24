# JAVA8
- [Functional Interface](#Functional-Interface)
- [Java가 기본으로 제공하는 함수형 인터페이스](#Java가-기본으로-제공하는-함수형-인터페이스)
- [람다 표현식](#람다-표현식)

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