# 3장 단위 테스트 구조
> 단위 테스트 구성, 테스트 픽스터 재사용, 테스트 매개변수화와 같은 몇 가지 기본 주제를 되짚어본다.
> 테스트 코드 구조로부터 운영 코드의 문제점을 찾을 수 있다.

## 3.1 단위 테스트를 구성하는 방법
- 이 절에서는 준비, 실행, 검증 패턴을 사용해 단위 테스트를 구성하는 방법, 피해야 할 함정 그리고 테스트를 가능한 한 읽기 쉽게 만드는 방법 등을 알아본다.

### 3.1.1 AAA 패턴 사용
- AAA 패턴은 준비(Arrange), 실행(Act), 검증(Assert)이라는 세 부분으로 나눌 수 있다.
- 두 숫자의 합을 계산하는 메서드만 있는 Calculator 클래스를 살펴보면

``` java
public class Calculator
{
    public double sum(double first, double second)
    {
        return first + second;
    }
}
```

위 클래스의 동작을 검증하는 AAA 패턴의 테스트를 작성한다면

``` java
public class CalculatorTests
{
    [Fact]
    public void Sum_of_two_numbers() 
    {
        // 준비
        double first = 10;
        double second = 20;
        var calculator = new Calculator();

        // 실행
        double result = calculator.Sum(first, second);

        // 검증
        Assert.Equal(30, result);
    }
}
```

- AAA 패턴은 스위트 내 모든 테스트가 단순하고 균일한 구조를 갖는 데 도움이 된다.
  - 이러한 일관성이 이 패턴의 가장 큰 장점 중 하나다.
  - 일단 익숙해지면 모든 테스트를 쉽게 읽을 수 있고 이해할 수 있다.
  - 결국 전체 테스트의 유지 보수 비용이 줄어든다.
- 구조는 다음과 같다.
  - 준비 구절에서는 SUT과 해당 의존성을 원하는 상태로 만든다.
  - 실행 구절에서는 SUT에서 메서드를 호출하고 준비된 의존성을 전달하며 출력이 있는 경우 캡쳐해둔다.
  - 검증 구절에서는 결과를 검증한다. 결과는 반환 값이나 SUT와 ㅎ벼력자의 최종 상태, SUT가 협력자에 호출한 메서드 등으로 표시될 수 있다.

> Given-When-Then 패턴
> - AAA와 유사한 Given When Then 패턴에 대해 들어봤을 것이다. 이 패턴도 테스트를 세 부분으로 나눈다.
>   - Given - 준비 구절에 해당
>   - When - 실행 구절에 해당
>   - Then - 검증 구절에 해당
> 테스트 구성 측면에서 두 가지 패턴 사이에 차이는 없다. 유일한 차이점은 프로그래머가 아닌 사람에게 Given-When-Then 구조가 더 읽기 쉽다는 것이다.

- 준비 구절 먼저 시작하는게 자연스럽긴 한데, 검증 구절로 시작하는 경우도 있다.
  - TDD의 경우와 같은데, 기능을 개발하기 전에 실패할 테스트를 만들 때는 아직 기능이 어떻게 동작할지 충분히 알지 못한다.
  - 하지만 제품 코드 전에 테스트를 작성할 때만 적용될 수 있다.
- 테스트 전에 제품 코드를 작성한다면 준비 구절부터 시작하는 것이 좋다.

### 3.1.2 여러 개의 준비, 실행, 검증 구절 피하기
- 준비, 실행 또는 검증 구절이 여러 개 있는 테스트를 만날 수 있는데, 이 경우는 테스트가 너무 많은 것을 한 번에 검증한다는 의미이다.
  - 이런 경우에는 여러 테스트로 나눠서 해결한다.
  - 이러한 테스트는 더 이상 단위 테스트가 아니라 통합 테스트이다.
  - 실행이 하나면 테스트가 단위 테스트 범주에 있게끔 보장하고, 간단하고, 빠르며, 이해하기 쉽다.
  - 각 동작을 고유의 테스트로 도출하라.

