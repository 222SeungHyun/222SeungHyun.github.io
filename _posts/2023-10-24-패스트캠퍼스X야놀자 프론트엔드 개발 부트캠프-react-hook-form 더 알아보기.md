---
published: true
title: '2023-10-24-패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프-react-hook-form 더 알아보기'
categories:
  - 패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프
tags:
toc: true
toc_sticky: true
toc_label: '패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프'
---

### 시작하며...

저번시간에 `useForm`의 "onBlur" 모드를 살펴보았는데, `react-hook-form`을 사용하면서 그 동작 방식이 잘 이해되지 않았다. 그래서 오늘은 `react-hook-form`에서 제공하는 주요 기능 및 hook들에 대해서 알아보겠다.

### `react-hook-form`의 주요 기능 및 hook

**1. useForm({ mode: "onBlur" })**

- `useForm` 훅을 사용하여 폼 상태를 관리
- `mode` 속성은 유효성 검사 모드를 설정하며 "onBlur" 모드는 입력 필드가 포커스를 잃을 때 유효성을 검사한다.

**2. register**

- `register` 함수는 입력 필드를 등록하여 `react-hook-form`이 해당 필드의 상태를 추적하고 유효성 검사를 수행할 수 있도록 한다.
- 예를 들어,

```javascript
const userName = {
  required: "필수 필드입니다.",
};

...

{...register("name", userName)}
```

위 코드는 "name"필드를 등록하고, 해당 필드의 이름을 "name"으로 설정하며, `userName`객체는 필드의 유효성 검사 규칙을 정의한다.

**3. handleSubmit**

- `handleSubmit` 함수는 제출 이벤트를 처리하고, 제출할 때 실행할 콜백 함수를 정의한다.
- 예를 들어,

```javascript
<form onSubmit={handleSubmit(onSubmit)}>
```

위 코드와 같이 `onSubmit` 함수가 제출(submit)될 때 실행된다.

**4. FormState**

- `formState` 객체는 현재 폼 상태에 대한 정보를 포함한다. 여기서 `errors` 속성은 현재 입력 필드에서 발생한 유효성 검사 오류를 포함하고 있다.

**5. reset**

- `reset` 함수는 폼 내의 모든 입력 필드를 초기화한다. 제출 후에 입력 필드를 초기 상태로 되돌리는 데 사용된다.

### 마무리

오늘은 간략하게 `react-hook-form`에서 제공하는 주요 기능들과 hook들에 대해 알아보았다. `react-hook-form`을 사용하면 간단하게 form을 생성할 수 있는만큼, 앞으로 적극적으로 활용해보아야겠다. 😊
