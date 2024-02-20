# Component 결합

> Clean Architecture 4부 14장

1. 의존성 비순환 원칙(ADP) : 컴포넌트 의존성 그래프에서 DIP 또는 추상 컴포넌트를 사용하여 '순환' 제거
2. 안정된 의존성 원칙(SDP) : 항상 불안정한 컴포넌트가 안정된 컴포넌트에 의존해야 한다. (의존성 그래프에서 아래로 갈 수록 I 지표의 값이 작아진다.)
3. 안정된 추상화 원칙(SAP) : 컴포넌트가 최고로 안정되면서도 변경에 유연하게 대응할 수 있도록(OCP) 추상 클래스나 인터페이스를 포함한다.

## 의존성 비순환 원칙 (Acyclic Dependencies Principle, ADP)

> 컴포넌트 의존성 그래프에 순환(cycle)이 있어서는 안된다.

- 숙취 증후군 : 많은 개발자가 동일한 소스 파일을 수정할 때 예상치 못하게 내가 의존하는 코드를 다른 개발자가 수정하여 정상적으로 동작하지 못하게 되는 문제
- 이를 해결하기 위한 방법
  - 주 단위 빌드 (weekly build)
  - 의존성 비순환 원칙 (ADP)

### 주단위 빌드

- 4일은 개발하고 5일째에 작업물을 모두 합치는 방식
- 4일동안 개발에 집중할 수 있지만, 5일 째에 그만큼 큰 대가를 치러야 한다.
- 프로젝트가 커지면서 통합에 드는 시간이 점점 느려지게 되고 결국 전체 개발 일정이 지연된다.

### 의존성 비순환 원칙

- 개발 환경을 독립적으로 릴리스 가능한 컴포넌트 단위로 분리하고 릴리스 번호를 부여하여 수정이 발생하더라도 다른 개발자들이 안정된 버전을 사용하는 것을 보장한다.
- 즉, 어떤 팀도 다른 팀에 의해 좌우되지 않을 수 있다. 수정할 시기를 스스로 결정할 수 있고, 작은 단위로 점진적으로 통합할 수 있다.
- 이 방법이 제대로 동작하려면 **'컴포넌트 간 의존성 구조'에 "순환"이 존재하면 안된다.**

컴포넌트 구조는 아래 그림과 같이 **비순환 방향 그래프(Directed Acyclic Graph)** 가 되어야 한다.

<p><img src="img/component-adp.png" width="450"></p>

- 비순환 방향 그래프에서는 어떤 컴포넌트에서 시작하더라도 의존성 관계를 따라가면서 최초 컴포넌트로 되돌아 갈 수 없다.
- 순환이 없기 때문에 어떤 컴포넌트가 변경되어 새로 릴리스되면 그 컴포넌트에 의존하고 있는 컴포넌트들만 영향을 받는다.
  - 영향을 받는 컴포넌트는 의존성 방향을 반대로 따라가면 알 수 있다. 'A->B' 관계에서 A는 B에 의존하므로 B가 변경되면 A가 영향을 받는다.
- 'Presenters' 컴포넌트
  - 'Presenters'가 변경되면 'Main'과 'Views' 컴포넌트만 영향을 받는다.
  - 'Presenters'를 테스트하려면 'Interactors'와 'Entities' 컴포넌트만 사용하면 된다. 즉, **적은 노력으로 테스트를 구성할 수 있고 고려해야 하는 변수도 적다.**
- 'Main' 컴포넌트가 변경되더라도 나머지 컴포넌트는 영향을 받지 않는다. 위 그림에서 'Main' 컴포넌트를 알고 있는 다른 컴포넌트는 없다.
- 그래프를 통해 **컴포넌트 간 의존성**을 파악하고 있으면 **시스템을 빌드하는 방법**을 알 수 있다.
  - 비순환 컴포넌트 구조에서 시스템 전체를 빌드할 때는 '**상향식**'으로 진행된다.
  - 의존성 그래프에서 아무 것도 의존하지 않는 것부터 시작해서 그 컴포넌트에 의존하는 것들로 확장해 나간다.
  - Entities -> Database, Interactors -> Presenters -> Views -> Controllers -> Authorizer -> Main

### 순환이 컴포넌트 의존성 그래프에 미치는 영향

요구사항이 변경되어 'Entities'에 포함된 클래스(`User`) 하나가 'Authorizer'에 포함된 클래스(`Permission`)를 사용하도록 변경해야 한다면 그래프에서는 아래와 같이 '**순환**'이 생긴다.

<p><img src="img/component-adp-cycle.png" width="450"></p>

