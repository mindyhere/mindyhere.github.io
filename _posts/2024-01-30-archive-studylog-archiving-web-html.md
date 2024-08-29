---
title: "[국비지원과정] 웹표준 - HTML"
excerpt: "국비 수강기록17. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, html, WEB]

permalink: /archive/studylog-archiving-web-html/

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

## 🖥️ HTML(Hyper Text Markup Language)

### 1. HTML의 개요

- __HTML__ : 웹페이지를 만들기 위한 언어, 1990년 발표
- __웹브라우저__ : HTML 소스를 해석하여 화면에 출력시키는 프로그램 (크롬, 인터넷 익스플로러, 엣지 등)

<br>

### 2. HTML 문서의 기본 구조

- __DOCTYPE 선언__<br>
HTML 문서의 버전을 표시. <br>
웹 페이지의 최상단에 위치하는 태그로, DOCTYPE을 선언하여 웹 페이지가 어떤 버전의 HTML, XHTML을 사용하는지 명시한다. <br>
` <!DOCTYPE html>` 은 문서가 HTML5 버전을 지켜 작성된 문서임을 뜻한다.<br>
	
  ```html
    <!DOCTYPE html>
    <html>
      <head>
          <meta charset="UTF-8">
          <title>Document</title>
      </head>
      <body>
          <h2>Hello html</h2>
      </body>
    </html>
  ```  

- __HTML 태그의 형식__

  ```html
  //<태그 속성="값">
  <img src = "my.png">
  ```

<br>

### 3. 기본 태그

|<태그>|설명|
|----|----|
|`<br>`|줄바꿈(line Break)|
|`<p>` |문단 나누기(Paragraph)|
|`<h1> ~ <h6>`|제목(Headline)|
|`<hr>`|수평선 
|__\*특수문자__<br>`&quot;`<br>`&amp;`<br>`&lt;` / `&gt;`<br>`&nbsp;`|<br>큰따옴표(")<br>앰퍼센드(&)<br>꺽쇠(< / >)<br>공백| 
|__\*리스트__<br>`<ul>`<br>`<ol>`<br>`<li>`| <br>번호없는 리스트(Unordered List)<br>번호있는 리스트(Ordered List)<br>항목|
|__\*테이블__<br>`<caption>`<br>`<th>`<br>`<tr>`<br>`<td>`<br>`<colspan>`<br>`<rowspan>`|<br>테이블 제목<br>테이블의 제목행(Table Header)<br>테이블의 행(Table Row)<br>테이블의 셀(Table Division)<br>셀 병합<br>행 병합|

<br>
<br>

\* 참조 링크<br>
__참조1__ &nbsp;&nbsp; [웹 표준과 웹 접근성](https://velog.io/@jos9187/%EC%9B%B9-%ED%91%9C%EC%A4%80%EA%B3%BC-%EC%9B%B9-%EC%A0%91%EA%B7%BC%EC%84%B1%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C)
<br>__참조2__ &nbsp;&nbsp; [HTML5 알아보기](https://velog.io/@sgyoon/2020-04-12)
<br>__참조3__ &nbsp;&nbsp; [HTML 구조 관련 요소](https://kde66034.tistory.com/30)

