# null 대신 Optional 클래스

> null 때문에 수 많은 문제들이 발생했다.

## 11.1 값이 없는 상황을 어떻게 처리할까?

~~~java
class Person {
	private Car car;
	public Car getCar() {
		return car;
	}
}

class Car {
	private Insurance insurance;
	public Insurance getInsurance() {
		return insurance;
	}
}

class Insurance {
	private String name;
	public String getName() {
		return name;
	}
}

public class Example11 {
	public String getCarInsuranceName(Person person) {
		return person.getCar().getInsurance().getName();
	}
}
~~~
- 만약 차를 소유하지 않은 사람을 호출한다면? > NullPointerException 발생  

### 11.1.1 보수적인 자세로 NullPointerException 줄이기
- NullPointerException 을 피하기 위해 null 확인 코드 추가 하는 방법
~~~java
// 가장 간단한 방법
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }

    }
    return "Unknown";
}
~~~
- 변수를 참조할 때마다 null 을 확인
- 우리가 확실히 알고 있는 영역을 모델링 할 때에는 null 확인을 생략할 수 있지만, 아닌 경우도 존재
- null 인지 확인할 때마다 중첩된 if가 추가되어 코드 들여쓰는 수준이 증가 > 이와 같은 반복 패턴 코드를 `깊은 의심` 이라고 함
  - 코드의 구조가 엉망
  - 가독성이 떨어짐
~~~java
// 들여쓰기를 줄이기 위한 발악
public String getCarInsuranceName(Person person) {
    if (person == null) {
        return "Unknown";
    }
    Car car = person.getCar();
    if (car == null) {
        return "Unknown";
    }
    
    Insurance insurance = car.getInsurance();
    if (insurance == null) {
        return "Unknown";
    }
    return insurance.getName();
}
~~~
- 중첩 if 블록을 없앤 코드
- if 를 여러번 사용하여 네 개의 출구가 생김
  - 유지보수가 어려워짐
- null 일 때 반환되는 기본값 "Unknown"이 반복

### 11.1.2 null 때문에 발생하는 문제
null 은...
- 에러의 근원이다
- 코드를 어지럽힌다
- 아무 의미 없다
- 자바 철학에 위배된다
- 형식 시스템에 구멍을 만든다

### 11.1.3 다른 언어는 null 대신 무얼 사용하나?
- 그루비 
  - 안전 내비게이션 연산자 `?.`
  - 예) def carInsuranceName = person?.car?.insurance?.name
  - 호출 할 때 null 인 참조가 있으면 결과로 null 이 반환된다.
- 함수형 언어(하스켈, 스칼라 등)
  - 하스켈
    - 선택형값을 저장할 수 있는 형식 `Maybe`
    - 주어진 형식의 값을 갖거나 아니면 아무 값도 갖지 않을 수 있다.
  - 스칼라
    - T 형식의 값을 갖거나 아무 값도 갖지 않을 수 있다. `Option[T]`
    - Option 형식에서 제공하는 연산을 사용하여 값의 여부를 명시적으로 확인해야 한다.

## 11.2 Optional 클래스 소개
- 하스켈과 스칼라의 영향을 받아 java.util.Optional<T> 라는 새로운 클래스 제공
- Optional : 선택형값을 캡슐화하는 클래스
  - 값이 있으면 Optional 클래스는 값을 감싼다
  - 값이 없으면 Optional.empty() 메서드로 Optional 을 반환한다.
    - null 참조 > NullPointerException 발생
    - Optional.empty()는 Optional 객체이므로 다양한 방식으로 활용 가능
~~~java
class Person {
	private Optional<Car> car;
	public Optional<Car> getCar() {
		return car;
	}	
}
class Car {
	private Optional<Insurance> insurance;
	public Optional<Insurance> getInsurance() {
		return insurance;
	}
}

class Insurance {
	private String name;
	public String getName() {
		return name;
	}
}
~~~
- Optional 클래스를 이용하면, 값이 있을 수도 없을 수도 있다는 정보를 줌
- Insurance 는 String 을 사용하여 반드시 이름을 가져야 함을 보여줌
- Optional 을 사용하면 값이 없는 상황이 데이터의 문제인지, 아니면 알고리즘의 버그인지 명확하게 구분 가능하다.
  - 그러나, 모든 null 참조를 Optional 로 대치하는 것은 바람직하지 않다.
  - 메서드의 시그니처만 보고도 선택형값인지 여부를 구별할 수 있다.

## 11.3 Optional 적용 패턴
- Optional 활용 방법

### 11.3.1 Optional 객체 만들기
- 빈 Optional
  - 정적 팩토리 메서드 Optional.empty 로 빈 Optional 객체 만들 수 있다.
  - Optional<Car> optCar = Optional.empty();
