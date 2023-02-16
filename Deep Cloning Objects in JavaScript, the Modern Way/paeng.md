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
- `Date`, `Set`, `Map`, `Error`, `RegExp`, `ArrayBuffer`, `Blob`, `File`, `ImageData`, [그 이상](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#supported_types)와 같은 다양한 JavaScript 유형 복제
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
