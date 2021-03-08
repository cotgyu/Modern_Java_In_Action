모던 자바 인 액션 Part 1
------------------------

#### Chap 1 자바 8,9,10,11 : 무슨 일이 일어나고 있는가?

##### 1.1 역사의 흐름은 무엇인가?

-	자바 8에서 가장 큰 변화가 일어났다. 다음의 코드를 간단한 방식으로 구현할 수 있다.

```java
Collection.sort(inventory, new Comparator<Apple>(){
  public int compare(Apple a1, Apple a2){
    return a1.getWeigth().compareTo(a2.getWeight());
  }
}

// 자바 8로 구현
inventory.sort(comparing(Apple::getWeight));
```

-	자바 8이 등장하기 이전에는 나머지 코어를 활용하려면 스레드를 사용하는 것이 좋다고 할 것임.

	-	하지만 스레드를 사용하면 관리하기 어렵고 많은 문제가 발생할 수 있음

-	자바는 병렬 실행환경을 쉽게 관리하고 에러가 덜 발생하는 방향으로 진화하려 노력했다.

	-	자바 1.0 : 스레드 와 락, 메모리 모델 지원 (온전히 활용하기 어려웠다.)
	-	자바 5 : 스레드 풀, 병렬 실행 컬렉션 등 강력한 도구 도입
	-	자바 7 : 병렬 실행에 도움을 줄 수 있는 포크/조인 프레임워크 제공 (여전히 개발자가 활용하기 어려움)
	-	자바 8 : 병렬 실행을 새롭고 단순한 방식으로 접근할 수 있는 방법 제공

-	자바 8에서 제공하는 새로운 기술

	-	스트림 API
	-	데이터베이스 질의 언어에서 표현식을 처리하는 것 처럼 병렬연산을 지원
	-	스트림을 이용하면 에러를 자주 일으키며 멀티코어 CPU를 이용하는 것보다 훨씬 비싼 키워드 sychronized를 사용하지 않아도 됨
	-	메서드에 코드를 전달하는 기법
	-	인터페이스의 디폴트 메서드

##### 1.2 왜 아직도 자바는 변화하는가?

-	이 책은 병렬성을 활용하는 코드, 간결한 코드를 구현할 수 있도록 자바8에서 제공하는 기능의 모태인 세 가지 프로그래밍 개념을 자세히 설명한다.

	-	스트림 처리 : 스트림이란 한번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임이다.

		-	스트림 API의 핵심은 기존에는 한 번에 한 항목을 처리했지만 이제 자바8에서는 우리가 하려는 작업을 고수준으로 추상화해서 일련의 스트림으로 만들어 처리할 수 있다는 것이다.

		-	또한 스트림 파이프 라인을 이용해서 입력 부분을 여러 CPU 코어에 쉽게 할당할 수 있다는 부가적인 이득도 얻을 수 있다. 스레드라는 복잡한 작업을 사용하지 않으면서도 공짜로 병렬성을 얻을 수 있다.

	-	동작 파라미터화(코드 일부를 API로 전달하는 기능) : 자바 8에서는 메서드를 다른 메서드의 인수로 넘겨주는 기능을 제공한다.

	-	병렬성과 공유 가변데이터 : 스트림 메서드로 전달하는 코드는 다른 코드와 동시에 실행하더라도 안전하게 실행될 수 있어야한다.

		-	안전하게 실행할 수있는 코드를 만들려면 공유된 가변 데이터에 접근하지 않아야 한다. (순수함수pure, 부작용없는 함수side-effect-free, 상태없는 함수stateless)

##### 1.3 자바함수

-	런타임에 메서드를 전달할 수 있다면, 즉 메서드를 일급 시민으로 만들면 프로그래밍에 유용하게 활용할 수 있다.

> File[] hiddenFiles = new File(".").listFiles(File::isHidden);

-	자바8의 메서드참조method reference :: ('이 메서드를 값으로 사용하라'는 의미) 를 이용해서 listFiles에 직접 전달할 수 있다.