- null 이 아닌 값으로 Optional 만들기
  - 정적 팩토리 메서드 Optional.of 로 null 이 아닌 값을 포함하는 Optional 을 만들 수 있다.
  - Optional<Car> optCar = Optional.of(car);
  - car 가 null 인 경우 즉시 NullPointerException 발생한다.
    - Optional을 사용하지 않았다면, car의 속성에 접근할 때 에러가 발생했을 것이다.
- null 값으로 Optional 만들기
  - 정적 팩토리 메서드 Optional.ofNullable 로 null 값을 저장할 수 있는 Optional 을 만들 수 있다.
  - Optional<Car> optCar = Optional.ofNullable(car);
  - car 가 null 이면 빈 Optional 객체 반환

### 11.3.2 맵으로 Optional 의 값을 추출하고 변환하기
- Optional 이 비어있으면, get을 호출할 때 예외가 발생한다 > null 을 사용했을 때와 같은 문제를 만난다.
- 객체의 정보 추출시 Optional 을 사용할 때가 많다.
~~~java
//보험회사의 이름을 추출
String name = null;
if(insurance != null) {
	name = insurance.getName();
}

// >> Optional의 map 메서드를 사용할 수 있다. 
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
~~~
- Optional 이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾼다
- Optional 이 비어있으면 아무일도 일어나지 않는다.
- 여러 메서드를 안전하게 호출하려면?

### 11.3.3 flatMap 으로 Optional 객체 연결
~~~java
Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
Optional<String> name = optInsurance.map(Insurance::getName);
// >> map 을 이용하여 코드 재구성
Optional<Person> optPerson = Optional.of(person);
Optional<String> name = optPerson // Optional<Person>
    .map(Person::getCar) // 여기서 Optional<Car>를 반환하므로 결과는 Optional<Optional<Car>>가 됨
    .map(Car::getInsurance)
    .map(Insurance::getName);
~~~
- 컴파일 되지 않는다.
- getCar >> Optional<Car> 형식을 반환
- map 연산 결과 >> Optional<Optional<Car>>
- 중첩 Optional 객체 구조

### Optional 의 중첩을 없애기 위해 flatMap 사용
~~~java
public String getCarInsuranceName(Optional<Person> person){
  return person
          .flatMap(Person::getCar) // Optional<Person>에서 Optional<Car>로 변환
          .flatMap(Car::getInsurance) // Optional<Car>에서 Optional<Insurance>로 변환
          .map(Insurance::getName) // Insurance.getName() 은 String을 반환하므로 flatMap 사용할 필요 없음
          .orElse("Unknown"); // 결과 Optional 이 비었으면 기본값 사용
}
~~~

### 11.3.4 Optional 스트림 조작
- 자바 9에서는 Optional 을 포함하는 스트림 처리를 지원한다.
  - 스트림 요소를 조작하려면 변환, 필터 등 일련의 긴 체인이 필요한데, Optional 로 값이 감싸져있으면 과정이 조금 더 복잡해진다.
~~~java
public Set<String> getCarInsuranceNames(List<Person> persons) {
  return persons.stream()
    .map(Person::getCar) //Stream<Optional<Car>>
    .map(optCar -> optCar.flatMap(Car::getInsurance)) //Stream<Optional<Insurance>>
    .map(optIns -> OptIns.map(Insurance::getName)) //Stream<Optional<String>>
    .flatMap(Optional::stream) //Stream<String>
    .collect(toSet()); //Set<String>
}
~~~

### 11.3.5 디폴트 액션과 Optional 언랩
- Optional 클래스는 Optional 인스턴스에 포함횐 값을 읽는 다양한 방법을 제공한다.
  - get() : 값을 읽는 가장 간단하면서, 가장 안전하지 않은 메서드이다. 래핑된 값이 있으면 해당 값을 반환하고, 없으면 NoSuchElementException을 발생시킨없.
  - orElse(T other) : Optional 이 값을 포함하지 않을 때 기본 값을 제공할 수 있다.
  - orElseGet(Supplier<? extends T> other) : orElse 메서드의 게으른 버전으로 Optional 에 값이 없을 때만 Supplier가 실행된다.
  - orElseThrow(Supplier<? extends X> exceptionSupplier) : Optional 이 비어있을 때 지정한 예외를 발생시킨다.
  - ifPresent(Consumer<? super T> consumer) : 값이 존재할 때 인수로 넘겨준 동작을 실행할 수 있다.
  - ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) : Optional이 비어있을 때만 실행할 수 있는 Runnable을 인수로 받는다.

### 11.3.6 두 Optional 합치기
~~~java
public Insurance findCheapestInsurance(Person person, Car car) {
  ...
  return cheapestCompany;
}

