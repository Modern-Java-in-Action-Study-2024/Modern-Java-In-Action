# 스트림 활용

> ### 데이터 처리 질의  
> 필터링, 슬라이싱, 매칭  
> 검색, 매칭, 리듀싱  
> 특정 범위의 숫자와 같은 숫자 스트림 사용하기  
> 다중 소스로부터 스트림 만들기  
> 무한 스트림

## 5.1 필터링
* 스트림 인터페이스는 filter 메서드를 지원한다.

### 5.1.1 프레디케이트로 필터링

~~~java
List<Dish> vegetarianMenu = menu.stream()
                            .filter(Dish::isVegeterian)  //채식 요리인지 확인하는 메서드
                            .collect(toList());
~~~

### 5.1.2 고유 요소 필터링
- 고유요소로 이루어진 스트림을 반환하는 distinct 메서드 지원한다.
~~~java
List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
numbers.stream().filter(i -> i % 2 == 0)  // list에 있는 숫자들이 i이며, 2로 나누어 떨어지는 것만 필터링
        .distinct()  // 중복되는 것 삭제
        .forEach(System.out::println);  // 하나씩 출력
~~~

## 5.2 스트림 슬라이싱
- 스트림의 요소를 선택하거나 스킵하는 다양한 방법
- 자바 9의 새 기능

### 5.2.1 프레디케이트를 이용한 슬라이싱
- 자바 9에서는 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두 가지 새로운 메스드를 지원한다.

### TakeWhile 활용

- 다음와 같은 요리 목록은 정렬되어 있다.
- 320 칼로리 이하의 요리 선택하는 방법은?
~~~java
List<Dish> specialMenu = Arrays.asList(
	new Dish("season fruit", true, 120, Type.OTHER),
	new Dish("prawns", false, 300, Type.FISH),
	new Dish("rice", true, 350, Type.OTHER),
	new Dish("chicken", false, 400, Type.MEAT),
	new Dish("french fries", true, 530, Type.OTHER)
);

List<Dish> filterMenu = specialMenu.stream().filter(dish -> dish.getCalories() < 320).collect(toList());

List<Dish> takeWhileMenu = specialMenu.stream().takeWhile(dish -> dish.getCalories() < 320).collect(toList());
~~~
- filter 연산을 이용하면 전체 스트림을 반복하면서 각 요소에 프레디케이드를 적용한다. > 즉 모든 요소를 하나씩 확인한다.
- takeWile 연산은 주어진 조건에 거짓이 되는 요소를 만나면 중단한다.


- 만약 정렬되어 있지 않는다면?
~~~java
List<Dish> specialMenu1 = Arrays.asList(
    new Dish("rice", true, 350, Type.OTHER),
    new Dish("chicken", false, 400, Type.MEAT),
    new Dish("season fruit", true, 120, Type.OTHER),
    new Dish("prawns", false, 300, Type.FISH),
    new Dish("french fries", true, 530, Type.OTHER)
);

List<Dish> takeWhileMenu1 = specialMenu1.stream().takeWhile(dish -> dish.getCalories() < 320).collect(toList());
System.out.println("takeWhileMenu : " + takeWhileMenu1);
~~~

<details><summary> 정답 </summary> 
takeWhileMenu : [] 
</details>

### DropWhile 활용
- 반대로 320 칼로리보다 큰 요소 탐색은 어떻게 할까?
~~~java
List<Dish> specialMenu = Arrays.asList(
    new Dish("season fruit", true, 120, Type.OTHER),
    new Dish("prawns", false, 300, Type.FISH),
    new Dish("rice", true, 350, Type.OTHER),
    new Dish("chicken", false, 400, Type.MEAT),
    new Dish("french fries", true, 530, Type.OTHER)
);

List<Dish> filterMenu = specialMenu.stream().filter(dish -> dish.getCalories() > 320).collect(toList());

