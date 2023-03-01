# `satisfies` 연산자를 통한 더 나은 타입스크립트 구성

![메인 이미지](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F2459dee391814240898b0e772bf8d2f9?format=webp&width=2000)

타입스크립트에서 `type-safe`한 구성을 위한 새롭고 더 나은 방법이 있습니다.

그것은 타입스크립트 4.9에서 릴리즈 된 새로운 `satisfies` 연산자입니다.

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: "/auth",
  },
} satisfies Routes; // 😍
```

## `satisfies`가 필요한 이유

위의 예시를 표준 타입 정의를 사용해 다시 작성해봅시다.

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes: Routes = {
  AUTH: {
    path: "/auth",
  },
};
```

지금까진 모든 게 괜찮아 보입니다. 우리는 IDE에서 적절한 타입 체크와 완성(?) 기능을 얻습니다.
그러나, 우리가 routes 객체를 사용할 때, 컴파일러(그리고 그 결과 우리의 IDE도)는 실제로 구성된 routes를 알지 못합니다.
예를 들어, 아래는 컴파일은 제대로 되지만, 런타임에러 에러를 반환합니다.

```js
// 🚩 타입스크립트 컴파일은 되지만, `routes.NONSENSE`는 실제로 존재하지 않음.
routes.NONSENSE.path;
```

왜 그럴까요? 이것은 우리의 `Routes` 타입이 키로 어떤 문자열이든 받을 수 있기 때문입니다. 그렇기 때문에 타입스크립트는 간단한 오타부터 말이 안되는 키를 포함한 모든 종류의 키 접근을 허용합니다. 또한 우리는 우리의 IDE로부터 유효한 route 키를 자동완성 해주는 도움을 받을 수 없습니다.  
당신은 "그럼 `as` 키워드는 어때요?"라고 말할 수 있습니다. 그러면 그것도 간단하게 살펴볼 수 있습니다.

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: "/auth",
  },
} as Routes;
```

이것은 타입스크립트에서 일반적인 관행이지만, 사실 상당히 위험합니다.

위에서와 같은 문제가 있을 뿐만 아니라, 존재하지 않는 키값을 사용할 수 있고 TypeScript will look the other way(여기를 어떻게 번역해야 할 지 모르겠어요)

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: "/auth",
    // 🚩 타입스크립트 컴파일은 제대로 되지만, 이것은 유효한 속성이 아닙니다!
    nonsense: true,
  },
} as Routes;
```

넓게 말하면, 보통의 경우 타입스크립트에서 `as` 키워드를 사용하는 것을 피하세요.

## `Satisfies`가 목숨을 살린다

이제, 우리는 마침내 위의 예시를 새로운 `satisfies` 키워드로 재작성할 수 있습니다.

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: "/auth",
  },
} satisfies Routes;
```

이를 통해, 우리는 우리가 바라던 모든 적절한 타입 체크를 할 수 있습니다.

```ts
routes.AUTH.path; // ✅
routes.AUTH.children; // ❌ routes.auth에는 `children` 속성이 없습니다.
routes.NONSENSE.path; // ❌ routes.NONSENSE는 존재하지 않습니다.
```

그리고 물론 IDE 자동완성도 됩니다.
![자동완성](https://cdn.builder.io/api/v1/image/assets%2FYJIGb4i01jvw0SRdL5Bt%2F4cd04efbeb764f6cb29bfce5cf091be9?format=webp&width=2000)

우리는 아래와 같은 좀 더 복잡한 예제도 다룰 수 있습니다.

```ts
type Route = { path: string; children?: Routes };
type Routes = Record<string, Route>;

const routes = {
  AUTH: {
    path: "/auth",
    children: {
      LOGIN: {
        path: "/login",
      },
    },
  },
  HOME: {
    path: "/",
  },
} satisfies Routes;
```

이전에 사용했던 좀 더 일반적인 방식과 반대로, 정확히 우리가 구성한 대로 IDE의 자동완성과 모든 객체 트리에 대한 타입 체크를 할 수 있습니다.
![동영상](./thompson-video.gif)

```ts
routes.AUTH.path; // ✅
routes.AUTH.children.LOGIN.path; // ✅
routes.HOME.children.LOGIN.path; // ❌ routes.HOME에 `children` 속성이 없습니다.
```

완벽하게 우리가 원하던 것입니다.

## `as const`와 결합
