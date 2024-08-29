---
title: "[국비지원과정] JSP(1)"
excerpt: "국비 수강기록16. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, JSP]

permalink: /archive/studylog-archiving-jsp1/

toc: true
toc_sticky: true

date: 2024-01-25
last_modified_at: 2024-08-29
---

## 1. 웹프로그래밍

웹브라우저를 통하여 웹서비스를 처리하는 프로그래밍 기법
- 웹서버 : 정적인 웹서비스를 제공하는 서버(html, css, image 등)
- 웹애플리케이션 서버(WAS) : 동적인 웹서비스를 제공하는 서버
- 데이터베이스 서버 : 데이터베이스에 저장된 자료를 제공하는 서버

<br>

> __\* cf. 웹프로그래밍 언어의 종류__<br>
>__ASP(Active Server Pages)__ : 마이크로소프트의 VB 언어 기반<br>
>__PHP(Personal HomePage, Professional Hypertext Preprocessor)__<br>
>__JSP(Java Server Pages)__ : 자바 언어 기반<br>
>__ASP.net__ : VB.net , C#.net 기반

<br>

## 2. JSP 기초

1. 서블릿과 JSP <br>
자바의 웹개발 표준
   - 서블릿(Servlet) : 서버에서 실행되는 자바 코드
   - JSP(Java Server Pages) : server에서 실행되는 자바 웹페이지

1. jsp의 기본 문법
	-	scriptlet(스크립틀릿)
    ```<% 자바코드 영역 %>```
	- expression(표현식) : 출력 기능<br>
```<%=식 or 값%>```<br>
```<% out.println(식 or 값); %>```

1. jsp page에서의 import 방법
    ```java    
    //ex. 1행에 1개 클래스를 import
    <%@ page import="java.text.SimpleDateFormat" %>
    <%@ page import="java.util.Calendar" %>   
    
    
	//ex. 1행에 여러 클래스를 import
    <%@ page import="java.text.SimpleDateFormat, java.util.Calendar" %>
	```


4. page 모듈화

	```java
    <%@ include file="페이지 주소" %>
	//1개의 클래스로 컴파일 됨(변수 공유 가능)
    
    <jsp:include page="페이지 주소" />
	//2개의 클래스로 컴파일 됨(변수 공유 x , 정적인 페이지)
	```

<br/>

## 3. 폼 데이터 처리

__*1. html 입력 양식*__
	
    <form action="수신주소" method="전송방식">
    	전송할 데이터들
    </form>
    
<br/>

__*2. 데이터의 전송방식*__

- post 방식<br>
body를 통해 정보 전송<br>
정보가 주소창에 노출되지 않음<br>
대용량 자료 전송 가능

- get 방식<br>
데이터를 header에 붙여서 전송(주소창에 표시)<br>
기본적인 방식, 보안에 취약<br>
\*cf. query string : url 에 덧붙여서 전달되는 데이터, 물음표 뒤의 데이터(?변수명=값&변수명=값 형태)

<br/>

## 4. 내장객체

1. JSP의 내장 객체<br>
request : 사용자의 요청을 처리하는 객체<br>
response : 서버의 응답을 처리하는 객체<br>
out : 웹브라우저에 출력 처리하는 객체<br>
session : 사용자의 인증 정보를 저장하는 객체<br>
application : 서버의 정보를 저장하는 객체<br>
exception : 에러 처리 관련 객체<br>
config : jsp의 환경정보를 처리하는 객체<br>
page : 현재 페이지의 정보를 처리하는 객체

2. 변수의 사용범위<br>
pageContext : 현재 페이지<br>
request : 요청+응답 페이지(2페이지)<br>
session : 사용자 변수(로그인~로그아웃)<br>
application : 서버 변수(모든 사용자)

<br/>

## 5. 에러처리

__*1. 에러 처리*__

에러가 발생하면 개발자를 위해 소스 코드 및 스택 추적 정보가 화면에 노출됨<br>
보안성 향상 및 사이트를 방문하는 사용자들에게 친숙한 안내 화면 제공을 위해 에러 처리가 필요함<br/>

__*2. http status code*__

|에러코드|의미|설명|
|:------:|:---:|---|
|1XX|Informational (정보)|요청을 받고 처리 중에 있음|
|2XX|Success (성공)|요청을 정상적으로 처리함|
|3XX|Redirection (리디렉션)|요청 완료를 위해 추가 동작이 필요함|
|4XX|Client Error (클라이언트 오류)|클라이언트 요청을 처리할 수 없어 오류 발생|
|5XX|Server Error (서버 오류)|서버의 논리적인 오류 발생(→ 톰캣의 console 확인)|

<br/>

__*3. 주요 상태코드*__

