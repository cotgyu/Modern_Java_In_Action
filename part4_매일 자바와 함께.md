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

#### Chap13 : 디폴트 메서드

-	전통적인 자바에서 인터페이스와 관련 메서드는 한 몸처럼 구성된다.

	-	인터페이스를 구현하는 클래스는 인터페이스에서 정의하는 모든 메서드 구현을 제공하거나 슈퍼클래스의 구현을 상속받아야 한다.

	-	평소에는 문제 없지만, 인터페이스에 새로운 메서드를 추가하는 등 인터페이스를 바꾸고 싶을 때는 문제가 발생한다. (인터페이스를 바꾸면 이전의 인터페이스를 구현한 모든 클래스의 구현도 바꿔야함.)

	-	하지만 자바8에서는 이 문제를 해결하는 새로운 기능을 제공한다.

		-	인터페이스 내부에 정적 메서드를 사용
		-	인터페이스의 기본 구현을 제공할 수 있도록 디폴트 메서드 기능을 사용

			```java
			// 자바8에서 List인터페이스의 새로 추가된 메서드
			default void sort(Comparator<? super E> c){
			    Collections.sort(this, c);
			}
			```

	-	디폴트 메서드를 이용하면 인터페이스의 기본 구현을 그대로 상속하므로 인터페이스에 자유롭게 새로운 메서드를 추가할 수 있게 된다.

##### 13.1 변화하는 API

-	공개 API를 고치면 기존 버전과의 호환성 문재가 발생한다.

	-	물론 예전버전과 새로운 버전을 관리하는 방법도 있지만 불편하다.
	-	**디폴트 메서드를 이용해서 API를 바꾸면 새롭게 바뀐 인터페이스에서 자동으로 기본 구현을 제공하므로 기존 코드를 고치지 않아도 된다.**

-	자바 프로그램을 바꾸는 것과 관련된 호환성 문제는 크게 세 가지로 분류할 수 있다.

	-	바이너리 호환성 : 변경 후에도 에러 없이 기존 바이너리가 실행될 수 있는 상황 (ex_ 인터페이스에 메서드를 추가했을 때 추가된 메서드를 호출하지 않는 한 문제가 일는 상황)

	-	소스 호환성 : 코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있음 (ex_ 인터페이스 메서드를 추가하면 소스 호환성이 아니다. 추가한 메서드를 구현하도록 클래스를 고쳐야하기 떄문)

	-	동작 호환성 : 코드를 바꾼 다음에도 같은 입력 값이 주어지면 프로그램이 같은 동작을 실행함 (ex_ 인터페이스에 메서드를 추가하더라도 프로그램에서 추가된 메서드를 호출할 일은 없으므로 동작 호환성은 유지된다.)

##### 13.2 디폴트 메서드란 무엇인가?

-	자바8에서는 호환성을 유지하면서 API를 바꿀 수 있도록 새로운 기능인 디폴트 메서드를 제공한다.

	-	이제 인터페이스는 자신을 구현하는 클래스에서 메서드를 구현하지 않을 수 있는 새로운 메서드 시그니처를 제공한다.

	-	(디폴트메서드)인터페이스를 구현하는 클래스에서 구현하지 않은 메서드는 인터페이스 자체에서 기본을 제공한다.

-	디폴트메서드는 default 라는 키워드로 시작하며, 다른 클래스에서 선언된 메서드처럼 메서드 바디를 포함한다.

-	추상 클래스와 자바 8의 인터페이스

	-	추상클래스와 인터페이스는 뭐가 다를까? 둘다 추상 메서드와 바디를 포함하는 메서드를 정의할 수 있다.

		-	클래스는 하나의 추상클래스만 상속받을 수 있지만 인터페이스를 여러 개 구현할 수 있다.
		-	추상 클래스는 인스턴스 변수(필드)로 공통 상태를 가질 수 있다. 인터페이스는 인스턴스 변수를 가질 수 없다.

##### 13.3 디폴트 메서드 활용 패턴

-	디폴트 메서드를 이용하는 두 가지 방식에 대해 알아본다. (선택형 메서드, 동작 다중 상속)

