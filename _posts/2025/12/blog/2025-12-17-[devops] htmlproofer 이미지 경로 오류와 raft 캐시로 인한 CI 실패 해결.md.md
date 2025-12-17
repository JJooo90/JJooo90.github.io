---
title: "[devops] htmlproofer 이미지 경로 오류와 raft 캐시로 인한 CI 실패 해결"
categories: [devops, ci]
tags: [github-pages, jekyll, chirpy, htmlproofer, ci, troubleshooting]
layout: post
series: "DevOps"
series_order: 20
toc: true
comments: true
---

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

## 문제 상황

GitHub Actions에서 `htmlproofer` 실행 시 이미지 경로 오류로 CI가 반복 실패하였다.

오류 메시지는 다음과 같은 형태였다.

    internal image /assets/img/posts/blog/2025-12-03/img_01.png does not exist

로컬 기준으로는 `_posts` 내 이미지 경로가 모두 정상으로 보였기 때문에, 처음에는 원인을 파악하기 어려운 상태였다.

---

## 초기 진단

`_posts` 디렉토리를 기준으로 잘못된 이미지 경로가 남아 있는지 확인하였다.

    grep -R "/assets/img/posts/blog/" _posts

결과는 없음.

즉, **포스트 원본에는 문제가 없었다.**

---

## 실제 원인 확인

전체 프로젝트 기준으로 검색 범위를 확장하였다.

    grep -R "/assets/img/posts/blog/" .

그 결과, 다음 경로에서 문제가 발견되었다.

- raft/posts/**/index.html
- _site/raft/posts/**/index.html

이 시점에서 문제의 정체가 명확해졌다.

---

## 원인 정리

- `raft/` 디렉토리는 Chirpy 테마에서 생성하는 카테고리/목록용 HTML 캐시
- Jekyll은 `raft/` 디렉토리를 자동으로 정리하지 않음
- 이미지 정책 변경 이전에 생성된 HTML 산출물이 그대로 남아 있었음
- htmlproofer는 `_site` 기준으로 모든 HTML 파일을 검사함
- 실제 소스와 무관한 과거 HTML 때문에 CI가 실패함

즉,

    이미지 경로는 정상이나,
    오래된 raft 산출물이 CI를 망가뜨리고 있었다.

---

## 해결 방법

### 1. raft 및 _site 강제 제거 (핵심 해결책)

로컬 및 CI 환경 모두에서 다음 정리 단계가 필요하다.

    rm -rf raft
    rm -rf _site

    bundle exec jekyll clean
    bundle exec jekyll build

---

### 2. CI 환경에서의 재발 방지 조치

GitHub Actions build 단계에 아래 Step을 추가하였다.

    - name: Clean generated artifacts
      run: |
        rm -rf raft
        rm -rf _site

이를 통해 과거 캐시 HTML이 CI에 포함되는 문제를 근본적으로 차단하였다. (아래 사진 참고)

![pages-deploy]({{ "/assets/img/posts/2025/12/trouble/2025-12-17/img_01.png" | relative_url }})

---

## 교훈

이번 이슈의 핵심은 다음 한 문장으로 요약된다.

> Jekyll 기반 블로그에서 보이지 않는 캐시 산출물은 CI 실패의 주요 원인이 될 수 있다.

특히 구조 개편, 이미지 정책 변경과 같은 작업 이후에는 산출물 정리 여부가 품질을 좌우한다.

---

## 재발 방지 체크리스트

이미지 경로 정책 변경 시 반드시 아래를 수행한다.

- `_posts` 기준 이미지 경로 전체 점검
- raft 및 _site 디렉토리 제거
- jekyll clean + build 실행
- htmlproofer 실행 결과 확인 (github Actions 내용)
- CI에 정리 Step 포함 여부 확인

---

## 마무리

이 문제는 단순한 이미지 경로 오류가 아니라, **Jekyll + Chirpy + CI 구조를 이해해야만 발견할 수 있는 이슈**였다.

이번 경험을 통해, 블로그 운영에서도 DevOps 관점의 사고가 얼마나 중요한지 다시 한 번 확인했다.


<details>
<summary><strong>(추가) htmlproofer ignore 전략을 사용하지 않은 이유</strong></summary>

<div markdown="1">

## (추가) htmlproofer ignore 전략을 사용하지 않은 이유

문제를 해결하는 과정에서 `htmlproofer`의 검사 대상에서 `raft` 경로를 제외하는 방법도 고려하였다.

실제로 htmlproofer는 다음과 같은 옵션을 제공한다.

    --ignore-files "/_site/raft/"

그러나 이번 이슈에서는 해당 전략을 **최종 해결책으로 채택하지 않았다.**

---

### ignore 전략의 한계

htmlproofer ignore는 결과적으로 다음과 같은 성격을 가진다.

- 오류를 해결하는 것이 아니라 **오류를 보지 않도록 하는 방식**
- 실제로 존재하는 잘못된 산출물을 그대로 둔 채 검사만 우회
- 다른 종류의 문제(링크, 스크립트, 마크업 오류)가 있어도 함께 가려질 수 있음

즉, CI 안정성은 확보할 수 있지만 **산출물의 신뢰성은 보장되지 않는다.**

---

### 이번 이슈의 본질

이번 문제의 핵심 원인은 다음과 같았다.

- 이미지 경로 자체가 잘못된 것이 아님
- `_posts` 및 실제 소스는 이미 정상 상태
- 과거에 생성된 `raft` HTML 산출물이 정리되지 않은 채 남아 있었음

> 따라서, 검사 대상을 제외하는 것이 아니라, 잘못된 산출물이 만들어지지 않도록 하는 것이 더 근본적인 해결책이라고 판단하였다.

---

### 선택한 해결 방향

이번 이슈에서는 다음 기준을 우선하였다.

- CI는 항상 **현재 소스 기준의 결과만 검사해야 한다**
- 과거 산출물에 의존하는 상태를 허용하지 않는다
- 구조 변경 이후에도 동일한 문제가 재발하지 않아야 한다

그 결과,

- `raft/` 및 `_site/` 디렉토리를 빌드 전에 강제 제거
- 항상 깨끗한 상태에서 Jekyll 빌드를 수행
- htmlproofer는 기본 검사 정책을 그대로 유지

하는 방향을 선택하였다.

---

### ignore 전략에 대한 최종 정리

htmlproofer ignore 전략은 다음과 같은 경우에만 고려할 수 있다.

- 외부 서비스 장애 등으로 통제 불가능한 링크가 포함된 경우
- 임시 대응이 필요한 상황에서 CI를 우선 통과시켜야 할 경우

그러나 이번 사례처럼 **내부 산출물 관리 문제**인 경우에는 ignore 전략보다 **생성 과정 자체를 바로잡는 방식이 더 적합**하다고 판단하였다.

이 결정은 단기적인 CI 통과보다 장기적인 블로그 운영 안정성을 우선한 선택이었다.

</div>