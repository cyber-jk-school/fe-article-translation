# 현대적인 방식의 자바스크립트 객체 깊은 복사

![이미지1](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F03f2036674724006ae64d9bc4d07ab6d?format=webp&width=2000)

이제 자바스크립트에서 객체의 깊은 복사를 수행하는 자체적인 방법이 있다는 걸 알고 있나요?

맞아요! `structuredClone` 함수가 자바스크립트 런타임에 추가되었습니다.

```javascript
const calendarEvent = {
  title: "Builder.io Conf",
  date: new Date(123),
  attendees: ["Steve"],
};

// 😍
const copied = structuredClone(calendarEvent);
```

위의 예시에서 객체뿐 아니라 중첩된 배열이나 Date 객체도 복사했다는 걸 눈치채셨나요?

그리고 예측한 대로 정확히 동작합니다.

```javascript
copied.attendees; // ["Steve"]
copied.date; // Date: Wed Dec 31 1969 16:00:00
cocalendarEvent.attendees === copied.attendees; // false
```

맞아요! `structuredClone`은 위의 동작 뿐 아니라 추가적으로 아래와 같은 동작도 가능합니다.

- 무한대로 중첩된 객체와 배열의 복제
- 순환 참조 복제
- 다양한 자바스크립트 타입의 복제, `Date`, `Set`, `Map`, `Error`, `RegExp`, `ArrayBuffer`, `Blob`, `File`, `ImageDate`, 그리고 이 외에도 [더 많은 타입들](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#supported_types)
- [transferable objects](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Transferable_objects) 전달

그렇기 때문에 예를 들어, 아래와 같은 미친 짓도 예측한 대로 동작할 것입니다.

```javascript
const kitchenSink = {
  set: new Set([1, 3, 3]),
  map: new Map([[1, 2]]),
  regex: /foo/,
  deep: { array: [new File(someBlobData, "file.txt")] },
  error: new Error("Hello!"),
};
kitchenSink.circular = kitchenSink;

// ✅ All good, fully and deeply copied!
const clonedSink = structuredClone(kitchenSink);
```

## 왜 객체 스프레드를 사용하지 않나요?

우리가 깊은 복사에 대해 이야기하고 있다는 것이 중요합니다. 만약 중첩된 객체나 배열을 복사하지 않는 얕은 복사를 수행해야 한다면, 객체 스프레드를 사용할 수 있습니다.

```javascript
const simpleEvent = {
  title: "Builder.io Conf",
};
// ✅ 중첩된 객체나 배열이 없기 때문에 문제 없음.
const shallowCopy = { ...calendarEvent };
```

또는 선호에 따라 아래의 방법들을 사용할 수 있습니다.

```javascript
const shallowCopy = Object.assign({}, simpleEvent);
const shallowCopy = Object.create(simpleEvent);
```

그러나 중첩된 아이템을 만나면, 우리는 문제에 처하게 됩니다.

```javascript
const calendarEvent = {
  title: "Builder.io Conf",
  date: new Date(123),
  attendees: ["Steve"],
};

const shallowCopy = { ...calendarEvent };

// 🚩 원본 객체와 복사된 객체 모두에 "Bob"을 추가했음.
shallowCopy.attendees.push("Bob");

// 🚩 원본 객체와 복사된 객체 모두의 Date 객체를 수정했음.
shallowCopy.date.setTime(456);
```

위에서 볼 수 있는 것처럼, 우리는 이 객체의 완전한 복사를 수행하지 못했습니다.

중첩된 Date나 배열은 두 객체(원본과 복사본)가 참조를 공유하고 있고, 이는 만약 우리가 복사된 `calenderEvent` 객체만을 수정하고 싶은 상황에서 문제를 발생시킬 수 있습니다.

## 왜 `JSON.parse(JSON.stringify(x))`를 사용하지 않나요?

아 예, 이 방법. 이 방법은 실제로 매우 좋은 방법이고 유용하지만, `structuredClone`으로 처리할 수 있는 몇 가지 단점들이 있습니다.

아래 예시를 보세요.

```javascript
const calendarEvent = {
  title: "Builder.io Conf",
  date: new Date(123),
  attendees: ["Steve"],
};

// 🚩 JSON.stringify가 date를 string으로 변환했음.
const problematicCopy = JSON.parse(JSON.stringify(calendarEvent));
```

만약 `problematicCopy`를 출력하면 아래와 같은 결과를 얻을 것입니다.

```javascript
{
  title: "Builder.io Conf",
  date: "1970-01-01T00:00:00.123Z"
  attendees: ["Steve"]
}
```

이것은 우리가 원했던 것이 아닙니다! `date`는 문자열이 아닌 `Date` 객체여야합니다.

이것은 `JSON.stringify`가 오직 기본 객체, 배열, 그리고 기본형만 다룰 수 있기 때문에 발생했습니다. 다른 타입은 예측한 방향으로 다루기 어렵습니다. 예를 들어, Date 객체는 string으로 변환되고, `Set`은 단순하게 `{}` 빈 객체로 변환됩니다.

심지어 `JSON.stringify`는 `undefined`나 함수 같은 특정한 것들을 완전히 무시합니다.

예를 들어, JSON.stringify를 사용해 아래 예시의 `kitchenSink`를 복사했다면

```javascript
const kitchenSink = {
  set: new Set([1, 3, 3]),
  map: new Map([[1, 2]]),
  regex: /foo/,
  deep: { array: [new File(someBlobData, "file.txt")] },
  error: new Error("Hello!"),
};

const veryProblematicCopy = JSON.parse(JSON.stringify(kitchenSink));
```

아래와 같은 결과를 얻을 것입니다.

```javascript
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

윽!
`JSON.stringify`를 사용하면 오류 중 하나를 만나면 오류를 반환하기 때문에 우리는 순환 참조를 제거해야 했습니다.
이 메서드는 우리의 요구사항과 메서드가 할 수 있는 것이 맞으면 매우 좋을 수 있지만, `structuredClone`을 사용하면 `JSON.stringify`는 하지 못하는 많은 작업(위에서 실패했던 모든 것들)을 할 수 있습니다.

## 왜 `_.cloneDeep`을 사용하지 않나요?

date의 문제에서 Lodash의 `cloneDeep` 함수는 매우 흔한 해결법입니다.

실제로 이것은 예측한 대로 동작합니다.

```javascript
import cloneDeep from "lodash/cloneDeep";

const calendarEvent = {
  title: "Builder.io Conf",
  date: new Date(123),
  attendees: ["Steve"],
};

// ✅ All good!
const clonedEvent = structuredClone(calendarEvent);
```

그러나 이 방법은 주의할 점이 한 가지 있습니다. 제 IDE의 [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)(import한 모든 것의 용량을 출력하는) 확장 프로그램에 따르면, 이 하나의 함수는 17.4kb로 제공됩니다. (gzip 압축 시 5.3k)
![lodash](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2Faa5ab14fd21741bf8e327dd6e6fb68b1?format=webp&width=2000)
그리고 위의 방법은 당신이 오직 함수만 import 한 경우를 가정합니다. 만약 당신이 더 흔한 방법, 트리 쉐이킹이 당신이 바라는 대로 동작하지 않는다는 사실을 깨닫지 못하고, 하나의 함수를 위해 최대 25kb까지 실수로 import 할 수 있습니다.
![treeshaking](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F5dbbee190753414bb31b720059a501db?format=webp&width=2000)

이것은 누구에게도 세상의 끝이 되지 않겠지만, 브라우저에 이미 `structuredClone`이 내장되어 있다면 필요하지 않습니다.

## `structuredClone`이 복제할 수 없는 것

### 함수는 복제할 수 없습니다.

함수는 `DataCloneError` exception을 반환합니다.

```javascript
// 🚩 Error!
structuredClone({ fn: () => {} });
```

### DOM 노드

마찬가지로 `DataCloneError` exception을 반환합니다.

```javascript
// 🚩 Error!
structuredClone({ el: document.body });
```

### 속성 설명자, 접근자(getter), 설정자(setter)

메타 데이터와 유사한 기능은 복제되지 않습니다.
예를 들어, 반환값은 복제되지만, 접근자 함수 자체(또는 다른 속성 메타 데이터)는 복제되지 않습니다.

```javascript
structuredClone({
  get foo() {
    return "bar";
  },
});
// Becomes: { foo: 'bar' }
```

### 객체 프로토타입

프로토타입 체인은 walked(?) 되거나 중복되지 않습니다. 그렇기 때문에 `MyClass`의 인스턴스를 복제할 때, 복제된 객체는 더 이상 해당 클래스의 인스턴스인지 알지 못할 것입니다. (그러나 클래스의 모든 유효한 속성은 복제될 것입니다.)

```javascript
class MyClass {
  foo = "bar";
  myMethod() {
    /* ... */
  }
}
const myClass = new MyClass();

const cloned = structuredClone(myClass);
// Becomes: { foo: 'bar' }

cloned instanceof myClass; // false
```

### 지원되는 타입 목록

간단하게, 아래 목록에 없는 타입은 복제할 수 없습니다.

#### [자바스크립트 빌트인](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#javascript_types)

`Array`, `ArrayBuffer`, `Boolean`, `DataView`, `Date`, `Error` 타입 (아래에 구체적으로 정리되어 있음.), `Map` , 간단한 `Object` (예를 들면 객체 리터럴), `Symbol`을 제외한 [원시 타입](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#primitive_values) (number, string, null, undefined, boolean, BigInt), `RegExp`, `Set`, `TypedArray`

#### Error 타입

`Error`, `EvalError`, `RangeError`, `ReferenceError`, `SyntaxError`, `TypeError`, `URIError`

#### [Web/API 타입](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#webapi_types)

`AudioData`, `Blob`, `CryptoKey`, `DOMException`, `DOMMatrix`, `DOMMatrixReadOnly`, `DOMPoint`, `DomQuad`, `DomRect`, `File`, `FileList`, `FileSystemDirectoryHandle`, `FileSystemFileHandle`, `FileSystemHandle`, `ImageBitmap`, `ImageData`, `RTCCertificate`, `VideoFrame`

## 브라우저와 런타임 지원

`structuredClone`은 모든 주요 브라우저와 Node.js, Deno까지 지원됩니다.
Web Workers와 사용하는 것은 지원에 더 많은 제약이 있다는 걸 명심하세요.
![supports](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F1fdbc5b0826240e487a4980dfee69661?format=webp&width=2000)
출처: [MDN](https://developer.mozilla.org/en-US/docs/Web/API/structuredClone)

## 결론

오랜 시간이 걸렸지만, 드디어 자바스크립트에서 쉽게 깊은 복사를 할 수 있는 `structuredClone`를 얻었습니다. 감사합니다 [Surma](https://web.dev/structured-clone/)