-	선택형 메서드

	-	인터페이스를 구현하는 클래스에서 메서드의 내용이 비어있는 상황을 많이 볼 수 있다. (ex_ 잘사용하지 않는 Iterator - remove 메서드)

	-	결과적으로 Iterator를 구현하는 많은 클래스에서는 remove에 빈 구현을 제공했다.

	-	디폴트메서드를 이용하면 remove 같은 메서드에 기본 구현을 제공할 수 있다. (이것으로 클래스에서 빈 구현을 제공할 필요가 없다.)

	-	Iterator 인터페이스를 구현하는 클래스는 빈 remove 메서드를 구현할 필요가 없어졌고, 불필요한 코드를 줄일 수 있다.

-	동작 다중 상속

	-	디폴트 메서드를 이용하면 기존에는 불가능했던 동작 다중 상속 기능도 구현할 수 있다.

	-	자바 8에서는 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작(구현 코드)을 상속받을 수 있다.

		```java
		public class ArrayList<E> extends AbstractList<E> implement List<E>, RandomAccess, Cloneable, Serializable { }
		```

-	옳지 못한 상속

	-	상속으로 코드 재사용 문제를 모두 해결할 수 있는 것은 아니다.

	-	예를 들어 한 개의 메서드를 재사용하려고 100개의 메서드와 필드가 정의되어 클래스를 상속받는 것은 좋은 생각이 아니다. 이럴 때는 델리게이션(delegation), 즉 멤버 변수를 이용해서 클래스에서 필요한 메서드를 직접 호출하는 메서드를 작성하는 것이 좋다.

	-	종종 final로 선언된 클래스를 볼 수 있는데 다른 클래스가 이 클래스를 상속받지 못하게 함으로써 원래 동작이 바뀌지 않길 원하기 때문

	-	필요한 기능만 포함하도록 인터페이스를 최소환으로 유지한다면 필요한 기능만 선택할 수 있으므로 쉽게 기능을 조립할 수 있다.

##### 13.4 해석 규칙

-	같은 시그니처를 갖는 디폴트 메서드를 상속받는 상황이 생길 수 있다. 이런 상황에서는 어떤 디폴트 메서드를 사용할까?

-	알아야 할 세 가지 해결 규칙

	-	1) 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.

	-	2) 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다. 상속 관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브 인터페이스가 이긴다. 즉 B가 A를 상속받는다면 B가 A를 이긴다.

	-	3) 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.

	```java
	public interface A{
	    default void hello(){
	        System.out.println("Hello from A");
	    }
	}


	public interface B extends A{
	    default void hello(){
	        System.out.println("Hello from B")
	    }
	}


	// 2번 규칙에 의해 Hello from B 출력!
	public class C implements B, A {
	    public static void main(String... args){
	        new C().hello();
	    }
	}


	public class D implements A{
	}


	/**
	D 인터페이스는 A의 디폴트 메서드 구현을 상속 받는다.
	2번 규칙에서는 클래스나 슈퍼 클래스에 메서드 정의가 없을 때는 디폴트 메서드를 정의하는 서브 인터페이스가 선택된다.
	따라서 컴파일러는 인터페이스 A의 hello나 인터페이스 B의 hello 둘 중 하나를 선택해야 한다.
	여기서 B가 A를 상속받는 관계이므로 또 Hello from B 가 출력된다.
	*/
	public class C extends D implements B, A {
	    public static void main(String... args){
	        new C().hello();
	    }
	}
	```

