---
title: "[국비지원과정] 웹표준 - CSS"
excerpt: "국비 수강기록18. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, CSS, WEB]

permalink: /archive/studylog-archiving-web-css/

toc: true
toc_sticky: true

date: 2024-01-30
last_modified_at: 2024-08-29
---

# 🌏 웹표준(Web Standards)

- 웹에서 표준적으로 사용되는 기술이나 규칙
- 표준화 단체인 W3C가 권고한 표준안에 따라 웹사이트를 작성할 때 이용하는 HTML, CSS, JavaScript 등에 대한 규정  
- 어떤 운영체제나 브라우저를 사용하더라도 웹페이지가 동일하게 보이고 정상 작동해야함을 의미한다.


<br>
<br>

## 🖥️ CSS(Cascading Style Sheets)

HTML로 만들어진 콘텐츠에 레이아웃과 디자인요소를 정의하는 기술, 웹페이지의 디자인 요소를 지원하는 문법(Style sheet 언어)<br>
디자인 부분은 CSS로 분리하는 것이 유지보수에 유리하며, 템플릿의 형태로 확장할 수 있다. → 웹사이트 전체의 일관성을 유지할 수 있다.<br>
자바스크립트와 연계해 콘텐츠의 내용이나 디자인을 동적으로 처리할 경우에도 유용하게 사용된다.

<br>

### 1. CSS  개요

__*1. 기본문법 : 선택자와 선언부*__ 

`태그 { 속성1:값1 ; 속성2:값2 ; ... }`
  - 선택자 : 스타일을 지정할 HTML 요소(태그, 아이디 등)
  - 선언부 : CSS 속성 이름과 값
    
    ```css
    p { background-color:yellow; }	
    // p 태그의 배경색상을 yellow로 설정
    
    h1 { color:blue; font-size:12px; }	
    //h1 태그의 폰트 색, 크기 설정
    ```

<br>

__*2. 선택자(selector)*__ : 스타일을 적용할 요소, 대상

|선택자|예시|설명|
|----|----|----|
|__태그__|태그 {속성:값}<br>`p {background-color:yellow;}` |문서 내 모든 `<p>` 태그 영역 선택|
|__태그, 태그__|`div, p {속성:값}` |그룹 선택자<br>모든 `<div>` 와 `<p>` 태그 영역 선택|
|__태그 태그__|`div p {속성:값}` |`<div>` 태그 안에 있는 모든 `<p>` 태그 영역 선택|
|__.class__|.class {속성:값}<br>`.type1 {color:red;}`|`class="type1"` 인 된 모든 태그영역 선택|
|__#id__ |#id {속성:값}<br>`#special {color:red;}`|`id="special"` 인 된 모든 태그영역 선택|
|__*__ |*|문서 내 모든 요소를 선택|

<br>

__*3. CSS 활용*__

- inline CSS : 태그 내부에 style 속성으로 지정
`<p style="background-color:yellow;">`
- 내부 스타일 시트 → `<head>` 태그 내부에 작성
- 외부 스타일 시트 → `<link type="text/css" rel="stylesheet" href="스타일시트파일">`
  
  ```css
  //include/mystyle.css → 스타일시트파일
  @CHARSET􀀁"UTF-8";
  h1 {color:red;}
  p {color:green;}
  ```

  ```html
  <!DOCTYPE􀀁html>
  <html>
    <head>
      <meta charset="UTF-8">
      <title>Document</title>
      <link type="text/css" href="../include/mystyle.css" rel="stylesheet">
    </head>
    <body>
      <h1>Heading</h1>
        <p>Paragrah</p>
    </body>
  </html>
  ```

<br>
<br>

### 2. CSS3 박스 모델

HTML 내의 요소들은 사각형의 박스 형태로 출력된다. → border(경계선), margin(바깥쪽 여백), padding(안쪽 여백)

__*1. border (경계선)*__

`border-style` : none , dotted , dashed , solid , double 등<br>
`border-width`<br>
`border-color`<br>
`border-radius`<br>

<br>

__*2. 요소의 크기*__

`width` : 가로 길이<br>
`height` : 세로 길이<br>

<br>

__*3. margin(바깥쪽 여백)*__

