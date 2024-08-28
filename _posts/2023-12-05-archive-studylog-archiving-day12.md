---
title: "[국비지원과정 Day 10-12] Java"
excerpt: "국비 수강기록9. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, Java]

permalink: /archive/studylog-archiving-day12/

toc: true
toc_sticky: true

date: 2023-12-05
last_modified_at: 2024-08-29
---

## 📕ch18 파일입출력

*1. 입출력 방법*
- Stream 스트림: 데이터의 논리적인 흐름. 데이터를 운반하는데 사용되는 연결통로<br/>
단방향통신만 가능하기 때문에 하나의 스트림으로 입력과 출력을 동시에 처리할 수 없다.<br/>
(1)byte 기반 스트림 <br/>
InputStream 입력스트림: 프로그램 → 데이터 <br/>
OutputStream 출력스트림: 프로그램 ← 데이터<br/>
	\*cf. 한글은 2byte 또는 3byte가 한글자로 구성되어 있다.<br/>
    (2)문자 단위 입출력 → InputStreamReader, OutputStreamWriter    <br/>
(3)buffer를 이용한 입출력 → BufferedReader, BufferedWriter<br/>

<br/>

*2. 버퍼 입출력*
- 버퍼(buffer): 데이터를 전송하기 전에 일시적으로 데이터를 보관하는 임시 메모리 영역(바이트배열)<br/>
	스트림의 입출력 효율(속도 향상)을 높이기 위해 버퍼를 사용한다.<br/>
 	BufferedReader(입력), BufferedWriter(출력)

- 버퍼 플러시(buffer flush): 버퍼에 남아 있는 데이터를 출력시키는 것. 버퍼를 비우는 동작<br/>
\*cf. 버퍼가 가득 찼을 때만 출력소스에 출력하기 때문에 BufferedOutputStream의 버퍼에 출력부분이 남아있는 채로 프로그램이 종료될 수 있으므로 주의해야한다. <br/>
\*cf. 모든 출력작업을 마친 후 close() 또는 flush() 를 호출해 버퍼에 있는 모든 내용이 출력되도록 한다.<br/>

<br/>

📌예제 check
- (에러메세지)java.io.IOException: 파일 이름, 디렉터리 이름 또는 볼륨 레이블 구문이 잘못되었습니다. → 파일경로에 백슬래시(\), 슬래시(/) 구분자 확인하기!
  