-	충돌 그리고 명시적인 문제 해결

	-	B가 A를 상속받지 않는 상황이라면???

	-	인터페이스 간 상속관계가 없으므로 2번 규칙을 적용할 수 없다. 자바 컴파일러는 어떤 메서드를 호출해야 할지 알 수 없으므로 "Error: class C inherits unrelated defaults for Hello() from types B and A" 의 에러를 표출한다.

		```java
		public interface A {
		    default void hello(){
		        System.out.println("Hello from A");
		    }
		}


		public interface B {
		    default void hello(){
		        System.out.println("Hello from B");
		    }
		}


		public class C implements B, A {
		}
		```

	-	충돌 해결

		-	클래스와 메서드 관계로 디폴트 메서드를 선택할 수 없는 상황에서는 선택할 수 있는 방법이 없다. 개발자가 직접 클래스 C에 사용하려는 메서드를 명시적으로 선택해야 한다. (C에서 메서드를 오버라이드한 다음에 호출하려는 메서드를 명시적으로 선택)

		-	자바8 에서는 X.super.m(...) 형태의 새로운 문법을 제공한다. (X는 호출하려는 메서드 m의 슈퍼인터페이스)

			```java
			public class C implements B, A{
			    void hello(){
			        B.super.hello();
			    }
			}
			```

-	다이아몬드 문제

	-	다이어그램의 모양이 다이아몬드를 닮았으므로 다이아몬드 문제라고 부른다. D는 B와 C 중 누구의 디폴트 메서드 정의를 상속 받을까?

		```java
		public interface A {
		    default void hello(){
		        System.out.println("Hello from A");
		    }
		}
		public interface B extends A {}
		public interface C extends A {}


		/** 실제로 선택할 수 있는 메서드 선언은 하나뿐이다. A만 디폴트 메서드를 정의하고 있으므로 Hello from A 가 출력된다.
		(B에도 hello가 있다면 2번 규칙으로 B가 선택됨)
		(C에 추상메서드를 추가한다면 C는 A를 상속받으므로 C의 추상 메서드 hello가 A의 디폴트 메서드 hello 보다 우선권을 갖는다. 따라서 컴파일 에러가 발생하며, 클래스 D가 어떤 hello를 사용할지 명시적으로 선택해야 한다.
		*/
		public class D implements B, C{
		    public static void main(String... args){
		        new D().hello();
		    }
		}
		```

#### Chap 14 - 자바 모듈 시스템

-	자바 9에서 가장 많이 거론되는 새로운 기능은 모듈 시스템이다.

-	14장에서는 모듈 시스템이란 무엇이며, 새로운 자바 모듈시스템이 어디에서 사용될 수 있으며, 개발자는 이로부터 어떤 이익을 얻을 수 있는지를 설명함.

-	깊게 보려면 니콜라이 팔로그 - The Java Module System 책을 추천한다고 함.

##### 14.1 압력 : 소프트웨어 유추

-	어떤 동기와 배경으로 자바 언어 설계자들이 목표를 정했는 지 이해해보자.

-	지금까지는 서술하는 듯한 코드 즉 이해하고 유지보수하기 쉬운 코드를 구현하는 데 사용할 수 있는 새로운 언어 기능을 소개했다. 궁극적으로 소프트웨어 아키텍처 즉 고수준에서 기반 코드를 바꿔야할 때 유추하기 쉬우므로 생산성을 높일 수 있는 소프트웨어 프로젝트가 필요하다. (관심사분리와 정보은닉을 살펴본다.)

-	관심사 분리

	-	컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙

	-	어플리케이션을 개발할 떄 SoC를 적용함으로 각 기능별 모듈(서로 겹치지 않는 코드 그룹)로 분리할 수 있다.

		-	SoC 원칙은 모델, 뷰, 컨트롤러 같은 아키텍쳐 관점 그리고 복구 기법을 비즈니스 로직과 분리하는 등의 하위 수준 접근 등의 상황에서 유용하다.

			-	개발 기능을 따로 작업할 수 있으므로 팀이 쉽게 협업할 수 있다.
			-	개별 부분을 재사용하기 쉽다.
			-	전체 시스템을 쉽게 유지보수할 수 있다.

-	정보 은닉

	-	세부 구현을 숨기도록 장려하는 원칙

		-	소프트웨어 개발 시 요구사항은 자주 바뀔 수 있다. 세부 구현을 숨김으로 프로그램의 어떤 부분을 바꿨을 때 다른 부붑까지 영향을 미칠 가능성을 줄여준다.

		-	**코드를 관리하고 보호하는 데 유용한 원칙** / 캡슐화와 관련됨

