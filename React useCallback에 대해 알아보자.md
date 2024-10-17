React 훅(Hook) 중 하나로, 성능 최적화를 위해 사용된다.
바로 개념부터 알아보자.

### 1.  `useCallback`의 기본 개념
 - 메모이제이션(기억된) 콜백 함수를 반환한다.
	 - 💡 메모이제이션이란? 이전에 계산한 값을 저장해두고 재사용하는 기법이다.
- 이는 불필요한 렌더링을 방지하고 성능을 향상 시키는 데에 도움이 된다.
	- 💡다른 함수에 인자로 전달되는 함수(콜백 함수)가 처음 호출 시 기억해두었다가 나중에 호출 시에도 기억되어있는 함수여서 성능을 향상 시키는데 도움이 된다는 뜻

***
### 2. 기본 구문
```ts
const memoizedCallback = useCallback(
	() => {
		doSomething(a, b);
	},
	[a, b]
);
```

***
### 3. 작동 방식
- `useCallback`은 두 개의 인자를 받는다.
- 콜백 함수와 의존성 배열
- 의존성 배열의 값이 변경되지 않는 한, 동일한 함수 인스턴스를 반환한다.
	- 💡 인스턴스란 메모리에 저장된 실제 데이터를 인스턴스라고 부른다.
	- 💡 동일한 인스턴스란 : 같은 메모리 주소를 가리키는 것이다. 내용이 같아도 새로 만들면 다른 인스턴스가 된다.
	- 💡 왜 중요하지? : React는 Props가 변경되었는지 확인할 때 객체의 참조(메모리 주소)를 보고 비교하는데, 같은 인스턴스면 "변경 없음"으로 판단하고 불필요한 재렌더링을 안하기 때문이야 !

***
### 4.주요 사용 사례
- 자식 컴포넌트에 콜백을 전달할 때, 특히 자식 컴포넌트가 React.memo()로 최적화 되어있을 때 유용
	- 💡React.memo: 컴포넌트의 `props`가 변경되지 않으면 재 렌더링을 막아주는 기능이다.
	- 💡왜 `useCallback`이랑 궁합이 좋냐 ?: `useCallback` 없이 그냥 함수를 `props`로 넘겨버리면, 부모 컴포넌트가 렌더링 될 때마다 새 함수가 만들어지는데, 그럼 `memo`로 감싼 자식 컴포넌트도 매번 재 렌더링된다. `useCallback`을 쓰면 함수가 새로 만들어지지 않아서 자식 컴포넌트의 불필요한 재렌더링을 막을 수 있다.
	
- 의존성 배열에 포함된 값이 변경될 때만 함수를 재 생성하고 싶을 때 사용된다.

```tsx
...임포트 생략
function parentComponent() {
	const [count, setCount] = useState(0);

	// 1. 빈 의존성 배열: 컴포넌트가 처음 렌더링될 때만 함수 생성
	const increment = useCallback(() => {
		setCount(c => c + 1);
	}, []) 
	
	// 2. count가 변경될 때마다 함수 재생성
	const incrementWithLog = useCallback(() => { 
		console.log(`Count is now ${count + 1}`);
		setCount(c => c + 1); }, [count]);
	}, [count]);

	return (
		<>
			<p>Count: {count}</p>
			<p>Other State: {otherState}</p>
			<ChildComponent onIncrement={increment} />
			<ChildComponentWithLog onIncrementWithLog={incrementWithLog} />
			
			<button onClick={() => setOtherState(s => s + 1)}>Update Other State</button>
		</>
	)
}

const ChildComponent = React.memo(({ onIncrement }) => { 
	console.log("ChildComponent rendered"); 
	return <button onClick={onIncrement}>Increment</button>; 
});
const ChildComponentWithLog = React.memo(({ onIncrementWithLog }) => {
	console.log("ChildComponentWithLog rendered");
	 return <button onClick={onIncrementWithLog}>Increment with Log</button>;
 });
```
  💡 
