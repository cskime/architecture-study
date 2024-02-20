# Architecture - 테스트 경계

> Clean Architecture 5부 28장

- 테스트도 시스템의 일부로서 아키텍처에 관여한다.
- 테스트가 세부사항에 의존하거나(e.g. UI 동작에 따라 수행되는 테스트) 애플리케이션 구조에 결합되면,
    - 테스트를 변경하기 어려워지고,
    - 상용 코드를 추상적인 상태로 유연하게 유지하기 어려워진다.
- 따라서, 아래와 같은 문제가 생긴다.
    - 테스트를 너무 많이 변경하지 않기 위해 상용 코드를 유연하게 수정하지 못하거나,
    - 상용 코드를 유연하게 유지하기 위해 테스트를 너무 많이 수정해야 한다. -> 테스트를 포기하게 만든다.
- 즉, **테스트와 애플리케이션 사이의 구조적 결합을 분리하고, 세부사항에 의존하지 않는 구조로 설계해야 한다.**

## 시스템 컴포넌트로서의 테스트

- 테스트 코드는 항상 대상이 되는 코드를 향해 의존하며, 반대 방향으로 의존하지 않는다. (**의존성 규칙을 따른다.**)
- 시스템 컴포넌트가 테스트 코드를 의존하는 일은 없으며, 테스트가 시스템 컴포넌트를 향해 항상 안쪽으로 의존한다.
- 테스트 코드는 독립적으로 배포 가능한 상태로 존재한다. (테스트 코드에 의존하는 컴포넌트가 없으므로. **고립되어 있기 때문**)
- 테스트는 운영에 꼭 필요하지는 않지만 **개발을 지원하는 역할**을 한다.
- 하지만, 테스트는 모든 시스템 컴포넌트가 반드시 지켜야 하는 모델을 표현해 준다.

## 테스트를 고려한 설계

- 테스트도 시스템의 설계와 잘 통합되도록 시스템 설계에 포함되어야 한다.
- 설계에서 테스트를 고려하지 않으면, **테스트는 깨지기 쉬워지고 시스템은 변경하기 어려워진다.**
- 시스템에 강하게 결합된 테스트라면, 시스템 컴포넌트에서 생긴 사소한 변경도 이와 결합된 수 많은 테스트를 망가뜨릴 수 있다.

### 꺠지기 쉬운 테스트 문제

- 시스템의 공통 컴포넌트가 변경될 때, 수백 수천 개의 테스트가 망가질 수도 있는 문제
- 가령, UI를 사용해서 업무 규칙을 테스트한다면, UI와 관련된 변경 때문에 업무 규칙에 대한 테스트가 쉽게 깨진다.
- 즉, **변동성이 있는 것에 의존하지 않고 업무 규칙을 테스트 할 수 있어야 한다.**

## 테스트 API

- 깨지기 쉬운 테스트 문제는 테스트가 모든 업무 규칙을 검증하는데 사용할 수 있도록 특화된 API를 만들어서 해결한다.
- 보안 제약사항을 무시하고, DB같은 비용이 많이 드는 자원은 건너 뛰고, 시스템을 가능한 특정 상태로 강제해서 테스트할 수 있다.
- 테스트 API는 테스트를 애플리케이션으로부터 분리할 목적으로 사용한다.

### 구조적 결합으로부터 분리

- 테스트 구조가 애플리케이션 구조와 결합되지 않도록 분리한다.
- 모든 상용 클래스 또는 메서드에 테스트 클래스가 각각 존재하는 경우, 애플리케이션 구조에 강하게 결합된다.
- 이런 구조는 상용 클래스가 변경되면 연관된 다수의 테스트가 변경되어야 하므로, **테스트가 깨지기 쉬워지고 상용 코드를 변경하기 어려워진다.**
- 테스트 API를 사용하면, 애플리케이션 구조를 테스트로부터 숨길 수 있다.
- 시간이 지날 수록 테스트는 더 구체적인 형태로 변화하고, 상용 코드는 더 추상적이고 범용적인 형태로 변화한다.
- 따라서, 구조적으로 강하게 결합되면 테스트를 더 구체적으로 작성하고, 상용 코드를 더 추상적으로 만드는 것을 어렵게 만든다.

### 보안으로부터 분리

- 테스트 API 자체와 테스트 API 중 위험한 부분의 구현부는 독립적으로 배포할 수 있는 컴포넌트로 분리한다.