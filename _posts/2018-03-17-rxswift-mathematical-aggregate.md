---
layout: post
title: "[ RX_SWIFT ] 7. Mathematical and Aggregate Operators"
description: ""
tags: [swift]

---

Observable에 의해 방출되는 모든 요소에 대한 연산을 제공하는 Operator에 대해 알아보자.  

## toArray
Observable을 하나의 배열을 원소로 갖는 Observable로 변환한다. [toArray](http://reactivex.io/documentation/operators/to.html)

```swift
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 10)
    .toArray()
    .subscribe { print($0) }
    .disposed(by: disposeBag)

//Output
next([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
completed
```

## reduce
누산기 역할을 하며 Observable의 각 요소를 초기값과 더해서 1개의 요소를 갖는 Observable을 리턴한다. [reduce](http://reactivex.io/documentation/operators/reduce.html)
```swift
let disposeBag = DisposeBag()

Observable.of(10, 100, 1000)
    .reduce(1, accumulator: +)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

//Output
1111    
```


## concat 
두개의 Observable을 합친다. 첫번째 Observable의 방출이 끝날 때까지 다음 Observable의 방출은 대기한다. [concat](http://reactivex.io/documentation/operators/concat.html)

```swift
    let disposeBag = DisposeBag()
    
    let subject1 = BehaviorSubject(value: "🍎")
    let subject2 = BehaviorSubject(value: "🐶")
    
    let variable = Variable(subject1)
    
    variable.asObservable()
        .concat()
        .subscribe { print($0) }
        .disposed(by: disposeBag)
    
    subject1.onNext("🍐")
    subject1.onNext("🍊")
    
    variable.value = subject2

    subject2.onNext("I would be ignored") //첫번째 방출이 완료되기 전에 방출되었기때문에 무시됨
    subject2.onNext("🐱")
    
    subject1.onCompleted()
    
    subject2.onNext("🐭")

//Output
next(🍎)
next(🍐)
next(🍊)
next(🐱)
next(🐭)
```