List<Dish> dropMenu = specialMenu.stream().dropWhile(dish -> dish.getCalories() < 320).collect(toList());
~~~
- dropWhil은 takeWhile과 정반대이다.
- 처음으로 거짓이 되는지점까지 발견된 요소를 버린다. 즉, 거짓이 되면 그 지점에서 작업을 중단하고 남은 모든 요소를 반환한다.

### 5.2.2 스트림 축소
- 스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드 지원한다.


- 300 칼로리 이상의 세 요리를 반환하는 방법은?
~~~java
List<Dish> specialMenu = Arrays.asList(
    new Dish("season fruit", true, 120, Type.OTHER),
    new Dish("prawns", false, 300, Type.FISH),
    new Dish("rice", true, 350, Type.OTHER),
    new Dish("chicken", false, 400, Type.MEAT),
    new Dish("french fries", true, 530, Type.OTHER)
);

List<Dish> dishes = specialMenu.stream().filter(dish -> dish.getCalories() > 300).limit(3).collect(toList());
~~~

### 5.2.3 요소 건너뛰기
- 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원한다.
- n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환된다.


- 300칼로리 초과학고 처음 두 요리를 건너뛴 다음 300칼로리 넘는 나머지 요리 반환하는 방법은?
~~~java
List<Dish> specialMenu = Arrays.asList(
    new Dish("season fruit", true, 120, Type.OTHER),
    new Dish("prawns", false, 300, Type.FISH),
    new Dish("rice", true, 350, Type.OTHER),
    new Dish("chicken", false, 400, Type.MEAT),
    new Dish("french fries", true, 530, Type.OTHER)
);
List<Dish> dishes = specialMenu.stream()
                .filter(dish -> dish.getCalories() > 300) //300 초과인 음식
                .skip(2) //2개 뛰어넘기
                .collect(toList());
System.out.println("dishes : " + dishes);
~~~

> dishes : [french fries]

## 5.3 매핑
- 스트림 API 의 map 과 flatMap 메서드는 특정 데이터를 선택하는 기능을 제공한다.
- 데이터 처리 과정에서 자주 수행되는 연산이다.

### 5.3.1 스트림의 각 요소에 함수 적용하기
- 스트림은 함수를 인수로 받는 map 메서드를 지원한다.


- 단어 리스트가 주어졌을 때 각 단어가 포함하는 글자 수의 리스트 반환하는 방법은?
~~~java
List<String> words = Arrays.asList("Modern", "Java", "In", "Action");
List<Integer> wordsLength = words.stream().map(String::length).collect(toList());
~~~

> [6, 4, 2, 6]
- 
- 만약, 각 요리명의 길이를 알고 싶다면?
~~~java
List<Dish> specialMenu = Arrays.asList(
    new Dish("season fruit", true, 120, Type.OTHER),
    new Dish("prawns", false, 300, Type.FISH),
    new Dish("rice", true, 350, Type.OTHER),
    new Dish("chicken", false, 400, Type.MEAT),
    new Dish("french fries", true, 530, Type.OTHER)
);
List<Integer> dishNameLengths = specialMenu.stream()
                                            .map(Dish::getName)
                                            .map(String::length)
                                            .collect(toList());
~~~

> dishNameLengths : [12, 6, 4, 7, 12]

### 5.3.2 스트림 평면화
- 리스트에서 고유문자로 이루어진 리스트를 반환하기
- 즉, {"Hello", "World"} -> {"H", "e", "l", "o", "W", "r", "d"} 반환하기

~~~java
List<String> wordsTest = Arrays.asList("Hello", "World");
wordsTest.stream()  // hello, world : Stream<Sting>
    .map(tests -> tests.split("")) // {h,e,l,l,o}, {w,o,r,l,d} : Stream<String[]>
    .distinct() // ?? : Stream<STring[]>
    .collect(toList()); : List<String[]>
~~~

<details><summary>정답</summary>
wordsTest : [Hello, World]
</details>

- map 으로 전달한 람다는 각 단어의 String[]을 반환한다는 점에서 문제가 발생한다.
- 즉 map 메소드가 반환한 형식은 Stream<String[]> 이고, 우리가 원하는 것은 Stream<String> 이다.

