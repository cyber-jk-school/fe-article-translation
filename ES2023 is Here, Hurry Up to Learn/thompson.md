# ES2023이 등장했습니다. 배우기 위해 서두르세요.

## 서문

ES6는 2015년에 제안되었고, 이 논리에 따라 ES2023은 ES14로 불려야 하지만, 혼란을 방지하기 위해 우리는 이름으로 연도를 사용할 것입니다. ES 표준에 관심을 쏟던 최근으로 돌아가 생각해보면, 당신은 여전히 ES6에 머물러 있었습니까? ES 표준의 주기를 따라가기 위해, 최근 몇 년 간 추가된 새로운 기능들을 살펴봅시다.
![es이미지](https://haeyum.notion.site/image/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A700%2F1*rIn5fuVGF942QDjmlhApuA.png?id=8069917b-8ce6-476c-baf8-a8c1d2ebc2d5&table=block&spaceId=c246955d-673f-42a3-b7a2-0b9a31a59058&width=2000&userId=&cache=v2)

## ES2023

> 2023년에 완성 예정인 제안

## 배열이 변경되었을 때 복사본을 반환합니다.

Array와 **TypedArray**는 배열 스스로를 변경하는 많은 방법(sort/splice 등)을 가지고 있습니다.

```javascript
const array = [3, 2, 1];
const sortedArray = array.sort();
// [1, 2, 3]
console.log(sortedArray);
// The original array also becomes [1, 2, 3]
console.log(array);
```

만약 배열 자체를 변경하고 싶지 않다면 아래와 같이 할 수 있습니다.

```javascript
const array = [3, 2, 1];
const sortedArray = array.toSorted();
// [1, 2, 3]
console.log(sortedArray);
// The original array keep [3, 2, 1]
console.log(array);
```

유사한 방법은 아래와 같습니다.

```javascript
T.prototype.toReversed() -> T
T.prototype.toSorted(compareFn) -> t
T.prototype.toSpliced(start, deleteCount, ...items) -> T
T.prototype.with(index, value) -> T
```

reverse/sort/splice는 이해할 수 있는데, with은 무엇을 위한 걸까요? 이해를 위해 예시를 살펴보세요.

```javascript
const array = [1, 2, 3];
const newArray = array.with(1, false);
// [1, false, 3]
console.log(newArray);
// The original array keep [1, 2, 3]
console.log(array);
```

## WeakMap은 키로써 Symbol을 지원합니다.

원래 **WeakMap**은 키로 오직 **object 타입**만을 지원했지만, 이제 키로써 Symbol 타입을 지원합니다.

```javascript
const weak = new WeakMap();
weak.set(Symbol("symbol1"), {});
```

## Hashbang 문법

Hashbang 또는 Shebang이라고 불리는 문법은 파일을 실행하기 위해 어떤 인터프리터를 사용할 지 명시하는 # 문자와 ! 문자로 구성된 문자열입니다.

```javascript
// hashbang.js
#!/usr/bin/env node
console.log('hashbang');

// nohashbang.js
console.log('no hashbang')
```

터미널에서 실행할 때 **Hashbang**이 없으면, 실행을 위해 node 명령어를 사용해야 합니다.
![hashbang](https://haeyum.notion.site/image/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A492%2F1*EWzE-at-9kqkp26vtcUsBg.png?id=edcb6128-8d73-42c8-a91b-6ada3484cce7&table=block&spaceId=c246955d-673f-42a3-b7a2-0b9a31a59058&width=2000&userId=&cache=v2)

## 끝에서부터 찾아보세요.

Two functions are involved in **findLast** / **findLastIndex**
두 함수는 **findLast**와 **findLastIndex**와 관련되어 있습니다(??? 여기를 어떻게 번역할 지 모르겠슴다)

## ES2022

## 예외 체인

예제를 바로 살펴보면, err0.cause를 통해 err1을 얻을 수 있습니다. 만약 이 예외가 여러 번 발생한다면, 이것은 err1.cause.cause... 와 같습니다. 당신은 관련된 모든 예외를 얻을 수 있습니다.

```javascript
function willThrowError() {
  try {
    // do something
  } catch (err0) {
    throw new Error("one error", { cause: err });
  }
}
try {
  willThrowError();
} catch (err1) {
  // You can get err0 through err1.caus
}
```

## 클래스 정적 코드 블록

복잡한 정적 프로퍼티/메서드 초기화 로직을 수행하기 위핸 정적 코드 블록

```javascript
// When there is no static code block,
// the initialization editor is separate from the class definition
class C {
  static x = ...;
  static y;
  static z;
}
try {
  const obj = doSomethingWith(C.x);
  C.y = obj.y
  C.z = obj.z;
}
catch {
  C.y = ...;
  C.z = ...;
}
// Using static code blocks, the code logic is more convergent
class C {
  static x = ...;
  static y;
  static z;
  static {
    try {
      // Here this represents C not an instance of C
      const obj = doSomethingWith(this.x);
      this.y = obj.y;
      this.z = obj.z;
    }
    catch {
      this.y = ...;
      this.z = ...;
    }
  }
}
```

또한 private 프로퍼티에 접근할 수 있는 특별한 기능이 있습니다.

```javascript
let getPrivateField;
class D {
  #privateField;
  constructor(v) {
    this.#privateField = v;
  }
  static {
    getPrivateField = (d) => d.#privateField;
  }
}
getPrivateField(new D("private value"));
// → private value
```

## Object.hasOwn

우리는 종종 이러한 사용법을 봅니다. 특히 오픈 소스 라이브러리에서요.

```javascript
let hasOwnProperty = Object.prototype.hasOwnProperty;
if (hasOwnProperty.call(object, "foo")) {
  console.log("has property foo");
}
```

만약 Object.hasOwn을 사용한다면 아래와 같이 간략화할 수 있습니다.

```javascript
if (Object.hasOwn(object, "foo")) {
  console.log("has property foo");
}
```

## .at은 특정한 인덱스의 요소를 반환합니다.

인덱스로 접근 가능한 타입(String/Array/TypedArray)은 at을 통해 특정한 인덱스의 요소를 읽을 수 있고, 음수를 전달하는 것도 지원합니다.

```javascript
// return 4
[1, 2, 3, 4].at(-1);
```

## private 프로퍼티가 존재하는지 판단하기

in 키워드를 사용해 판별할 수 있습니다.

```javascript
class C {
  #brand = 0;

  static isC(obj: C) {
    return #brand in obj;
  }
}
```

## Top-level await

async 패키지를 사용할 필요 없이 await을 사용할 수 있습니다.

```javascript
function fn() {
  return Promise.resolve();
}
// Previously required to implement via IIFE
(async function () {
  await fn();
})();
// After supporting top-level await, you can directly call
await fn();
```

## 정규 표현식 슬라이싱

정규식 매칭을 통해, 우리는 일치하는 문자열을 얻을 수 있고, 원래 문자열에서 해당하는 문자열의 위치를 얻을 수 있습니다.

```javascript
const re1 = /a+(z)?/d;
const s1 = "xaaaz";
const m1 = re1.exec(s1);
```

![regular expression](https://haeyum.notion.site/image/https%3A%2F%2Fmiro.medium.com%2Fv2%2Fresize%3Afit%3A700%2F1*nhrc1rbqrEFQ_Ijt872Jgg.png?id=8384c359-ff94-471a-b913-f6075419fb35&table=block&spaceId=c246955d-673f-42a3-b7a2-0b9a31a59058&width=2000&userId=&cache=v2)

## ES2021

## 숫자 구분자

만약 숫자가 길어지면, 가독성이 떨어집니다. 구분자 \_ 는 가독성을 향상시킵니다.

```javascript
// 1000000
console.log(1_000_000);
// As long as the delimiter is not the first and last digit of the number,
// it is valid in any position 1_000000000_1_2_3
1_000000000_1_2_3;
```

## 논리 연산자 할당

산술 연산자 할당과 비슷하지만 논리 연산자는 각각을 비교할 수 있는 단축 평가가 가능합니다.

```javascript
// arithmetic operator
let x = 1;
// x is 2
x += 1;
// Logical Operators
let x = 1;
// x is still 1, because x is a true value,
// so it is short-circuited, which is equivalent to x = 1 || 3
x ||= 3;
```

## WeakRef

약한 참조는 객체가 가비지 콜렉팅의 대상이 되는 것을 막지 않는 객체 참조를 허용합니다.

```javascript
let obj = { name: "obj1" };
let weakRef = new WeakRef(obj);
// Get the referenced object
// If the reference object is recycled, it will get undefined
weakRef.deref();
```

## Promise.any
