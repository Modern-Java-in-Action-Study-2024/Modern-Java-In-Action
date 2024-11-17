# 람다 표현식

## 3.1 람다란 무엇인가?
<hr>

람다는 Java 8부터 도입된 함수형 프로그래밍 기능 중 하나로, 
간결한 코드 작성 가능하며, 코드 가독성과 유지보수 향상 도움이 된다.

- 익명성: 람다는 이름 없는 함수임. 코드 간결하게 작성 가능.
- 함수형 인터페이스: 람다는 함수형 인터페이스 구현함.
- 간결성: 람다는 기존 익명 클래스보다 간결한 문법 제공.
- 전달: 람다 표현식을 메서드 인수로 전달하거나 변수로 저장.

```Java
Comparator<Apple> byWight = new Comparator<Apple>() {
    @Override
    public int compare(Apple o1, Apple o2) {
        return o1.getWeight().compareTo(o2.getWeight());
    }
};
```

```Java
Comparator<Apple> byWightL = (Apple o1, Apple o2) -> o1.getWeight().compareTo(o2.getWeight());
```

## 3.2 어디에, 어떻게 람다를 사용할까?
<hr>

결론적으로는 함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.

![image](https://github.com/user-attachments/assets/d3f4e841-4264-4337-837b-836f4007d56a)

![image](https://github.com/user-attachments/assets/aa8a0d3b-66bc-4ca4-b07e-4e268dcd0320)
`default method`, `static method`는 상관 없고,

```Java
@FunctionalInterface
public interface Banana {
    void r();
    boolean equals(Object obj);
}

```

`@FunctionalInterface` 어노테이션을 통해서 함수형 인터페이스의 명시와 보증이 가능하다.

이러한 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있다.

- 언어 설계자들은 자바에 함수 형식(람다 표현식을 표현하는 데 사용한 시그니처와 같은 특별한 표기법)을 추가하는 방법도 대안으로 고려했으나, 언어를 더 복잡하게 만들지 않는 현재 방법을 선택
- 대부분의 자바 프로그래머가 이벤트 처리 인터페이스와 같이 하나의 추상 메서드를 갖는 인터페이스에 이미 익숙하다는 점도 고려

### 함수 디스크립터

함수형 인터페이스의 추상 메서드 시그니처(메서드 구조를 정의한 것)는 람다 표현식의 시그니처를 가리킨다. 함수 디스크립터는 람다 표현식의 시그니처를 서술하는 메서드를 의미한다.

예를 들어, `() -> void`는 파라미터 리스트가 없으며 void를 반환하는 함수를 의미하는 디스크립터며, `(int, int) -> double`는 두개의 `int`를 파라미터로 받아 `double`형 자료를 반환하는 함수를 설명한다고 보면 된다.

## 3.3 람다 활용 : 실행 어라운드 패턴
<hr>

자원 처리에 사용하는 순환 패턴(recurrent pattern)은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다.
> ex) BufferedReader

설정(setup)과 정리(cleanup) 과정은 대부분 비슷하다. 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다.
이와 같은 형식의 코드를 실행 어라운드 패턴(execute around pattern)이라고 부른다.

```Java
public String processFile() throws IOException {
    try (BufferedReader br = new BufferReader(new FileReader("data.txt"))) {
        return br.readline(); //파일에서 한 행을 읽는 코드
    }
}
```
<br>

현재 코드는 한 번에 한 파일에서 한 줄만 읽을 수 있다. 이후 요구사항이 변경되어 한 번에 두 줄을 읽거나, 가장 자주 사용되는 단어를 리턴하려면 어떻게 해야 할까?

`processFile` 메서드의 동작을 파라미터화하면 해결된다.
`processFile` 메서드가 `BufferedReader`를 이용해서 다른 동작을 수행할 수 있도록 `processFile` 메서드로 동작을 전달해야 한다.


### 1단계 : 동작 파라미터화를 기억하라

```Java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

### 2단계 : 함수형 인터페이스를 이용해서 동작전달

```Java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}

public String processFile(BufferedReaderProcessor p) throws IOException {
    //...
}
```

### 3단계 : 동작 실행

```Java
public String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
        return p.process(br); //BufferedReader 객체 처리
    }
}
```

### 4단계 : 람다 전달

```Java
String oneLine = processFile((BufferedReader br) -> br.readLine());

