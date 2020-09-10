모던 자바 인 액션 Part 4
------------------------

#### Chap 11 : null 대신 Optional 클래스

-	NullPointerException은 다양한 예외 중 자주 겪는 예외이다.

##### 11.1 값이 없는 상황을 어떻게 처리할까?

```java
// 자동차와 자동차 보험을 갖고 있는 사람 객체 중첩 구조
public class Person {
	private Car car;
	public Car getCar() {
		return car;
	}
}

public class Car {
	private Insurance insurance;
	public Insurance getInsurance() {
		return insurance;
	}
}

public class Insurance {
	private String name;
	public String getName() {
		return name;
	}
}


// 중간 값들이 null일 경우 NullPointerException 발생!
public String getCarInsuranceName(Person person) {
	return person.getCar().getInsurance().getName();
}

// null 안전 시도  
// (모든 변수를 의심하고 있다. 변수에 접근할 때마다 중첩된 if이 추가되면서 코드 들여쓰기 수준이 증가된다. : 깊은 의심 - 가독성이 떨어진다.)
public String getCarInsuranceName(Person person) {
	if(person != null){
		Car car = person.getCar();
		if(car != null){
			Insurance insurance = car.getInsurance();

			if(insurance != null){
				return insurance.getName();
			}
		}
	}
	return "Unknown";
}

// 중첩 if블록 줄이기.
// (많은 출구가 생기는데, 출구 때문에 유지보수가 어려워질 수 있다.)
public String getCarInsuranceName2(Person person){
	if(person == null){
		return "Unknown";
	}

	Car car = person.getCar();

	if(car == null){
		return "Unknown";
	}

	Insurance insurance = car.getInsurance();

	if(insurance == null){
		return "Unknown";
	}

	return insurance.getName();
}
```

-	null 때문에 발생하는 문제

	-	에러의 근원이다 : NullPointerException은 자바에서 가장 흔히 발생하는 에러다.
	-	코드를 어지럽힌다 : 중첩된 null 확인 코드를 추가해야 하므로 null 떄문에 코드 가독성이 떨어진다.
	-	아무 의미가 없다 : null은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.
	-	자바 철학에 위배된다 : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 null 포인터가 예외다.
	-	형식 시스템에 구멍을 만든다 : null은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null을 할당할 수 있다. 이런 식으로 null이 할당되기 시작되면서 시스템의 다른부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다.

-	다른 언어에서는 ?

	-	최신 그루비언어에서는 안전 내비게이션 연산자(?.) 을 도입함.

		-	def carInsuranceName = person?.car?.insurance?.name

	-	하스켈, 스칼라 등의 함수형 언어에서는 선택형값을 저장할 수있는 Maybe 형식, Option[T] 제공

##### 11.2 Optional 클래스 소개

-	자바8은 하스켈과 스칼라의 영향을 받아서 java.util.Optional<T> 라는 새로운 클래스를 제공한다.

	-	Optional은 선택형값을 캡슐화하는 클래스다.
	-	값이 있으면 Optional 클래스는 값을 감싼다. 값이 없으면 Optional.empty 메서드로 Optional을 반환한다.
	-	Optional을 이용하면 값이 없는 상황이 우리 데이터에 문제가 있는 것인지 아니면 알고리즘의 버그인지 명확하게 구분할 수 있다.
	-	모든 null 참조를 Optional로 대치하는 것은 바람직하지 않다.
	-	Optional의 역할은 더 이해하기 쉬운 API를 설계하도록 돕는 것이다.

		-	메서드의 시그니처만 보고도 선택형 값인지 여부를 구별할 수 있다.
		-	Optional이 있다면 값이 없을 수 있는 상황에서 적절하게 대응하도록 강제하는 효과가 있다.

		```java
		// Optional을 활용한 데이터모델 재정의
		public class Person{
		    private Optional<Car> car;
		    private Optional<Car> getCar(){
		        return car;
		    }
		}


		public class Car{
		    private Optional<Insurance> insurance;
		    private Optional<Insurance> getInsuracne(){
		        return insurance;
		    }
		}


		// 반드시 값이 있어야하는 경우 사용 X (없는 경우는 문제를 해결해야함)
		public class Insurance{
		    private String name;
		    public String getName(){
		        return name;
		    }
		}


		```

