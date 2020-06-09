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
