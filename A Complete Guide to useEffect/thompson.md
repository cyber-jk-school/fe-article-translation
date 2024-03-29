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

### 질문: 어떻게 `useEffect`를 사용해서 `componentDidMount`를 따라할 수 있을까?

당신이 `useEffect(fn, [])`을 사용할 수 있지만, 이것은 완벽하게 같지는 않습니다. `componentDidMount`와 다르게, useEffect는 props와 state를 캡처할 것입니다. 그래서 callbacks 내부에서도 초기 props와 state를 확인할 수 있을 것입니다. 만약 최신의 값을 보고 싶다면, ref를 사용할 수 있습니다. 그러나 보통 그럴 필요 없이 코드를 설계할 수 있는 간단한 방법이 있습니다. effect의 개념이 `componentDidMount`나 다른 생명 주기와 다르다는 것을 기억하고, 그것들과 완벽하게 동일한 것을 찾으려고 하면 오히려 혼란스러울 수 있다는 사실을 기억하세요. 생산성을 위해, "effects로 생각"하고, 그들의 멘탈 모델은 생명 주기 이벤트에 응답하는 것보다 동기화를 구현하는 것에 더 가깝습니다.

### 질문: `useEffect` 내에서 데이터를 어떻게 올바르게 가져올 수 있을까? `[]`는 뭘까?

[이 글](https://www.robinwieruch.de/react-hooks-fetch-data/)은 `useEffect`에서 데이터를 불러오는 좋은 입문서입니다. 끝까지 읽어보세요! 지금 읽는 글처럼 길지 않습니다. `[]`는 이펙트가 리액트 데이터 흐름에 관여하는 어떤 값도 사용하지 않는다는 것을 의미합니다. 그리고 이것은 한번만 적용되어도 안전하다는 이유입니다. 이것은 또한 값이 실제로 사용될 때, 흔한 버그의 원인이기도 합니다. 당신은 몇 가지 전략(주로 `useReducer`와 `useCallback`)을 공부할 필요가 있습니다. 이 전략은 의존성을 생략하는 것 대신에 의존성을 필요로 하는 상황을 제거할 수 있습니다.

### 질문: `useEffect`의 의존성으로 함수를 지정해야 할까?

추천하는 방법은 props나 state가 필요 없는 함수를 컴포넌트 바깥으로 끌어올리고, effect 내부에서 사용하는 것은 effect 내부에서 정의하는 것입니다. 만약 effect가 끝나고 렌더 스코프에서 함수를 사용한다면(props에서 받아온 함수 포함), 그것들을 정의할 때 `useCallback`을 사용해 감싸고 과정을 반복하세요. 왜 이게 중요할까요? 함수는 props와 state로 부터 온 값으로 볼 수 있습니다. 그래서 그들은 데이터 흐름에 포함되어 있습니다. 우리의 FAQ에 [더 자세한 답변](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)이 있습니다.

### 질문: 가끔씩 무한 refetch 루프에 빠지는 이유는 뭘까?

이것은 두 번째 의존성 인자 없이 effect 내부에서 데이터 페칭을 할 때 발생합니다. 의존성 배열이 없으면, effect는 매 렌더 이후에 실행됩니다 ㅡ 그리고 상태를 설정하는 것은 effect를 다시 실행하게 됩니다. 무한 루프는 의존성 배열에 항상 바뀌는 값을 명시할 경우에 발생할 수도 있습니다. 당신은 의존성 배열의 값을 하나씩 지우면서 확인할 수 있습니다. 그러나, 사용하는 의존성을 지우는 것은 (또는 맹목적으로 `[]`를 명시하는 것) 보통 잘못된 수정 방법입니다. 대신에, 문제가 발생하는 부분을 수정하세요. 예를 들어, 함수가 이 문제의 원인일 수 있는데, effect 안에 넣고, 밖으로 꺼내거나 `useCallback`으로 감싸는 게 도움이 될 수 있습니다. 객체를 재생성하는 걸 피하기 위해서, `useMemo`를 비슷한 목적으로 사용할 수 있습니다.

### 질문: 가끔 `useEffect` 내에서 예전 상태나 prop 값을 가져오는 이유는 뭘까?

effect는 항상 그것들이 정의된 렌더에서 props와 state를 "봅니다". 그것은 버그를 막는데 도움이 되지만, 몇몇 경우에는 짜증이 날 수 있습니다. 그런 경우에, mutable ref에 값을 명시적으로 저장할 수 있습니다. (링크된 글의 마지막 부분에서 설명하고 있습니다.) 만약 당신이 이전 렌더의 props와 state 값을 보고 있고 예상하지 않았다면, 아마 몇 가지 의존성을 깜빡했을 수 있습니다. 의존성 확인하는 연습을 위해 린트 규칙을 사용해보세요. 몇 일이 지나면, 그것은 자연스러워질 것입니다. 우리의 FAQ에서 [이 답변](https://reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function)도 확인해보세요.

위 요약이 도움이 되길 바랍니다! 그렇지 않으면, 가봅시다.
