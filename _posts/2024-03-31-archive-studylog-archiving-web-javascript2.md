---
title: "[국비지원과정] 웹표준 - JavaScript(2)"
excerpt: "국비 수강기록21. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, JavaScript, WEB]

permalink: /archive/studylog-archiving-web-javascript2/

toc: true
toc_sticky: true

date: 2024-03-31
last_modified_at: 2024-08-29
---

## 🖥️ Javascript

- HTML과 CSS로 구성된 웹 페이지를 동적으로 만들어주는 객체 기반의 스크립트 언어
- 타입을 명시할 필요가 없는 인터프리터 언어로, 모든 웹브라우저에 자바스크립트 해석기가 내장되어 있다.
- 객체 지향형 프로그래밍과 함수형 프로그래밍을 모두 표현할 수 있다.
- 자바스크립트의 활용
  - jQuery : 자바스크립트 라이브러리
  - JSON(JavaScript Object Notation) : 자바스크립트의 객체 표기법, 데이터 전달을 위한 표준 형식

<br>
<br>

### 4. Jquery

__*1. Jquery란?*__

자바스크립트 언어를 간소화, 단순화시킨 오픈소스 기반의 자바스크립트 라이브러리.<br>
문서 객체 모델 (DOM)과 이벤트에 관한 처리를 간단하게 구현할 수 있고, Ajax 응용 프로그램 등을 활용해 생산성을 향상시킬 수 있다.<br>
(\*cf. 라이브러리: 프로그램 제작 시 자주 사용하는 서브루틴이나 함수 등 여러 기능을 재활용할 수 있도록 유통가능하게 만들어 묶어둔 파일을 의미함)

__*2. Jquery 적용하기?*__
  
- http://jquery.com 에서 다운로드
  > 1.1. jQuery의 버전<br>
  > 1.x 버전 : 구버전 브라우저까지 지원되는 버전<br>
  > 2.x 버전 : 구버전 브라우저(Internet Explorer 6,7,8 등)를 지원하지 않는 버전<br>
  > 3.x 버전 : 2014년부터 개발, 더 빠르고 풍부한 API<br>
 
- jQuery를 사용하는 방법<br>
1. 다운로드하여 사용할 경우 ```<script src="jquery-3.7.1.js"></script>``` 경로를 포함한 파일명을 지정하여 설정 <br>
경로는 webapp 하위 폴더부터 시작<br>
다운로드한 파일을 webapp 하위 폴더 내부에서 원하는 위치에 복사해두고 그 위치를 경로로 지정해야 주어야함<br>

2. 다운로드 하지 않고 사용할 경우 ```<script src="http://code.jquery.com/jquery-3.7.1.js"></script>``` import
   

<br>

__*3. 기본 문법*__

선택한 요소에 어떤 동작을 수행
```
$(selector).action()
$(selector) : tag, class, id
action() : 해당 요소에서 수행할 동작
```

<br>

__*4. 선택자*__
```
태그 이름 : $("p")
id: $("#test")
class : $(".t1“)
```

<br>

__*5.이벤트 처리*__
```
$(선택자).이벤트( 실행할 코드 );
$(선택자).bind( "이벤트", 실행할 코드 );
이벤트의 종류 : click, mouseover, focus, blur 등
```
<br>

__*6. jQuery 주요 함수*__

- $(선택자).text() 텍스트 읽기
- $(선택자).text("변경할 내용") 텍스트 변경
- $(선택자).html() html 읽기
- $(선택자).html("변경할 내용") html 코드 변경
- $(선택자).val() 입력필드의 값
- $(선택자).val( "변경할 내용" ) 입력필드의 값 변경
- $(선택자).attr( "속성" ) 요소의 속성

<br>

__*7. css, html 다루기*__
```
$(선택자).css("속성") CSS 속성 읽기
$(선택자).css("속성", "값") CSS 속성 변경
```

  \* 참고 링크
<br>__참조1__ &nbsp;&nbsp; [제이쿼리 기초](https://www.tcpschool.com/jquery/jq_intro_basic)
<br>__참조2__ &nbsp;&nbsp; [제이쿼리 [ jQuery ] 기본](https://velog.io/@bi-sz/jQuery-%EA%B8%B0%EB%B3%B8)
<br>__참조4__ &nbsp;&nbsp; [jQuery 란](https://www.elancer.co.kr/blog/view?seq=176)
<br>__참조4__ &nbsp;&nbsp; [jQuery 기본문법](https://m.blog.naver.com/how1_/222620948292)
