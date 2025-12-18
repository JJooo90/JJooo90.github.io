---
title: "블로그 제작기 9탄 - Chirpy 테마 Related Posts를 오버라이드해서 제어하기"
categories: [blog, 제작기]
tags: [github-pages, jekyll, chirpy, related-posts, override, troubleshooting]
layout: post
series: "블로그 제작기"
series_order: 81
toc: true
comments: true
related_posts: false
---

## 1. 개요 - 이 글을 쓰게 된 이유

Chirpy 테마에서 포스트 하단에 노출되는 **관련된 글(Related Posts)** 은 `_config.yml`이나 Front Matter 설정만으로는 제어되지 않았다.

설정이 적용되지 않는 것처럼 보였고, 결국 **테마 구조를 직접 이해하고 오버라이드해야 하는 상황**에 도달했다.
이 글은 해당 과정을 정리한 기록이다.

---

## 2. 문제 상황 요약

Chirpy 테마에서 포스트 하단에 노출되는 **관련된 글(Related Posts)** 을 끄기 위해 `_config.yml` 및 포스트 Front Matter에 다음 설정을 추가하였다.

  related_posts: false

하지만 설정과 무관하게 관련 글 영역은 계속 노출되었다.
그래서, 다음 3가지 항목을 다시 점검하였다.
- 설정 오류 아님 (config & Front Matter 점검)
- 캐시 문제 아님 (브라우저 캐시 삭제 후 재시작 & 시크릿모드 & 다른 브라우저)
- 배포 문제 아님 (clean → build → serve)

### 2.1 처음에 가졌던 가정

  - Chirpy 테마를 clone 해서 사용 중이기 때문에, Front Matter 옵션으로 포스트 단위 제어가 가능할 것이라 예상
  - 따라서, 설정이 반영되지 않는 원인은 캐시 또는 빌드 문제일 것이라 판단

### 2.2 실제 동작 확인

DevTools를 통해 DOM 구조를 확인한 결과, 하단에 노출되는 영역은 다음과 같은 구조를 가지고 있었다.

    <aside id="related-posts" aria-label="related-label">
      <h3 id="related-label">관련된 글</h3>
      ...
    </aside>

구조와 주석을 통해 이는 **Chirpy 기본 Related Posts 컴포넌트**임을 확인했다.

---

### 2.3 원인 분석

Chirpy 테마의 구조를 기준으로 보면 다음과 같다.

- `related_posts: false` 값은 Front Matter 및 `_config.yml`에서 정의 가능
- 그러나 Chirpy 레이아웃 / include 내부에서 해당 값을 조건으로 사용하는 코드가 존재하지 않음
- 즉, 설정 값은 존재하지만 **읽히지 않는 상태**

결과적으로,

- 설정이 잘못된 것이 아님
- Jekyll이 설정을 무시한 것도 아님
- **테마 설계상 Related Posts는 항상 렌더링되도록 구현됨**

---

### 2.4 왜 설정만으로는 해결되지 않는가

Chirpy 테마의 기본 구조는 다음과 같다.

- `related-posts.html` include가 레이아웃에서 **조건 없이 호출됨**
- `page.related_posts` 값을 참조하지 않음

> 즉, 설정 값은 존재하지만, 테마 코드에서 **사용하지 않기 때문에** 아무 효과가 없는 상태였다.

이 구조에서는 **오버라이드 외에는 해결 방법이 없다.**

이를 다시 정리하면 다음과 같다.

    Jekyll 테마는 다음 우선순위를 따른다.

      1. 로컬 프로젝트에 존재하는 파일
      2. gem(테마) 내부 파일

      따라서,

      - 테마를 clone 해서 사용하더라도 관련 레이아웃 또는 include 파일이 로컬에 없으면 해당 로직을 수정하거나 제어할 수 없다.

> 즉, clone 여부와 무관하게, 로컬에 파일이 없으면 gem 코드가 사용된다
> 따라서, Chirpy의 Related Posts는 레이아웃에서 조건 없이 include 되므로, 이를 제어하려면 **조건문을 직접 추가하는 오버라이드가 필수**다.

> 최종적으로, **설정 문제처럼 보였던 이슈는 테마 코드가 설정을 읽지 않는 구조 때문임**
---

## 3. 해결 전략

목표는 다음과 같다.

- Chirpy 기본 UX 유지
- 포스트 단위로 Related Posts 제어
- 테마 업데이트 시 충돌 최소화

이를 위해 **`_includes/related-posts.html` 오버라이드 방식**을 진행했다.

---

### 3.1 오버라이드 작업 절차

#### 3.1.1. Chirpy gem 경로 확인

    bundle show jekyll-theme-chirpy

해당 경로가 Chirpy 원본 코드 위치다.

---

#### 3.1.2. 원본 include 파일 확인

gem 경로 내부에서 다음 파일을 찾는다.

    _includes/related-posts.html

이 파일이 관련 글을 렌더링하는 실제 원인이다.

---

#### 3.1.3. 로컬 프로젝트에 include 오버라이트(로컬 디렉토리 생성)

프로젝트 루트에 다음 구조를 만든다.

    _includes/
      related-posts.html

`_includes` 폴더가 없다면 새로 생성한다.

---

#### 3.1.4. 원본 파일 그대로 복사

gem 내부의 `related-posts.html` 내용을 **수정 없이 그대로** 로컬 파일에 복사한다.

이 단계의 목적은 **오버라이드가 정상 동작하는지 확인하는 것**이다. 

> 즉, 먼저 덮어쓰기가 정상 동작하는지 확인하기 위해 이 단계에서는 수정하지 않는다

---

#### 3.1.5. 오버라이드 정상 여부 확인

로컬 서버를 재시작한다.

    bundle exec jekyll clean
    bundle exec jekyll serve

이 상태에서 관련 글이 여전히 보인다면, include 오버라이드는 `정상적`으로 적용된 것이다.

---

#### 3.1.6. 조건문 추가

이제 로컬의 `_includes/related-posts.html` 파일을 수정한다.

파일 전체를 아래 조건문으로 감싼다.

    {% if page.related_posts != false %}
      (기존 related-posts.html 전체 내용)
    {% endif %}

이 조건으로 인해 다음 2가지 기능이 가능해진다.

- 기본 상태 → Related Posts 표시
- `related_posts: false` → Related Posts 미표시

---

### 3.2 포스트 단위 제어 방법

related_posts를 숨기고 싶은 포스트의 Front Matter에 아래 옵션을 추가한다.

    related_posts: false

이 설정이 있는 포스트에 한해서 Related Posts 영역이 렌더링되지 않는다.

---

## 4. 결과

- 기본 포스트에서는 관련 글이 정상적으로 노출됨
- `related_posts: false`가 있는 포스트에서는 관련 글 미노출
- Chirpy 기본 UX를 해치지 않으면서 제어 가능
- 설정과 실제 동작이 일치하는 상태 확보

---

## 5. 정리

- `related_posts: false`가 동작하지 않는 것은 설정 오류가 아님
- Chirpy 테마가 해당 값을 사용하지 않는 구조이기 때문임
- 이를 해결하기 위해서는 오버라이드가 필수임
- include 오버라이드는 Jekyll의 정상적인 확장 방식