String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

## 3.4 함수형 인터페이스 사용
<hr>

함수형 인터페이스의 추상 메서드 시그니처를 함수 디스크립터라고 한다.

다양한 람다 표현식을 사용하기 위해서는 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다.

자바 8에서는 `java.util.function` 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다.

### Predicate

```Java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();
    for (T t : list) {
        if (p.test(t)) {
            results.add(t);
        }
    }
    return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```
`Predicate` 인터페이스는 `test`라는 추상 메서드를 정의하며 `test`는 제네릭 형식 T의 객체를 인수로 받아 `boolean`을 반환한다.

우리가 만들었던 인터페이스와 같은 형태인데 따로 정의할 필요 없이 바로 사용할 수 있다는 점이 특징이다.

즉 제네릭 T 형식의 객체를 사용하는 `boolean` 표현식이 필요한 상황에서 `Predicate` 인터페이스를 사용할 수 있다.

### 기본형 특화
자바의 모든 형식은 참조형(Byte,Integer,Object,List) 아니면 기본형(int,double,char,byte..)에 해당한다.

하지만 제네릭 내부 구현 때문에 제네릭 파라미터에는 참조형만 사용할 수 있다.

자바에서는 `기본형 -> 참조형`으로 변환하는 기능을 제공하는데, 이 기능을 박싱 이라고 한다.
반대로 `참조형 -> 기본형` 으로 변환하는 동작을 언방식 이라고 한다.

박싱과 언박싱을 자동으로 처리하는 오토박싱 기능도 지원한다.
하지만 이러한 변환 과정에는 비용이 들기 때문에 자바 8에서는 오토박싱을 피할 수 있도록 특별한 버전의 함수형 인터페이스도 제공한다.


