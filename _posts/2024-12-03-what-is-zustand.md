---
title: "zustand 뜯어보기"
date: 2024-12-03 20:00:00 +0900
categories: [React]
tags: [React, zustand를]
---

이번 토이 프로젝트를 진행하며 zustand를 처음 사용해보고 있는 중이다.

zustand를 컴포넌트에서 사용하기 위해서는 action과 state를 불러와야하는데, state를 전체로 반환하면 사용하지 않는 상태에 의해서도 렌더링이 발생된다.

```tsx
//bad case
const state = useCountStore();
```

따라서 일반적으로 권장되는 방법은

```tsx
const count = useCountStore((state) => state.count);
const increase = useCountStore((state) => state.increase);
```

으로 액션과 state를 구분해서 작성하는 방식이었다.

그러나, 사용해야 하는 state값이 여러개거나 actions이 여러개라면 useCountStore을 호출하는 코드가 길어질 수 있고 action과 state가 한눈에 파악되지 않는다.

그래서 객체의 구조 분해할당을 활용하여 다음과 같이 시도하였다.

```tsx
const { state, action } = useCountStore((state) => {
  state, action;
});
```

그리고 나온 결과가, 바로 이글을 쓰게 된 계기였다. 화면은 max call stack size exceeded 에러를 뱉으며 죽어버렸다.

