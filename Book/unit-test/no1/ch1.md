# 1장 단위 테스트의 목표
> 단위 테스트의 목표를 정의하고 좋은 테스트와 좋지 않은 테스트를 구별하는 방법을 개략적으로 살펴본다
- 단위 테스트에 시간을 투자할 때는 항상 최대한 이득을 얻도록 노려갷야 하며, 테스트에 드는 노력을 가능한 줄이고, 그에 따르는 이득을 최대화해야 한다. 물론 쉽지 않다.
- 일부 테스트는 훌륭한 결과를 만들고 소프트웨어 품질을 지키는데 도움이 된다. 그와 반대의 경우엔 그다지 도움이 되지 않고 자주 고장나며 유지 보수가 많이 필요하다.

## 1.1 단위 테스트 현황
- 그냥 쓰고 버리는 프로젝트가 아니라면, 단위 테스트는 늘 적용해야 한다.
- 많은 프로젝트에 단위 테스트가 적용되어 있지만, 개발자가 원하는 결과를 얻지 못하는 경우가 많다. 
  - 이는 제대로 동작하지 않는 단위 테스트의 결과이다.
  - 어떤 것이 단위 테스트를 더 좋게 만들 수 있을까?

## 1.2 단위 테스트의 목표
- 단위 테스트를 하기 위해 도움이 되는 목표는 무엇이 있을까?
  - 더 나은 설계로 이어지는 것?
    - 코드베이스에 대해 단위 테스트 작성이 필요하면 일반적으로 더 나은 설계로 이어진다.
    - 그러나 주 목표는 아니다. 사이드 이펙트일 뿐이다.
      - 코드를 단위 테스트하기 어렵다면 코드 개선이 반드시 필요하다는 것을 의미하고 보통 강결합되어 코드가 서로 충분히 분리되지 않아 테스트하기 어려워진다.
- **단위 테스트의 주 목표는 소프트웨어 프로젝트의 지속 가능한 성장을 가능하게 하는 것**이다
  - 테스트가 없는 일반 프로젝트는 빨리 시작할 수 있다.
    - 아직 잘못된 아키텍처 결정이 없고, 걱정할 만한 코드가 있지도 않다.
    - 이후 시간이 지나면서 점점 더 많은 시간을 들여야 처음과 같은 속도가 나올 수 있다.
  - 개발 속도가 빠르게 감소하는 현상을 소프트웨어 엔트로피라고도 한다.
    - 품질을 떨어뜨리는 코드 형태로 나타나고 변경 시 무질서도는 증가한다.
    - 지속적인 정리와 리팩터링 등의 적절한 관리를 하지 않고 방치하면 시스템이 점점 더 복잡해지고 무질서해진다.
    - 버그 수정 시 또 다른 버그가 발생하고, 결국 코드베이스를 신뢰할 수 없게 된다.
    - 최악의 경우 안정되게 복구하는 것은 어렵다.
  - 테스트를 통해 이러한 경향을 막을 수 있다. 안정망 역할을 하고 버그에 대한 보험을 제공한다.
- 한 가지 단점은, 테스트는 초반에 노력이 필요하다. 하지만 프로젝트 후반에도 잘 성장할 수 있도록 하므로 장기적으로 보면 그 비용을 메꿀 수 있다.
- **지속성과 확장성이 핵심이며, 이를 통해 장기적으로 개발 속도를 유지할 수 있다.**

### 1.2.1 좋은 테스트와 좋지 않은 테스트를 가르는 요인
- 잘못 작성한 테스트는 초반에 도움을 줄 순 있지만, 결국 나중에 테스트의 부재와 같이 개발 속도가 느려질 수 있다.
- 테스트의 가치와 유지 비용을 모두 고려해야 한다.
  - 기반 코드를 리팩터링할 때 테스트도 리팩터링하라.
  - 각 코드 변경 시 테스트를 실행하라.
  - 테스트가 잘못된 경고를 발생시킬 경우 처리하라.
  - 기반 코드가 어떻게 동작하는지 이해하려고 할 때는 테스트를 읽는 데 시간을 투자하라.
- 지속 가능한 프로젝트 성장을 위해서는 고품질 테스트에만 집중해야 한다.

