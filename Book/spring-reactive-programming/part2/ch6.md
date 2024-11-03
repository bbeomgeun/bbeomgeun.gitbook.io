# Chapter 06 마블 다이어그램(Marble Diagram)
- 마블 다이어그램과 마블 다이어그램을 통해 Reactor의 Publisher와 Operator를 이해해보자.

## 6.1 마블 다이어그램(Marble Diagram)이란?
- 리액티브 프로그래밍을 경험한 적이 있다면 마블 다이어그램이라는 용어를 한 번쯤 들어봤을 것이다.
- 마블은 우리말로 구슬이라는 의미가 있고, 다이어그램은 여러 가지 도형들로 그려진 도표를 뜻한다.
- 즉, 여러 가지 구슬 모양의 도형으로 구성된 도표로 Reactor에서 지원하는 Operator를 이해하는 데 중요한 역할을 한다.
- 비동기적인 데이터 흐름을 시간의 흐름에 따라 시각적으로 표시한 다이어그램이라고 표현할 수 있다.
- 마블 다이어그램을 몰라도 리액티브 프로그래밍 코드를 짤 수는 있다.
  - 하지만 마블 다이어그램을 먼저 보고 난 다음 API 문서를 읽으면 이해도에 상당한 도움을 받을 수 있을 것이다.
  - 처음 사용해보는 Operator를 올바르게 이해하고 사용하기 위해서는 마블 다이어그램을 먼저 확인하는 습관을 들이자.

## 6.2 마블 다이어그램으로 Reactor의 Publisher 이해하기
### Mono 다이어그램
- Mono는 단 하나의 데이터를 emit하는 Publisher이기에 마블 다이어그램에서도 단 하나의 데이터만 표현한다.
  - 정확히 말하면 Mono는 0개 또는 1개의 데이터를 emit하는 Publisher이다.
- 마블 다이어그램을 통해 이해한 Mono를 어떤 식으로 사용하는지 코드로 알아보자

``` java
public class Example6_1 {
    public static void main(Strings[] args) {
        Mono.just("Hello Reactor")
            .subscribe(System.out::println);
    }
}
// 실행결과 : Hello Reactor
```
- just() Operator는 한 개 이상의 데이터를 emit하기 위한 대표적인 Operator로서 2개 이상의 데이터를 파라미터로 전달할 경우, 내부적으로 fromArray() Operator를 이용해서 데이터를 emit한다.

``` java
public class Example6_2 {
    public static void main(String[] args) {
        Mono
            .empty()
            .subscribe(
                // onNext Signal 전송 시 실행되는 람다 표현식
                none -> System.out.println("# emitted onNext signal"),
                // onError Signal 전송 시 실행되는 람다 표현식
                error -> {},
                // onComplete Signal 전송 시 실행되는 람다 표현식
                () -> System.out.println("# emitted onComplete signal")
            )
    }
}
// 실행 결과 : # emitted onComplete signal
```
- 데이터를 한 건도 emit하지 않는 예제 코드이다. 4번 라인의 empty() Operator를 사용하면 데이터를 emit하지 않고 OnComplete signal을 전송한다.
- 이처럼 데이터를 한 건도 emit하지 않는 empty() Operator는 주로 어떤 특정 작업을 통해 데이터를 전달받을 필요는 없지만 **작업이 끝났음을 알리고 이에 따른 후처리를 하고 싶을 때 사용할 수 있다.**

``` java
public class Example6_3 {
    public static void main(String[] args) {
        URI worldTimeUri = UriComponentsBuilder.newInstance().scheme("http")
            .host("worldtimeapi.org")
            .port(80)
            .path("/api/timezone/Asia/Seoul")
            .build()
            .encode()
            .toUri();

        RestTemplate restTemplate = new RestTemplate();
        HttpHeaders headers = new HttpHeaders();
        headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));


        Mono.just(
            restTemplate
                .exchange(worldTimeUri,
                    HttpMethod.GET,
                    new HttpEntity<String>(headers),
                    String.class)
            )
            .map(response -> {
                DocumentContext jsonContext = 
                    JsonPath.parse(response.getBody());
                String dateTime = jsonContext.read("$.datetime");
                return dateTime;
            })
            .subscribe(
                data -> System.out.println("# emitted data: " + data),
                error -> {
                    System.out.println(error);
                },
                () -> System.out.println("# emitted onComplet signal")
            );
    }
}
// 실행 결과
// # emitted data : 2022-02-08T16:15:15
// # emitted onComplete signal
```

