# [번역] JavaScript 가비지 컬렉터 실험

원문: https://dev.to/codux/experiments-with-the-javascript-garbage-collector-2ae3

\#javascript \#performance \#webdeb \#debugging

웹 어플리케이션에서 메모리 누수는 디버깅하기 어려운 것으로 악명이 높습니다. 가비지 컬렉터가 어떻게 수집가능한 객체와 수집 불가능한 객체를 결정하는지 이해하면 이를 피하는데 도움이 됩니다. 이 아티클에서 해당 동작이 너를 놀라게 할 수 있는 몇 가지 시나리오를 살펴보겠습니다.

가비지 컬렉터의 기초에 익숙하지 않는다면, Lin Clark의 [A Crash Course in Memory Management](https://hacks.mozilla.org/2017/06/a-crash-course-in-memory-management/) 또는
MDN의 [Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)은 좋은 시작입니다. 계속 읽기 나가기 전에 이것들 중 하나를 읽는 것이 좋습니다.

## 객체 처분 감지

최근, 객체가 가비지에 수집되는 시기를 프로그래밍 방식으로 접근가능한 JavaScript에서 제공하는 [FinalizationRegistry](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry) 클래스를 배우게 되었습니다. 이것은 주요 웹 브라우저와 Node.js에서 사용가능합니다.

기본적인 사용 예시:

```js
const registry = new FinalizationRegistry((message) => console.log(message));

function example() {
  const x = {};
  registry.register(x, 'x has been collected');
}

example();

// Some time later: "x has been collected"
```

`example()` 함수가 반환될 때, `x`가 참조하는 객체는 더 이상 접근할 수 없어 삭제할 수 있습니다.

하지만, 대부분, 즉시 폐기되지 않습니다. 엔진은 더 중요한 작업을 먼저 처리하거나 더 많은 개체에 도달할 수 없을 때까지 기다린 다음 대량으로 처리하도록 되어있습니다. 그러나 DevTools ➵ Memory 탭에서 작은 휴지통 아이콘을 선택하여 강제로 가비지 컬렉터를 할 수 있습니다. Node.js는 휴지통 아이콘은 없지만, `--expose-gc` 플래그로 시작할 때 `gc()` 전역 함수를 제공합니다.

![7](https://github.com/GeonWooPaeng/GeonWooPaeng/assets/53526987/e223b223-f9e9-4ae9-b115-d14082e9ce81)

도구 모음에 있는 `FinalizationRegistry`로, 가비지 컬렉터가 어떻게 동작할지 확신하지 않은 몇몇의 시나리오를 검사하기로 결정했습니다.
밑의 예시들을 보고 어떻게 동작할지 예측해보시기 바랍니다.

## Example 1: 중첩된 객체들

```js
const registry = new FinalizationRegistry((message) => console.log(message));

function example() {
  const x = {};
  const y = {};
  const z = { x, y };

  registry.register(x, 'x has been collected');
  registry.register(y, 'y has been collected');
  registry.register(z, 'z has been collected');

  globalThis.temp = x;
}

example();
```

변수 `x`는 `example()` 함수가 반환된 후에 더 이상 존재하지 않고
`x`를 참조한 객체는 여전히 변수 `globalThis.temp`에 의해 엮여있습니다. 반면에 `z`와 `y`는 더 이상 전역 객체 또는 실행 스택에 도달할 수 없어 수집됩니다. `globalThis.temp = undefined`를 실행한다면, 이전에 `x`로 알려진 객체도 수집됩니다. 놀랄일이 아닙니다.

## Example 2: Closures

```js
const registry = new FinalizationRegistry((message) => console.log(message));

function example() {
  const x = {};
  const y = {};
  const z = { x, y };

  registry.register(x, 'x has been collected');
  registry.register(y, 'y has been collected');
  registry.register(z, 'z has been collected');

  globalThis.temp = () => z.x;
}

example();
```

`globalThis.temp()`를 불러와서 `x`에 도달할 수 있습니다. 더 이상 `z` 또는 `y`에 도달할 수 없습니다. 그러나 더 이상 접근할 수 없음에도 불구하고 `z`와 `y`가 수집되지 않습니다.

가능한 이론은 `z.x`가 속성 조회이므로, 엔진이 조희를 `x`에 대한 직접적인 참조로 대체할 수 있는지 알지 못합니다. 예를 들어 `x`가 getter이면 어떻합니까? 따라서 엔진은 `z`에 대한 참조를 유지하고, 결과적으로 `y`에 대한 참조를 유지해야 합니다. 이 이론을 테스트하기 위해 예시를 수정하겠습니다.: `globalThis.temp = () => { z; };` `z`에 도달하는 방법은 명백히 없지만, 여전히 수집되지 않습니다.

내 생각에 가비지 컬렉터가 `z`가 temp에 할당된 클로저의 [lexical scope](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#lexical_scoping)에 있다는 사실에만 주의를 기울이고 그 이상은 보지 않는다는 것 입니다. 전체 객체 그래프를 순회하고 여전히 "살아있는" 객체를 마킹하는 것은 빠르게 하는 성능의 중요한 작업입니다. 가비지 컬렉터는 이론적으로 `z`가 사용되지 않는 것을 알아낼 수 있지만, 비싼 비용을 지불합니다. 그리고 특별히 유용하지 않은데, 코드에서는 일반적으로 *chilling*한 변수를 포함하지 않기 때문입니다.

## Example 3: Eval

```js
const registry = new FinalizationRegistry((message) => console.log(message));

function example() {
  const x = {};

  registry.register(x, 'x has been collected');

  globalThis.temp = (string) => eval(string);
}

example();
```

`temp('x')`를 호출하여 global scope에서 `x`에 여전히 접근할 수 있습니다. 엔진은 `eval`의 lexical scope 내에서 객체를 안전하게 수집할 수 없습니다. 그리고 eval이 받는 인수를 분석조차 하지 않습니다. `globalThis.temp = () => eval(1)`과 같이 무지하게 사용하는 것은 가비지 수집을 방해합니다.

eval이 별칭 뒤에 숨어 있다면 어떻게 될까? (e.g. `globalThis.exec = eval`) 또는 명시적으로 언급하지 않고 사용하면 어떻게 됩니까? (e.g. :)

```js
console.log.constructor('alert(1)')(); // opens an alert box
```

이것은 모든 함수 호출이 안전하게 가비지 수집을 하지 못하는 것을 의미합니까? 다행이도 아닙니다. JavaScript는 [직접 eval과 간접 eval](https://2ality.com/2014/01/eval.html)을 구분합니다.
`eval(string)`을 직접 호출할때만 lexical scope내에서 코드가 실행됩니다. `eval?.(string)`과 같은 덜 직접적인 코드는, global scope내에서 코드를 실행하고 둘러 쌓여진 함수 변수에 접근할 수 없습니다.

## Example 4: DOM Elements

```js
const registry = new FinalizationRegistry((message) => console.log(message));

function example() {
  const x = document.createElement('div');
  const y = document.createElement('div');
  const z = document.createElement('div');

  z.append(x);
  z.append(y);

  registry.register(x, 'x has been collected');
  registry.register(y, 'y has been collected');
  registry.register(z, 'z has been collected');

  globalThis.temp = x;
}

example();
```

이 예제는 첫번째와 유사하지만 일반 객체 대신에 DOM 요소를 사용합니다. 일반 객체와 달리 DOM 요소에는 부모 및 형제에 대한 링크가 있습니다. `temp.parentElement`를 통해 `z`에 접근하고 `temp.nextSibling`을 통해 `y`에 접근합니다. 따라서 세가지 요소 모두 살아있습니다.

`temp.remove()`를 실행하면 `x`가 부모에서 분리되기 때문에 `y`와 `z`가 수집됩니다. 그러나 `x`는 여전히 `temp`에 의해 참조되기 때문에 수집되지 않습니다.

## Example 5: Promises

> 경고: 비동기 작업과 promises와 관련된 시나리오를 보여주는 복잡한 예제입니다. 건너 띄고 아래 요약으로 이동해도 됩니다.

resolved 되거나 rejected되지 않은 promises는 어떻게 됩니까? `.then`의 전체 체인이 연결된 상태로 계속 메모리에 떠 있습니까?

현실적인 예시로, React 프로젝트의 일반적인 [anti-pattern](https://legacy.reactjs.org/blog/2015/12/16/ismounted-antipattern.html) 입니다.

```js
function MyComponent() {
  const isMounted = useIsMounted();
  const [status, setStatus] = useState('');

  useEffect(async () => {
    await asyncOperation();
    if (isMounted()) {
      setStatus('Great success');
    }
  }, []);

  return <div>{status}</div>;
}
```

`asyncOperation()`이 해결되지 않으면, effect 함수는 어떻게 됩니까? 컴포넌트가 마운트 해제된 후에도 계속 promise를 기다립니까?
`isMounted`와 `setStatus`를 활성화 상태로 유지합니까?

예제를 React 없이 기본적인 형태로 축소하겠습니다.:

```js
const registry = new FinalizationRegistry((message) => console.log(message));

function asyncOperation() {
  return new Promise((resolve, reject) => {
    /* never settles */
  });
}

function example() {
  const x = {};
  registry.register(x, 'x has been collected');
  asyncOperation().then(() => console.log(x));
}

example();
```

이전에 우리는 가비지 컬렉터가 어떤 종류의 정교한 분석을 하지 않고, 객체에서 객체로 포인터를 따라가서 객체의 "활성"을 결정한다는 것을 확인했습니다. 따라서 이 경우 `x`가 수집된다는 것은 놀라운 일이 될 수 있습니다!

Promise `resolve`에 대한 참조를 여전히 가지고 있을 때 이 예제가 어떻게 보이는지 확인하겠습니다. 실제 시나리오에서는 `setTimeout()` 또는 `fetch()`가 될 수 있습니다.

```js
const registry = new FinalizationRegistry((message) => console.log(message));

function asyncOperation() {
  return new Promise((resolve) => {
    globalThis.temp = resolve;
  });
}

function example() {
  const x = {};
  registry.register(x, 'x has been collected');
  asyncOperation().then(() => console.log(x));
}

example();
```

`globalThis`는 `temp`를 유지하고, `resolve`를 유지하고, `.then(...)` 콜백을 유지하고, `x`를 유지합니다.
`globalThis.temp = undefined`를 실행하자마자, `x`를 수집합니다. 그래도, promise 자체에 대한 참조를 저장해도 `x`가 수집되는 것을 막을 수 없습니다.

React 예제로 돌아가서: Promise `resolve`에 대한 참조를 하고 있는 경우 해당 효과와 lexical scope의 모든 항목은 컴포넌트가 마운트 해제된 후에도 활성 상태로 유지됩니다. promise가 확정되거나 가비지 컬렉터가 더이상 promise의 `resolve` 와 `reject`의 경로를 추적할 수 없을 때 수집됩니다.

## 결론

이 article에서는 `FinalizationRegistry`를 살펴보고 어떻게 객체가 수집되는 시기를 감지하는지 알아보았습니다. 때때로 가비지 컬렉터가 메모리를 회수하는 것이 안전함에도 불구하고 메모리 회수를 할 수 없는 경우도 보았습니다. 이것은 할 수 있는 것과 할 수 없는 것을 인식하는데 도움이 되는 이유입니다.

다른 JavaScript 엔진과 동일한 엔진의 다른 버전의 가비지 컬렉터의 실행은 크게 다를 수 있으며, 외부에서 관찰하는 것에서 차이가 있을 수 있습니다.

실제로, [ECMAScript 사양](https://tc39.es/ecma262/#sec-managing-memory)은 특정 동작을 규정하는 것을 고사하고 가비지 컬렉터가 있는 구현을 요구하지 않습니다.

그러나 위의 모든 예제는 V8 (Chrome), JavaScriptCore (Safari), Gecko (Firefox)에서 동일하게 작동합니다.