-	자바8에서는 메서드를 일급값으로 취급할 뿐 아니라 람다(익명함수)를 포함하여 함수도 값으로 취급할 수 있다.

	-	위의 개념에 대한 예제이다. 필터링하는 소스를 자바8 전, 후로 비교하여 메서드를 전달하는 것으로 보여주고 있음. (51p~53p)
	-	프레디케이트 : true나 false를 반환하는 함수

		```java
		// 메서드 전달
		filterApples(inventory, Apple::isGreenApple);


		// 람다 전달
		filterApples(inventory, (Apple a) -> a.getWeight() < 80 || GREEN.equals(a.getColor()) );
		```

> 한 번만 사용할 메서드는 따로 정의를 구현할 필요가 없지만(람다), 람다가 몇 줄 이상으로 길어진다면 익명람다보다는 코드가 수행하는 일을 잘 설명하는 일므을 가진 메서드를 정의하고 메서드 참조를 활용하는 것이 바람직하다.

##### 1.4 스트림

-	스트림 API를 이용하여 중첩된 제어 흐름 문장이 많아지는 문제를 해결할 수 있다. (4장 ~ 7장에 걸쳐 설명함)

	-	컬렉션에서는 반복 과정을 직접 처리해야한다. (외부반복)
	-	스트림을 이용하면 라이브러리 내부에서 모든 데이터가 처리된다. (내부반복)

-	스트림 API(java.util.stream)로 '컬렉션을 처리하면서 발생하는 모호함고 반복적인 코드 문제', '멀티코어 활용 어려움'이라는 두 가지 문제를 모두 해결해였다.

-	컬렉션은 어떻게 데이터를 저장하고 접근할지에 중점을 두는 반면 스트림은 데이터에 어떤 계산을 할 것인지 묘사하는 것에 중점을 둔다.

-	스트림은 스트림 내의 요소를 쉽게 병렬로 처리할 수 있는 환경을 제공한다는 것이 핵심 (라이브러리 내에서 분할처리)

##### 1.5 디폴터 메서드와 자바 모듈

-	인터페이스를 업데이트하려면 해당 인터페이스를 구현하는 모든 클래스도 업데이트가 필요하지만 자바8에서는 디폴트 메서드로 문제를 해결할 수 있다.

	-	자바9의 모듈시스템은 모듈을 정의하는 문법을 제공하므로 이를 이용해 패키지 모음을 포함하는 모듈을 정의할 수 있다. (14장)

		-	모듈 덕분에 JAR 같은 컴포넌트에 구조를 적용할 수 있으며 문서화와 모듈확인 작업이 용이해졌다.

	-	자바8에서는 인터페이스를 쉽게 바꿀 수 있도록 디폴트 메서드를 지원한다. (13장)

	-	구현클래스에서 구현하지 않아도 되는 메서드를 인터페이스에 추가할 수 있는 기능을 제공한다. 메서드 본문은 클래스 구현이 아니라 인터페이스의 일부로 포함된다

##### 1.6 함수형 프로그래밍에서 가져온 다른 유용한 아이디어

-	자바에 포함된 함수형 프로그래밍의 핵심적인 두 아이디어

	-	메서드와 람다를 일급값으로 사용하는 것
	-	가변 공유 상태가 없는 병렬실행을 이용해서 효율적으로 안전하게 함수나 메서드를 호출할 수 있다는 것

-	자바8에서는 NullPointer 예외를 피할 수 있도록 도와주는 Optional<T> 클래스를 제공한다.

	-	Optional<T>는 값을 갖거나 가지 않을 수있는 컨테이너 객체다.
	-	값이 없는 상황을 어떻게 처리지 명시적으로 구현하는 메서드를 가지고있음.

-	자바 8의 스트림 개념 중 일부는 컬렉션에서 가져온 것이다. 스트림과 컬렉션을 적절하게 활용하면 스트림의 인수를 병렬로 처리할 수 있으며 더 가독성이 좋은 코드를 구현할 수 있다.

---

#### Chap2 동작 파라미터화 코드 전달하기

-	동작 파라미터화를 이용하면 자주 바뀌는 요구사항에 효과적으로 대응할 수 있다.

> 어떻게 실행할 것인지 결정하지 않은 코드블록을 의미하며 코드 블록의 실행을 나중으로 미룬다.

