모던 자바 인 액션 Part 2
------------------------

#### Chap 4 : 스트림 소개

-	거의 모든 자바 애플리케이션은 컬렉션을 만들고 처리하는 과정을 포함한다. 컬렉션으로 데이터를 그룹화하고 처리할 수 있다.

##### 4.1 스트림이란 무엇인가?

-	스트림은 자바 8 API에 새로 추가된 기능으로 스트림을 이용하면 선언형으으로 컬렉션 데이터를 처리할 수 있다.

	-	멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다.

	```java
	  // 기존 코드 (자바7)
	  List<Dish> lowCaloricDishes = new ArrayList<>():
	  for(Dish dish: menu){
	    if(dish.getCalories() < 400){
	      lowCaloricDishes.add(dish);
	    }
	  }
	   // 익명클래스로 정렬
	  Collections.sort(lowCaloricDishes, new Comparator<Dish>(){
	    public int compare(Dish dish, Dish dish2){
	      return Integer.compare(dish1.getCalories(), dish2.getCalories());
	    }
	  });
	  List<String> lowCaloricDishesName = new ArrayList<>();
	  for(Dish dish : lowCaloricDishes){
	    lowCaloricDishesName.add(dish.getName());
	  }


	//최신 코드 (자바8)
	import static java.util.Comparator.comparing;
	import static java.util.stream.Collectors.toList;
	List<String> lowCaloricDishesName =
	        menu.stream()
	            .filter(d -> d.getCalories() < 400) // 필터링
	            .sorted(comparing(Dish::getCalories)) // 칼로리로 요리정렬
	            .map(Dish::getName) // 요리명 추출
	            .collect(toList()); // 리스트에 저장


	// 병렬로 실행
	List<String> lowCaloricDishesName =
	        menu.parallelStream()
	            .filter(d -> d.getCalories() < 400) // 필터링
	            .sorted(comparing(Dish::getCalories)) // 칼로리로 요리정렬
	            .map(Dish::getName) // 요리명 추출
	            .collect(toList()); // 리스트에 저장


	```

-	스트림이 주는 이득

	-	선언형으로 코드를 구현할 수 있다. (루프와 if조건문 등 제어블럭을 사용해서 어떻게 동작을 구현할지 지정할 필요없이 특정 동작의 수행을 지정할 수 있음)
	-	filter, sorted, map, collect 같은 연러 빌딩 블록 연산을 연결해서 복잡한 데이터 처리 파이프라인을 만들 수 있다. (filter 메서드의 결과는 sorted 메서드로, 다시 sorted의 결과는 map 메서드로 ...)

	> filter (sorted, map, collect) 같은 연산은 고수준 빌딩 블록으로 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있다. 결과적으로 우리는 데이터 치리과정을 병렬화하면서 스레드와 락을 걱정할 필요가 없다.

-	스트림 API의 특징

	-	선언형 : 더 간결하고 가독성이 좋아진다
	-	조립할 수 있음 : 유연성이 좋아진다
	-	병렬화 : 성능이 좋아진디

##### 4.2 스트림 시작하기

-	스트림이란? : 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

	-	연속된 요소 : 컬렉션과 마찬가지로 스트림은 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스 제공

		-	컬렉션은 자료구조이므로 시간과 공간의 복잡성과 관련된 요소 저장 및 접근연산이 주를 이룬다(ArrayList를 사용할 것인지 LinkedList를 사용할 것인지)
		-	반면 스트림은 filter, sorted, map처럼 표현 계산식이 주를 이룬다. (컬렉션의 주제는 데이터 / 스트림의 주제는 계산)

	-	소스 : 스트림은 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다. 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지된다. 즉, 리스트로 스트림을 만들면 스트림의 요소는 리스트의 요소와 같은 순서를 유지한다.

	-	데이터 처리 연산 : 스트림은 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 데이터베이스와 비슷한 연산을 지원한다. 예를 들어 filter, map, reduce, find, match, sort 등으로 데이터를 조작할 수 있다. 스트림 연산은 순차적으로 또는 병렬로 실행할 수 있다.

	-	파이프라이닝 : 대부분의 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프 라인을 구성할 수 있도록 스트림 자산을 반환한다. (laziness, short-circuiting 같은 최적화를 얻을 수 있다) 연산 파이프 라인은 데이터 소스에 적용하는 데이터베이스 질의와 비슷하다.

	-	내부 반복 : 반복자를 이용해서 명시적으로 반복하는 컬렉션과 달리 스트림은 내부반복을 지원한다.

	-	위의 특징들을 예제로 확인한다.

		```java
		import static java.util.stream.Collectors.toList;
		// collect를 제외한 모든 연산은 서로 파이프라인을 형성할 수 있도록 스트림을 반환한다.
		// 마지막 collect 연산을 통해 결과를 반환한다.
		List<String> threeHighCaloricDishNames =
		    menu.stream()   // 메뉴에서 스트림을 얻는다
		            .filter(dish -> dish.getCalories() > 300) // 파이프라인 연산 만들기(고칼로리 필터 )
		            .map(Dish::getName) // 요리명 추출
		            .limit(3)   // 선착순 3개 선택
		            .collect(toList()); // 결과를 다른 리스트로 저장


		System.out.println(threeHighCaloricDishNames);
		```

