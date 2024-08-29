---
title: "[국비지원과정] 웹표준 - JavaScript(1)"
excerpt: "국비 수강기록19. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, JavaScript, WEB]

permalink: /archive/studylog-archiving-web-javascript1/

toc: true
toc_sticky: true

date: 2024-02-06
last_modified_at: 2024-08-29
---

# 🌏 웹표준(Web Standards)

- 웹에서 표준적으로 사용되는 기술이나 규칙
- 표준화 단체인 W3C가 권고한 표준안에 따라 웹사이트를 작성할 때 이용하는 HTML, CSS, JavaScript 등에 대한 규정  
- 어떤 운영체제나 브라우저를 사용하더라도 웹페이지가 동일하게 보이고 정상 작동해야함을 의미한다.


<br>
<br>

## 🖥️ Javascript

- HTML과 CSS로 구성된 웹 페이지를 동적으로 만들어주는 객체 기반의 스크립트 언어
- 타입을 명시할 필요가 없는 인터프리터 언어로, 모든 웹브라우저에 자바스크립트 해석기가 내장되어 있다.
- 객체 지향형 프로그래밍과 함수형 프로그래밍을 모두 표현할 수 있다.
- 자바스크립트의 활용
  - jQuery : 자바스크립트 라이브러리
  - JSON(JavaScript Object Notation) : 자바스크립트의 객체 표기법, 데이터 전달을 위한 표준 형식

<br>
<br>

### 1. Javascript  개요 : 기본문법 

__*1. 소스코드 작성*__

웹 브라우저는 HTML의 구조와 CSS 스타일을 렌더링하는 도중 스크립트를 만나게 되면 스크립트의 해석과 구현이 완료될때까지 브라우저 렌더링을 멈추게 된다.<br>
즉, 스크립트의 삽입 위치가 스크립트 실행순서와 브라우저 렌더링에 영향을 미치기 때문에 적절한 위치선정을 고려하여 코드를 작성해야 한다.

- inline : 태그 내부에 직접 작성

  ```js
  // 예시
  <input type="button" onclick="alert('ok')">
  ```



- 내부 자바스크립트	

  ```js
    // 예시
    <script>
    	자바 스크립트 코드
	</script>
  ```

- 외부 자바스크립트	
  ```js
  // 예시
  <script src="자바스크립트파일.js"></script>
  ```

- 함수

  ```js
	// 함수 이름이 있는 경우  
    function 함수이름(매개변수){
    	실행코드;
    }
    
    // 함수 이름이 없는 경우(무명함수)
    function(매개변수) {
    	실행코드; 
    }
  ```  
    
- 자바스크립트의 대화상자<br>
`alert("메시지");` 단순 메시지박스<br>
`prompt("메시지", "기본값");` 입력받은 값을 변수에 저장<br>
`confirm("메시지")` 사용자의 확인을 받을 경우<br>

<br>

__*2. 변수선언*__

- `var 변수이름;` &nbsp;&nbsp; or &nbsp;&nbsp; `let 변수이름=값;`
- `const 상수이름=값;`
    ```js
       let 변수이름       let ex1      //  선언
      =  할당할 값       = "hello";     //  할당
    ```

<br>

__*3. HTML 요소에 접근하는 방법*__

- __id__ 로 접근 : `document.getElementById("태그의 id")`
- __name__ 으로 접근 : `document.getElementsByName("태그의 name")`
 
<br>

__*4. 출력문*__

- `window.alert()` : 브라우저와는 별도의 대화 상자를 띄워 사용자에게 데이터를 전달

  ```javascript
  // (ex)
  <script>
      function check() {
          let name=document.getElementById("name");

          if (name.value == "") {
              alert("input your name");	// alert창(확인창) 호출
              name.focus();	// 입력란으로 커서이동
              return;
          }
  </script>
  ```

- `document.write()` : HTML 문서에 출력
  
  ```javascript
  // (ex)
  <script>
    document.write(4 * 5);
	// () 안의 연산결과를 웹브라우저에 출력함
  </script>
  ```

- `innerHTML` : HTML 문서의 특정부분에 출력. 기존 HTML 소스를 유지하고, 특정요소를 찾아 해당부분만 코드를 해석해 출력한다.

  ```javascript
  // (ex)
  <script>
  	document.getElementById("result").innerHTML = 5 + 6;
  </script>

  <body>
  <h2>Hello World</h2>
  	<div id="result"></div>
	//  5 + 6의 연산결과를 출력함
  </body>
  ```
	
    \*cf. `innerText` : Element 내에서 사용자에게 보여지는 텍스트값을 가져오거나 설정할 수 있다. 이때 태그를 해석하지 않고 텍스트 그대로 가져온다는 점에서 `innerHTML` 과 차이가 있다.

- `console.log()` : 웹 브라우저의 콘솔을 통해 데이터를 출력한다.
  
  ```javascript
  // (ex)
  function msg() {
 	 console.log("msg 함수 호출...");
    //F12 개발자도구의 콘솔창에 출력내용 확인
  }
  ```


<br>
<br>

### 2. 문서구조와 이벤트 처리

