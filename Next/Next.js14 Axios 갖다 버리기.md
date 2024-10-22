#Next14 #Fetch #라이브러리 #ky
### 😶‍🌫️ 들어가며..
Next.js 14 환경에서 발생하는 `axios` 이슈에 대해 정리해보았다.
현재 `axios`를 사용하면 다음과 같은 주요 문제점들이 발생하고 있다.
- 서버 컴포넌트에서 window 객체 참조 문제
- Next.js의 자체 캐싱 시스템과의 충돌
- JavaScript가 비활성화된 환경에서 무한 로딩 이슈


### axios와 fetch의 통신 차이
`axios`는 `XMLHttpRequest`를 기반으로 만들어진 라이브러리이고,
`fetch`는 브라우저 내장 `Web API`라는 점이 가장 큰 차이점이다.

실제 사용에서도 큰 차이를 보인다.
```js
// axios는 자동으로 JSON 처리
const response = await axios.get('/api/data')
console.log(response.data)

// fetch는 명시적 JSON 변환 필요
const response = await fetch('/api/data')
const data = await response.json()
console.log(data)
```
axios가 자동 JSON 변환이나 인터셉터 같은 편리한 기능을 제공하지만,
fetch는 브라우저와 더 가까운 저수준 API이기 때문에 Next.js의 서버 컴포넌트와 더 잘 어울린다는 것을 알게 되었다.

### fetch를 사용하려고 한다..
fetch를 사용하면 얻을 수 있는 장점이 여러 가지였다.
우선 **Server Components와의 호환성**이 뛰어나고, Next.js의 캐싱 시스템과도 자연스럽게 통합된다. 
더불어 Next.js 13부터 도입된 `parallel` `routes`나 `intercepting routes`와 같은 고급 라우팅 기능들도 fetch API를 사용할 때 더 안정적으로 동작한다는 것을 확인했다.

### ky 라이브러리 선택 이유 
`axios` 대신 `fetch`를 사용할 때 아래와 같은 기능들을 기본적으로 지원하지 않는다.


1. `fetch`는 HTTP 오류 상태 코드(404, 500 등)를 자동으로 `throw` 하지 않는다.
   -> 대신 `response.ok` 속성을 확인하여 요청 성공 여부를 확인해야 한다.
   
2. `fetch`는 응답 및 `request body`를 자동적으 JOSN 으로 변환하지 않는다.
   -> `response.json()` 메서드를 호출하여 수동으로 반환해야 한다.
   
3. `fetch`는 기본적으로 타임아웃 설정이 없다.
   -> `AbortController` 사용하여 직접 구현해야 한다.
   
4. `fetch`는 요청 및 응답 인터셉터가 없다.
   -> 요청 전 공통 로직을 수동으로 추가해야한다.
   
이러한 이유로 나는 `fetch` 기반이면서도 위 기능들을 모두 지원하는 `ky` 라이브러리를 선택했다.
ky는 fetch의 장점은 그대로 가져가면서 부족한 부분을 보완해주는 완벽한 대안이었다.

#### ky 코드 훔쳐보기
```js
import ky from 'ky'

const api = ky.create({
  prefixUrl: 'https://api.example.com',
  timeout: 5000,
  hooks: {
    beforeRequest: [
      request => {
        request.headers.set('Authorization', `Bearer ${getToken()}`)
      }
    ]
  }
})

// 자동 JSON 변환 & 에러 처리
try {
  const data = await api.get('users').json()
} catch (error) {
  // 4xx, 5xx 에러가 자동으로 catch됨
  console.error('API 에러:', error)
}
```
ky는 번들 사이즈도 axios보다 작고(~4KB), TypeScript 지원도 훌륭하며, Next.js의 서버 컴포넌트와도 완벽하게 호환된다. 이는 fetch API를 기반으로 하면서도 개발자 경험을 크게 향상시켜주는 현대적인 HTTP 클라이언트라고 할 수 있다.