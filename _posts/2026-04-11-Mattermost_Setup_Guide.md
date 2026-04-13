# 🚀 소프트웨어 개발 관리 인프라 구축 가이드

> **Oracle Linux 8.10 + Docker 기반 Mattermost Self-Hosted 완전 설치 매뉴얼**  
> TA/개발자를 위한 무료 오픈소스 조합으로 엔터프라이즈급 개발 환경 구축

---

## 📋 목차

1. [아키텍처 개요](#1-아키텍처-개요)
2. [사전 요구사항](#2-사전-요구사항)
3. [Docker 및 Docker Compose 설치](#3-docker-및-docker-compose-설치)
4. [디렉터리 구조 및 권한 설정](#4-디렉터리-구조-및-권한-설정)
5. [docker-compose.yml 구성](#5-docker-composeyml-구성)
6. [방화벽 설정](#6-방화벽-설정)
7. [Mattermost 실행 및 상태 확인](#7-mattermost-실행-및-상태-확인)
8. [초기 설정 및 관리자 콘솔](#8-초기-설정-및-관리자-콘솔)
9. [기본 사용법](#9-기본-사용법)
10. [운영 명령어 치트시트](#10-운영-명령어-치트시트)
11. [문제 해결 (Troubleshooting)](#11-문제-해결-troubleshooting)

---

## 1. 아키텍처 개요

본 가이드는 소프트웨어 개발 프로젝트를 **완전 무료**로 운영하기 위한 Self-Hosted 핵심 인프라 구성의 일환입니다.

```
┌─────────────────────────────────────────────────────────┐
│              개발 관리 인프라 전체 구성                      │
├──────────────┬──────────────────────────────────────────┤
│ 영역          │ 도구                                      │
├──────────────┼──────────────────────────────────────────┤
│ 소스 관리     │ GitHub (버전 관리 / PR 코드 리뷰)            │
│ 팀원 소통     │ ✅ Mattermost [Self-Hosted] ← 본 문서      │
│ 문서/공유     │ Wiki.js [Self-Hosted]                     │
│ 일정/태스크   │ GitHub Projects (칸반 보드)                 │
│ CI/CD        │ GitHub Actions + Self-hosted Runner        │
│ 상태 알림     │ Mattermost Incoming Webhook               │
└──────────────┴──────────────────────────────────────────┘
```

### 핵심 가치

| 가치 | 설명 |
|------|------|
| 🔒 **데이터 주권** | 대화·파일을 사설 서버에 보관, 완전한 보안 및 영구 기록 |
| 💰 **운영 비용 제로** | Slack, Notion 유료 없이 무제한에 가까운 기능 |
| 🛠️ **TA 친화적** | Docker 기반으로 인프라 직접 제어, 기술 자산 내재화 |
| 📈 **확장성** | Webhook · GitHub Actions 연동으로 자동화 무한 확장 |

---

## 2. 사전 요구사항

### 시스템 환경

```
OS       : Oracle Linux 8.10 (RHEL 계열)
아키텍처  : x86_64
RAM      : 최소 2GB (권장 4GB 이상)
Disk     : 최소 20GB 여유 공간
네트워크  : 포트 8065 외부 접근 가능
```

### 필요 패키지 목록

| 패키지 | 용도 |
|--------|------|
| `docker-ce` | 컨테이너 런타임 엔진 |
| `docker-compose-plugin` | 멀티 컨테이너 오케스트레이션 |
| `curl` | 연결 테스트 및 다운로드 |

---

## 3. Docker 및 Docker Compose 설치

### 3-1. 시스템 업데이트

```bash
sudo dnf update -y
```

### 3-2. Docker 공식 저장소 추가

```bash
sudo dnf install -y dnf-utils
sudo dnf config-manager --add-repo \
  https://download.docker.com/linux/rhel/docker-ce.repo
```

### 3-3. Docker Engine 설치

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### 3-4. Docker 서비스 시작 및 자동 시작 등록

```bash
# 서비스 시작
sudo systemctl start docker

# 부팅 시 자동 시작 설정
sudo systemctl enable docker

# 서비스 상태 확인
sudo systemctl status docker
```

### 3-5. 현재 사용자에게 Docker 권한 부여

> `sudo` 없이 `docker` 명령어를 사용하려면 아래 명령을 실행 후 **재로그인** 필요

```bash
sudo usermod -aG docker $USER
# 재로그인 또는 아래 명령으로 즉시 적용
newgrp docker
```

### 3-6. 설치 버전 확인

```bash
docker --version
docker compose version
```

**예상 출력:**
```
Docker version 26.x.x, build xxxxxxx
Docker Compose version v2.x.x
```

---

## 4. 디렉터리 구조 및 권한 설정

### 4-1. 작업 디렉터리 생성

```bash
# 홈 디렉터리에 mattermost 폴더 생성
mkdir -p ~/mattermost
cd ~/mattermost

# 필수 하위 디렉터리 생성
mkdir -p config data db
```

### 4-2. 디렉터리 구조

```
~/mattermost/
├── docker-compose.yml   ← 컨테이너 정의 파일
├── config/              ← Mattermost 설정 파일 (config.json)
├── data/                ← 업로드 파일, 이미지 등 데이터
└── db/                  ← PostgreSQL 데이터베이스 파일
```

### 4-3. 디렉터리 권한 설정

> Mattermost 컨테이너는 내부적으로 `uid:2000`으로 실행됩니다.  
> 컨테이너가 파일을 읽고 쓸 수 있도록 아래 명령을 **반드시** 실행하세요.

```bash
sudo chmod -R 777 ./config ./data ./db
```

> ⚠️ **보안 참고**: `777`은 초기 구동 및 테스트 환경용입니다.  
> 운영 환경에서는 `chown -R 2000:2000 ./config ./data ./db` 후 `755`로 설정 권장.

---

## 5. docker-compose.yml 구성

### 5-1. 파일 작성

```bash
vi ~/mattermost/docker-compose.yml
```

### 5-2. 전체 내용

```yaml
version: '3'

services:
  # ─────────────────────────────────
  # PostgreSQL 데이터베이스
  # ─────────────────────────────────
  db:
    image: postgres:15-alpine
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=mmuser
      - POSTGRES_PASSWORD=mmuser_password      # ← 운영 시 반드시 변경
      - POSTGRES_DB=mattermost

  # ─────────────────────────────────
  # Mattermost 애플리케이션 서버
  # ─────────────────────────────────
  mattermost:
    image: mattermost/mattermost-team-edition:latest
    restart: unless-stopped
    command: ["/mattermost/bin/mattermost", "server"]
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

### 5-3. 구성 요소 설명

| 항목 | 값 | 설명 |
|------|-----|------|
| `postgres:15-alpine` | DB 이미지 | 경량 Alpine 기반 PostgreSQL 15 |
| `mattermost-team-edition` | 앱 이미지 | Mattermost 무료 팀 에디션 |
| `8065:8065` | 포트 | 호스트:컨테이너 포트 매핑 |
| `POSTGRES_USER` | `mmuser` | DB 접속 계정 |
| `POSTGRES_PASSWORD` | `mmuser_password` | DB 비밀번호 **(운영 시 변경 필수)** |
| `POSTGRES_DB` | `mattermost` | 데이터베이스 이름 |
| `sslmode=disable` | DB 연결 옵션 | 내부 네트워크이므로 SSL 비활성화 |

---

## 6. 방화벽 설정

Oracle Linux 8.10은 기본적으로 `firewalld`가 활성화되어 있습니다.  
Mattermost 포트(8065)를 외부에서 접근할 수 있도록 허용해야 합니다.

```bash
# 포트 8065 영구 허용
sudo firewall-cmd --permanent --add-port=8065/tcp

# 방화벽 규칙 즉시 적용
sudo firewall-cmd --reload

# 적용 확인
sudo firewall-cmd --list-ports
```

**예상 출력:**
```
8065/tcp
```

### 포트 리스닝 확인

```bash
# 서비스 기동 후 확인
sudo netstat -tulpn | grep 8065
```

---

## 7. Mattermost 실행 및 상태 확인

### 7-1. 컨테이너 시작

```bash
cd ~/mattermost

# 백그라운드로 실행 (-d: detached mode)
sudo docker compose up -d
```

### 7-2. 실행 상태 확인

```bash
# 컨테이너 상태 확인
sudo docker compose ps

# 또는 전체 목록 확인
sudo docker ps -a
```

**정상 실행 시 예상 출력:**
```
NAME                       IMAGE                                    STATUS
mattermost-db-1            postgres:15-alpine                       Up X minutes
mattermost-mattermost-1    mattermost/mattermost-team-edition:...   Up X minutes
```

### 7-3. 로그 확인

```bash
# Mattermost 최근 로그 20줄 확인
sudo docker compose logs mattermost | tail -n 20

# 실시간 로그 스트리밍
sudo docker compose logs -f mattermost

# DB 로그 확인
sudo docker compose logs db
```

### 7-4. 설정 관련 로그 필터링

```bash
# 설정(config) 관련 메시지만 확인
sudo docker compose logs mattermost-mattermost-1 2>&1 | grep -i "config"
```

### 7-5. 접속 테스트

```bash
# 로컬에서 서비스 응답 확인
curl -v telnet://localhost:8065

# 서버 IP로 외부 접근 확인 (예: 192.168.93.138)
curl -v telnet://192.168.93.138:8065
```

### 7-6. 웹 브라우저 접속

```
http://<서버_IP>:8065
```

---

## 8. 초기 설정 및 관리자 콘솔

### 8-1. 최초 접속 및 관리자 계정 생성

1. 브라우저에서 `http://<서버_IP>:8065` 접속
2. **"계정 만들기"** 클릭
3. 이메일, 사용자명, 비밀번호 입력 후 가입
4. **첫 번째 가입자가 자동으로 시스템 관리자**가 됩니다

### 8-2. 시스템 관리자 콘솔 접근

```
http://<서버_IP>:8065/admin_console
```

### 8-3. 주요 관리 설정

#### 파일 공유 설정 (File Sharing and Downloads)

```
경로: System Console → Site Configuration → File Sharing and Downloads
URL:  http://<서버_IP>:8065/admin_console/site_config/file_sharing_downloads
```

| 설정 항목 | 권장 값 | 설명 |
|-----------|---------|------|
| Allow File Sharing | `True` | 파일 업로드/다운로드 허용 |

#### 필수 확인 설정 항목

| 메뉴 경로 | 설정 내용 |
|-----------|-----------|
| Site Configuration → Localization | 언어를 한국어로 설정 |
| Authentication → Email | 이메일 인증 활성화 여부 |
| Environment → Web Server | 사이트 URL 설정 (외부 접근 URL 입력) |
| Site Configuration → Users and Teams | 팀 생성 권한, 최대 인원 설정 |

### 8-4. Site URL 설정 (중요)

> 파일 공유, 알림 링크가 올바르게 동작하려면 **Site URL**을 반드시 설정해야 합니다.

```
System Console → Environment → Web Server → Site URL
값: http://192.168.93.138:8065
```

---

## 9. 기본 사용법

### 9-1. 팀(Team) 생성 및 채널(Channel) 구성

```
팀 생성: 상단 메뉴 → "팀 만들기"
채널 생성: 왼쪽 사이드바 → "채널 추가" → 채널명 입력
```

#### 추천 채널 구성 예시

| 채널명 | 목적 |
|--------|------|
| `#general` | 팀 전체 공지 및 일반 소통 |
| `#dev-alert` | GitHub Actions CI/CD 알림 수신 |
| `#code-review` | PR 리뷰 요청 및 피드백 |
| `#incident` | 장애 대응 및 이슈 공유 |

### 9-2. Incoming Webhook 설정 (GitHub Actions 연동)

> CI/CD 빌드 결과를 Mattermost 채널로 자동 수신하는 설정

#### 웹훅 생성

1. **Main Menu** → **Integrations** → **Incoming Webhooks**
2. **Add Incoming Webhook** 클릭
3. 채널 선택 (예: `#dev-alert`)
4. **Save** → **Webhook URL 복사**

```
생성된 URL 예시:
http://192.168.93.138:8065/hooks/xxxxxxxxxxxxxxxxxxxxxxxx
```

#### GitHub Actions에서 Webhook 호출 예시

```yaml
# .github/workflows/notify.yml
- name: Notify Mattermost
  run: |
    curl -i -X POST \
      -H 'Content-Type: application/json' \
      -d '{"text": "✅ **빌드 성공**: `${{ github.repository }}` - `${{ github.ref_name }}`"}' \
      ${{ secrets.MATTERMOST_WEBHOOK_URL }}
```

### 9-3. 메시지 서식 (Markdown 지원)

Mattermost는 메시지 작성 시 Markdown을 지원합니다.

```markdown
**굵게** / *기울임* / ~~취소선~~
`인라인 코드`

```python
# 코드 블록
print("Hello, Mattermost!")
```

> 인용문

- 목록 항목 1
- 목록 항목 2
```

### 9-4. 멘션 및 알림

| 기능 | 사용법 |
|------|--------|
| 개인 멘션 | `@username` |
| 전체 알림 | `@all` 또는 `@channel` |
| 파일 첨부 | 드래그 앤 드롭 또는 클립 아이콘 |
| 이모지 반응 | 메시지 위 마우스 오버 후 😊 클릭 |

---

## 10. 운영 명령어 치트시트

```bash
# ─── 서비스 제어 ───────────────────────────────────────────
sudo docker compose up -d          # 백그라운드 시작
sudo docker compose down           # 완전 종료 (데이터 유지)
sudo docker compose restart        # 전체 재시작
sudo docker compose restart mattermost  # Mattermost만 재시작

# ─── 상태 확인 ────────────────────────────────────────────
sudo docker compose ps             # 컨테이너 상태
sudo docker ps -a                  # 전체 컨테이너 목록
sudo docker compose logs mattermost | tail -n 50  # 최근 로그

# ─── 실시간 모니터링 ──────────────────────────────────────
sudo docker compose logs -f        # 전체 실시간 로그
sudo docker stats                  # CPU/메모리 사용량 실시간

# ─── 네트워크 확인 ────────────────────────────────────────
sudo netstat -tulpn | grep 8065    # 포트 리스닝 확인
sudo netstat -natp | grep 5432     # DB 포트 확인
curl -v telnet://localhost:8065    # 서비스 응답 확인

# ─── 유지보수 ─────────────────────────────────────────────
sudo docker compose pull           # 이미지 최신 버전으로 업데이트
sudo docker image prune -f         # 사용하지 않는 이미지 정리

# ─── 백업 ─────────────────────────────────────────────────
tar -czf mattermost_backup_$(date +%Y%m%d).tar.gz \
  ~/mattermost/config \
  ~/mattermost/data \
  ~/mattermost/db
```

---

## 11. 문제 해결 (Troubleshooting)

### ❌ 컨테이너가 시작되지 않을 때

```bash
# 상세 로그 확인
sudo docker compose logs --tail=50

# DB 컨테이너 상태 우선 확인
sudo docker compose logs db
```

**주요 원인:**
- `config/`, `data/`, `db/` 디렉터리 권한 부족 → `sudo chmod -R 777 ./config ./data ./db`
- 포트 8065가 이미 사용 중 → `sudo netstat -tulpn | grep 8065`로 확인

---

### ❌ 브라우저에서 접속이 안 될 때

```bash
# 1. 컨테이너 실행 여부 확인
sudo docker compose ps

# 2. 포트 바인딩 확인
sudo netstat -natp | grep 8065

# 3. 방화벽 규칙 확인
sudo firewall-cmd --list-ports

# 4. 방화벽 포트 재등록
sudo firewall-cmd --permanent --add-port=8065/tcp
sudo firewall-cmd --reload
```

---

### ❌ DB 연결 오류 (database connection error)

```bash
# docker-compose.yml 환경변수 확인
cat docker-compose.yml | grep -A5 environment

# DB 컨테이너 단독 접속 테스트
sudo docker compose exec db psql -U mmuser -d mattermost
```

**확인 사항:**
- `POSTGRES_USER`, `POSTGRES_PASSWORD`가 `MM_SQLSETTINGS_DATASOURCE`와 일치하는지 확인
- `depends_on: db` 설정이 있는지 확인

---

### ❌ 권한 오류 (Permission denied)

```bash
# config, data, db 디렉터리 권한 재설정
cd ~/mattermost
sudo chmod -R 777 ./config ./data ./db

# 컨테이너 재시작
sudo docker compose restart
```

---

### ❌ 설정 변경 후 적용이 안 될 때

```bash
# Mattermost 컨테이너만 재시작
sudo docker compose restart mattermost

# 또는 전체 재배포 (다운타임 발생)
sudo docker compose down
sudo docker compose up -d
```

---

## 📚 참고 자료

| 자료 | URL |
|------|-----|
| Mattermost 공식 문서 | https://docs.mattermost.com |
| Mattermost Docker 설치 | https://docs.mattermost.com/install/install-docker.html |
| Docker 공식 문서 | https://docs.docker.com |
| GitHub Actions 공식 문서 | https://docs.github.com/en/actions |

---

## 🗺️ 다음 단계

본 가이드 완료 후 아래 구성 요소를 추가하여 완전한 개발 관리 인프라를 구축하세요.

```
✅ Step 1: Mattermost Self-Hosted 설치  ← 현재 문서
⬜ Step 2: Wiki.js Self-Hosted 설치 (기술 문서 위키)
⬜ Step 3: GitHub Actions Self-hosted Runner 등록
⬜ Step 4: GitHub Actions ↔ Mattermost Webhook 연동
⬜ Step 5: GitHub Projects 칸반 보드 구성
```

---

<div align="center">

**본 문서는 소프트웨어 개발 프로젝트의 무료 Self-Hosted 인프라 구축을 목적으로 작성되었습니다.**  
`Oracle Linux 8.10` · `Docker` · `Mattermost Team Edition` · `PostgreSQL 15`

</div>
