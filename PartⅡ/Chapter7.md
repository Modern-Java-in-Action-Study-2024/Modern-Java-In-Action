# 병렬 데이터 처리와 성능

> 스트림으로 데이터 컬렉션 관련 동작을 얼마나 쉽게 병렬로 실행할 수 있는지 설명하는 장이다.

자바 7은 더 쉽게 병렬화를 수행하면서 에러를 최소화할 수 있도록 `포크/조인 프레임워크` 기능을 제공한다.  
스트림을 이용하면 순차 스트림을 병렬 스트림으로 자연스럽게 바꿀 수 있다.

## 7.1 병렬 스트림
> 병렬 스트림이란 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림이다.
* 컬렉션에서 parallelStream 을 호출하면 병렬 스트림이 생성된다.  
* 숫자 n을 인수로 받아서 1부터 n까지 모든 숫자의 합계를 반환하는 메서드 구현해보자.

~~~java
// 전통적인 자바에서 반복문 
public long iterativeSum(long n) {
    long result = 0;
    for (long i = 1L; i <= n; i++) {
        result += i;
    }
    return result;
}

// 무한 스트림을 만든 다음 인수로 주어진 크기로 스트림 제한하고, 두 숫자를 더하는 BinaryOperator 로 리듀싱 작업
public long sequentialSum(long n) {
    return Stream.iterate(1L, i -> i + 1).limit(n).reduce(0L, Long::sum);
}
~~~

특히 n이 커진다면 병렬로 처리하는 것이 좋을 것이다.
### 7.1.1 순차 스트림을 병렬 스트림으로 변환하기
* 순차 스트림에 parallel 메서드를 호출하면 기존의 함수형 리듀싱 연산이 병렬로 처리된다. 

~~~java
public long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1).limit(n).parallel.reduce(0L, Long::sum);
}
~~~