> 제품 코드 대 테스트 코드
> 테스트가 많으면 좋다고 생각할 수 있지만, 코드는 자산이 아니라 책임이다. 많아질수록 결국 유지비용은 증가한다. 따라서 가능한 적은 코드로 문제를 해결하는 것이 좋다.
> 테스트 역시 코드다. 어플리케이션의 정확성을 보장하는 것을 목표로 하는 코드 베이스의 일부로 봐야하고, 단위 테스트 역시 버그에 취약하고 유지 보수가 필요하다.

## 1.3 테스트 스위트 품질 측정을 위한 커버리지 지표
> 가장 널리 사용되는 두 가지 커버리지 지표 (코드와 분기 커버리지)를 어떻게 계산하고 사용하는지, 관련된 문제점도 알아본다.

- 특정 커버리지 숫자를 목표로 하는 것은 해롭고, 테스트 스위트 품질을 결정하는 데 커버리지 지표에 의존할 수 없는 이유를 알아본다.

커버리지 지표는 테스트 스위트가 소스 코드를 얼마나 실행하는지를 백분율로 나타낸다.

- 일반적으로는 커버리지 숫자가 높을수록 더 좋지만. 그렇게 간단하지는 않다.
  - 코드 커버리지가 너무 적으면 테스트가 충분치 않다는 좋은 증거다.
  - 그러나 커버리지가 100%라고 해서 반드시 양질의 테스트 스위트라고 보장하지는 않는다.
    - 중요하지 않은 테스트가 많아도 아무런 도움을 주지 않는다.

### 1.3.1 코드 커버리지 지표에 대한 이해
- 코드 커버리지 = 실행 코드 라인 수 / 전체 라인 수

```
{
    if (input.Length > 5 )
        return true;
    return false;
}

bool result = isStringLong("abc")
Assert.equal(false, result);
```

위 테스트의 경우, 테스트 라인 수는 5줄이지만 실제로는 return true를 제외하고 실행하는 라인수는 4줄로, 코드 커버리지는 80%이다.

```
{
    return input.Length > 5;
}
```
로 변경하면 커버리지는 100%가 된다.

- 단순히 메서드 내 코드만 변경했을 뿐, 테스트 스위트를 개선했다고 볼 수 없다.
- 코드를 더 작게 해도 테스트 스위트의 가치나 기반 코드베이스의 유지보수성이 변경되지는 않는다.

### 1.3.2 분기 커버리지 지표에 대한 이해
- 분기 커버리지 = 통과 분기 / 전체 분기 수
- 분기 커버리지는 코드 커버리지보다 더 정확한 결과를 제공한다. 
- 원시 코드 라인 수를 사용하는 대신 if문과 switch문과 같은 제어 구조에 중점을 둔다.

위 테스트 케이스에서 true, false 2개의 분기 중 1개만 적용되는 테스트이므로 분기 커버리지는 50%이다.

### 1.3.3 커버리지 지표에 관한 문제점
- 테스트 스위트의 품질을 결정하는 데 어떤 커버리지 지표도 의존할 수 없는 이유는 다음과 같다.
1. 테스트 대상 시스템의 모든 가능한 결과를 검증한다고 보장할 수 없다.
- 결과가 여러 개 있을 수 있고, 반드시 적절한 검증이 있어야 한다.
- 일부만 실행되도 커버리지는 동일할 수 있고, 심지어 검증이 없는 테스트가 있을 수 있는데 항상 통과한다. => 전혀 쓸모가 없다.

2. 외부 라이브러리의 코드 경로를 고려할 수 있는 커버리지 지표는 없다.
- 외부 라이브러리 코드를 사용하는 경우, 해당 라이브러리를 사용 시 발생할 수 있는 코드 경로에 대해 테스트를 하지 않을 수 있다.
```
int Parse(string input) {
    return int.Parse(input);
}

int result = Parse("5");
Assert.equal(5, result);

위 테스트의 분기 커버리지 지표는 100%이다.
하지만 입력 매개변수를 변경 시 다른 결과로 이어질 수 있는 분기가 많다.
null, 빈 문자열, 정수가 아닌 값 등 수많은 예외 상황이 있다.
```
- 이는 외부 라이브러리의 코드 경로를 고려해야 한다는 것이 아닌, 커버리지 지표만으로 단위 테스트가 좋은지 나쁜지 판단할 수 없다는 것이다.

### 1.3.4 특정 커버리지 숫자를 목표로 하기
- 테스트의 품질을 결정하기에 커버리지 지표만으로는 충분치 않다.
- 병원에 있는 환자의 열이 높다는 것 자체는 유용한 관찰이다.
  - 하지만 적절한 체온만을 목표로 해서 에어컨을 설치하는 등의 접근은 의미가 없다.
