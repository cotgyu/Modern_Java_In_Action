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

---

#### Chap 9 : 리팩터링, 테스팅, 디버깅

-	이 챕터에서는 람다표현식을 이용해 가독성과 유연성이 높이려면 기존 코드를 어떻게 리팩터링해야하는지 설명한다.
-	람다표현식과 스트림 API를 사용하는 코드를 테스트하고 디버깅하는 방법을 설명한다.

##### 9.1 가독성과 유연성을 개선하는 리팩터링

-	지금까지 학습한 람다, 메서드 참조, 스트림 등을 이용하여 가독성을 높이고 유연성한 코드로 리팩터링해보자.

-	가독성이 높이려면? : 코드의 문서화를 잘하고, 표준 코딩 규칙을 준수하는 등의 노력이 필요함. (다른사람이 쉽게 이해하고 유지보수할 수 있어야함) (9장에선 가독성을 개선할 수 있는 3가지 예제를 소개한다. )

	-	익명 클래스를 람다표현식으로 리팩터링

		```java
		// 익명클래스 사용
		Runnable r1 = new Runnable(){
		    public void run(){
		        System.out.println("Hello");
		    }
		};


		// 람다표현식으로 개선
		Runnable r2 = () -> System.out.println("Hello");
		```

		-	하지만, 모든 익명클래스를 람다 표현식으로 변환할 수 있는 것은 아님.

			-	익명 클래스에서 사용한 this와 super는 람다표현식에서 다른 의미를 갖는다.
				-	익명클래스 - this: 익명클래스 자신
				-	람다 - this: 람다를 감싸는 클래스
			-	익명클래스는 감싸고 있는 클래스의 변수를 가릴 수 있다. (람다는 불가능)

			```java
			int a = 10;
			Runnable r1 = new Runnable(){
			    public void run(){
			        int a = 2; // 동작
			    }
			};


			Runnable r2 = () -> {
			    int a = 2; // 컴파일 에러
			};
			```

			-	익명클래스를 람다표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.

				-	익명클래스는 인스턴스화할 때 명시적으로 형식이 정해짐
				-	람다형식은 콘텍스트에 따라 달라짐
				-	하지만, IDE에서 제공하는 리팩터리를 사용하면 모호함의 문제는 자동으로 해결된다고함.

				```java
				interface Task{
				    public void execute();
				}
				public static void doSomeThing(Runnable r){
				    r.run();
				}
				public static void doSomeThing(Task a){
				    a.execute();
				}


				// 익명클래스
				doSomeThing(new Task(){
				    public void execute(){
				        System.out.println("Danger!");
				    }
				});


				// 람다 표현식 - doSomeThing(Runnable)과 doSomeThing(Task) 중 어느 것을 가리키는 지 알 수없는 모호함 발생함
				doSomeThing(() -> System.out.println("Danger!!"));


				// 명시적 형변환을 통해 모호함 제거 가능
				doSomeThing(Task() -> System.out.println("Danger!!"));
				```

	-	람다 표현식을 메서드 참조로 리팩터링

		-	람다표현식 대신 메서드 참조를 이용하면 가독성을 높일 수 있다.

		```java
		// 칼로리 수준으로 요리를 그룹화
		Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu
		.stream()
		.collect(
		groupingBy(dish -> {
		     if (dish.getCalories() <= 400) return CaloricLevel.DIET;
		     else if (dish.getCalories() <= 700) return CaloricLevel.NORAML;
		     else return CaloricLevel.FAT;
		 }));


		// 람다표현식을 벼롣의 메서드로 추출 개선!
		public class Dish{
		    ...
		    public CaloricLevel getCaloricLevel{
		        if (dish.getCalories() <= 400) return CaloricLevel.DIET;
		        else if (dish.getCalories() <= 700) return CaloricLevel.NORAML;
		        else return CaloricLevel.FAT;
		    }
		}


		Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(groupingBy(Dish::getCaloricLevel));


		```

-	명령형 데이터 처리를 스트림으로 리팩터링

	-	스트림 API는 데이터 처리 파이프라인의 의도를 더 명확하게 보여준다. (쇼트서킷과 게으름이라는 강력한 최적화뿐 아니라 멀티코어 아키텍처를 활용할 수 있는 지름길을 제공한다.)

	```java
	// 명령형 코드
	List<String> dishNames = new ArrayList<>();
	for(Dish dish: menu){
	    if(dish.getCalories() > 300){
	        dishNames.add(dish.getName());
	    }
	}


	// 스트림 (병렬화, 문제를 좀 더 직접적으로 가술)
	menu.parallelStream().filter( d -> d.getCalories() > 300).map(Dish::getName).collect(toList());


	```

