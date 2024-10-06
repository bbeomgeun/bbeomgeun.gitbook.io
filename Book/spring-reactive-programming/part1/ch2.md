# Chapter 02 리액티브 스트림즈
> 리액티브 스트림즈가 무엇인지 살펴보고, 리액티브 스트림즈의 핵심 컴포넌트인 Publisher와 Subscriber의 동작 과정을 자세히 알아보자

## 2.1 리액티브 스트림즈 (Reactive Streams)란?
- 개발자가 리액티브한 코드를 작성하기 위해서는 이러한 코드 구성을 용이하게 해 주는 리액티브 라이브러리가 있어야 한다.
- 이 리액티브 라이브러리를 어떻게 구현할지 정의해 놓은 별도의 표준 사양이 있는데, 이것이 바로 **리액티브 스트림즈**라고 부른다.

> 한마디로 '**데이터 스트림을 Non-Blocking이면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리의 표준 사양**'이라고 표현할 수 있다.

리액티브 스트림즈 명세를 구현한 구현체로 RxJava, Reactor, Akka Streams, Java 9 Flow API 등이 있는데, 그 중 Spring과 궁합이 잘 맞는 Reactor에 대해 Part 2에서 살펴볼 예정이다.

## 2.2 리액티브 스트림즈 구성요소
- 리액티브 스트림즈를 통해 구현해야 하는 API 컴포넌트에는 Publisher, Subscriber, Subscription, Processor가 있다.

|컴포넌트|설명|
|---|---|
|Publisher|데이터를 생성하고 통지(발행, 게시, 방출)하는 역할을 한다.|
|Subscriber|구독한 Publisher로부터 통지(발행, 게시, 방출)된 데이터를 전달받아서 처리하는 역할을 한다|
|Subscription|Publisher에 요청할 데이터의 개수를 지정하고, 데이터의 구독을 취소하는 역할을 한다.|
|Processor|Publisher와 Subscriber의 기능을 모두 가지고 있다. 즉, Subscriber로서 다른 Publisher를 구독할 수 있고, Publisher로서 다른 Subscriber가 구독할 수 있다.|

Publisher와 Subscriber 간에 데이터가 전달되는 동작 과정

1. 먼저 Subscriber는 전달받을 데이터를 구독한다. (subscribe)
2. Publisher는 데이터를 통지(발행, 게시, 방출)할 준비가 되었음을 Subscriber에 알린다. (onSubscribe)
3. Publisher가 데이터를 통지할 준비가 되었다는 알림을 받은 Subscriber는 전달받기를 원하는 데이터의 개수를 Publisher에게 요청한다 (Subscription.request)
4. Publisher는 Subscriber로부터 요청받은 만큼의 데이터를 통지한다 (onNext)
5. 이렇게 Publisher와 Subscriber 간에 데이터 통지, 수신, 요청 과정을 반복하다가 Publisher가 모든 데이터를 통지하게 되면 마지막으로 데이터 전송이 완료되었음을 Subscriber에게 알린다. (onComplete)
6. 만약 데이터 처리 과정에서 에러가 발생 시 Subscriber에게 알린다. (onError)

데이터 요청 개수를 지정하는 이유
- Publisher와 Subscriber가 마치 같은 스레드에서 동기적으로 상호작용하는 것처럼 보이지만 실제로 Publisher와 Subscriber는 각각 다른 스레드에서 비동기적으로 상호작용하는 경우가 대부분이다.
- 따라서 Publisher가 통지하는 속도가 Subscriber가 처리하는 속도보다 더 빠르면 처리를 기다리는 데이터는 쌓이게 되고, 이는 결과적으로 시스템 부하가 커지는 결과를 낳게 된다.

## 2.3 코드로 보는 리액티브 스트림즈 컴포넌트
- 리액티브 스트림즈의 컴포넌트는 실제 코드에서 인터페이스 형태로 정의되며, 이 인터페이스를 구현해서 해당 컴포넌트에서 사용한다.

### 2.3.1 Publisher

