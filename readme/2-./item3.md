# Item3: 생성자나 열거 타입으로 싱글턴임을 보증하라

Singleton이란 인스턴스를 오직 하나만 생성할 수 있는 클래스이다. 하지만 <mark style="background-color:orange;">**클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려워질 수 있다**</mark>. 왜냐하면 싱글턴 인스턴스를 mocking 할 수 없기 때문이다.

## 싱글톤을 만드는 방식

싱글톤을 만드는 방식은 보통 2가지가 존재한다.

### 1. public static final 필드



```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    
    public void someMethod() {}
}
```

private 기본 생성자 Elvis()는 public static final 필드(Elvis.INSTANCE)를 초기화할 때 딱 한번만 호출된다. 이로써 Elvis 클래스가 전체 시스템에서 오직 하나뿐임을 보장할 수 있다.

다만 reflection을 활용하여 외부에서 클래스의 private 메소드를 호출하는 방법(공격)이 존재한다. (`AccessibleObject.setAccessible`) 이 공격을 사전에 방지하려면 private 생성자가 두 번째 호출 되는 시점에 예외를 던지게 해야 한다.

> 장점

해당 클래스가 싱클턴이라는 것이 API에 명백히 드러나며 간결하다.

### 2. 정적 팩토리

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    public static Elvis getInstance() { return INSTANCE; }
    
    public void someMethod() {}
}
```

Elvis.getInstance 메소드는 항상 동일한 객체 INSTANCE를 반환한다.

> 장점

API를 변경하지 않고도 해당 클래스가 싱글턴이 아니게 변경할 수 있다.

* 유일한 인스턴스를 반환하지 않고 thread 별로 다른 인스턴스를 반환하도록 수정할 수 있음

또한 정적 팩토리 메소드를 generic singleton factory 메소드로 만들 수 있다. 이러한 장점을 활용하지 않는다면, 싱글톤 객체를 만들기 위해  정적 팩토리 방식보다 public static final 필드를 활용한 방식이 좋다.



#### 싱글톤 객체의 직렬화

싱글톤으로 만들 클래스를 직렬화하기 위해 단순히 Serializable 인터페이스를 구현한다고 선언하는 것만으로는 부족하다. <mark style="background-color:orange;">**모든 인스턴스 필드를 transient로 선언하고 readResolve 메소드를 통해 싱글턴임을 보장해야 한다**</mark>.



### 3. 열거 타입

```java
public enum Elvis {
    INSTANCE;
    
    public void someMethod() {}
}
```

열거 타입을 통해 싱글턴을 구현하면 직렬화를 위한 추가적인 작업이 필요없다. 또한 reflection을 통한 제 2의 인스턴스를 생성하는 것을 막을 수 있다. <mark style="background-color:orange;">**대부분의 상황에서 원소가 하나뿐인 열거 타입으로 싱글턴을 만드는게 가장 바람직하다**</mark>.

* 단, 만드려는 싱글톤 클래스가 열거 타입 이외의 클래스를 상속한다면 이 방식으로 싱글톤 클래스를 만들 수 없다.
