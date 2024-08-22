---
title: "#취준일기_240822"
excerpt: "블로그 이사 중입니다. (드디어 내게도 깃허브 블로그가..!!)"

categories:
  - Blog
tags:
  - [GitHub, blog, memo]

permalink: /blog/diary-moving-blog/

toc: true
toc_sticky: true

date: 2024-08-22
last_modified_at: 2024-08-22
---

>🌱 블로그 이사 중입니다... 
>- [x] GitHub Page 만들기 및 환경설정(Ruby, Jekyll 설치)
>- [x] Jekyll 테마 적용 및 기본 커스텀 설정 적용 
>- [ ] 이전 포스팅 옮기기 & 사용방법 익히기 👉🏻 진행중!
>- [ ] 커스텀 요소 추가 적용(구글 애드센스, 댓글, 태그 기능 + ...기타등등 🧐)

### 1. 굳이굳이 옮기는 이유?

깃허브랑 조금 더 가까워져 보려는 노력의 일환이랄까. 구글링하다보면 좀 멋있다- 싶은 페이지들이 모두 깃허브 블로그여서 혹 했던 것도 있다.

__벨로그__ 는 별도 카테고리 설정 없이 태그로만 포스팅을 분류하는게 사용하다보니 좀 불편하더라. 포스팅이 많지 않은데도 뭔가 아카이빙이 잘 되지 않을 것 같다는 생각이 들었고, 옮기려면  지금 옮겨야겠다는 생각이 들었다.<br/>
__티스토리__ 는 네이버 블로그와 비교했을 때 딱히 차이점을 모르겠고, 넘쳐나는 구글 광고 배너까지 더해져 도저히 예쁘게 보이지를 않더라.🥲<br/>
__노션__ 으로 그냥 정리할까 싶어서 레퍼런스를 찾아보는데, 어쩐지 더 복잡해보여서 해당 선택지도 빠르게 접었다.

__깃허브 페이지__ 는 정말 하나부터 열까지 다~ 만들어야한다는.. 어마무시한 장벽이 있다. 뭐든지 환경설정만 잘 세팅해도 절반은 가는데, 버전 문제였던 건지 맨 처음 눈도장 찍어뒀던 Jekyll 테마 적용단계에서 자꾸 에러가 나서 리파지토리 생성-삭제를 몇번이나 반복해야했다.😰<br/>
그래도 구글링하면서 뚝딱뚝딱 해보는 과정 자체가 재미있기도 하고! 이제 어느정도 틀을 잡고 나니 쫌- 뿌듯한 기분

요약하자면... 
- 다양한 테마 & 커스터마이징으로 예쁘게 만들 수 있고
- Syntax Highlighting을 지원하고
- HTML과 마크다운 문법을 모두 지원하기 때문에

틈틈이 기존 포스팅도 옮기고, 이것저것 찾아보며 가꿔볼 예정이다.

---

<br/>

### 2. minimal-mistakes ⁉️ SIMPLE IS THE BEST 👌🏻

내가 적용한 __minimal-mistakes__ 은 __choiiis__ 님이 커스터미이징한 버전을 포크해 온 것이다.<br/>
해당 테마가 원래도 심플하니 예쁘긴한데, 이걸 정말 !!깔끔!! 그 자체로 커스터마이징 해놓으셨더라. 완전 취향저격. 

현재는 choiiis 버전에서 메인컬러 지정하고, 여기에 키워드 검색툴을 활성화까지만 해놓은 상태다.

자세한 내용은 
[minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#site-search  "github page로 이동합니다") 공식문서를 참조할 것!


```yml
//_config.yml 파일에서 search부분 수정

search: true # true, false (default)
search_full_content: true # true, false (default)
search_provider: # google
// lunr 의 search 기능 활성화
lunr:
  search_within_pages : true # true, false (default)
```

최종적으로 내가 추가하고 싶은 기능으로는 댓글, 태그 기능, 구글 애드센스 3가지가 남았다. 

오늘은 이만하고 앞으로 차근차근 적용해야지💪🏻

