# Routing Details

> CleanSwift handbook ch. 11~13을 공부하고 내용을 정리합니다.

## Routing

Routing logic에서는 3가지 동작을 수행한다.

1. Destination `View`와 `DataStore`를 가져온다.
2. Source `DataStore`에서 destination `DataStore`로 데이터를 전달한다. => **Data Passing**
3. Source `View`에서 destination `View`로 화면을 전환한다. => **Navigation**

Data passing은 `DataStore`를 갖는 protocol로, 화면 사이에 전달하려고 하는 데이터를 정의한다.

```swift
protocol ListOrderDataPassing {
    var dataStore: ListOrderDataStore? { get set }
}
```

Navigation은 `Routing Logic` protocol에서 실제로 화면을 전환하는 logic을 구현한다.

```swift
protocol ListOrderRoutingLogic {
    func routeToShowOrder()
}
```

## 구현

`ListOrderRouter`는 다음과 같이 구현할 수 있다.

1. `ListOrderRoutingLogic`과 `ListOrderDataPassing` 구현
    ```swift
    class ListOrderRouter: ListOrderRoutingLogic, ListOrderDataPassing {
        weak var source: ListOrderViewController?

        // MARK: - Data Passing

        var dataStore: ListOrderDataStore?

        // MARK: - Routing Logic

        func routeToShowOrder() {
            ...
        }
    }
    ```
2. VIP cycle을 setup 한다. Storyboard에서 segue를 사용하면 segue를 전달하고, segue로부터 destination view를 얻어올 수 있지만, code base로 UI를 구현할 때는 아무 것도 전달하지 않는다.
    ```swift
    func routeToShowOrder() {
        let destination = ShowOrderViewController()
        setUpVIPCycle(destination: destination)
        ...
    }

    private func setUpVIPCycle(destination: ShowOrderViewController) {
        let interactor = ShowOrderInteractor()
        let presenter = ShowOrderPresenter()
        let router = ShowOrderRouter()

        view.interactor = interactor
        interactor.presenter = presenter
        presenter.view = view

        let router = ShowOrderRouter()
        router.source = view
        router.dataStore = interactor
        view.router = router
    }
    ```
3. 데이터를 전달하고, 화면을 전환한다.
    ```swift
    func routeToShowOrder() {
        ...
        passData(from: dataStore, to: destination.dataStore)
        navigateToShowOrder(destination)
    }

    private func passData(from source: ListOrderDataStore?, to destination: ShowOrderDataStore?) {
        destination?.order = source.order
    }

    private func navigateToShowOrder(_ destination: ShowOrderViewController) {
        source.navigationController?.pushViewController(destination, animated: true)
    }
    ```

## Pass data to Backwards

- 이전 화면으로 되돌아갈 때 데이터를 전달하는 방법도 동일하게 구현할 수 있다.
    - `ShowOrderViewController`에서 `ListOrderViewController`로 돌아간다면,
    - `ShowOrderViewController`가 **source**가 되고, `ListOrderViewController`가 **destination**이 된다.
- 동일한 방법으로 데이터를 전달할 수 있다.
    ```swift
    class ShowOrderRouter: ShowOrderRoutingLogic, ShowOrderDataPassing {

        func passData(from source: ShowOrderDataStore?, to destination: ListOrderDataStore?) {
            destination?.order = source.order
        }

        func navigateToListOrder() {
            // source가 ShowOrderViewController가 됨
            source.navigationController?.popViewController(animated: true)
        }
    }
    ```

## Routing 방식의 장단점

### 장점

- 일반적인 화면 전환에서 backwards로 data passing을 구현하려면 delegate pattern을 사용해야 했다.
- Delegate pattern은 protocol을 더 만들어야 하고, 작성해야 하는 코드도 늘어나는 문제가 있었다.
- Clean Swift의 routing 방식은 delegate pattern을 사용하지 않고 간단하게 데이터를 전달할 수 있다.

### 단점

- Boilerplate 코드가 많이 생성되는 것 같다.
- VIP cycle은 명확하게 역할들이 나뉘는 것 같지만, routing은 애매한 감이 있다.
- Code base로 UI를 개발할 때는 VIP cycle을 초기화하는 코드를 깔끔하게 유지하기 어려운 것 같다. 의존성을 갖는 객체들이 많아서 의존성 주입을 깔끔하게 구현하기도 쉽지 않아 보인다.
