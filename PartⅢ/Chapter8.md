# 컬렉션 API 개선

자바 8, 9 에서 추가된 새로운 컬렉션 API 기능을 배우는 장 (컬렉션 팩토리, 관용 패턴 등)

## 8.1 컬렉션 팩토리

자바에서 컬렉션 객체를 생성하는 간편한 방법을 제공함<br/>
불변 컬렉션을 생성할 수 있는 메서드가 추가됨 (리스트, 집합, 맵)

```Java
List<String> friends = new ArrayList<>();
friends.add("은양");
friends.add("동현");
friends.add("지현");
// 고작 세 개의 문자열 추가하는데 너무 많은 코드가 필요함
```

Arrays.asList() 팩토리 메서드를 이용해서 코드를 간단히 줄일 수 있음
```Java
List<String> friends = Arrays.asList("은양", "동현", "지현");
// 고정 크기이기 때문에 요소 갱신이 아닌 추가, 삭제를 할 경우 UnsupportedOperationException 발생함
// 데이터 안전성 보장, 가독성 향상, 불필요한 오류 방지
```
> `UnsupportedOperationException` : 지원되지 않는 요청이 들어왔을 때 발생하는 예외

### 8.1.1 리스트 팩토리
- `List.of`

```Java
List<String> friends = List.of("은양", "동현", "지현");
```

### 8.1.2 집합 팩토리
- `Set.of`

```Java
Set<String> friends = Set.of("은양", "동현", "지현");
```

### 8.1.3 맵 팩토리
- 열 개 이하의 작은 맵 : `Map.of`
- 그 이상 : `Map.ofEntries`

```Java
Map<String, Integer> ageOfFriends
    = Map.of("은양", 1, "동현", 2, "지현", 3);
```
```Java
Import static java.util.Map.entry;

Map<String, Integer> ageOfFriends
    = Map.ofEntries(entry("은양", 1),
                    entry("동현", 2),
                    entry("지현", 3));
```

## 8.2 리스트와 집합 처리
`removeIf`, `replaceAll`, `sort` 추가됨

### 8.2.1 removeIf 메서드

특정 조건을 만족하는 요소를 리스트나 집합에서 쉽게 제거할 수 있음
> List나 Set을 구현하거나 그 구현을 상속받은 모든 클래스에서 이용할 수 있음

```Java
list.removeIf(s -> s.startsWith("A"));
// 리스트 s에서 "A"로 시작하는 모든 요소를 제거함
```

```Java
for(Iterator<Transaction> iterator = transaction.iterator();
    iterator.hasNext();) {
        Transaction transation = iterator.next();
        if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
            iterator.remove();
        }
    }
// Iterator를 사용하여 transaction 컬렉션을 순회하면서
// 각 Transaction 객체의 참조 코드의 첫 번째 문자가 숫자인지 검사하고
// 조건에 맞는 경우 제거
// remove() 메서드로 안전하게 요소를 삭제할 수 있지만 코드가 복잡함

transaction.removeIf(transaction ->
    Character.isDigit(transaction.getReferenceCode().charAt(0)));
// removeIf 메서드를 사용해서 조건에 맞는 요소를 컬렉션에서 제거함
// 코드가 간결하고 직관적임
```

### 8.2.2 replaceAll 메서드

모든 요소를 특정 규칙에 따라 변환할 수 있음
> List에서 이용함

```Java
list.replaceAll(String::toUpperCase);
// 리스트의 모든 요소를 대문자로 변환함
```

```Java
for(ListIterator<String> iterator = referenceCodes.listIterator();
    iterator.hasNext();) {
        String code = iterator.next();
        iterator.set(Charactor.toUpperCase(code.charAt(0)) + code.substring(1));
    }
// ListIterator를 사용하여 직접 요소를 순회하면서 업데이트하는 방식

referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.substring(1));
// replaceAll 메서드를 사용하여 각 요소를 변환된 문자열로 대체함
```

## 8.3 맵 처리
### 8.3.1 forEach 메서드
```Java
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
    String friend = entry.getKey();
    Integer age = entry.getValue();
    System.out.println(friend + " is " + age + " years old");
}

ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " + age + " years old"));
```

### 8.3.2 정렬 메서드
```Java
Map<String, String> favouriteSingers
    = Map.ofEntries(entry("은양", "이창섭"),
                    entry("동현", "신류진"),
                    entry("지현", "도경수"));

favouriteSingers.entrySet()
                .stream()
                .sorted(Entry.comparingByKey())
                .forEachOrdered(System.out::println);
// 동현=신류진
// 은양=이창섭
// 지현=도경수
```

### 8.3.3 getOrDefault 메서드
찾으려는 키가 존재하지 않으면 기본값이 반환됨