### map 과 Arrays.stream 활용 > flatMap 사용
~~~java
List<String> wordsTest = Arrays.asList("Hello", "World");

List<String> uniqueChar = wordsTest.stream()
    .map(word -> word.split(""))  // Stream<String[]>
    .flatMap(Arrays::stream)  // Stream<String>
    .distinct()
    .collect(toList());
~~~
> uniqueChar : [H, e, l, o, W, r, d]
- flatMap 은 각 배열을 스트림의 콘텐츠로 매핑한다.
- 즉, flatMap 메서드는 스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다

## 5.4 검색과 매칭
- 특정 속성이 데이터 집합에 있는지 `여부` 를 `검색`하는 데이터 처리도 자주 사용된다.
- allMatch, anyMatch, noneMatch, findFrist, findAny 등의 메서드를 제공한다. 

### 5.4.1 프레디케이트가 적어도 한 요소와 일치하는지 확인
- anyMatch 메서드 : 적어도 한 요소와 일치하는지 학인

~~~java
if (specialMenu.stream().anyMatch(Dish::isVegetarian)) {
    System.out.println("This menu is somewhat vegetarian friendly!");
}
~~~

### 5.4.2 프레디케이트가 모든 요소와 일치하는지 검사
- allMatch 메서드 : 모든 요소가 주어진 프레디케이트와 일치하는지 검사

~~~java
boolean isHealthy = specialMenu.stream().allMatch(dish -> dish.getCalories() < 1000);
System.out.println("HealyFood : " + isHealthy);
~~~

### NONEMATCH
- nonMatch 메서드 : 일치하는 요소가 없는지 확인

~~~java
boolean isHealthyNone = specialMenu.stream().noneMatch(dish -> dish.getCalories() >= 1000);
System.out.println("isHealthy : " + isHealthyNone);
~~~

> anyMatch, allMatch, noneNatch, findfirst, findAny 메서드는 스트림 쇼트서킷 기법, 즉 자바의 &&, ||과 같은 연산을 활용한다.  
> 쇼트서킷이란 전체를 처리하지 않았더라도 결과를 반환할 수 있는 것을 맗한다.  
> 예를 들어, A && B 연산에서 A가 False 이면, B를 고려하지 않고 넘어간다.   
> 또한, A || B 연산에서 A 가 True 이면, B를 고려하지 않고 넘어간다.

### 5.4.3 요소 검색
- findAny 메서드 : 현재 스트림에서 임의의 요소를 반환한다.
- 다른 스트림 연산과 연결해서 사용할 수 있다.
- 쇼트서킷으로 결과를 찾는 즉시 실행을 종료한다.
~~~java
List<Dish> specialMenu = Arrays.asList(
    new Dish("season fruit", true, 120, Type.OTHER),
    new Dish("prawns", false, 300, Type.FISH),
    new Dish("rice", true, 350, Type.OTHER),
    new Dish("chicken", false, 400, Type.MEAT),
    new Dish("french fries", true, 530, Type.OTHER)
);

Optional<Dish> dish = specialMenu.stream().filter(Dish::isVegetarian).findAny();
System.out.println(dish);

specialMenu.stream().filter(Dish::isVegetarian).findAny().ifPresent(d -> System.out.println(d.getName()));
~~~

> `Optional`이란?   
> 값의 존재나 부재 여부를 표현하는 컨테이너 클래스이다.
> 값이 존재하는지 확인하고, 값이 없을 때 어떻게 처리할지 강제하는 기능을 제공한다.  
> `isPresent()` : Optional 이 값을 포함하면 True / 값을 포함하지 않으면 False 반환  
> `ifPresent(Consumer<T> block)` : 값이 있으면 주어진 블럭을 실행한다.  
> `T get()` : 값이 존재하면 값을 반환 / 값이 없으면 NoSuchElementException 을 일으킨다.  
> `T orElse(T other)` : 값이 있으면 값을 반환 / 없으면 기본값을 반환한다.

