# Architecture - 부분적 경계, 계층과 경계

> Clean Architecture 5부 24 ~ 25장

## 부분적 경계

- 아키텍처 경계를 완벽하게 만드는 데는 비용이 많이 든다.
    - 양방향의 다형적 boundary interface
    - Input과 Output을 위한 데이터 구조
    - 두 영역을 독립적으로 컴파일하고 배포할 수 있는 컴포넌트로 격리하기 위한 의존성 관리
- 이 비용이 너무 크다고 생각할 수도 있지만, 한편으로는 나중에 필요할 수도 있으니 경계를 나누기를 원할 수도 있다.
    - 'YAGNI(You Aren't Going to Need It)' 원칙을 위배한다고 주장할 수도 있다.
- 이럴 때, 부분적 경계를 구현해 볼 수 있다.

### 마지막 단계를 건너뛰기

- 독립적으로 컴파일할 수 있는 여러 개의 컴포넌트로 분리하는 작업을 생략한다.
- 즉, 모든 것들이 단일 컴포넌트로 관리된다.
- 장점
    - 여러 컴포넌트를 관리하지 않아도 된다.
    - 버전 번호, 배포 관리 부담도 없다.
- 단점
    - 완벽한 경계를 만들 때와 비용에 큰 차이가 없다.

### 일차원 경계

- 완벽한 경계는 양방향으로 격리된 상태를 유지하므로, 양방향 boundary interface를 사용한다.
- Strategy pattern을 활용하여 부분적 경계를 구현하고, 완벽한 경계로 확장하기 위한 공간을 확보할 수 있다.
    <p><img src="img/architecture-partial-boundary-strategy.png" width="350"></p>
- `ServiceBoundary`를 사용하여 `Client`를 `Service Impl`로부터 격리시키기 위한 의존성 역전이 이미 적용되어 있다.
- 하지만, `Service Impl`에서 `Client`로 향하는 점선 의존성을 막을 방법은 없다.

### 퍼사드

- Facade pattern을 활용하여 의존성 역전까지도 희생하면서 더 단순한 경계를 구현할 수 있다.
    <p><img src="img/architecture-partial-boundary-facade.png" width="350"></p>
- `Facade` 클래스로 경계가 간단하게 정의되며, `Client`는 `Service` 클래스들에 직접 접근할 수 없다.
- 하지만, `Client`가 모든 `Service` 클래스들에 대해 **추이 종속성**을 갖게 된다.
    - `Service`들 중 하나가 변경되면 `Client`도 다시 컴파일 해야 한다.

### 결론

- 부분적 경계를 구현하는 세 가지 방법
    - 컴포넌트를 분리하지 않고 단일 컴포넌트를 유지하기
    - Strategy pattern을 활용해서 의존성을 역전시켜 분리하기
    - Facade pattern을 활용해서 단순한 경계만 유지하기
- 각 방법의 장단점을 비교하여 상황에 따라 적절한 방법을 선택해서 사용할 수 있어야 한다.
    - 경계가 언제, 어디에 존재해야 할지
    - 완벽한 경계를 만들지
    - 부분적 경계를 어떤 방식으로 구현할지

## 계층과 경계