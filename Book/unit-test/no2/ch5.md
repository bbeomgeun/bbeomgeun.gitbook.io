# 5장 목과 테스트 취약성
> mock에 대한 사례를 구축하고 테스트 취약성과의 관계를 알아본다.

- 4장에서 배운 단위 테스트 분석 기준이 실제로 적용되는 것을 보고, 목에 대한 주제를 분석한다.
- 테스트에서 목을 사용하는 것은 논란의 여지가 있는 주제다.
  - 목은 훌륭한 도구이며 대부분의 테스트에 적용해야 한다.
  - 혹은 목이 테스트 취약성을 초래하며 사용하지 말아야 한다고 주장한다.
- 이 장에서는 목이 취약한 테스트, 즉 리팩터링 내성이 부족한 테스트를 초래하는 것을 살펴본다.
- 런던파는 테스트 대상 코드 조각을 서로 분리하고 불변 의존성을 제외한 모든 의존성에 테스트 대역을 쓰는 것을, 고전파는 테스트 간의 공유하는 의존성에 대해서만 테스트 대역을 사용한다.
- 목과 테스트 취약성 사이에는 깊고 불가피한 관련이 있다.

## 5.1 목과 스텁 구분
- 목은 SUT과 그 협력자 사이의 상호 작용을 검사할 수 있는 테스트 대역이다.
  - 테스트 대역에 또 다른 유형이 있는데 바로 Stub 스텁이다.

### 5.1.1 테스트 대역 유형
- 테스트 대역은 모든 유형의 비운영용 가짜 의존성을 설명하는 포괄적인 용어다.
  - 대신하는 스턴트 대역이라는 개념에서 비롯됐다.
- 크게 목과 스텁의 두 가지 유형으로 나눌 수 있다.

1. 목
   - 목은 외부로 나가는 상호 작용을 모방하고 검사하는 데 도움이 된다.
   - SUT가 상태를 변경하기 위한 의존성을 호출하는 것에 해당한다.
   - 목과 스파이가 있다.
2. 스텁
   - 내부로 들어오는 상호 작용을 모방하는 데 도움이 된다.
   - SUT가 입력 데이터를 얻기 위한 의존성을 호출하는 것에 해당한다.
   - 스텁, 더미, 페이크가 있다.

### 5.1.2 도구로서의 목과 테스트 대역으로서의 목
- Mock 클래스는 도구로서의 목인 데 반해
- 해당 클래스의 인스턴스인 mock은 테스트 대역으로서의 목이다.
- 도구(라이브러리)로서의 목 vs 실제 테스트 대역으로서의 목 인스턴스

### 5.1.3 스텁으로 상호 작용을 검증하지 말라
- 목은 SUT에서 관련 의존성으로 나가는 상호 작용을 모방하고 검사하고
- 스텁은 내부로 들어오는 상호 작용만 모방하고 검사하지 않는다
- 이 두 가지 차이는 스텁과의 상호 작용을 검증하지 말라는 지침에서 비롯된다.
  - SUT에서 스텁으로 호출은 SUT가 생성하는 최종 결과가 아니다.
  - 단순 최종 결과를 산출하기 위한 수단일 뿐이다.
  - 스텁은 SUT가 출력을 생성하도록 입력을 제공한다.

> 스텁과의 상호 작용을 검증하는 것은 취약한 테스트를 야기하는 일반적인 안티 패턴이다.

**테스트에서 거짓 양성을 피하고 리팩터링 내성을 향상시키는 방법은 구현 세부사항이 아니라 최종 결과를 검증하는 것 뿐이다.**

``` java
public void Creating_a_report() {
    var stub = new Mock<IDatabase>();
    stub.Setup(x => x.GetNumberOfUsers()).Returns(10);

    var sut = new Controller(stub.Object);

    Report report = sut.CreateReport();

    Assert.Equal(10, report.NumberOfUsers);
    stub.Verify(
        x => x.GetNumberOfUsers(),
        Times.Once);
        // 스텁으로 상호 작용 검증
}
```

최종 결과가 아닌 사항을 검증하는 이러한 관행을 **과잉 명세**라고 부른다.
- 과잉 명세는 상호 작용을 검사할 때 가장 흔하게 발생한다.
- 스텁과의 상호 작용을 확인하는 것은 쉽게 발견할 수 있는 결함이다.

### 5.1.4 목과 스텁 함께 쓰기
- 때로는 목과 스텁의 특성을 모두 나타내는 테스트 대역을 만들 필요가 있다.
  - 하나의 mock이 준비된 응답을 반환하고 SUT에서 수행한 메서드 호출을 검증한다.
  - 그러나 이는 두 가지의 서로 다른 메서드다.
