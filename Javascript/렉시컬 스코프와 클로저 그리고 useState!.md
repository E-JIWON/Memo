# 💡렉시컬 스코프 이해하기

- **정의** : 함수가 정의된 위치에 따라 변수를 검색하는 방식
- **특징** : 내부 함수는 자신을 포함하고 있는 외부 함수의 변수를 기억할 수 있다.
- 🌚 흠... 그러니까 함수가 정의될 때 외부 변수를 기억하고 실행될 때가 아니라 정의될 때의 스코프를 기준으로 변수를 접근! 한다는 것이다.

***
## 🚀 코드 예제 및 설명
### ✏️ 첫번째 예제

```javascript
var name = 'bong' // log()가 실행된 다음에 name이 chhil로 바뀜

function log() {
  console.log(name); // bong
}
function wrapper() {
  name = 'chil';
  log(); // chill
}
wrapper();
```
- `log()` 함수는 **전역 스코프에 정의**된 `name` 변수를 참조한다.
- `wrapper()`함수 실행 후에는 `name`이 **chil**로 변경되어 있으므로, `log()` 함수는 **chil**을 출력

***
### ✏️ 두번째 예제
`wrapper()` 함수 안에 별도의 `name` 변수가 있을 경우 ?

```javascript
var name = 'bong'

function log() {
  console.log(name); // bong
}
function wrapper() {
  var name = 'chil'; // 새로운 지역 변수를 선언
  log();
  
  
  // 여기서 헷갈리는 이유가 
  // log안에 있는 console.log(name)이 여기있는게 아님 !
}
wrapper();
```
🌚 코드 상으로 볼 경우 `chil` 으로 출력될 줄 알았지만,,,,,
- `wrapper()` 내부에서 새로운 `name` 변수를 선언하더라도, `log()` 함수는 전역 스코프의 `name`을 참조한다.
	`wrapper()` 함수는 **자신이 정의된 시점**에서의 스코프를 따라가기 때문에 전역 스코프의 `name`을 참조한다. 따라서 `chil이 아닌 bong`이 출력된다.

# 💡 스코프 체인 이해하기

- **스코프 체인**: 변수를 찾기 위해 함수는 자신의 스코프에서 시작하여 부모 스코프, 최종적으로 전역 스코프까지 올라간다.
- **구조**: **자신의 스코프 -> 부모 스코프 -> 전역 스코프** 순으로 탐색한다.
- 🌚 이 과정에서 스코프가 연결되어 체인처럼 동작하여 스코프 체인이라 한다 🌚

# 💡드디어 클로저를 이해해보자.
쉽게 설명하고싶다

클로저는 함수와 그 함수가 선언될 때의 환경이 함께 기억되는 구조다.

클로저는 함수와 그 함수가 선언될 때의 환경이 함께 기억되는 구조를 말합니다. 이는 특히 내부 함수가 외부 함수의 변수를 참조할 때 중요합니다.



#### 참고
[제로초 강의](https://www.youtube.com/watch?v=jVP4fFtSvsg)
[poiemaweb](https://poiemaweb.com/js-scope#7-%EB%A0%89%EC%8B%9C%EC%BB%AC-%EC%8A%A4%EC%BD%94%ED%94%84)