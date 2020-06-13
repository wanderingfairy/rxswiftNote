# rxswiftNote
---
이 레포는 RxSwift 공부를 하며 정리한 내용입니다.



# RxSwift 정리

목차

- [1. Observable & Observer](#1-observable--observer)
- [2. Disposables](#2-disposables)
  - [메모리 정리](#%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%A0%95%EB%A6%AC)
- [3. Operators](#3-operators)
- [4. Subject & Relay](#4-subject--relay)
- [5. PublishSubject](#5-publishsubject)
- [**6. BehaviorSubject**](#6-behaviorsubject)
- [7. ReplaySubject](#7-replaysubject)
- [8. AsyncSubject](#8-asyncsubject)
- [9. Relays](#9-relays)
- [10. Create Operators](#10-create-operators)
  - [Of](#of)
  - [From](#from)
  - [repeatElement](#repeatelement)
  - [deferred](#deferred)
  - [create](#create)
    - [error](#error)
- [11.Filtering Operators](#11filtering-operators)
  - [**ignoreElements**](#ignoreelements)
  - [elementAt](#elementat)
  - [filter](#filter)
  - [skip, skipWhile, skipeUntil](#skip-skipwhile-skipeuntil)
    - [skip](#skip)
    - [skipWhile](#skipwhile)
    - [skipUntil](#skipuntil)
  - [take, takeWhile, takeUntil, takeLast](#take-takewhile-takeuntil-takelast)
    - [take](#take)
    - [takeWhile](#takewhile)
    - [takeUntil](#takeuntil)
    - [takeLast](#takelast)
  - [single Operator](#single-operator)
  - [distinctUntilChanged](#distinctuntilchanged)
  - [debounce, throttle](#debounce-throttle)
    - [debounce](#debounce)
    - [throttle](#throttle)
    - [throttle 연산자의 두 번째 파라미터에 대하여](#throttle-%EC%97%B0%EC%82%B0%EC%9E%90%EC%9D%98-%EB%91%90-%EB%B2%88%EC%A7%B8-%ED%8C%8C%EB%9D%BC%EB%AF%B8%ED%84%B0%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC)
- [12.Transforming Operators](#12transforming-operators)
  - [toArray Operator](#toarray-operator)
  - [map](#map)
  - [flatMap](#flatmap)
  - [flatMapFirst](#flatmapfirst)
  - [flatMapLatest](#flatmaplatest)
  - [scan](#scan)
  - [buffer](#buffer)
  - [window](#window)
  - [groupBy](#groupby)
- [13. Combining Operators](#13-combining-operators)
  - [startWith](#startwith)
  - [concat](#concat)
  - [merge](#merge)
  - [combineLatest](#combinelatest)
  - [zip](#zip)
  - [withLatestFrom](#withlatestfrom)
  - [sample](#sample)
  - [switchLatest](#switchlatest)
  - [reduce](#reduce)
- [14. Conditional Operators](#14-conditional-operators)
  - [amb](#amb)
- [15. Time-based Operators](#15-time-based-operators)
  - [interval](#interval)
  - [timer](#timer)
  - [timeout](#timeout)
  - [delay](#delay)
  - [delaySubscription](#delaysubscription)
- [16. Sharing Subscription](#16-sharing-subscription)
  - [multicast Operator](#multicast-operator)
    - [connect Method](#connect-method)
  - [publish](#publish)
  - [replay](#replay)
  - [refCount](#refcount)
  - [share](#share)
- [17.Scheduler](#17scheduler)
  - [Scheduler의 사용](#scheduler%EC%9D%98-%EC%82%AC%EC%9A%A9)
- [18. Error Handling](#18-error-handling)
  - [catchError](#catcherror)
  - [catchErrorJustReturn](#catcherrorjustreturn)
  - [retry, retryWhen](#retry-retrywhen)
    - [retry](#retry)
    - [retryWhen](#retrywhen)
- [19. RxCocoa](#19-rxcocoa)
  - [UIButton + Rx](#uibutton--rx)
  - [UILable + Rx](#uilable--rx)
  - [Cocoa --> RxCocoa](#cocoa----rxcocoa)
  - [Binding](#binding)
    - [Binding 구현](#binding-%EA%B5%AC%ED%98%84)

---

## 1. Observable & Observer

Observable은 Event를 전달한다. 이 이벤트는 옵저버로 전달되고, 옵저버는 옵저버블에서 전달되는 이벤트를 처리한다. 이것을 구독한다고 표현한다. 그래서 옵저버를 구독자라고 부르기도 한다.

옵저버가 구독을 시작하는 방법은, 옵저버블에서 subscribe method를 호출하는 것이다.

subscribe method 역시 subscribe 연산자(operator)라고 부르기도 한다.

subscribe method는 observer와 obervable을 연결한다. 두 요소를 연결해야 이벤트가 전달되므로, rxswift에서 가장 기초적이고 필수적인 요소이다.


```swift
let o1 = Observable<Int>.create { (observer) -> Disposable in
  observer.on(.next(0))
  observer.onNext(1)
  
  observer.onCompleted()
  
  return Disposables.create()
}

o1.subscribe { //값은 $0.element 속성을 통해 접근할 수 있음. 옵셔널이기 때문에 처리해줘야함.
  //이 메소드는 클로저를 파라미터로 받는다.
  //이 클로저로 이벤트가 전달되고, 여기에서 이벤트를 직접 처리한다.
  //바로 이 클로저, 이 부분이 observer.
  print("== Start ==")
  print($0)
  
  if let elem = $0.element {
    print(elem)
    //실제 값 꺼내기 (optional 처리)
  }
  print("== End ==")
}

/* 실행값
== Start ==
next(0)
0
== End ==
== Start ==
next(1)
1
== End ==
== Start ==
completed
== End ==
*/
```


그 외에, 이 메소드를 사용하는 방법도 있다. 

![스크린샷 2020-06-03 오후 9.00.31](https://tva1.sinaimg.cn/large/007S8ZIlgy1gffcpahj1qj30cj03g74v.jpg)

observable이 전달한 Event를 하나의 클로저 안에서 모두 처리해야했던 위의 방법과 달리,

개별 Event를 별도의 클로저에서 처리하고 싶을 때 사용한다. 

파라미터 기본값은 nil로 선언되어 있어서, 사용하지 않는 클로저는 삭제해도 된다.

```swift
o1.subscribe(onNext: { elem in
  //next event의 값이 클로저 파라미터인 elem으로 바로 전달된다. 때문에 element 속성에 접근할 필요가 없다. 
  print("== Start ==")
  print(elem)
  print("== End ==")
})

/*실행값
== Start ==
0
== End ==
== Start ==
1
== End ==
*/
```



Observable은 이벤트가 어떤 순서로 전달돼야 하는지 정의할 뿐, 이 시점에 실제로 이벤트가 전달되거나 값이 방출되는 게 아님

방출과 이벤트 전달은 Observer가 Obervable을 구독하기 시작하는 시점에 이뤄짐.

Observer는 동시에 두 개 이상의 이벤트를 처리하지 않음.

Observable은 observer가 하나의 이벤트를 처리하면 그 다음 이벤트를 순차적으로 전달함.



---

## 2. Disposables

![스크린샷 2020-06-03 오후 9.00.31](https://tva1.sinaimg.cn/large/007S8ZIlgy1gffcpahj1qj30cj03g74v.jpg)

이 메소드에서, onDisposed는 옵져버블과 관련된 모든 리소스가 제거된 후 호출된다.

 메모리에서 해제되는 순간에 onDisposed 클로저 안의 무언가를 실행시키고 싶다면, 다음과 같이 사용해야한다.

```swift
let subscription1 = Observable.from([1, 2, 3])
  .subscribe(onNext: { elem in
    print("Next", elem)
  }, onError: { (error) in
    print("Error", error)
  }, onCompleted: {
    print("Completed")
  }, onDisposed: {
    print("Disposed") //이 부분에 작성
  })

/*출력값
Next 1
Next 2
Next 3
Completed
Disposed
*/
```



아래처럼 하면 Disposed가 안 나옴.

```swift
Observable.from([1, 2, 3])
  .subscribe {
    print($0)
}
/*출력값
next(1)
next(2)
next(3)
completed
*/
```



그렇다면 이 때에는 Observable이 정상적으로 메모리에서 해제되지 않았을까?

아니다. Observable이 error나 completed event로 종료되었을 경우에는 관련된 리소스가 메모리에서 정상적으로 해제된다. 

그럼 왜 두번째에서는 disposed가 출력되지 않았냐면

onDisposed는 Observable이 호출하는 함수가 아니라, 해당 리소스가 모두 해제되면 자동으로 호출될 뿐이기 때문이다.



### 메모리 정리

Observable이 error or completed event를 호출하고 종료되면,

따로 리소스 관리를 해주지 않아도 자동으로 메모리에서 해제되지만,

RxSwift의 가이드라인에 따르면, 이 경우에도 리소스 정리를 해주라고 나와있다.

공식적인 제안이기 때문에, 이를 따르기 위해 수동으로 메모리 정리를 해주는데, 이 경우에 사용하는 것이 바로 Disposable이다.



![스크린샷 2020-06-03 오후 9.21.38](https://tva1.sinaimg.cn/large/007S8ZIlgy1gffdbaetafj30dx0do12i.jpg)



subscribe의 레퍼런스를 보면, 리턴형이 Disposable이다.

이 메서드가 리턴하는 Disposable을 subsciption disposable이라고도 한다.

이는 리소스 해제와 실행 취소에 사용된다.

1. 리소스 해제에 사용되는 경우

   1. dispose() method 직접 호출

      ```swift
      let subscription1 = Observable.from([1, 2, 3])
        .subscribe(onNext: { elem in
          print("Next", elem)
        }, onError: { (error) in
          print("Error", error)
        }, onCompleted: {
          print("Completed")
        }, onDisposed: {
          print("Disposed")
        })
      
      subscription1.dispose() //dispose()를 호출함으로써 리소스를 메모리에서 해제시킬 수 있음.
      ```

   2. DisposedBag 사용.(공식문서 권장) 

      ```swift
      var bag = DisposeBag() //여기에 disposable을 담았다가, 한 번에 해제할 수 있다.
      //담을 때는 disposed(by: )메소드를 사용하면 된다.
      
      Observable.from([1, 2, 3])
        .subscribe {
          print($0)
      }
      .disposed(by: bag)
      ```

      이렇게 bag을 만들어서 .disposed(by: bag)을 해주면,

      subscribe가 리턴한 Disposable이 해당 bag에 추가됨.

      이때 bag에 추가된 Disposable은 disposeBag이 해제되는 시점에 함께 해제됨.

      ARC에서의 오토릴리즈풀과 비슷한 개념으로 생각하면 됨. 

      이렇게 bag에 담긴 disposable들을, disposeBag이 해제되는 시점까지 기다리지 않고 즉시 해제시키고 싶다면,

      ```swift
      bag = DisposeBag() 
      //이런식으로 bag을 또다른 DisposeBag으로 초기화하면 그 이전까지 bag에 담겨 있던 Disposable들은 모두 해제되고 새로운 DisposeBag()으로 초기화 되는 것임.
      ```

2. Subscription Disposable을 실행 취소에 사용하는 방법.

   ```swift
   let subscription2 = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
     .subscribe(onNext: { elem in
       print("Next", elem)
     }, onError: { (error) in
       print("Error", error)
     }, onCompleted: {
       print("Completed")
     }, onDisposed: {
       print("Disposed")
     })
   //1씩 증가하는 정수를 1초마다 방출하는 Observable.
   //무한정 방출하기 때문에 중단시킬 방법이 필요함.
   ```

   여기에 사용할 수 있는 게 dispose() 메소드임.

   ```swift
   DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
     subscription2.dispose() //이 시점에서 dispose() 호출 
   }
   
   /* 실행값 
   Next 0
   Next 1
   Next 2
   Disposed
   */
   ```

   이렇게 3초 후에 메모리에서 해제시킴으로써 이후의 실행을 취소시킴.

   dispose 메소드를 호출하는 즉시 모든 리소스가 메모리에서 해제되기 때문에,

   Next 까지만 호출되고, Completed 메서드는 호출되지 않고 dispose 돼버림.

   이런 이유로 dispose 메서드를 직접 호출하는 것은 가급적 피해야함.

   대신 Take, Until 등의 연산자를 활용해서 구현할 수 있음.



---

## 3. Operators

![스크린샷 2020-06-03 오후 9.36.40](https://tva1.sinaimg.cn/large/007S8ZIlgy1gffdqwcp7pj30he05mwf7.jpg)

RxSwift가 제공하는 여러 타입 중에, ObservableType 프로토콜이 있다.

여기에는 RxSwift의 근간을 이루는 다양한 메소드가 선언되어 있다.

새로운 Observable을 생성하는 메소드도 있고, 방출되는 요소를 필터링하거나, 여러 Observable을 하나로 합치는 메소드들도 있다.



RxSwift에서는 이런 메소드들을 연산자라고 부른다.

연산자는 몇 가지 특징을 가지고 있다. 대부분의 연산자는 Observable 상에서 동작하고, 새로운 Observable을 리턴한다. 

Observable을 리턴하기 때문에, 두 개 이상의 연산자를 연달아 호출할 수 있다.

```swift
let bag = DisposeBag()

Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9])

   .subscribe { print($0) }
   .disposed(by: bag)
```



위의 코드는, 1에서 9까지의 숫자를 연속해서 방출한다.

여기에 연산자를 추가해보자. 연산자는 보통 subscribe메소드 앞에 추가한다. 그래야 구독자로 전달되는 최종 데이터가 내가 원하는 데이터일 수 있기 때문.



```swift
let bag = DisposeBag()

Observable.from([1, 2, 3, 4, 5, 6, 7, 8, 9])
   .take(5) 
//take 연산자는 source Observable이 방출하는 요소 중에서, 파라미터로 지정한 수만큼 방출하는 새로운 Observable을 리턴한다.
//말하자면 처음 5개의 요소만 방출한다.
  .filter { $0.isMultiple(of: 2) }
//filter 연산자는 조건에 해당하는 요소만 방출하는 Observable을 리턴한다.
   .subscribe { print($0) }
   .disposed(by: bag)

/*실행값
next(2)
next(4)
completed
*/
```

이 코드를 실행하면, 구독자에게 2와 4가 전달된다.

take 연산자는 처음 5개의 요소를 다음 연산자로 전달하고,

filter 연산자는 여기에서 짝수만 다음 연산자로 전달한다.

그래서 1과 5 사이에 존재하는 짝수인 2와 4만 subscribe로 전달되었다. 

연산자는 필요한만큼 얼마든지 연결할 수 있다. 하지만 호출 순서에 주의해야한다.

take 연산자와 filter 연산자의 순서를 바꿔 실행하면

2,4,6,8이 출력된다. 짝수 요소를 먼저 필터링 하고(2,4,6,8)

그 중 첫 5개의 요소를 take가 리턴하기 때문에 이전과 전혀 다른 값이 나왔다.



정리 - 연산자는 새로운 옵져버블을 리턴하기 때문에 두 개 이상 연달아 호출할 수 있지만, 호출 순서에 따라 다른 결과가 나오기 때문에 항상 호출 순서에 주의해야 한다.



---

## 4. Subject & Relay

Subject를 이해하기 위해서는 observable과 observer를 이해해야한다.

Observable은 event를 전달한다.

observer는 observable을 구독하고 전달되는 event를 처리한다. 

observable은 observer와 달리 다른 옵져버블을 구독하지 못한다.

마찬가지로 옵져버는 다른 옵저버로 이벤트를 전달하지 못한다.


반면 Subject는 다른 옵져버블로부터 이벤트를 받아서 구독자로 전달할 수 있다.

Subject는 옵저버블인 동시에 옵저버이다.


RxSwift는 4가지 Subject를 제공한다.

가장 기본적인 PublishSubject는 Subject로 전달되는 새로운 이벤트를 구독자로 전달한다.

BehaviorSubject는 생성시점의 시작이벤트를 지정한다. 그리고 subject로 전달되는 이벤트 중에서 가장 마지막으로 전달된 최신 이벤트를 저장해두었다가, 새로운 구독자에게 최신 이벤트를 전달한다.

ReplaySubject는 하나 이상의 최신 이벤트를 버퍼에 저장한다. 옵저버가 구독을 시작하면, 버퍼에 있는 모든 이벤트를 전달한다.

마지막 AsyncSubject는 Subject로 completed이벤트가 전달되는 시점에 마지막으로 전달된 Next event를 구독자로 전달한다. 



또한 RxSwift는 Subject를 래핑하고 있는 두 가지 Relay를 제공한다.

이전 버전에서 제공하던 variable이 Relay로 대체되었다.

PublishRelay는 PublishSubject를 래핑한 것이고, 

BehaviorRelay는 BehaviorSubject를 래핑한 것이다.

Relay는 일반적인 Subject와 달리, 

Next Event만 받고 나머지 -Completed와 Error event는 받지 않는다.

주로 종료 없이 계속 전달되는 Event 시퀀스를 처리할 때 활용한다.



---

## 5. PublishSubject

`PublishSubject`는 subject로 전달된 Event를 Observer로 전달하는 가장 기본적인 형태의 `Subject` 입니다.

```swift
let subject = PublishSubject<String>()
```

여기에서는 타입 파라미터를 `String`으로 선언하고 있다. 이렇게 하면 문자열이 포함된 Next Event를 받아서 다른 Observer에게 전달할 수 있다.

생성자를 호출할 때에는 파라미터를 전달하지 않는다. 

이 Subject는 비어있는 상태로 생성된다. 다시 말하면 Subject가 생성되는 시점에는 내부에 아무런 이벤트도 저장되어있지 않다. 그래서 생성 직후에 Observer가 구독을 시작하면 아무 이벤트도 전달되지 않는다. 



Subject는 Observable인 동시에 Observer이다. 다른 소스로부터 이벤트를 전달 받을 수 있고, 다른 Observer로 이벤트를 전달할 수 있다. 

Observable의 경우 Observer로 Next event를 전달하기 위해Observer에서 onNext 메소드를 호출하고 파라미터로 요소를 전달했었는데, 



Subject 역시 Observer이기 때문에 onNext 메소드를 호출할 수 있다.

 

```swift
let subject = PublishSubject<String>()

subject.onNext("Hello")
```



이렇게 하면 subject로 next("Hello") 이벤트가 전달된다.

현재는 subject를 구독하는 Observer가 없기 때문에 이 이벤트는 방출되지 않고 사라진다. 

(Observable과 마찬가지로 subject가 방출하기로 한 요소들 역시 Observer가 해당 Subject를 '구독하기 시작해야만' 방출(전달)이 시작된다.)

이 코드에서 subject는 `Observable`이고, 다른 `Observer`가 이것을 구독할 수 있다. 



```swift
let o1 = subject.subscribe { print(">> 1", $0)}
o1.disposed(by: disposeBag)
//실행시 출력값 없음
```



`PublishSubject`는 구독 이후에 전달되는 새로운 이벤트만 구독자에 전달한다.

그래서 구독자가 구독을 시작하기 전에 방출되었던 Next event "Hello"는 `o1`옵저버로 전달되지 않는다.

다시 여기에서 `PublishSubject`로 Next Event를 전달해보자.

```swift
let o1 = subject.subscribe { print(">> 1", $0)}
o1.disposed(by: disposeBag)

subject.onNext("RxSwift")
//실행값
//>> 1 next(RxSwift)
```

그러면 "RxSwift"를 담은 Next event가 subject로 전달되고, subject는 이 이벤트를 구독자로 전달한다. 그래서 출력값 **>> 1 next(RxSwift)**가 나온다.



이 코드 아래에 새로운 구독자를 추가해보자.

```swift
...(전략)

let o2 = subject.subscribe { print(">> 2", $0) }
o2.disposed(by: disposeBag)
```

o2 옵저버는 두 개의 next event가 전달되고 난 이후에 구독을 시작했기 때문에, 이 시점에는 아무런 이벤트도 전달받지 않는다. 다시 subject로 next event를 전달하면

```swift
let o1 = subject.subscribe { print(">> 1", $0)}
o1.disposed(by: disposeBag)

subject.onNext("RxSwift")

let o2 = subject.subscribe { print(">> 2", $0) }
o2.disposed(by: disposeBag)

subject.onNext("Subject")
/*
실행값
>> 1 next(RxSwift)
>> 1 next(Subject)
>> 2 next(Subject)
*/
```

두 구독자에게 next event인 "Subject"가 모두 전달된다. 



이 subject에 onCompleted() 메소드를 호출하면



```swift
...(전략)

subject.onCompleted()

/*
실행값
>> 1 next(RxSwift)
>> 1 next(Subject)
>> 2 next(Subject)
>> 1 completed
>> 2 completed
*/
```



이렇듯 두 구독자 `o1`, `o2` 모두에게 `Completed` 이벤트가 전달된다. 



Subject가 완료된 이후에 새로운 구독자를 추가한다면 어떤 결과가 나올까?

 

```swift
...(전략)

let o3 = subject.subscribe { print(">> 3", $0) }
o3.disposed(by: disposeBag)
/*
실행값
>> 1 next(RxSwift)
>> 1 next(Subject)
>> 2 next(Subject)
>> 1 completed
>> 2 completed
>> 3 completed 
--> o3에 대해 onCompleted()를 호출하지 않았는데도,
바로 컴플리티드 이벤트가 전달됨.
*/
```



새로운 구독자 `o3`에게는 completed event가 바로 전달된다. 옵저버블에서 completed 이벤트가 전달된 이후에는 더 이상 next event가 전달되지 않는다. 이건 subject도 마찬가지이다. 

새로운 구독자에게 전달할 next event가 없기 때문에 바로 completed 이벤트를 전달하여 종료하는 것이다. 



마지막으로,  error이벤트를 전달하여 종료된 상태의 subject에 구독자를 추가하면 어떻게 될까?

```swift
let disposeBag = DisposeBag()

enum MyError: Error {
   case error
}

let subject = PublishSubject<String>()

subject.onNext("Hello")

let o1 = subject.subscribe { print(">> 1", $0)}
o1.disposed(by: disposeBag)

subject.onNext("RxSwift")

let o2 = subject.subscribe { print(">> 2", $0) }
o2.disposed(by: disposeBag)

subject.onNext("Subject")

//subject.onCompleted()
subject.onError(MyError.error)

let o3 = subject.subscribe { print(">> 3", $0) }
o3.disposed(by: disposeBag)
/*
실행값
>> 1 next(RxSwift)
>> 1 next(Subject)
>> 2 next(Subject)
>> 1 error(error)
>> 2 error(error)
>> 3 error(error)
*/
```

마찬가지로 바로 error이벤트가 전달된다. 



`PublishSubject`는 이벤트가 전달되면, 즉시 구독자에게 전달한다. 

그래서 Subject가 최초로 생성되는 시점부터 첫번째 구독이 시작되는 시점 사이에 전달(방출)된 이벤트는 그냥 사라진다. 



이것이 `PublishSubject`의 큰 특징이다.

특정 시점의 이벤트가 사라지는 것이 문제가 된다면, Replay Subject를 사용하거나, Cold Observable을 사용해야 한다. 



---

## **6. BehaviorSubject**

`BehaviorSubject`는 PublishSubject와 유사한 방식으로 동작한다. Subject로 전달된 이벤트를 구독자에게 전달하는 것은 동일하다. 하지만 Subject를 생성하는 방식에 차이가 있다. 



PublishSubject와 BehaviorSubject를 하나씩 생성해보자.



```swift
let p = PublishSubject<Int>()

let b = BehaviorSubject<Int>(value: 0)
```



PublishSubject는 빈값으로 생성할 수 있지만, BehaviorSubject를 생성할 때에는 하나의 값을 전달해야 한다. 



또한 구독하는 방식에 차이가 있다. 



```swift
let p = PublishSubject<Int>()

p.subscribe { print("PublishSubject >>", $0) }
  .disposed(by: disposeBag)

```



PublishSubject는 내부에 이벤트가 저장되지 않은 상태로 생성된다. 그래서 subject로 이벤트가 전달되기 전까지, 구독자로 이벤트가 전달되지 않는다.



```swift
let b = BehaviorSubject<Int>(value: 0)

b.subscribe { print("BehaviorSubject >>", $0) }
.disposed(by: disposeBag)

//출력값 
//BehaviorSubject >> next(0)
```



구독하자마자 next Event가 전달되어온다. 해당 값은 BehaviorSubject를 생성할 때 저장한 값이다. 

BehaviorSubject를 생성하면, 내부에 next Event가 하나 만들어지는 것이다. 여기에는 생성자로 전달한 값이 저장된다. 새로운 구독자가 추가되면, 저장되어있는 next Event가 바로 전달된다. 



```swift
...(전략)

b.onNext(1)

//출력값 
//BehaviorSubject >> next(0)
//BehaviorSubject >> next(1)
```



이렇게 새로운 event를 전달하면 총 두 개의 이벤트가 observer로 전달된다. 



이 시점에서 새로운 옵져버가 추가되면



```swift
let b = BehaviorSubject<Int>(value: 0)

b.subscribe { print("BehaviorSubject >>", $0) }
.disposed(by: disposeBag)

b.onNext(1)

b.subscribe { print("BehaviorSubject2 >>", $0) }
.disposed(by: disposeBag) //새로 추가된 구독자

/* 실행값
BehaviorSubject >> next(0)
BehaviorSubject >> next(1)
BehaviorSubject2 >> next(1) //두번째 구독자의 next 이벤트에 1이 저장되어있다.
*/
```



BehaviorSubject는 생성시점에 만들어진 next event를 저장하고 있다가 새로운 observer로 전달한다. 이후 subject로 새로운 next event가 전달되면 (이 경우 1) 기존에 저장되어있던 이벤트를 교체한다. 

결과적으로 가장 최신 next event를 옵저버로 전달한다.  



이후 Completed 이벤트를 전달하고, 새로운 구독자를 추가할 경우에도 저장되어있는 next event가 전달될까?



```swift
...(전략)

b.onCompleted()

b.subscribe { print("BehaviorSubject3 >>", $0) }
.disposed(by: disposeBag)

/* 실행값
BehaviorSubject >> next(0)
BehaviorSubject >> next(1)
BehaviorSubject2 >> next(1)
BehaviorSubject >> completed
BehaviorSubject2 >> completed
BehaviorSubject3 >> completed //completed 함수 호출 이후에 추가된 구독자에게는 더 이상 저장되어있던 next Event가 전달되지 않는다
*/
```



위와 같이 completed 함수 호출 이후에 추가된 구독자에게는 더 이상 저장되어있던 next Event가 전달되지 않는다



Error이벤트의 경우에도 completed 이벤트의 경우와 같다. 



---

## 7. ReplaySubject

Behavior Subject의 경우 최신 event 하나를 저장하고 있다가 새로운 구독자에게 전달한다. 그래서 그보다 이전의 event들은 모두 사라진다. 두 개 이상의 이벤트를 저장하여 새로운 구독자에게 전달하고 싶다면 ReplaySubject를 사용한다. 



```swift
let rs = ReplaySubject<Int>.create(bufferSize: 3)
```



PublishSubject나 BehaviorSubject와 달리 create 메소드를 이용해 생성하고, bufferSize를 지정해주어야한다. 3을 값으로 주면 3개의 이벤트를 저장하는 버퍼를 가지게 된다. 



이 ReplaySubject에 1부터 10까지의 next Event를 전달한 후에 구독해보면



```swift
let rs = ReplaySubject<Int>.create(bufferSize: 3)

(1...10).forEach { rs.onNext($0) }
rs.subscribe { print("Observer 1 >>", $0)}
.disposed(by: disposeBag)

/* 실행값
Observer 1 >> next(8)
Observer 1 >> next(9)
Observer 1 >> next(10)
*/
```



세 개의 next 이벤트가 구독자로 전달되었다. 결과가 이렇게 나온 이유는 버퍼의 크기를 3으로 지정했기 때문이다.

Buffer에는 size만큼의 마지막 event 들이 저장된다.

여기에 새로운 구독자를 추가하면



```swift
rs.subscribe { print("Observer 1 >>", $0)}
.disposed(by: disposeBag)

rs.subscribe { print("Observer 2 >>", $0)}
.disposed(by: disposeBag)

/*실행값
Observer 1 >> next(8)
Observer 1 >> next(9)
Observer 1 >> next(10)
Observer 2 >> next(8)
Observer 2 >> next(9)
Observer 2 >> next(10)
*/
```



동일한 값을 가진 next Event들이 모든 구독자들에게 전달된다. 



이후 여기에 새로운 이벤트를 전달해보면



```swift
rs.subscribe { print("Observer 1 >>", $0)}
.disposed(by: disposeBag)

rs.subscribe { print("Observer 2 >>", $0)}
.disposed(by: disposeBag)

rs.onNext(11)

/*실행값
Observer 1 >> next(8)
Observer 1 >> next(9)
Observer 1 >> next(10)
Observer 2 >> next(8)
Observer 2 >> next(9)
Observer 2 >> next(10)
Observer 1 >> next(11)
Observer 2 >> next(11)
*/
```



즉시 구독자에게 전달된다. 이 부분은 다른 subject와 동일하다. 

그리고 버퍼에 새로운 next event(11)가 저장되고, 가장 오래된 이벤트(8)이 버퍼에서 삭제된다.

```swift
let rs = ReplaySubject<Int>.create(bufferSize: 3)
(1...10).forEach { rs.onNext($0) }

rs.subscribe { print("Observer 1 >>", $0)}
.disposed(by: disposeBag)

rs.subscribe { print("Observer 2 >>", $0)}
.disposed(by: disposeBag)

rs.onNext(11)

rs.subscribe { print("Observer 3 >>", $0)}
.disposed(by: disposeBag)

/*실행값
Observer 1 >> next(8)
Observer 1 >> next(9)
Observer 1 >> next(10)
Observer 2 >> next(8)
Observer 2 >> next(9)
Observer 2 >> next(10)
Observer 1 >> next(11)
Observer 2 >> next(11)
Observer 3 >> next(9)
Observer 3 >> next(10)
Observer 3 >> next(11)
*/
```



마지막 구독자에게는 9, 10, 11 세 개의 이벤트만 전달된다. 

버퍼는 메모리에 저장되기 때문에 항상 메모리 사용량에 신경써야한다. 필요 이상의 큰 버퍼를 사용하는 것은 피해야한다.



이 상태에서 completed 이벤트를 전달하면 모든 구독자에게 즉시 completed 이벤트가 전달된다. 그 이후에 새로운 구독자를 추가하면, 버퍼에 저장된 값들이 next event로 해당 구독자에게 전달된 이후에 completed 이벤트가 전달된다.



```swift
...(전략)
rs.onCompleted()

rs.subscribe { print("Observer 4 >>", $0)}
.disposed(by: disposeBag)

/*실행값
Observer 1 >> completed
Observer 2 >> completed
Observer 3 >> completed
Observer 4 >> next(9)
Observer 4 >> next(10)
Observer 4 >> next(11)
Observer 4 >> completed
*/
```



error의 경우에도 마찬가지다.

ReplaySubject의 경우 종료 여부에 관계 없이 항상 버퍼에 저장되어있는 이벤트를 새로운 구독자에게 전달한다.



---

## 8. AsyncSubject

`AsyncSubject`는 다른 subject와 이벤트를 전달하는 시점에 차이가 있다. 

Publish, Behavior, Replay Subject는 Subject로 이벤트가 전달되면 즉시 구독자에게 전달한다.

 반면 AsyncSubject는 Subject로 Completed Event가 전달되기 전까지 어떤 이벤트도 구독자로 전달하지 않는다.

Completed 이벤트가 전달되면, 그 시점에서 가장 최근에 전달된 next event 하나를 구독자에게 전달한다. 

```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}
//여기까지 기본 세팅

let subject = AsyncSubject<Int>() //AsyncSubject 생성

subject
  .subscribe { print($0) }
  .disposed(by: bag)

subject.onNext(1)

//실행값 없음
```



이 시점엔 아직 subject로 completed 이벤트가 전달되지 않았기 때문에 아무런 이벤트도 구독자에게 전달되지 않는다. 

next Event 2개를 더 전달하고, 이어서 onCompleted() 메소드를 호출해보자.

```swift
let subject = AsyncSubject<Int>()

subject
  .subscribe { print($0) }
  .disposed(by: bag)

subject.onNext(1)

subject.onNext(2)
subject.onNext(3)

subject.onCompleted()

/*실행값
next(3)
completed
*/
```



이렇듯 completed 이벤트를 전달한 시점에서 가장 최근의 next event가 구독자로 전달되는 것을 확인할 수 있다. 

이어 completed 이벤트가 전달되고, 구독이 종료된다.

만약 AsyncSubject로 전달된 최근 next 이벤트가 없다면 그냥 completed 이벤트만 전달하고 종료된다. 



```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let subject = AsyncSubject<Int>()

subject
  .subscribe { print($0) }
  .disposed(by: bag)

subject.onNext(1)

subject.onNext(2)
subject.onNext(3)

subject.onError(MyError.error) //에러 이벤트 전달

/*실행값
error(error)
*/
```



하지만 Completed 이벤트 대신 Error 이벤트를 전달할 경우 최근 next event가 전달되지 않고, error 이벤트만 전달되고 종료된다. 

---

## 9. Relays


RxSwift는 두 가지 Relay를 제공한다. Relay는 Subject와 유사한 특징을 가지고 있고, 내부에 Subject를 래핑하고 있다. 

PublishRelay는 PublishSubject를, BehaviorRelay는 BehaviorSubject를 래핑하고 있다. 

Relay는 Subject와 마찬가지로 Source로부터 이벤트를 전달 받아 구독자로 전달한다. 하지만 가장 큰 차이는 오직 Next Event만 전달할 수 있다는 것이다. 

Completed 와 Error 이벤트는 전달 받지도, 전달 하지도 않는다. 그래서 Subject와 달리 구독자가 disposed 되기 전까지는 메모리에서 해제되지 않는다. 그래서 주로 UI Event 처리에 사용된다. 

Relay를 사용하기 위해서는 RxCocoa를 import 해야한다. 



1. PublishRelay

```swift
import UIKit
import RxSwift
import RxCocoa

let bag = DisposeBag()

let prelay = PublishRelay<Int>() //생성 
prelay.subscribe { print("1: \($0)") } //구독
  .disposed(by: bag)
```

빈 생성자로 생성한다는 점은  PublishSubject와 동일하다. 이어서 relay에 next event를 전달하기 위해서는 Observable이나 Subject의 onNext() 메소드가 아닌 accept() 메소드를 사용해야한다. 



```swift
let prelay = PublishRelay<Int>()
prelay.subscribe { print("1: \($0)") }
  .disposed(by: bag)

prelay.accept(1)
//실행값
//1: next(1)
```



2. BehaviorRelay

```swift
let brelay = BehaviorRelay(value: 1)
brelay.accept(2)

brelay.subscribe { print("2: \($0)") }
  .disposed(by: bag)

brelay.accept(3)
//실행값
//2: next(2)
//2: next(3)
```



생성시 하나의 이벤트를 저장해야한다는 점은 BehaviorSubject와 같다. 

이후 즉시 accept(2)를 통해 next event를 전달하면 내부에 저장된 next event의 값이 2로 바뀐다.

이후 구독하기 때문에 바뀌어 저장된 2라는 next event 값이 구독자로 전달된다. 

그 이후에 accept(3)을 통해 BehaviorRelay로 값을 전달하면 그 즉시 바뀐 값이 구독자로 전달된다. 

BehaviorRelay는 value라는 속성을 제공하는데, 이 속성은 relay가 저장하고 있는 next event에 접근해서, 여기에 저장된 값을 리턴한다. 

```swift
...(전략)

print(brelay.value)
//실행값
//3
```

이 속성은 읽기 전용이고, 저장된 값을 바꿀 수는 없다. 새로운 값으로 바꾸고 싶다면 accept 메소드를 통해 새로운 next 이벤트를 전달해야 한다. 

```swift
brelay.value = 4 //error
```



---

## 10. Create Operators

###Just

just는 하나의 항목을 방출하는 Observable을 생성한다. 

![스크린샷 2020-06-04 오후 8.31.06](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfghh21ozpj30rm08k0uv.jpg)

파라미터로 하나의 요소를 받아서 Obsevable을 리턴한다. 



```swift
let disposeBag = DisposeBag()
let element = "😀"

Observable.just(element)
//문자열 하나를 방출하는 옵저버블 생성
   .subscribe { event in print(event) }
   .disposed(by: disposeBag)

//출력값
//next(😀)
//completed
```



```swift
Observable.just([1, 2, 3]) //파라미터로 배열 전달
   .subscribe { event in print(event) }
   .disposed(by: disposeBag)
//출력값
//next([1, 2, 3])
//completed
```



파라미터로 배열을 준 뒤 결과를 보면, 배열 하나를 그대로 방출한다. 

from 오퍼레이터와 혼동할 수 있는데, just로 생성한 옵저버블은 파라미터로 전달한 요소를 그대로 전달한다.

---

### Of

만약 두 개 이상의 요소를 방출하는 옵저버블을 만들어야한다면, just로는 불가능하다. 이때 사용하는 연산자가 of이다.

![스크린샷 2020-06-04 오후 8.36.09](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfghm8v8zdj30wq0cw77p.jpg)

파라미터가 가변 파라미터로 설정되어 있어서 방출할 요소를 원하는 수만큼 전달할 수 있다. 



```swift
let disposeBag = DisposeBag()
let apple = "🍏"
let orange = "🍊"
let kiwi = "🥝"

Observable.of(apple, orange, kiwi)
   .subscribe { element in print(element) }
   .disposed(by: disposeBag)

/*실행값
next(🍏)
next(🍊)
next(🥝)
completed
*/
```



그래서 문자열을 담은 next event가 세 번 전달되고, 마지막에 컴플리티드 이벤트가 전달된다. 



```swift
Observable.of([1, 2], [3, 4], [5, 6])
   .subscribe { element in print(element) }
   .disposed(by: disposeBag)

/*실행값
next([1, 2])
next([3, 4])
next([5, 6])
completed
*/
```



위와 같이 세 개의 배열을 전달할 경우, 세 개의 배열이 연달아 방출된다. 



배열에 저장된 요소를 하나씩 방출하고 싶을 경우에는 from 연산자를 사용해야한다. 



---

### From



![스크린샷 2020-06-05 오전 3.02.30](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgss8jvzbj30yu07nac1.jpg)



이 연산자도 Observable protocol에 Type Method로 선언되어 있다.

첫 번째 파라미터로 배열을 받고, 리턴형은 배열 형식이 아닌 배열에 포함된 요소의 형식이다.

즉, 배열에 포함된 요소를 하나씩 순서대로 방출한다.

```swift
let disposeBag = DisposeBag()
let fruits = ["🍏", "🍎", "🍋", "🍓", "🍇"]

Observable.from(fruits)
   .subscribe { element in print(element) }
   .disposed(by: disposeBag)

/*실행값
next(🍏)
next(🍎)
next(🍋)
next(🍓)
next(🍇)
completed
*/
```



출력값을 보면, 배열에 저장된 요소들이 하나씩 방출된 것을 볼 수 있다.

---

하나의 요소를 방출하는 Observable을 생성할 때에는 just

두 개 이상의 요소를 방출해야한다면 of

배열에 저장된 요소를 하나씩 순서대로 방출하는 Observable이 필요하다면 from 연산자를 사용해야한다.

---

### repeatElement

동일한 요소를 반복적으로 방출하는 Observable을 생성할 때 사용한다.

![스크린샷 2020-06-05 오전 3.08.17](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgsy8s5aij30jl03rwfh.jpg)

첫 번째 파라미터로 요소를 전달하면, 해당 요소를 반복적으로 방출하는 Observable을 Return 한다.

반복적이라는 뜻은, '무한정' 반복한다는 뜻.

```swift
let disposeBag = DisposeBag()
let element = "❤️"

Observable.repeatElement(element)
  .subscribe { print($0) }
.disposed(by: disposeBag)

/*실행값
next(❤️)
next(❤️)
next(❤️)
next(❤️)
next(❤️)
...(후략)
*/
```



무한정 방출되기 때문에, 이 연산자를 사용할 때에는 방출되는 요소의 수를 제한해주는 것이 중요함. (take 등을 사용)

```swift
Observable.repeatElement(element)
  .take(7) //방출되는 요소 수 제한
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*실행값
next(❤️)
next(❤️)
next(❤️)
next(❤️)
next(❤️)
next(❤️)
next(❤️)
completed
*/
```



take 연산자로 방출되는 요소의 수를 제한해주면, 정해진 수만큼만 방출한 뒤 completed event를 방출한다. 

---

### deferred

이 연산자를 사용하면 특정 조건에 따라 Observable을 생성할 수 있다.

![스크린샷 2020-06-05 오전 3.14.15](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgt4gltb4j30gj03x0tj.jpg)

Observable을 리턴하는 클로저를 파라미터로 받는다. 



```swift
let disposeBag = DisposeBag()
let animals = ["🐶", "🐱", "🐹", "🐰", "🦊", "🐻", "🐯"]
let fruits = ["🍎", "🍐", "🍋", "🍇", "🍈", "🍓", "🍑"]
var flag = true

let factory: Observable<String> = Observable.deferred {
  flag.toggle()
  
  if flag {
    return Observable.from(animals)
  } else {
    return Observable.from(fruits)
  }
}

factory //#1
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/* 실행값
next(🍎)
next(🍐)
next(🍋)
next(🍇)
next(🍈)
next(🍓)
next(🍑)
completed
*/
```



위의 코드는, flag의 상태에 따라 방출하는 Observable이 바뀌는 코드이다.

구독자가 추가되는 시점(`#1`)에 deferred의 클로저 내부 코드가 실행된다.

따라서 하나의 구독자가 새롭게 추가될 때마다 다른 값을 방출하는 Observable이 완성되었다.

위의 코드에서

```swift
let factory: Observable<String>
```

이렇게 타입 annotation 을 선언해주지 않으면 타입 추론이 불가능하기 때문에 꼭 추가해줘야한다.

---

### create

다른 연산자들은 파라미터로 전달된 요소를 방출하는 Observable을 생성한다. 이렇게 생성된 Observable은 모든 요소를 방출한 뒤 마지막으로 Completed event를 전달하고 종료된다. 

이게 Observable의 기본 동작이기 때문에 이것을 조작할 수는 없다. 

Observable이 동작하는 방식 자체를 조작하고 싶다면 `create`연산자를 사용해야한다.

![스크린샷 2020-06-05 오전 3.29.50](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgtkpd7szj30c605mmxk.jpg)

create 연산자는 Observable을 파라미터로 받아서  Disposable을 리턴하는 클로저를 전달한다.



URL에서 html을 다운로드한 다음 문자열을 방출하는 Observable을 만들어보자. 



```swift
let disposeBag = DisposeBag()

enum MyError: Error {
   case error
}

Observable<String>.create { (observer) -> Disposable in
  guard let url = URL(string: "https://www.apple.com")
    else { 
      observer.onError(MyError.error)
      //정상적이지 않은 URL일 시 onError 메소드를 방출해야한다.
      return Disposables.create()
      //리턴형은 항상 Disposable이라는 걸 주의하자.
  }
  
  guard let html = try? String(contentsOf: url, encoding: .utf8)
    else {
      //html -> String 변환에 실패했을 경우 역시 onError()로 error 방출.
    observer.onError(MyError.error)
    return Disposables.create()
  }
  
  observer.onNext(html)
  observer.onCompleted()
//정상적으로 변환되면 문자열로 변환된 html을 onNext()메소드를 사용하여 next event로 구독자에게 전달한다.
//이어 onComplted()메소드로 종료시켜준다. 
  
  return Disposables.create()
}
```



코드는 완성되었으나 실행해보면 아무것도 출력되지 않는다. 

아직 구독하지 않았기 때문이다. 



```swift
...(앞에 이어서)
  .subscribe { print($0) }
  .disposed(by: disposeBag)
```



구독을 시작하는 시점에 create 연산자의 클로저가 실행된다는 것을 유의해야한다. 

이제 정상적으로 출력되는 것을 볼 수 있다. 

만약 클로저 내부에서

```swift
  observer.onNext(html)
  observer.onCompleted()
  
  observer.onNext("After completed")
```

이런 식으로 onCompleted() 이후에 새로운 문자열을 next event로 전달한다면?

아무것도 출력되지 않는다. 

이미 Completed(Error도 마찬가지) 시점에 메모리에서 해제된 상태이기 떄문이다. 



create 연산자로 Observable을 직접 구현할 때에는 몇 가지 규칙을 지켜야한다.

1. 요소를 방출할 때에는 보통 onNext() 메소드를 사용하고, 그 파라미터로 방출할 요소를 전달해야 한다. 하지만 언제나 그렇지는 않은데, Observable은 보통 하나 이상의 요소를 방출하지만 그렇지 않은 경우도 있기 때문에 언제나 onNext() 메소드만을 사용해야하는 것은 아니다.

2. 반면 Observable을 종료하기 위해서는 onError(), onCompleted() 메소드를 반드시 호출해야한다. Observable 중에는 영원히 종료되지 않는 경우도 있는데, 이런 경우가 아니라면 둘 중 하나는 반드시 호출해야한다. 둘 중 하나라도 호출하는 순간 Observable이 종료되기 때문에 이후에 onNext() 메소드를 호출하면 요소가 방출되지 않는다. 그래서 onNext()를 호출하려면 두 메소드(onError(), onCompleted())가 호출되기 전에 호출해야한다. 그래야 파라미터로 전달한 요소가 구독자에게 정상적으로 전달된다.

---

###empty, error

이 두 연산자가 생성한 Observable은 next event를 전달하지 않는다는 공통점이 있다.  즉, 어떠한 요소도 방출하지 않는다. 

empty 연산자는 completed 이벤트를 전달하는 Observable을 생성한다.



```swift
let disposeBag = DisposeBag()

Observable<Void>.empty() //타입은 중요하지 않기에 보통 Void로 선언한다.
  .subscribe { print($0) }
  .disposed(by: disposeBag)
//실행값
//completed
```



이 연산자는 Observer가 아무런 동작 없이 종료되어야 할 때 자주 사용된다.



#### error

이 연산자는 주로 error를 처리할 때 사용한다.

```swift
let disposeBag = DisposeBag()

enum MyError: Error {
   case error
}

Observable<Void>.error(MyError.error) //#1
  .subscribe { print($0) }
  .disposed(by: disposeBag)

//출력값
//error(error)
```

error event가 전달되고 종료된다.

error event에는 `#1`에서 전달한 MyError.error가 연관값으로 저장되어있다.



---

## 11.Filtering Operators



### **ignoreElements**

이 연산자는 Observable이 방출하는 next event를 필터링하고, Completed 이벤트와 Error 이벤트만 구독자로 전달한다.

![스크린샷 2020-06-05 오전 5.08.36](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgwfgvqvnj30ch05mwf1.jpg)

이 연산자는 파라미터를 받지 않고, 리턴형은 Completable이다.

Completable은 Completed event 또는 Error event 만 전달하고, next event는 무시한다. 

ignoreElements 연산자는 작업의 성공과 실패만 중요할 때에 사용한다.

```swift
let disposeBag = DisposeBag()
let fruits = ["🍏", "🍎", "🍋", "🍓", "🍇"]

Observable.from(fruits)
  .ignoreElements()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

//출력값
//completed
```



Observable은 from연산자를 통해 요소를 방출하지만, ignoreElements()에서 필터링되어 completed event만 구독자에게 전달된다.

---

### elementAt

특정 index에 위치한 요소를 제한적으로 방출하는 연산자.

![스크린샷 2020-06-05 오전 6.41.21](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgz458gv8j30bw04mdgg.jpg)

elementAt은 정수 인덱스를 파라미터로 받아 하나의 요소만을 방출하는 Observable을 리턴한다. 

결과적으로 구독자에게는 하나의 요소만이 전달되고, 나머지 요소들은 무시된다.



```swift
let disposeBag = DisposeBag()
let fruits = ["🍏", "🍎", "🍋", "🍓", "🍇"]

Observable.from(fruits)
  .elementAt(1)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next(🍎)
completed
*/
```



index 1에 해당하는 빨간 사과만 방출하고, completed 이벤트를 방출한다.

---

### filter



![스크린샷 2020-06-05 오전 6.45.06](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgz7vkkz7j30bd03iq3c.jpg)

filter 연산자는 클로저를 파라미터로 받는다. 이 클로저가 predicate로 사용된다. 이 클로저에서 true를 리턴하는 요소가 filter 연산자가 방출할 Observable의 요소에 포함된다. 

```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .filter { $0.isMultiple(of: 2)}
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*실행값
next(2)
next(4)
next(6)
next(8)
next(10)
completed
*/
```



짝수만 next event에 담겨 구독자에게 전달되고 있다.

---

### skip, skipWhile, skipeUntil

특정 요소를 무시하는 연산자들이다.



#### skip

​	정수를 파라미터로 받는다. Observable이 방출하는 요소들을 정수만큼 무시하고, 그 이후에 방출되는 요소들만 구독자로 전달한다.

```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .skip(3)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next(4)
next(5)
next(6)
next(7)
next(8)
next(9)
next(10)
completed
*/
```



주의할 점은 skip 연산자가 받는 파라미터는 인덱스값이 아닌 갯수를 나타내는 정수라는 것이다. 그래서 3개가 무시된 것이다. 만약 인덱스 값을 파라미터로 받는 연산자였다면 index 3인 4까지 무시되고 next(5)부터 출력되었을 것이다.

#### skipWhile

​	![스크린샷 2020-06-05 오전 6.55.54](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgzj3qrn0j30eo03zt9i.jpg)



이 연산자는 filter와 마찬가지로 클로저를 파라미터로 받는다. 이 클로저는 p redicate로 사용되고, 클로저에서 true를 리턴하는 동안 방출된 요소들을 무시한다. 클로저에서 false를 리턴하면, 그때부터 요소를 방출하고, 이후에는 조건에 관계없이 모든 요소를 방출한다. 연산자는 방출되는 요소를 포함한 Observable을 리턴한다.



```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .skipWhile { !$0.isMultiple(of: 2) }
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next(2)
next(3)
next(4)
next(5)
next(6)
next(7)
next(8)
next(9)
next(10)
completed
*/
```



Observable이 1을 리턴하면 true이기 때문에 무시되고, 2를 리턴하는 시점에 skipWhile 클로저 내의 조건이 false가 된다. 그때부터 skipWhile은 더이상 필터링을 하지 않고 모든 요소를 구독자에게 전달한다. 

filter 연산자는 true <-> false 와 관계없이 모든 요소를 판별하여 필터링하지만, skipWhile 연산자는 첫 false 이후에는 더 이상 요소를 필터링하지 않는다.



#### skipUntil

​	![스크린샷 2020-06-05 오전 7.05.16](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfgzsuw5jjj30dz04t3za.jpg)

skipUntil 연산자는 ObservableType을 파라미터로 받는다. 즉 다른 Observable을 파라미터로 받는다. 파라미터로 받는 Observable이 next event를 전달하기 전까지, 원본 Observable이 전달하는 이벤트를 무시한다.

이러한 특성 때문에 skipUntil 연산자가 파라미터로 받는 Observable을 Trigger라고 부르기도 한다.

```swift
let disposeBag = DisposeBag()

let subject = PublishSubject<Int>()
let trigger = PublishSubject<Int>()

subject.skipUntil(trigger)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(1)
trigger.onNext(0)
subject.onNext(2)

//출력값
//next(2)
```



 skipUntil은 trigger가 next 이벤트를 방출한 이후부터 원본 Observable이 방출하는 event를 구독자에게 전달한다. 때문에 trigger가 값을 방출하기 이전의 next event는 무시되고 next(2)만 출력되었다.

---

### take, takeWhile, takeUntil, takeLast

- 요소의 방출 조건을 다양하게 설정하는 연산자들

#### take

정수를 파라미터로 받아서, 해당 숫자만큼의 요소만을 방출함.

```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .take(3)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next(1)
next(2)
next(3)
completed
*/
```



결과를 보면 처음 3개의 요소만 방출되고, 나머지 요소는 무시된다.

take 연산자는 next event를 제외한 나머지  event에는 영향을 주지 않는다. 그래서 completed/Error 이벤트는 정상적으로 전달된다.



#### takeWhile

![스크린샷 2020-06-07 오후 8.35.04](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfjyg3nelej30jc03bdgl.jpg)

클로저를 파라미터로 받아서, predicate로 사용한다. predicate가 true를 리턴하면 구독자에게 요소를 전달한다. 연산자가 리턴하는 Observable에는 최종적으로 조건을 만족시키는 요소만 포함된다.



```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .takeWhile { !$0.isMultiple(of: 2) }
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next(1)
completed
*/
```

홀수만 전달하는 Observable을 만들었다. 

next event로 1만 방출되는 것을 확인할 수 있다. 이후에도 홀수가 방출되지만, 구독자로는 전달되지 않는다. 

takeWhile 연산자는 클로저가 false를 리턴한 시점부터의 요소는 더이상 방출하지 않는다. 다만 completed/Error 이벤트에는 영향을 미치지 않으며, 구독자에게 정상적으로 전달한다. 



#### takeUntil

![스크린샷 2020-06-07 오후 8.47.12](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfjysoupitj30ky043mxx.jpg)

takeUntil은 ObservableType을 파라미터로 받는다. 파라미터로 전달한 `Observable`(trigger)에서 next event를 전달하기 전까지, 원본 옵저버블이 방출하는 next event를 구독자에게 전달한다. 



```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let subject = PublishSubject<Int>()
let trigger = PublishSubject<Int>()

subject.takeUntil(trigger) //원본 옵저버블에 trigger Observable을 파라미터로 전달.
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(1) //출력값 next(1)
subject.onNext(2) //출력값 next(2)
trigger.onNext(0) //출력값 completed #1
subject.onNext(3) //출력값 없음 #2
```



위와 같이, trigger 옵저버블이 next event를 방출하면, 원본 옵저버블(subject)가 completed 이벤트를 방출하게 된다. 

그 이후에 다시 suject(원본 Observable)에 요소를 전달해도, 구독자에게 요소가 전달되지 않는다. `#1`에서 이미 completed 되었기 때문에, 더 이상의 next event는 전달되지 않는 것이다. 



#### takeLast

![스크린샷 2020-06-07 오후 8.53.33](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfjyzapyv4j30gz03l0t8.jpg)

takeLast 연산자는 정수를 파라미터로 받아서 옵저버블을 리턴한다. 리턴되는 옵저버블에는 원본 옵저버블이 방출하는 요소들 중에서, 마지막에 방출한 n개의 요소가 포함되어 있다. 

이 연산자에서 가장 중요한 것은, 구독자로 전달되는 시점이 딜레이 된다는 것이다.

```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

let subject = PublishSubject<Int>()

subject.takeLast(2)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

numbers.forEach { subject.onNext($0) } // #1
// 출력값 없음 (9, 10 were saved in buffer) #1

subject.onNext(11) // #2
// 출력값 없음 (10, 11 were saved in buffer) #2

subject.onCompleted() // #3
/* #3 출력값 
next(10)
next(11)
completed
*/

// 앞에 있던 #3의 subject.onCompleted() 코드 삭제
enum MyError: Error {
  case error
}
subject.onError(MyError.error) // #4
// #4 출력값
// error(error)
```



1. 위의 코드를 `#1`까지만 실행해보면, 아무것도 출력되지 않는다. 

   하지만 분명 코드는 실행되었고, takeLast는 마지막에 방출된 9, 10 두 개의 요소를 버퍼에 저장하고 있다. 

2. 이때 `#2`를 통해 새로운 요소를 방출하고 실행해보면, 여전히 아무것도 출력되지 않는다. 하지만 버퍼에 저장된 값은 (9, 10) 에서 (10, 11)로 업데이트 된다.

   아직은 Observable이 다른 요소를 방출할지 아니면 종료할지 판단할 수 없기 때문에 요소를 방출하는 시점을 계속해서 지연시키고 있는 것이다. 

3. 그러다 #3의 코드를 통해 Observable에 Completed Event를 전달하면, 이 시점까지 버퍼에 저장되어있던 요소들과  Completed event가 연달아 방출된다. 

4. 이제  `#3`의 subject.onCompleted() 코드를 지우고 `#4`처럼 error event를 Observable에 전달하면, 버퍼에 저장되어있던 요소들은 방출되지 않고 error이벤트만 전달된다. 

---

### single Operator

`single`은 원본 Observable에서 첫번째 요소만 방출하거나, 조건과 일치하는 첫번째 요소만 방출한다. 이름처럼 하나의 요소만 방출을 허락하고, 두 개 이상의 요소가 방출되면 error가 발생한다. 

```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.just(1)
  .single()
  .subscribe { print($0) }
  .disposed(by: disposeBag) // #1

/*출력값
next(1)
completed
*/

Observable.from(numbers)
  .single()
  .subscribe { print($0) }
  .disposed(by: disposeBag) //#2

/* #2 출력값 
next(1)
completed
next(1)
error(Sequence contains more than one element.)
*/
```

`#1`처럼 하나의 요소만 방출하는 Observable에 single 연산자를 사용할 경우 정상적으로 작동한다. 

반면 `#2`처럼 여러 요소를 방출하도록 하자, 값은 `#1`과 같이 정상적으로 방출되지만, Completed 이벤트가 아닌 Error 이벤트를 전달한다. 

에러 메시지를 보면, 시퀀스가 하나보다 많은 요소를 포함하고 있다고 나온다. 

single 연산자는 단 하나의 요소만 방출되어야 정상적으로 종료된다. 원본 Observable이 요소를 방출하지 않거나, 두 개 이상의 요소가 방출된다면 지금처럼 에러가 발생한다. 

![스크린샷 2020-06-07 오후 9.23.55](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfjzuyjubdj30eu03qjsj.jpg)

single 연산자는 두 가지 형태를 가진다. 파라미터가 없는 연산자와,  predicate를 받는 연산자를 제공한다. 



```swift
let disposeBag = DisposeBag()
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Observable.from(numbers)
  .single { $0 == 3}
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/* 출력값
next(3)
completed
*/
```



이런 식으로 사용하면, 배열에 3이 한 개 밖에 없으니 최종적으로 하나만 방출된다. 그래서 위와 같이 3이 방출되고, 이어서 Completed event가 전달된다. 

이 연산자는 하나의 요소만이 방출되는 것을 보장한다. subject가 생성된 다음에 전달 시점을 확인해보자. 

```swift
let subject = PublishSubject<Int>()

subject.single()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(100)
//출력값
//next(100)

subject.onNext(200)
//출력값 
//next(100)
//error(Sequence contains more than one element.)
```

이렇게 새로운 요소를 방출하면 구독자에게 바로 next event가 전달된다. 

다른 요소가 방출될 수도 있기 때문에, 하나의 요소가 방출되었다고 해서 바로 Completed event가 전달되면 안 된다. 그래서 single 연산자가 방출하는 Observable은 원본 Observable에서 Completed event를 전달할 때까지  대기한다. 

Completed event가 전달된 시점까지 하나의 요소만 방출되었다면 구독자에게 Completed 이벤트가 전달되고, 그 사이에 다른 요소가 방출되었다면 구독자에게는 error 이벤트가 전달된다. 

```swift
let subject = PublishSubject<Int>()

subject.single()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(100)
subject.onCompleted()

//출력값
//next(100)
//completed
```

위와 같이 하나의 요소만 방출된 다음 Completed 이벤트가 전달되면, 구독자에게 정상적으로 Completed 이벤트를 전달한다. 

---

### distinctUntilChanged

동일한 항목이 연속적으로 방출되지 않도록 필터링해주는 연산자. 

![스크린샷 2020-06-07 오후 9.37.32](/Users/pro/Library/Application Support/typora-user-images/스크린샷 2020-06-07 오후 9.37.32.png)

우선 이 연산자는 파라미터가 없다. 원본 Observable에서 방출한 요소 두 개를 순서대로 비교하여 만약 이전 요소와 같다면 방출하지 않는다. 두 개의 요소를 비교할 때에는 비교 연산자로 비교한다. 



```swift
let disposeBag = DisposeBag()
let numbers = [1, 1, 3, 2, 2, 3, 1, 5, 5, 7, 7, 7]

Observable.from(numbers)
  .distinctUntilChanged()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next(1) #1
next(3)
next(2)
next(3)
next(1) #2
next(5)
next(7)
completed
*/
```



위의 코드를 보면, 원본 Observable이 두 개의 1을 연속으로 방출하지만, distinckUntilChanged 연산자가 두 번째 1은 무시한다. (`#1`)

이후 `#2`에서 다시 1이 방출되는데, 한 번 필터링 됐던 요소라고 해도 그 직전에 동일한 요소가 연속으로 방출되지 않았다면 그대로 방출한다. 

이 연산자는 단순히 연속적으로 방출되는 요소만 확인/필터링 한다. 

---

### debounce, throttle

이 두 연산자는 짧은 시간 동안 반복적으로 반복되는 이벤트를 제어한다는 공통점이 있다. 연산자로 전달하는 파라미터도 동일하다. 하지만 연산의 결과는 완전히 다르다. 



#### debounce

![스크린샷 2020-06-08 오전 3.14.38](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfka01eq6cj30n50atwh6.jpg)

먼저 debounce 연산자는 두 개의 파라미터를 받는다. 첫 번째 파라미터로는 시간을 전달한다. 이 시간 파라미터는 연산자가 next event를 방출할지 말지 결정하는 조건으로 사용된다. Observable이 next event를 방출한 다음 파라미터로 입력된 시간 만큼 다음 next event를 방출하지 않는다면, 해당 시점에 가장 마지막으로 방출된 next event를 구독자로 전달합니다. 반면 지정된 시간 이내에 또 다른 next event를 방출했다면, 타이머를 초기화한다. 

타이머를 초기화한 다음, 다시 지정된 시간 동안 대기한다. 이 시간 이내에 다른 이벤트가 방출되지 않는다면 마지막 이벤트를 방출하고, 이벤트가 방출된다면 타이머를 다시 초기화한다. 

두 번째 파라미터로는 타이머를 실행할 스케쥴러를 입력 받는다. 



```swift
let disposeBag = DisposeBag()

let buttonTap = Observable<String>.create { observer in
   DispatchQueue.global().async {
      for i in 1...10 {
         observer.onNext("Tap \(i)")
         Thread.sleep(forTimeInterval: 0.3)
      }
      Thread.sleep(forTimeInterval: 1) //#1
      
      for i in 11...20 {
         observer.onNext("Tap \(i)")
         Thread.sleep(forTimeInterval: 0.5)
      }
      
      observer.onCompleted()
   }
   
   return Disposables.create {
      
   }
}

buttonTap
  .debounce(.milliseconds(1000), scheduler: MainScheduler.instance)
   .subscribe { print($0) }
   .disposed(by: disposeBag)

/*출력값
next(Tap 10)
next(Tap 20)
completed
*/
```



코드를 보면, 0.3초마다 1부터 10까지의 숫자를 방출하고, 1초 동안 멈췄다가, 다시 0.5초마다 11부터 20까지의 숫자를 방출하는 옵져버블이 있다. 

그리고 그걸 구독하고 있다. 

출력값을 보면 단 두 개의 next event만 구독자에게 전달된 것을 볼 수 있다. 

1초의 시간을 파라미터로 준 debounce 연산자는 1초 동안 Observable이 next event를 방출하지 않을 경우 구독자에게 마지막 next event를 전달하는데, 0.3초마다 1~10을 방출하는 코드이므로 아무것도 방출하지 않는다.  

`#1`에서 1초 동안 방출이 멈추는 시점에 드디어 debounce의 시간 조건이 충족되어 마지막 next event인 "Tap 10"이 구독자에게 전달되었다. 이후 debounce 연산자의 타이머는 다시 초기화된다. 

그 뒤 0.5초마다 11~20을 방출하는 코드 역시 debounce 연산자의 타이머 조건(1초 동안 아무런 next event 방출이 없을 것)을 충족시키지 못하기 때문에 아무것도 방출하지 않다가, 20이 방출된 후 1초가 경과하자 타이머 조건이 충족되어 next(Tap 20)을 구독자에게 전달한다. 이후 completed event를 구독자에게 전달하고 구독이 종료된다. 



#### throttle

![스크린샷 2020-06-08 오전 3.37.43](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfkanu2u1cj30nm0csadn.jpg)

throttle 연산자는 debounce처럼 두 개의 파라미터를 받지 않고, 사실은 세 개의 파라미터를 받는다. 하지만 기본 값 true를 갖는 두 번째 파라미터를 생략하는 경우가 많기 때문에 실제로는 debounce와 받는 파라미터가 같다고 생각해도 무방하다. 

첫 번째 파라미터에는 반복 주기를 전달하고, 세 번째 파라미터에는 스케쥴러를 전달한다. throttle은 지정된 주기 동안 하나의 이벤트만 구독자에게 전달한다. 보통 두 번째 파라미터는 기본값을 사용하는데, 이때는 주기를 엄격하게 지킨다. 항상 지정된 주기마다 지정된 event를 전달한다. 반면 이 파라미터에 false를 입력하면 반복 주기가 경과한 다음 가장 먼저 방출되는 이벤트를 구독자에게 전달한다. 

```swift
let disposeBag = DisposeBag()

let buttonTap = Observable<String>.create { observer in
  DispatchQueue.global().async {
    for i in 1...10 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.3)
    }
    
    Thread.sleep(forTimeInterval: 1)
    
    for i in 11...20 {
      observer.onNext("Tap \(i)")
      Thread.sleep(forTimeInterval: 0.5)
    }
    
    observer.onCompleted()
  }
  
  return Disposables.create()
}


buttonTap
  .throttle(.milliseconds(1000), scheduler: MainScheduler.instance)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next(Tap 1)
next(Tap 4)
next(Tap 7)
next(Tap 10)
next(Tap 11)
next(Tap 12)
next(Tap 14)
next(Tap 16)
next(Tap 18)
next(Tap 20)
completed
*/
```



throttle operator는 next event를 지정된 주기마다 하나씩 구독자에게 전달한다. 

반면 debounce 연산자는 next event가 전달된 다음 지정된 시간이 경과하기까지 다른 이벤트가 전달되지 않는다면 마지막으로 방출된 이벤트를 구독자에게 전달한다. 

짧은 시간동안 반복되는 Tap event나 delegate 메시지를 처리할 때 보통 throttle 연산자를 사용한다. 

debounce 연산자는 주로 검색 기능을 구현할 때 사용한다. 사용자가 키워드를 입력할 때마다 network request를 보내거나 데이터 베이스를 검색하는 것은 비효율적이기 때문이다. 

#### throttle 연산자의 두 번째 파라미터에 대하여

```swift
let disposeBag = DisposeBag()

func currentTimeString() -> String {
   let f = DateFormatter()
   f.dateFormat = "yyyy-MM-dd HH:mm:ss.SSS"
   return f.string(from: Date())
}


Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
   .debug()
   .take(10)
   .throttle(.milliseconds(2500), latest: true, scheduler: MainScheduler.instance)
   .subscribe { print(currentTimeString(), $0) }
   .disposed(by: disposeBag)


Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
   .debug()
   .take(10)
   .throttle(.milliseconds(2500), latest: false, scheduler: MainScheduler.instance)
   .subscribe { print(currentTimeString(), $0) }
   .disposed(by: disposeBag)
```



1초마다 정수를 방출하는 Observable이 두 개 있고, 차이는 오직 throttle 연산자의 두 번째 파라미터인 latest: 가 true - false 로 다르다는 것이다. 

그리고 이벤트의 발생시간을 정확히 체크하기 위해 debug() 연산자를 추가했다. 

아래쪽 옵저버블을 주석처리하고, 첫 번째 옵저버블만 실행한 결과는 아래와 같다.

```swift
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
   .debug()
   .take(10)
   .throttle(.milliseconds(2500), latest: true, scheduler: MainScheduler.instance)
   .subscribe { print(currentTimeString(), $0) }
   .disposed(by: disposeBag)

/* 출력값
2020-06-08 03:54:52.203: 2.xcplaygroundpage:43 (__lldb_expr_87) -> subscribed
2020-06-08 03:54:53.218: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(0)
2020-06-08 03:54:53.218 next(0)
2020-06-08 03:54:54.217: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(1)
2020-06-08 03:54:55.218: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(2)
2020-06-08 03:54:55.719 next(2)
2020-06-08 03:54:56.218: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(3)
2020-06-08 03:54:57.217: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(4)
2020-06-08 03:54:58.217: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(5)
2020-06-08 03:54:58.220 next(5)
2020-06-08 03:54:59.218: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(6)
2020-06-08 03:55:00.217: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(7)
2020-06-08 03:55:00.722 next(7)
2020-06-08 03:55:01.217: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(8)
2020-06-08 03:55:02.217: 2.xcplaygroundpage:43 (__lldb_expr_87) -> Event next(9)
2020-06-08 03:55:02.217: 2.xcplaygroundpage:43 (__lldb_expr_87) -> isDisposed
2020-06-08 03:55:03.224 next(9)
2020-06-08 03:55:03.224 completed
*/
```



이렇듯 true를 주거나 아무 값도 주지 않으면 시간 주기를 정확히 지켜 가장 최근에 방출되었던 이벤트를 구독자에게 전달한다. 

이 코드에서는 throttle 연산자의 첫 번째 파라미터로 주었던 시간 2.5초의 주기로 마지막 event를 전달하고 있다. 

```swift
//Event next(0)
//next(0)
```

우선 옵저버블이 방출한 첫번째 이벤트는 방출 즉시 구독자에게 전달된다. 

```swift
//Event next(1)
//Event next(2)
//next(2)
//Event next(3)
```

그리고 이어서 옵저버블이 1과 2를 방출한다. 그리고 3을 방출하기 전에 2.5초가 경과하기 때문에 그 순간에서 가장 최근 이벤트였던 next event(2)가 구독자에게 전달된다. 그래서 2가 담긴 `next(2)`가 subscribe의 클로저로 전달되어 출력되었다. 

이것이 throttle 연산자의 기본 동작이다. 0, 2, 5, 7, 9 총 5개의 next event가 방출되었다. 



이번에는 latest: false를 준 옵저버블의 결과를 살펴보자. 

```swift
Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
   .debug()
   .take(10)
   .throttle(.milliseconds(2500), latest: false, scheduler: MainScheduler.instance)
   .subscribe { print(currentTimeString(), $0) }
   .disposed(by: disposeBag)

/*출력값
2020-06-08 04:02:52.301: 2.xcplaygroundpage:51 (__lldb_expr_89) -> subscribed
2020-06-08 04:02:53.315: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(0)
2020-06-08 04:02:53.316 next(0)
2020-06-08 04:02:54.314: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(1)
2020-06-08 04:02:55.314: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(2)
2020-06-08 04:02:56.315: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(3)
2020-06-08 04:02:56.315 next(3)
2020-06-08 04:02:57.315: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(4)
2020-06-08 04:02:58.314: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(5)
2020-06-08 04:02:59.314: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(6)
2020-06-08 04:02:59.314 next(6)
2020-06-08 04:03:00.315: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(7)
2020-06-08 04:03:01.315: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(8)
2020-06-08 04:03:02.315: 2.xcplaygroundpage:51 (__lldb_expr_89) -> Event next(9)
2020-06-08 04:03:02.315 next(9)
2020-06-08 04:03:02.316 completed
2020-06-08 04:03:02.316: 2.xcplaygroundpage:51 (__lldb_expr_89) -> isDisposed
*/
```

이번에는 0, 3, 6, 9가 담긴 Next event가 구독자에게 전달되어 출력되었다. 

그리고 첫 번째로 전달된 next event 0과 두 번째로 전달된 next event 3의 시간 차이를 보면 throttle 연산자의 파라미터로 주었던 2.5초 간격이 아니라 3초 간격을 두고 있다. 

두 번째 파라미터로 false를 전달하면 next event가 방출된 다음 지정된 주기가 지나고, 그 이후에 첫 번째로 방출되는 next event를 전달한다. 

```swift
//Event next(0)
//next(0)
//Event next(1)
//Event next(2)
//Event next(3)
//next(3)
```



  첫 번째 next event는 구독자에게 바로 전달된다. 이어서 Observable이 1과 2를 방출하고, 0.5초 후에 주기가 끝난다. 만약 두 번째 파라미터로 true를 전달했었다면 마지막에 방출된 next event(2)가 구독자에게 전달되었겠지만 이번에는 false를 전달했기 때문에 Observable이 새로운 next event를 방출할 때까지 기다린다. 그러다가 0.5초 뒤에 3이 담긴 next event가 방출되면 이 event를 구독자에게 전달한다.

 두 번째 파라미터로 어떤 값을 전달하더라도 지정된 주기 동안 하나의 next event만 전달한다는 점은 달라지지 않는다. 

 다만 차이는 next event가 구독자로 전달되는 주기이다. true를 전달하면 주기를 엄격하게 지키지만 false 를 전달하면 지정된 주기를 초과할 수 있다. 



---

## 12.Transforming Operators



### toArray Operator

toArray 연산자는 Observable이 방출하는 모든 요소를 배열에 담은 다음, 이 배열을 방출하는 Observable을 생성한다. 

![스크린샷 2020-06-08 오전 4.16.33](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfkbs9p5jhj30mg08dtaf.jpg)

이 연산자는 별도의 파라미터를 받지 않는다. 하나의 요소를 방출하는 Single Observable 타입으로 변환한다. 

`Single`은 하나의 요소를 방출하거나 error 이벤트를 전달하는 특별한 Observable이다. 그러므로 결국 Single을 구독한다면 구독자에게 전달되는 값은 success, error 두 개의 이벤트 중 하나가 된다. 때문에 세 개를 처리해야하는 여타의 Observable과는 달리 처리해야할 이벤트가 하나 줄어들어 코드가 조금 간결해지는 장점이 있다. 

특히 네트워크 요청을 구독하는 기능을 만들면 성공 시 하나의 값을 넣어 success event를 구독자로 전달하고, 실패 시 error 이벤트를 구독자로 전달하는 식으로 편리하게 사용할 수 있다.

하지만 Single을 사용하기 위해서는 Single로 '시작'해야한다. Observable로 시작한 Stream을 중간에 asSingle로 바꿔 Single화 시키게 되면 문제가 생긴다. 

Single이 아닌 Observable은 completed 이벤트를 방출하는데, Single은 Completed 이벤트를 방출할 수 없기 때문에 asSingle이전의 원본 Observable이 next event를 방출하지 않고 Completed 이벤트만을 전달하는 경우 구독자에게 Completed 이벤트를 전달하지 않고 error 이벤트를 전달하게 된다. completed event 이전에 next event를 반드시 방출하는 Observable에만 제한적으로 asSingle 연산자를 사용해야 의도치않은 error event 전달을 피할 수 있다. 



```swift
let disposeBag = DisposeBag()

let subject = PublishSubject<Int>()

subject
  .toArray()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(1)
//출력값 없음.
```



이때의 경우에는 출력값이 없는데, 이는 모든 요소를 하나의 배열로 만들어 전달하는 toArray 연산자의 특성 때문이다. Observable이 종료되어야 그때까지 전달된 요소들을 하나의 배열에 넣어 구독자로 전달한다. 



```swift
let disposeBag = DisposeBag()

let subject = PublishSubject<Int>()

subject
  .toArray()
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(1)
subject.onNext(2)
subject.onCompleted()

//출력값
//success([1, 2])
```



이처럼 onCompleted() 이벤트가 방출되어야만 그 시점까지 방출된 요소들을 배열에 담아 구독자에 success event로 전달하는 것을 알 수 있다. 

---

### map

map 연산자는 Observable이 방출한 요소들을 대상으로 함수를 실행한다. 그런 다음 실행 결과를 방출하는 Observable을 리턴하고, 구독자에게 전달한다.



```swift
let disposeBag = DisposeBag()
let skills = ["Swift", "SwiftUI", "RxSwift"]

Observable.from(skills)
  .map { "Hello, \($0)" }
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/* 출력값
next(Hello, Swift)
next(Hello, SwiftUI)
next(Hello, RxSwift)
completed
*/
```



Swift의 고차함수에 익숙하다면, Observable을 리턴한다는 것 이외엔 모두 동일하다고 봐도 무방하다. 전달 받은 요소들의 타입과 리턴하는 요소들의 타입이 일치할 필요가 없다. 

---

### flatMap

원본 Observable이 항목을 방출하면, flatMap 연산자가 변환 함수를 실행한다. 변환 함수는 방출된 항목을 Observable로 변환한다. 

방출된 항목의 값이 바뀌면, flatMap 연산자가 변환한 Observable이 새로운 항목을 방출한다. 이러한 특징 때문에 원본 Observable이 방출한 항목을 지속적으로 감시하고, 최신 값을 확인할 수 있다. 

flatMap은 그렇게 방출된 Observable들을 모아서 하나의 Observable로 만들어 리턴한다. 즉 개별 항목이 개별 Observable로 변환되었다가, 다시 하나의 Observable로 합쳐진다. 

```swift
let disposeBag = DisposeBag()

let a = BehaviorSubject<Int>(value: 1)
let b = BehaviorSubject<Int>(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
  .flatMap { $0.asObservable() } // Subject -> Observable로 변환하는 메소드
  .subscribe { print($0) }
  .disposed(by: disposeBag)

subject.onNext(a)
subject.onNext(b)

a.onNext(11)
b.onNext(22)

/* 출력값
next(1)
next(2)
next(11)
next(22)
*/
```



위 코드는 `BehaviorSubject<Int>`를 방출하는 `PublishSubject`인 `subject`에 대해 `flatMap` 연산자를 사용하는 코드이다. 

subject를 구독한 뒤 `BehaviorSubject<Int>`인 `a`와 `b`를 차례대로 next event에 담아 전달하면 방출된 순서대로 구독자로 전달된다. 

이후 a와 b에 새로운 값을 전달할 때마다 다시 새로운 값이  구독자에게 전달된다. 

`flatMap`연산자는 원본 Observable이 방출하는 항목을 새로운 Observable로 변환한다. 

새로운 Observable은 항목이 업데이트될 때마다 새로운 항목을 방출한다. 이렇게 생성된 모든 Observable은 최종적으로 하나의 단일 Observable로 합쳐지고, 모든 새로운 항목들이 이 Observable을 통해 구독자로 전달된다. 

단순히 처음에 방출된 항목만 구독자로 전달되는 것이 아니라, 업데이트된 최신항목도 구독자로 전달된다. 이 연산자는 네트워크 요청을 구현할 때 자주 사용된다.

그러니 flatMap 연산자를 사용한 원본 Observable에 `BehaviorSubject<Int>`를 next event로 전달했다면, 그`BehaviorSubject<Int>`가 새로운 요소를 방출할 때마다 해당 요소를 새롭게 원본 Observable의 구독자에게 전달하게 된다.



---

### flatMapFirst

flatMap과 동일한 파라미터/리턴값을 갖는다. 하지만 연산자라 리턴하는 Observable에는 처음에 변환된 Observable이 방출하는 항목만 포함된다. 

```swift
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
   .flatMapFirst { $0.asObservable() }
   .subscribe { print($0) }
   .disposed(by: disposeBag)

subject.onNext(a) //여기까지 실행할 경우 next(1) 출력.
/* 여기까지 실행하면 PublishSubject가 a에 저장된 BehaviorSubject를 방출한다. flatMapFirst는 a가 방출하는 요소를 새로운 Observable로 변환한다. 그리고 현재는 a에 저장된 초기값 1이 구독자로 전달된다. */

subject.onNext(b) //이 코드를 실행해도 next(1)만이 출력됨.
//이렇듯 flatMapFirst는 처음으로 변환한 Observable만을 방출함.

a.onNext(11)
b.onNext(22)
b.onNext(222)
a.onNext(111)
```



a가 방출하는 11과 111은 구독자로 전달된다. 하지만 b가 방출하는 항목들은 무시되고, 구독자로 전달되지 않는다. 

---

### flatMapLatest

원본 Observable이 방출하는 항목을 새로운 Observable로 변환하는 것은 동일하지만,

모든 Observable이 방출하는 항목을 하나로 병합하지 않는다. 

대신 가장 최근에 항목을 방출한 Observable을 제외한 나머지는 모두 무시한다.

```swift
let disposeBag = DisposeBag()

let a = BehaviorSubject(value: 1)
let b = BehaviorSubject(value: 2)

let subject = PublishSubject<BehaviorSubject<Int>>()

subject
   .flatMapLatest { $0.asObservable() }
   .subscribe { print($0) }
   .disposed(by: disposeBag)

subject.onNext(a) //이 시점에는 flatMapLatest가 가장 최근에 변환한 Observable이 a이기 때문에 a가 방출하는 요소들을 구독자에게 전달한다.
a.onNext(11)
//여기까지의 출력값
//next(1)
//next(11)

subject.onNext(b) //하지만 flatMapLatest는 이때 새로운 Observable b를 변환한다. 이때부터 이전에 변환했던 Observable a가 방출하는 항목은 무시하고, b가 방출하는 항목만 구독자로 전달한다. 
b.onNext(22)

a.onNext(33) // #1

subject.onNext(a) //이 시점에 다시 subject에 a를 전달하면, #1에서 33을 가지게 된 a의 value가 구독자로 전달된다.(a가 BehaviorSubject이기 때문에 값을 저장하여 가지고 있기 때문) 이후부터는 b에서 방출되는 값을 무시하게 된다.

b.onNext(44)
a.onNext(55)
/*출력값
next(1)
next(11)
next(2)
next(22)
next(33)
next(55)
*/
```



가장 최근에 변환된 Observable이 방출하는 요소만 구독자에게 전달한다. 



---

### scan

 이 연산자는 기본값으로 연산을 시작한다.

 원본 Observable이 방출하는 항목을 대상으로 변환을 실행한 다음, 결과를 방출하는 하나의 Observable을 리턴한다. 

그래서 원본이 방출하는 항목의 수와 구독자로 전달되는 항목의 수가 동일하다. ![스크린샷 2020-06-08 오전 6.50.35](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfkg8jlz03j30de049gm3.jpg)

첫 번째 파라미터로 기본값 seed를 전달한다. 두 번째 연산자로는 클로저를 전달한다.

![스크린샷 2020-06-08 오전 6.51.39](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfkg9oc7g1j30n70c941j.jpg)

클로저 형식을 보면 파라미터가 두 개이다. `(A, Self.Element)`. 첫 번째 파라미터는 기본값의 형식과 같고, 두 번째 파라미터는 Observable이 방출하는 항목의 형식과 같다. 그리고 클로저의 리턴형은 첫 번째 파라미터와 같다. 

scan 연산자로 전달하는 클로저는 Accumulator Function / Accumulator Closure 라고 부른다. 기본값이나 Observable이 방출하는 항목을 대상으로 Accumulator Closure를 실행한 다음, 결과를 Observable로 리턴한다. 클로저가 리턴한 값은 이어서 실행되는 클로저의 첫 번째 파라미터로 전달된다. 



```swift
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 10)
.scan(0, accumulator: +)
```



이 코드에서는 accumulator closure로 `+`연산자를 전달한다. 

Observable이 1을 방출하면 클로저로 기본값 0과 1이 전달되고,  두 수를 합한 값이 리턴된다. 결과적으로 구독자에게는 1이 전달된다. 

Observable이 다시 2를 방출하면 이전 결과인 1과 새로 방출된 2가 클로저로 전달된다. 그러면 구독자에게는 3이 전달된다. 



```swift
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 10)
  .scan(0, accumulator: +)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/* 출력값
next(1)
next(3)
next(6)
next(10)
next(15)
next(21)
next(28)
next(36)
next(45)
next(55)
completed
*/
```



Observable이 모든 항목을 방출할 때까지 (즉, completed event를 통해 종료될 때까지) 계속 누적된 값을 전달하고 있다. 이 연산자는 작업 결과를 누적시키면서, 중간 결과와 최종 결과가 모두 필요한 경우에 사용된다. 

Swift의 고차함수인 reduce와 비슷하다. 하지만 reduce 연산자는 중간 결과 없이 최종 결과만을 단일 요소로 전달한다. 때문에 중간 결과가 필요한 경우 scan, 최종 결과만이 필요한 경우에는 reduce를 사용한다. 



```swift
let disposeBag = DisposeBag()

Observable.range(start: 1, count: 10)
  .reduce(0, accumulator: +)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

//출력값
//next(55)
//completed
```

---

### buffer

이 연산자는 특정 주기 동안 Observable이 방출하는 항목을 수집하고, 하나의 배열로 리턴한다. RxSwift에서는 이러한 동작을 Controlled Buffering이라고 한다. 

![스크린샷 2020-06-08 오전 7.04.01](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfkgmikxx4j30n10dntcj.jpg)

이 연산자는 세 개의 파라미터를 받는다. 

첫 번째 파라미터는 항목을 수집할 시간이다. 여기에서 지정한 시간이 도래할 때마다 수집된 항목들을 방출한다. (시간이 아직 경과하지 않은 경우에도 항목을 방출할 수도 있다.)

두 번째 파라미터는 수집할 항목의 갯수이다. 정확한 숫자가 아니라 최대 갯수이다. 최대 갯수보다 적은 갯수를 수집했더라도, 시간이 경과하면 수집된 항목만 방출한다. 그래서 count가 아니라 Maximum element count로 표기한다. 

마지막 스케쥴러는 연산자가 실행될 쓰레드를 지정할 수 있다.

연산자의 리턴 형은 요소들의 배열이다. 즉 수집한 요소들을 배열에 담아 리턴한다. 



```swift
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .buffer(timeSpan: .seconds(2), count: 3, scheduler: MainScheduler.instance)
  .take(5)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/* 출력값
next([0])
next([1, 2, 3])
next([4, 5])
next([6, 7])
next([8, 9])
completed
*/
```



Observable은 1초마다 항목을 방출하고 있고, buffer 연산자는 2초마다 3개씩 수집하고 있다. 

buffer 연산자는 첫 번째 파라미터로 전달한 timeSpan이 경과하면 그 시점까지 수집된 항목들을 즉시 방출한다. 두 번째 파라미터로 전달한 갯수만큼 수집되지 않았더라도 즉시 방출한다. 

2초마다 수집하고 있으니 방출되는 배열에는 보통 두 개의 요소가 포함되어 있어야하지만 시간 상의 오차로 인해 첫 번째 배열처럼 하나만 포함되어있거나 두 번째 배열처럼 세 개가 포함되는 경우도 있다.

```swift
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .buffer(timeSpan: .seconds(5), count: 3, scheduler: MainScheduler.instance)
  .take(5)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next([0, 1, 2])
next([3, 4, 5])
next([6, 7, 8])
next([9, 10, 11])
next([12, 13, 14])
completed
*/
```



이번에는 timeSpan을 5초로 바꾸었다. 그러자 3초마다 3개의 요소가 담긴 배열이 리턴되었다. 

5초가 경과하지 않았더라도 최대 갯수인 3개가 충족되는 순간 그때까지 수집된 요소들을 배열에 담아 전달하기 때문이다. 

---

### window

이 연산자는 buffer 연산자처럼 timeSpan과 Max Count를 지정해서 원본 Observable이 방출하는 항목들을 작은 단위의 Observable로 분해한다. buffer 연산자는 수집된 항목을 배열 형태로 리턴하지만, window 연산자는 수집된 항목을 방출하는 Observable을 리턴한다. 그래서 리턴된 Observable이 무엇을 방출하고 언제 완료되는지 이해하는 것이 중요하다. 

![스크린샷 2020-06-08 오전 7.17.20](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfkh0dc96fj30my09stbr.jpg)

파라미터 3개는 모두 buffer 연산자와 동일하다. 

buffer operator와의 차이는 리턴형이다. buffer 연산자는 수집된 배열을 방출하는 Observable을 리턴한다.

반면 window operator는 Observable을 방출하는 `Observable`을 리턴한다. 이렇게 'Observable이 방출하는 Observable'을 Inner Observable이라고 부른다. 

Inner Observable은 지정된 최대 항목 수만큼 방출하거나, 지정된 시간이 경과하면 Completed 이벤트를 전달하고 종료된다. 



```swift
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .window(timeSpan: .seconds(2), count: 3, scheduler: MainScheduler.instance)
  .take(5)
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/* 출력값
next(RxSwift.AddRef<Swift.Int>)
next(RxSwift.AddRef<Swift.Int>)
next(RxSwift.AddRef<Swift.Int>)
next(RxSwift.AddRef<Swift.Int>)
next(RxSwift.AddRef<Swift.Int>)
completed
*/
```



결과를 살펴보면 2초마다 항목이 방출되고 있다. next 이벤트에 담긴 AddRef가 바로 Inner Observable이다. AddRef는 Observable이고, 그렇기 때문에 구독할 수도 있다.



```swift
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .window(timeSpan: .seconds(2), count: 3, scheduler: MainScheduler.instance)
  .take(5)
  .subscribe { print($0)
    
    if let observable = $0.element {
      observable.subscribe { print("inner: ", $0)}
    }
  }
  .disposed(by: disposeBag)

/*출력값
next(RxSwift.AddRef<Swift.Int>)
inner:  next(0)
inner:  completed
next(RxSwift.AddRef<Swift.Int>)
inner:  next(1)
inner:  next(2)
inner:  next(3)
inner:  completed
next(RxSwift.AddRef<Swift.Int>)
inner:  next(4)
inner:  next(5)
inner:  completed
next(RxSwift.AddRef<Swift.Int>)
inner:  next(6)
inner:  next(7)
inner:  completed
next(RxSwift.AddRef<Swift.Int>)
completed
inner:  next(8)
inner:  next(9)
inner:  completed
*/
```



Observable은 1초마다 하나씩 항목을 방출하고, window 연산자는 2초마다 3개씩 수집하고 있기 때문에 Max count는 채우지 못한다. 결과를 보면 max count가 채워질 때까지 기다리지 않고 2초동안 항목을 방출한 다음 바로 종료하는 것을 확인할 수 있다. 하지만 시간 오차로 인해 3개의 요소를 방출하는 Inner Observable이 결과로 출력되기도 한다.



```swift
let disposeBag = DisposeBag()

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .window(timeSpan: .seconds(5), count: 3, scheduler: MainScheduler.instance)
  .take(5)
  .subscribe { print($0)
    
    if let observable = $0.element {
      observable.subscribe { print("inner: ", $0)}
    }
  }
  .disposed(by: disposeBag)

/*출력값
next(RxSwift.AddRef<Swift.Int>)
inner:  next(0)
inner:  next(1)
inner:  next(2)
inner:  completed
next(RxSwift.AddRef<Swift.Int>)
inner:  next(3)
inner:  next(4)
inner:  next(5)
inner:  completed
next(RxSwift.AddRef<Swift.Int>)
inner:  next(6)
inner:  next(7)
inner:  next(8)
inner:  completed
next(RxSwift.AddRef<Swift.Int>)
inner:  next(9)
inner:  next(10)
inner:  next(11)
inner:  completed
next(RxSwift.AddRef<Swift.Int>)
completed
inner:  next(12)
inner:  next(13)
inner:  next(14)
inner:  completed
*/
```



이번에는 5초동안 3개의 항목을 수집하는 window 연산자를 사용한 것이다. 실행해보면 5초가 지나지 않았음에도 inner Observable이 종료되는 것을 볼 수 있는데, 이는 buffer 연산자와 마찬가지로 Max Count가 충족되었기 때문이다.

---

### groupBy

Observable이 방출하는 요소를 원하는데로 grouping 할 때 사용한다. 

![스크린샷 2020-06-08 오전 7.37.28](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfkhlboxq0j30mo0avgoo.jpg)

파라미터로 클로저를 받고, 클로저는 요소를 파라미터로 받아서 키를 리턴한다. Key의 형식은 Hashable 프로토콜은 채용하는 형식으로 제한되어 있다. 연산자를 실행하면 클로저에서 동일한 값을 리턴하는 요소끼리 그룹으로 묶이고, 그룹에 속한 요소들은 개별 Observable을 통해 방출된다. 

groupBy 연산자가 리턴하는 리턴형을 보면 Type parameter가 `GroupedObservable`로 선언되어 있다. 여기에는 방출하는 요소와 함께 Key가 저장되어 있다. 



```swift
let disposeBag = DisposeBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
  .groupBy { $0.count } //이 클로저에서 문자열의 길이를 리턴하면 Key 형식이 Int가 된다. 그리고 문자열 길이에 따라 그룹핑된다.
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/* 출력값
 next(GroupedObservable<Int, String>(key: 5, source: RxSwift.(unknown context at $110763888).GroupedObservableImpl<Swift.String>))
 next(GroupedObservable<Int, String>(key: 6, source: RxSwift.(unknown context at $110763888).GroupedObservableImpl<Swift.String>))
 next(GroupedObservable<Int, String>(key: 4, source: RxSwift.(unknown context at $110763888).GroupedObservableImpl<Swift.String>))
 next(GroupedObservable<Int, String>(key: 3, source: RxSwift.(unknown context at $110763888).GroupedObservableImpl<Swift.String>))
 completed
*/
```



출력값을 보면 String이 아니라 그룹으로 묶인 문자열을 방출하는 Observable이 방출되고 있다. 그리고 그 Observable은 GroupedObservable이며 Key가 함께 저장되어 있다. 총 4개의 Observable이 방출되고 있는데, 그 이유는 문자열 길이를 기준으로 그룹핑을 했을 때 4개의 그룹이 나왔기 때문이다.



```swift
let disposeBag = DisposeBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
  .groupBy { $0.count }
  .subscribe(onNext: { groupedObservable in
    print("== \(groupedObservable.key)")
    groupedObservable.subscribe { print("  \($0)")}
  })
  .disposed(by: disposeBag)

/* 출력값
== 5
  next(Apple)
== 6
  next(Banana)
  next(Orange)
== 4
  next(Book)
  next(City)
== 3
  next(Axe)
  completed
  completed
  completed
  completed
*/
```



이런 식으로 GroupedObservable이 방출하는 Key를 출력하고, Key에 저장된 Inner Observable을 구독하여 내부 값들을 출력할 수 있다. 

Key 5에 저장된 Observable은 Apple을 구독자로 전달했다.

groupBy 연산자를 사용할 때에는 보통 flatMap과 toArray 연산자를 활용해서 그룹핑된 최종 결과를 하나의 배열로 방출하도록 구현한다. 

```swift
let disposeBag = DisposeBag()
let words = ["Apple", "Banana", "Orange", "Book", "City", "Axe"]

Observable.from(words)
  .groupBy { $0.count }
  .flatMap { $0.toArray() }
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/* 출력값
next(["Apple"])
next(["Axe"])
next(["Banana", "Orange"])
next(["Book", "City"])
completed
*/
```



   `.flatMap { $0.toArray() }` 코드가 GroupingObservable들과 내부의 Inner Observable들을 받아서 배열로 만든다.

1. flatMap이 Inner Observable들을 새로운 Observable로 변환한다.
2. Inner Observable이 방출하는 항목들이 flatMap이 변환한 새로운 Observable에서 방출된다.
3. 하지만 toArray연산자가 구독자로의 전달을 막는다. (Completed event가 전달될 때까지 toArray 연산자는 배열을 만들 뿐 전달하지 않고 있기 때문에)
4. GroupingObservable의 Key에 따른 각각의 Inner Observable들이 종료되는 순서대로 toArray() 연산자가 구독자로 완성된 배열을 전달한다.



```swift
Observable.range(start: 1, count: 10)
  .groupBy{ $0.isMultiple(of: 2) }
  .flatMap { $0.toArray() }
  .subscribe { print($0) }
  .disposed(by: disposeBag)

/*출력값
next([2, 4, 6, 8, 10])
next([1, 3, 5, 7, 9])
completed
*/
```



위의 코드는 groupBy 연산자를 사용하여 숫자를 홀수와 짝수로 나누어 출력하는 코드이다. groupBy 연산자를 활용하면 조건에 따라 해당하는 요소들을 손쉽게 grouping할 수 있다.



---

## 13. Combining Operators

### startWith

startWith 연산자는 Observable이 요소를 방출하기 전에 다른 항목들을 앞부분에 추가합니다. 

주로 기본값이나 시작값을 지정할 때 사용한다. 

![스크린샷 2020-06-08 오후 11.06.25](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfl8fxq0yqj30n8071jt5.jpg)

startWith Operator의 parameter는 가변 파라미터이다. 파라미터로 전달하는 하나 이상의 값을 Observable 시퀀스 앞 부분에 추가한다. 그 다음 새로운 Observable을 리턴한다. 



```swift
let bag = DisposeBag()
let numbers = [1, 2, 3, 4, 5]

Observable.from(numbers)
  .startWith(0)
  .subscribe { print($0) }
  .disposed(by: bag)

/* 출력값
next(0)
next(1)
next(2)
next(3)
next(4)
next(5)
completed
*/
```



파라미터로 0을 전달하면 이렇게 0부터 방출한다. 

startWith 연산자 역시 다른 연산자와 함께 사용할 수 있다. 

```swift
let bag = DisposeBag()
let numbers = [1, 2, 3, 4, 5]

Observable.from(numbers)
  .startWith(0)
  .startWith(-1, -2)
  .startWith(-3)
  .subscribe { print($0) }
  .disposed(by: bag)

/*출력값
next(-3)
next(-1)
next(-2)
next(0)
next(1)
next(2)
next(3)
next(4)
next(5)
completed
*/
```



startWith 연산자를 여러번 적용하면 위와 같이 마지막에 추가한 값부터 차례대로 출력된다. startWith는 기존 Observable 앞부분에 값을 추가하기 때문이다. 여러번 중첩한 startWith 연산자는 LIFO 형식을 가진다. (마지막에 추가한 값이 가장 앞에 옴)

---

### concat

​	concat 연산자는 두 개의 Observable을 연결할 때 사용한다. concat 연산자는 type method와 instance method로 구현되어있다. 

![스크린샷 2020-06-08 오후 11.15.40](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfl8phxadhj30ms0dotc6.jpg)

type method로 구현된 concat 연산자는 파라미터로 전달된 컬렉션에 있는 모든 Observable을 순서대로 연결한 하나의 Observable을 리턴한다. 



```swift
let bag = DisposeBag()
let fruits = Observable.from(["🍏", "🍎", "🥝", "🍑", "🍋", "🍉"])
let animals = Observable.from(["🐶", "🐱", "🐹", "🐼", "🐯", "🐵"])

Observable
  .concat([fruits, animals])
  .subscribe { print($0) }
  .disposed(by: bag)

/*출력값
next(🍏)
next(🍎)
next(🥝)
next(🍑)
next(🍋)
next(🍉)
next(🐶)
next(🐱)
next(🐹)
next(🐼)
next(🐯)
next(🐵)
completed
*/
```



결과를 보면 모든 과일이 방출된 후에 동물들이 방출된다. 그리고 completed이벤트는 하나로 연결된 Observable이 모든 요소를 방출한 후에 한 번 전달된다.



![스크린샷 2020-06-09 오전 3.56.36](https://tva1.sinaimg.cn/large/007S8ZIlgy1gflgtsyggvj30n409vtb8.jpg)

instance method로 구현된 concat 연산자는 대상 Observable이 completed 이벤트를 전달한 경우 파라미터로 전달한 Observable을 연결한다. 만약 error event가 전달된다면 Observable은 연결되지 않는다. 대상 Observable이 방출한 요소만 전달되고, error 이벤트가 전달된 다음에 바로 종료된다. 이것은 type method로 구현된 concat 연산자도 마찬가지이다.

```swift
fruits
  .concat(animals)
  .subscribe { print($0) }
  .disposed(by: bag)

/*출력값
next(🍏)
next(🍎)
next(🥝)
next(🍑)
next(🍋)
next(🍉)
next(🐶)
next(🐱)
next(🐹)
next(🐼)
next(🐯)
next(🐵)
completed
*/
```



instace method로 구현된 concat도 출력값은 동일하다. 

concat 연산자는 두 Observable을 연결한다. 단순히 하나의 Observable 뒤에 다른 Observable을 연결하기 때문에 연결된 모든 Observable이 방출하는 요소들이 방출 순서대로 정렬되지는 않는다.(순서를 보장하지 않음)

이전 Observable이 모든 요소를 방출하고 completed event를 전달하면 이어지는 Observable이 방출을 시작한다.

---

### merge

merge 연산자는 여러 Observable이 방출하는 항목들을 하나의 Observable이 방출하도록 병합한다. 이 연산자는 concat 연산자와 혼동하기 쉬운데, concat 연산자와는 동작 방식이 다르다. 

concat 연산자는 하나의 Observable이 모든 요소를 방출하고 completed event를 전달하면 이어지는 Observable을 연결한다. 

반면 merge 연산자는 두 개 이상의 Observable을 병합하고, 모든 Observable이 방출하는 ''요소''들을 순서대로 방출하는 하나의 Observable을 리턴한다. 

![스크린샷 2020-06-09 오전 4.14.39](https://tva1.sinaimg.cn/large/007S8ZIlgy1gflhckwwraj30mo08sabs.jpg)

단순히 뒤에 연결하는 것이 아니라, 하나의 Observable로 합쳐준다. 그래서 Observable이나 subject로 전달된 이벤트가 순서대로 구독자에게 전달된다.



```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let oddNumbers = BehaviorSubject(value: 1)
let evenNumbers = BehaviorSubject(value: 2)
let negativeNumbers = BehaviorSubject(value: -1)

let source = Observable.of(oddNumbers, evenNumbers)

source
  .subscribe { print($0) }
  .disposed(by: bag)

/*출력값
next(RxSwift.BehaviorSubject<Swift.Int>)
next(RxSwift.BehaviorSubject<Swift.Int>)
completed
*/
```



of 연산자를 이용하여 `oddNumbers`와 `evenNumbers`를 전달하는 Observable `source`를 만들고 구독하면 당연히 두 개의 BehaviorSubject가 next event에 담겨 방출된다.



```swift
source
  .merge()
  .subscribe { print($0) }
  .disposed(by: bag)

/*출력값
next(1)
next(2)
*/
```



이전과 달리 next event에는 두 subject가 아닌, subject들이 방출한 항목이 저장되어있다.

이 상태에서 `oddNumbers`와 `evenNumbers`에 새로운 값을 전달해보자.

```swift
source
  .merge()
  .subscribe { print($0) }
  .disposed(by: bag)

oddNumbers.onNext(3)
evenNumbers.onNext(4)

evenNumbers.onNext(6)
oddNumbers.onNext(5)

/*출력값
next(1)
next(2)
next(3)
next(4)
next(6)
next(5)
*/
```



이렇듯 대상 subject 내의 요소들이 순서대로 next event에 담겨 구독자에게 전달된다.

```swift
oddNumbers.onCompleted()
evenNumbers.onNext(8)
//출력값 #1
//next(8) 

evenNumbers.onCompleted()
//출력값 #2
//next(8)
//completed
```

 `#1`과 같이 merge 연산자의 대상 subject/Observable 중 일부가 completed event를 방출해도 나머지 subject/Observable들은 여전히 값을 구독자에게 전달한다.

 `#2`에서야 모든 대상 subject/Observable이 종료되는데, 그래야 비로소 completed 이벤트를 구독자에게 전달한다. 

merge 연산자는 병합한 모든 Observable로 부터 completed event를 받아야만 구독자로 completed 이벤트를 전달한다.  그 전까지는 계속해서 next event를 전달한다. 



```swift
source
  .merge()
  .subscribe { print($0) }
  .disposed(by: bag)

oddNumbers.onNext(3)
evenNumbers.onNext(4)

evenNumbers.onNext(6)
oddNumbers.onNext(5)

//error 이벤트 전달
oddNumbers.onError(MyError.error)

evenNumbers.onNext(8)

evenNumbers.onCompleted()

/*출력값 
next(1)
next(2)
next(3)
next(4)
next(6)
next(5)
error(error)
*/
```



하지만 error event의 경우에는 다르다. 대상 Observable 중 하나라도 error event를 전달하면 그 즉시 구독자에게 전달되고, 더 이상 다른 이벤트를 전달하지 않는다. 그래서 이전과 달리 8이 저장된 next event와 마지막 completed event는 구독자에게 전달되지 않는다. 

merge연산자로 병합할 수 있는 Observable의 갯수는 정해져있지 않다. 

만약 병합할 Observable의 갯수를 제한하고 싶다면 `.merge(maxConcurrent: Int)`메서드를 대신 사용하면 된다.

```swift
let oddNumbers = BehaviorSubject(value: 1)
let evenNumbers = BehaviorSubject(value: 2)
let negativeNumbers = BehaviorSubject(value: -1)

let source = Observable.of(oddNumbers, evenNumbers, negativeNumbers)

source
  .merge(maxConcurrent: 2) //#1
  .subscribe { print($0) }
  .disposed(by: bag)

oddNumbers.onNext(3)
evenNumbers.onNext(4)

evenNumbers.onNext(6)
oddNumbers.onNext(5)

negativeNumbers.onNext(-2)

/*출력값
next(1)
next(2)
next(3)
next(4)
next(6)
next(5)
*/
```

위의 코드를 보면 source가 방출하는 BehaviorSubject는 총 3개인데, `#1`에서 병합 가능한 Observable의 갯수를 두 개까지로 제한하고 있다. 

때문에 이미 앞에서 `oddNumbers`와 `evenNumbers`를 병합한 시점에 최대 갯수에 도달하기 때문에 `negativeNumbers`는 병합 대상에서 제외되고 당연히 방출하는 값도 처리하지 않는다. 

merge 연산자는 이렇게 병합 대상에서 제외한 Observable들을 Queue에 저장했다가, 병합 대상 중 하나가 completed event를 전달하면 순서대로 병합 대상에 추가한다. 



```swift
source
  .merge(maxConcurrent: 2)
  .subscribe { print($0) }
  .disposed(by: bag)

oddNumbers.onNext(3)
evenNumbers.onNext(4)

evenNumbers.onNext(6)
oddNumbers.onNext(5)

negativeNumbers.onNext(-2) //#2

oddNumbers.onCompleted() //#1

/*출력값
next(1)
next(2)
next(3)
next(4)
next(6)
next(5)
next(-2)
*/
```

 

`#1`에서 `oddNumbers`가 종료되자 Queue에 저장되어있던 `negativeNumbers `가 병합되는 것을 볼 수 있다. 

하지만 `oddNumbers`가 종료되기 이전인 `#2`시점에 이미 값을 방출했던 `negativeNumbers`의 `-2`라는 값이 어떻게 즉시 구독자에게 전달되었을까?

이는 `negativeNumbers`가 BehaviorSubject이기 때문이다. BehaviorSubject는 항상 하나의 기본값을 가지고, 가장 최근에 전달된 값으로 기본값을 업데이트한다. 때문에 `oddNumbers`가 completed event를 전달하여 종료된 시점에 병합된 `negativeNumbers`에 저장되어있던 `-2`라는 값이 병합과 동시에 구독자에게 전달되어 출력된 것이다. 



---

### combineLatest

![스크린샷 2020-06-09 오전 4.55.28](https://tva1.sinaimg.cn/large/007S8ZIlgy1gflijd1o2gj30nz0ltdis.jpg)

[공식 문서 설명](http://reactivex.io/documentation/operators/combinelatest.html)

위의 marvel diagram을 보면, 1에서 5까지의 정수를 방출하는 Observabler과 A~D까지의 알파벳 문자를 방출하는 Observable이 있다. 

combineLatest의 대상이 되는 이 두 Observable을 source Observable로 칭하기로 한다. combineLatest 연산자는 source Observable을 combine, 즉 결합한다. 소스 옵저버블을 결합한 후 combineLatest에 파라미터로 전달한 함수를 실행하고, 결과를 방출하는 새로운 Observable을 리턴한다. 다이어그램에서는 숫자와 문자를 연결해서 문자열을 리턴하고 있다.

combineLatest 연산자의 핵심은 연산자가 리턴한 Observable이 언제 event를 방출하는지 이해하는 것이다. 

제일 아래에 있는 stream은 연산자가 리턴한 Result Observable이다.

가장 위의 소스 옵저버블 중 숫자를 방출하는 Observable을 보면 맨 처음 1을  방출하고 있는데, 이 시점에는 문자 Observable이 어떤 문자도 event로 방출하지 않았기 때문에 Result Observable도 event를 방출하지 않는다. 

이어서, 문자를 방출하는 두 번째 Source Observable이 "A"를 방출하게되면 두 Source Observable 모두 하나씩 next event를 방출한 상태가 된다.

바로 이 순간에 두 Source Observable이 방출한 최신 next event를 대상으로 combineLatest의 파라미터로 전달한 함수를 실행하고, 그 결과가 combineLatest 연산자가 리턴하는 Result Observable을 통해 구독자에게 전달된다. 그래서 구독자는 "1A"가 저장된 next event를 받는다. 이후에는 source observable 중에서 하나라도 next event를 방출하면 다시 최근 값들을 대상으로 함수가 실행되고 Result Observable이 event를 방출한다. 

다음으로 숫자 옵저버블이 2를 방출하면 각 Source Observable의 최근 값들인 `2`, `A`를 대상으로 함수가 실행되어 구독자에게 `"2A"`가 전달된다. 

연산자의 이름이 combine이 아니라 combineLatest인 이유가 바로 이것이다. 오직 가장 마지막 값들만을 대상으로 함수가 실행되기 때문이다. 

![스크린샷 2020-06-09 오전 5.25.18](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfljed8kupj30mn0c6q6l.jpg)

combineLatest는 여러 형태가 있지만 가장 기본으로 사용되는 형태는 이러하다. 

두 개의 Observable을 파라미터 source1과 source2로 받은 뒤, resultSelector 파라미터로 클로저를 받는다. 이 클로저에는 source1과 source2가 방출하는 next event의 요소가 파라미터로 전달되고, 클로저의 실행 결과인 하나의 요소를 리턴한다. 이어 최종적으로 combineLatest 연산자는 실행 결과로 클로저가 리턴한 하나의 요소를 방출하는 Observable을 리턴한다.

![스크린샷 2020-06-09 오전 5.32.17](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfljli8o5ij30my0a1mzp.jpg)

오버로딩되어있는 다른 combineLatest 중에는 위와 같이 클로저를 파라미터로 받지 않는 경우도 있는데, 이 때에는 리턴형이 달라진다. 파라미터로 전달한 Observable들이 방출한 요소들을 하나의 튜플로 합친 다음 이 튜플을 방출하는 Observable을 리턴한다. 

combineLatest는 최대 8개까지 source Observable을 받을 수 있도록 선언되어있다. 파라미터의 수만 다를 뿐 동작 방식은 동일하다.

```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let greetings = PublishSubject<String>()
let languages = PublishSubject<String>()

Observable.combineLatest(greetings, languages) { "\($0 + " " + $1)" }
  .subscribe { print($0) }
  .disposed(by: bag)

greetings.onNext("Hi")
languages.onNext("RxSwift")
languages.onNext("Kotlin")
greetings.onNext("Yo,")
languages.onNext("Java!")

/* 출력값
next(Hi RxSwift)
next(Hi Kotlin)
next(Yo, Kotlin)
next(Yo, Java!)
*/

greetings.onCompleted() //#2
languages.onNext("SwiftUI")
//출력값
//next(Yo, SwiftUI)

languages.onCompleted() //#3
//출력값
//completed

```



위와 같이 동작한다. 만약 `#2`와 같이 하나의 Source Observable이 completed event를 전달했다면, completed event를 전달하기 바로 전의 next event를 사용한다. 따라서 직전 next event의 요소인 "Yo"를 사용하여 "Yo, SwiftUI"가 구독자에게 전달된 것이다. 

 `#3`에서처럼 모든 Source Observable이 completed event를 전달하면 구독자에게 completed event가 전달된다. 



```swift
greetings.onNext("Hi")
languages.onNext("RxSwift")
languages.onNext("Kotlin")
greetings.onNext("Yo,")
languages.onNext("Java!")

greetings.onError(MyError.error) //error event 전달 #4
languages.onNext("SwiftUI")
languages.onCompleted()

/*출력값
next(Hi RxSwift)
next(Hi Kotlin)
next(Yo, Kotlin)
next(Yo, Java!)
error(error)
*/
```



하지만 하나의 Source event라도 Error event를 전달한다면, 그 즉시 Observable이 종료되고 구독자에게 error event가 전달된다. 그래서 당연히 그 이후의 값들은 구독자에게 전달되지 않는다.



---

### zip



![스크린샷 2020-06-09 오전 5.50.28](https://tva1.sinaimg.cn/large/007S8ZIlgy1gflk49bqurj30nv0nngoe.jpg)

zip 연산자는 소스 옵저버블이 방출하는 요소를 결합한다. combineLatest 연산자는 가장 최근 요소들을 기준으로 클로저를 실행하지만, zip 연산자는 클로저에게 중복되는 요소를 전달하지 않는다. 반드시 index를 기준으로 짝을 맞춰 전달한다. 첫 번째 요소는 첫 번째 요소와 결합하고, 두 번째 요소는 두 번째 요소와 결합한다. 

 위의 마블 다이어그램을 보면 `숫자 observable`에서 `1`이 방출되었을 때 `문자 observable`은 아무 값도 방출하지 않은 시점이기 때문에 `Result Observable`은 어떤 결과도 방출하지 않는다. 이후 `문자 Observable`이 `A`를 방출하면 각각의 source observable이 첫 번째 요소를 방출한 시점이 된다. 이때 `zip` 연산자가 클로저를 실행하고 결과를 `Result Observable`로 전달한다. `Result Observable`은 전달 받은 결과를 `next event`에 담아 구독자에게 전달하는데, 그래서 이 경우 `1A`가 구독자에게 전달되었다.

이어서 `숫자 Observable`에서 `2`를 방출하고 있다. `combineLatest` 연산자 였다면 이 시점에 `Result Observable`이 클로저를 실행하고 결과값인 `2A`를 구독자에게 전달했겠지만, `zip` 연산자는 그렇지 않다. `숫자 Observable`의 두 번째 요소인 `2`는 반드시 `문자 Observable`의 두 번째 요소와 결합해야한다. 하지만 이 시점에는 `문자 Observable`이 두 번째 요소를 방출하지 않은 상태이기 때문에 어떤 Event도 방출하지 않고 `문자 Observable`이 두 번째 요소를 방출하기를 기다린다. 

그리고 `문자 Observable`이 두 번째 요소인 `B`를 방출하는 시점에 `zip` 연산자가 다시 클로저를 실행하고, 결과인 `2B`를 `Result Observable`로 전달한다.

숫자 Observable이 방출하는 마지막 요소 5는 구독자에게 전달되지 않는다. 그에 대응하는 문자 Observable의 요소가 방출되지 않기 때문이다. 

zip 연산자의 동작처럼 Source Observable이 방출하는 요소들의 순서를 1:1로 대응, 일치시켜 결합하는 것을 Indexed Sequencing이라고 한다. 



```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let numbers = PublishSubject<Int>()
let strings = PublishSubject<String>()

Observable.zip(numbers, strings) { "\($0) - \($1)" }
  .subscribe { print($0) }
  .disposed(by: bag)

numbers.onNext(1)
strings.onNext("One")
numbers.onNext(2) // #1
//출력값
//next(1 - One)

strings.onNext("two") // #2
//출력값
//next(2 - two)
```



combineLatest 였다면 위 코드의 첫 번째 출력값이 `next(1 - One)`, `next(2 - One)` 두 개여야 하겠지만, zip 연산자는 Indexed Sequencing을 요구하기 때문에 `next(1 - One)`만 출력되고, `numbers`가 `#1`에서 방출한 `2`는 구독자에게 전달되지 않은 채 `strings`의 두 번째 요소가 방출되기를 기다렸다가, `#2`에서 `strings`가 두 번째 요소인 `two`를 방출하자 그때서야 `next(2 - two)`가 구독자에게 전달되고 있다.

항상 방출된 순서대로 짝을 맞춘다는 것이 중요하다.



```swift
...(전략)

numbers.onCompleted() // #1
strings.onNext("three")

/*출력값
next(1 - One)
next(2 - two)
*/

strings.onCompleted() // #2
//출력값
//completed
```



그리고 `#1`처럼 Source Observable 중 어느 하나라도 Completed Event를 전달하게되면 다른 Source Observable이 next event를 방출하더라도 짝을 맞춰야할 Source Observable은 이미 종료되었기 때문에 구독자에게 전달되지 않는다. combineLatest와 다른 점이다. 

최종적으로 구독자에게 completed event가 전달되는 시점은 `#2`에서처럼 모든 Source Observable이 completed event를 전달하는 시점이다. 

Error Event의 경우 combineLatest 연산자와 마찬가지로 어느 한 Source Observable이라도 Error Event를 전달할 경우 즉시 구독자에게 Error Event가 전달되며, 그 이후에 다른 Source Observable들이 next event를 방출하더라도 그 요소들은 구독자에게 전달되지 않는다. 



---

### withLatestFrom

```swift
triggerObservable.withLatestFrom(dataObservable)
```

이 연산자는 위와 같은 방식으로 사용한다. 연산자를 호출하는 Observable을 triggerObservable이라고 부르고, 파라미터로 전달하는 Observable을 dataObservable이라고 부른다. triggerObservable이 next event를 방출하면, data Observable이 가장 최근에 방출한 next event를 구독자에게 전달한다. 예를 들어 회원가입 버튼을 탭 하는 시점에 textField에 입력된 값을 가져오는 기능을 구현할 때 사용할 수 있다. 

![스크린샷 2020-06-10 오전 4.22.41](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfmn7fuykfj30na0ncjyu.jpg)

이 연산자는 두 형태로 사용한다.

 첫 번째는 data Observable과 클로저를 파라미터로 받는다. 클로저에는 두 Observable이 방출하는 요소가 전달되고, 여기에서 결과를 리턴한다. 연산자가 최종적으로 리턴하는 Observable은 클로저가 리턴하는 결과를 방출한다. 

두 번째 형태는 trigger observable에서 next event를 전달하면, 파라미터로 전달한 데이터 Observable이 가장 최근에 방출한 next event를 가져온다. 그 다음 이벤트에 포함된 요소를 방출하는 Observable을 리턴한다. 

```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()

trigger.withLatestFrom(data)
  .subscribe { print($0) }
  .disposed(by: bag)

data.onNext("first") //#1
data.onNext("last")

trigger.onNext(())
trigger.onNext(())
//출력값
//next(last)
//next(last)

data.onCompleted() //#2
trigger.onNext(())
//출력값
//next(last)

```

data Observable이 `#1`에서 방출한 "first"는 구독자에게 전달되지 않고, trigger Observable이 next 이벤트를 방출한 시점의 data Observable의 가장 최근 방출된 next event의 요소인 "last"만이 전달된다.

또한 코드에서처럼, withLatestFrom 연산자는 trigger Observable이 next event를 방출할 때마다 data Observable의 최근 next event 값을 구독자에게 전달한다. 때문에 trigger Observable에 next event를 전달할 때마다 구독자에게는 data Observable의 최근 방출 값인 "last"가 전달되고 있다. 중복되는 값이어도 상관 없이 전달된다. 

이어 `#2`에서 data Observable에 Completed event를 전달한 뒤 trigger Observable에 next event를 전달했는데, 여전히 data Observable의 가장 최근 next Event 요소인 "last"가 구독자에게 전달되는 것을 볼 수 있다. 

```swift
data.onError(MyError.error) //#2-1
//출력값
//error(error)

trigger.onNext(()) //#3
//출력값 없음
```



`#2`의 completed event 대신 `#2-1`처럼 data Observable에 error event를 전달할 경우, trigger Observable이 next event를 방출하기 전이더라도 구독자에게 즉시 error event가 전달된다. 그 이후에는 trigger Observable이 Next Event를 방출하더라도 더 이상 어떤 값도 구독자에게 전달되지 않는다. 



```swift
trigger.onCompleted()
//출력값
//completed
```

이밖에 trigger Observable이 completed/error event를 전달할 경우에는 언제나 즉시 구독자에게 event가 전달된다.

---

### sample

```swift
dataObservable.sample(triggerObservable)
```

sample 연산자는 data Observable에서 연산자를 호출하고, 파라미터로 trigger Observable을 전달한다.

 trigger Observable에서 next event를 전달할 때마다 data Observble이 자신의 최근 next event를 방출한다. 이 부분은 withLatestFrom 연산자와 동일하다. 하지만 동일한 next event를 반복해서 방출하지 않는다는 차이가 있다. 



```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let trigger = PublishSubject<Void>()
let data = PublishSubject<String>()

data.sample(trigger)
  .subscribe { print($0) }
  .disposed(by: bag)

trigger.onNext(())
data.onNext("first")
//출력값 없음

data.onNext("last")
trigger.onNext(())
//출력값
//next(last)

trigger.onNext(()) //#1
//출력값 없음

data.onCompleted() //#2
//출력값 없음

trigger.onNext(()) //#3
//출력값
//completed
```



`#1`의 trigger 연산자가 연달아 next event를 방출하더라도 중복되는 값이 다시 방출되지는 않는다.

`#2`처럼 data Observable이 completed event를 전달하더라도 구독자에게는 어떤 값도 전달되지 않는다.

`#3`에서 trigger Observable이 next event를 방출하는 시점에, data Observable의 completed event가 구독자에게 전달된다. withLatestFrom 연산자는 이 경우 completed 대신 data Observable이 completed 이벤트를 방출하기 이전의 next event 값을 방출하지만, sample 연산자는 최근 next event 대신 completed 이벤트를 구독자에게 전달한다.

data Observable이 Error event를 전달할 경우에는 withLatestFrom 연산자와 마찬가지로 trigger Observable의 next event 방출 시점과 무관하게 그 즉시 구독자에게 error event를 전달한다.



---

### switchLatest

가장 최근 Observable이 방출하는 Event를 구독자에게 전달한다. 어떤 Observable이 가장 최근 Observable인지 이해해야한다. 

![스크린샷 2020-06-10 오전 4.56.29](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfmo6je228j30m60bgq5n.jpg)

이 연산자는 파라미터가 없다. 그리고 주로 Observable을 방출하는 Observable에서 사용한다.

```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let a = PublishSubject<String>()
let b = PublishSubject<String>()

let source = PublishSubject<Observable<String>>()

source
  .switchLatest()
  .subscribe { print($0) }
  .disposed(by: bag)

a.onNext("1")
b.onNext("b")

source.onNext(a) // #1
//출력값 없음

a.onNext("2") // #2
b.onNext("b")
//출력값
//next(2)

source.onNext(b) // #3
a.onNext("3")
b.onNext("c")
//출력값
//next(c)

a.onCompleted() // #4
b.onCompleted()
//출력값 없음

source.onCompleted() // #5
//출력값
//completed
```



switchLatest 연산자는 source Observable이 가장 최근에 방출한 Observable을 구독하고, 여기에서 전달하는 next event를 방출하는 새로운 Observable을 리턴한다. 

`#1`에서 source Observable에 `a` Observable을 전달하면, `a` Observable이 최신 Observable이 된다. switchLatest 연산자는 최신 Observable인 `a`를 구독하고, `a`에서 방출되는 event를  구독자에게 전달한다. 

이때 `#1`보다 이전에 `a`로 전달했던 "1"은 출력되지 않지만, 이후 전달한 `#2`의 "2"는 즉시 구독자에게 전달된다. 

`#3`에서 source Observable에 next event로 `b`를 전달하면, switchLatest 연산자는 `a`에 대한 구독을 종료하고, `b`를 구독하기 시작한다. 

`#3`의 출력값을 보면 switchLatest 연산자의 최신 Observable인 `b`가 방출한 next event인 "c"가 구독자에게 전달된 것을 확인할 수 있다. 

`#4`처럼 `a`와 `b` 모두에게 Completed event를 전달해도 switchLatest 연산자는 구독자에게 completed event를 전달하지 않는다. 

`#5`와 같이 source Observable에 completed 연산자를 전달해야 구독자에게 completed event가 전달된다.

하지만 Error event의 경우에는, switchLatest 연산자의 최신 Observable이 error event를 전달하는 즉시 구독자에게 error 이벤트를 전달한다.



---

### reduce

이 연산자는 scan 연산자와 비교하면 쉽게 이해할 수 있다. 

아래는 scan 연산자를 사용한 예제 코드이다.

```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let o = Observable.range(start: 1, count: 5)

print("== scan")

o.scan(0, accumulator: +)
   .subscribe { print($0) }
   .disposed(by: bag)
```

scan 연산자는 위의 코드에서처럼 기본값과 source Observable이 방출하는 값을 대상으로 두 번째 파라미터로 전달한 accumulator closures를 실행한 이후 해당 클로저의 실행 결과를 Observable을 통해 방출하고, 다시 클로저로 전달한다. 

source Observable이 새로운 요소를 방출하면, 이전 결과와 함께 클로저를 다시 실행한다. 이 과정이 반복되기 때문에 source Observable이 방출하는 이벤트의 수와 구독자로 전달되는 이벤트의 수가 같다. 

주로 작업의 결과를 누적시키면서 중간 결과와 최종 결과가 모두 필요한 경우에 사용한다. 



![스크린샷 2020-06-10 오전 5.19.39](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfmoumz6zqj30mq0c9dj9.jpg)

reduce 연산자는 seed value와 accumulator 클로저를 파라미터로 받는다.   

seed value와  source Observable이 방출하는 요소를 대상으로 클로저를 실행하고, result Observable을 통해 결과를 방출한다. 이 부분은 scan 연산자와 동일하다. accumulator 클로저의 실행결과가 클로저로 다시 전달되는 것 역시 동일하다. 

하지만 reduce 연산자는 result Observable을 통해 최종 결과 하나만 방출한다. 중간 결과까지 모두 방출하는 scan 연산자와는 차이가 있다. 

![스크린샷 2020-06-10 오전 5.23.45](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfmoywzgr2j30m20ecn1d.jpg)

그리고 세 번째 파라미터로 클로저를 받는 reduce 연산자도 있는데, 최종 결과를 다른 형식으로 바꾸고 싶을 때 주로 사용한다. reduce 연산자 뒤에 map 연산자를 연결하는 것과 동일한 효과를 낼 수 있다. 

```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let o = Observable.range(start: 1, count: 5)

print("== reduce")

o.reduce(0, accumulator: +)
.subscribe { print($0) }
.disposed(by: bag)
/* 출력값
== reduce
next(15)
completed
*/
```

결과를 보면, scan 연산자를 사용했을 때와는 달리 이번에는 최종 결과인 15만 출력된다. reduce 연산자와 scan 연산자의 가장 큰 차이이다. 중간 결과와 최종 결과가 모두 필요하다면 scan 연산자를 사용하고, 최종 결과 하나만 필요하다면 reduce 연산자를 사용한다. 

---

## 14. Conditional Operators

### amb

---

amb는 두 개 이상의 source Observable 중에서 가장 먼저 next event를 전달한 Observable을 구독하고, 나머지는 무시한다. 

![스크린샷 2020-06-10 오전 5.49.52](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfmpq2jg3zj30l90heq47.jpg)

여러 개의 Source Observable 중에서 두 번째 source observable이 가장 먼저 next event를 전달하기 때문에, 나머지 Observable은 무시되고 두 번째 Observable만 구독자에게 전달된다. 

amb 연산자를 이용하면 여러 서버로 한 번에 요청을 전달하고, 가장 빠른 응답을 처리하는 패턴을 구현할 수 있다. 

![스크린샷 2020-06-10 오전 5.55.02](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfmpvig8acj30nj0ng797.jpg)

아랫 부분에 선언되어있는 amb 연산자를 보면, 하나의 Observable을 파라미터로 받는다. 두 Observable 중에서 먼저 event를 전달하는 Observable을 구독하고, 이 Observable의 이벤트를 구독자에게 전달하는 새로운 Observable을 리턴한다. 

만약 세 개 이상의 Observable을 대상으로 amb 연산자를 사용해야 한다면, 윗 부분의 Type method로 구현되어있는 연산자를 사용한다. 이 때는 모든 source Observable을 배열 형태로 전달해야한다.

```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let a = PublishSubject<String>()
let b = PublishSubject<String>()
let c = PublishSubject<String>()

a.amb(b)
  .subscribe { print($0) }
  .disposed(by: bag)

a.onNext("A")
b.onNext("B")
b.onCompleted()
//출력값
//next(A)
a.onCompleted()
//출력값
//completed
```



이 경우 `a` subject가 `b` subject 보다 먼저 event를 방출한다. 그래서 amb는 `a` Observable을 구독하고,  `b`는 무시한다. 때문에 `a` subject가 전달한 next event만이 구독자에게 전달되었다. `b`가 전달한 completed event 역시 무시된다. 

반면 a가 전달한 completed 이벤트는 즉시 구독자에게 전달되었다.

```swift
Observable.amb([a, b, c])
  .subscribe { print($0) }
  .disposed(by: bag)
```

만약 세 개 이상의 Observable에 대해 amb 연산자를 사용하고 싶다면 위와 같이 사용하면 된다. 



---

## 15. Time-based Operators

### interval

특정 주기마다 정수를 방출하는 Observable이 필요하다면 이 연산자를 활용한다. 

![스크린샷 2020-06-11 오전 4.34.37](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfnt5xj5bzj30nj0ffwis.jpg)

interval 연산자는 Type Method로 구현되어있다. 

첫 번째 파라미터로 반복 주기를 받는다. 두 번째 파라미터는 정수를 방출할 스케쥴러를 받는다. 

연산자가 리턴하는 Observable은 지정된 주기마다 정수를 반복적으로 방출한다. 

종료시점을 지정하지 않기 때문에, 직접 dispose 하기 전까지 계속해서 방출한다. 방출하는 정수의 형태는 Int로 제한되지 않는다. 요소의 형식이 RxAbstarctInteger 형식인데, 이는 FixedWidthInteger와 같다. 그래서 Int를 포함한 다른 정수 형식을 모두 사용할 수 있다. 반대로 Double이나 문자열 형태는 사용할 수 없다. 

```swift
let i = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)

let subscription1 = i.subscribe { print("1 >> \($0)") } //#1

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
  subscription1.dispose()
} //#2

/*출력값
1 >> next(0)
1 >> next(1)
1 >> next(2)
1 >> next(3)
1 >> next(4)
*/

var subscription2: Disposable?

DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
  subscription2 = i.subscribe { print("2 >> \($0)") }
} // #3

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
  subscription2?.dispose()
} // #4

/* 출력값 
1 >> next(0)
1 >> next(1)
1 >> next(2)
2 >> next(0)
1 >> next(3)
2 >> next(1)
1 >> next(4)
2 >> next(2)
2 >> next(3)
*/
```



`#1`과 같이 interval 연산자를 사용한 Observable을 구독하면, 1초마다 무한하게 정수가 구독자에게 전달된다. interval 연산자에는 종료시점이 없기 때문에 직접 종료해줘야하는데, `#2`와 같은 코드를 이용하여 종료시킬 수 있다. 

interval 연산자가 생성하는 Observable은 내부에 타이머를 가지고 있다. 이 타이머가 시작되는 시점은 생성 시점이 아니다. 바로 구독자가 구독을 시작하는 시점이다. 

그래서 Observable에 새로운 구독자가 추가될 때마다 새로운 타이머가 생성된다. 

i에 새로운 구독자를 `#3`과 같이 2초 후에 추가하기로 하고, `#4`처럼 7초 뒤에 dispose 시키기로 하고 실행시킨 코드의 출력값을 보면 새로운 구독자가 추가될 때 새로운 타이머가 생성, 시작된다는 것을 알 수 있다. 



---

### timer

timer 연산자는 interval 연산자와 마찬가지로 정수를 반복적으로 방출하는 Observable을 생성한다. 하지만 지연 시간과 반복 주기를 모두 지정할 수 있고, 두 값에 따라 동작 방식이 달라진다. 

![스크린샷 2020-06-11 오전 4.51.44](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfntnresu2j30ok0f442h.jpg)

timer 연산자도 Type Method로 구현되어 있다. 그리고 리턴되는 Observable이 방출하는 요소는 FixedWidthInteger 프로토콜을 채택한 형식으로 제한되어있다. 

파라미터는 총 세 개가 선언되어있다. 

첫 번째 파라미터는 첫 번째 요소가 방출되는 시점까지의 상대적인 시간이다. 구독을 시작하고 첫 번째 요소가 구독자에게 전달되기까지의 시간을 가리킨다. 첫 번째 파라미터에 1초를 전달하면, 구독을 시작하고 1초 뒤에 요소가 전달된다. 

두 번째 파라미터는 반복 주기이다. 반복 주기는 기본값이 nil로 선언되어있다. 이 값에 따라서 timer 연산자의 동작 방식이 달라진다. 

세 번째 파라미터에는 타이머가 동작할 스케쥴러를 전달한다. 



```swift
let bag = DisposeBag()

Observable<Int>.timer(.seconds(1), scheduler: MainScheduler.instance) // #1
  .subscribe { print($0) }
  .disposed(by: bag)

/*출력값
next(0)
completed
*/
```

`#1`에서 timer 연산자에 첫 번째 파라미터로 전달한 `.seconds(1)`은  반복주기가 아니다. 구독 후 구독자에게 1초 후에 전달된다는 '지연 시간'을 나타낸다. 두 번째 파라미터가 반복 주기인데, `#1`에서처럼 생략되어있을 경우에는 하나의 요소만 방출한 뒤 종료된다. 

그래서 실행결과를 보면 코드가 실행된지 1초 후에 하나의 요소를 방출하고 구독자에게 completed 이벤트가 즉시 전달되며 종료된다. 



```swift
let bag = DisposeBag()


Observable<Int>.timer(.seconds(1), period: .milliseconds(500), scheduler: MainScheduler.instance)
.subscribe { print($0) }
.disposed(by: bag)

/*출력값
next(0)
next(1)
next(2)
next(3)
next(4)
next(5)
next(6)
next(7)
next(8)
...
(이하 생략)
*/
```

이번에는 두 번째 파라미터인 반복주기를 전달한 timer 연산자의 용례이다. 

이 경우 코드 실행 후 1초 뒤부터 0.5초마다 무한정 정수를 방출한다.

timer 연산자도 종료 시점을 지정하지 않기 때문에 직접 종료시켜주어야 한다.



---



### timeout 

![스크린샷 2020-06-11 오전 5.29.32](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfnur2x9w3j30nu0bdjul.jpg)

timeout 연산자는 source Observable이 방출하는 모든 요소에 timeout 정책을 적용한다. 첫 번째 파라미터로 timeout 시간을 전달하는데, 이 시간 안에 next event를 방출하지 않으면 error event를 전달하고 종료된다. 

error 형식은 표기되어있는 것처럼 `RxError.timeout`이다. 반대로 timeout 시간 이내에 새로운 이벤트를 방출하면, 구독자에게 그대로 전달한다. 



![스크린샷 2020-06-11 오전 5.31.46](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfnutfokskj30ml0d4wiu.jpg)

세 개의 파라미터를 받는 timeout 연산자도 선언되어있는데, 두 번째 파라미터로 Observable을 전달하는 것을 제외하면 나머지는 동일하다. 

여기에서는 timeout이 발생하면 error event를 전달하는 것이 아니라 구독 대상을 두 번째 파라미터로 전달한 Observable로 교체한다. 

```swift
let bag = DisposeBag()

let subject = PublishSubject<Int>()

subject.timeout(.seconds(3), scheduler: MainScheduler.instance) // #1
  .subscribe { print($0) }
  .disposed(by: bag)

Observable<Int>.timer(.seconds(1), period: .seconds(1), scheduler: MainScheduler.instance)
// #2
  .subscribe(onNext: { subject.onNext($0) })
  .disposed(by: bag)

/*출력값
next(0)
next(1)
next(2)
next(3)
next(4)
*/
```



`#1`은 PublishSubject인 subject에 timeout 연산자로 제한 시간 3초를 준 코드이다. 

`#2`에서 timer 연산자를 이용해서 '코드 실행 후 1초 뒤부터', '1초마다 반복해서', 'MainScheduler에서 실행' 하는 정수 방출 Observable을 만들고, 이 Observable이 방출하는 next event의 element를 subject에게 next event에 담아 전달하고 있다.

이 때는 timeout 시간 이내에 새로운 next event가 subject로 전달되기 때문에 계속해서 구독자에게 전달되고, 에러 이벤트는 발생하지 않는다. 



```swift
let bag = DisposeBag()

let subject = PublishSubject<Int>()

subject.timeout(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print($0) }
  .disposed(by: bag)

Observable<Int>.timer(.seconds(5), period: .seconds(1), scheduler: MainScheduler.instance)
  .subscribe(onNext: { subject.onNext($0) })
  .disposed(by: bag)

//출력값
//error(Sequence timeout.)
```

하지만 만약 이 코드처럼 Observable이 정수를 방출하기 시작하는 시점을 코드 실행 후 5초 뒤로 설정하면, 코드 실행 3초 후에 error event를 구독자에게 전달하고 종료한다. 3초 내에 이벤트가 전달되지 않았기 때문이다. 



```swift
let bag = DisposeBag()

let subject = PublishSubject<Int>()

subject.timeout(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print($0) }
  .disposed(by: bag)

Observable<Int>.timer(.seconds(2), period: .seconds(5), scheduler: MainScheduler.instance)
  .subscribe(onNext: { subject.onNext($0) })
  .disposed(by: bag)

/*출력값
next(0)
error(Sequence timeout.)
*/
```

이 코드의 경우 Observable이 코드 실행 2초 뒤에 최초의 next event로 정수를 방출하고, 그 다음부터는 5초의 반복 주기로 정수를 방출한다. 

이 경우 Observable의 첫 번째 next event는 subject의 timeout 시간 내에 전달되기 때문에 subject의 구독자에게 전달되지만, Observable의 두 번째 이벤트는 그로부터 5초 후에 방출되기 때문에 timeout의 제한시간을 초과하게 된다. 그래서 이 경우에는 error 이벤트를 방출한다. 





```swift
let bag = DisposeBag()

let subject = PublishSubject<Int>()

// #1
subject.timeout(.seconds(3), other: Observable.just(0), scheduler: MainScheduler.instance)
.subscribe { print($0) }
.disposed(by: bag)

// #2
Observable<Int>.timer(.seconds(2), period: .seconds(5), scheduler: MainScheduler.instance)
  .subscribe(onNext: { subject.onNext($0) })
  .disposed(by: bag)

/*출력값
next(0) //정수 방출 Observable이 subject에게 전달한 이벤트
next(0) //timeout이 발생하여 구독 대상이 된 timeout의 'other' Observable이 방출한 이벤트
completed //other Observable이 just 연산자로 0을 방출한 뒤 전달한 이벤트
*/
```

만약 timeout의 제한시간을 초과했을 때 구독자에게 error event 대신 0을 전달하고 싶다면 timeout의 두 번째 파라미터를 활용하면 된다. 

`#1`에서 timeout의 두 번째 연산자로 0을 방출하고 종료하는 Observable을 전달해주었는데, 

결과를 보면 구독자에게 next event가 두 번 전달되고 completed event가 전달되었다. 



출력값의 첫 번째 이벤트는 subject가 `#2`의 Observable로 부터 전달 받은 이벤트이다. 하지만 이후 전달 받을 이벤트는 5초 뒤에 방출되기 때문에 그 전에 timeout의 제한 시간을 초과한다. 

출력값의 두 번째 next event는 정수를 방출하는 Observable로부터 전달 받은 것이 아니라, timeout으로 인해 구독 대상이 된 timeout 연산자의 두 번째 파라미터인 other Observable이 방출한 것이다. timeout의 두 번째 파라미터로 Observable을 전달하면, timeout이 발생한 시점에 그 Observable이 구독 대상으로 설정된다. 그리고 이 Observable이 전달하는 event가 구독자에게 전달된다. 

이어서 마지막으로 other Observable이 just(0)을 방출한 뒤 전달한 completed 이벤트가 구독자에게 전달되었다. 



---

### delay



![스크린샷 2020-06-11 오전 6.21.01](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfnwwcmig1j30n50cc0w0.jpg)

delay 연산자는 next event가 구독자로 전달되는 시점을 지정한 시간만큼 지연시킨다. 

첫 번째 파라미터에는 지연시킬 시간을 전달하고, 

두 번째 파라미터에는 delay 타이머를 실행할 스케쥴러를 전달한다. 

연산자 리턴하는 Observable은 원본 Observable과 동일한 형식을 가지고 있지만, next event가 구독자에게 전달되는 시점이 첫 번째 파라미터에 전달한 시간만큼 지연된다. 

그리고 위 설명에 나와있는대로, error event는 지연되지 않고 즉시 전달된다. 

```swift
let bag = DisposeBag()

func currentTimeString() -> String {
  let f = DateFormatter()
  f.dateFormat = "yyyy-MM-dd HH:mm:ss.SSS"
  return f.string(from: Date())
}

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .take(10)
  .debug()
  .delay(.seconds(5), scheduler: MainScheduler.instance) //#1
  .subscribe { print(currentTimeString(), $0) } //#2
  .disposed(by: bag)

/* 출력값
2020-06-11 06:24:50.787: delay.playground:40 (__lldb_expr_494) -> subscribed
2020-06-11 06:24:51.814: delay.playground:40 (__lldb_expr_494) -> Event next(0)
2020-06-11 06:24:52.814: delay.playground:40 (__lldb_expr_494) -> Event next(1)
2020-06-11 06:24:53.814: delay.playground:40 (__lldb_expr_494) -> Event next(2)
2020-06-11 06:24:54.814: delay.playground:40 (__lldb_expr_494) -> Event next(3)
2020-06-11 06:24:55.815: delay.playground:40 (__lldb_expr_494) -> Event next(4)
2020-06-11 06:24:56.814: delay.playground:40 (__lldb_expr_494) -> Event next(5)

2020-06-11 06:24:56.815 next(0) 
// #3

2020-06-11 06:24:57.814: delay.playground:40 (__lldb_expr_494) -> Event next(6)
2020-06-11 06:24:57.816 next(1)
2020-06-11 06:24:58.814: delay.playground:40 (__lldb_expr_494) -> Event next(7)
2020-06-11 06:24:58.817 next(2)
2020-06-11 06:24:59.814: delay.playground:40 (__lldb_expr_494) -> Event next(8)
2020-06-11 06:24:59.818 next(3)
2020-06-11 06:25:00.814: delay.playground:40 (__lldb_expr_494) -> Event next(9)
2020-06-11 06:25:00.815: delay.playground:40 (__lldb_expr_494) -> Event completed
2020-06-11 06:25:00.815: delay.playground:40 (__lldb_expr_494) -> isDisposed
2020-06-11 06:25:00.819 next(4)
2020-06-11 06:25:01.820 next(5)
2020-06-11 06:25:02.822 next(6)
2020-06-11 06:25:03.823 next(7)
2020-06-11 06:25:04.825 next(8)
2020-06-11 06:25:05.826 next(9)
2020-06-11 06:25:05.827 completed
*/
```

위 코드는 정수를 매 초 방출하는 Observable을 만들고, take 연산자를 이용하여 전달할 next event를 10개로 제한한 것이다. 

그 상태에서 `#1`처럼 delay 연산자를 호출하는데, 지연 시킬 시간은 5초로 설정했다. delay 연산자는 구독시점을 연기하지는 않는다. 

구독자가 추가되면 바로 시퀀스가 시작된다. 로그를 보면 Observable이 1초마다 계속 next event를 방출하고 있다는 걸 확인할 수 있다. 하지만 `#2`의 구독자에서 추가한 로그는 출력되지 않고 있다. 

구독자에서 추가한 로그는 `#3`에서 확인할 수 있는데, 원본 Observable이 최초의 event를 방출한지 5초 후에 출력되었다. 다시 말해 원본 Observable이 방출한 next 이벤트가 5초 뒤에 구독자에게 전달된 것이다.

delay 연산자는 이렇게 원본 Observable에서 next event가 방출된 다음에 구독자로 전달되는 시점을 지연시킨다. (구독한 시점부터 바로 지연시키는 게 아님)



---

### delaySubscription



만약 구독자로 전달되는 시점이 아니라 구독 시작시점 자체를 지연시키고 싶다면 delaySubscription 연산자를 사용한다. 

```swift
let bag = DisposeBag()

func currentTimeString() -> String {
  let f = DateFormatter()
  f.dateFormat = "yyyy-MM-dd HH:mm:ss.SSS"
  return f.string(from: Date())
}

Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
  .take(10)
  .debug()
  .delaySubscription(.seconds(7), scheduler: MainScheduler.instance)
  .subscribe { print(currentTimeString(), $0) }
  .disposed(by: bag)

/*출력값
2020-06-11 06:37:02.302: delaySubscription.playground:40 (__lldb_expr_501) -> subscribed
2020-06-11 06:37:03.304: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(0)
2020-06-11 06:37:03.304 next(0)
2020-06-11 06:37:04.304: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(1)
2020-06-11 06:37:04.304 next(1)
2020-06-11 06:37:05.303: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(2)
2020-06-11 06:37:05.304 next(2)
2020-06-11 06:37:06.304: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(3)
2020-06-11 06:37:06.305 next(3)
2020-06-11 06:37:07.305: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(4)
2020-06-11 06:37:07.305 next(4)
2020-06-11 06:37:08.304: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(5)
2020-06-11 06:37:08.304 next(5)
2020-06-11 06:37:09.304: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(6)
2020-06-11 06:37:09.304 next(6)
2020-06-11 06:37:10.304: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(7)
2020-06-11 06:37:10.304 next(7)
2020-06-11 06:37:11.304: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(8)
2020-06-11 06:37:11.305 next(8)
2020-06-11 06:37:12.304: delaySubscription.playground:40 (__lldb_expr_501) -> Event next(9)
2020-06-11 06:37:12.304 next(9)
2020-06-11 06:37:12.305: delaySubscription.playground:40 (__lldb_expr_501) -> Event completed
2020-06-11 06:37:12.305 completed
2020-06-11 06:37:12.305: delaySubscription.playground:40 (__lldb_expr_501) -> isDisposed
*/
```



이 경우 코드 실행 이후 delaySubscription의 파라미터로 준 7초 동안은 아무런 로그도 출력되지 않는다. 그러다가 7초가 지나면 원본 Observable이 next event를 방출하기 시작한다. 

그렇게 방출되기 시작한 next event들은 어떠한 지연도 없이 구독자에게 바로 전달된다. 

delaySubscription은 구독 시점을 지연시킬 뿐 next event가 전달되는 시점은 지연시키지 않는다.





---

## 16. Sharing Subscription

RxSwift에서는 하나의 Observable을 여러 구독자가 구독할 경우 모든 구독자에 대해 각각의 구독 시점마다 Observable의 개별적인 시퀀스가 시작되고 전달된다. 

하지만 어떤 경우에는 모든 구독자가 하나의 시퀀스를 공유하는 것이 더 효율적일 때가 있다.

그래서 RxSwift에는 구독 공유를 통해 불필요한 작업을 방지할 수 있는 다양한 연산자가 있다.

---

### multicast Operator



```swift
let bag = DisposeBag()
let subject = PublishSubject<Int>()

let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5)

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

/* 출력값
🔵 next(0)
🔵 next(1)
🔵 next(2)
🔵 next(3)
🔴 next(0)
🔵 next(4)
🔵 completed
🔴 next(1)
🔴 next(2)
🔴 next(3)
🔴 next(4)
🔴 completed
*/
```

 위의 코드는 1초마다 한 개씩 총 5개의 정수를 방출하는 Observable에 두 개의 구독자가 추가되어있는 형태이다. 다만 두 번째 구독은 `delaySubscription`연산자를 통해 3초 지연되고 있다. 

Observable은 새로운 구독자가 구독을 시작할 때마다 새로운 시퀀스가 시작된다. 출력값을 보면 두 개 구독자가 0에서 4까지 5개의 정수를 각각 출력한다. 두 개의 시퀀스가 개별적으로 시작되었고, 서로 공유되지 않고 있다. 이것이 RxSwift의 가장 규칙이다. 



이 때 여러 구독자에게 하나의 시퀀스를 공유하는 연산자 중 하나가 multicast 연산자이다. 

![스크린샷 2020-06-12 오전 6.28.09](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp22ga1xjj30og0xt48b.jpg)

multicast 연산자는 Subject를 파라미터로 받는다. 

원본 Observable이 방출하는 이벤트는 구독자에게 전달되는 것이 아니라 이 서브젝트로 전달된다. 그리고 subject는 전달 받은 이벤트를 등록된 다수의 구독자에게 전달한다. 기본적으로 unicast 방식(수신자와 발신자가 1:1관계인 네트워크 통신의 한 방식)으로 동작하는 Observable을 multicast 방식으로 바꿔주는 것이다. 

이것을 위해서 특별한 형식의 Observable을 리턴한다. multicast 연산자의 리턴 형식은 ConnectableObservable로 선언되어 있는 걸 볼 수 있는데, ConnectableObservable은 일반적인 Observable과 구분되는 특징을 가지고 있다. 

일반 Observable은 구독자가 추가되면 새로운 시퀀스가 시작된다.(== 이벤트 방출을 시작한다.) 

하지만 ConnectableObservable은 시퀀스가 시작되는 시점이 다르다. 구독자가 추가되어도 시퀀스가 시작되지 않는다. Connect method를 실행하는 시점에 시퀀스가 시작된다. 

따라서 원본 Observable이 전달하는 이벤트는 구독자에게 바로 전달되는 것이 아니라 첫 번째 파라미터로 전달한 subject로 전달된다. 그리고나서 이 subject가 등록된 모든 구독자들에게 이벤트를 전달한다.

이렇게 동작하기 때문에 모든 구독자가 등록된 이후에 하나의 시퀀스를 시작하는 패턴을 구현할 수 있다. 

ConnectableObservableAdaptor는 원본 Observable과 subject를 연결해주는 특별한 class 이다.



```swift
let bag = DisposeBag()
let subject = PublishSubject<Int>()

let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).multicast(subject) // #1

source
  .subscribe { print("🔵", $0) }
  .disposed(by: bag)

source
  .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
  .subscribe { print("🔴", $0) }
  .disposed(by: bag)
//여기까지 실행시 출력값 없음

source.connect() //#2
/*출력값
🔵 next(0)
🔴 next(0)
🔵 next(1)
🔴 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
*/
```

이제 위의 코드의 `#1`부분에 multicast 연산자를 사용하고 파라미터로 PublishSubject인 `subject`를 전달한다. 이러면 `source`에는 일반 Observable이 아니라 ConnectableObservable이 저장된다. 

그리고 코드를 실행해보면, 아무것도 출력되지 않는다. 

왜냐면 ConnectableObservable은 단순히 구독자가 추가되었다고 해서 시퀀스를 시작하지 않기 때문이다. 반드시 `#2`에서처럼 원본 Observable에 대해 connect 메소드를 명시적으로 실행해야 시퀀스가 시작된다. 

- multicast 연산자의 동작 순서
  - 원본 Observable에서 시퀀스가 시작되면, 
  - 모든 이벤트는 파라미터로 전달한 subject로 전달된다. 그리고 이 subject는 등록된 모든 구독자에게 이벤트를 전달한다. 
  - 여기까지의 모든 과정은 connect method가 실행되는 시점에 시작된다.

`#2`까지의 코드를 작성하고 실행해보면, 출력값이 이전과는 다르다. 

두 번째 구독자는 코드 실행 시점으로부터 3초 뒤에 `source` Observable에 대한 구독을 시작하는데, 출력된 결과를 보면 구독 시작 후 `source` Observable로부터 받은 첫 next event의 값이 `2`라는 걸 확인할 수 있다. 

multicast 연산자를 사용하지 않는 경우 구독자마다 개별 시퀀스가 시작되기 때문에, 두 번째 구독자 같은 경우에도 구독 시점 지연과 무관하게 완전한 시퀀스를 전달 받을 수 있었다. 

지금과 같은 경우에는 multicast 연산자를 사용하여 원본 Observable을 ConnectableObservable로 바꾸었다. 그러면 모든 구독자가 원본 Observable을 공유하게 된다. 

그래서 두 번째 구독자의 구독이 지연되는 3초 동안 원본 Observable이 전달한 두 개의 이벤트는 두 번째 구독자에게 전달되지 않는다. 두 번째 구독자가 처음으로 받게 되는 이벤트는 `2`가 저장되어 있는 next event이다. 

#### connect Method

![스크린샷 2020-06-12 오전 9.02.20](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp6iu81hij30mn06v0ui.jpg)

connect 메소드의 리턴형을 보면, Disposable을 리턴하도록 되어있다.

그래서 원하는 시점에 dispose method를 실행하여 공유 시퀀스를 중지하거나 다른 Observable들처럼 disposeBag에 넣어 리소스를 정리할 수 있다.

![스크린샷 2020-06-12 오전 9.03.09](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp6jo8rk7j30d903pdgy.jpg)

multicast 연산자는 하나의 Observable을 공유할 때 사용하는 가장 기본적인 연산자이다. 

원하는 기능을 자유롭게 구현할 수 있지만 subject를 직접 만들어야하고 connect 메소드를 직접 호출해야한다는 점에서 번거롭다. 그래서 multicast Operator를 직접 사용하기보다 이 연산자를 활용하는 다른 연산자들을 주로 사용한다. 



---

### publish



![스크린샷 2020-06-12 오전 9.18.04](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp6z8702rj30mg0aldia.jpg)



publish 연산자는 간단하다. 바로 위의 multicast 연산자를 호출하고, 새로운 PublishSubject를 만들어서 파라미터로 전달한다. 그러면 multicast가 리턴하는 ConnectableObservable을 그대로 리턴한다. 



```swift
let bag = DisposeBag()
//let subject = PublishSubject<Int>() // #2
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).publish() // #1

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

source.connect() // #3

/*출력값
🔵 next(0)
🔵 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
*/
```

multicast 연산자에서 사용했던 코드에서 `#1`에 사용했었던 multicast operator 대신 publish 연산자를 사용했다. 

publish 연산자 내부에서 PublishSubject를 생성하고, multicast 연산자로 전달해주기 때문에 별도의 파라미터를 전달해줄 필요가 없다. 그래서 `#2`의 `subject`는 필요 없어진다. 

실행 결과 역시 당연히 multicast 연산자를 설명할 때 사용했던 코드와 동일하다. 

multicast 연산자는 Observable을 공유하기 위해서 내부적으로 subject를 사용한다. 

만약 multicast 연산자를 사용하며 파라미터로 PublishSubject를 전달한다면, publish 연산자를 사용하는 편이 코드가 단순해진다는 점에서 더 좋다. 

PublishSubject를 자동으로 생성해준다는 걸 제외하면 multicast연산자와 동일하다.

그래서 connect method를 호출하는 `#3`구문은 절대 생략할 수 없다. 



---

### replay



우선 ConnectableObservable에 Buffer를 추가하고, 새로운 구독자에게 최근 이벤트를 전달하는 방법을 알아보고나서 replay 연산자를 활용해서 코드를 단순하게 바꿔보자.



multicast 연산자를 구현할 때 사용했던 코드를 다시 가져왔다.

```swift
let bag = DisposeBag()
let subject = PublishSubject<Int>() // #1
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).multicast(subject)

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

source.connect()

/*출력값
🔵 next(0)
🔵 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
*/
```

이번에는 multicast 연산자에 파라미터로 전달하는 `subejct`에 집중해보자.

connect 메소드가 실행되는 시점에 원본 Observable에서 시퀀스가 시작되고, 구독자에게 이벤트가 전달되기 시작한다. 

첫 번째 구독자는 지연 없이 즉시 구독을 시작하기 때문에 모든 이벤트를 전달 받는다.

하지만 두 번째 구독자는 3초 뒤에 구독을 시작한다. 그래서 구독 전에 `subject`가 전달한 이벤트는 전달 받지 못한다.

만약 두 번째 구독자에게 이전에 전달되었던 이벤트도 함께 전달하고 싶다면 어떻게 해야할까?

우선 PublishSubject는 별도의 버퍼를 가지고 있지 않아서 그것이 불가능하다.

이럴 때는 `subject`의 타입을 PublishSubject에서 ReplaySubject로 바꾸면 된다. (`#1`)



그러면 아래와 같이 된다.

```swift
let bag = DisposeBag()
let subject = ReplaySubject<Int>.create(bufferSize: 5)
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).multicast(subject)

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

source.connect()

/*출력값
🔵 next(0)
🔵 next(1)
🔴 next(0)
🔴 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
*/
```

ReplaySubject의 버퍼 사이즈는 5로 지정해주었다. 

출력 결과를 보면, 두 번째 구독자가 구독을 시작하고 처음 전달된 next event의 element인 `2`와 함께  이전에 방출되었던 next event들까지 한꺼번에 전달되는 것을 볼 수 있다. 

![화면 기록 2020-06-12 오전 9.43.51.mov](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp7rj1bf9g30a808ygmu.gif)

(한번에 전달 받는 걸 볼 수 있다.)

최대 5개의 이벤트를 buffer에 저장하고 있기 때문이다. 그래서 이 이후에 새롭게 추가되는 구독자가 있다면, 구독 시점에 최대 5개까지의 이벤트를 전달 받는다. 



이제 replay 연산자를 활용하여 코드를 더 단순화시켜보자.

![스크린샷 2020-06-12 오전 9.48.12](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp7uiu826j30m80be41n.jpg)

`replay` 연산자는 `publish` 연산자와 마찬가지로 내부에서 `multicast` 연산자를 호출하는데, 이때 `ReplaySubject`를 만들어서 `multicast` 연산자의 파라미터로 전달한다. 

결국 `multicast` 연산자의 파라미터로 `PublishSubject`를 전달한다면 `publish` 연산자를 사용하고, `ReplaySubject`를 전달한다면 `replay` 연산자를 사용하면 된다. 

두 연산자 모두 `multicast` 연산자를 더 쉽게 사용할 수 있게 도와주는 utility 연산자들이다. 

`replay` 연산자를 사용할 때에는 보통 파라미터를 통해 버퍼의 크기를 지정하지만, 버퍼 크기의 제한이 없는 `replayAll` 연산자도 있다. 하지만 구현에 따라서 메모리 사용량이 급격하게 증가하는 문제가 있기 때문에 특별한 이유가 없다면 사용하지 않아야한다. 



```swift
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).take(5).replay(5)

source
   .subscribe { print("🔵", $0) }
   .disposed(by: bag)

source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }
   .disposed(by: bag)

source.connect()

/*출력값
🔵 next(0)
🔵 next(1)
🔴 next(0)
🔴 next(1)
🔵 next(2)
🔴 next(2)
🔵 next(3)
🔴 next(3)
🔵 next(4)
🔴 next(4)
🔵 completed
🔴 completed
*/
```

이전 코드에서 `subject`를 제거하고, multicast 연산자 대신 replay 연산자를 사용했다. 파라미터로는 버퍼 사이즈를 전달하면 된다. 실행 결과는 이전과 완전히 동일하다.

replay 연산자를 사용할 때는 항상 버퍼 크기를 신중하게 지정해야한다. 필요 이상으로 크게 지정하게 되면 필연적으로 메모리 문제가 발생하기 때문에, 필요한 선에서 가장 작은 크기로 지정해야 한다. 그리고 버퍼 크기에 제한이 없는 replayAll 연산자는 가능하다면 사용하지 않아야 한다.

그리고 multicast, publish 연산자와 마찬가지로 connect 메소드를 꼭 호출해주어야한다.

---

### refCount 



이전까지의 Sharing Operator(multicast, publish, replay)들은 아래와 같이 모두 `ObservableType` Protocol의 extension으로 구현되어있다. 

 ![스크린샷 2020-06-12 오전 10.05.03](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp8c3h9cxj30cb0emmzg.jpg)



하지만 refCount 연산자는 `ConnectableObservableType`의 extension으로 구현되어 있다.

![스크린샷 2020-06-12 오전 10.03.07](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp8a1ymsoj30m8097mzd.jpg)

다시 말해서 일반 Observable에서는 사용할 수 없고, `ConnectableObservable`에서만 사용할 수 있다. 

구현을 보면 파라미터는 없고, Observable을 리턴한다. 그리고 그 내부에 있는 `RefCount(source: self)`의 RefCount는 `ConnectableObservable`을 통해 생성하는 특별한 Observable이다. 앞으로 이것을 `RefCountObservable`이라고 지칭한다.



`RefCountObservable`은 내부에 `ConnectableObservable`을 유지하면서, 새로운 구독자가 추가되는 시점에 자동으로 `connect` Method를 호출한다. 

이후 구독자가 구독을 중지하고 더 이상 다른 구독자가 없다면 `ConnectableObservable`의 시퀀스를 중지한다. 

그러다가 새로운 구독자가 추가되면 다시 connect 메소드를 호출한다. 이때 `ConnectableObservable`에서는 새로운 시퀀스가 시작된다.



```swift
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().publish() // #1

let observer1 = source
.refCount()
  .subscribe { print("🔵", $0) } // #2

source.connect()

DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
   observer1.dispose() // #3
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) { // #4
   let observer2 = source.subscribe { print("🔴", $0) } // #5

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) { // #6
      observer2.dispose() // #7
   }
}
/*출력값
2020-06-12 10:24:03.508: refCount.playground:31 (__lldb_expr_47) -> subscribed
2020-06-12 10:24:04.510: refCount.playground:31 (__lldb_expr_47) -> Event next(0)
🔵 next(0)
2020-06-12 10:24:05.510: refCount.playground:31 (__lldb_expr_47) -> Event next(1)
🔵 next(1)
2020-06-12 10:24:06.510: refCount.playground:31 (__lldb_expr_47) -> Event next(2)
🔵 next(2)
2020-06-12 10:24:07.510: refCount.playground:31 (__lldb_expr_47) -> Event next(3)
2020-06-12 10:24:08.510: refCount.playground:31 (__lldb_expr_47) -> Event next(4)
2020-06-12 10:24:09.510: refCount.playground:31 (__lldb_expr_47) -> Event next(5)
2020-06-12 10:24:10.509: refCount.playground:31 (__lldb_expr_47) -> Event next(6)
2020-06-12 10:24:11.510: refCount.playground:31 (__lldb_expr_47) -> Event next(7)
🔴 next(7)
2020-06-12 10:24:12.510: refCount.playground:31 (__lldb_expr_47) -> Event next(8)
🔴 next(8)
2020-06-12 10:24:13.510: refCount.playground:31 (__lldb_expr_47) -> Event next(9)
🔴 next(9)
2020-06-12 10:24:14.510: refCount.playground:31 (__lldb_expr_47) -> Event next(10)
2020-06-12 10:24:15.510: refCount.playground:31 (__lldb_expr_47) -> Event next(11)
2020-06-12 10:24:16.510: refCount.playground:31 (__lldb_expr_47) -> Event next(12)
2020-06-12 10:24:17.510: refCount.playground:31 (__lldb_expr_47) -> Event next(13)
2020-06-12 10:24:18.511: refCount.playground:31 (__lldb_expr_47) -> Event next(14)
2020-06-12 10:24:19.509: refCount.playground:31 (__lldb_expr_47) -> Event next(15)
2020-06-12 10:24:20.510: refCount.playground:31 (__lldb_expr_47) -> Event next(16)
...
(이하 생략)
*/
```



예제를 보면, `#1`에는 1초마다 정수를 방출하는 Observable이 생성되어 있다. 자주 사용한 코드지만 이번에는 take 연산자를 사용하지 않고 있다. 그래서 코드를 실행하면 계속해서 정수를 방출한다. event 발생 시점을 자세히 확인할 수 있도록 debug 연산자가 추가되어 있고, publish 연산자로 Observable을 공유하고 있다. 

그리고 `#2`에서 첫 번째 구독자를 추가한다. 첫 번째 구독자는 3초 뒤에 `#3`을 통해 구독이 중지된다. 

두 번째 구독자는 `#4`에서 설정한대로 7초 뒤에 `#5`를 통해 구독을 시작하고, `#6`의 설정대로 구독 시작 후 3초 뒤에 `#7`로 인해 구독이 중지된다. 

![화면 기록 2020-06-12 오전 10.23.48.mov](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp8x33v3jg30dw05pe81.gif)

실행 결과를 보면 구독이 시작되었다가 3초 뒤에 첫 번째 구독이 중지된다. 그리고 7초 뒤에 두 번째 구독이 시작되고, 다시 3초 뒤에 두 번째 구독이 중지된다. 하지만 `ConnectableObservable`은 계속해서 정수를 방출하고 있다. `ConnectableObservable`을 중지하고 싶다면 connect 메소드가 리턴하는 Disposable을 저장해두었다가, 원하는 시점에 dispose() 메소드를 호출해야한다. 

로그를 자세히 보면, 두 번째 구독자가 처음으로 받은 next event에는 7이 저장되어있다. 하나의 구독을 공유하기 때문에 당연한 결과이다. 



이제 refCount 연산자를 사용하는 코드로 바꾸고 결과를 비교해보자.

```swift
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().publish().refCount() // #1

let observer1 = source
  .subscribe { print("🔵", $0) }

// #2

DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
   observer1.dispose() // #3
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
   let observer2 = source.subscribe { print("🔴", $0) } // #4

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
      observer2.dispose() // #5
   }
}

/*출력값
2020-06-12 10:32:50.640: refCount.playground:31 (__lldb_expr_51) -> subscribed
2020-06-12 10:32:51.643: refCount.playground:31 (__lldb_expr_51) -> Event next(0)
🔵 next(0)
2020-06-12 10:32:52.642: refCount.playground:31 (__lldb_expr_51) -> Event next(1)
🔵 next(1)
2020-06-12 10:32:53.642: refCount.playground:31 (__lldb_expr_51) -> Event next(2)
🔵 next(2)
2020-06-12 10:32:53.936: refCount.playground:31 (__lldb_expr_51) -> isDisposed
2020-06-12 10:32:58.342: refCount.playground:31 (__lldb_expr_51) -> subscribed
2020-06-12 10:32:59.343: refCount.playground:31 (__lldb_expr_51) -> Event next(0)
🔴 next(0)
2020-06-12 10:33:00.343: refCount.playground:31 (__lldb_expr_51) -> Event next(1)
🔴 next(1)
2020-06-12 10:33:01.342: refCount.playground:31 (__lldb_expr_51) -> Event next(2)
🔴 next(2)
2020-06-12 10:33:01.635: refCount.playground:31 (__lldb_expr_51) -> isDisposed
*/
```



`#1`의 publish 연산자 뒤에 refCount 연산자를 추가한다. 그러면 publish 연산자가 리턴하는 `ConnectableObservable`이 `RefCountObservable`로 변경된다. `RefCountObservable`은 내부에서 connect 메소드를 자동으로 호출하기 때문에, `#2`에 있던 `source.connect()`구문은 더 이상 필요 없어졌으므로 지워주었다. 

![화면 기록 2020-06-12 오전 10.32.37.mov](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp96yk8wzg30dw05pqmi.gif)

실행 결과가 완전히 달라졌다. 

첫 번째 구독자가 추가되면 `RefCountObservable`이 connect 메소드를 호출한다. 그러면 `ConnectableObservable`은 subject를 통해서 모든 구독자에게 이벤트를 전달한다. 

`#3`에서 첫 번째 구독자가 구독을 중지한다. 이 시점에 다른 구독자는 없기 때문에 `ConnectableObservable` 역시 중지된다. 그래서 isDisposed 로그가 출력된다.

RxSwift Docs에서는 이러한 동작을 Disconnect라고 표현한다. 

첫 번째 구독자가 추가되면 connect 되고, 더 이상 구독자가 없다면 disconnect 된다. 



이어 `#4`코드를 통해 7초 뒤에 새롭게 두 번째 구독자가 구독을 시작한다. 그러면 connect 된다. 이 때는 `ConnectableObservable`에서 새로운 시퀀스가 시작된다. 그래서 구독자가 처음으로 받는 next event에는 7이 아니라 0이 저장되어 있는 것이다. 

`#5`코드로 인해 구독 시작 3초 뒤에 구독이 중지되면, 이전처럼 `ConnectableObservable`이 중지된다. 그래서 다시 isDisposed 로그가 출력된다. 



multicast, publish, replay 연산자를 사용할 때에는 connect 메소드를 직접 호출해야 하고, 필요한 시점에 dispose 메소드나 take 연산자를 활용해서 리소스가 정리되도록 구현해야한다. 

하지만 refCount 연산자를 활용하면 이런 부분이 자동으로 처리되기 때문에 코드를 단순하게 구현할 수 있다. 



---

### share 



![스크린샷 2020-06-12 오전 10.48.08](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp9l0fbf8j30mn10447o.jpg)

share 연산자 구현 코드이다. 이전에 다뤘던 다양한 Sharing Operator들을 활용하고 있기 때문에 복잡하게 보인다. 



먼저 share 연산자는 두 개의 파라미터를 받는다.

첫 번째 파라미터는 replay buffer의 크기이다. 파라미터로 0을 전달하면, 

![스크린샷 2020-06-12 오전 10.50.58](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp9nwf5pvj30e601haar.jpg)

케이스로 선언되어있듯 multicast를 호출할 때 PublishSubject를 전달한다. 

만약 0보다 큰 값을 전달한다면 

![스크린샷 2020-06-12 오전 10.52.29](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp9pf98rvj30f70253zi.jpg)

이렇게 multicast를 호출할 때 ReplaySubject를 전달한다. 

기본값이 0으로 선언되어 있기 때문에, 다른 값을 전달하지 않는다면 새로운 구독자는 구독 이후에 방출된 event만 전달 받는다. 

내부적으로 multicast 연산자를 호출하여 사용하기 때문에 multicast 연산자와 마찬가지로 하나의 Subject를 통해 시퀀스를 공유한다. 두 번째 파라미터는 바로 이 Subject의 수명을 결정한다. 

연산자의 리턴형은 Observable로 선언되어 있다. 

share 연산자는 내부적으로 먼저 multicast 연산자를 호출하고 이어서 refCount 연산자를 호출한다. share 연산자가 리턴하는 Observable은 이전에 다룬 `RefCountObservable`이다.

> `RefCountObservable은` 내부에 `ConnectableObservable`을 유지하면서, 새로운 구독자가 추가되는 시점에 자동으로 connect() Method를 호출한다. 
>
> 이후 구독자가 구독을 중지하고 더 이상 다른 구독자가 없다면 `ConnectableObservable`의 시퀀스를 중지한다. 
>
> 그러다가 새로운 구독자가 추가되면 다시 connect 메소드를 호출한다. 이때 `ConnectableObservable`에서는 새로운 시퀀스가 시작된다.

그래서 새로운 구독자가 추가되면 자동으로 connect 되고, 구독자가 더 이상 없다면 disconnect 된다. 



두 번째 파라미터는 기본 값이 `.whileConnected`로 선언되어 있다. 

![스크린샷 2020-06-12 오전 10.58.54](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfp9w4jse0j30gp011dge.jpg)

이 경우 새로운 구독자가 추가되면(즉 새로운 connection이 시작되면) 새로운 subject가 생성된다. 그리고 connection이 종료되면 그 subject는 사라진다. connection마다 새로운 subject가 생성되기 때문에, 각 connection들은 다른 connection들과 격리된다. 



반대로 두 번째 파라미터로 `.forever`를 전달하면 모든 connection이 하나의 subject를 공유한다. 

```swift
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().share() // #1

let observer1 = source
   .subscribe { print("🔵", $0) }

let observer2 = source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {// #2
   observer1.dispose() 
   observer2.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
   let observer3 = source.subscribe { print("⚫️", $0) } // #3

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
      observer3.dispose()
   }
}

/*출력값
2020-06-12 11:21:07.026: share.playground:31 (__lldb_expr_55) -> subscribed
2020-06-12 11:21:08.042: share.playground:31 (__lldb_expr_55) -> Event next(0)
🔵 next(0)
2020-06-12 11:21:09.042: share.playground:31 (__lldb_expr_55) -> Event next(1)
🔵 next(1)
2020-06-12 11:21:10.042: share.playground:31 (__lldb_expr_55) -> Event next(2)
🔵 next(2)
2020-06-12 11:21:11.042: share.playground:31 (__lldb_expr_55) -> Event next(3)
🔵 next(3)
🔴 next(3)
2020-06-12 11:21:12.042: share.playground:31 (__lldb_expr_55) -> Event next(4)
🔵 next(4)
🔴 next(4)
2020-06-12 11:21:12.544: share.playground:31 (__lldb_expr_55) -> isDisposed
2020-06-12 11:21:14.743: share.playground:31 (__lldb_expr_55) -> subscribed
2020-06-12 11:21:15.744: share.playground:31 (__lldb_expr_55) -> Event next(0)
⚫️ next(0)
2020-06-12 11:21:16.743: share.playground:31 (__lldb_expr_55) -> Event next(1)
⚫️ next(1)
2020-06-12 11:21:17.744: share.playground:31 (__lldb_expr_55) -> Event next(2)
⚫️ next(2)
2020-06-12 11:21:17.745: share.playground:31 (__lldb_expr_55) -> isDisposed
*/
```



1초마다 정수를 방출하는 source Observable이 있고, `#1`에서 share 연산자를 호출한다. 

즉시 source Observable을 구독하는 `observer1`과, 3초의 지연 후에 구독을 시작하는 `observer2`가 있다.

코드 실행 5초 뒤에 두 구독자 모두 dispose 된다.

코드 실행 7초 후에 `observer3`가 source Observable에 새로운 구독자를 추가한다. 그로부터 3초 후에 `observer3`가 dispose 된다. 

 `source`의 형식을 확인해보면 Observable로 나온다.

![스크린샷 2020-06-12 오전 11.18.19](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfpagpo9p5j30e303nac3.jpg)

하지만 share 연산자가 리턴하는 Observable은 단순한 Observable이 아니라 `RefCountObservable`라는 걸 꼭 기억해야한다. 



코드를 실행하면 아래와 같이 출력된다.

![화면 기록 2020-06-12 오전 11.20.40.mov](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfpal3cb89g30dw05qnig.gif)

첫 번재 파라미터는 replay buffer 파라미터이고 기본값이 0이다. 그래서 3초 뒤에 구독을 시작한 두 번째 구독자 `observer2`는 이전에 source` Observable로부터 방출되었던 3개의 next event를 받지 못한다. 

그리고 5초 후에 `#2`에서 `observer1`, `observer2`의 구독이 종료되면, 내부에 있는 `ConnectableObservable` 역시 중지된다. 그래서 isDisposed 로그가 발생한다. 이것은 share 연산자 내부에서 refCount 연산자를 호출하기 때문이다. 

이어 7초 뒤에 새로운 구독자 `observer3`가 추가되면 `ConnectableObservable`에서 새로운 시퀀스가 시작된다. 그래서 세 번째 구독자 `observer3`가 처음 받는 이벤트에는 0이 포함되어 있다. 

![스크린샷 2020-06-12 오전 11.28.16](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfpaqndq3wj302v00ojrb.jpg)

이제 share 연산자의 두 번째 파라미터에 대해 생각해보자. `ConnectableObservable` 내부에 있는 Subject의 수명을 결정하는데, 기본값이 `.whileConnected`이다. 이 경우 새로운 구독자가 추가되면 Subject를 생성하고, 이후에 추가되는 구독자들은 이 Subject를 구독한다. 

그래서 첫 번째 구독자와 두 번째 구독자는 동일한 Subject로부터 이벤트를 전달 받는다(`observer1`이 추가된 시점에 생성된 Subject). 

이것이 두 번째 구독자가 처음으로 받은 next event에 0이 아니라 3이 저장되어있는 이유이다.

이어 `#2`에서 두 구독자 모두 종료된 시점에 `2020-06-12 11:21:12.544: share.playground:31 (__lldb_expr_55) -> isDisposed` 로그가 출력되는데, 이때 Subject가 사라진다. 

그 이후 새로운 구독자 `observer3`가 추가되는 시점`#3`에   `2020-06-12 11:21:14.743: share.playground:31 (__lldb_expr_55) -> subscribed`가 출력되고, 

이때 새로운 Subject가 생성된다. 그래서 세 번째 구독자가 처음으로 받는 next event에는 0이 저장되어 있다. 

앞에서 다룬 Sharing Operator들인 multicast, publish, replay, refCount 들이 모두 사용되어있는 연산자가 share 연산자이다.



이번에는 `#1`의 share 연산자에 replay 파라미터로 5를 전달해보자.

```swift
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().share(replay: 5) // #1

let observer1 = source
   .subscribe { print("🔵", $0) }

let observer2 = source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
   observer1.dispose()
   observer2.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
   let observer3 = source.subscribe { print("⚫️", $0) }

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
      observer3.dispose()
   }
}

/*출력값
2020-06-12 11:40:17.845: share.playground:31 (__lldb_expr_57) -> subscribed
2020-06-12 11:40:18.848: share.playground:31 (__lldb_expr_57) -> Event next(0)
🔵 next(0)
2020-06-12 11:40:19.847: share.playground:31 (__lldb_expr_57) -> Event next(1)
🔵 next(1)
2020-06-12 11:40:20.847: share.playground:31 (__lldb_expr_57) -> Event next(2)
🔵 next(2)
🔴 next(0)
🔴 next(1)
🔴 next(2)
2020-06-12 11:40:21.847: share.playground:31 (__lldb_expr_57) -> Event next(3)
🔵 next(3)
🔴 next(3)
2020-06-12 11:40:22.847: share.playground:31 (__lldb_expr_57) -> Event next(4)
🔵 next(4)
🔴 next(4)
2020-06-12 11:40:23.349: share.playground:31 (__lldb_expr_57) -> isDisposed
2020-06-12 11:40:25.549: share.playground:31 (__lldb_expr_57) -> subscribed
2020-06-12 11:40:26.550: share.playground:31 (__lldb_expr_57) -> Event next(0)
⚫️ next(0)
2020-06-12 11:40:27.550: share.playground:31 (__lldb_expr_57) -> Event next(1)
⚫️ next(1)
2020-06-12 11:40:28.549: share.playground:31 (__lldb_expr_57) -> Event next(2)
⚫️ next(2)
2020-06-12 11:40:28.849: share.playground:31 (__lldb_expr_57) -> isDisposed
*/
```



이제 실행 결과를 gif로 확인해보며 시점을 생각해보자.

![화면 기록 2020-06-12 오전 11.40.17.mov](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfpb50810xg30dw05q1kx.gif)

이제 새로운 구독자는 구독이 시작되는 시점에 버퍼에 저장되어있는 최대 5개의 최근 값을 한꺼번에 전달 받는다. 그래서 두 번째 구독자 `observer2`는 이전에 전달되었던 next event도 함께 전달 받는다. 

하지만 세 번째 구독자 `observer3`는 새로운 Subject로부터 이벤트를 전달 받기 때문에, 구독 시점에 하나의 next event만 받는다. 



마지막으로 `#1`share 연산자에 두 번째 파라미터인 scope로 `.forever`를 전달해보자. 

```swift
let bag = DisposeBag()
let source = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance).debug().share(replay: 5, scope: .forever)

let observer1 = source
   .subscribe { print("🔵", $0) }

let observer2 = source
   .delaySubscription(.seconds(3), scheduler: MainScheduler.instance)
   .subscribe { print("🔴", $0) }

DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
   observer1.dispose()
   observer2.dispose()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 7) {
   let observer3 = source.subscribe { print("⚫️", $0) }

   DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
      observer3.dispose()
   }
}

/*출력값
2020-06-12 11:46:25.277: share.playground:31 (__lldb_expr_59) -> subscribed
2020-06-12 11:46:26.292: share.playground:31 (__lldb_expr_59) -> Event next(0)
🔵 next(0)
2020-06-12 11:46:27.291: share.playground:31 (__lldb_expr_59) -> Event next(1)
🔵 next(1)
2020-06-12 11:46:28.291: share.playground:31 (__lldb_expr_59) -> Event next(2)
🔵 next(2)
🔴 next(0)
🔴 next(1)
🔴 next(2)
2020-06-12 11:46:29.291: share.playground:31 (__lldb_expr_59) -> Event next(3)
🔵 next(3)
🔴 next(3)
2020-06-12 11:46:30.292: share.playground:31 (__lldb_expr_59) -> Event next(4)
🔵 next(4)
🔴 next(4)
2020-06-12 11:46:30.790: share.playground:31 (__lldb_expr_59) -> isDisposed
⚫️ next(0)
⚫️ next(1)
⚫️ next(2)
⚫️ next(3)
⚫️ next(4)
2020-06-12 11:46:32.994: share.playground:31 (__lldb_expr_59) -> subscribed
2020-06-12 11:46:33.995: share.playground:31 (__lldb_expr_59) -> Event next(0)
⚫️ next(0)
2020-06-12 11:46:34.995: share.playground:31 (__lldb_expr_59) -> Event next(1)
⚫️ next(1)
2020-06-12 11:46:35.995: share.playground:31 (__lldb_expr_59) -> Event next(2)
⚫️ next(2)
2020-06-12 11:46:35.996: share.playground:31 (__lldb_expr_59) -> isDisposed
*/
```

share 연산자의 scope 파라미터로 `.forever`를 전달하면 모든 구독자가 하나의 Subject 공유한다. 

![화면 기록 2020-06-12 오전 11.46.12.mov](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfpbanriimg30dw05q4o2.gif)

이번에는 세 번째 구독자 `observer3`가 추가되는 시점에 replay buffer에 저장되어있는 5개의 이벤트가 함께 전달된다. 

하지만 buffer에 저장되어있던 값 5개를 받은 직후 `observer3`가 새롭게 전달 받은 next event에는 5가 아니라 0이 저장되어 있다.

![스크린샷 2020-06-12 오전 11.49.33](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfpbct6updj303305et9b.jpg)

share 연산자가 내부에서 refCount 연산자를 사용하고 `RefCountObservable`을 리턴하기 때문에 그렇다. share 연산자가 리턴하는 `RefCountObservable`의 특징을 생각해보면 당연하다. 

**두 번째 파라미터로 `.forever`를 전달함으로써 모든 구독자가 하나의 Subject로부터 전달되는 이벤트를 공유하게 됐다고 하더라도, 중간에 시퀀스가 중지된 다음에 새로운 구독자가 추가되면 언제나 새로운 시퀀스가 시작된다. 한 번 중지된 시퀀스가 '이어서' 공유되는 것이 아니다.**

scope를 `.forever`로 지정하면, 하나의 subject를 공유할 뿐이다. share 연산자가 리턴하는 `RefCountObservable`의 특징에는 아무런 영향을 끼치지 못한다. 

***어떤 시퀀스를 공유하는 모든 구독자가 dispose되면 해당 시퀀스 역시 종료된다. 그리고 새로운 구독자가 추가되면 -> 새로운 시퀀스가 생성된다.



---

## 17.Scheduler



 iOS 앱을 만들다가 Multi Threading 처리가 필요하면 GCD(Grand Central Dispatch)를 사용하지만, RxSwift에서는 Scheduler를 사용한다. 

![스크린샷 2020-06-13 오후 7.24.53](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqu63peywj30nz0gijt1.jpg)

스케쥴러는 특정 코드가 실행되는 Context를 추상화한 것이다. Context는 Low level thread가 될 수도 있고, Dispatch Queue나 Operation Queue가 될 수도 있다. 

스케쥴러는 추상화된 Context이기 때문에 쓰레드와 1:1로 매칭되지 않는다. 하나의 쓰레드에 두 개 이상의 개별 스케쥴러가 존재하거나, 하나의 스케쥴러가 두 개의 쓰레드에 걸쳐있는 경우도 있다. 

하지만 역시 큰 틀에서 보면 GCD와 유사하며, 몇 가지 규칙 하에서 스케쥴러를 사용하면 된다. 



예를 들어 UI update와 관련된 코드는 Main 쓰레드에서 실행해야하는데, 

이를 GCD에서는 Main Queue에서 실행하고, RxSwift에서는 Main Scheduler에서 실행한다. 

그리고 Network request나 파일 처리 같은 작업을 메인 쓰레드에서 실행하면 Blocking이 발생한다. 그래서 GCD에서는 이러한 작업을 Global Queue에서 실행하는데, RxSwift에서는 Background Scheduler에서 실행한다. 



![스크린샷 2020-06-13 오후 7.32.53](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqud9otgcj30v20ch75v.jpg)

RxSwift는 GCD와 마찬가지로 다양한 기본 스케쥴러를 제공한다. 내부적으로 GCD와 유사한 방식으로 동작하고, 실행할 작업을 스케쥴링한다. 

스케쥴링 방식에 따라 Serial Scheduler와 Concurrent Scheduler로 구분한다.

- 가장 기본적인 스케쥴러는 Serial Scheduler에 속하는 `CurrentThreadScheduler`이다. 별도로 스케쥴러를 별도로 지정하지 않는다면, 이 스케쥴러가 사용된다.

- Main Thread와 연관된 스케쥴러는 Serial Scheduler의 `MainScheduler`이다. 메인 큐처럼 UI를 업데이트 할 때 사용한다. 

- 작업을 실행할 Dispatch Queue를 직접 지정하고 싶다면, Serial Scheduler의 `SerialDispatchQueueScheduler`와 Concurrent Scheduler의 `ConcurrentDispatchQueueScheduler`를 사용한다. 

- Background 작업을 실행할 때는 Serial Scheduler의 `SerialDispatchQueueScheduler`와 Concurrent Schedulerd의`ConcurrentDispatchQueueScheduler`를 사용한다. 

- 실행 순서를 지정하거나 동시에 실행 가능한 작업 수를 제한하고 싶다면 OperationQueueScheduler를 사용한다. 이 스케쥴러는 DispatchQueue가 아닌 OperationQueue를 사용해서 생성한다.

- 이 밖에도 Unit test에 사용되는 `Test Scheduler`가 제공된다.
-  스케쥴러를 직접 구현하는 것도 가능하다.



#### Scheduler의 사용

스케쥴러를 잘 사용하기 위해서 두 가지가 선행되어야한다. 

1. 먼저 Observable이 생성되는 시점을 이해해야한다. 

```swift
let bag = DisposeBag()

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
   .filter { num -> Bool in
      print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
      return num.isMultiple(of: 2)
   }
   .map { num -> Int in
      print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
      return num * 2
   }

//실행 결과 없음
```

 코드를 보면 1에서 9까지 방출하는 Observable이 선언되어있다. 그리고 filter 연산자로 짝수를 필터링하고, map 연산자로 개별 결과에 2를 곱한다. 또 연산자가 어느 쓰레드에서 실행되는지 확인하는 로그도 추가되어있다. 하지만 이대로 실행할 경우 로그도, 아무것도 출력되지 않는다. 

즉 Observable이 생성되는 것도 아니고, 연산자가 호출된 것도 아니다. 여기에 있는 코드는 Observable이 어떤 요소를 방출하고, 어떻게 처리해야하는지 나타낼 뿐이다. 

**실제 Observable이 생성되고, 연산자가 호출되는 시점은 바로 구독이 시작되는 시점이다.**



```swift
let bag = DisposeBag()

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
  .filter { num -> Bool in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
    return num.isMultiple(of: 2)
}
.map { num -> Int in
  print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
  return num * 2
}
.subscribe { // #1
  print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> subscribe")
  print($0)
}
  .disposed(by: bag)

/*실행 결과
Main Thread >> filter
Main Thread >> filter
Main Thread >> map
Main Thread >> subscribe
next(4)
Main Thread >> filter
Main Thread >> filter
Main Thread >> map
Main Thread >> subscribe
next(8)
Main Thread >> filter
Main Thread >> filter
Main Thread >> map
Main Thread >> subscribe
next(12)
Main Thread >> filter
Main Thread >> filter
Main Thread >> map
Main Thread >> subscribe
next(16)
Main Thread >> filter
Main Thread >> subscribe
completed

*/
```

`#1`처럼 구독자가 추가되는 시점에 Observable이 생성되고, 연산자를 거쳐서 최종 결과가 구독자에게 전달된다.



2. 두 번째는 스케쥴러를 지정하는 방법

위 코드에는 따로 스케쥴러를 지정하지 않았는데, 이 경우에는 기본 스케쥴러인 `CurrentThreadScheduler`가 사용된다.

플레이그라운드가 실행되는 쓰레드는 Main Thread이고, 여기에서 작성한 모든 코드는 Main Thread에서 실행된다. 그래서 결과를 보면 모두 Main Thread라고 출력되어있다. 

map 연산자를 Main Thread가 아니라 Background Thread에서 실행하고 싶다면?

RxSwift에서 스케쥴러를 지정할 때에는 observeOn 메소드와 subscribeOn 메소드를 사용한다. 





ObserveOn 메소드는 연산자를 실행할 스케쥴러를 지정한다. 

코드 앞 부분에 backgroundScheduler를 하나 만들어보자.

```swift
let backgroundScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global()) // #1
```

이어서 observeOn 메소드를 이용하여 map 연산자를 실행할 스케쥴러로 `backgroundScheduler`를 지정해준다. ( 아래 코드의 `#2`)

```swift
let bag = DisposeBag()

let backgroundScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global()) // #1

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
  .filter { num -> Bool in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
    return num.isMultiple(of: 2)
}
.observeOn(backgroundScheduler) // #2
.map { num -> Int in
  print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
  return num * 2
}
.subscribe { // #3
  print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
  print($0)
}
  .disposed(by: bag)

/* 실행 결과
Main Thread >> filter
Main Thread >> filter
Background Thread >> map
Main Thread >> filter
Main Thread >> filter
Main Thread >> filter
Background Thread >> subscribe
Main Thread >> filter
next(4)
Main Thread >> filter
Background Thread >> map
Main Thread >> filter
Main Thread >> filter
Background Thread >> subscribe
next(8)
Background Thread >> map
Background Thread >> subscribe
next(12)
Background Thread >> map
Background Thread >> subscribe
next(16)
Background Thread >> subscribe
completed

*/
```

결과를 보면 map 연산자가 Background Thread에서 실행되는 걸 볼 수 있다.

ObserveOn 메소드는 이어지는 연산자들이 작업을 실행할 스케쥴러를 지정한다. 그래서 뒤에 있는 map은 background Scheduler에서 실행되지만, 앞에 있는 filter에는 영향을 주지 않는다.

그리고 결과를 자세히 보면 subscribe 내에 작성된 코드도 background에서 실행되고 있다. ObserveOn 메소드로 지정한 스케쥴러는 다른 스케쥴러로 변경하기 전까지 계속 사용된다.



subscribeOn 메소드는 구독을 시작하고 종료할 때 사용할 스케쥴러를 지정한다. 구독을 시작하면 Observable에서 새로운 이벤트가 방출되는데 이 이벤트를 방출할 Scheduler를 지정하는 것이다. 그리고 create 연산자로 구현한 코드 역시 subscribeOn 메소드로 지정한 스케쥴러에서 실행된다. 이 메소드를 사용하지 않는다면 subscribe 메소드가 호출된 스케쥴러에서 새로운 시퀀스가 시작된다.



```swift
let bag = DisposeBag()

let backgroundScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global())

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
  .filter { num -> Bool in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
    return num.isMultiple(of: 2)
}
.observeOn(backgroundScheduler)
.map { num -> Int in
  print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
  return num * 2
}
.subscribeOn(MainScheduler.instance)
.subscribe {
  print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> subscribe")
  print($0)
}
  .disposed(by: bag)
/*실행 결과
Main Thread >> filter
Main Thread >> filter
Main Thread >> filter
Background Thread >> map
Main Thread >> filter
Main Thread >> filter
Main Thread >> filter
Background Thread >> subscribe
next(4)
Main Thread >> filter
Main Thread >> filter
Background Thread >> map
Main Thread >> filter
Background Thread >> subscribe
next(8)
Background Thread >> map
Background Thread >> subscribe
next(12)
Background Thread >> map
Background Thread >> subscribe
next(16)
Background Thread >> subscribe
completed
*/
```

`#3`에 subscribeOn 메소드를 추가했다. `MainScheduler`는 `instance`  속성으로 쉽게 얻을 수 있다.

하지만 결과를 보면 **subscribe**에서 실행되는 코드는 여전히 background에서 실행되고 있다. 메소드 이름 때문에 혼동하게 되는데, 

- **subscribeOn 메소드는 subscribe 메소드가 실행되는 스케쥴러를 지정하는 것이 아니다. **
- **그리고 이어지는 연산자가 호출되는 스케쥴러를 지정하는 것도 아니다. **
- **Observable이 시작되는 시점에 어떤 스케쥴러를 사용할지 지정하는 것이다.** 

이 차이를 확실하게 구분해야 한다. 그리고 observeOn 메소드와 달리, 호출 시점이 중요하지 않다. 어차피 Observable은 subscribe 메소드가 실행되는 시점에 생성되기 때문에, subscribe 메소드 호출 시점보다 앞이기만 하면 어느 부분에서 호출하더라도 순서가 중요하지 않다.

만약 subscribe 메소드를 MainScheduler에서 실행하고 싶다면 subscribe를 호출하기 전에 observeOn 메소드를 호출하고, 파라미터로 MainScheduler.instance를 전달한다.

```swift
let bag = DisposeBag()

let backgroundScheduler = ConcurrentDispatchQueueScheduler(queue: DispatchQueue.global())

Observable.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
  .filter { num -> Bool in
    print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> filter")
    return num.isMultiple(of: 2)
}
.observeOn(backgroundScheduler)
.map { num -> Int in
  print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> map")
  return num * 2
}
.subscribeOn(MainScheduler.instance)
.observeOn(MainScheduler.instance) // #1
.subscribe {
  print(Thread.isMainThread ? "Main Thread" : "Background Thread", ">> subscribe")
  print($0)
}
  .disposed(by: bag)

/*실행 결과
Main Thread >> filter
Main Thread >> filter
Main Thread >> filter
Background Thread >> map
Main Thread >> filter
Main Thread >> filter
Main Thread >> filter
Main Thread >> filter
Main Thread >> filter
Background Thread >> map
Main Thread >> filter
Background Thread >> map
Background Thread >> map
Main Thread >> subscribe
next(4)
Main Thread >> subscribe
next(8)
Main Thread >> subscribe
next(12)
Main Thread >> subscribe
next(16)
Main Thread >> subscribe
completed
*/
```

`#1`에서 observeOn 메소드를 호출하자 이제 subscribe 메소드가 MainScheduler에서 실행된다.

정리하자면

- **subscribeOn 메소드는 Observable이 시작될 스케쥴러를 지정한다.** 

- **observeOn메소드는 이어지는 연산자가 실행될 스케쥴러를 지정한다.**



---

## 18. Error Handling

RxSwift에서는 에러를 처리하기 위한 여러 방법을 사용할 수 있다.

Observable에서 전달한 에러 이벤트가 구독자에게 전달되면, 구독이 종료되고 더 이상 새로운 이벤트가 전달되지 않는다. 즉, 더 이상 새로운 이벤트를 처리할 수 없게 된다.

예를 들어 Observable이 네트워크 요청을 처리하고, 구독자가 UI를 Update하는 패턴을 생각해보자.

보통 UI를 업데이트 하는 코드는 next event가 전달되는 시점에 실행된다. 에러 이벤트가 전달되면 구독이 종료되고, 더 이상 next event가 전달되지 않는다. 그래서 UI를 업데이트하는 코드는 실행되지 않는다(ex. 에러가 발생했다는 내용으로의 UI 업데이트가 불가능해짐). 

RxSwift는 두 가지 방법으로 이런 문제를 해결한다. 

![스크린샷 2020-06-13 오후 9.15.13](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqxbtblq2j30qr0h30u8.jpg)

첫 번째 방법은 error 이벤트가 전달되면 새로운 Observable을 리턴하는 것이다. 여기에서는 catchError 연산자를 사용한다. Observable이 전달하는 next event와 completed event는 그대로 구독자에게 전달된다. 반면 error event가 전달되면, 새로운 Observable을 구독자에게 전달한다. 

다시 네트워크 요청을 생각해보면 기본 값이나 로컬 캐시를 방출하는 Observable을 구독자에게 전달할 수 있다. 그래서 에러가 발생한 경우에도 UI는 적절한 값으로 업데이트 된다.

![스크린샷 2020-06-13 오후 9.19.11](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqxfyt37uj30s50kddha.jpg)

두 번째 방법은 에러가 발생한 경우 Observable을 다시 구독하는 것이다. 이때는 retry 연산자를 사용한다. 에러가 발생하지 않을 때까지 무한정 재시도하거나, 재시도 횟수를 제한할 수 있다.



---

### catchError

catchError 메소드는 next event와 completed event는 구독자에게 그대로 전달한다.

하지만 에러 이벤트는 전달하지 않고, 새로운 Observable이나 기본 값을 전달한다.

이 연산자는 다양한 상황에서 사용되지만, 특히 네트워크 요청을 구현할 때 많이 사용한다. 

올바른 응답을 받지 못한 상황에서 로컬 캐시를 사용하거나 기본 값을 사용하도록 구현할 수 있다. 

```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let subject = PublishSubject<Int>()
let recovery = PublishSubject<Int>()

subject
// #1
   .subscribe { print($0) }
   .disposed(by: bag)

subject.onError(MyError.error)

//실행 결과
//error(error)
```

이렇게 PublishSubject인 `subject`에 구독자를 추가하고, 에러 이벤트를 전달하면 구독자에게 그대로 전달된다. 이어서 즉시 구독이 종료되기 때문에 더 이상 다른 이벤트는 구독자에게 전달되지 않는다.  

이제 `#1`에 catchError 연산자를 추가해보도록 하자.

![스크린샷 2020-06-13 오후 9.43.36](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqy57rt0ej30mr0bgn0e.jpg)

catchError 연산자는 클로저를 파라미터로 받는다. 에러 이벤트는 클로저 파라미터로 전달되고, 클로저는 새로운 Observable을 리턴한다. 그리고 Observable이 방출하는 요소의 형식은 

![스크린샷 2020-06-13 오후 9.45.11](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqy6ur2ldj304s00pwei.jpg)

source Observable이 방출하는 요소의 형식과 동일하다. catchError 연산자는 source Observable가 에러 이벤트를 전달하면, source Observable을 catchError연산자의 클로저가 리턴하는 Observable로 교체한다. 원본 source Observable은 더 이상 다른 이벤트를 전달하지 못하지만, 교체된 새로운 source Observable은 문제가 없기 때문에 계속해서 다른 이벤트를 전달할 수 있다.

```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}

let subject = PublishSubject<Int>()
let recovery = PublishSubject<Int>()

subject
  .catchError { _ in recovery } // #1
   .subscribe { print($0) }
   .disposed(by: bag)

subject.onError(MyError.error)
//출력값 없음
```

`#1`에서 catchError 연산자를 선언하고 `recovery` 를 파라미터로 전달했다. 

그리고나서 코드를 실행해보면, 이번에는 구독자에게 에러 이벤트가 전달되지 않았음을 확인할 수 있다. 

이는 원본 source Observable인 `subject`가 에러 이벤트를 방출하고 종료되자 catchError 연산자가 source Observable을 `subject` 대신 `recovery`로 교체했기 때문이다. 

```swift
...(전략)

subject.onNext(11) // #1
recovery.onNext(22) // #2
recovery.onCompleted() // #3

//출력값
//next(22)
//completed
```

그 이후에 `#1`처럼 `subject`에 next event를 전달해보면, 이미 에러 이벤트를 전달하고 종료된 Observable이기 때문에 더 이상 구독자에게 next event가 전달되지 않는다. 

반면 `#2`에서 `recovery`로 전달한 next event는 구독자에게 전달되고 있다. 

또 `#3`처럼 completed event를 전달하면 구독이 에러 없이 정상적으로 종료된다.

catchError 연산자는 source Observable에서 발생한 에러를 새로운 Observable로 교체하는 방식으로 처리한다.



---

### catchErrorJustReturn



catchErrorJustReturn 연산자는 에러 발생시 Observable이 아니라 기본값을 리턴한다. 

![스크린샷 2020-06-13 오후 10.05.52](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqyshe2z9j30m30bnacu.jpg) 

Source Observable에서 에러 이벤트가 발생하면 파라미터로 전달한 기본값을 구독자에게 전달한다. 그리고 파라미터의 타입은 항상 Source Observable이 방출하는 요소의 타입과 일치해야한다.

```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

let subject = PublishSubject<Int>()

subject
  .catchErrorJustReturn(-1) // #1
  .subscribe { print($0) }
  .disposed(by: bag)

subject.onError(MyError.error)

/*출력값
next(-1)
completed
*/
```



코드를 실행하면 subject로 에러 이벤트가 전달되고, `#1`에서 파라미터로 전달한 값이 구독자에게 전달된다. Source Observable은 더 이상 다른 이벤트를 전달할 수 없고, 파라미터로 전달한 것은 Observable이 아니라 그냥 하나의 값이기 때문에 더 이상은 전달될 이벤트가 없다. 그래서 바로 completed event를 전달하고 구독이 종료된다.

- 에러가 발생했을 때 사용할 수 있는 기본값이 있다면 `catchErrorJustReturn` 연산자를 사용한다. 하지만 발생한 에러의 종류와 무관하게 항상 같은 값이 리턴된다는 단점이 있다. 
- 나머지 경우에는 `catchError` 연산자를 사용한다. 클로저를 통해 에러 처리 코드를 자유롭게 작성할 수 있다는 장점이 있다. 
- 두 연산자와 달리 작업을 처음부터 다시 시작하고 싶다면 `retry` 연산자를 사용해야한다.



---

### retry, retryWhen



#### retry

![스크린샷 2020-06-13 오후 9.19.11](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqyzrv24qj30s50kddha.jpg)

`retry` 연산자는 Observable에서 에러가 발생하면, Observable에 대한 구독을 해제하고 새로운 구독을 시작한다. 새로운 구독이 시작되기 때문에 당연히 Observable 시퀀스는 처음부터 다시 시작된다. Observable에서 에러가 발생하지 않는다면 정상적으로 종료되고, 에러가 발생하면 또 다시 새로운 구독을 시작한다.



```swift
let bag = DisposeBag()

enum MyError: Error {
   case error
}

var attempts = 1

let source = Observable<Int>.create { observer in
   let currentAttempts = attempts
   print("#\(currentAttempts) START") // #1
   
   if attempts < 3 {
      observer.onError(MyError.error)
      attempts += 1
   }
   
   observer.onNext(1)
   observer.onNext(2)
   observer.onCompleted()
         
   return Disposables.create {
      print("#\(currentAttempts) END") // #2
   }
}

source
// #3
   .subscribe { print($0) }
   .disposed(by: bag)

/* 출력값
#1 START
error(error)
#1 END
*/
```



이 코드는 `attempts` 변수에 저장된 값이 3보다 작다면 에러 이벤트를 방출하고 변수를 1 증가시킨다.

그리고 `#1`, `#2`를 보면 시퀀스의 시작과 끝을 확인할 수 있는 로그가 추가되어있다.

코드를 실행해보면 구독자에게 바로 에러 이벤트가 전달되고 실행이 중지된다.

이제 `#3`에 retry 연산자를 추가해보자.

![스크린샷 2020-06-13 오후 10.17.11](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqz46d0dkj30mz0j2wj6.jpg)

retry 연산자는 두 가지 형태로 선언되어있다. 

첫 번째 형태처럼 파라미터 없이 호출하면 Observable이 정상적으로 완료될 때까지 계속해서 재시도한다. 만약 Observable에서 반복적으로 에러가 발생하면, 재시도 횟수가 늘어나고 그만큼 리소스가 낭비된다. 심한 경우 무한 루프에 빠져 터치 이벤트를 처리할 수 없거나 앱이 강제 종료되는 문제가 발생한다. 그래서 가급적 파라미터 없이 retry 연산자를 호출하는 것은 피해야 한다.

이어서 아래 쪽에 있는 두 번째 형태를 보면 최대 재시도 횟수를 파라미터로 받는다. 그래서 앞에서 설명한 문제는 발생하지 않는다. 재시도 횟수를 파라미터로 전달할 때에는 

![스크린샷 2020-06-13 오후 10.19.54](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqz6zq9r1j30j601bt9b.jpg)

**이 설명처럼 `원하는 재시도 횟수 + 1`을 전달해야한다**

```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

var attempts = 1

let source = Observable<Int>.create { observer in
  let currentAttempts = attempts
  print("#\(currentAttempts) START")
  
  if attempts < 3 { // #2
    observer.onError(MyError.error)
    attempts += 1
  }
  
  observer.onNext(1)
  observer.onNext(2)
  observer.onCompleted()
  
  return Disposables.create {
    print("#\(currentAttempts) END")
  }
}

source
  .retry() // #1
  .subscribe { print($0) }
  .disposed(by: bag)

/* 출력값
#1 START
#1 END
#2 START
#2 END
#3 START
next(1)
next(2)
completed
#3 END
*/
```

`#1`에 retry 연산자를 사용하고 코드를 실행하면 처음 두 번의 시도는 실패하고, 세 번째 시도에 성공한다. 

이런 결과가 나온 이유는 `#2`에서 설정된 조건으로 인해 `attempts`변수에 저장된 값이 3보다 작을 때에만 에러 이벤트를 전달하기 때문이다. 만약 조건을 `attempts > 0`으로 바꾼다면 계속해서 무한한 에러 이벤트가 전달될 것이다. 



```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

var attempts = 1

let source = Observable<Int>.create { observer in
  let currentAttempts = attempts
  print("#\(currentAttempts) START")
  
  if attempts > 0 { // #1
    observer.onError(MyError.error)
    attempts += 1
  }
  
  observer.onNext(1)
  observer.onNext(2)
  observer.onCompleted()
  
  return Disposables.create {
    print("#\(currentAttempts) END")
  }
}

source
  .retry(7) // #2
  .subscribe { print($0) }
  .disposed(by: bag)

/*출력값
#1 START
#1 END
#2 START
#2 END
#3 START
#3 END
#4 START
#4 END
#5 START
#5 END
#6 START
#6 END
#7 START
#7 END
error(error)
*/
```

 `#1`과 같이 조건을 변경하고 실행하자 7번의 시도가 콘솔에 출력되었다.

이는 `#2`에서 최대 재시도 횟수를 7로 설정해주었기 때문이다. 마지막 결과 역시 실패라면 구독자에게 에러 이벤트를 전달하고 구독이 종료된다. 

하지만 이건 엄밀히 말해 7번의 재시도가 아니라, 6번의 재시도이다. 왜냐면 첫 번째 시도는 재시도가 아니기 때문이다. 첫 번째 실행에서 에러 이벤트가 전달되면 다시 처음부터 실행하게 되는데, `#2 START`가 출력된 시점이 첫 번째 재시도인 것이다. 그래서 실제 재시도 횟수는 7회가 아니라 6회이다. 

이렇듯 최대 재시도 횟수를 전달할 때에는 원하는 횟수에 1을 더해서 전달해야한다. 그래야 원하는 횟수만큼 재시도할 수 있다.

그리고 만약 최대 재시도 횟수 이내에 작업이 성공하면 더는 시도하지 않는다.

retry 연산자는 에러 이벤트가 발생한 즉시 재시도하기 때문에 재시도 시점을 제어하는 것은 불가능하다. 네트워크 요청에서 에러가 발생했다면 정상적인 응답을 받거나 최대 횟수에 도달할 때까지 계속해서 재시도한다. 

만약 사용자가 재시도 버튼을 탭하는 순간에만 재시도하도록 하고 싶다면, `retryWhen` 연산자를 사용해야한다.



#### retryWhen

![스크린샷 2020-06-13 오후 10.30.36](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfqzi4oi5bj30mo0frwjg.jpg)

retryWhen 연산자는 클로저를 파라미터로 받는다. 클로저의 파라미터로는 발생한 에러를 방출하는 Observable이 전달된다. 그리고 클로저는 TriggerObservable을 리턴한다. TriggerObservable이 next event를 전달하는 시점에 Source Observable에서 새로운 구독을 시작한다. 다시 말해 `재시도`한다.

```swift
let bag = DisposeBag()

enum MyError: Error {
  case error
}

var attempts = 1

let source = Observable<Int>.create { observer in
  let currentAttempts = attempts
  print("START #\(currentAttempts)")
  
  if attempts < 3 {
    observer.onError(MyError.error)
    attempts += 1
  }
  
  observer.onNext(1)
  observer.onNext(2)
  observer.onCompleted()
  
  return Disposables.create {
    print("END #\(currentAttempts)")
  }
}

let trigger = PublishSubject<Void>() // #1

source
  .retryWhen { _ in trigger} // #2
  .subscribe { print($0) }
  .disposed(by: bag)
 // #3
/* 출력값
START #1
END #1
*/

trigger.onNext(()) // #4
/* 출력값
START #2
END #2 
*/

trigger.onNext(()) // #5
/*출력값
START #3
next(1)
next(2)
completed
END #3
*/
```

이전과 같은 코드에 `trigger` 상수를 `#1`에서 선언하고, `#2`의 retryWhen 연산자의 클로저가 trigger를 리턴하도록 작성된 코드이다. `#3`까지의 코드를 실행하면, Source Observable에서 에러 이벤트가 발생하기 때문에 구독자에게 전달되지 않는데, 바로 재시도하지 않고 `trigger`에서 next event가 발생하기를 기다린다.

이어 `#4`에서 `trigger` subject에 next event를 전달하면 Source Observable에서 새로운 구독이 시작된다. 그리고 이번에도 Source Observable에서 에러 이벤트가 발생하기 때문에 다시 대기한다. 

마지막 재시도에서는 에러가 발생하지 않는다. 이때는 next event와 completed 이벤트가 정상적으로 전달되고, 구독이 종료된다.



---

## 19. RxCocoa

RxCocoa는 기존 Cocoa Framework에 Reacive Library의 장점을 더 해주는 라이브러리이다. iOS 플랫폼 뿐아니라 Apple에서 지원하는 모든 플랫폼을 지원한다.

RxCocoa는 RxSwift를 기반으로하는 별도의 라이브러리이다. 그래서 RxCocoa를 사용하려면 Podfile 내부에 개별적으로 추가해야한다.

![스크린샷 2020-06-14 오전 12.36.12](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr34tl9tcj309m024mxj.jpg)



---

#### UIButton + Rx

Pods>Pods>RxCocoa > UIButton+Rx.swift 파일을 보자.

![스크린샷 2020-06-14 오전 12.37.33](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr3689m5jj30g205bq3s.jpg)

여기에는 UIButton Class를 확장한 코드가 있다. 

extention으로 Reactive 형식을 확장하고 있는데, 기존 Cocoa 클래스를 확장한 경우 대부분 이런 식으로 구현되어 있다. 

![스크린샷 2020-06-14 오전 12.39.57](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr38osjlsj30ce08hab3.jpg)

우선 위에서 확장하고 있는 `Reactive`가 어떤 형식인지 보자. Reactive는 RxSwift 라이브러리에 제네릭 구조체로 선언되어있다. 



![스크린샷 2020-06-14 오전 12.40.53](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr39pcizrj30lw026glv.jpg)

이 설명과 같이, 형식을 Reactive 형식으로 확장할 때 사용하는 형식이다. 

Reactive 구조체 안에는 `base` 라는 형식이 선언되어 있는데, 확장할 형식의 인스턴스가 저장된다. 

![스크린샷 2020-06-14 오전 12.42.01](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr3awdsk5j30iv0au76g.jpg)

아래쪽에는 `ReactiveCompatible` 프로토콜이 선언되어 있다. 

이 프로토콜의 역할은 기존 형식에 `rx`라는 속성을 추가하는 것이다.

이것은 rx라는 네임 스페이스를 추가하는 것과 같다. RxCocoa가 제공하는 대부분의 extension은 이 rx속성, 다시 말해 네임 스페이스를 통해 제공된다.

![스크린샷 2020-06-14 오전 12.44.01](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr3cx89k8j30i80g440h.jpg)

다시 아래 쪽을 보면, `ReactiveCompatible` 프로토콜에 기본 구현을 추가하는 프로토콜 extension이 있다. 이 프로토콜에는 `rx`라는 타입 프로퍼티와, instance 프로퍼티가 자동으로 추가된다. 

> ####  + 프로토콜 기본 구현에 대해 (Default Implementation)
>
> Swift의 프로토콜은 기본적으로는 objc에서의 프로토콜과 같은 기능을 수행한다. 그렇지만 Swift에서는 Protocol을 Extension 할 수 있게 됨으로써 '기본 구현'이 가능해졌다.
>
> 부가적인 기능을 추가할 수 있는 기능인 Extension을 통해 프로토콜 자체에 메서드를 구현해줄 수 있고, 어떤 연산 프로퍼티 또한 제공해줄 수 있게 되었다.
>
> objc의 경우 프로토콜에 카테고리를 추가해줄 수 없었는데 Swift에서는 그것이 가능해졌다.
>
> 그래서 Protocol에 특정 타입이 할 일을 명시해주는 것과 동시에 그 역할들을 수행하는 기능을 구현해주는 것까지 한 번에 가능하다.



![스크린샷 2020-06-14 오전 12.45.36](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr3embjqjj30bo03bq3f.jpg)

코드 마지막 부분에는 NSObject가 `ReactiveCompatible` 프로토콜을 채용하도록 하는 코드가 있다. NSObject는 Cocoa Framework에 있는 모든 클래스가 상속하는 Root 클래스이기 때문에 결과적으로 모든 클래스에 `rx`라는 속성, 즉 네임 스페이스가 자동으로 추가된다. 

> 네임 스페이스에 대한 글은 유명한 블로그인 ZeddiOS의 https://zeddios.tistory.com/353 를 참고하면 좋다.



![스크린샷 2020-06-14 오전 12.37.33](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr3689m5jj30g205bq3s.jpg)

다시 처음의 UIButton에 관한 Extension을 보자. extension은 조금 전에 다룬 Reactive 형식을 확장하고 있고, Base를 UIButton으로 제한하고 있다. UIButton은 NSObject를 상속한 속성이기 때문에 `rx` 속성이 자동으로 추가된다. 

이 `rx` 속성을 통해서 extension에 선언되어있는 멤버(이 때에는 `tap`)에 접근할 수 있다.

이 익스텐션에는 `tap`이라는 멤버가 ControlEvent 형식으로 선언되어 있다. ControlEvent는 RxCocoa가 제공하는 `Traits`이다. `Traits`는 UI 처리에 특화된 Observable이고, ControlEvent 뿐만 아니라 `driver`, `signal` 등 고유한 특성을 가진 `Traits`가 제공된다. 

이후에 `Traits`에 대해 다루겠지만, 지금 당장 더 궁금하다면 아래 주소에서 조금 더 많은 정보를 확인할 수 있다.

https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Traits.md

https://kampro.github.io/swift/ios/2019/03/20/rxswift-traits.html



다시 돌아가서, `tap`은 특별한 Observable이기 때문에 구독할 수 있다. 버튼에서 TouchUpInside Event가 발생할 때마다, 구독자에게 next event가 전달된다.



---

#### UILable + Rx

![스크린샷 2020-06-14 오전 1.16.30](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr4aqeqvzj30g90f90uz.jpg)

UILable + Rx의 익스텐션 구조는 UIButton과 동일하다. 여기에는 `text` 속성과 `attributedText` 속성이 선언되어 있다. 

그런데 이 속성은 기존 Cocoa의 UILable에 있는 속성과 동일하다. 하나의 형식에 같은 이름을 가진 멤버가 두 개 이상 존재할 수 없지만 여기에서는 전혀 문제가 되지 않는다. 

앞에서 설명했듯이 여기에 있는 멤버는 `rx` Name Space에 추가되기 때문이다.

```swift
let label = UILable()
label.text // #1
label.rx.text // #2
```

UILable에 있는 text 속성에 접근할 때는 인스턴스 이름을 통해 `#1`과 같이 바로 접근하고,

RxCocoa가 확장한 text 속성에 접근할 때는 `rx` 네임스페이스를 통해 접근하기 때문이다. 

그리고 같은 이름을 가지고 있기 때문에 오히려 기존 속성과의 연관성이 더 직관적으로 표현된다. 

`text` 속성의 형식을 보면 `Binder`형식으로 지정되어 있다. `Binder`는 Interface Binding에 사용되는 특별한 Observer이다. 

![스크린샷 2020-06-14 오전 1.23.16](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr4hydgydj306d0iydjp.jpg)

나머지 파일을 보아도 기존 Cocoa에서 익숙하게 사용하던 이름들이 보인다. RxCocoa는 이런 자주 사용하는 기본 기능들에 대한 확장을 제공한다. 만약 필요한 확장이 없다면 직접 구현하는 것도 가능하다. 

---

#### Cocoa --> RxCocoa



![스크린샷 2020-06-14 오전 1.30.18](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr4p4tzsij30ea0cc0u9.jpg)

lable과 button 하나를 갖는 화면이다. 버튼을 탭하면 Lable의 text를 "Hello, RxCocoa"로 바뀌는 걸 구현해보도록 하자.

RxCocoa에서 탭 이벤트를 사용할 때에는, 위에서 확인했던 `tap` 속성을 활용해야 한다.

![스크린샷 2020-06-14 오전 1.27.07](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr4lshuzsj30i803xq4k.jpg)

`tap`속성은 ControlEvent 형식으로 선언되어 있고, TouchUpInside Event가 발생하면 Next Event를 방출하는 특별한 Observable이다. 

이때 `map` 연산자를 사용하면 `tap`에서 방출된 next event를 문자열로 바꿀 수 있다.

```swift
   override func viewDidLoad() {
      super.viewDidLoad()
    
    tapButton.rx.tap
      .map { "Hello, RxCocoa" }
    
   }
```

이후, 구독자를 추가해야 Observable이 생성되고 이벤트를 방출하기 시작할 것이다.

RxSwift에서는 구독자를 추가할 때 `subscribe` 메소드를 이용했는데, RxCocoa는 더 쉬운 방법을 제공한다.

```swift
    tapButton.rx.tap
      .map { "Hello, RxCocoa" }
      .bind(to: valueLabel.rx.text) // #1
      .disposed(by: bag)
```

`#1`의 `bind` 메소드인데, 이렇게 하면 전달된 문자열이 `valueLable`의 `rx.text` 속성에 바인딩된다. 실행하여 버튼을 탭하면 레이블이 업데이트된다.



---

### Binding

![스크린샷 2020-06-14 오전 2.28.03](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr6dacjqkj30kt09laar.jpg)

바인딩은 여러 의미로 사용되지만, 이번에는 데이터를 UI에 표시하기 위한 방법으로 사용한다.

바인딩에는 데이터 생산자(Data Producer)와 데이터 소비자(Data Consumer)가 있다. 

데이터 생산자는 Observable이다. ObservableType을 채용한 모든 형식이 생산자가 된다.

데이터 소비자는 Label이나 ImageView 같은 UI Component이다. 생산자가 생산한 데이터는 소비자에게 전달되고, 소비자는 적절한 방식으로 데이터를 소모한다. 예를 들어 Label은 전달된 Text를 화면에 표시한다. 

반대로 소비자가 생산자에게 데이터나 이벤트를 전달하는 경우는 없다. 

Binder는 UI Binding에 사용되는 특별한 Observer이다. 데이터 소비자의 역할을 수행한다.

Observer이기 때문에 Observable이 바인더에게 새로운 값을 전달할 수는 있지만, 

Binder는 Observable이 아니기 때문에 구독자를 추가하는 것은 불가능하다. 

![스크린샷 2020-06-14 오전 2.26.10](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr6b9b0kfj30mr0c2t9e.jpg)

그리고 바인더는 에러 이벤트를 받지 않는다. 만약 Error Event를 전달하면, 실행 모드에 따라 Crash가 발생하거나, 에러 메시지가 출력된다. 

Observer에서 에러 이벤트가 전달되면, Observable 시퀀스가 종료되는데 이 경우 더 이상의 next event가 전달되지 않기 때문에 Binding된 UI가 더 이상 업데이트 되지 않는 문제가 있다.

이 문제를 피하기 위해 Error 이벤트를 받지 않는 것이다. 

Binding이 성공하면 UI가 업데이트 된다. UI 코드는 메인 스레드에서 실행해야한다.

바인더는 바인딩이 메인 쓰레드에서 실행되는 것을 '보장'한다.

  

#### Binding 구현



![스크린샷 2020-06-14 오전 2.58.13](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr78mzu7nj30v30u0al8.jpg)

텍스트 필드에 어떤 텍스트를 입력할 때마다, `textLable`의 텍스트가 `textfield`에 입력한 텍스트로 바뀌는 기능을 RxCocoa로 구현해보자.

먼저 Pods > Pods > RxCocoa > UITextField + Rx.swift 파일을 보자.

 ![스크린샷 2020-06-14 오전 3.00.43](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr7b62dp7j30da04gwf5.jpg)

여기에 보면 `text` 속성이 ControlProperty 타입으로 선언되어있다. ControlProperty는 데이터를 특정 UI에 Binding할 때 사용하는 특별한 Observable이다. 

그리고 ControlProperty의 타입 파라미터가 Optional String으로 선언되어있다. 

text 속성이 변경될 때마다 next event를 방출하는데, 이 next event에는 Optional String 타입의 데이터가 저장되어 있는 것이다. 

```swift
  let bag = DisposeBag() //먼저 DisposeBag() 인스턴스 선언

override func viewDidLoad() {
    super.viewDidLoad()
    
    textField.becomeFirstResponder()
    
    textField.rx.text // #1
    
    setupUI()
  }
```



먼저 `textField`속성의 text에 접근해야한다. `#1`과 같이 `rx` 네임 스페이스를 통해 접근한다. `textField.rx.text` 속성은 ControlProperty 형식으로 선언되어있고, ControlProperty은 특별한 Observable이다. 한 종류의 Observable이기 때문에 구독이 가능하다. 

구독 방법은 이전까지와 완전히 동일하다.

```swift
  override func viewDidLoad() {
    super.viewDidLoad()
    
    textField.becomeFirstResponder()
    
    textField.rx.text
      .subscribe(onNext: { [weak self] str in
        self?.textLabel.text = str
      })
      .disposed(by: bag)
    
    setupUI()
  }
```

RxCocoa가 확장한 text 속성은 텍스트 필드에 입력된 값이 업데이트 될 때마다 next event를 전달한다. 구독자는 이벤트에 포함된 문자열을 Label에 포함하고 있다. 

![스크린샷 2020-06-14 오전 3.09.20](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr7k5sxvrj30u00vmamk.jpg) 

실행해보면 잘 동작한다. 하지만 Cocoa로도 구현해본다고 생각하고 비교해보면 훨씬 단순해졌다는 걸 알 수 있다.

UITextFieldDelegate를 구현할 필요가 없고, 최종 문자열을 조합하는 코드도 필요하지 않아졌다. 그리고 코드만으로 데이터 흐름을 파악하기가 용이해졌다. 

```swift
    textField.rx.text
      .subscribe(onNext: { [weak self] str in
        self?.textLabel.text = str // #1
      })
      .disposed(by: bag)
```

하지만 이 코드에는 한 가지 문제가 있다. 

UI를 업데이트하는 `#1` 코드는 반드시 메인 쓰레드에서 실행되어야 한다. 

지금은 subscribe 메소드가 Main Thread에서 호출되기 때문에 문제가 없지만, 구현에 따라서는 Background Thread에서 실행될 수도 있다. 

이런 문제를 해결하는 방법은 크게 두 가지가 있다. 

1. GCD를 사용하여 쓰레드를 지정해주는 방법

   ```swift
   textField.rx.text
         .subscribe(onNext: { [weak self] str in
           DispatchQueue.main.async {
             self?.textLabel.text = str
           }
         })
         .disposed(by: bag)
   ```

   

2. observeOn 메소드를 사용하는 방법

   ```swift
       textField.rx.text
         .observeOn(MainScheduler.instance)
         .subscribe(onNext: { [weak self] str in
             self?.textLabel.text = str
         })
         .disposed(by: bag)
   ```

   

1번처럼 기존 GCD를 이용하거나, 2번과 같이 rx를 활용하여 메인 쓰레드에서 동작하도록 지정해줄 수 있다. 

하지만 이 두 개의 방법은 RxCocoa에서는 거의 쓰이지 않는다. 더 나은 방법이 있기 때문이다. 

다시 Pods > RxCocoa폴더의 UILabel + Rx.swift 파일을 보자. 

![스크린샷 2020-06-14 오전 3.20.19](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr7vm352bj30gc0d340j.jpg)

여기에는 두 가지 속성이 선언되어 있는데, 이들의 타입을 보면 앞에서 공부한 `Binder`로 선언되어있다. Binder는 UI Binding에 활용하는 특별한 Observer이다. 이 속성을 활용해보자. 

```swift
  override func viewDidLoad() {
    super.viewDidLoad()
    
    textField.becomeFirstResponder()
    
    textField.rx.text
      .bind(to: textLabel.rx.text)
      .disposed(by: bag)
    
    setupUI()
  }
```

subscribe 메소드 대신 bind를 사용한다. bind 메소드 다양한 파라미터를 받는데, 이중에서 ObserverType을 받는 메소드를 사용해보았다. `UILabel.rx.text`의 형식인 `Binder`는 ObserverType을 채용한 형식이다. 그래서 `#1`처럼 text 속성을 파라미터로 전달할 수 있다. 

![화면 기록 2020-06-14 오전 4.19.53.mov](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfr9nu7iedg30720dwh0g.gif)

실행하여 text를 입력하면, UITextField에 선언되어있는 text 속성이 업데이트 될 것이다. 

그러면 RxCocoa가 확장한 rx.text 속성이 next event를 방출한다. 이 next event에는 현재 text가 저장되어있다. 

이때 bind(to: )메소드는 Observable이 전달한 이벤트를 Observer에게 전달한다. 

여기에서는 RxCocoa가 UILabel에 추가한 text속성으로 전달된다. 이 text는 Binder 형식이다. next event에 포함되어있는 값을 꺼내서, UILabel에 있는 기본 text 속성에 저장한다.

결과적으로 textField의 text와 textLabel의 text가 바인딩되어서, 항상 동일한 값을 표시한다.

이전의 코드들보다 훨씬 적은 코드로 같은 기능을 구현할 수 있을 뿐만 아니라, Binder는 언제나 Binding 작업을 Main Thread에서 실행하기 때문에 더 이상 쓰레드에 대해 신경쓸 필요가 없다는 장점이 있다. 



---

