---
title: "GitHub/fork한 저장소 작업에 대한 커밋 반영하기"
excerpt: "잔디심는 재미로 만든 블로그인데 왜 커밋이 보이지 않을까"

categories:
  - Studylog
tags:
  - [GitHub, Git]

permalink: /studylog/github-forked-repo-mirroring/

toc: true
toc_sticky: true

date: 2024-08-29
last_modified_at: 2024-08-29
---

>🌱 __fork는 또 처음이라...__ <br/>
>
> 깃허브로 블로그를 옮기는 험난한 시행착오를 거쳐 드디어! 쫌 쓸만해졌군- 싶었는데 내 커밋이 누락되고 있었다는 걸 뒤늦게 알아차렸다🤨 <br/>
> 커밋과 푸시를 한 후 웹에서 정상적으로 페이지가 노출되는 것 까지 다 확인이 되는데 잔디만 보이지 않는 이유가 뭘까-싶어 찾아보니, 테마를 적용하는 과정에서 해당 repository를 fork해왔기 때문이었다.
> 생각해보니 타인의 저장소를 포크해온 것 자체가 처음이었던 것. 조금 번거롭기는 해도 5분 정도면 해결할 수 있는 방법이 있었다.

<br/>

✏️ 내 커밋이 프로필에 반영되지 않았던 이유?
---------------------

__Fork한 저장소__ 는 __"원래 주인이 따로 있는 저장소"__ 라는 사실을 기억하자. 저장소를 단순히 복제해오는 clone과는 다르다! </br>
때문에 내가 해당 저장소에서 어떤 작업을 하고 푸시를 해도 원래 주인에게 __Pull Request를 보내서 merge하지 않는 이상, 내 commit은 contribution으로 <u>*인정되지 않는다.*__</u> <br>

> *cf. contribution을 추가하기 위한 조건 3가지
> - commit한 계정이 GitHub계정과 같아야 한다.
> - commit이 fork한 repository가 아니어야 한다.
> - commit이 default 브랜치(메인/마스터 브랜치)에서 일어나야 한다.

<br/>

✏️ 해결방법?
---------------------


>- [ ] ~~그냥 이대로 쓴다...는 해결방법이라 할 수 없겠지~~
>- [ ] 새 리파지토리를 만들어서 옮긴다 → 여태까지의 commit이 남지 않기 때문에 반쪽짜리 해결이라 할 수 있겠다.
>- [ ] chatbot-virtual-assistant 로 fork된 리파지토리 분리 요청하기 → ticket을 제출하면 일정시간 소요 후 처리된 것을 확인할 수 있다고 한다. 자세한 내용은 [<u>해당 포스팅 참조</u>](https://jonghoonpark.com/2023/10/02/fork%ED%95%9C-github-%EC%A0%80%EC%9E%A5%EC%86%8C-%EB%B6%84%EB%A6%AC%ED%95%98%EA%B8%B0)
>- [x] bare clone 후 push mirror 로 진행

<br>

1. GitHub에 새로운 저장소 생성 : 최종적으로 옮겨 갈 __new-repo__ 를 생성한다.
2. 터미널을 열어 로컬에 기존 저장소(fork해 온 저장소. 잔디심기가 반영이 안되는 old-repo)를 __bare clone__ 한다.
   <br>→ ```git clone --bare {fork한-old-repo}.git```
   <br>(* __bare clone?__ Git에서 작업 디렉터리 없이 저장소 데이터만 포함하는 클론을 의미) 
3. new-repo로 __mirror__ push<br>→ ```git push --mirror {new-repo}.git```
   <br/>이 단계까지 정상적으로 실행이 되었다면 내 기존 commit이 반영된 것을 확인할 수 있다.
4. 기존 리파지토리(old-repo)와 bare clone으로 인해 생성된 로컬 저장소를 삭제해준다.