![image](https://hongchangsub.com/content/images/2023/07/DraggedImage.png)
- 리듀싱 연산을 여러 청크에 병렬로 수행할 수 있다.
- 사실 순차 스트림에 parallel 을 호출해도 스트림 자체에는 아무 변화도 일어나지 않는다.

### 7.1.2 스트림 성능 측정
- 실제 성능 측정하여 확인하자

~~~java
@State(Scope.Benchmark)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(java.util.concurrent.TimeUnit.MILLISECONDS)
@Fork(value = 2, jvmArgs = {"-Xms4G", "-Xmx4G"})
public class ParallelStreamBenchMark {
    private static final long N = 10_000_000L;
    
    @Benchmark
    public long sequentialSum() {
        return java.util.stream.Stream.iterate(1L, i -> i + 1)
            .limit(N)
            .reduce(0L, Long::sum);
    }
    
    @TearDown(Level.Invocation)
    public void tearDown() {
        System.gc();
    }
}
~~~

![image](https://github.com/user-attachments/assets/b857aa47-59d0-436b-99a2-281b6b918b4e)

~~~java
@Benchmark
public long iterativeSum() {
    long result = 0;
    for (long i = 1L; i <= N; i++) {
        result += i;
    }
    return result;
}
~~~

![image](https://github.com/user-attachments/assets/a704197a-9ca0-44f7-ae08-85f8b1c67236)

~~~java
@Benchmark
public long parallelSum() {
    return Stream.iterate(1L, i -> i + 1).limit(N).parallel().reduce(0L, Long::sum);
}
~~~

![image](https://github.com/user-attachments/assets/0487e867-b284-45fe-b850-b8ee60680326)
* 
* 병렬 버전이 순차 버전에 비해 느린 결과가 나온 이유는?
  * 반복 결과로 박싱된 객체가 만들어지므로 숫자를 더하려면 언박싱을 해야한다.
  * 반복 작업은 병렬로 수행할 수 있는 독립 단위로 나누기 어렵다.
* 리듀싱 과정을 시작하는 시점에 전체 숫자 리스트가 준비되지 않았으므로 스트림을 병렬로 처리할 수 있도록 청크로 분할할 수 없다.
* 병렬 처리되도록 구현했지만 순차적으로 동작하는 iterate()연산에 의해 순차적으로 스트림 요소를 추가하는 과정을 거친다.
* 순차적으로 완성된 스레드에서 멀티 스레드를 이용해 각각의 합계를 구한다.
* 하지만, 순차 스트림에 여러 스레드를 할당하는 오버헤드만 발생한다.
* 즉, parallel 내부 동작을 알고 사용해야 한다.

### 더 특화된 메서드 사용
* LongStream.rangeClosed 는 기본형 long을 직접 사용하므로 박싱/언박싱 오버헤드가 사라진다.
* LongStream.rangeClosed 는 쉽게 청크로 분할할 수 있는 숫자 범위를 생산한댜.

LongStream.rangeClosed 를 이용하여 박싱/언박 비용을 줄여보자.
~~~java
@Benchmark
public long rangeSum() {
    return LongStream.rangeClosed(1, N).reduce(0L, Long::sum);
}
~~~

![image](https://github.com/user-attachments/assets/91425a70-0655-43b2-bbc9-72bb5388d152)

- 올바른 자료구조를 선택해야 병렬 실행도 최적의 성능을 발휘한다.

### 7.1.3 병렬 스트림의 올바른 사용법
- 병렬 스트림을 잘못 사용하면서 발생하는 많은 문제는 > 공유된 상태를 바꾸는 알고리즘을 사용하기 때문에 일어난다.
~~~java
public long sideEffectSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1, n).forEach(accumulator::add);
    return accumulator.total;
}

public class Accumulator {
    public long total = 0;
    public void add(long value) {
        total += value;
    }
}
~~~
- 즉, 누적자를 초기화 하고, 리스트의 각 요소를 하나씩 탐색하면서 누적자에 숫자를 추가할 수 있다.
- 순차 실행하도록 구현되어 있으므로 병렬로 실행시 참사가 일어난다.
- total 을 접근할 때마다 데이터 레이스 문제가 발생한다.
  - 데이터 레이스 : 멀티 스레딩 환경에서 여러 스레드가 동시에 공유된 데이터를 수정하려고 할 때 발생

### 7.1.4 병렬 스트림 효과적으로 사용하기
- 확신이 서지 않으면 직접 측정하라.
  - 순차 스트림 > 병렬 스트림 변경을 쉽기 때문에 직접 성능을 측정하는 것이 바람직하다.
- 박싱을 주의하라
  - 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있다.
  - 되도록이면 기본형 특화 스트림을 사옹하는 것이 좋다.
- 병렬 스트림에서 성능이 떨어지는 연산이 있다.
  - limit, findFirst 처럼 요소의 순서에 의존하는 연산을 병렬 스트림에서 성능이 더 떨어진다.
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하자.
  - 요소 수가 많고 요소 당 연산 비용이 높다면 병렬 스트림으로 성능 개선할 여지가 있다.
  - 소량의 데이터에서는 병렬 스트림이 도움되지 않는다.
- 스트림을 구성하는 자료구조가 적절한지 확인한다.
- 스트림의 특성과 파이프라인 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.
  - SIZED 스트림은 정확히 같은 두 스트림으로 분할할 수 있으므로 효과적으로 병렬처리가 가능하다.
- 최종 연산의 병합 과정 비용이 비싸다면 병렬 스트림으로 얻은 이익이 상쇄될 수 있다.

## 7.2 포크/조인 프레임 워크
- 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할하여 서브태스크로 처리한 뒤, 각각의 결과를 합쳐서 전체 결과로 만드는 방식이다.

### 7.2.1 Recursive Task 활용
- 스레드 풀을 이용하려면 RecursiveTask<R>의 서브클래스를 만들어야 하고, 추상메서드 compute 를 구현해야 한다.
~~~java
protected abstract R compute();
~~~
- compute 메서드 구현 형식은 분할 정복 알고리즘의 병렬화 버전을 사용한다.
~~~java
if(태스크가 충분히 작거나 더이상 분할할 수 없으면) {
  순차적으로 태스크 계산
} else {
  태스크를두 서브태스크로 분할
  태스크가 다시 서브태스크로 분할되도록 메시지를 재귀적으로 호출
  모든 서브태스크의 연산이 왑료될때까지 대기
  각 서브태스크의 결과를 합침
}
~~~


### 7.2.2 포크/조인 프레임워크 제대로 사용하는 방법
- join 메서드를 태스크에 호출하면 태스크가 생산하는 결과가 준비될 때까지 호출자를 블록시킨다. 따라서 두 서브태스크가 모두 시작된 다음에 join을 호출하지 않으면, 각각의 서브태스크가 다른 서브태스크를 기다리는 일이 발생할 수 있다.
- RecursiveTask 내에서는 compute나 fork 메서드를 사용하며, 순차코드에서 병렬 계산을 시작할때만 ForkJoinPool의 invoke 메서드를 사용해야 한다.
- 서브태스크에 fork 메서드를 호출해서 ForkJoinPool의 일정을 조절할 수 있다. 한쪽 작업에만 fork를 호출하고 다른쪽에는 compute를 호출하면, 한 태스크에는 같은 스레드를 재사용할 수 있으므로 불필요한 오버헤드를 피할 수 있다.
- 포크/조인 프레임워크의 병렬 계산은 디버깅하기 어렵다. fork라 불리는 스레드에서 compute를 호출하므로 스택 트레이스가 도움이 되지 않는다.
- 병렬 처리로 성능을 개선하려면 태스크를 여러 독립적인 서브태스크로 분할할 수 있어야 하며, 각 서브태스크의 실행 시간은 새로운 태스크를 포킹하느데 드는 시간보다 길어야한다.

포크/조인 분할 전략에서는 주어진 서브테스크를 더 분할할 것인지 결정할 기준을 정해야 한다.

### 7.2.3 작업 훔치기
- 이론적으로는 CPU 코어 개수만큼 병렬화된 태스크로 작업 부하를 분할하면 모든 코어에서 태스크를 실행할 것이고, 같은 시간에 종료될 것이라고 생각할 수 있다. 
- 하지만 다양한 이유로 각각의 서브태스크의 작업 완료 시간이 크게 달라질 수 있다.
- `포크/조인 프레임워크` 에서는 `작업 훔치기` 라는 기법으로 이 문제를 해결한다.
  - ForJoinPool 의 모든 스레드를 거의 공정하게 분할한다.
  - 각각의 스레드는 자신에게 할당된 작업이 끝날때마다 다른 스레드의 큐에서 작업을 가져와 처리한다.
  - 태스크의 크기를 작게 나누어야 스레드 간의 작업 부하를 비슷한 수준으로 유지할 수 있다.

### 7.3 Spliterator 인터페이스
Spliterator
- 분할할 수 있는 반복자
- iterator 처럼 소스의 요소 탐색기능을 제공하며, 병렬 작업에 특화되어 있다.
~~~java
public interface Spliterator<T> {
  boolean tryAdvance(Consumer<? super T> action);
  Spliterator<T> trySplit();
  long estimateSize();
  int characteristics();
}
~~~
- tryAdvance : Spliterator 의 요소를 순차적으로 소비하면서 탐색해야할 요소가 있으면 참을 반환
- trySplit : Spliterator 의 일부 요소를 분할해 두 번째 Spliterator 를 생성하는 메서드
- estimateSize : 탐색해야할 요소 수 정보 제공
- characteristics : Spliterator 의 특성을 정의

### 7.3.1 분할 과정
- 스트림을 여러 스트림으로 분할하는 과정은 재귀적으로 일어난다.  
![img_3.png](img_3.png)
- tyrSplit 의 결과가 null 이 될 때까지 과정을 반복한다.
- null 을 반환했다는 것은 더 이상 자료구조를 분할할 수 없음을 의미한다.

### Spliterator 특성
- Spliterator 의 특성
- Characteristics 추상 메서드로 정의하며, Spliterator 자체 특성 집합을 포함하는 int 를 반환한다.

| 특성         | 의미                                                              |
|------------|-----------------------------------------------------------------|
| ORDERED    | 리스트처럼 요소에 정해진 순서가 있으므로 Spliterator 는 요소를 탐색하고 분해할 때 순서에 유의 해야 함 |
| DISTINCT   | x, y 두 요소를 방문했을 때 x.equals(y) 는 항상 false 반환                     |
| SORTED     | 탐색된 요소는 미리 정의된 정렬 순서를 따름                                        |
| SIZED      | 크기가 알려진 소스로 Spliterator 를 생성했으므로 estimatedSized() 는 정확한 값을 반환   |
| NON-NULL   | 탐색하는 모든 요소는 null 이 아님                                           |
| IMMUTABLE  | 이 Spliterator 의 소스는 불변. 요소를 탐색하는 동안 요소 추가하거나 삭제하거나, 고칠 수 없다.    |
| CONCRUUENT | 동기화 없이 Spliterator 의 소스를 여러 스레드에서 동시에 고칠 수 없다.                  |
| SUBSIZED   | 이 Spliterator 그리고 분할되는 모든 Spliterator 는 SIZED 특성을 갖는다.          |
- 특성에 해당하는 상수
- Spliterator 가 데이터 소스를 탐색하거나 분할할 때 이 특성의 유무에 따라 동작 방식이 달라진다.
- characteristics() 메서드를 사용하여 특성을 확인할 수 있다. 

### 7.3.2 커스텀 Spliterator 구현하기
- 반복형으로 단어 수를 세는 메서드
~~~java
public int countWordsIteratively(String s) {
    int counter = 0;
    boolean lastSpace = true;
    for (char c : s.toCharArray()) { //문자열의 모든 문자를 하나씩 탐색
        if (Character.isWhitespace(c)) {
            lastSpace = true;
        } else {
            if (lastSpace) counter++; //공백문자 탐색 시 이전까지의 문자를 단어를 간주하여 단어 수 counting
            lastSpace = false;
        }
    }
    return counter;
}
~~~

- 함수형으로 단어 수를 세는 메서드
~~~java
class WordCounter {
	private final int counter; // 단어의 개수 
	private final boolean lastSpace; // 마지막으로 처리한 문자가 공백인지 여부 

	public WordCounter(int counter, boolean lastSpace) {
		this.counter = counter;
		this.lastSpace = lastSpace;
	}

	public WordCounter accumulate(Character c) {
		if (Character.isWhitespace(c)) { // 입력 문자가 공백인 경우
			return lastSpace ? this : new WordCounter(counter, true); // 이전 문자가가 공백이면 상태를 유지 : 공백이 아니면 true 로 변경
		} else { // 입력 문자가 공백이 아닌 경우
			return lastSpace ? new WordCounter(counter+1, false) : this; // 이전 문자가 공백이면 counter +1 + lastSpace 를 false 로 변경 : 공백이 아니면 상태 유지
		}
	}

	public WordCounter combine(WordCounter wordCounter) { // 병렬 처리시 여러 조각에서 계산된 결과를 합칠 때 사용
		return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
		// 현재 객체에서 계산된 단어의 수 + 다른 스레드에서 계산된 단어의 수, 마지막 처리된 문자가 공백인지 확인
	}

	public int getCounter() {
		return counter;
	}
}
~~~
- 스트림을 탐색하며 새로운 문자를 찾을 때마다 accumulate 를 호출
- accumulate 메서드는 새로운 WordCounter 클래스를 어떤 상태로 생성할 것인지 정의
- combine 은 문자열 서브스트림을 처리한 WordCounter 의 결과를 합한다.

~~~java
private int countWord(Stream<Character> stream) {
  WordCounter wordCounter = stream.reduce(
    new WordCounter(0, true),
        WordCounter::accumulate,
        WordCounter::combine);
  return wordCounter.getCounter(); 
}
~~~
- 문자열 스트림의 리듀싱 연산으로 변경한 모습

- 원하는 결과가 나오지 않음
- 순차 스트림을 병렬 스트림으로 바꿀 때 스트림 분할 위치에 따라 잘못된 결과가 나온다.
- 문자열을 임의의 위치에서 분할하지 말고 단어가 끝나는 위치에서만 분할하는 방법으로 문제 해결 가능
### WordCounter 병렬로 수행하기

~~~java
class WordCounterSpliterator implements Spliterator<Character> { // Spliterator 를 구현한 클래스, 주어진 문자열을 문자단위로 처리하고, 병렬로 처리할 수 있도록 분할한다.

  public static final String SENTENCE =
          " Nel   mezzo del cammin  di nostra  vita "
                  + "mi  ritrovai in una  selva oscura"
                  + " che la  dritta via era   smarrita ";

  private final String string;
  // 현재 문자열에서 어떤 문자를 처리할지
  private int currentChar = 0;

  WordCounterSpliterator(String string) {
    this.string = string;
  }

  @Override
  public boolean tryAdvance(Consumer<? super Character> action) { // 문자열에서 한 문자를 소비하고, 소비할 문자가 남으면 true 반환
    //현재 문자를 소비
    action.accept(string.charAt(currentChar++));
    //소비할 문자가 남아있으면 true 반환
    return currentChar < string.length();
  }

  @Override
  public Spliterator<Character> trySplit() { // 문자열 두 개의 Spliterator 로 분할할 수 있는지 결정
    int currentSize = string.length() - currentChar;
    //파싱할 문자열을 순차 처리할 수 있을만큼 충분히 작아졌음(10미만)을 알리는 null 반환
    if (currentSize < 10) {
      return null;
    }
    //파싱할 문자열의 중간을 분할 위치로 설정
    for (int splitPos = currentSize / 2 + currentChar; splitPos < string.length(); splitPos++) {
      //다음 공백이 나올 때까지 분할 위치를 뒤로 이동시킴
      if (Character.isWhitespace(string.charAt(splitPos))) {
        //처음부터 분할 위치까지 문자열을 파싱할 새로운 WordCounterSpliterator를 생성
        Spliterator<Character> spliterator = new WordCounterSpliterator(string.substring(currentChar, splitPos));
        //이 WordCounterSpliterator의 시작 위치를 분할 위치로 설정
        currentChar = splitPos;
        //공백을 찾았고 문자열을 분리했으므로 루프를 종료
        return spliterator;
      }
    }
    return null;
  }

  //탐색해야 할 요소의 개수
  @Override
  public long estimateSize() { // 남은 문자수 반환
    //파싱할 문자열 전체 길이 - 현재 반복중인 위치
    return string.length() - currentChar;
  }

  @Override
  public int characteristics() {
        /*
          ORDERED: 문자열의 문자 등장 순서가 유의미함
          SIZED: estimatedSize 메서드의 반환값이 정확함
          SUBSIZED: trySplit으로 생성된 Spliterator도 정확한 크기를 가짐
          NONNULL: 문자열에는 null 문자가 존재하지 않음
          IMMUTABLE: 문자열 자체가 불변 클래스이므로 문자열을 파싱하면서 속성이 추가되지 않음
        */
    return ORDERED + SIZED + SUBSIZED + NONNULL + IMMUTABLE;
  }

  public static void main(String[] args) {
    System.out.println("Found " + countWords(SENTENCE) + " words");
  }

  public static int countWords(String s) {  // 주어진 문자열에서 단어의 개수를 계산
    Spliterator<Character> spliterator = new WordCounterSpliterator(s);
    // true는 병렬 스트림 생성 여부를 지시
    Stream<Character> stream = StreamSupport.stream(spliterator, true);

    // 단어 수를 세는 로직을 추가
    // 병렬 처리를 위해 상태 저장하고, 문자 단위로 처리된 결과를 합칠 수 있게 함
    WordCounter wordCounter = stream
            .reduce(new WordCounter(0, true), WordCounter::accumulate, WordCounter::combine);

    return wordCounter.getCounter();
  }
}
~~~


## 7.4 마치며
- 내부 반복을 이용하면 명시적으로 다른 스레드를 사용하지 않고도 스크림을 병렬로 처리할 수 있다.
- 항상 병렬 처리가 빠른건 아니므로 성능을 직접 측정해야 한다.