##### 4.3 스트림과 컬렉션

-	자바의 기존 컬렉션과 새로운 스트림 모두 연속된 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다. ( **데이터를 언제 계산하느냐가 컬렉션과 스트림의 가장 큰 차이** )

	-	컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 구조 : 컬렉션의 모든 요소는 컬력센에 추가하기 전에 계산되어야 함(적극적 생성 - 모든 값을 계산할 때까지 기다림)

	-	스트림은 이론적으로 요청할 때만 요소를 계산하는 고정된 자료구조이다. (게으른 생성 - 필요할 때만 값을 계산)

-	스트림은 반복자와 마찬가지로 한 번만 탐색할 수 있다. **탐색된 스트림요소는 소비된다.**

-	컬렉션 인터페이스를 사용하려면 사용자가 직접요소를 반복해야한다. (ex for-each) **외부반복** 이라 한다.

-	스트림 라이브러리는 반복을 알아서 처리하고 결과 스트림 값을 어딘가에 저장해주는 **내부반복** 을 사용한다.

	```java
	// for-each 루프
	List<String> names = new ArrayList<>();
	for(Dish dish: menu){
	    names.add(dish.getName());
	}


	//스트림 내부반복
	List<String> names = menu.stream()
	                         .map(Dish::getName)
	                         .collect(toList());
	```

-	내부반복을 이용하면 작업을 투명하게 병렬로 처리하거나 더 최적화된 다양한 순서로 처리할 수 있다.

	-	스트림 라이브러리의 내부반복은 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다.
	-	for-each를 이용하는 외부반복에서는 병렬성을 스스로 관리해야 한다.

##### 4.4 스트림 연산

-	스트림 인터페이스의 연산을 크게 두 가지로 구분할 수 있다.

	-	중간연산 : 연결할 수 있는 스트림 연산
	-	최종연산 : 스트림을 닫는 연산

		```java
		List<String> threeHighCaloricDishNames =
		        menu.stream()   // 메뉴에서 스트림을 얻는다
		            .filter(dish -> dish.getCalories() > 300) // 파이프라인 연산 만들기(고칼로리 필터 ) - 중간연산
		            .map(Dish::getName) // 요리명 추출 - 중간 연산
		            .limit(3)   // 선착순 3개 선택 - 중간연산
		            .collect(toList()); // 결과를 다른 리스트로 저장 - 최종연산
		```

-	중간 연산의 중요한 특징은 단말 연산을 스트림 파이프라인에 실행하기 전까지 아무 연산도 수행하지 않는 다는 것이다. (게으르다) 중간 연산을 합친 다음에 합쳐진 중간 연산을 최종 연산으로 한 번에 처리하기 때문이다.

-	최종 연산은 스트림 파이프라인에서 결과를 도출한다. (스트림 이외의 결과가 반환된다.)

-	스트림 이용과정은 세 가지로 요약할 수 있다.

	-	질의를 수행할 데이터소스 (컬렉션)
	-	스트림 파이프라인을 구성할 중간 연산 연결
	-	스트림 파이프라인을 실행하고 결과를 만들 최종 연산

---

#### Chap 5 : 스트림 활용

