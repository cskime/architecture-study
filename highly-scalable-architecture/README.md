# Building a Scalable App Architecture for SuperApp Operations

## Tools

- 🎥 [The RED: 슈퍼앱 운영을 위한 확장성 높은 앱 아키텍처 구축 by 노수진](https://fastcampus.co.kr/dev_red_rsj)
- 🌐 [MiniSuperApp-fastcampus | GitHub](https://github.com/nsoojin/MiniSuperApp-fastcampus?tab=readme-ov-file)
- 📦 [ModernRIBs](https://github.com/DevYeom/ModernRIBs)
- 📦 [Combine Schedulers](https://github.com/pointfreeco/combine-schedulers)
- 📦 [CombineExt](https://github.com/CombineCommunity/CombineExt)
- 📦 [Swifter](https://github.com/httpswift/swifter)
- 📦 [Hammer](https://github.com/lyft/Hammer)
- 📦 [SnapshotTesting](https://github.com/pointfreeco/swift-snapshot-testing)

## Contents

### Subjects

- Composition
- Composition root
- Modular architecture
- Dependency control
- Loose coupling & OCP
- Test pyramid: Unit test, Snapshot Test, UI test, Integration test
- Feature flag

### Summary

- Massive object를 피하려면 architecture 외에 composition도 함께 고려해서 설계해야 한다.
- 불필요한 코드를 모두 컴파일하면서 늘어나는 빌드 시간에 의해 생산성이 떨어지는 것을 방지하기 위해 모듈화 구조를 적용한다.
- 의존성 역전을 통해 모듈간 의존 관계를 제어하여 빌드 시간을 더욱 단축한다.
- 의존성 주입은 trade-off 이므로 꼭 필요한 곳에서만 의존성을 역전시킨다.
- app root를 composition root로 만들어서 모듈 간 의존성을 완전히 끊어내어 느슨한 결합을 유지할 수 있다.
- 자동화 테스트를 통해 문제를 미리 예방하고, 발생한 문제를 빠르게 발견해서 수정할 수 있는 장치를 마련한다.

### Log

|     |                                       Name                                        |    Date    |
| :-: | :-------------------------------------------------------------------------------: | :--------: |
|  1  | [모바일 개발자의 Scalability와 앱 아키텍처](./01.scalability-app-architecture.md) | 2024.02.14 |
|  2  |               [코드 레벨 아키텍처](./02.code-level-architecture.md)               | 2024.02.20 |
|  3  |              [모듈 레벨 아키텍처](./03.module-level-architecture.md)              | 2024.02.23 |
|  4  |                    [자동화 테스팅](./04.automated-testing.md)                     | 2024.02.28 |
|  5  |                        [확장성 있는 인프라](./05.infra.md)                        | 2024.02.28 |
