---
layout: post
title: "[ RX_SWIFT ] 7. Connectable Operators"
description: ""
tags: [swift]

---

Observable은 subscribe 시 이벤트를 방출한다. Connectable Observable은 connect 호출 시 이벤트가 방출된다. 
Observable을 connectable로 만들어주는 operator에 대해 살펴보자. 

## connectable Operator 없는 상황에서의 Observable의 예 
```swift
public func delay(_ delay: Double, closure: @escaping () -> Void) {

    DispatchQueue.main.asyncAfter(deadline: .now() + delay) {
        closure()
    }
}

func sampleWithoutConnectableOperators() {
    let interval = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
    
    _ = interval
        .subscribe(onNext: { print("Subscription: 1, Event: \($0)") })      //이벤트 방출 시작 
    
    delay(5) {
        _ = interval
            .subscribe(onNext: { print("Subscription: 2, Event: \($0)") })  
    }
}


sampleWithoutConnectableOperators()



//output
Subscription: 1, Event: 0
Subscription: 1, Event: 1
Subscription: 1, Event: 2
Subscription: 1, Event: 3
Subscription: 1, Event: 4
Subscription: 1, Event: 5
Subscription: 2, Event: 0
Subscription: 1, Event: 6
Subscription: 2, Event: 1
Subscription: 1, Event: 7
Subscription: 2, Event: 2
Subscription: 1, Event: 8
Subscription: 2, Event: 3
Subscription: 1, Event: 9
Subscription: 2, Event: 4
Subscription: 1, Event: 10
Subscription: 2, Event: 5
Subscription: 1, Event: 11
Subscription: 2, Event: 6
Subscription: 1, Event: 12
Subscription: 2, Event: 7
Subscription: 1, Event: 13
Subscription: 2, Event: 8
Subscription: 1, Event: 14
...

```

## publish 
Observable sequence를 connectable sequence로 바꿔준다. [publish](http://reactivex.io/documentation/operators/publish.html)
```swift
func sampleWithPublish() {
    printExampleHeader(#function)
    
    //interval은 일정시간 마다 요소를 방출하는 Observable을 만든다.
    let intSequence = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        .publish()
    
    //이벤트를 방출하지 않음 
    _ = intSequence
        .subscribe(onNext: { print("Subscription 1:, Event: \($0)") })
    
    //connect호출하면서 이벤트 방출 시작 
    delay(2) { _ = intSequence.connect() }
    
    delay(4) {
        _ = intSequence
            .subscribe(onNext: { print("Subscription 2:, Event: \($0)") })
    }
    
    delay(6) {
        _ = intSequence
            .subscribe(onNext: { print("Subscription 3:, Event: \($0)") })
    }
}

sampleWithPublish()


//output
--- sampleWithPublish() example ---
Subscription 1:, Event: 0
Subscription 1:, Event: 1
Subscription 2:, Event: 1
Subscription 1:, Event: 2
Subscription 2:, Event: 2
Subscription 1:, Event: 3
Subscription 2:, Event: 3
Subscription 3:, Event: 3
Subscription 1:, Event: 4
Subscription 2:, Event: 4
Subscription 3:, Event: 4
Subscription 1:, Event: 5
Subscription 2:, Event: 5
Subscription 3:, Event: 5
Subscription 1:, Event: 6
Subscription 2:, Event: 6
Subscription 3:, Event: 6
Subscription 1:, Event: 7
Subscription 2:, Event: 7
Subscription 3:, Event: 7
Subscription 1:, Event: 8
Subscription 2:, Event: 8
Subscription 3:, Event: 8
Subscription 1:, Event: 9
Subscription 2:, Event: 9
Subscription 3:, Event: 9

```

## replay
Observable sequence를 connectable sequence로 바꿔준다. 버퍼사이즈만큼 이전에 발생했던 이벤트를 각 subscriber에게 전달한다. [replay](http://reactivex.io/documentation/operators/replay.html)

```swift
func sampleWithReplayBuffer() {
    
    let intSequence = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        .replay(5)
    
    _ = intSequence
        .subscribe(onNext: { print("Subscription 1:, Event: \($0)") })
    
    delay(2) { _ = intSequence.connect() }
    
    delay(10) {
        _ = intSequence
            .subscribe(onNext: { print("Subscription 2:, Event: \($0)") })
    }
}

//output
Subscription 1:, Event: 0
Subscription 1:, Event: 1
Subscription 1:, Event: 2
Subscription 1:, Event: 3
Subscription 1:, Event: 4
Subscription 1:, Event: 5
Subscription 1:, Event: 6
Subscription 2:, Event: 2
Subscription 2:, Event: 3
Subscription 2:, Event: 4
Subscription 2:, Event: 5
Subscription 2:, Event: 6
Subscription 1:, Event: 7
Subscription 2:, Event: 7
Subscription 1:, Event: 8
Subscription 2:, Event: 8
```

## multicast
Observable sequence를 connectable sequence로 바꿔준다. multicast 에 subject를 지정할 수 있고 이 subject에 함께 이벤트를 방출한다. 
```swift
func sampleWithMulticast() {
    let subject = PublishSubject<Int>()
    let subject2 = PublishSubject<Int>()
    
    _ = subject
        .subscribe(onNext: { print("\tSubject 1: Event: \($0)") })
    
    let intSequence = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        .multicast(subject)
    
    _ = intSequence
        .subscribe(onNext: { print("\tSubscription 1:, Event: \($0)") })
    
    delay(2) { _ = intSequence.connect() }
    
    delay(4) {
        _ = intSequence
            .subscribe(onNext: { print("\tSubscription 2:, Event: \($0)") })
    }
    
    delay(6) {
        _ = intSequence
            .subscribe(onNext: { print("\tSubscription 3:, Event: \($0)") })
    }
    
    
}

sampleWithMulticast()

//output
Subject 1: Event: 0
Subscription 1:, Event: 0
Subject 1: Event: 1
Subscription 1:, Event: 1
Subscription 2:, Event: 1
Subject 1: Event: 2
Subscription 1:, Event: 2
Subscription 2:, Event: 2
Subject 1: Event: 3
Subscription 1:, Event: 3
Subscription 2:, Event: 3
Subscription 3:, Event: 3
Subject 1: Event: 4
Subscription 1:, Event: 4
Subscription 2:, Event: 4
Subscription 3:, Event: 4
Subject 1: Event: 5
Subscription 1:, Event: 5
Subscription 2:, Event: 5
Subscription 3:, Event: 5
Subject 1: Event: 6
Subscription 1:, Event: 6
Subscription 2:, Event: 6
Subscription 3:, Event: 6
Subject 1: Event: 7
Subscription 1:, Event: 7
Subscription 2:, Event: 7
Subscription 3:, Event: 7
Subject 1: Event: 8
Subscription 1:, Event: 8
Subscription 2:, Event: 8
Subscription 3:, Event: 8
Subject 1: Event: 9
Subscription 1:, Event: 9
Subscription 2:, Event: 9
Subscription 3:, Event: 9

```


