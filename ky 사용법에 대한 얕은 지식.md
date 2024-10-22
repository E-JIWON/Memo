
### 1. fetch와 ky 기본 사용법 비교
일반 `fetch`와 `ky`의 가장 큰 차이점은 사용 편의성이다.

```js
// 일반 fetch 사용시
const response = await fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ name: 'John' }) // 데이터를 직접 문자열로 변환 해야 한다.
});
const data = await response.json();

// ky 사용시
import ky from 'ky';

const data = await ky.post('https://api.example.com/users', {
  json: { name: 'John' } // 자동으로 JSON 변환
}).json(); // 메서드 체이닝으로 간단하게 JSON  변환이 가능하다.
```

`ky` 사용 시 이점
1. JSON 데이터 자동 변환 (`JSON.stringify` 불필요)
2. 깔끔한 메서드 체이닝
3. 타입스크립트 지원이 잘 되어있음
4. 기본적인 에러 처리 제공

### 2. 기본 설정 세팅 비교
#### 1. 기본 설정 부분 비교
`fetch`와 `ky`의 기본 설정 방식 비교

```js
// ky 설정
const api = ky.create({
  prefixUrl: API_BASE_URL,    // API 기본 주소
  timeout: 10000,             // 10초 후 타임아웃
});

// fetch로 했다면
async function fetchWithConfig(endpoint, options = {}) {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options.headers,
    }
  });
  // fetch는 타임아웃 기능이 없어서 직접 구현해야 함
  const timeoutPromise = new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Request timeout')), 10000)
  );
  return Promise.race([response, timeoutPromise]);
}
```
- ky는 타임아웃 같은 기능이 내장되어 있음
- fetch는 기본적인 기능도 직접 구현해야 하는 경우가 많음
#### 2. hooks(인터셉터) 비교
ky의 가장 강력한 기능 중 하나는 hooks 시스템이다.
```js
// ky의 hooks - 깔끔하고 모듈화된 방식
const api = ky.create({
  hooks: {
    // 요청 전에 실행되는 훅
    beforeRequest: [
      request => {
        const token = localStorage.getItem('token');
        if (token) {
          request.headers.set('Authorization', `Bearer ${token}`);
        }
      }
    ],
    // 응답 후에 실행되는 훅
    afterResponse: [
      async (request, options, response) => {
        if (response.status === 401) {
          localStorage.removeItem('token');
          window.location.href = '/login';
        }
        return response;
      }
    ]
  }
});

// fetch로 구현시 - 모든 로직을 한 함수에 넣어야 함
async function fetchWithInterceptor(endpoint, options = {}) {
// beforeRequest 로직과 afterResponse 로직이 
// 하나의 함수에 혼재되어 있어 유지보수가 어려움

// beforeRequest
  const token = localStorage.getItem('token');
  const headers = {
    'Content-Type': 'application/json',
    ...(token ? { 'Authorization': `Bearer ${token}` } : {}),
    ...options.headers
  };

  try {
    const response = await fetch(`${API_BASE_URL}${endpoint}`, {
      ...options,
      headers
    });

    // afterResponse 로직
    if (response.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
      throw new Error('Unauthorized');
    }

    return response;
  } catch (error) {
    throw error;
  }
}
```

ky의 hooks 시스템의 장점
- 요청 전/후 로직을 명확하게 분리
- 모듈화된 구조로 유지보수 용이
- 여러 개의 훅을 배열로 관리 가능
- 코드의 가독성이 높음

#### 3. ky의 주요 특징들
```js
// 1. 기본 인스턴스 생성
const api = ky.create({
  prefixUrl: 'https://api.example.com',  // 기본 URL 설정
  timeout: 5000,                         // 타임아웃 설정
  headers: {                             // 기본 헤더 설정
    'Authorization': 'Bearer token'
  }
});

// 2. HTTP 메서드들
await ky.get(url);      // GET 요청
await ky.post(url);     // POST 요청
await ky.put(url);      // PUT 요청
await ky.delete(url);   // DELETE 요청

// 3. 요청 옵션 예시
const response = await ky.post('users', {
  json: { name: 'John' },           // 자동으로 JSON 변환
  searchParams: { type: 'admin' },  // URL 파라미터 추가
  timeout: 3000,                    // 이 요청만의 타임아웃
  retry: 2                          // 실패시 재시도 횟수
});

// 4. 응답 처리
const jsonData = await response.json();          // JSON으로 파싱
const textData = await response.text();          // 텍스트로 받기
const blobData = await response.blob();          // 파일 등의 바이너리 데이터

// 5. 에러 처리
try {
  const data = await ky.get('users').json();
} catch (error) {
  if (error instanceof ky.HTTPError) {
    // 서버에서 에러 응답이 온 경우 (4xx, 5xx)
    console.log(error.response.status);
  }
}
```

#### ky.create 이점
```js
// 인스턴스 생성 시 공통 설정
const api = ky.create({
  prefixUrl: 'https://api.example.com',  // 기본 URL 설정
  headers: {
    'Authorization': 'Bearer token'      // 공통 헤더
  },
  timeout: 5000                         // 공통 타임아웃
});

// 이제 짧게 사용 가능
await api.post('users', {
  json: { name: 'John' }
});

await api.post('posts', {
  json: { title: 'Hello' }
});

// 공통 설정은 유지되면서 특별한 설정도 추가 가능
await api.post('special', {
  json: { data: 'unique' },
  headers: {
    'Special-Header': 'value'  // 기존 Authorization 헤더는 유지됨
  }
});
```
- 위 방식처럼 인스턴스 만들어서 사용할 경우
	- 동일한 코드 감소, 설정 관리 편함, 실수 방지(URL이나 헤더 누락 등), 변경사항 적용이 쉬움(URL 바뀔 경우 한 곳 수정)

### 필수 요소만 넣어 공통 fetcher 만들기
```js
// lib/fetcher.ts
import ky from 'ky';

// 1. 공통으로 사용할 ky 인스턴스 생성 (필수 설정만)
const api = ky.create({
  prefixUrl: process.env.NEXT_PUBLIC_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// 2. GET 요청 함수
export const getData = async <T>(endpoint: string) => {
  try {
    const response = await api.get(endpoint).json<T>();
    return response;
  } catch (error) {
    console.error('Get request failed:', error);
    throw error;
  }
};

// 3. POST 요청 함수
export const postData = async <T>(endpoint: string, data: any) => {
  try {
    const response = await api.post(endpoint, {
      json: data
    }).json<T>();
    return response;
  } catch (error) {
    console.error('Post request failed:', error);
    throw error;
  }
};
```