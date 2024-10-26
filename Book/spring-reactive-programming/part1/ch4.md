# Chapter 04 리액티브 프로그래밍을 위한 사전 지식
> - 리액티브 프로그래밍을 잘 사용하기 위해서 기본적으로 알아야 되는 최소한의 사전 지식이 있다. 함수형 프로그래밍 기법인데, Java 8에 람다 표현식이 도입됨으로써 java에서도 함수형 프로그래밍 기법을 사용할 수 있게 되었다. 더 정확히 얘기하자면 함수형 인터페이스를 이용해서 함수형 프로그래밍과 같은 효과를 낼 수 있게 되었다.
> - 리액티브 프로그래밍을 학습하는 데 무리가 없을 정도의 수준까지만 함수형 프로그래밍 기법을 살펴보자

## 4.1 함수형 인터페이스 (Functional Interface)
- Java에서 인터페이스는 명세 또는 사양이라고 표현할 수 있다. Java의 인터페이스는 몸체가 없는 추상 메서드로만 이루어져 있다. (8부터 default method는 제외)
- Java 8에서부터 지원하는 함수형 인터페이스란?
  - 역시 인터페이스인데, 기존 인터페이스에 비해 함수형 인터페이스는 단 하나의 추상 메서드만 정의되어 있다.
  - 그렇다면 왜 굳이 함수형 인터페이스라고 부를까?
- 함수형 프로그래밍 세계에서는 함수를 일급 시민으로 취급한다.
  - 일급 시민이란 함수를 값으로 취급한다는 의미를 조금 더 고급지게 표현한 것인데, 함수를 값으로 취급하기 때문에 어떤 함수를 호출할 때 함수 자체를 파라미터로 전달할 수 있다.
  - Java에서도 이렇게 함수를 값으로 취급할 수 있는 기능이 Java 8부터 추가되었는데, 그것이 바로 함수형 인터페이스이다.
  - 물론, Java 8 이전에도 SAM 인터페이스는 존재했기 때문에 함수형 인터페이스를 사용한 것이나 마찬가지이다.

``` java
...
Collections.sort(cryptoCurrencies, new Comparator<CryptoCurrency>() {
    @Override
    public int compare(CryptoCurrency c1, CryptoCurrency c2) {
        return c1.getUnit().names().compareTo(c2.getUnit().name());
    }
}); // Comparator는 compare()라는 단 하나의 추상 메서드를 가진 함수형 인터페이스이다.
...
```
- 익명 구현 객체의 형태로 파라미터로 전달하고 있는데, 사실 객체를 메서드의 파라미터로 전달하는 것은 Java의 방식이라 과연 이게 함수형 프로그래밍이 맞는지 의문이 든다.
- 그리고 인터페이스 자체를 익명 구현 객체로 전달하는 방식은 코드 자체가 길어져 지저분하게 보이기도 한다.
- 그래서 Java 8 버전부터 기존에 이미 사용하던 인터페이스의 익명 구현 객체를 전달하는 방식을 조금 더 함수형 프로그래밍 방식에 맞게 표현할 수 있는 방식을 추가했는데 그것이 바로 **람다 표현식**이다.

## 4.2 람다 표현식 (Lambda Expression)
- Javascript나 Scala 같은 언어는 이미 함수가 값으로 취급되어 어떤 함수 호출 시 파라미터로 전달할 수 있었다.
- 이렇게 함수를 값으로 전달할 수 있는 언어에서는 함수를 조금 더 간결한 표현으로 전달하는 방법이 있다.
    - Javascript에서는 화살표 함수를 통해 함수를 파라미터로 전달한다.
- Java에서도 인터페이스의 익명 구현 클래스를 화살표 함수처럼 간결하게 표현하는 방법을 Java 8에서부터 지원하는데, 그것이 바로 람다 표현식이다.
- 람다 표현식은 화살표 함수와 비슷한 형태로 Java에서 함수를 값으로 취급하기 위해 생겨난 간결한 형태의 표현식이다.

``` java
(String a, String b) -> a.equals(b)
// 람다 파라미터 / 화살표 / 람다 몸체
```

- 람다 표현식은 단 하나의 추상 메서드를 가지는 인터페이스, 즉 함수형 인터페이스를 구현한 클래스의 메서드 구현을 단순화한 표현식이다.