``` java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

왜 Publisher가 subscribe 메서드를 정의할까?
- Kafka 같은 메시지 기반의 시스템에서 Pub/Sub 모델과는 의미가 조금 다르다.
- Kafka의 경우 중간에 메시지 브로커가 있고 이 브로커 내에 여러 개의 토픽이 존재하는데, Publisher와 Subscriber는 각각 브로커 내의 특정 토픽만 바라보면 되기 때문에 데이터를 전송하기만, 구독해서 전달받기만 하면 된다.
- 그러나 리액티브 스트림즈에서의 Publisher와 Subscriber는 Publisher가 subscribe 메서드의 파라미터인 Subscriber를 등록하는 형태로 구독이 이루어진다.

### 2.3.2 Subscriber

``` java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```

- onSubscribe 메서드는 구독 시작 시점에 어떤 처리를 하는 역할을 한다.
  - 처리는 Publisher에게 요청할 데이터의 개수를 지정하거나 구독을 해지하는 것을 의미하는데, 이것은 Subscription 객체를 통해 이루어진다.
- onNext 메서드는 Publisher가 통지한 데이터를 처리하는 역할을 한다.
- onError 메서드는 Publisher가 데이터 통지를 위한 처리 과정에서 에러가 발생 시 해당 에러를 처리하는 역할을 한다.
- onComplete 메서드는 Publisher가 데이터 통지를 완료했음을 알릴 때 호출된다. 완료 후 후처리를 진행 시 해당 메서드에서 처리 코드를 작성하면 된다.

### 2.3.3 Subscription

``` java
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```

- request 메서드를 통해 데이터의 개수를 요청한다.
- cancel 메서드를 통해 구독을 해지할 수 있다.

### 2.3.4 Processor

``` java
public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {

}
```

Processor는 Publisher와 Subscriber의 기능을 모두 갖고 있다.

## 2.4 리액티브 스트림즈 관련 용어 정의

- Signal
  - 신호라는 의미인데, 리액티브 스트림즈에서 signal은 **Publisher와 Subscriber 간에 주고 받는 상호작용**을 의미한다.
  - 리액티브 스트림즈 인터페이스 코드에서 볼 수 있는 onSubscirbe, onNext, onComplete 등의 메서드를 Signal이라고 볼 수 있다.
- Demand
  - 수요, 요구 등의 의미인데, Subscriber가 Publisher에게 요청하는 데이터를 의미한다.더 구체적으로 이야기하면 아직 Subscriber가 전달받지 못한 요청한 데이터를 의미한다.
- Emit
  - Publisher가 Subscriber에게 데이터를 전달하는 것을 일컬을 때 데이터를 '통지(발행, 게시, 방출)한다'라고 표현했다.
  - 데이터를 내보내다 정도로 이해하면 될 것 같다.
  - onNext Signal을 줄여서 데이터를 Emit한다라고 표현할 수 있다.
- Upstream/Downstream
  - 위로 흐르다, 아래로 흐르다 정도로 이해할 수 있다.
    ``` java
    public class Example2_5 {
        public static void main(String[] args) {
            Flux
                .just(1, 2, 3, 4, 5, 6)
                .filter(n -> n % 2 == 0)
                .map(n -> n * 2)
                .subscribe(System.out::println);
        }
    }
    ```
  - 위 코드에서처럼
    - just 메서드를 사용해서 데이터를 생성 후 emit한다.
    - 메서드 체이닝 방식으로 호출이 되는데, 모두 같은 타입 객체를 반환하기 때문에 체인 방식으로 호출이 가능한 것이다. 모두 Flux 타입이다.
    - 위와 같이 4번 라인이 5번 라인보다 Upstream이다.
- Sequence
  - Sequence는 Publisher가 emit하는 데이터의 연속적인 흐름을 정의해 놓은 것 자체를 의미하는데, Operator 체인 형태로 정의된다.
  - Flux를 통해 데이터를 생성, Emit하고 Filter 메서드를 통해 필터링한 후, map 메서드를 통해 변환하는 이러한 과정 자체를 의미한다.
  - **다양한 Operator로 데이터의 연속적인 흐름을 정의한 것.**
- Operator
  - 연산자 정도로 해석할 수 있다.
  - just, filter, map 같은 메서드들을 연산자라고 부른다.
  - 리액티브 프로그래밍은 Operator로 시작해서 Operator로 끝난다고 해도 과언이 아닐 만큼 리액티브 프로그래밍의 핵심이다.
- Source
  - 최초 data의 시작점
  - Original이라는 용어도 사용된다.

## 2.5 리액티브 스트림즈의 구현 규칙
- 직접 리액티브 스트림즈를 구현해서 라이브러리를 제작하는 것이 아니기 때문에 모든 규칙을 살펴보진 않을 것이다.
- 다만 기본적인 구현 규칙은 알고 있으면 좋다.

Publisher 구현을 위한 주요 기본 규칙

|번호|규칙|
|---|---|
|1|Publisher가 Subscriber에게 보내는 onNext signal의 총 개수는 항상 해당 Subscriber의 구독을 통해 요청된 데이터의 총 개수보다 더 작거나 같아야 한다.|
|2|Publisher는 요청된 것보다 적은 수의 onNext signal을 보내고 onComplete 또는 onError를 호출하여 구독을 종료할 수 있다.|
|3|Publisher의 데이터 처리가 실패하면 onError signal을 보내야 한다.|
|4|Publisher의 데이터 처리가 성공적으로 종료되면 onComplete signal을 보내야 한다.|
|5|Publisher가 Subscriber에게 onError 또는 onComplete signal을 보내는 경우 해당 Subscriber의 구독은 취소된 것으로 간주되어야 한다.|
|6|일단 종료 상태 signal을 받으면(onError, onComplete) 더 이상 Signal이 발생되지 않아야 한다.|
|7|구독이 취소되면 Subscriber는 결국 signal을 받는 것을 중지해야 한다.|

Subscriber 구현을 위한 주요 기본 규칙
|번호|규칙|
|---|---|
|1|Subscriber는 Publisher로부터 onNext signal을 수신하기 위해 Subscription.request(n)을 통해 Demand signal을 Publisher에게 보내야 한다.|
|2|Subscriber.onComplete() 및 Subscriber.onError(Throwable t)는 Subscription 또는 Publisher의 메서드를 호출해서는 안 된다.|
|3|Subscriber.onComplete() 및 Subscriber.onError(Throwable t)는 signal을 수신한 후 구독이 취소된 것으로 간주해야 한다.|
|4|구독이 더 이상 필요하지 않은 경우 Subscriber는 Subscription.cancel()을 호출해야 한다.|
|5|Subscriber.onSubscribe()는 지정된 Subscriber에 대해 최대 한 번만 호출되어야 한다.|

Subscription 구현을 위한 주요 기본 규칙
|번호|규칙|
|---|---|
|1|구독은 Subscriber가 onNext 또는 onSubscribe 내에서 동기적으로 Subscription.request를 호출하도록 허용해야 한다.|
|2|구독이 취소된 후 추가적으로 호출되는 Subscription.request(long n)은 효력이 없어야 한다.|
|3|구독이 취쇠된 후 추가적으로 호출되는 Subscription.cancel()은 효력이 없어야 한다.|
|4|구독이 취소되지 않은 동안 Subscription.request(long n)의 매개변수가 0보다 작거나 같으면 IllegalArgumentException과 함께 onError signal을 보내야 한다.|
|5|구독이 취소되지 않은 동안 Subscription.cancel()은 Publisher가 Subscriber에게 보내는 signal을 결국 중지하도록 요청해야 한다.|
|6|구독이 취소도죄 않은 동안 Subscription.cancel()은 Publisher에게 해당 구독자에 대한 참조를 결국 삭제하도록 요청해야 한다.|
|7|Subscription.cancel(), Subscription.request() 호출에 대한 응답으로 예외를 던지는 것을 허용하지 않는다.|
|8|구독은 무제한 수의 Request 호출을 지원해야 하고 최대 2^63-1개의 demand를 지원해야 한다.|

## 2.6 리액티브 스트림즈 구현체

RxJava
- Rxjava에서 Rx는 Reactive Extentions(리액티브 확장)이라는 의미이다. RxJava는 .NET 환경의 리액티브 확장 라이브러리를 넷플릭스에서 Java 언어로 포팅하여 만든 JVM 기반의 대표적인 리액티브 확장 라이브러리이다.

Project Reactor
- Project Reactor는 Spring Framework 팀에 의해 주도적으로 개발된 리액티브 스트림즈의 구현체이다.
- Reactor 3.x가 Spring Framework 5 버전부터 리액티브 스택에 포함되어 리액티브 프로그래밍의 핵심 역할을 담당한다.
- Spring WebFlux 기반의 리액티브 어플리케이션 도입을 위해서 Reactor를 충분히 이해하지 않으면 분명 한계에 부딪히게 될 것이다.

Akka Streams
- Akka는 JVM 상에서의 동시성과 분산 어플리케이션을 단순화해 주는 오픈소스 툴킷이다.
- 이 Akka라는 Actor 기반의 동시성 모델을 사용하는 툴킷 위에 리액티브 스트림즈를 구현한 것이 바로 Akka Streams이다.
- Akka Streams API와 다르다.

Java Flow API
- Java 9부터 Flow API를 사용하여 리액티브 스트림즈를 지원하게 되었다.
- 그러나 다른 구현체와 차이점이 있는데, Flow API는 리액티브 스트림즈를 구현한 구현체가 아닌, 리액티브 스트림즈의 표준 사양이 SPI(Service Provider Interface)로써 Java API에 정의되어 있다.

## 마무리
- 리액티브 스트림즈는 데이터 스트림을 Non-Blocking이면서 비동기적인 방식으로 처리하기 위한 리액티브 라이브러리의 표준 사양이다.
- 리액티브 스트림즈는 Publisher, Subscriber, Subscription, Processor라는 네 개의 컴포넌트로 구성되어 있다. 구현체는 이 네 개의 컴포넌트를 사양과 규칙에 맞게 구현해야 한다.
- 리액티브 스트림즈 구현체 중에서 어떤 구현체를 학습하든지 핵심 동작 원리는 같다.