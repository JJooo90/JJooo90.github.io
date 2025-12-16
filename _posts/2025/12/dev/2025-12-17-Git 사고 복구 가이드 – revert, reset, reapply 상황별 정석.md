---
title: "Git 사고 복구 가이드 – revert, reset, reapply 상황별 정석"
categories: [devops, git]
tags: [git, branch-strategy, revert, reset, incident]
layout: post
------------

## 문서 목적

이 문서는 Git 사용 중 발생하는 대표적인 사고 상황에서 **개인 저장소와 팀 저장소를 구분**하여, 안전하고 재현 가능한 복구 절차를 정리한다. 특히 `revert`, `reset`, `reapply`, PR 충돌로 인해 브랜치 비교가 불가능해지는 상황을 중점적으로 다룬다.

---

## 기본 전제

* `main`: 배포(운영) 브랜치
* `dev`: 통합 개발 브랜치
* `feature/*`: 기능 단위 작업 브랜치
* PR 기반 협업을 기본 원칙으로 한다

---

## 사고 유형 분류

### 유형 1. 잘못된 PR이 dev 또는 main에 병합됨

* 기능이 의도와 다르게 반영됨
* 이후 revert PR이 들어감
* 다시 원래 기능을 살리고 싶어짐

### 유형 2. revert 이후 PR 비교가 되지 않는 상태

* GitHub PR 화면에서 "There isn’t anything to compare" 발생
* 로컬에서는 `git diff`가 존재함
* 히스토리 상으로는 이미 처리된 커밋으로 인식됨

---

## 개인 저장소 기준 복구 전략

> 개인 블로그, 개인 프로젝트 등 협업자가 없는 경우

### 선택지 A. 기준 브랜치 강제 동기화 (가장 단순)

```bash
git checkout main
git reset --hard feature/정상브랜치
git push origin main --force
```

* 히스토리를 단순화할 수 있음
* 빠른 복구 가능
* **팀 저장소에서는 절대 사용 금지**

### 선택지 B. dev 기준으로 복구 후 main 반영

```bash
git checkout dev
git reset --hard feature/정상브랜치
git push origin dev --force
```

이후 `dev → main` PR 생성

---

## 팀 저장소 기준 복구 전략 (정석)

> force push, reset 사용 불가

### 핵심 원칙

* 히스토리는 절대 수정하지 않는다
* 잘못된 변경은 **되돌린 것을 다시 되돌린다 (Revert of Revert)**

### 표준 흐름

```
A: 기능 PR 병합
B: A를 revert
C: B를 다시 revert (정상 복구)
```

---

## 팀 기준 실제 절차

### 1. dev 기준 복구 브랜치 생성

```bash
git checkout dev
git pull origin dev
git checkout -b feature/restore-issue
```

### 2. revert 커밋 식별

```bash
git log --oneline
```

예시:

```text
c3d4e5f revert: issue-11 category structure
```

### 3. revert의 revert 수행

```bash
git revert c3d4e5f
```

* 충돌 발생 시 해결 후 commit
* 새로운 복구 커밋 생성됨

### 4. PR 생성

* base: `dev`
* compare: `feature/restore-issue`

PR 설명에 반드시 명시:

> This PR reverts a previous revert to restore the intended behavior.

---

## dev → main 반영 규칙 (팀)

* 항상 PR 사용
* Merge commit 사용
* Squash / Rebase 금지 (사고 복구 맥락에서는)

---

## 절대 금지 목록 (팀 기준)

* `git reset --hard` + force push
* main / dev 직접 수정
* PR 없이 병합
* cherry-pick으로 히스토리 재작성

---

## 사고 예방을 위한 운영 규칙

1. main / dev 보호 브랜치 활성화
2. PR 승인 최소 1인 이상
3. revert PR은 리뷰 필수
4. reapply는 revert of revert 방식만 허용
5. force push 권한은 관리자 한정

---

## 핵심 요약

* 개인 저장소: 빠른 복구를 위해 reset/force 가능
* 팀 저장소: 히스토리 보존이 최우선
* 팀에서는 **revert를 다시 revert하는 방식만이 정답**

---

## 실제 경험 기반 사례 (Issue #19)

### 사고 타임라인

1. feature/issue-11 브랜치에서 카테고리 및 시리즈 구조 정리 작업 진행
2. 실수로 dev가 아닌 main 브랜치에 PR 병합
3. 잘못된 병합을 인지하고 revert PR 수행
4. 이후 동일 변경 사항을 다시 적용하려 했으나 PR 비교 불가 현상 발생
    * GitHub PR 화면에서 "There isn’t anything to compare" 메시지 표시
    * 로컬에서는 `git diff` 결과가 존재

---

### 당시 확인한 상태

* `git log origin/main..origin/dev` 결과 없음
* 브랜치 간 파일 내용은 상이해 보이나, 커밋 히스토리 기준으로는 동일하게 인식
* cherry-pick, 신규 PR 생성 모두 실패

---

### 원인 판단

* GitHub PR 비교는 파일 스냅샷이 아니라 커밋 DAG 기준
* revert 이후 동일 커밋 SHA가 히스토리에 남아 Git이 이미 처리된 변경으로 판단
* 개인 저장소에서는 reset 기반 복구 가능하나, 팀 기준에서는 허용되지 않음

---

### 실제 적용한 해결 전략

#### 개인 저장소 기준

* 정상 상태를 보유한 브랜치를 기준으로 main 브랜치를 강제 동기화
* 사고 복구 이후 dev → main 흐름 정상화

#### 팀 기준으로 적용했어야 할 정석

* revert 커밋을 다시 revert 하는 Revert of Revert 전략
* 히스토리 보존 및 PR 기록 유지

---

### 이번 사고에서 얻은 교훈

1. PR 비교 불가 문제는 Git 히스토리 문제일 가능성이 높다
2. 개인 저장소와 팀 저장소의 복구 전략은 명확히 구분해야 한다
3. revert 이후 재적용은 reset이 아닌 revert of revert로 접근해야 한다
4. 사고 대응 과정을 Issue와 문서로 남기는 것이 재발 방지에 가장 효과적이다

---

## 마무리

Git 사고는 실수가 아니라 **복잡한 워크플로우를 실제로 사용하고 있다는 증거**다. 중요한 것은 사고 이후의 대응 방식이며, 본 사례와 절차를 따르면 히스토리를 보존하면서도 안전하게 복구할 수 있다.
