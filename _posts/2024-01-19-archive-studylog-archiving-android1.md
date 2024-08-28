---
title: "[국비지원과정] Android(1)"
excerpt: "국비 수강기록15. 240829 블로그 옮김"

categories:
  - Archive
tags:
  - [국비지원, Java, Android]

permalink: /archive/studylog-archiving-android1/

toc: true
toc_sticky: true

date: 2024-01-19
last_modified_at: 2024-08-29
---

## 📱 안드로이드 스튜디오

### __*1. 안드로이드 애플리케이션의 구성요소*__

- 화면 → 액티비티
- 백그라운드 처리 → 서비스
- 데이터 공유 → 컨텐트 프로바이더
- 시스템 이벤트 수신 → 브로드캐스트리시버
- 애플리케이션 통신 → 인텐트
  
<br/> 
        
### __*2. 4대 컴포넌트*__

- 액티비티 Acivity
	1) 애플리케이션 구성의 기본 단위		<br/> 
    (ex. 연락처 조회, 편집)<br/> 
	2) 비주얼 인터페이스를 가지는 애플리케이션<br/> 
      - 하나의 액티비티에는 그리기가 가능한 윈도우를 부여함<br/> 
      - 하나의 애플리케이션은 여러 액티비티로 구성 가능<br/> 
	3) Activity class를 상속받음<br/> 
	4) 정해진 라이프 사이클을 따라 움직임<br/> 
      - __onCreate__ : 액티비티 생성(최초 1회), 가로/세로 전환시 호출됨<br/> 
      - __onResume__ : 다른 화면으로 갔다가 다시 돌아올 때<br/> 
      - __onPause__ : 다른 화면으로 이동할 때(잠시 멈춤, 화면상에서만 숨김 처리됨)<br/> 
      - __onDestroy__ : 현재 액티비티가 종료될 때(메모리에서 제거됨)
    
    
- 서비스 Service
	1) 보이지 않는 애플리케이션
	2) 화면없이 백그라운드로 실행
	3) Service 클래스를 상속받음<br/> (ex.음악재생, RSS 확인 등)

- 브로드캐스트 리시버 Broadcast receiver
	1) 브로드캐스트되는 메시지에 응답 → 주로 시스템상태에 관련된 메시지에 반응
	2) BroadcastReceiver 클래스를 상속받음
	3) 사용자에게 알리기 위해서는 알림화면(notification, toast)을 사용    <br/> (ex.시간대 변경, 환경설정 변경, 파일 다운로드 상태)
        
- 컨텐트 프로바이더 Content Provider
	1) 다른 애플리케이션이 사용할 수 있도록 특정 애플리케이션의 데이터를 제공해주는 인터페이스
	12) ContentProvider 클래스를 상속<br/> (ex.SQLite, File, Memory 등)
    
 - 인텐트 Intent
	1) 화면 전환 또는 서비스 호출 방법.  컴포넌트 간의 통신을 담당
	2) 액티비티에서 다른 액티비티나 서비스 호출
	3) 액티비티, 서비스, 브로드캐스트 리시버를 호출하거나 컴포넌트를 호출함과 동시에 데이터를 전달

<br/> 
     
### __*3. 안드로이드 애플리케이션의 작성과 구동*__

- 작성과 배포
1) 안드로이드 애플리케이션은 java 로 작성 후 컴파일
2) 컴파일된 java 클래스는 애플리케이션에 필요한 데이터가 추가되어 \*.apk 파일로 생성된다.
<br/> (\*cf. \*.apk 는 안드로이드 배포, 설치 버전)

- 애플리케이션은 리눅스 프로세스 내에서 실행
1) 각 애플리케이션에는 고유한 리눅스 userid가 부여
2) 각 프로세스는 자기 자신의 java 가상 머신 사용
<br/> 
     
### __*4. 안드로이드 프로젝트의 구조*__

> <app\><br/> 
> - manifests(__AndroidManifest.xml__) →  앱의 환경설정 파일,사용권한 등의 정보
> - java → 소스코드
>    - 액티비티 (화면)<br/> (ex.class 클래스 extends ... Activity)
>    - 일반클래스
> - res: 리소스
>   -  drawable – 이미지, 아이콘
>    -  layout : 화면 레이아웃
>   -  values : 텍스트, 색상, 배열<br/>  \*cf. 자바 클래스(컨트롤러)와 리소스(뷰)의 연결을 위해 R.java 클래스가 자동으로 작성됨

<br/>

### __*5. 안드로이드 코드 작성 시 기본 사항*__

1. __스트링__은 다국어 처리를 위해 정적인 텍스트보다는 strings.xml에 작성할 것을 권장
```
@string/hello_world <br/> 
→ res/values/strings.xml 의 name이 hello_world인 태그
```

2. __Toast__ : 간단한 팝업 메시지 표시에 사용 → ```Toast.makeToast(컨텍스트,메시지,시간).show()```

3. __Activity(액티비티)__ : 화면을 가지는 프로그램

4. __Service(서비스)__ : 화면이 없이 백그라운드에서 실행되는 프로그램

5. __Context(컨텍스트)__ : 프로그램의 흐름(현재 실행중인 화면)

6. __getApplicationContext()__  :  현재 실행중인 액티비티(화면)의 이름

7. __Widget__ 의 길이 설정 :
```
wrap_content : 필요한 사이즈만큼
fill_parent  : 100%
match_parent : 상위 Widget을 기준으로 100%
```

8. __텍스트__ 의 폰트 사이즈
>android : textSize = “36sp”
>- sp : System Pixel Size (디바이스에 상대적인 사이즈, 텍스트에 사용됨)
>- dp : Dependent Pixel Size ( widget에 사용됨 )
>- pt, px : 비추천

9. __Log__ 출력 방법(로그캣에 출력됨)<br/>
Log.옵션( "태그", "메시지" ) : 로그의 옵션은 기능적으로는 동일하며 색상 구분 처리됨
```
Log.i
Log.v
Log.e
Log.w
```

10.  화면 이동 방법
 ```
 // Intent(인텐트) - 활동(화면 전환)
 new Intent(현재화면.this, 다음화면.class)
 startActivity(인텐트)
 ```

 <br/>

### __*6. 안드로이드 스튜디오 설정*__

InteliJ 기반의 툴 <br/>
Gradle : InteliJ에서 사용하는 빌드 도구
- __Keymap__ → Eclipse로 설정
- __Editor__ → General – Auto import (자동 import 옵션)<br/>
  Add unambiguous imports on the fly *체크*<br/>
    Optimize imports on the fly *체크*

<br/>

### __*7. 이벤트 처리 방법*__

1. 위젯 변수 선언
2. 위젯 객체 생성 ( R.java 에서 아이디 참조)<br/>
  `findViewById(리소스아이디)`
3. 이벤트 처리<br/>
  `뷰.setOn이벤트Listener( 이벤트 핸들러 구현 );`

15. 레이아웃의 종류<br/>
RelativeLayout : 다른 위젯을 기준으로 상대적인 배치<br/>
LinearLayout : 위에서 아래로, 좌에서 우로 순서대로 배치<br/>

16. 리소스의 id 규칙
> android:id="@+id/아이디"<br/>
>    @ → Resource를 의미<br/>
>    +id → R.java에 id를 추가<br/>

	android:id="@id/아이디"
    	//(에시1) R.java에 있는 id를 검색(추가가 아님)
        
    android:id="@android:id/아이디"
    	//(에시2) android 내장 R.java에 있는 id를 검색