-	해당 챕터에서는 변화하는 요구사항에 유연하게 대응할 수 있게 코드를 구현하는 방법을 예제를 이용해서 살펴본다.

##### 2.1 변화하는 요구사항에 대응하기

-	2.1에서는 사과를 필터링하면서 변화하는 요구사항에 대해 대처하는 것을 예제를 통해 보여준다.

	```java
	// 색에 따라 필터링
	public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color){
	  List<Apple> result = new ArrayList<>();
	  for(Apple apple: inventory){
	    if(apple.getColor().equals(color) ){
	      result.add(apple);
	    }
	  }
	  return result;
	}


	// 무게에 따라 필터링
	public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight){
	  List<Apple> result = new ArrayList<>();
	  for(Apple apple: inventory){
	    if(apple.getWeight() > weight ){
	      result.add(apple);
	    }
	  }
	  return result;
	}
	```

##### 2.2 동작 파라미터화

-	프레디케이트 : 선택 조건을 결정하는 인터페이스

	```java
	// 사과 선택전략을 캡슐화한다.
	public interface ApplePredicate{
	  boolean test (Apple apple);
	}


	// 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있음
	public class AppleHeavyWeightPredicate implements ApplePredicate{
	  public boolean test(Apple apple){
	    return apple.getWeight() > 150;
	  }
	}


	public class AppleGreenColorPredicate implements ApplePredicate{
	  public boolean test(Apple apple){
	    return GREEN.equals(apple.getColor());
	  }
	}
	```

-	위의 전략 디자인 패턴이라고 한다.

	-	전략 디자인 패턴 : 각 알고리즘(전략)을 캡슐화하는 알고리즘 패밀리를 정해해둔 다음에 런타임에 알고리즘을 선택하는 기법

	```java
	// 위의 Predicate 를 사용한 코드
	public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p){
	  List<Apple> result = new ArrayList<>();
	  for(Apple apple : inventory){
	    if( p.test(apple) ){
	      result.add(apple);
	    }
	  }
	  return result;
	}


	// 필터 사용하기. 전달한 ApplePredicate 객체에 의해 filterApples 메서드의 동작이 결정됨.
	List<Apple> greenApples = filterApples(inventory, new AppleGreenColorPredicate());
	```

-	퀴즈 2-1 우연한 prettyPrintApple 메서드 구현하기

	```java
	public interface ToStringPredicate{
	  String test (Apple apple);
	}


	public class AppleWeightToStringPredicate implements ToStringPredicate p{
	  public String test(Apple apple){
	    return Integer.toString(test.getWeight);
	  }
	}
	```

##### 2.3 복잡한 과정 간소화

-	자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 익명클래스라는 기법을 제공한다. 익명클래스를 이용하면 코드의 양을 줄일 수 있다.

	-	즉석에서 필요한 구현을 만들어서 사용할 수 있다.

-	익명클래스를 이용해서 ApplePredicate를 구현하는 객체를 만드는 방법으로 필터링하기

	```java
	List<Apple> redApples = filterApples(inventory, new ApplePredicate(){
	  public boolean test(Apple apple){
	    return RED.equals(apple.getColor());
	  }
	});
	```

-	하지만 여전히 만족스럽지 않다. 선택기준을 가리키는 불리언 표현식을 전달하는 과정에서 결국 객체를 만들고 명시적으로 새로운 동작을 정의하는 메서드를 구현해야 한다는 점은 변하지 않는다.

-	람다 표현식을 사용하면 다음처럼 간단하게 재구현 가능하다.

	```java
	List<Apple> result = filterApples(invetory, (Apple apple) -> RED.equals(apple.getColor()));
	```

-	동작 파라미터화의 실전 활용

	-	컬렉션 정렬 (Comparator로 정렬하기)

		```java
		// Comparator를 구현하여 sort 동작 다양화하기
		inventory.sort(new Comparator<Apple>(){
		  public int compare(Apple a1, Apple a2){
		    return a1.getWeight().compareTo(a2.getWeight());
		  }
		});


		// 람다
		inventory.sort( (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()) );
		```

	-	Runnable로 코드 블록 실행하기

		```java
		Thread t = new Thread(new Runnalbe(){
		  public void run(){
		    System.out.println("Hello world");
		  }
		});


		// 람다
		Thread t = new Thread( ()-> System.out.println("Hello world") );
		```

