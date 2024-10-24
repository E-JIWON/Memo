React-Query는 **공식 문서에서 fetching, caching, 서버 데이터와의 동기화를 지원해주는 라이브러리** 라고 정의

1. React Query는 React Application에서 서버 상태를 불러오고, 캐싱하며, 지속적으로 동기화하고 업데이트하는 작업을 도와주는 라이브러리입니다.
2. 복잡하고 장황한 코드가 필요한 다른 데이터 불러오기 방식과 달리 React Component 내부에서 간단하고 직관적으로 API 사용할 수 있습니다.
3. 더 나아가 React Query에서 제공하는 캐싱, Window Focus Refetching 등 다양한 기능을 활용하여 API 요청과 관련된 번잡한 작업 없이 "핵심 로직"에 집중할 수 있습니다.

즉, React-Query는 프론트엔드에서 비동기 데이터를 불러오는 과정 중 발생하는 문제들을 해결해준다는 건데, 어떤 문제들을 어떻게 해결한다는 걸까?

## 캐싱(Caching)
리쿼는 데이터를 캐싱한다는 점이 장점이다.
> 캐싱이란 특정 데이터의 복사본을 저장하여 이후 동일한 데이터의 재접근 속도를 높이는 것을 뜻한다.

리쿼는 캐싱을 통해 동일한 데이터에 대한 **반복적인 비동기 호출을 방지**하고, 이는 불필요한 API 콜을 줄여 서버에 대한 부화를 줄이는 좋은 결과를 가져온다.

### 아니 최신 데이터인건 어케 판별?
만일 서버 데이터를 불러와 캐싱한 후, 실제 서버 데이터를 확인했을 때 서버 상에서 데이터의 상태가 변경되어있다면, 사용자는 실제 데이터가 아닌 변경 전의 데이터를 바라볼 수밖에 없게 된다. 이는 사용자에게 잘못된 정보를 보여주는 에러를 낳는다.

 참고로, React-Query에서는 최신의 데이터를 fresh한 데이터, 기존의 데이터를 stale한 데이터라고 말한다!!