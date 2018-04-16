---
layout: post
title: "[ RX_SWIFT ]  RxSwift 예제 Simple Validation Sample 을 분석해보자."
description: ""
tags: [swift]

---


RxSwift의 공식 두번째 예제 Simple Validation 소스를 분석하고 여기 사용된 RxSwift 메소드와 동작 원리에 대해서 생각해보자.



## Simple Validation

사용자 이름을 입력 받아서 특정 조건을 만족하면 패스워드 입력을 활성화하고 패스워드 입력값이 특정 조건을 만족했을 때 

사용자 이름과 패스워드가 모두 정상입력되었으면  버튼을 활성화 하는 프로그램을 만들어보자. 



### 프로그램 실행 화면 



<img src="https://github.com/wonwoo518/wonwoo518.github.io/blob/master/images/simplevalidation.png?raw=true" width="286" height="550" alt="">



### Source Code  

***usernameValidOutlet*** 

- UILabel 로 usernameOutlet.text값이 조건을 만족하지 않을 때 만족해야할 조건을 출력한다. 

***passwordValidOutlet*** 

- UILabel 로 passwordOutlet.text값이 조건을 만족하지 않을 때 만족해야할 조건을 출력한다. 

```swif
usernameValidOutlet.text = "Username has to be at least \(minimalUsernameLength) characters"
passwordValidOutlet.text = "Password has to be at least \(minimalPasswordLength) characters"
```





***usernameOutlet*** 

-  사용자 이름 입력을 받는 UITextField

***passwordOutlet*** 

- 비밀번호 입력을 받는 UITextField

***usernameValid*** 

- usernameOutlet.text값이 valid한지를 Bool값으로 방출하는 Observable이다. 
- usernameOutlet.rx.text는 subject로 사용자 입력을 observe하고 이벤트를 방출한다. 

***passwordValid*** 

- passwordOutlet.text값이 valid한지를 Bool값으로 방출하는 Observable이다. 

***everythingValid*** 

- 가장 최근에 usernameValid, passwordValid에서 방출된 이벤트 모두 true인지를 Bool값으로 방출하는 Observable이다. 

```swift
        
let usernameValid:Observable<Bool> = usernameOutlet.rx.text.orEmpty 
	.map { $0.count >= minimalUsernameLength }                      
	.share(replay: 1)                                               

let passwordValid:Observable<Bool> = passwordOutlet.rx.text.orEmpty
	.map { $0.count >= minimalPasswordLength }
	.share(replay: 1)

let everythingValid:Observable<Bool> = Observable.combineLatest(usernameValid, passwordValid) { $0 && $1 }
            .share(replay: 1)
```



***Observable에 Observer 적용하기*** 

- usernameValid, passwordValid, everythingValid 각 Observable 의 이벤트를 수신할 observer를 지정한다. 
- Observer 지정시 bind 메소드를 이용한다.
- rx.isEnabled, rx.isHidden는 Binder타입이며 Binder는 ObserverType을 상속받는다.  

```swift
usernameValid
    .bind(to: passwordOutlet.rx.isEnabled)
    .disposed(by: disposeBag)

usernameValid
    .bind(to: usernameValidOutlet.rx.isHidden)
    .disposed(by: disposeBag)

passwordValid
    .bind(to: passwordValidOutlet.rx.isHidden)
    .disposed(by: disposeBag)

everythingValid
    .bind(to: doSomethingOutlet.rx.isEnabled)
    .disposed(by: disposeBag)

```





***bind***

- observer 적용시 사용하며 파라미터는 observerType이다. 

```swift
public func bind<O>(to observer: O) -> Disposable where O : ObserverType, Self.E == O.E
```



***rx.isEnabled***

- RxCocoa의 UIControl extension으로 지원되는 isEnabled는 이벤트 발생 시 이벤트값을 control.isEnabled에 다시 할당하는 observerType을 상속받는 Binder이다. 

```swift
extension Reactive where Base: UIControl {
    
    /// Bindable sink for `enabled` property.
    public var isEnabled: Binder<Bool> {
        return Binder(self.base) { control, value in
            control.isEnabled = value
        }
    }
    ...
}

```





***rx.isHidden***

- RxCocoa의 UIView extension으로 지원되는 isHidden은 이벤트 발생 시 이벤트값을 view.isEnabled에 다시 할당하는 observerType을 상속받는 Binder이다. 

```swift
extension Reactive where Base: UIView {
    /// Bindable sink for `hidden` property.
    public var isHidden: Binder<Bool> {
        return Binder(self.base) { view, hidden in
            view.isHidden = hidden
        }
    }
    ...
}

```





## Full Source###

```swift
import UIKit
import RxSwift
import RxCocoa

fileprivate let minimalUsernameLength = 5
fileprivate let minimalPasswordLength = 5

class SimpleValidationViewController : ViewController {

    @IBOutlet weak var usernameOutlet: UITextField!
    @IBOutlet weak var usernameValidOutlet: UILabel!

    @IBOutlet weak var passwordOutlet: UITextField!
    @IBOutlet weak var passwordValidOutlet: UILabel!

    @IBOutlet weak var doSomethingOutlet: UIButton!

    override func viewDidLoad() {
        super.viewDidLoad()

        usernameValidOutlet.text = "Username has to be at least \(minimalUsernameLength) characters"
        passwordValidOutlet.text = "Password has to be at least \(minimalPasswordLength) characters"
        
        

        let usernameValid:Observable<Bool> = usernameOutlet.rx.text.orEmpty 
            .map { $0.count >= minimalUsernameLength }                      
            .share(replay: 1)                                               

        let passwordValid:Observable<Bool> = passwordOutlet.rx.text.orEmpty
            .map { $0.count >= minimalPasswordLength }
            .share(replay: 1)

        let everythingValid:Observable<Bool> = Observable.combineLatest(usernameValid, passwordValid) { $0 && $1 }
            .share(replay: 1)

        usernameValid
            .bind(to: passwordOutlet.rx.isEnabled)
            .disposed(by: disposeBag)

        usernameValid
            .bind(to: usernameValidOutlet.rx.isHidden)
            .disposed(by: disposeBag)

        passwordValid
            .bind(to: passwordValidOutlet.rx.isHidden)
            .disposed(by: disposeBag)

        everythingValid
            .bind(to: doSomethingOutlet.rx.isEnabled)
            .disposed(by: disposeBag)

        doSomethingOutlet.rx.tap
            .subscribe(onNext: { [weak self] _ in self?.showAlert() })
            .disposed(by: disposeBag)
    }

    func showAlert() {
        let alertView = UIAlertView(
            title: "RxExample",
            message: "This is wonderful",
            delegate: nil,
            cancelButtonTitle: "OK"
        )

        alertView.show()
    }
}

```

