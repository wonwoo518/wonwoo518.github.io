---
layout: post
title:  "[ RX_SWIFT ] 3. Subject"
description: ""
tags: [swift]
---

Subject에 대해서 알아본다.

## Subject
Observable, observer 두가지 기능을 하는 프록시 또는 브릿지이다. observer이기에 하나 이상의 Observable을 구독하거나 Observable이기에 아이템을 내보낼 수 있다. subject 종류에 따른 동작은 [RxSwift](http://reactivex.io/documentation/subject.html) 참고하자. 
 


## PublishSubject
모든 옵저버에게 이벤트를 전달하지만 옵저버는 구독한 시간 이후에 발생한 이벤트만  전달된다.
```swift
    let disposeBag = DisposeBag()
    let subject = PublishSubject<String>()
    
    subject.subscribe{ print("observer1 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("a")
    subject.onNext("b")
    
    subject.subscribe{ print("observer2 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("c")
    subject.onNext("d")

	//결과 
    observer1 Event:next(a)
    observer1 Event:next(b)
    observer1 Event:next(c)
    observer2 Event:next(c)
    observer1 Event:next(d)
    observer2 Event:next(d)
```
## ReplaySubject
모든 옵저버에게 이벤트를 전달하고 옵저버는 구독한 시간 이전에 발생한 이벤트도  bufferSize만큼 받을 수 있다. 

```swift
    let disposeBag = DisposeBag()
    let subject = ReplaySubject<String>.create(bufferSize: 2)
    
    subject.subscribe{ print("observer1 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("a")
    subject.onNext("b")
    
    subject.subscribe{ print("observer2 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("c")
    subject.onNext("d")

    //결과 
    observer1 Event:next(a)
    observer1 Event:next(b)
    observer1 Event:next(a)
    observer1 Event:next(b)
    observer1 Event:next(c)
    observer1 Event:next(c)
    observer1 Event:next(d)
    observer1 Event:next(d)
```
## BehaviorSubject
모든 옵저버에게 이벤트를 전달하며 옵저버가 생기기 이전 가장 마지막 이벤트가 옵저버에 전달된다. 
```swift
    let disposeBag = DisposeBag()
    let subject = BehaviorSubject(value: "z")
    
    subject.subscribe{ print("observer1 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("a")
    subject.onNext("b")
    
    subject.subscribe{ print("observer2 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("c")
    subject.onNext("d")
    
    subject.subscribe{ print("observer3 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("e")
    subject.onNext("f")

    //결과 
    observer1 Event:next(z)
    observer1 Event:next(a)
    observer1 Event:next(b)
    observer2 Event:next(b)
    observer1 Event:next(c)
    observer2 Event:next(c)
    observer1 Event:next(d)
    observer2 Event:next(d)
    observer3 Event:next(d)
    observer1 Event:next(e)
    observer2 Event:next(e)
    observer3 Event:next(e)
    observer1 Event:next(f)
    observer2 Event:next(f)
    observer3 Event:next(f)
```