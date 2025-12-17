---
title: "[devops] 빌드 산출물(tex-svg-full.js)이 Git에 포함된 사고와 복구 과정"
categories: [devops, trouble]
tags: [git, jekyll, chirpy, build, gitignore]
layout: post
series: "DevOps"
series_order: 30
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

  ## 문제 요약

  로컬에서 `jekyll build` 또는 `bundle exec jekyll build` 실행 시, `tex-svg-full.js` 파일이 자동 생성되며, 해당 파일이 Git 변경 사항으로 인식되어 다음 문제가 발생했다.

  - dev → main 브랜치 이동 불가
  - PR 생성 시 의도하지 않은 변경 파일 포함
  - CI 기준과 로컬 기준 불일치
  - 빌드만 했을 뿐인데 커밋 대상이 생기는 상태

  이 문제는 [#Issue #26](https://github.com/JJooo90/jjooo90.github.io/issues/26)로 등록하여 대응했다.
  

  ---

  ## 사고 발생 배경

  - Chirpy 테마 + MathJax 사용 환경
  - Jekyll 빌드 과정에서 MathJax 관련 JS 파일 자동 생성
  - 생성 파일이 Git 추적 대상 상태로 남아 있었음

  즉,

  > “빌드 산출물”과 “소스 코드”의 경계가 Git에서 명확히 분리되지 않은 상태였다.

  ---

  ## 문제의 핵심 원인

  ### 1. 자동 생성 파일을 Git이 추적하고 있었음

  - `tex-svg-full.js`는 사람이 수정하는 파일이 아님
  - 환경/빌드 조건에 따라 내용이 달라질 수 있음
  - Git에 포함될 경우 브랜치 이동 시마다 충돌 가능

  ### 2. `.gitignore` 정책 부재

  - `_site`는 ignore 되어 있었으나, 빌드 중 생성되는 JS 파일은 별도 ignore 처리되지 않음

  ---

  ## 해결 전략

  ### 선택한 방식

  - **자동 생성 파일을 Git 추적 대상에서 완전히 제외**
  - `.gitignore`에 명시적으로 경로 추가

  ### `.gitignore` 처리 방향

  - 원칙: “사람이 직접 관리하지 않는 파일은 Git에 남기지 않는다”
  - 결과: 빌드 실행 후에도 `git status`가 clean 상태 유지

  ---

  ## 해결 후 확인 사항

  - Jekyll build 재실행
  - 브랜치 이동(dev ↔ main) 정상
  - PR 생성 시 불필요한 파일 포함 없음
  - CI / Actions 환경과 로컬 환경 일치

  ---

  ## 왜 이 문제는 중요했는가

  이 이슈는 단순한 파일 하나의 문제가 아니라,

  - Git 신뢰성 저하
  - PR 리뷰 혼란
  - 팀 작업 시 불필요한 충돌 유발

  로 이어질 수 있는 **구조적인 사고 포인트**였다.

  ---

  ## 이번 이슈를 통해 정리한 원칙

  - 빌드 결과물은 절대 Git에 남기지 않는다
  - 브랜치 이동 시 변경 파일이 생긴다면 구조를 의심한다
  - “환경에 따라 달라지는 파일”은 무조건 ignore 대상이다
  - CI 기준과 로컬 기준은 반드시 동일해야 한다

  ---

  ## 관련 이슈 / PR

  - Issue:
    - [#26 빌드 산출물(tex-svg-full.js) Git 추적 문제](https://github.com/JJooo90/jjooo90.github.io/issues/26)

  - PR: Issue #26 – ignore generated tex-svg-full.js

  ---

  ## 마무리

  이 문제는 기능 개발보다도 **개발 환경을 신뢰할 수 있는 상태로 되돌리는 작업**이었다.

  작은 설정 하나가 브랜치 전략, PR 흐름, CI 안정성까지 영향을 줄 수 있다는 점에서 기록으로 남길 가치가 충분한 사례였다.