-	스트림 API가 지원하는 다양한 연산에 대해 살펴본다. (필터링, 슬라이싱, 매핑, 검색, 매칭, 리듀싱 등 다양한 데이터 처리 질의 표현 가능)

##### 5.1 필터링

-	스트림의 요소를 선택하는 방법에 대해 배운다

	-	프레디케이트 필터링 : filter 메서드는 프레디케이트(불리언을 반환하는 함수)를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다.

		```java
		List<Dish> vegetarianMenu = menu.stream()
		                                .filter(Dish::isVegetarian)
		                                .collect(toList());
		```

	-	고유 요소 필터링 : 고유 요소로 이루어진 스트림을 반환하는 distinct 메서드도 지원한다.

		```java
		List<Integer> numbers = Arrays.asList(1,2,1,3,3,2,4);


		numbers.steam()
		                .filter(i -> i % == 0)
		                .distinct()
		                .forEach(System.out::println);
		```

##### 5.2 스트림 슬라이싱

-	스트림의 요소를 선택하거나 스킵하는 다양한 방법에 대해 살펴본다.

-	프레디케이트를 이용한 슬라이싱 : 자바 9는 스트림의 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 두 가지 새로운 메서드를 지원한다.

	-	takeWhile : filter연산은 전체 스트림을 반복하면서 각 요소에 프레디케이트를 적용하지만, 이미 정렬된 상태의 리스트라면 비효율적일 수 있다. takeWhile을 이용하면 무한 스트림을 포함한 모든 스트림에 프레디케이트를 적용해 스트림을 슬라이스할 수 있다.

		```java
		List<Dish> slicedMenu1 = specialMenu.stream()
		                                    .takeWhile(dish -> dish.getCalories() < 320)
		                                    .collect(toList());
		```

	-	dropWhile : takeWhile과 정반대 작업이다. 남은 요소!

		```java
		List<Dish> slicedMenu2 = specialMenu.stream()
		                                     .dropWhile(dish -> dish.getCalories() < 320)
		                                     .collect(toList());
		```

-	스트림 축소

	-	스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드를 지원한다.

		```java
		List<Dish> dishes = sepcialMenu.stream()
		                                .filter(dish -> dish.getCalories() > 300)
		                                .limit(3)
		                                .collect(toList());
		```

-	요소 건너뛰기

	-	스트림은 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원한다. (n개 이하의 스트림에 skip(n)을 호출하면 빈 스트림이 반환된다.)

	```java
	List<Dish> dishes = sepcialMenu.stream()
	                               .filter(dish -> dish.getCalories() > 300)
	                               .skip(3)
	                               .collect(toList());
	```

##### 5.3 매핑

-	스트림 API의 map과 flatMap 메서드는 특정 데이터를 선택하는 기능을 제공한다.

-	함수를 인수로 받는 map 메서드 : 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다.

```java
List<Dish> dishes = menu.stream()
												.map(Dish::getName)
												.collect(toList());

// map을 연결할 수도 있음
List<Dish> dishes = menu.stream()
												.map(Dish::getName)
												.map(String::length)
												.collect(toList());

```

-	flatMap : 각 배열을 스트림이 아니라 스트림의 콘텐츠로 매핑한다. 즉 map (Arrays::stream)과 달리 하나의 평면화된 스트림을 번환한다. (스트림의 각 값을 다른 스트림으로 만든 다음에 모든 스트림을 하나의 스트림으로 연결)

-	퀴즈 5.2 : 두 개의 숫자 리스트가 있을 때 모든 숫자 쌍의 리스트를 반환하시오. 예를 들어 두 개의 리스트 [1,2,3]과 [3,4]가 주어지먄 [(1,3),(1,4),(2,3),(3,3),(3,4)] 를 반환해야함.

```java
List<Integer> number1 = Arrays.asList(1,2,3);
List<Integer> number2 = Arrays.asList(3,4);

List<int[]> pairs = number1.stream().flatMap(i -> number2.stream().map(j -> new int[]{i,j})).collect(toList());
```

##### 5.4 검색과 매칭

-	스트림 API는 특성속성이나 데이터 집합에 있는지 여부를 검색하는 데이터처리의 allMatch, anyMatch, nonMatch, findFirst, findAny 등 다양한 메서드를 제공한다.

