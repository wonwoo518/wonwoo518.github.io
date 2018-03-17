---
layout: post
title: "[ RX_SWIFT ] 5. Transforming Operator"
description: ""
tags: [swift]

---

 Transforming Operatoer는 Observable에 의해 방출되는 요소를 변형시키는 operator이다. 
 

## map
transforming closure를 Observable에 의해 방출되는 각 요소에 적용해서 새로운 Observable을 반환한다. [map](http://reactivex.io/documentation/operators/map.html)
```swift
let disposeBag = DisposeBag()
Observable.of(1, 2, 3)
    .map { $0 * $0 }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
//Output
1
4
9
```


## flatmap
Observable에 의해 방출된 요소를 변환하여 원래 Observable의 sequence를 구성합니다.  [flatmap](http://reactivex.io/documentation/operators/flatmap.html)
```swift
let disposeBag = DisposeBag()
    
struct Player {
    var score: Variable<Int>
}

let p1 = Player(score: Variable(80))
let p2 = Player(score: Variable(90))

let player = Variable(p1)
player.asObservable()                         // Variable 타입은 Observable을 상속하지 않고 composition으로 갖고 있는 형태이므로 asObservable로 변환함.
    .flatMap {return $0.score.asObservable()} // $0은 Player 타입이다. flatMap을 통해 Variable(<Int>) 타입으로 flat하게 만듦. asObservable로 변환하는 것은 return type이 Observable이기 때문. 
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

p1.score.value = 85
player.value = p2
p1.score.value = 95 // Will be printed when using flatMap, but will not be printed when using flatMapLatest
p2.score.value = 100

//Output
80
85
90
95
100
```

## scan 
함수형 언어의 각 요소를 더한 중간 값들을 리턴한다. [scan](http://reactivex.io/documentation/operators/scan.html)

```swift
let disposeBag = DisposeBag()

Observable.of(10,100,1000)
    .scan(1) { aggregateValue, newValue in
        aggregateValue + newValue
    }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

//Output
11
111
1111
```
