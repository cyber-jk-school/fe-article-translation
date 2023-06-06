# A Complete Guide to useEffect

### useEffect에 대한 완벽한 가이드

> You wrote a few components with [Hooks](https://reactjs.org/docs/hooks-intro.html). Maybe even a small app.

여러분은 hooks로 몇 개의 컴포넌트들을 작성해봤을 겁니다. 매우 작은 앱에서라도 말입니다.

> You’re mostly satisfied. You’re comfortable with the API and picked up a few tricks along the way.

대부분 만족 하셨을겁니다. API를 사용하며 편리함을 느끼고, 몇 개의 방법들을 고르기도 했을 겁니다.

> You even made some [custom Hooks](https://reactjs.org/docs/hooks-custom.html) to extract repetitive logic (300 lines gone!) and showed it off to your colleagues. “Great job”, they said.

여러분은 심지어 반복되는 로직을 추출하기 위해 몇 개의 custom hooks를 만들었으며 (300줄이 사라졌네요!), 당신의 동료들에게 보여줬을 것 입니다. 그들은 “대단해요”라고 말했을 겁니다.

> But sometimes when you `useEffect`, the pieces don’t quite fit together.

하지만 가끔 당신이 `useEffect`를 사용할 때, 뭔가 조각들이 서로 잘 맞지 않습니다.

> You have a nagging feeling that you’re missing something. It seems similar to class lifecycles… but is it really? You find yourself asking questions like:

여러분은 어떤것을 놓치고 있다고 생각하며 짜증이 나기도 합니다. 이는 class lifecycles와 비슷하게 보입니다만… 진짜 그럴까요? 여러분은 스스로에게 다음과 같은 질문들을 할 것 입니다.

> - 🤔 How do I replicate `componentDidMount` with `useEffect`?

- `useEffect`를 사용하여 `componentDidMount`를 흉내내려면 어떻게 해야할까?

> - 🤔 How do I correctly fetch data inside `useEffect`? What is `[]`?

- `useEffect` 안에서 데이터를 fetch하려면 어떻게 해야할까? `[]`는 뭐지?

> - 🤔 Do I need to specify functions as effect dependencies or not?

- 특정 함수를 effect 의존성으로서 작성해야 할까 말까?

> - 🤔 Why do I sometimes get an infinite refetching loop?

- 왜 가끔 데이터 refetching 무한 루프에 빠지는 걸까?

> - 🤔 Why do I sometimes get an old state or prop value inside my effect?

- effect 안에서 왜 가끔 이전의 state 또는 prop 값을 참조하게 될까?

> When I just started using Hooks, I was confused by all of those questions too.

저도 hooks를 처음 사용하면서, 이런 모든 질문들에 대해 혼란스러웠습니다.

> Even when writing the initial docs, I didn’t have a firm grasp on some of the subtleties.

초기 문서를 작성 하면서도, 저는 몇 가지 미묘한 점에 대해서 확실히 파악하지 못했습니다. (grasp : 터득하다/파악하다, subtleties: 미묘, 희박)

> I’ve since had a few “aha” moments that I want to share with you. **This deep dive will make the answers to these questions look obvious to you.**

몇 번의 ‘아하’ (깨닫는) 순간들을 가진 후 저는 여러분에게 이 점들을 공유하고 싶습니다. 이 심층 분석(deep dive)은 당신에게 이 질문들에 대한 분명한 답을 줄 것입니다.

> To *see* the answers, we need to take a step back. The goal of this article isn’t to give you a list of bullet point recipes. It’s to help you truly “grok” `useEffect`. There won’t be much to learn. In fact, we’ll spend most of our time *un*learning.

답을 보기 위해서는, 우리는 한 걸음 뒤로 물러서야 할 필요가 있습니다. 이 문서의 목표는 단순히 여러분에게 목록 형태의 레시피를 주는 것이 아닙니다. 진짜 목표는 여러분이 `useEffect`를 “진정으로 이해하도록” 돕는 것입니다. (grok : 진정으로 이해하다, 공감하다) 배워야 할 것이 많지는 않습니다. 사실, 우리는 대부분의 시간을 배웠던 점들을 잊는 것에 쓸 예정입니다.

> **It’s only after I stopped looking at the `useEffect` Hook through the prism of the familiar class lifecycle methods that everything came together for me.**

단순히 `useEffect` hook을 친숙한 class lifecycle 메소드라는 프리즘을 통해 바라보는 것을 멈춘 뒤에야, 모든 내용이 저에게 다가왔습니다.

> “Unlearn what you have learned.” — Yoda

“배운 것들을 모두 잊으세요” - Yoda

![https://overreacted.io/static/6203a1f1f2c771816a5ba0969baccd12/fce5f/yoda.jpg](https://overreacted.io/static/6203a1f1f2c771816a5ba0969baccd12/fce5f/yoda.jpg)

---

> **This article assumes that you’re somewhat familiar with `[useEffect](https://reactjs.org/docs/hooks-effect.html)` API.**

이 문서는 여러분이 어느 정도 `useEffect` API를 사용하는 것에 익숙하다는 것을 전제로 합니다.

> **It’s also *really* long. It’s like a mini-book. That’s just my preferred format. But I wrote a TLDR just below if you’re in a rush or don’t really care.**

이 문서는 또한 정말 깁니다. 작은 책과 비슷합니다. 이는 제가 선호하는 방식입니다. 하지만 저는 여러분이 빠르게 살펴봐야 하거나 이 글을 별로 상관하고 싶지 않은 경우를 위해 TLDR을 작성했습니다.

> **If you’re not comfortable with deep dives, you might want to wait until these explanations appear elsewhere. Just like when React came out in 2013, it will take some time for people to recognize a different mental model and teach it.**

심층 분석 글이 불편한 경우, 여러분은 이 설명 내용이 다른 어딘가에 올라오기를 기다리는 것이 좋습니다. 리엑트가 2013년에 처음 나오고, 사람들이 다른 mental 모델을 이해하고 가르치기까지 약간의 시간이 걸렸던 것과 비슷하게 말입니다.

---

> ## **TLDR**

> Here’s a quick TLDR if you don’t want to read the whole thing. If some parts don’t make sense, you can scroll down until you find something related.

전체를 읽고 싶지 않은 여러분들을 위해, 여기 빠른 TLDR이 있습니다. 만약 몇 부분이 이해되지 않는다면, 연관된 부분으로 스크롤을 아래로 내리시면 됩니다.

> Feel free to skip it if you plan to read the whole post. I’ll link to it at the end.

만약 여러분이 글 전체를 읽으실 거라면 편하게 스킵하셔도 됩니다. 이 부분 링크를 끝에 달아놓겠습니다.

> **🤔 Question: How do I replicate `componentDidMount` with `useEffect`?**

**🤔** 질문 : `useEffect`를 사용하여 `componentDidMount`를 흉내내려면 어떻게 해야할까?

> While you can `useEffect(fn, [])`, it’s not an exact equivalent.

정확하게 같지는 않지만, 여러분은 `useEffect(fn, [])` 을 사용하시면 됩니다.

> Unlike `componentDidMount`, it will *capture* props and state. So even inside the callbacks, you’ll see the initial props and state.

`componentDidMount`와 다르게, 이는 props와 state를 잡아둘 것 입니다. 그래서 콜백함수 안에서도 초기의 props와 state를 확인할 수 있습니다.

> If you want to see “latest” something, you can write it to a ref. But there’s usually a simpler way to structure the code so that you don’t have to.

만약 “최신의” 것(props, state)를 원한다면, 여러분은 ref를 사용하여 작성할 수 있습니다. 하지만 보통 코드를 구조화하는 더 간단한 방법이 있기 때문에 그렇게 작성할 필요는 없습니다.

> Keep in mind that the mental model for effects is different from `componentDidMount` and other lifecycles, and trying to find their exact equivalents may confuse you more than help.

effect를 다루는 mental 모델은 `componentDidMount`나 다른 lifecycles와 다르다는 것을 인지해야 하며, 완벽하게 같게 만드려고 시도하면 여러분을 돕기보다는 혼란을 줄 수 있습니다.

> To get productive, you need to “think in effects”, and their mental model is closer to implementing synchronization than to responding to lifecycle events.

생산성을 높이기 위해서, 여러분은 “effect에 대해 생각”할 필요가 있으며, 이 mental 모델은 lifecycle 이벤트에 응답하는 것보다 동기화를 구현하는 것에 더 가깝습니다.

> **🤔 Question: How do I correctly fetch data inside `useEffect`? What is `[]`?**

**🤔** 질문 : `useEffect` 안에서 데이터를 fetch하려면 어떻게 해야할까? `[]`는 뭐지?

> [This article](https://www.robinwieruch.de/react-hooks-fetch-data/) is a good primer on data fetching with `useEffect`. Make sure to read it to the end! It’s not as long as this one.

[이 문서](https://www.robinwieruch.de/react-hooks-fetch-data/)는 useEffect를 사용하여 데이터를 fetch하는 것에 대한 좋은 기초 문서입니다. 글을 끝까지 읽어보세요! 이 글만큼 길지 않습니다.

> `[]` means the effect doesn’t use any value that participates in React data flow, and is for that reason safe to apply once. It is also a common source of bugs when the value actually *is* used. You’ll need to learn a few strategies (primarily `useReducer` and `useCallback`) that can *remove the need* for a dependency instead of incorrectly omitting it.

`[]`는 effect가 React 데이터 흐름의 어떠한 값도 사용하지 않겠다는 의미이며, 이는 한번 동작해도 안전한 이유이기도 합니다. 이는 또한 값이 실제로 사용되어야 할때 버그의 주된 원인이 되기도 합니다. 여러분은 올바르지 않게 의존성 체크를 생략하는 것 대신에 의존성이 필요한 부분을 제거하는 몇 개의 전략(주로 `useReducer` 또는 `useCallback`)을 익혀야 할 필요가 있습니다.

(the need for a dependency : 의존성이 필요한 부분? 상황?, 의존성의 필요성?)

> **🤔 Question: Do I need to specify functions as effect dependencies or not?**

**🤔** 질문: 특정 함수를 effect 의존성으로서 작성해야 할까 말까?

> The recommendation is to hoist functions that don’t need props or state *outside* of your component, and pull the ones that are used only by an effect *inside* of that effect. If after that your effect still ends up using functions in the render scope (including function from props), wrap them into `useCallback` where they’re defined, and repeat the process. Why does it matter? Functions can “see” values from props and state — so they participate in the data flow. There’s a [more detailed answer](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies) in our FAQ.

추천하는 방법은 컴포넌트 외부에서 props나 state가 필요하지 않는 함수를 호이스팅시키고, effect 내부에서 효과를 미치는 것들만 넣는 (작성하는) 것입니다. 만약 랜더 범위 내 함수(props로 내려오는 함수 포함)를 effect가 여전히 사용하고 있다면, 해당 함수가 정의된 곳에서 `useCallback`으로 묶은 후, 이 과정을 반복하세요. 왜 이 부분이 중요할까요? 함수들은 props와 state로 오는 값을 “추적”할 수 있습니다. — 따라서 리엑트의 데이터 흐름과 관련이 있습니다. 여기 FAQ에 이에 대한 더 자세한 답변이 있습니다.

> **🤔 Question: Why do I sometimes get an infinite refetching loop?**

**🤔** 질문: 왜 가끔 데이터 refetching 무한 루프에 빠지는 걸까?

> This can happen if you’re doing data fetching in an effect without the second dependencies argument. Without it, effects run after every render — and setting the state will trigger the effects again. An infinite loop may also happen if you specify a value that *always* changes in the dependency array. You can tell which one by removing them one by one. However, removing a dependency you use (or blindly specifying `[]`) is usually the wrong fix. Instead, fix the problem at its source. For example, functions can cause this problem, and putting them inside effects, hoisting them out, or wrapping them with `useCallback` helps. To avoid recreating objects, `useMemo` can serve a similar purpose.

이는 effect 안에서 데이터 fetch를 할 때 두번째 의존성 인자를 전달하지 않았을 때 발생합니다. 이것(두번째 의존성 인자)를 제외하게 되면, effect는 모든 랜딩 이후마다 실행되게 됩니다. — 또한 state를 세팅하는 것 또한 effect를 재 실행 시킵니다. 주로 여러분이 항상 변하는 특정한 값을 의존성 배열에 넣었을 때 이러한 무한 루프가 발생하게 됩니다. 여러분은 하나씩 값을 지우면서 어떠한 값이 원인인지 파악할(말할) 수 있습니다. 하지만, 여러분이 사용하고 있는 의존성을 삭제하는 것(또는 무턱대고 `[]`를 작성하는 것)은 주로 잘못된 해결방법입니다. 대신에, 문제의 근원을 해결하세요. 예를 들어, 함수들은 이 문제를 일으킬 수 있으며, effect 안에 해당 함수를 넣거나, 이를 호이스팅 시키거나, `useCallback`으로 감쌈으로서 문제를 해결할 수 있습니다. 객체가 재생성되는 것을 피하기 위해서, `useMemo`를 비슷한 목적으로 사용할 수 있습니다.

> **🤔 Why do I sometimes get an old state or prop value inside my effect?**

**🤔** 질문 : effect 안에서 왜 가끔 이전의 state 또는 prop 값을 참조하게 될까?

> Effects always “see” props and state from the render they were defined in. That [helps prevent bugs](https://overreacted.io/how-are-function-components-different-from-classes/) but in some cases can be annoying. For those cases, you can explicitly maintain some value in a mutable ref (the linked article explains it at the end). If you think you’re seeing some props or state from an old render but don’t expect it, you probably missed some dependencies. Try using the [lint rule](https://github.com/facebook/react/issues/14920) to train yourself to see them. A few days, and it’ll be like a second nature to you. See also [this answer](https://reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function) in our FAQ.

Effect는 props와 state를 그들이 정의된 곳이 랜딩될 때 항상 지켜보고 있습니다. 이는 버그를 예방하는 데에 도음을 주지만 어떤 경우에서는 방해가 될 수 있습니다. 이러한 케이스에서는, 여러분은 명확하게 어떠한 값을 가변성을 가진 ref를 사용함으로써 유지할 수 있습니다. (링크되어 있는 문서 끝에 설명되어 있습니다.) 만약에 여러분이 기대하지 않은 오래된 상태의 어떠한 props나 state가 보인다면, 여러분은 아마도 어떠한 의존성을 놓쳤을 수 있습니다. lint rule을 사용해서 이를 확인할 수 있도록 스스로 연습해보세요. 몇 일 뒤에 이는 자연스럽게 여러분께 녹아들 것 입니다. 여기 FAQ에 있는 이 답변 또한 확인해보세요.

(second nature to you : 제 2의 천성이 되다 → 익숙해지다, 녹아들다)

---

> I hope this TLDR was helpful! Otherwise, let’s go.

저는 이 TLDR이 도움이 되었으면 좋겠습니다! 그렇지 않으면, 갑시다.

---

> ## **Each Render Has Its Own Props and State**

각각의 랜딩은 각자의 고유한 Props와 State를 가집니다.

> Before we can talk about effects, we need to talk about rendering.

우리가 effect에 대해 이야기하기 전에, 우리는 렌더링에 대해 먼저 이야기해볼 필요가 있습니다.

> Here’s a counter. Look at the highlighted line closely:

여기 카운터가 있습니다. 하이라이트된 줄을 자세히 봐주세요.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

> What does it mean? Does `count` somehow “watch” changes to our state and update automatically? That might be a useful first intuition when you learn React but it’s *not* an [accurate mental model](https://overreacted.io/react-as-a-ui-runtime/).

이게 무슨 뜻일까요? `count`가 어떻게든 state의 변화를 “감지”하고 자동으로 업데이트 한다는 것일까요? 이는 여러분이 처음 리엑트를 배우면서 직관적으로 유용할 수 있지만 이는 정확한 mental 모델은 아닙니다.

> **In this example, `count` is just a number.** It’s not a magic “data binding”, a “watcher”, a “proxy”, or anything else. It’s a good old number like this one:

이 예시에서, `count`는 단순하게 숫자입니다. 마법의 “데이터 바인딩”, “wacher”, “프록시”가 아닌, 또는 그 어떤 것도 아닙니다. 이는 다음과 같은 그저 좋은 숫자입니다.

```jsx
const count = 42;
// ...
<p>You clicked {count} times</p>;
// ...
```

> The first time our component renders, the `count` variable we get from `useState()` is `0`. When we call `setCount(1)`, React calls our component again. This time, `count` will be `1`. And so on:

우리의 컴포넌트가 랜더링 된 첫번째 시점에, `useState`로부터 가져온 `count` 변수는 0입니다. 우리가 `setCount(1)`을 호출하면, 리엑트는 우리의 컴포넌트를 다시 호출합니다. 이 시점에, `count`는 1이 됩니다.

```jsx
// During first render
function Counter() {
  const count = 0; // Returned by useState()
  // ...
  <p>You clicked {count} times</p>;
  // ...
}

// After a click, our function is called again
function Counter() {
  const count = 1; // Returned by useState()
  // ...
  <p>You clicked {count} times</p>;
  // ...
}

// After another click, our function is called again
function Counter() {
  const count = 2; // Returned by useState()
  // ...
  <p>You clicked {count} times</p>;
  // ...
}
```

> **Whenever we update the state, React calls our component. Each render result “sees” its own `counter` state value which is a *constant* inside our function.**

우리가 state를 업데이트할 때 마다, 리엑트는 우리의 컴포넌트를 호출합니다. 각각의 렌더링의 결과는 우리의 함수 안에 상수로 존재하는 고유한 `counter` state 값을 “지켜봅니다”.

> So this line doesn’t do any special data binding:

따라서 이 줄은 특별한 데이터 바인딩이 일어나지 않습니다.

```jsx
<p>You clicked {count} times</p>
```

> **It only embeds a number value into the render output.** That number is provided by React. When we `setCount`, React calls our component again with a different `count` value. Then React updates the DOM to match our latest render output.

이는 단지 렌더링 결과물에 숫자값을 임베드하는 과정입니다. 해당 숫자는 리엑트에 의해 제공됩니다. 우리가 `setCount`를 실행할 때, 리엑트는 다른 `count` 값과 함께 우리의 컴포넌트를 다시 호출합니다. 그리고 리엑트는 최신의 랜더링 결과물에 맞게 돔을 업데이트합니다.

> The key takeaway is that the `count` constant inside any particular render doesn’t change over time. It’s our component that’s called again — and each render “sees” its own `count` value that’s isolated between renders.

중요한 점은 특정 랜더 내부의 count 상수는 시간에 따라 변경되지는 않는다는 것입니다. 우리의 컴포넌트가 다시 호출되고 — 각각의 랜더링은 랜더링 사이에 분리된 고유한 count 값을 “지켜봅니다”.

> _(For an in-depth overview of this process, check out my post [React as a UI Runtime](https://overreacted.io/react-as-a-ui-runtime/).)_

이 과정에 대해 깊게 들여다보고 싶다면, 제가 작성한 *[React as a UI Runtime](https://overreacted.io/react-as-a-ui-runtime/)*을 확인해보세요.