- 16~22번 라인에서 Mono의 just() Operator에 외부 시스템의 API 응답으로 수신한 데이터를 전달한다.
- 그리고 map() Operator에서 응답으로 수신한 JSON 형식의 데이터를 파싱해서 Subscriber에게 전달한 후 로그로 출력한다.
- 이처럼 Mono는 단 한 건의 데이터를 응답으로 보내는 HTTP 요청/응답에 사용하기 아주 적합한 Publisher 타입이다.
- 물론 6-3 예제코드는 Non-Blocking I/O 방식의 통신이 아니기에 Non-Blocking 통신의 이점은 얻을 수 없다. 하지만 Mono를 이용 시 요청과 응답을 하나의 Operator 체인으로 깔끔하게 처리할 수 있는 장점이 있다는 사실을 기억하자.

### Flux 다이어그램
- Mono와 다르게 emit되는 구슬 모양의 데이터가 여러 개이다.
- Flux는 여러 건의 데이터를 emit할 수 있는 Publisher로, 0 ~ 1개 이상의 데이터를 emit할 수 있기에 Mono를 포함한다.

``` java
public class Example6_4 {
    public static void main(String[] args) {
        Flux.just(6, 9, 13)
            .map(num -> num % 2)
            .subscribe(System.out::println);
    }
}
// 실행 결과
// 0 1 1
```
- emit된 세 개의 숫자를 Operator를 통해 연산 후 Subscriber에게 전달하여 출력한다.

``` java
public class Example6_5 {
    public static void main(String[] args){
        Flux.fromArray(new Integer[]{3, 6, 7, 9})
            .filter(num -> num > 6)
            .map(num -> num * 2)
            .subscribe(System.out::println);
    }
}
// 14 18
```
- 데이터 소스로 사용되는 배열을 처리하기 위해 fromArray Operator를 사용한다.

``` java
public class Example6_6 {
    public static void main(String[] args) {
        Flux<String> flux = 
            Mono.justOrEmpty("Steve")
                .concatWith(Mono.justOrEmpty("Jobs"));
        flux.subscribe(System.out::println);
    }
}
// Steve\n Jobs
```
- 2개의 Mono를 연결하여 Flux로 변환하는 예제
- concat은 String에서 익숙할 수 있지만, 문자열 연산처럼 데이터 자체를 이어붙여서 하나의 데이터를 emit하는 것이 아니라, emit할 데이터를 일렬로 줄 세워서 하나의 데이터 소스를 만든 후에 차례차례 데이터를 emit한다.
  - 두 개의 데이터 소스를 연결해서 하나의 데이터 소스를 만든다는 점을 기억하자!

``` java
public class Example6_7 {
    public static void main(String[] args) {
        Flux.concat(
                Flux.just("Mercury", "Venus", "Earth"),
                Flux.just("Mars", "Jupiter", "Saturn"),
                Flux.just("Uranus", "Neptune", "Pluto"))
            .collectList()
            .subscribe(plantets -> System.out.println(planets));
    }
}
```
- concatWith()은 두 개의 데이터소스를 연결한다면, concat()은 여러 개의 데이터 소스를 원하는 만큼 연결할 수 있다.

Quiz
1. 3번 라인의 concat() Operator에서 리턴하는 Publisher는 Mono일까 Flux일까?
2. 7번 라인의 collectList() Operator에서 리턴하는 Publisher는 Mono일까 Flux일까?
3. 8번 라인에서 최종적으로 출력되는 데이터의 형태는?

- 1번 정답은 Flux이다.
  - 3번 라인의 concat()을 이용해서 총 세 개의 FLux의 9개 데이터를 데이터 소스로 연결하기 때문에 concat()의 리턴 값은 Flux가 될 수밖에 없다.
- 2번 정답은 Mono이다.
  - collectList() Operator는 여러 개의 데이터를 하나의 List에 원소로 포함시킨다.
  - 즉, List Size는 N이지만, List 자체는 하나이기 때문에 한 개의 데이터만 Emit하는 Mono를 리턴한다.
- 3번 정답은 List 객체 출력이기에 List 원소가 차례대로 출력된다.

## 정리
- 이번 챕터에서는 마블 다이어그램이 무엇인지, 마블 다이어그램을 보는 방법, 마블 다이어그램을 통해 Reactor에서 지원하는 Publisher 등에 대해서 살펴봤다.
- 마블 다이어그램은 Reactor Sequence에서 진행되는 데이터의 흐름을 이해하는데 굉장히 중요한 역할을 한다.