- 테스트 대역은 목이면서 스텁이지만, 여전히 목이라고 부르지 스텁이라고 부르지는 않는다.
  - 이름을 하나 골라야 하기도 하고, 목이라는 사실이 스텁이라는 사실보다 더 중요하기 때문에 대체로 목이라고 한다.

### 5.1.5 목과 스텁은 명령과 조회에 어떻게 관련돼 있는가?
- 목과 스텁 개념은 명령 조회 분리(CQS, Command QuerySeparation) 원칙과 관련이 있다.
  - 원칙에 따르면 모든 메서드는 명령이거나 조회여야 하며, 이 둘을 혼용해서는 안 된다.
1. 명령은 사이드 이펙트를 일으키고 어떤 값도 반환하지 않는 메서드다.
  - 사이드 이펙트의 예로는 객체 상태 변경, 파일 시스템 내 파일 변경 등이 있다.
2. 조회는 그 반대로, 사이드 이펙트가 없고 값을 반환한다. (조회성)

이 원칙을 따르고자 할 경우, 메서드가 사이드 이펙트를 일으키면 해당 메서드의 반환 타입이 void인지 확인하라. 그리고 메서드가 값을 반환하면 사이드 이펙트가 없어야 한다.
- 물론 사이드 이펙트를 초래하고 값을 반환하는 것이 적절한 메서드는 있기 마련이다.
- 그래도 가능할 때마다 CQS 원칙을 따르는 것이 좋다.
- 명령 대체 테스트 대역은 목이다.
- 조회 대체 테스트 대역은 스텁이다.

> 테스트 코드 작성 중, mocking과 stubbing을 구분하지 않고 모두 stub하듯이 코드를 작성했던 것 같다. 명령에 해당하는 상태 변경 등의 액션 메서드는 null을 반환하긴했는데, stub 코듳럼 작성하고 DoNothing().whenver로 작성했던 것 같다.

## 5.2 식별할 수 있는 동작과 구현 세부 사항
- 취약성을 일으키는 원인을 알아보자.
  - 테스트 취약성은 리팩터링 내성에 해당한다.
  - 테스트가 단위 테스트 영역에 있고 E2E의 범주로 바뀌지 않는 한 리팩터링 내성을 최대한 활용하는 것이 좋다.
- 거짓 양성이 있는 주요 이유는 코드의 구현 세부 사항과 결합돼 있기 때문이다.
- 이런 강결합을 피하는 방법은 코드가 생성하는 최종 결과를 검증하고 구현 세부 사항과 테스트를 가능한 한 떨어뜨리는 것뿐이다.
  - 어떻게?가 아닌 무엇에 중점을 둬야 한다.

### 5.2.1 식별할 수 있는 동작은 공개 API와 다르다
- 코드는 2차원으로 분류 가능하다.
  - 공개 API 또는 비공개 API
  - 식별할 수 있는 동작 또는 구현 세부 사항

코드가 시스템의 식별할 수 있는 동작이려면 다음 중 하나를 해야 한다.
1. 클라이언트가 목표를 달성하는 데 도움이 되는 연산을 노출하라.
   - 연산은 계산을 수행하거나 사이드 이펙트를 초래하거나 둘 다 하는 메서드다.
2. 클라이언트가 목표를 달성하는 데 도움이 되는 상태를 노출하라.
   - 상태는 시스템의 현재 상태다.

식별할 수 있는 동작이라는 것은 클라이언트의 목표에 직접적인 관계가 있어야 한다.
- 이상적으로 시스템의 공개 API는 식별할 수 있는 동작과 일치해야 하며, 모든 구현 세부 사항은 클라이언트 눈에 보이지 않아야 한다.
  - 이러한 시스템이 API 설계가 잘돼 있다고 할 수 있다.
- 그러나 종종 시스템의 공개 API가 식별할 수 있는 동작의 범위를 넘어 구현 세부 사항을 노출하기 시작한다.
  - 이러한 시스템의 구현 세부 사항은 공개 API로 유출된다.

> 해당 클라이언트 (메서드 사용처)에서 메서드의 결과를 이루기 위해 호출하는 step에 직접적으로 연관되어 있는 동작만 노출해라.

### 5.2.2 구현 세부 사항 유출 : 연산의 예

