---
layout: post
title: "[ RX_SWIFT ] shared에 대해 알아보자."
description: "shared에 대해 알아보자."
tags: [swift]

---



Observable을 여러 observer에서 사용할 때 당연히 Observable 인스턴스는 하나의 공유된 인스턴스를 쓴다고 생각한다. 하지만 실제 코드를 동작시키다보면 예상과는 다르게 동작하는 경우가 있는데 그런 경우를 대비해 정확한 동작을 알아야 하는 것이 shared 메소드이다. 



## shared는 무엇인가?

ObservableType의 extension으로 하나의 subscription을 공유하는 Observable을 리턴해 준다. 



### Prototype

```swift
extension ObservableType {

    /**
         Returns an observable sequence that **shares a single subscription to the underlying sequence**, and immediately upon subscription replays  elements in buffer.
         
         This operator is equivalent to:
         * `.whileConnected`

         // Each connection will have it's own subject instance to store replay events.
         // Connections will be isolated from each another.
         source.multicast(makeSubject: { Replay.create(bufferSize: replay) }).refCount()

         * `.forever`

         // One subject will store replay events for all connections to source.
         // Connections won't be isolated from each another.
         source.multicast(Replay.create(bufferSize: replay)).refCount()

         
         It uses optimized versions of the operators for most common operations.
    
         - parameter replay: Maximum element count of the replay buffer.
         - parameter scope: Lifetime scope of sharing subject. For more information see `SubjectLifetimeScope` enum.
    
         - seealso: [shareReplay operator on reactivex.io](http://reactivex.io/documentation/operators/replay.html)
    
         - returns: An observable sequence that contains the elements of a sequence produced by multicasting the source sequence.
         */
    public func share(replay: Int = default, scope: RxSwift.SubjectLifetimeScope = default) -> RxSwift.Observable<Self.E>
}
```



정의만 봐서는 무슨 말인지 모르겠다. 예제를 만들어서 테스트해보자. 

예제는  UIViewController에 1개의 TextField가 있고 여기에 값을 변경할 때마다 Network로 request를 하는 것이다.

​```swift
        let ob:Observable<String> = usernameTextField.rx.text.orEmpty
            .map { text in 
                //request username update
                // 
                print("Update username \(text)")
                return text
            }
                
        _ = ob.subscribe{
            print("observer 1 \($0)") 
        }
        
        _ = ob.subscribe{
            print("observer 2 \($0)")
        }

```

ob.subscribe를 통해 network에 요청이 발생 후 observer를 추가했다. 

위 코드에 따라 usernameTextField의 값이 바뀔때마다 network 요청이 발생할 것이고 "oberver 1", "observer 2"가 출력될 것을 예상할 것이다. 프로그램을 실행 후 usernameTextField에 "rxswift"를 입력하고 출력을 확인해보자. 

```
Update username r
observer 1 next(r)
Update username r
observer 2 next(r)
Update username rx
observer 1 next(rx)
Update username rx
observer 2 next(rx)
Update username rxs
observer 1 next(rxs)
Update username rxs
observer 2 next(rxs)
Update username rxsw
observer 1 next(rxsw)
Update username rxsw
observer 2 next(rxsw)
Update username rxswi
observer 1 next(rxswi)
Update username rxswi
observer 2 next(rxswi)
Update username rxswif
observer 1 next(rxswif)
Update username rxswif
observer 2 next(rxswif)
Update username rxswift
observer 1 next(rxswift)
Update username rxswift
observer 2 next(rxswift)
```

위의 결과대로라면 TextField의 값을 바꿀때마다 2번의 network request가 호출된다. 이것은 의도한 결과가 아니다. 

그렇다면 shared 메소드를 사용해보자. 



shared Observable인 경우 

```swift
        let ob:Observable<String> = usernameTextField.rx.text.orEmpty
            .map { text in 
                //request username update
                // ...
                print("Update username \(text)")
                return text
            }.shared() 
                
        _ = ob.subscribe{
            print("observer 1 \($0)")
        }
        
        _ = ob.subscribe{
            print("observer 2 \($0)")
        }


```



shared를 적용한 후 아래와 같은 실행결과를 얻을 수 있다. 원래 의도했던 동작이다. 

```
Update username r
observer 1 next(r)
observer 2 next(r)
Update username rx
observer 1 next(rx)
observer 2 next(rx)
Update username rxs
observer 1 next(rxs)
observer 2 next(rxs)
Update username rxsw
observer 1 next(rxsw)
observer 2 next(rxsw)
Update username rxswi
observer 1 next(rxswi)
observer 2 next(rxswi)
Update username rxswif
observer 1 next(rxswif)
observer 2 next(rxswif)
Update username rxswift
observer 1 next(rxswift)
observer 2 next(rxswift)
```



참고로 shared와 동일한 동작을 하지만 connectable한 Observable을 반환하는 메소드는 publish이다. 

[참고사이트](https://medium.com/@_achou/rxswift-share-vs-replay-vs-sharereplay-bea99ac42168)



