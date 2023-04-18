# Item1: 생성자 대신 정적 팩토리 메소드를 고려하라

클래스는 public 생성자와 별도로 static factory method를 통해 자신의 인스턴스를 제공할 수 있다. 지금부터  static factory method가지는 장단점을 알아보자.

## static factory method가 public 생성자보다 좋은 <mark style="color:blue;">장점</mark>

### _**1. 이름을 가질 수 있다**_

이름을 통해 반활될 객체의 특성을 쉽게 묘사할 수 있다.

### _**2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.**_

반복되는 요청에 같은 객체를 반환하여 살아있게 할 인스턴스를 제어하므로써 인스턴스를 통제(instance-controlled)할 수 있다. 인스턴스를 통제하면 `Singleton` 클래스  또는 `noninstantiable(인스턴스화 불가)`로 만들 수 있다. 불변 값 클래스에서 동치인 인스턴스가 오직 한개뿐임을 보장할 수 도 있다.

인스턴스의 통제는 `Flyweight Pattern` 의 근간이 되며, Enum 타입은 인스턴스가 하나만 만들어짐을 보장한다.

### _**3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**_

이는 반환 객체의 클래스를 자유롭게 선택할 수 있는 "유연성"을 제공한다. 결국 구현 클래스를 공개하지 않고 객체를 반환할 수 있게되어 api를 작게 유지할 수 있다. (인터페이스를 정적 팩토리 메소드의 반환 타입으로 사용하는 `인터페이스 기반 프레임워크` 의 핵심 기술)

자바의 Collection 프레임워크는 핵심 인터페이스에 추가 기능(추가불가, 동기화 기능)을 덧붙힌 45개의 구현체를 제공한다. 하지만 45개의 구현 클래스를 공개하지 않으므로써 api를 작게 만들 수 있었다.\
api를 작게 만들면 사용 난이도를 낮춰줄 수 있다.

java8 이전에는 인터페이스에 정적 메소드를 선언할 수 없었다. 그래서 인터페이스를 반환하는 정적 메소드가 필요하다면 `companion class(동반 클래스)` 를 만들어 해당 클래스에 정적 메소드를 정의해야 했다.&#x20;

java8 부터 인터페이스에 정적 메소드를 가질 수 있어서 동반 클래스를 둘 이유가 사라졌다. java9에서는 private 정적 메소드까지 허락하게 되었다. 하지만 정적 필드, 정적 멤버 클래스는 여전히 public이어야 하므로 인터페이스에 정적 메소드를 구현하기 위한 코드들을 package-private 클래스에 두어야할 수 도 있다.

### _**4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다**_

해당 클래스가 반환 타입의 하위 타입이기만 한다면 static factory method의 반환 타입에 제약이 없다.

EnumSet 클래스는 public 생성자가 없다. 오직 static factory method를 통해 원소가 64개 이하인 경우 `RegularEnumSet` , 원소가 65개 이상인 경우 `JumboEnumSet` 라는 인스턴스를 반환한다.

클라이언트는 EnumSet 클래스의 정적 팩토리 메소드가 반환하는 클래스의 타입의 종류를 모른다. JumboEnumSet의 이점이 없어질 경우 이를 삭제해도 아무런 문제가 없다. 성능을 개선한 버전의 새로운 인스턴스를 반환해도 이 또한 아무 문제가 되지 않는다.

### _5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다._

이는 `service provider framework(서비스 제공자 프레임워크)` 의 근간이된다.&#x20;

대표적인 서비스 제공자 프레임워크에는 JDBC(java database connectivity)가 있다. provider는 service의 구현체이며,   provider는 framework를 통해 클라이언트에 제공된다. 다시말해, framework가 클라이언트와 구현체(provider)를 분리해주는 역할을 한다.

서비스 사용자 프레임워크는 `3개의 핵심 컴포넌트` + `1개의 컴포넌트`로 이뤄진다.

1. service interface : 구현체의 동작을 정의 (JDBC에서 Connection)
2. provider registration API : provider가 구현체를 등록할 때 사용하는 `제공자 등록 API` (JDBC에서 DriverManager.registerDriver)
3. service access API : 클라이언트가 service의 인스턴스를 얻을 때 사용하는 `서비스 접근 API` (JDBC에서 DriverManager.getConnection)
4. service provider interface : 서비스 제공자 인터페이스 (JDBC에서 Driver)

클라이언트는 `서비스 접근 API` 를 사용할 때 원하는 구현체의 조건을 명시할 수 있다. 구현체의 조건을 명시하지 않으면 default 구현체를 반환하거나, 지원하는 구현체들을 하나씩 돌아가며 반환한다. 이처럼 `서비스 접근 API` 가 유연함을 가질 수 있는 이유는 static factory method 덕분이다.

종종 `서비스 제공자 인터페이스` 를 통해 서비스 인터페이스의 인스턴스를 생성하는 factory 객체를 설명해준다. 만약 서비스 제공자 인터페이스가 없다면, 각 구현체의 인스턴스를 만들 때 reflection을 사용해야 한다.

서비스 제공자 프레임워크 패턴에는 여러 variation이 존재한다.

* Bridge Pattern
* Dependency Injection Framework

java6 부터는 `ServiceLoader` 라는 범용 서비스 제공자 프레임워크가 제공되어 서비스 제공자 프레임워크를 직접 만들 필요가 없어졌다. (JDBC는 java6 이전에 등장한 개념이라 ServiceLoader가 사용되지 않는다)



## static factory method의 <mark style="color:red;">단점</mark>

### 1. 상속을 위해 public, protected 생성자가 필요하므로 static factory method만 존재해선 안된다.

다르게 해석한다면, Collection 프레임 워크의 utility 구현 클래스들은 상속할 수 없다는 의미가 된다. 이 제약은 `상속` 이 아니라 `Composition` 을 사용하도록 유도하기 때문에 장점으로 받아들일 수 있다.

### 2. 정적 팩토리 메소드는 프로그래머가 찾기 어렵다.

api 설명에 명확히 드러나지 않기 때문에 api 문서화를 잘해두어야 한다.

흔히 사용하는 static factory method의 명명 방식이 존재한다.

* from
* of
* valueOf
* instance/getInstance
* create/newInstance
* getType
* newType
* type