### 5.4.4 첫 번째 오소 찾기
- 리스트 또는 정렬된 연속 데이터로부터 생성된 스트림처럼 일부 스트림에 `논리적 아이템순서` 가 정해져 있는 경우 
- 첫 번째 요소 찾기

~~~java
List<Integer> someNumbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
                                                        .map(n -> n * n)
                                                        .filter(n -> n % 3 == 0)
                                                        .findFirst();
~~~


> 왜 findFirst 와 findAny 메서드가 모두 필요할까?  
> 병렬 실행에서는 첫 번째 요소를 찾기 어렵다.   
> 만약, 요소의 반환 순서가 상관 없다면, 병렬 제약이 적은 findAny 를 사용한다.

## 5.5 리듀싱
- 스트림 요소를 조합해서 더 복잡한 질의를 표한하는 방법에 대해 설명하는 장이다. 
- 예) 메뉴에서 가장 칼로리가 높은 요리는?
- 질의를 수행하려면 Integer 와 같은 결과가 나올때 까지 스트림의모든 요소를 반복적으로 처리해야 한다. > 리듀싱 연산
- 리듀싱 연산 : 모든 스트림 요소를 처리해서 값으로 도출하는 것
  - 함수형 프로그래밍 언어에서는 폴드 라고 부른다.
- 6 장에서는 더 복잡한 형식읠 리듀스를 설명한다. (Map 으로 리듀스)

### 5.5.1 요소의 합

~~~java
	int sum = 0;
	int[] numbers = {4, 5, 3, 9};
	for (int x : numbers) {
	sum += x;
	}
~~~
- 리스트의 숫자 요소 더하는 코드
- 파라미터를 두 개 사용함 : sum 변수의 초기갖ㅅ 0, 리스트의 모든 요소를 조합하는 연산 (+)

~~~java
	List<Integer> numbersList = Arrays.asList(4, 5, 3, 9);
	int sumReduce = numbersList.stream().reduce(0, (a, b) -> a + b);
	int sumTest = numbersList.stream().reduce(0, Integer::sum);
~~~
- reduce를 이용하여 반복된 패턴을 추상화할 수 있다.
- 두 개의 인수 : 초깃값 0, 두 요소를 조합해서 새로운 값을 만드는 (a,b) -> a + b

### 5.5.2 최대값과 최솟값
~~~java
Optional<Integer> max = numbersList.stream().reduce(Integer::max);
Optional<Integer> min = numbersList.stream().reduce(Integer::min);
~~~

### reduce 메서드의 장점과 병렬화   
기존의 단계적 반복으로 합계를 구하는 것과 reduce 를 이용해서 합계 구하는 것의 차이점은 무엇일까?   
- reduce 를 이용하면 내부 반복이 추상화되면서 병렬로 reduce를 실행할 수 있게 된다. 
- 그러나 반복적인 합계에서는 sum 변수를 공유해야 하므로 쉽게 병렬화하기 어렵다.
- 코드를 병렬로 만드는 방법인 parallelStream()이 있으나 대가를 지불해야 한다.
  - 람다의 상태가 바뀌지 말아야하며, 연산이 어떤 순서로 실행 되더라도 결과가 바뀌지 않는 구조여야 한다.

### 스트림 연산 : 상태 없음과 상태 있음
- 각각의 연산은 내부적인 상태를 고려해야 한다.
- map. filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 스트림으로 보낸다 > 내부 상태를 갖지 않는 연산이다.
- reduce, sum, max 같은 연산은 결과를 누적할 내부 상태가 필요하다. 내부 상태의 크기는 한정되어 있다.
- sorted, distnct 같은 연산은 과거의 이력을 알고 있어야 한다. > 내부 상태를 갖는 연산이다.

