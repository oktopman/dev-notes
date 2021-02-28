# Generic

- [Generic이란](#Generic이란)
- [Generic의 사용 이유](#Generic의-사용-이유)
- [Generic의 특징](#Generic의-특징)
- [Generic의 제한](#Generic의-제한)
- [GenericRepository 를 통한 이해](#GenericRepository-를-통한-이해)
- [Erasure(소거)](#Erasure(소거))

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

## Generic의 특징
클래스 내에서 여러개의 제네릭을 필요로 할 때의 예제이다.  
```java
class Employee {
    public int rank;
    Employee(int rank) { this.rank = rank; }
}
class Person<T, S> {
    public T info;
    public s id;
    Person(T info, S id) {
        this.info = info;
        this.id = id;
    }
}

public class GenericPractice {
    public static void main(String[] args) {
        Person<EmployeeInfo, Integer> p1 = new Person<>(new Employee(1), 1);
    }
}
```
<T, S> 와 같은 형식으로 복수의 제네릭을 사용한다.  
T, S 대신에 어떠한 문자를 사용해도 되지만 묵시적인 약속이 있기는 하다.  
제네릭의 타입은 참조형 타입만 올 수 있다. 기본 데이터타입은 올 수 없기때문에 wrapper 클래스를 사용할 수 있다.  

제네릭은 클래스레벨 뿐만아니라 메소드 레벨에서도 사용할 수 있다.  
```java
class Family {
    public String name;

    public Family(String name) {
        this.name = name;
    }
}
class MethodGeneric {

    public int value;
    public int data;

    public <U> void printValue(U value) {
        System.out.println(value);
    }

    public static void main(String[] args) {
        Family family = new Family("family");
        MethodGeneric methodGeneric = new MethodGeneric();
        methodGeneric.<Family>printValue(family);
    }

}
```
printValue 의 U 는 Family 타입이 된다.  value 에 어떠한 값이 들어왔는지에 따라 데이터타입을 추정할 수 있기때문에 생략이 가능하다.
```
methodGeneric.printValue(family);
```

## Generic의 제한
데이터타입을 인스턴스화 할때 제네릭으로 지정할 수 있기때문에 잡다한 것들이 다 들어올 수 있기때문에 `extends` 키워드를 이용하여 그것을 제한 할 수 있다.  
```java
class Info {
    public int value;
}
class GenericExtends extends Info {
    public int rank;

    public GenericExtends(int rank) {
        this.rank = rank;
    }
}

class PersonExtends<T extends Info> {
    public T info;

    public PersonExtends(T info) {
        this.info = info;
    }
}
class GenericExtendsPractice {
    public static void main(String[] args) {
        PersonExtends<Info> personExtends1 = new PersonExtends(new Info());
        PersonExtends<GenericExtends> personExtends2 = new PersonExtends(new GenericExtends(1));
        PersonExtends<String> personExtends3 = new PersonExtends<String>("aa"); // 컴파일에러 
    }
}
```
위의 코드처럼 T 로 올수 있는 데이터타입을 Info 클래스이거나 Info의 자식들만 올 수 있게 `extends` 키워드를 이용해 제한 할 수 있다.  
Info 를 `Class`가 아닌 `interface`로 사용해도 된다.  

## GenericRepository 를 통한 이해
GenericRepository 을 만드는 예제를 만들어 Generic에 대한 이해를 높여보자.  
과일을 관리하는 CURD 메소드를 만들어본다.
다음은 제네릭이 아닌 AppleRepository 이다.  
```java
@Getter
@Setter
@ToString
public class Apple {
    private Long id;
    private String color;
}

public class AppleRepository {
    private Map<Long, Apple> map = new HashMap<>();
    // create, update
    public void save(Apple apple) {
        map.put(apple.getId(), apple);
    }
    // read
    public List<Apple> findAll() {
        return new ArrayList<>(map.values());
    }
    // delete
    public void delete(long id) {
        map.remove(id);
    }
}

public class FruitTest {

    public static void main(String[] args) {
        // AppleRepository 를 통한 CRUD
        Apple apple = new Apple();
        apple.setId(1L);
        apple.setColor("red");
        AppleRepository appleRepository = new AppleRepository();
        appleRepository.save(apple);

        Apple updateApple = new Apple();
        updateApple.setId(1L);
        updateApple.setColor("red2");
        appleRepository.save(updateApple);
        List<Apple> apples = appleRepository.findAll();
        System.out.println("apples size : " + apples.size());
        apples.forEach(System.out::println);

        appleRepository.delete(1L);
        apples = appleRepository.findAll();
        System.out.println("apples size : " + apples.size());
    }
}
```
```
apples size : 1
Fruit{id=1, color='red2'}
apples size : 0
```

위와 같이 사과를 저장하는 메소드를 만들게 된다면, 바나나, 포도, 오렌지와 같은 메소드를 만들때 마다 각자의 Repository 가 필요하다.  
제네릭을 통해 중복된 코드를 제거 하여 과일들을 저장할 수 있는 공통클래스인 GenericRepository 를 만들어보자.   
Spring Data Jpa 에 있는 `JpaRepository<T, ID>` 와 같은 클래스를 만들어보겠다.  
```java
@Getter
@Setter
public class Fruit<T> {
    private T id;
    private String color;
}
public class Apple extends Fruit<Long> {

}

public class Banana extends Fruit<Long> {
}
```
`Fruit` 클래스를 제네릭하여 만들고 `Apple`, `Banana` 같은 과일 클래스들은 `Fruit` 를 상속받게 지정한다.  

```java
public class GenericRepository<E extends Fruit<K>, K> {

    private Map<K, E> map = new HashMap<>();

    // create, update
    public void save(E fruit) {
        map.put(fruit.getId(), fruit);
    }

    // read
    public List<E> findAll() {
        return new ArrayList<>(map.values());
    }

    // delete
    public void delete(K id) {
        map.remove(id);
    }
}
```
`Entity` 는 Fruit 를 상속받은 클래스만 올 수 있고, `Key`는 `Entity` 의 id 타입만 받게 지정 한다.
  
```java
public class BananaRepository extends GenericRepository<Banana, Long> {
}
```
Spring Data Jpa 처럼 과일을 저장하는 BananaRepository 를 생성하고 GenericRepository 를 상속받는다.  
이렇게 되면 GenericRepository에 구현해놓은 CRUD 메소드를 사용할 수 있게되고,   
`GenericRepository<E extends Fruit<K>, K>` 에 E 로는 Fruit 를 상속받은 Banana 로 지정하고, K 는 Long 을 지정한다.  
`new GenericRepository<Banana, Long>` 을 통해 생성하여 사용할 수도 있지만, BananaRepository 만이 가지는 메소드가 더 구현될수도 있기 때문에,  
클래스로 생성하여 사용하는것이 좋다고 생각한다.  

```java
public class FruitTest {

    public static void main(String[] args) {
        // GenericRepository 를 통한 CRUD
        GenericRepository<Banana, Long> genericRepository = new BananaRepository();
        Banana genericBanana = new Banana();
        genericRepository.save(genericBanana);
        List<Banana> genericBananas = genericRepository.findAll();
        System.out.println("genericBananas size : " + genericBananas.size());

        // GenericRepository를 상속받은 BananaRepository 를 통한 CRUD
        BananaRepository bananaRepository = new BananaRepository();
        Banana banana = new Banana();
        banana.setId(1L);
        banana.setColor("yellow");
        bananaRepository.save(banana);

        List<Banana> bananas = bananaRepository.findAll();
        System.out.println("bananas size: " + bananas.size());
        bananaRepository.delete(1L);
        bananas = bananaRepository.findAll();
        System.out.println("bananas size: " + bananas.size());        
    }
}
```
```
genericBananas size : 1
bananas size: 1
bananas size: 0
```
이러한 형태로 Generic을 사용하여 중복적으로 만들게 되는 클래스, 메소드 등을 제거할 수 있다.  

메소드를 제네릭하게 만들어 공통적으로 사용할 수도 있다.  
```java
public class RestTemplateTest {
    public static void main(String[] args){
      Apple apple = new Apple();
      RestTemplateTest restTemplateTest = new RestTemplateTest();
      restTemplateTest.send(1L, apple, Apple.class);
    }

    public <E extends Fruit<K>, T, K> T send(K key, E dto, Class<T> classes) {
        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<T> responseEntity = restTemplate.exchange("url", HttpMethod.GET, null, classes, dto);
        return responseEntity.getBody();
    }
}
```
RestTemplate 을 이용하여 API 를 호출하는 메소드를 제네릭하게 생성하였다.  
전송하는 body 와 리턴받는 타입을 제네릭한 파라미터로 받게 하였다.  
dto 는 Fruit 를 상속받지 않은 클래스는 Upper Bounded Wildcard를 이용하여 사용할 수 없게 하였다.  

## Erasure(소거)
Erasure(소거)란 원소타입을 컴파일 타임에만 검사하고 런타임에는 해당 타입 정보를 알 수 없는 것이다.  
컴파일러는 제네릭 타입을 이용해 소스파일을 체크하고, 필요한 곳에 형변환을 넣어준다.  
그리고 제네릭 타입을 제거한다.  
즉 컴파일된 파일에는 제네릭 타입에 대한 정보가 없다.  
소거 과정은 제네릭 타입이 `<T extends Fruit>` 라면 T 는 Fruit로 치환 되고 <T> 인 경우는 Object로 치환 된다.    
- AS-IS
```
class Box<T extends Fruit>{
	void add(T t){
		...
	}
}
```
- TO-BE
```
class Box{
	void add(Fruit t){
		...
	}
}
```
와일드 카드가 포함되어 있는 경우 다음과 같이 적절한 타입으로 형변환이 추가 된다.  
- AS-IS  
```
static Juice makeJuice(FruitBox<? extends Fruit> box){
	String tmp = "";
	for(Fruit f : box.getList()) temp += f + " ";
	return new Juice(temp);
}
```
- TO-BE
```
static Juice makeJuice(FruitBox box){
	String tmp = "";
	Iterator it = box.getList().iterator();
	while(it.hasNext()){
		tmp += (Fruit)it.next() + " " ;
	}
	return new Juice(temp);
}
```
컴파일된 파일에는 제네릭 타입에 대한 정보가 없다고 했지만 파라미터타입 레퍼런스는 런타임에서 클래스의 타입을 알아낼 수 있다.  
자세한 내용은 토비의 봄 슈퍼타입토큰을 참고하자.  
https://www.youtube.com/watch?v=01sdXvZSjcI&t=30s  
## 참고
생활코딩 : https://www.opentutorials.org/course/1223/6237  