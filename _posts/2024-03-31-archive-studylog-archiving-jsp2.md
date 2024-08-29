---
title: "[국비지원과정] JSP(2)"
excerpt: "국비 수강기록20. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, JSP]

permalink: /archive/studylog-archiving-jsp2/

toc: true
toc_sticky: true

date: 2024-03-31
last_modified_at: 2024-08-29
---

## 9. EL-JSTL
### 1. Expression Language(표현언어) 
JSP 스크립트의 표현식을 대신하여 속성 값을 쉽게 출력하도록 고안된 언어로, 출력, 반복처리를 태그기반으로 제공.<br>
JSP 페이지에서 자바 코드를 간단히 표현하는 것이 가능하고, 값이 null 이어도 예외가 발생하지 않는다.

값 표현 방법
 ```jsp
//기존 Expression Tag(<%= %>) 대체
${변수 or 표현식}
```

<br/>

__*1. 변수의 사용 범위*__

- 현재 페이지에서만 사용<br>
pageContext →	 ```${pageScope.변수}```
- 요청페이지+응답페이지<br>
request →	```${requestScope.변수}```
- 사용자 단위(로그인~로그아웃)<br>
session →	```${sessionScope.변수}```
- 서버 단위(모든 사용자)<br>
application →	```${applicationScope.변수}```
- 폼, 쿼리스트링변수<br>
request.getParameter →	```${param.변수}```

<br>

__*2. 객체의 표현*__

```jsp
${객체.변수명}
${sessionScope.세션변수명}
${param.변수명}
${paramValues.배열변수명}
```

<br/>
<br/>

### 2. JSTL(Jsp Standard Tag Library)

JSP 기본 태그 외의 표준 사용자정의 태그(Custom Tag)<br>
Java 코드를 바로 사용하지 않고 HTML 태그(<>) 형태로 직관적인 코딩을 지원하는 라이브러리로, 변수 선언, 조건문, 반복문 등의 기능을 제공한다.

<br>

__*1. JSTL 라이브러리 다운로드*__

- jakarta.servlet.jsp.jstl-3.0.0.jar<br>
https://mvnrepository.com/artifact/org.glassfish.web/jakarta.servlet.jsp.jstl/3.0.0
- jakarta.servlet.jsp.jstl-api-1.2.2.1.jar<br>
https://mvnrepository.com/artifact/com.guicedee.services/jakarta.servlet.jsp.jstl-api/1.2.2.1 <br>
jar 파일 2개를 다운로드한 후 webapp/WEB-INF/lib에 복사

<br>

__*2. JSTL의 사용*__

라이브러리들은 URI 형식으로 제공되며 태그에서 사용할 때는 접두어(prefix)를 사용해 불러온다.

```jsp
//<%@ taglib prefix="접두어" uri="JSTL라이브러리의 URI"%>
//<접두어:태그>로 사용

<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<c:if ...>
	//코드 작성
</c:if>
```

<br>

__*3. core 태그*__

- uri : http://java.sun.com/jsp/jstl/core
- prefix : c
- 제공 기능 : 변수 선언, 조건문, 반복문 등

(1) 변수 선언문 

```jsp
<c:set>
```

(2) 조건문 

```jsp
//조건문
<c:if test= "조건식" ></c:if>

//다중조건문
<c:choose>
	<c:when> 조건에 맞는 경우
	<c:otherwise> 나머지 경우
```
        
(3) 반복문

```jsp
<c:forEach var="개별변수" items="집합" begin="시작" end="종료">
```

(4) 출력문

```jsp
<c:out> 
//화면 출력. JSP의 표현식 대체
```

(5) 다른 페이지로 이동

```jsp
<c:redirect> 
//response.sendRedirect()를 대체하는 태그
```

<br>

__*4. format 태그*__

- uri : http://java.sun.com/jsp/jstl/fmt
- prefix : fmt
- 주요 기능 : 날짜, 숫자의 출력 형식 제공
  
  ```jsp
  <fmt:formatNumber> 	// 숫자 형식
  <fmt:formatDate>	// 날짜 형식
  ```

<br>

__*5. functions 태그*__

- uri : http://java.sun.com/jsp/jstl/functions
- prefix : fn
- 주요 기능 : 다양한 함수 제공


<br/>
<br/>

## 10. mybatis 활용

__*1. mybatis *__

쿼리 기반 웹 애플리케이션을 개발할 때 많이 사용되는 __SQL 매퍼(Mapper) 프레임워크__로, JDBC에서 처리하는 코드 상당부분과 파라미터 설정, 결과 매핑을 지원한다.<br>
mybatis 활용으로 자바 코드와 sql 명령어를 분리하여 작성하는 것이 가능하며, 코딩량이 절감되고 유지보수에 용이하다는 장점이 있다. 

<br>

__*2. mybatis 설정 방법*__

mybatis-3.5.11.jar → lib 폴더에 복사<br>
MybatisManager.java → sql을 실행할 수 있는 세션 생성 기능<br>
sqlMapConfig.xml → mybatis 기본설정 파일<br>
mapper 파일 → 실제 sql query 문장<br>