##### 11.3 Optional 적용 패턴

-	Optional 객체 만들기

	-	빈 Optional

		-	Optional<Car> optCar = Optional.empty();

	-	null이 아닌 값으로 Optional 만들기 (car가 null이면 바로 NullPointerException 발생)

		-	Optional<Car> optCar = Optional.of(car);

	-	null 값으로 Optional 만들기 (car가 null이면 빈 Optional 객체가 반환됨)

		-	Optional<Car> optCat = Optional.ofNullable(car);

-	맵으로 Optional의 값 추출하고 변환하기

	-	Optional이 비어있으면 get을 호출했을 때 예외가 발생한다. 즉, 잘못사용하면 null을 사용했을 때와 같은 문제를 겪을 수 있다. 따라서 먼저 Optional로 명시적인 검사를 제거할 수 있는 방법을 살펴본다.

		```java
		// 보험회사의 이름을 추출한다고 가정한다. (정보에 접근하기전에 null 체크를 해야함)
		String name = null;
		if(insurance != null){
		    name = insurance.getName();
		}


		// map을 사용 (Optional이 값을 포함하면 map의 인수로 제공된 함수가 값을 바꾼다. Optional이 비어있으면 아무일도 일어나지 않음)
		Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
		Optional<String> name = optInsurance.map(Insurance::getName);
		```

-	flatMap으로 Optional 객체 연결

	```java
	// 컴파일되지 않음!
	// getCar의 반환 값이 Optional<Car> 형식이기 때문에 map의 연산결과는 Optional<Optional<Car>> 가 된다.
	Optional<Person> optPerson = Optional.of(person);
	Optional<String> name = optPerson.map(Person::getCar).map(Car::getInsurance).map(Insurance::getName);


	// 스트림의 flatMap은 함수를 인수로 받아서 다른 스트림을 반환함.
	public String getCarInsuranceName(Optional<Person> person){
	    return person.flatMap(Person::getCar).flatMap(Car::getInsurance).map(Insurance::getName).orElse("Unknown");
	}
	```

> Optional을 인수로 받거나 Optional을 반환하는 메서드를 정의한다면 결과적으로 이 메서드를 사용하는 모든 사람에게 이 메서드가 빈 값을 받거나 빈 결과를 반환할 수 있음을 잘 문서화해서 제공하는 것과 같다.

-	Optional 클래스는 필드 형식으로 사용할 것을 가정하지 않았으므로 Serializable 인터페이스를 구현하지 않는다.

	-	도메인 모델에 Optional을 사용한다면 직렬화 모델을 사용하는 도구나 프레임워크에서 문제가 생길 수 있다.
	-	직렬화 모델이 필요하다면 Optional로 값을 반환받을 수 있는 메서드를 추가하는 방식을 권장한다.

		```java
		public class Person{
		    private Car car;
		    public Optional<Car> getCarAsOptional(){
		        return Optional.ofNullable(car);
		    }
		}
		```

-	Optional 스트림 조작

	-	자바9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다. (Optional 스트림을 값을 가진 스트림으로 변환할 때 이 기능을 유용하게 활용할 수 있다.)

		```java
		public Set<String> getCarInsuranceNames(List<Person> persons){
		    return persons.stream()
		        .map(Person::getCar)
		        .map(optCar -> optCar.flatMap(Car::getInsurance))
		        .map(optIns -> optIns.flatMap(Insurance::getNames))
		        .flatMap(Optional::stream)
		        .collect(toSet());
		}
		```

