---
title: "[devops] 초심자를 위한 GitHub Actions: dev에서는 확인만, main에서만 배포하기"
categories: [blog, devops]
tags: [github-actions, ci, github-pages, chirpy]
series: "GitHub Pages DevOps 설정기"
layout: post
toc: true
---

{% assign series_posts = site.posts | where: "series", page.series | sort: "date" %}

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


## 이 글의 목적

GitHub Pages 블로그를 운영하다 보면 이런 고민이 생긴다.

- dev 브랜치에서 작업한 코드가 **문제없는지 먼저 확인하고**, 확인이 끝난 뒤에만 **main 브랜치로 배포**하고 싶다

이 글은 **GitHub Actions를 처음 접하는 사람도 이해할 수 있도록**, dev와 main 브랜치를 나눠 사용하는 **가장 안전한 CI 구조**를 정리한 글이다.

---

## 왜 dev와 main을 나눠야 할까?

GitHub Pages는 보통 다음 구조로 운영된다.

- `dev` : 작업·실험·수정용 브랜치
- `main` : 실제 사용자에게 보이는 배포 브랜치

만약 dev에서 작업하다가 `빌드 오류`, `설정 실수`, `글 작성 실수`가 생겼는데 바로 배포된다면, **사이트가 깨진 상태로 공개**될 수 있다.

그래서 목표는 다음과 같다.

> dev에서는 “확인만”, main에서만 “배포”.

---

## GitHub Actions에서 해야 할 일 요약

초심자 기준으로 정리하면, 할 일은 딱 2가지다.

1. dev 브랜치에서도 **Action은 실행되게 한다**
2. 하지만 **배포는 main 브랜치에서만 되게 막는다**

---

## 최종 deploy 설정 코드

아래 설정이 **초심자에게 가장 안전한 구성**이다.

    deploy:
      if: github.ref == 'refs/heads/main'
      environment:
        name: github-pages
        url: ${{ steps.deployment.outputs.page_url }}
      runs-on: ubuntu-latest
      needs: build
      steps:
        - name: Deploy to GitHub Pages
          id: deployment
          uses: actions/deploy-pages@v4

이 코드는 `.github/workflows/pages-deploy.yml` 파일 안에 들어간다.

---

## 이 코드가 의미하는 것 (한 줄씩 설명)

- `deploy:`  
  - GitHub Pages에 배포하는 작업(job)

- `if: github.ref == 'refs/heads/main'`  
  - **main 브랜치일 때만 deploy 작업을 실행**
  - dev 브랜치에서는 deploy 작업 자체가 생기지 않는다

- `environment: github-pages`  
  - GitHub Pages 배포용 환경
  - GitHub가 자동으로 보호하는 영역

- `needs: build`  
  - build 작업이 성공해야 deploy가 가능

- `uses: actions/deploy-pages@v4`  
  - 실제로 GitHub Pages에 업로드하는 공식 Action

---

## 브랜치별 실제 동작 모습

| 브랜치 | build | deploy | 결과 |
|------|------|--------|------|
| dev  | 실행됨 | 실행 안 됨 | 배포 없음 |
| main | 실행됨 | 실행됨 | 사이트 반영 |

> 핵심은 이것!
> - dev에서는 **에러 확인용 Action만 실행**되고, main에서만 **실제 사이트가 바뀐다**.

---

## 초심자가 헷갈리기 쉬운 포인트

- dev에서 deploy가 안 보인다  
  → 정상이다

- dev에서 Pages 주소가 안 생긴다  
  → 정상이다

- deploy job이 skipped 또는 아예 없다  
  → 정상이다

이 구조에서는  
**dev에서 배포가 되면 오히려 잘못된 상태**다.

---

## 실제 작업 흐름 예시

1. dev 브랜치에서 글 작성 또는 설정 수정
2. dev에 push
3. GitHub Actions에서 build 성공 확인
4. 이상 없으면 main으로 merge
5. main에서 자동으로 Pages 배포

이 흐름을 만들기 위한 설정이 바로 위의 deploy 조건이다.

---

## 결론

> GitHub Actions에서 deploy 작업을 **job 레벨에서 main으로 제한**하면, 초심자도 안전하게 dev 검증 / main 배포 구조를 만들 수 있다.

이 구조를 한 번 만들어 두면, 이후에는 **실수로 사이트를 깨뜨릴 걱정 없이** dev 브랜치에서 편하게 작업할 수 있다.
