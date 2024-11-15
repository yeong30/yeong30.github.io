---
title: "자바스크립트에서 다중 상속이 가능할까?"
date: 2024-06-11 20:00:00 +0900
categories: [JavaScript]
tags: [JavaScript]
---

커뮤니티를 구경하다 "자바스크립트에서 다중 상속이 가능한가?"라는 질문에 대한 글을 보게 되었다.

사실 "예", "아니요"로도 답할 수 있는 간단한 질문이지만 자바스크립트를 사용하면서도 클래스를 자주 사용하지 않았기 때문에 해당 질문이 낯설게 느껴졌다. <br>
프로토타입을 비롯해 자바스크립트의 기본 개념을 많이 잊어버렸다는 반성의 시간을 가지며 다시 하나하나 정리해 보고자 한다.

-----

우선, 다중 상속이란 어떤 클래스가 두개 이상의 클래스를 상속받는 것을 말한다. 위키백과에 따른 정의는 아래와 같다.

> 다중상속(Multiple inheritance)이란 객체 지향 프로그래밍의 특징 중 하나이며, 어떤 클래스가 하나 이상의 상위 클래스로부터 여러 가지 행동이나 특징을 상속받을 수 있는 것을 말한다.


클래스의 다중 상속이라는 문장을 처음 보았을때 과거 자바에서  클래스를 하나만 상속하였던 것과 타입스크립트 선언 시 extends 로 여러개의 인터페이스를 연결하였던 것이 떠올랐다. 


### 1. 자바에서 클래스 상속은 하나만 가능한가?
자바스크립트의 클래스 상속을 찾아보기 전에, 다른 객체 지향 프로그래밍 언어인 자바를 우선 찾아보았다. 결론부터 말하자면 Java는 클래스로부터 다중 상속을 허용하지 않는다. 다중 상속의 가장 큰 문제인 [다이아몬드 문제](https://f-lab.kr/insight/solving-java-diamond-problem) 가 발생할 수 있기 때문이다.<br>
반면, 파이썬,C++등은 다중상속울 지원한다. 파이썬은 다이아몬드 상속에 대한 해결책으로 메서드 탐색순서(MRO)를 통해 클래스 순서에 따라 메서드를 사용한다.

### 2. 타입스크립트의 interface는 여러개를 상속할 수 있다?
그렇다. 타입스크립트의 인터페이스는 다중 상속이 가능하다. 그리고 자바의 인터페이스 모두 다중 상속이 가능하다. 이유는 인터페이스는 기능에 대한 선언만 해두는 것이기 때문에 다이아몬드 상속으로 인한 오류가 발생하지 않기 때문이다. 다만 각 인터페이스에 속성 명이 동일한 속성의 타입이 동일하지 않은 경우에는 오류가 발생한다. 

### 3. 자바스크립트에서 클래스 다중 상속을 시도하면 어떻게 될까?
당연히 오류가 발생한다. 다만 어떠한 방식으로 어떤 오류 메시지가 출력되는지 궁금하여 다중상속을 테스트해보았다.

```
class Person {
  talk(){}
}
class Student {
 talk(){}
}
class Me extends Student, Person{} 
//SyntaxError: Unexpected token ','

```
실행 결과는 역시 오류가 발생하였지만 런타임에서 오류가 발생할 것이라는 예상과 다르게 문법 오류가 발생하였다. 


### 4. 그렇다면 자바스크립트에서 다중 상속은 절대 불가능한가?
문법상으로는 그러하나, 다른 방법으로 구현이 가능하다. 믹스인을 사용하여 다중상속처럼 구현이 가능하다. 
믹스인은 다른 클래스를 상속받을 필요 없이 이들 클래스에 구현되어 있는 메서드를 담고 있는 클래스라고 정의된다. 
믹스인을 활용하면 특정 클래스를 상속받는 동시에, 다른 믹스인의 메서드도 사용할 수 있다.

참고
-----

[위키백과 - 다중 상속](https://ko.wikipedia.org/wiki/%EB%8B%A4%EC%A4%91_%EC%83%81%EC%86%8D)   
[자바의 다이아몬드 문제와 인터페이스를 통한 해결 방법](https://f-lab.kr/insight/solving-java-diamond-problem)   
[javascript-ifno - 믹스인](https://ko.javascript.info/mixins)   
[제이크서 위키 블로그- 타입스크립트 타입, 인터페이스 상속과 오버라이딩](https://jake-seo-dev.tistory.com/697)   
[면접 때 잘못 말한 기술 질문 답변들 - 이제부터는 제대로 답변하자](https://www.linkedin.com/posts/%EB%AC%B8%EB%8C%80%EC%8A%B9_%EB%A9%B4%EC%A0%91-%EB%95%8C-%EC%9E%98%EB%AA%BB-%EB%A7%90%ED%95%9C-%EA%B8%B0%EC%88%A0-%EC%A7%88%EB%AC%B8-%EB%8B%B5%EB%B3%80%EB%93%A4-%EC%9D%B4%EC%A0%9C%EB%B6%80%ED%84%B0%EB%8A%94-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%8B%B5%EB%B3%80%ED%95%98%EC%9E%90-activity-7203154039369269249-KOG_?utm_source=share&utm_medium=member_desktop)

