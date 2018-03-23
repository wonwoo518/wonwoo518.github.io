---
layout: post
title: "[ RX_COCOA ] 1. Adding numbers"
description: ""
tags: [swift]

---
 
RxCocoa의 샘플을 통해서 RxCocoa의 사용법을 익혀보도록 하자. RxCocoa의 첫번째 샘플은 Adding numbers이다. 각 TextField에 숫자를 변경하면 3개의 더한 값이 자동으로 계산되어 출력되는 간단한 프로그램이다. 


## 실행화면 
<img src="https://github.com/wonwoo518/wonwoo518.github.io/blob/master/images/addnumber.png" width="246" height="500" alt="">


## 소스코드 
```swift
import UIKit
import RxSwift
import RxCocoa
class NumbersViewController: ViewController {
    @IBOutlet weak var number1: UITextField!
    @IBOutlet weak var number2: UITextField!
    @IBOutlet weak var number3: UITextField!
    @IBOutlet weak var result: UILabel!
    override func viewDidLoad() {
        super.viewDidLoad()
        Observable.combineLatest(number1.rx.text.orEmpty, number2.rx.text.orEmpty, number3.rx.text.orEmpty) { textValue1, textValue2, textValue3 -> Int in
                return (Int(textValue1) ?? 0) + (Int(textValue2) ?? 0) + (Int(textValue3) ?? 0)
            }
            .map { $0.description }
            .bind(to: result.rx.text)
            .disposed(by: disposeBag)
    }
}
```


## 설명

### combineLatest
다수의 Observable을 합쳐서 하나의 새로운 Observable을 만드는 연산자 중에   combineLatest가 있다. combineLatest라는 이름으로도 
알수 있듯이 합산 연산의 소스가 되는 Observable의 가장 마지막 값이 합산 연산에 사용된다. 
이 예제에선  number1.rx.text.orEmpty , number2.rx.text.orEmpty, number3.rx.text.orEmpty가 합산연산의 소스가 되는 Observables 이다. 
초기 number1의 값이 1, number2값이 2, number3의 초기값이 3이면 combineLatest의 클로져 코드에 따라 6을 방출하는 Observable이 만들어진다. 
number1값을 10으로 바꾸게 된다면 10 + 2 + 3 이 되어 15를 방출하게 된다. 이 상태에서 number2의 값을 10으로 바꾸면 10 + 10 + 3이 되어 33이 방출된다. 
중요한 것은 각 observable의 가장 마지막 값을 사용되었다는 것이다. 

이젠 combineLatest의 prototype을 살펴보자. combineLatest의 인자가 3개의  ObservableType을 취하고 클로져는 01.E, 02.E, 03.E를 인자로 받고 Self.E를 리턴한다. 
그렇다면 01.E, 02.E, 03.E는 무엇을 의미하는 것인지, Self.E는 무엇인지 살펴보도록 하자. 

```swift
public static func combineLatest<O1, O2, O3>(_ source1: O1, _ source2: O2, _ source3: O3, resultSelector: @escaping (O1.E, O2.E, O3.E) throws -> Self.E) -> RxSwift.Observable<Self.E> where O1 : ObservableType, O2 : ObservableType, O3 : ObservableType
```

01, 02, 03은 combineLatest의 인자인 number1.rx.text.orEmpty , number2.rx.text.orEmpty, number3.rx.text.orEmpty가 되는 것이다. 
여기에 .E가 붙었다. 이를 확인하기  .rx.text.orEmpty가 어떤 타입인지 살펴볼 필요가 있다. 
RxCocoa.ControlProperty<String>

RxCocoa.ControlProperty 타입이다. RxCocoa.ControlProperty를 살펴보자. 
```swift
public struct ControlProperty<PropertyType> : ControlPropertyType {
    public typealias E = PropertyType
}
```

RxCocoa.ControlProperty.E는 RxCocoa.ControlProperty<String> 인 경우에 String이 되는 것이다. 
따라서 01.E, 02.E, 03.E에서 E는 String이다. 

이젠 Self.E 타입이 어떤 타입인지 생각해보자. 
Self.E는 샘플코드상에서 아래 구문을 통해서 Int가 되는 것을 알 수 있다. 
```swift
textValue1, textValue2, textValue3 -> Int
```
이 값은 클로져의 return 타입에 의해 결정된다. 만약 클로져의 리턴타입이 Int가 아닌 String이라면 String으로 변경가능하다. 


### map
map연산은 Observable이 방출하는 각 요소에 연산을 더해 새로운 Observable을 리턴한다. 그런데 왜 여기서 map을 사용했을까 하는 것이다. 
이는 combineLatest를 통해 생성된 Observable이 방출하는 타입이 Int이다. 이 Int를 String으로 변환하는 역할을 한다. 
String으로 변환하는 이유는 bind에서 사용할 observer의 타입이 String이기 때문이다. 
map을 굳지 쓰지 않고 싶다면 combineLatest의  return타입을 String으로 바꾸면 된다. 


### bind
옵저버를 등록한다. combineLatest가 이벤트를 방출할때 그 방출한 값을 받아서 observer.E에 값을 할당한다. 
```swift
public func bind<O>(to observer: O) -> Disposable where O : ObserverType, O.E == Self.E?
```
