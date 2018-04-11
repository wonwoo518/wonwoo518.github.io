---
layout: post
title: "[ RX_SWIFT ] Why Subject?"
description: ""
tags: [swift]

---
 
Subject에 대해 생각해보기


## Subject는 무엇인가?
Subject는 Observable의 이벤트를 수신하는 observer이자 다른 observer로 이벤트를 방출하는 Observable이다. RxSwift공식 문서에 따르면 이른 일종의 브릿지 또는 프락시라고 명시되어 있다. 즉 Obsrevable 의 이벤트를 수신해서 다시 observer에게 리다이렉션하는 것이 Subject이다. Subject는 Observable 이벤트를 수신하기 때문에 observer이며 다른 observer로 이벤트를 방출하기 때문에 Observable이다. 

그림으로 보면 아래와 같이 << Observable >> — << Subject>>  —  << observer >> 식의 이벤트 전파 구조가 되는 것이다. 



```flow
st=>start: Observable
op=>operation: Subject
e=>end: observer

st->op->e
```





A에서 B로 이벤트를 직접 전파하지 않고 중간에 Subject를 쓰는 이유는 무엇일까? 



## Subject는 왜 필요하지? 

- "Cold"  Observable을 "Hot" Observable로 바꾸는 효과


- 동적인 이벤트 요소 추가 



### 이전에 발생한 이벤트에 대한 핸들링 

subject가 구독하는 Observable이 "cold"인 경우 subject가 구독을 시작함으로 subject자체가 "hot" Observable이 되는 효과를 지니게 된다. 



### 동적인 이벤트 요소 추가 

"Reactive Programming with Swift" 책에보면 앱을 개발할때 Observable에 동적으로 요소를 추가해야 할때가 많은데 
Observable에 동적으로 요소들을 추가해주는 메소드가 없다. 만약 "A"라는 Observable이 하나의 observer가 되고 동적으로 observer.on(.next)를 호출해준다면 "A"에 동적으로 방출할 이벤트가 생기게 되는 것이다. 
 

 사실 이런식의 접근 방법이 앱 개발 시 사용하는 일반적인 방법이다. RxSwift를 처음 접할때의 의문점이 왜 onNext 또는 on.(.next) 를 직접 호출해주는 것인지 의아했던 경험이 있는데 이런 이유에서 이다. 



아래 예제 코드를 보자. 

```swift
    let disposeBag = DisposeBag()
    let subject = PublishSubject<String>()
    
    subject.subscribe{ print("observer1 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("a")
    subject.onNext("b")
    
    subject.subscribe{ print("observer2 Event:\($0)") }.disposed(by:disposeBag)
    subject.onNext("c")
    subject.onNext("d")
```

subject.onNext를 호출해주고 있다. 이는 subject가 observer이기때문에 onNext를 호출할 수 있다. onNext는 Observable에 의해 호출되어야 하지만 동적으로 subject에 이벤트가 추가되는 것을 시뮬레이션 하기 위해 onNext를 강제로 호출하였다.subject.onNext의 호출은 subject가 이벤트를 수신했다는 의미가 되고 동시에 Observable로써 방출할 요소가 추가 되었다는 것을 의미한다. 따라서 subject.onNext("a")가 호출되면 event("a")가 subject에 방출할 요소로 생긴 것이고 동시에 이 이벤트를 구독하고 있는 subject의 observer 핸들러 { print("observer1 Event:\($0)") } 코드 블록이 실행되는 것이다. 



```sequence
Note right of Observable: Subject에 .on(.next)를 호출함으로 event가 추가 된다.
Observable->Subject: ObserverType::.on(.next)
Subject->Observer: 이벤트 방출 
```







