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

당신이 마주할 수 있는 마지막 상황은 `satisfies`의 평범한 사용으로, 구성 객체가 이상적인 것보다 조금 더 느슨하게 캡처되는 것입니다.

예를 들어, 다음과 같은 코드를 사용합니다.:

```ts
const routes = {
  HOME: { path: '/' },
} satisfies Routes;
```

`path` property type을 확인하면 `string` type을 얻을 수 있습니다.

```ts
routes.HOME.path; // Type: string
```

그러나, 구성과 관련하여 [const assertions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions) (aka `as const`)이 빛을 발하는 순간입니다. 만약 `as const`를 사용한다면, 적절한 문자열 리터럴 `/`까지 더 정확한 types을 얻을 수 있습니다.

```ts
const routes = {
  HOME: { path: '/' },
} as const;

routes.HOME.path; // Type: '/'
```

이것은 "ok, 멋진 트릭인데 왜 내가 신경써야 하는가"처럼 보일 수 있지만, 정확한 경로까지 type-safe하게 사용할 수 있는 방법인지 고려해봐야 합니다.:

```ts
function navigate(path: '/' | '/auth') { ... }
```

각 `path`가 `string`으로만 알려지게 하는 `satisfies`를 사용하면, type 에러를 발생합니다.:

```ts
const routes = {
  HOME: { path: '/' },
} satisfies Routes;

navigate(routes.HOME.path);
// ❌ Argument of type 'string' is not assignable to parameter of type '"/" | "/auth"'
```

앍! `HOME.path`는 유효 문자열(`/`)이지만, TypeScript는 아닙니다.

음, 여기서 `satisfies`와 `as const`를 결합하여 최고의 결과를 얻을 수 있습니다.

```ts
  const routes = {
    HOME: { path: '/' }
- } satisfies Routes
+ } as const satisfies Routes
```

이젠 우리가 사용한 정확한 리터럴 values까지 type 검사를 통해 훌륭한 해결책을 가지게 되었습니다. 아름답군.

```ts
const routes = {
  HOME: { path: '/' },
} as const satisfies Routes;

navigate(routes.HOME.path); // ✅ - as desired
navigate('/invalid-path'); // ❌ - as desired
```

마지막으로, 왜 `satisfies` 대신에 `as const`를 사용하지 않는가?를 물을 지도 모릅니다.

음, `as const`와 같이 객체 자체에서 any type 검사를 하지 않습니다. 즉, IDE에서 자동 완성 기능이 없거나 작성 시 오타 및 기타 문제에 대한 경고가 표시되지 않습니다.

이것이 왜 조합이 효과적인지에 대한 이유입니다.

## 가까운 libraries 와 frameworks로 이동

새로운 연산자에서 얻을 수 있는 가장 큰 이점은 인기있는 libraries와 frameworks에서 지원이 된다는 것입니다.

예를 들어, 다음과 같이 작성한 경우 NextJS를 사용합니다.

```ts
export const getStaticProps: GetStaticProps = () => {
  return {
    hello: 'world',
  };
};

export default function Page(
  props: InferGetStaticPropsType<typeof getStaticProps>
) {}
```