-	자바에서는 public, protected, private 등의 접근 제한자와 패키지 수준 접근 권한 등을 이용해 메서드, 필드 클래스의 접근을 제어했다. 하지만, 이런 방식으로는 원하는 접근 제한을 달성하기 어려우며 심지어 최종 사용자에게 원하지 않는 메서드도 공개해야하는 상황이 발생했다.

##### 14.2 자바 모듈 시스템을 설계한 이유

-	모듈화의 한계

	-	자바 9 이전까지는 모듈화된 소프트웨어 프로젝트를 만드는 데 한계가 있었다.
	-	자바는 클래스, 패키지, JAR 세 가지 수준의 코드 그룹화를 제공한다.
	-	클래스와 관련해 자바는 접근제한자와 캡슐화를 지원하지만 패키지와 JAR 수준에서는 캡슐화를 거의 지원하지 않는다.

	-	제한된 가시성 제어

		-	많은 애플리케이션은 다양한 클래스 그룹을 정의한 여러 패키지가 있는데 패키지의 가시성 제어 기능은 유명무실할 수준이다.
		-	한 패키지의 클래스와 인터페이스를 다른 패키지로 공개하려면 public으로 이들을 선언해야한다.
		-	결국 이들 클래스와 인터페이스는 모두에게 공개된다.

	-	클래스 경로

		-	자바는 클래스를 모두 컴파일한 다음 보통 한 개의 평범항 JAR파일에 넣고 클래스에 경로에 이 JAR파일을 추가해 사용할 수 있다. 그러면 JVM이 동적으로 클래스 경로에 정의된 클래스를 필요할 때 읽는다.

		-	클래스 경로와 JAR 조합에는 몇 가지 약점이 존재한다.

			-	클래스 경로에는 같은 클래스를 구분하는 버전 개념이 없다. (ex_ 다른버전의 같은라이브러리 존재 시 어떤 일이 발생할 지 예측할 수 없음)
			-	클래스 경로는 명시적인 의존성을 지원하지 않는다. (빠진게 있는지?, 충돌이 있는지? 파악불가)

	-	메이븐이나 그레이들 같은 빌드 도구는 이런 문제를 해결하는 데 도움을 준다. 하지만 자바 9 이전에는 자바, JVM 누구도 명시적인 의존성 정의를 지원하지 않았다.

		-	결국 JVM이 ClassNotFoundException같은 에러를 발생시키지 않고 애플레케이션을 정상적으로 실행할 때 까지 클래스 경로에 클래스 파일을 더하거나 클래스 경로에 클래스를 제거해보는 수 밖에 없다.
		-	자바 9 모듈 시스템을 이용하면 컴파일 타임에 이런 종류의 에러를 모두 검출할 수 있다.

-	거대한 JDK

	-	JDK는 자바 프로그램을 만들고 실행하는 데 도움을 주는 도구의 집합이다.
		-	JDK 라이브러리의 많은 내부 API는 공개되지 않아야 한다. 안타깝게도 자바 언어의 낮은 캡슐화 지원 때문에 내부 API가 외부에 공개되었다.
		-	Spring, Netty, Mockito 등 여러 라이브러리에서 su.misc.Unsafe라는 클래스를 사용했는데, 이 클래스는 JDK 내부에서만 사용하도록 만든 클래스다. 결과적으로 호환성을 깨지않고는 관련 API를 바꾸기가 아주 어려운 상황이 되었다.
		-	이런 문제들 때문에 JDK자체도 모듈화할 수 있는 자바 모듈 시스템 설계의 필요성이 제기되었다,
			-	JDK에서 필요한 부분만 골라 사용하고, 클래스 경로를 쉽게 유추할 수 있으며, 플랫폼을 진화시킬 수 있는 강력한 캡슐화를 제공할 새로운 건축 구조가 필요했다.