-	코드 유연성 개선

	-	람다표현식을 이용하면 동작 파라미터화를 쉽게 구현할 수 있다. (다양한 람다를 전달하여 다양한 동작 표현 가능, 변화하는 요구사항에 대응할 수 있는 코드를 구현할 수 있다.)

	-	함수형 인터페이스 적용

		-	람다 표현식을 이용하려면 함수형 인터페이스가 필요하다.

	-	조건부 연기 실행

		-	클라이언트 코드에서 객체 상태를 자주 확인하거나, 객체의 일부 메서드를 호출하는 상황이라면 내부적으로 객체의 상태를 확인한 다음에 메서드를 호출하도록 구현하는 것이 좋다.

		-	이를 통해 코드 가독성이 좋아질 뿐 아니라 캡슐화도 강화된다.

		```java
		// logger 상태가 isLoggable 메서드에 의해 클라이언트 코드로 노출된다.
		// 메시지를 로깅할 때마다 logger 객체 상태를 매번 확인해야함.
		if(logger.isLoggable(Log.FINER)){
		    logger.finer("Problem: "+ generateDiagnostic());
		}


		// 람다를 이용해 특정 조건에서만 메시지가 생서오딜 수 있도록 메시지 생성과정을 연기
		logger.log(Level.FINER, () -> "Problem: "+ generateDiagnostic());


		public void log(Level level, Supplier<String> msgSupplier){
		    if(logger.isLoggable(level)){
		        log(level, msgSupplier.get());
		    }
		}
		```

-	실행 어라운드

	-	매번 같은 준비, 종료 과정을 반복적으로 수행하는 코드가 있다면 람다로 변환할 수 있다. (준비, 종료과정을 처리하는 로직을 재사용함할 수 있어 코드 중복을 줄일 수 있다.)

	```java
	String oneLine = processFile((BufferedReader b) -> b.readLine());


	String twoLine = processFile((BufferedReader b) -> b.readLine() + b.readLine());


	public static String processFile(BufferedReaderProcessor p) throws IOException{
	    try(BufferedReader br = new BufferedReader(new FileReader ("test/data.txt"))){
	        return p.process(br);
	    }
	}


	// 람다로 BuffredReader 객체의 동작을 결정할 수 있는 것은 함수형 인터페이스 BufferedReaderProcessor 덕분임!!
	public interface BufferedReaderProcessor {
	    String process(BufferedReader b) throws IOException;
	}
	```

##### 9.2 람다로 객체지향 디자인 패턴 리팩터링하기

-	디자인 패턴은 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는 검증된 청사진을 제공한다.

-	람다 표현식으로 기존의 많은 객체지향 디자인 패턴을 제거하거나 간결하게 재구현할 수 있다.

-	전략 패턴

	-	전략 패턴 : 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법

		-	ex_ 다양한 기준을 갖는 입력값을 검증, 다양한 파싱방법 사용, 입력 형식 설정 등

	-	전략 패턴은 세 부분으로 구성된다.

		-	알고리즘을 나타내는 인터페이스
		-	다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현
		-	전략 객체를 사용하는 한 개 이상의 클라이언트

		```java
		// 알고리즘을 나타내는 인터페이스
		public interface ValidationStrategy {
		    boolean execute(String s);
		}


		// 다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현
		public class IsAllLowerCase implements ValidationStrategy {
		    public boolean execute(String s){
		        return s.matches("[a-z]+");
		    }
		}


		public class IsNumeric implements ValidationStrategy {
		    public boolean execute(String s){
		        return s.matches("\\d+");
		    }
		}


		// 구현한 클래스 사용 (클라이언트)
		public class Validator {
		    private final ValidationStrategy strategy;
		    public Validator(ValidationStrategy v){
		        this.strategy = v;
		    }
		    public boolean validate(String s){
		        return strategy.execute(s);
		    }
		}


		Validator numericValidator = new Validator(new IsNumeric());
		boolean b1 = numericValidator.validate("aaaa");
		Validator lowerValidator = new Validator(new IsAllLowerCase());
		boolean b2 = lowerCaseValidator.validate("bbbb");


		// 람다 표현식 개선
		// - 다양한 전략을 구현하는 새로운 클래스 구현할 필요 없이 람다 표현식을 직접 전달!
		Validator newNumericValidator = new Validator((String s) -> s.matches("[a-z]+"));
		boolean b1 = numericValidator.validate("aaaa");
		Validator newLowerValidator = new Validator((String s) -> s.matches("\\d+"));
		boolean b2 = lowerCaseValidator.validate("bbbb");


		```

