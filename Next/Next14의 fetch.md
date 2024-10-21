Next.js 14가 출시되면서 데이터 `fetching` 방식에 큰 변화가 있다.
이번 글에서는 Next.js14의 새로운 `fetch`기능을 살펴보고, 이전 버전 및 `React Query`와 비교해 보겠습니다.
또한, Next.js 14 환경에서 React Query를 사용하는게 적절한가 ? 와 효과적으로 사용하는 방법에 대해서도 알아보겠습니다.

### 1. Next 14의 새로운 fetch 기능
Next 14에서는 내장 `fetch`함수에 중요한 개선사항이 추가 되었다.
- **자동 중복 제거** :동일한 요청은 한 번만 실행된다.
- **자동 캐싱** : 결과가 자동으로 캐시된다.

```tsx
// app/(home)/page.js
async function getData() {
  const res = await fetch('https://api.example.com/data');
  
export default async function Page() {
  const data = await getData()
  return <main>{/* 데이터를 사용하여 UI 렌더링 */}</main>
}
```
- 이 코드에서 `fetch` 함수는 **자동으로 결과를 캐시**하고, **동일한 요청이 여러 번 발생해도 한 번만 실행**됩니다.

### 2. Next 13, Next 14 fetch 비교
```mermaid
sequenceDiagram
    participant C as Component
    participant F13 as Next 13 Fetch
    participant F14 as Next 14 Fetch
    participant S as Server
    participant Cache as Cache

    C->>F13: 데이터 요청
    F13->>S: 매번 새로운 요청
    S-->>F13: 응답
    F13-->>C: 데이터 반환

    C->>F14: 데이터 요청
    F14->>Cache: 캐시 확인
    alt 캐시 있음
        Cache-->>F14: 캐시된 데이터
    else 캐시 없음
        F14->>S: 새로운 요청
        S-->>F14: 응답
        F14->>Cache: 캐시 저장
    end
    F14-->>C: 데이터 반환
```
- 위 다이어그램에서 볼 수 있듯이, Next.js 14의 `fetch`는 **자동 캐싱**과 **중복 제거 기능**을 제공하여 **불필요한 서버 요청을 줄이고 성능을 향상** 시킨다.

### Next.js 13의 한계와 React Query의 등장
💡 해당 부분은 Next 14버전에서 React Query를 쓰는 이유를 찾으려면, Next 13 이전 버전에서 사용한 이유를 알아야 할 것 같아서 추가했습니다.

💡 Next.js 13에서는 데이터 fetching에 몇 가지 한계가 있었습니다
- **수동 캐싱 관리:** 개발자가 직접 캐싱 로직을 구현해야 했습니다.
- **중복 요청 가능성:** 동일한 데이터에 대해 여러 번 요청이 발생할 수 있었습니다.
- **복잡한 상태 관리:** 로딩, 에러 상태 등을 수동으로 관리해야 했습니다.

💡 이러한 이슈들 때문에 React Query같은 서버 상태 관리 라이브러리를 사용했습니다. (React Query만 있는게 아닙니다!)
	- 자동 캐싱 및 재검증
	- 중복 요청 방지
	- 로딩 및 에러 상태 자동 관리
	- 데이터 동기화 및 백그라운드 업데이트
	- 예제 코드
```js
// React Query 사용 예시
import { useQuery } from 'react-query'

function MyComponent() {
  const { data, isLoading, error } = useQuery('myData', fetchData)

  if (isLoading) return <div>로딩 중...</div>
  if (error) return <div>에러 발생!</div>

  return <div>{data}</div>
}
```

### Next 14와 React Query 적절한 사용
Next 14에서 데이터 fetching이 크게 개선되었기 때문에, 많은 경우에 React Query를 사용하지 않아도 ㄷ