![image](https://github.com/user-attachments/assets/6ae47a81-e63a-44a8-a3cc-d05f348e9685)

## 3.5 형식 검사, 형식 추론, 제약
<hr>

람다 표현식 자체에서 람다가 어떤 함수형 인터페이스를 구현하는지의 
정보가 포함되어 있지 않으므로 람다 표현식을 제대로 이해하기 위해서는 람다의 실제 형식을 파악해야 한다.

### 1. 형식 검사

람다가 사용되는 콘텍스트를 이용해서 람다의 형식(Type)을 추론할 수 있다. 
어떤 콘텍스트(문맥, 예를 들면 람다가 전달될 메서드 파라미터나 람다가 할당되는 변수 등)에서 
기대되는 람다 표현식의 형식을 대상 형식이라고 부른다.

```Java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

public <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> results = new ArrayList<>();
    for(T t: list) {
        if(p.test(t)) {
            result.add(t);
        }
        return results;
    }
}

List<Apple> heavierThan150g = 
filter(inventory, (Apple apple) -> apple.getWeight() > 150);
```

>1. filter 메서드의 선언을 확인한다.
>2. filter 메서드는 두 번째 파라미터로 Predicate<Apple> 형식(대상 형식)을 기대한다.
>3. Predicate<Apple>은 test라는 한 개의 추상 메서드를 정의하는 함수형 인터페이스다.
>4. test 메서드는 Apple을 받아 boolean을 반환하는 함수 디스크립터를 묘사한다.
>5. filter 메서드로 전달된 인수는 이와 같은 요구사항을 만족해야 한다.

### 2. 같은 람다, 다른 함수형 인터페이스

대상 형식이라는 특징 때문에 같은 람다 표현식이라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.

예를 들어 `Callable`과 `PrivilegedAction` 인터페이스는 인수를 받지 않고 제네릭 형식 `T`를 반환하는 함수를 정의한다.

```Java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

>**다이아몬드 연산자**
> 
>주어진 클래스 인스턴스 표현식을 두 개 이상의 다양한 콘텍스트에 사용할 수 있다.
>이때 인스턴스 표현식의 형식 인수는 콘텍스트에 의해 추론된다.

```Java
List<String> listOfStrings = new ArrayList<>(); // String 타입으로 추론된다.
Map<String, Integer> mapOfContext = new HashMap<>(); // String, Integer 타입으로 추론된다.
```


>**특별한 void 호환 규칙**
> 
>람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 디스크립터와 호환된다.
>물론 파라미터 리스트도 호환되어야 한다.

```Java
// Predicate 는 boolean 반환값을 갖는다.
Predicate<String> p = s -> list.add(s);
// Consumer는 void 반환값을 갖는다.
Consumer<String> b = s -> list.add(s);
```

### 3. 형식 추론

우리 코드를 좀 더 단순화할 수 있는 방법이 있다.
자바 컴파일러는 람다 표현식이 사용된 콘텍스트(대상 형식)를 이용해서
람다 표현식과 관련된 함수형 인터페이스를 추론한다.

즉, 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다.
결과적으로 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략 할 수 있다.

![image](https://github.com/user-attachments/assets/b76b1593-56cb-4b86-b0ce-6002f3a5f460)

상황에 따라 명시적으로 형식을 포함하는 것이 좋을 때도 있고 형식을 배제하는 것이 가독성을 향상시킬 때도 있다.

람다에 형식 추론 대상 파라미터가 하나 뿐일때는 해당 파라미터명을 감싸는 괄호도 생략할 수 있다.

### 4. 지역 변수 사용

지금까지 살펴본 모든 람다 표현식은 인수를 자신의 바디 안에서만 사용했다.

하지만 람다 표현식에서는 익명 함수가 하는 것처럼 자유 변수(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다.

이와 같은 동작을 람다 캡처링이라고 부른다.
![image](https://github.com/user-attachments/assets/e1ebb0cf-6f11-442e-a77e-6401a0535711)

하지만 자유 변수에도 약간의 제약이 있다.

람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처할 수 있지만 
그러려면 지역 변수는 명시적으로 `final`로 선언되어 있어야 하거나
실직적으로 final로 선언된 변수와 똑같이 사용되어야 한다.

즉, 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡처할 수 있다.

![image](https://github.com/user-attachments/assets/a7e5df1f-139d-4bab-9347-c1a53a6c2735)

또한, 람다는 자기 자신을 참조할 수 없다.

```Java
public class LambdaReference {

    public static void main(String[] args) {
        LambdaReference lambdaReference = new LambdaReference();
        lambdaReference.run(); // LambdaReference
    }

    public void run() {
        Runnable runnable = () -> System.out.println(this);
        runnable.run();
    }

    @Override
    public String toString() {
        return "LambdaReference";
    }
}
```

### 지역변수의 제약

왜 지역 변수에 이런 제약이 필요할까?

우선 내부적으로 인스턴스 변수와 지역 변수는 저장되는 위치가 다르다. <br>
인스턴스 변수는 힙에 저장되는 반면 지역 변수는 스택에 저장된다.

람다에서 지역 변수에 바로 접근할 수 있다는 가정 하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서
변수 할당이 해제 되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다.

따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다.<br>
따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한 번만 값을 할당해야 한다는 제약이 생긴다.

## 3.6 메서드 참조
<hr>

- 특정 메서드만을 호출하는 람다 표현식의 축약형

- 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다.

- 때로는 람다 표현식보다 더 가독성이 좋으며 자연스러울 수 있다.

```Java
inventory.sort((Apple a1, Apple a2) ->
 				a1,getWeight().compareTo(a2.getWeight()));


inventory.sort(comparing(Apple::getWeight));
```

>언제 유용한가?
>
>람다가 "이 메서드를 직접 호출해!"라고 명령할 경우 → 메서드명을 직접 참조하는 것이 편리함
>이 때 명시적으로 메서드명을 참조할 수 있음 → 가독성 향상

구분자 `::`를 붙여서 사용한다.

>Apple::getWeight : Apple 클래스에 정의된 getWeight의 메서드 참조<br>
>(= 람다표현식 (Apple a) -> a.getWeight())

### 람다 표현식 -> 메서드 참조

```Java
List<String> list = Arrays.asList("a","b","A","B");
list.sort((s1, s2) -> s1.compareToIgnoreCase(s2));
```

`List`의 `sort` 메서드 : 인수로 `Comparator`을 기대한다.
`Comparator` : `(T, T) -> int`라는 함수 디스크립터를 갖는다.
`String` 클래스 : `compareToIgnoreCase`라는 메서드를 갖는다.

```Java
List<String> list = Arrays.asList("a","b","A","B");
list.sort(String::compareToIgnoreCase);
```

컴파일러는 메서드 참조가 주어진 함수형 인터페이스와 호환하는지 확인한다. (like 람다 표현식의 형식을 검사하던 방식)

따라서, 메서드 참조는 콘텍스트 형식과 일치해야 한다.

### 메서드 참조 방법

1. 정적 메서드 참조
   - `Integer::parseInt`
2. 다양한 형식의 인스턴스 메서드 참조
   - `String::length`
   - 메서드 참조를 이용해서 람다 표현식의 파라미터로 전달할 때 사용
3. 기존 객체의 인스턴스 메서드 참조
   - Transaction 객체를 할당받은 expensiveTransaction 지역변수가 있고, 객체에 getValue 메서드가 있다면
     `expensiveTransaction::getValue`

### 생성자 참조

`ClassName::new` 처럼 클래스명과 `new` 키워드를 이용해서 기존  생성자의 참조를 만들 수 있다.

```Java
Supplier<Apple> c1 = () -> new Apple();
Supplier<Apple> c2 = Apple::new;

Apple a1 = c1.get();
Apple a2 = c2.get();
```

`Apple(Integer weight)` 라는 시그니처를 갖는 생성자는 `Function` 인터페이스와 시그니처가 같다. 
따라서 다음과 같은 코드를 구현할 수 있다.

```Java
Function<Integer, Apple> c3 = (weight) -> new Apple(weight);
Function<Integer, Apple> c4 = Apple::new;

Apple a3 = c3.apply(110);
Apple a4 = c4.apply(110);

BiFunction<Color, Integer, Apple> c5 = (color, weight) -> new Apple(color, weight);
BiFunction<Color, Integer, Apple> c6 = Apple::new;

Apple a5 = c5.apply(GREEN, 110);
Apple a6 = c6.apply(GREEN, 110);
```

## 3.7 람다, 메서드 참조 활용하기
<hr>

리스트를 다양한 정렬 기법으로 정렬하는 문제를, 더욱 간결하게 해결하는 방법을 정리한다. 
2장부터 3장까지 다룬 동작 파라미터화, 익명 클래스, 람다 표현식, 메서드 참조를 통해 최종적으로 
아래와 같은 코드가 만들어진다.

`inventory.sort(comparing(Apple::getWeight));`

처음에 다룬 사과 리스트를 다양한 정렬 기법으로 정렬하는 문제로 돌아가보자

### 1단계 : 코드전달

List API에서 `sort` 메서드를 제공하므로 직접 정렬 메서드를 구현할 필요는 없다. 
`sort` 메서드는 다음과 같은 시그니처를 갖는다.

```Java
void sort(Comparator<? super E> c);
```

`sort` 메서드는 `Comparator` 객체를 인수로 받아 두 사과를 비교한다.


```Java
@FunctionalInterface
public interface Comparator<T> {
	int compare(T o1, T o2);
}
```

객체 안에 동작을 포함시키는 방식으로 다양한 전략을 전달할 수 있다. <br>
즉, 이제 '`sort`의 동작은 파라미터화 되었다'라고 말할 수 있다.

`sort`에 전달된 `compare`의 정렬 전략에 따라 `sort`의 동작이 달라질 것이다.

### 2단계 : 익명 클래스 사용

한 번만 사용할 `Comparator`를 구현하는 것보다는 익명 클래스를 이용하는 것이 좋다.

### 3단계 : 람다 표현식 사용

한 번만 사용할 익명 클래스를 만든 건 좋았으나 코드가 장황하다. 
함수형 인터페이스를 기대하는 곳 어디에서나 람다 표현식을 사용할 수 있으니 사용해보자.

```Java
Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());