-	디폴트 액션과 Optional 언랩

	-	앞에서 빈 Optional 상황에서 기본 값을 반환하도록 orElse로 읽었다. Optional 클래스는 Optional 인스턴스에 포함된 값을 읽는 다양한 방법을 제공한다.

		-	get() : 값을 읽는 가장 간단한 방법이지만, 안전하지 않다. (값이 없는 경우 문제) 따라서 Optional에 값이 반드시 있다고 가정할 수 있는 상황이 아니면 get메서드를 쓰지 않는 것이 좋다.

		-	orElse : Optional이 값이 포함할지 않을 때 기본 값 제공가능

		-	orElseGet(Supplier<? extends T> other) : orElse 메서드에 대응히는 게으른 버전 메서드.

			-	Optional에 값이 없을 때만 Supplier 가 실행 됨.
			-	디폴트메서드를 만드는 데 시간이 걸리거나 Optional이 비어있을 때만 기본 값을 생성하고 싶다면 사용

		-	orElseThrow(Supplier<? extends X> exceptionSupplier) : Optional이 비어있을 때 에외를 발생시킴. (예외 종류 선택가능)

		-	ifPresent(Consumer<? super T> consumer) : 값이 존재할 때 인수로 넘겨준 동작 실행 가능. (값이 없으면 아무 일도 일어나지 않음)

		-	ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) : 자바 9에서 추가됨. Optional이 비었을 때 실행할 수 있는 Runnable을 인수로 받음.

-	두 Optional 합치기

	-	두 Optional을 인수로 받아 Optional<Insurance>을 반환하는 null 안전 버전 메서드

		```java
		// null 확인 코드와 크게 다른 점이 없다!
		public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car){
		    if(person.isPresent() && car.isPresent()){
		        return Optional.of(findCheapestInsurance(person.get(), car.get()));
		    }else{
		        return Optional.empty();
		    }
		}


		// map과 flat map 활용
		// 첫ㅎ번째 Optional에 flatMap을 호출했으므로 Optional이 비어있다면 인수로 전달한 람다표현식이 실행되지 않고 그대로 빈 Optional이 반환됨.
		public Optional<Insurance> nullSafeFindCheapestInsurance2(Optional<Person> person, Optional<Car> car){
		    return person.flatMap(p -> car.map( c -> findCheapestInsurance(P, c)));
		}
		```

-	필터로 특정 값 거르기

	-	객체의 메서드를 호출해서 어떤 프로퍼티를 확인해야할 때가 있다. 이때도 null체크가 필요하다.

	-	filter 메서드는 프레디케이트를 인수로 받는다. Optional 객체가 값을 가지며 프레디케이트와 일치하면 filter 메서드는 그 값을 반환하고 그렇지 않으면 빈 Optional 객체를 반환한다.

		-	Optional은 최대 한 개 요소를 포함할 수 있는 스트림과 같다.
		-	Optional이 비어있다면 filter 연산은 아무 동작하지 않음. 값이 있다면 그 값에 프레디케이트 적용

		```java
		Insurance insurance = ...;


		if(insurance != null && "CambridgeInsurance".equals(insurance.getName())){
		    System.out.println("ok");
		}


		// Optional 객체에 filter 사용하여 재구현
		Optional<Insurance> optInsurance = ...;


		optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName())).ifPresent( x -> System.out.println("ok"));
		```

-	383p [표 11-1] : Optional 클래스의 메서드를 보여줌. 참고할 것

##### 11.4 Optional을 사용한 실용 예제

-	기본 자바 API는 Optional을 적절하게 활용하지 못하고 있다. Optional 기능을 활용할 수 있도록 코드에 작은 유틸리티 메소드를 추가하는 방식으로 이 문제를 해결할 수 있다.

-	잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

	-	기존 자바API에서는 null을 반환하면서 요청한 값이 없거나 어떤 문제로 계산에 실패했음을 알린다.
	-	null을 반환하는 것보다는 Optional을 반환하는 것이 더 바람직하다.
	-	get메서드의 시그니처는 고칠 수 없지만 get메서드의 반환 값은 Optional로 감쌀 수 있다.

		```java
		Object value = map.get("key");


		// 개선
		Optional<Object> value2 = Optional.ofNullable(map.get("key"));
		```

-	예외와 Optional 클래스

	-	자바 API는 어떤 이유에서 값을 제공할 수 없을 때 null을 반환하는 대신 예외를 발생시킬 때도 있다. (ex_ Integer.parseInt(String) - NumberFormatExcetion)

	-	유틸리메서드를 구현하여 Optional을 반환할 수 있도록 감싸자. 매번 try/catch로 감쌀 필요 없다.

		```java
		public static Optional<Integer> stringToInt(String s){
		    try{
		        return Optional.of(Integer.parseInt(s));
		    } catch(NumberFormatException e){
		        return Optional.empty();
		    }
		}
		```

