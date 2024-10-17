- React 훅(Hook) 중 하나로, 성능 최적화를 위해 사용된다.
-
#### 1.  `useCallback`의 기본 개념
 - 메모이제이션(기억된) 콜백 함수를 반환한다.
- 이는 불필요한 렌더링을 방지하고 성능을 향상 시키는 데에 도움이 된다.
  💡다른 함수에 인자로 전달되는 함수(콜백 함수)가 처음 호출 시 기억해두었다가 나중에 호출 시에도 기억되어있는 함수여서 성능을 향상 시키는데 도움이 된다는 뜻
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
- `useCallback`은 두 개의 인자를 받는다.
- 콜백 함수와 의존성 배열
- 의존성 배열의 값이 변경되지 않는 한, 동일한 함수 인스 턴스를 반환한다.
	- 💡 인스턴스란 메모리에 저장된 실제 데이터를 인스턴스라고 부른다.
	- 💡 동일한 인스턴스란 : 같은 메모리 주소를 가리키는 것이다. 내용이 같아도 새로 만들면 다른 인스턴스가 된다.
	- 💡 왜 중요하지? : React는 Props가 변경되었는지 확인할 때 객체의 참조(메모리 주소)를 보고 비교하는데, 같은 인스턴스면 "변경 없음"으로 판단하고 불필요한 재렌더링을 안하기 때문이야 !

#### 4.주요 사용 사례
- 자식 컴포넌트에 콜백을 전달할 때, 특히 자식 컴포넌트가 React.memo()로 최적화 되어있을 때 유용
	- 💡React.memo: 컴포넌트의 `props`가 변경되지 않으면 재 렌더링을 막아주는 기능이다.
	- 💡왜 `useCallback`이랑 궁합이 좋냐 ?: `useCallback` 없이 그냥 함수를 `props`로 넘겨버리면, 부모 컴포넌트가 렌더링 될 때마다 새 함수가 만들어지는데, 그럼 `memo`로 감싼 자식 컴포넌트도 매번 재 렌더링된다. `useCallback`을 쓰면 함수가 새로 만들어지지 않아서 자식 컴포넌트의 불필요한 재렌더링을 막을 수 있다.
- 의존성 배열에 포함된 값이 변경될 때만 함수를 재 생성하고 싶을 때 사용.

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
- 💡 그러니까 지금 이 함수가 `count`가 변화되어서 리 렌더링이 되는데, `increment`함수를 props로 전달되고 있기 때문에, 자식 요소도 리렌더링이 되고 리렌더링이 되면 함수는 새로운 인스턴스를 생성한다.
- 하지만 `useCallback`으로 함수를 전달하고 있어서 같은 인스턴스를 참조를 하고 있기 때문에 리렌더링이 되지 않는다.
- 추가로, `ChildComponent`도 
#### 6. useCallback 과 useMemo의 차이
- useCallback(fn, deps)는 useMemo(() => fn, dept)와 동등합니다
- useCallback은 함수 자체를 메모이제이션하고, useMemo는 함수의 결과값을 메모이제이션 합니다.

#### 7. 주의사항
- 모든 함수에 `useCallback`을 사용하는 것은 불필요할 수 있습니다. 실제로 성능 향상이 필요한 경우에만 사용
- 의존성 배열 주의

#### 8. 성능 고려사항:
- `useCallback` 자체도 약간의 오버헤드가 있다. 매우 간단한 함수나 자주 변경되는 의존성이 있는 경우에는 사용하지 않는 것이 더 효율적일 수 있다.
- 