-	요소 검사

	-	스트림 쇼트기법 즉, 자바의 &&, || 와 같은 연산을 활용한다.

		-	전체 스트림을 처리하지 않았더라도 결과를 반환할 수 있다. (표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식 결과아 상관없이 전체 결과도 거짓이 된다.)
		-	원하는 요소를 찾았으면 즉시 결과를 반환할 수 있다. (limit도 쇼트서킷 연산)

```java
// 적어도 한 요소와 일치하는지 확인
if(menu.stream().anyMatch(Dish::isVegetarian)){
	System.out.println("적어도 한 요소와 일치")
}

// 모든 요소와 주어진 프레디케이트와 일치하는지 확인
boolean isHealthy = menu.stream().allMatch(dish -> dish.getCalories() < 1000);

// 주어진 프레디케이트와 일치하는 요소가 없는지 확인
boolean isHealty = menu.stream.noneMatch(d -> d.getCalories() >= 1000);
```

-	요소 검색

```java
// 현재 스트림에서 임의의 요소를 반환한다. (다음의 filter와 같이 다른 스트림과 연결해서 사용가능)
Optional<Dish> dish = menu.stream().filter(Dish::isVegetarian).findAny();

// 첫 번째 요소 찾기
List<Integer> someNumbers = Arrays.asList(1,2,3,4,5);
Optional<Integer> firstSquareDivisibleByTree = someNumber.stream()
																													.map(n -> n*n)
																													.filter(n -> n % 3 == 0)
																													.findFirst();
```

##### 5.5 리듀싱

-	리듀싱 : 모든 스트림 요소를 처리해서 값으로 도출하는 연산

```java
// for-each 루프
int sum = 0;
for(int x : numbers){
	sum += x;
}

// 리듀싱 사용
int sum = numbers.stream().reduce(0, (a, b) -> a + b);

// 메서드 참조
int sum = numbers.stream().reduce(0, Integer::sum);

// 초기 값이 없는 reduce (스트림에 아무 요소도 없는 상황일 때 초기값이 없으므로 Optional 처리)
Optional<Integer> sum = numbers.stream().reduce((a,b) -> (a+b));

// 최대값
Optional<Integer> max = numbers.stream().reduce(Integer::max);

// 최솟값  ( (x,y) -> x<y ? x:y 를 사용해도 가능)
Optional<Integer> min = numbers.stream().reduce(Integer::min);

```

-	reduce 메서드의 장점

	-	reduce를 사용하면 내부 반복이 추상화되면서 내부 구현에서 병렬도 reduce를 실행할 수 있게 됨
	-	하지만 병렬로 실행 시 reduce에 넘겨준 람다의 상태가 바뀌지 말아야하며, 연산이 어떤 순서로 실행되더라도 결과가 바뀌지 않는 구조여야한다.

-	중간연산과 최종연산 정리표 176p 참조

##### 5.6 실전 연습

-	스트림을 사용하여 거래를 실행하는 거래자 문제를 풀어본다.

	1.	2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정리하시오 .
	2.	거래자가 근무하는 모든 도시를 중복 없이 나열하시오.
	3.	케임브리지에 근무하는 모든 거래를 찾아서 이름순으로 정렬하시오.
	4.	모든 거래자의 이름을 알파벳순으로 정렬해서 반환하시오.
	5.	밀라노에 거래자가 있는가?
	6.	케임브리지에 거주하는 거래자의 모든 트랜잭션 값을 출력하시오.
	7.	전체 트랜잭션 중 최댓값은 얼마인가?
	8.	전체 트랜잭션 중 최솟값은 얼마인가?

	```java
	// 1.
	    List<Transaction> List2011 = transactions.stream()
	                                                .filter(transaction -> transaction.getYear() == 2011)
	                                                .sorted(comparing(Transaction::getYear))
	                                                .collect(toList());


	    // 2.
	    List<String> allCitys = transactions.stream()
	            .map(Transaction::getTrader)
	            .map(Trader::getCity)
	            .distinct().collect(toList());


	    // 3.
	    List<Trader> CambridgeTraders = transactions.stream()
	            .map(Transaction::getTrader)
	            .filter(trader -> trader.getCity().equals("Cambridge"))
	            .distinct()
	            .sorted(comparing(Trader::getName))
	            .collect(toList());


	    // 4.
	    String allTraderName = transactions.stream()
	            .map(transaction -> transaction.getTrader().getName())
	            .distinct()
	            .sorted()
	            .reduce("", (n1,n2) -> n1 + n2)
	           ;


	    // 5.
	    boolean isPresentTraderInMilan = transactions.stream()
	            .anyMatch(transaction -> transaction.getTrader().getCity().equals("Milan"))
	            ;


	    // 6.
	    List<Transaction> allTransactionInCam = transactions.stream()
	            .filter(transaction -> transaction.getTrader().getCity().equals("Cambridge"))
	            .collect(toList());


	    // 7.
	    Optional<Integer> max = transactions.stream()
	            .map(Transaction::getValue)
	            .reduce(Integer::max);


	    // 7.
	    Optional<Integer> min = transactions.stream()
	            .map(Transaction::getValue)
	            .reduce(Integer::min);
	```