-	기본형 Optional을 사용하지 말아야 하는 이유

	-	스트림처럼 Optional도 기본형으로 특화된 OptionalInt, OptionalLong 등의 클래스를 제공한다.
	-	하지만 Optional의 최대 요소 수는 한 개이므로 기본형 특화 클래스로 성능을 개선할 수 없다.
	-	기본형 특화 Optional은 map, flatMap, filter 등을 지원하지 않는다.
	-	스트림과 마찬가지로 기본형 특화 Optional로 생성한 결과는 다른 일반 Optional과 혼용할 수 없다.

#### Chap 12 : 새로운 날짜와 시간 API

-	대부분의 자바 개발자는 날짜와 시간관련 기능에 만족하지 못했다. 자바8에서는 지금까지의 날짜와 시간 문제를 개선하는 새로운 날짜와 시간 API를 제공한다.

-	자바 1.0의 java.util.Date 클래스

	-	날짜를 의미하는 Date라는 클래스의 이름과 달리 밀리초단위로 포현, 1900년을 기준으로하는 오프셋, 0에서 시간하는 달 인덱스, 가변클래스 등 모호한 설계임.

-	자바 1.1의 java.util.Calendar 클래스

	-	여전의 달의 인덱스는 0으로 시작, Date와 Calendar로 사용자에게 혼란을 줌, DateFormat 동작X , 가변클래스임

-	부실한 날짜와 시간 라이브러리 때문에 많은 개발자는 Joda-Time 같은 서드파티 날짜와 시간 라이브러리를 사용했음.

	-	자바8에서는 Joda-Time의 많은 기능을 java.time 패키지로 추가함.

##### 12.1 LocalDate, LocalTime, Duration, Period 클래스

-	java.time 패키지는 LocalDate, LocalTime, LocalDateTime, Instant, Duration, Period 등 새로운 클래스를 제공한다.

-	LocalDate와 LocalTime 사용

	-	LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다. (시간대 정보를 포함하지 않는다.)

	```java
	LocalDate date = LocalDate.of(2017, 9, 21);
	int year = date.getYear();
	Month month = date.getMonth();
	...
	```

	-	get 메서드에 TemporalField를 전달해서 정보를 얻을 수 도 있음. (TemporalField는 시간 관련 객체에서 어떤 필드의 값에 접근할지 정의하는 인터페이스이다.)

	```java
	// 열거자 ChronField 는 TemporalField 인터페이스를 정의한다.
	int year = date.get(ChronField.YEAR);
	int month = date.get(ChronField.MONTH_OF_YEAR);
	int day = date.get(ChronFiled.DAY_OF_MONTH);
	```

	-	시간은 LocalTime 클래스로 표현할 수 있다.

	```java
	LocalTime time = LocaTime.of(13, 45, 20); //13:45:20
	int hour = time.getHour();
	int minute = time.getMinute();
	int second = time.getSecond();
	```

	-	문자열로 LocalDate와 LocalTime 인터턴스를 만들 수 있음 (parse 메서드에 DateTimeFormatter 를 전달할 수 도 있다.)

	```java
	LocalDate date = LocalDate.parse("2020-09-08");
	LocalTime time = LocalTime.parse("13:20:11")
	```

-	날짜와 시간 조합

	-	LocalDateTime은 LocalDate와 LocalTime 을 쌍으로 갖는 복합 클래스이다.

	```java
	LocalDateTime dt1 = LocalDateTime.of(2020, Month.SEPTEMBER, 21, 13, 40, 20);
	LocalDateTime dt2 = date.atTime(13, 40, 20);


	// LocalDate 추출
	LocalDate date1 = dt1.toLocalDate();
	```

-	Instant 클래스 : 기계의 날짜와 시간

	-	새로운 java.time.Instant 클래스에서는 기계적인 관점(연속된 시간에서 특정 지점을 큰 수로 표현)에서 시간을 표현한다.

	-	Instant 클래스는 유닉스 에포크 시간(1970년 1월 1일 0시 0분 0초 UTC)을 기준으로 특정 지점까지의 시간을 초로 표현한다.

	```java
	// 결과 : 1970-01-01T00:00:03Z
	System.out.println(Instant.ofEpochSecond(3));
	```

