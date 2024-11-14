---
title: "제너레이터 함수"
author:
  name: yeong30
  link: https://github.com/yeong30
date: 2022-07-18 20:00:00 +0900
categories: [JavaScript]
tags: [JavaScript]
---

redux-saga를 사용방법을 보다보면 제너레이터 문법이 등장한다.
현재는 redux thunk만을 사용하고 있지만 saga로 전환을 고민하고 있기에 제너레이터 문법에 대해 우선 학습하고자 한다.

그렇다면 
## 제너레이터 문법이란?
제너레이터 문법이란 제너레이터 함수 라는 특별한 문법으로 생성되는 객체이다. 
### 제너레이터 함수란? 
제너레이터 객체를 반환하는 특별한 함수로 <code>function*</code> 이라는 문법을 사용해서 정의되는 함수이다.
내부에서 yield 라는 특별한 키워드를 사용한다. 

일반적으로 함수는 하나 또는 아무값도 반환하지않는다(void).  그러나 제너레이터함수와 그 반환 객체 제너레이터를 활용하면 여러개의 값을 순차적으로 반환할 수 있다.

예를들어 
```
function* generateSomething(){
	yield 1;
	yield 2;
    return 3;
}
const generator = generateSomething();
```
위 코드를 실행하면 generator 변수에 제너레이터 객체가 담기게 된다. 
### 제너레이터  객체
제너레이터 객체는 각각 next()라는 메서드를 가지고 있다. 이 next메서드를 실행하면 원본 함수에서 아직 미실행된 yield중 가장 가까운 yield문을 만날때까지 함수가 실행된다. 그리고 yield를 만나면 함수 실행이 중단되고 yield에 해당하는 반환된다. 
정확히는 value , done의 키값을 가진 객체가 반환된다.

우리는 이러한 형태(next메서드와 value,done객체)를 많이 봐았다. 바로 이터레이터 객체이다. 
간단하게 이터레이터 객체를 복습하자면
> 이터레이터 객체
:  Symbole.iterator가 호출되었을때 반환되는 객체로서 next()라는 메서드를 키값으로 가진다. 각각의 next는 value와 done을 반환하며 done 이 true일 때 반복이 종료됨을 의미한다.

위와 같은 이터레이터 객체를 보았을때 제너레이터 객체는 이터레이터 객체와 거의 유사함을 알 수 있다. 

또한 next메서드를 가진 가진 제너레이터 객체를 통해 반복이 가능하므로 이터러블이라고도 표현할 수 있다.

## 제너레이터 사용하기 
그렇다면 이러한 제너레이터 객체를 어떻게 사용할 수 있을까. 
위에 서술했듯이 제너레이터 객체는 next메서드를 가졌으며 이를 통해 함수의 반환값을 얻을 수 있다. 
위에 예시로 든 코드에서 <code>generateSomething</code> 함수를 대상으로 예시를 보자.
```
const generator = generateSomething();
let first  = generator.next();
console.log(first) // {value : 1 , done : false}

let second  = generator.next();
console.log(second) // {value : 2 , done : false}

let third = generator.next();
console.log(third); // {value : 3 , done : true}

let fourth = generator.next();
console.log(fourth); // {value : undefined, done : true}

```

처음으로  <code>generaotr.next()</code> 를 실행한다면 가장 첫번째 값 1이 반환 될 것이다. 그리고 다른 yield가 남아있으므로  <code>done:false</code>이다.
현재 함수는 첫번째 줄에 멈춰있는 상태이다. 
다음으로  <code>generaotr.next()</code>를 실행하면 두번째 반환값인 2가 반환되며  아직 함수가 종료되지않아서 <code>done:false</code>이다. 
다음 <code>generaotr.next()</code>는 3이 반환되며 함수가 종료되고 <code>done:true</code>가 된다.

함수가 완전히 종료되었으므로 이제 <code>generaotr.next()</code>를 실행해도  <code>value : undefined , done:true</code>가 반환된다.

