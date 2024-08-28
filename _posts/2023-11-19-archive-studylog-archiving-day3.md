---
title: "[국비지원과정 Day 3] Java"
excerpt: "국비 수강기록3. 240822 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, Java]

permalink: /archive/studylog-archiving-day3/

toc: true
toc_sticky: true

date: 2023-11-19
last_modified_at: 2024-08-22
---

## 📕ch05 배열
*1. 배열 Array*
- __같은 자료형__ 으로 구성된 여러 개의 데이터를 한꺼번에 저장하고 처리하기 위한 자료형. 
- 많은 양의 데이터를 처리할 때 유용하다.


*2. 배열의 선언과 생성*
- Stack 영역: method 호출, 지역변수 저장 등에 사용되는 메모리 영역. (Last In First Out의 구조를 갖는다.)
- heap 영역: 인스턴스, 배열이 저장되는 대용량 메모리 영역. (Garbage Collection의 대상이 되는 영역) 
  ```
  자료형[] 배열참조변수 = new 자료형[데이터의 개수];<br/>
  int[] number = new int[5];<br/>
  -> new 로 생성한 배열 데이터는 동적 메모리 할당영역(heap 영역)에 생성된다.
  ```

*3. 배열의 활용*
- 배열의 인덱스는 0부터 시작함.
- 배열의 크기(데이터의 개수): 배열참조변수.length
- 배열의 초기화: 배열은 생성과 동시에 자동적으로 기본값으로 초기화된다. 원하는 데이터를 저장하기 위해서는 배열의 요소에 해당 데이터를 지정해주어야 한다. 
  ```
  배열참조변수[인덱스] = 값;
  int[] score = {100, 90, 80, 70, 75 };
  ```
- 이차원 배열: 1차원 배열을 확장한 개념으로, 행의 개수는 1차원 배열의 개수를 의미. 배열참조변수 **[행][열]** 로 표기


*4. String 배열*
- String: 1차원 문자 배열
  ```
  String cha = new String("Hello");
  String[] words = {"Hello", "Java", "DB", "Python" };  
  ```

<br/>  

---
✏️Quiz
(Q1) 배열로 구구단 테이블 만들기
```
public class Gugu {
   public static void main(String[] args) {
      int dan=3;
      for(int i=1; i<=9; i++) {
         System.out.println(dan+"x"+i+"="+dan*i);
      }
   }
}
```