### 3.1.3 테스트 내 if 문 피하기
- 준비, 실행, 검증 구절이 여러 차례 나타나는 것과 비슷하게, if 문이 있는 단위 테스트를 만날 수 있다. 이것 역시 안티 패턴이다.
  - 테스트는 분기가 없는 간단한 일련의 단계여야 한다.
  - if 문은 테스트가 한 번에 너무 많은 것을 검증한다는 표시다.
  - 그러므로 이러한 테스트는 반드시 여러 테스트로 나눠야 한다.
  - 테스트에 분기가 있어서 얻는 이점은 없다. 읽고 이해하는 것을 더 어렵게 만든다.

### 3.1.4 각 구절은 얼마나 커야 하는가?
- 각 구절의 크기가 얼마나 되는가?
- 테스트가 긑난 후에 정리하는 종료 구절은 어떻게 하는가?

#### 준비 구절이 가장 큰 경우
- 일반적으로 준비 구절이 세 구절 중 가장 큰다, 이보다 훨씬 크면, 같은 테스트 클래스 내 비공개 메서드 또는 별도의 팩토리 클래스로 도출하는 것이 좋다. (text fixture)
- 준비 구절에서 코드 재사용에 도움이 되는 두 가지 패턴으로 오브젝트 마더와 테스트 데이터 빌더가 있다.

#### 실행 구절이 한 줄 이상인 경우를 경계하라
- 실행 구절은 보통 코드 한 줄이다.
- 실행 구절이 두 줄 이상인 경우 SUT의 공개 API에 문제가 있을 수 있다.
  - 여러 액션이 발생한다 하더라도, 하나의 공개 메서드로 묶어야 한다.
  - 단일 공개 메서드가 아닐 경우, 클라이언트 코드가 첫 번째를 호출하고 두 번째를 또 호출하지 않으면 모순이 생긴다.
  - 모순을 불변 위반(invariant violation)이라고 하며, 잠재적 모순으로부터 코드를 보호하는 행위를 캡슐화(encapsulation)라고 한다.
  - 데이터베이스에 모순이 생기면, 단순 어플리케이션 재시작으로는 해결이 될 수 없다.
- 해결책은 코드 캡슐화를 항상 지키는 것이다.

### 3.1.5 검증 구절에는 검증문이 얼마나 있어야 하는가
- 마지막으로 검증 구절이 있다. 테스트당 하나의 검증을 갖는 지침을 들어봤을 것이다.
  - 가능한 한 가장 작은 코드를 목표로 하는 전제에 기반을 두고 있다.
  - 이미 알고 있듯이 이 전제는 올바르지 않은게, 단위 테스트의 단위는 동작의 단위이지 코드의 단위가 아니다.
  - 하지만 검증 구절이 너무 커지는 것은 경계해야 한다. 제품 코드에서 추상화가 누락됐을 수 있다.
    - 예를 들어, SUT에서 반환된 객체 내에서 모든 속성을 검증하는 대신 객체 클래스 내에 적절한 동등 멤버를 정의하는 것이 좋다. (Equals와 같은)
    - 그러면 단일 검증문으로 객체를 기대값과 비교할 수 있다.

### 3.1.6 종료 단계는 어떤가
- 예를 들면 테스트에 의해 작성된 파일을 지우거나 데이터베이스 연결을 종료하는 등의 행위이다.
- 일반적으로 별도의 메서드로 도출돼, 클래스 내 모든 테스트에서 재사용된다.
- 하지만 대부분 단위 테스트는 종료 구절이 필요 없다.
  - 단위 테스트는 프로세스 외부에 종속적이지 않으므로 처리해야 할 사이드 이펙트를 남기지 않는다.

### 3.1.7 테스트 대상 시스템 구별하기
- SUT는 테스트에서 중요한 역할을 하는데, 어플리케이션에서 호출하고자 하는 동작에 대한 진입점을 제공한다.
  - 동작은 여러 클래스에 걸쳐 클 수도 있고, 단일 메서드로 작을 수도 있다.
  - 그러나 진입점은 오직 하나만 존재할 수 있다.
- 따라서 SUT을 의존성과 구분하는 것이 중요하다.
  - 특히 SUT가 꽤 많은 경우, 테스트 대상을 찾는 데 시간을 너무 많이 들일 필요가 없다.
    - 따라서 테스트 내 SUT 이름을 sut으로 하라.
```
var sut = new Calculator();
```