| 연산        | 형식                | 반환형식        | 사용된 함수형 인터페이스 형식       | 함수 디스크립터       |
|-----------|-------------------|-------------|------------------------|----------------|
| filter    | 중간연산              | Stream<T>   | Predicate<T>           | T -> boolean   |
| distinct  | 중간연산 (상태 있는 언바운드) | Stream<T>   | -                      | -              |
| takeWhile | 중간연산              | Stream<T>   | Predicate<T>           | T -> boolean   |
| dropWhile | 중간연산              | Stream<T>   | Predicate<T>           | T -> boolean   |
| skip      | 중간연산 (상태 있는 언바운드) | Stream<T>   | long                   | -              |
| limit     | 중간연산 (상태 있는 언바운드) | Stream<T>   | long                   | -              |
| map       | 중간연산              | Stream<T>   | Function<T,R>          | T -> R         |
| flatMap   | 중간연산              | Stream<T>   | Function<T, Stream<R>> | T -> Stream<R> |
| sorted    | 중간연산 (상태 있는 언바운드) | Stream<T>   | Comparator<T>          | (T, T) -> int) |
| anyMatch  | 최종연산              | boolean     | Predicate<T>           | T -> boolean   |
| noneMatch | 최종연산              | boolean     | Predicate<T>           | T -> boolean   |
| allMatch  | 최종연산              | boolean     | Predicate<T>           | T -> boolean   |
| findAny   | 최종연산              | Optional<T> | -                      | -              |
| findFirst | 최종연산              | Optional<T> | -                      | -              |
| forEach   | 최종연산              | void        | Consumer<T>            | T -> void      |
| collect   | 최종연산              | R           | Collector<T, A, R>     | -              |
| reduce    | 최종연산(상태있는 바운드)    | Optional<T> | BinaryOperator<T>      | (T, T) -> T    |
| count     | 최종연산              | long        | -                      | -              |

## 5.6 실전 연습
- 각자 해봤을 것이라고 믿겠습니다 ^^

## 5.7 숫자형 스트림

~~~java
// reduce 메서드로 칼로리 합을 구하기
int calories = specialMenu.stream().map(Dish::getCalories).reduce(0, Integer::sum);
~~~
- 박싱 비용이 숨어있다. (Integer > int)
- 스트림 API 숫자 스트림을 효율적으로 처리할 수 잇도록 기본형 특화 스트림을 제공한다. 

### 5.7.1 기본형 특화 스트림
- int 요소에 특화된 IntStream
- double 요소에 특화된 DoubleStream
- long 요소에 특화된 LongStream
- 각가의 인터페이스는 숫자 스트림의 합계를 계산하는 sum 등 자주 사용하는 숫자 관련 리듀싱 연산 수행 메서드 제공한다.
- 필요할 때 다시 객체 스트림으로 복원하는 기능도 제공한다.
- 특화 스트림은 오직 박싱 과정에서 일어나느 효율성과 관련이 있으며, 추가 기능을 제공하지 않는다!

### 숫자 스트림으로 매핑
- 스트림을 특화 스트림으로 변환할 때는 mapToInt, mapToDouble, mapToLong 세가지를 많이 사용한다.

~~~java
int caloriesToInt = specialMenu.stream()
                                .mapToInt(Dish::getCalories) // IntStream 반환
                                .sum();
~~~
- 스트림이 비어 있으면 sum 은 기본값 0을 반환한다.
- IntStream 은 max, min, average 등 다양한 유틸리티 메서드 지원한다.

### 객체 스트림으로 복원하기
- 정수가 아닌 dish 같은 다른 값으로 반환하고 싶은 경우는 어떻게 해야 할까?
- boxed 메서드를 이용하여 특화 스트림을 일반 스트림으로 변환할 수 있다. 

~~~java
IntStream intStream = specialMenu.stream().mapToInt(Dish::getCalories); // 스트림 > 숫자
Stream<Integer> stream = intStream.boxed();  // 숫자 > 스트림
~~~