- 순환이 생기면 cycle에 연결된 컴포넌트들이 사실상 하나의 거대한 컴포넌트가 된다.
  - 'Database'를 개발하려면 'Entities'가 필요하다.
  - 'Entities'는 'Authorizer'에서 'Interactors'를 거쳐 '순환'되므로, 'Database'를 개발하려면 'Authorizer'와 'Interactors'까지도 알아야 한다.
- 순환에 의해 컴포넌트의 범위가 커지면서 **어떤 컴포넌트가 변경될 때 영향을 받는 컴포넌트가 많아지고**, '숙취 증후군'이 다시 발생한다.
  - 직접적인 연관이 없는 컴포넌트까지 영향을 받으므로 가능성이 더 높아진다.
  - 순환 관계에 놓인 컴포넌트의 개발자들은 서로에게 얽매이게 되어 모두 동일한 릴리스를 사용해야만 한다.
- 즉, '순환'은 **컴포넌트를 분리하기 어렵게** 만들고, 변경이 발생할 때 영향을 받는 컴포넌트의 수가 많아지므로 **유지보수하기 어렵게** 만든다.

### 순환을 끊는 방법

1. **DIP**를 사용해서 'Entities'에서 'Authorizer'로 가는 의존성을 역전시킨다. (클래스 수준에서의 해결)
    <p><img src="img/component-adp-remove-cycle-dip.png" width="400"></p>
2. 순환을 끊는 새로운 컴포넌트를 추가한다. (컴포넌트 수준에서 해결)
    <p><img src="img/component-adp-remove-cycle-new.png" width="400"></p>

두 번째 방법은 컴포넌트의 구조 자체가 변경된다. 즉, 컴포넌트 의존성 구조는 계속 변화하므로 개발이 진행되는 중에도 순환이 발생하지는 않는지 항상 관찰해야 한다. **순환이 발생하면 어떤 방식으로든 끊어야 한다.**

### 하향식 설계 (Top-down 설계)

- **컴포넌트 구조는 하향식으로 설계될 수 없다.**
- 컴포넌트는 가장 먼저 설계할 수 없고, 시스템이 성장하고 변경될 때 함꼐 진화한다.
- 컴포넌트 의존성 다이어그램은 '**빌드 가능성(buildability)**'과 '**유지보수성(maintainability)**'을 보여주는 것으로, 기능을 기술하는 것과는 거의 관련이 없다.
- 즉, 개발 초기에는 빌드하거나 유지보수할 소프트웨어가 없으므로 컴포넌트 구조를 설계할 수 없다.

### 개발이 진행됨에 따라 의존성 설계를 하게 되는 과정

1. 개발 초기에는 의존성 설계를 고려하지 않고 개발한다.
2. 개발이 점차 진행되면서 '숙취 증후군'을 겪지 않고 개발하기 위해 '**의존성 관리**'에 대한 요구가 늘어난다.
3. 변경되는 범위를 최소화하기 위해 **SRP**와 **OCP**를 적용하여 '**함께 변경되는 클래스는 같은 위치에 배치**'하도록 설계한다.
    - 의존성 구조를 사용하여 "**변동성을 격리**"한다.
    - '**자주 변경되는 컴포넌트로부터 안정적이며 가치가 높은 컴포넌트를 보호**'한다.
    - 안정적인 컴포넌트 : 쉽게 바뀌지 않는 컴포넌트
4. 개발이 진행되면서 어떤 것들은 '재사용' 가능한 구조로 만들기 위해 **CRP**를 적용하게 된다.
5. 의존성 구조를 발전시켜 나가면서 순환이 발생하면 **ADP**를 적용해서 순환을 없앤다.

## 안정된 의존성 원칙 (Stable Dependencies Principle, SDP)

> 더 안정된(잘 변경되지 않는) 컴포넌트 쪽으로 의존하라.

- 설계를 유지하다 보면 변경은 불가피하다. 어떤 컴포넌트를 변경해야 한다면, 다른 컴포넌트에는 영향을 주지 않게 만들 수 있다.
- **쉽게 변경되지 않는 컴포넌트가 변동이 예상되는 컴포넌트에 의존하게 만들면 안된다.**
  - 변경되지 않는 컴포넌트 A가 쉽게 변경될 수 있는 컴포넌트 B에 의존할 때, 
  - B가 변경하기 쉽다면 A도 결국 변경해야 하는데, A는 변경되지 않아야 하므로 문제가 생긴다.
  - 결국 변동성이 큰 컴포넌트도 변경이 어려워진다.
