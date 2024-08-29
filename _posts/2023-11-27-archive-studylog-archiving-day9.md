---
title: "[국비지원과정 Day 8-9] Java"
excerpt: "국비 수강기록7. 240828 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, Java]

permalink: /archive/studylog-archiving-day9/

toc: true
toc_sticky: true

date: 2023-11-27
last_modified_at: 2024-08-28
---

## 📕ch15 Thread의 우선순위

*1. Thread의 우선순위(priority)*

- Thread 클래스는 우선순위를 속성(멤버변수)으로 갖는다.
- 스레드가 수행하는 작업의 중요도에 따라 스레드의 우선순위(1~10) 지정할 수 있, 숫자가 높을수록 높은 우선순위에 해당한다.
- 멀티스레드가 실행될 때 우선순위에 따라 해당 스레드가 얻는 실행시간이 달라지게 된다. 즉, 우선순위가 높을수록 Thread에 CPU 실행시간을 더 할애하고 더 빠르게 실행된다.

  ```
  setPriority( 숫자 )	//1~10 의 우선순위를 지정할 수 있다.
  NORM_PRIORITY(5)	//기본값
  MIN_PRIORITY(1)
  MAX_PRIORITY(10)
  ```

*2. Thread의 동기화(synchronized)*
- 한 스레드가 진행 중인 작업을 다른 스레드가 간섭하지 못하도록 막는 것
- 멀티스레드 프로그램에서 단 하나의 스레드만 실행할 수 있도록 제한한다.
- 순서대로 처리되어야 하는 부분에서, 한번에 하나의 스레드만 접근할 수 있도록 락(lock)을 걸어서 정확한 자료 처리가 되도록 해야 함
- 키워드 synchronized: 임계역역을 설정(lock)하기 위해 사용
  ```
  //특정한 객체에 lock을 거는 경우
  synchronized(객체의 참조변수) { 
  }

  //method에 lock을 거는 경우
  public synchronized void test(){
  }
  ```
<br/>

## 📕ch16 내부클래스

*1. 내부클래스*
- 내부 클래스(inner class, nested class): 클래스 안에 선언된 클래스. 일반적인 클래스와 크게 다르지 않다.
- GUI애플리케이션(AWT, Swing)의 이벤트처리에 주로 사용됨
- 컴파일 했을 때 생성되는 파일명: __외부클래스명$내부클래스명.class__ 형식으로 컴파일됨

<br/>

*2 내부클래스의 특징*

- 장점
(1) 내부 클래스에서 외부 클래스의 멤버들을 쉽게 접근할 수 있다.<br>
(2) 코드의 복잡성을 줄일 수 있다.(캡슐화)
- 단점: 코드의 재사용이 어려움
  ```
  class Outer {
    private int a;
    class Inner {
    }
  }
  ```

<br/>

*3. 무명내부클래스(익명클래스, anonymous inner class)*

- 이름이 없는 내부클래스
- 클래스의 선언과 객체의 생성을 동시에 하기 때문에 이벤트 처리 등의 일회성 코드에 사용한다.
- 무명내부클래스는 이름이 없기 때문에 __외부클래스명$번호.class__ 의 형식으로 컴파일됨


<br/>


## 📕ch17 자바의 GUI 프로그래밍 기술

- CLI(Command Line Interface): 텍스트 기반의 인터페이스
- GUI(Graphical User Interface): 그래픽 기반의 인터페이스
- AWT(Abstract Window Toolkit): GUI 프로그래밍을 위한 도구로, GUI 프로그래밍에 필요한 다양한 컴포넌트를 제공한다. Java와 C로 구현
- Swing: AWT를 확장한 GUI프로그래밍 도구. 자바에서 그래픽 사용자 인터페이스를 구현하기 위하여 제공되는 클래스.


<br/>

*1. Component와 Container*

- Component : 화면 구성 요소(ex.button, lable)
- Container : 컴포넌트의 일종으로, 컴포넌트를 담는 틀 또는 그릇 역할. (ex. panel, frame)


<br/>


*2. Container*

- 독립적인 Container: Frame,Dialog
- 종속적인 Container<br/>
(1) Panel: 2개 이상의 컴포넌트를 하나로 묶어서 처리할 때 사용. Panel 안에 Panel을 넣을 수 있음(다양한 화면 배치에 활용됨)<br/>
(2) ScrollPane: 스크롤 기능 제공<br/>
- Component를 Container에 추가하는 방법
  ```
  //대상컨테이너객체.add(추가할 컴포넌트 객체)
  frame.add(button);
  ```

<br/>

*3. 프레임(JFrame)*

- Swing클래스에서 구현되는 창(독립적인 윈도우창)
  ```
  JFrame f = new JFrame("프레임의 타이틀");	//프레임 생성
  setSize(가로길이, 세로길이)		//프레임의 사이즈 설정
  pack();		// 필요한만큼 자동으로 사이즈 설정
  setVisible(true)	//프레임을 화면에 표시
  ```

