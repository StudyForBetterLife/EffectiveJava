# Item4: 인스턴스화를 막으려거든 private 생성자를 사용하라

유틸성 클래스처럼 단순히 정적 메소드, 정적 필드만 담은 클래스를 만들 경우가 있다.

* ex) java.lang.Math, java.util.Arrays, java.util.Collections

유틸성 클래스는 인스턴스를 생성하여 설계한게 아니다. 하지만 클래스에 생성자를 명시하지 않으면 compiler가 자동으로 public 기본 생성자를 만들기 때문에 의도치 않게 인스턴스화할 수 있다. 그리고 <mark style="background-color:orange;">**유틸성 클래스를 추상 클래스로 만들어도 인스턴스화를 막지 못한다. 구상 클래스를 통해 인스턴스화가 가능하기 때문이다.**</mark>

### private 생성자를 추가하여 클래스의 인스턴스화를 방지할 수 있다.

```java
public class UtilityClass {
    private UtilityClass() {
        throw new AssertionError(); // 클래스 내부에서도 생성자를 호출하지 않도록 막아둔다.
    }
}
```

이처럼 private으로 생성자의 호출을 막아두면 상속도 방지할 수 있다. 모든 생성자는 명시적.묵시적으로 상위 클래스의 생성자를 호출하는데, 이를 private으로 선언하여 상위 클래스의 생성자에 접근하지 못하게 되어 상속을 방지한다.

### lombok @UtilityClass&#x20;

{% embed url="https://projectlombok.org/features/experimental/UtilityClass" %}

```java
@UtilityClass
public class UtilityClass {
}
```
