
### 1. 일반 패치와 기본 사용법 비교
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
아래 내용은 요청 전에  token 확인 후 header에 set하는 내용과, 응답 후에
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
