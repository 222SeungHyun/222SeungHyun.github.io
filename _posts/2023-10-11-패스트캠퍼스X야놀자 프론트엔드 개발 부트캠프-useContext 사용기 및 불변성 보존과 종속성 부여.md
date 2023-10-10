---
published: true
title: '2023-10-11-패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프-useContext 사용기 및 불변성 보존과 종속성 부여'
categories:
  - 패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프
tags:
toc: true
toc_sticky: true
toc_label: '패스트캠퍼스X야놀자 프론트엔드 개발 부트캠프'
---

### `useContext`란?

`useContext`는 React에서 상태 관리를 간편하게 하기 위한 Hook 중 하나이다. 이 Hook을 사용하면 React 컴포넌트 간에 데이터를 전달하고 공유할 수 있다. 주로 전역 상태 관리나 테마 설정과 같이 컴포넌트 트리를 횡단하는 데이터를 처리하는 데 사용된다.
<br />
<br />

### `useContext`를 사용하는 주요 단계

**1. Context 객체 생성**

```javascript
export const OrderContext = createContext();
```

**2. Context 제공자(Component) 생성**

- 데이터를 다른 컴포넌트에 제공하는 컴포넌트를 생성한다. 이 컴포넌트는 `OrderContext.Provider`컴포넌트로 감싸야 한다.
- `value` prop을 사용하여 공유할 데이터를 설정한다.

```javascript
return (
  <OrderContext.Provider value={value}>{props.children}</OrderContext.Provider>
);
```

이때, 위 코드를 다음과 같이 전개 연산자를 사용하여 표현할 수도 있다.

```javascript
return <OrderContext.Provider value={value} {...props} />;
```

**3. Context 사용**

- `useContext` Hook을 사용하여 컴포넌트 내에서 공유 데이터에 접근한다.
- 이 Hook은 `OrderContext.Provider`에서 제공한 `value`를 반환한다.

```javascript
import React, { useContext } from "react";

...

const [orderData, updateItemCount] = useContext(OrderContext);

...

return <p>총 가격: {orderData.totals[orderType]}</p>
```

여기에서 배열의 형태로 반환된 `value`를 받아온 이유는 `OrderContext.Provider`에서 다음과 같은 형태로 prop을 내려주기 때문이다.

```javascript
const value = useMemo(() => {
    ...

    return [{ ...orderCounts, totals }, updateItemCount];
  }, [orderCounts, totals]);
```

**4. 컴포넌트 트리에 Context 제공자 추가**

- 앱의 루트 컴포넌트 또는 필요한 범위에서 `OrderContextProvider`를 사용하여 Context 제공자를 추가한다.

```javascript
root.render(
  <OrderContextProvider>
    <App />
  </OrderContextProvider>
);
```

### 불변성 보존(Immutable)

`OrderContext.Provider`에서 prop으로 내리는 `value`부분의 코드는 다음과 같다.

```javascript
const [orderCounts, setOrderCounts] = useState({
    products: new Map(),
    options: new Map(),
});

...

const value = useMemo(() => {
  function updateItemCount(itemName, newItemCount, optionType) {
    // 불변성을 지키기 위해 새로운 Map을 만들어서 반환
    const oldOrderMap = orderCounts[optionType];
    const newOrderMap = new Map(oldOrderMap);

    newOrderMap.set(itemName, parseInt(newItemCount));

    // orderCounts를 업데이트
    const newOrderCounts = { ...orderCounts };
    newOrderCounts[optionType] = newOrderMap;

    setOrderCounts(newOrderCounts);
  }

  return [{ ...orderCounts, totals }, updateItemCount];
}, [orderCounts, totals]);
```

위 코드의 `updateItemCount` 함수 내에서, `orderCounts`라는 상태는 `optionType`별로 각각의 객체가 존재한다. 이때, 직접 `orderCount[optionType]`에 접근하여 값을 수정하면 불변성을 유지하지 못한다. 따라서, 새로운 객체인 `newOrderMap`을 생성하여 `itemName`이라는 `key`를 가지고 있는 요소의 `value`를 `parseInt(newItemCount)` 형태로 설정해주었다.
<br />
<br />
이후, `orderCounts` 상태의 불변성 역시 유지되어야하므로, 전개 연산자를 이용하여 기존 상태를 얕은 복사(Shallow Copy) 해준 후, 새로 생성된 `newOrderCounts`내의 `optionType`에 해당하는 객체를 위에서 생성한 `newOrderMap`으로 설정하고, `setOrderCounts` 함수로 상태를 update한다.
<br />
<br />
추가로, 의존성 배열과 `useMemo`를 사용하여 `orderCounts` 및 `total`값이 변경될 때만 콜백함수를 다시 실행하도록 하여 불필요한 렌더링을 방지해주었다.

