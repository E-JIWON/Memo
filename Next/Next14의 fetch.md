Next 14와 Next 13의 `fetch`함수 사용의 차이점

1. 자동 재검증(AUtomatic Revalidation)
	- Next 14에서는 `fetch` 옵션에 `next.revalidate`를 사용하지 않아도 된다.
	- 대신 라우트 세그먼트 설정을 통해 전체 라우트나 레이아웃에 대한 재검증을 설정할 수 있다.
2. 캐싱 동작
	- Next 14에서는 기본적으로 모든 `fetch`요청이 캐시됩니다. 캐시를 비활성화 하려면 `{cache: 'no-store}`옵션을 사용해야한다.
	- Next 13에서는 동적 함수 (예: `header()`, `cookies()` )를 사용할 때 자동으로 동적 렌더링으로 전환되었지만, 14에서는 명시적으로 `{ cache: 'no-store }` 옵션을 설정해야 한다.
3. 새로운 옵션
4.