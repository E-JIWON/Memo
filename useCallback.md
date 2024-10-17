- React 훅(Hook) 중 하나로, 성능 최적화를ㄹ 위해 사용된다.
-
#### 1.  `useCallback`의 기본 개념
 - 메모이제이션(기억된) 콜백 함수를 반환한다.
- 이는 불필요한 렌더링을 방지하고 성능을 향상시키는데에 도움이된다
  💡다른 함수에 인자로 전달되는 함수(콜백함수)가 처음 호출 시 기억해두었다가 나중에 호출 시에도 기억되어있는 함수여서 성능을 향상 시키는데 도움이 된다는 뜻
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
- 의존성 배열의 값이 변경되지 않는 한, 동일한 함수 인스턴스를 반환한다.
	- 💡 인스턴스란 메모리에 저장된 실제 데이터를 인스턴스라고 부른다.
	- 💡 동일한 인스턴스란 : 같은 메모리 주소를 가리키는 것이다. 내용이 같아도 새로 만들면 다른 인스턴스가 된다.
	- 💡 왜 중요하지?  

#### 4.주요 사용 사례
- 자식 컴포넌트에 콜백을 전달할 때, 특히 자식 컴포넌트가 React.memo()로 최적화 되어있을 때 유용
- 의존성 배열에 포함된 값이 변경될 때만 함수를 재생성하고 싶을 때 사용.

```tsx
...임포트 생략
function parentComponent() {
	const [count, setCount] = useState(0);

	const increment = useCallback(() => {
		setCount(c => c + 1);
	}, [])  // 빈 의존성 배열: 함수가 재생성되지 않

	return (
		<>
			Count : {count}
			<ChildCompoentn onIncrement={increment} />
		</>
	)
}

const ChildComponent = React.memo(({ onIncrement }) => { 
	console.log("ChildComponent rendered"); 
	return <button onClick={onIncrement}>Increment</button>; 
});
```

#### 6. useCallback 과 useMemo의 차이
- useCallback(fn, deps)는 useMemo(() => fn, dept)와 동등합니다
- useCallback은 함수 자체를 메모이제이션하고, useMemo는 함수의 결과값을 메모이제이션 합니다.

#### 7. 주의사항
- 모든 함수에 `useCallback`을 사용하는 것은 불필요할 수 있습니다. 실제로 성능 향상이 필요한 경우에만 사용
- 의존성 배열 주의

#### 8. 성능 고려사항:
- `useCallback` 자체도 약간의 오버헤드가 있다. 매우 간단한 함수나 자주 변경되는 의존성이 있는 경우에는 사용하지 않는 것이 더 효율적일 수 있다.
- 