- JFrame 객체: Frame(java.awt.Frame), 메뉴바(Menu Bar), 컨텐트팬(Content Pane)의 세 공간으로 구성.<br/>
(1)메뉴바: 메뉴를 부착시키는 공간이다.<br/>
	- MenuBar: 메뉴를 부착시키는 공간
	- MenuItem: 메뉴의 각 세부 항목
	- Menu: 풀다운 메뉴
	- PopupMenu: 마우스를 우클릭할 때 표시되는 메뉴
	- CheckboxMenuItem: 메뉴아이템에 체크박스 표시

  (2)컨탠트팬: 메뉴를 제외한 모든 GUI 컴포넌트를 부착시키는 공간.<br/>
	- 스윙에서는 컨텐츠팬(content pane)에만 컴포넌트를 부착할 수 있기 때문에, 화면에 출력하고자 하는 모든 GUI 컴포넌트들은 컨탠트팬에 부착하여야 한다.	
	- Container 타입
    - add() 메소드를 이용하여 컴포넌트를 부착할 수 있고, 프레임이 출력될 때 함께 출력된다.

<br/>

*4. 스윙 응용프로그램의 종료*

- 프로그램 종료 명령어 → System.exit(0);
  ```
  //cf.setDefaultCloseOperation
  public static final int DO_NOTHING_ON_CLOSE = 0;	//아무 동작 없음
  public static final int HIDE_ON_CLOSE = 1;	//화면 숨김
  public static final int DISPOSE_ON_CLOSE = 2;	//종료(현재창)
  public static final int EXIT_ON_CLOSE = 3;	//종료(모든창)
  ```

- 프레임 닫기: 프레임 윈도우의 오른쪽 상단 종료버튼(X)은 프레임 윈도우(창)를 닫는 버튼으로, 프로그램이 완전히 종료되지 않을 수 있다.
  ```
  frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);	//프레임을 닫을 때 스윙 프로그램도 함께 종료
  setDefaultCloseOperation(DISPOSE_ON_CLOSE);		//현재 창 종료
  setDefaultCloseOperation(EXIT_ON_CLOSE);	//모든 창을 닫고 프로그램을 완전히 종료
  setDefaultCloseOperation(HIDE_ON_CLOSE);	//화면 숨김(종료 X): 
  ```
<br/>

*5. Layout(화면배치)*

- FlowLayout: 왼쪽에서 오른쪽으로, 위에서 아래로 순차적으로 배치. JPanel 기본 레이아웃 
-  BorderLayout: East, West, South, North, Center(default) 5개의 영역으로 화면을 나누고 각 영역에 1개의 컴포넌트만 배치 가능함. JFrame, Dialog 기본 레이아웃<br/>
\*cf. 한 영역에 2개 이상의 컴포넌트를 넣어야하는 경우, JPanel을 활용한다.
- 기본 Layout 변경
  ```
  setLayout( new 레이아웃클래스() );
  setLayout( new FlowLayout() );
  ```
- AbsoluteLayout 절대좌표: Layout을 사용하지 않고 직접 좌표를 지정할 경우
  ```
  setLayout(null);
  JButtonbutton=newJButton("버튼");
  button.setBounds(0,0,50,25); 	// x,y,width,height
  add(button);
  ```

*cf.스윙 응용프로그램에서 main() 메소드의 기능과 위치<br/>
스윙 응용프로그램에서는 main()의 기능을 최소화하는 것이 유리하다.<br/>
main()에는 실행되는 시작점으로서 프레임을 생성하는 코드 정도만 두고, 나머지 기능은 프레임 클래스에 작성하는 것이 좋다.<br/>

<br/>

*6. 이벤트 처리(Event Handling)*
- user와 system 간의 상호작용: user : request(요청) → system : response(응답)
- 이벤트 처리의 3요소<br/>
(1) 이벤트 소스 : 이벤트의 대상<br/>
(2) 이벤트 리스너 : 이벤트가 발생했는지 검사<br/>
(3) 이벤트 핸들러 : 이벤트가 발생했을 때 처리할 코드<br/>
- 컴포넌트에 이벤트 처리 기능을 추가하는 코드
  ```
  //이벤트소스.이벤트리스너( 이벤트핸들러)
  button.addActionListener(this);
  ```
- 대표적인 이벤트 처리 클래스<br/>
(1) ActionEvent - 버튼, 메뉴 아이템 등을 클릭할 때<br/>
(2) MouseEvent - 마우스 이벤트<br/>
(3) KeyEvent – 키이벤트<br/>
- Listener와 Adapter<br/>
Listener : 이벤트 처리 클래스를 호출하기 위한 인터페이스<br/>
Adapter : 이벤트 리스너를 구현한 클래스<br/>

<br/>

*7. Window Builder*

- 위지윅(WYSIWYG: What You See Is What You Get, "보는 대로 얻는다")<br/>
인쇄된 문서, 웹 페이지, 슬라이드 프레젠테이션 등 완성된 결과물로서 인쇄 또는 표시될 때의 모습과 닮은 형태로 콘텐츠의 편집이 가능한 시스템
- Window Builder 플러그인 설치: 이클립스에서 제공하는 GUI환경 플러그인<br/>
Help → Install New Software → "https://download.eclipse.org/windowbuilder/updates/release/latest" → 플러그인 설치 후 이클립스 재시작<br/>
New → Other → Window Builder → JFrame: 해당 경로로 확인/활용 가능(Design x탭)











