---
layout: post
title:  "GitHub Projects 구축 및 운영 가이드"
date:   2026-04-13 01:00:00 +0900
---

# 📋 GitHub Projects 구축 및 운영 가이드

> **소스코드 · 이슈 · 일정을 하나로 — GitHub 네이티브 프로젝트 관리**  
> TA/개발자를 위한 GitHub Projects 초기 구축부터 실전 운영까지 완전 매뉴얼

---

## 📋 목차

1. [GitHub Projects란 무엇인가](#1-github-projects란-무엇인가)
2. [사전 요구사항](#2-사전-요구사항)
3. [Project 생성](#3-project-생성)
4. [뷰(View) 구성 — 칸반 · 테이블 · 로드맵](#4-뷰view-구성--칸반--테이블--로드맵)
5. [커스텀 필드 설정](#5-커스텀-필드-설정)
6. [저장소(Repository) 연결 및 이슈 연동](#6-저장소repository-연결-및-이슈-연동)
7. [이슈(Issue) 생성 및 관리](#7-이슈issue-생성-및-관리)
8. [마일스톤(Milestone) 설정](#8-마일스톤milestone-설정)
9. [워크플로우 자동화 (Automation)](#9-워크플로우-자동화-automation)
10. [GitHub Actions 연동 — Mattermost 알림](#10-github-actions-연동--mattermost-알림)
11. [팀 협업 실전 운영 가이드](#11-팀-협업-실전-운영-가이드)
12. [운영 치트시트](#12-운영-치트시트)
13. [문제 해결 (Troubleshooting)](#13-문제-해결-troubleshooting)

---

## 1. GitHub Projects란 무엇인가

GitHub Projects는 **GitHub 저장소에 네이티브로 통합된 프로젝트 관리 도구**입니다.  
이슈(Issue), PR(Pull Request), 커밋이 코드와 직접 연결되어 별도 도구 없이 개발 흐름 전체를 관리합니다.

### 개발 인프라 내 역할

```
┌─────────────────────────────────────────────────────────┐
│              개발 관리 인프라 전체 구성                      │
├──────────────┬──────────────────────────────────────────┤
│ 영역          │ 도구                                      │
├──────────────┼──────────────────────────────────────────┤
│ 소스 관리     │ GitHub (버전 관리 / PR 코드 리뷰)            │
│ 팀원 소통     │ Mattermost [Self-Hosted]                  │
│ 문서/공유     │ Wiki.js [Self-Hosted]                     │
│ 일정/태스크   │ ✅ GitHub Projects ← 본 문서               │
│ CI/CD        │ GitHub Actions + Self-hosted Runner        │
│ 상태 알림     │ Mattermost Incoming Webhook               │
└──────────────┴──────────────────────────────────────────┘
```

### GitHub Projects (New) 핵심 특징

| 특징 | 설명 |
|------|------|
| 🔗 **네이티브 연동** | 이슈·PR·커밋과 직접 연결, 상태 자동 반영 |
| 📊 **다중 뷰** | 칸반(Board) · 테이블(Table) · 로드맵(Roadmap) 동시 지원 |
| ⚙️ **자동화** | 이슈 생성/종료 시 칸반 열 자동 이동 |
| 🏷️ **커스텀 필드** | 우선순위, 스프린트, 공수 등 자유롭게 추가 |
| 💰 **완전 무료** | Public·Private 저장소 모두 무제한 |
| 🌐 **설치 불필요** | 브라우저만 있으면 즉시 사용 |

### 핵심 개념 정리

```
Organization / User
    └── Project (프로젝트 보드)
            ├── View 1: Board (칸반)
            ├── View 2: Table (목록)
            └── View 3: Roadmap (일정)
                    ↕ 연동
            Repository (저장소)
                    ├── Issue (이슈/태스크)
                    ├── Pull Request (코드 리뷰)
                    └── Milestone (마일스톤)
```

---

## 2. 사전 요구사항

### 필요 조건

| 항목 | 내용 |
|------|------|
| GitHub 계정 | 개인 또는 Organization 계정 |
| 저장소(Repository) | 연동할 저장소 1개 이상 |
| 권한 | 저장소 `Write` 이상 권한 |
| 브라우저 | Chrome, Edge, Firefox 최신 버전 권장 |

### 용어 정리

| 용어 | 설명 |
|------|------|
| **Issue** | 개발 태스크, 버그, 기능 요청의 단위 |
| **Label** | 이슈 분류 태그 (bug, feature, docs 등) |
| **Milestone** | 버전 또는 스프린트 단위 목표 |
| **Assignee** | 담당자 지정 |
| **Project** | 이슈들을 모아 시각화하는 보드 |
| **Field** | Project에 추가하는 커스텀 속성 |

---

## 3. Project 생성

### 3-1. Organization 레벨 Project 생성 (권장)

> 여러 저장소를 하나의 프로젝트로 관리할 수 있어 팀 단위 운영에 적합합니다.

1. GitHub 상단 메뉴 → **Your organizations** → 해당 Organization 클릭
2. **Projects** 탭 클릭
3. **New project** 버튼 클릭
4. 템플릿 선택 화면에서 원하는 형태 선택

### 3-2. 개인 계정 레벨 Project 생성

1. GitHub 우측 상단 프로필 클릭 → **Your projects**
2. **New project** 버튼 클릭

### 3-3. 템플릿 선택

| 템플릿 | 설명 | 권장 상황 |
|--------|------|-----------|
| **Board** | 칸반 형태로 시작 | 애자일 스프린트 운영 |
| **Table** | 스프레드시트 형태 | 백로그 목록 관리 |
| **Roadmap** | 타임라인 형태 | 릴리즈 일정 관리 |
| **Blank** | 완전 빈 상태 | 직접 커스터마이즈 |

> ✅ **권장**: `Board` 템플릿으로 시작 후 Table · Roadmap 뷰를 추가하는 방식

### 3-4. Project 기본 정보 설정

```
Project name : [팀명 또는 제품명] Development Board
              예) MyProject Dev Board
Description  : 소프트웨어 개발 태스크 및 일정 관리
Visibility   : Private (팀 내부용) 또는 Public
```

---

## 4. 뷰(View) 구성 — 칸반 · 테이블 · 로드맵

하나의 Project에 여러 뷰를 추가해 상황에 맞는 시각화를 제공합니다.

### 4-1. Board 뷰 (칸반) 구성

칸반 열(Column)을 개발 흐름에 맞게 구성합니다.

**뷰 추가 방법:**
```
Project 상단 [+ New view] 클릭 → Board 선택
```

**추천 칸반 열 구성:**

| 열 이름 | 의미 | 이슈 상태 |
|---------|------|-----------|
| `📥 Backlog` | 대기 중인 태스크 전체 목록 | Open |
| `🔍 To Do` | 이번 스프린트에서 할 작업 | Open |
| `⚡ In Progress` | 현재 진행 중인 작업 | Open |
| `👀 In Review` | PR 리뷰 대기 중 | Open |
| `✅ Done` | 완료된 작업 | Closed |

**열 추가/수정 방법:**
```
Board 뷰 → 열 상단 [+] 또는 [···] 클릭 → 열 이름 입력
```

### 4-2. Table 뷰 (목록) 구성

백로그 전체를 스프레드시트처럼 관리합니다.

```
Project 상단 [+ New view] 클릭 → Table 선택
```

**유용한 그룹화 설정:**
```
Table 뷰 → [Group by] → Milestone 또는 Status 선택
→ 마일스톤별 또는 상태별로 이슈 묶음 표시
```

### 4-3. Roadmap 뷰 (타임라인) 구성

릴리즈 일정과 마일스톤을 시각적으로 확인합니다.

```
Project 상단 [+ New view] 클릭 → Roadmap 선택
```

**날짜 필드 설정:**
```
Roadmap 뷰 → [Date fields] → Start date / Target date 지정
```

> Roadmap 뷰 활용을 위해서는 **커스텀 날짜 필드** 설정이 필요합니다. (5장 참고)

---

## 5. 커스텀 필드 설정

Project에 사용자 정의 필드를 추가해 팀 맞춤형 정보를 관리합니다.

### 5-1. 필드 추가 방법

```
Project → Table 뷰 → 우측 끝 [+] 버튼 클릭 → New field
```

### 5-2. 추천 커스텀 필드 구성

| 필드명 | 타입 | 옵션 값 예시 | 용도 |
|--------|------|-------------|------|
| `Status` | Single select | Backlog / To Do / In Progress / In Review / Done | 진행 상태 |
| `Priority` | Single select | 🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low | 우선순위 |
| `Sprint` | Iteration | Sprint 1 / Sprint 2 … | 스프린트 구분 |
| `Start Date` | Date | - | 작업 시작일 |
| `Due Date` | Date | - | 작업 마감일 |
| `Story Points` | Number | 1 / 2 / 3 / 5 / 8 | 작업 공수 |
| `Type` | Single select | Feature / Bug / Docs / Infra | 이슈 유형 |

### 5-3. Iteration (스프린트) 필드 설정

```
New field → Iteration 선택
→ 스프린트 기간 설정 (예: 2주)
→ [Add iteration] 으로 Sprint 1, 2, 3 … 추가
```

---

## 6. 저장소(Repository) 연결 및 이슈 연동

### 6-1. 저장소 연결

```
Project → 우측 상단 [···] 메뉴 → Settings
→ Linked repositories → [Link a repository] 클릭
→ 연동할 저장소 검색 후 선택
```

> 여러 저장소를 하나의 Project에 연결할 수 있습니다.  
> 연결 후 해당 저장소의 이슈를 Project에 추가할 수 있습니다.

### 6-2. 기존 이슈를 Project에 추가

**방법 1 — Project에서 직접 추가:**
```
Project → [+ Add item] 클릭 → # 입력 후 이슈 검색 → 선택
```

**방법 2 — 이슈 페이지에서 추가:**
```
이슈 상세 페이지 → 우측 사이드바 → [Projects] → 해당 Project 선택
```

**방법 3 — 저장소 이슈 목록에서 일괄 추가:**
```
저장소 → Issues 탭 → 이슈 선택 (체크박스) → [Projects] → Project 선택
```

---

## 7. 이슈(Issue) 생성 및 관리

### 7-1. 이슈 생성

**저장소에서 이슈 생성:**
```
Repository → Issues 탭 → [New issue] 버튼
```

**Project에서 Draft 이슈 생성:**
```
Project Board → 해당 열 하단 [+ Add item] → 제목 입력 → Enter
(Draft 상태 → 나중에 저장소 이슈로 변환 가능)
```

### 7-2. 이슈 작성 템플릿

팀 전체가 일관된 형식으로 이슈를 작성하도록 템플릿을 설정합니다.

**템플릿 파일 생성 위치:**
```
저장소 루트/.github/ISSUE_TEMPLATE/
```

**기능 개발 템플릿 예시** (`.github/ISSUE_TEMPLATE/feature.md`):

```markdown
---
name: ✨ 기능 개발
about: 새로운 기능 개발 요청
title: "[FEAT] "
labels: feature
assignees: ''
---

## 📌 개요
<!-- 어떤 기능을 개발하는지 한 줄로 요약하세요 -->

## 🎯 목적 및 배경
<!-- 왜 이 기능이 필요한지 설명하세요 -->

## ✅ 완료 조건 (Acceptance Criteria)
- [ ] 조건 1
- [ ] 조건 2
- [ ] 조건 3

## 📋 작업 목록 (Sub Tasks)
- [ ] 세부 작업 1
- [ ] 세부 작업 2

## 🔗 관련 문서
<!-- Wiki.js 링크, 설계 문서 등 -->

## 📎 참고 사항
```

**버그 리포트 템플릿 예시** (`.github/ISSUE_TEMPLATE/bug.md`):

```markdown
---
name: 🐛 버그 리포트
about: 버그 신고 및 수정 요청
title: "[BUG] "
labels: bug
assignees: ''
---

## 🐛 버그 설명
<!-- 어떤 문제가 발생했는지 설명하세요 -->

## 🔁 재현 방법
1. ...
2. ...
3. ...

## ✅ 기대 동작
<!-- 정상적으로 어떻게 동작해야 하는지 -->

## ❌ 실제 동작
<!-- 실제로 어떻게 동작하는지 -->

## 🖥️ 환경 정보
- OS: 
- 브라우저/런타임: 
- 버전: 

## 📸 스크린샷
```

### 7-3. 라벨(Label) 설정

저장소별로 라벨을 표준화합니다.

```
Repository → Issues → Labels → [New label]
```

**추천 라벨 구성:**

| 라벨명 | 색상 | 의미 |
|--------|------|------|
| `feature` | `#0075ca` 파란색 | 새 기능 개발 |
| `bug` | `#d73a4a` 빨간색 | 버그 수정 |
| `docs` | `#0052cc` 남색 | 문서 작업 |
| `infra` | `#e4e669` 노란색 | 인프라/DevOps |
| `refactor` | `#cfd3d7` 회색 | 코드 개선 |
| `hotfix` | `#b60205` 진빨강 | 긴급 수정 |
| `blocked` | `#d93f0b` 주황색 | 다른 작업에 의해 차단됨 |

### 7-4. 이슈 상세 설정

이슈 생성/수정 시 우측 사이드바에서 아래 항목을 설정합니다.

```
Assignees   → 담당자 지정 (복수 지정 가능)
Labels      → 유형 태그 선택
Projects    → 연결할 Project 보드 선택
Milestone   → 속하는 마일스톤(스프린트/버전) 선택
```

---

## 8. 마일스톤(Milestone) 설정

마일스톤은 **스프린트** 또는 **릴리즈 버전** 단위로 이슈를 묶어 진척도를 추적합니다.

### 8-1. 마일스톤 생성

```
Repository → Issues → Milestones → [New milestone]
```

| 항목 | 입력 예시 |
|------|-----------|
| **Title** | `Sprint 1` 또는 `v1.0.0` |
| **Due date** | `2026-04-30` |
| **Description** | `인증 기능 및 기본 CRUD 완성` |

### 8-2. 추천 마일스톤 구성 예시

```
v0.1.0 - 환경 구축 및 기본 설정 (Due: 2026-04-20)
v0.2.0 - 핵심 기능 개발 1차     (Due: 2026-05-10)
v0.3.0 - 핵심 기능 개발 2차     (Due: 2026-05-30)
v1.0.0 - 최초 릴리즈             (Due: 2026-06-15)
```

### 8-3. 마일스톤 진척도 확인

```
Repository → Issues → Milestones
→ 마일스톤별 완료 이슈 수 / 전체 이슈 수 (%) 자동 표시
```

---

## 9. 워크플로우 자동화 (Automation)

이슈 상태 변경에 따라 칸반 카드를 자동으로 이동시킵니다.

### 9-1. 기본 자동화 설정

```
Project → 우측 상단 [···] → Workflows
```

**기본 제공 자동화 목록:**

| 트리거 이벤트 | 자동 동작 | 설정 권장 |
|--------------|-----------|-----------|
| Item added to project | Status → `Backlog` 설정 | ✅ 활성화 |
| Item reopened | Status → `To Do` 설정 | ✅ 활성화 |
| Item closed | Status → `Done` 설정 | ✅ 활성화 |
| Pull request merged | Status → `Done` 설정 | ✅ 활성화 |
| Code change requested | Status → `In Review` 설정 | ✅ 활성화 |
| Code review approved | Status → `Done` 설정 | ✅ 활성화 |

### 9-2. 자동화 활성화 방법

```
Workflows 페이지 → 각 워크플로우 클릭 → [Edit] → 조건 설정 → [Save and turn on]
```

### 9-3. GitHub Actions를 활용한 고급 자동화

이슈에 `hotfix` 라벨 추가 시 자동으로 담당자를 지정하는 예시:

```yaml
# .github/workflows/auto-assign.yml
name: Auto Assign on Hotfix Label

on:
  issues:
    types: [labeled]

jobs:
  assign:
    if: github.event.label.name == 'hotfix'
    runs-on: ubuntu-latest
    steps:
      - name: Assign to lead developer
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.addAssignees({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              assignees: ['lead-developer-username']
            })
```

---

## 10. GitHub Actions 연동 — Mattermost 알림

이슈 생성, PR 오픈, 빌드 결과를 Mattermost 채널로 자동 전송합니다.

### 10-1. Mattermost Webhook Secret 등록

```
GitHub 저장소 → Settings → Secrets and variables → Actions
→ [New repository secret]
```

| Secret 이름 | 값 |
|------------|-----|
| `MATTERMOST_WEBHOOK_URL` | `http://192.168.93.138:8065/hooks/xxxxxxxx` |

### 10-2. 이슈 생성 시 Mattermost 알림

```yaml
# .github/workflows/issue-notify.yml
name: Issue Notification to Mattermost

on:
  issues:
    types: [opened, closed, reopened]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Send to Mattermost
        run: |
          STATUS="${{ github.event.action }}"
          if [ "$STATUS" = "opened" ]; then EMOJI="🆕"; fi
          if [ "$STATUS" = "closed" ]; then EMOJI="✅"; fi
          if [ "$STATUS" = "reopened" ]; then EMOJI="🔄"; fi

          curl -i -X POST \
            -H 'Content-Type: application/json' \
            -d "{
              \"text\": \"$EMOJI **이슈 ${STATUS}**: [${{ github.event.issue.title }}](${{ github.event.issue.html_url }})\n담당자: ${{ github.event.issue.assignee.login || '미지정' }}\n저장소: \`${{ github.repository }}\`\"
            }" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}
```

### 10-3. PR 오픈 및 머지 시 알림

```yaml
# .github/workflows/pr-notify.yml
name: PR Notification to Mattermost

on:
  pull_request:
    types: [opened, closed, review_requested]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Send PR notification
        run: |
          ACTION="${{ github.event.action }}"
          if [ "$ACTION" = "opened" ]; then
            MSG="🔀 **PR 오픈**: [${{ github.event.pull_request.title }}](${{ github.event.pull_request.html_url }})"
          elif [ "$ACTION" = "closed" ] && [ "${{ github.event.pull_request.merged }}" = "true" ]; then
            MSG="🎉 **PR 머지 완료**: [${{ github.event.pull_request.title }}](${{ github.event.pull_request.html_url }})"
          elif [ "$ACTION" = "review_requested" ]; then
            MSG="👀 **코드 리뷰 요청**: [${{ github.event.pull_request.title }}](${{ github.event.pull_request.html_url }})"
          fi

          curl -i -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"$MSG\n저장소: \`${{ github.repository }}\`\"}" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}
```

### 10-4. CI/CD 빌드 결과 알림

```yaml
# .github/workflows/build-notify.yml
name: Build Result Notification

on:
  push:
    branches: [main, develop]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Tests
        run: |
          echo "테스트 실행..."
          # npm test 또는 실제 빌드 명령어

      - name: Notify Success
        if: success()
        run: |
          curl -i -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"✅ **빌드 성공** \`${{ github.repository }}\` → \`${{ github.ref_name }}\`\n커밋: ${{ github.event.head_commit.message }}\"}" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}

      - name: Notify Failure
        if: failure()
        run: |
          curl -i -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"❌ **빌드 실패** \`${{ github.repository }}\` → \`${{ github.ref_name }}\`\n즉시 확인이 필요합니다!\"}" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}
```

---

## 11. 팀 협업 실전 운영 가이드

### 11-1. 브랜치 전략 (Git Flow 간소화)

```
main          ← 운영 배포 브랜치 (직접 push 금지)
  └── develop ← 개발 통합 브랜치
        ├── feature/이슈번호-기능명   예) feature/42-user-auth
        ├── bugfix/이슈번호-버그명    예) bugfix/55-login-error
        └── hotfix/이슈번호-긴급명   예) hotfix/60-payment-fix
```

**브랜치 생성 규칙:**
```bash
# 이슈 #42 기능 개발 브랜치 생성
git checkout develop
git pull origin develop
git checkout -b feature/42-user-auth
```

### 11-2. 커밋 메시지 컨벤션

```
[타입] #이슈번호 작업 내용 요약

타입 목록:
feat     : 새 기능 추가
fix      : 버그 수정
docs     : 문서 수정
style    : 코드 포맷팅
refactor : 코드 리팩터링
test     : 테스트 코드
infra    : 인프라/빌드 설정
```

**예시:**
```bash
git commit -m "feat: #42 사용자 로그인 JWT 인증 구현"
git commit -m "fix: #55 로그인 페이지 무한 리다이렉트 수정"
git commit -m "docs: #60 API 명세서 업데이트"
```

> 커밋 메시지에 `#이슈번호`를 포함하면 GitHub에서 해당 이슈와 커밋이 **자동으로 연결**됩니다.

### 11-3. PR (Pull Request) 규칙

**PR 제목 규칙:**
```
[타입] #이슈번호 작업 내용 요약
예) [feat] #42 사용자 로그인 JWT 인증 구현
```

**PR 본문 템플릿** (`.github/PULL_REQUEST_TEMPLATE.md`):

```markdown
## 🔗 관련 이슈
Closes #이슈번호

## 📝 변경 내용
<!-- 어떤 변경을 했는지 요약 -->

## ✅ 테스트 확인
- [ ] 로컬 테스트 완료
- [ ] 기존 테스트 통과 확인

## 📸 스크린샷 (UI 변경 시)

## 💬 리뷰어에게
<!-- 리뷰 시 참고할 사항 -->
```

> `Closes #42` 형식으로 작성하면 PR 머지 시 **이슈가 자동으로 Close**되고  
> Project 칸반에서도 **Done으로 자동 이동**됩니다.

### 11-4. 스프린트 운영 사이클

```
월요일 (스프린트 시작)
  └── 스프린트 계획 회의
       ├── Backlog 이슈 검토 및 우선순위 조정
       ├── 이슈에 Sprint 필드 → 현재 스프린트 지정
       └── 담당자 배정

매일 (데일리 스탠드업)
  └── Board 뷰 기준으로 진행 상황 공유
       ├── In Progress 항목 현황
       ├── 블로커(Blocked) 이슈 공유
       └── 오늘 할 작업 공유

금요일 (스프린트 종료)
  └── 스프린트 리뷰
       ├── Milestone 진척도 확인
       ├── Done 항목 검토
       └── 다음 스프린트 Backlog 정리
```

### 11-5. 인사이트 및 진척도 확인

```
Project → Insights 탭
```

| 차트 | 내용 |
|------|------|
| **Burn up** | 완료 이슈 누적 추이 |
| **Burn down** | 남은 이슈 감소 추이 |
| **Velocity** | 스프린트별 완료 공수 |
| **Status** | 현재 상태별 이슈 분포 |

---

## 12. 운영 치트시트

### 자주 쓰는 GitHub 단축키

| 단축키 | 동작 |
|--------|------|
| `C` | 새 이슈 생성 |
| `?` | 전체 단축키 목록 보기 |
| `G` + `I` | 이슈 목록으로 이동 |
| `G` + `P` | PR 목록으로 이동 |
| `/` | 저장소 검색 포커스 |

### 이슈 검색 필터 문법

```
# 내가 담당한 열린 이슈
is:issue is:open assignee:@me

# 특정 라벨의 이슈
is:issue label:bug is:open

# 특정 마일스톤 이슈
is:issue milestone:"Sprint 1"

# 리뷰 요청받은 PR
is:pr is:open review-requested:@me

# 특정 기간 내 생성된 이슈
is:issue created:>2026-04-01
```

### GitHub CLI (선택 사항)

터미널에서 이슈와 PR을 관리하는 도구입니다.

```bash
# 설치
sudo dnf install gh -y

# 로그인
gh auth login

# 이슈 생성
gh issue create --title "[FEAT] 로그인 기능" --label "feature" --assignee "@me"

# 이슈 목록 확인
gh issue list --assignee "@me"

# PR 생성
gh pr create --title "[feat] #42 로그인 구현" --base develop

# PR 목록
gh pr list
```

---

## 13. 문제 해결 (Troubleshooting)

### ❌ Project에 이슈가 추가되지 않을 때

**원인:** 저장소가 Project에 연결되지 않은 상태

```
Project → [···] → Settings → Linked repositories
→ 이슈가 있는 저장소를 연결
```

---

### ❌ 자동화(Workflow)가 동작하지 않을 때

```
Project → [···] → Workflows
→ 해당 워크플로우가 활성화(ON) 상태인지 확인
→ [Edit] → [Save and turn on] 다시 클릭
```

---

### ❌ GitHub Actions Mattermost 알림이 전송되지 않을 때

```bash
# 1. Secret 등록 여부 확인
저장소 → Settings → Secrets and variables → Actions
→ MATTERMOST_WEBHOOK_URL 존재 여부 확인

# 2. Webhook URL 직접 테스트 (서버에서)
curl -i -X POST \
  -H 'Content-Type: application/json' \
  -d '{"text": "테스트 메시지"}' \
  http://192.168.93.138:8065/hooks/xxxxxxxx

# 3. Actions 로그 확인
저장소 → Actions 탭 → 실패한 워크플로우 클릭 → 로그 확인
```

---

### ❌ PR 머지 후 이슈가 자동 Close되지 않을 때

**원인:** PR 본문에 연결 키워드 누락

```markdown
# 올바른 연결 키워드 (대소문자 무관)
Closes #42
Fixes #42
Resolves #42

# 주의: 아래는 자동 Close 안 됨
Related to #42   ← 단순 참조만 됨
See #42          ← 단순 참조만 됨
```

---

### ❌ Roadmap 뷰에서 날짜가 표시되지 않을 때

**원인:** Start Date / Due Date 커스텀 필드 미설정

```
Project → Table 뷰 → [+] → New field → Date 타입으로 추가
→ Roadmap 뷰 → [Date fields] → 생성한 날짜 필드 선택
```

---

## 📚 참고 자료

| 자료 | URL |
|------|-----|
| GitHub Projects 공식 문서 | https://docs.github.com/en/issues/planning-and-tracking-with-projects |
| GitHub Actions 공식 문서 | https://docs.github.com/en/actions |
| GitHub CLI 공식 문서 | https://cli.github.com/manual |
| 이슈 자동 Close 키워드 | https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue |

---

## 🗺️ 다음 단계

```
✅ Step 1: Mattermost Self-Hosted 설치       (완료)
✅ Step 2: Wiki.js Self-Hosted 설치           (완료)
✅ Step 3: GitHub Projects 구축              ← 현재 문서
⬜ Step 4: GitHub Actions Self-hosted Runner 등록
⬜ Step 5: GitHub Actions ↔ Mattermost Webhook 전체 통합
```

---

<div align="center">

**본 문서는 소프트웨어 개발 프로젝트의 무료 Self-Hosted 인프라 구축을 목적으로 작성되었습니다.**  
`GitHub Projects` · `GitHub Actions` · `Mattermost Webhook` · `Git Flow`

*작성일: 2026-04-13*  
*작성자: Kim Jong-in (Technical Architect)*

</div>