`margin-top`, `margin-right`, `margin-bottom`, `margin-left` 

	// 상하좌우 여백 설정(시계방향으로)
    // margin: top right bottom left
    margin: 10 20 30 40;
    
    // 상하좌우 여백을 동일하게 설정
    margin: 10;

<br>

__*4. padding(안쪽 여백)*__

`padding-top`, `padding-right`, `padding-bottom`, `padding-left`
	
    // 상하좌우 여백 설정(시계방향으로)
    // padding: top right bottom left
    padding: 10 20 30 40;
    
    // 상하좌우 여백을 동일하게 설정
    padding: 10;
    
<br>

__*5. 정렬 방식*__

`text-align` : left, center, right, justify    

<br>

__*6. 배경 설정*__

`background-color` : 배경 색상<br>
`background-image: '이미지의 경로/url'` : 배경 이미지

<br>

__*7. 하이퍼링크 스타일*__

`a:link` 클릭하지 않은 링크<br>
`a:visited` 클릭한 링크<br>
`a:hover` 마우스가 올라갔을 때<br>
`a:active` 마우스로 클릭할 때

<br>
<br>

### 3. CSS의 레이아웃

__*1. block 요소와 inline 요소*__

- block 요소 : 한 줄에 한개 요소만 배치할 수 있다. → h, p, ul, li, table, div, form 태그 등<br>
\*ex. `태그 { display:block; }`

- inline 요소 : 한 줄에 여러개 요소를 배치할 수 있다. → a, img, input, span 태그 등<br>
\*ex. `태그 { display:inline; }`

<br>

__*2. 좌표 지정*__

정적 배치(static) : 정상적인 흐름에 따라 배치&nbsp → &nbsp`태그 { position:static }`<br>
상대적 배치(relative) : 정상적인 흐름을 기준으로 상대적인 좌표에 배치&nbsp → &nbsp`태그 { position:relative }`<br>
절대좌표 배치(absolute) : 전체 페이지를 기준으로 절대 좌표에 배치&nbsp → &nbsp`태그 { position:absolute }`<br>
고정좌표 배치(fixed) : 스크롤에 관계없이 고정된 좌표에 배치&nbsp → &nbsp`태그 { position:fixed }`

<br>

__*3. float(배치 방향)*__

`태그 { float:left; }` : 왼쪽에 배치<br>
`태그 { float:right; }` : 오른쪽에 배치

<br>

__*4. 레이어와 z-index*__

`태그 { z-index:값 }`&nbsp → &nbsp<u>__z-index__</u>가 클수록 높은 우선순위를 갖는다.<br>
즉, <u>__z-index__</u>가 클수록 상대적으로 위쪽/앞쪽에 표시되고 z-index가 낮으면 다른 요소들에 의해 가려지게 된다.

<br>

__*5. opacity(투명도)*__

```css
태그 { opacity:0.0~1.0 } 
// 0.0(투명) ~ 1.0(불투명) 사이값
```

<br>

__*6. visible(가시성)*__

`태그 { visibility:visible; }` : 화면에 표시<br>
`태그 { visibility:hidden; }` : 화면에서 숨김

<br>
<br>

### 4. 애니메이션 효과

- 이동(transition)

  ```css
  //태그 { transition:전환속성 시간; }
  div { transition: width 5s; }
  ```
    
- 평행이동(translate)
  
  ```css
  translate(100px, 0px) // x축 100px, y축 0px 이동
  scale(1.2, 1.2) // x축 1.2배, y축 1.2배 확대
  rotate(30deg) // 30도 회전
  ```

- 비틀기(skew)

  ```css
  skew(30deg, 20deg); // x축 30도, y축 20도 비틀기
  ```

<br>
<br>

  \* 참고 링크
<br> __참조1__ &nbsp;&nbsp; [HTML+CSS](https://nozeroslope.tistory.com/category/Programming/HTML%2BCSS)
<br> __참조2__ &nbsp;&nbsp; [CSS를 HTML에서 사용](https://www.edwith.org/htmlcss/lecture/16610?isDesc=false)
<br> __참조3__ &nbsp;&nbsp; [CSS 기초](https://webdir.tistory.com/338)