# Item22. 일반적인 알고리즘을 구현할 때 제네릭을 사용하라

## 제네릭이란

- 타입 아규먼트를 사용하면 함수에 타입을 전달할 수 있다.
- 타입 아규먼트(타입 파라미터)를 사용하는 함수를 제네릭이라고 한다.

``` kotlin
inline fun <T> Iterable<T>.filter(
    predicate: (T) -> Boolean
): List<T> {
    val destination = ArrayList<T>()
    for (element in this) {
        if (predicate(element)){
            destination.add(element)
        }
    }
    return destination
}
```

타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공하여 컴파일러가 타입을 조금이라도 더 정확하게 추측할 수 있게 해준다.
따라서 프로그램이 조금 더 안전해지고, 개발자는 프로그래밍이 편해진다.

> 컴파일 과정에서 최종적으로 이러한 타입 정보는 사라지지만, 개발 중에는 특정 타입을 사용하게 강제할 수 있다.

코틀린과 자바의 제네릭 시스템은 타입 소거(Type Erasure)를 사용하기에, 제네릭 클래스나 함수가 컴파일될 때, 타입 파라미터가 제거되고 해당 타입 파라미터 대신 실제 특정 타입으로 대체되거나 Any(Object)로 대체된다.
따라서 런타임 시 제네릭 타입 정보가 유지되지 않는다.

타입 소거 시
1. 컴파일러는 타입 파라미터를 제거하고
2. 타입 파라미터에 상한이 적용되어 있으면 상한 타입으로 대체한다. 지정되지 않은 경우 코틀린의 Any (자바의 Object)로 대체된다.


타입 소거의 영향으로
1. 런타임 타임에 정보 손실이 되어 제네릭 타입 정보는 런타임 시점에 사용할 수 없게 된다. 이 의미는 런타임 시점에 제네릭 타입을 검사하거나 반영할 수 없는 것을 의미한다.
2. 타입 변환이 필요하여 런타임 시 제네릭 타입 사용하기 위해 명시적인 타입 캐스팅이 필요할 수 있다.

런타임 시점에 타입 소거를 유지하고 싶다면 `refied` 키워드를 사용하면 된다.

``` kotlin
inline fun <reified T> isInstance(value: Any): Boolean {
    return value is T
}

fun main() {
    println(isInstance<String>("Hello")) // true
    println(isInstance<Int>("Hello"))    // false
}

```

Q. 왜 타입 소거를 하는 걸까?

- 이전 버전의 자바와의 호환성을 유지하기 위함이라고 한다.
- 제네릭은 자바 5부터 등장했는데, 1.4 이하의 컴파일된 소스 코드와 충돌을 일으키지 않기 위해 타입 소거를 적용했다고 한다. (하위 호환성?)

## 제네릭 제한
타입 파리미터의 중요한 기능 중 하나는 *구체적인 타입의 서브 타입*만 사용하게 타입을 제한하는 것이다.

``` kotlin
fun <T : Comparable<T>> Iterable<T>.sorted() : List<T> {

}
```

## 정리
코틀린에서 타입 파라미터는 굉장히 중요한 부분이다. 이를 이용하여 type-safe 제네릭 알고리즘과 제네릭 객체를 구현한다. 또한, 구체 자료형의 서브 타입을 제한할 수 있는데 이를 통해 특정 자료형이 제공하는 메서드를 안전하게 사용할 수 있다.