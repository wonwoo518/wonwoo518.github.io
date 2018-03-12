---
layout: post
title: "[ RX_SWIFT ] 2. Observable"
description: ""
tags: [swift]

---
Observable을 생성하는 여러 방법과 event subscription에 대한 예입니다. 

## Creating and Subscribing to Observables

	* never 
	* empty
	* just
	* of
	* from
	* create
	* range
	* repeatElement
	* Generate
	* deferred
	* error
	* do

```swift
//Observable의 create method
//"Never"
//어떤 이벤트도 방출하지 않음.
example("never") {
    let disposeBag = DisposeBag()
    let neverSequence = Observable<String>.never()
   
    let neverSequenceSubscription = neverSequence
        .subscribe { _ in
            print("This will never be printed")
    }
   
    neverSequenceSubscription.disposed(by: disposeBag)
}
//"Empty"
//Empty Observable sequence는 .completed 이벤트만 방출한다.
example("empty") {
    let disposeBag = DisposeBag()
    Observable<Int>.empty()
        .subscribe{ event in
            print(event)
        }
        .disposed(by: disposeBag)
}
//"just"
//1개의 요소만 있는 Observable 있음.
example("just") {
    let disposeBag = DisposeBag()
   
    Observable.just("🔴")
        .subscribe { event in
            print(event)
        }
        .disposed(by: disposeBag)
}
//"of"
//고정된 수의 요소에서 Observable 생성
//subscribe(onNext:)는 오직 다음 요소에 대한 이벤트만 핸들링
//error, complete에 대해 핸들링이 필요하다면 subscribe(onNext:onError:onCompleted:onDisposed) 이용
example("of") {
    let disposeBag = DisposeBag()
   
    Observable.of("🐶", "🐱", "🐭", "🐹")
        .subscribe(onNext: { element in
            print(element)
        })
        .disposed(by: disposeBag)
}
//"from"
//Array, Dictionary, Set으로부터 Observable 생성
example("from") {
    let disposeBag = DisposeBag()
   
    Observable.from(["🐶", "🐱", "🐭", "🐹"])
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
//"create"
//custom Observable sequence 생성
example("create") {
    let disposeBag = DisposeBag()
   
    let myJust = { (element: String) -> Observable<String> in
        return Observable.create { observer in
            observer.on(.next(element))
            observer.on(.completed)
            return Disposables.create()
        }
    }
   
    myJust("🔴")
        .subscribe { print($0) }
        .disposed(by: disposeBag)
}
//"range"
//상수 범위의 Observable sequence생성
example("range") {
    let disposeBag = DisposeBag()
   
    Observable.range(start: 1, count: 10)
        .subscribe { print($0) }
        .disposed(by: disposeBag)
}
//"repeatElement"
example("repeatElement") {
    let disposeBag = DisposeBag()
   
    Observable.repeatElement("🔴")
        .take(3)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
//"generate"
//조건에 따라 값을 생성하는 Observable sequence생성
example("generate") {
    let disposeBag = DisposeBag()
   
    Observable.generate(
        initialState: 0,
        condition: { $0 < 3 },
        iterate: { $0 + 1 }
        )
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
//"deferred"
//각각의 subscriber를 위한 Observable sequence생성
example("deferred") {
    let disposeBag = DisposeBag()
    var count = 1
   
    let deferredSequence = Observable<String>.deferred {
        print("Creating \(count)")
        count += 1
       
        return Observable.create { observer in
            print("Emitting...")
            observer.onNext("🐶")
            observer.onNext("🐱")
            observer.onNext("🐵")
            return Disposables.create()
        }
    }
   
    deferredSequence
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
   
    deferredSequence
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
//"error"
//바로 종료됨
example("error") {
   
    enum TestError: Error {
        case dummyError
        case dummyError1
        case dummyError2
    }
    let disposeBag = DisposeBag()
   
    Observable<Int>.error(TestError.dummyError)
        .subscribe { print($0) }
        .disposed(by: disposeBag)
}
//"doOn"
//이벤트 방출전에 전처리
example("doOn") {
    let disposeBag = DisposeBag()
   
    Observable.of("🍎", "🍐", "🍊", "🍋")
        .do(onNext: { print("Intercepted:", $0) }, onError: { print("Intercepted error:", $0) }, onCompleted: { print("Completed")  })
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```