-	OSGi와 비교

	-	자바 9에서 직소 프로젝트에 기반한 모듈화 기능이 추가되기 전에는 자바에는 이미 OSGi라는 강력한 모듈 시스템이 존재했다. (공식기능은 아님)
	-	OSGi와 자바 9 모듈 시스템은 상호 배타적인 관계가 아님.

		-	두 기능은 부분적으로만 중복됨.
		-	OSGi는 훨씬 더 광범위한 영역을 가지며 직소에서 제공하지 않는 여러 기능을 제공한다.
		-	번들이라고 불리는 OSGi 모듈은 특정 OSGi 프레임워크 내에서만 실행된다.
		-	대표적인 OSGi 프레임워크 구현으로는 아파치 펠릭스, 애퀴녹스

	-	시스템을 재시작하지 않고도 애플리케이션의 다른 하위 부분을 핫스왑할 수 있다는 점이 직소와 다른 OSGi만의 강점

	-	번들의 동작에 필요한 외부 패키지가 무엇이며 어떤 내부 패키지가 외부로 노출되어 다른 번들로 제공되는지를 서술하는 텍스트 파일로 각 번들을 정의함

	-	동시에 프레임워크 내에 같은 번들의 다른 버전을 설치할 수 있음

	-	OSGi의 각 번들이 자체적인 클래스 로더를 갖는 반면, 자바 9 모듈 시스템의 직소는 애플리케이션당 한 개의 클래스를 사용하므로 버전 제어를 원하지 않는다.

##### 14.3 자바 모듈 : 큰 그림

-	자바는 모듈이라는 새로운 자바 프로그램 구조 단위를 제공한다.

	-	모듈은 module 이라는 새 키워드에 이름과 바디를 추가해서 정의한다.
	-	모듈 디스크립터는 module-info.java라는 특별한 파일에 저장된다.
	-	모듈 디스크립터는 보통 패키지와 같은 폴더에 위치하면서 한 개 이상의 패키지를 서술하며 캡슐화할 수 있지만 단순한 상황에서는 이들 패키지 중 한 개만 외부로 노출시킨다.

```text
자바 모듈 디스크립터의 핵심 구조 (module-info.java)

module 모듈명
exports 패키지명
requires 모듈명
```

##### 14.4 자바 모듈 시스템으로 애플리케이션 개발하기

-	간단한 모듀화 애플리케이션을 기초부터 만들면서 자바 9 모듈 시스템 전반을 살펴본다.

-	애플리케이션 셋업

	-	비용 관리 문제를 해결해줄 애플리케이션을 구현해보자.

		-	애플리케이션은 다음의 작업을 처리해야 한다.

			-	파일이나 URL에서 비용목록을 읽는다.
			-	비용의 문자열 표현을 파싱한다.
			-	통계를 계산한다.
			-	유용한 요약 정보를 표시한다.
			-	각 태스트의 시작, 마무리 지점을 제공한다.

		-	여러기능(관심사) 분리

			-	다양한 소스에서 데이터를 읽음(Reader, HttpReader, FileReader)
			-	다양한 포맷으로 구성된 데이터를 파싱(Parser, JOSNParser, ExpenseJSON-Parser)
			-	도메인 객체를 구체화(Expense)
			-	통계를 계산하고 반환(SummaryCalculator, SummaryStatistics)
			-	다양한 기능을 분리 조정(ExpensesApplication)

		-	기능 그룹화

			-	expense.readers
			-	expense.readers.http
			-	expense.readers.file
			-	expense.parsers
			-	expense.parsers.json
			-	expense.model
			-	expense.statistics
			-	expense.application

