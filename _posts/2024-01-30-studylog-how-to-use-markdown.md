---
title: "마크다운 문법 정리"
excerpt: "내가 보려고 정리한 마크다운 문법. 240822 블로그 옮김"

categories:
  - Studylog
tags:
  - [Markdown]

permalink: /studylog/how-to-use-markdown/

toc: true
toc_sticky: true

date: 2024-01-30
last_modified_at: 2024-08-22
---

### ✏️ 내가 보려고 정리하는 마크다운 문법 
마크다운(Markdown)은 일반 텍스트 기반의 경량 마크업 언어로, 일반 텍스트로 서식이 있는 문서를 작성하는 데 사용된다.

그간 필요에 따라 구글링해가며 사용해 본 바로는, 환경에 따로 출력결과가 다소 상이한 것 같다는 단점 외에 비교적 문법이 쉽고 간단한 것이 특징적이다. 또 마크다운만으로 표현이 부족한 부분은 HTML 태그를 활용해 보완이 가능하다.

때문에 이번 기회에 마크다운 기본 문법을 정리해보려고 한다. 

<br>

## 1. 제목
```
# Header1
## H2
### H3
#### H4
##### H5
###### Header6 
```
`#` 개수가 커질수록 그만큼 낮은 수준의 소제목이 된다. <br/>
즉, __\### Header 3__ 이라면 아래와 같이 출력된다.

>### Header 3

<br/>
아래와 같이 등호(=) 또는 하이픈(-) 기호와 함께 간단히 표현하는 것도 가능하다.

```
큰 제목
====================


중간 제목
---------------------
// 출력 결과 확인
```

>큰 제목
>====================
>
><br>
>
>중간 제목
>---------------------

---

<br/>
<br/>

## 2. 폰트 스타일 적용
~~~
__BOLD__ or **볼드체**
_italic_ or *이탤릭체*
~~Strikethrough  취소선~~
<u>underline  밑줄</u>
<span style="color:red">글자색</span>
<span style="background-color:yellow">하이라이트</span>
<span style="background-color:#ffe6e6">RGB코드 참조</span>
~~~

>__BOLD__&nbsp;&nbsp; or &nbsp;&nbsp;**볼드체** <br/>
>_italic_&nbsp;&nbsp; or &nbsp;&nbsp;*이탤릭체*  <br/>
>~~Strikethrough &nbsp;&nbsp; 취소선~~ <br/>
><u>underline &nbsp;&nbsp; 밑줄</u> <br/>
><span style="color:red">글자색</span> <br/>
><span style="background-color:yellow">하이라이트</span> <br/>
><span style="background-color:#ffe6e6">RGB코드 참조</span>

---
<br/>
<br/>

## 3. 목록
```
1.  순번이 있는 목록일 경우
	1. 하위 수준 목록표시 가능
2.  순번의 경우
	1. 동일 수준의 목차일 경우 
    1. 실제 표기한 __숫자에 관계없이__ 순번이 매겨지는 것 같다.
    	3. 탭(tab)키로도 적용 가능
3.  출력 결과 확인!
```

>1.  순번이 있는 목록일 경우
>	1. 하위 수준 목록표시 가능
>2.  순번의 경우
>	1. 동일 수준의 목차일 경우 
>    1. 실제 표기한 __숫자에 관계없이__ 순번이 매겨지는 것 같다.
>    <br/>~~3. 탭(tab)키로도 적용 가능 : 되는 곳이 있고 안되는 곳도 있고 하나봄~~
>3.  출력 결과 확인!


<br/>

```
- 번호 없는 목록일 경우: -, *, + 기호 사용 
- 하위 수준 목록표시
    * 별표(asterisks) 
    + 덧셈기호(plus sign)
    - 하이픈(hyphen)
    	- 실제 기호에 상관없이 동일 수준일 경우,
       - 모두 동일하다
    	\* 탭(tab)키를 적절히 사용해서 표현하는 것도 적용 가능
- 출력 결과 확인!
```

>- 번호 없는 목록일 경우: -, *, + 기호 사용 
>- 하위 수준 목록표시
>    * 별표(asterisk) 
>    + 덧셈기호(plus sign)
>    - 하이픈(hyphen)
>    	- 실제 기호에 상관없이 동일 수준일 경우,
>       - 모두 동일하다
>    	\* 탭(tab)키를 적절히 사용해서 표현하는 것도 적용 가능
>- 출력 결과 확인!


<br/>
<br/>

## 4. 인용, 첨자, 각주
__1. 텍스트 인용__
```
> 인용문 표시
BlockQuote
>> 중첩인용문
nested blockquote
break
>>> 출력 결과 확인
```