|상태코드|메세지|설명|
|------|---|---------|
|200|OK|요청 정상 처리|
|400|Bad Request|클라이언트의 요청 구문이 잘못된 경우|
|401|Unauthorized|요청 처리를 위해 HTTP 인증정보가 필요함|
|403|Forbidden|접근 금지 응답. <br>Directory Listing 요청(서버 파일 디렉토리 목록 표시) 및 지 접근 등을 차단하는 경우의 응답.<br>(파일 시스템 퍼미션 거부, 허가되지 않은 IP 주소를 통한 액세스의 거부 등)|
|404|Not Found|클라이언트가 요청한 리소스가 서버에 없음|
|500|Internal Server Error|서버에서 클라이언트 요청을 처리 중에 에러 발생|
|503|Service Unavailable|서버가 일시적으로 요청을 처리할 수 없음<br>서버가 과부하 상태이거나 점검중이므로 요청을 처리할 수 없는 상태|

<br/>

## 6. 데이터베이스 연동실습ava DataBase Connectivity)

__*1. context.xml 코드작성*__

```xml
<?xml􀀁version="1.0"􀀁encoding="UTF-8"?>
<Context>
	<WatchedResource>
		WEB-INF/web.xml
	</WatchedResource>
	<Resource name="oracleDB"
              auth="Container"
      		  driverClassName="oracle.jdbc.driver.OracleDriver"
		      maxTotal="50"
     		  maxIdle="10"
              maxWaitMillis="-1"
    		  type="javax.sql.DataSource"
      		  url="jdbc:oracle:thin:@localhost:1521/xe"
		      username="아이디" password="패스워드"/>
 	 <!-- 
		name → 코드에서 참조할 변수명,
		maxTotal → 초기 dbcp size,
		maxIdle → 유휴숫자,
		maxWaitMillis → 대기시간(-1:연결 즉시발급),
		url → 연결문자열 , username/password → 아이디/패스워드
		 -->
</Context>
```

<br/>

__*2. 라이브러리 추가*__
Dynamic Web Project 프로젝트 생성 ▶ /webapp/WEB-INF/lib 디렉토리에 __ojdbc8.jar__ 추가

<br/>

## 7. AJAX(Asynchronous JavaScript And XML)

__*1. 동기식과 비동기식*__

- 동기식(Synchronized) : 요청을 보낸 후 해당 요청의 응답을 받아야 다음 동작을 실행하는 방식<br>
먼저 시작된 하나의 작업이 끝날 때까지 다른 작업을 시작하지 않고 기다렸다가 다 끝나면 새로운 작업을 시작함. <br>
작업이 직렬로 배치되어 실행되며 작업 실행의 순서가 확실히 정해져있다.<br>
- 비동기식(Asynchronized) : 요청을 보낸 후 응답과 관계없이 다음 동작을 실행할 수 있는 방식<br>
웹페이지를 리로드 하면서 발생되는 불필요한 리소스 낭비를 줄이고, 필요한 데이터만 불러오는 것이 가능함.


__*2. Ajax*__

- 클라이언트와 서버 간에 XML 데이터를 주고받는 기술<br>
- 자바스크립트를 이용해 서버와 브라우저가 비동기 방식으로 데이터를 교환할 수 있는 통신기능<br>
(\*ex. 키워드 검색 기능, 드래그 앤 드롭 등)

  ```html
  //사용 예
  $.ajax({
    type: "post",
    url: "login.do",
    data: params,
    success: function(data) { }
  });
  ```

<br/>

## 8. 쿠키와 세션

### 1. HTTP 프로토콜의 특징

클라이언트가 서버에 요청(Request)을 했을 때, 그 요청에 맞는 응답(Response)을 보낸 후 연결을 끊는 처리방식.<br>
웹에서는 다른 페이지로 이동하면 현재 페이지에 저장된 값들이 모두 소멸된다.<br>
Connectionless(비연결지향), Stateless(상태정보 유지 안함)<br>

> \*cf. 비연결성의 장단점
>- 장점 : 서버에 접속한 클라이언트 수가 많아도 서버의 부담이 적다.
>- 단점 : 정보를 유지해야 할 부분도 있는데 정보를 유지할 수 없음

즉, 다수의 페이지로 구성된 웹애플리케이션에서 사용자의 편의를 위해 인증을 유지할 필요가 있기 때문에, 이러한 비연결성의 단점을 보완하기 위해 __쿠키 / 세션__ 을 사용함

<br/>

### 2. 쿠키와 세션

__*(1) 쿠키(Cookie)*__

__클라이언트(브라우저) 로컬에 저장__ 되는 키와 값이 들어있는 __작은 데이터 파일__<br> 
클라이언트의 상태 정보를 로컬(브라우저)에 저장 후 참조/재사용할 수 있다.<br>
사용자가 따로 요청하지 않아도 브라우저가 Request 시, Response Header를 넣어 자동으로 서버에 전송한다.<br>
(\*cf. Response Header Set-Cookie 속성을 사용하면 클라이언트에 쿠키를 만들 수 있다.)