Comparator<Apple> c = Comparator.comparing(apple -> apple.getWeight());
```

### 4단계 : 메서드 참조 사용

메서드 참조를 이용하면 람다 표현식의 인수를 더 깔끔하게 전달할 수 있으니 메서드 참조를 이용해서 코드를 조금 더 간소해보자.

```Java
// java.util.Comparator.comparaing은 정적으로 임포트했다고 가정
inventory.sort(comparing(Apple::getWeight));
```

단지 코드만 짧아진 것이 아니라 코드의 의미도 명확해졌다. 
코드 자체로 '`Apple`을 `weight`별로 비교해서 `inventory`를 `sort`하라'는 의미를 전달할 수 있다.

## 3.8 람다 표현식을 조합할 수 있는 유용한 메서드
<hr>

간단한 여러개의 람다 표현식을 조함해서 복잡한 람다 표현식을 만들 수 있다.

자바에서는 함수끼리의 연산을 돕는 디폴트 메서드를 통해 구현한다.

### Comparator 조합

#### **역정렬**

`Comparator.comparing` 에 reversed 라는 `default method`가 있다.

해당 메서드를 콜하면, 정렬 순서를 바꾼 함수 인터페이스의 인스턴스를 생성한다.

```Java
inventory.sort(comparing(Apple::getWeight).reversed());
```

무게가 같은 사과에서는 원산지 국가별로 사과를 정렬하고 싶다. 
이럴때에는 `thenComparing` 메서드를 사용하여 두 번째 비교자를 만들 수 있다.

```Java
inventory.sort(comparing(Apple::getWeight)
          .reversed()     // 내림차순 정렬
          .thenComparing(Apple::getCountry));
