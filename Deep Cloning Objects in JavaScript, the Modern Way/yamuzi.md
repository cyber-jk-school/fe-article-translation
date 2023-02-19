# Deep Cloning Objects in JavaScript, the Modern Way

<aside>
🎁 원문 링크 : [https://www.builder.io/blog/structured-clone](https://www.builder.io/blog/structured-clone)

</aside>

> Did you know, there's now a native way in JavaScript to do deep copies of objects?

혹시 자바스크립트에서의 객체 깊은 복사를 하는 native 방식이 있다는 것을 아시나요?

> That's right, this `structuredClone` function is built into the JavaScript runtime:

맞습니다, 이 structuredClone 함수는 자바스크립트 런타임에 내장되어 있습니다.

```jsx
const calendarEvent = {
  title: "Builder.io Conf",
  date: new Date(123),
  attendees: ["Steve"]
}

*// 😍*
const copied = structuredClone(calendarEvent)
```

> Did you notice in the example above we not only copied the object, but also the nested array, and even the Date object?

위의 예시에서 단지 객체를 복사한 것 뿐만 아닌, 중첩된 배열과, 심지어 Date 객체도 복사했다는 것을 알고 계셨나요?

> And all works precisely as expected:

그리고 모든 것은 예상한 대로 정확히 작동합니다.

```jsx
copied.attendees *// ["Steve"]*
copied.date *// Date: Wed Dec 31 1969 16:00:00*
cocalendarEvent.attendees === copied.attendees *// false*
```

> That’s right, `structuredClone` can not only do the above, but additionally:

맞습니다. structuredClone은 위의 작업들 뿐만 아닌, 더불어 다음과 같은 작업들도 수행할 수 있습니다.

> Clone infinitely nested objects and arrays

무한으로 중첩된 객체들과 배열들을 복사합니다

> Clone circular references

순환 참조들을 복사합니다

> Clone a wide variety of JavaScript types, such as `Date`, `Set`, `Map`, `Error`, `RegExp`, `ArrayBuffer`, `Blob`, `File`, `ImageData`, and [many more](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#supported_types)

Date, Set, Map, Error, RegExp, ArrayBuffer, Blob, File, ImageData, 그 외 등 폭넓은 다양한 자바스크립트 타입들을 복사합니다.

> Transfer any [transferable objects](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Transferable_objects)

모든 전달 가능한 객체들(transferable objects)을 전달합니다.

> So for example, this madness would even work as expected:

예를 들어, 이 무모한 코드는 예상한 대로 작동할 것입니다.

(madness : 광기, 광란, 무모한 짓, 바보 짓, 격노, 열광 …)

```jsx
const kitchenSink = {
  set: new Set([1, 3, 3]),
  map: new Map([[1, 2]]),
  regex: /foo/,
  deep: { array: [ new File(someBlobData, 'file.txt') ] },
  error: new Error('Hello!')
}
kitchenSink.circular = kitchenSink

*// ✅ All good, fully and deeply copied!*
const clonedSink = structuredClone(kitchenSink)
```

## \***\*[Why not just object spread?](https://www.builder.io/blog/structured-clone#why-not-just-object-spread) (객체 spread만 하는 것은 왜 안되나요?)**

> It is important to note we are talking about a deep copy. If you just need to do a shallow copy, aka a copy that does not copy nested objects or arrays, then we can just do an object spread:

우리가 깊은 복사에 대해서만 이야기 하고 있다는 것이 중요합니다.

만약 중첩된 객체 또는 배열들을 복사하지 않는, 즉 얕은 복사를 수행해야 하는 경우, 우리는 단지 객체 spread만 수행하면 됩니다.

```jsx
const simpleEvent = {
  title: "Builder.io Conf",
}
*// ✅ no problem, there are no nested objects or arrays*
const shallowCopy = {...calendarEvent}
```

> Or even one of these, if you prefer

또는 심지어 다음 코드들 중 선호하는 방식을 수행할 수 있습니다.

```jsx
const shallowCopy = Object.assign({}, simpleEvent);
const shallowCopy = Object.create(simpleEvent);
```

> But as soon as we have nested items, we run into trouble:

하지만 중첩된 항목들이 있을 경우, 우리는 문제에 빠집니다.

```jsx
const calendarEvent = {
  title: 'Builder.io Conf',
  date: new Date(123),
  attendees: ['Steve'],
};

const shallowCopy =
  { ...calendarEvent } * // 🚩 oops - we just added "Bob" to both the copy *and* the original event*
  shallowCopy.attendees.push('Bob') * // 🚩 oops - we just updated the date for the copy *and* original event*
  shallowCopy.date.setTime(456);
```

> As you can see, we did not make a full copy of this object.

보시다시피, 우리는 이 객체의 전체 복사본을 만들지 않았습니다.

> The nested date and array are still a shared reference between both, which can cause us major issues if we want to edit those thinking we are only updating the copied calendar event object.

중첩된 날짜 그리고 배열들은 여전히 둘 사이에 공유된 참조가 있으므로, 만약 복사된 일정 이벤트 객체만 업데이트하려는 생각으로 이를 수정한다면, 우리는 주요 문제를 일으킬 수 있습니다.

## \***\*[Why not `JSON.parse(JSON.stringify(x))` ?](https://www.builder.io/blog/structured-clone#why-not-code-json-parse-json-stringify-x-code) (JSON.parse(JSON.stringify(x)) 사용하는 것은 왜 안될까요?**

> Ah yes, this trick. It is actually a great one, and is surprisingly performant, but has some shortcomings that `structuredClone` addresses.

네 그렇습니다. 이 속임수가 있습니다. 이는 정말 훌륭하며, 놀라운 성능을 가지지만, structuredClone이 해결하는 몇 가지의 단점을 가집니다.

(addresses : 해결하다)

> Take this as an example:

이 예시를 들어봅시다.

```jsx
const calendarEvent = {
  title: "Builder.io Conf",
  date: new Date(123),
  attendees: ["Steve"]
}

*// 🚩 JSON.stringify converted the `date` to a string*
const problematicCopy = JSON.parse(JSON.stringify(calendarEvent))
```

> If we log `problematicCopy`, we would get:

만약 problematicCopy를 출력하보면, 우리는 다음과 같은 결과를 얻습니다.

```jsx
{
  title: "Builder.io Conf",
  date: "1970-01-01T00:00:00.123Z"
  attendees: ["Steve"]
}
```

> That’s not what we wanted! `date` is supposed to be a `Date` object, not a string.

이 결과는 우리가 원했던 것이 아닙니다! date 속성은 문자열이 아닌 Date 객채여야 합니다.

> This happened because `JSON.stringify` can only handle basic objects, arrays, and primitives. Any other type can be handled in hard to predict ways. For instance, Dates are converted to a string. But a `Set` is simply converted to `{}`.

이 문제는 JSON.stringify가 오직 기초적인 객체들, 배열들, 그리고 원시 요소들만 다룰 수 있기 때문에 일어납니다. 그리고 다른 타입들은 예측하기 어려운 방법으로 다뤄질 수 있습니다. 예를 들어, Date들은 문자열로 변환됩니다. 그러나 Set은 단순하게 {}로 변환됩니다.

> `JSON.stringify` even completely ignores certain things, like `undefined` or functions.

JSON.stringify는 심지어 undefined 또는 함수들과 같은, 특정한 요소들을 완전히 무시하곤 합니다.

> For instance, if we copied our `kitchenSink` example with this method:

예를 들어, 만약 다음과 같은 method로 kitchenSink를 복사한다면,

```jsx
const kitchenSink = {
  set: new Set([1, 3, 3]),
  map: new Map([[1, 2]]),
  regex: /foo/,
  deep: { array: [new File(someBlobData, 'file.txt')] },
  error: new Error('Hello!'),
};

const veryProblematicCopy = JSON.parse(JSON.stringify(kitchenSink));
```

> We would get:

다음과 같은 결과를 얻을 수 있습니다.

```jsx
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

> Ew! → 으악

> Oh yeah, and we had to remove the circular reference we originally had for this, as `JSON.stringify` simply throws errors if it encounters one of those.

네, 그리고 JSON.stringify가 순환 참조들 중 하나를 만나면 단순하게 errors를 발생시키기 때문에, 우리는 원래부터 가지고 있던 순환 참조를 삭제해야만 했습니다.

> So while this method can be great if our requirements fit what it can do, there is a lot that we can do with `structuredClone` (aka everything above that we failed to do here) that this method cannot.

따라서 이 method가 만약 우리의 요구사항들과 동작하는 방식이 맞는다면 훌륭할 수 있지만, 이 method로는 할 수 없는 structuredClone으로 할 수 있는 많은 일들(이 방법으로 실패한 위의 모든 코드들)이 있습니다.

## \***\*[Why not `_.cloneDeep`?](https://www.builder.io/blog/structured-clone#why-not-code-clone-deep-code) (\_.cloneDeep은 왜 안될까요?)**

> To date, Lodash’s `cloneDeep` function has been a very common solution to this problem.

오늘날, Lodash의 cloneDeep 함수는 이 문제에 대해 매우 일반적인 해결 방법이었습니다.

> And this does, in fact, work as expected:

그리고 이것은, 실제로, 예상대로 작동합니다.

```jsx
import cloneDeep from 'lodash/cloneDeep'

const calendarEvent = {
  title: "Builder.io Conf",
  date: new Date(123),
  attendees: ["Steve"]
}

*// ✅ All good!*
const clonedEvent = structuredClone(calendarEvent)
```

> But, there is just one caveat here. According to the [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost) extension in my IDE, that prints the kb cost of anything I import, this one function comes in at a whole 17.4kb minified (5.3kb gzipped):

하지만 여기에는 하나의 경고가 있습니다. IDE에서 import하는 모든 것들의 kb 비용을 출력해주는 Import Cost 확장 프로그램에 따르면, 이 하나의 함수는 17.4kb(압축 : 5.3kb)로 제공됩니다.

[https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2Faa5ab14fd21741bf8e327dd6e6fb68b1?width=800](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2Faa5ab14fd21741bf8e327dd6e6fb68b1?width=800)

> And that assumes you import just that function. If you instead import the more common way, not realizing that tree shaking doesn’t always work the way you hoped, you could accidentally import up to [25kb](https://bundlephobia.com/package/lodash@4.17.21) just for this one function 😱

그리고 이 함수를 import 한다고 가정해봅시다. tree shaking이 우리가 원하는 방법으로만 동작하지는 않는다는 것을 알지 못하고, 만약 더 일반적인 방법으로 이를 import 한다면, 이 한 함수를 위해 실수로 25kb를 import 할 수 있습니다. 😱

[https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F5dbbee190753414bb31b720059a501db?width=800](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F5dbbee190753414bb31b720059a501db?width=800)

> While that will not be the end of the world to anyone, it’s simply not necessary in our case, not when browsers already have `structuredClone` built in.

이는 누구에게도 세상의 종말이 되지는 않겠지만, 이는 브라우저가 이미 structuredClone을 내장하고 있다면, 간단하게 필요하지 않습니다.

## \****[What can `structuredClone` *not\* clone](https://www.builder.io/blog/structured-clone#what-can-code-structured-clone-code-em-not-em-clone) (structuredClone이 복사하지 못할 수 있는 것들)**

### \***\*[Functions cannot be cloned](https://www.builder.io/blog/structured-clone#functions-cannot-be-cloned) (함수들은 복사되지 않습니다.)**

> They will a throw a `DataCloneError` exception:

이는 DataCloneError 예외를 발생시킵니다.

```jsx
*// 🚩 Error!*
structuredClone({ fn: () => { } })
```

### \***\*[DOM nodes](https://www.builder.io/blog/structured-clone#dom-nodes) (Dom 노드들)**

> Also throws a `DataCloneError` exception:

또한 DataCloneError 예외를 발생시킵니다.

```jsx
*// 🚩 Error!*
structuredClone({ el: document.body })
```

### \***\*[Property descriptors, setters, and getters](https://www.builder.io/blog/structured-clone#property-descriptors-setters-and-getters)\*\***

프로퍼티(속성) descriptors, setters 그리고 getters

> As well as similar metadata-like features are not cloned.

게다가 비슷한 metadata-like 기능들은 복사되지 않습니다.

> For instance, with a getter, the resulting value is cloned, but not the getter function itself (or any other property metadata):

예를 들어, getter를 사용하면, 결과 값은 복사되지만, getter 함수 자체(또는 다른 프로퍼티 metadata)는 복사되지 않습니다.

```jsx
structuredClone({ get foo() { return 'bar' } })
*// Becomes: { foo: 'bar' }*
```

### \***\*[Object prototypes](https://www.builder.io/blog/structured-clone#object-prototypes) (객체 프로토타입)**

> The prototype chain is not walked or duplicated. So if you clone an instance of `MyClass`, the cloned object will no longer be known to be an instance of this class (but all valid properties of this class will be cloned)

프로토타입 체인은 이동하거나 복사되지 않습니다. 따라서 만약 MyClass의 인스턴스를 복사한다면, 복사된 객체는 더 이상 이 클래스의 인스턴스로 알려져 있지 않습니다. (그러나 이 클래스의 모든 유효한 프로퍼티들은 복사됩니다.)

```jsx
class MyClass {
  foo = 'bar'
  myMethod() { */* ... */* }
}
const myClass = new MyClass()

const cloned = structuredClone(myClass)
*// Becomes: { foo: 'bar' }*

cloned instanceof myClass *// false*
```

### \***\*[Full list of supported types](https://www.builder.io/blog/structured-clone#full-list-of-supported-types) (지원되는 타입들의 전체 목록)**

> More simply put, anything not in the below list cannot be cloned:

더 간단하게, 이 목록에 없으면 복사될 수 없습니다.

(해석 X)

**[JS Built-ins](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#javascript_types)**

`Array`, `ArrayBuffer`, `Boolean`, `DataView`, `Date`, `Error` types (those specifically listed below), `Map` , `Object` but only plain objects (e.g. from object literals), [Primitive types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#primitive_values), except `symbol` (aka `number`, `string`, `null`, `undefined`, `boolean`, `BigInt`), `RegExp`, `Set`, `TypedArray`

\***\*[Error types](https://www.builder.io/blog/structured-clone#error-types)\*\***

`Error`, `EvalError`, `RangeError`, `ReferenceError` , `SyntaxError`, `TypeError`, `URIError`

**[Web/API types](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#webapi_types)**

`AudioData`, `Blob`, `CryptoKey`, `DOMException`, `DOMMatrix`, `DOMMatrixReadOnly`, `DOMPoint`, `DomQuad`, `DomRect`, `File`, `FileList`, `FileSystemDirectoryHandle`, `FileSystemFileHandle`, `FileSystemHandle`, `ImageBitmap`, `ImageData`, `RTCCertificate`, `VideoFrame`

## \***\*[Browser and runtime support](https://www.builder.io/blog/structured-clone#browser-and-runtime-support) (브라우저 그리고 런타임 지원)**

> And here is the best part - `structuredClone` is supported in all major browsers, and even Node.js and Deno.

그리고 여기에 가장 좋은 부분이 있습니다. - structuredClone은 주요 브라우저들, 그리고 심지어 Node.js와 Deno에서도 지원됩니다.

> Just note the caveat with Web Workers having more limited support:

Web Workers의 제한적인 지원에 대한 주의 사항만 주목해주시면 됩니다.

[https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F1fdbc5b0826240e487a4980dfee69661?width=800](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F1fdbc5b0826240e487a4980dfee69661?width=800)

Source: [MDN](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone)

## \***\*[Conclusion](https://www.builder.io/blog/structured-clone#conclusion)\*\***

> It’s been a long time coming, but we finally now have `structuredClone` to make deep cloning objects in JavaScript a breeze. Thank you, [Surma](https://web.dev/structured-clone/).

오랜 시간이 지났지만, 마침내 자바스크립트의 객체들의 깊은 복사를 만들 수 있는 structuredClone을 가지게 되었습니다. Surma, 고마워요!