-	세부적인 모듈화와 거친 모듈화

	-	시스템을 모듈화할 때 모듈 크기를 결정해야 한다.

		-	가장 좋은 방법은 시스템을 실용적으로 분해하면서 진화하는 소프트웨어 프로젝트가 이해하기 쉽고 고치기 쉬운 수준으로 적절하게 모듈화되어 있는지 주기적으로 확인하는 프로세스를 갖는다.

	-	자바 모듈 시스템 기초

		-	한 개의 모듈만 갖는 기본적인 모듈화 애플리케이션 구조이다.

			```text
			|-- expenses.application
			    |-- module-info.java
			    |-- com
			        |-- example
			            |-- expenses
			                |-- application
			                    |-- ExpensesApplication.java
			```

		-	module-info.java : 모듈 디스크립터로 모듈의 소스 코드 파일 로트에 위치해야 하며 모듈의 의존성 그리고 어떤 기능을 외부로 노출할지를 정의한다.

		-	모듈화 애플리케이션 실행 방법

			-	java 프로그램으로 자바 .class 파일을 실행할 때 두 가지 옵션이 추가되었다.
				-	--module-path : 어떤 모듈을 로드할 수 있는지 지정한다. 이 옵션은 클래스 파일을 지정하는 --classpath 인수와는 다르다.
				-	--module : 실행할 메인 모듈과 클래스를 지정한다.

			```text
			javac module-info.java com/example/expenses/application/ExpensesApplication.java -d target


			jar cvfe expenses-application.jar com.example.expenses.application.ExpensesApplication -C target


			jar --module-path expenses-application.jar \ --module expenses/com.example.expenses.application.ExpensesApplication
			```

##### 14.5 여러 모듈 활용하기

-	다양한 모둘과 관련된 실용적인 예제를 살펴보자

	-	비용 애플리케이션이 소스에서 비용을 읽을 수 있어야 한다. (이 기능을 캡슐화한 expense.reader 라는 새 모듈을 만들 것이다.)

-	exports 구문

	-	exports는 다른 모듈에서 사용할 수 있도록 특정 패키지를 공개 형식으로 만든다.
	-	기본적으로 모듈 내의 모든 것은 캡슐화된다.
	-	모듈 시스템은 화이트 리스트 기법을 이용해 강력한 캡슐화를 제공하므로 다른 모듈에서 사용할 수 있는 기능이 무엇인지 명시적으로 결정해야 한다.

	```text
	// expenses.readers 모듈의 선언
	module expenses.readers{
	        exports com.example.expenses.readers;
	        exports com.example.expenses.readers.file;
	        exports com.example.expenses.readers.http;
	}


	// 프로젝트의 두 모듈의 디렉토리 구조
	|-- expenses.application
	        |-- module-info.java
	        |-- com
	                |-- example
	                        |-- expenses
	                                |-- application
	                                        |-- ExpensesApplication.java


	|-- expenses.readers
	    |-- module-info.java
	    |-- com
	        |-- example
	            |-- expenses
	                |-- readers
	                    |-- Reader.java
	                |-- file
	                    |-- FileReader.java
	                |-- http
	                    |-- HttpReader.java
	```

-	requires 구분

	-	requires는 의존하고 있는 모듈을 지정한다.
	-	기본적으로 모든 모듈은 java.base 라는 플랫폼 모듈에 의존하는데 이 플랫폼 모듈은 net, io, util 등의 자바 메인 패키지를 포함한다. (명시적으로 정의할 필요는 없음)

		```text
		module expenses.readers{
		    requires java.base;


		    exports com.example.expenses.readers;
		    exports com.example.expenses.readers.file;
		    exports com.example.expenses.readers.http;
		}
		```

	-	자바 9에서는 requires 와 exports 구문을 이용해 좀 더 정교하게 클래스 접근을 제어할 수 있다.

		-	447p 표 14-2 참조

-	이름 정하기

	-	모듈명은 패키지명처럼 인터넷 도메인명을 역순으로 이름을 정하도록 권고한다.

##### 14.6 컴파일과 패키징

-	위에선 프로젝트를 설정하고 모듈을 정의했다. 메이븐 등의 빌드 도구를 이용해 프로젝트를 컴파일할 수 있다.

