---
title: 기술 자산 관리 홈
description: 
published: true
date: 2026-04-13T06:42:22.718Z
tags: 
editor: markdown
dateCreated: 2026-04-13T06:42:22.718Z
---

# Wiki.js 구축 및 Git 연동 가이드

본 문서는 RHEL/Oracle Linux 환경에서 Docker를 이용해 Wiki.js를 구축하고, GitHub 저장소와 실시간 동기화를 설정한 과정을 기록한 문서입니다.

---

## 1. 인프라 구성 (Docker Compose)
PostgreSQL 15 버전을 백엔드 데이터베이스로 사용하여 컨테이너 기반으로 구축되었습니다.

* **포트**: 3000 (Wiki.js 기본 포트)
* **데이터베이스**: PostgreSQL 15-alpine
* **방화벽 설정**: `firewall-cmd`를 통해 3000/tcp 포트 개방 완료

---

## 2. 주요 설정 사항

### 2.1 Localization (한글화)
- `Site > Locale` 메뉴에서 **Korean** 언어팩을 다운로드하여 시스템 기본 언어를 한국어로 변경하였습니다.

### 2.2 Git Storage 연동 (Docs as Code)
가장 핵심적인 설정으로, 작성된 모든 문서는 GitHub 저장소에 자동 백업됩니다.
- **대상 저장소**: `kji777kji777/kji777kji777.github.io`
- **인증 방식**: Fine-grained Personal Access Token (Basic Auth)
- **동기화 브랜치**: `main`
- **주요 특징**: 위키에서 저장 시 GitHub에 `.md` 파일이 자동 생성 및 커밋됩니다.

---

## 3. Wiki.js 활용 팁 (TA 관점)

1. **에디터 선택**: 기술 문서의 구조화를 위해 **Markdown 에디터** 사용을 지향합니다.
2. **경로 구조화**: `/` 슬래시를 사용하여 `Project/Phase1/Guide`와 같이 논리적인 폴더 구조를 생성합니다.
3. **자산 내재화**: 서버 장애 시에도 GitHub에 마크다운 원본이 보존되므로 기술 자산의 영구 보존이 가능합니다.

---
> **작성 정보**
> - 작성자: Kim Jong-in (TA)
> - 작성일: 2026. 04. 13.
> - 프로젝트: 인프라 기술 위키 구축 테스트