---

#### Chap3 람다 표현식

-	해당 장에서는 더 깔끔한 코드로 동작을 구현하고 전달하는 자바 8의 새로운 기능인 람다표현식을 설명한다.

> 람다표현식은 익명 클래스처럼 이름 없는 함수면서 메서드를 인수로 전달할 수 있으므로 일단은 람다 표현식이 익명 클래스와 비슷하다고 생각하자.

##### 3.1 람다란 무엇인가?

-	람다표현식은 메서드로 전달할 수 있는 익명함수를 단순화한 것이라고 할 수 있다.

	-	이름은 없지만, 파라미터 리스트, 바디, 반환형식, 발생할 수 있는 예외리스트는 가질 수 있다.

-	람다표현식은 파라미터, 화살표, 바디 로 이루어진다.

	```java
	//   람다 파라미터       화살표                  람다 바디
	(Apple a1, Apple a2)   ->   a1.getWeight().compareTo(a2.getWeight());
	```

	-	파라미터 리스트 : Comparator의 compare 메서드 파라미터
	-	화살표 : 람다의 파라미터 리스트와 바디를 구분한다.
	-	람다 바디 : 두 사과의 무게를 비교한다. 람다의 반환값에 해당하는 표현식이다.

-	자바8의 유용한 람다 표현식

	```java


	  // int를 반환한다. 람다 표현식에는 return이 함축되어 있으므로 return문을 명시적으로 사용하지 않아도 된다.
	  (String s) -> s.length()


	  // boolean을 반환
	  (Apple a) -> a.getWeight() > 150


	  // 람다표현식은 여러 행의 문장을 포함할 수 있다.
	  (int x, int y) -> {
	    System.out.println("Result: ");
	    System.out.println(x + y);
	  }


	  // 파라미터가 없으며 int 42를 반환한다.
	  () -> 42


	  // int(두 사과의 무게 비교 결과)를 반환한다.
	  (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())
	```

-	람다 예제

	```java
	(List<String> list) -> list.isEmpty() // 불리언 표현식
	() -> new Apple(10) // 객체 생성
	(Apple a) -> {System.out.println(a.getWeight())} //객체에서 소비
	(String s) -> s.length() //객체에서 선택추출
	(int a, int b) -> a * b // 두 값을 조합
	(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()) // 두 객체 비교
	```

##### 3.2 어디에, 어떻게 람다를 사용할까?

-	함수형 인터페이스라는 문맥에서 람다 표현식을 사용할 수 있다.
	-	정확히 하나의 추상메서드를 지정하는 인터페이스 (Comparator, Runnable 등)
	-	인터페이스는 디폴트메서드를 포함할 수 있다. 많은 디폴트메서드가 있더라도 추상메서드가 하나면 함수형인터페이스이다.

> 람다 표현식으로 함수형 인터페이스의 추상메서드 구현을 직접 전달할 수 있으므로 전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.

```java
//람다 사용
Runnable r1 = () -> System.out.println("Hello World 1");

//익명 클래스 사용
Runnable r2 = new Runnable() {
  public void run(){
    System.out.println("Hello World2");
  }
};

public static void process(Runnable r){
  r.run();
}

process(r1);
process(r2);
process( () -> System.out.println("Hello World 3")); // 직접 전달된 람다 표현식으로 출력

```

-	새로운 자바 API를 살펴보면 함수형 인터페이스에 @FunctionalInterface 어노테이션이 추가되었다. 해당 어노테이션은 함수형 인터페이스임을 가리키는 것으로 선언했지만 실제로 함수형인터페이스가 아니면 컴파일러가 에러를 발생시킨다.

##### 3.3 람다 활용 : 실행 어라운드 패턴

