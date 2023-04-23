# useEffect 완벽 가이드

당신은 [훅](https://reactjs.org/docs/hooks-intro.html)과 함께 몇몇 컴포넌트를 작성했습니다. 아마 작은 앱이라도요. 대부분 만족했을 겁니다. 당신은 Hooks API를 사용하는 게 편하고, 몇 가지 트릭을 배웠습니다. 당신은 반복되는 로직을 추출하기 위해 [커스텀 훅](https://reactjs.org/docs/hooks-custom.html)을 만들기도 하고(300줄이 줄었습니다!) 그것을 동료들에게 보여줬습니다. "잘했어" 라고 그들이 말했습니다.   

그러나 가끔 `useEffect`를 사용할 때, 조각들이 서로 맞지 않습니다. 당신은 무언가 놓치고 있다는 느낌을 계속해서 가지고 있습니다. 클래스 생명주기와 비슷해보이는데.. 이게 진짜일까? 스스로에게 아래와 같은 질문을 하는 당신을 발견하게 됩니다. 
- 어떻게 `useEffect`를 사용해서 `componentDidMout`를 따라할 수 있을까?
- `useEffect` 내에서 데이터를 어떻게 올바르게 가져올 수 있을까? `[]`는 뭘까?
- `useEffect`의 의존성으로 함수를 지정해야 할까?
- 가끔씩 무한 refetch 루프에 빠지는 이유는 뭘까?
- 가끔 `useEffect` 내에서 예전 상태나 prop 값을 가져오는 이유는 뭘까?

제가 처음 훅을 사용하기 시작했을 때, 저 또한 위의 질문들로 혼란스러웠습니다. 초기 문서를 작성할 때 조차, 몇 가지 미묘한 부분들에 대한 확실한 이해가 없었습니다. 나는 당신에게 공유하고 싶은 몇 가지 아하 모먼트를 겪었습니다. 이 딥 다이브는 당신이 위의 문제에 대한 명확한 해답을 할 수 있도록 만들 것입니다.  

정답을 보기 위해, 우리는 한 걸음 뒤로 물러날 필요가 있습니다. 이 글의 목적은 당신에게 요점 정리 목록을 전달하는 것이 아닙니다. 당신이 완전히 `useEffect`를 "이해"하도록 돕는 것입니다. 배울 것이 많지 않을 것입니다. 사실, 대부분의 시간을 아는 것을 버리는 데에 사용할 것입니다.  

익숙한 클래스 생명 주기 메서드를 통해 `useEffect` 훅을 바라보는 것을 멈추고 나서야 모든 것이 이해되었습니다.

> "배웠던 것을 버려라" - 요다

![요다 이미지](https://overreacted.io/static/6203a1f1f2c771816a5ba0969baccd12/fce5f/yoda.jpg) 

이 글은 당신이 [`useEffect`](https://reactjs.org/docs/hooks-effect.html) API에 어느 정도 익숙하다고 가정합니다.  

또한 이 글은 매우 깁니다. 미니북에 가깝습니다. 단지 제가 선호하는 형식입니다. 그러나 당신이 바쁘거나 크게 신경쓰지 않을 수 있기 때문에, 바로 아래에 요약을 적어두었습니다.  

만약 당신이 깊게 파고드는 것이 불편하다면, 이 설명이 다른 곳에 등장할 때까지 기다려야할 수 있습니다. 리액트가 2013년에 등장했을 때 처럼, 사람들이 다른 멘탈 모델을 깨닫고 가르치는 데 까지 시간이 좀 걸릴 것입니다.

## TLDR
당신이 모든 걸 읽고 싶지 않다면, 여기 요약본이 있습니다. 몇몇 부분이 이해되지 않는다면, 스크롤을 내려 관련된 내용을 찾을 수 있습니다.  

모든 글을 읽을 계획이라면 넘어가도 좋습니다. 마지막에 링크를 걸어두겠습니다.  

### 질문: 어떻게 `useEffect`를 사용해서 `componentDidMout`를 따라할 수 있을까?
당신이 `useEffect(fn, [])`을 사용할 수 있지만, 이것은 완벽하게 같지는 않습니다. `componentDidMound`와 다르게, useEffect는 props와 state를 캡처할 것입니다. 그래서 callbacks 내부에서도 초기 props와 state를 확인할 수 있을 것입니다. 만약 최신의 값을 보고 싶다면, ref를 사용할 수 있습니다. 그러나 보통 그럴 필요 없이 코드를 설계할 수 있는 간단한 방법이 있습니다. effect의 개념이 `componentDidMound`나 다른 생명 주기와 다르다는 것을 기억하고, 그것들과 완벽하게 동일한 것을 찾으려고 하면 오히려 혼란스러울 수 있다는 사실을 기억하세요. 생산성을 위해, "effects로 생각"하고, 그들의 멘탈 모델은 생명 주기 이벤트에 응답하는 것보다 동기화를 구현하는 것에 더 가깝습니다.(번역을 어떻게 하면 더 깔끔하게 할 수 있을지 모르겠네요.)  

### 질문: `useEffect` 내에서 데이터를 어떻게 올바르게 가져올 수 있을까? `[]`는 뭘까?
[이 글](https://www.robinwieruch.de/react-hooks-fetch-data/)은 `useEffect`에서 데이터를 불러오는 좋은 입문서입니다. 끝까지 읽어보세요! 지금 읽는 글처럼 길지 않습니다. `[]`는 이펙트가 리액트 데이터 흐름에 관여하는 어떤 값도 사용하지 않는다는 것을 의미합니다. 그리고 이것은 한번만 적용되어도 안전하다는 이유입니다. 이것은 또한 값이 실제로 사용될 때, 흔한 버그의 원인이기도 합니다. 당신은 몇 가지 전략(주로 `useReducer`와 `useCallback`)을 공부할 필요가 있습니다. 이 전략은 옳지 않게 의존성을 생략하는 것 대신에 의존성의 필요를 없앨 수 있습니다. 
### 질문: `useEffect`의 의존성으로 함수를 지정해야 할까?
추천하는 방법은 props나 state가 필요 없는 함수를 컴포넌트 바깥으로 끌어올리고, effect 내부에서 사용하는 것은 effect 내부에서 정의하는 것입니다. 만약 effect가 끝나고 렌더 스코프에서 함수를 사용한다면(props에서 받아온 함수 포함), 그것들을 정의할 때 `useCallback`을 사용해 감싸고 과정을 반복하세요. 왜 이게 중요할까요? 함수는 props와 state로 부터 온 값으로 볼 수 있습니다. 그래서 그들은 데이터 흐름에 포함되어 있습니다. 우리의 FAQ에 [더 자세한 답변](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)이 있습니다.
### 질문: 가끔씩 무한 refetch 루프에 빠지는 이유는 뭘까?
이것은 두 번째 의존성 인자 없이 effect 내부에서 데이터 페칭을 할 때 발생합니다. 의존성 배열이 없으면, effect는 매 렌더 이후에 실행됩니다 ㅡ 그리고 상태를 설정하는 것은 effect를 다시 실행하게 됩니다. 무한 루프는 의존성 배열에 항상 바뀌는 값을 명시할 경우에 발생할 수도 있습니다. 당신은 의존성 배열의 값을 하나씩 지우면서 확인할 수 있습니다. 그러나, 사용하는 의존성을 지우는 것은 (또는 맹목적으로 `[]`를 명시하는 것) 보통 잘못된 수정 방법입니다. 대신에, 문제가 발생하는 부분을 수정하세요. 예를 들어, 함수가 이 문제의 원인일 수 있는데, effect 안에 넣고, 밖으로 꺼내거나 `useCallback`으로 감싸는 게 도움이 될 수 있습니다. 객체를 재생성하는 걸 피하기 위해서, `useMemo`를 비슷한 목적으로 사용할 수 있습니다.
### 질문: 가끔 `useEffect` 내에서 예전 상태나 prop 값을 가져오는 이유는 뭘까?
effect는 항상 그것들이 정의된 렌더에서 props와 state를 "봅니다". 그것은 버그를 막는데 도움이 되지만, 몇몇 경우에는 짜증이 날 수 있습니다. 그런 경우에, mutable ref에 값을 명시적으로 저장할 수 있습니다. (링크된 글의 마지막 부분에서 설명하고 있습니다.) 만약 당신이 이전 렌더의 props와 state 값을 보고 있고 예상하지 않았다면, 아마 몇 가지 의존성을 깜빡했을 수 있습니다. 의존성 확인하는 연습을 위해 린트 규칙을 사용해보세요. 몇 일이 지나면, 그것은 자연스러워질 것입니다. 우리의 FAQ에서 [이 답변](https://reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function)도 확인해보세요.  

위 요약이 도움이 되길 바랍니다! 그렇지 않으면, 가봅시다.

## 렌더는 각각의 Props와 State를 가지고 있다.
이펙트에 대해 이야기하기 전에, 렌더링에 대해 이야기 할 필요가 있습니다.

여기 카운터가 있습니다. 하이라이트 된 줄을 자세하게 봅시다.
```javascript
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p> {/* highlighted line */}
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
무슨 의미인가요? `count`가 어떻게든 상태 변화를 "감지하고" 자동으로 업데이트 하는건가요? 이건 리액트를 배울 때 유용한 직관일 수 있지만 이것은 [정확한 멘탈 모델](https://overreacted.io/react-as-a-ui-runtime/)이 아닙니다.

위 예시에서, `count`는 단지 number입니다. 이것은 데이터 바인딩, watcher, proxy 등의 마법이 아닙니다. 이것은 아래와 같은 좋은 오래된 number입니다. 
```js
const count = 42;
// ...
<p>You clicked {count} times</p>
// ...
```
컴포넌트가 처음 렌더되었을 때, `useState()`로부터 우리가 얻는 `count` 변수는 `0`입니다. `setState(1)`을 호출하면, 리액트는 컴포넌트를 다시 호출합니다. 이 때, `count`는 `1`이 될 것입니다. 
```js
// 첫 번째 렌더
function Counter() {
  const count = 0; // useState()에 의해 반환
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// 클릭 이후, 함수가 다시 호출됨
function Counter() {
  const count = 1; // useState()에 의해 반환
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// 또 다른 클릭 이후, 함수가 다시 호출됨
function Counter() {
  const count = 2; // useState()에 의해 반환
  // ...
  <p>You clicked {count} times</p>
  // ...
}
```
우리가 상태를 업데이트 할 때마다, 리액트는 컴포넌트를 호출합니다. 각 렌더 결과는 함수 내의 상수값인 `counter` 상태값을 바라 봅니다.

그래서 아래의 라인은 특별한 데이터 바인딩을 하지 않습니다.
```html
<p>You clicked {count} times</p>
```
이것은 오직 렌더 결과에 number 값을 끼워 넣습니다. 그 number는 리액트에 의해 제공됩니다. `setCount`를 호출하면, 리액트는 다른 `count` 값과 함께 컴포넌트를 다시 호출합니다. 그리고 리액트는 최신 렌더 결과에 맞게 DOM을 업데이트합니다.  

여기서 핵심은 어떤 특정한 렌더 내의 `count` 상수는 시간에 따라 변화하지 않는다는 것입니다. 컴포넌트가 다시 호출하고 각각의 렌더는 렌더 간 독립적인 `count` 값을 바라봅니다. 

(이 과정에 대한 좀 더 상세한 개요를 위해, 제 글 [React as a UI Runtime](https://overreacted.io/react-as-a-ui-runtime/) 을 확인해보세요.)

## 렌더는 각각의 이벤트 핸들러를 가지고 있다.
지금까지 매우 좋습니다. 이벤트 핸들러는 어떨까요?

아래 예시를 봅시다. 3초 후에 `count` 값이 있는 alert를 보여줍니다.
```js
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```
이 일련의 단계를 수행한다고 해보겠습니다.
- counter를 3으로 증가시킨다
- "Show alert"를 누른다
- timeout이 실행되기 전에 5까지 증가시킨다

![image](https://overreacted.io/46c55d5f1f749462b7a173f1e748e41e/counter.gif)

alert가 어떤 값을 보여줄 거라고 예상하시나요? alert가 보일 때의 counter 상태인 5를 보여줄까요? 또는 Show alert를 클릭했을 때의 상태인 3을 보여줄까요?

---
스포일러가 등장합니다.

---
[스스로 테스트해보세요!](https://codesandbox.io/s/w2wxl3yo0l)

만약 동작이 이해되지 않는다면, 좀 더 실용적인 예시를 상상해보세요: 현재 받는 사람 ID가 상태에 있고, 전송 버튼이 있는 채팅 앱. [이 글](https://overreacted.io/how-are-function-components-different-from-classes/)은 올바른 답이 3인 이유를 깊게 탐구합니다.

alert는 버튼을 클릭했을 때의 상태를 "캡처"할 것입니다.

(다른 동작을 수행하는 방법들도 있지만 지금은 일반적인 상황에 집중하겠습니다. 멘탈 모델을 형성할 때, 선택할 수 있는 탈출구와 "최소 저항 경로"를 구분하는 것이 중요합니다.)

그런데 어떻게 동작하는 걸까요?

우리는 `count` 값이 특정 함수가 호출될 때의 상수라는 이야기를 해왔습니다. 이것을 강조하는 것은 중요합니다. ㅡ 우리의 함수는 많이 호출되지만(렌더 당 한 번), 각각의 렌더 시 `count` 값은 상수이고 특정한 값(그 렌더의 상태)으로 설정된다는 것

이것은 리액트에만 특별하게 있는 것이 아닙니다. 일반 함수도 비슷한 방법으로 동작합니다.

```js
function sayHi(person) {
  const name = person.name;
  setTimeout(() => {
    alert('Hello, ' + name);
  }, 3000);
}

let someone = {name: 'Dan'};
sayHi(someone);

someone = {name: 'Yuzhi'};
sayHi(someone);

someone = {name: 'Dominic'};
sayHi(someone);
```
[이 예시](https://codesandbox.io/s/mm6ww11lk8)에서 외부의 `someone` 값은 몇 번 재할당됩니다(리액트처럼, 현재 컴포넌트 상태는 변할 수 있습니다). 그러나, `sayHi` 안에는, 특정 호출로부터 얻은 `person`으로 구성된 지역 `name` 상수가 있습니다. 그 상수는 지역 변수이기 때문에 각 호출 간 독립적입니다! 그 결과로, timeout이 호출 됐을 때, 각각의 alert는 그 때의 `name`을 "기억"합니다.

이것은 클릭했을 때 우리의 이벤트 핸들러가 어떻게 `count`를 캡처하는지를 설명합니다. 만약 우리가 같은 원리를 적용한다면, 각 렌더는 각 `count`를 보는 것입니다:
```js
// 첫 번째 렌더
function Counter() {
  const count = 0; // useState()에 의해 반환
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}

// 클릭 이후, 함수가 다시 호출됨
function Counter() {
  const count = 1; // useState()에 의해 반환
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}

// 또 다른 클릭 이후, 함수가 다시 호출됨
function Counter() {
  const count = 2; // useState()에 의해 반환
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  // ...
}
```
그래서 효과적으로, 각 렌더는 각각의 `handleAlertClick` 버전을 반환합니다. 각 버전들은 각각의 `count`를 "기억"합니다.
```js
// 첫 번째 렌더
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 0);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // 내부에 0을 기억함.
  // ...
}

// 클릭 이후, 함수가 다시 호출됨
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 1);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // 내부에 1을 기억함.
  // ...
}

// 또 다른 클릭 이후, 함수가 다시 호출됨
function Counter() {
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + 2);
    }, 3000);
  }
  // ...
  <button onClick={handleAlertClick} /> // 내부에 2를 기억함.
  // ...
}
```

이것이 [데모](https://codesandbox.io/s/w2wxl3yo0l)에서 이벤트 핸들러가 특정한 렌더에 속해 있던, 그리고 클릭 했을 때, 그 렌더로부터 얻은 `counter` 상태를 계속해서 사용하는 이유입니다. 

어떤 특정한 렌더 내부에서, props와 state는 영원히 같은 값으로 남아있습니다. 그러나 props와 state가 렌더 사이에 독립적이라면, 그들이 사용하는 값도 그렇습니다(이벤트 핸들러를 포함해서). 그들은 또한 특정한 렌더에 속해있습니다. 그래서 이벤트 핸들러 내부의 비동기 함수도 같은 `count` 값을 바라볼 것입니다.

참고: 위에서 `handleAlertClick` 함수 바로 오른쪽에 `count` 값을 표시해두었습니다. 이 방법은 `count`가 특정 렌더 내에서 변화할 수 없기 때문에 안전합니다. 값은 `const` 처럼 선언되었고, 그것은 number입니다. 상태 변경을 피하는 것만 동의하면, 객체와 같은 다른 값처럼 생각해도 안전합니다. 객체를 변경하는 것 대신에 새롭게 생성된 객체로 `setSomething(newObj)`를 호출해도 괜찮습니다. 이전 렌더에 속한 상태는 변경되지 않기 때문이죠.