# [번역] memo() 하기 전에

원문: https://overreacted.io/before-you-memo/

React 성능 최적화에 대한 글이 많이 있다. 일반적으로, state 업데이트가 느리면, 다음과 같은 것을 해야합니다:

1. production 빌드가 동작 중인지 확인합니다. (Development 빌드들은 극단적인 상황일때 엄청나게 의도적으로 느려집니다.)

2. 필요 이상으로 state를 tree에서 높은 부분에 놓여있는지 확인해야 합니다. (예를 들어, input state를 중앙 store에 놓는 것은 최고의 방법이 아닐 수 있습니다.)

3. React DevTools Profiler을 실행하여 re-rendered을 확인하고 가장 expensive subtrees를 `memo()`로 감쌉니다. (그리고 필요한 곳에 `useMemo()`를 추가합니다.)

특히, 마지막 방법은 components 사이에 있는 경우 성가시더라도 미래에는 이성적으로 compiler가 너를 위해 해줄 지도 모릅니다.

이 글에서는, 2가지 다른 기술을 공유하려고 합니다. 이것들은 놀랄만큼 기초적인 것이여서, 사람들이 이것들로 인해 rendering 성능이 향상된다는 것을 잘 알지 못합니다.

이 기술들은 당신이 이미 알고 있는 것들을 보안해줍니다! 이것들이 `memo` 또는 `useMemo`를 대처하지는 못하지만 먼저 시도해 보면 좋습니다.

## 인위적으로 느린 Component