-	람다와 동작파라미터화로 유연하고 간결한 코드를 구현하는 데 도움을 주는 실용적인 에제

	-	자원처리에 사용하는 순환패턴은 자원을 열고, 처리한 다음에, 자원을 닫는 순서로 이루어진다. : 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다. (실행 어라운드 패턴)

	-	예제를 통해 람다를 활용하는 과정을 설명하고 있다.

	```java
	    // try-with-resource 구문을 사용한 파일에서 한 행을 읽기
	    public String processFile() throws IOException{
	      try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
	        return br.readLine();
	      }
	    }


	    // 동작파라미터화를 사용한 두 행을 출력하기
	    String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());


	    // 함수형 인터페이스를 이용한 동작 전달하기
	    @FunctionalInterface
	    public interface BufferedReaderProcessor{
	      String process(BufferedReader b) throws IOException;
	    }


	    // 동작 실행
	    public String processFile(BufferedReaderProcessor p) throws IOException{
	      try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
	        return p.process(br);
	      }
	    }


	    // 람다 전달을 통해 다양한 동작을 processFile 메서드로 전달하기
	    String oneLine = processFile((BuffferedReader br) -> br.readLine());


	    String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
	```

##### 3.4 함수형 인터페이스 사용

-	함수형 인터페이스는 오직 하나의 추상 메서드를 지정한다. 추상 메서드는 람다 표현식의 시그니처를 묘사한다. (함수 디스크립터)

-	자바 8 라이브러리 설계자들은 java.util.funciton 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다. (그 중 Predicate, Consumer, Function 인터페이스를 설명한다.)

-	Predicate : test라는 추상 메서드를 정의하며 test는 제네릭 형식 T의 객체를 인수로 받아 불리언을 반환한다.

	```java
	@FunctionalInterface
	public interface Predicate<T>{
	  boolean test(T t);
	}


	public <T> Lint<T> filter(List<T> list, Predicate<T> p){
	  List<T> results = new ArrayList<>();


	  for(T t: list){
	    if(p.test(t)){
	      results.add(t);
	    }
	  }


	  return result;
	}


	Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
	List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
	```

-	Consumer : 제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상메서드를 정의한다.

	-	T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 Consumer 인터페이스를 사용할 수 있다.

	```java
	@FunctionalInterface
	public interface Consumer<T>{
	  void accept(T t);
	}


	public <T> void forEach(List<T> list, Consumer<T> c){
	  for(T t: list){
	    c.accept(t);
	  }
	}


	forEach( Arrays.asList(1,2,3,4,5), (Integer i) -> System.out.println(i) );
	```

-	Function : 제네릭 형식 T 객체를 인수로 받아서 제네릭 형식 R객체를 반환하는 추상 메서드 apply를 정의한다.

	```java
	// String 인수로 받아 각 String의 길이를 포함하는 Integer 리스트 변환하는 map 메서드
	@FunctionalInterface
	public interface Function<T, R>{
	  R apply(T t);
	}


	public <T, R> List<R> map(List<T> list, Function<T, R> f){
	  List<R> result = new ArrayList<>();


	  for(T t: list){
	    result.add(f.apply(t));
	  }
	  return result;
	}


	List<Integer> l = map(
	    Arrays.asList("lamdas", "in", "action"),
	    (String s) -> s.lenth()
	);
	```

-	자바 8에서는 기본형을 입출력으로 사용하는 상황에서 오토박싱 동작을 피할 수 있도록 특별한 버전의 함수형 인터페이스를 제공한다.

	-	기본형 특화 105p 참조
	-	람다와 함수형인터페이스 예제 106p 참조

-	함수형 인터페이스는 확인된 예제를 던지는 동작을 허용하지 않는다.

	-	예외를 던지는 람다표현식을 만드려면 람다를 try/catch 블록으로 감싸야한다.

	```java
	Function<BufferedReader, String> f = (BufferedReader b) -> {
	  try{
	    return b.readLine();
	  }catch(IOException e){
	    throw new RuntimeException(e);
	  }
	};
	```

##### 3.5 형식 검사, 형식 추론, 제약

-	컴파일러가 람다의 형식을 어떻게 확인하는지, 피해야할 사항은 무엇인지 알아본다.

-	람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구햔하는지의 정보가 포함되어 있지 않다.

	-	람다 표현식을 더 제대로 이해하려면 람다의 실제 형식을 파악해야 한다.

