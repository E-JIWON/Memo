```mermaid
graph TD
    subgraph Next.js13
        A1[컴포넌트] -->|개별 fetch| B1[서버]
        B1 -->|응답| A1
        A1 -->|수동 캐시 관리| C1[로컬 상태]
    end
    subgraph Next.js14
        A2[컴포넌트] -->|자동 중복제거 fetch| B2[서버]
        B2 -->|응답| A2
        B2 -->|자동 캐싱| C2[서버 캐시]
    end
    subgraph ReactQuery
        A3[컴포넌트] -->|useQuery| B3[React Query]
        B3 -->|캐시된 데이터| A3
        B3 -->|필요시 fetch| C3[서버]
        C3 -->|응답| B3
        B3 -->|자동 캐시 관리| D3[Query 캐시]
    end