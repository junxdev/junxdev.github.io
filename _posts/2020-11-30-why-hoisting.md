---
title: "왜 Hoisting 같은 것을 만든 것이죠"
excerpt: "브랜던 아이크도 '급하게 만들다 보니'라는 말을 한다!"
tags: [JavaScript, Hoisting, Interpreter Language]
---
* toc
{:toc}

## 1. Hoisting이란

* Hoisting: 끌어올림, 특정 스코프에서 선언되거나 초기화된 `var` 변수를 실행 시점 이전에 `실행 상황 또는 유효 범위;Execution Context`의 최상단에서 선언하는 것
* 하지만 특정 값이나 객체를 대입하거나 참조하는 초기화는 실행 시점에 진행됨
* 따라서 아래와 같은 경우 `참조 에러;ReferenceError`가 발생하는 대신 Undefined가 출력됨

``` javascript
function lie() {
    console.log(message);
    var message = "It's fun to learn JavaScript!";
}

lie();  // undefined
```

* 중요한 점: 오직 `var`로 선언한 변수만 Hoisting(Only variables defined with `var` are hoisted!)

## 2. 원리

* continue...

## 3. 왜 만들었을까?

* 내가 아는 선에서 Java에는 비슷한 것이 없음
* 궁금해서 찾아봄
* 다음은 JavaScript를 만든 `브랜던 아이크;Brendan Eich`가 위와 같은 질문에 [트위터에 남긴 답변](https://twitter.com/BrendanEich/status/522394590301933568){:target="_blank"}

> 95년에 하도 급하게 만들다 함수 끌어올림이랑 블록 범위 취소를 개발했더니 변수가 끌어올려졌네? 
> > *var hoisting was thus unintended consequence of function hoisting, no block scope, JS as a 1995 rush job.*
> ES6에 'let'이 있으니까 그거 써
> > *ES6 'let' may help.*

* JavaScript는 95년 12월에 출시... 벼락치기의 유산... var hoisting
* `블록 범위 취소;no block scope`는 아마 [var로 선언한 변수는 블록 범위를 가지지 않는 것](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/block){:target="_blank"}을 의미?

## 4. 왜 이런 식으로 설계했을까?

* 브라우저에서 사용하는 언어 특성? continue...