```Java
Map<String, String> favouriteSingers
    = Map.ofEntries(entry("은양", "이창섭"),
                    entry("동현", "신류진"));

System.out.println(favouriteSingers.getOrDefault("동현", "황예지"));
// 신류진 -> "동현"이라는 키에 대한 값을 찾아 "신류진"을 출력함

System.out.println(favouriteSingers.getOrDefault("지현", "황예지"));
// 황예지 -> "지현"이라는 키에 대한 값을 찾지만 map에 존재하지 않으므로 기본값인 "황예지"를 출력함
```

### 8.3.4 계산 패턴
- `computeIfAbsent` : 주어진 키가 map에 존재하지 않을 경우에만 해당 키에 대한 값을 계산해서 추가함
```Java
Map<String, Integer> map = new HashMap<>();

map.computeIfAbsent("은양", k -> 1);
// "은양"이라는 키가 없으므로 1을 추가

map.computeIfAbsent("은양", k -> 2);
// "은양"이라는 키가 이미 있으므로 아무 일도 일어나지 않음

System.out.println(map);
// 은양=1
```

- `computeIfPresent` : 주어진 키가 map에 존재할 경우에만 해당 키에 대한 값을 계산해서 업데이트함
```Java
Map<String, Integer> map = new HashMap<>();
map.put("동현", 1);

map.computeIfPresent("동현", (k, v) -> v + 1);
// "동현"이라는 키가 있으므로 값을 2로 업데이트

map.computeIfPresent("지현", (k, v) -> v + 1);
// "지현"이라는 키가 없으므로 아무 일도 일어나지 않음

System.out.println(map);
// 동현=2
```

- `compute` : 키가 존재하지 않으면 새로 추가하고, 존재하면 업데이트함
```Java
Map<String, Integer> map = new HashMap<>();
map.put("은양", 1);

map.compute("은양", (k, v) -> v + 1);
// "은양"이라는 키가 있으므로 값을 2로 업데이트

map.compute("동현", (k, v) -> v == null ? 1 : v + 1);
// "동현"이라는 키가 없으므로 1을 추가

System.out.println(map);
// 은양=2
// 동현=1
```

### 8.3.5 삭제 패턴
- `remove`
```Java
favouriteSingers.remove(key, value);
```

### 8.3.6 교체 패턴
- `replaceAll` : map의 모든 값을 일괄적으로 업데이트함
```Java
Map<String, Integer> ageOfFriends = new HashMap<>();
ageOfFriends.put("은양", 1);
ageOfFriends.put("동현", 2);

ageOfFriends.replaceAll((friend, age) -> age + 10);
System.out.prinln(ageOfFriends);
// 은양=11
// 동현=12
```

- `replace`
  - `map.replace(key, newValue)` : 특정 키의 값을 직접 지정한 새로운 값으로 업데이트함
  - `map.replace(key, oldValue, newValue)` : 특정 키의 현재 값이 oldValue인 경우에만 새 값으로 업데이트함
```Java
Map<String, Integer> ageOfFriends = new HashMap<>();
ageOfFriends.put("은양", 1);
ageOfFriends.put("동현", 2);

ageOfFriends.replace("동현", 5);

System.out.println(ageOfFriends);
// 은양=1
// 동현=5
```

```Java
Map<String, Integer> ageOfFriends = new HashMap<>();
ageOfFriends.put("은양", 1);
ageOfFriends.put("동현", 2);

boolean replaced = ageOfFriends.replace("은양", 1, 3);

System.out.println(replaced);
// 은양=1이므로 true (replace 성공)
System.out.println(ageOfFriends);
// 은양=3
// 동현=2

replaced = ageOfFriends.replace("은양", 2, 4);

System.out.println(replaced);
// 은양=3이므로 false (replace 실패)
System.out.println(ageOfFriends);
// 은양=3
// 동현=2
```

### 8.3.7 합침
- `putAll` : 하나의 map의 모든 엔트리를 다른 map에 추가함. 기존의 키가 있는 경우 해당 키의 값이 업데이트됨
```Java
Map<String, Integer> ageOfFriends1 = new HashMap<>();
ageOfFriends1.put("은양", 1);
ageOfFriends1.put("동현", 2);

Map<String, Integer> ageOfFriends2 = new HashMap<>();
ageOfFriends2.put("동현", 3);
ageOfFriends2.put("지현", 4);

ageOfFriends1.putAll(ageOfFriends2);
// ageOfFriends1에 ageOfFriends2의 모든 엔트리를 추가함
// "동현"이라는 동일한 키가 존재하므로 value가 3으로 업데이트됨

System.out.prinln(ageOfFriends1);
// 은양=1
// 동현=3
// 지현=4
```

