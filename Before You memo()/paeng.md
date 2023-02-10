# [번역] memo() 하기 전에

원문: https://overreacted.io/before-you-memo/

React 성능 최적화에 대한 글이 많이 있다. 일반적으로, state 업데이트가 느리면, 다음과 같은 것을 해야합니다:

1. production 빌드가 동작 중인지 확인합니다. (Development 빌드들은 극단적인 상황일때 엄청나게 의도적으로 느려집니다.)

2. 필요 이상으로 state를 tree에서 높은 부분에 놓여있는지 확인해야 합니다. (예를 들어, input state를 중앙 store에 놓는 것은 최고의 방법이 아닐 수 있습니다.)

3. React DevTools Profiler을 실행하여 re-rendered을 확인하고 가장 expensive subtrees를 `memo()`로 감쌉니다. (그리고 필요한 곳에 `useMemo()`를 추가합니다.)

특히, 마지막 방법은 components 사이에 있는 경우 성가시더라도 미래에는 이성적으로 compiler가 너를 위해 해줄 지도 모릅니다.

이 글에서는, 2가지 다른 기술을 공유하려고 합니다. 이것들은 놀랄만큼 기초적인 것이여서, 사람들이 이것들로 인해 rendering 성능이 향상된다는 것을 잘 알지 못합니다.

이 기술들은 당신이 이미 알고 있는 것들을 보안해줍니다! 이것들이 `memo` 또는 `useMemo`를 대처하지는 못하지만 먼저 시도해 보면 좋습니다.

## 인위적으로 느린 Component

다음은 심각한 렌더링 성능 문제가 있는 컴포넌트 입니다.

```js
import { useState } from 'react';

export default function App() {
  let [color, setColor] = useState('red');
  return (
    <div>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
      <ExpensiveTree />
    </div>
  );
}

function ExpensiveTree() {
  let now = performance.now();
  while (performance.now() - now < 100) {
    // 인위적인 지연 -- 100ms 동안 아무것도 하지 않음
  }
  return <p>저는 아주 느린 컴포넌트 트리입니다.</p>;
}
```

([Try it here](https://codesandbox.io/s/frosty-glade-m33km?file=/src/App.js:23-513))

문제는 `App` 내부에서 `color`를 변경할 때마다 `<ExpensiveTree />`를 인위적으로 딜레이시켜 리-렌더하면 매우 느려질 것이다.

[`memo()`](https://codesandbox.io/s/amazing-shtern-61tu4?file=/src/App.js)를 써서 마무할 수 있지만, 이에 관해서 많은 Article이 많으니 시간을 아끼겠습니다. 2가지 해결책을 소개하겠습니다.

### Solution 1: State를 아래로 내리기

리랜더링 코드를 자세히 보면 반환된 tree의 특정 부분만 `color`와 관련이 있습니다. (관련된 부분 ❗️로 표시)

```js
export default function App() {
  let [color, setColor] = useState('red'); ❗️
  return (
    <div>
      <input value={color} onChange={(e) => setColor(e.target.value)} /> ❗️
      <p style={{ color }}>Hello, world!</p> ❗️
      <ExpensiveTree />
    </div>
  );
}
```

그래서 해당 부분을 `Form` component로 빼고 state를 아래로 내려보겠습니다.: (관련된 부분 ❗️로 표시)

```js
export default function App() {
  return (
    <>
      <Form /> ❗️
      <ExpensiveTree />
    </>
  );
}

function Form() {
  let [color, setColor] = useState('red'); ❗️
  return (
    <>
      <input value={color} onChange={(e) => setColor(e.target.value)} /> ❗️
      <p style={{ color }}>Hello, world!</p> ❗️
    </>
  );
}
```

([Try it here](https://codesandbox.io/s/billowing-wood-1tq2u?file=/src/App.js:64-380))

`color`가 변하면, 오직 `Form`만 리-랜더 됩니다. 문제 해결.

### Solution 2: Content 끌어올리기