``` java
public class User {
    public string Name {get; set;}

    public string NormalizeName(string name) {
        string result = (name ?? "").Trim();

        if(result.Length > 50)
            return result.SubString(0, 50);

        return result;
    }
}

public class UserController {
    public void RenameUser(int userId, string newName) {
        User user = GetUserFromDatabase(userId);

        string normalizedName = user.NormalizeName(newName);
        user.Name = normalizedName;

        SaveUserToDatabase(user);
    }
}
```

이 메서드의 목표는 사용자 이름을 변경하는 것이다.
- 속성과 메서드가 모두 공개다.
  - UserController가 사용자 이름 변경이라는 목표를 달성할 수 있도록 속성의 setter는 노출한다.
  - NormalizeName은 목표와 직결되지 않는다.

``` java
get => _name
set => _name = NormalizeName(value);

private string NormalizeName(string name) ...
```

- 식별할 수 잇는 동작(Name 속성)만 공개돼 있고, 구현 세부 사항은 private으로 비공개 API가 되었다.

단일한 목표를 달성하고자 클래스에서 호출해야 하는 연산의 수가 1보다 크면 해당 클래스에서 구현 세부 사항을 유출할 가능성이 있다. 이상적으로는 단일 연산으로 개별 목표를 달성해야 한다.
- 예제에서도 리팩터링 후 연산 수가 1로 감소했다.

### 5.2.3 잘 설계된 API와 캡슐화
- 잘 설계된 API를 유지 보수하는 것은 캡슐화 개념과 관련이 있다.
- 캡슐화는 불변성 위반이라고도 하는 모순을 방지하는 조치이다.
  - 이전 예제 User 클래스에는 사용자 이름이 50자를 초과하면 안 된다는 불변성이 있었다.
  - 불변성 위반으로 구현 세부 사항을 노출하게 된다.

장기적으로 코드 베이스 유지 보수에서는 캡슐화가 중요하다. 복잡도 때문이다.
- 코드 복잡도는 소프트웨어 개발에서 가장 큰 어려움 중 하나다.
- 복잡해질수록 작업하기가 더 어려워지고, 개발 속도가 느려지고, 버그 수가 증가하게 된다.

계속해서 증가하는 코드 복잡도에 대처할 수 있는 방법은 실질적으로 캡슐화 말고는 없다. 코드 API가 해당 코드로 할 수 잇는 것과 할 수 없는 것을 알려주지 않으면 코드가 변경됐을 때 모순이 생기지 않도록 많은 정보를 염두에 둬야 한다.
- 이는 정신적 부담을 증대한다.
- 최대한 부담을 덜어라.
- 실수할 가능성을 최대한 없애라.
- 이렇게 하는 데 가장 좋은 방법은 캡슐화를 올바르게 유지해 코드베이스에서 잘못할 수 있는 옵션조차 제공하지 않도록 하는 것이다.

> 캡슐화는 궁극적으로 단위 테스트와 동일한 목표를 달성한다.
> - 즉, 소프트웨어 프로젝트의 지속적인 성장을 가능하게 하는 것이다.
> - [묻지 말고 말하라](https://martinfowler.com/bliki/TellDontAsk.html)라는 유사한 원칙이 있다.
>   - 데이터와 연산 기능을 결합하는 것을 통해 구현 세부 사항을 숨기는 의미가 있다.
>   - 데이터를 묻고, 외부에서 연산 -> 연산을 숨긴 public method를 통해 tell

1. 구현 세부 사항을 숨기면 클라이언트 시야에서 클래스 내부를 가릴 수 있기 때문에 내부를 손상시킬 위험이 적다.
2. 데이터와 연산을 결합하면 해당 연산이 클래스의 불변성을 위반하지 않도록 할 수 있다.
   - 맘대로 불변을 해칠 수 없고 public API만 사용해야 하기 때문에 규칙을 지킬 수 있다.

### 5.2.4 구현 세부 사항 유출 : 상태의 예
- 좋은 단위 테스트와 잘 설계된 API 사이에는 본질적인 관계가 있다.
- **모든 구현 세부 사항을 비공개로 하면 테스트가 식별할 수 있는 동작을 검증하는 것 외에는 다른 선택지가 없기 때문에**, 리팩터링 내성도 자동으로 좋아진다.
- 또한, 연산과 상태를 최소한으로 노출해야 한다.
  - 목표를 달성하는 데 직접적으로 도움이 되는 코드만 공개해야 한다.

|구분|식별할 수 있는 동작 | 구현 세부 사항 |
|--|--|--|
|공개|좋음|나쁨|
|비공개|해당 없음|좋음|