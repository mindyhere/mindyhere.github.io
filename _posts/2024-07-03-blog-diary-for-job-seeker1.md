---
title: "#취준일기_240703"
excerpt: "생각 날 때마다 남기는 이런저런 기록. 240822 블로그 옮김"

categories:
  - Blog
tags:
  - [JavaScript, MyBatis, TIL]

permalink: /blog/diary-for-job-seeker1/

toc: true
toc_sticky: true

date: 2024-07-03
last_modified_at: 2024-08-22
---

>🌱 Today I Learned... 
다른 사람이 적은 코드를 잘 읽고 이해하는 것도 중요하다

### 1. MyBatis, 동적 쿼리문 작성 시 Java 함수 호출하는 방법

프로젝트에서 공통으로 사용되는 패키지(ex.util 등)에 static 함수를 정의해 mapper(.xml)에서 활용할 수 있다.
아래는 ChatGPT를 활용해 작성한 예시!

~~~java
package com.example.util;

public class Utility {
	
    // static 으로 정의하는 것에 유의
    public static boolean isEmpty(String str) {
        return str == null || str.isEmpty();
    }
}
~~~

MyBatis 매퍼 파일에서 `Utility.isEmpty()` 을 호출 해 동적 쿼리를 작성한다.
이때 `@패키지.클래스명@호출할메소드(파라미터)` 형태로 Java 함수를 호출할 수 있다.

~~~
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.MyMapper">
  
    <select id="selectUsers" parameterType="map" resultType="com.example.model.User">
        SELECT * FROM users
        WHERE 1=1
        
        /* @패키지.클래스명@호출할메소드(파라메터) */
        <if test="@com.example.util.Utility@isEmpty(name) == false">
            AND name = #{name}
        </if>
        <if test="@com.example.util.Utility@isEmpty(email) == false">
            AND email = #{email}
        </if>
    </select>
  
</mapper>

~~~

<hr/>

### 2. JavaScript에서 nvl함수 구현

오라클에서 종종 썻던 nvl함수를 JavaScript에서 구현해서 쓰길래... 배운 김에 간단히 기록하기 + 새로 배운 "null 병합 연산자(Nullish Coalescing Operator, ??)"에 대한 메모

~~~JavaScript
// NVL: expr1 값이 null(undefined) 또는 공백인 경우 expr2 값을 리턴, 그렇지 않으면 expr1 값을 리턴
function nvl(expr1, expr2) {
	if (expr1 === undefined || expr1 == null || expr1 == "") {
		expr1 = expr2;
	}
	return expr1;
}
~~~

#### ✏️ &nbsp *null 병합 연산자(Nullish Coalescing Operator, ??) 란?*

- 왼쪽 피연산자가 null 또는 undefined일 때 오른쪽 피연산자를 반환하고, 그렇지 않으면 왼쪽 피연산자를 반환하는 논리 연산자
- `a ?? b` : a가 null 이나 undefined 가 아니면 a, 그 외의 경우 b를 리턴

~~~JavaScript
// x 는 a가 null이나 undefined가 아니면 a이고 그 외는 b를 리턴
x = (a !== null && a !== undefined) ? a : b;
x = a ?? b;
~~~

위의 두 가지는 같은 동작을 의미한다. 
연산자 ?? 를 활용해 앞서 만든 nvl함수를 구현하면 다음이 정의할 수 있다.

~~~JavaScript
function nvl(expr1, expr2) {
    return expr1 ?? expr2;
}
~~~
