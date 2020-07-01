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

#### Chap 6 스트림으로 데이터 수집

-	이 장에서는 reduce가 그랬던 것 처럼 collect 역시 다양한 요소 누적방식을 인수로 받아서 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행할 수 있음을 설명한다.

-	Stream에 toList를 사용하는 대신 더 범용적인 컬렉션 파라미터를 collect 메서드에 전달함으로써 원하는 연산을 간결하게 구현할 수 있음을 배우게 된다.

##### 6.1 컬렉터란 무엇인가?

-	Collector 인터페이스 구현은 스트림 요소를 어떤 식으로 도출할지 지정한다.

-	collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다는 점이 컬렉터의 최대 강점이다.

-	Collectors 클래스에서 제공하는 팩토리메서드 기능을 설명한다. (groupingBy 등 )

	-	스트림 요소를 하나의 값으로 리듀스하고 요약
	-	요소 그룹화
	-	요소 분할

##### 6.2 리듀싱과 요약

-	컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있다. (트리를 구성하는 다수준 맵, 메뉴의 칼로리 합계를 가리키는 단순한 정수 등)

```java
// 카운팅
long howManyDishes = menu.stream().collect(Collectors.counting());
// 위와 같다
long howManyDishes = menu.stream().count();

// 최댓값 최솟값
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(dishCaloriesComparator));


```

-	요약 연산 : 스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산

```java
// 메뉴의 총 칼로리 계산
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));

// 평균
double avgCalories = menu.stream().collect(averageInt(Dish::getCalories));

// 종합 정보
// IntSummaryStatistics{count=9, sum=4300, min=120, average=477.77788, max=800}

IntSummaryStatistics menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));
```

-	문자열 연결 : joining 팩토리 메서드를 통해 문자열을 하나의 연결해서 반환할 수 있다.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining());

// 구분자를 넣을 수있게 오버로드된 joining 메서드도 있음
String shortMenu = menu.stream().map(Dish::getName).collect(joining(","));

```

-	범용 리듀싱 요약 연산

	-	지금까지나온 collect 는 reducing 으로 구현할 수 있다.

	```java
	int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i,j) -> i + j));
	```

-	collect와 reduce

	-	collect 메서드는 도출하려는 결과를 누적하는 컨테이너를 바꾸도록 설계된 메서드인 반면, reduce는 두 값을 하나로 도출하는 불변형 연산이이다.

	-	reduce를 잘못 사용하면 실용성 문제가 발생할 수 있다. (여러 스레드가 동시에 같은 데이터 구조체를 고치면 리스트 자체가 망가져 리듀싱 연산을 병렬로 수행할 수 없다.)

	-	가변 컨테이너 관련 작업이면서 병렬성을 확보하려면 collect 메서드로 리듀싱 연산을 구현하는 것이 바람직하다.

##### 6.3 그룹화

-	데이터 집합을 하나 이상의 특성으로 분류해서 그룹화하는 연산도 데이터베이스에서 많이 수행하는 작업이다.

	-	자바8의 함수형을 이용하면 가독성 있는 한 줄의 코드로 그룹화를 구현할 수 있다.

-	그룹화된 요소 조작

	-	요소를 그룹화 한 다음에는 각 결과 그룹의 요소를 조작하는 연산이 필요하다

	```java
	Map<Dish.Type, List<Dish>> caloricDishesByType = menu.stream().collect(groupingBy(Dish::getType, filtering(dish -> dish.getCalories() > 500, toList())));


	```

-	다수준 그룹화

	-	두 인수를 받는 팩토리 메서드 Collectors.groupingBy 를 이용해서 항목을 다수준으로 그룹화할 수 있다.

	```java
	Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream().collect(
	    groupingBy(Dish::getType,
	        groupingBy(dish -> {
	            if(dish.getCalories() <= 400){
	                return CaloricLevel.DIET;
	            }
	            else if(dish.getCalories() <= 700){
	                return CaloricLevel.NOMAL;
	            }
	            else{
	                return CaloricLevel.FAT;
	            }
	            }
	        )
	    ));
	```

-	서브그룹으로 데이터 수집

	-	다수준 그룹화에서 두 번째 groupingBy 컬렉터를 외부 컬렉터로 전달해서 다수준 그룹화 연산을 구현했다. 이 때 넘겨주는 컬렉터의 형식의 제한은 없다.

	```java
	Map<Dish.Type, Long> typesCount =  menu.stream().collect(groupingBy(Dish::getType, counting()));
	```

-	컬렉터 결과를 다른 형식에 적용하기

	-	맵의 모든 값을 Optional로 감쌀 필요가 없을 수 있다.

	> Collectors.collectingAndThen으로 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있다.

	```java
	Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
	.collect(groupingBy(Dish::getType, collectingAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)));
	```

-	groupingBy 활용 예제

	```java
	Map<Dish.Type, Integer> totalCaloriesByType = menu.stream().collect(groupingBy(Dish::getType, summingInt(Dish::getCalories)));


	Map<Dish.Type, Set<CaloricLevel>> caloricLevelsByType = menu.stream().collect(
	    groupingBy(Dish::getType, mapping(dish -> {
	        if(dish.getCalories() <= 400) return CaloricLevel.DIET;
	        else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
	        else return CaloricLevel.FAT;
	    }, toSet() ))
	);
	```

##### 6.4 분할

-	분할 함수는 불리언을 반환하므로 맵의 키 형식은 Boolean 이다. 결과적으로 최대 두 개의 그룹으로 분류된다.

```java

Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegetrian));