### 기본값 : OptionalInt
- 스트림에 요소가 없는 상황과 실제 최댓값이 0인 상황을 어떻게 구별할 수 있을까?
- Optional 을 이용한다.
- Optional 을 Integer, String 등의 참조 형식으로 파라미터화할 수 있다.
- OptionaInt, OptionalDouble, OptionalLong 세 가지 기본형 특화 스트림 버전도 제공한다.

~~~java
OptionalInt maxCalories = specialMenu.stream().mapToInt(Dish::getCalories).max();
int max = maxCalories.orElse(1);
~~~

### 5.7.2 숫자 범위
- 특정 범위의 숫자를 하는 상황에서 자주 쓰인다.
- 예를 들어 1 ~ 100사이의 숫자 생성하려면 > IntStream과LongStream 에서는 range와 rangeClosed 정적 메서드 제공한다.
- 첫번째 인수 : 시작값, 두번째 인수 : 종료값
- range : 시작값과 종료값이 포함되지 않음 (초과 미만)
- rangeClosed : 시작값과 종료값이 포함 (이상 이하)
~~~java
IntStream evenNumbers = IntStream.range(1, 100).filter(n -> n % 2 == 0);
IntStream evenNumbersClosed = IntStream.rangeClosed(1, 100).filter(n -> n % 2 == 0);

System.out.println(evenNumbers.count());
System.out.println(evenNumbersClosed.count());
~~~
> 49   
> 50


### 5.7.3 숫자 스트림 활용 : 피타고라스 수
-> 힝.. 활용 안 배워도 되는디...

### 피다고라스 수
- a * a + b * b = c * c 공식을 만족하는 세 개의 정소 a,b,c가 존재하며, 직각 삼각형이다.
- 만약 세 수  중에서 a, b두 수만 제공했다고 가정하고, 피타고라스 수의 일부가 될 수 잇는 좋은 조합인지 확인하는 방법은?
~~~java
// 일단 a * a + b * b 의 제곱근이 정수인지 확인 할 수 있다.
Math.sqrt(a*a + b*b) % 1 == 0
>
filter(b -> Math.sqrt(a*a + b*b) % 1 == 0)
// 세 번째 수를 찾기
stream.filter(b-> Math.sqrt(a*a + b*b) % 1 == 0)
        .map(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)}); 
// b 값 생셩
IntStream.rangeClosed(1, 100)
        .filter(b-> Math.sqrt(a*a + b*b) % 1 == 0)
        .boxed()
        .map(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)}); 
>
IntStream.rangeClosed(1, 100)
        .filter(b-> Math.sqrt(a*a + b*b) % 1 == 0)
        .mapToObj(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)}); 
// a 값 생성
Stream<int[]> pytha = IntStream.rangeClosed(1, 100).boxed() // 1 ~ 100
                        .flatMap(a -> IntStream.rangeClosed(a, 100) // a > 1 ~ 100
                        .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0) 
                        .mapToObj(b -> new int[]{a, b, (int) Math.sqrt(a * a + b * b)}));
~~~
* 만약 rangeClosed (a, 100)이 아니라 (1, 100)으로 작성 했다면?
* (3, 4, 5), (4, 3, 5) 가 발생 된다.


## 5.8 스트림 만들기
- 스트림은 데이터 처리 질의를 표현하는 강력한 도구이다.
- 일련의 값, 배열, 파일, 심지어 함수를 이용한 무한 스트림 만들기 등 다양한 방식으로 스트림을 만드는 방법을 설명한다.

### 5.8.1 값으로 스트림 만들기
- 임의의 수를 인수로 받는 정적 메서드 Stream.of 를 이용해서 스트림을 만들 수 있다.
~~~java
Stream<String> stream = Stream.of("Modern ", "java ", "In ", "Action");
stream.map(String::toUpperCase).forEach(System.out::println);
~~~

### 5.8.2 null 이 될 수 있는 객체로 스트림 만들기
- null 이 될 수 있는 개체를 스트림으로 만들 수 있는 새로운 메소드가 추가되어있다.
- Stream.ofNullable

~~~java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = homeValue == null ? Stream.empty() : Stream.of(value);
>>
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
~~~