public Optional<Insurance> nullSafeFindCheapestInsurance(
    Optional<Person> person, Optional<Car> car) {
  if (person.isPresent() && car.isPresent()) {
    return Optional.of(findCheapestInsurance(person.get(), car.get()));
  } else {
    return Optional.empty();
  }
}
~~~
- 두 Optional 을 인수로 받아서 둘 중 하나라도 비어 있으면 빈 Optional<Insurance>를 반환한다.
- person 과 car 의 시그니처만으로 둘 다 아무 값도 반환하지 않을 수 있다는 정보를 명시적으로 보여준다.
- 구현 코드는 null 확인 코드와 크게 다르지 않다.

~~~java
public Optional<Insutrance> nullSafeFindCheapestInsurance(
    Optional<Person> person, Optional<Car> car) {
  return person.flatMap( p -> car.map( c -> findCheapestInsurance(p, c)));
}
~~~
- 한 줄의 코드로 재구현
- Optional에 flatMap을 호출했으므로 첫 번째 Optional이 비어있다면 인수로 전달된 표현식이 실행되지 않고 빈 Optional을 반환한다.

### 11.3.7 필터로 특정값 거르기
~~~java
// 보험 회사 이름이 "CambridgeInsurance"인지 확인
Insurance insurance = ...;
if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){
  System.out.println("ok");
}
~~~

~~~java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
  .ifPresent(x -> System.out.println("ok"));
~~~

- Optional 클래스의 메서드

| 특성              | 의미                                                                       |
|-----------------|--------------------------------------------------------------------------|
| empty           | 빈 Optional 인스턴스 반환                                                       |
| filter          | 값이 존재하며 프레디케이트와 일치하면 값을 포함하는 Optional을 반환하고, 없거나 일치 하지 않으면 빈 Optinal을 반환 |
| flatMap         | 값이 존재하면 인수로 제공된 함수를 적용한 결과 Optional을 반환하고, 없으면 빈 Optional을 반환            |
| get             | 값이 존재하면 Optional을 감싸고 있는 값을 반환 없으면 NoSuchElementException 발생             |
| ifPresent       | 값이 존재하면 저장된 Consumer 실행, 없으면 아무일도 일어나지 않음                                |
| ifPresentOrElse | 값이 존재하면 저장된 Consumer 실행, 없으면 아무일도 일어나지 않음                                |
| isPresent       | 값이 존재하면 true 반환하고, 없으면 false 반환                                          |
| map             | 값이 존재하면 제공된 매핑 함수 적용                                                     |
| of              | 값이 존재하면 값을 감싸는 Optional을 반환하고, 값이 null 이면 nullPointerException 발생        |
| ofNullable      | 값이 존재하면 값을 감싸는 Optional을 반환하고, 값이 null 이면 빈 Optional 반환                  |
| or              | 값이 존재하면 같은 Optional을 반환하고, 없으면 Supplier에서 만든 Optional을 반환                |
| orElse          | 값이 존재하면 값을 반환하고, 없으면 기본값 반환                                              |
| orElseGet       | 값이 존재하면 값을 반환하고, 없으면 Supplier에서 제공하는 값을 반환                               |
| orElseThrow     | 값이 존재하면 값을 반환하고, 없으면 Supplier에서 생성한 예외 발생                                |
| stream          | 값이 존재하면 존재하는 값만 포함하는 스트림을 반환하고, 없으면 빈 스트림 반환                             |



## 11.4 Optional 을 사용한 실용 예제

### 11.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기
- 참조하는 객체에 대하여 null 이 될 수 있는 경우가 있다면 Optional 로 감싸서 개선할 수 있다.

### 11.4.2 예외와 Optional 클래스
- null 을 확인할 때는 if 문을 사용하지만, 예외를 발생시키는 메서드는 `try/catch 블록`을 사용해야 한다.
~~~java
public static Optional<Integer> stringToInt(String s){
    try{
        return Optional.of(Integer.parseInt(s));
    catch(NumberformatException e){
        return Optional.empty();
    }
}
~~~

### 11.4.3 기본형 Optional 을 사용하지 말아야 하는 이유
- Optional도 기본형으로 특화된 OptionalInt, OptionalLong, OptionalDouble 등의 클래스를 제공한다.
- 기본형 특화 Optional은 map, flatMap, filter등을 제공하지 않고, 생성한 결과를 다른 일반 Optional과 혼용할 수 없다.

## 11.5 마치며
- 자바 8에서는 값이 있거나 없음을 표한할 수 있는 클래스 java.util.Optional<T> 제공
- 팩토리 메서드 Optional.empty, Optional.of, Optional.ofNullable 등을 이용하여 Optional 객체 만들 수 있다.
- Optional 클래스는 스트림과 비슷한 연산 수행하는 map, flatMap, filter 등의 메서드를 제공한다.
- Optional 로 값이 없는 상황을 적절하게 처리하도록 강제할 수 있다.
- 메서드의 시그니처만 보고도 Optional 값이 사용되거나 반환되는지 예측 가능하다.