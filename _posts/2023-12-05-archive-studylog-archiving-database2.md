---
title: "[국비지원과정] Database(2)"
excerpt: "국비 수강기록10. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, DB]

permalink: /archive/studylog-archiving-database2/

toc: true
toc_sticky: true

date: 2023-12-05
last_modified_at: 2024-08-29
---

## 📕ch04 트랜잭션 처리

*1. 트랜잭션(transaction)*

분리되어서는 안되는 논리적인 작업단위<br/>
데이터베이스 내에서 한꺼번에 수행되어야 할 일련의 연산들<br/>
트랜잭션은 한꺼번에 완료가 되거나 한꺼번에 취소가 되어야 함(원자성)

*2. TCL(transaction control language; 트랜잭션 제어어)*

__commit__: insert 추가, update 수정, delete 삭제 등, 명령어의 실행 결과를 영구적으로 DB에 반영하는 명령어. 전체 변경사항을 확정<br/>
__rollback__: insert, update, delete 명령어의 실행 결과를 취소하는 명령어<br/>
__savepoint__: transaction의 임시 저장점<br/>
```
rollback to a;	
-- savepotin a까지 취소
```
\*cf. 실수로 커밋한 자료의 복구 방법
undo_retention → COMMIT 이후 지정된 시간(초)까지는 임시로 저장된 데이터로 복구할 수 있다. 기본값은 900sec(= 15분)

<br/>

📌예제check
(Q)1500초(25분)로 시간 변경가능 및 데이터 복구
```
alter system set undo_retention = 1500; 

SELECT * FROM emp AS OF TIMESTAMP(SYSTIMESTAMP - INTERVAL '15' MINUTE) WHERE empno = 7369;

INSERT INTO emp SELECT * 
FROM emp 
	AS OF TIMESTAMP(SYSTIMESTAMP - INTERVAL '15' MINUTE) 
WHERE empno = 7369;
```

<br/>

## 📕ch05 group by & having

*1. group by*

특정한 컬럼을 기준으로 집계된 데이터를 보기 위한 명령어 → 1개 이상의 집계기준 필드 사용<br/>
select절의 컬럼은 집계함수를 제외하고 모두 group by절에 명시해야 한다. <br/>

\*cf. 집계함수: 레코드 개수 count(), 최소 min(), 최대 max(), 합계 sum(), 평균 avg()<br/>
\*실행 순서<br/>
(3) __select__ 필드명     필드선택<br/>
(1) __from__ 테이블    테이블선택<br/>
(2) __where__       행 선택<br/>
(4) __group by__ 집계기준필드     요약<br/>
(5)_ __order by__ 정렬기준필드     정렬<br/>

<br/>

📌예제check

```
select e.deptno, dname, count(*), sum(sal), round(avg(sal),2), min(sal), max(sal)
from emp e, dept d      
where e.deptno = d.deptno
group by e.deptno, dname    
	-- select 절에 있는 필드명과 동일한 필드명 "모두" 명시되어있어야 한다. 
	-- 에러메세지 → (1)d.deptno 필드 선택 시 or  (2)dname 누락 시
order by deptno;
```

<br/>

*2. having*

group by의 결과 중 조건에 맞는 행을 선택하기 위한 명령어

<br/>

✏️Quiz

__(Q1)__ stud 테이블과 major 테이블을 조인한 후 전공코드별로 집계하여 전공코드, 전공이름, 학생수를 출력하시오.

```
select m.majorno, mname, count(*)
from stud s, major m
where s.majorno = m.majorno
group by m.majorno, mname
order by m.majorno;
```

<br/>

__(Q2)__ 지도교수사번, 지도교수이름, 지도학생수를 출력하시오.(stud 테이블과 prof 테이블을 조인하여 지도교수사번별로 집계)

```
select p.profno, pname, count(*)
from stud s, prof p
where s.profno = p.profno
group by p.profno, pname
order by p.profno;
```

<br/>