##### 5.7 숫자형 스트림

-	reduce 메서드로 스트림요소의 합을 구할 수 있는데, 박싱비용이 발생할 수 있다.

```java
// 내부적으로 합계를 계산하기전에 Integer를 기본형으로 박싱해야함.
int calories = menu.stream().map(Dish::getCalories).reduce(0, Integer::sum);
```

-	스트림은 API숫자 스트림을 효율적으로 처리할 수 있도록 기본형 특화 스트림을 제공한다.
-	각 인터페이스는 숫자 스트림의 합계를 계산하는 sum, 최댓값 요소를 검색하는 max 등의 연산 메서드를 제공한다.

	-	IntStream (mapToInt)
	-	DoubleStream (mapToDouble)
	-	LongStream (mapToLong)

	```java
	// 기본형 특화 스트림
	int caloreis = menu.stream()
	.mapToInt(Dish:getCalories) // Stream<T> 대신 IntStream을 반환한다.
	.sum();


	// 객체 스트림으로 복원
	IntStream intStream = menu.stream().mapToInt(Dish:getCalories);
	Stream<Integer> stream = intStream.boxed();


	// Optional 의 기본형 특화 스트림 버전
	OptionalInt maxCalories = menu.stream().mapToInt(Dish::getCalories).max();
	// 값이 없을 때 기본 최댓값을 명시적으로 설정
	int max = maxCalories.orElse(1);


	// 특정 범위 메서드 (IntStream과 LongStream은 range, rangeClose 제공   )
	// range : 시작값과 종료값이 결과에 포함되지 않음
	// rangeClosed : 시작값과 종료값이 결과에 포함됨
	IntStream evenNumbers = IntStream.rangeClosed(1,100).filter(n -> n % 2 == 0);
	```

##### 5.8 스트림 만들기

-	다양한 방식으로 스트림을 만들 수 있다.

```java
// 값으로 스트림 만들기
Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");

// null이 될 수 있는 객체로 스트림 만들기
Stream<String> homeValueSteam = Stream.ofNullable(System.getProperty("home"));

// 배열로 스트림 만들기
int[] numbers = {2,3,5,7,11,13};
int sum = Arrays.steam(numbers).sum();

// 파일로 스트림 만들기
long uniqueWords = 0;
try(Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())) {
	uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" "))).distinct().count();
}

// 함수로 무한 스트림 만들기
// iterate : 초깃값과 람다를 인수로 받아 새로운 값을 끊임없이 생산할 수 있음
Stream.iterate(0, n -> n+2).limit(10).forEach(System.out::println);

// 자바 9에서는 두번 째 인수로 프레디케이트를 받아 언제까지 작업을 수행할지를 정할 수 있음
IntStream.iterate(0, n -> n < 100, n -> n+4).forEach(System.out::println);

// generate : Supplier<T>를 인수로 받아 새로운 값 생성 (IntStream을 통해 박싱연산 문제도 해결 가능 IntStream.generate( )
Stream.generate(Math::random).limit(5).forEach(System.out::println);

IntSupplier fib = new IntSupplier(){
	private int previous = 0;
	private int current = 1;
	public int getAsInt(){
		int oldPrevious = this.previous;
		int nextValue = this.previous + this.current;
		this.previous = this.current;
		this.current = nextValue;
		return oldPrevious;
	}
};
IntStream.generate(fib).limit(10).forEach(System.out::println);

```

---