-	템플릿 메서드

	-	알고리즘의 개요를 제시한 다음에 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야할 때 템플릿 메서드 디자인 패턴을 사용한다.

	```java
	// OnlineBanking 클래스를 상속받아 makeCustomerHappy 메서드가 원하는 동작을 수행하도록 구현할 수 있다.
	abstract class OnlineBanking {
	    public void processCustomer(int id) {
	            Customer c = Database.getCustomerWithId(id);
	            makeCustomHappy(c);
	    }


	    abstract void makeCustomerHappy(Customer c);
	}


	// 람다표현식 사용
	// - 클래스를 상속받지 않고 직접 람다 표현식을 전달해서 다양한 동작을 추가할 수 있음
	public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
	    Customer c = Database.getCustomerWithId(id);
	    makeCustomerHappy.accept(c);
	}


	new OnlineBankingLamda().processCustomer(1337, (Custom c) -> System.out.println("Hello" + c.getName()));
	```

-	옵저버

	-	어떤 이벤트가 발생했을 때 한 객체가 다른 객체 리스트에 자동으로 알림을 보내야하는 상황에서 사용한다.
	-	옵저버 패턴의 경우, 옵저버가 상태를 가지며 여러 메서드를 정의하는 등 복잡하다면 람다 표현식 보다 기존의 클래스 구현 방식을 고수하는 것이 바람직할 수 있다.

	```java
	// 다양한 옵저버를 그룹화할 Observer 인터페이스
	interface Observer {
	    void notify(String tweet);
	}


	// 다양한 키워드에 다른 동작을 수행할 수 있는 여러 옵저버 정의
	class NYTimes implements Observer {
	    public void notify(String tweet){
	        if( tweet != null && tweet.contains("money")){
	            System.out.println("Breaking news in NY! "+ tweet);
	        }
	    }
	}


	class Guardian implements Observer {
	    public void notify(String tweet){
	        if( tweet != null && tweet.contains("queen")){
	            System.out.println("Yet more news from London "+ tweet);
	        }
	    }
	}


	// 주제 구현
	interface Subject {
	    void registerObserver(Observer o);
	    void notifyObservers(String tweet);
	}


	// 옵저버등록, 트윗의 오저버에게 알리기
	Class Feed implements Subject {
	    private final List<Observer> observers = new ArrayList<>();


	    public void registerObserver(Observer o) {
	        this.observers.add(o);
	    }
	    public void notifyObservers(String tweet){
	        observers.forEach( o -> o.notify(tweet));
	    }
	}


	Feed f = new Feed();
	f.registerObserver(new NYTimes());
	f.registerObserver(new Guardian());
	f.notifyObservers("The queen said her favorite book is JAVA");


	// 람다표현식 사용
	// - 옵저버를 명시적으로 인스턴스화 하지 않고 람다표현식을 직접 전달해서 실행할 동작을 지정할 수 있다.
	f.registerObserver( (String tweet) -> {
	    if(tweet != null && tweet.contains("money")){
	        System.out.println("Breaking news in NY! "+ tweet);
	    }
	});


	f.registerObserver( (String tweet) -> {
	    if(tweet != null && tweet.contains("queen")){
	        System.out.println("Yet more news from London "+ tweet);
	    }
	});


	```

