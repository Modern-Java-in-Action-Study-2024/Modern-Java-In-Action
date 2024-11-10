# 동작 파라미터화 코드 전달하기

> 동작 파라미터화 : 아직 어떻게 실행할 것인지 결정하지 않은 코드 블록
> - 사용 방법 : 요구사항에 맞게 프로그램에서 호출한다.

<details> <summary> 자바 8 람다 표현식 </summary> 어떻게 만들고 어디에 사용하고 등의 내용은 3장에서 시작!!!</details>

## 2.1 변화하는 요구사항에 대응하기
### 요구사항 1.1 녹색사과 필터링

~~~ java
enum Color {Red. Green}
~~~

~~~java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (Green.equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
~~~

만약 다양한 색으로 필터링 하는 등 조건을 바꾸고 싶다면?

### 요구사항 1.2 다양한 색 필터링
filterGreenApples 의 코드를 반복 사용하지 않고 filterRedApples 를 구현하는 방법은 무엇이 있을까?

-> 색을 파라미터화할 수 있도록 메서드에 파라미터 주가
~~~java
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getColor().equals(color)) {
            result.add(apple);
        }
    }
    return result;
}
~~~
~~~java
List<Apple> greenApples = filterApplesByColor(inventory, Green);
List<Apple> redApples = filterApplesByColor(inventory, Red);
~~~

만약 요구사항에 무게가 추가가 된다면?

### 요구사항 1.3 무게 필터링

~~~java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getweight() > weight) {
            result.add(apple);
        }
    }
    return result;
}
	

List<Apple> heavyApples = filterApplesByWeight(inventory, 150);
~~~

요구사항 1.2와 요구사항 1.3은 기준이 서로 다르다.
만약 어떤 기준으로 필터링할지 정하는 것까지 필요하다면? 
### 요구사항 1.4 가능한 모든 속성으로 필터링

여러 요구사항을 들어주기 위해서 모든 속성을 메서드 파라미터로 추가

~~~java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ((flag && apple.getColor().equals(color)) || (!flag && apple.getweight() > weight)) {
            result.add(apple);
        }
    }
    return result;
}

~~~java
	List<Apple> greenApples = filterApples(inventory, Green, 0, true);
	List<Apple> heavyApples = filterApples(inventory, null, 150, false);
~~~
> "형편없는 코드다." : 상처다..

## 2.2 동작 파라미터화
> 프레디케이트(Predicate) : 참 또는 거짓을 반환하는 함수
> 
> 인터페이스(Interface) : 선택 조건을 결정한다.

~~~java
public interface ApplePredicate {
    boolean test (Apple apple);
}


public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        returen apple.getWeight() > 150;
    }
}

public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        returen Green.equals(apple.getColor());
    }
}
~~~

<details><summary>전략 패턴</summary>

- 알고리즘을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음 런타임에 알고리즘을 선택하는 기법
- 객체가 할수 있는 행위들을 각각 전략으로 만들어 놓고, 동적으로 행위의 수정이 필요한 경우 전략을 바꾸어 행위의 수정이 가능하다.
</details>

### 요구사항 2.1 추상적 조건으로 필터링
~~~java
public static List<Apple> filterApples(List<Apple> inventory, AppleRedicate p) {
    List<Apple> result = new ArrayList<>();
    for(Apple apple : inventory) {
        if(p.test(apple)) {
            test.add(apple);
        }
    }
    return result;
}
~~~

~~~java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        returen Red.equals(apple.getColor()) && apple.getWeight() > 150;
    }
}

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
~~~

## 2.3 복잡한 과정 간소화

~~~java
class AppleHeavyWeightPredicate implements ApplePredicate {
	public boolean test(Apple apple) {
		return apple.getWeight() > 150;
	}
}
class AppleGreenColorPredicate implements ApplePredicate {

	public boolean test(Apple apple) {
		return "Green".equals(apple.getColor());
	}
}


public class FilteringApples {

	public static void main(String... args) {
		List<Apple> inventory = Arrays.asList(new Apple(80, "Green"), new Apple(155, "Green"),
			new Apple(120, "Red"));
		List<Apple> heavyApples = filterApples(inventory, new AppleHeavyWeightPredicate());
		List<Apple> greenApple = filterApples(inventory, new AppleGreenColorPredicate());
		System.out.println("Heavy Apples: " + heavyApples);
	}

