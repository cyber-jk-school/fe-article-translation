# [번역] useEffect 완벽 가이드

원본: https://overreacted.io/a-complete-guide-to-useeffect/

[Hook](https://reactjs.org/docs/hooks-intro.html)과 함께 몇몇 component를 작성해 봤을 것입니다. 아마도 작은 앱까지도. 대부분 만족 했을 것입니다. API가 편안하다는 느낌을 받고 몇가지 트릭을 익혔을 것입니다. 심지어 반복된 코드를 [custom Hooks](https://reactjs.org/docs/hooks-custom.html)으로 만들어 내었습니다.(300 줄이 줄었습니다!) 그리고 동료들에게 보여주면 "잘했어"할 것입니다.

하지만 `useEffect`를 사용할 때마다 잘 들어맞지 않습니다. 뭔가 놓치고 있다는 느낌을 계속 받게 됩니다. class lifecycles와 비슷하게 보이지만... 정말 그렇습니까? 아래의 질문을 하게되는 자기자신을 보게 됩니다:

- 🤔 `useEffect`로 `componentDidMount`를 흉내내려면 어떻게 해야합니까?
- 🤔 `useEffect` 안에서 데이터 패칭(Data Fetching)은 어떻게 합니까? 배열([])은 무엇입니까?
- 🤔 이펙트를 일으키는 의존성 배열에 함수를 명시해도 됩니까?
- 🤔 왜 가끔 데이터 패칭이 무한 루프에 빠집니까?
- 🤔 왜 가끔 이펙트안에서 예전 state와 prop을 참조하는 것입니까?

Hooks를 시작할 때 나 역시 이 질문들에 대해서 혼란스러웠습니다. 초기 문서를 작성할 때도 일부 미묘함을 확실히 파악하지 못했습니다. 그 이후에 '아하' 하는 상황을 여러분들과 나누고 싶습니다. **이 딥다이브는 이러한 질문들에 대한 답을 명확하게 보여줄 것입니다.**

답을 보기위해서는, 한발 물러나야 합니다. 이 article의 목표는 요약된 정보를 목록 형태로 제공하는 것이 아닙니다. 이것은 `useEffect`를 '완전이해'하게 도와줍니다. 배워야 할 것이 많지않습니다. 사실, 대부분의 시간을 잊어버리는데 사용할 것입니다.

**`useEffect` Hook을 class lifecycle이라는 익숙한 프리즘을 통해 보는 것을 멈추자 명확하게 다가왔습니다.**

> "아는 것을 전부 잊어버려라" - 요다

<p align="center"><img src="https://user-images.githubusercontent.com/53526987/226270370-09ed9226-f617-48b4-b893-2671fea702c6.jpg">
</p>

---

이 article은 독자들이 [`useEffect`](https://legacy.reactjs.org/docs/hooks-effect.html) API에 익숙하다는 전제로 합니다.

이 글은 아주 깁니다. 작은 책과 같습니다. 내가 선호하는 방식입니다. 그러나 빠르게 보고 싶거나 긴 글을 선호하지 않은 분들을 위해 요약본(TLDR)을 작성해 두었습니다.

만약 deep dive하는 것이 불편하다면, 설명이 다른 곳에 나타날 때까지 기다리는 것이 좋습니다. 2013년에 React가 나왔을 때처럼 사람들에게 다른 멘탈 모델을 인식하고 가르치는데 시간이 걸릴 것입니다.

---

## TLDR

전체 읽기 원하지 않다면 요약본(TLDR)이 있습니다. 일부 부분이 이해가 되지 않는 경우 관련 부분까지 스크롤하여 찾을 수 있습니다.

전체 포스트를 읽으려는 경우 건너띄어도 됩니다. 마지막에 링크를 걸겠습니다.

🤔 **Question: `useEffect`로 `componentDidMount`를 흉내내려면 어떻게 해야합니까?**

정확히 같지는 않지만 `useEffect(fn, [])`으로 가능합니다. `componentDidMount`와 달리 props와 state를 잡아둘 것입니다. 그래서 callback 안에서도 초기 props와 state를 확인 할 수 있습니다. "최신"의 것들을 알고 싶다면, ref에 담아둘 수 있습니다. 그러나 코드를 구성할 필요도 없는 간단한 방법이 있습니다. 이펙트가 있는 멘탈 모델은 `componentDidMount`나 다른 라이프사이클들과 다르다는 것을 인지해야 하고, 이것들(`componentDidMount`, 다른 라이프사이클)과 유사한 것들을 찾으려고 하면 오히려 혼란을 야기할 뿐입니다. 생산적으로 접근하기 위해, "이펙트 기준으로 생각"하는 것이 필요하고, 이 멘탈 모델은 라이프사이클 이벤트에 응답하는 것보다 동기화 구현 도구에 더 가깝습니다.

🤔 **Question: `useEffect` 안에서 데이터 패칭(Data Fetching)은 어떻게 합니까? 배열([])은 무엇입니까?**

[이 article](https://www.robinwieruch.de/react-hooks-fetch-data/)은 `useEffect`로 데이터 패칭하는 아주 좋은 기본서입니다. 글을 끝까지 꼭 읽어보세요! 지금 글 같이 길지 않습니다. `[]`는 이펙트에 리액트 데이터 흐름에 참여하는 어떠한 값도 사용하지 않겠다는 것을 의미하여, 한 번 적용해도 안전합니다. 빈 배열에 실제 값이 사용되는 것이 버그를 일으키는 주된 원인 중 하나입니다. 의존성 체크를 생략하는 것보다는 의존성을 필요로 하는 상황을 제거하는 몇가지 전략(주로 `useReducer`와 `useCallback`)을 익혀야 할 필요가 있습니다.

🤔 **Question: 이펙트를 일으키는 의존성 배열에 함수를 명시해도 됩니까?**
추천하는 방법은 props또는 state를 필요하지 않는 함수를 컴포넌트 외부에서 호이스팅하고, 이펙트 안에서 사용되는 것은 이펙트 내부에서 선언해야 합니다. 이펙트가 여전히 랜더 범위(props로 내려오는 함수 포함) 안의 함수를 사용하고 있다면, 그것들을 정의하고 반복되는 과정에서 `useCallback`로 감싸면 됩니다. 왜 중요한가요? 함수들은 props와 state에서 값들을 "볼" 수 있습니다. - 그래서 데이터 흐름과 연관이 있습니다. FAQ에 [자세한 답변](https://legacy.reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)이 있습니다.

🤔 **Question: 왜 가끔 데이터 패칭이 무한 루프에 빠집니까?**
두번째 인자로 의존성 배열에 값을 전달하지 않고 데이터 패칭을 할때 생길 수 있는 문제입니다. 이러한 것이 없다면 모든 랜더 과정 후에 실행되고 - state를 설정하면 이펙트가 다시 트리거 됩니다. 의존성 배열에 계속 바뀌는 값을 넣어도 무한 루프가 발생할 수 있습니다. 하나씩 값을 지워가며 확인할 수 있습니다. 그러나 사용하고 있는 의존 값을 지우는 것(또는 맹목적으로 `[]` 지정)은 잘못된 해결방법입니다. 그 대신에 근원적인 문제를 찾아서 해결해야 합니다. 함수가 문제를 일으킨다고 예를 들면, 이펙트 내부에 함수를 넣거나, 함수들을 꺼내어 호이스팅하거나, `useCallback`으로 감싸면 해결될 것입니다. 객체가 재생성되는 것을 막기 위해, `useMemo`를 비슷한 용도로 사용할 수 있습니다.

🤔 **Question: 왜 가끔 이펙트안에서 예전 state와 prop을 참조하는 것입니까?**
이펙트는 이펙트가 정의된 곳에서 랜더링이 일어날 때마다 props와 state를 "봅"니다. 이것은 [버그를 막을 수 있지만](https://overreacted.io/ko/how-are-function-components-different-from-classes/) 때때로 짜증날 수 있습니다. 이럴 경우, 명시적인 값을 가변성 ref에 넣어 관리할 수 있습니다.(링크 연결된 article 설명들은 끝 부분에 있습니다.) 기대한 것과 달리 예전 랜더링에서 props또는 state가 보인다면, 의존성 배열에 값을 넣는 것을 잊어버렸을 것입니다. [lint 규칙](https://github.com/facebook/react/issues/14920)
을 사용하여 그것들을 파악할 수 있게 연습해야합니다. 몇일 안에 자연스러워 질 것입니다. 또한 FAQ에서 이 [답변](https://legacy.reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function)을 읽어 보시면 됩니다.

---

TLDR이 도움이 되길 희망합니다.! 이젠 시작합니다.

---

## 각 렌더는 Props와 State를 가지고 있다.

이펙트에 대해서 말하기 전에, 렌더링에 대해서 이야기 해봅시다.

여기 카운터가 있습니다. 하이라이트된 부분(\*)을 자세히 봅시다:

```js
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>*
      <button onClick={() => setCount(count+1)}>
    </div>
  );
}
```

이것은 무슨 의미 입니까? `count`가 state의 변화를 "보고" 자동으로 업데이트 하는 겁니까? 처음 리액트를 배울때 직관적으로 보기 좋지만 [정확한 멘탈 모델](https://overreacted.io/react-as-a-ui-runtime/)은 아닙니다.

이 예제에서 `count`는 단지 숫자입니다. "데이터 바인딩", "watcher", "proxy", 그외와 같은 마법적인 것이 아닙니다. 이것은 그냥 숫자입니다.

```js
const count = 42;
//...
<p>You clicked {count} times</p>;
//...
```

컴포넌트를 처음 렌더링 할때, `count`가 `useState()`로 부터 가져온 값은 `0`입니다. `setCount(1)`을 호출하면, 리엑트는 컴포넌트를 다시 호출합니다. 이 때, `count`는 `1`이 되는 식이 됩니다.

```js
// During first render
function Counter() {
  const count = 0;* // Returned by useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// After a click, our function is called again
function Counter() {
  const count = 1;* // Returned by useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// After another click, our function is called again
function Counter() {
  const count = 2;* // Returned by useState()
  // ...
  <p>You clicked {count} times</p>
  // ...
}
```