### 3.1.8 준비, 실행, 검증 주석 제거하기
- 테스트 내에서 특정 부분이 어떤 구절에 속해 있는지 파악하는 데 시간을 많이 들이지 않도록 세 구절을 서로 구분하는 것 역시 중요하다.
- 이를 위한 한 가지 방법은 각 구절을 시작하기 전에 주석 (// 준비, // 실행, // 검증)을 다는 것이다.
  - 다른 방법은 빈 줄로 분리하는 것이다.
- 빈 줄로 구절을 구분하면 대부분의 단위 테스트에서 효과적이며, 간결성 가독성 사이에서 균형을 잡을 수 있다.
  - AAA 패턴을 따르고 준비 및 검증 구절에 빈 줄을 추가하지 않아도 되면 구절 주석을 제거하라.
  - 그렇지 않으면 구석 주석을 유지하라.

## 3.2 xUnit 테스트 프레임워크 살펴보기
- 단위 테스트 프레임워크에 대한 설명

## 3.3 테스트 간 테스트 픽스터 재사용
- 테스트에서 언제 어떻게 코드를 재사용하는지 아는 것이 중요하다.
  - **매번 다른 기준으로 재사용하지 말고 나만의 기준을 정하자.**
- 준비 구절에서 코드를 재사용하는 것이 테스트를 줄이면서 단순화하기 좋은 방법이고, 이 절에서는 올바른 방법을 알아본다.
- 테스트 픽스처를 준비하기 위해 코드를 너무 많이 작성해야 하는데, 이러한 준비는 별도의 메서드나 클래스로 도출한 후 테스트간에 재사용하는 것이 좋다.

> 테스트 픽스처
> - 테스트 픽스처라는 단어는 다음과 같이 두 가지 공통된 의미가 있다.
>   - 테스트 픽스처는 테스트 실행 대상 객체다. 이 객체는 정규 의존성, 즉 SUT으로 전달되는 인수다. 이러한 객체는 각 테스트 실행 전에 알려진 고정 상태로 유지하기 때문에 동일한 결과를 생성한다. 따라서 픽스처라는 단어가 나왔다.

- 테스트 픽스처를 재사용하는 첫 번째 올바르지 않은 방법은 테스트 생성자에서 픽스처를 초기화하는 것이다. (setup)

``` java
public class CustomerTests {
    private readonly Store _store;
    private readonly Customer _sut;

    public CustomerTests() {
        _store = new Store();
        _store.AddInventory(Product.Shampoo, 10);
        _sut = new Customer();
    }
    // 클래스 내 각 테스트 이전에 호출
}
```

- 실제로 준비 구절이 동일하므로 CustomerTests의 생성자로 완전히 추출할 수 있었다.
- 테스트 코드의 양을 크게 줄일 수 있었지만, 두 가지 중요한 단점이 있다.

1. 테스트 간 결합도가 높아진다.
2. 테스트 가독성이 떨어진다.

### 3.3.1 테스트 간의 높은 결합도는 안티 패턴이다.
- 모든 테스트가 서로 결합돼 있다.
- 즉, 테스트의 준비 로직을 수정하면 클래스의 모든 테스트에 영향을 미친다.
- 이는 중요한 지침을 위반하는데, 테스트를 수정해도 다른 테스트에 영향을 주어서는 안된다.
  - 테스트는 서로 격리돼 실행해야 한다는 것과 비슷하다.
  - 이 지침을 따르려면 테스트 클래스에 공유 상태를 두지 말아야 한다.

### 3.3.2 테스트 가독성을 떨어뜨리는 생성자 사용
- 테스트만 보고는 더 이상 전체 그림을 볼 수 없다.
- 테스트 메서드가 무엇을 하는지 이해하기 위해서는 다른 부분도 봐야한다.

### 3.3.3 더 나은 테스트 픽스처 재사용법
- 테스트 픽스처를 재사용할 때 생성자 사용이 최선의 방법은 아니다.
- 두 번째 방법은 다음 예제와 같이 테스트 클래스에 비공개 팩토리 메서드(private factory method)를 두는 것이다.

``` java
private Store CreateStoreWithInventory(Product product, int quantity) {
    Store store = new Store();
    store.addInventory(product, quantity);
    return store;
}
```

- 공통 초기화 코드를 비공개 팩토리 메서드로 추출해 테스트 코드를 짧게 하면서, 동시에 테스트 진행 상황에 대한 전체 맥락을 유지할 수 있다.
  - 또한 테스트가 서로 결합되지 않는다.
  - 팩토리 메서드를 통해 생성 시 매우 읽기 쉽고 재사용이 가능하다.
- 한 가지 예외 규칙이 있는데,
  - 테스트 전부 또는 대부분에 사용되는 생성자에 픽스처를 인스턴스화할 수 있다.
  - 이는 데이터베이스와 작동하는 통합 테스트에 종종 해당한다.

## 3.4 단위 테스트 명명법
- 테스트에 표현력이 있는 이름을 붙이는 것이 중요하다.
  - 올바른 명칭은 테스트가 검증하는 내용과 기본 시스템의 동작을 이해하는 데 도움이 된다.
- 가장 유명하지만 가장 도움이 되지 않는 관습이 있는데,
  - [테스트 대상 메서드]_ [시나리오] _[예상 결과]
    - 테스트 중인 메서드 이름 / 메서드를 테스트하는 조건 / 현재 시나리오에서 테스트 대상 메서드에 기대하는 것
  - 동작 대신 구현 세부 사항에 집중하게끔 부추기기 때문에 분명히 도움이 되지 않는다.
  - 간단하고 쉬운 영어 구문이 훨신 더 효과적이다.
- 다른 사람의 코드를 읽는 것은 이미 충분히 어렵다.
  - 테스트가 정확히 무엇을 검증하는지, 비즈니스 요구 사항과 어떤 관련이 있는지 파악하려면 머리를 더 써야 한다.
  - 전체 테스트 스위트의 유지비가 천천히 늘어난다.

### 3.4.1 단위 테스트 명명 지침
- 표현력 있고 읽기 쉬운 테스트 이름을 지으려면 다음 지침을 따르자.
  - 엄격한 명명 정책을 따르지 않는다. 표현의 자유를 허용하자.
- 문제 도메인에 익숙한 비개발자들에게 시나리오를 설명하는 것처럼 테스트 이름을 짓자. 도메인 전문가나 비즈니스 분석가가 좋은 예다.
- 단어를 밑줄 표시로 구분한다. 그러면 특히 긴 이름에서 가독성을 향상시키는 데 도움이 된다.

### 3.4.2 예제: 지침에 따른 테스트 이름 변경
- 테스트는 DeliveryService가 잘못된 날짜의 배송을 올바르게 식별하는지 검증한다.

1. IsDeliveryValid_InvalidDate_ReturnFalse()
- -> 쉬운 영어로 변경
2. Delivery_with_invalid_Date_should_be_considered_invalid()
- -> 사람들에게도 납득되고, 프로그래머도 이해가 가능하다. SUT의 메서드 이름은 더 이상 포함되지 않는다. (SUT의 메서드 이름을 테스트 이름에 포함하지 말자.)
3. Delivery_with_past_date_should_be_considered_invalid()
- -> 이상적이지 않고 장황하다. considered를 제거한다.
4. Delivery_with_past_Date_should_be_invalid()
- -> should be 는 안티패턴으로 is로 바꿔보자.
5. Delivery_with_past_date_is_invalid()
- -> 마지막으로 기초 영문법을 지키자.
6. Delivery_with_a_past_date_is_invalid()

## 3.5 매개변수화된 테스트 리팩터링하기 (parameterized test)
- 보통 테스트 하나로는 동작 단위를 완전하게 설명하기에 충분하지 않다.
- 각 구성 요소는 자체 테스트로 캡쳐해야 한다.
  - 이를 설명하는데 테스트 수가 급격히 증가할 수 있지만, 매개변수화된 테스트를 통해 유사한 테스트를 묶을 수 있는 기능을 제공한다.
- 가장 빠른 배송일이 오늘로부터 이틀 후가 되도록 작동하는 배송 기능이 있다고 가정해보자.
  - 분명히 테스트 하나로는 충분하지 않다.
  - 배송날짜만 변경되지만 테스트 메서드를 계속 생성할 수는 없다.
- 이런 경우 테스트 코드의 양을 줄이고자 이러한 테스트를 하나로 묶는 것이다.
- 매개변수화된 테스트를 사용하면 테스트 코드의 양을 크게 줄일 수 있지만, 비용이 발생하는데,
  - 테스트 메서드가 나타내는 사실을 파악하기가 어려워졌다.
  - 그리고 매개변수가 많을수록 더 어렵다.
  - 절충안으로 긍정적인 테스트 케이스는 고유한 테스트로 도출하고, 가장 중요한 부분을 잘 설명하는 이름을 쓰면 좋다.
    - 이렇게 되면, 붖어적인 테스트 케이스를 단순하게 할 수 있는데, 테스트 메서드에서 예상되는 Boolean 매개변수를 제거할 수 있기 때문이다.
      - **긍정/부정까지 한꺼번에 테스트하지 말고 메서드 자체를 분리하기**
      - 테스트 코드의 양과 그 코드의 가독성은 서로 상충된다.
        - 입력 매개변수만으로 테스트 케이스를 판단할 수 있다면 긍정적인 테스트 케이스와 부정적인 테스트 케이스 모두 하나의 메서드로 두는 것이 좋다.
        - 그렇지 않으면 긍정적인 테스트 케이스를 도출하라.
        - 그리고 동작이 너무 복잡하면 매개변수화된 테스트를 조금도 사용하지 말라.

### 3.5.1 매개변수화된 테스트를 위한 데이터 생성
- 데이터 설명

## 3.6 검증문 라이브러리를 사용한 테스트 가독성 향상
- 검증문 라이브러리를 사용하면 테스트 가독성을 높힐 수 있다.
  - 검증문을 쉬운 영어로 읽을 수 있다.
``` java
var sut = new Calculator();
double result = sut.Sum(10, 20);
result.Should().Be(30);
```
  - 주어 행동 목적어.
  - 코드에도 같은 규칙이 적용된다.

## 요약
- 모든 단위 테스트는 AAA 패턴(준비, 실행, 검증)을 따라야 한다.
  - 테스트 내 구절이 여러 개 있으면, 테스트가 여러 동작을 한 번에 검증한다는 표시다.
  - 단위 테스트일 경우 여러 테스트로 나눠야 한다.
- 실행 구절이 한 줄 이상이면 SUT의 API에 문제가 있다는 뜻이다.
  - 하나의 공개된 단일 메서드로 표현하자.
  - 불변 위반의 모순으로 이어질 수 있다.
- SUT의 이름을 sut으로 지정해 SUT을 테스트에서 구별하자.
  - 구절 사이에 빈 줄을 추가하거나, 준비 / 실행/ 검증 구절 주석을 둬서 구분하자.
- 테스트 픽스처 초기화 코드는 생성자에 두지 말고 팩토리 메서드를 도입해서 재사용하자.
  - 이러한 재사용은 테스트 간 결합도를 상당히 낮게 유지하고 가독성을 향상시킨다.
- 엄격한 테스트 명명 정책을 시행하지 말라.
  - 밑줄로 단어를 구분하고 테스트 대상 메서드 이름을 넣지 말자.
- 매개변수화된 테스트로 유사한 테스트에 필요한 코드의 양을 줄일 수 있다.
  - 단점은 테스트 이름이 포괄적으로 만들수록 이름을 읽기 어렵게 하는 것이다.
- 검증문 라이브러리를 이용하면 쉬운 영어처럼 읽을 수 있게 되어 테스트 가독성을 더욱 향상시킬 수 있다.

> 1. 테스트 코드로 운영 코드가 잘 짜여졌는지 검증이 가능하다고 느꼈다.
> 2. 테스트 픽스처를 사용 시, 재사용성과 결합도를 낮추기 위해 private 팩토리 메서드로 만들어서 사용했었는데 테스트 생성자 혹은 setUp 단계에서 초기화를 해버림으로써 결합도가 높아지는 상황에 대해 공감이 되었다.
> 3. 파라미터화된 테스트를 즐겨쓰는데, 예시에 나온 것처럼 긍정/부정 테스트를 모두 한꺼번에 파라미터로 구분했더니 확실히 복잡도가 증가했었다. 코드 리뷰에서 받은 내용으로 긍정과 부정을 나누어 그 둘을 구분하는 Boolean 파라미터를 줄일 수 있다는 의견을 받았고 공감이 많이 되었는데 해당 내용이 나와서 반가웠다.
> 4. 확실히 나도 작성하면서 파라미터가 많아지면서 헷갈렸는데 코드의 양과 코드의 가독성은 서로 상충된다는 점이 공감됐다.