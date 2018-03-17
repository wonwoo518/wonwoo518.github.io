---
layout: post
title: "[ RX_SWIFT ] 6. Filtering and Aggregating Operators"
description: ""
tags: [swift]

---

 Observable sequence를 선택적으로 방출하는 연산자에 대해 알아보자. 
 

## filter
Observable 요소를 특정 조건에 따라 방출하게 한다. [filter](http://reactivex.io/documentation/operators/filter.html)
```swift
let disposeBag = DisposeBag()

Observable.of(
    "A", "B", "C",
    "E", "F", "G",
    "H", "I", "G")
    .filter {
        $0 == "A"
    }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)//Output

//Output
A
```


## distinctUntilChanged
Observable의 순서상 중복된 요소는 제거한다.   [distinctUntilChanged](http://reactivex.io/documentation/operators/distinct.html)
```swift
let disposeBag = DisposeBag()

Observable.of("A", "B", "A", "A", "A", "C", "A")
    .distinctUntilChanged()
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)


//Output
A
B
A
C
A
```

## elementAt 
Observable의 특정 index 값만 방출한다. [elementAt](http://reactivex.io/documentation/operators/elementat.html)

```swift
let disposeBag = DisposeBag()

Observable.of("0", "1", "2", "3", "4", "5")
    .elementAt(3)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
//Output
3
```


## single 
Observable의 첫번째 요소 또는 조건을 만족하는 첫번째 요소만 방출한다. Observable의 요소가 2개 이상이면 error을 던진다. [single](http://reactivex.io/documentation/operators/elementat.html)

```swift
//single 
let disposeBag = DisposeBag()

Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
    .single()
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
//Output
🐱
Unhandled error happened: Sequence contains more than one element.
 subscription called from:



//single with condition
Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
    .single { $0 == "🐸" }
    .subscribe { print($0) }
    .disposed(by: disposeBag)

Observable.of("🐱", "🐰", "🐶", "🐱", "🐰", "🐶")
    .single { $0 == "🐰" }
    .subscribe { print($0) }
    .disposed(by: disposeBag)

Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
    .single { $0 == "🔵" }
    .subscribe { print($0) }
    .disposed(by: disposeBag)
//Output
next(🐸)
completed
next(🐰)
error(Sequence contains more than one element.)
error(Sequence doesn't contain any elements.)


```

## take 
지정된 수 만큼만 Observable의 요소를 방출한다. [take](http://reactivex.io/documentation/operators/take.html)

```swift
let disposeBag = DisposeBag()

Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
    .take(3)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
//Output
🐱
🐰
🐶
```

## takeLast 
지정된 수 만큼만 Observable의 요소를 끝에서부터 방출한다. [takeLast](http://reactivex.io/documentation/operators/takelast.html)

```swift
 let disposeBag = DisposeBag()
    
Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
    .takeLast(3)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

//Output
🐸
🐷
🐵
```

## takeWhile
조건이 참이 되기 전까지 Observable 요소를 방출한다. [takeWhile](http://reactivex.io/documentation/operators/takewhile.html)

```swift
example("takeWhile") {
    let disposeBag = DisposeBag()
    
    Observable.of(1, 2, 3, 4, 5, 6)
        .takeWhile { $0 < 4 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}

//Output
1
2
3

```

## takeUntil
참조하는 Observable이 요소를 방출하기 전까지만 요소를 방출한다. [takeUntil](http://reactivex.io/documentation/operators/takeuntil.html)

```swift
let disposeBag = DisposeBag()

let sourceSequence = PublishSubject<String>()
let referenceSequence = PublishSubject<String>()

sourceSequence
    .takeUntil(referenceSequence)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

sourceSequence.onNext("🐱")
sourceSequence.onNext("🐰")
sourceSequence.onNext("🐶")

referenceSequence.onNext("🔴") //referenceSequence Observable이 요소를 방출하기 때문에 sourceSequence의 방출은 멈춘다. 

sourceSequence.onNext("🐸")
sourceSequence.onNext("🐷")
sourceSequence.onNext("🐵")

//Output
next(🐱)
next(🐰)
next(🐶)
completed
```

## skip 
N개의 요소는 건너뛰고 방출한다. [skip](http://reactivex.io/documentation/operators/skip.html)

```swift
let disposeBag = DisposeBag()

Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
    .skip(2)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

//Output
🐶
🐸
🐷
🐵
```

## skipWhile 
조건이 만족되기 전까지의 요소는 건너뛰고 조건이 만족된 후부터 요소를 방출한다. [skipWhile](http://reactivex.io/documentation/operators/skipwhile.html)

```swift
let disposeBag = DisposeBag()

Observable.of(1, 2, 3, 4, 5, 6)
    .skipWhile { $0 < 4 }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

//Output
4
5
6
```

## skipWhileWithIndex 
skipWhile과 동일하며 추가로 조건 closure에 index가 인자로 전달된다.  [skipWhileIndex](http://reactivex.io/documentation/operators/skipwhilewithindex.html)

```swift
let disposeBag = DisposeBag()
    
Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
    .skipWhileWithIndex { element, index in
        index < 3
    }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

//Output
🐸
🐷
🐵
```



## skipUntil 
참조하는 Observable이 방출하기 전까지의 요소는 스킵하며 참조하는 Observable이 방출하면 방출을 시작한다. [skipUntil](http://reactivex.io/documentation/operators/skipUntil.html)

```swift
let disposeBag = DisposeBag()

let sourceSequence = PublishSubject<String>()
let referenceSequence = PublishSubject<String>()

sourceSequence
    .skipUntil(referenceSequence)
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)

sourceSequence.onNext("🐱")
sourceSequence.onNext("🐰")
sourceSequence.onNext("🐶")

referenceSequence.onNext("🔴")

sourceSequence.onNext("🐸")
sourceSequence.onNext("🐷")
sourceSequence.onNext("🐵")

//Output
🐸
🐷
🐵
```