-	각 모듈에 pom.xml을 추가해야한다. 전체 프로젝트 빌드를 조정할 수 있도록 모든 모듈의 부모 모듈에도 추가해야함!

	```text


	    // 프로젝트의 두 모듈의 디렉토리 구조
	    |-- pom.xml
	    |-- expenses.application
	        |-- pom.xml
	        |-- src
	            |-- main
	                |-- java
	                  |-- module-info.java
	                  |-- com
	                          |-- example
	                                  |-- expenses
	                                          |-- application
	                                                  |-- ExpensesApplication.java


	    |-- expenses.readers
	      |-- pom.xml
	      |-- src
	          |-- main
	            |-- java
	              |-- module-info.java
	              |-- com
	                  |-- example
	                      |-- expenses
	                          |-- readers
	                              |-- Reader.java
	                          |-- file
	                              |-- FileReader.java
	                          |-- http
	                              |-- HttpReader.java
	```

-	expenses.reader 프로젝트의 pom.xml 에는 부모 모듈을 지정해줘야한다.

	```text
	...
	<project ...>
	...
	    <groupId>com.example</groupId>
	    <artifactId>expenses.readers</artifactId>
	    <version>1.0</version>
	    <packaging>jar</packaging>
	    <parent>
	        <groupId>com.example</groupId>
	        <artifactId>expenses</artifactId>
	        <version>1.0</version>
	    </parent>
	</project>
	```

-	expenses.application 의 pom.xml 에는 ExpenseApplication이 필요로 하는 클래스와 인스터페이스가 있으므로 expenses.readers를 의존성으로 추가해야한다.

	```text
	...
	<project ...>
	...
	    <groupId>com.example</groupId>
	    <artifactId>expenses.application</artifactId>
	    <version>1.0</version>
	    <packaging>jar</packaging>
	    <parent>
	        <groupId>com.example</groupId>
	        <artifactId>expenses</artifactId>
	        <version>1.0</version>
	    </parent>


	   <dependencies>
	    <dependency>
	        <groupId>com.example</groupId>
	        <artifactId>expenses.readers</artifactId>
	        <version>1.0</version>
	    </dependency>
	  </dependencies>
	</project>
	```

-	전역(루트프로젝트) pom.xml 설정 (메이븐은 특별한 XML 요소 <module>을 가진 여러 메이븐 모듈을 가진 프로젝트를 지원한다.)

	```text
	...
	<project ...>
	...


	    <groupId>com.example</groupId>
	    <artifactId>expenses</artifactId>
	    <version>1.0</version>
	    <packaging>pom</packaging>


	    <modules>
	        <module>expenses.application</module>
	        <module>expenses.readers</module>
	    <modules>


	    <build>
	        <pluginManagement>
	            <plugins>
	                <plugin>
	                  <groupId>com.apache.maven.plugins</groupId>
	                  <artifactId>maven-compiler-plugin</artifactId>
	                      <version>3.7.0</version>
	                      <configuration>
	                          <source>9</source>
	                          <target>9</target>
	                      </configuration>
	                </plugin>
	            </plugins>
	        </pluginManagement>
	    </build>


	</project>
	```

	-	mvc clean package 명령을 실행하면 프로젝트의 모듈을 JAR로 만들 수 있다.

		```text
		 ./expenses.application/target/expenses.application-1.0.jar
		 ./expenses.readers/target/expenses.reader-1.0.jar


		 // 실행 방법
		 java --module-path \
		 ./expenses.application/target/expenses.application-1.0.jar:\
		 ./expenses.readers/target/expenses.reader-1.0.jar \
		    --module \
		    expenses.application/com.example.expenses.application.ExpensesApplication
		```

##### 14.7 자동모듈

-	HttpReader를 만일 httpclient 라이브러리를 사용해 구현하면 어떻게 추가할까?

	-	expenses.readers 프로젝트의 module-info.java 에 requires 구문을 사용해 추가한다.
	-	pom.xml 에 dependency 에 추가한다.

-	자바는 JAR를 자동 모듈이라는 형태로 적절하게 변환한다.

-	모듈 경로상에 있으나 module-info 파일을 가지지 않는 모든 JAR는 자동 모듈이다.

-	자동 모듈은 암묵적을 자신의 모든 패키지를 노출시킨다.

-	추가 후 실행

	```text
	java --module-path \
	./expenses.application/target/expenses.application-1.0.jar:\
	./expenses.readers/target/expenses.reader-1.0.jar \
	./expenses.readers/target/dependency/httpclient-4.5.3.jar \
	     --module \
	     expenses.application/com.example.expenses.application.ExpensesApplication
	```