- 쿠키 특징
  - 보안 취약 : 쿠키 조작 및 변조 가능
  - 복잡한 코드 작성
  - 유효한 시간을 명시할 수 있다. → 유효 시간이 정해지면 브라우저가 종료되어도 인증이 유지됨.

- 구성요소
  - 이름 : 각각의 쿠키를 구별하는 데 사용되는 이름
  - 값 : 쿠키의 이름과 관련된 값
  - 유효시간(만료일) : 쿠키의 유지시간
  - 도메인 : 쿠키를 전송할 도메인
  - 경로 : 쿠키를 전송할 요청 경로

<br>

>__\*cf. 쿠키의 동작 방식__
> 1. 클라이언트가 페이지를 요청
> 2. 서버에서 쿠키를 생성
> 3. HTTP 헤더에 쿠키를 포함 시켜 응답
> 4. 브라우저가 종료되어도 쿠키 만료 기간이 있다면 클라이언트에서 보관하고 있음
> 5. 같은 요청을 할 경우 HTTP 헤더에 쿠키를 함께 보냄
> 6. 서버에서 쿠키를 읽어 이전 상태 정보를 변경 할 필요가 있을 때, 쿠키를 업데이트 하여 변경된 쿠키를 HTTP 헤더에 포함시켜 응답

<br>

>__\*cf. 간단한 쿠키test__
>
>	1. 쿠키 조회 :  웹브라우저 주소창 → ```javascript:alert(document.cookie)``` 입력 
>	2. 쿠키 생성 및 수정 → ```//response.addCookie( new Cookie("쿠키변수명", "값"));``` <br>
>       response.addCookie( new Cookie("age", "10") );
>
> 3. 쿠키 삭제  
>  ```Cookie cookie = new Cookie("name", "");
>    //쿠키.setMaxAge(seconds) 쿠키의 유효기간을 초단위로 설정
>    //쿠키.setMaxAge(0) 쿠키가 즉시 삭제됨
>    //쿠키.setMaxAge(-1) 브라우저를 닫으면 쿠키가 삭제됨

<br/>

__*(2) 세션(Session)*__

일정 시간 동안 같은 사용자(브라우저 기준)로부터 들어오는 일련의 요구를 하나의 상태로 보고, 그 상태를 유지시키는 기술. <br>
클라이언트가 Request를 보내면 서버에서는 클라이언트를 구분하기 위해 유일한 ID(세션ID)를 부여하며, 웹 브라우저가 서버에 접속해서 브라우저를 종료할 때까지 인증상태를 유지한다. <br>
즉, __방문자가 웹 서버에 접속해 있는 상태__ 를 하나의 단위로 보고 그것을 __세션__ 이라고 한다.


- 세션 특징
  - 사용자 정보 파일을 서버 측(웹)에서 관리함. 즉, 웹 서버에 저장되는 쿠키(세션 쿠키, session cookie)
  - 쿠키보다 코드 작성이 간단함.
  - 사용자에 대한 정보를 서버에 두기 때문에 쿠키보다 보안에 강하다
  - 저장 데이터에 제한이 없다.
  - 동접자 수가 많은 웹 사이트인 경우, 서버에 과부하를 일으켜 성능저하를 불러일으킬 수 있다.

<br>

>__\*cf. 세션의 동작 방식__
> 1. 클라이언트가 서버에 접속 시 세션 ID를 발급 받음
> 2. 클라이언트는 세션 ID에 대해 쿠키를 사용해서 저장하고 가지고 있음
> 3. 클라리언트는 서버에 요청할 때, 이 쿠키의 세션 ID를 같이 서버에 전달해서 요청
> 4. 서버는 세션 ID를 전달 받아서 별다른 작업없이 세션 ID로 세션에 있는 클라언트 정보를 가져와서 사용
> 5. 클라이언트 정보를 가지고 서버 요청을 처리하여 클라이언트에게 응답

<br/>

__*(3) 쿠키와 세션의 차이*__

||쿠키|세션|
|------|---|---------|
|저장위치|클라이언트 로컬<br>(브라우저, 개인PC)|서버(웹사이트)|
|보안|약|강|
|라이프사이클|브라우저를 종료해도<br>만료시점을 지나지 않으면 삭제X|브라우저 종료 시 삭제|
|속도|빠름|느림|
|저장형식|text|object|
|용량|제한O<br>총 300개<br>하나의 도메인 당 20개<br>하나의 쿠키당 4KB|서버가 허용하는 한<br>제한X

<br/>

__*cf. 캐시(cache)*__

사용빈도가 높은 데이터에 고속으로 접근할 수 있는 위치에 두는 것. 임시저장소<br/>
데이터 출력위치와 가까운 지점에 일시적으로 저장하여 빠른 접근이 가능하지만 데이터 손실 가능성이 있다.

