### `useEffect`를 활용한 종속성 부여

```javascript
// orderCounts가 업데이트 될 때마다 totals도 update
// -> useEffect 사용
const [totals, setTotals] = useState({
  products: 0,
  options: 0,
  total: 0,
});

useEffect(() => {
  const productsTotal = calculateSubtotal('products', orderCounts);
  const optionsTotal = calculateSubtotal('options', orderCounts);
  const total = productsTotal + optionsTotal;
  setTotals({
    products: productsTotal,
    options: optionsTotal,
    total,
  });
}, [orderCounts]);
```

해당 컴포넌트에서는 `orderCounts`가 업데이트 될 떄마다 `totals`라는 상태가 자동으로 업데이트되는 기능이 필요했다. 따라서 `useEffect`를 활용하여 의존성 배열에 `orderCounts`를 추가하여 그 값이 변할 때마다 `setTotals` 함수로 상태를 update해주었다.

### for in 문과 for of 문의 차이 및 `Array.from()`

위에서 사용한 `calculateSubtotal` 함수를 구현하다가 갑자기 궁금한 점이 생겼다.

```javascript
function calculateSubtotal(orderType, orderCounts) {
  let optionCount = 0;
  for (const count of orderCounts[orderType].values()) {
    optionCount += count;
  }

  return optionCount * pricePerItem[orderType];
}
```

`calculateSubtotal` 함수는 위와 같이 구현하였는데, `orderCounts[orderType].values()`를 순회할 때 `for of`문을 사용하였다. **"객체의 속성을 순회할 때는 보통 `for in`문을 사용하는 것으로 알고 있는데, 왜 `for of`문을 사용했을까?"** 라는 의문이 들었다.
<br />
<br />
그 답은 `values()` 함수가 무엇을 반환하는지에 있었다. `orderCounts[orderType]`는 위에서 설명하였듯이 key-value 쌍을 가지고 있는 `Map` 객체이다. 이때, `Map` 객체에서 사용할 수 있는 함수 중 하나인 `values()`는 객체 내의 모든 값들을 나타내는 **이터레이터(iterator)** 를 반환한다.
<br />
<br />
따라서, `orderCounts[orderType].values()`가 반환하는 것은 **반복 가능한(iterable)** 객체이고, 이를 순회하는 데에는 `for of`문이 사용되는 것이었다.
<br />
<br />
추가로, `SummaryPage`에서는 다음과 같은 형태로 option들의 key값을 가져와서 배열 형태로 저장해주었다.

```javascript
const optionsArray = Array.from(orderData.options.keys());
```

위에서 설명한 것처럼, `values()`나 `keys()` 함수는 배열이 아닌 **iterable** 한 객체를 반환하기 때문에 배열 형태로 저장하기 위해서는 `Array.from()` 함수를 통하여 배열로 변환해주어야 했다.

### `step`을 사용한 페이지 이동 구현

React에서 페이지의 이동을 구현하는 방법에는 다양한 방법이 존재한다. 오늘은, 일련의 과정이 존재하는 페이지간의 이동에 대하여 `step`이라는 상태에 변화를 줌으로써 페이지 이동을 구현해보았다.

```javascript
function App() {
  const [step, setStep] = useState(0);
  return (
    <div style={{ padding: '4rem' }}>
      {step === 0 && <OrderPage setStep={setStep} />}
      {step === 1 && <SummaryPage setStep={setStep} />}
      {step === 2 && <CompletePage setStep={setStep} />}
    </div>
  );
}
```

```javascript
<button onClick={() => setStep(1)}>주문</button>
```

방법은 간단하다. 우선, App 컴포넌트에서 `step` 상태를 0으로 초기화해주었다. `step`이 0(초깃값)일 때는 `OrderPage`가 렌더링되고, props로 `setStep` 함수를 내려주어 특정 버튼을 클릭하면 다음 `step`으로 상태가 바뀌고, 자동으로 해당 `step`에 해당하는 페이지가 렌더링되도록 구현하였다.

### 마무리

React에서의 상태 관리 방법 중 하나인 `useContext`를 통하여 간단한 상태관리 앱을 만들어 보았다. 처음이라 그런지 어렵다고 느껴졌지만 열심히 공부하고 실습해보며 얼른 상태 관리에 익숙해져야겠다 🙌🙌
