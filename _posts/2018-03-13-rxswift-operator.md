Operator는 여러 Observable를 하나의 Observable로 결합하는 역할을 한다. 

## startWith
Observable의 sequence요소보다 먼저 방출할 요소를 지정한다. [startWith](http://reactivex.io/documentation/operators/startwith.html)

```swift
    let disposeBag = DisposeBag()
    
    Observable.of("5", "6", "7", "8")
        .startWith("1", "2", "3")
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)


//결과
1
2
3
5
6
7
```

## merge
여러 Observable을 새로운 Observable로 합친다. [merge](http://reactivex.io/documentation/operators/merge.html)
```swift
    let disposeBag = DisposeBag()
    
    let subject1 = PublishSubject<String>()
    let subject2 = PublishSubject<String>()
    
    Observable.of(subject1, subject2)
        .merge()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
    
    subject1.onNext("1")
    
    subject1.onNext("2")
    
    subject2.onNext("a")
    
    subject2.onNext("b")
    
    subject1.onNext("3")
    
    subject2.onNext("c")

//결과
1
2
a
b
3
c
```

## zip
Observable 8개까지 결합해 새로운 Observable을 만든다. 이 Observable에서 방출되는 요소는 소스가 되는 Observable 요소의 결합형태로 방출된다. [zip](http://reactivex.io/documentation/operators/zip.html)
```swift
    let disposeBag = DisposeBag()
    
    let stringSubject = PublishSubject<String>()
    let intSubject = PublishSubject<Int>()
    
    Observable.zip(stringSubject, intSubject) { stringElement, intElement in
        "\(stringElement) \(intElement)"
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
    
    stringSubject.onNext("a")
    stringSubject.onNext("b")
    
    intSubject.onNext(1)
    
    intSubject.onNext(2)
    
    stringSubject.onNext("c")
    intSubject.onNext(3)

//결과
a 1
b 2
c 3
```

## combineLatest
zip과 거의 동일하지만 소스 Observable의 가장 마지막에 방출된 요소가 다른 Observable의 요소와 짝이된다.  [combineLastest](http://reactivex.io/documentation/operators/combinelatest.html)
```swift
    let disposeBag = DisposeBag()
    
    let stringSubject = PublishSubject<String>()
    let intSubject = PublishSubject<Int>()
    
    Observable.combineLatest(stringSubject, intSubject) { stringElement, intElement in
            "\(stringElement) \(intElement)"
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
    
    stringSubject.onNext("a")
    
    stringSubject.onNext("b")
    intSubject.onNext(1)
    
    intSubject.onNext(2)
    
    stringSubject.onNext("c")

//결과
b 1
b 2
c 2
```

## switchLast 
Observable을 중간에 스위칭한다.RxSwift에서 API로 구현되어 있지는 않으나 샘플과같이 구현할 수 있다.[switchLast](http://reactivex.io/documentation/operators/switch.html)
```swift
    let disposeBag = DisposeBag()
    
    let subject1 = BehaviorSubject(value: "a")
    let subject2 = BehaviorSubject(value: "f")
    
    let variable = Variable(subject1)
        
    variable.asObservable()
        .switchLatest()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
    
    subject1.onNext("b")
    subject1.onNext("c")
    
    variable.value = subject2
    
    subject1.onNext("d")
    subject1.onNext("e")
    
    subject2.onNext("g")

//결과
a
b
c
f
g
```

```swift
```
