---
layout: post
title: "[ RX_SWIFT ] RxSwift Tutorial 1"
description: >
  https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground 를 토대로 RxSwift 익히기  
tags: [swift]

---
#[ RX_SWIFT ] RxSwift Tutorial 1

## Why use RxSwift?
@IBAction 이벤트 핸들링, keyboard 위치 변화, URL 세션 응답, KVO 같은 다양한 응답/요청 시스템은 코드를 복잡하게 만든다. 
다양한 시스템에서 요청/응답에 대한 코드를 일관성있게 만들어 주는 것이  Rx 시스템이다. 



## Concepts
모든 Observable 인스턴스는 Sequence이다. 
Observable sequence Swift sequence의 주요 이점은 요소를 비동기 적으로 수신 할 수 있다는 것이고 이것이 RxSwift의 본질입니다. 다른 모든 것들은 개념을 확장한다.
Observable (ObservableType) 은 sequence와 동일하다 
ObservableType.subscribe(_:) 는 Sequence.makeIterator()과 동일하다
ObservableType.subscribe(_:) 는 ObserverType 파라미터를 가지고 Observable이 방출하는 이벤트를 자동으로 수신한다. 


~~~
Playground 에서  Observable 생성 예
~~~
~~~java
import RxSwift
func example(_ description:String, action:()->Swift.Void){
    print("--\(description)--")
    action()
    print("\n")
}

example("Observable with no subscribers"){
    _ = Observable<String>.create { obervableString -> Disposable in 
        print("This will never be printed")
        obervableString.on(.next("😬"))
        obervableString.on(.completed)
        return Disposables.create()
    }
}
example("Observable with no subscribers"){
    _ = Observable<String>.create { obervableString in
        print("Observable created")
        obervableString.on(.next("😉"))
        obervableString.on(.completed)
        return Disposables.create()
        }
        .subscribe { event in
            print(event)
    }
}

~~~