제목: [번역] ES2023 is Here, Hurry Up to Learn

# ES2023있으니 어서 배웁시다.

원문: https://javascript.plainenglish.io/es2023-is-here-hurry-up-to-learn-2de85c61f0ae

## 서문

ES6는 2015년도에 제안되었으며, 이 논리에 따르면 ES2023은 ES14로 불려야 되지만 혼동을 피하기 위해 이름에 연도를 사용합니다. ES 표준에 집중했던 때를 돌이켜보면, 여전히 ES6에 머물러 있습니까?
ES 표준이 되풀이 되는 것을 위해 최근에 어떤 새로운 특징이 추가되었는지 보겠습니다.

![번역1](https://user-images.githubusercontent.com/53526987/234828530-b38244ba-4b6f-43c5-b3a4-9bb8c9d35877.png)

## ES2023

> 2023년도에 완료될 제안들

### Array가 수정되면 수정본을 반환

Array, **TypedArray**에는 다음과 같이 Array 자체를 변경하는 다양한 방법들(sort/splice, etc)이 있습니다.:

```js
const array = [3, 2, 1];
const sortedArray = array.sort();
// [1, 2, 3]
console.log(sortedArray);
// 본래의 array 또한 [1, 2, 3]이 됩니다.
console.log(array);
```

array 자체를 변경하고 싶지 않으면 다음과 같이 할 수 있습니다.:

```js
const array = [3, 2, 1];
const sortedArray = array.toSorted();
// [1, 2, 3]
console.log(sortedArray);
// 본래의 array는 [3, 2, 1]로 유지됩니다.
console.log(array);
```

유사한 방법들은 여기에 포함되어 있습니다.:

```js
T.prototype.toReversed() -> T
T.prototype.toSorted(compareFn) -> t
T.prototype.toSpliced(start, deleteCount, ...items) -> T
T.prototype.with(index, value) -> T
```

reverse/sort/splice는 이해할 수 있지만 with는 무엇입니까? 이해를 위해 예시를 보시면 됩니다.:

```js
const array = [1, 2, 3];
const newArray = array.with(1, false);
// [1, false, 3]
console.log(newArray);
// 본래의 array는 [1, 2, 3]으로 유지됩니다.
console.log(array);
```

### WeakMap은 Symbol을 키로서 지원합니다.

**WeakMap** 원래 **object 타입**의 키만 지원했지만 지금은 Symbol 타입도 키로써 지원합니다.

```js
const weak = new WeakMap();
weak.set(Symbol('symbol1'), {});
```

### Hashbang syntax

Hashbang은 Shebang으로도 불리우며, 파일을 실행하기 위해 사용할 인터프리터를 지정하는 `#!` 파운드 기호와 느낌표로 구성된 일련의 문자로 이루어져 있습니다.:

```js
// hashbang.js
#!/usr/bin/env node
console.log('hashbang');

// nohashbang.js
console.log('no hashbang')
```

**Hashbang** 없이 터미널에서 실행은 **node** 명령어로 실행해야 합니다.:

![번역2](https://user-images.githubusercontent.com/53526987/235352159-ceb58b37-b1a1-4301-97b5-c719b7bebfdd.png)

### 끝에서 부터 봅시다.

**findLast / findLastIndex**는 두가지 기능에 관련이 있습니다.

```js
const array = [1, 2, 3];
array.findLast((n) => n.value % 2 === 1);
array.findLastIndex((n) => n.value % 2 === 1);
```

## ES2022

### 예외 체인

예시를 바로 보면, `err0.cause`를 통해 `err1`을 얻을 수 있습니다.; 만약 이 예외가 계속 반복된다면, `err1.cause.cause`... 모든 관련 예외를 얻을 수 있습니다.

```js
function willThrowError() {
  try {
    // do something
  } catch (err0) {
    throw new Error('one error', { cause: err });
  }
}
try {
  willThrowError();
} catch (err1) {
  // You can get err0 through err1.caus
}
```

### 클래스 정적 코드 블록

복잡한 정적 property/method 초기 논리를 구현하기 위한 정적 코드 블록:

```js
// 정적 코드 블록이 아닐때,
// 초기 에디터는 클래스 정의에서 나누어 집니다.
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
// 정적 코드 블록을 사용하면, 코드 논리는 더 집중됩니다.
class C {
  static x = ...;
  static y;
  static z;
  static {
    try {
      // 여기서 this는 C의 인스턴스가 아닌 C를 나타냅니다.
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

특별한 기능은: **private properties에 접근할 수 있습니다.**

```js
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
getPrivateField(new D('private value'));
// → private value
```

### Object.hasOwn

특히 오픈 소스 라이브러리에서 이러한 사용을 볼 수 있습니다.

```js
let hasOwnProperty = Object.prototype.hasOwnProperty;
if (hasOwnProperty.call(object, 'foo')) {
  console.log('has property foo');
}
```

Object.hasOwn을 사용하면, 간단하게 사용할 수 있습니다.:

```js
if (Object.hasOwn(object, 'foo')) {
  console.log('has property foo');
}
```

### `.at`은 특정 인덱스의 요소를 반환합니다.

인덱스 가능한 유형들 (String/Array/**TypedArray**)은 `at`을 통해 특정 인덱스의 요소를 읽을 수 있고, 음수를 사용한 수 있습니다.

```js
// return 4
[1, 2, 3, 4].at(-1);
```

### private property 존재하는지를 결정

`in` 키워드로 판단할 수 있습니다.

```js
class C {
  #brand = 0;

  static isC(obj: C) {
    return #brand in obj;
  }
}
```

### Top-level await

`async` 패키지 없이, 다음과 같이 `await`를 사용할 수 있습니다.:

```js
function fn() {
  return Promise.resolve();
}
// 이전에는 IIFE를 통해 구현해야 했습니다.
(async function () {
  await fn();
})();
// top-level await가 제공된 후에, 직접적으로 사용할 수 있습니다.
await fn();
```

### 정규식 slicing

정규 매칭을 통해 매칭된 문자열을 얻을 수 있고, slicing을 통해 기존 문자열에서 이 문자열의 위치를 얻을 수 있습니다.

```js
const re1 = /a+(z)?/d;
const s1 = 'xaaaz';
const m1 = re1.exec(s1);
```

![번역3](https://user-images.githubusercontent.com/53526987/235408592-bd7cf222-5d12-4a0b-b361-2c72046d07d8.png)

## ES2021

### 숫자 구분 기호

숫자가 너무 길다면, 읽는 것이 불편할 수 있어 `_` 구분 기호를 이용 할 수 있습니다.:

```js
// 1000000
console.log(1_000_000);
// 구분 기호가 첫 / 마지막 글자가 아닌 이상 어디든 위치할 수 있습니다.
1_000000000_1_2_3;
```

### 논리 연산자 할당

산술 연산자 할당이랑 유사하지만, 논리 연산자 각각 비교할 수 있는 단락 기능을 갖습니다.

```js
// 산술 연산자
let x = 1;
//x 은 2
x += 1;
// 논리 연산자
let x = 1;
// x는 여전히 1, x는 참 값이기 때문
// x = 1 || 3과 같은 단락 기능
x ||= 3;
```

### WeakRef

객체를 가비지 컬렉터에서 수집되는 것을 막지 않고 참조할 수 있게 해주는 약한 참조.

```js
let obj = { name: 'obj1' };
let weakRef = new WeakRef(obj);
// 객체 참조 얻기
// 객체 참조를 반복하면 undefined를 얻을 것입니다.
weakRef.deref();
```

### Promise.any

몇몇 다른 함수들과 비교합니다.

#### Promise.any

하나의 입력이 해결한 것과 같이 해결합니다.

```js
Promise.any([
  Promise.reject('reject 1'),
  Promise.reject('reject 2'),
  Promise.reject('reject 3'),
  Promise.resolve('1'),
  Promise.resolve('2'),
]).then(
  (first) => {
    // 해결한 것 같이 여기서 실행됩니다.
    // prints 1
    console.log(first);
  },
  (error) => {
    // 모두 reject되면 여기서 실행됩니다.
    console.log(error);
  }
);
```

#### Promise.allSettled

입력된 promise가 reject 인지 reslove인지 상관없이 then까지 롤업됩니다.

```js
Promise.allSettled([
  Promise.resolve('1'),
  Promise.resolve('2'),
  Promise.reject('e1'),
  Promise.resolve('3'),
  Promise.resolve('4'),
]).then((res) => {
  console.log('then', res);
});
```

![번역4](https://user-images.githubusercontent.com/53526987/235410362-094632c0-61d1-4455-aedb-5b027fb4ad36.png)

#### Promise.all

reject가 있으면 then reject가 됩니다. 다음 예제는 catch로 이동합니다.:

```js
Promise.all([
  Promise.resolve('1'),
  Promise.resolve('2'),
  Promise.reject('e1'),
  Promise.resolve('3'),
])
  .then((res) => {
    console.log('then', res);
  })
  .catch((err) => {
    // catch e1 출력
    console.log('catch', err);
  });
```

#### Promise.race

reject(or resolve)가 있으면, then reject(or resolve)가 됩니다.

```js
function delay(type: 'resolve' | 'reject', timeout: number) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (type === 'reject') {
        reject(type);
      } else {
        resolve(type);
      }
    }, timeout);
  });
}
// print then resolve
Promise.race([delay('resolve', 1000), delay('reject', 2000)])
  .then((res) => {
    console.log('then', res);
  })
  .catch((err) => {
    console.log('catch', err);
  });

