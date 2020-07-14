모던 자바 인 액션 Part 3
------------------------

#### Chap 8 : 컬렉션 API 개선

-	이 챕터에서는 자바 8,9에서 추가된 새로운 컬렉션 API의 기능에 대해 소개한다.

##### 8.1 컬렉션 팩토리

-	작은 컬렉션 객체를 쉽게 만들 수 있는 몇 가지 방법을 제공한다.

```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");


// Arrays.asList() 팩토리 메서드를 이용하면 코드를 줄일 수 있음
List<String> friends2 = Arrays.asList("Raphael", "Olivia", "Thibaut");

// 고정된 리스트 이기 때문에 요소를 업데이트하는 건 괜찮지만, 요소를 추가하거나 삭제를 하진 못한다
// UnsupportedOperationExcetion!
friends.add("addFriend");

// 리스트를 인수로 받는 HashSet
Set<String> friends3 = new HashSet<>(Arrays.asList("Raphael", "Olivia", "Thibaut"));
```

-	자바 9에서는 작은 리스트, 집합, 맵을 쉽게 만들 수 있는 팩토리 메서드를 제공한다.

```java
// List.of (데이터 처리 형식을 설정하거나, 데이터를 변환할 필요가 없다면 사용 권장)
/** (add, set 메서드 사용 시 예외발생, 리스트를 바꿔야하는 상황이라면 직접 만들 것)
내부적으로 가변인수 버전은 추가 배열을 할당해서 리스트로 감싼다.
따라서 배열을 할당하고 초기화하면 나중에는 가비지 컬렉션을 하는 비용을 지불해야한다.
List.of 는 최대 10개까지만 고정된 숫자요소를 정의하므로써 이 비용을 제거한다. (열 개 이상은 가변인수를 이용하는 메소드가 나온더)
*/
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");

// Set.of (중복 요소 불가)
Set<String> friends2 = Set.of("Raphael", "Olivia", "Thibaut");

// Map.of, Map.ofEntries (맵 팩토리 메서드)
Map<String, Integer> ageOfFriends = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);

Map<String, Integer> ageOfFriends2 = Map.ofEntries(entry("Raphael", 30), entry("Olivia", 25), entry("Thibaut", 26));
```

##### 8.2 리스트와 집합 처리

-	자바8에서 List,Set 인터페이스에 다음 메서드가 추가되었다. 다음 메서드들은 호출한 컬렉션 자체를 바꾼다.
	-	removeIf : 프레디케이트를 만족하는 요소를 제거. List나 Set을 구현하거나 구현을 상속받은 모든 클래스에서 이용 가능
	-	replaceAll : 리스트에 이용할 수 있는 기능으로 UnaryOperator 함수를 이용해 요소를 바꿈
	-	sort : List 인터페이스에 제공하는 기능. 정렬한다

```java
for(Interator<Transcation> iterator = trasactions.iterator(); iterator.hasNext();){
	Transcation transaction = iterator.next();
	if(Character.isDigit(transaction.getReferenceCode().CharAt(0))){
		iterator.remove();
	}
}

// removeIf로 변경
transactions.removeIf(transaction -> Character.isDigit(transaction,getReferenceCode().CharAt(0)));


...

for(ListIterator<String> iterator = referenceCodes.listIterator(); iterator.hasNext(); ){
	String code = iterator.next();
	iterator.set(Character.toUpperCase(code.CharAt(0))+ code.subString(1));
}

// replaceAll로 변경
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.subString(1));
```

##### 8.3 맵 처리

-	자바 8에서는 Map 인터페이스에 몇 가지 디폴트 메서드를 추가했다.

