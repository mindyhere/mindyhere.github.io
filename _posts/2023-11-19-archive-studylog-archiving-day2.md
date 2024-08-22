---
title: "[국비지원과정 Day 2] Java"
excerpt: "국비 수강기록2. 240822 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, Java]

permalink: /archive/studylog-archiving-day2/

toc: true
toc_sticky: true

date: 2023-11-19
last_modified_at: 2024-08-22
---

## *📕ch03 조건문*
*1. if문*
>```
if (조건식) {
	조건식에 대해 true일 때 실행할 명령어;
}

>```
if (조건식) {
	조건식에 대헤 true 일 경우;
	} else {
		조건식에 대해 false 일 경우;
}

>```
if (조건식1) {
	조건식1 에 대해 true 일 경우;
	} else if (조건식2) {
		조건식2 에 대해 true 일 경우;
	} else if (조건식3)  {
		조건식3에 대해 true 일 경우;
	} else {
		앞서 작성한 모든 조건식에 해당하지 않을 때 수행할 명령어;
}
```

기본적으로 위와 같은 구조로 작성하고, 그 외 if문 안에 또 다른 if문을 추가한 중첩if문으로 활용 가능하다.
간단한 if문의 경우 삼항연산자를 활용해 구현할 수도 있다.


*2. switch문*
>```
switch (조건식 ) {
	case 값1: 
		수행할 문장;
    	break;
	case2 : 
    	수행할 문장;
		break;
	case3 : 
    	수행할 문장;
		break; 
	defauel;
    	조건식의 결과와 일치하는 case문이 없을 때 수행할 문장;
}
```

- switch문의 조건식은 정수식 또는 문자열 이어야 하고, case문의 값은 정수, 상수, 문자열만 가능하며, 중복되지 않아야 한다.
- if문보다 코드가 간결하지만, break문이 없을 경우 다음 코드를 계속해서 진행하기 때문에 주의하여하 한다. 
- 주로 반복횟수가 고정적인 경우 활용가능


## *📕ch04 반복문*
*1. for문*
>```
for (초기식; 조건식; 증감식) {
	조건식이 true일 동안 반복 수행할 문장;
}
```

(1)초기식 > (2)조건식 > (3)수행할 문장 > (4)증감식 ... 의 순서로 조건식에 대해 false일 때까지 (2)~(4)가 반복되기 때문에 (1)초기식은 최초 한번만 실행된다.

*2. while문*
>```
while (조건식) {
	조건식이 true일 동안 반복할 문장;
}
```

- 조건을 먼저 평가한 후 {}블럭을 수행하며, 조건식에 대해 false일 때까지 과정이 반복된다. 
- 초기식, 증감식이 필요하지 않고 반복횟수가 가변적인 경우 활용가능

*3. do~while문*
>```
do {
	수행할 문장;
} while (조건식);
```

while문의 조건식과 {}블럭의 순서를 바꾸어 {}의 명령어를 먼저 수행 후 조건식을 평가하는 구조로, 최소 한번은 명령어를 수행할 것이 보장된다.

*4. break문 & continue문*
- break문: 자신이 포함된 가장 가까운 반복문을 벗어나게 한다.
- continue문: 반복문 내에서만 사용되는 문장. 블록 내 다음 단계로 이동하게 한다. 반복문 전체를 벗어나지 않고 다음 반복을 계속 수행한다는 점에서 break문과 다르다.




---
✏️Quiz
(Q1) 윤년 계산 프로그램
```
Scanner sc = new Scanner(System.in);
		System.out.println("연도를 입력하세요:");
		int year = sc.nextInt();
		//	키입력=>정수로
		if (year % 4 == 0 && year % 100 != 0 || year % 400 == 0) {
			System.out.println(year + "==> 윤년입니다.");
		} else {
			System.out.println(year + "==> 평년입니다.");
		}
```


(Q2)구구단 테이블
```
package ch04;

import java.util.Scanner;

public class MultiTable1 {
   public static void main(String[] args) {
      //키보드로 단을 입력
      Scanner sc=new Scanner(System.in);
      System.out.println("단을 입력하세요:");
      int num = sc.nextInt();
      for (int i = 1; i <= 9; i++) {
         System.out.println(num + "x" + i + "=" + num * i);
      }
   }
}
```


(Q3) 운행거리 2km까지는 기본요금 4800원을 적용한다. 운행거리가 1.6km를 넘으면 131m마다 100원씩으로 계산한다.
운행거리: 3000m 일 때, 택시요금을 계산.
```
package quiz;

import java.util.Scanner;

public class Taxi {
   public static void main(String[] args) {
      Scanner sc = new Scanner(System.in);
      System.out.println("거리를 입력하세요(km):");
      double km = sc.nextDouble();
      double m = km * 1000;
      int fee = 0;
      if (m <= 2000) {
         fee = 4800;
      } else {
         double temp = m - 1600;
         fee = 4800 + (((int) Math.ceil(temp / 131.0)) * 100);
      }
      System.out.println("요금:" + fee);
   }
}
```

(Q4) 랜덤 숫자 맞추기 게임
1~100 사이의 값을 반복적으로 입력하여 컴퓨터가 랜덤으로 생성한 값을 몇 회만에 맞췄는지 출력한다.
```
package quiz;

import java.util.Random;
import java.util.Scanner;

public class RandomNumber {
   public static void main(String[] args) {
      Random r = new Random(); 
      int com = r.nextInt(100) + 1; 
      System.out.println("컴퓨터의 숫자:"+com);
      Scanner sc = new Scanner(System.in);
      int user;
      int count = 0;
      while (true) {
         System.out.print("숫자를 입력하세요:");
         user = sc.nextInt();
         count++;
         if (com == user) {
            System.out.println("정답입니다.");
            System.out.println(count + "회 시도");
            break;
         } else if (com > user) {
            System.out.println("더 큰 수를 입력하세요.");
         } else if (com < user) {
            System.out.println("더 작은 수를 입력하세요.");
         }
      }
   }
}
```

(Q5)섭씨온도를 화씨온도로 변환하는 프로그램
화면에서 섭씨온도를 입력받아 처리. 
```
package quiz;

import java.util.Scanner;

public class Degree {
   public static void main(String[] args) {
      Scanner sc = new Scanner(System.in);
      while (true) {
         System.out.println("섭씨온도를 입력하세요:");
         double input = sc.nextDouble();
         if (input == 0)
            break;
         double output = (input * 1.8) + 32;
         System.out.println(output);
      }
      System.out.println("종료 되었습니다.");
   }
}
```