-	Duration과 Period 정의

	-	지금까지 살펴본 모든 클래스는 Temporal 인터페이스를 구현한다. Temporal 인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.

	-	Duration클래스는 정적팩토리 메서드 between으로 두 시간 객체 사이의 지속시간을 만들 수 있다.

	```java
	Duration d1 = Duration.between(time1, time2);
	Duration d2 = Duration.between(dateTime1, dateTime2);
	Duration d3 = Duration.between(instant1, intsant2);
	```

	-	Duration 클래스는 초와 나노초로 시간단위를 표현한다. 년,월,일로 시간을 표현할 때는 Period 클래스를 사용한다.

	```java
	Period tenDays = Period.between(LocalDate.of(2020, 1, 1), LocalDate.of(2020, 9, 8));
	```

	-	Duration과 Period 클래스는 자신의 인스턴스를 만들 수 있도록 다양한 팩토리 메서드를 제공한다.

	```java
	Duration threeMinutes = Duration.ofMinutes(3);      
	Duration threeMinutes = Duration.of(3, ChronUnit.MINUTES);


	Period tenDays = Period.ofDays(10);
	Period threesWeeks = Period.ofWeeks(3);
	Period twoYearsSixMonthsOneDay = Period.of(2,6,1);
	```

-	지금까지 본 모든 클래스는 불변이다.

	-	불변 클래스는 함수프로그래밍 그리고 스레드 안전성과 도메인 모델의 일관성을 유지하는 데 좋은 특징이다.
	-	하지만, 새로운 날짜와 시간 API에서는 변경된 객체 버전을 만들 수 있는 메서드를 제공한다.

##### 12.2 날짜 조정, 파싱, 포메팅

-	withAttribute 메서드로 기존의 LocalDate를 바꾼 버전을 직접 간단하게 만들 수 있다.

```java
// 절대적인 방식으로 속성 변경
LocalDate date1 = LocalDate.of(2017, 9, 21);
LocalDate date2 = LocalDate.withYear(2011);
LocalDate date3 = LocalDate.with(ChronField.MONTH_OF_YEAR, 2);

// 상대적인 방식으로 속성 변경
LocalDate date4 = LocalDate.of(2017, 9, 21);
LocalDate date5 = date4.plusWeeks(1);
LocalDate date6 = date5.minusYears(6);
```

-	TemporalAdjusters 사용하기

	-	with 메서드에 좀 더 다양한 동작을 수행할 수 있도록 하는 기능을 제공하는 TemporalAdjuster 를 전달할 수 있다.

		```java
		import static java.time.temporal.TemporalAdjusters.*;


		LocalDate date1 = LocalDate.of(2020, 9, 10);
		LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY)); // 2020-09-13
		LocalDate date3 = date2.with(lastDayOfMonth()); // 2020-09-30
		```

	-	또한 TemporalAdjuster 를 커스텀 구현하여 사용할 수 있다. TemporalAdjuster 인터페이스는 하나의 메서드만 정의한다. (하나의 메서드만 정의하므로 함수형 인터페이스이다.)

		```java
		@FunctionalInterface
		public interface TemporalAdjuster{
		    Temporal adjustInto(Temporal temporal);
		}
		```

	-	날짜와 시간 객체 출력과 파싱

		-	날짜와 시간 관련작업에서 포매팅과 파싱은 서로 떨어질 수 없는 관계이다.

		-	DateTimeFormatter 클래스를 통해 정적 팩토리 메서드와 상수를 이용해서 손쉽게 포매터를 만들 수 있다.

			```java
			LocalDate date = LocalDate.of(2014, 3, 18);
			String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); // 20140318
			String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); // 2014-03-19


			// 반대 파싱
			LocalDate date1 = LocalDate.parse("20140318", DateTimeFormatter.BASIC_ISO_DATE);
			```

		-	기존의 java.util.DateFormat 클래스와 달리 모든 DateTimeFormatter는 스레드에서 안전하게 사용할 수 있는 클래스다.

		-	특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드를 제공한다.

			```java
			DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyy");


			LocalDate date1 = LocalDate.of(2014, 3, 18);
			String formatterDate = date1.format(formatter);
			LocalDate date2 = LocalDate.parse(formatterDate, formatter);
			```

		-	DateTimeFormatterBuilder 클래스로 복잡한 포매터를 정의해서 좀 더 세부적으로 포매터를 제어할 수 있다.

			-	대문자를 구분하는파싱, 관대한 규칙을 적용하는 파싱, 패딩, 포매터의 선택사항 등 활용 가능

				```java
				DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
				    .apppendText(ChronField.DAY_OF_MONTH)
				    .appendLiteral(". ")
				    .appendText(ChronField.MONTH_OF_YEAR)
				    .appendLiteral(" ")
				    .appendText(ChronField.YEAR)
				    .parseCaseInsenitive()
				    .toFormatter(Local.ITALIAN);
				```

