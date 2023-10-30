---
published: true
title: '2023-10-30-패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프-만들어보며 이해하는 React & Redux-외부 변수에 의한 함수 호출 방법과 currying, closure 복습'
categories:
  - 패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프
tags:
toc: true
toc_sticky: true
toc_label: '패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프'
---

### 시작하며...

최근 새로 입과된 강의 중에 김민태 강사님의 "만들어보며 이해하는 React & Redux" 라는 강의를 그룹스터디원에게 추천받아 들어보게 되었다. 김민태 강사님이 그렇게 명강사라고 들었는데 첫 강의를 듣자 마자 왜 그렇게 좋다고 추천하는지 느꼈다. 동시에 내가 놓치고 있던 많은 부분에 대해서도 느껴 강의를 들으면서 부족하다고 느꼈던 부분을 다시 한번 되짚어보고자 한다.

### `createElement` 함수

```javascript
function createElement(type, props) {
  switch (type) {
    case 'h1':
      // DOM 노드 생성
      return [document.createElement('h1')].map((element) => {
        // 속성 추가 및 내용 설정
        Object.entries({ ...props, 'data-id': 'subject' }).forEach(
          // props 객체 주입
          ([name, value]) => element.setAttribute(name, value)
        );
        // element 반환
        return element;
      })[0];
    case 'div':
      return [document.createElement('div')].map((element) => {
        Object.entries({ ...props, 'data-id': 'layout' }).forEach(
          ([name, value]) => element.setAttribute(name, value)
        );
        return element;
      })[0];
  }
}
// document.querySelector('#root').appendChild(createElement('h1', {})); 로 태그 추가
```

위와 같이 element를 생성하는 간단한 소프트웨어가 있다고 가정해보자. 강사님께서는 소프트웨어는 끊임없이 변화할 수 있다고 하셨다. 위 소프트웨어에서 일어날 수 있는 변화에 대해서 살펴보자.

1. `createElement`의 case 추가
2. case `h1`의 로직 변경

위 2가지정도의 변경사항이 있을 것이다. 이 중에서 `h1`의 로직 변경은 `div` case와는 상관 없는 변경이다. 하지만, 이 변경으로 테스트 시 `createElement` 함수를 모두 테스트해야한다. 따라서 위 소프트웨어는 변경에 용이하지 않은 구조이다. 그럼 아래와 같이 소프트웨어를 리팩토링해보자.

```javascript
function createH1(props) {
  // DOM 노드 생성
  return [document.createElement('h1')].map((element) => {
    // 속성 추가 및 내용 설정
    Object.entries({ ...props, 'data-id': 'subject' }).forEach(
      // props 객체 주입
      ([name, value]) => element.setAttribute(name, value)
    );
    // element 반환
    return element;
  })[0];
}

function createDiv(props) {
  return [document.createElement('div')].map((element) => {
    Object.entries({ ...props, 'data-id': 'layout' }).forEach(([name, value]) =>
      element.setAttribute(name, value)
    );
    return element;
  })[0];
}

function createElement(type, props) {
  switch (type) {
    case 'h1':
      return createH1(props);
    case 'div':
      return createDiv(props);
  }
}
// document.querySelector('#root').appendChild(createElement('h1', {})); 로 태그 추가
```

위와 같이 리팩토링을 진행했을 때, 이전 소프트웨어와의 차이점은 다음과 같을 것이다.

1. `h1`의 로직이 변경될 때, `createH1` 함수 범위 내에서만 변경이 일어남
2. `createH1`만 온전히 테스트하면 문제없이 소프트웨어 동작

하지만 이 소프트웨어도 문제가 있다. `createElement`에 새로운 case가 추가되면 `h1`과 아무 상관 없음에도 `createElement` 전체를 테스트해보아야 한다. 결국 변경 범위와 테스트 범위가 달라지는 문제가 반복되는 것이다.

이 취약점을 보완하기 위해 소프트웨어를 다시 한 번 리팩토링 해보겠다.

```javascript
function createH1(props) {
  // DOM 노드 생성
  return [document.createElement('h1')].map((element) => {
    // 속성 추가 및 내용 설정
    Object.entries({ ...props, 'data-id': 'subject' }).forEach(
      // props 객체 주입
      ([name, value]) => element.setAttribute(name, value)
    );
    // element 반환
    return element;
  })[0];
}

function createDiv(props) {
  return [document.createElement('div')].map((element) => {
    Object.entries({ ...props, 'data-id': 'layout' }).forEach(([name, value]) =>
      element.setAttribute(name, value)
    );
    return element;
  })[0];
}

const creatorMap = {
  h1: createH1,
  div: createDiv,
};

function createElement(type, props) {
  return creatorMap[type](props);
}
// document.querySelector('#root').appendChild(createElement('h1', {})); 로 태그 추가
```

위와 같이 리팩토링을 진행했을 때, 이전 소프트웨어와의 차이점은 `createElement`가 구조적으로 안정적으로 변한다. 즉, 새로운 type이 추가되어도 `createElement`의 변경은 없다.  
하지만, 위 소프트웨어는 `createElement`가 `creatorMap`이라는 외부 변수에 의존한다. 이는, 외부 변수가 바뀌었을 때 `createElement`의 동작이 잘못될 수 있다.

#### 외부 변수에 의한 함수 호출

위 소프트웨어에서 `creatorMap` 객체는 type(`h1`이나 `div`)과 해당 요소를 만드는 함수(`createH1` 및 `createDiv`)간의 매핑으로 작동한다.

