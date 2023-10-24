---
published: true
title: '2023-10-24-패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프-javascript 배열 메소드 복습'
categories:
  - 패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프
tags:
toc: true
toc_sticky: true
toc_label: '패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프'
---

### 시작하며...

오늘은 react-note-app을 만들며 자주 사용했던 배열 메소드인 `find`와 `filter` 메소드에 대한 복습을 해보려고 한다. 앱을 만들면서 다양한 배열 데이터를 다뤘는데 아직 어디서 어떤 메소드를 써야 효율적인지 감이 잡히지 않아 다시 한 번 짚고 넘어가려고 한다.

### `find` 메소드

- `find`메소드는 배열에서 주어진 조건에 맞는 첫 번째 요소를 찾아 반환하는 메소드이다.
- `find`는 조건을 만족하는 **첫 번째 요소를 찾으면 검색을 중단**하며 배열을 끝까지 순회하지 않는다.
- 만약 조건에 맞는 요소가 없다면 `undefined`를 반환한다.
- 예시

```javascript
const numbers = [10, 20, 30, 40, 50];
const result = numbers.find((number) => number > 25);
console.log(result); // 30
```

```javascript
setMainNotes: (state, { payload }) => {
  // 해당 노트 수정
  if(state.mainNotes.find(({ id }) => id === payload.id)) {
    // 같은 것이 있으면 변경
    state.mainNotes = state.mainNotes.map((note) => note.id === payload.id ? payload : note);
  }
  // 노트를 새롭게 생성
  else {
    state.mainNotes.push(payload);
  }
},
```

### `filter` 메소드

- `filter`메소드는 배열에서 주어진 **조건을 만족하는 모든 요소**를 찾아서 새로운 배열로 반환하는 메소드이다.
- `filter`는 조건을 만족하는 모든 요소를 찾아 배열로 반환하므로, 여러 개의 요소를 포함하는 배열을 반환할 수 있다.
- 조건에 맞는 요소가 없으면 빈 배열을 반환한다.
- 예시

```javascript
const numbers = [10, 20, 30, 40, 50];
const result = numbers.filter((number) => number > 25);
console.log(result); // [30, 40, 50]
```

```javascript
deleteTags: (state, { payload }) => {
  state.tagsList = state.tagsList.filter(({ id }) => id !== payload); // filter 메소드
  toast.info('태그가 삭제되었습니다.');
};
```

### 전개연산자를 사용하여 객체의 속성 중 한 속성만 바꾸는 방법

react-note-app을 만들면서 `payload`에 담긴 `isPinned`라는 속성을 바꿔줘야할 상황이 발생했다. 이때, 아래와 같이 전개연산자를 활용하면 쉽게 특정 속성만 바꿀 수 있는 방법이 있었다.

```javascript
setArchiveNotes: (state, { payload }) => {
    state.mainNotes = state.mainNotes.filter(({id}) => id !== payload.id);
    state.archiveNotes.push({...payload, isPinned: false}); // 여기 있는 속성 중 하나를 바꿔줘야하기 때문에 전개연산자 사용
},
```

### 마무리

오늘은 javascript에서 배열을 다룰 때 유용한 `find`와 `filter`메소드와, 전개 연산자를 활용한 객체 속성 변경 방법을 알아보았다. 자주 사용하는 메소드들이지만 자꾸 헷갈려서 정리를 해보았다. 정리를 하니 확실히 이해가 되는 느낌이라서 좋았다. 😊
