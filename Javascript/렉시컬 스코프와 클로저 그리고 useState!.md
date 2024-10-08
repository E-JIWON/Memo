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

저를 위해 쉽게 적고 싶지만 쉽지가 않다...
### **클로저의 정의**부터 알아보자.
클로저는 함수와 그 함수가 생성될 때의 **렉시컬 환경의 조합!** 쉽게말해, 내부 함수가 외부 함수의 변수에 접근할 수 있는 상태를 말한다. 이 덕분에 내부 함수는 외부 함수의 스코프에 있는 변수를 기억하고 사용할 수 있다.

### 클로저의 작동 원리

1. **변수의 생명주기**: 외부 함수가 실행되면 그 함수의 변수는 메모리에 저장되고, 외부 함수가 종료되어도 이 변수는 사라지지 않고 내부 함수가 계속 접근할 수 있다
2. **상태 유지**: 클로저를 사용하면 함수의 상태를 유지할 수 있어. 예를 들어, **카운터**를 만들 때 유용하다

### 🚀 클로저 예제 및 설명

아래 예제를 통해 클로저의 작동 방식을 살펴보자.

```javascript
function makeCounter() {
  let count = 0; // count는 makeCounter의 스코프에 속함
  
  return function() {
    count++; // 내부 함수에서 count를 접근하고 증가시킴
    return count;
  };
}

const counter = makeCounter(); // counter는 클로저를 갖는 함수
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```
- `MakeCounter()` 함수가 호출되면 `count` 변수가 메모리에 생성되고, 반환된 함수는 이 변수를 계속 기억한다.
- `counter()`를 호출할 때마다 `count`가 증가하는 것을 볼 수 있다.

#### 클로저 사용 이유 ?

자바스크립트 엔진은 렉시컬 환경에 의해 상위 스코프를 기억한다.
항상 하위에서 상위 스코프 방향으로 검색하기 때문에 상위에서 하위 방향으로는 식별자를 검색할 수 없다.
이러한 특성을 이용한 클로저는 상태를 안전하게 변경하고 유지하기 위해 사용한다.

1. 🚀 클로저 사용 X

```javascript
```


### 클로저의 다양한 활용

1. **정보 은닉**: 클로저를 사용하면 변수를 외부에서 접근하지 못하도록 숨길 수 있다.
	```javascript
	function secret() {
	let secretValue = '숨겨진 값';
	  
	  return function() {
		return secretValue; // 외부에서 접근할 수 없음
	  };
	}

	const getSecret = secret();
	console.log(getSecret()); // 숨겨진 값
	```
    
2. **다양한 상태 관리**: 여러 개의 클로저를 사용하여 각각의 상태를 독립적으로 관리
    ```javascript
    function createUser(name) {
      return {
        getName: function() {
          return name;
        },
        setName: function(newName) {
          name = newName;
        },
      };
    }
    
    const user1 = createUser('Alice');
    console.log(user1.getName()); // Alice
    user1.setName('Bob');
    console.log(user1.getName()); // Bob
    ```
    
### 클로저의 단점

1. **메모리 누수**: 클로저가 너무 많은 변수를 기억하면 메모리 사용량이 증가할 수 있다.
2. **디버깅 어려움**: 클로저를 사용할 때 변수의 생명주기가 복잡해져서 디버깅이 어려울 수 있다.
### 그러면 모든 함수가 클로저가 아녀?

    

### 클로저가 발생하는 경우

클로저는 일반적으로 다음과 같은 경우에 발생해:

javascript

```javascript
function outer() {
  let outerVar = 'I am from outer';

  return function inner() {
    console.log(outerVar); // inner는 outerVar를 참조하고 있어
  };
}

const innerFunc = outer(); // outer가 호출되면서 클로저 생성
innerFunc(); // 'I am from outer'
```

- 여기서 `inner` 함수는 `outer` 함수의 변수를 기억하고 있으니까 클로저야.

#### 참고
[제로초 강의](https://www.youtube.com/watch?v=jVP4fFtSvsg)
[poiemaweb](https://poiemaweb.com/js-scope#7-%EB%A0%89%EC%8B%9C%EC%BB%AC-%EC%8A%A4%EC%BD%94%ED%94%84)
[재능 넷](https://www.jaenung.net/tree/1560)
[damagucci-juice님 티스토리 글](https://damagucci-juice.tistory.com/entry/8%EC%9D%BC%EC%B0%A8-%ED%81%B4%EB%A1%9C%EC%A0%80)
[클로저 프로그래밍의 즐거움 책..](https://www.yes24.com/Product/Goods/24555451)
[블로그보다 쉽게 알려주시는 라매개발자님](https://www.youtube.com/watch?v=LL0DGc5pg7A)


