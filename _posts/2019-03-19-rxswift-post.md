---
layout: post
title: "RxSwift"
description: "RxSwift 시작하기"
date: 2019-3-14
tags: [reactiveX, RxSwift, swift, iOS]
comments: true
share: true
---

# RxSwift 시작하기
rxSwift는 swift에 ReactiveX를 접목시킨 라이브러리로 비동기 처리를 간결하게 도와주기 때문에 많이 사용되고 있습니다.  

아래 글은 코드로 작성해 보기 전 간략하게 개념을 짚고 넘어가기 위해 [reactiveX](http://reactivex.io/) 문서를 참고하여 작성하였습니다.

## ReactiveX의 특징

- **Functional**  
observable stream을 통하여 가변데이터가 없는(stateless) 함수형 프로그래밍을 지향합니다.
- **Less is more**  
operator는 복잡한 코드를 짧게 줄여줍니다.
- **Async Error Handling**  
오류 처리를 위한 핸들링이 강력합니다. (전통적인 try, catch에서는 캐치할 수 없는 에러)
- **Concurrency made easy**  
Observables, Schedulers는 단순한 threading과 동기화 및 동시성 문제를 추상화 시켜줍니다.

## 모든것은 스트림이다
***사실 rxSwift의 핵심은 스트림입니다.***    
rxSwift는 비동기처리를 stream으로 만들어 선형처럼 표현합니다. 

## rxSwift 구성요소 

### observable
observer는 observable을 구독하며, observable이 배출하는(emit) 아이템, 혹은 시퀀스에 반응(react)합니다. 따라서 observable이 배출하는 것을 감지하는 observer를 observable안에 두고 알림을 받으면 되기 때문에 동시적인 연산(Concurrency)이 가능합니다.  
즉, 개발자가 작성한 코드의 순서가 아니라, observer(reactor)가 정한 임의의 순서대로 병렬로 실행되고, 결과는 나중에 연산되기 때문에 하나의 코드블럭이 실행결과를 리턴할 때 까지 기다릴 필요가 없습니다. (병렬실행으로 인해 가장 긴 시간이 소요되는 코드블럭이 최종적으로 수행되는 시간이 됩니다)  

observer와 observable은 subscribe를 통해 연결됩니다. (구독하여 리액트되면 데이터를 처리). 

##### Subscribe의 종류
- **onNext** : observable이 새로운 항목을 배출(emit)할 때 마다 호출.
- **onCompleted** : 오류가 없다면 onNext 다음에 호출.
- **onError** : 오류가 발생하면 호출되고, onNext, onComplete는 호출되지 않는다.

### operators
operator들은 observable상에서 동작하고 observable을 리턴하기 때문에 체이닝이 가능합니다.  
**마치 스트림처럼** 이전 operator가 리턴한 observable에 따라 동작하고 변형합니다.  

operator의 동작방식은 마블을 통해서 쉽게 알아볼 수 있습니다. 아래 페이지를 참조해 주세요.  
[Try Rx Marbles](https://rxmarbles.com)  


### single
observable과 유사하지만 onSuccess(하나의 값만 배출), onError 두가지 메서드만 사용합니다.


### subject
subject는 observer이면서 observable이기도 합니다.
그래서 observable을 구독하면서 동시에 아이템을 재배출하고 관찰하며 새로운 항목들을 배출할 수 있습니다.


### subject의 종류
- **AsyncSubject** : 맨마지막 값만 배출, observable 동작이 끝난 후 동작
- **BehaviorSubject** : 가장 최근에 발생한 아이템을 배출, 이 후 observable에 의한 결과 아이템을 계속 배출
- **PublishSubject** : 소스 observable이 배출한 아이템만 다음 observer에게 전달
- **ReplaySubject** : 구독시점과 관계없이 소스observable이 배출한 아이템 모두를 전달


### scheduler
멀티스레딩 적용을 하고 싶을 때 스케쥴러를 사용할 수 있습니다.
observable과 operator 는 스케쥴러를 통해 동작 
subscribe 메서드가 스케쥴러를 통해 observer에게 알림을 보냄

- **sucscribeOn** : observable이 사용할 스레드 지정, operators chain중 아무데서나 호출가능
- **observeOn** : 특정 operator를 별도의 스레드에서 실행하기 위함이므로 operators chain중 한 군데 이상에서 호출됨


## Rx는 어떻게 사용해야 하는가
rxSwift로 실제 코드를 작성하기 위해서는 함수형 프로그래밍과 반응형 프로그래밍 두가지 개념을 먼저 익히고 코드를 짜는 사고방식을 전환하는 연습이 필요하다고 생각합니다.
실제 코드를 작성하면서 어떻게 rxSwift에 익숙해질 수 있는지에 대해서도 다루어 보고자 합니다.