- 내가 변경하기 쉬운 모듈을 만들더라도, 다른 누군가가 내가 만든 모듈에 의존하면 내 모듈이 변경하기 어려워 질 수도 있다.
- **안정된 의존성 원칙**을 준수하면 **변경하기 어려운 모듈이 변경하기 쉬운 모듈에 의존하지 않도록** 만들 수 있다.

### 안정성(Stability) 지표

- 안정성 : 상태를 쉽게 변경할 수 있는 정도
  - 쉽게 변경할 수 없을 수록 안정성이 높다.
  - 쉽게 변경할 수 있을 수록 불안정하다.
- 안정적인 컴포넌트 : 쉽게 변경하기 어려운 컴포넌트
- 컴포넌트를 변경하기 어렵게 만드는 요인
  - 수많은 다른 컴포넌트가 어떤 컴포넌트에 의존하면, 해당 컴포넌트는 안정적이다.
  - 즉, **컴포넌트 안쪽으로 들어오는 의존성이 많아질수록** 안정적이다. == 변경하기 어려워진다.

<p><img src="img/component-sdp-independance.png" width="400"></p>

- 위 그림에서 컴포넌트 X에 의존하는 컴포넌트는 3개. (X가 '책임지는' 컴포넌트가 3개)
- 컴포넌트 X는 다른 컴포넌트에 의존하지 않다.
- 컴포넌트 X는 '**독립적이다**'.

<p><img src="img/component-sdp-dependance.png" width="400"></p>

- 위 그림에서 컴포넌트 Y가 의존하는 컴포넌트는 3개.
- 컴포넌트 Y는 다른 컴포넌트가 의존하지 않는다. (컴포넌트 Y는 책임성이 없다.)
- 컴포넌트 Y는 '**의존적이다**'.

즉, 컴포넌트의 안정성은 **컴포넌트로 들어오고 나가는 의존성의 개수**로 측정해 볼 수 있다.

- Fan-in : 안으로 들어오는 의존성. 컴포넌트 '내부의 클래스에 의존'하는 '컴포넌트 외부의 클래스' 개수
- Fan-out : 밖으로 나가는 의존성. 컴포넌트 '외부의 클래스에 의존'하는 '컴포넌트 내부의 클래스' 개수
- 불안정성 `I = Fan-out / (Fan-in + Fan-out)`
  - `I`는 0 ~ 1 사이의 값
  - `I = 0` : 최고로 안정된 컴포넌트
  - `I = 1` : 최고로 불안정한 컴포넌트
- 아래 그림에서 `Cc`의 안정성을 계산한다면,
  <p><img src="img/component-stability.png" width="400"></p>

### 의존성 지표의 의미

- I가 1이면,
  - 어떤 컴포넌트도 해당 컴포넌트에 의존하지 않지만, (Fan-in = 0)
  - 해당 컴포넌트는 다른 컴포넌트에 의존한다. (Fan-out > 0)
  - 즉, 이 컴포넌트는 **책임성이 없으며 의존적이다.**
  - 자신에게 의존하는 컴포넌트가 없기 때문에 변경하지 말아야 할 이유가 없다.
  - 반면에, 다른 컴포넌트에는 의존하므로 언젠가 변경할 이유가 생길 것이다.
  - 즉, **I가 1이면 가장 불안정한 상태로 언제든 변경될 여지가 있다.**
- I가 0이면,
  - 해당 컴포넌트에 의존하는 다른 컴포넌트가 있지만, (Fan-in > 0)
  - 해당 컴포넌트 자체는 다른 컴포넌트에 의존하지 않는다. (Fan-out = 0)
  - 즉, 이 컴포넌트는 **다른 컴포넌트를 책임지며 독립적이다.**
  - 자신에게 의존하는 컴포넌트가 있으므로 해당 컴포넌트는 변경하기 어렵다.
  - 반면에, 다른 컴포넌트에는 의존하지 않으므로 해당 컴포넌트를 변경하도록 강제하는 의존성은 없다.
  - 즉, **I가 0이면 가장 안정된 상태로 변경되지 않는다.**
- SDP에서 '**컴포넌트의 I 지표는 그 컴포넌트가 의존하는 다른 컴포넌트들의 I보다 커야 한다**'고 말한다.
- 즉, **컴포넌트의 의존성 방향은 I 지표가 작은 쪽으로 향해야 한다.**
  - 의존성 그래프의 끝으로 갈 수록 다른 컴포넌트에 의존하는 개수가 줄어든다.
  - `Fan-out` 감소 -> I 감소 -> 안정된 컴포넌트
  - 즉, **불안정한(쉽게 변경되는) 컴포넌트가 안정된(쉽게 변경되지 않는) 컴포넌트에 의존해야 한다.**