- `increment` 함수는 항상 같은 인스턴스를 참조하므로 `ChildComponent`는 `count`나 `otherState`가 변경되어도 리 렌더링되지 않는다.
- `incrementWithLog` 함수는 `count`가 변경될 때마다 새로운 인스턴스가 생성되므로 `count`가 변경될 때마다 `ChildComponentWithLog`가 리 렌더링된다.
- `otherState`가 변경되면 `ParentComponent`는 리렌더링되지만, 두 자식 컴포넌트 모두 재렌더링되지 않는다.
*** 
### 5. useCallback을 사용하지 않았을 때의 예시
```ts
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  // 이 함수는 ParentComponent가 리렌더링될 때마다 새로 생성됨
  const onClickUpCount = () => setCount(c => c + 1);
  
  return (
    <div>
      Count: {count}
      <ChildComponent onIncrement={onClickUpCount} count={count} />
    </div>
  );
}
```
- 💡 `count`가 변경될 때마다 `ParentComponent`가 리렌더링된다.
  리렌더링될 때마다 `onClickUpCount` 함수가 새로 생성된다.
  결과적으로 `ChildComponent`가 매번 리렌더링된다.

-  주의사항
- `ChildComponent`가 복잡하거나 무거운 연산을 포함하고 있다면, 불필요한 리렌더링으로 성능이 저하될 수 있다.
- `count`만 사용하고 `onClickUpCount`는 사용하지 않는 경우에도 함수가 새로 생성되어 리렌더링이 발생한다.

***
### 6. useCallback과 useMemo의 차이
- `useCallback(fn, deps)`는 `useMemo(() => fn, deps)`와 동등하다.
- `useCallback`은 함수 자체를 메모이제이션하고, `useMemo`는 함수의 결과값을 메모이제이션한다.

```tsx
// useCallback
const memoizedFunction = useCallback(() => {
  return expensiveCalculation(a, b);
}, [a, b]);

// useMemo
const memoizedResult = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a, b]);
```
- `useCallback`은 함수 정의 자체를 기억하고, `useMemo`는 함수 실행 결과를 기억한다.
- `useCallback`은 이벤트 핸들러나 자식 컴포넌트에 전달할 콜백 함수를 최적화할 때 주로 사용한다.
- `useMemo`는 계산 비용이 높은 값을 메모이제이션할 때 사용한다.

***
### 7. 주의사항
- 모든 함수에 `useCallback`을 사용하는 것은 불필요할 수 있다. 실제로 성능 향상이 필요한 경우에만 사용해야 한다.
- 의존성 배열을 올바르게 관리해야 한다. 필요한 의존성을 누락하면 오래된 클로저 문제가 발생할 수 있다.

- 💡 아래는 클로저 문제이다 확인해보자.
```tsx
const [count, setCount] = useState(0);

// 잘못된 사용: count를 의존성 배열에 추가하지 않음
const incorrectCallback = useCallback(() => {
  console.log(`Count: ${count}`);
}, []); // 항상 초기 count 값(0)을 참조

// 올바른 사용
const correctCallback = useCallback(() => {
  console.log(`Count: ${count}`);
}, [count]); // count가 변경될 때마다 함수 재생성	
```

***
### 8. 성능 고려사항:
- `useCallback` 자체도 약간의 오버헤드가 있다. 
  매우 간단한 함수나 자주 변경되는 의존성이 있는 경우에는 사용하지 않는 것이 더 효율적일 수 있다.
- 대규모 목록을 렌더링하거나 복잡한 계산이 필요한 컴포넌트에서 특히 유용하다.

***
### 9. 실제 사용 시나리오:
1. 대규모 목록 렌더링: 각 항목에 대한 이벤트 핸들러를 `useCallback`으로 최적화
2. 데이터 페칭: 의존성이 변경될 때만 API 호출을 다시 하도록 `useCallback` 사용
3. 차트나 그래프 컴포넌트: 데이터 처리 함수를 `useCallback`으로 최적화

```tsx
const DataGrid = ({ data }) => {
  const handleRowClick = useCallback((id) => {
    console.log(`Row ${id} clicked`);
  }, []);

  return (
    <div>
      {data.map(item => (
        <Row 
          key={item.id} 
          data={item} 
          onClick={() => handleRowClick(item.id)}
        />
      ))}
    </div>
  );
};
```
💡 이 예제에서 `handleRowClick`은 `useCallback`으로 메모이제이션되어, `data`가 변경되어도 각 `Row` 컴포넌트가 불필요하게 재렌더링되는 것을 방지한다.

***
### 마무리
... 어렵다!
