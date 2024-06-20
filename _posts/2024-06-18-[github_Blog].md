---
title:  "[github_Blog] Minimal-Mistakes 테마의 디렉터리 구조"
excerpt: "github Blog  Minimal-Mistakes 테마의 디렉터리 구조."

categories:
  - Blog
tags:
  - [Blog, github, git]

toc: true
toc_sticky: true
 
date: 2024-06-18
last_modified_at: 2024-06-18
---


## 📁 minimal-mistakes 
기본적으로 Jekyll 디렉터리 구조를 뼈대로 하고 있지만, 테마들마다 디렉터리 구조가 조금씩 다른거 같다. 내가 쓰고 있는 minimal-mistakes의 디렉터리 구조를 살펴보고자 한다
```
minimal-mistakes
├── 📁_data                 # data files for customizing the theme
├── 📁_includes
├── 📁_layouts
├── 📁_sass                 # SCSS partials
├── 📁assets
├── 📝_config.yml           # site configuration
├── 📝Gemfile               # gem file dependencies
├── 📝index.html            # paginated home page showing recent posts
└── 📝package.json          # NPM build scripts
```

## 📁_data 폴더 
테마를 커스터마이징하기 위한 데이터 파일들이 모여있는 폴더.  
사이트에 사용할 데이터를 적절한 포맷으로 정리하여 보관하는 디렉터리다.  

이 디렉터리에 `.yml` `.yaml`, `json`, `csv`, `tsv`  같은 파일들을 둔다면 이 파일들을 자동으로 읽어들여 `site.data`로 사용할 수 있다. 

예를 들어 _data 디렉터리에 `members.yml`라는 파일이 있다면, `site.data.members`로 입력하여 그 파일을 사용할 수 있다.

### 구조
```
├── 📁_data                      # data files for customizing the theme
|  ├── 📘navigation.yml          # main navigation links
|  └── 📘ui-text.yml             # text used throughout the theme's UI
```

### 📘navigation.yml
상단 메뉴바 네비게이션으로 초기 상태는 `Quick-Start Guide`의 URL로 quick-start-guide 문서 페이지로 이동한다.
``` 
# 
# <navigation.yml 초기 내용>
# 
# main links
main:
  - title: "Quick-Start Guide"
    url: https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/
  # - title: "About"
  #   url: https://mmistakes.github.io/minimal-mistakes/about/
  # - title: "Sample Posts"
  #   url: /year-archive/
  # - title: "Sample Collections"
  #   url: /collection-archive/
  # - title: "Sitemap"
  #   url: /sitemap/
```
