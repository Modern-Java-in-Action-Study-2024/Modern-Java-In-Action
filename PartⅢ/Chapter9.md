# Chapter9 - 리팩터링, 테스팅, 디버깅

<hr>

## 9.1. 가독성과 유연성을 개선하는 리팩터링

<hr>

### 더 간결한 코드를 만드는 방법
- 람다 표현식은 익명 클래스보다 코드를 간결하게 만듦
- 파라미터로 넣을 메서드가 이미 구현되어 있을 경우 -> 메서드 참조가 더 간결하게 만듦
- 람다 표현식은 동작 파라미터화 형식을 지원함 -> 더 큰 유연성을 갖출 수 있음 (다양한 요구사항 변화에 대응할 수 있도록 동작을 파라미터화함)

<br>

## 9.1.1. 코드 가독성 개선

> 코드 가독성이 좋다 = 코드를 다른 사람도 쉽게 이해할 수 있다.

- 코드의 문서화를 잘 해야 한다.
- 표준 코딩 규칙을 준수해야 한다.

## 9.1.2. 익명 클래스를 람다 표현식으로 리팩터링하기

- 하나의 추상 메서드를 구현하는 익명 클래스는 람다 표현식으로 리팩터링할 수 있다.
- 익명 클래스 : 코드를 장황하게 만들며, 쉽게 에러를 일으킴<br>
  → 람다 표현식을 이용하여 간결하고 가독성 좋은 코드를 구현할 수 있음

```java
// 익명 클래스를 이용한 코드
Runnable r1 = new Runnable() {
	public void run() {
		System.out.println("해위");
	} 
};

// 람다 표현식을 이용한 코드
Runnable r2 = () -> System.out.println("해위");
```

### 유의 사항
모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다.
1. 익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 갖는다.
   (익명 클래스의 this = 자신, 람다 this = 람다를 감싸는 클래스)
2. 익명 클래스 : 감싸고 있는 클래스의 변수를 가릴 수 있지만(shadow), 람다 표현식에서는 불가능하다.
  ```java
  int a = 10;
  Runnable r1 = () -> {
	  int a = 2;    // 컴파일에러 
	  System.out.println(a);
  };
  
  Runnable r2 = new Runnable(){ 
	  public void run(){ 
		  int a = 2;     // 잘 작동한다. 
		  System.out.println(a); 
	  } 
  };
  ``` 

3. 익명 클래스를 람다 표현식으로 바꾸면, Context Overloading에 따른 모호함이 초래될 수 있다.
    - 익명 클래스 : 인스턴스화할 때 명시적으로 형식이 정해짐
    - 람다 : context에 따라 달라진다.
  ```java
  // Task라는 Runnable과 같은 시그니처를 갖는 함수형 인터페이스
  interface Task {
	  public void execute();
  }
  
  public static void doSomething(Runnable r) { r.run(); }
  public static void doSomething(Task a) { r.run(); }
  ```
  ```java
  
  // Task를 구현하는 익명 클래스
  doSomething(new Task() { 
	  public void execute() { 
		  System.out.println("Holy Moly"); 
	  }
  });
  ```
이럴 경우, 익명 클래스를 람다 표현식으로 바꾸면 메서드를 호출할 때 Runnable과 Task 모두 대상 형식이 될 수 있으므로 문제가 생긴다. 즉, 아래와 같다.
  ```java
  doSomething(() -> System.out.println("Holy Moly"));    // doSomething(Runnable)과 doSomething(Task) 중 어느 것을 가리키는지 알 수 없는 모호함이 발생
  ```
명시적 형변환(Task)를 이용해서 모호함을 제거할 수 있다.
  ```java
  doSomething((Task)() -> System.out.printin("Holy Moly"));
  ```

## 9.1.3. 람다 표현식을 메서드 참조로 리팩터링하기

람다 표현식 대신 메서드 참조를 이용하면 가독성을 높일 수 있다. 메서드명으로 코드의 의도를 명확하게 알릴 수 있기 때문이다.

```java
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = 
	menu.stream()
		.collect( 
			groupingBy(dish -> {
				if (dish.getCalories() <= 400) return CaloricLevel.DIET; 
				else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL; 
				else return CaloricLevel.FAT; 
			})
		);
```

Dish 클래스에 새로운 메서드를 추가하면 메서드 참조를 활용하여 위의 코드를 더 간단히 할 수 있다.

