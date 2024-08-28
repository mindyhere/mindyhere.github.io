---
title: "[국비지원과정] Database(5) 오라클 백업 및 복원"
excerpt: "국비 수강기록14. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, DB]

permalink: /archive/studylog-archiving-database5/

toc: true
toc_sticky: true

date: 2024-01-19
last_modified_at: 2024-08-29
---

## 📕 13 백업 및 복원

*1. 기본툴을 이용한 백업 및 복원 (cmd창 입력)*

- exp.exe 를 이용한 백업
  
  ```
  cmd창 → exp userid=java/java1234 file=c:/data/test.bak ; 
  --                  아이디/비번                백업파일이름
  ```

- imp.exe 를 이용한 복원
  ```
  --cmd창 → imp java/java1234 file=c:/data/test.bak full=y ignore=y; 
  --       복원  아이디/비번               백업파일이름  전체복구 에러메시지무시
  ```


*2. SQL Developer를 이용한 백업 및 복원*

도구 → 데이터베이스 익스포트: 형식에서 엑셀파일, csv파일, insert script 등 다양한 형식을 선택할 수 있음.

*3. 스케줄러를 이용한 자동 백업*

(1) 백업을 위한 디렉토리 생성 → c:\backup <br/>
(2) 백업을 위한 batch file 작성 → c:\backup\oracle_backup.bat     <br/>
\*cf.exp 명령어는 두줄이 아닌 한줄로 작성해야 함, ORACLE_SID는 대문자로 작성<br/>

  ```
  @echo off
  set ORACLE_SID=xe
  for %%a in (%date%) do set day=%%a
  md c:\backup\%day%        --md: make directory 디렉토리 생성

  C:/app/본인계정/product/21c/dbhomeXE/bin/exp java/java1234 file=c:\backup\%day%\%day%-backup.dmp

  --cmd 창 실행
  cd c:/backup      --cd: change directory 디렉토리 이동
  oracle_backup.bat     배치파일 실행 → 백업실행
  ```
    
(3)작업스케줄러 실행(윈도우 응용프로그램)
- 일반 > 이름 지정
- 트리거 > 스케줄 지정
- 동작 > 프로그램 시작<br/>
작업을 생성한 후 우클릭하여 실행을 하면 실행 중으로 상태가 바뀌고 지정한 스케줄러대로 작업 실행(준비상태인 경우 실행되지 않음)


