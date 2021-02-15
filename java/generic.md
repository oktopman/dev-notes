# Generic

- [Generic이란](#Generic이란)
- [Generic의 사용 이유](#Generic의-사용-이유)

## Generic이란
제네릭은 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법을 의미한다.
매개변수는 변수에 들어갈 값과 관련되어 있는데, generic은 그 변수의 데이터타입과 관련되어 있다.  
```java
@Data
public class Person<T> {

    public T info;

    public static void main(String[] args) {
        Person<String> p1 = new Person<>();
        p1.setInfo("a");

        Person<Integer> p2 = new Person<>();
        p2.setInfo(1);

        System.out.println(p1.getInfo());
        System.out.println(p2.getInfo());
    }
}
```
```
a
1
```
`T` 는 info 의 데이터 타입이다.  Person 클래스를 정의하는 시점에서는 info의 데이터타입을 명시적으로 지정하지 않고 있다가, 인스턴스화 할때 데이터타입을 지정 할 수 있다.  

## Generic의 사용 이유
```java
class Student {
    public int grade;
    public Student(int grade) { this.grade = grade; }
}
class Employee {
    public String rank;
    public Employee(String rank) { this.rank = rank; }
}
class Person {
    public Object data;
    public Person(Object data) { this.data = data; }
}

public class GenericPractice {
    public static void main(String[] args) {
        Person person = new Person("1등급");
        Student student = (Student) person.data;
        System.out.println(student.grade);
    }
}
```
위의 코드는 성공적으로 컴파일된다. 하지만 실행을 하면 아래와 같은 오류가 발생한다.  
```
Exception in thread "main" java.lang.ClassCastException: java.lang.String cannot be cast to me.oktop.java8inaction.generic.Student
	at me.oktop.java8inaction.generic.GenericPractice.main(GenericPractice.java:19)

```
Person 생성자의 매개변수 데이터타입이 Object 이기 때문에 컴파일 에러가 발생하지 않는다. 하지만 실행시 런타임 에러가 발생한다.  
이러한 것을 타입이 안전하지않다, `Type Safe 하지 않다` 라고 한다.  
컴파일언어를 사용하는데 있어서 중요한 장점은 컴파일러가 실제로 우리가 작성한코드가 실행되기전에 사용자의 실수나 착오를 미리 검출해준다는 것이다.  
우리가 유발시킬수 있는 에러는 컴파일타임에서 검출될수 있도록 코딩을 해야 컴파일언어가 제공하는 컴파일이라는 불편한과정의 혜택을 얻을 수 있다.  

이 코드를 제네릭으로 바꿔보자.
```java
class Student {
    public int grade;
    public Student(int grade) { this.grade = grade; }
}
class Employee {
    public String rank;
    public Employee(String rank) { this.rank = rank; }
}
class Person<T> {
    public T data;
    public Person(T data) { this.data = data; }
}

public class GenericPractice {
    public static void main(String[] args) {
        Person<Employee> person = new Person<>(new Employee("aa"));
        Employee employee = person.data;
        System.out.println(employee.rank); // 성공

        Person<Student> person2 = new Person<>(new Student(1));
        Student data = person2.data;
        System.out.println(data.grade);
    }
}
```
인스턴스화 하는 과정에서 타입을 지정해야 하기 때문에 컴파일 단계에서 오류가 검출되고 타입 안전성을 동시에 추구할 수 있게 되었다.  

## 참고
생활코딩 : https://www.opentutorials.org/course/1223/6237  