##### 14.8 모듈 정의와 구문들

-	모듈 정의 언어에서 사용할 수 있는 몇 가지 키워드를 간단하게 살펴보면서 무엇을 할 수 있는지 보여준다.

-	requires

	-	컴파일 타임과 런타임에 한 모듈이 다른 모듈에 의존함을 정의한다.

		```text
		// com.iteratrlearning.application 모듈은  com.iteratrlearning.ui 에 의존한다.
		//  com.iteratrlearning.ui 에서 외부로 노출한 공개 형식을 com.iteratrlearning.application 에서 사용할 수 있다.
		module com.iteratrlearning.application {
		    requires com.iteratrlearning.ui;
		}
		```

-	exports

	-	지정한 패키지를 다른 모듈에서 이용할 수 있도록 공개 형식으로 만든다.
	-	아무 패키지도 공개하지 않는 것이 기본 설정임.
	-	exports는 패키지명을 인수로 받지만 requires는 모듈명은 인수로 받는다

		```text
		module com.iteratrlearning.application {
		        requires com.iteratrlearning.ui;
		        exports com.iteratrleanring.ui.panels;
		        exports com.iteratrleanring.ui.widgets;
		}
		```

-	requires transitive

	-	전이성 선언
	-	com.iteratrlearning.application 에서 com.iteratrlearning.core를 다시 선언할 필요 없음

		```text
		module com.iteratrlearning.ui {
		    requires transitive com.iteratrlearning.core;


		    export com.iteratrlearning.ui.panel;
		    export com.iteratrlearning.ui.widget;
		}


		// com.iteratrlearning.application 은 com.iteratrlearning.core 에서 노출한 공개 형식에 접근할 수 있다.
		module com.iteratrlearning.application{
		    requires com.iteratrlearning.ui;
		}
		```

-	exports to

	-	사용자에게 공개할 기능을 제한함으로 가시성을 좀 더 정교하게 제어할 수 있다.

		```text
		// com.iteratrlearning.widgets 의 접근권한을 가진 사용자의 권한을 com.iteratrlearning.ui.widgetuser로 제한할 수 있다.
		module com.iteratrlearning.ui {
		    requires com.iteratrlearning.core;


		    exports com.iteratrlearning.ui.panels;
		    exports com.iteratrlearning.ui.widgets to com.iteratrlearning.ui.widgetusers;
		}
		```

-	open 과 opens

	-	모듈 선언에 open 한정자를 이용하면 모든 패키지를 다른 모듈에 반사적으로 접근을 허용할 수 있다.

	-	자바 9 이전에는 리플렉션으로 객체의 비공개 상태를 확인할 수 있었다. (진전한 캡슐화는 존재하지 않았다.)

		-	하이버네이트 같은 객체 관계 매핑 도구에서는 이런 기능을 이용해 상태를 직접 고치곤 한다.

	-	자바 9에서는 기본적으로 리플렉션이 이런 기능을 허용하지 않는다.

		-	그런 기능이 필요하면 open 구문을 명시적으로 사용해야 한다.

	-	리플렉션 때문에 전체 모듈을 개방하지 않고도 opens 구문을 모듈 선언에 이용해 필요한 개별 패키지만 개방할 수 있다.

		-	open에 to를 붙여서 반사적인 접근을 특정 모듈에만 허용할 수 있다.

		```text
		open module com.iteratrlearning.ui {
		}
		```

-	uses 와 provides

	-	자바 모듈 시스템은 provides 구문으로 서비스 제공자를 uses 구문으로 서비스 소비자를 지정할 수 있는 기능을 제공한다.
	-	심화 내용은 The Java Module System 책 참고할 것

> 자바 EE 갭라자라면 애플리케이션을 자바 9로 이전할 때 EE와 관련한 여러 패키지가 모듈화된 자바9 가상머신에서 기본적으로 로드되지 않는다는 사실을 기억해야 한다.