/* 결과는
{false=[pork, beef, chicken],
true=[rice, season fruit]} 의 맵이 반환된다.
*/

// 채식 요리 얻기
List<Dish> vegetarianDishes = partitionedMenu.get(true);
```

-	분할 함수가 반환하는 참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지한다는 것이 분할의 장점이다.

	-	컬렉터를 두번째 인수로 전달도 가능하다

	```java
	Map<Boolean, Dish> mostCaloricPartitionedByVegetarian = menu.stream().collect(partitioningBy(Dish::isVegetarian, collectingAndThen(maxBy)comparingInt(Dish::getCalories)),Optional::get)));
	```

-	223p Collectors 클래스의 정적 팩토리 메서드 표

##### 6.5 Collector 인터페이스

-	Collector 인터페이스는 리듀싱 연산(즉, 컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.

	-	직접 Collector 인터페이스를 구현하는 리듀싱 연산을 만들 수 있다. (직접 구현하여 더 효율적으로 문제를 해결하는 컬렉터를 만드는 방법을 살펴본다.)

```java
public interface Collector<T, A, R> {
	Supplier<A> supplier();
	BiConsumer<A, T> accumulator();
	Function<A, R> finisher();
	BinaryOperator<A> combiner();
	Set<Characteristics> characteristics();
}

