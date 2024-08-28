---
title: "[국비지원과정] Database(4) PL/SQL"
excerpt: "국비 수강기록12. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, DB]

permalink: /archive/studylog-archiving-database4/

toc: true
toc_sticky: true

date: 2023-12-07
last_modified_at: 2024-08-29
---

## 📕 12 PL/SQL

- PL/SQL(Oracle's Procedural Language extension to SQL)<br/>
오라클에 내장되어 있는 절차형 언어. 변수 선언, 조건문, 반복문 등을 지원함

- PL/SQL의 형식
  ```
  declare   선언부(변수, 상수, CURSOR 등)
  begin      실행부(SQL 명령어, 반복문, 조건문 등)
  exception  예외처리부
  end;
  ```

- 명령어의 종류
	- Anonymous Block(익명 블록): 이름이 없는 블록
	- Procedure(프로시저): DB에 저장되어 반복적으로 사용할 수 있는 블록. 매개 변수 입출력 가능
	- Function(함수): 입력 매개변수만 사용 할 수 있고 "리턴타입"을 반드시 지정해야 함(저장 프로시저와의 차이점)

<br/>

*1.저장 프로시저 Stored Procedure(SP)*

```
create or replace procedure sal_p(p_empno number)
--create or replace procedure 프로시저이름(매개변수)

is --(변수선언)
    begin --(문장)
        update emp
        set sal = sal*1.1
        where empno = p_empno;
    end;
/
```

<br/>

*📌예제*

```
create or replace procedure memo_insert_p(p_writer varchar, p_memo varchar, p_ip varchar)
--프로시저 생성                 프로시저이름    변수명   자료형(입력매개변수)
is  --(변수선언)
    begin   --(프로시저 실행부)
        insert into memo (idx, writer, memo, ip)
        values (memo_seq.nextval, p_writer, p_memo, p_ip);
    end;
/

execute memo_insert_p('김철수', '메모...', '192.168.0.10');

select * from memo;

select * from user_source where name='MEMO_INSERT_P';
--     시스템테이블(프로시저 내용확인)        '대문자로'
```

<br/>

*2. 함수(Function)*

```
CREATE OR REPLACE FUNCTION 함수이름(입력매개변수)
RETURN 리턴자료형
IS    변수 선언
BEGIN     실행할 문장
return    →저장프로시저와 달리 반드시 리턴타입이 있어야한다.
END;
```

<br/>

*📌예제 *

```
create or replace function sal_f(p_empno number)
--                   함수    함수이름(입력값)
return number
--     리턴타입
is       --변수선언
    v_sal number;   
begin   --실행부
        update emp set sal=sal*1.1 where empno=p_empno;
        select sal into v_sal from emp
        where empno=p_empno;
        return v_sal; 
end;
/

select empno, ename, sal, sal*1.1, sal_f(empno) from emp;

var salary number;
execute :salary :=sal_f(7499);
--      :변수 := → 대입
print salary;
```

<br/>

- if 문
  ```
  if 조건 then
  elsif 조건 then
  else
  end if;
  ```
<br/>

```
create or replace procedure dept_p(p_empno number)
-- 저장프로시저               프로시저이름  매개변수

is  
    v_deptno number;    --변수선언
begin 
    select deptno into v_deptno from emp  where empno=p_empno;
--          필드        변수
    dbms_output.put_line('부서코드:'||v_deptno); 
--        패키지.(함수)             || 결합

    if v_deptno=10 then
        dbms_output.put_line('교육팀 직원입니다.');
    elsif v_deptno=20 then
        dbms_output.put_line('홍보팀 직원입니다.');
    elsif v_deptno=30 then
        dbms_output.put_line('기획팀 직원입니다.');
    else 
        dbms_output.put_line('기타부서 직원입니다.');
    end if;
end;
/
```

<br/>

- for loop문
  ```
  for 카운트변수 in [reverse] 시작값 .. 마지막값 loop
    --반복할 문장들
  end loop;
  ```

<br/>

```
delete from emp where empno<=100;

이름이 없는 블록
begin
    for cnt in 1..100 loop
--   카운트변수 시작..마지막값   
    insert into emp (empno, ename, hiredate) values (cnt, 'test'||cnt, sysdate);
    end loop;
dbms_output.put_line('100개의 레코드가 입력되었습니다.');
end;
/
```

<br/>

- Loop 문
  ```
  EXIT : LOOP 종료
  EXIT WHEN : LOOP 종료 조건 설정
  loop
    --반복할 문장들
    exit [when 조건문]
  end loop;
  ```

<br/>

```
declare cnt number := 1;
--     변수명 자료형 := 대입

begin 
    loop
    --반복할 명령어
    insert into emp (empno, ename, hiredate) values (cnt, 'test'||cnt, sysdate);
    exit when cnt>=100;     
--EXIT : LOOP 종료
--EXIT WHEN 조건문 : LOOP 종료 조건 설정   
    cnt := cnt+1;
    end loop; --loop의 끝
dbms_output.put_line('100개의 레코드가 입력되었습니다.');
end; --bigin의 끝
/--프로시저의 끝
```

<br/>

- WHILE LOOP; FOR 문과 비슷하며 조건이 TRUE일 경우만 반복되는 LOOP
  ```
  declare cnt number := 1;
  --선언부 변수명 자료형:=초기값;
  begin
      while cnt<=100 loop
      insert into emp (empno, ename, hiredate) values (cnt, 'test'||cnt, sysdate);
      cnt := cnt+1;
      end loop; --while반복문의 끝
  dbms_output.put_line ('100개의 레코드가 입력되었습니다.');
  end; --begin의 끝
  /--프로시저의 끝
  ```

<br/>

- 커서(Cursor): select 명령어의 실행 결과를 하나의 행 단위로 탐색하는 객체
  ```
  OPEN 커서이름; →	커서 열기
  FETCH 커서이름 INTO 변수; → 커서 페치(커서가 현재 가리키는 레코드를 변수에 저장)
  CLOSE 커서이름;	→ 커서 닫기
  ```
    
    <br/>

📌예제check
```
create or replace procedure cursor_p(p_deptno number)
is
  cursor cursor_avg is
    select dname,count(empno) cnt, round(avg(sal),1) sal
    from emp e, dept d
    where e.deptno=d.deptno
      and e.deptno=p_deptno
    group by dname;
  dname varchar(50);
  cnt number;
  sal_avg number;
begin
  open cursor_avg;
  fetch cursor_avg into dname, cnt, sal_avg;
  dbms_output.put_line('부서명:'|| dname);
  dbms_output.put_line('사원수:'|| cnt);
  dbms_output.put_line('평균급여:'|| sal_avg);
  close cursor_avg;
end;
/

execute cursor_p(10);
```

<br/>

📌예제check
```
create or replace procedure cursor2_p
is
 cursor cursor_avg is
     select dname, count(empno) cnt, round(avg(sal),1) sal
     from emp e, dept d
     where e.deptno=d.deptno
     group by dname;
begin
    for row in cursor_avg loop
    dbms_output.put_line('부서명: '||row.dname);
    dbms_output.put_line('사원수: '||row.cnt);
    dbms_output.put_line('평균급여: '||row.sal);
    end loop;
end;
/

execute cursor2_p;
```

<br/>

- Trigger(방아쇠) : 연쇄적인 동작을 정의하는 객체
	- INSERT, UPDATE, DELETE 문이 실행될 때 자동으로 실행되는 기능
    -	Before Trigger : INSERT, UPDATE, DELETE 문이 실행되기 전에 실행
    - After Trigger : INSERT, UPDATE, DELETE 문이 실행된 후 실행

  ```
  create or replace trigger 트리거이름
  before/after
    insert/update/delete on 테이블이름
    --실행할 명령어들
  ``` 

<br/>

```
create or replace trigger sum_t
after   --이벤트가 발생한 후의 트리거 동작
    insert or update or delete on emp
declare     --변수선언
    avg_sal number;
begin 
    select avg(sal) into avg_sal from emp;
--            필드값  →  변수에 저장    
    dbms_output.put_line('급여평균: '||avg_sal);
end;
/
```