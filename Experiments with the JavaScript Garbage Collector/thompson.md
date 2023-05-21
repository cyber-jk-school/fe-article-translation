# 자바스크립트 가비지 콜렉터 실험

웹 애플리케이션에서의 메모리 누수는 일반적이고 디버깅이 악명 높을 정도로 어렵습니다. 메모리 누수를 피하고 싶다면, 어떤 객체가 수집될 수 있는지 이해하는 것이 도움이 됩니다. 이 글에서는 당신을 놀라게 할 수 있는 몇 가지의 가비지 콜렉터 동작 시나리오를 살펴 볼 것입니다.

만약 가비지 콜렉터의 기본에 익숙하지 않다면, Lin Clark의 [A Crash Course in Memory Management](https://hacks.mozilla.org/2017/06/a-crash-course-in-memory-management/) 또는 MDN의 [Memory Management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)가 좋은 시작점이 될 수 있습니다. 이 글을 계속 읽기 전에 위의 글을 읽는 걸 고려해보세요.

## 객체 폐기 감지
최근에 자바스크립트에서 객체가 가비지 콜렉터에 의해 수집됐을 때를 programmatically 감지하도록 하는 [FinalizationRegistry](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry)라는 클래스를 제공한다는 것을 배웠습니다. 모든 주요 웹 브라우저와 Node.js에서 사용 가능합니다.

기본 사용 예시입니다.
```javascript
const registry = new FinalizationRegistry(message => console.log(message));

function example() {
    const x = {};
    registry.register(x, 'x has been collected');
}

example();

// Some time later: "x has been collected"
```

`example()` 함수가 반환될 때, `x`가 참조하는 객체는 더 이상 도달할 수 없고 폐기될 수 있습니다.

그렇지만, 대부분의 경우 객체가 즉시 폐기되진 않습니다. 자바스크립트 엔진은 더 중요한 작업을 먼저 처리하거나, 더 많은 객체가 도달할 수 없는 상태가 될 때까지 기다리도록 결정할 수 있고 그 다음 객체들을 대량으로 폐기합니다. 그러나 개발자 도구 > 메모리 탭에서 작은 휴지통 아이콘을 클릭해 가비지 콜렉션을 강제할 수 있습니다. Node.js는 휴지통 아이콘이 없지만, `--expoes-gc` 플래그와 실행했을 때 전역 `gc()` 함수를 제공합니다. 
![collect-garbage](https://res.cloudinary.com/practicaldev/image/fetch/s--FkDDfyAF--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l23os4ss0asqatwrmvq2.png)
`FinalizationRegistry`를 알게 되었을 때, 가비지 콜렉터가 어떻게 동작할 지 확실할 수 없는 몇 가지 시나리오를 조사하기로 결정했습니다. 당신이 아래의 예시를 보고 그들이 어떻게 동작할 지 예측할 수 있도록 도와드리겠습니다.

## 예시 1: 중첩된 객체
```javascript
const registry = new FinalizationRegistry(message => console.log(message));

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

여기에서 변수 `x`는 `example()` 함수가 반환된 후 더 이상 존재하지 않지만, `x`가 참조하는 객체는 `globalThis.temp` 변수가 잡고 있습니다. 반면에 `z`와 `y`는 전역 객체 또는 실행 컨텍스트 스택에서 접근할 수 없기 때문에 수집될 것입니다. 만약 `globalThis.temp = undefined`를 실행하면, `x`가 이전에 알고 있던 객체는 수집될 것입니다. 놀라운 게 없죠.

## 예시 2: 클로저
```javascript
const registry = new FinalizationRegistry(message => console.log(message));

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
위 예시에서 `globalThis.temp()`를 호출함으로써 `x`에 접근할 수 있습니다. `z` 또는 `y`에는 더 이상 접근할 수 없습니다. 그러나 `z`와 `y`에 접근할 수 없음에도 불구하고, `z`와 `y`는 수집되지 않습니다. 

가능한 이론은 `z.x`가 속성 조회이기 때문입니다. 자바스크립트 엔진은 조회 대신 직접 `x`를 참조할 수 있는지 알 수 없습니다. 예를 들어, `x`가 getter라면 어떨까요. 엔진은 강제로 `z`에 대한 참조를 유지해야 하고, 결과적으로 `y`도 그렇습니다. 이 이론을 테스트하기 위해, 예시를 변경해봅시다: `globalTest.temp = () => { z; };`. 이제 명확하게 `z`에 접근할 방법이 없지만, 여전히 수집되지 않습니다.

무슨일이 일어나는지 제가 생각한 건 가비지 콜렉터는 오직 temp에 할당된 클로저의 렉시컬 스코프 내부에 있는 `z`라는 사실에만 관심이 있고, 그 이상으로는 관심이 없다는 것입니다. 전체 객체 트리를 탐색하고 아직까지 살아있는 객체인지 확인하는 것은 빠른 속도가 필요한 성능이 중요한 작업입니다. 비록 가비지 콜렉터는 이론적으로 `z`가 사용되지 않는다고 판단할 수 있지만, 그것은 비용이 많이 듭니다. 그리고 당신의 코드에는 일반적으로 위에서 처럼 놀고 있는 변수가 포함되어 있지 않기 때문에 특히 유용하지는 않습니다.

## 예시 3: Eval
```javascript
const registry = new FinalizationRegistry(message => console.log(message));

function example() {
    const x = {};

    registry.register(x, 'x has been collected');

    globalThis.temp = (string) => eval(string);
}

example();
```
여기서는 전역 스코프에서 `temp('x')`를 호출함으로써 `x`에 접근할 수 있습니다. 자바스크립트 엔진은 `eval`의 렉시컬 스코프 내부에 있는 어떤 객체도 안전하게 수집할 수 없습니다. 그리고 eval이 받는 인자가 무엇인지 분석하려고 하지 않습니다. 심지어 `globalThis.temp = () => eval(1)`과 같은 무고한 것도 가비지 콜렉션이 일어나지 않습니다.

만약 eval이 `globalthis.exec = eval`과 같이 별칭으로 숨어있으면 어떨까요? 또는 명시적으로 언급되지 않고 사용되면 어떨까요? 예를 들면 
```javascript
console.log.constructor('alert(1)')(); // opens an alert box
```

모든 함수 호출이 대상이고, 안전하게 수집될 수 없는 것일까요? 다행히도 그렇지 않습니다. 자바스크립트는 [직접과 간접 eval](https://2ality.com/2014/01/eval.html)을 구분합니다. 오직 `eval(string)` 으로 직접 호출했을 때만 현재 렉시컬 스코프의 코드를 실행합니다. 그러나 `eval?.(string)`과 같은 조금 직접적인 것은 전역 스코프에서 코드가 실행되고, 함수 내부의 변수에 접근할 수 없습니다.

## 예시 4: DOM 요소
```javascript
const registry = new FinalizationRegistry(message => console.log(message));

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
이 예시는 첫 번째 예시와 비슷하지만, 객체 대신 DOM 요소를 사용했습니다. 일반 객체와 다르게, DOM 요소는 부모 요소와 형제 요소에 연결되어 있습니다. `temp.parentElement`를 통해 `z`에 접근할 수 있고, `temp.nextSibling`을 통해 `y`에 접근할 수 있습니다. 그래서 세 요소 모두 살아 있게 됩니다.

만약 `temp.remove()`를 실행하면, `y`와 `z`는 `x`가 부모 요소에서 떨어지기 때문에 수집될 것입니다. 그러나 `x`는 `temp`를 통해 참조되기 때문에 수집되지 않습니다.

## 예시 5: Promise
> 주의: 이 예시는 더 복잡하고, 비동기 작업과 프로미스를 포함하는 시나리오를 보여주고 있습니다. 편하게 아래의 요약으로 넘어가도 됩니다.

resolve 또는 reject 되지 않는 프로미스는 어떤 일이 일어날까요? 프로미스에 달린 `.then` 체인과 함께 메모리에 남아있을까요?

실제 예시로, 리액트 프로젝트의 [안티 패턴](https://reactjs.org/blog/2015/12/16/ismounted-antipattern.html)이 있습니다.
```javascript
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
만약 `asyncOperation()`이 settled 되지 않으면, effect 함수에서는 어떤 일이 일어날까요? 컴포넌트가 언마운트 된 후에도 프로미스를 기다리고 있을까요? `isMounted`와 `setStatus`는 살아있을까요?

위 예시를 리액트가 필요하지 않은 더 기본적인 형태로 줄여봅시다.
```javascript
const registry = new FinalizationRegistry(message => console.log(message));

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
앞에서 우리는 가비지 콜렉터가 정교한 분석을 수행하지 않고 객체가 살아있는 지 확인하기 위해 객체의 참조를 타고 이동하지 않는다는 것을 확인했습니다. 그렇기 때문에 위 예시에서 `x`가 수집된다는 사실이 놀라울 수 있습니다!

위 예시에서 프로미스의 `resolve`를 계속해서 참조할 때 어떻게 되는지 살펴봅시다. 실제 시나리오에서 이는 `setTimeout()`이나 `fetch()`가 될 수 있습니다.
```javascript
const registry = new FinalizationRegistry(message => console.log(message));

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

여기에서 `globalThis`가 `temp`를 살아있게 하고, `temp`는 `resolve`를, `resolve`는 `.then(...)` 살아있게 하고, 이는 `x`를 살아있게 합니다. `globalThis.temp = undefined`를 실행하면, `x`는 수집됩니다. 그런데, 프로미스의 참조를 저장하는 것만으로는 `x`가 수집되는 것을 막을 수 없습니다. 

리액트 예시로 다시 돌아가 봅시다: 무언가 프로미스의 `resolve`를 참조하고 있다면, effect와 렉시컬 스코프 내의 모든 것들은 컴포넌트가 언마운트 된 후에도 살아있게 됩니다. 프로미스가 settled 됐을 때, 또는 가비지 콜렉터가 더 이상 프로미스의 `resolve`나 `reject`를 추적할 수 없을 때 수집됩니다. 

## 결론
이 글에서 우리는 `FinalizationRegistry`와 이 클래스가 객체가 수집될 때를 감지하기 위해 어떻게 사용하는 지를 살펴보았습니다. 또한 가비지 콜렉터가 안전하게 동작할 수 있을 때에도 메모리 회수할 수 없는 몇 가지 경우도 살펴보았습니다. 할 수 있는 일과 할 수 없는 일이 왜 그런 지 아는 것이 도움이 됩니다.

다른 자바스크립트 엔진, 그리고 다른 버전의 같은 엔진의 가비지 콜렉터가 완전히 다르게 동작할 수 있고, 외부에서 차이를 관찰할 수 있다는 사실을 확인하는 것은 가치가 있습니다. 

사실, [ECMAScript 명세](https://tc39.es/ecma262/#sec-managing-memory)는 가비지 콜렉터의 구현을 요구하지 않고, 특정 동작을 규정하지도 않습니다.

그러나, 위의 모든 예시는 V8 (Chrome), JavaScriptCore (Safari), 그리고 Gecko (Firefox) 에서 같은 동작을 하는 것을 검증했습니다.