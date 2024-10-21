### Next.14 fetch
일단 사용한 코드부터 보겠습니다.

```js
async function getUser(id) {
  const res = await fetch(`https://api.example.com/user/${id}`, {
    next: { revalidate: 60 } // 60초마다 데이터 재검증
  });
  
  if (!res.ok) throw new Error('Failed to fetch user');
  return res.json();
}

export default async function UserProfile({ params }) {
  const user = await getUser(params.id);
  return <div>
    <h1>{user.name}의 프로필</h1>
    <p>이메일: {user.email}</p>
  </div>
}
```

#### Next 14 fetch 특징
- 서버 컴포넌트에서 직접 `async/await`를 사용할 수 있다.
- `fetch`함수는 **자동으로 결과를 캐시**한다.
- `next : revalidate : 60` 옵션으로 60초마다 데이터를 재검증한다.
- 아래와 같은 흐름이다.
	- ![[Pasted image 20241021152721.png]]
	- Next 서버가 외부 API에 `fetch` 요청을 보낸다.
	- 외부 API가 데이터를 반환
	- Next 서버가 데이터로 HTML을 렌더링하여 브라우저에 전송한다.
	- 

1. 자동 재검증(AUtomatic Revalidation)
	- Next 14에서는 `fetch` 옵션에 `next.revalidate`를 사용하지 않아도 된다.
	- 대신 라우트 세그먼트 설정을 통해 전체 라우트나 레이아웃에 대한 재검증을 설정할 수 있다.
2. 캐싱 동작
	- Next 14에서는 기본적으로 모든 `fetch`요청이 캐시됩니다. 캐시를 비활성화 하려면 `{cache: 'no-store}`옵션을 사용해야한다.
	- Next 13에서는 동적 함수 (예: `header()`, `cookies()` )를 사용할 때 자동으로 동적 렌더링으로 전환되었지만, 14에서는 명시적으로 `{ cache: 'no-store }` 옵션을 설정해야 한다.
3. 새로운 옵션
	- Next 14에서는 `fetch`에 `{ next: {tags: ['tag1' , 'tag2`