---
title: "[국비지원과정 Day 0-1] OT, Java"
excerpt: "국비 수강기록1. 240822 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, Java]

permalink: /archive/studylog-archiving-day1/

toc: true
toc_sticky: true

date: 2023-11-16
last_modified_at: 2024-08-22
---

# 2023.11.13 ~ 2024.6.10
### 💻1120시간, 140일, 7개월 간의 기록 

세간에서 말도 많고 탈도많은 국비지원 웹개발자 양성과정이라지만, 모든 일이 다 그렇듯이 각자하기 나름 아닐까.
대략 한 달 동안 혼자서 나름 선수학습도 진행했는데, 나홀로 낙오되는 일 없도록 이곳에 앞으로의 학습기록을 남겨보려 한다.

### *0. Java 개발환경 설정하기*
첫째날이라 오전 대부분의 시간은 프로그램을 설치하고 세팅하는 것으로 마무리.
Java, JDK, Eclipse 를 설치, 디렉토리 지정, 윈도우 환경변수 설정 기타등등을 세팅하고 나면
본격적으로 수업 준비 완료...!

---
## *📕ch01 자료형*
*1. 자료형과 변수*
- 정보의 저장단위
bit : 정보의 최소 저장단위
(8bit = 1byte, 1024byte = 1KB)
  
- 변수(Variable)와 상수(Constant)
변수: 변하는 값, 하나의 값을 저장할 수 있는 기억공간.
변수를 사용하기 전에 먼저 변수 선언을 해야하며, 변수의 이름은 같은 영역 내에서 중복될 수 없다.
상수: 변하지 않는 변수. 값이 한 번 값을 저장하고 나면 다른 값으로 변경할 수 없다. 키워드로 __final__ 을 붙이고 변수명을 대문자로 표기하여 구분한다.
리터럴(Literal): 고정된 값, 변하지 않는 __데이터 그 자체__를 의미.

		int count = 10;
	// 자료형 변수명 => 10(값)을 변수에 대입, 저장
	// 변수의 자료형과 이름을 컴파일러에게 알려주어야 함.
    
    final int MAX_NUM = 100;
    //	  상수 MAX_NUM => 100(리터럴)을 대입, 저장


*2. 기본자료형 & 참조자료형*
기본형 변수는 실제 값(data)을 저장하지만, 참조형 변수는 데이터가 저장되어 있는 주소가 메모리에 저장된다.
- 8가지 기본자료형(Primitive type)
![](https://velog.velcdn.com/images/92miindy/post/d4c707ac-e8ab-4748-9cae-da2926638c63/image.jpeg)

- 참조자료형(Reference type) : 기본자료형을 제외한 나머지 자료형, 데이터의 주소(위치)를 값으로 저장.


*3. 문자와 문자열의 구분*
- 문자(char): 1개의 문자만 저장 가능하며, 작은따옴표(') 안에 작성. 
- 문자열(String): 문자들의 집합. 문자열.
  ```
  char ch = 'A';
  String str = "Hello";
  ```

## *📕ch02 연산자*
*1. 연산자의 종류(항의 개수로 분류)*
- 단항연산자: + - (부호연산자), ++ -- (증감연산자), ! (논리부정), (자료형)
- 이항연산자: + - * / % (산술연산자), < > <= >= == != (비교연산자), && || (논리연산자)
- 삼항연산자: (조건식)? 식1 : 식2
		   조건식이 true 이면 식1의 결과를, false 이면 식2의 결과를 반환
- 대입연산자: = (좌변의 값을 우변에 __대입__)


*2. 연산자의 우선순위*
- 기본적인 우선순위: 괄호의 우선순위가 제일 높고, 기본적으로 "산술 > 비교 > 논리 > 대입", "단항 > 이항 > 삼항" 의 우선순위를 가진다.
- 단항연산자와 대입연산자를 제외한 모든 연산의 방향은 왼쪽에서 오른쪽으로 진행된다.


---
(Q1) 만 나이 계산하기.
이름과 출생연도는 변수에 고정값으로 입력하거나 사용자가 입력한 값으로 처리한다.
나이는 현재 연도(4자리 숫자)에서 출생연도값을 뺀 값으로 처리한다.
```
package quiz;
import java.util.Scanner;
public class Ex01 {
   public static void main(String[] args){
      Scanner sc = new Scanner(System.in);
      System.out.print("이름:");
      String name = sc.next();
      System.out.print("출생연도:");
      int year = sc.nextInt();
      int age = 2023 - year;
      System.out.println(
            name + "님의 나이는 만 " + age + "세입니다.");
   }
}
```


(Q2) 성적 계산
시험점수는 키보드로 입력받아 처리. 70점 이상이면 합격 아니면 불합격 처리.
```
package quiz;

import java.util.Scanner;

public class Ex02 {
   public static void main(String[] args) {
      Scanner sc = new Scanner(System.in);
      System.out.println("점수를 입력하세요:");
      int point = sc.nextInt();
      System.out.println(point >= 70 ? "합격" : "불합격");
      if (point >= 70) {
         System.out.println("합격입니다.");
      } else {
         System.out.println("불합격입니다.");
      }
   }
}
```