-	의무체인

	-	한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체도 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 식의 동작체인을 만들 때 의무 체인 패턴을 사용한다.

	-	일반적으로 다음으로 처리할 객체 정보를 유지하는 필드를 포함하는 작업 처리 추상 클래스로 의무 체인 패턴을 구성한다. 작업 처리 객체가 자신의 작업을 끝냈으면 다음 작업 처리 객체로 결과를 전달한다.

	```java


	public abstract class ProcessingObject<T> {
	    protected ProcessingObject<T> successor;
	    public void setSuccessor(ProcessingObject<T> successor){
	        this.successor = successor;
	    }


	    public T handle(T input){
	        T r = handleWork(input);
	        if(successor != null){
	            return successor.handle(r);
	        }
	        return r;
	    }


	    abstract protected T handleWork(T input);
	}


	// ProcessingObject 클래스를 상속받아 handlework 메서드를 구현하여 다양한 종류의 작업 처리 객체를 만들 수 있다.
	public class HeaderTextProcessing extends ProcessingObject<String>{
	    public String hanleWork(String text){
	        return "From Raoul, Mario and Alan: " + text;
	    }
	}


	public class SpellCheckerProcessing extends ProcessingObject<String>{
	    public String hanleWork(String text){
	        return text.repalceAll("labda", "lambda");
	    }
	}


	ProcessingObejct<String> p1 = new HeaderTextProcessing();
	ProcessingObejct<String> p2 = new SpellCheckerProcessing();
	// 두 작업처리 객체를 연결함.
	p1.setSuccessor(p2);


	String result = p1.handle("Aren't ladbas really good?!!");


	// 람다 표현식 사용
	// - 작업 처리 객체를 UnaryOperator<String> 형식의 인스턴스로 표현할 수 있다.
	// - andThen 메서드로 이들 함수를 조합하여 체인을 만들 수 있다.
	UnaryOperator<String> headerProcessing = (String text) -> "Text 1" + text;
	UnaryOperator<String> spellCheckerProcessing = (String text) text.replaceAll("Text", "Text2");


	Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing);


	String result = pipeline.apply("Text3");
	```

-	팩토리

	-	인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 때 팩토리 디자인 패턴을 사용한다.

	```java
	public class ProductFactory{
	    public static Product createProduct(String name){
	        switch(name){
	            case "loan" : return new Loan();
	            case "stock" : return new Stock();
	            default : throw new RuntimeException("No Such product" + name);
	        }
	    }
	}
	// 생성자와 설정을 외부로 노출하지 않음으로써 클라이언트는 단순하게 상품을 생산할 수 있다.
	Product p = ProductFactory.createProduct("loan");


	// 람다표현식 사용  - 생성자도 메서드참조 처럼 접근할 수 있다.
	Supplier<Product> loanSupplier = Loan::new;
	Loan loan = loanSupplier.get();


	final static Map<String, Supplier<Product>> map = new HashMap<>();


	static {
	    map.put("loan", Loan::new);
	    map.put("stock", Stock::new);
	}


	public static Product createProduct(String name){
	    Supplier<Product> p = map.get(name);
	    if(p != null) {
	        return p.get();
	    }


	    throw new IllegalArgumentExcetion("No Such product" + name);
	}


	```

##### 9.3 람다 테스팅

> 일반적으로 좋은 소프트웨어 공학자라면 프로그램이 의도대로 동작하는지 확인할 수 있는 단위테스팅을 진행한다.

-	람다는 익명이므로 테스트 코드이름을 호출할 수 없다.

	-	람다를 필드에 저장해서 재사용할 수 있으며 람다의 로직을 테스트할 수 있다.
	-	람다 표현식 자체를 테스트하는 것 보다는 람다 표현식이 사용되는 메서드의 동작을 테스트하는 것이 바람직하다.

	```java
	@Test
	public void testComparingTwoPoints() throws Exception{
	    Point p1 = new Point(10, 15);
	    Point p2 = new Point(10, 20);


	    int result = Point.compareByXAndThenY.comport(p1, p2);
	    assertTrue(result < 0);
	}


	```

	-	람다 표현식을 메서드 참조로 바꾸면 일반 메서드를 테스트하듯 복잡한 람다 표현식을 테스트할 수 있다.

##### 9.4 디버깅

-	디버깅할 때 개발자는 다음 두 가지를 확인해야한다.

	-	스택 트레이스 (하지만, 람다 표현식 내부에서 발생하는 에러는 스택 트레이스를 이해하기 힘들다.)
	-	로킹

-	정보 로깅

	-	스트림 파이프 라인에서 요소를 처리할 때 peek메서드로 중간 값을 확인할 수 있다.

	```java
	List<Integer> numbers = Arrays.asList(2,3,4,5);


	// forEach를 호출하는 순간 전체 스트림이 소비된다.
	numbers.stream().map(x -> x + 17).filter(x -> x % 2 == 0).limit(3).forEach(System.out::println);


	// peek 은 스트림의 각 요소를 소비한 것 처럼 동작을 실행한다. (소비하진 않는다.)
	numbers.stream().
	.peek(x -> System.out.println("from stream: " + x))
	.map(x -> x + 17)
	.peek(x -> System.out.println("after map: " + x))
	.filter(x -> x % 2 == 0)
	.peek(x -> System.out.println("after filter: " + x))
	.limit(3)
	.peek(x -> System.out.println("after limit: " + x))
	.collect(toList());


	```
