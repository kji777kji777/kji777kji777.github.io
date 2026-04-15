---
layout: post
title:  "(폐쇄망)소프트웨어 개발 관리 아키텍처 구축 가이드"
date:   2026-04-13 12:00:00 +0900
---

# 🏗️ 소프트웨어 개발 관리 아키텍처 구축 가이드

> **Self-Hosted 완전 자립 — 폐쇄망 완전 무료 인프라 로드맵**  
> 슬랙·노션·Jira·GitHub 유료 없이 엔터프라이즈급 개발 관리 환경을 내 서버에서 완전히 구축합니다.

---

## 📋 목차

1. [왜 이 조합인가 — 설계 철학](#1-왜-이-조합인가--설계-철학)
2. [전체 아키텍처 한눈에 보기](#2-전체-아키텍처-한눈에-보기)
3. [핵심 인프라 구성 및 역할](#3-핵심-인프라-구성-및-역할)
4. [CI/CD 자동화 파이프라인](#4-cicd-자동화-파이프라인)
5. [구축 로드맵 — 단계별 설치 순서](#5-구축-로드맵--단계별-설치-순서)
6. [Step 1 — Mattermost 구축](#6-step-1--mattermost-구축)
7. [Step 2 — Wiki.js 구축](#7-step-2--wikijs-구축)
8. [Step 3 — Gitea 구축](#8-step-3--gitea-구축)
9. [Step 4 — Gitea Issues · 마일스톤 · 칸반 구축](#9-step-4--gitea-issues--마일스톤--칸반-구축)
10. [Step 5 — Gitea Actions Self-hosted Runner 구축](#10-step-5--gitea-actions-self-hosted-runner-구축)
11. [Step 6 — 전체 시스템 통합 (Webhook 연동)](#11-step-6--전체-시스템-통합-webhook-연동)
12. [아키텍처 핵심 가치](#12-아키텍처-핵심-가치)
13. [운영 관리 가이드](#13-운영-관리-가이드)
14. [전체 시스템 트러블슈팅](#14-전체-시스템-트러블슈팅)

---

## 1. 왜 이 조합인가 — 설계 철학

### 기존 상용 도구의 한계

| 상용 도구 | 문제점 |
|-----------|--------|
| **Slack** | 무료 플랜 90일 메시지 제한, 팀 규모 확장 시 고비용 |
| **Notion** | 무료 플랜 블록 제한, 외부 서버에 데이터 보관 |
| **Jira** | 10인 초과 시 유료 전환, 복잡한 설정 |
| **GitHub** | 인터넷 필수 · 폐쇄망 불가, CI/CD 월 2,000분 제한 |

### 우리의 대안 — 완전 무료 + 완전 데이터 주권

```
상용 도구 (유료 · 외부 서버)          무료 대안 (Self-Hosted · 내 서버)
─────────────────────────────    ──────────────────────────────────
Slack            →              Mattermost  [Self-Hosted]
Notion           →              Wiki.js     [Self-Hosted]
GitHub           →              Gitea       [Self-Hosted · 폐쇄망 가능]
Jira             →              Gitea Issues + Milestones [Self-Hosted]
GitHub Actions   →              Gitea Actions + Act Runner [Self-Hosted]
```

### 설계 원칙

```
1. 데이터는 내 서버에만  →  Mattermost · Wiki.js · Gitea 모두 Self-Hosted
2. 코드와 일정은 한 곳에  →  Gitea (소스 · 이슈 · 마일스톤)
3. 반복 작업은 자동화    →  Gitea Actions · Webhook
4. 운영 비용은 제로      →  모든 도구 무료 오픈소스
5. 인터넷 없이도 동작    →  폐쇄망 완전 자립 구조
```

---

## 2. 전체 아키텍처 한눈에 보기

```
┌─────────────────────────────────────────────────────────────────────┐
│                    개발자 / TA 워크스테이션                            │
│   코드 작성 → git push → Gitea 저장소 (내 서버)                       │
└────────────────────────┬────────────────────────────────────────────┘
                         │ push / PR / Issue 이벤트
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│              Oracle Linux 8.10 서버 (Self-Hosted · 완전 자립)          │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐   │
│  │  Gitea (Docker · 포트 3001 / SSH 2222)                         │   │
│  │  ┌──────────────┐  ┌──────────────────┐  ┌────────────────┐  │   │
│  │  │  Repository  │  │  Issues·Milestone │  │  Gitea Actions │  │   │
│  │  │  (소스코드)   │  │  (태스크·칸반)    │  │  (CI/CD 정의)  │  │   │
│  │  └──────────────┘  └──────────────────┘  └───────┬────────┘  │   │
│  └──────────────────────────────────────────────────┼───────────┘   │
│                                                     │ 트리거          │
│  ┌──────────────────────────────────────────────────▼───────────┐   │
│  │  Act Runner (Gitea Actions 실행 에이전트)                       │   │
│  │  → 빌드 · 테스트 · 배포 직접 수행 (시간 제한 없음)               │   │
│  └─────────────────────────────┬─────────────────────────────────┘   │
│                                │ 결과 알림 (Webhook)                  │
│  ┌─────────────────────────────▼─────────────────────────────────┐   │
│  │  Mattermost (Docker · 포트 8065)                               │   │
│  │  → 팀 소통 · 빌드 알림 · 이슈 알림 수신                         │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐   │
│  │  Wiki.js (Docker · 포트 3000)                                  │   │
│  │  → 기술 문서 · 설계 문서 → Gitea 저장소 자동 백업               │   │
│  └───────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐   │
│  │  PostgreSQL DB (각 서비스별 독립 인스턴스)                       │   │
│  │  mattermost DB (5432)  |  wikijs DB (5432)  |  gitea DB (5432)│   │
│  └───────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### 데이터 흐름 요약

```
개발자 코드 push (Gitea 내 서버로)
    → Gitea Actions 트리거
    → Act Runner 에서 빌드·테스트
    → 결과를 Mattermost 채널로 알림 전송
    → 이슈 Close → Gitea 마일스톤 진척도 자동 업데이트
    → 기술 문서 → Wiki.js → Gitea 저장소 자동 백업
    ※ 모든 데이터가 내 서버 안에서만 이동 (인터넷 불필요)
```

---

## 3. 핵심 인프라 구성 및 역할

### 전체 도구 스택

| 영역 | 도구 | 유형 | 핵심 역할 |
|------|------|------|-----------|
| **소스 관리** | Gitea | Self-Hosted | 폐쇄망 Git 서버. 버전 관리 및 PR 기반 코드 리뷰. GitHub 완전 대체 |
| **팀원 소통** | Mattermost | Self-Hosted | 내부 소통용 메신저. 데이터 주권 확보 및 메시지·파일 영구 보존 |
| **문서/공유** | Wiki.js | Self-Hosted | 프로젝트 기술 위키. Gitea 연동으로 문서를 코드처럼 백업·관리 |
| **일정/태스크** | Gitea Issues + Milestones | Self-Hosted | 이슈 기반 태스크 관리. 마일스톤으로 스프린트 진척도 추적 |
| **CI/CD 워크플로우** | Gitea Actions | Self-Hosted | 코드 변경 시 빌드·테스트·배포를 자동 트리거. GitHub Actions 문법 100% 호환 |
| **CI/CD 실행** | Act Runner | Self-Hosted | 리눅스 서버에서 직접 빌드 수행. 무료 시간 제한 없이 100% 활용 |
| **상태 알림** | Gitea Webhook + Mattermost | Self-Hosted | 빌드 성공/실패 및 이슈 이벤트를 Mattermost로 즉시 전송 |

### 서버 포트 구성 요약

| 서비스 | 포트 | 프로토콜 | 비고 |
|--------|------|----------|------|
| Mattermost | `8065` | HTTP | 팀 메신저 웹 UI |
| Wiki.js | `3000` | HTTP | 위키 웹 UI |
| Gitea 웹 UI | `3001` | HTTP | Git 저장소 웹 UI |
| Gitea SSH | `2222` | TCP | Git SSH 클론·푸시 |
| Mattermost DB | `5432` | TCP | 내부 전용 |
| Wiki.js DB | `5432` | TCP | 내부 전용 |
| Gitea DB | `5432` | TCP | 내부 전용 |

> ⚠️ Gitea 웹 포트를 `3001`로 설정하는 이유: Wiki.js가 `3000`을 이미 사용하고 있기 때문입니다.

---

## 4. CI/CD 자동화 파이프라인

### 파이프라인 전체 흐름

```
코드 작성 (개발자)
    │
    ▼
git push / PR 오픈 → Gitea 서버 (내 서버)
    │
    ▼
Gitea Actions 워크플로우 트리거 (.gitea/workflows/*.yml)
    │
    ├──▶ Act Runner 선택 (runs-on: self-hosted)
    │         │
    │         ├── 1. 코드 체크아웃 (actions/checkout)
    │         ├── 2. 의존성 설치 (npm install / pip install)
    │         ├── 3. 빌드 (npm run build / gradle build)
    │         ├── 4. 테스트 (npm test / pytest)
    │         └── 5. 배포 (docker compose up / scp / rsync)
    │
    └──▶ 결과 알림 → Mattermost Webhook
              ├── ✅ 성공: 브랜치명 · 커밋 메시지 전송
              └── ❌ 실패: 즉시 알림 · 로그 링크 첨부
```

> **GitHub Actions와의 차이점**: 워크플로우 파일 경로가 `.github/workflows/` → `.gitea/workflows/`로,  
> 컨텍스트 변수가 `${{ github.xxx }}` → `${{ gitea.xxx }}`로 변경됩니다. 그 외 YAML 문법은 100% 동일합니다.

### CI/CD 구성 요소 상세

| 구성 요소 | 기술 스택 | 자동화 핵심 역할 |
|-----------|-----------|-----------------|
| **워크플로우 정의** | Gitea Actions YAML | 빌드·테스트·배포 단계를 코드로 정의 (GitHub Actions 문법 호환) |
| **실행 환경** | Act Runner | 내 서버에서 빌드 수행. 시간 제한 없음. 외부 인터넷 불필요 |
| **상태 알림** | Mattermost Incoming Webhook | 빌드 성공·실패를 채널로 즉시 전송 |
| **이슈 연동** | Gitea 자동 Close 키워드 | PR 머지 시 `Close #이슈번호` 키워드로 이슈 자동 완료 |
| **문서 동기화** | Wiki.js Git Storage | 문서 변경 시 Gitea 저장소에 자동 커밋 |

---

## 5. 구축 로드맵 — 단계별 설치 순서

> 아래 순서대로 진행하면 의존성 문제 없이 완전한 환경을 구축할 수 있습니다.

```
사전 준비
  └── Oracle Linux 8.10 서버 준비
  └── Docker · Docker Compose 설치
        │
        ▼
✅ Step 1: Mattermost 구축          ← 팀 소통 채널 먼저 확보
        │  (포트 8065, PostgreSQL)
        ▼
✅ Step 2: Wiki.js 구축              ← 문서 저장소 구축
        │  (포트 3000, PostgreSQL)
        │  (Gitea 저장소 연동)
        ▼
✅ Step 3: Gitea 구축                ← 소스코드 저장소 구축 (GitHub 대체)
        │  (포트 3001/2222, PostgreSQL)
        │  (Organization · Team · 저장소 · 브랜치 보호)
        ▼
✅ Step 4: Gitea Issues · 마일스톤 구축  ← 일정/태스크 관리
        │  (이슈 · 마일스톤 · 라벨 · 이슈 템플릿)
        ▼
✅ Step 5: Act Runner 등록           ← CI/CD 실행 환경 구축
        │  (Gitea Actions 연결)
        ▼
✅ Step 6: 전체 통합 (Webhook)       ← 모든 시스템 연결
           (Actions → Mattermost 알림)
           (이슈 → PR 머지 시 자동 Close)
           (Wiki.js → Gitea 저장소 동기화)
```

---

## 6. Step 1 — Mattermost 구축

> 📄 **상세 설치 가이드**: `Mattermost_Setup_Guide.md` 참고

### 핵심 요약

**디렉터리 생성 및 권한:**
```bash
mkdir -p ~/mattermost/{config,data,db}
sudo chmod -R 777 ~/mattermost
```

**docker-compose.yml:**
```yaml
version: '3'
services:
  db:
    image: postgres:15-alpine
    volumes:
      - ./db:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=mmuser
      - POSTGRES_PASSWORD=mmuser_password   # 변경 필수
      - POSTGRES_DB=mattermost

  mattermost:
    image: mattermost/mattermost-team-edition:latest
    ports:
      - "8065:8065"
    volumes:
      - ./data:/mattermost/data
      - ./config:/mattermost/config
    environment:
      - MM_SQLSETTINGS_DRIVERNAME=postgres
      - MM_SQLSETTINGS_DATASOURCE=postgres://mmuser:mmuser_password@db:5432/mattermost?sslmode=disable
    depends_on:
      - db
```

**실행 및 방화벽:**
```bash
cd ~/mattermost
sudo docker compose up -d
sudo firewall-cmd --permanent --add-port=8065/tcp
sudo firewall-cmd --reload
```

**접속 확인:**
```
http://<서버_IP>:8065
```

### Mattermost 초기 설정 체크리스트

```
□ 관리자 계정 생성 (첫 번째 가입자 = 자동 관리자)
□ Site URL 설정: System Console → Environment → Web Server
□ 팀(Team) 생성
□ 채널 구성: #general · #dev-alert · #code-review · #incident
□ 파일 공유 활성화: System Console → File Sharing → Allow File Sharing: True
□ Incoming Webhook 생성 (Step 6에서 사용)
```

---

## 7. Step 2 — Wiki.js 구축

> 📄 **상세 설치 가이드**: `Wikijs_Setup_Guide.md` 참고

### 핵심 요약

**디렉터리 생성 및 권한:**
```bash
mkdir -p ~/wikijs/{config,data/db}
sudo chmod -R 777 ~/wikijs
```

**docker-compose.yml:**
```yaml
version: "3"
services:
  wiki-db:
    image: postgres:15-alpine
    restart: always
    environment:
      - POSTGRES_DB=wikijs
      - POSTGRES_USER=wikiuser
      - POSTGRES_PASSWORD=wiki_password     # 변경 필수
    volumes:
      - ./data/db:/var/lib/postgresql/data

  wikijs:
    image: requarks/wiki:2
    restart: always
    depends_on:
      - wiki-db
    environment:
      - DB_TYPE=postgres
      - DB_HOST=wiki-db
      - DB_PORT=5432
      - DB_USER=wikiuser
      - DB_PASS=wiki_password
      - DB_NAME=wikijs
    ports:
      - "3000:3000"
```

**실행 및 방화벽:**
```bash
cd ~/wikijs
sudo docker compose up -d
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload
```

**접속 확인:**
```
http://<서버_IP>:3000
```

### Wiki.js 초기 설정 체크리스트

```
□ 관리자 계정 생성 (Setup Wizard)
□ 한글화: Administration → Site → Locale → Korean
□ Gitea PAT 생성 (Gitea → Settings → Applications → Access Token)
□ Git Storage 연동: Administration → Storage → Git 설정
  → Repository URL: http://<서버_IP>:3001/<ORG>/wiki-docs.git
□ 에디터 기본값을 Markdown으로 설정
□ 문서 경로 구조 설계 (/project · /guide · /ops)
□ Mattermost Webhook 연동 (문서 변경 알림)
```

### 권장 Wiki.js 문서 구조

```
/
├── project/
│   ├── architecture/    아키텍처 설계 문서
│   ├── api/             API 명세서
│   └── database/        DB 스키마 · ERD
├── guide/
│   ├── onboarding/      신규 팀원 가이드
│   └── convention/      개발 컨벤션
└── ops/
    ├── setup/           인프라 설치 매뉴얼
    └── runbook/         운영 절차서
```

---

## 8. Step 3 — Gitea 구축

> 📄 **상세 설치 가이드**: `Gitea_Setup_Guide.md` 참고

Gitea는 **Go 언어로 작성된 경량 Self-Hosted Git 서비스**입니다.  
GitHub과 동일한 UI·기능을 제공하면서 내 서버에서 완전히 운영할 수 있습니다.  
인터넷이 차단된 폐쇄망 환경에서 GitHub을 100% 대체할 수 있는 가장 현실적인 선택입니다.

### GitHub vs Gitea 비교

| 항목 | GitHub | Gitea |
|------|--------|-------|
| **운영 방식** | SaaS (외부 서버) | Self-Hosted (내 서버) |
| **인터넷 필요** | 필수 | ❌ 불필요 (폐쇄망 가능) |
| **데이터 위치** | GitHub 서버 | 내 서버 (완전 통제) |
| **비용** | 팀 규모 증가 시 유료 | **완전 무료** |
| **저장소 수** | 플랜 제한 | **무제한** |
| **사용자 수** | 플랜 제한 | **무제한** |
| **CI/CD** | GitHub Actions (월 2,000분 제한) | Gitea Actions (제한 없음) |
| **리소스** | — | RAM 150MB ~ (매우 경량) |

### 8-1. 디렉터리 생성

```bash
mkdir -p ~/gitea/{data,config,db}
sudo chmod -R 777 ~/gitea
```

### 8-2. docker-compose.yml 작성

```bash
cd ~/gitea
vi docker-compose.yml
```

```yaml
# ~/gitea/docker-compose.yml
version: "3"

networks:
  gitea-net:
    driver: bridge

services:

  # ─── PostgreSQL 데이터베이스 ──────────────────────────────
  gitea-db:
    image: postgres:15-alpine
    container_name: gitea-db
    restart: always
    networks:
      - gitea-net
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea_password        # ← 반드시 변경
      - POSTGRES_DB=gitea
    volumes:
      - ./db:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U gitea"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ─── Gitea 서버 ──────────────────────────────────────────
  gitea:
    image: gitea/gitea:latest
    container_name: gitea
    restart: always
    networks:
      - gitea-net
    depends_on:
      gitea-db:
        condition: service_healthy
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - GITEA__database__DB_TYPE=postgres
      - GITEA__database__HOST=gitea-db:5432
      - GITEA__database__NAME=gitea
      - GITEA__database__USER=gitea
      - GITEA__database__PASSWD=gitea_password  # ← DB 비밀번호와 동일하게 변경
    ports:
      - "3001:3000"     # 웹 UI (호스트 3001 → 컨테이너 3000)
      - "2222:22"       # SSH  (호스트 2222 → 컨테이너 22)
    volumes:
      - ./data:/data
      - ./config:/etc/gitea
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
```

### 8-3. 컨테이너 실행

```bash
cd ~/gitea
sudo docker compose up -d

# 실행 상태 확인
sudo docker compose ps
sudo docker compose logs -f gitea   # 로그 실시간 확인 (Ctrl+C로 종료)
```

**정상 기동 로그 확인:**
```
gitea  | ... server.go:xxx: HTTP Listener: 0.0.0.0:3000
```

### 8-4. 방화벽 포트 허용

```bash
sudo firewall-cmd --permanent --add-port=3001/tcp
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

### 8-5. 초기 설정 마법사 (Install Wizard)

브라우저에서 `http://<서버_IP>:3001` 접속 → **Install Gitea** 설정 페이지 표시

**데이터베이스 설정:**

| 항목 | 입력값 |
|------|--------|
| Database Type | `PostgreSQL` |
| Host | `gitea-db:5432` |
| Username | `gitea` |
| Password | `gitea_password` |
| Database Name | `gitea` |

**서버 도메인 설정 (폐쇄망에서 가장 중요):**

| 항목 | 입력값 | 설명 |
|------|--------|------|
| Server Domain | `192.168.xx.xxx` | 서버의 실제 IP 주소 |
| Gitea HTTP Listen Port | `3000` | 컨테이너 내부 포트 (변경 금지) |
| Gitea Base URL | `http://192.168.xx.xxx:3001/` | 외부에서 접속하는 실제 URL |
| SSH Server Port | `2222` | 호스트 SSH 포트 |

**선택 기능 설정:**

| 항목 | 권장값 | 설명 |
|------|--------|------|
| Enable Registration | ✅ 활성화 | 초기 팀원 가입용 (이후 비활성화 권장) |
| Require Sign-In | ✅ 활성화 | 로그인 없이는 접근 불가 |
| Enable Git Hooks | ✅ 활성화 | 서버사이드 Git Hook 사용 |
| Disable CDN | ✅ 활성화 | **폐쇄망 필수** — 외부 CDN 차단 |

**관리자 계정 생성:**
```
Admin Username : admin
Admin Password : Admin@1234!    ← 8자 이상, 특수문자 포함
Admin Email    : admin@local.dev
```

→ **[Install Gitea]** 버튼 클릭

### 8-6. 관리자 초기 설정

```
우측 상단 프로필 아이콘 → Site Administration
또는: http://<서버_IP>:3001/-/admin
```

**보안 설정 — 회원가입 제한** (팀원 가입 완료 후):
```
Site Administration → Settings → User Settings
□ Disable Self-Registration  → 체크
□ Require Sign-In To View Pages → 체크
[Save]
```

**Git 저장소 기본 설정:**
```
Site Administration → Settings → Repository Settings
Default Branch      : main
Enable Git LFS      : ✅ 체크
[Save]
```

**Webhook 허용 주소 설정 (Mattermost 연동 필수):**
```
Site Administration → Settings → Webhook Settings
Allowed Hosts : <서버_IP>   ← Mattermost 서버 IP
[Save]
```

**Gitea Actions 활성화:**
```
Site Administration → Settings → Repository Settings
→ Enable Repository Actions : ✅ 체크
→ [Save]
```

### 8-7. Organization 및 Team 구성

**Organization 생성:**
```
우측 상단 [+] → New Organization
Organization Name : my-dev-team   (영문·숫자·하이픈만)
Visibility        : Private
[Create Organization]
```

**Team 구성:**

| Team 이름 | 권한 | 대상 |
|-----------|------|------|
| `Owners` | Owner | 조직 관리자 (자동 생성) |
| `Developers` | Write | 일반 개발자 — push·PR 가능 |
| `Reviewers` | Write | 시니어 개발자 — 리뷰·머지 권한 |
| `Viewers` | Read | 열람 전용 — 외부 협력사 등 |

```
Organization 페이지 → Teams 탭 → [New Team]
Team Name  : Developers
Permission : Write
[Create Team]
```

### 8-8. 저장소 생성 및 브랜치 보호

**저장소 생성:**
```
우측 상단 [+] → New Repository
Owner                : my-dev-team (Organization 선택)
Repository Name      : my-app
Visibility           : Private
Initialize Repository: ✅ 체크
Default Branch       : main
[Create Repository]
```

**브랜치 보호 규칙 설정 (main 브랜치 직접 push 금지):**
```
저장소 → Settings → Branches → [Add Branch Protection Rule]
Branch Name Pattern  : main
□ Protect Branch        : ✅ 체크
□ Block Force Push      : ✅ 체크
□ Require Merge Request : ✅ 체크 (PR 필수)
□ Required Approvals    : 1
[Save]
```

**로컬 저장소 Clone:**
```bash
# HTTPS 방식
git clone http://<서버_IP>:3001/my-dev-team/my-app.git

# SSH 방식 (SSH 키 등록 후)
git clone ssh://git@<서버_IP>:2222/my-dev-team/my-app.git
```

### 8-9. SSH 키 등록

```bash
# 로컬 PC에서 SSH 키 생성
ssh-keygen -t ed25519 -C "gitea-key"

# 공개키 내용 복사
cat ~/.ssh/id_ed25519.pub
```

```
Gitea → 우측 상단 프로필 → Settings → SSH / GPG Keys → [Add Key]
Key Name : my-laptop
Content  : (위에서 복사한 공개키 전체 붙여넣기)
[Add Key]
```

**SSH 연결 테스트:**
```bash
ssh -T git@<서버_IP> -p 2222
# 성공: Hi <username>! You've successfully authenticated with Gitea.
```

### 8-10. Gitea 초기 설정 체크리스트

```
□ 컨테이너 정상 기동 확인
□ Install Wizard 완료 (DB · 도메인 · 관리자 계정)
□ 회원가입 제한 설정
□ Gitea Actions 활성화
□ Webhook 허용 주소 등록 (Mattermost IP)
□ Organization 생성 및 Team 구성
□ 저장소 생성 (main 브랜치 보호 규칙 설정)
□ 팀원 계정 생성 및 Team 배정
□ SSH 키 등록 안내
□ 로컬 git config 설정 (user.name · user.email)
```

---

## 9. Step 4 — Gitea Issues · 마일스톤 · 칸반 구축

> 📄 **상세 설치 가이드**: `Gitea_Setup_Guide.md` 8~9장 참고

Gitea의 이슈(Issues)와 마일스톤(Milestones)을 활용하여 기존 GitHub Projects 칸반 보드와 동일한 일정·태스크 관리 환경을 구축합니다.

### 9-1. 라벨 표준화

이슈 분류를 위한 라벨을 표준화합니다.

```
저장소 → Issues → Labels → [Create Default Labels] 또는 [New Label]
```

**권장 라벨 구성:**

| 라벨명 | 색상 코드 | 의미 |
|--------|----------|------|
| `feature` | `#0075ca` | 새 기능 개발 |
| `bug` | `#d73a4a` | 버그 수정 |
| `docs` | `#0052cc` | 문서 작업 |
| `infra` | `#e4e669` | 인프라·DevOps |
| `refactor` | `#cfd3d7` | 코드 개선 |
| `hotfix` | `#b60205` | 긴급 수정 |
| `blocked` | `#d93f0b` | 다른 작업에 의해 차단됨 |

### 9-2. 마일스톤 생성 (스프린트 관리)

마일스톤은 스프린트 또는 릴리즈 버전 단위로 이슈를 묶어 진척도를 추적합니다.

```
저장소 → Issues → Milestones → [New Milestone]
```

| 항목 | 입력 예시 |
|------|-----------|
| Title | `Sprint 1` 또는 `v1.0.0` |
| Description | `2026년 4월 3주차 스프린트` |
| Due Date | `2026-04-25` |

**마일스톤 진척도 확인:**
```
저장소 → Issues → Milestones
→ 각 마일스톤의 Open/Closed 이슈 비율로 진척도 자동 표시 (% 게이지)
```

### 9-3. 이슈 템플릿 설정

팀 전체가 일관된 형식으로 이슈를 작성하도록 템플릿을 등록합니다.

**템플릿 파일 생성 위치:**
```
저장소 루트/.gitea/ISSUE_TEMPLATE/
```

**기능 개발 템플릿** (`.gitea/ISSUE_TEMPLATE/feature.md`):
```markdown
---
name: ✨ 기능 개발
about: 새로운 기능 개발 요청
title: "[FEAT] "
labels: feature
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
```

**버그 리포트 템플릿** (`.gitea/ISSUE_TEMPLATE/bug.md`):
```markdown
---
name: 🐛 버그 리포트
about: 버그 신고 및 수정 요청
title: "[BUG] "
labels: bug
---

## 🐛 버그 설명

## 🔁 재현 방법
1.
2.
3.

## ✅ 기대 동작

## ❌ 실제 동작

## 🖥️ 환경 정보
- OS:
- 버전:

## 📸 스크린샷
```

### 9-4. PR 템플릿 설정

```
저장소 루트/.gitea/PULL_REQUEST_TEMPLATE.md
```

```markdown
## 🔗 관련 이슈
Close #이슈번호

## 📝 변경 내용
<!-- 어떤 변경을 했는지 요약 -->

## ✅ 테스트 확인
- [ ] 로컬 테스트 완료
- [ ] 기존 기능 영향 없음 확인

## 📸 스크린샷 (UI 변경 시)

## 💬 리뷰어에게
<!-- 리뷰 시 참고할 사항 -->
```

> PR 본문에 `Close #42` 형식으로 작성하면 PR 머지 시 해당 이슈가 **자동으로 Close** 됩니다.

### 9-5. 이슈 자동 Close 키워드

```
Close #42       ← 권장
Closes #42
Fix #42
Fixes #42
Resolve #42
Resolves #42
```

### 9-6. 브랜치 전략

```
main          ← 운영 배포 (PR + 승인 필수, 직접 push 금지)
  └── develop ← 개발 통합
        ├── feature/이슈번호-기능명   예) feature/42-user-auth
        ├── bugfix/이슈번호-버그명    예) bugfix/55-login-error
        └── hotfix/이슈번호-긴급명   예) hotfix/60-payment-fix
```

### 9-7. 커밋 메시지 컨벤션

```
[타입]: #이슈번호 작업 내용 요약

타입 목록:
  feat     : 새 기능 추가
  fix      : 버그 수정
  docs     : 문서 수정
  style    : 코드 포맷팅
  refactor : 코드 리팩터링
  test     : 테스트 코드
  infra    : 인프라·빌드 설정
  chore    : 기타 (패키지 업데이트 등)
```

**예시:**
```bash
git commit -m "feat: #42 사용자 로그인 JWT 인증 구현"
git commit -m "fix: #55 로그인 페이지 무한 리다이렉트 수정"
git commit -m "docs: #60 API 명세서 Swagger 추가"
```

> 커밋 메시지에 `#이슈번호`를 포함하면 Gitea에서 해당 이슈와 커밋이 **자동으로 연결**됩니다.

### 9-8. 스프린트 운영 사이클

```
월요일 (스프린트 시작)
  └── 스프린트 계획 회의
       ├── 마일스톤 생성 또는 선택 (Sprint N)
       ├── 이슈 생성 및 우선순위 설정
       ├── 마일스톤에 이슈 할당
       └── 담당자(Assignee) 배정

매일 (데일리 스탠드업)
  └── Issues 탭의 Open 이슈 기준으로 현황 공유
       ├── 진행 중 이슈 상태 공유
       ├── 블로킹(blocked) 이슈 공유
       └── 오늘 진행할 이슈 확인

금요일 (스프린트 종료)
  └── 마일스톤 페이지에서 진척도 확인
       ├── Closed 이슈 비율 확인
       ├── 미완료 이슈 → 다음 마일스톤으로 이전
       └── 다음 스프린트 마일스톤 생성
```

### 9-9. Gitea 이슈 초기 설정 체크리스트

```
□ 라벨 표준화 (feature · bug · docs · infra · refactor · hotfix · blocked)
□ 마일스톤 생성 (스프린트 또는 버전 단위)
□ 이슈 템플릿 생성 (.gitea/ISSUE_TEMPLATE/)
□ PR 템플릿 생성 (.gitea/PULL_REQUEST_TEMPLATE.md)
□ 브랜치 보호 규칙: main · develop 직접 push 금지
□ 이슈 검색 필터 활용법 팀 공유
```

---

## 10. Step 5 — Gitea Actions Self-hosted Runner 구축

> 📄 **상세 설치 가이드**: `Gitea_Setup_Guide.md` 10장 참고

Gitea Actions의 Act Runner는 **내 리눅스 서버에서 직접 빌드를 수행**하는 실행 에이전트입니다.  
시간 제한 없이 서버 자원을 100% 활용할 수 있으며, 인터넷이 없는 폐쇄망에서도 동작합니다.

### 10-1. Act Runner 설치

```bash
# act_runner 다운로드 (최신 버전: https://gitea.com/gitea/act_runner/releases)
RUNNER_VERSION="0.2.11"

wget -O act_runner \
  https://gitea.com/gitea/act_runner/releases/download/v${RUNNER_VERSION}/act_runner-${RUNNER_VERSION}-linux-amd64

# 실행 권한 부여 및 시스템 경로로 이동
chmod +x act_runner
sudo mv act_runner /usr/local/bin/act_runner

# 버전 확인
act_runner --version
```

> ⚠️ **폐쇄망 환경**: 인터넷이 되는 PC에서 바이너리를 다운로드한 후 서버로 복사합니다.
> ```bash
> scp act_runner user@<서버_IP>:~/
> ```

**Runner 전용 디렉터리 생성:**
```bash
sudo useradd -m runner          # Runner 전용 사용자 생성 (권장)
sudo mkdir -p /opt/act-runner
sudo chown runner:runner /opt/act-runner
```

### 10-2. Runner 등록 토큰 발급

```
Site Administration → Actions → Runners → [Create new Runner]
→ 표시된 Registration Token 복사
```

또는 저장소 단위 Runner:
```
저장소 → Settings → Actions → Runners → [Create new Runner]
→ Registration Token 복사
```

### 10-3. Runner 등록

```bash
cd /opt/act-runner

act_runner register \
  --instance http://<서버_IP>:3001 \
  --token <REGISTRATION_TOKEN> \
  --name oracle-linux-runner \
  --labels self-hosted,linux,oracle-linux \
  --no-interactive
```

### 10-4. Runner systemd 서비스 등록 (부팅 시 자동 시작)

```bash
sudo tee /etc/systemd/system/act-runner.service << 'EOF'
[Unit]
Description=Gitea Act Runner
After=network.target

[Service]
Type=simple
User=runner
WorkingDirectory=/opt/act-runner
ExecStart=/usr/local/bin/act_runner daemon
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable act-runner
sudo systemctl start act-runner

# 상태 확인
sudo systemctl status act-runner
```

### 10-5. Runner 상태 확인

```
Site Administration → Actions → Runners
→ 등록한 Runner가 "Idle" 상태로 표시되면 성공 ✅
```

### 10-6. 필수 빌드 도구 설치

Runner가 실행될 서버에 프로젝트 빌드에 필요한 도구를 설치합니다.

```bash
# Node.js (프론트엔드 · Node 기반 프로젝트)
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo dnf install -y nodejs

# Python (Python 기반 프로젝트)
sudo dnf install -y python3 python3-pip

# Java / Gradle (Spring Boot 등)
sudo dnf install -y java-17-openjdk-devel

# Docker (컨테이너 배포 시 필요 — 이미 설치된 경우 생략)
# Git
sudo dnf install -y git

# 설치 확인
node --version && python3 --version && java -version && docker --version && git --version
```

### 10-7. Gitea Actions 워크플로우 예시

> **GitHub Actions와의 핵심 차이**:
> - 워크플로우 파일 위치: `.github/workflows/` → `.gitea/workflows/`
> - 컨텍스트 변수: `${{ github.xxx }}` → `${{ gitea.xxx }}`
> - YAML 문법 자체는 100% 동일

```yaml
# .gitea/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build-and-test:
    # ✅ Act Runner에서 실행
    runs-on: self-hosted

    steps:
      # 1. 코드 체크아웃
      - name: Checkout code
        uses: actions/checkout@v4

      # 2. 의존성 설치
      - name: Install dependencies
        run: npm install

      # 3. 빌드
      - name: Build
        run: npm run build

      # 4. 테스트
      - name: Run tests
        run: npm test

      # 5. 배포 (main 브랜치만)
      - name: Deploy
        if: gitea.ref == 'refs/heads/main'
        run: |
          docker compose -f /opt/myapp/docker-compose.yml pull
          docker compose -f /opt/myapp/docker-compose.yml up -d

      # 6. 성공 알림 → Mattermost
      - name: Notify Success
        if: success()
        run: |
          curl -i -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"✅ **빌드 성공** \`${{ gitea.repository }}\`\n브랜치: \`${{ gitea.ref_name }}\`\n커밋: ${{ gitea.event.head_commit.message }}\"}" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}

      # 7. 실패 알림 → Mattermost
      - name: Notify Failure
        if: failure()
        run: |
          curl -i -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"❌ **빌드 실패** \`${{ gitea.repository }}\`\n브랜치: \`${{ gitea.ref_name }}\`\n[로그 확인](${{ gitea.server_url }}/${{ gitea.repository }}/actions/runs/${{ gitea.run_id }})\"}" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}
```

### 10-8. Secrets 등록

```
저장소 → Settings → Actions → Secrets → [Add Secret]

Name  : MATTERMOST_WEBHOOK_URL
Value : http://<서버_IP>:8065/hooks/xxxxxxxxxx
```

---

## 11. Step 6 — 전체 시스템 통합 (Webhook 연동)

모든 구성 요소를 연결하여 **완전 자동화된 개발 관리 환경**을 완성합니다.

### 11-1. Mattermost Incoming Webhook 생성

```
Mattermost → 주 메뉴 → Integrations → Incoming Webhooks
→ [Add Incoming Webhook]
→ 채널: #dev-alert
→ [Save] → Webhook URL 복사
```

### 11-2. Gitea 저장소 Webhook 설정

Gitea의 이슈·PR·push 이벤트를 Mattermost로 자동 전송합니다.

```
저장소 → Settings → Webhooks → [Add Webhook] → Gitea 선택
```

| 항목 | 입력값 |
|------|--------|
| Target URL | `http://<서버_IP>:8065/hooks/xxxxxxxxxx` |
| HTTP Method | `POST` |
| Content Type | `application/json` |

**권장 트리거 이벤트:**
```
✅ Push            ← 코드 push 시 알림
✅ Pull Request    ← PR 생성·머지·댓글 시 알림
✅ Issues          ← 이슈 생성·완료 시 알림
✅ Issue Comment   ← 이슈 댓글 시 알림
```

**Webhook 동작 테스트:**
```
Webhooks 목록 → 해당 Webhook → [Test Delivery]
→ Mattermost #dev-alert 채널에 테스트 메시지 수신 확인
```

### 11-3. Gitea Secrets 등록

```
저장소 → Settings → Actions → Secrets → [New repository secret]
```

| Secret 이름 | 값 |
|------------|-----|
| `MATTERMOST_WEBHOOK_URL` | Mattermost에서 복사한 Webhook URL |

### 11-4. 통합 알림 워크플로우 전체 구성

```yaml
# .gitea/workflows/notify-all.yml
name: Team Notification Hub

on:
  issues:
    types: [opened, closed, assigned]
  pull_request:
    types: [opened, closed, review_requested]
  push:
    branches: [main]

jobs:
  # ─── 이슈 알림 ───────────────────────────────────────────
  issue-notify:
    if: gitea.event_name == 'issues'
    runs-on: self-hosted
    steps:
      - name: Send Issue Notification
        run: |
          ACTION="${{ gitea.event.action }}"
          TITLE="${{ gitea.event.issue.title }}"
          URL="${{ gitea.event.issue.html_url }}"
          ASSIGNEE="${{ gitea.event.issue.assignee.login }}"

          if [ "$ACTION" = "opened" ]; then
            MSG="🆕 **새 이슈 등록**: [$TITLE]($URL)\n담당자: ${ASSIGNEE:-미지정}"
          elif [ "$ACTION" = "closed" ]; then
            MSG="✅ **이슈 완료**: [$TITLE]($URL)"
          elif [ "$ACTION" = "assigned" ]; then
            MSG="👤 **담당자 배정**: [$TITLE]($URL)\n담당자: $ASSIGNEE"
          fi

          curl -s -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"$MSG\"}" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}

  # ─── PR 알림 ─────────────────────────────────────────────
  pr-notify:
    if: gitea.event_name == 'pull_request'
    runs-on: self-hosted
    steps:
      - name: Send PR Notification
        run: |
          ACTION="${{ gitea.event.action }}"
          TITLE="${{ gitea.event.pull_request.title }}"
          URL="${{ gitea.event.pull_request.html_url }}"
          MERGED="${{ gitea.event.pull_request.merged }}"

          if [ "$ACTION" = "opened" ]; then
            MSG="🔀 **PR 오픈**: [$TITLE]($URL)"
          elif [ "$ACTION" = "closed" ] && [ "$MERGED" = "true" ]; then
            MSG="🎉 **PR 머지 완료**: [$TITLE]($URL)"
          elif [ "$ACTION" = "review_requested" ]; then
            MSG="👀 **코드 리뷰 요청**: [$TITLE]($URL)"
          fi

          [ -n "$MSG" ] && curl -s -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"$MSG\n저장소: \`${{ gitea.repository }}\`\"}" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}

  # ─── 배포 알림 ───────────────────────────────────────────
  deploy-notify:
    if: gitea.event_name == 'push' && gitea.ref == 'refs/heads/main'
    runs-on: self-hosted
    steps:
      - name: Send Deploy Notification
        run: |
          curl -s -X POST \
            -H 'Content-Type: application/json' \
            -d "{\"text\": \"🚀 **배포 시작**: \`${{ gitea.repository }}\`\n커밋: ${{ gitea.event.head_commit.message }}\n작성자: ${{ gitea.event.head_commit.author.name }}\"}" \
            ${{ secrets.MATTERMOST_WEBHOOK_URL }}
```

### 11-5. 통합 완성 후 자동화 흐름 전체

```
개발자가 이슈 생성 (Gitea Issues)
    → #dev-alert 채널에 이슈 알림 자동 전송 (Mattermost)

개발자가 브랜치 생성 후 코드 push (Gitea 서버로)
    → Act Runner에서 빌드·테스트 자동 실행

PR 오픈
    → #dev-alert 채널에 PR 알림 자동 전송
    → 리뷰어에게 Mattermost 알림

PR 머지 (main 브랜치)
    → 자동 배포 실행
    → 빌드 성공/실패 알림 전송
    → 연결된 이슈 자동 Close (Close #N 키워드)
    → 마일스톤 진척도 자동 업데이트

Wiki.js 문서 작성/수정
    → Gitea 저장소에 .md 파일 자동 커밋·백업
    → (선택) Mattermost 문서 변경 알림 전송

※ 모든 데이터가 내 서버 내부에서만 이동 (인터넷 완전 불필요)
```

---

## 12. 아키텍처 핵심 가치

### 🔒 데이터 주권 확보 (강화됨)

```
이전 구성 (GitHub 기반)         현재 구성 (완전 Self-Hosted)
─────────────────────────  →  ──────────────────────────────
Mattermost → 내 서버             Mattermost → 내 서버 ✅
Wiki.js    → 내 서버             Wiki.js    → 내 서버 ✅
GitHub     → 외부 서버 ❌        Gitea      → 내 서버 ✅ (강화)
CI/CD 로그 → GitHub 서버 ❌     CI/CD 로그 → 내 서버 ✅ (강화)

→ 코드·이슈·빌드 로그·문서 모두 내 서버에서 완전 통제
→ 서버 장애 시에도 Gitea ↔ Wiki.js 상호 백업으로 즉시 복구 가능
```

### 💰 운영 비용 제로

| 대체 상용 도구 | 10인 기준 월 비용 | 우리의 도구 | 비용 |
|--------------|-----------------|------------|------|
| Slack Pro | ~$85/월 | Mattermost | **$0** |
| Notion Team | ~$80/월 | Wiki.js | **$0** |
| GitHub Team | ~$40/월 | Gitea | **$0** |
| Jira Standard | ~$85/월 | Gitea Issues | **$0** |
| GitHub Actions | ~$16/월 초과분 | Act Runner | **$0** |
| **합계** | **~$306/월** | | **$0/월** |

### 🛠️ TA 친화적 구조

```
모든 서비스가 Docker 컨테이너로 운영
    → 어떤 Linux 서버에서도 동일하게 재현 가능
    → docker compose up -d 한 줄로 전체 서비스 기동
    → 버전 관리 · 롤백 · 스케일업 용이
    → 폐쇄망 · 인터넷 차단 환경에서도 100% 동작
```

### 📈 유연한 확장성

```
현재 구성에서 확장 가능한 방향:
    ├── HTTPS 적용 (Nginx Reverse Proxy + Let's Encrypt 또는 자체 CA)
    ├── LDAP/SSO 연동 (사내 계정 통합)
    ├── 모니터링 추가 (Grafana + Prometheus)
    ├── Harbor (컨테이너 이미지 레지스트리) 추가
    ├── 멀티 Runner 구성 (병렬 빌드)
    └── Gitea ↔ Wiki.js 미러링 이중화
```

---

## 13. 운영 관리 가이드

### 13-1. 일일 운영 체크리스트

```bash
# 전체 서비스 상태 확인
sudo docker ps -a

# Mattermost 상태
cd ~/mattermost && sudo docker compose ps

# Wiki.js 상태
cd ~/wikijs && sudo docker compose ps

# Gitea 상태
cd ~/gitea && sudo docker compose ps

# Act Runner 상태
sudo systemctl status act-runner

# 디스크 사용량 확인
df -h
du -sh ~/mattermost ~/wikijs ~/gitea
```

### 13-2. 전체 서비스 한 번에 시작

```bash
# Mattermost 시작
cd ~/mattermost && sudo docker compose up -d

# Wiki.js 시작
cd ~/wikijs && sudo docker compose up -d

# Gitea 시작
cd ~/gitea && sudo docker compose up -d

# Act Runner는 systemd 서비스로 자동 실행됨
sudo systemctl start act-runner
```

### 13-3. 정기 백업 스크립트

```bash
#!/bin/bash
# /opt/scripts/backup.sh
# cron에 등록: 0 2 * * * /opt/scripts/backup.sh

BACKUP_DIR="/opt/backups"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Mattermost 백업
tar -czf $BACKUP_DIR/mattermost_$DATE.tar.gz \
  ~/mattermost/config ~/mattermost/data ~/mattermost/db

# Wiki.js 백업
tar -czf $BACKUP_DIR/wikijs_$DATE.tar.gz \
  ~/wikijs/config ~/wikijs/data

# Gitea 백업 (저장소 · 설정 · DB)
tar -czf $BACKUP_DIR/gitea_data_$DATE.tar.gz ~/gitea/data ~/gitea/config
tar -czf $BACKUP_DIR/gitea_db_$DATE.tar.gz ~/gitea/db

# 30일 이상 된 백업 자동 삭제
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete

echo "[$DATE] 백업 완료: $BACKUP_DIR"
```

**cron 등록:**
```bash
crontab -e
# 매일 새벽 2시 자동 백업
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

### 13-4. 이미지 업데이트 절차

```bash
# 1. 최신 이미지 다운로드
cd ~/mattermost && sudo docker compose pull
cd ~/wikijs && sudo docker compose pull
cd ~/gitea && sudo docker compose pull

# 2. 서비스 재시작 (순차적으로)
cd ~/mattermost && sudo docker compose up -d
cd ~/wikijs && sudo docker compose up -d
cd ~/gitea && sudo docker compose up -d

# 3. 업데이트 확인
sudo docker compose ps
sudo docker compose logs --tail=20
```

### 13-5. 전체 서비스 모니터링

```bash
# 실시간 컨테이너 리소스 모니터링
sudo docker stats

# 포트 리스닝 전체 확인
sudo netstat -tulpn | grep -E "8065|3000|3001|2222|5432"

# 방화벽 허용 포트 목록
sudo firewall-cmd --list-ports

# Gitea 전체 상태 한 번에 확인
echo "=== Gitea ===" && cd ~/gitea && sudo docker compose ps
echo "=== Act Runner ===" && sudo systemctl status act-runner --no-pager | grep -E "Active|Loaded"
echo "=== 포트 ===" && sudo netstat -tulpn | grep -E "3001|2222"
echo "=== 디스크 ===" && du -sh ~/gitea/
```

---

## 14. 전체 시스템 트러블슈팅

### ❌ 서버 재부팅 후 서비스가 뜨지 않을 때

```bash
# docker-compose.yml에 restart: always 설정 확인
grep "restart" ~/gitea/docker-compose.yml

# 수동 재시작
cd ~/mattermost && sudo docker compose up -d
cd ~/wikijs && sudo docker compose up -d
cd ~/gitea && sudo docker compose up -d

# Act Runner 자동 시작 확인
sudo systemctl enable act-runner
sudo systemctl start act-runner

# Docker 서비스 자동 시작 활성화
sudo systemctl enable docker
```

---

### ❌ 디스크 공간 부족

```bash
# 사용 중인 Docker 리소스 확인
sudo docker system df

# 미사용 이미지 · 컨테이너 · 볼륨 정리
sudo docker system prune -f

# 오래된 로그 정리
sudo find /var/lib/docker/containers -name "*.log" -size +100M
```

---

### ❌ Gitea 웹 페이지에 접속되지 않을 때

```bash
# 1. 컨테이너 실행 상태 확인
cd ~/gitea && sudo docker compose ps
sudo docker compose logs gitea --tail=30

# 2. 포트 리스닝 확인
sudo netstat -tulpn | grep 3001

# 3. 방화벽 확인
sudo firewall-cmd --list-ports

# 4. 컨테이너 재시작
cd ~/gitea && sudo docker compose restart
```

---

### ❌ Gitea SSH clone 시 "Connection refused" 오류

```bash
# 1. SSH 포트(2222) 방화벽 허용 확인
sudo firewall-cmd --list-ports | grep 2222

# 2. 미허용 시 추가
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --reload

# 3. SSH 연결 테스트
ssh -T git@<서버_IP> -p 2222 -v
```

---

### ❌ Act Runner가 Offline으로 표시될 때

```bash
# 서비스 상태 확인
sudo systemctl status act-runner

# 서비스 재시작
sudo systemctl restart act-runner

# 로그 확인
journalctl -u act-runner -n 50

# Runner 재등록 필요 시 (토큰 만료 등)
cd /opt/act-runner
act_runner register \
  --instance http://<서버_IP>:3001 \
  --token NEW_TOKEN \
  --name oracle-linux-runner \
  --labels self-hosted,linux,oracle-linux \
  --no-interactive
sudo systemctl restart act-runner
```

---

### ❌ Gitea Actions 워크플로우가 실행되지 않을 때

```bash
# 1. Actions 기능 활성화 확인
# Site Administration → Settings → Repository Settings → Enable Repository Actions

# 2. Act Runner 상태 확인
sudo systemctl status act-runner
journalctl -u act-runner -n 50

# 3. Runner 등록 상태 확인
# Site Administration → Actions → Runners → 등록된 Runner가 Idle 상태인지 확인

# 4. 워크플로우 파일 경로 확인 (GitHub과 다름!)
# → .gitea/workflows/*.yml  (GitHub: .github/workflows/)
ls -la .gitea/workflows/
```

---

### ❌ Mattermost ↔ Gitea 연동이 끊어졌을 때

```bash
# 1. 각 서비스 상태 확인
sudo docker ps -a | grep -E "mattermost|wiki|gitea"

# 2. Webhook URL 유효성 확인
curl -s -X POST \
  -H 'Content-Type: application/json' \
  -d '{"text": "연결 테스트"}' \
  http://<서버_IP>:8065/hooks/xxxxxxxxxxxx

# 3. Gitea Webhook 허용 주소 재확인
# Site Administration → Settings → Webhook Settings → Allowed Hosts

# 4. Gitea Secrets 재확인
# 저장소 → Settings → Actions → Secrets → MATTERMOST_WEBHOOK_URL 값 확인

# 5. Gitea Webhook 최근 전송 기록 확인
# 저장소 → Settings → Webhooks → 해당 Webhook → [Recent Deliveries]
```

---

### ❌ Wiki.js ↔ Gitea 동기화가 끊어졌을 때

```bash
# 1. Wiki.js GitHub Storage 설정에서 Gitea PAT 만료 여부 확인
# Gitea → Settings → Applications → Access Token → 유효 토큰 재발급

# 2. Wiki.js 관리자 콘솔에서 재동기화
# Administration → Storage → Git → [Force Sync]

# 3. Gitea 저장소 접근 권한 확인
# Gitea → 저장소 → Settings → Collaborators 또는 Team 권한 확인
```

---

## 📚 전체 가이드 문서 목록

| 문서 | 내용 |
|------|------|
| `Project_Management_Architecture_Guide_Gitea.md` | 📍 현재 문서 — 전체 로드맵 (Gitea 기준) |
| `Mattermost_Setup_Guide.md` | Step 1 — Mattermost 상세 설치 가이드 |
| `Wikijs_Setup_Guide.md` | Step 2 — Wiki.js 상세 설치 가이드 |
| `Gitea_Setup_Guide.md` | Step 3~5 — Gitea 설치·이슈·CI/CD 상세 가이드 |

---

## 🗺️ 구축 완료 현황

```
✅ Step 1: Mattermost Self-Hosted 구축      (팀 소통 채널)
✅ Step 2: Wiki.js Self-Hosted 구축         (기술 문서 위키)
✅ Step 3: Gitea Self-Hosted 구축           (소스코드 저장소 · GitHub 완전 대체)
   ├── Organization · Team 구성
   ├── 저장소 생성 · 브랜치 보호 설정
   └── SSH 키 등록 · 팀원 계정 설정
✅ Step 4: Gitea Issues · 마일스톤 구축     (태스크 관리 · 스프린트 운영)
   ├── 이슈 · 마일스톤 · 라벨 설정
   ├── 이슈 템플릿 · PR 템플릿
   └── 브랜치 전략 · 커밋 컨벤션 수립
✅ Step 5: Act Runner 등록                  (CI/CD 실행 환경 · 시간 제한 없음)
✅ Step 6: 전체 통합 — Webhook 연동         (자동화 완성)

🎉 완전 Self-Hosted 엔터프라이즈급 개발 관리 인프라 구축 완료!
   운영 비용: $0 / 월
   데이터 위치: 내 서버 (완전 통제 · 폐쇄망 가능)
   인터넷 의존도: 0% (모든 서비스 내부 운영)
```

---

<div align="center">

**본 문서는 소프트웨어 개발 프로젝트의 완전 무료 Self-Hosted 인프라 구축을 목적으로 작성되었습니다.**

`Oracle Linux 8.10` · `Docker` · `Mattermost` · `Wiki.js` · `Gitea` · `Gitea Actions` · `Act Runner`

*작성일: 2026-04-15*  
*작성자: Kim Jong-in (Technical Architect)*

</div>