### 모든 컴포넌트가 안정되어야 하는 것은 아니다

- 모든 컴포넌트가 최고로 안정된 시스템은 변경이 불가능하므로 쓸모가 없다.
- 컴포넌트 구조에서는 **불안정한 컴포넌트와 안정된 컴포넌트가 균형을 이룬다.**

아래 다이어그램은 SDP를 준수하는 컴포넌트 구조를 보여준다.

<p><img src="img/component-sdp-diagram.png" width="400"></p>

- 관례적으로 불안정한(instable) 컴포넌트를 위에 둔다.
- 의존성 화살표가 위로 향하면 SDP에 위배된다. (안정된 컴포넌트가 불안정한 컴포넌트에 의존한다는 뜻이므로)

만약 Stable 컴포넌트 개발자가 불안정하기를 원하는 Flexible 컴포넌트를 의존하게 된다면 SDP를 위배하게 된다.

<p><img src="img/component-sdp-diagram-violation.png" width="400"></p>

- 불안정하길 원하는 Flexible 컴포넌트는 Stable 컴포넌트보다는 I 지표가 클 것이다. (다른 컴포넌트에 더 의존할 것)
- I 지표가 작은 컴포넌트가 I 지표가 더 큰 컴포넌트에 의존하게 된다.
- 즉, **불안정한 컴포넌트가 안정된 컴포넌트를 의존한다. => "SDP 위배"**

이런 문제를 해결하기 위해 '**DIP**'를 사용한다.

<p><img src="img/component-sdp-violation-dip.png" width="300"></p>

- `US`라는 인터페이스를 만들고 `UServer`에 넣는다.
- `C`가 `US` 인터페이스를 구현한다.
- `U`는 `US`에 의존한다.
- 결과적으로, `Stable`(`U`)과 `Flexible`(`C`) 사이 의존성을 끊고 I가 큰 컴포넌트(`U`, `C`)가 I가 더 작은 컴포넌트(`US`)에 의존하도록 하여 SDP를 만족한다.
- 이 때, `UServer`는 매우 안정된 상태가 된다.

### 추상 컴포넌트

- 추상 컴포넌트 : 오직 인터페이스만 포함하는 컴포넌트
- 추상 컴포넌트를 DIP로 활용할 때, 이 컴포넌트는 `I = 0`으로 상당히 안정적인 컴포넌트다.
- **추상 컴포넌트와 DIP를 사용해서 SDP 규칙을 준수한다.**

## 안정된 추상화 원칙 (Stable Abstractions Principle, SAP)

> 컴포넌트는 안정된 정도만큼만 추상화되어야 한다.

### 고수준 정책을 어디에 위치시켜야 하는가?

- 고수준 아키텍처나 정책 결정과 관련된 소프트웨어(business logic)는 자주 변경해서는 절대로 안된다.
- 즉, **시스템에서 고수준 정책을 캡슐화하는 소프트웨어는 반드시 안정된 컴포넌트(`I=0`)에 위치해야 한다.**
- 하지만, 고수준 정책을 안정된 컴포넌트에 위치시키면, 그 정책을 포함하는 소스 코드는 수정하기 어려워지고, 시스템 전체 아키텍처가 유연성을 잃는다.
- 컴포넌트가 **최고로 안정된 상태(`I=0`)이면서도 동시에 변경에 충분히 대응할 수 있도록 유연하게** 만들어야 한다.
  - **OCP** 사용 : OCP는 클래스를 수정하지 않고도 확장할 수 있게 만든다.
  - OCP에서 이를 위해 사용하는 것이 "**추상 클래스**"

### 안정된 추상화 원칙

- 안정성(stability)과 추상화 정도(abstractness) 사이의 관계
- 안정된 컴포넌트는 추상 컴포넌트여야 하며, 이를 통해 **안정성이 컴포넌트를 확장하는 일을 방해해서는 안된다.**
- 불안정한 컴포넌트는 코드를 쉽게 변경할 수 있어야 하므로, 반드시 구체 컴포넌트여야 한다.
- 즉, **안정된 컴포넌트는 반드시 인터페이스와 추상 클래스로 구성되어 쉽게 확장할 수 있어야 한다.**
- **SDP와 SAP를 결합하면 DIP와 같다.**
  - SDP : 의존성이 반드시 안정성의 방향으로 향해야 한다.
  - SAP : 안정성이 결국 추상화를 의미한다.
  - DIP : 변동성이 큰 구체 클래스에 직접 의존하지 않고(SDP), 안정된 추상 인터페이스에 의존(SAP)하도록 만드는 것 => 의존성을 제어 흐름에 역전시킴
  - DIP는 클래스에 대한 원칙. 클래스는 추상적이거나 아니거나 둘 중 하나.
  - SDP, SAP는 컴포넌트에 대한 원칙. 컴포넌트는 일부는 추상적이고 나머지는 안정적일 수 있다.