	public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
		List<Apple> result = new ArrayList<>();
		for (Apple apple : inventory) {
			if (p.test(apple)) {
				result.add(apple);
			}
		}
		return result;
	}
}
~~~

> Heavy Apples: [Apple{color='Green', weight=155}]

### 요구사항 3.1 익명클래스 사용
<details><summary>익명클래스란?</summary>

- 자바의 지역클래스와 비슷한 개념
- 클래스 이름이 없고, 객체가 생성될때 즉시 정의되며, 한번만 사용된다.
- 상위 클래스나 인터페이스를 구현한다.
</details>

~~~java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolaean test(Apple apple) {
        return Red.equals(apple.getColor());
    }
});
~~~
> 익명클래스의 단점 
> 1. 반복되는 코드
> 2. 코드의 장황함
> 
> 단점을 보완하기 위해 자바 8에서 람다 표현식을 사용할 수 있다.

### 요구사항 3.2 람다 표현식 사용

~~~java
public class filter {
	public static <T> List<T> filter(List<T> list, Predicate<T> p) {
		List<T> result = new ArrayList<>();
		for (T e: list) {
			if (p.test(e)) {
				result.add(e);
			}
		}
		return result;
	}

	public static void main(String[] args) {
		// 사과 목록
		List<Apple> inventory = Arrays.asList(
			new Apple(100, "Red"),
			new Apple(150, "Green"),
			new Apple(120, "Red"),
			new Apple(200, "Yellow")
		);

		// Red 색상의 사과만 필터링
		List<Apple> redApples = filter(inventory, (Apple apple) -> "Red".equals(apple.getColor()));
		System.out.println("Color of Apples: " + redApples);

	}
}
~~~

## 2.4 실전 예제
### Comparator로 정렬하기

~~~java
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getweight());
    )
));

>>

inventory.sort((Apple a1, Apple a2) -> a.getWeight().compareTo(a2.getWeight()));
~~~

<details><summary>새로운 문법?</summary> 3장에서 람다 표현식 구현하는 방법을 자세히 설명한다!</details>

### Runnable로 코드 블록 실행하기
자바 스레드를 이용하면 병렬로 코드 블록을 실행할 수 있고, Runnable을 이용하여 실행할 코드 블록을 지정할 수 있다.
<details><summary>추가 설명</summary>
스레드 : 자바 프로그램에서 작업단위로, 여러 개의 스레드를 동시에 실행하면 병렬 처리가 이루어진다.
</details>

~~~java
Thread t = new Thread(new Runnable() {
    public void run() {
        System.out.println("Hello world");
    }
});

>>
    
Thread t = new Thread(() -> System.out.println("Hello world"));
~~~

### Callable을 결과로 반환하기
Callable 인터페이스를 이용해 결과를 반환하는 태스트를 만든다.

<details><summary>추가 설명</summary>
Callable : 비동기적 작업을 실행할 때 사용할 수 있는 인터페이스로, 값을 반환하거나 예외를 던질 수 있는 작업을 정의할 수 있다.
</details>

~~~java
ExcutorService excutorService = Excutors.newCachedThredPool();
Futre<String> threadName = execustorService.submit(new Callabel<String>() {
    @Override 
    public Strig call() throws Exception {
        return Thread.currentThread().getName();
    }
});

>>

Future<String> threadName = executorService.submit( () -> Thread.currentThread().getName());
~~~

### GUI 이벤트 처리하기
마우스 클릭이나 문자열 위로 이동하는 등의 이벤트에 대응하는 동작을 수행한다.

## 2.5 결론
- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달
- 동작 파라미터화 이용하면, 변화하는 요구사항에 대해 더 잘 대응할 수 있는 코드 구현 가능
- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달 가능
  - 자바 8 이전도 가능은 했지만 코드가 지저분해졌다
- 자바 API의 많은 메서드는 정렬, 스레드, GUI처리 등을 포함한 다양한 동작으로 파라미터화 가능