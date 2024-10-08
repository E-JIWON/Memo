## 💡렉시컬 스코프 이해하기

변수가 어디에 선언 되었는지에 따라서 그 변수를 검색하는 방식을 말한다.
자바스크립트에서는 함수가 정의된 위치에 따라 변수를 찾기 때문에, 외부 함수에서 정의된 변수를 내부 함수에서도 사용할 수 있다. 이게 바로 렉시컬 스코프다.

> [!NOTE] 📒 그러니까
> 함수가 정의될 때 외부 변수를 기억하고 실행될 때가 아니라 정의될 때의 스코프를 기준으로 변수에 접근한다는 것이다~
## 🖥️ 코드 예제
### ✏️ 첫번째 예제

```javascript
var name = 'bong' // log()가 실행된 다음에 name이 chhil로 바뀜

function log() {
  console.log(name); // bong
}
function wrapper() {
  name = 'chil';
  log();
}
wrapper();
```
- 위 코드에서 <span style="color:CornflowerBlue">`log()`</span> 함수는 `name`을 내부 스코프부터 외부 스코프로 순차적으로 접근한다.
- 처음에는 <span style="color:CornflowerBlue">`log()`</span> 함수 내부에서 name 변수를 찾지만, 내부에 없으므로 한 단계 바깥인 전역 스코프의 `name` 변수를 참조하게 된다.
- <span style="color:CornflowerBlue">`wrapper()`</span> 함수 실행 후에는 `name`이 <span style="color:indianred">chil</span>로 변경되어 있으므로, <span style="color:CornflowerBlue">`log()`</span> 함수는 <span style="color:indianred">chil</span>을 출력


> [!NOTE] 💡 스코프 체인
> - 스코프 체인은 변수를 찾기 위해 함수가 어떤 순서로 스코프를 탐색하는지 나타낸다.
> - 자스에서 변수를 찾을 때 **자신의 스코프 → 부모 스코프 → 최종적으로 전역 스코프**까지 올라가는 구조를 따르는디 
> - 이 과정에서 각 스코프가 연결되어 **체인처럼 동작**하며, 함수 내부에서 변수를 찾지 못하면 바깥쪽 스코프로 이동하며 계속 탐색하는 것을 말한다.


### ✏️ 두번째 예제
wrapper 함수 안에 별도의 name 변수가 있을 경우

```javascript
var name = 'bong'

function log() {
  console.log(name); // bong
}
function wrapper() {
  var name = 'chil'; // wrapper 함수의 지역 스코프에 새로운 'name' 변수를 선언
  log();
  
  
  // 여기서 헷갈리는 이유가 
  // log안에 있는 console.log(name)이 여기있는게 아님 !
}
wrapper();
```
🌚 코드 상으로 볼 경우 `chil` 으로 출력될 줄 알았지만,,,,,

- <span style="color:CornflowerBlue">`wrapper()`</span> 함수 안에서 새로운 `name` 변수를 선언하면, <span style="color:CornflowerBlue">`wrapper()`</span> 함수는 자신이 정의된 시점에서의 스코프를 따라 여전히 전역 스코프의 `name`을 참조한다.
- **따라서 'chil'이 아닌 'bong'이 출력**된다. 
- 함수가 정의된 위치를 기준으로 변수 스코프가 결정된다는 점을 보여준다.- 


#### 참고
[제로초 강의](https://www.youtube.com/watch?v=jVP4fFtSvsg)
[poiemaweb](https://poiemaweb.com/js-scope#7-%EB%A0%89%EC%8B%9C%EC%BB%AC-%EC%8A%A4%EC%BD%94%ED%94%84)