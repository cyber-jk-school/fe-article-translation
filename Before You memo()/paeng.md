# [번역] memo() 하기 전에

원문: https://overreacted.io/before-you-memo/

React 성능 최적화에 대한 글이 많이 있습니다. 일반적으로, state 업데이트가 느리면, 다음과 같은 것을 해야합니다:

1. production 빌드가 동작 중인지 확인합니다. (Development 빌드들은 극단적인 상황일때 엄청나게 의도적으로 느려집니다.)

2. 필요 이상으로 state를 tree에서 높은 부분에 놓여있는지 확인해야 합니다. (예를 들어, input state를 중앙 store에 놓는 것은 최고의 방법이 아닐 수 있습니다.)

3. React DevTools Profiler을 실행하여 어떤 요소가 리렌더링을 발생시키는지 확인하고 가장 expensive subtrees를 `memo()`로 감쌉니다. (그리고 필요한 곳에 `useMemo()`를 추가합니다.)

특히, 마지막 방법은 components 사이에 있는 경우 성가시더라도 미래에는 이성적으로 compiler가 너를 위해 해줄 지도 모릅니다.

이 글에서는, 2가지 다른 기술을 공유하려고 합니다. 이것들은 놀랄만큼 기초적인 것이여서, 사람들이 이것들로 인해 렌더링 성능이 향상된다는 것을 잘 알지 못합니다.

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

문제는 `App` 내부에서 `color`를 변경할 때마다 `<ExpensiveTree />`를 인위적으로 딜레이시켜 리렌더링되면 매우 느려질 것이다.

[`memo()`](https://codesandbox.io/s/amazing-shtern-61tu4?file=/src/App.js)를 써서 마무할 수 있지만, 이에 관해서 많은 Article이 많으니 시간을 아끼겠습니다. 2가지 해결책을 소개하겠습니다.

### Solution 1: State를 아래로 내리기

렌더링 코드를 자세히 보면 반환된 tree의 특정 부분만 `color`와 관련이 있습니다. (관련된 부분 ❗️로 표시)

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

`color`가 변하면, 오직 `Form`만 리렌더링 됩니다. 문제 해결.

### Solution 2: Content 끌어올리기

state의 조각이 expensive tree 위에 있으면 위의 해결책은 작동하지 않습니다. 예를 들어, 부모 `<div>`에 `color`을 적용해보면:

```js
export default function App() {
  let [color, setColor] = useState('red'); ❗️
  return (
    <div style={{color}}> ❗️
      <input value={color} onChange={(e) => setColor(e.target.value)}/>
      <p>Hello, world!</p>
      <ExpensiveTree />
    </div>
  )
}
```

([Try it here](https://codesandbox.io/s/bold-dust-0jbg7?file=/src/App.js:58-313))

이젠 `<ExpensiveTree />` 포함 부모 `<div>`까지 포함하니 `color`를 사용되지 않는 부분을 다른 component로 추출할 수 없습니다. `memo`를 피할 수 없습니다. 맞지?

아니면 할 수 있습니까?

이 sandbox를 실행해 보고 알아낼 수 있는지 확인해 봅시다.
<br/>
...
<br/>
...
<br/>
...
<br/>
답은 의외로 간단합니다:

```js
export default function App() {
  return (
    <ColorPicker>
      <p>Hello, world!</p> ❗️
      <ExpensiveTree /> ❗️
    </ColorPicker>
  );
}

function ColorPicker({ children }) { ❗️
  let [color, setColor] = useState("red");
  return (
    <div style={{ color }}>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      {children} ❗️
    </div>
  );
}
```

`App` component를 2개로 나눴습니다. `color` state 변수를 포함해 `color`에 의존하는 부분은 `ColorPicker`로 옮겼습니다.

`color`와 상관없는 부분은 `App` component에 있고 JSX content로 알려져있는 `children` prop을 `ColorPicker`에 전달합니다.

`color`가 변할 때, `ColorPicker`은 리렌더링됩니다. 그러나
`App`은 이전과 같은 `children` prop을 가지고 있어, React는 subtree에 방문하지 않습니다.

결과적으로, `<ExpensiveTree />`은 리렌더링되지 않습니다.

## 교훈은 뭔가요?

`memo` 또는 `useMemo`같이 최적화 적용전에, 변화가 없는 부분에서 변화가 있는 부분을 나눠보는 것이 합리적일 수 있습니다.

이러한 접근에 대한 흥미로운 부분은 **성능과는 전혀 상관 없다는 것**입니다. component를 나누기 위해 `children` prop를 사용하면 application의 데이터 흐름을 쉽게 알 수 있고 tree를 통해 내려오는 prop의 수를 줄일 수 있습니다. 이와 같은 경우 성능 향상이 최종 목표가 아니었지만 최고의 방법입니다.

흥미롭게도, 이러한 pattern은 미래에 더 좋은 성능 이점을 제공합니다.

예를 들어, [Server Components](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html)가 안정화 되고 준비가 되면, `ColorPicker` component는 `children`을 [서버로부터 받아](https://www.youtube.com/watch?v=TQQPAU21ZUw&t=1314s)올 수 있습니다. 게다가 전체 `<ExpensiveTree />` component 또는 일부분이 서버에서 실행될 수 있으며, 최상위 수준의 React state 업데이트도 client에서 해당 부분을 "skip over(건너뛰다)" 것입니다.

`memo`로도 할 수 없는 일 입니다.! 그러나 두 접근들은 상호보완적입니다. state를 내리는 것을 등한시 하지 마세요(그리고 content 올리는 것도!)

그러고 나서도 충분하지 않으면, Profiler를 사용하고 memo를 끼얹시면 됩니다.

## 전에도 다음과 같은 글을 읽은 적이 있습니까?

[예, 아마도요.](https://kentcdodds.com/blog/optimize-react-re-renders)

이것은 새로운 아이디어가 아닙니다. React 합성 모델의 자연스러운 결과입니다. 제대로 인정받지 못할 만큼 간단하지만, 더 사랑 받을 자격이 있습니다.

[Discuss on Twitter](https://mobile.twitter.com/search?q=https%3A%2F%2Foverreacted.io%2Fbefore-you-memo%2F) • [Edit on GitHub](https://github.com/gaearon/overreacted.io/edit/master/src/pages/before-you-memo/index.ko.md)
