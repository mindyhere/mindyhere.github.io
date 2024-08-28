---
title: "[국비지원과정] Java-DB 연동실습"
excerpt: "국비 수강기록13. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, Java, DB]

permalink: /archive/studylog-archiving-jdbc/

toc: true
toc_sticky: true

date: 2023-12-12
last_modified_at: 2024-08-29
---

## java - 데이터베이스(oracle/mySQL) 연동

__*1. 데이터베이스 프로그래밍의 일반적인 순서*__

DBMS에 연결 → SQL 명령어 실행 → 결과 리턴(select 명령어의 경우) → 연결 종료


__*2. JDBC(Java DataBase Connectivity)*__

자바와 데이터베이스의 연동 작업을 지원하는 기술 <br/>
자바에서는 interface만 제공하고 DBMS 제조사에서 class 구현<br/>
JDBC 드라이버 : DBMS에 대한 세부 작업을 담당하는 라이브러리, jar 파일로 제공<br/>


__*3. JDBC를 이용한 데이터베이스 프로그래밍*__

(1) JDBC 드라이버 로드(DriverManager)<br/>
(2) 데이터베이스에 접속(Connection)<br/>
(3) SQL 명령어 실행(PreparedStatement)<br/>
(4) select 명령어의 경우 결과셋 리턴(ResultSet)<br/>
(5) 데이터베이스 연결 종료<br/>


__*4. JDBC에서의 트랜잭션 처리*__

기본적으로 auto commit<br/>
트랜잭션 기능을 사용하려면 auto commit 옵션을 비활성화<br/>
    setAutoCommit(false)<br/>
    commit();<br/>
    rollback();<br/>

  <br/>

__*📌실습*__

*__nextLine() & next()__ 메서드의 차이 알기*

아래 예제 실습 중 next()를 써야하는 상황에서 nextLine() 으로 잘못처리해서 데이터를 입력 받아야하는 한 라인(line, 이름)을 건너뛰게되는 논리적 오류가 발생하였다.

nextLine의 경우, __개행문자를 포함하여 한 라인__을 읽어오기 때문에 __nextInt()__ 다음에 사용했을 때 앞에서 처리되지 못하고 남아있던 엔터값(\n)을 읽어오게 되면서 명령어가 바로 종료된다.
때문에 아래와 같은 경우에서는 '이름'에 대한 데이터 입력을 건너뛰고 그 다음 행인 '직급'으로 넘어가게 되었던 것.

nextLine() 과 next()는 대부분 비슷한 결과를 반환하지만, 이처럼
논리적 오류가 발생하는 경우가 있기 때문에 차이점을 분명히 알고 적절하게 활용해야할 것이다.



```java
ackage jdbc;

import java.util.List;
import java.util.Scanner;

public class ManageEmp {
	EmpDAO dao = new EmpDAO();

	void delete() {
		Scanner sc = new Scanner(System.in);
		System.out.print("삭제할 사원번호를 입력하세요: ");
		int empno = sc.nextInt();
		int result = dao.delete_emp(empno);
		if (result == 1) {
			System.out.println("삭제되었습니다.");
		} else {
			System.out.println("사원번호를 확인하세요.");
		}
	}

	void insert() {
		Scanner sc = new Scanner(System.in);
		System.out.print("사원번호: ");
		int empno = sc.nextInt();
// nextInt() 다음에 nextLine()를 실행할 경우, nextLine()가 그냥 넘어가버리는 오류발생
// nextLine(): Enter값을 기준으로 메소드를 종료시키기 때문에 nextInt()실행 후 남아있던 Enter값를 그대로 읽어 바로 종료되고 그 다음Line이 출력됨
		System.out.print("이름: ");
		String ename = sc.next();

		System.out.print("직급: ");
		String job = sc.next();

		System.out.print("입사연도: ");
		String hiredate = sc.next();

		System.out.print("연봉: ");
		double sal = sc.nextDouble();

		System.out.print("부서번호: ");
		int deptno = sc.nextInt();

		EmpDTO dto = new EmpDTO(empno, ename, job, hiredate, sal, deptno);
		dao.insert_emp(dto);
		System.out.println("추가되었습니다.");
	}

	void list() {
		List<EmpDTO> items = dao.list_emp();
		System.out.println("사원번호\t이름\t직급\t입사연도\t연봉\t부서번호");
		System.out.println("=======================================");

		for (EmpDTO dto : items) {
			System.out.print(dto.getEmpno() + "\t");
			System.out.print(dto.getEname() + "\t");
			System.out.print(dto.getJob() + "\t");
			System.out.print(dto.getHiredate() + "\t");
			System.out.print(dto.getSal() + "\t");
			System.out.print(dto.getDeptno() + "\n");
		}
	}

	public static void main(String[] args) {
		ManageEmp emp = new ManageEmp();
		Scanner sc = new Scanner(System.in);
		while (true) {
			System.out.print("작업을 선택하세요(1:목록 2:추가 3:삭제 0:종료): ");
			int code = sc.nextInt();
			switch (code) {
			case 0:
				System.out.println("프로그램을 종료합니다.");
				System.exit(0);
				break;
			case 1:
				emp.list();
				break;
			case 2:
				emp.insert();
				break;
			case 3:
				emp.delete();
				break;
			}
		}
	}
}
```

