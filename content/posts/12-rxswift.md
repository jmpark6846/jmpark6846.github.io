---
title: "RxSwift 간단 정리"
date: 2023-03-02T19:23:57+09:00
draft: false
summary: " "
tags: ["MVVM", "RxSwift", "ios"]
---

### RxSwift란
* 비동기로 생기는 데이터를 편하게 사용하기 위해 사용한다. 
* 비동기로 생기는 데이터를 클로져를 Completion Handler로 받아서 호출해왔지만 사용하기가 불편하다. 
* 클로져가 아닌 값으로 받아서 다루기 위해 만들어진 유틸리티이다.
    
비동기로 처리된다는 건 나중에 처리된다는 것이다.
* '나중에 생기는 데이터'를 `Observable`, `Promise` 등 으로 부르고
* '나중에 생기면 실행'할 작업을 명시하는 것을 `subscribe`, `then` 으로 부른다. 

### Observable 실행과 종료
`Observable`은 생성하는 것만으로는 실행되지 않고 `subscribe`해야 실행된다.   
subscribe를 실행하면 크게 세 가지 이벤트가 실행된다.  
* next: 데이터 전송
* complete: 실행 완료 
* error:에러 

complete와 error 이벤트를 호출하거나 subscribe의 리턴타입인 Disposables를 이용해 dispose하면 실행이 종료된다. 실행이 종료되면 클로져가 사라진다.  
  
클로져 내에서 self를 사용하더라도 complete, error 이벤트 혹은 dispose 호출하면 클로져가 사라져서 self에 대한 참조카운트가 감소한다. 따라서 순환참조 문제가 해결된다.

Observable이 subscribe 후 complete, error 혹은 dispose 되어 실행이 종료된다.
종료된 Observable은 후에 subscribe하거나 사용하더라도 다시 실행되지 않는다.



### 처음 RxSwift를 사용한다면 크게 두 가지를 신경쓰자.  

1. 비동기로 생기는 데이터를 Observable로 감싸서 리턴하는 방법
    1. Observable.create {} 속에서 비동기 처리를 실행
    2. onNext(data)로 데이터 전달, onComplete, onError 실행
    3. disposable 생성하여 리턴

2. Observable로 오는 데이터를 받아서 처리하는 방법
    1. Observable에 subscribe하여 이벤트를 처리할 클로져를 넘긴다.
    2. event에 맞는 처리
    3. disposable을 disposeBag에 추가



### Operator

1번과 2번을 반복해서 사용하다보니 더 간편한 방법이 필요하다! 그래서 reactive 에서는 여러가지 `Operator`를 제공한다.

* 간단한 생성 : `just`, `from`
* 필터링 : `filter`, `take`
* 데이터 변형 : `map`, `flatMap`
* `subscribe(onNext)`
* `subscribeOn`: 데이터를 받아오는 작업을 실행할 스케줄러 지정
* `observeOn`: 지정되는 데이터를 보낼 작업을 실행할 스케줄러 지정. 이후 추가되는 작업들에 적용.
* `combine`, `merge`, `zip`
* `disposed`: 비동기작업을 클래스 변수 `DisposeBag`에 추가하면 해당 클래스가 종료될때 자동으로 dispose됨

cf) 스케줄러는 작업큐의 래퍼 클래스



## 역할 정리

### Observable
* 값을 전송하는 역할. 값을 전송만 하고 외부에서 데이터를 받지는 못한다.

### Observer
* 값을 전송받아 작업을 수행하는 역할

### Subject
* 값을 받기도 하고 전송하기도 하는 역할. 즉 `Observable`이자 `Observer`
* `Observable`과 마찬가지로 `complete`, `error` 이벤트가 발생되거나 dispose되면 실행이 종료된다. 이게 의도된 설계이지만 실제로 사용해보면 문제가 될수있다. 
	* 실수로 `onComplete`을 호출하거나 에러가 발생할 경우 `Subject`의 실행이 종료되어 더이상 사용할 수 없고, `Subject`를 사용하는 UI 역시 동작하지 않는다.
	* 따라서 이런 문제를 해결하기 위해 사용하는 것이 `Relay`

### Relay
* `Subject`와 동일한데 `onComplete`, `onError`를 호출하지 않고 데이터를 전송만한다. 