### 5.8.3 배열로 스트림 만들기 
- 배열을 인수로 받는 정적 메서드 Arrays.stream 을 이용해서 스트림을 만들 수 있다.
~~~java
int[] numbers = {2, 3, 5, 7, 11, 13};
int sum = Arrays.stream(numbers).sum();
~~~

### 5.8.4 파일로ㅑ 스트림 만들기
- java.nio.file.File의 많은 정적 메서드가 스트림을 반환한다.

~~~java
long uniqueWords = 0;
try (Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
    uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))) // 고유 단어 수 세기
        .distinct()  // 중복 제거
        .count();   // 단어 스트림 생성
} catch (IOException e) {
    
}
~~~

### 5.8.5 함수로 무한 스트림 만들기
- 스트림을 만들 수 있는 두 정적 메서드 Stream.iterate와 Stream.generate 를 제공한다.
- 두 연산을 이용하여 무한 스트림 즉, 크기가 고정되지 않은 스트림을 만들 수 있다. > 하지만 그러면 안 돼... 컴퓨터 죽어.. limit(n) 과 주로 사용

### iterate 메서드
- 요청할 때마다 값을 생산할 수 있으며 끝이 없으므로 무한 스트림을 만든다.
- 이러한 스트림을 언바운드 스트림이라고 표현한다. > 이것이 스트림과 컬렉션의 가장 큰 차이점이다.

- 0에서 시작하여 100보다 크면 숫사 생성 중단하는 코드
~~~java
// 틀린 방법 > filter 메소드는 언제 중단해야하는지 알 수 없다.
IntStream.iterate(0, n -> n + 4)
      .filter(n -> n < 100)
      .forEach(System.out::println);
~~~

~~~java
IntStream.iterate(0, n -> n < 100, n -> n + 4)
    .takeWhile(n -> n < 100)
    .forEach(System.out::println);
~~~


### generate 메서드
- 요구할 때 값을 계산하는 무한 스트림을 만들 수 있다.
- 생산된 각 값을 연속적으로 계산하지 않는다.
~~~java
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
~~~
- iterate 는 초기값과 다음 값을 생성하는 함수를 사용하여 순차적으로 값을 생성한다.
- generate 는 공급자를 사용하여 매번 새로운 값을 생성한다. 
따라서..
- iterate 는 특정 규칙에 따라 값을 생성할 때 유용하고
- generate 는 무작위 값이나 특정한 조건 없이 값을 생성할 때 유용하다.

## 5.9 마치며
- 스트릠 API를 이용하면 복잡한 데이터 처리 질의를 표한할 수 있다.
- filter, distinct, takeWhile, dropWhile, skip, limit 메소드로 스트림을 필터링하거나 자를 수 있다.
- map, flatMap 메서드로 스트림의 요소를 추출하거나 변환할 수 있다.
- findFirst, findAny  메서드로 스트림의 요소를 검색할 수 있다. allMatch, noneMatch, anyMatch 메서드를 이용해서 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색할 수 있다.
- 이들 메서드는 소트서킷, 즉 결과를 찾는 즉시 반환하며, 전체 스트림을 처리하지는 않는다.
- reduce 메서드로 스트림의 모든 요소를 반복 조합하며 값을 도출할 수 있다.
- filter, map 등은 상태를 저장하지 않는 상태 없는 연산 이다. reduce 같은 연산은 값을 계산하는데 필요한 상태를 저장한다. sorted, distinct 등의 메서드는 새로운 스트림을 반환하기에 앞서 스트림의 모든 요소를 버퍼에 저장해야 하므로, 상태 있는 연산이라고 부른다.
- IntStream, DoubleStream, LongStream 은 기본형 특화 스트림이다. 각각의 기본형에 맞에 특화되어 있다.
- 컬렉션뿐 아니라 값, 배열, 파일, iterate와 generate 같은 메서드로도 스트림을 만들 수 있다.
- 무한한 개수의 요소를 가진 스트림을 무한 스트림이라 한다.