### for~of
이터러블을 배우며 이터러블 객체는 for-of로 이터러블 객체를 실행하면 next()메서드가 자동으로 호출되며 done:true일때까지 반복 실행됨을 배웠다.
제너레이터 객체도 이터러블 객체이므로 for-of로 반복이 가능하다. 
단 for-of는 done:true일때 value를 무시하므로 마지막 값을 제대로 출력하기 위해서는 마지막값을 return이 아닌 yield로 표현하는등의 처리가 별도로 필요하다.

## 제너레이터 컴포지션
제너레이터 컴포지션이란 제너레이터 안에 제너레이터를 임베딩하는 특별 기능이다.
쉽게 설명하면, 제너레이터 안에 또다시 제너레이터를 정의함으로서 제너레이터안에 제너레이터를 끼워넣을 수 있다. 이렇게 제너레이터를 끼워넣으면 중첩된 제러네이터 ( = 끼워넣어진 제너레이터)로 실행자가 위임된다. 즉 외부 제너레이터를 순서대로 호출하더라도 첫번째 yield가 가진 제너레이터가 차례대로 호출되며 첫번째 yield가 가진 제너레이터가 모두 호출되고 나서야 외부 제너레이터의 다음 yield가 실행된다. 

사실, 중첩된 제너레이터의 외부 제너레이터에 작성하여도 똑같은 결과가 나타나나 중첩을 통해 조금 더 깔끔하게 코드를 작성할 수 있으며 중간 결과 저장용도의 메모리 사용이 줄어든다. 

## 값 전달하기
제너레이터의 이터레이터 객체와 가장 큰 차이 중 하나는 외부에서 객체 안으로 값을 전달할 수 있다는 것이다.
<code>generator.next(value)</code>를 사용해서 직전 호출된 yield의 결과값을 할당 할 수 있다.

위에 작성한 코드를 조금 수정해서 예시를 보자.
```
function* generateSomething() {
  let first = yield 1;
  let second = yield 2;
  console.log(first, second); //first call! second call!
}
const generator = generateSomething();
let first = generator.next();
let second = generator.next("first call!");
let third = generator.next("second call!");
```

우선 가장 처음에 첫번째 반환값을 조회한다. 이때는 next에 아무리 값을 전달해도 제너레이터 내부로 전달되지않는다. 
두번째 값 호출 시 next에 값을 전달하면, 해당값을 첫번째 yield의 반환값이 된다. 
서로 다른 순서의 호출임에도 가능한 이유는 첫번째 yield 를 호출 후 실행자가 첫번째 yield 줄에서 대기를 하다가, 다음  next가 호출되면 next에 들어온 매개변수를 첫번째줄에 할당하고 다음 yield로 이동하기 때문이다. 
### 에러 전달
값을 전달할 수 있기 때문에 오류도 전달이 가능하다. 일반적인 값 전달과 다른점은 next메서드 대신 generator.throw 메서드를 사용해서 명시적으로 에러임을 알려주어야한다.
generaotr객체 내부에 에러가 전달되면 해당 객체는 더이상 실행되지 않는다.(에러처리를 하지않으면 스크립트가 죽으므로 try..catch등 에러 핸들링은 필수이다!.. )


## 정리
사실, 모던 자바스크립트에서는 제너레이터를 잘 사용하지 않는다고 한다. 그렇기도 한게, 난수를 생성할때를 제외하고는 위와 같은 방식으로 상태값이 드러나지 않는 함수를 순차적으로 호출할 경우는 많지 않을것이라 본다. 

다만 이터레이터 객체가 필요할 경우 제너레이터를 사용하면 편리하게 사용할 수 있으며 함수를 실행중에 데이터를 주고받을 수 있다는 장점이 있다.

비동기 이터레이터를 사용하면 더 유용하게 사용할 수 있다. 


참고
----- 
[자바스크립트 인포](https://ko.javascript.info/generators)   