![](https://velog.velcdn.com/images/92miindy/post/2b9955be-c8da-4432-bb39-de55762aa85e/image.png)


- (에러메세지)The local variable reader may not have been initialized<br/>
→ 지역변수의 초기화 작업을 하지 않을 경우 예외가 발생할 가능성이 있기 때문에 되도록 변수선언 및 초기화 작업을 하는 습관을 들이자!

![](https://velog.velcdn.com/images/92miindy/post/bb30c4dc-d3e1-474b-8d0a-abf2989e31bb/image.png)

<br/>

## 📕ch19 네트워킹 

두 대 이상의 컴퓨터를 케이블로 연결하여 네트워크를 구성하는 것.<br/>
컴퓨터를 서로 연결하여 데이터를 손쉽게 주고받거나 주변기기를 함께 공유하는 것을 가능하게 한다.<br/>

*1. 클라이언트 & 서버 용어/개념정리*
- 서버: 서비스를 제공하는 컴퓨터 service provider
- 클라이언트: 서비스를 사용하는 컴퓨터 service user
- 서비스: 서버가 클라이어트로부터 요청바은 작업을 처리하여 결과를 제공하는 것
- ip주소: 컴퓨터(호스트 host) 를 구별하는데 사용되는 고유한 값. 인터넷에 연결된 모든 컴퓨터는 ip주소를 갖는다.
- InetAddress: 자바에서 ip주소를 다루기위한 메서드들이 정의되어 잇는 클래스

<br/>

*2. 주요 네트워크 관련 명령어*
- ipconfig: 현재 연결된 IP 구성을 확인 할 수 있다.
- ping: 네트워크를 통해 상대에게 접근할 수 있는가를 확인하기 위한 명령어
- nslookup: 네임서버를 조회하는 명령어
- netstat: NETwork STATus, 네트워크 상태 정보
- tracert: 서비스 경로 추적
- arp: IP address와 Mac Address 조회

<br/>

*3. URL uniform resource locator*

- 인터넷에 존재하는 여러 서버들이 제공하는 자원에 접근할 수 있는 주소를 표현
- 형태 → 프로토콜://호스트명:포트번호/경로명/파일명?쿼리스트링#참조 (포트번호, 쿼리, 참조는 생략가능)
- URL 클래스: 자바에서 URL을 다루기위한 메서드를 정의해놓은 클래스

<br/>

*4. 프로토콜 Protocol: 서로 다른 컴퓨터 간의 의사소통을 위한 통신 규약*

- 프로토콜의 종류<br/>
TELNET : 텍스트 기반의 원격접속 서비스<br/>
IP (Internet Protocol)<br/>
TCP (Transmission Control Protocol)<br/>
UDP (User Datagram Protocol)<br/>
FTP (File Transfer Protocol)<br/>
SMTP (Simple Mail Transfer Protocol)<br/>
HTTP (Hyper Text Transfer Protocol)  <br/>
POP3 (Post Office Protocol)<br/>
DHCP (Dynamic Host Control Protocol)  <br/>
ARP (Address Resolution Protocol): IP 주소를 물리적 주소로 변환

<br/>

*5. 소켓(socket)프로그래밍*

- 소켓: 프로세스 간의 통신에 사용되는 양쪽 끝단(endpoint)을 의미.<br/>
서버 소켓(서비스를 제공하는 소켓) ↔ 클라이언트 소켓(서비스에 접속하는 소켓)

- 소켓(Socket)을 이용한 네트워크 프로그래밍<br/>
(1) 서버는 서비스 제공을 위한 서버소켓을 생성하고 클라이언트의 접속을 기다린다<br/>
(2) 클라이언트가 서버에 접속한다.<br/>
(3) 서버에서 연결을 수락한 후 데이터 송수신 처리<br/>
\*cf. 자바에서는 java.net패키지를 통해 소켓 프로그래밍을 지원함.(소켓통신에 사용되는 프로토콜에 따라 다른 종류의 소켓을 구현하여 제공

- TCP와 UDP<br/>
TCP/IP 프로토콜: 이기종 시스템간의 통신을 위한 표준 프로토콜(프로토콜의 집합). OSI 7계층의 전송계층(transport layer)에 해당.<br/>
TCP: 연결된 상태에서 데이터를 전송하는 방식. 1:1통신방식. <br/>
UDP: 연결되지 않은 상태에서 일방적으로 데이터를 전달하는 비연결기반 통신방식.

- TCP소켓 프로그래밍<br/>
(1) Socket 클래스: 프로세스간 통신을 담당. InputStream, OutputStream을 가지고 있고, 이 두스트림을 통해 프로세스간의 통신(입출력)이 이루어진다.<br/>
(2) ServerSocket 클래스: 포트와 연결(bind)되어 대기, 외부의 연결요청이 들어오면 Socket을 생성해서 소켓과 소켓간의 통신이 이루어지도록한다. 동일한 프로토콜에서는 한 포트에 하나의 ServerSocket만 연결할 수있다.

<br/>

📌예제 check
Chat Server & Client → 간단한 채팅 프로그램(콘솔창에 텍스트 입력)

![](https://velog.velcdn.com/images/92miindy/post/7a9d1df3-ecd1-4054-bfed-a0c598102b6b/image.png)


![](https://velog.velcdn.com/images/92miindy/post/d3980c94-9199-4b10-b786-29b247c9281e/image.png)