##### 12.3 다양한 시간대와 캘린더 활용 방법

-	새로운 날짜와 시간 API에서는 시간대를 간단하게 처리할 수 있다. (java.time.ZoneId 클래스)

-	시간대 사용하기

	-	ZoneId의 getRules() 를 이용해서 해당 시간대의 규정을 획득할 수 있다.

		-	ZoneId romeZone = ZoneId.of("Europe/Rome"); // {지역}/{도시} 형식

	-	ZoneId의 새로운 메서드인 toZoneId로 기존의 객체를 ZondeId 객체로 변환할 수 있다.

		-	ZoneId zoneId = TimeZone.getDefault().toZone();

	-	ZoneId 객체를 얻은 다음에는 LocalDate, LocalDateTime, Instant 를 이용해서 ZoneDateTime 인스턴스로 변환할 수 있다. (ZonedDateTime은 지정한 시간대에 상대적인 시점을 표현한다.)

		```java
		LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
		ZonedDateTime zdt1 = date.atStartOfDay(romeZone);
		LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
		ZonedDateTime ztd2 = date.atZone(romeZone);
		Instant instant = Instant.now();
		ZonedDateTime ztd3 = instant.atZone(romeZone);
		```

	-	ZoneId 를 이용해서 LocalDateTime을 Instant로 바꾸는 방법도 있다. (기존의 Date 클래스를 처리하는 코드를 사용해야 하는 상황이 있을 수 있으므로 Instant로 작업하는 것이 유리하다.)

		```java
		Instant instant = Instant.now();
		LocaDateTime timeFromInstant = LocalDateTime.ofInstant(instant, romeZone);
		```

-	UTC/Greenwich 기준의 고정 오프셋

	-	때로는 UTC(협정 세계시)/GMT(그리니치 표준시)를 기준으로 시간대를 표현하기도 한다.

		```java
		ZoneOffset newYorkOffset = ZoneOffset.of("-5:00");


		LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
		OffsetDateTime dateTimeInNewWork = OffsetDateTime.of(date, newYorkOffset);
		```

-	대안 캘린터 시스템 사용하기

	-	ISO-8601 캘린더 시스템은 전 세계에서 통용된다. 하지만 자바8에서는 추가로 4개의 캘린더 시스템을 제공한다.

	-	ThaiBuddhistDate, MinguoDate, JapaneseDate, HijrahDate

		-	4개의 클래스와 LocaDate 클래스는 ChronoLocalDate 인터페이스를 구현하는데, ChronoLocalDate는 임의의 연대기에서 특정 날짜를 표현할 수 있는 기능을 제공하는 인터페이스다.

		```java
		LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
		JapaneseDate japaneseDate = JapaneseDate.from(date);


		// 특정 Local과 Locale에 대한 날짜 인스턴스로 캘린터 시스템을 만들 수 있다.
		// Chronology는 캘린더 시스템을 의미하며 정적 팩토리 메서드 ofLocal을 이용하여 Chronology의 인스턴스를 획득할 수 있다.
		Chronoy japaneseChronology = Chronology.ofLocale(Locale.JAPANE);
		ChronoLocalDate now = japaneseChronology.dateNow();
		```

	-	날짜와 시간 API 설계자는 ChronoLocalDate 보다는 LocalDate를 사용하라고 권장한다.

		-	프로그램의 입출력을 지역화하는 상황을 제외하고는 모든 데이터 저장, 조작, 비즈니스 규칙 해석 등의 작업에서 LocalDate를 사용해야 한다.
