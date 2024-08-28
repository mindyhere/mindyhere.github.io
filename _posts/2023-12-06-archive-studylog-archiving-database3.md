---
title: "[국비지원과정] Database(3) 뷰, 인덱스, 일련번호, 고급함수"
excerpt: "국비 수강기록11. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, DB]

permalink: /archive/studylog-archiving-database3/

toc: true
toc_sticky: true

date: 2023-12-06
last_modified_at: 2024-08-29
---

## 📕ch08 뷰(View)

- 테이블에 대한 가상의 테이블(논리적인 개념)
- 테이블에 대한 보안 기능을 설정해야 하는 경우 활용
- 장점: 복잡하며 자주 사용하는 SQL 명령어를 쉽고 간단하게 사용할 수 있다.
- View 생성 및 변경 → __CREATE__ OR __REPLACE VIEW__  뷰이름 __AS__ (select명령어);
- View 삭제  → __drop view__ 뷰이름;

<br/>

📌예제check

```
CREATE OR REPLACE VIEW test_v
AS  //실행할 명령어
	SELECT empno, ename, e.deptno, dname
    FROM emp e, dept d
    WHERE e.deptno = d.deptno;
    
SELECT * FROM user_views;	//뷰의 세부 정보 확인(데이터 사전)
```

<br/>

## 📕ch09 인덱스(Index)

- 색인/인덱스: 대량의 레코드가 저장된 테이블의 데이터를 빠르게 검색할 수 있도록 지원해주는 객체
- 인덱스 생성 → __CREATE INDEX__ 인덱스명 __ON__ 테이블명 (컬럼명);
- 인덱스 삭제 → __DROP INDEX__ 인덱스명;

<br/>

📌예제check

```
CREATE INDEX emp_name_idx on emp3(name, sal);   
//인덱스 추가                     복합인덱스(and)
//*cf. 복합인덱스는 and 연산에서는 사용 가능, or 연산에서는 사용불가

SELECT * FROM emp3 WHERE name = 'shin691' and sal > 200;    
//F10: 인덱스를 사용해 range scan, cost(cost:1) 감소확인

SELECT * FROM user_indexes WHERE table_name = 'EMP3';   
//인덱스 정보확인(uniqueness → nonunique 인덱스)
```

<br/>

## 📕ch10 일련번호

- 시퀀스 (Sequence): 일련번호를 만들어주는 객체

*1. 시퀀스 생성*
  ```
  create sequence 시퀀스이름
      start with 숫자
      increment by 숫자
        minvalue 숫자
        maxvalue 숫자
        cycle or nocycle		//일련번호 순환여부
        cache or nocache		//시퀀스의 값을 메모리에 저장
  ```

<br/>

📌예제check

  ```
  create sequence c_emp_seq
      start with 1    //시작
        increment by 1; //1씩 증가
      
  select c_emp_seq.nextval from dual;		//nextval: 다음번호
  select c_emp_seq.currval from dual;		//currval: 현재번호
  ```

<br/>

*2. 서브쿼리를 이용한 일련번호 설정*

📌예제check

(Q) c_emp 테이블에 id를 입력하기 위한 sequence 생성하시오. 단, 300부터 시작하여 1씩 증가하고 최대값은 999로 설정

```
create sequence c_emp_sq
	   start with 300	//시작번호 조건
       increment by 1	//증가 조건
       maxvalue 999;	//최대값 조건
    
insert into c_emp values(c_emp_sq.nextval,'kim', 1000, '02-123-4567', 10);

//select * from c_emp;  결과확인
```

<br/>

(Q) c_emp 테이블에 새로운 레코드를 입력하시오. <br/>
조건: 기존내용 삭제 후, 사번: 시퀀스로 입력, 이름: 김철수, 부서번호: 10번

```
delete from c_emp;    기존 레코드 입력내용 삭제
//select nvl(max(id)+1,1) from c_emp 	→서브쿼리 응용

insert into c_emp (id, name, dept_id) values
	   ((select nvl(max(id)+1,1) from c_emp), '김철수', 10);

//select * from c_emp;  결과확인
```

<br/>

## 📕ch11 고급함수

*1. nvl* : null에 대한 대체값을 지정하는 함수

- __nvl(A,B)__ → A의 값이 null이면 B, null이 아니면 A를 반환

*2. decode*

- __decode(A, B, C, D)__ → A,B가 같으면 C, 다르면 D

*3. case* : 복잡한 조건문을 처리할 때 사용하는 함수

<br/>

📌예제check

score 테이블에서 이름, 국어점수, 영어점수, 수학점수, 총점, 평균, 등급을 나타내시오.<br/>
단, 평균점수가 90~100점이면 A등급, 80점대이면 B등급, 70점대이면 C등급, 60점대이면 D등급, 60점 미만이면 F등급

(1) decode 함수 활용
```
select name, kor, eng, mat, (kor+eng+mat) 총점, round((kor+eng+mat)/3,2) 평균,
	   decode(trunc(((kor+eng+mat)/3)/10), 10, 'A', 9, 'A', 8, 'B', 7, 'C', 6, 'D', 'F') 등급
       // *cf.trunc 버림함수                점수 구간별 등급(A~F) 설정 & 			   alias 등급
from score;
```
	
(2) case문 활용
```
select name, kor, eng, mat, kor+eng+mat 총점, round((kor+eng+mat)/3,2)  평균,
	   case //점수 구간별 등급(A~F) 설정 & alias(등급) 설정
    	 when (kor+eng+mat)/3 >= 90 then 'A'
    	 when (kor+eng+mat)/3 >= 80 then 'B'
         when (kor+eng+mat)/3 >= 70 then 'C'
         when (kor+eng+mat)/3 >= 60 then 'D'
         else 'F'
    end 등급
from score;
```

*4. rank* : 순위를 구하는 함수
- rank() over
- dense_rank() over → 동률 순위 무시
- partition by → 그룹에 대한 순위

<br/>

📌예제check

사원정보 테이블에서 전체 사원의 부서번호, 이름, 급여, 급여순위를 조회
```
select deptno, ename, sal,
	   rank() over(order by sal desc) 순위
from emp;


select deptno, ename, sal,
  	   dense_rank() over(order by sal desc) 순위  
       //dense_rank() → 동률 순위를 무시
from emp;


select deptno, ename, sal,
  	   rank() over(partition by deptno order by sal desc) 순위
       //partition by → 부서별 순위
from emp;

```