- 마찬가지로 특정 커버리지 숫자만을 목표로 하는 것은 단위 테스트의 목표와 반대된다.
- 커버리지 지표가 낮으면 테스트가 부족하다는 지표는 될 순 있지만, 높다고해서 테스트의 품질이 좋다는 의미는 아니다.

## 1.4 무엇이 성공적인 테스트 스위트를 만드는가?
- 믿을만한 방법은 스위트 내 각 테스트를 하나씩 따로 평가하는 것 뿐이다.
- 요점은 테스트 스위트가 얼마나 좋은지 자동으로 확인할 수 없다는 것이다.
- 성공적인 테스트 스위트는 다음과 같은 특성을 갖고 있다.
  - 개발 주기에 통합돼 있다.
  - 코드베이스에서 가장 중요한 부분만을 대상으로 한다.
  - 최소한의 유지비로 최대의 가치를 끌어낸다.

### 1.4.1 개발 주기에 통합돼 있음
- 자동화된 테스트를 할 수 있는 방법은 끊임없이 하는 것뿐이다.
- 모든 테스트는 개발 주기에 통합돼야 하고, 코드가 변경될 때마다 아무리 작은 것이라도 실행해야 한다.

### 1.4.2 코드베이스에서 가장 중요한 부분만을 대상으로 함
- 시스템의 가장 중요한 부분에 단위 테스트 노력을 기울이고, 다른 부분은 간략하게 또는 간접적으로 검증하는 것이 좋다.
  - 가장 중요한 부분은 비즈니스 로직 (도메인 모델)이 있는 부분이다.
  - 비즈니스 로직 테스트가 시간 투자 대비 최고의 수익을 낼 수 있다.

- 통합 테스트와 같이 일부 테스트는 도메인을 넘어 시스템이 전체적으로 어떻게 동작하는지 확인할 수 있어서 괜찮지만, 초점은 도메인 모델에 머물러 있어야 한다.
- 이 지침을 따르기 위해선 도메인 모델을 코드베이스 중 중요하지 않은 부분과 분리해야 한다.

### 1.4.3 최소 유지비로 최대 가치를 끌어냄
1. 가치 있는 테스트 식별하기
   - 식별하기 위해선 기준틀이 필요하다.
2. 가치 있는 테스트 작성하기
   - 단위 테스트와 기반 코드는 서로 얽혀 있으므로 코드 베이스에 노력을 많이 기울어야만 가치 있는 테스트를 만들 수 있다.

## 1.5 이 책을 통해 배우는 것
- 테스트 스위트 내의 모든 테스트를 분석하는 데 사용할 수 있는 기준틀을 설명한다.
- 그러고 나서 새로운 관점에서 많은 테스트를 볼 수 있으며, 어떤 테스트가 프로젝트에 기여하는지, 어떤 테스트를 리팩터링해야 하거나 제거해야 하는지 알 수 있게 된다.
- 단위 테스트 기법과 좋은 사례를 통해 경험을 쌓을 수 있다.

## 요약
- 코드는 점점 나빠지고 무질서도가 증가한다.
  - 테스트를 통해 무질서도가 높은 상황에서도 개발을 이어나갈 수 있다.
- 단위 테스트를 작성하는 것도 중요하고, 좋은 테스트를 작성하는 것도 중요하다.
- 단위 테스트의 목표는 소프트웨어 프로젝트가 지속적으로 성장하게 하는 것이다.
  - 좋은 테스트는 개발 속도를 지키면서 침체 단계에 빠지지 않게 한다.
  - 리팩터링이나 새로운 기능 추가하는 것이 더 쉬워진다.
- 가치 있는 테스트만 남기고 모두 제거하라. 코드는 자산이 아니라 부채다.
- 단위 테스트 지표는 좋은 부정 지표지만 나쁜 긍정 지표다.
  - 낮다는 것은 문제의 징후지만, 높다고해서 품질이 높은 것은 아니다.
- 분기 커버리지를 통해 테스트가 충분한지는 여전히 알 수 없다.
- 특정 커버리지 숫자에 집착하면 테스트 동기 부여가 잘못된 것이다.
- 단위 테스트 목표를 달성하기 위해선 좋은 / 좋지 않은 테스트를 구별하고, 테스트를 작성해서 더 가치 있게 만든다.