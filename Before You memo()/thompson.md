# Before You memo()

# memo()를 사용하기 전에

리액트 성능 최적화와 관련된 많은 아티클이 있습니다. 보통 몇 가지 상태 업데이트가 느릴 때, 당신은

1. 프로덕션 빌드로 실행중인지 확인해야 합니다. (개발 빌드는 극단적인 경우에 수십 배 정도 의도적으로 느립니다.) ([an order of magnitude](https://www.82cook.com/entiz/read.php?num=1312648))
2. 불필요하게 상태를 트리 내에서 더 높은 곳에 넣지 않았는지 확인해야 합니다. (예를 들어, input 상태를 중앙 저장소에 넣는 것은 좋은 생각이 아닙니다.)
3. 어떤 부분에서 리렌더링이 발생하는 지 확인하기 위해 React DevTools Profiler를 실행하고, 가장 비용이 많이 드는 서브트리를 `memo()`로 감싸야 합니다. (그리고 필요한 곳에 `useMemo()`를 사용합니다.)

마지막 단계는 사이에 있는 컴포넌트(?)에서 특히 짜증이 납니다. 이론적으로는 컴파일러가 해당 작업을 진행해 줍니다. 미래에는 가능할 수도 있습니다.

이 글에서, 두 가지의 다른 기술들을 공유하려고 합니다. 그것들은 놀랍도록 기본적이라 사람들은 렌더링 성능이 향상된다고 깨닫지 못합니다.

이 기술들은 당신이 이미 알고 있는 것과 보완적입니다. 그것들은 `memo`나 `useMemo`를 대체할 수 없지만, 먼저 시도해 보기 좋습니다.

## (인위적으로) 느린 컴포넌트

여기 심각한 렌더링 성능 문제가 있는 컴포넌트가 있습니다.

```javascript
import { useState } from "react";

export default function App() {
  let [color, setColor] = useState("red");
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
    // Artificial delay -- do nothing for 100ms
  }
  return <p>I am a very slow component tree.</p>;
}
```

([여기서 실행하기](https://codesandbox.io/s/frosty-glade-m33km?file=/src/App.js:23-513))

위에서의 문제는 `App` 내부에서 `color` 가 변경될 때마다, 인위적으로 지연시킨 `<ExpensiveTree />`가 리렌더링 된다는 것입니다.

`<ExpensiveTree />`에 [`memo()`를 사용](https://codesandbox.io/s/amazing-shtern-61tu4?file=/src/App.js)하고 끝낼 수도([call it a day](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=wannajoinme&logNo=220914541748)) 있지만, 이미 많은 아티클들이 다루고 있기 때문에, 다른 두 가지 해결법을 보여주고자 합니다.

## 해결법 1: 상태를 아래로 내려라

렌더링 코드를 자세히 보면, 반환되는 트리의 일부분만 현재 `color`와 관련이 있다는 걸 알 수 있습니다.

```javascript
export default function App() {
  let [color, setColor] = useState("red");
  return (
    <div>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
      <ExpensiveTree />
    </div>
  );
}
```

그 부분을 `Form`이라는 컴포넌트로 나누고 상태를 해당 컴포넌트로 내려보겠습니다.

```javascript
export default function App() {
  return (
    <>
      <Form />
      <ExpensiveTree />
    </>
  );
}

function Form() {
  let [color, setColor] = useState("red");
  return (
    <>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
    </>
  );
}
```

([여기서 실행하기](https://codesandbox.io/s/billowing-wood-1tq2u?file=/src/App.js:64-380))

이제 `color`가 변경되면 `Form` 컴포넌트만 리렌더링 됩니다. 문제가 해결됐네요.

## 해결법 2: 컨텐츠를 위로 올려라

앞서 설명한 해결법은 `ExpensiveTree`의 상위에서 상태를 사용하는 경우에는 동작하지 않습니다. 예를 들어, 부모 `<div>`에 `color`를 넣는다고 해봅시다.

```javascript
export default function App() {
  let [color, setColor] = useState("red");
  return (
    <div style={{ color }}>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p>Hello, world!</p>
      <ExpensiveTree />
    </div>
  );
}
```

([여기서 실행하기](https://codesandbox.io/s/bold-dust-0jbg7?file=/src/App.js:58-313))

이제 `color`를 사용하지 않는 부분을 다른 컴포넌트로 추출할 수 없는 것 같습니다. 부모 `<div>`도 포함되어 있고, 부모 요소는 `<ExpensiveTree />`를 포함하고 있으니까요. 이번에는 `memo`를 피할 수 없죠? 맞죠??

아니면 `memo`를 사용하지 않을 수 있을까요?
샌드박스를 실행해보고 알 수 있는지 확인해보세요.

답은 매우 간단합니다.

```javascript
export default function App() {
  return (
    <ColorPicker>
      <p>Hello, world!</p>
      <ExpensiveTree />
    </ColorPicker>
  );
}

function ColorPicker({ children }) {
  let [color, setColor] = useState("red");
  return (
    <div style={{ color }}>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      {children}
    </div>
  );
}
```

([여기서 실행하기](https://codesandbox.io/s/wonderful-banach-tyfr1?file=/src/App.js:58-423))

우리는 `App` 컴포넌트를 두 개로 분리했습니다. `color`에 의존하는 부분은 `color` 상태 변수와 함께 `ColorPicker` 컴포넌트 내부로 이동시켰습니다.
`color` 상태와 관련 없는 부분은 그대로 `App` 컴포넌트 내부에 두고, JSX 컨텐츠인 `children`으로 `ColorPicker`에 전달합니다.
`color`가 변경되면 `ColorPicker`는 리렌더링 됩니다. 그러나 마지막으로 `App`이 렌더링 될 때의 `children`과 여전히 동일하기 때문에 리액트는 서브트리를 방문하지 않습니다. 그 결과로 `<ExpensiveTree />`는 렌더링 되지 않습니다.

## 그래서 교훈은?

`memo`나 `useMemo`와 같은 최적화를 적용하기 전에, 변경되는 부분과 변경되지 않는 부분을 나눌 수 있는지 확인할 수 있습니다.

이러한 접근 방식에 대해 흥미로운 점은 그들이 실제로 성능에 영향을 주지 않는다는 것입니다. 컴포넌트를 쪼개기 위해 `children` prop을 사용하는 것은 대개 앱의 데이터 흐름을 더 쉽게 파악할 수 있게 만들고 트리 하위로 전달되는 prop의 수를 줄여줍니다. 이런 경우에서 향상된 성능은 최종 목표가 아닌 cherry on top(금상첨화? 화룡점정?)입니다.

신기하게도, 이 패턴은 미래에 더 많은 성능적 이점을 제공합니다.

예를 들어, [서버 컴포넌트](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html)가 안정적이고 적용할 준비가 되었다면, `ColorPicker` 컴포넌트는 `children`을 [서버로부터](https://www.youtube.com/watch?v=TQQPAU21ZUw&t=1314s) 전달받을 수 있습니다. `<ExpensiveTree />` 컴포넌트 또는 컴포넌트의 일부분은 서버에서 실행될 수 있고, 최상위 리액트 상태 업데이트는 클라이언트에서 스킵될 수 있습니다.

이건 `memo`가 할 수 없습니다. 그런데 다시 말하지만 모든 접근법은 상호보완적입니다. 상태를 아래로 내리거나, 컨텐츠를 위로 올리는 걸 잊지 마세요!

그러고 나서 충분하지 않다면 Profiler나 memo를 추가하세요.

## 이전에 이것과 관련된 글을 읽어본 적이 없었나요?

[아마 있을거예요](https://kentcdodds.com/blog/optimize-react-re-renders)

이것은 새로운 아이디어가 아닙니다. 이건 리액트 구성 모델의 자연스러운 결과입니다. 이것은 인정 받지 못할 만큼 충분히 간단하지만, 더 사랑 받을 자격이 있습니다.