```java
// Dish 클래스
public class Dish{ 
	...
	
	public CaloricLevel getCaloricLevel() { 
		if (this.getCalories() <= 400) return CaloricLevel.DIET; 
		else if (this.getCalories() <= 700) return CaloricLevel.NORMAL; 
		else return CaloricLevel.FAT; 
	}
}
```

```java
// 람다 표현식을 메서드로 추출한 코드
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(groupingBy(Dish::getCaloricLevel));
```

### 정적 헬퍼 메서드

#### `comparing`

```java
// 리팩터링 전 : 비교 구현에 신경 써야 함
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 리팩터링 후 : 코드가 문제 자체를 설명한다.
inventory. sort(comparing (Apple::getWeight));
```

- sum, maximum 등 자주 사용하는 리듀싱 연산 : 내장 헬퍼 메서드 제공
    - ex) 람다 표현식 + 저수준 리듀싱 연산보다 ⇒ Collectors API 가 더 코드의 의도가 명확함
  ```java
  // 저수준 리듀싱 연산을 조합한 코드
  int totalCalories = menu.stream().map(Dish::getCalories)
								   .reduce(0, (c1, c2) -> c1 + c2);
								   
  // 내장 컬렉터 이용 : 코드 자체로 문제를 더 명확히 설명할 수 있음 (ex: summingInt)
  int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
  ```

## 9.1.4. 명령형 데이터 처리를 스트림으로 리팩터링하기

- 이론적으로는, 반복자를 이용한 기존의 모든 컬렉션 처리 코드를 스트림 API로 바꿔야 한다.<br>
  → 데이터 처리 파이프라인의 의도를 더 명확히 보여주기 때문!

- stream의 장점
    - short-circuit (불필요한 연산 생략)
    - laziness (요청할 때만 요소를 계산)
    - multicore architecture를 활용하기에 좋음


### 예시 1.

**개선 전**
두 가지 패턴 (필터링, 추출)로 엉켜있는 코드. 가독성이 떨어지며, 병렬 실행이 매우 어렵다.
```java
List<String> dishNames = new ArrayList<>(); 
for(Dish dish: menu) { 
	if(dish.getCalories() > 300) { 
		dishNames.add(dish.getName()); 
	} 
}
```

**개선 후**
스트림 API를 이용하면 문제를 더 직접적으로 기술할 수 있을 뿐 아니라 쉽게 병렬화할 수 있다.

```java
menu.parallelStream() 
	.filter(d -> d.getCalories() > 300) 
	.map(Dish::getName) 
	.collect(toList());
```

