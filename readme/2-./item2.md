# Item2: 생성자에 매개변수가 많다면 빌더를 고려하라

## 점층적 생성자 패턴

정적 팩토리 메소드와 생성자는 매개변수가 많을 때 대응하기 어렵다. 굳이 대응하기 위해서 `점층적 생성자 패턴(telescoping constructor pattern)`을 사용할 수 있지만 매개변수가 많아질수록 클라이언트 코드를 작성하거나 읽기 어려운 단점이 있다.

* 매개변수 가운데 같은 타입이 있을 경우 순서가 바뀌어도 컴파일러가 이를 알아채지 못하여 런타임시에 엉뚱한 동작을 하게 된다.

## 자바 빈즈 패턴

JavaBeans Pattern은 매개변수가 없는 기본 생성자로 인스턴스를 생성한 뒤, setter를 통해 매개변수 값을 설정하는 방식이다. 이는 객체의 일관성이 깨지고 불변으로 만들 수 없다는 큰 단점이 존재한다.

* 인스턴스 하나를 완전하게 만들기 위해 여러 setter를 호출해야하고, 그 전까지 객체는 일관성이 무너진 상태에 놓인다.
* 일관성이 무너졌다는 측면에서 클래스를 불변으로 만들 수 없고 thread-safe하지 않다.

## 빌더 패턴 ✔️

<mark style="background-color:green;">**점층적 생성자 패턴의 "안정성"과 자바 빈즈 패턴의 "가독성"을 합친 Builder Pattern**</mark>은 클라이언트가 객체를 직접 만드는 대신, <mark style="background-color:green;">**필수 매개변수를 파라미터로 받 정적 팩토리 메소드를 호출하여 빌더 객체를 얻고 -> setter 메소드를 통해 매개변수를 설정한 뒤 -> build 메소드를 호출하여 불변 객체를 얻는다**</mark>.

{% code lineNumbers="true" %}
```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;

    public static class Builder {
        /* 필수 매개변수 */
        private final int servingSize;
        private final int servings;

        /* 선택 매개변수 */
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;

        /* 필수 매개변수만 파라미터로 받는 정적 팩토리 메소드 Builder  */
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        /* 일종의 setter 메소드. 단 Builder 객체를 반환한다. */
        public Builder calories(int val) {
            this.calories = val;
            return this;
        }

        public Builder fat(int val) {
            this.fat = val;
            return this;
        }

        public Builder sodium(int val) {
            this.sodium = val;
            return this;
        }

        /* 매개변수가 없는 build 메소드. 불변객체를 반환한다. */
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
    }
}

// 클라이언트 코드
final NutritionFacts fact = new NutritionFacts.Builder(240, 8)
                                    .calories(100)
                                    .sodium(35)
                                    .fat(10)
                                    .build(); 
```
{% endcode %}

빌더 패턴은 파이썬과 스칼라의 `명명된 선택적 매개변수(named optional parameters)`를 흉내낸 것이다.

## 계층적으로 설계된 클래스 & 빌더 패턴

{% tabs %}
{% tab title="Pizza" %}
{% code title="Pizza.java" lineNumbers="true" %}
```java
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
    private final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        final EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        public abstract Pizza build();
        
        /*
        하위 클래스에서 self메소드를 오버라이딩하여 하위 클래스의 "this"를 반환하도록 한다.
        */
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = EnumSet.copyOf(builder.toppings);
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="NyPizza" %}
{% code title="NyPizza.java" lineNumbers="true" %}
```java
public class NyPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size;

    public static class NyBuilder extends Pizza.Builder<NyBuilder> {
        private final Size size;

        private NyBuilder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        public static NyBuilder builder(Size size) {
            return new NyBuilder(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected NyBuilder self() {
            return this;
        }
    }

    NyPizza(NyBuilder builder) {
        super(builder);
        size = builder.size;
    }
}
```
{% endcode %}
{% endtab %}

{% tab title="Calzone" %}
{% code title="Calzone.java" overflow="wrap" lineNumbers="true" %}
```java
public class Calzone extends Pizza{
    private final boolean sauceInside;

    public static class CalzoneBuilder extends Pizza.Builder<CalzoneBuilder> {
        private boolean sauceInside = false;

        private CalzoneBuilder() {
            this.sauceInside = true;
        }

        public static CalzoneBuilder builder() {
            return new CalzoneBuilder();
        }

        public CalzoneBuilder sauceInside() {
            this.sauceInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected CalzoneBuilder self() {
            return this;
        }
    }

    Calzone(CalzoneBuilder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```
{% endcode %}
{% endtab %}
{% endtabs %}

```java
public class Main {
    public static void main(String[] args) {
        NyPizza nyPizza = NyPizza.NyBuilder.builder(NyPizza.Size.SMALL)
                .addTopping(Pizza.Topping.SAUSAGE)
                .addTopping(Pizza.Topping.ONION)
                .build();

        Calzone calzone = Calzone.CalzoneBuilder.builder()
                .addTopping(Pizza.Topping.HAM)
                .sauceInside()
                .build();
    }
}
```

객체를 만들기 위해서 점층적 생성자 패턴보다 코드가 장황해지므로 매개변수가 4개 이상이 되어야 그 값어치를 한다. 하지만 시간이 지날수록 매개변수가 많아지는 경향이 있기 때문에 애초에 빌더 패턴으로 시작하는편이 나을 경우가 많다.

## lombok @Builder와 @SuperBuilder &#x20;

{% embed url="https://projectlombok.org/features/experimental/SuperBuilder" %}

{% embed url="https://projectlombok.org/features/Builder" %}

롬복의 SuperBuilder 어노테이션을 활용하여 Pizza 빌더 패턴을 구현해보자. 훨씬 간편하게 빌더 패턴을 활용할 수 있다.

{% tabs %}
{% tab title="Pizza" %}
```java
@SuperBuilder
@NoArgsConstructor
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}
    private Set<Topping> toppings;
}

```
{% endtab %}

{% tab title="NyPizza" %}
```java
@SuperBuilder
@NoArgsConstructor
public class NyPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE}
    private Size size;
}
```
{% endtab %}

{% tab title="Calzone" %}
```java
@SuperBuilder
@NoArgsConstructor
public class Calzone extends Pizza {
    private boolean sauceInside;
}

```
{% endtab %}

{% tab title="Main" %}
```java
public class Main {
    public static void main(String[] args) {
        NyPizza nyPizza = NyPizza.builder()
                .toppings(Set.of(Pizza.Topping.SAUSAGE, Pizza.Topping.ONION))
                .build();

        Calzone calzone = Calzone.builder()
                .toppings(Set.of(Pizza.Topping.HAM))
                .sauceInside(true)
                .build();
    }
}
```
{% endtab %}
{% endtabs %}