> 인용문 표시
BlockQuote
>> 중첩인용문
nested blockquote
>>> 출력 결과 확인

__2. 첨자__
- 위첨자 `<sup>` 태그 활용 
	`<sup>위첨자</sup>텍스트 확인하기` → `<sup>` 태그를 활용한 <sup>위첨자</sup>텍스트 확인하기
- 아래첨자 `<sub>` 태그 활용
`<sub>아래첨자</sub>텍스트 확인하기` → `<sub>` 태그를 활용한 <sup>아래첨자</sup>텍스트 확인하기


__3. 각주__
`[^]`을 활용해 각주(footnote)를 적용할 수 있다.
~~(벨로그에서는 제대로 렌더링 되지 않는 것 같지만)~~
```
This is a general sentence. This is a first Footnotes[^1]
This is a second footnotes[^2]
각주 테스트[^test]

[^1]: 첫번째 각주
[^2]: 두번째 각주
[^test]: 세번째 각주
```

This is a general sentence. This is a first Footnotes[^1]
This is a second footnotes[^2]
각주 테스트[^test]

[^1]: 첫번째 각주
[^2]: 두번째 각주
[^test]: 세번째 각주

---
<br/>
<br/>

## 5. 코드블록
 - 기본 코드블록 : 백틱(\`), 물결표(~), 공백, 탭 활용
1. 백틱(\```) or 물결표(\~~~) 연속 3개
이 때 코드블록 시작에 언어 이름까지 적어주면 예약어, 변수 등등에 하이라이팅이 들어가게 된다.

>\```java
//코드블록 예시 <br/>
public class Main { <br/>
&nbsp;&nbsp;&nbsp;&nbsp;public static void main(String[] args) { <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System. out.println("Hello World" ); <br/>
&nbsp;&nbsp;&nbsp;&nbsp;} <br/>
}\```

  ```java
  //코드블록 예시
  public class Main {
      public static void main(String[] args) {
         System. out.println("Hello World" );
      }
  }
  ```

2. `[ ][ ][ ][ ]` 4개 이상의 공백 <br/>
또는 <br/>
`[tab]` 한개 이상의 탭 <br/>
 (\*cf. 이 경우, 들여쓰기 끝나는 지점까지가 코드블록에 해당한다.)

- 인라인 코드 : 문장을 백틱(\`inline\`)으로 감싸서 적용할 수 있다. → `inline 예시` 

---
<br/>
<br/>

## 6. 문단구분 및 줄바꿈
- 문단구분
1. 수평선활용 : --- or ___ or *** or `<hr>` 태그

---
___
***
<hr>

2. 라인(line) 추가 : 구분하고자 하는 문단 사이에는 엔터키 2번으로 단락을 구분한다. 즉, 공백 두 줄(line)을 추가해 문단을 나눌 수 있다. 

- 줄바꿈
1) 연속 2개 이상의 공백+엔터
2) `<br>` 태그 활용     

\*cf. 문장 들여쓰기 : 띄어쓰기는 기본적으로 한개만 인식하기 때문에, 들여쓰기를 하고 싶다면 `<&nbsp>` 태그를 활용한다.
    
    &nbsp 태그활용
    &nbsp&nbsp&nbsp&nbsp 결과확인 

 
>`&nbsp` 태그활용 <br/>
&nbsp;&nbsp;&nbsp;&nbsp; 결과확인 

---
<br/>
<br/>

## 7. 표(Table) 만들기
\- (hyphen), | (Vertical bar), : (colon) 을 활용해 본문에 표를 넣을 수 있다.
 - 3개 이상의 하이픈(-) → 헤더 구분, 콜론(:) → 정렬방향 설정
 - 가장 좌측/우측의 세로선(|)은 생략 가능
 - 줄바꿈이 필요한 경우 `<br>` 태그를 활용할 수 있다.

```
왼쪽정렬(기본)|가운데정렬|오른쪽정렬|비고
---|:----:|----:|----
왼쪽|가운데정렬|오른쪽정렬|줄바꿈테스트
1|마크다운|Markdown|---										
2|표 만들기|table test|줄바꿈이 필요한 경우,<br>`<br>` 태그를 활용할 수 있다.                            
3|출력 결과 확인|test12341234|---
```


>왼쪽정렬(기본)|가운데정렬|오른쪽정렬|비고
>---|:----:|----:|----
>왼쪽|가운데정렬|오른쪽정렬|줄바꿈테스트
>1|마크다운|Markdown|---										
>2|표 만들기|table test|줄바꿈이 필요한 경우,<br/>`<br>` 태그를 활용할 수 있다.                            
>3|출력 결과 확인|test12341234|---

---
<br/>
<br/>


## 8. 링크
1. 인라인 링크 : 기본적으로 외부주소를 괄호(`[]()`)로 감싸 링크를 적용할 수 있으며, 일반URL 또는 <> 안의 URL은 자동적으로 링크를 사용한다. 
```
1) [Google](https://google.com)
2) [Naver](https://naver.com "네이버로 이동합니다")	// "hint" 적용가능
3) https://velog.io/	 or		<https://velog.io/>
```
    
>__*적용 결과 확인*__
>1) [Google](https://google.com)
>2) [Naver](https://naver.com "네이버로 이동합니다")
>3) https://velog.io/ &nbsp;&nbsp; or	&nbsp;&nbsp; <https://velog.io/>

<br/>

2. 참조링크 방법
```
Link : [my_velog][velogLink]
Link : [my_github][1]
[velogLink]: https://velog.io/@92miindy/posts 
[1]: https://github.com/mindyhere "go github" 	// "hint" 적용가능
```

> __*적용 결과 확인*__ <br/>
> Link : [my_velog][velogLink] <br/>
> Link : [my_github][1] <br/>
> [velogLink] : https://velog.io/@92miindy/posts <br/>
> [1] : https://github.com/mindyhere "go github"  <br/>

3. 문서 내부 링크
목차를 대괄호 `[]`로 묶고 링크를 걸 페이지의 제목을 소괄호 안에 `(#제목)` 써주면 해당 라인으로 이동하게된다. → `[목차](#이동할-헤드-제목)`
```
(example)
## 📌목차
[__1. 프로젝트 개요__](#-프로젝트-개요)
   - [프로젝트 설명](#-기간)
   - [진행과정](#-팀-구성-및-사용기술)
   
   ## 📌 프로젝트 개요
   #### 📅 기간
   #### 👥 팀 구성 및 사용기술
```
example : [readme][1]
[1]: https://github.com/mindyhere/Team-Projects/tree/master/CafeManagement "Team-Projects/CafeManagement/"

\*cf. 주의할 점
 - 띄어쓰기는 -(하이픈)으로 연결한다.
 - 영어는 소문자로 작성한다.
 - 괄호()나 이모지(::)는 제외하고 작성한다.

---
<br/>
<br/>

## 9. 이미지
1. 이미지 삽입
```
![설명](파일경로)	
![설명](파일경로 "이미지 설명")
![example](https://images.immediate.co.uk/production/volatile/sites/10/2018/02/fe6d2c05-5ad9-4613-bc68-30e09d14634c-449be0c.jpg?quality=90&resize=940,627 "buttercup")
```    
![example](https://images.immediate.co.uk/production/volatile/sites/10/2018/02/fe6d2c05-5ad9-4613-bc68-30e09d14634c-449be0c.jpg?quality=90&resize=940,627 "buttercup")
    
2. 이미지 링크 : 마크다운 이미지코드를 링크코드(`[]()`) 묶어주면 된다.   
```
[![설명](이미지링크)](URL "링크 설명")
[![example](https://upload.wikimedia.org/wikipedia/commons/thumb/4/48/Markdown-mark.svg/175px-Markdown-mark.svg.png)](https://ko.wikipedia.org/wiki/%EB%A7%88%ED%81%AC%EB%8B%A4%EC%9A%B4 "Go wiki-markdown")
``` 
[![example](https://upload.wikimedia.org/wikipedia/commons/thumb/4/48/Markdown-mark.svg/175px-Markdown-mark.svg.png)](https://ko.wikipedia.org/wiki/%EB%A7%88%ED%81%AC%EB%8B%A4%EC%9A%B4 "Go wiki-markdown")

(\*cf. 이미지 크기, 정렬 등 세부적인 조절은 마크다운 문법으로는 해결이 어렵다. 따라서 html의 `<img>` 태그를 활용해야 한다.)

---
<br/>
<br/>

## 10. 기타 유용한 기능 추가정리

__1. 내용접기/펼치기__
```
<details><summary>CLICK</summary>
  hello :)
</details> 
```
<details><summary>CLICK</summary>
  hello :)
</details>

<br/>

__2. 체크박스/Task List__
```
- [x] This is Task Lists1
- [ ] This is Task Lists2
- [ ] This is Task Lists3
```
▼ 출력 결과 확인하기
- [x] This is Task Lists1
- [ ] This is Task Lists2
- [ ] This is Task Lists3

<br/>

__3. 이모지 단축키__
- __window10__ : 윈도우 키 + 마침표(.)
- __mac__ : Command + Control + 스페이스 바
