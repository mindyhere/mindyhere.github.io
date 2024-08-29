---
title: "[국비지원과정] Database(1)"
excerpt: "국비 수강기록8. 240828 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, DB]

permalink: /archive/studylog-archiving-database1/

toc: true
toc_sticky: true

date: 2023-11-27
last_modified_at: 2024-08-28
---

![](https://velog.velcdn.com/images/92miindy/post/2d6ad5b5-c177-4e10-9117-2cf0efe3a8ce/image.svg)

<br/>

## 📕ch01 데이터베이스

*1. 데이터베이스*

빠른 탐색과 검색을 위해 조직된 데이터의 집합체

<br/>

*2. DBMS(Database Management System)*

- DBMS의 장점<br/>
  데이터 중복(redundancy)의 최소화<br/>
  데이터의 공유(sharing)<br/>
  일관성(consistency) 유지<br/>
  무결성(integrity) 유지	<br/>

- DBMS의 단점<br/>
   비용 증가(H/W, DBMS, 운영비, 교육비, 개발비 등)<br/>
   프로그램의 복잡화<br/>
   성능상의 오버헤드<br/>

- DBMS의 주요 기능<br/>
CRUD(Create, Read, Update, Delete)<br/>
데이터의 무결성(integrity) 유지<br/>
트랜잭션 관리<br/>
데이터의 백업 및 복원<br/>

- 주요 용어<br/>
데이터베이스(Database): Table의 집합<br/>
테이블(Table): Record의 집합<br/>
레코드(Record): 테이블의 행(row), 한건의 데이터. 컬럼(필드)들의 집합<br/>
컬럼(Column), 필드(Field): 레코드의 열<br/>
Primary Key: 기본키, 테이블에서 각 레코드의 식별자<br/>
Foreign key: 외래키, 다른 테이블의 Primary Key를 참조하는 필드<br/>

<br/>

*3. SQL(Structured Query Language: 구조화된 질의언어*

- SQL 명령어의 종류<br/>
DQL(Data Query Language, 데이터 질의어): select<br/>
DML(Data Manipulation Language, 데이터 조작어): insert, update, delete<br/>
DDL(Data Definition Language, 데이터 정의어): create, alter, dropv
TCL(Transaction Control Language, 트랜잭션 제어어): commit, rollback, savepoint<br/>
DCL(Data Control Language): grant, revoke(사용권한 부여 및 회수)

(1) select
>__SELECT__ 컬럼명1, 컬럼명2 , ... <br/>
>__FROM__ 테이블명<br/>
>__WHERE__ 조건절 __ORDER BY__ 정렬기준컬럼명 asc or desc
	

(2) distinct / all
- distinct: 중복된 데이터를 허용하지 않음
- all: 중복된 데이터를 허용함

(3) order by 정렬기준: asc 오름차순, desc 내림차순

(4) alias 별칭: 컬럼명 [as] 별칭

(5) where: 데이터 검색 조건

(6) 연산자의 종류<br/>
산술연산자 : +, -, *, /<br/>
비교연산자 : =, !=, >, >=, <, <=<br/>
논리연산자 : and, or, not<br/>
기타연산자: in, all, between, like, is null, is not null<br/>
결합연산자: ||

(7) 연산자의 우선순위  
- 비교연산자, SQL연산자, 산술연산자 __>>__ not __>>__ and __>>__ or
- __괄호( )__: 연산자 우선순위보다 우선함

<br/>

✏️Quiz
(Q1) emp 테이블에서 입사일(hiredate)이 2015년 1월 1일 이전인 사원에 대해 사원의 이름(ename), 입사일, 부서번호(deptno)를 출력하시오.
  ```
  SELECT ename, hiredate, deptno FROM emp
  WHERE hiredate < '2015-01-01'
  ORDER BY hiredate;
  ```

(Q2) emp 테이블에서 부서번호가 20번이나 30번인 부서에 속한 사원들에 대하여 이름, 직업코드(job), 부서번호를 출력하시오.
  ```
  SELECT ename, job, deptno FROM emp
  WHERE deptno in (20, 30)
  ORDER BY ename;
  ```

  ```
  SELECT ename, job, deptno FROM emp
  WHERE deptno=20 or deptno=30
  ORDER BY ename;
  ```

<br/>

## 📕ch02 내장함수

*1. 단일행 함수*

레코드별로 한개의 결과값을 반환하는 함수

*2. 집계함수*

여러 개의 레코드를 집계하여 결과값을 반환하는 함수: count(), sum(), avg(), max(), min()

*3. 문자함수*

- concat('문자열1', '문자열2'): 문자열1과 문자열2를 연결(2개만 연결 가능)
  ```
  SELECT concat(ename, '의 직급은'), job FROM emp; 
  ```

- 3개 이상을 한꺼번에 연결하려면 결합연산자( || ) 사용 
  ```
  SELECT ename||'의 직급은'||job FROM emp;
  ```

- replace('문자열1', '문자열2', '문자열3'): 문자열1 중에 있는 문자열2를 문자열3으로 변경
  ```
  SELECT replace('java program','java','자바') FROM dual;
  --replace(원래내용, A, B) => A를 B로 repalce(대체)
  --dual: FROM 절의 형식을 맞추기 위해 추가한 가상 테이블
  ```

- substr('문자열', 자리수, 글자수): 문자열의 자리수부터 지정된 개수만큼의 부분 문자열 리턴
  ```
  SELECT substr('java program', 4,3) FROM dual;
  --substr('문자열', 자리수, 추출하려는글자수) → 부분 문자열 생성
  --인덱스는 1 부터 시작 
  ```

*4. 날짜 함수*

sysdate: 시스템의 현재 시각<br/>
add_months(날짜데이터, 숫자): 날짜값에 개월 수를 더해서 결과값을 반환함. 월만 증가/감소되고 날짜는 그대로<br/>
months_between(날짜1, 날짜2): 두 날짜 사이의 개월수(날짜1-날짜2)<br/>
to_char(날짜컬럼 or 날짜데이터, '출력형식')<br/>
to_date('날짜 형태의 문자열', '날짜 변환 포맷'): 문자열을 날짜로 변환<br/>
  ```
  SELECT to_char(sysdate, 'yyyy-mm-dd am hh:mi:ss day') FROM dual;
  --현재 시스템날짜(sysdate)의 포맷('...') 지정
  --요일 → d: 일요일~토요일을 1~6 숫자코드로
  --day: 요일 이름 전체 (ex.일요일, 월요일...)

  SELECT to_date('2018-01-26', 'yy-mm-dd') FROM dual;
  --to_date: 문자→날짜로 바꾸는 함수, '형식지정'
  ```

*5. 숫자 함수*

숫자변환 함수: to_number('숫자 형태의 문자열')<br/>
trunc(숫자, 자리수) : 지정된 자리수 이하의 소수를 버림. 자리수를 생략하면 소수 부분을 버림<br/>
round(숫자, 자리수) : 반올림<br/>
ceil(숫자) : 올림

*6. 조건검사 함수*

- nvl(컬럼, 대체값) → 필드의 값이 null일 때의 대체값
  ```
  --(ex)커미션이 null 인 경우, 0로 처리
  SELECT ename, sal, comm, sal*12+nvl(comm,0) FROM emp;
  ```
- decode(A, B, A==B일때의값, A!=B일때의값)

<br/>

✏️Quiz
(Q1) 직원의 이름, 직급, 급여를 출력하시오.(월급이 300 이상인 직원만 출력합니다.)
```
SELECT ename, job, sal fROM emp where sal>=300;   
```

<br/>

(Q2) 직원의 이름과 근무개월수를 출력하시오.(근무개월수가 100개월 이상인 직원만 출력합니다.)
```
SELECT ename, hiredate, round(months_between(sysdate, hiredate)) 근무개월수
FROM emp 
WHERE round(months_between(sysdate, hiredate)) >=100;
```

<br/>

(Q3) 직원의 이름과 직급, 총 근무주(week)수를 출력하시오. (근무주수 내림차순, 근무주수가 같으면 이름에 대하여 오름차순 정렬합니다.)
  ```
  SELECT ename, job, round((sysdate - hiredate)/7) 총근무주수 
  FROM emp 
  OPRDER BY round((sysdate - hiredate)/7) desc, ename; 
  --주차 수(week)를 구하는 함수는 따로 없어서 근무일을 7로 나눈 후 계산함
  ```

<br/>

## 📕ch03 테이블 join

*1. join*

- 2개 이상의 테이블을 논리적으로 결합하여 원하는 컬럼 정보를 참조하는 방법
- 전제조건<br/>
(1)논리적으로 결합되는 2개 이상의 테이블에는 반드시 공통 컬럼이 있어야 한다.<br/>
(2)공통 컬럼은 데이터타입과 테이터가 동일해야 한다.

*2. join 형식*

>__SELECT__  컬럼 리스트<br/>
>__FROM__ 조인대상 테이블들(컴머로 구분, 별칭사용)<br/>
>__WHERE__ 조인조건 __AND__ 일반조건;


*3. 종류*

- 내부조인(inner join): 일반적인 형태. __WHERE 절__ 에 사용된 공통컬럼들이 동등연산자(=)에 의해 비교되는 조인(양쪽 테이블 모두에 데이터가 있는 경우만 출력됨)
- self join: 참조해야 할 컬럼이 자신의 테이블에 있는 다른 컬럼인 경우에 사용하는 조인 
- 외부(outer)조인: 한쪽 테이블에만 자료가 있을 경우에도 출력되도록 하려면 외부 조인을 해야 함

*4. ANSI 조인*

- ANSI SQL 문법에 따른 테이블 조인 방법
- 내부조인: __FROM 절__에 컴머(,) 대신 __JOIN__ & WHERE 대신 __ON__ 사용
- 외부조인<br/>
데이터가 있는 테이블을 기준으로 left, right, full<br/>
left, right outer join의 경우 __+ 기호__ 를 사용 → 데이터가 없는 테이블 쪽에 __+__
  ```
  --left outer join
  SELECT sname, s.major, pname
  FROM stud s left outer JOIN prof p
  ON s.profno =p.profno(+);

  --right outer join
  SELECT sname, s.majorno, pname
  FROM stud s right outer JOIN prof p
  ON s.profno = p.profno;

  --full outer join
  SELECT sname, s.majorno, pname
  FROM stud s left outer JOIN prof p
  ON s.profno = p.profno;
  ```

*5. 합집합 union*

```
  --A union B (중복값 제거)
  UNION
  SELECT * FROM emp WHERE deptno=10;

  --union all  (중복값을 제거하지 않음)
  UNION all
  SELECT * FROM emp WHERE deptno=10;
  ```
<br/>
*cf. View(뷰, 가상테이블)

  ```
  CREATE OR REPLACE view 뷰이름
  AS  (sql 명령어) ;
  ```

  ```
  (ex)
  CREATE OR REPLACE view product_sales_v
  --생성        변경    뷰    뷰이름
  AS
  SELECT p.product_code, product_name, price, company, amount, price*amount money, make_date
  FROM product  p, product_sales  s
  WHERE p.product_code = s.product_code;
  ```

<br/> 

✏️Quiz
(Q1) emp와 dept 테이블을 조인하여 사원이름, 부서명, 급여를 출력하시오.
  ```
  SELECT ename, e.deptno, sal
  FROM emp e, dept d
  WHERE e.deptno=d.deptno;

  SELECT ename, dname, sal FROM emp_v;      
  --뷰 emp_v 를 활용
  ```

<br/>
(Q2) 직급이 ‘사원’인 사원이름, 부서명을 출력하시오.

  ```
  SELECT ename, dname
  FROM emp e, dept d
  WHERE e.deptno=d.deptno and job = '사원' ;

  SELECT ename, dname FROM emp_v where job = '사원';      
  --뷰 emp_v를 활용
  ```

<br/> 
(Q3) 이름이 ‘손기철’인 사원의 부서명을 출력하시오.

  ```
  SELECT dname
  FROM emp e, dept d
  WHERE e.deptno=d.deptno and ename = '손기철';

  SELECT dname FROM emp_v where ename = '손기철';      
  --뷰 emp_v를 활용
  ```

<br/> 
(Q4) emp테이블에 있는 empno, mgr을 이용하여 서로의 관계를 다음과 같이 출력하시오. “박종수의 매니저는 박성환이다” → <b>셀프조인(self join)</b>

  ```
  SELECT a.ename || '의 매니저는 ' || b.ename || '이다.' 	 -- A || B 결합, 연결
  
  FROM emp a, emp b       
  WHERE a.mgr = b.empno;      --조인 조건: 사원.관리자사번 = 팀장.본인사번
  ```


