
### 1. fetch()와 ky  기본 사용법 비교
```js
// 일반 fetch 사용시
const response = await fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ name: 'John' })
});
const data = await response.json();

// ky 사용시
import ky from 'ky';

const data = await ky.post('https://api.example.com/users', {
  json: { name: 'John' }
}).json();
```
- 이유랑 자세한 내용 설명 필요할 것으로 보임

### 2. 기본 설정 세팅 비교
#### 1. 기본 설정 부분 비교
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

#### 2. ky에서 사용되는 hooks 비교
아래 내용은 요청 전에 실행되는 훅(코드 내용 - token 확인 후 header에 set)과, 응답 후에 실행되는 훅(401일 경우 token 삭제 후 /login)을 fetch로 했을 때 비교이다.
```js
// ky의 hooks
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

// fetch로 구현한다면
async function fetchWithInterceptor(endpoint, options = {}) {
  // beforeRequest 로직
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
- ky는 ~특징이 있어서 ~  설명

### 3. ky의 주요 특징들
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