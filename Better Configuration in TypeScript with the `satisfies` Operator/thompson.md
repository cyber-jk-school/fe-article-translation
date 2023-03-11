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

당신이 마주할 수 있는 마지막 상황은, 평범하게 `satisfies`를 사용했을 때, 우리가 구성한 객체는 생각했던 것보다 더 느슨해질 수 있습니다. (is captured를 뭐라고 번역해야 할까요?)  
예를 들어, 아래 코드에서

```ts
const routes = {
  HOME: { path: "/" },
} satisfies Routes;
```

만약 우리가 `path` 속성의 타입을 체크하면, 우리는 `string` 타입을 얻습니다.

```ts
routes.HOME.path; // Type: string
```

그러나 구성과 관해서는, [const 단언](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions) (`as const`)가 빛을 발합니다. 만약 여기서 `as const`를 사용한다면 우리는 더 정확한 문자열 리터럴 `'/'` 타입을 얻을 수 있습니다.

```ts
const routes = {
  HOME: { path: "/" },
} as const;

routes.HOME.path; // Type: '/'
```

"그래, 이건 교묘한 수법인데 내가 왜 신경 써야돼"와 같이 생각할 수 있습니다. 그리나 아래와 같이 매개변수로 받는 정확한 경로까지 type-safe한 메소드를 고려해봅시다.(여기 문장을 어떻게 번역해야할 지 애매함)

```ts
function navigate(path: '/' | '/auth') { ... }
```

만약 우리가 `satisfies`만 사용한다면, 각 `path`가 문자열만 될 수 있다고 알려질 경우, 타입 에러가 발생하게 됩니다.

```ts
const routes = {
  HOME: { path: "/" },
} satisfies Routes;

navigate(routes.HOME.path);
// ❌ Argument of type 'string' is not assignable to parameter of type '"/" | "/auth"'
```

윽! `HOME.path`는 유효한 문자열이지만(`'/'`), 타입스크립트는 그렇지 않다고 말하고 있습니다.

이것이 우리가 `satisfies`와 `as const`를 결합해서 가장 좋은 결과를 얻을 수 있는 부분입니다.

```ts
  const routes = {
    HOME: { path: '/' }
- } satisfies Routes
+ } as const satisfies Routes
```

이제, 우리는 우리가 사용하는 정확한 리터럴 값까지 타입 체크하는 좋은 해결법을 얻었습니다.

```ts
const routes = {
  HOME: { path: "/" },
} as const satisfies Routes;

navigate(routes.HOME.path); // ✅ - as desired
navigate("/invalid-path"); // ❌ - as desired
```

마지막으로, 왜 `satisfies` 대신에 그냥 `as const`를 사용하면 안되는건지 물어볼 수 있습니다.  
음, `as const`만 사용하면, 우리가 그것을 작성할 때에 객체의 타입 체크를 할 수 없습니다. 그것은 우리의 IDE가 자동완성을 할 수 없거나 오타나 작성하는 동안 발생하는 다른 이슈들을 알려주지 않는다는 의미입니다.  
이것이 `satisfies`와 `as const`를 결합하는 것이 효율적인 이유입니다.

## Coming to libraries and frameworks near you (coming to 를 뭐라고 번역해야 할까요?)

당신이 이 새로운 연산자로부터 얻을 수 있는 가장 큰 이점은 당신이 사용하는 유명한 라이브러리와 프레임워크에서 지원한다는 것입니다.  
예를 들어 NextJS에서, 이전의 방식으로 작성하면

```ts
export const getStaticProps: GetStaticProps = () => {
  return {
    hello: 'world'
  }
}

export default function Page(
  props: InferGetStaticPropsType<typeof getStaticProps>
) {
  // 🚩 Typo not caught, because `props` is of type `{ [key: string]: any }`
  return <div>{props.hllo></div>
}
```

이 경우 `props`의 타입은 `{ [key: string]: any}` 입니다. 하, 도움이 되지 않네요.  
그러나 `satisfies`와 함께라면, 당신은 이렇게 작성할 수 있습니다.

```ts
export const getStaticProps = () => {
  return {
    hello: 'world'
  }
} satisfies GetStaticProps

export default function Page(
  props: InferGetStaticPropsType<typeof getStaticProps>
) {
  // 😍 `props` is of type `{ hello: string }`, so our IDE makes sure our
  // code is correct
  return <div>{props.hello}</div>
}
```

이렇게 작성하면 당신이 이전에 했던 타입 체크와 완성 기능을 얻을 수 있지만, 이제 적절한 타입 `{ hello: string }`이 됩니다.

## 결론

타입스크립트 4.9는 새로운 `satisfies` 키워드를 소개했고, 이것은 대부분의 타입스크립트의 구성 관련 작업에서 매우 편리합니다.  
표준 타입 정의와 비교해서, 이것은 타입 체크와 최적의 타입 안전성을 위한 명확한 세부 구성 정보를 이해하는 것 사이의 우아한 균형을 이룰 수 있습니다. (여기 문장이 제일 어려운 것 같아요)  
routes 예제를 제안한 [u/sauland](https://www.reddit.com/r/webdev/comments/zrt1rb/comment/j15fffv/?utm_source=share&utm_medium=web2x&context=3)와 NextJS 에제를 제안한 [Matt pocock](https://www.youtube.com/watch?v=Danki1DyiuI)과 [Lee Robinson](https://twitter.com/leeerob/status/1563540593003106306?lang=en)에게 감사합니다