-	형식 검사

	-	람다가 사용되는 context를 이용해서 람다의 형식을 추론할 수 있다.
	-	람다가 전달될 메서드 파라미터나 람다가 할당되는 변수를 통해 추론

-	같은 람다, 다른 함수형 인터페이스

	-	대상형식 이라는 특징 때문에 같은 람다 표현식이라도 호환되는 추상메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.

		```java
		Comparator<Apple> c1 = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.Weight());


		ToIntBiFunction<Apple> c1 = (Apple a1, Apple a2) ->  a1.getWeight().compareTo(a2.Weight());
		```

	-	특별한 void 호환 규칙

		-	람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 스크립터와 호환된다.(물론 파라미터 리스트도 호환되어야 함)

-	형식 추론

	-	자바 컴파일러는 람다 표현식이 사용된 콘텍스트를 이영해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다. (대상형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있음)

> 컴파일러는 람다 표현식의 파라미터 형식에 접근할 수 있으므로 람다 문법에서 이를 생략할 수 있다.

```java
  // 형식을 추론하지 않음
  Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.Weight());

  // 형식 추론
  Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.Weight());
```

-	지역 변수 사용

	-	람다 캡쳐링 : 익명함수가 하는 것 처럼 외부에서 정의된 변수(자유변수)를 활용할 수 있는 동작

	-	자유 변수의 제약 : 람다 표현식은 한 번만 할당할 수 있는 지역 변수를 캡쳐할 수 있다. (final)

		-	람다가 지역변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려할 수 있다. 따라서 자바 구현에는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 따라서 복사본의 값이 바뀌지 않아야 하므로 지역 변수에서는 한 번만 값을 할당해야 한다는 제약이 생긴 것이다.

		```java
		// 컴파일 할 수 없는 코드
		int portNumber = 1337;
		Runnalbe r = () -> System.out.println(portNumber);
		portNumber = 31337;
		```

##### 3.6 메서드 참조

-	메서드 참조를 이용하면 기존의 메서드 정의를 재활용해서 람다처럼 전달할 수 있다. (**가독성을 높일 수 있다.**\)

	```java
	//기존 코드
	inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));


	//메서드참조와 java.util.Comparator.comparing을 활용 코드
	inventory.sort(comparing(Apple::getWeight));
	```

-	메서드 참조를 만드는 방법

	-	정적메서드 참조 : Integer 의 parseInt 메서드는 Integer::parseInt
	-	다양한 형식의 인스턴스 메서드 참조 : String 의 length 메서드는 String::length
	-	기존 객체의 인서턴스 메서드 참조 : Transaction 객체를 할당받은 expensiveTransaction 지역변수가 있고, Transaction 객체에 getValue 메서드가 있다면, expensiveTransaction::getValue

-	생성자 참조

	-	ClassName::new 처럼 클레스명과 new 키워드를 이용해서 기존 생성자의 참조를 만들 수 있다.

		```java
		  // Supplier 의 get메서드를 호출해서 새로운 Apple 객체를 만들 수 있다.
		  Supplier<Apple> c1 = () -> new Apple();
		  Apple a1 = c1.get();


		  // 다음과 같이 만들 수 있다.
		  Supplier<Apple> c1 = Apple::new;
		  Apple a1 = c1.get();


		  // Apple (Integer weight)라는 시그니처를 갖는 생성자는 Function 인터페이스와 시그니처와 같다.
		  Function<Integer, Apple> c2 = (weigth) -> new Apple(weigth);
		  Apple a2 = c2.apply(110);


		  // 다음과 같이 만들 수 있다.
		  Function<Integer, Apple> c2 = Apple::new; // Apple(Integer weight)의 생성자 참조
		  Apple a2 = c2.apply(110);


		```

-	인수가 세 개인 생성자의 생성자 참조는 어떻게?

	-	해당 시그니처를 제공하는 함수형 인터페이스를 직접만들어야한다.

		```java
		  public interface TriFunction<T, U, V, R>{
		    R.apply(T t, U u, V v);
		  }
		```

##### 3.7 람다, 메서드 참조 활용하기