__(Q2)__ 교수 중에서 급여총액(급여+보너스)이 가장 높은 교수와 가장 낮은 교수, 급여총액의 평균금액을 출력하시오.(전공코드별로 집계)<br/>
→ __nvl() 함수__ 를 사용하여 대체값을 지정

```
select majorno, max(pay+nvl(bonus,0)) 최대급여총액, min(pay+nvl(bonus,0)) 최소급여총액,
	   round(avg(pay+nvl(bonus,0)),1) 평균급여총액
from prof
group by majorno;
```

<br/>

## 📕ch06 서브쿼리

메인 쿼리 내부에 존재하는 또 다른 select 명령어

- 인라인 뷰(inline view): from절에 위치한 서브쿼리
- scalar 서브쿼리: 한개의 행, 한개의 컬럼을 반환하는 서브쿼리

\*cf. View(뷰, 가상테이블: 테이블명은 중복 불가)
```
create or replace view test_view 
-- 생성      변경          뷰 이름
as select * from emp where sal > 400;
-- as (실제 명령어)
```

<br/>

✏️Quiz

__(Q1)__ 조재철 교수보다 나중에 입사한 교수명, 입사일, 전공명을 출력하시오.    

```
select pname, hiredate, mname
from prof p, major m
where p.majorno = m.majorno
      and hiredate > (select hiredate from prof where pname='조재철')
order by hiredate;
```

<br/>

__(Q1)__ 심상수 교수보다 나중에 입사한 교수 중에서 박철호 교수보다 월급을 적게 받는 교수명, 급여, 입사일을 출력하시오.

```
select pname, pay, hiredate
from prof
where hiredate > (select hiredate from prof where pname='심상수')
      and pay < (select pay from prof where pname='박철호')
order by hiredate, pay;    
```

<br/>


## 📕ch07 제약조건(constraint)

📌 review
- 테이블명 수정 → rename 테이블명 to 수정할 테이블명
- 테이블에 데이터 입력(레코드 추가) → insert into 테이블명 (필드목록...) values(...);<br/>
\*cf. 테이블의 필드목록 개수와 레코드 추가 시에 입력한 데이터 개수가 다를 경우, 에러 발생

- 테이블에 컬럼 추가, 수정, 삭제 
  ```
  alter table s_emp add hire_date date;   		--컬럼추가
  alter table s_emp modify phone varchar2(50);    --컬럼 수정(자료형 변경)
  alter table s_emp rename column id to t_id;		--컬럼 이름변경
  alter table s_emp drop column dept_name;		--컬럼 삭제
  ```
- insert, update, delete + __where(조건)__ → 별도 조건(where절)이 없을 경우, 전체 내용에 대해 수정/변경되므로 주의하여야 한다.

*1. 제약조건(constraint)*

- 레코드의 컬럼에 부정확한 데이터가 입력, 변경, 삭제되는 것을 방지하기 위해 테이블을 만들거나 변경할 때 설정하는 조건
- 종류: primary key, check, foreign key, unique, not null<br/>
unique 제약조건: 중복불가 +  null값 허용<br/>
__primary key__ : unique(중복안됨) + not null(필수입력)<br/>
check 제약조건: 조건 설정 > 데이터의 유효성을 검사<br/>
__Foreign key__ 제약조건: 외래키, 다른 테이블의 primary key를 참조<br/>


*2. 제약조건 이름 검색*

select * from __user_constraints__ where table_name='테이블명(__대문자__)';
  ```
  select * from user_constraints;
  select constraint_name from user_constraints;
  --        제약조건 이름       시스템테이블(데이터사전)
  ```

*3. 제약조건 추가*

alter table 테이블명 __add constraint__ 제약조건이름 종류(필드);

  ```
  alter table c_emp modify name varchar2(25) not null;
  -- not null 제약조건은 add로 할 수 없고 "modify"로 추가해야 한다.
  ```

*4.  제약조건의 삭제*

alter table 테이블 __drop constraint__ 제약조건이름
