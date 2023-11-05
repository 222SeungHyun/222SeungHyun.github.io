---
published: true
title: '2023-10-04-Front-end-useRef와 커스텀 훅 사용기'
categories:
  - Front-end
tags:
toc: true
toc_sticky: true
toc_label: 'Front-end'
---

### `useRef`를 사용하게 된 계기

오늘 실시간 강의를 들으면서 disney plus app을 구현하다가 `useRef`를 사용하여 모달창 외의 영역을 클릭 시에 모달이 닫히는 기능을 만들어보게 되었다.

### `useRef`란?

`useRef`는 React에서 DOM 요소에 접근하거나 다른 React 요소를 참조하는 데 사용되는 Hook이다. 이 Hook을 사용하면 함수 컴포넌트 내에서 DOM 요소나 다른 컴포넌트 인스턴스에 접근하고 제어할 수 있다. `useRef`는 컴포넌트의 렌더링과 관련 없이 일관된 참조를 유지하며 업데이트 시에는 컴포넌트를 다시 렌더링하지 않는다.

### `useRef`의 특징

**1. 초기화**  
`useRef`로 생성된 객체는 `.current` 프로퍼티를 통해 현재 값에 접근한다. 예를 들어,

```javascript
const myRef = useRef(initialValue);
```

로 초기화 한 경우 `myRef.current`에 `initialValue`가 저장된다.
<br />
<br />
**2. 렌더링과 무관**  
`useRef`로 생성한 참조는 컴포넌트의 렌더링과 관련이 없으므로 값이 변경되어도 컴포넌트가 다시 렌더링되지 않는다.
<br />
<br />
**3. 주로 DOM 조작에 사용**  
`useRef`는 주로 DOM 요소에 접근하고 조작하기 위해 사용된다. 예를 들어, 어떤 DOM 요소의 참조를 저장하고 그 요소의 크기나 위치를 변경할 수 있다.
<br />
<br />
**4. 컴포넌트 간 데이터 공유**  
`useRef`를 사용하여 컴포넌트 간 데이터를 공유할 수 있다. 컴포넌트 A에서 `useRef`로 생성한 객체를 만들고, 이를 컴포넌트 B로 전달하여 데이터를 공유할 수 있다.

### `useRef`를 사용한 예제

```javascript
import React, { useRef, useEffect } from 'react';

function MyComponent() {
  // useRef를 사용하여 DOM 요소에 접근
  const myInputRef = useRef(null);

  // 컴포넌트가 마운트될 때 input 요소에 포커스를 줌
  useEffect(() => {
    myInputRef.current.focus();
  }, []);

  return <input ref={myInputRef} />;
}
```

### 이전에 사용했던 모달 구현 방법

이전에 모달을 구현했을 때는 `setIsModalOpen`이라는 state를 설정하고, 다음과 같이 모달을 여닫는 함수들을 만들어주었다.

```javascript
const openModal = (e: React.MouseEvent) => {
  e.preventDefault();

  ...

  setIsModalOpen(true);
};

const closeModal = () => {
  setIsModalOpen(false);
};
```

`openModal`함수는 아래와 같이 버튼을 클릭시에 호출되게 해주었고,

```javascript
...

<S.WikiButton
  type="button"
  onClick={openModal}
>

...
```

`setIsModalOpen` state와 `closeModal` 함수는 다음과 같이 Modal 컴포넌트의 Props로 내려주었다.

```javascript
<Modal
  isOpen={isModalOpen}
  onClose={closeModal}
  ...
/>
```

Props로 받은 `isOpen`은 다음과 같이 true값을 가질 때만 Modal 컴포넌트가 열리도록 설정하는 데에 사용하였다.

```javascript
if (!isOpen) return null;

return (
...
```

한편, `onClose`는 Props로 받은 후, 다음과 같이 `modalClose` 함수를 통해 호출될 수 있게 하였다.

```javascript
const modalClose = () => {
  onClose();
};
```

마지막으로, `modalClose` 함수는 다음과 같이 Exit 버튼을 누르거나, Modal Content 부분을 제외한 Wrapper 부분을 클릭하면 호출되도록 하였다.

```javascript
<ExitButton
  type="button"
  onClick={modalClose}
>
```

```javascript
<ModalWrapper
      onClick={() => {
        modalClose();
      }}
    >
      <ModalContent
      ...
```

### 이번에 사용한 모달 구현 방법

이번에는 위에서 설명했던 `useRef`와 커스텀 훅을 이용하여 모달을 구현해보았다.  
먼저, 다음과 같이 `ref`변수를 생성하고 초기값을 `null`로 설정해 주었다.

```javascript
const ref = useRef(null);
```

이후, 모달 요소에 `ref`를 설정해주었다.

```javascript
<div className="modal" ref={ref}>
```

이렇게 하면 `ref`객체는 컴포넌트 내부에서 DOM 요소에 대한 참조를 생성하고, 이 참조를 사용해서 해당 DOM 요소에 접근할 수 있다.
<br />
<br />
이후, 아래와 같이 `useOnClickOutside`라는 커스텀 훅을 사용하여 모달 외부를 클릭했을 때 모달을 닫는 동작을 구현하였다.

```javascript
useOnClickOutside(ref, () => {
  setIsModalOpen(false);
});
```

해당 훅은 `ref`객체와 콜백 함수를 인자로 받으며, 모달 외부를 클릭하면 모달을 닫기 위해 `setIsModalOpen(false)`를 호출한다.

### 커스텀 훅 (useOnClickOutside)

`useOnClickOutside` 훅의 내용은 아래와 같이 구현하였다.

```javascript
import { useEffect } from 'react';

export default function useOnClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      // contains: 클릭한 요소가 ref.current 안에 포함되어 있는지 확인
      // 내부 클릭 시
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      // 외부 클릭 시
      else {
        handler();
      }
    };

    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);

    // unmoount 시에 listener 제거
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}
```

해당 훅에서는 `useEffect`를 사용하여 이벤트 리스너를 등록하고, 이벤트 리스너가 호출될 때 `ref`객체를 사용하여 클릭한 요소가 특정 요소 내부에 속하는지를 확인한다.  
이렇게 해서 외부 클릭(터치)를 감지하여 특정 DOM 요소를 클릭(터치)한 경우에는 아무 작업도 하지 않고, 외부 요소를 클릭한 경우에는 `handler` 함수를 호출하여 모달을 닫도록 구현하였다.

### 마무리

이번 강의에서는 `useRef`와 커스텀 훅을 사용하여 모달을 열고닫는 기능을 구현해보았다.  
그동안에는 state와 Props를 활용한 기존 방법을 주로 사용하였지만, 이번에 사용한 방법을 사용하면 모달과 간련된 로직을 별도의 훅으로 분리하여 관리함으로써 컴포넌트의 코드가 간결해지고, 외부 클릭을 처리하기 용이하며, 모달의 상태가 변경되어도 컴포넌트가 리렌더링되지 않으므로 성능적으로도 유리할 것 같다.  
거의 모든 프로젝트에서 모달을 사용하는만큼, 앞으로 이 방법을 사용하여 모달을 구현해보면서 손에 익혀야할 것 같다.
