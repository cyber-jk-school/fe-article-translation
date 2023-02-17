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
- `Date`, `Set`, `Map`, `Error`, `RegExp`, `ArrayBuffer`, `Blob`, `File`, `ImageData`, [그 이상](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#supported_types)과 같은 다양한 JavaScript 유형 복제
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
