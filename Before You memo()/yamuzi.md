# Before you memo()

> Before you memo()

<br/><br/><br/>

# memo()를 사용하기 전.

> There are many articles written about React performance optimizations. In general, if some state update is slow, you need to:

리엑트 성능 측정에는 많은 문서들이 있습니다.

일반적으로, 일부 상태들이 늦게 업데이트 된다면 개발자는 이런 조치들을 취해야 합니다.

> Verify you’re running a production build. (Development builds are intentionally slower, in extreme cases even by an order of magnitude.)

1. 운영 빌드가 실행중인지 확인하세요. (특정(심각한) 경우에는 개발 빌드가 의도적으로 느려질 수 있습니다.)

> Verify that you didn’t put the state higher in the tree than necessary. (For example, putting input state in a centralized store might not be the best idea.)

1. 상태를 필요 이상으로 상위 트리에 넣지 않았는지 확인하세요. (예를 들어, input 상태값을 중앙 저장소에 넣는 것은 좋지 않은 생각입니다)

> Run React DevTools Profiler to see what gets re-rendered, and wrap the most expensive subtrees with `memo()`. (And add `useMemo()` where needed.)

1. 어떤 요소가 리렌더링을 발생시키고 있는지 React DevTools 프로파일러를 실행시켜 확인하고, 가장 높은 비용을 가진 하위 트리를 memo() 함수로 감싸세요. (그리고 useMemo() 함수를 필요한 곳에 삽입하세요)

> This last step is annoying, especially for components in between, and ideally a compiler would do it for you. In the future, it might.

이 마지막 방법은 사이에 위치한 컴포넌트들에 대해 특히 더 귀찮을 수 있으며, 이상적으로는 컴파일러가 이 작업을 해줄지도 모릅니다. 미래에는 그럴겁니다.

> **In this post, I want to share two different techniques.** They’re surprisingly basic, which is why people rarely realize they improve rendering performance.

이 글에서는, 저는 두 가지의 각각 다른 방법들을 소개하려 합니다. 이 방법들은 놀랍게도 기초적이기 때문에, 사람들은 렌더링 성능 개선이 되었음을 드물게 알아차리곤 합니다.

> **These techniques are complementary to what you already know!** They don’t replace `memo` or `useMemo`, but they’re often good to try first.

이 방법들은 여러분이 이미 아는 방법들을 보완합니다. memo, useMemo를 대체하는 방법들은 아니지만, 먼저 시도해보는 것이 좋을 수 있습니다.

> **An (Artificially) Slow Component**

<br/><br/><br/>

## (의도적으로) 느린 컴포넌트

> Here’s a component with a severe rendering performance problem:

심각한 렌더링 성능 문제를 가진 컴포넌트가 있습니다:

```tsx
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
    // Artificial delay -- do nothing for 100ms
  }
  return <p>I am a very slow component tree.</p>;
}
```

