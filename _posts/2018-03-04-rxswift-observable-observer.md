---
layout: post
title: "[ RX_SWIFT ] 0. ObservableType vs ObserverType"
description: ""
tags: [swift]

---

RxSwift의 가장 기본이 되는 ObservableType, ObserverType에 대해 코드 레벨에서 알아보자. 

## ObservableType 

- ObservableType은 Obsevable의 super class이다 
- ObservableType은 subscribe 메소드를 갖는다. 
- subscribe 메소드는 ObserverType을 인자로 리턴값은 ‘observer’의 subscription이며 이는 disposable하다. 


```swift
public protocol ObservableType : ObservableConvertibleType {
    //- returns: Subscription for `observer` that can be used to cancel production of sequence elements and free resources.
    func subscribe<O: ObserverType>(_ observer: O) -> Disposable where O.E == E
}

extension ObservableType {
   
    public func asObservable() -> Observable<E> {
        return Observable.create { o in
            return self.subscribe(o)
        }
    }
}


public class Observable<Element> : ObservableType {
    public typealias E = Element
    init() {
#if TRACE_RESOURCES
        let _ = Resources.incrementTotal()
#endif
    }
    public func subscribe<O: ObserverType>(_ observer: O) -> Disposable where O.E == E {
        rxAbstractMethod()
    }
    public func asObservable() -> Observable<E> {
        return self
    }
    
    deinit {
#if TRACE_RESOURCES
        let _ = Resources.decrementTotal()
#endif
    }
    internal func composeMap<R>(_ transform: @escaping (Element) throws -> R) -> Observable<R> {
        return _map(source: self, transform: transform)
    }
}
```

## Observer

- RxSwift의 모든 observer는 “ObserverType”을 상속받는다. 
- observer의 subclass에는 AnyObserver, AmbObserver, SinkForward, Binder 등이 있다. 
- on 메소드로 observer에 sequence event를 전달한다. 


```swift
public protocol     ObserverType {
    /// The type of elements in sequence that observer can observe.
    associatedtype E
    /// Notify observer about sequence event.
    ///
    /// - parameter event: Event that occurred.
    func on(_ event: Event<E>)
}

/// Convenience API extensions to provide alternate next, error, completed events
extension ObserverType {
    
    /// Convenience method equivalent to `on(.next(element: E))`
    ///
    /// - parameter element: Next element to send to observer(s)
    public func onNext(_ element: E) {
        on(.next(element))
    }
    
    /// Convenience method equivalent to `on(.completed)`
    public func onCompleted() {
        on(.completed)
    }
    
    /// Convenience method equivalent to `on(.error(Swift.Error))`
    /// - parameter error: Swift.Error to send to observer(s)
    public func onError(_ error: Swift.Error) {
        on(.error(error))
    }
}
```