> **💡 유의!**
> - 명령형 코드의 break, continue, return 등의 제어 흐름문을 모두 분석해서 같은 기능을 수행하는 스트림 연산으로 유추해야 함 <br>
     ⇒ 명령형 코드를 스트림 API로 바꾸는 것은 쉬운 일이 아님! <br>
     ⇒ 명령형 코드를 스트림 API로 바꾸도록 도와주는 몇 가지 도구가 있음 (ex: https://goo.gl/Mal5w(http://refactoring.info/tools/LambdaFicator)) (링크 작동 안 됨)

## 9.1.5. 코드 유연성 개선

- 람다 표현식을 이용하면, 동작 파라미터화를 쉽게 구현할 수 있음<br>
  ⇒ 다양한 람다를 전달해서 다양한 동작을 표현할 수 있음<br>
  ⇒ 변화하는 요구사항에 대응할 수 있는 코드를 구현할 수 있음 (ex: Predicate로 필터링 기능 구현, 비교자로 다양한 비교 기능 구현 등)

### 함수형 인터페이스 적용

- 람다 표현식을 이용하려면 함수형 인터페이스가 필요함 → 함수형 인터페이스를 코드에 추가해야 함
- 람다 표현식 리팩터링에 자주 사용하는 2가지 패턴 : conditional deferred execution(조건부 연기 실행), execute around(실행 어라운드)

### Conditional Deferred Execution (조건부 연기 실행)

- 실제 작업을 처리하는 코드 내부에 제어 흐름문이 복잡하게 얽힌 경우 활용 (흔히 보안 검사, 로깅 관련 코드)


#### 예시 : 내장 Java Logger 클래스 사용 예제

**개선 전**
```java
if (logger.isLoggable(Long.FINER)) {
  logger.finer("Problem: " + generateDiagnostic());
}
```
- 문제점들
    - logger의 상태가 isLoggable이라는 메서드에 의해 클라이언트 코드로 노출됨
    - 메시지를 로깅할 때마다 logger 객체의 상태를 매번 확인할 필요는 없음 → 코드를 어지럽힘

**1차 개선** : 내부 메서드로 숨기기
logger 객체를 내부적으로 확인하는 log() 메서드 생성
  ```java
  logger.log(Level.FINER, "Problem: " + generateDiagnostic());
  ```
- 개선된 점 및 문제점
    - 불필요한 if 문 제거
    - logger의 상태를 노출하지 않음
    - 단, 아직 인수로 전달된 메시지 수준에서 logger가 활성화되어 있지 않더라도 항상 로깅 메시지를 평가하게 된다는 문제 존재


**2차 개선 : 람다 활용**

- 특정 조건(Level.FINER)에서만 메시지가 생성될 수 있도록 메시지 생성 과정을 연기하도록 설정하기
```java
// 2차 개선 : 위에서 만든 log() 메서드를 람다를 활용하여 개선
// log() 메서드
public void log(Level level, Supplier msgSupplier) { 
	if(logger.isLoggable(level)){ 
		log (level, msgSupplier.get());
	}
}
```
```java
// log() 메서드 호출
logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic());
```

- log() 메서드는 logger의 수준이 적절하게 설정되어 있을 때만 인수로 넘겨진 람다를 내부적으로 실행함
- 만약 클라이언트 코드에서 객체 상태를 자주 확인하거나, 객체의 일부 메서드를 호출하는 상황이라면? → 내부적으로 객체의 상태를 확인한 다음에 호출하도록 새로운 메서드를 구현하는 것이 좋다. (람다, 메서드를 파라미터로 사용!)
- 효과 : 코드 가독성 ↑, 캡슐화 ↑ (객체의 상태가 클라이언트 코드로 노출되지 않음)

### Execute Around (실행 어라운드)

- 매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드 → 람다로 준비, 종료하는 과정을 처리하는 로직 재사용 → 코드 중복 줄임

```java
String oneLine = processFile((BufferedReader b) -> b.readLine());    // 람다 전달
String twoLines = processFile((BufferedReader b) -> b.readLine() + b.readLine());    // 다른 람다 전달


public static String processFile(BufferedReaderProcessor p) throws IOException {
	try(BufferedReader br = new BufferedReader(new FileReader("ModernJavaInAction/chap9/data.txt"))) { 
		return p.process(br);    // 인수로 전달된 BufferedReaderProcessor 실행
	}
}

// IOException을 던질수 있는 람다의 함수형 인터페이스
public interface BufferedReaderProcessor { String process(BufferedReader b) throws IOException; }
```

- 함수형 BufferReaderProcessor => 람다로 BufferedReader 객체의 동작을 결정할 수 있도록 해줌



## 9.2 람다로 객체지향 디자인 패턴 리팩터링하기

<hr>

## 9.2.1 전략(strategy)

전략 패턴은 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법

다양한 기준을 갖는 입력값 검증, 다양한 파싱 방법 사용, 입력 형식 설정 등에 활용 가능

전략 패턴 구성

1. 알고리즘을 나타내는 인터페이스
2. 다양한 알고리즘을 나타내는 인터페이스 구현
3. 전략 객체를 사용하는 클라이언트

| ValidationStrategy.java

```java
public interface ValidationStrategy {
    boolean execute(String s);
}
```

| IsAllLowerCase.java

```java
public class IsAllLowerCase implements ValidationStrategy {
    public boolean execute(String s) {
        return s.matches("[a-z]+");
    }
}
```

| IsNumeric.java

```java
public class IsNumeric implements ValidationStrategy {
    public boolean execute(String s) {
        return s.matches("[0-9]+");
    }
}
```

| Validator.java

```java
public class Validator {
    private final ValidationStrategy validationStrategy;

    public Validator(ValidationStrategy validationStrategy) {
        this.validationStrategy = validationStrategy;
    }

    public boolean validate(String s) {
        return validationStrategy.execute(s);
    }
}
```

| Main.java

```java
public class Main {

    public static void main(String[] args) {
        Validator numericValidator = new Validator(new IsNumeric());
        System.out.println(numericValidator.validate("aaaa"));    // false
        Validator lowerCaseValidator = new Validator(new IsAllLowerCase());
        System.out.println(numericValidator.validate("bbbb"));    // true
    }
}
```

### 람다 표현식 사용

| Main.java

```java
public class Main {
    public static void main(String[] args) {
        Validator lowerCaseValidator = new Validator(s -> s.matches("[0-9]+"));
        System.out.println(numericValidator.validate("aaaa"));    // false
        Validator numericValidator = new Validator(s -> s.matches("[a-z]+"));
        System.out.println(numericValidator.validate("bbbb"));    // true
    }
}
```

## 9.2.2 템플릿 메서드(template method)

알고리즘의 개요를 제시한 다음 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야 할 때 사용

| OnlineBanking.java

```java
abstract class OnlineBanking {
    public void processCustomer(int id) {
        Customer customer = Database.getCustomerWithId(id);
        makeHappy(customer);
    }

    abstract void makeHappy(Customer customer);
}
```

### 람다 표현식 사용

| OnlineBanking.java

```java
abstract class OnlineBanking {
    public void processCustomer(int id, Consumer<<Customer>makeHappy) {
        Customer customer = Database.getCustomerWithId(id);
        makeHappy.accept(customer);
    }
}
```

```java
new OnlineBanking().processCustomer(1337, customer -> System.out.println("Hello " + customer.getName()));
```

## 9.2.3 옵저버(observer)

어떤 이벤트가 발생했을 때 한 객체(**주제, subject**)가 다른 객체 리스트(**옵저버, observer**)에 자동으로 알림을 보내야 하는 상황에서 사용

### 옵저버
| Observer.java

```java
interface Observer {
    void notify(String tweet);
}
```

| NewYorkTimes.java

```java
class NewYorkTimes implements Observer {
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("money")) {
            System.out.println("Breaking news in NewYork!" + tweet);
        }
    }
}
```

| Guardian.java

```java
class Guardian implements Observer {
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("queen")) {
            System.out.println("Yet more news from London... " + tweet);
        }
    }
}
```

| LeMonde.java

```java
class LeMonde implements Observer {
    public void notify(String tweet) {
        if (tweet != null && tweet.contains("wine")) {
            System.out.println("Today cheese, wine and news! " + tweet);
        }
    }
}
```

### 주제

| Subject.java

```java
interface Subject {
    void register(Observer observer);
    void notifyObservers(String tweet);
}
```

| Feed.java

```java
class Feed implements Subject {
    private static final List<Observer> OBSERVERS = new ArrayList<>();
    
    public void register(Observer observer) {
        OBSERVERS.add(observer);
    }
    
    public void notifyObservers(String tweet) {
        OBSERVERS.forEach(observer -> observer.notify(tweet));
    }
}
```

| Main.java

```java
public class Main {
    public static void main(String[] args) {
        Feed feed = new Feed();
        feed.register(new NewYorkTimes());
        feed.register(new Guardian());
        feed.register(new LeMonde());
        feed.notifyObservers("The queen said her favourite book is Modern Java in Action!");
    }
}
```

### 람다 표현식 사용하기

| Main.java

```java
public class Main {
    public static void main(String[] args) {
        Feed feed = new Feed();
        feed.register(twwet -> {
            if (tweet != null && tweet.contains("money")) {
                System.out.println("Breaking news in NewYork!" + tweet);
            }
        });
        feed.register(tweet -> {
            if (tweet != null && tweet.contains("queen")) {
                System.out.println("Yet more news from London... " + tweet);
            }
        });
    }
}
```

> 옵저버가 상태를 가지며, 여러 메소드를 정의하는 등 복잡하다면 람다 표현식보다 기존의 클래스 구현방식이 바람직

## 9.2.4 의무 체인(chain of responsibility)

작업 처리 객체의 체인(동작 체인 등)을 만들 때 사용

한 객체가 어떤 작업을 처리한 다음 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음 또 다른 객체로 전달

| Processing.java

```java
public abstract class Processing<T> {
    protected Processing<T> successor;
    
    public void setSuccessor(Processing<T> successor) {
        this.successor = successor;
    }
    
    public T handle(T input) {
        T work = handleWork(input);
        if (successor != null) {
            return successor.handle(work);
        }
        return work;
    }
    
    abstract protected T handleWork(T input);
}
```

| HeaderTextProcessing.java

```java
public class HeaderTextProcessing extends Processing<String> {
    public String handleWork(String text) {
        return "From Raoul, Mario and Alan: " + text;
    }
}
```

| SpellCheckerProcessing.java

```java
public class SpellCheckerProcessing extends Processing<String> {
    public String handleWork(String text) {
        return text.replaceAll("labda", "lambda");
    }
}
```

| Main.java

```java
public class Main {
    public static void main(String[] args) {
        Processing<String> processing1 = new HeaderTextProcessing();
        Processing<String> processing2 = new SpellCheckerProcessing();
        processing1.setSuccessor(processing2);
        System.out.println(processing1.handle("Aren't labdas really sexy?"));   // From Raoul, Mario and Alan: Aren't lambdas really sexy?
    }
}
```

### 람다 표현식 사용

```java
UnaryOperator<String> headerProcessing = text -> "From Raoul, Mario and Alan: " + text; // 첫 번째 작업 처리 객체
UnaryOperator<String> spellCheckerProcessing = text -> text.replaceAll("labda", "lambda");  // 두 번째 작업 처리 객체
Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);   // 동작 체인으로 두 함수 조합
String result = pipeline.apply("Aren't labdas really sexy?!!");
```

## 9.2.5 팩토리(factory)

인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용

| ProductFactory.java

```java
public class ProductFactory {
    public static Product createProduct(String name) {
        swtich(name) {
            case "loan":
                return new Loan();
            case "stock":
                return new Stock();
            case "bond":
                return new Bond();
            default:
                throw new RuntimeException("No such product " + name);
        }
    }
}
```

```java
Product product = ProductFactory.createProduct("loan");
```

### 람다 표현식 사용

```java
Supplier<Product> loanSupplier = Loan::new; // 생성자 참조
Loan loan = loanSupplier.get();
```

```java
final static Map<String, Supplier<Product>> map = new HashMap<>();  // 상품명을 생성자로 연결하는 Map 생성
static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
    map.put("bond", Bond::new);
}
```

```java
public static Product createProduct(String name) {  // Map 을 이용해 팩토리 디자인 패턴처럼 다양한 상품 인스턴스화
    Supplier<Product> product = map.get(name);
    if (product != null) {
        return product.get();
    }
    throw new IllegalArgumentException("No such product " + name)
}
```

생성자로 여러 인수가 필요할 때 TriFunction 등의 새로운 함수형 인터페이스를 만들어야 한다.

```java
// 세 인수가 필요할 때 아래와 같이 구현 가능
public interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
}

Map<String, TriFunction<Integer, Integer, String, Product>> map = new HashMap<>();
```

## 9.3 람다 테스팅

<hr>

프로그램이 의도대로 동작하는지 확인할 수 있는 단위 테스팅을 진행한다.

## 9.3.1 보이는 람다 표현식의 동작 테스팅
일반적인 메서드는 이름이 존재하기 때문에 단위 테스트를 문제없이 진행할 수 있지만, 람다는 익명이므로 테스트 코드 이름을 호출할 수 없다.<br>
따라서 필요하다면 람다를 필드에 저장해 테스트 할 수 있다. 람다 표현식은
함수형 인터페이스의 인스턴스를 생성한다.

````java
public class OrderProduct {
    private String name;
    private int count;
    private int price;
	....
}

public static class Order {
    public static final ToIntFunction<Order> getTotalPrice =
            (Order o) -> o.getProducts().stream().mapToInt(p -> p.getPrice() * p.getCount()).sum();

    private List<OrderProduct> products;
    ...
}

    @Test
    void test() {
        Order order = new Order(List.of(
                new OrderProduct("TV", 1, 300_000),
                new OrderProduct("공책", 3, 1_000),
                new OrderProduct("컴퓨터", 1, 1_000_000)
        ));
        int totalPrice = Order.getTotalPrice.applyAsInt(order);
        Assertions.assertEquals(totalPrice, 1_303_000);
    }

````

## 9.3.2 람다를 사용하는 메서드의 동작에 집중하라

람다의 목표는 정해진 동작을 다른 메서드에서 사용할 수 있도록 하나의 조각으로 캡슐화하는 것이다.<br>
그러려면 세부 구현을 포함하는 람다 표현식을 공개하지 말아야 한다.<br>
람다 표현식을 사용하는 메서드의 동작을 테스트함으로써 람다를 공개하지 않으면서도 람다 표현식을 검증할 수 있다.<br>

````java
public static class Order {
...
    public static Order limitProductPrice(Order order, int maxPrice) {
        List<OrderProduct> newProducts = order.getProducts().stream()
                .filter(p -> p.getPrice() <= maxPrice).collect(Collectors.toList());
        return new Order(newProducts);
    }
}

@Test
void test() {
    Order order = new Order(List.of(
            new OrderProduct("TV", 1, 300_000),
            new OrderProduct("공책", 3, 1_000),
            new OrderProduct("컴퓨터", 1, 1_000_000)
    ));
    Order newOrder = Order.limitProductPrice(order, 990_000);
    int totalPrice = Order.getTotalPrice.applyAsInt(newOrder);
    Assertions.assertEquals(totalPrice, 303_000);
}
````

## 9.3.4 고차원 함수 테스팅
함수를 인수로 받거나 다른 함수를 반환하는 메서드(고차원 함수)는 좀 더 사용하기 어렵다.<br>
메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트 할 수 있다.<br>
```java
@Test
public void testFilter() throws Exception {
List<Integer> numbers = Arrays.asList(1,2,3,4);
List<Integer> even = filter(numbers, i -> i%2 == 0);
List<Integer> smallerThanThree = filter(numbers, i->i<3);
assertEquals(Arrays.asList(2,4), even);
assertEquals(Arrays.asList(1,2), smallerThanThree);
```

## 9.4 디버깅

<hr>

디버깅 시 두가지를 확인해야 한다.
- 스택 트레이스
- 로깅

하지만 람다 표현식과 스트림은 기존의 디버깅을 무력화 한다.   
이를 디버깅 하는 방법을 살펴보자

## 9.4.1 스택 트레이스 확인

### 람다와 스택 트레이스
- 람다 표현식은 이름이 없기 때문에 복잡한 스택 트레이스가 생성된다.
- 메서드 참조를 사용해도 스택 트레이스에는 메서드 명이 나타나지 않고, 임의의 이름을 만들어낸다.

메서드 참조를 사용하는 클래스와 같은 곳에 선언되어있는 메서드를 참조할 때는 메서드 참조 이름이 스택 트레이스에 나타난다.
```java
public class Debugging {
  public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(1,2,3);
    numbers.stream().map(Debugging::devideByZero).forEach(System.out::println);
  }
  public static int divideByZero(int n) {
    return n / 0;
  }
}
```

위 처럼 작성한다면 스택 트레이스에 Debugging.divideByZero가 출력된다.

>책 출판 시간대(자바 10) 기준 과거에는 람다를 디버깅하기 어려웠다.
>
>자바 17 기준 해결된 듯 하다.

## 9.4.2 정보 로깅

```java
numbers.stream()
  .map(x -> x + 17)
  .filter(x -> x % 2 == 0)
  .limit(3)
  .forEach(System.out::println);

// 출력
// 20
// 22
```

위 연산에서 forEach를 호출하는 순간 전체 스트림이 소비된다.   
스트림 파이프라인에 적용된 각각의 연산(map, filter, limit)이 어떤 결과를 도출하는지 확인하려면 어떻게 해야할까?

바로 peek이라는 스트림 연산을 활용하면 된다.   
peek은 스트림의 각 요소를 소비한 것처럼 동작하지만, forEach처럼 실제로 소비하지는 않는다.

```java
numbers.stream()
  .peek(x -> System.out.println("from stream: " + x))
  .map(x -> x + 17)
  .peek(x -> System.out.println("after map: " + x))
  .filter(x -> x % 2 == 0)
  .peek(x -> System.out.println("after filter: " + x))
  .limit(3)
  .peek(x -> System.out.println("after limit: " + x))
  .collect(toList());

```

이제 각 단계별로 결과 값을 출력할 수 있다.

## 정리

<hr>

- 리팩터링은 코드 가독성과 유연성을 개선하는 데 중요하며, 람다 표현식과 메서드 참조를 활용하여 간결한 코드를 작성할 수 있다.
- 익명 클래스를 람다로 변환할 때 주의해야 할 점은 this와 super의 의미가 다르며, 모호한 타입 문제도 발생할 수 있다.
- 스트림 API를 사용하면 데이터 처리 파이프라인의 의도를 명확히 보여주고, 병렬 처리가 용이하다.
- 람다 표현식은 동작 파라미터화를 통해 유연성을 높이며, 함수형 인터페이스를 통해 다양한 기능을 구현할 수 있다.
- 람다의 테스트는 익명성 때문에 어려울 수 있지만, 필드에 저장하여 테스트하거나 메서드의 동작을 검증함으로써 가능하다.
- 디버깅 시 스택 트레이스를 확인하고, 로깅을 통해 각 스트림 연산의 결과를 추적해야 한다.
- peek 메서드를 사용하면 스트림의 중간 결과를 출력하여 흐름을 이해하는 데 도움을 줄 수 있다.
