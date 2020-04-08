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
