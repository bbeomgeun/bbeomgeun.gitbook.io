# Chapter 05 Reactor 개요

## 5.1 Reactor란?
- Reactor는 Spring Framework 팀의 주도하에 개발된 리액티브 스트림즈의 구현체로, Spring Framework 5 버전부터 리액티브 스택에 포함되어 Spring WebFlux 기반의 리액티브 애플리케이션을 제작하기 위한 핵심 역할을 담당한다.
- 리액티브 스트림즈의 구현체인 Reactor는 리액티브 프로그래밍을 위한 라이브러리이다.
  - 따라서 Reactor Core 라이브러리는 Spring Webflux 프레임워크에 라이브러리로 포함되어 있다.
  - spring-boot-starter-webflux

Reactor의 특징
1. Reactive Streams
   - Reactor는 리액티브 스트림즈 사양을 구현한 리액티브 라이브러리이다.
2. Non-Blocking
   - Reactor는 JVM 위에서 실행되는 Non-Blocking 어플리케이션을 제작하기 위한 필요한 핵심 기술이다.
3. Java's functional API
   - Reactor에서 Publisher와 Subscriber 간의 상호 작용은 Java의 함수형 프로그래밍 API를 통해서 이루어진다.
   - 람다 표현식을 활용한 함수 호출 시 파라미터로 전달하는 함수형 프로그래밍 방식이다.
4. Flux[N]
   - Reactor의 Publisher 타입은 크게 두 가지인데, 그 중 하나는 Flux이다.
   - Flux[N]이라는 의미는 N개의 데이터를 emit한다는 것이고, 0~N, 즉 무한대의 데이터를 Emit할 수 있는 Reactor의 Publisher이다.
5. Mono[0|1]
   - Mono 역시 Publisher 타입인데, 단 한 건도 emit하지 않거나 단 한 건만 emit하는 단발성 데이터 emit에 특화된 Publisher이다.
6. Well-suited for microservices
   - Non-Blocking I/O 특징을 가지는 Reactor는 마이크로 서비스 기반 시스템에서 수많은 서비스들 간에 지속적으로 발생하는 I/O를 처리하기에 적합한 기술이다.
7. Backpressure-ready network
   - Reactor는 Publisher로부터 전달 받은 데이터를 처리하는 데 있어 과부하가 걸리지 않도록 제어하는 Backpressure를 지원한다.
   - Backpressure는 배압이라고도 하는데 현실 세계에서는 배관으로 유입되는 가스나 액체 등의 흐름을 제어하기 위해 역으로 가해지는 압력을 의미한다.
   - 리액티브에서도 비슷한데, Publisher로부터 전달되는 대량의 데이터를 Subscriber가 적절하게 처리하기 위한 제어 방법이기 때문이다.

## 5.2 Hello Reactor 코드로 보는 Reactor의 구성요소

``` java
public class Example5_1 {
    public static void main(String[] args) {
        Flux<String> sequence = Flux.just("Hello", "Reactor");
        sequence.map(data -> data.toLowerCase())
            .subscribe(data -> System.out.println(data));
    }
}
```

- 3번 라인의 Flux는 Reactor에서 Publisher 역할을 한다.
  - just 메서드의 파라미터로 전달된 문자열이 바로 입력으로 들어오는 데이터이고, Publisher가 최초로 제공하는 가공되지 않은 데이터로서 **데이터 소스**라고 불린다.
  - N건의 데이터이기에 Flux를 사용했다.
- Publisher가 있으니 Subscriber도 있을텐데, 5번 라인의 람다 표현식인 (data -> System.out.println(data))가 바로 Subscriber 역할을 한다.
  - 이 람다 표현식은 Consumer 함수형 인터페이스이며, 내부적으로는 LambdaSubscriber라는 클래스에 전달되어 데이터를 처리하는 역할을 한다.
- just(), map()은 Reactor에서 지원하는 Operator 메서드인데, just()는 데이터를 생성해서 제공하는 역할, map()은 전달받은 데이터를 가공하는 역할을 한다.

## 정리
1. 데이터를 생성해서 제공하고 (Publisher)
2. 데이터를 가공한 후에 (Operator)
3. 전달 받은 데이터를 처리한다 (Subscriber)
- 이 3가지 단계는 데이터를 가공 처리하는 단계가 얼마나 복잡해지느냐, 스레드를 어떤 식으로 제어하느냐 등의 추가 작업과 상관없이 수행되는 필수 단계이다.