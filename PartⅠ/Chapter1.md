# 자바 8, 9, 10, 11 : 무슨 일이 일어나고 있는가?
자바는 1995년에 처음 출시된 이후, **객체지향 프로그래밍 언어**로서 널리 사용되고 있다.<br/> 
시간이 지나면서 자바는 다양한 버전을 통해 많은 기능이 추가되었다.

특히 자바 8은 `람다 표현식`과 `스트림 API`를 도입하여 프로그래밍 패러다임에 큰 변화를 가져왔다.<br/> 
객체지향 프로그래밍에 **함수형 프로그래밍**을 도입함으로써, 코드의 가독성과 유지보수성을 높였다.
> **함수형 프로그래밍** : 데이터를 변형하기보다는 함수를 조합하여 처리하는 방식

~~그냥 자바 8 찬양만 주구장창함~~

## 1. 메서드 참조 (method reference)
기존에 객체 참조(object reference)를 이용해서 객체를 주고받았던 것처럼 메서드 참조를 만들어 전달할 수 있게 되었다.

> (예시1)
> + 자바 8 이전의 방법
> ```
> File[] hiddenFiles = new File(".").listFiles(new FileFilter()) {
>     public boolean accept(File file) {
>         return file.isHidden();
>     }
> });
> 
> // File 클래스에서 제공하는 isHidden 메서드를 사용해서
> // FileFilter 객체 내부에 위치한 isHidden의 결과를
> // File.listFiles 메서드로 전달하는 방법으로 숨겨진 파일을 필터링
> ```
> 
> + 자바 8 이후의 방법
> ```
> File[] hiddenFiles = new File(".").listFiles(File::isHidden);
> 
> // 이미 isHidden이라는 함수가 준비되어 있으므로
> // 메서드 참조(::)를 이용해서 listFiles에 직접 전달
> ```

> (예시2)
> + 자바 8 이전의 방법
> ```
> public static List<Apple> filterGreenApples(List<Apple> inventory) {
>     List<Apple> result = new ArrayList<>();
>     for (Apple apple : inventory) {
>         if ("green".equals(apple.getColor())) {  // 녹색 사과만 필터
>             result.add(apple);
>       }
>     }
>     return result;
> }
> 
> public static List<Apple> filterHeavyApples(List<Apple> inventory) {
>     List<Apple> result = new ArrayList<>();
>     for (Apple apple : inventory) {
>         if (apple.getWeight() > 150) {  // 무게가 150 이상의 사과만 필터
>             result.add(apple);
>         }
>     }
>     return result;
> }
> 
> // 전체 코드 복사&붙여넣기 해서 필요 부분만 수정
> // -> 어떤 코드에 버그가 있다면 다 수정해야함ㅜ;;;;;
> ```
> 
> + 자바 8 이후의 방법
> ```
> public static boolean isGreenApple(Apple apple) {
>     return "green".equals(apple.getColor());
> }
> 
> public static boolean isHeavyApple(Apple apple) {
>     return apple.getWeight() > 150;
> }
> 
> public static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) {
>     List<Apple> result = new ArrayList<>();
>     for (Apple apple : inventory) {
>         if (p.test(apple)) {
>             result.add(apple);
>         }
>     }
>     return result;
> }
> 
> // filter 메서드를 중복으로 구현할 필요가 없기 때문에 이렇게 구현하고
> 
> List<Apple> greenApples = filterApples(inventory, Apple::isGreenApple);
> System.out.println(greenApples);
> 
> List<Apple> heavyApples = filterApples(inventory, Apple::isHeavyApple);
> System.out.println(heavyApples);
> 
> // 이렇게 호출
> ```

### 람다 (익명 함수)
자바 8 에서는 메서드 뿐 아니라 람다를 포함하여 함수도 값으로 취급할 수 있다.
> 람다식 : 함수형 프로그래밍을 구성하기 위한 함수식. 메서드를 하나의 식으로 표현한 것

```
List<Apple> greenApples2 = filterApples(inventory, (Apple a) -> "green".equals(a.getColor()));
System.out.println(greenApples2);

List<Apple> heavyApples2 = filterApples(inventory, (Apple a) -> a.getWeight() > 150);
System.out.println(heavyApples2);

List<Apple> weirdApples = filterApples(inventory, (Apple a) -> a.getWeight() < 80 || "brown".equals(a.getColor()));
System.out.println(weirdApples);

// 한 번만 사용할 메서드는 따로 구현할 필요없이 바로 호출
```

## 2. 스트림 (stream)
> 스트림 : 컬렉션, 배열 등에 대해 저장되어있는 요소들을 하나씩 참조하며 반복적인 처리가 가능하게 하는 기능<br/> 
> ( 불필요한 for문과 그 안에서 이루어지는 if문 등의 분기처리를 쓰지 않고도 깔끔하고 직관적인 코드로 변형 가능 )

+ 자바 8 이전의 방법
```
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();
for (Transaction transaction : transations) {
    if (transaction.getPrice() > 1000) {
        Currency currency = transaction.getCurrency();
        List<Transaction> transactionsForCurrency =
            transactionsByCurrencies.get(currency);
        if (transactionsForCurrency == null) {
            transactionsForCurrency = new ArrayList<>();
            transactionsBycurrencies.put(currency, transactionsForCurrency);
        }
    transactionsForCurrency.add(transaction);
    }
}

// for-each 문으로 각 요소를 반복하면서 작업을 수행하는 `외부 반복`

// 코드가 명시적이지만 길어지고 복잡해질 수 있음
// 병렬 처리를 구현하기 위해서 사용자가 명시적으로 스레드를 관리하고, 동기화를 신경써야 함
```

+ 자바 8 이후의 방법
```
import static java.util.stream.Collectors.groupingBy;

Map<Currency, List<Transaction>> transactionsByCurrencies =
    transaction.stream()
            .filter((Transaction t) -> t.getPrice() > 1000)
            .collect(groupingBy(Transaction::getCurrency));

// 라이브러리 내부에서 모든 데이터가 처리되는 `내부 반복`

// 코드의 간결성과 유연성을 제공
// 병렬 처리를 내부적으로 지원해서 간단히 병렬 처리를 활성화할 수 있음
// 핵심은 스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다는 것
```

## 3. 디폴트 메서드 (default method)
> 디폴트 메서드 : 인터페이스의 기본 구현을 그대로 상속하면서 자유롭게 새로운 메서드를 추가할 수 있는 기능<br/> 
> 인터페이스에 새로운 메서드를 추가하는 등 인터페이스를 바꾸고 싶을 때 유용

```
default void sort(Comparator<? super E> c) {
    Collections.sort(this.c);
}

// List 인터페이스의 sort 메서드가 자바 8에 새롭게 추가된 디폴트 메서드

// 기존 구현을 고치지 않고도 인터페이스를 바꿀 수 있으므로 java API의 호환성 유지 및 확장에 용이
```

## 4. Optional<T> 클래스
> Optional<T> : 값을 갖거나 갖지 않을 수 있는 컨테이너 객체<br/>
> Optional 클래스 : NullPointer 예외를 피할 수 있도록 도와주는 클래스