-	사과 리스트를 정렬하는 문제를 동작파라미터화, 익명 클래스, 람다 표현식, 메서드 참조 등을 동원하여 간결하게 만든다.

	```java


	// 1단계 : 코드 전달
	public class AppleComparator implements Comparator<Apple>{
	  public int compare(Apple a1, Apple a2){
	    return a1.getWeight().compareTo(a2.Weight());
	  }
	}


	inventory.sort(new AppleComparator());


	// 2단계 : 익명 클래스 사용
	// 1단계처럼 한번 사용할 Comparator를 구현하는 것보다는 익명클래스를 이용하는 것이 좋다.
	inventory.sort(new Comparator<Apple>(){
	  public int compare(Apple a1, Apple a2){
	    return a1.getWeight().compareTo(a2.Weight());
	  }
	});


	// 3단계 : 람다 표현식 사용
	// 추상메서드의 함수디스크립터 (Comparator의 함수 디스크립터는 (T,T) -> int)
	inventory.sort((Apple a1, Apple a2) ->
	    a1.getWeight().compareTo(a2.Weight())
	);


	// 형식 추론
	inventory.sort((a1, a2) ->
	    a1.getWeight().compareTo(a2.Weight())
	);


	// 정적 메서드 comparing
	Comparator<Apple> c = Comparator.comparing((Apple a) -> a.getWeight());


	// 간소화
	import static java.util.Comparator.comparing;
	inventory.sort(comparing(apple -> apple.getWeight()));


	// 4단계 : 메서드 참조 사용
	inventory.sort(comparing(Apple::getWeight));


	```

##### 3.8 람다 표현식을 조합할 수 있는 유용한 메서드

-	Comparator, Function, Predicate 같은 함수형 인터페이스는 람다표현식을 조합할 수 있도록 유틸리티 메서드를 제공한다.

> 여러 개의 람다표현식을 좋바해서 복잡한 람다표현식을 만들 수 있다.

-	Comparator 조합

	-	reverse, thenComparing 제공

	```java
	inventory.sort(comparing(Apple::getWeight)
	       .reversed() // 내림차순으로 정렬
	       .thenComparing(Apple::getCountry) // 두 사과의 무게가 같으면 국가별로 정렬
	);
	```

-	Predicate 조합

	-	negate, and, or 제공

	```java
	// negate 특정 프레디케이트를 반전
	Predicate<Apple> notRedApple = redApple.negate();


	// and 를 이용한 조합
	Predicate<Apple> redAndHeavyApple = redApple.and(apple -> apple.getWeight() > 150);


	// or 를 이용한 조건
	Predicate<Apple> = redAndHeavyAppleOrGreen =
	redApple.and(apple -> apple.getWeight() > 150)
	        .or(apple -> GREEN.equals(a.getColor()));


	```

-	Function 조합

	-	andThen, compose 제공

	```java
	// andThen : 주어신 함수를 먼저 적용한 결과를 다른 함수의 입력으로 전달하는 함수
	Function<Integer, Integer> f = x -> x+1;
	Function<Integer, Integer> g = x -> x+2;
	Function<Integer, Integer> h = f.andThen(g); // g(f(x))
	int result = h.apply(1); // 4 반환


	// compose : 인수로 주어진 함수를 먼저 실행한 다음에 그 결과를 외부 함수의 인수로 제공
	Function<Integer, Integer> f = x -> x+1;
	Function<Integer, Integer> g = x -> x+2;
	Function<Integer, Integer> h = f.compose(g); // f(g(x))
	int result = h.apply(1); // 3 반환
	```

##### 3.10 마치며

-	람다 표현식은 익명 함수의 일종이다. 이름은 없지만, 파라미터 리스트, 바디, 반환 형식을 가지며 예외를 던질 수 있다.

-	함수형 인터페이스는 하나의 추상 메서드만들 정의하는 인터페이스다.

-	람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉성으로 제공할 수 있으며 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다.

-	자바 8은 Predicate<T> 와 Function<T,R> 같은 제네릭 함수형 인터페이스와 관련한 박싱동작을 피할 수 있는 IntPredicate, IntToLongFunction 등과 같은 기본형 특화 인터페이스도 제공한다.

-	메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.

-	Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는 다양한 디폴트 메서드를 제공한다.
