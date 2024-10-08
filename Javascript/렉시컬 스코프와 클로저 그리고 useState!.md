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
- wrapper() 함수는 **자신이 정의된 시점**에서의 스코프를 따라가기 때문에 여전히 전역 스코프의 `name`을 참조하기
- **따라서 'chil'이 아닌 'bong'이 출력**된다. 
- 함수가 정의된 위치를 기준으로 변수 스코프가 결정된다는 점을 보여준다.- 


#### 참고
[제로초 강의](https://www.youtube.com/watch?v=jVP4fFtSvsg)
[poiemaweb](https://poiemaweb.com/js-scope#7-%EB%A0%89%EC%8B%9C%EC%BB%AC-%EC%8A%A4%EC%BD%94%ED%94%84)