``` java
// 람다 표현식으로 변경

Collections.sort(cryptoCurrencies,
     (cc1, cc2) -> cc1.getUnit().name().compareTo(cc2.getUnit().name())
);
// 매우 깔끔해졌다.
```
또한 람다 표현식에서는 파라미터 타입이 생략되는데, 이처럼 메서드 파라미터의 타입이 내부적으로 추론되어 생략이 가능해진다.
- 헷갈리지 않아야 할 부분은, 함수형 인터페이스의 추상 메서드를 람다 포현식으로 작성해서 메서드의 파라미터로 전달한다는 의미는 (메서드 자체를 파라미터)로 전달하는 것이 아니라 **함수형 인터페이스를 구현한 클래스의 인스턴스를 람다 표현식으로 작성해서 전달한다는 것이다.**
- 람다로 변경하기 전, 인터페이스의 익명 구현 객체를 전달했으므로 람다 표현식 역시 익명 구현 객체와 마찬가지로 **함수형 인터페이스를 구현한 클래스의 객체라는 사실**을 명심해야 한다.

람다 캡쳐링
- 람다 표현식 외부에서 정의된 변수 역시 사용이 가능하다.

``` java
String korBTC = "비트코인";
// korBTC = "빗코인";
cryptoCurrencies.stream()
    .filter(cc -> cc.getUnit() == CurrencyUnit.BTC)
    .map(cc -> cc.getName() + "(" + korBTC + ")")
    .forEach(cc -> System.out.println(cc));

// 람다 외부에서 정의된 korBtc를 가져다 사용하고 있다.
// 6번 라인 주석을 해제하면 오류가 발생한다. 람다 표현식에서 사용되는 캡처링 변수는 final과 같은 효력을 지녀야하는데, 변경이 되므로 final의 호력을 잃기에 오류가 발생한다.
```

## 4.3 메서드 레퍼런스 (Method Reference)
- 위에서 우리는 람다 표현식을 사용해서 간결하게 작성할 수 있었다.
- 조금 더 간결하게 작성할 수 있는 방법이 있는데, 그것이 바로 메서드 레퍼런스이다. = 메서드 참조

```
(Car car) -> car.getCarName() ==> Car::getCarName
```

- :: 구분자를 사용한다.

ClassName :: static method 유형
- 메서드 레퍼런스로 표현할 수 있는 가장 흔한 유형은 클래스의 static 메서드이다.

``` java
// 동일한 코드
.map(name -> StringUtils.upperCase(name))
.map(StringUtils::upperCase)
```

- StringUtils 클래스의 upperCase 메서드 파라미터는 람다 파라미터로 전달되는 값(name -> )을 사용하는데, 컴파일러가 내부적으로 람다 표현식의 파라미터 형식을 추론할 수 있기 때문에 람다 파라미터와 upperCase 메서드의 파라미터 부분을 생략할 수 있게 된다.

ClassName :: instance method 유형
- Instance method == 일반 메서드

``` java
// 동일한 코드
.map(name -> name.toUpperCase())
.map(String::toUpperCase)
```

object :: instance method 유형
- ClassName::instanceMethod 형태가 아니라 클래스의 객체인 object::instance method 형태로 축약

``` java
// 동일한 코드
.map (pair -> calculator.getTotalPayment(pair))
.map (calculator::getTotalPayment)
```
- 람다 표현식 외부에 있는 PaymentCalculator 클래스의 객체는 람다 표현식 내부에서 사용하는 것이므로 형식 추론을 통해 하위 라인과 같이 축약 가능

ClassName :: new 유형
- 생성자. 람다 표현식 내부에서 어떤 클래스의 생성자를 사용해야 될 경우, 람다 표현식에서 형식 추론이 가능해지면 ClassName::new와 같이 축약 가능

``` java
// 동일한 코드
.map(pair -> new PaymentCalculator(pair))
.map(PaymentCalculator::new)
```
- 람다 표현식 내부에서 생성자가 사용될 경우 람다 표현식의 형식 추론이 가능해지면 축약 가능

## 4.4 함수 디스크립터 (Function Descriptor)
- 람다 표현식의 파라미터 개수나 파라미터 타입, 리턴 타입 등을 보면서 이 람다 표현식이 Java에서 지원하는 어떤 함수형 인터페이스에 해당되는지 알아야 할 때가 있다.
- 함수 디스크립터는 함수 서술자, 함수 설명자 정도로 이해할 수 있는데, 실제로는 일반화된 람다 표현식을 통해서 이 함수형 인터페이스가 어떤 파라미터를 가지고, 어떤 값을 리턴하는지 설명해 주는 역할을 한다.