// print catch reject
Promise.race([delay('resolve', 2000), delay('reject', 1000)])
  .then((res) => {
    console.log('then', res);
  })
  .catch((err) => {
    console.log('catch', err);
  });
```

### String.prototype.replaceAll;

매칭된 모든 문자열 변환

```js
// 12c12d12
'abcabdab'.replaceAll('ab', '12');
// 12cabdab
'abcabdab'.replace('ab', '12');
```

## ES2020

### import.meta

host 관련 모듈 메타 정보 제공

### 연산자와 병합되는 Null 값

오른쪽 피연산자는 왼쪽 피연산자가 null 또는 undefined인 경우에만 반환됩니다.

```js
// "default"
let a = null || 'default';
// "default"
let a = null ?? 'default';
// "default"
let a = '' || 'default';
// ""
let a = '' ?? 'default';
// "default"
let a = 0 || 'default';
// 0
let a = 0 ?? 'default';
```

### Optional chain

코드에서 보겠습니다.

```js
// optional chaining 없이
const street = user.address && user.address.street;
// optional chaining
const street = user.address?.street;
```

### BigInt

2의 53제곱근 보다 큰 수(js `Number` 유형에서 나타낼 수 있는 가장 큰 수)를 나타낼 수 있습니다.

```js
// 숫자 마지막에 n 추가
const theBiggestInt = 9007199254740991n;
// BigInt 구조체 사용
const alsoHuge = BigInt(9007199254740991);
```

### import()

모듈들이 런타임에서 동적으로 로드됩니다.

### String.prototype.matchAll

`match`는 오직 매칭된 문자열을 리턴하고, 그룹 정보는 없지만; `matchAll`은 매칭된 문자열과 그룹 정보와 함께 리턴합니다.

![번역5](https://user-images.githubusercontent.com/53526987/235412799-8c010485-7118-4d1f-98ee-96bbb82b405d.png)

### 요약

ES2016 ~ ES2019는 하나씩 나열되지 않았지만 모두가 숙지해야 합니다. 명확하지 않는 특징이 있으면 메시지를 남겨주시면 됩니다.

### 참고 문서

[github.com/tc39/propos…](https://github.com/tc39/proposals/blob/main/finished-proposals.md)
More content at [PlainEnglish.io.](https://plainenglish.io/)
Sign up for our [free weekly newsletter](https://ipenewsletter.substack.com/). Follow us on [Twitter](https://twitter.com/inPlainEngHQ), [LinkedIn](https://www.linkedin.com/company/inplainenglish/), [YouTube](https://www.youtube.com/channel/UCtipWUghju290NWcn8jhyAw), and [Discord](https://discord.com/invite/GtDtUAvyhW).
Interested in scaling your software startup? Check out [Circuit](https://circuit.ooo/?utm=publication-post-cta).
