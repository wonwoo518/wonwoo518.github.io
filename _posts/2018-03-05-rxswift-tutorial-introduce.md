---
layout: post
title: "[ SWIFT ] Key Value Observing"
description: >
  Key-Value Observing의 개념과 방법에 대해 알아본다. 
tags: [swift]

---

Key-Value Observing은 객체에 옵저버 패턴을 적용하는 하나의 방식이며
아래와 같은 특징이 있다. 
- 옵저빙하고자 하는 객체는 NSObject를 상속받아야 한다. 
- 옵저빙하는 객체에 알람을 등록할 시 식별자는 객체의 KeyPath를 사용한다. 
- 구현해야할 이벤트 핸들러는 NSObject의 observeValue 메소드를 재정의해서 사용한다.

KeyPath는 옵저빙할 객체의 property를 문자열 형태로 표현한 것이다. 
예를들어 UIView 인스턴스의 frame값을 옵저빙하고 싶을때 KeyPath는 "instance명.frame"이 되는 것이다. 
keyPath매크로를 이용해서 #keyPath(instance명.frame)으로 표현할 수도 있다. 

KVO를 객체에 적용하는 방법을 아래와 같이 Observer 등록, 이벤트 핸들러 구현 두 단계로 진행된다. 

## Observer 등록하기
~~~
addObserver(self, forKeyPath: #keyPath(self.view.frame), options: [.old, .new], context: nil)
~~~

## Event Handler등록하기 
~~~
override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
    if keyPath == #keyPath(self.view.frame) {
        // frame이 변경될 때 하고자 하는 작업
    }
}
~~~



## Why use RxSwift?
@IBAction 이벤트 핸들링, keyboard 위치 변화, URL 세션 응답, KVO 같은 시스템은 다양한 응답/요청 시스템은 코드를 복잡하게 만든다. 
다양한 시스템에서 요청/응답에 대한 코드를 일관성있게 만들기 위한 것이 Rx 시스템이다. 



## Concepts
모든 Observable 인스턴스는 Sequence이다. 
Observable sequence Swift sequence의 주요 이점은 요소를 비동기 적으로 수신 할 수 있다는 것이고 이것이 RxSwift의 본질입니다. 다른 모든 것들은 개념을 확장한다.
Observable (ObservableType) 은 sequence와 동일하다 
ObservableType.subscribe(_:) 는 Sequence.makeIterator()과 동일하다
ObservableType.subscribe(_:) 는 ObserverType 파라미터를 가지고 Observable이 방출하는 이벤트를 자동으로 수신한다. 


## Rx.playground
~~~
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