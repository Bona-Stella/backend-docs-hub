# 📚 Backend Docs Hub

이 저장소는 프론트엔드 팀과의 협업을 위해 **모든 프로젝트의 백엔드 API 명세, 변경 내역, 접속 정보**를 통합 관리하는 곳입니다.

소스 코드는 포함되어 있지 않으며, 아래 표에서 프로젝트별 **최신 릴리즈 버전**과 **문서**를 확인하실 수 있습니다.

---

## 🚦 Project Status & Release

각 프로젝트의 배포 상태와 API 버전을 한눈에 확인하세요.
`Docs` 링크를 클릭하면 상세 가이드(계정 정보, 변경 로그)로 이동합니다.

| Project Name | Current Version | Stage | Last Updated | Link |
| :--- | :---: | :---: | :---: | :---: |
| **💩 WeLog (위장 질환 위험 예측)** | `v0.8.0` | 🟡 Dev | 2026-05-02 | [Go to Docs](./welog/README.md) |
| **😴 Somnus (수면 사이클 계산)** | `v1.0.0` | 🟢 Live | 2026-01-31 | [Go to Docs](./somnus/README.md) |

> **Stage 설명:**
> - 🟢 **Live**: 운영 서버 배포 완료 (안정적)
> - 🔵 **Dev**: 개발 서버 배포 완료 (배포 중)
> - 🟡 **Dev**: 개발 서버 배포 완료 (테스트 중)
> - ⚪ **Local**: 로컬 개발 중
> - 🟤 **Stop**: 이슈 발생 및 배포 중단
> - 🔴 **End**: 배포 및 관리 종료

---

## 📂 Quick Links (프로젝트별 바로가기)

### 1. [WeLog (장 건강 예보 서비스)](./README.md)
- **주요 기능**: 식단/발병 기록, 질병 발생 확률 예측 알고리즘, 건강 리포트, 검색 및 자동완성
- **기술 스택**: Java 25, Spring Boot 4.0.5, PostgreSQL 17, Redis 7.2, MinIO
- **보안 기능**: JWT + OAuth2, Rate Limiting, WEBP 이미지 전용
- **비고**: 검색 API, Rate Limiting, OAuth2 통합 완료. Docker 배포 환경 구축.

### 2. [Somnus (수면 사이클 비서)](./somnus/README.md)
- **주요 기능**: REM 수면 주기 계산, 기상/취침 시간 추천, 타이머 설정
- **비고**: UI/UX 디자인(Figma) v1.0이 업데이트 되었습니다.

### 3. 프로젝트 폴더 구조
```text
📦 Project Root
 ┣ 📂 somnus
 ┃ ┣ 📜 README.md (Somnus)
 ┃ ┗ 📜 SPECIFICATION.md
 ┗ 📂 welog
   ┣ 📜 BACKEND_SPEC_v1.0.md
   ┣ 📜 DESIGN_LOG_V1.0.md
   ┣ 📜 ERD_V1.0.md
   ┣ 📜 FUNCTIONAL_SPEC_v1.0.md
   ┗ 📜 README.md (Welog)
```
---

## 📢 공통 공지사항 (Notice)
- **[2026-05-02]** `WeLog` API Server Dev 외부 API 설정 및 배포 완료.
- **[2026-04-14]** `WeLog` Welog-PWA, API Server Dev 서버 배포 완료.
- **[2026-04-14]** `WeLog` docker-compose.yml 재구성. PWA, API, MinIO 분리 배포 환경 구축.

---

### 📞 Contact
API 관련 문의나 이슈는 **Issues** 탭을 이용해주시거나 아래로 연락주세요.
- **Backend Dev**: `bona.stella.91@google.com`