### 추상화(Abstractness) 지표

- 추상적인 컴포넌트 : 추상 클래스와 인터페이스가 포함된 컴포넌트
- `Nc` : 컴포넌트에 포함된 클래스 개수
- `Na` : 컴포넌트에 포함된 추상 클래스와 인터페이스의 개수
- 추상화 지표 `A = Na / Nc`
  - `A`는 0 ~ 1 사이의 값
  - `A = 0` : 컴포넌트에 추상 클래스가 하나도 없다.
  - `A = 1` : 컴포넌트가 오로지 추상 클래스와 인터페이스만을 포함한다.

### 주계열(Main Sequence)과 배제 구역(Zone of Exclusion)

안정성 지표(`I`)와 추상화 지표(`A`) 사이의 관계를 나타내는 그래프

<p><img src="img/component-sap-a-i-graph.png" width="300"></p>

- `(0,1)` : 최고로 안정적이며 추상화된 컴포넌트
- `(1,0)` : 최고로 불안정하며 구체화된 컴포넌트

보통 컴포넌트는 두 점이 아닌 영역에서 다른 어딘가에 위치한다.

- 다른 추상 클래스에서 파생해서 만드는 추상 클래스의 경우, 파생된 클래스는 추상적이면서도 의존성을 갖는다.
- 이런 클래스는 최고로 추상적이지만 최고로 안정적인 것은 아니다. (의존성 때문에 안정성 감소)

컴포넌트가 위치할 수 있는 합리적인 지점을 정의하는 것을 "**주계열**"이라고 한다. 반대로, 컴포넌트가 절대로 위치해서는 안되는 영역을 "**배제 구역**"이라고 한다.

<p><img src="img/component-sap-zone-of-exclusion.png" width="300"></p>

- 고통의 구역 : `(0,0)` 주변
  - 매우 안정적이며(`I=0`) 구체적(`A=0`)인 상태
  - 안정적이기 때문에 변경하기 어렵고, 추상적이지 않기 때문에 확장하기 어렵다.
  - 즉, 이 영역에 위치하는 컴포넌트는 **변경하기 매우 어렵다.**
  - 데이터베이스 스키마같은 Entity는 이 고통의 구역에 속한다.
    - 데이터베이스 스키마는 변동성이 높고 극단적으로 구체적이며 많은 컴포넌트가 의존함
    - 애플리케이션과 데이터베이스 사이에 위치한 인터페이스는 데이터베이스의 변경에 따라 자주 변경되며 영향을 받는 범위도 넓다.
  - 변동성이 없는 컴포넌트도 이 구역에 속한다.
    - OS, 프레임워크 등 `I=1`이라도 실제로 거의 변경되지 않는 컴포넌트
    - 고통의 구역에서 문제가 되는 경우는 '**변동성이 큰 컴포넌트**'
- 쓸모없는 구역 : `(1,1)` 주변
  - 매우 불안정하며(`I=1`) 최고로 추상적(`A=1`)인 상태.
  - 매우 불안정하다는 것은 아무것도 이 컴포넌트에 의존하지 않는다는 것
  - 즉, 이 영역에 위치하는 컴포넌트는 **쓸모가 없다.**

'배제 구역'에서 벗어나려면 배제 구역에서 가장 멀리 떨어져 있는 선분인 '주계열' 궤적의 주변으로 컴포넌트를 위치시켜야 한다.

<p><img src="img/component-sap-main-sequence-plot.png" width="300"></p>

- 주계열과의 거리 `D = |A + I - 1|`
- D가 0이면 컴포넌트가 주계열 바로 위에 위치
- D가 1이면 컴포넌트가 주계열에서 가장 멀리 위치
- **`D`가 커질 수록 컴포넌트가 배제 구역에 위치하게 되는 것**
- 표준 편차(`Z = 1`)인 영역을 벗어난 컴포넌트들은,
  - 자신에게 의존하는게 없는데도(안정적임에도) 너무 추상적이거나 (주계열 선분의 아래, A > I)
  - 자신에게 의존하는 컴포넌트가 많은데도(불안정함에도) 너무 구체적일 것 (주계열 선분의 위, A < I)