```

### Predicate 조합

negate, and, or 디폴트 메서드를 제공한다.

### Function 조합

andThen, compose 두 가지 디폴트 메서드를 제공한다.

## 3.9 비슷한 수학적 개념
<hr>

람다 표현식과 함수전달은 비슷하다.

공학에서는 함수가 차지하는 영역을 묻는 질문이 자주 등장한다.

$$\int_{3}^{7} (x + 10) dx $$

![image](https://github.com/user-attachments/assets/69d04982-af68-4975-968a-e7d7c4a0f53a)

사다리꼴 범위를 구하는 방법은 다음과 같다

`1/2 x ((3+10) + (7+10)) x (7-3) = 60`

이것을 함수로 나타내면 `integrate(f,3,7)` 처럼 사용할 수 있다.

이것을 다시 자바 버전으로 바꾸면 `integrate((double x) -> x + 10, 3, 7)` 로 바꿀 수 있다.

`(double x) -> x + 10` 는 `Function<T, R>` 을 사용해서 
메소드 시그니쳐는 `public double integrate(Function<T,R> f, double a, double b)` 가 된다.

사다리꼴 범위를 구하는 방법을 구현코드로 바꾸면 아래처럼 된다.

```Java
public double integrate(Function<T,R> f, double a, double b) {
	return (f.apply(a) + f.apply(b)) * (b - a) / 2.0;
}
```

수학처럼 f(a) 라고 표현을 할 수 없고 f.apply(a) 라고 구현하는 것은 <br>
자바가 진정으로 함수를 허용하지 않고 모든 것을 객체로 여기는것을 포기할 수 없기 때문이다.


## 정리
<hr>

- 람다 표현식은 익명 함수의 일종이다. 이름은 없지만, 파라미터 리스트, 바디, 반환 형식
을 가지며 예외를 던질 수 있다.
- 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용할 수 있다.
- 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공할 수 있으며 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.
- 실행 어라운드 패턴을 람다와 활용하면 유연성과 재사용성을 추가로 얻을 수 있다.
- 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.
- `Comparator`, `Predicate`, `Function` 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다.

## 문제

1. 메서드 참조와 생성자 참조에 대한 설명으로 틀린 것은 무엇인지 구하시오. 
   1. 타입 추론을 위해서 인터페이스의 추상 메서드 형태와 반환 메서드의 시그니처 형태가 같아야 한다.
   2. 인스턴스 메서드는 "참조변수::메서드"로 작성한다. 
   3. 정적 메서드는 "클래스::메서드"로 작성한다. 
   4. 생성자 참조인 "클래스::new"는 매개변수가 없는 디폴트 생성자만 호출 가능하다.

2. 아래 코드의 문제점은?
```Java
String name = () -> "sin";
```

3. 아래 코드의 문제점은?

```Java
import java.util.ArrayList;

@FunctionalInterface
public interface ABCD<T> {
    boolean e(T t);
    default int getNum(){
        return 0;
    }
    static void setNum(){
    }
    boolean equals(Object obj);
}

public class Apple {
    private int weight;

    public int getWeight() {
        return this.weight;
    }
}

public class Main {
    public static void main(String[] args) {
        List<Apple> appleList = List.of(...);

        ABCD<Apple> abcd = (Apple a) -> a.getWeight();

        List<Integer> appleWeightList = new ArrayList<>();

        for (Apple a : appleList) {
            appleWeightList.add(abcd.e(a));
        }
    }
}
```