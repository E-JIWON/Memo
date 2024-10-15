 문서 객체 모델(Document Object Model)은 XML을 HTML에서 사용할 수 있도록 확장한 애플리케이션 프로그래밍 인터페이스(Application Programming Interface)입니다.
DOM은 전체 페이지를 노드 계층 구조로 변환합니다
```html
<html>
	<head>
		<title>sample page</title>
	</head>
	<body>
		<h1>Hello World!</h1>
	</body>
</html>
```
- 이 코드는 아래처럼 나타낼 수 있습니다.
![[Pasted image 20241015163250.png]]\
- DOM은 문서를 표현하는 트리를 생성하고, DOM API 를 통해 노드를 제거하고, 추가하고, 교체할 수 있습니다.

브라우저 객체 모델(BOM)
브라우저 창에 접근하고 조작할 수 있게하는 인터페이스입니다.
