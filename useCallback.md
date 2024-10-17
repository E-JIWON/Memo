- React 훅(Hook) 중 하나로, 성능 최적화를ㄹ 위해 사용된다.
-
#### 1.  `useCallback`의 기본 개념
	- 메모이제이션(기억된) 콜백 함수를 반환한다.
	- 이는 불필요한 렌더링을 방지하고 성능을 향상시키는데에 도움이된다
#### 2. 기본 구문
```ts
const memoizedCallback = useCallback(
	() => {
		doSomething(a, b);
	},
	[a, b]
);
```

#### 3. 작동 방식
- `useCallback`은 두개의 인자를 받는다.
	- 콜백 함수와 의존성 배열
- 의존성 배열의 값이 변경되지 않는 한