/**
T : 수집될 스트림 항목의 제네릭 형식
A : 누적자, 즉 수집과정에서 중간 결과를 누적하는 객체의 형식
R : 수집 연산 결과 객체의 형식
**/
```

-	Collector 인터페이스의 메서드

	-	Supplier : 새로운 결과 컨테이너 만들기

		-	supplier는 수집과정에서 빈 누적자 인스턴스를 만드는 파리미터가 없는 함수다.

	-	accumulator : 결과 컨테이너에 요소 추가하기

		-	accumulator 메서드는 리듀싱 연산을 수행하는 함수를 반환한다.

	-	finisher : 최종 변환 값을 결과 컨테이너로 적용하기

		-	finisher 메서드는 스트림 탐색을 끝내고 누적자 객체를 최종 객체로 변호나하면서 누적과정을 끝낼 때 호출할 함수를 반환해야한다.

	-	combiner : 두 결과 컨테이너 병합

		-	리듀싱 연산에서 사용할 함수를 반환
		-	combiner는 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할 지 정의.

	-	Characteristics

		-	컬렉터의 연산을 정의하는 Characteristics 형식의 불변 집합을 반환
		-	Characteristics는 스트림을 병렬로 리듀스할 것인지 그리고 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트 제공
		-	Characteristics는 열거형임
			-	UNORERED : 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다.
			-	CONCURRENT : 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 스트림의 병렬 리듀싱을 수행할 수 있다. 데이터소스가 정렬되어 있지 않은 상황에서만 병렬 리듀싱 수행 가능
			-	IDENTITY_FINISH : finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 이를 생략할 수 있다. 따라서 리듀싱 과정의 최종 결과로 누적자 객체를 바로 사용할 수 있다. (누적자 A를 결과 R로 안전하게 형변환할 수 있음)

-	컬렉터 구현하지 않고 커스텀 수집 수행하기

	-	Stream은 세 함수(발행, 누적, 합침)를 인수로 받는 collect 메서드를 오버로드하여 각각의 메서드는 Collector 인터페이스의 메서드가 반환하는 함수와 같은 기능을 수행한다.  

	```java
	List<Dish> dishes = menuStream.collect(
	    ArrayList::new,
	    List::add,
	    List::addAll
	);
	```

-	책에서는 소수와 비소수 분류 예제를 커스텀 컬렉터를 구현하여 성능 개선하는 작업을 보여줬다.

##### 6.7 마치며

-	collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터)을 인수로 갖는 최종 연산이다.

-	컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계되었다.

-	Collector 인터페이스에 정의된 메서드를 구현해서 커스텀 컬렉터를 개발할 수 있다.

---

#### Chap 7 병렬 데이터 처리와 성능

-	자바 7 이전에는 데이터 컬렉션을 병렬로 처리하기 어려웠으나, 자바7은 포크/조인 프레임워크를 제공하여 더 쉽게 병렬화를 수행하면서 에러를 최소화할 수 있게해준다.

-	스트림으로 데이터 컬렉션 관련 동작을 얼마나 쉽게 병렬로 실행할 수 있는지 소개한다.

	-	스트림을 이용하면 순차스트림을 병렬스트림으로 자연스럽게 바꿀 수 있다.

##### 7.1 병렬 스트림

-	병렬스트림 : 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림
	-	병렬 스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있다.

```java
public long parallelSum(long n){
	return Stream.iterate(1L, i -> i + 1)
		.limit(n)
		.parallel() // 스트림을 병렬 스트림으로 변환
		.reduce(0L, Long::sum);
}
```

-	병렬 스트림에서 사용하는 스레드 풀 설정

	-	병렬 스트림은 내부적으로 ForkJoinPool 을 사용한다.
	-	기본적으로 ForkJoinPool은 프로세서 수, 즉 Runtime.getRuntime().availableProcessors()가 반환하는 값에 상응하는 스레드를 갖는다.

-	병렬화를 이용하였다고 더 좋아질 것이란 추측은 하지말자. 성능을 최적화할 때는 측정은 필수다.

	-	자바 마이크로벤치마크 하니스(JMH)라는 라이브러리를 이용해 벤치마크를 구할 수 있다. (어노테이션 방식 지원)

	-	병렬버전이 순차버전보다 느린 경우도 있다. (박싱/안박싱문제, 병렬로 수행할 수 있는 단위로 못나눌때 등)

	-	parallel 메서드를 호출했을 때 내부적으로 어떤 일이 일어나는지 꼭 이해해야한다.

	-	특화된 메서드로 성능향상을 이룰 수 있다. LongStream.rangeClosed (박싱언박싱 오버헤드 사라짐, 분할할 수 있는 숫자 범위생성함)

-	병렬 스트림은 잘못사용하면서 발생하는 많은 문제는 공유된 상태를 바꾸는 알고리즘을 사용하기 때문에 일어난다.

	-	여러스레드에서 공유하는 객체의 상태가 바뀌어서 결과가 이상해짐..

-	병렬 스트림 효과적으로 사용하기

	-	확신이 서지 않으면 직접 측정하라
	-	박싱을 주의하라 (자바 8은 기본형특화 스트림 IntStream, LongStream, DoubleStream을 제공한다.)
	-	limit 나 findFirst 처럼 요소의 순서에 의존하는 연산을 병렬스트림에서 수행하려면 비싼 비용을 치러야한다.
	-	소량의 데이터에서는 병렬 스트림이 도움 되지 않는다.
	-	스트림을 구성하는 자료구조가 적절한지 확인하라. (ArrayList는 요소를 탐색하지 않고도 리스트를 분할할 수 있기 때문에 LinkedList보다 효율적으로 분할할 수 있음)
	-	스트림의 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다. (필터 연산이 있으면 스트림의 길이를 예측할 수 없으므로 효과적으로 스트림을 병렬처리할 수 있을지 모른다)
	-	최종연산의 병합과정 비용을 살펴보라 .(Ex_ Collector의 combiner 메서드) 병합과정의 비용이 비싸면 병렬 스트림으로 얻은 성능의 이익이 서브스트림의 부분의 결과를 합치는 과정에서 상쇄될 수 있다.

-	스트림 소스와 분해성

	-	ArrayList : 훌륭함
	-	LinkedList : 나쁨
	-	IntStream.range : 훌륭함
	-	Stream.iterate : 나쁨
	-	HashSet : 좋음
	-	TreeSet : 좋음

##### 7.2 포크/조인 프레임워크

-	포크/조인 프레임워크는 병렬화할 수 있는 작업을 재귀적으로 작은 작업으로 분할한 다음 서브 태스크 각각의 결과를 합쳐서 전체결과를 만든다.

	-	서브태스크를 스레드 풀의 작업자 스레드에 분산 할당하는 ExcutorService 인터페이스를 구현한다.
	-	스레드 풀을 이용하려면 RecursiveTask<R>의 서비스클래스를 만들어야한다.

		-	RecursiveTask를 정의하려면 추상 메서드 compute를 구현해야한다.
		-	compute 메서드는 태스크를 서브태스크로 분할하는 로직과 더 이상 분할할 수 없을 때 개별 서브태스크의 결과를 생산할 알고리즘을 정의한다.

	-	분할 후 정복 알고리즘의 병렬화 버전이다.

		-	ForkJoinSumCalculator를 ForkJoinPool로 전달하면 풀 스레드가 ForkJoinSumCalculator의 compute 메서드를 실행하면서 작업을 수행한다.
		-	compute 메서드는 병렬로 실행할 수 있을 만큼 태스크의 크기가 충분히 작아졌는지 확인하며, 아직 태스크의 크기가 크다고판단되면 배열을 반으로 분할하여 두 개의 새로운 ForkJoinSumCalculator를 실행한다.
		-	이 과정이 재귀적을 반복됨

-	포크/조인 프레임워크를 효과적으로 사용하는 방법

	-	두 서브태스크가 모두 시작된 다음에 join을 호출해야한다. 그렇지 않으면 각각의 서브태스크가 끝나길 기다리는 일이 발생하여 순차보다 느릴 수 있다.
	-	RecursiveTask 내에서는 ForkJoinPoll의 invoke 메서드를 사용하지 말아야한다. 대신 compute나 fork 메서드를 직접 호출할 수 있다. (순차코드에서 병렬계산을 할때만 invoke를 사용한다)
	-	포크/조인 프레임워크에서는 fork라 불리는 다른 스레드에서 compute를 호출하므로 스택 트레이스가 도움되지 않는다.
	-	멀티코어에서 포크/조인 프레임워크를 사용하는 것이 순차 처리보다 무조건 빠를 거라는 생각은 버려야 한다.

-	포크/조인 분할 전략은 서브태스크를 더 분할할 것인지 결정하는 기준을 정해야하는데, 효율적이지 않게 분할되는 문제가 생길 수 있다. 해당 문제를 작업 훔치기 기법으로 해결할 수 있다.

	-	각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날 때 마다 큐의 헤드에서 다른 태스크를 가져와서 처리한다.
	-	빠르게 작업을 끝낸 스레드는 다른 스레드 큐 꼬리에서 작업을 훔쳐와서 처리할 수 있다.

##### 7.3 Spliterator 인터페이스

-	자바 8은 Spliterator 인터페이스를 제공한다. Iterator 처럼 소스의 요소 탐색 기능을 제공한다는 점은 같지만, 병렬 작업에 특화되어 있다.

	-	tryAdvance : Spliterator의 요소를 하나씩 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 참을 반환한다.
	-	trySplit : Spliterator의 일부 요소를 분할해서 두 번째 Spliterator를 생성한다.
	-	estimateSize : 탐색해야할 요소 수 정보 제공.
	-	charateristics : Spliterator의 자체 특성 집합을 포함하는 int 반환. (이를 참고하여 Spliterator을 더 잘 제어하고 최적화 가능)
		-	ORDERED
		-	DISTINCT
		-	SORTED
		-	SIZED
		-	NON-NULL
		-	IMMUTABLE
		-	CONCURRENT
		-	SUBSIZED

	```java
	public interface Spliterator<T>{
	    boolean tryAdvance(Consumer<? super T) action);
	    Spliterator<T> trySplit();
	    long estimateSize();
	    int characteristics();
	}
	```

-	분할 과정

	-	첫 번째 Spliterator에 trySplit을 호출하면 두 번째 Spliterator 생성됨
	-	두 개의 Spliteraotr에 trySplit을 호출하면 네 개의 Spliterator 생성됨
	-	trySplit 의 결과가 null이 될 때까지 반복

-	책에선 문자열 단의 수를 계산하는 예제를 커스텀 Spliterator를 활용하여 구현하는 예제를 설명한다. 265p

##### 7.4 마치며

-	내부반복을 이용하면 명시적으로 다른 스레드를 사용하지 않고도 스트림을 병렬로 처리할 수 있다.
-	항상 병렬처리가 빠른 것은 아니다.
-	올바른 자료구조 선택(기본형 특화 스트림 등)이 성능에 큰 영향을 미칠 수 있다.
-	Spliterator는 탐색하려는 데이터를 포함하는 스트림을 어떻게 병렬화할 것인지 정의한다.
