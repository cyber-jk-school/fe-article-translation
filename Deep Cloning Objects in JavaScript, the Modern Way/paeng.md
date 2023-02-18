# [번역] JavaScript Deep Cloning 객체 최신 방법

원문: https://www.builder.io/blog/structured-clone

![1](https://user-images.githubusercontent.com/53526987/219279557-d2258315-5a59-42df-8c85-e039710ea3ab.png)

JavaScript에 객체의 깊은 복사를 하는 기본적인 방법이 있다는 것을 알고 있었습니까?

맞습니다, `structuredClone` 함수는 JavaScript 런타임에 내장되어 있습니다.:

```js
const calendarEvent = {
  title: 'Builder.io Conf',
  date: new Date(123),
  attendees: ['Steve'],
};

// 😍
const copied = structuredClone(calendarEvent);
```

위의 예시에서 객체 뿐만 아니라 중첩 배열, 심지어 Date 객체까지 복사된다는 것을 알았습니까?

그리고 정확하게 모두 작동합니다:

```js
copied.attendees; // ["Steve"]
copied.date; // Date: Wed Dec 31 1969 16:00:00
cocalendarEvent.attendees === copied.attendees; // false
```

`structuredClone`은 위의 상황뿐만 아니라 추가적인 것을 할 수 있습니다.:

- 무한 중첩 객체와 배열 복제
- 복제 순환 참조
- `Date`, `Set`, `Map`, `Error`, `RegExp`, `ArrayBuffer`, `Blob`, `File`, `ImageData`, [그 이상](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#supported_types)과 같은 다양한 JavaScript types 복제
- 모든 [transferable objects](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Transferable_objects) 변환

예를 들어, 이러한 미친 코드도 정확히 동작합니다:

```js
const kitchenSink = {
  set: new Set([1, 3, 3]),
  map: new Map([[1, 2]]),
  regex: /foo/,
  deep: { array: [new File(someBlobData, 'file.txt')] },
  error: new Error('Hello!'),
};
kitchenSink.circular = kitchenSink;

// ✅ All good, fully and deeply copied!
const clonedSink = structuredClone(kitchenSink);
```

## 왜 객체 spread가 아닙니까?

깊은 복사에 대해 이야기를 하고 있는 것이 중요합니다. 만약 중첩된 객체나 배열들을 복사하지 않은 얕은 복사가 필요하다면 객체 spread를 사용하면 됩니다.:

```js
const simpleEvent = {
  title: 'Builder.io Conf',
};
// ✅ no problem, there are no nested objects or arrays
const shallowCopy = { ...calendarEvent };
```

또는 이중 하나라도 원하는 경우

```js
const shallowCopy = Object.assign({}, simpleEvent);
const shallowCopy = Object.create(simpleEvent);
```

그러나 중첩된 것들을 하자마자 문제가 발생합니다.

```js
const calendarEvent = {
  title: 'Builder.io Conf',
  date: new Date(123),
  attendees: ['Steve'],
};

const shallowCopy = { ...calendarEvent };

// 🚩 oops - we just added "Bob" to both the copy *and* the original event
shallowCopy.attendees.push('Bob');

// 🚩 oops - we just updated the date for the copy *and* original event
shallowCopy.date.setTime(456);
//
```

이 객체를 완전히 복사할 수 없는 것을 볼 수 있습니다.

중첩된 date와 배열은 여전히 둘 사이를 공유하는 참조이므로 calendar event 객체만 업데이트한다는 것은 큰 문제를 야기합니다.

## 왜 `JSON.parse(JSON.stringify(x))`가 아닙니까?

이것은 트릭 맞습니다. 이것은 훌륭하고 놀라울 정도의 성능을 가지고 있지만, 몇몇의 단점들을 `structuredClone`이 해결할 수 있습니다.

예를 들어보겠습니다.:

```js
const calendarEvent = {
  title: 'Builder.io Conf',
  date: new Date(123),
  attendees: ['Steve'],
};

// 🚩 JSON.stringify converted the `date` to a string
const problematicCopy = JSON.parse(JSON.stringify(calendarEvent));
```

`problematicCopy`를 얻을 수 있습니다.:

```js
{
  title: "Builder.io Conf",
  date: "1970-01-01T00:00:00.123Z"
  attendees: ["Steve"]
}
```

이것을 원한 것이 아니었습니다.! `date`는 문자열이 아니라 `Date` 객체 여야만 합니다.

이러한 상황은 `JSON.stringify`는 기본 객체, 배열, 원시값만 다룰 수 있기 때문입니다. 이외 다른 type은 예측하기 힘든 방식으로 처리될 수 있습니다. 예를 들어, Dates를 문자열로 `Set`은 단순히 `{}`로 변환됩니다.

`JSON.stringify`는 `undefined` 또는 함수 같은 특정 항목들을 완전히 무시합니다.

예를 들어, 이러한 방법으로 `kitchenSink`를 복사한다면:

```js
const kitchenSink = {
  set: new Set([1, 3, 3]),
  map: new Map([[1, 2]]),
  regex: /foo/,
  deep: { array: [new File(someBlobData, 'file.txt')] },
  error: new Error('Hello!'),
};

const veryProblematicCopy = JSON.parse(JSON.stringify(kitchenSink));
```

다음과 같이 얻습니다.:

```js
{
  "set": {},
  "map": {},
  "regex": {},
  "deep": {
    "array": [
      {}
    ]
  },
  "error": {},
}
```

이우!

`JSON.stringify`는 이러한 오류 중 하나가 발생하면 간단한 에러를 발생하기 때문에 본래 가지고 있던 순환 참조를 제거해야 합니다.

그래서 이 방법은 우리 요구사항이 할 수 있는 일에 맞으면 훌륭한 방법일 수 있지만 `structuredClone`(aka 위에서 실패한 모든 일)로 하는 많은 것들을 이 방법으로는 못할 수 있습니다.

## 왜 `_.cloneDeep`가 아닙니까?

지금까지, 이 문제를 해결하는 가장 보편적인 방법은 Lodash's `cloneDeep`을 사용하는 것이었습니다.

실제로 이것은 예상대로 동작합니다.:

```js
import cloneDeep from 'lodash/cloneDeep';

const calendarEvent = {
  title: 'Builder.io Conf',
  date: new Date(123),
  attendees: ['Steve'],
};

// ✅ All good!
const clonedEvent = structuredClone(calendarEvent);
const LodashClonedEvent = cloneDeep(calendarEvent);
```

그러나 하나의 주의사항이 있습니다. 저의 IDE에서 import한 kb 비용을 출력해주는 [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost) 확장자에 따르면 이 함수는 전체 17.4kb minified (5.3kb gzipped)로 제공합니다.:

![image2](https://user-images.githubusercontent.com/53526987/219849612-2aa00230-3cd8-4733-9c1c-095f724abf04.png)

단지 이 함수만 import한다고 가정해봅시다. 만약 tree shaking이 항상 원하는 대로 동작하지 않는 것을 알지 못하고 더 일반적인 방법으로 import한다면 이 하나의 함수에 최대 [25kb](https://bundlephobia.com/package/lodash@4.17.21)를 사용하게 될 것입니다. 😱

![image3](https://user-images.githubusercontent.com/53526987/219850090-18da1e14-329f-46f2-bbbd-614fa2bcb929.png)

이것이 누가에게도 세상의 종말은 아니지만, 브라우저에 이미 `structuredClone`이 내장되어 있는 경우에는 필요하지 않습니다.

## `structuredClone`에서 복제할 수 없는 것

### 함수들은 복제할 수 없습니다.

`DataCloneError` 예외가 발생합니다.:

```js
// 🚩 Error!
structuredClone({ fn: () => {} });
```

### DOM nodes

`DataCloneError` 예외가 발생합니다.:

```js
// 🚩 Error!
structuredClone({ el: document.body });
```

### Property descriptors, setters, and getters

게다가 유사한 metadata와 유사한 기능은 복제되지 않습니다.

예를 들어, getter에서 결과 값은 복제되지만 getter 함수 자체(또는 다른 property metadata)는 복제되지 않습니다.:

```js
structuredClone({
  get foo() {
    return 'bar';
  },
});
// Becomes: { foo: 'bar' }
```

### Object prototypes(객체 속성들)

prototype chain은 이어나가지거나 복제되지 않습니다. 따라서 `MyClass`의 인스턴스를 복제하면, 이 복제된 객체는 더이상 이 클래스의 인스턴스가 아닙니다. (그러나 이 클래스의 모든 properties들은 복제됩니다.)

```js
class MyClass {
  foo = 'bar';
  myMethod() {
    /* ... */
  }
}
const myClass = new MyClass();

const cloned = structuredClone(myClass);
// Becomes: { foo: 'bar' }

cloned instanceof myClass; // false
```

### 지원되는 types의 전체 목록

단순하게, 아래 목록에 없는 것들은 복제할 수 없습니다.:

[JS Build-ins](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#javascript_types)

`Array`, `ArrayBuffer`, `Boolean`, `DataView`, `Date`, `Error`types (아래에 구체적으로 나열된 것들), `Map`, `Object` 그러나 평범한 객체만 (예: 객체 리터럴), [Primitive types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#primitive_values), `symbol`을 제외한 원시유형(`number`, `string`, `null`, `undefined`, `boolean`, `BigInt`), `RegExp`, `Set`, `TypedArray`

Error Types

`Error`, `EvalError`, `RangeError`, `ReferenceError`, `SyntaxError`, `TypeError`, `URIError`

[Web/API types](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#webapi_types)

`AudioData`, `Blob`, `CryptoKey`, `DOMException`, `DOMMatrix`, `DOMMatrixReadOnly`, `DOMPoint`, `DomQuad`, `File`, `FileList`, `FileSystemDirectoryHandle`, `FileSystemFileHandle`, `FileSystemHandle`, `ImageBitmap`, `ImageData`, `RTCCertificate`, `VideoFrame`

## 브라우저 및 런타임 지원

여기에 가장 중요한 부분이 있습니다. `structuredClone`은 모든 주요 브라우저들과 심지어 Node.js및 Deno에서도 지원이 됩니다.

지원이 더 제한적인 웹 작업자들에 대한 주의 사항에 유의해야 됩니다.

![image4](https://user-images.githubusercontent.com/53526987/219856782-01851be7-5e15-4986-8994-7f75df1e6253.png)

출처: [MDN](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone)

## 결말

오랜 시간이 흘렀지만 마침내 우리는 JavaScript에서 객체를 깊은 복사할 수 있는 `structuredClone`을 가지게 되었습니다. 고마워, [Surma](https://web.dev/structured-clone/).