`createElement` 함수는 `type`을 제공된 `type`에 따라 `creatorMap`에서 적절한 생성 함수를 찾는다. 이후, 해당 함수를 `props`와 함께 호출하고 생성된 요소를 반환한다.

#### `creatorMap`에서 함수를 호출하지 않는 이유

결론부터 말하자면, `creatorMap` 내에서 다음과 같이 함수를 호출하는 것은 부적합하다.

```javascript
const creatorMap = {
  h1: createH1(),
  div: createDiv(),
};
```

그 이유는, 함수를 호출한 결과를 `creatorMap`에 저장하는 것이 아니라, 함수 자체를 매핑해야되기 때문이다. `creatorMap` 에서 위와 같이 함수를 호출하는 경우, 해당 함수에는 `props`가 전달되지 않을 것이다.

#### 요소의 동적인 생성 방법

###

추가로,

```javascript
return creatorMap[type](props);
```

위와 같이 `type`에 해당하는 함수를 호출하고, 해당 함수 내에서 `props`를 사용하여 요소를 생성한 다음, 생성된 요소를 `createElement`함수의 반환값으로 반환함으로써 원하는 요소 유형과 해당 속성을 가진 요소를 동적으로 생성하는 방법에 대해서 잘 기억해놔야할 것 같다.

### `createElement` 함수의 최종 형태

위에서 리팩토링했던 `createElement`의 단점인 `creatorMap`이라는 외부 변수에 의존한다는 것, 이는 외부 변수가 바뀌었을 때 `createElement`의 동작이 잘못될 수 있다는 점을 해결하기 위해 다시 한 번 `createElement`함수를 리팩토링해보겠다.

```javascript
function createH1(props) {
  // DOM 노드 생성
  return [document.createElement('h1')].map((element) => {
    // 속성 추가 및 내용 설정
    Object.entries({ ...props, 'data-id': 'subject' }).forEach(
      // props 객체 주입
      ([name, value]) => element.setAttribute(name, value)
    );
    // element 반환
    return element;
  })[0];
}

function createDiv(props) {
  return [document.createElement('div')].map((element) => {
    Object.entries({ ...props, 'data-id': 'layout' }).forEach(([name, value]) =>
      element.setAttribute(name, value)
    );
    return element;
  })[0];
}

const creatorMap = {
  h1: createH1,
  div: createDiv,
};

// currying
const coupler = (map) => (type, props) => map[type](props);
const createElement = coupler(creatorMap);

// document.querySelector('#root').appendChild(createElement('h1', {})); 로 태그 추가
```

위와 같이 소프트웨어를 변경했을 때, `coupler`는 `creatorMap`을 `map`으로 받고, 이를 closure에 담아 `createElement`를 생성한다. 이와 같이 currying을 사용하면 방식은 코드의 모듈화와 확장성을 높일 수 있으며, `createElement`` 함수의 일관성을 유지하는 데 도움이 된다.

#### **Currying**

```javascript
const coupler = (map) => (type, props) => map[type](props);
const createElement = coupler(creatorMap);
```

위 코드에서 사용된 **Currying**은 함수를 여러 개의 단계로 분해하는 기법이다. 각 단계는 함수의 일부 인자를 받고, 그 결과로 새로운 함수를 반환한다. **Currying** 사용하면 함수의 재사용성을 높일 수 있으며, 함수의 인자를 부분적으로 채워서 쉽게 다른 함수로 전달할 수 있다.

위 코드에서 사용된 **Currying**은 `coupler`라는 함수를 정의하고, 이 함수는 `creatorMap`과 함께 사용된다. 위 코드는 두 단계의 **Currying**을 사용하여 `createElement`함수를 생성한다.

**1. 첫 번째 단계**

- `coupler`함수는 `map`이라는 단일 인자를 받는다.
- 이 함수는 두 번째 인자로 `type`과 `props`를 받는 함수를 반환한다.

**2. 두 번째 단계**

- `createElement` 함수는 `creatorMap`을 `map`인자로 받는 `coupler`함수의 결과이다.
- `createElement`는 `type`과 `props`를 인자로 받으며, 내부에서 `map[type](props)`를 호출하여 요소를 생성한다.

```javascript
(type, props) => map[type](props); // createElement 함수
```

#### **Closure**

```javascript
const coupler = (map) => (type, props) => map[type](props);
```

위 코드에서 사용된 **Closure**는 함수 내부에서 정의된 함수가 해당 함수의 외부 스코프(lexical scope)에 접근할 수 있는 현상을 의미한다.

위 코드에서는 외부 함수(`coupler`)의 변수를 내부 함수(`(type, props) => map[type](props)`)에서 사용할 수 있다. 따라서, `map` 변수는 외부 스코프에 있지만 내부 함수에서 사용되는데, 이것이 **Closure**이다.

### 마무리

오늘은 김민태 강사님의 첫 강의 영상을 보고 강의 내용과 강의에서 다뤘던 개념 중 부족했던 개념들을 정리해보았다. 아직 강의 한 개밖에 안봤는데 모르는 것이 이렇게 많을 줄 몰랐다. 확실히 그간 기본 개념들에 대한 이해와 응용이 많이 부족했던 것 같다. 앞으로 얼마나 모르는 것이 많이 나올지 몰라 무섭지만 다 실력 향상을 위한 길이라고 생각하고 열심히 해야겠다. 😂
