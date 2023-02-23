# TypeScript에서 더 나은 구성을 위한 `satisfies` 연산자

![1](https://user-images.githubusercontent.com/53526987/220627936-836bb750-eb2a-41ad-b2aa-e3a494cabe5a.png)

TypeScript에서 type-safe 구성을 하는 새롭고 더 나은 방법이 있습니다.

TypeScript 4.9에서 발표된 새로운 `satisfies` 연산자를 사용합니다.

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: '/auth',
  },
} satisfies Routes; // 😍
```

## 왜 `satisfies`가 필요합니까?

위 예시를 표준 type 선언으로 다시 작성하는 것부터 시작하겠습니다.

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes: Routes = {
  AUTH: {
    path: '/auth',
  },
};
```

지금까지 모든건 괜찮아 보입니다. IDE에서 적절한 type 검사와 완성을 하게 됩니다.

그러나, routes 객체를 사용할 때, 컴파일러(및 이후의 IDE)는 실제 구성된 routes가 무엇인지 알지 못합니다.

예를 들아, 이 컴파일러는 잘 동작하지만 런타임에서 error을 발생시킵니다.

```ts
// 🚩 TypeScript compiles just fine, yet `routes.NONSENSE` doesn't actually exist
routes.NONSENSE.path;
```

왜 이런일이? `Routes` type은 어떤 문자열도 key로 사용할 수 있기 때문입니다. 그래서 TypeScript는 간단한 오타부터 완전한 nonsense key에 이르기까지 모든 key의 접근을 허용합니다. 또한 route keys를 자동완성하기 위한 도움을 IDE에서 줄 수 없습니다.

"`as` 키워드는 어때"라고 말할 수 있습니다. 그러면, 다음과 같이 간단하게 살펴볼 수 있습니다.:

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: '/auth',
  },
} as Routes;
```

TypeScript에서는 평범한 습관이지만 실제로는 매우 위헙합니다.

위와 같은 문제가 있을 뿐만 아니라 실제로 존재하지 않는 key와 value 쌍을 작성할 수 있으며 TypeScript에서는 다른 방식으로 보여질 수 있습니다.:

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: '/auth',
    // 🚩 TypeScript compiles just fine, yet this is not even a valid property!
    nonsense: true,
  },
} as Routes;
```

일반적으로, TypeScript에서 `as` 키워드를 사용하는 것을 피합시다.

## `Satisfies`는 하루를 절약할 수 있습니다.

이제 `satisfies` 키워드를 사용하여 위의 예제들을 다시 작성하겠습니다.

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: '/auth',
  },
} satisfies Routes;
```

이것을 통해 원하는 type의 검사를 모두 수행할 수 있습니다.:

```ts
routes.AUTH.path; // ✅
routes.AUTH.children; // ❌ routes.auth has no property `children`
routes.NONSENSE.path; // ❌ routes.NONSENSE doesn't exist
```

물론, IDE에서도 자동완성됩니다.:

![2](https://user-images.githubusercontent.com/53526987/220920276-90f3a461-049c-4eef-93e4-de42a91b7311.png)

다음과 같이 좀 더 복잡한 예를 들 수 있습니다.:

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: '/auth',
    children: {
      LOGIN: {
        path: '/login',
      },
    },
  },
  HOME: {
    path: '/',
  },
} satisfies Routes;
```

그리고 이제 우리는 우리가 사용했던 일반적인 type을 기반으로 한 것과 반대로, IDE 자동완성과 type 체킹을 트리 아래까지 정확하게 파악하는 방법을 얻습니다.:

![3](https://user-images.githubusercontent.com/53526987/220932259-5f03b1a0-1b74-4689-b439-d8004465822f.gif)

```ts
routes.AUTH.path; // ✅
routes.AUTH.children.LOGIN.path; // ✅
routes.HOME.children.LOGIN.path; // ❌ routes.HOME has no property `children`
```

정확히 우리가 원하던 거였습니다.

## `as const`와의 결합