__*1. 문서 객체 모델(DOM, Document Object Model)*__

XML이나 HTML 문서에 접근하기 위한 인터페이스의 일종.<br>
문서 내의 모든 요소를 정의하고, 각각의 요소에 접근하는 방법을 제공한다.

- document 객체 : 웹페이지의 최상위 객체. 웹페이지 그 자체.
  - id로 요소 선택 → `getElementById("태그의 id")`
  - 태그 내부의 내용 → `태그.innerHTML`
  - 태그의 입력값 → `태그.value`
  - 태그의 속성 변경 → `태그.src = "값"`
  - 태그의 스타일 변경 → `태그.style.속성이름 = "속성값"`

- window 객체
  - 팝업창 열기 : `window.open(url, 윈도우의 name, 옵션)`
  - 타이머 설정 : `setTimeout()`&nbsp or &nbsp `setInterval()`
    ```js
    setTimeout( 코드, 밀리세컨드 )  //한번만 호출
    setInterval( 코드, 밀리세컨드 )  //반복적으로 호출
    ```
  - 포커스 이동 : `focus()`

<br>

__*2. 주요 이벤트*__

지정된 타입의 이벤트가 특정 요소에서 발생하면, 웹 브라우저는 그 요소에 등록된 이벤트 리스너/핸들러(event listener or event handler)를 통해 처리한다.<br>
이때 이벤트 리스너는 인수로 이벤트 객체(event object)를 전달받으며, 식별자를 통해 전달받은 이벤트 객체를 참조한다.

  이벤트명|설명|이벤트명|설명
  ---|---|---|---
  click | 클릭 시|change | 변동사항이 있을 때
  load | 페이지 로딩이 완료되었을 때|unload|페이지가 언로드 되었을 때
  mouseover | 마우스가 특정 객체 위에 올라갔을 때|mouseout | 마우스가 특정 객체 밖으로 벗어났을 때
  mousedown|마우스를 클릭했을 때|mousemove|마우스가 움직였을 때
  focus | 포커스를 얻었을 때|blur | 포커스를 잃었을 때
  keydown | 키를 입력할 때|keyup|키 입력을 완료했을 때
  select|option 태그 등에서 값을 선택했을 때 |submit|form 제출 요청할 때

<br>

__*3. 정규표현식(Regular Expression)*__

문자열에서 특정한 규칙을 가지는 문자열의 집합을 찾아내기 위한 검색 패턴.

  | 코드 | 설명 | 코드 | 설명 |
  |----|----|----|----|
  | s | 공백 문자 | d | 숫자|
  | D | 숫자가 아닌 문자<br> [^0-9] 와 같은 표현|w | 알파벳, 숫자<br> [A-Za-z0-9]|
  | W | w의 반대<br> [^A-Za-z0-9]|특수문자 | 특수문자 자체<br> 예) + (+: plus기호)|
  | * | 0회 이상 반복|+ | 1회 이상 반복|
  | ? | 0 또는 1개의 문자 매칭|. | 정확히 1개 문자 매칭|
  | g | 전역매칭|m | 여러 라인 매칭|
  | i | 대소문자 무시|\| | 또는|
  | {} | 반복 횟수|


<br>
<br>

### 3. 그래픽처리

__*canvas*__

`<canvas id="아이디" width="가로길이" height="세로길이"></canvas>`
- canvas : 도화지. (화면의 좌측상단 → canvas의 원점)
- context : canvas에 그래픽을 출력하는 객체. 붓

```javascript
//(ex)
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>Line ex.</title>
	<script>
		function init() {
			//canvas 그림판, context 붓(도구)
			canvas = document.getElementById("myCanvas");
			context = canvas.getContext("2d");
			
			context.setLineDash([10, 4]);	//[선굵기, 공백사이즈]
			
			context.strokeStyle = "blue";
			context.beginPath();	//선 그리기 시작
			context.moveTo(0, 200);	//시작좌표
			context.lineWidth = 20;	//굵기
			context.lineTo(100, 100);	//이동
			context.stroke();	//선그리기 마무리
			
			context.beginPath();
			context.moveTo(100, 100);
			context.strokeStyle = "red";
			context.lineWidth = 10;
			context.lineTo(150, 50);
			context.lineTo(300, 200);
			context.stroke();
		}
	</script>
	
	<style>
		body {
			margin: 0;
		}
	</style>
</head>

<body onload="init()">
	<canvas id="myCanvas" width="300px" height="200px"></canvas>
</body>
</html>

```

<br>
<br>




  \* 참고 링크
<br>__참조1__ &nbsp;&nbsp; [프론트엔드 자바스크립트 기초](https://dinfree.com/lecture/frontend/123_js_1.html)
<br> __참조2__ &nbsp;&nbsp; [자바스크립트 개요](https://www.tcpschool.com/javascript/intro)
<br> __참조4__ &nbsp;&nbsp; [Java Script 기초 및 문법](https://www.codestates.com/blog/content/javascript-%EA%B8%B0%EC%B4%88-%EB%B0%8F-%EB%AC%B8%EB%B2%95)