> 함수형 인터페이스는 하나의 추상 메서드만 가지는 인터페이스로, @FunctionalInterface 어노테이션을 붙여서 정의한다. 이 인터페이스는 단일 메서드를 통해 특정 기능을 수행하는 람다 표현식이나 메서드 참조와 함께 사용된다.
> - 단 하나의 추상 메서드만을 포함하기에, 람다 표현식을 통해 해당 인터페이스의 구현체를 간결하게 생성할 수 있다.
> 
> 함수 디스크립터는 함수형 인터페이스의 추상 메서드 시그니처를 정의한 것으로, 함수형 인터페이스의 매개변수와 반환 타입을 묘사한다.
> - 람다 표현식과 메서드 참조를 사용할 때, Java 컴파일러는 함수 디스크립터를 통해 람다 표현식을 해당 함수형 인터페이스로 타입 추론한다.
> - 즉, 함수 디스크립터에 맞는 시그니처가 일치하면 람다 표현식 ( () -> T 등의)이 해당 함수형 인터페이스의 인스턴스로 취급될 수 있다.
>
> 또한, 코틀린에서는 Single Abstract Method(SAM) => 자바의 함수형 인터페이스를 Kotlin 람다로 변환할 수 있다. 코틀린은 자체적인 함수형 인터페이스를 정의하지 않는데, 함수 타입 자체가 일급 객체로 람다 표현식을 직접 사용할 수 있기 떄문이다.


|함수형 인터페이스|함수 디스크립터|
|--|--|
|Predicate<T> | T -> Boolean|
|Consumer<T> | T -> void | 
|Function<T, R> | T -> R |
|Supplier<T> | () -> T |
|BiPredicate<L, R> | (L, R) -> Boolean|
|BiConsumer<T, U> | (T, U) -> void|
|BiFunction<T, U, R> | (T, U) -> R|

### Predicate
- Java에서는 함수형 인터페이스 중에서 구현해야 되는 추상 메서드가 하나의 파라미터를 가지고, 리턴 값으로 Boolean 타입의 값을 반환하는 함수형 인터페이스를 Predicate라고 정의해 두었다.
- 결과적으로 T->boolean은 T 타입의 람다 파라미터와 boolean 타입의 값을 리턴하는 Predicate의 함수 디스크립터가 되는 것이다.

``` java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

// 스트림의 Filter 메서드에서 볼 수 있다. T 타입을 받아 boolean을 리턴한다.
```

### Consumer
- 데이터를 소비하는 역할 -> 리턴 값이 없다는 의미와 같다.
- Consumer에 정의된 추상 메서드의 파라미터가 T 타입이고 리턴 값이 void라는 의미.

``` java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

- Consumer를 사용하기 좋은 예로, 일정 주기별로 특정 작업을 수행한 후, 결과 값을 리턴할 필요가 없는 경우가 대부분인 배치 처리를 들 수 있다.

### Function
- 하나의 파라미터 T를 받아서 리턴 값으로 R 타입의 값을 반환한다.

``` java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

// Java 스트림의 map 메서드에서 볼 수 있다. T 타입을 받아 R 타입을 리턴한다.
```

### Supplier
- 추상 메서드가 파라미터를 갖지 않으며 리턴 값으로 T 타입의 값만 반환한다.

``` java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

### Bixxxx
- Bi로 시작하는 함수형 인터페이스는 함수형 인터페이스에서 구현해야 하는 추상 메서드에 전달하는 파라미터가 하나 더 추가되어 두 개의 파라미터를 가지는 함수형 인터페이스이다. Java에서 지원하는 기본 함수형 인터페이스의 확장형이라고 보면 된다.

## 마무리
- 함수형 인터페이스는 Java에서 함수형 프로그래밍 방식을 지원하기 위해 Java 8부터 지원하는 인터페이스이다.
- 함수형 인터페이스는 단 하나의 추상 메서드를 가진다.
- 람다 표현식은 함수형 인터페이스에 정의된 추상 메서드를 표현식으로 구현한 것이다.
- 메서드 레퍼런스를 통해 람다 표현식을 조금 더 간결하게 표현할 수 있다.
- 함수 디스크립터를 통해 함수형 인터페이스의 파라미터 형식과 리턴 값의 형태를 알 수 있다.