_([Try it here](https://codesandbox.io/s/frosty-glade-m33km?file=/src/App.js:23-513))_

> The problem is that whenever `color` changes inside `App`, we will re-render `<ExpensiveTree />` which we’ve artificially delayed to be very slow.

(해당 코드에서의) 문제는 App안에서 color가 바뀔 때, 의도적으로 매우 느리게 딜레이되는 <ExpensiveTree> 컴포넌트가 리렌더링 된다는 것 입니다.

> I could [put `memo()` on it](https://codesandbox.io/s/amazing-shtern-61tu4?file=/src/App.js) and call it a day, but there are many existing articles about it so I won’t spend time on it. I want to show two different solutions.

이때 저는 memo()를 넣어 해당 컴포넌트를 날마다 호출하려 했지만, memo()와 관련하여 존재하는 문서들이 너무 많아 더 이상 시간을 소비하고 싶지 않습니다. 저는 여기서 두 개의 각기 다른 해결 방법을 보여주려 합니다.

<br/><br/>

> **Solution 1: Move State Down**

## 방법 1: 상태값을 하위로 옮기세요.

> If you look at the rendering code closer, you’ll notice only a part of the returned tree actually cares about the current `color`:

렌더링 코드를 자세히 보면, 현재 색상에 일부 반환된 트리만 관여하고 있음을 알 수 있습니다.

```tsx
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
```

> So let’s extract that part into a `Form` component and move state *down* into it:

따라서, Form 컴포넌트를 추출해 상태를 하위로 옮길 수 있습니다.

```tsx
export default function App() {
  return (
    <>
      <Form />
      <ExpensiveTree />
    </>
  );
}

function Form() {
  let [color, setColor] = useState('red');
  return (
    <>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p style={{ color }}>Hello, world!</p>
    </>
  );
}
```

_([Try it here](https://codesandbox.io/s/billowing-wood-1tq2u?file=/src/App.js:64-380))_

> Now if the `color` changes, only the `Form` re-renders. Problem solved.

이제 만약 색상이 바뀌게 된다면, Form 컴포넌트만 리렌더링됩니다. 문제 해결!

<br/><br/>

> **Solution 2: Lift Content Up**

## 방법 2: 콘텐츠를 위로 끌어올리기

> The above solution doesn’t work if the piece of state is used somewhere *above* the expensive tree. For example, let’s say we put the `color` on the *parent* `<div>`:

위의 해결방법은 값비싼 (복잡한) 트리 구조에 있는 상태에서는 동작하지 않습니다. 예를 들어, 부모 div에 색상을 넣어봅시다.

```tsx
export default function App() {
  let [color, setColor] = useState('red');
  return (
    <div style={{ color }}>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      <p>Hello, world!</p>
      <ExpensiveTree />
    </div>
  );
}
```

_([Try it here](https://codesandbox.io/s/bold-dust-0jbg7?file=/src/App.js:58-313))_

> Now it seems like we can’t just “extract” the parts that don’t use `color` into another component, since that would include the parent `<div>`, which would then include `<ExpensiveTree />`. Can’t avoid `memo` this time, right?

부모 div가 ExpensiveTree를 포함하기 때문에, 색상을 사용하지 않는 다른 컴포넌트를 추출할 수 없는 것처럼 보입니다. 이럴 때는 memo를 피할 수 없겠죠?

> Or can we?

또는 우리는 어떤 방식을 사용할 수 있을까요?

> Play with this sandbox and see if you can figure it out.

다음 코드 블록을 실행시켜보고, 다른 방법을 알아내봅시다.

…

…

…

> The answer is remarkably plain:

정답은 매우 분명합니다.

```tsx
export default function App() {
  return (
    <ColorPicker>
      <p>Hello, world!</p>
      <ExpensiveTree />
    </ColorPicker>
  );
}

function ColorPicker({ children }) {
  let [color, setColor] = useState('red');
  return (
    <div style={{ color }}>
      <input value={color} onChange={(e) => setColor(e.target.value)} />
      {children}
    </div>
  );
}
```

_([Try it here](https://codesandbox.io/s/wonderful-banach-tyfr1?file=/src/App.js:58-423))_

> We split the `App` component in two. The parts that depend on the `color`, together with the `color` state variable itself, have moved into `ColorPicker`.

우리는 App 컴포넌트를 두개로 나눌 수 있습니다. 색상에 의존하고 있는 부분과 색상 상태 변수를 직접 가지고 있는 부분이 ColorPicker로 이동했습니다.

> The parts that don’t care about the `color` stayed in the `App` component and are passed to `ColorPicker` as JSX content, also known as the `children` prop.

색상과 상관이 없는 부분은 App 컴포넌트에 두고, children prop인 JSX 로서 ColorPicker에 전달됩니다.

> When the `color` changes, `ColorPicker` re-renders. But it still has the same `children` prop it got from the `App` last time, so React doesn’t visit that subtree.

색상이 바뀌게 된다면, ColorPicker은 리렌더링됩니다. 하지만 지난 시점에 App에서 받아온 children prop을 여전히 가지고 있으므로 리엑트는 해당 하위 트리를 방문하지 않습니다.

> And as a result, `<ExpensiveTree />` doesn’t re-render.

결과적으로, ExpensiveTree는 리렌더링 되지 않습니다.

<br/><br/><br/>

> **What’s the moral?**

## 무엇이 맞을까요?

> Before you apply optimizations like `memo` or `useMemo`, it might make sense to look if you can split the parts that change from the parts that don’t change.

memo, useMemo와 같은 최적화를 적용하기 전에, 변경되지 않는 부분에서 변경되는 부분을 분리할 수 있는지 살펴보는 것이 좋습니다.

> The interesting part about these approaches is that **they don’t really have anything to do with performance, per se**. Using the `children` prop to split up components usually makes the data flow of your application easier to follow and reduces the number of props plumbed down through the tree. Improved performance in cases like this is a cherry on top, not the end goal.

이 접근에 대한 흥미로운 부분은 이들은 성능과 실제 관련이 없다는 것입니다. 컴포넌트를 분리하기 위해 children prop을 사용하는 것은 어플리케이션의 데이터 흐름을 파악하는데 쉽게 만들어 주며 하위 트리로 통하는 props의 수를 줄여줍니다. 성능 향상은 최종 목표가 아닌 화룡점정입니다.

(cherry on top : 화룡점정? 101%의 1%를 채우는 마무리 요소..?)

> Curiously, this pattern also unlocks *more* performance benefits in the future.

흥미롭게도, 이 페턴은 미래에 더 많은 성능적인 장점을 제공합니다.

> For example, when [Server Components](https://reactjs.org/blog/2020/12/21/data-fetching-with-react-server-components.html) are stable and ready for adoption, our `ColorPicker` component could receive its `children` [from the server](https://youtu.be/TQQPAU21ZUw?t=1314). Either the whole `<ExpensiveTree />` component or its parts could run on the server, and even a top-level React state update would “skip over” those parts on the client.

예를 들어, 서버 컴포넌트들이 안정적이고 선택될 준비가 된 경우, 우리의 ColorPicker 컴포넌트는 서버로부터 children을 받을 수 있습니다. 이는 ExpensiveTree 컴포넌트 전체 또는 해당 부분을 서버에서 실행시킬 수 있으며, 또는 최상위 리엑트 상태 업데이트에서도 클라이언트에서 해당 부분을 넘겨버릴 수 있습니다.

> That’s something even `memo` couldn’t do! But again, both approaches are complementary. Don’t neglect moving state down (and lifting content up!)

이는 memo로도 할 수 없는 일입니다! 다시말해, 두 접근 방법들은 상호보완적입니다. 상태를 하위로 내리는 것(그리고 콘텐츠를 위로 올리는 것)을 게을리 하지 마세요!

> Then, where it’s not enough, use the Profiler and sprinkle those memo’s.

그리고, 이것이 충분하지 않다면 Profiler를 사용하고 메모들(memo, useMemo)들을 뿌리세요.

<br/><br/><br/>

> **Didn’t I read about this before?**

### 전에 본 적 없나요?

[Yes, probably.](https://kentcdodds.com/blog/optimize-react-re-renders) → 네 아마도요

> This is not a new idea. It’s a natural consequence of React composition model. It’s simple enough that it’s underappreciated, and deserves a bit more love.

이는 새로운 아이디어가 아닙니다. 이는 리엑트 중첩(합성) 모델의 자연스러운 결과입니다. 이는 너무 간단한 나머지 과소 평가되었고, 조금 더 많은 사랑을 받을 자격이 있습니다.

[Discuss on Twitter](https://mobile.twitter.com/search?q=https%3A%2F%2Foverreacted.io%2Fbefore-you-memo%2F) • [Edit on GitHub](https://github.com/gaearon/overreacted.io/edit/master/src/pages/before-you-memo/index.md)