- `merge` : 키가 존재하지 않으면 새로 추가하고, 존재하면 지정된 함수를 사용해 값을 생성함
```Java
Map<String, Integer> ageOfFriends = new HashMap<>();
ageOfFriends.put("은양", 1);
ageOfFriends.put("동현", 2);

ageOfFriends.merge("은양", 3, Integer::sum);
// "은양"의 값은 1 + 3 = 4
ageOfFriends.merge("동현", 1, Integer::sum);
// "동현"의 값은 2 + 1 = 3
ageOfFriends.merge("지현", 5, Integer::sum);
// "지현"이라는 키가 없으므로 5가 생성됨

System.out.prinln(ageOfFriends1);
// 은양=4
// 동현=3
// 지현=5
```

## 8.4 개선된 ConcurrentHashMap
- 동시성 : 여러 스레드가 동시에 안전하게 읽고 쓸 수 있도록 설계됨 (데이터 일관성 유지)
- 세분화된 잠금 : 세그먼트 단위로 잠금을 관리하므로 동시에 여러 스레드가 다른 세그먼트에 접근 가능함

### 8.4.1 리듀스와 검색
병렬 처리를 활용하여 요소를 리듀스하거나 검색할 수 있는 기능을 제공함
- `reduce`, `search`, `forEach`

```Java
ConcurrentHashMap<String, Integer> ageOfFriends = new ConcurrentHashMap<>();
ageOfFriends.put("은양", 1);
ageOfFriends.put("동현", 2);
ageOfFriends.put("지현", 3);

int sum = ageOfFriends.reduceValues(1, Integer::sum);
// 1은 병렬 처리를 위한 스레드 수를 지정
System.out.println(sum);
// 6
```

```Java
int sum = ageOfFriends.values()
                        .stream()
                        .reduce(0, Integer::sum);
// ConcurrentHashMap에 reduce() 메서드는 제공되지 않아 reduceValues() 메서드를 사용함
// 단, 단일 스레드에서 값을 집계하고 싶다면 stream()으로 스트림을 생성하고 reduce()를 호출할 수 있음
```

### 8.4.2 계수
- `mappingCount` : ConcurrentHashMap에 저장된 키-값 쌍의 수를 long 타입으로 반환함
> 동시에 여러 스레드가 해당 map에 접근하더라도 정확한 결과를 제공함
```Java
ConcurrentHashMap<String, Integer> ageOfFriends = new ConcurrentHashMap<>();
ageOfFriends.put("은양", 1);
ageOfFriends.put("동현", 2);
ageOfFriends.put("지현", 3);

long count = ageOfFriends.mappingCount();
System.out.println(count);
// 3
```

### 8.4.3 집합뷰
- `keySet` : ConcurrentHashMap의 모든 키를 포함하는 집합(Set) 뷰를 반환함
> 맵의 상태가 변경될 때 자동으로 업데이트됨
```Java
ConcurrentHashMap<String, Integer> ageOfFriends = new ConcurrentHashMap<>();
ageOfFriends.put("은양", 1);
ageOfFriends.put("동현", 2);

Set<String> keys = ageOfFriends.keySet();
keys.forEach(System.out::println);
// 은양, 동현

ageOfFriends.put("지현", 3);
keys.forEach(System.out::println);
// 은양, 동현, 지현
```

- `newKeySet` : ConcurrentHashMap과 (독립적으로? 일방적으로?) 연결된 새로운 집합을 생성함
> newKeySet으로 생성된 집합에 키를 추가하면 원본 map에도 추가됨<br/>
> 그러나 newKeySet에 있는 키를 삭제해도 원본 map에서는 삭제되지 않음
```Java
ConcurrentHashMap<String, Integer> ageOfFriends = new ConcurrentHashMap<>();
ageOfFriends.put("은양", 1);
ageOfFriends.put("동현", 2);

Set<String> newKeys = map.newKeySet();
newKeys.add("지현");

System.out.println(ageOfFriends);
// 은양=1
// 동현=2
// 지현=null

newKeys.remove("지현");
// 삭제할 때 원본에 영향을 주지 않음

System.out.println(ageOfFriends);
// 은양=1
// 동현=2
// 지현=null
```

## 8.5 마치며
- 자바 9에서 도입된 컬렉션 팩토리 메서드는 불변 리스트, 집합, 맵을 쉽게 생성할 수 있게 하여 코드의 간결성과 안정성을 높임
- removeIf()와 replaceAll() 등의 메서드는 리스트와 집합을 보다 직관적으로 처리할 수 있게 함
- forEach(), merge(), compute() 등의 메서드는 맵의 요소를 처리하는 방법을 다양화함
- 개선된 ConcurrentHashMap은 동시성 문제를 효과적으로 해결하고, 멀티스레드 환경에서도 높은 성능을 유지함
