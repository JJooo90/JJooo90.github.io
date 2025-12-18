---
  title: "[blog] Chirpy 테마에서 Related Posts가 설정으로 꺼지지 않는 이유와 오버라이드로 해결하기"
  categories: [blog, 제작기]
  tags: [github-pages, jekyll, chirpy, related-posts, override, troubleshooting]
  layout: post
  series: "블로그 제작기"
  series_order: 96
  toc: true
  comments: true
  related_posts: false
---

  ## 문제 상황

 

  ---

  

  ---

  ## 실제 동작 확인

  DevTools를 통해 DOM 구조를 확인한 결과,  
  하단에 노출되는 영역은 다음과 같은 구조를 가지고 있었다.

    <aside id="related-posts" aria-label="related-label">
      <h3 id="related-label">관련된 글</h3>
      ...
    </aside>

  구조와 주석을 통해 이는 **Chirpy 기본 Related Posts 컴포넌트**임을 확인했다.

  ---

  ## 원인 분석

  Chirpy 테마의 구조를 기준으로 보면 다음과 같다.

  - `related_posts: false` 값은 Front Matter 및 `_config.yml`에 정의 가능
  - 그러나 Chirpy 레이아웃 / include 내부에서
    해당 값을 조건으로 사용하는 코드가 존재하지 않음
  - 즉, 설정 값은 존재하지만 **읽히지 않는 상태**

  결과적으로,

  - 설정이 잘못된 것이 아님
  - Jekyll이 설정을 무시한 것도 아님
  - **테마 설계상 Related Posts는 항상 렌더링되도록 구현됨**

  ---

  ## 왜 오버라이드가 필요한가

  Jekyll 테마는 다음 우선순위를 따른다.

  - 로컬 프로젝트에 존재하는 파일
  - gem(테마) 내부 파일

  따라서,

  - 테마를 clone 해서 사용하더라도
  - 관련 레이아웃 또는 include 파일이 로컬에 없으면
  - 해당 로직을 수정하거나 제어할 수 없다.

  Chirpy의 Related Posts는 레이아웃에서 조건 없이 include 되므로,  
  이를 제어하려면 **조건문을 직접 추가하는 오버라이드가 필수**다.

  ---

  ## 해결 전략

  목표는 다음과 같다.

  - Chirpy 기본 UX 유지
  - 포스트 단위로 Related Posts 제어
  - 테마 업데이트 시 충돌 최소화

  이를 위해 **`_includes/related-posts.html` 오버라이드 방식**을 선택했다.

  ---

  ## 오버라이드 작업 절차

  ### 1. Chirpy gem 경로 확인

    bundle show jekyll-theme-chirpy

  해당 경로가 Chirpy 원본 코드 위치다.

  ---

  ### 2. 원본 include 파일 확인

  gem 경로 내부에서 다음 파일을 찾는다.

    _includes/related-posts.html

  이 파일이 관련 글을 렌더링하는 실제 원인이다.

  ---

  ### 3. 로컬 프로젝트에 include 오버라이드

  프로젝트 루트에 다음 구조를 생성한다.

    _includes/
      related-posts.html

  `_includes` 폴더가 없다면 새로 생성한다.

  ---

  ### 4. 원본 파일 그대로 복사

  gem 내부의 `related-posts.html` 내용을  
  **수정 없이 그대로** 로컬 파일에 복사한다.

  이 단계의 목적은  
  include 오버라이드가 정상적으로 적용되는지 확인하는 것이다.

  ---

  ### 5. 조건문 추가

  로컬의 `_includes/related-posts.html` 파일 전체를  
  아래 조건문으로 감싼다.

    {% if page.related_posts != false %}
      (기존 related-posts.html 전체 내용)
    {% endif %}

  이 조건을 통해:

  - 기본 상태 → Related Posts 표시
  - `related_posts: false` → Related Posts 미표시

  가 가능해진다.

  ---

  ## 포스트 단위 제어 방법

  Related Posts를 숨기고 싶은 포스트의 Front Matter에 다음 옵션을 추가한다.

    related_posts: false

  해당 설정이 있는 포스트에 한해  
  Related Posts 영역이 렌더링되지 않는다.

  ---

  ## 결과

  - 기본 포스트에서는 관련 글이 정상적으로 노출됨
  - `related_posts: false`가 있는 포스트에서는 관련 글 미노출
  - Chirpy 기본 UX를 해치지 않으면서 제어 가능
  - 설정과 실제 동작이 일치하는 상태 확보

  ---

  ## 정리

  - `related_posts: false`가 동작하지 않는 것은 설정 오류가 아님
  - Chirpy 테마가 해당 값을 사용하지 않는 구조이기 때문
  - 이를 해결하기 위해서는 오버라이드가 필수
  - include 오버라이드는 Jekyll의 정상적인 확장 방식

  이번 작업을 통해  
  Chirpy 테마를 “설정으로만 사용하는 단계”에서  
  **“구조를 이해하고 제어하는 단계”로 넘어가게 되었다.