처음에는 원인을 파악하지 못하고 어디서 호출을 잘못하나를 한참 찾아보다가, gpt의 도움으로 useStore에 지정된 콜백 ‘state => { state, action }’ 는 항상 새로운 객체를 반환하며 이로 인해 불필요한 렌더링을 유발함을 파악할 수 있었다. (결론을 조금 더하자면 `useSyncExternalStore` 내부에서 [`Object.is`](http://Object.is) 비교가 수행되며 다른 state로 인식되기 때문이다.)

그래서,

도대체 `useStore`와 `zustand`는 어떤 방식으로 이루어져 있으며 어떤 기준으로 화면을 렌더링하는가 라는 의문이 생겨 코드를 구경갔다가 생각보다 단순한 구조에 놀라게 되었다.

특히, 단순 hook 호출과 관련된 `zustand`의 코드는 매우 간단하였다. 외부 의존성도 보이지 않아 쉽게 코드와 구조를 파악할 수 있었다.

우선 우리가 `zustand`에서 스토어를 생성할 때 사용하는 `create` 함수부터 찾아보았다.

```tsx
//zustand/react.js

//...
const createImpl = (createState) => {
  const api = vanilla.createStore(createState);
  const useBoundStore = (selector) => useStore(api, selector);
  Object.assign(useBoundStore, api);
  return useBoundStore;
};
const create = (createState) =>
  createState ? createImpl(createState) : createImpl;
exports.create = create;
exports.useStore = useStore;
```

`create`함수는 `createState`여부에 따라 `createImpl`를 호출하거나 `createImpl` 함수객체를 반환한다. 여기서 `createState`는 우리가 store를 `create`시 전달하는 콜백함수가 되겠다. (왜 이런식으로 구현했는가를 찾아보니 유연성을 위해 다음과 같은 구조를 사용했다고 한다.)

다시 `createImpl`를 따라가보면, `createImpl`는 `createState`를 매개변수로 받아 `vanilla.createStore`를 호출한다.

```tsx
//zustand/vanilla.js
const createStoreImpl = (createState) => {
  let state;
  const listeners = /* @__PURE__ */ new Set();
  const setState = (partial, replace) => {
    const nextState = typeof partial === "function" ? partial(state) : partial; //setState가 함수면 호출
    if (!Object.is(nextState, state)) {
      const previousState = state;
      state = (
        replace != null
          ? replace
          : typeof nextState !== "object" || nextState === null
      )
        ? nextState
        : Object.assign({}, state, nextState);
      listeners.forEach((listener) => listener(state, previousState)); //linstenr에서 변경된 값을 알림
    }
  };
  const getState = () => state; //현재 값 반환
  const getInitialState = () => initialState;
  const subscribe = (listener) => {
    listeners.add(listener);
    return () => listeners.delete(listener);
  };
  const api = { setState, getState, getInitialState, subscribe };
  const initialState = (state = createState(setState, getState, api));
  return api;
};
const createStore = (createState) =>
  createState ? createStoreImpl(createState) : createStoreImpl;

exports.createStore = createStore;
```

`vanilla.createStore`도 매개변수 여부에 따라 `createStoreImpl`호출하거나 함수객체를 반환한다. `createStoreImpl`를 따라가면 익숙한 변수들을 찾을 수 있다. 바로 `state`,`setState`,`getState`이다. 이제 `createStoreImpl`가 `state`를 처음 생성하고 `setState`, `getState`를 선언 후 반환하는 함수라는것을 알 수 있다. `setState`, `getState`는 `subscribe`와 함께 객체로 묶여 반환된다.

이제 다시 zustand/react.js로 돌아와 `createImpl`의 다음 코드를 확인해보자.

```tsx
//zustand/react.js

//...
const createImpl = (createState) => {
  const api = vanilla.createStore(createState);
  const useBoundStore = (selector) => useStore(api, selector);
  Object.assign(useBoundStore, api);
  return useBoundStore;
};
const create = (createState) =>
  createState ? createImpl(createState) : createImpl;
exports.create = create;
exports.useStore = useStore;
```

`createImpl`는 `useBoundStore`라는 함수를 반환하는데, `useBoundStore`는 `useStore`이라는 함수를 호출하며 방금 생성한 api 객체와 인수로 받은 selector를 반환한다.

결국 `create`의 최종 값이 `useBoundStore`라는 것은 우리가 `cretae`함수를 호출하여 생성 후 컴포넌트에서 사용하는 바로 그 훅임을 추측할 수 있다.

또한, 결국 `useBoundStore`이 호출될 때 전달되는 selector는 앞서 보았던,

```tsx
const count = useCountStore((state) => state.count);
```

즉 `state => state.count` 라는 콜백함수 임을 알 수 있다.

이제 useStore의 정체만 파악하면 된다.

```tsx
function useStore(api, selector = identity) {
  const slice = React.useSyncExternalStore(
    api.subscribe,
    () => selector(api.getState()),
    () => selector(api.getInitialState())
  );
  React.useDebugValue(slice);
  return slice;
}
```

useStore는 내부에서 `useSyncExternalStore`를 사용하고 있다.

잠깐 `useSyncExternalStore`에 대해 정리하자면

- `useSyncExternalStore`는 React 18에서 도입된 Hook으로, 외부 스토어의 상태를 React 상태와 동기화하는 데 사용된다. React의 상태 관리 흐름과 외부 스토어의 상태 변경 알림을 연결하며, SSR(Server-Side Rendering) 시에도 동일한 상태 스냅샷을 제공하기 위해 설계되었다.
- [`useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`](https://ko.react.dev/reference/react/useSyncExternalStore#usesyncexternalstore) 형태로 사용되며 `subscribe` 함수는 store를 구독 및 구독 취소하는 함수를 반환하고 `getSnapshot` 함수는 store에서 데이터의 스냅샷을 읽어온다. `getSnapshot` 함수가 반환하는 값이 달라지면 (Object.is비교) React는 컴포넌트를 리렌더링한다.

`useSyncExternalStore`를 찾아보았을 때, `useSyncExternalStore` 를 완벽하게 이해하지는 못하였으나 비로소 문제점이 어디인지 파악할 수 있었다. selector에 의해 호출되는 콜백함수가 계속해서 새로운 객체를 반환하며 `useSyncExternalStore`에 의해 리렌더링이 발생하는 것이었다.

이제 문제점을 파악하였으니 해결책을 찾아볼 차례이다.

뒤늦게 알았지만 `zustand`는 이러한 중첩된 객체를 비교하기 위한 shallow 함수를 제공한다. `shallow`함수는 얕은 비교를 수행한다. 중첩된 객체나 깊이 중첩된 속성은 확인하지 않고 , 속성의 참조만 비교한다.

`shallow` 함수대신 `useShallow` hook을 제공하여 이것을 사용하면된다.

```tsx
const { count, increase } = useCountStore(
  useShallow((state) => ({ count: state.count, increase: state.increase }))
);
```

사실 코드를 따라가며 완벽히 이해하지는 못하였지만, zustand의 동작원리를 파악할 수 있었다.
또한 라이브러리 구조를 뜯어보고 파악하는데 두려움을 조금 없앨 수 있는 시간이었다.