```java
//  맵의 항목 집합 반복하기
for(Map.Entry<String, Integer> entry : ageOfFriends.entrySet()) {
	String friend = entry.getKey();
	Integer age = entry.getValeu();
	System.out.println(friend + " is " + age + "year old");
}

//forEach 메서드 로 개선
ageOfFriends.forEach( (friend, age) -> System.out.println(friend + " is " + age + "year old"));


// 정렬
// Entry.comparingByValue, Entry.comparingByKey 로 정렬할 수 있다.
Map<String, String> favoriteMovies = Map.ofEntries(entry("Rapheal", "Start was"), entry("Cristina", "Matrix"), entry("Olivia", "James Bond"));

favouriteMovies.entrySet().stream().sorted(Entry.comparingByKey()).forEach(System.out::println)

// 찾으려는 키가 존재하지 않을 때
// getOrDefault ( key, 기본값)
Map<String, String> favoriteMovies = Map.ofEntries(entry("Raphael", "Star Wars"), entry("Olivia", "James Bond"));

// key가 없으므로 기본 값 Matirx가 출력됨
System.out.println(favoriteMovies.getOrDefault("Thibaut", "Matrix"));

/* 맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야하는 상황(키가 존재하면 결과를 다시 계산할 필요 없음)에 유용한 연산
computeIfAbsent : 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 계산을 하고 맵에 추가한다.
computeInPresent : 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한댜.
compute : 제공된 키로 새 값을 계산하고 맵에 저장한다.
*/

// computeIfAbsent : 제공된 키에 해당하는 값이 없으면, 키를 이용해 새 계산을 하고 맵에 추가한다.
lines.forEach(line -> dataToHash.computeIfAbsent(line, this::calculateDigest));

// 다음과 같이 간결하게 remove를 사용할 수 있다.
// 기존 : 키 존재하는지 검사, 키, 밸류 검사 후 remove(key)로 제거
favoriteMovies.remove(key, value);

// 교체패턴
// replaceAll : BiFunction을 적용한 결과로 각 항목의 값을 교체한다.
// Replace : 키가 존재하면 맵의 값을 바꾼다.
Mpa<String, String> favouriteMovies = new HashMap<>();
favouriteMovies.put("Rapael", "Star Wars");
favouriteMovies.put("Olivia", "james bond");
favouriteMovies.replaceAll((friend, movies) -> movies.toUpperCase());

// 중복된 키가 없다면 동작한다.
everyone.putAll(friends);

// 유연하게 합칠 수 있는 새로운 merge 메서드 (중복될 키를 어떻게 합칠지 결정하는 BiFunction을 인수로 받는다.)
Map<String, String> family = Map.ofEntries(entry("Teo", "Start Wars"), entry("Cristina", "James Bond"));

Map<String, String> friends = Map.ofEntries(entry("Cristina", "Matrix"), entry("Rapeal", "Star Wars"));

Map<String, String> everyone = new HashMap<>(family);
friends.forEach( (k,v) -> everyone.merge(k, v, (movie1, movie2) -> movie1 + "&" + movie2));

```

-	자바 8에서 HashMap 성능을 개선했다고한다.
	-	맵의 항목은 키로 생성한 해시코드로 접근할 수 있는 버켓에 저장하는데, 이때 많은 키가 같은 해시코드를 반환하는 상황이 되면 O(n)의 시간이 걸리는 LinkedList로 버킷을 반환해야해서 성능이 저하됨.
	-	버킷이 너무 커질 경우 이를 O(log(n))의 시간이 소요되는 정렬된 트리를 이용해 동적으로 치환해 충돌이 일어나는 요소 반환 성능을 개선했다고 한다.

##### 8.4 개선된 ConcurrentHashMap

-	ConcurrentHashMap는 동시성 친화적이며 최신 기술을 반영한 HashMap 임

	-	내부 자료구조의 특정 부분만 잠궈 동시 추가, 갱신 작업을 허용한다.
	-	동기화된 Hashtable 버전에 비해 읽기 쓰기 연산 성능이 월등하다. (표준 HashMap은 비동기)

-	스트림과 비슷한 종류의 세 가지 새로운 연산을 지원함.

	-	forEach : 각 (키,값) 쌍에 주어진 액션을 실행
	-	reduce : 모든 (키,값) 쌍에 제공된 리듀스 함수를 이용해 결과로 합침
	-	search : 널이 아닌 값을 반환할 때까지 각 (키,값) 쌍에 함수를 적용

-	네 가지 연산 형태를 지원함

	-	키, 값으로 연산 : forEach, reduce, search
	-	키로 연산 : forEachKey, reduceKeys, searchKeys
	-	값으로 연산 : forEachValue, reduceValues, searchValues
	-	Map.Entry 객체로 연산 : forEachEntry, reduceEntries, searchEntries

```java
//reduceVlaues메서드를 이용한 맵의 최댓값 구하기
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Integer> maxValue = Optional.ofNullalbe(map.reduceValues(parallelismThreshold, Long::max));
```

-	위의 연산들은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행함 (이들 연산에 제공한 함수는 계산이 제공되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야함)

-	reduceValuesToInt와 같이 int, long, double 등의 기본값에는 전용 each reduce 연산이 존재함. 이를 이용하자 (박싱 필요없음, 효율적임)

-	mappingCount : 맵의 매핑개수 반환 (size보다 이 메서드를 쓸 것)

-	ConcurrentHashMap을 집합 뷰로 반환하는 keySet 메서드 제공.

	-	newKeySet 메서드를 이용해 ConcurrentHashMap으로 유지되는 집합을 만들 수도 있음.
