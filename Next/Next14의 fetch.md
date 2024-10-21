
### Next.14 fetch

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

### Next 13, Next 14, React-Query 데이터 흐름도 비교
```mermaid
graph TD
    subgraph Next.js13
        A1[컴포넌트] -->|개별 fetch| B1[서버]
        B1 -->|응답| A1
        A1 -->|수동 캐시 관리| C1[로컬 상태]
    end
    subgraph ReactQuery
        A3[컴포넌트] -->|useQuery| B3[React Query]
        B3 -->|캐시된 데이터| A3
        B3 -->|필요시 fetch| C3[서버]
        C3 -->|응답| B3
        B3 -->|자동 캐시 관리| D3[Query 캐시]
    end
    
    subgraph Next.js14
        A2[컴포넌트] -->|자동 중복제거 fetch| B2[서버]
        B2 -->|응답| A2
        B2 -->|자동 캐싱| C2[서버 캐시]
    end
```
- Next 14 
	- 클라이언트와 서버 컴포넌트 모두 동일한 `fetch` 매커니즘 사용한다.
	- 자동 중복 제거 기능이 내장되어 있다.
	- 서버 측에서 자동 캐싱이 이루어진다.
	- 재 검증 옵션을 통해 캐시 갱신을 제어 할 수 있다.
- Next.js 13:
    - 클라이언트와 서버 컴포넌트가 개별적으로 fetch 요청을 수행한다.
    - 캐싱은 수동으로 관리해야 합니다.
    - 중복 요청 방지를 위해 추가 로직이 필요할 수 있습니다.
- Next.js 14:
    - 클라이언트와 서버 컴포넌트 모두 동일한 fetch 메커니즘을 사용합니다.
    - 자동 중복 제거 기능이 내장되어 있습니다.
    - 서버 측에서 자동 캐싱이 이루어집니다.
    - 재검증 옵션을 통해 캐시 갱신을 제어할 수 있습니다.
- React Query:
    - 컴포넌트는 useQuery 훅을 통해 데이터를 요청합니다.
    - React Query가 중앙에서 캐시를 관리합니다.
    - 자동 재시도, 백그라운드 갱신 등의 고급 기능을 제공합니다.
    - invalidateQueries를 통해 캐시를 수동으로 무효화할 수 있습니다.
    - 
- Next.js 14는 13버전에 비해 더 간단하고 효율적인 데이터 흐름을 제공합니다. 그러나 React Query는 여전히 더 복잡한 클라이언트 사이드 상태 관리 시나리오에서 강력한 도구로 남아있습니다.

프로젝트의 요구사항에 따라 Next.js 14의 내장 기능만으로 충분할 수도 있고, React Query의 고급 기능이 필요할 수도 있습니다. 때로는 두 접근 방식을 혼합하여 사용하는 것이 최적의 솔루션이 될 수 있습니다.