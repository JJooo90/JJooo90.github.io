---
title: "블로그 제작기 8탄 - GitHub 블로그 댓글(Giscus)을 기본으로 달고, 한 곳에서 관리하며 안 보이던 문제까지 정리"
categories: [blog, 제작기]
tags: [github-pages, comments, giscus, discussions, chirpy, troubleshooting]
layout: post
series: "블로그 제작기"
series_order: 80
toc: true
comments: true
related_posts: false
---

{% assign series_posts = site.posts | where: "series", page.series | sort: "series_order" %}

<div class="series-box">
<div class="series-title">바로가기</div>
<ul>
  {% for post in series_posts %}
    <li {% if post.url == page.url %}class="current"{% endif %}>
      <a href="{{ post.url }}">{{ forloop.index }}. {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
</div>

## 1. 이 글에서 다루는 내용

GitHub Pages 블로그를 운영하면서 다음 요구가 동시에 발생했다.

- 모든 게시글에 댓글을 기본으로 달고 싶다
- 댓글이 달리면 어디에 달렸는지 한눈에 알고 싶다
- 댓글을 한 곳에서 관리하고 싶다
- 서버 없이, GitHub Pages 환경을 유지하고 싶다

또한 실제 적용 과정에서,

- 댓글이 정상적으로 설정되었음에도 “댓글이 안 보인다”라고 오해하게 되는 상황도 발생했다

이 글은 **댓글 시스템 설계 결정**과 **적용 후 발생한 UX 문제의 원인 분석**을 단계별로 정리한 기록이다.

---

## 2. 환경적 제약 (GitHub 블로그에서 댓글이 어려운 이유)

GitHub Pages 기반 블로그는 구조적으로 다음 제약을 가진다.

- 서버 없음
- 로그인 시스템 없음
- 데이터베이스 직접 사용 불가

따라서 일반적인 블로그 댓글 시스템을 그대로 사용할 수 없고, 현실적인 해법은 **댓글 기능을 외부 시스템에 위임**하는 것이다.

---

## 3. 설계 선택
### 3-1. 선택한 구조

> **Giscus + GitHub Discussions**

이 조합은 다음과 같이 역할이 분리된다.

- 블로그 화면의 댓글 UI → Giscus
- 실제 댓글 데이터 저장 → GitHub Discussions
- 댓글 관리 및 알림 → GitHub (Github 저장소의 Discussions 탭)

> 즉, 블로그 댓글 = GitHub Discussions

### 3-2. 이 구조를 선택한 이유

- 추가 서버 불필요
- GitHub 계정 기반 인증
- 스팸 및 관리 기능을 GitHub에 위임
- 댓글 관리 UI를 따로 만들 필요 없음

블로그 운영 부담을 최소화하면서 **관리 가능한 댓글 시스템**을 만들 수 있다.

---

## 4. 운영 관점의 동작 방식 (How it works)
### 4-1. 댓글이 달리는 흐름

1. 사용자가 게시글에 댓글 작성
2. GitHub Discussions에 Discussion 자동 생성 또는 갱신
3. 관리자는 Discussions 탭에서 댓글 확인

### 4-2. 모든 게시글에 댓글을 기본으로 다는 방법

Chirpy 테마 기준으로 전역 설정은 다음과 같다. (_config.yml에서 작성)

    comments:
      provider: giscus

이 설정이 있으면:

- 모든 게시글에 댓글이 기본 활성화된다 (기본 값)
- 특정 글만 끄고 싶을 경우 Front Matter에 `comments: false`를 추가하면 된다.  
(※ 포스트 맨 위의 설정을 frontMatter이라 함)

### 4-3. 댓글 관리 페이지는 어디에 있는가?

댓글을 관리하는 페이지는 블로그 내부가 아니라 GitHub 저장소에 있다.

  GitHub 저장소 → Discussions 탭

이 한 페이지에서 다음을 모두 처리할 수 있다.

1. 댓글 목록 확인
2. 어떤 게시글에 달린 댓글인지 식별
3. 댓글 삭제, 수정, 잠금 등 관리 작업

---

## 5. 댓글 매핑 전략 (Identification)

Giscus는 댓글과 게시글을 연결하는 기준을 선택할 수 있다. (_config.yml에서 작성)

### 5-1. mapping: title

- 게시글 title 기준으로 댓글과 게시글을 1:1로 연결
- GitHub Discussions에서 작성한 제목으로 출력되어 가독성이 좋음 (한글이면 한글, 영어면 영어)
- 단점:
  - 제목이 변경되면 기존 댓글과 연결 끊김
  - 댓글이 삭제되는 것은 아님 (즉, 새로운 댓글이 생성 → 새 Discussion이 생성)

### 5-2. mapping: pathname

- 게시글 URL 경로 기준으로 연결
- 장점:
  - 제목 변경과 무관하게 댓글 유지 (제목이 바뀌어도 URL만 유지되면 댓글은 그대로 유지됨)
- 단점:
  - GitHub Discussions에서 제목이 URL 인코딩 형태로 보임

이 블로그에서는 **관리 가독성을 우선하여 `mapping: title`을 선택**했다.

---

## 6. 적용 후 발생한 오해 (What went wrong)
### 6-1. 댓글이 안 보였다고 느꼈던 문제 상황

Giscus 설정을 완료한 뒤, **댓글이 안 보였다**. 그래서, 다음 상태 확인했다.

- `_config.yml` 설정 완료
- 게시글에 `comments: true` 설정
- GitHub Actions 배포 성공
- DevTools에서 `giscus.app/client.js` 로드 확인

그럼에도 불구하고 **댓글이 안 보였다**.

처음에는 다음을 의심했다.

- URL 구조 문제인가?
- permalink 설정 문제인가?
- Giscus mapping 설정이 꼬인 건가?

결론적으로 **전부 아니었다**.

### 6-2. 실제 원인

댓글은 이미 정상적으로 렌더링되고 있었다.
다만 Chirpy 테마의 기본 레이아웃 구조상 댓글 영역이 화면 아래쪽으로 밀려 있었을 뿐이었다.

즉, 문제는 **Chirpy의 기본 레이아웃 순서**였다. 
Chirpy 기본 렌더링 순서는 다음과 같다.
1. 본문 내용
2. 관련 글(Related Posts)
3. 댓글(Comments, Giscus)

위 순서에서 관련 글 영역이 길어지는 경우,
- 댓글은 화면 하단 깊숙이 밀려 스크롤하지 않으면 보이지 않는 상태가 된다

> 결과적으로, 스크롤을 충분히 내리지 않으면 댓글이 없는 것처럼 보이게 된다.

### 6-3. (추가) 확인 과정에서 알게 된 사실

DevTools에서 다음을 확인했다.

- `giscus.app/client.js` 스크립트 정상 로드
- Console 메시지:
  - `[giscus] Discussion not found. A new discussion will be created if a comment/reaction is submitted.`

이는 오류가 아니라 **정상 동작 로그**다.

- 아직 Discussion이 없다는 의미
- 첫 댓글 작성 시 자동으로 Discussion 생성

즉, 댓글 기능 자체는 처음부터 정상 동작 중이었다.

---

### 6-4. 해결 방향 정리

이 문제를 해결하는 방법은 다음 선택지가 있다.

#### 6-4-1. 관련 글 영역 비활성화 (**내 경우에는 적용 안됨**)

가장 단순한 방법 (Front Matter에 설정 추가)

    related_posts: false

댓글이 본문 바로 아래에 노출되도록 하는 Front Matter 설정

#### 6-4-2. 관련 글을 접기 UI로 변경 (**시도 안함**)

관련 글은 유지하면서 댓글 접근성을 높이는 방식이다.

#### 6-4-3. 댓글 → 관련 글 순서로 변경 (적용한 내용 -> 다음 포스트 참고)

UX 관점에서 가장 자연스럽지만 `_layouts/post.html` 오버라이드가 필요하다.

---

## 7. 정리 및 결론

이번 이슈의 본질은 다음 한 줄로 정리된다.

> **댓글이 안 보였던 게 아니라, 관련 글 영역 아래에 가려져 있었던 것이다.**

그리고 동시에,

> GitHub Pages 환경에서도 Giscus + GitHub Discussions 조합을 사용하면, 한 곳에서 안정적으로 관리할 수 있다.

기능 문제로 보였던 이슈의 상당수는 실제로는 **레이아웃과 UX 문제**였다.
