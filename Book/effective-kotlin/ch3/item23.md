# Item23. 타입 파라미터의 섀도잉을 피하라

## 섀도잉이란
- 프로퍼티와 파라미터가 같은 이름을 가질 수 있는데, 이런 상황에서 지역 파라미터가 외부 스코프에 있는 프로퍼티를 가리키는 경우

``` kotlin
class Forest(val name: String) {

    fun addTree(name: String) {
        //...
    }
}
```

또한 클래스 타입 파라미터와 함수 타입 파라미터 사이에서도 발생할 수 있다.

``` kotlin
interface Tree
interface Birch: Tree
interface Spruce: Tree

class Forest<T: Tree> {

    fun <T: Tree> addTree(tree: T) {
        ///..
    }
}

val forest = Forest<Birch>()
forest.addTree(Birch())
forest.addTree(Spruce())
// 의도하지 않은 상황. 타입 파라미터가 독립적으로 동작한다.
```

따라서 클래스 타입 파라미터 T를 그대로 사용하게 한다.

``` kotlin
class Forest<T: Tree> {

    fun addTree(tree: T) {
        // ...
    }
}

val forest = Forest<Birch>()
forest.addTree(Birch())
forest.addTree(Spruce()) // Error. Type mismatch
```

혹은 독립적인 타입 파라미터를 의도했더라면, 이름을 아예 다르게 해서 구분할 수 있게 하자.

``` kotlin
class Forest<T: Tree> {

    fun <ST: T> addTree(tree: ST) {
        // ...
    }
}
```

## 정리
타입 파라미터 섀도잉을 피하자. 가장 큰 이유는 이해하기 어려울 수 있다.