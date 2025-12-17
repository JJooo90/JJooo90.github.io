---
title: "[blog] SCSS @import 사용시 에러"
categories: [blog, 삽질기록]
tags: [blog, troble, import, use, scss, 에러, sass, spoiler, chirpy]
toc: false
comments: true
---
해당 포스트는 scss을 자세히 알아야 해결이 가능할 것 같다. 
따라서, frontend를 공부하면서 수정 보완해야 하는 것을 느꼈다.
우선, 현재 맞딱들인 jekyll에서 scss를 로드할 때, custom.css가 main.css에 적용이 안되는 부분에 대해 gpt와 블로그를 통해 조사한 내용을 임시로 적었다.
다만, 내게는 해당 방법이 성공적으로 해결되지 못했기에, scss를 공부하고 다시 작성하는 것으로 남겼다.

<details>
<summary><strong>1) jekyll에서의 scss 적용 과정 정리</strong></summary>

<div markdown="1">

### 🎯 목표

custom.scss → Jekyll 빌드 → style.css 출력 → 페이지에 로드되는 상태 만들기

이를 위해 다음 3가지를 정확하게 구성해야 한다:

✅ 1단계: Chirpy 테마의 CSS override 규칙 이해

Chirpy는 기본 테마 CSS를 아래 경로에서 가져온다:

assets/css/jekyll-theme-chirpy.css   ← 최종 출력되는 CSS


즉, 우리가 override해야 하는 파일명은 "style.css"가 아니라
jekyll-theme-chirpy.scss 이다.

이 부분이 헷갈렸고, 그래서 style.scss 방식이 먹히지 않았던 것.

🔥 2단계: Chirpy가 인식하는 정확한 override 파일 만들기
반드시 아래 경로와 파일명을 사용해야 한다:
assets/css/jekyll-theme-chirpy.scss


절대 다른 이름 안 된다.

📌 파일 내용은 이렇게 작성
---
---

/* import Chirpy base */
@import "main";

/* import your custom style */
@import "custom";


여기서 "custom"은 custom.scss가 다음 위치에 있을 때만 인식된다:

assets/css/custom.scss

🔥 3단계: custom.scss 내용 다시 확인

경로:

assets/css/custom.scss


내용(테스트용):

/* TEST */
body {
  background: red !important;
}


이제 이게 적용되어야 한다.

🔥 4단계: Jekyll 캐시 삭제 + 재빌드

반드시 다음 명령 실행:

bundle exec jekyll clean
bundle exec jekyll serve


이걸 안 하면 SCSS 캐시가 남아서 아무리 파일을 바꿔도 적용 안 됨.

🔥 5단계: 브라우저 강력 새로고침
Ctrl + Shift + R

🧪 6단계: 실제 CSS가 생성되고 로드되는지 확인

브라우저 개발자도구에서 확인:

Sources → /assets/css/ 에 아래 파일이 있어야 함:
jekyll-theme-chirpy.css


그리고 Elements 탭 → <head> 내부에 아래가 있어야 함:

<link rel="stylesheet" href="/assets/css/jekyll-theme-chirpy.css">


이때 새로 생성된 CSS 안에 custom.scss의 내용이 포함되어 있어야 함.


</div>
</details>

---

<details>

<summary><strong>2) scss에 대해서 자세히 알고 수정이 필요함</strong></summary>

<div markdown="1">

### 🎯 목표

</div>
</details>

