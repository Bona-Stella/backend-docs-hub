# 💩 WeLog Backend Guide

**WeLog 프로젝트**의 프론트엔드 연동을 위한 백엔드 가이드 문서입니다.
서버 접속 정보, 테스트 계정, 그리고 **API 변경 내역(Changelog)**을 이곳에서 관리합니다.

> **Current Version**: `v0.8.0`
> **Last Updated**: 2026-04-14

---

## 1. 🌐 Server Environments (서버 접속 정보)

API 요청을 보낼 기본 Base URL입니다.

| 환경 (Env) | 상태 (Status) | Base URL | 비고 |
| :--- | :---: | :--- | :--- |
| **Local** | 💻 My PC | `http://localhost:8080` | 로컬 개발용 |
| **Dev-API** | 🟡 Running | `http://api-welog.comma.my` | 개발 서버 (Docker) |
| **Dev-PWA** | 🟡 Running | `http://welog.comma.my` | 프론트엔드 (Docker) |
| **Dev-MinIO** | 🟡 Running | `http://s3-welog.comma.my` | 이미지 서빙 (Docker) |
| **Prod** | 🔴 Stop | `https://api.welog.com` | 운영 서버 (준비 중) |

---

## 2. 🔑 Test Accounts (테스트 계정)

개발 및 테스트 시 아래 계정을 사용해주세요. **비밀번호는 주기적으로 변경될 수 있습니다.**

| Role | Username (ID) | Password | 설명 |
| :--- | :--- | :--- | :--- |
| **Swagger** | `admin` | `문의주세요.` | Swagger Docs 사용자 |
| **User A** | `user1@welog.com` | `Password123!` | 일반 사용자 (야식광팬/야식 위주) |
| **User B** | `user2@welog.com` | `Password123!` | 일반 사용자 (헬시걸/건강식 위주) |
| **User C** | `user3@welog.com` | `Password123!` | 일반 사용자 (직장인A/직장인 위주) |
| **User D** | `user4@welog.com` | `Password123!` | 일반 사용자 (비건러버/비건 위주) |
| **OAuth2** | Google Login | - | Google 소셜 로그인 지원 |

> ⚠️ **주의**: 테스트 계정을 포함한 모든 데이터는 `매일 새벽 4시`마다 주기적으로 초기화됩니다.
> 🔒 **보안**: Rate Limiting 적용 (검색 40회/분, 식사기록 3회/분, 증상기록 10회/일 등)

---

## 3. 🚀 API Change Log (변경 내역)

프론트엔드 연동에 영향을 주는 **API 변경 사항(CRUD)**을 최신순으로 기록합니다.

#### [v0.8.0] - 2026-04-14
- **✨ New Features**
    - `GET /api/v1/search/suggest`: 검색어 자동완성 기능 (Rate Limit: 40회/분)
    - `GET /api/v1/search`: 검색 결과 조회 (Rate Limit: 10회/분, Redis 캐싱)
    - `POST /api/v1/search/recent`: 최근 검색어 저장
    - `DELETE /api/v1/search/recent`: 최근 검색어 삭제
    - OAuth2 Google 소셜 로그인 연동
    - Bucket4j 기반 Rate Limiting 적용 (전체 API)
    - WEBP 이미지 전용 업로드 보안 강화
- **🛠 Updates**
    - `POST /api/v1/meals`: 이미지 검증 로직 강화 (WEBP 전용)
    - `POST /api/v1/symptoms`: 10분 throttle + 일일 10회 제한 적용
    - JWT 토큰 만료 시간 조정 (Access: 30분, Refresh: 14일)
- **🔧 Infrastructure**
    - Docker Compose 완전 재구성 (PWA, API, MinIO 분리)
    - Resilience4j Circuit Breaker 적용
    - Testcontainers 통합 테스트 환경 구축

#### [v0.7.0]
- **✨ New Features**
    - Google OAuth2 Client 연동
    - Resend 이메일 서비스 연동 (회원가입/비밀번호 찾기)
- **🛠 Updates**
    - Spring Security 7.0.2 업그레이드

#### [v0.6.0]
- **✨ New Features**
    - `POST /api/v1/meals`: 식사 기록 기능 구현 완료
    - `GET /api/v1/meals`: 식사 목록 조회 (페이지네이션 적용)
    - `POST /api/v1/symptoms`: 증상 기록 기능
    - `GET /api/v1/analysis/home`: 메인 화면 분석 데이터
    - `GET /api/v1/analysis/reports`: 통계 리포트
- **🛠 Updates**
    - `POST /auth/login`: 응답 값에 `refreshToken` 필드 추가됨
- **🔥 Removals**
    - `GET /api/temp`: 임시 테스트용 API 삭제

---

## 4. 📚 API Specification (상세 명세)

요청 파라미터(Request Body) 및 응답 값(Response)에 대한 상세 스펙은 아래 링크를 참고하세요.

- **Swagger UI**: [http://api-welog.comma.my/docs](http://localhost:8080/docs) (로컬 개발 시)
- **API Docs**: [http://api-welog.comma.my/api-docs](http://localhost:8080/api-docs) (OpenAPI JSON)
- **Health Check**: [http://api-welog.comma.my/actuator/health](http://localhost:8080/actuator/health)

---

## 5. 🗣 Q&A & Issues

## 6. 🛠️ Tech Stack (기술 스택)

### Backend
- **Language**: Java 25 (LTS) - Virtual Threads 활용
- **Framework**: Spring Boot 4.0.5, Spring Security 7.0.2
- **Database**: PostgreSQL 17, Redis 7.2 + Redisson
- **Storage**: MinIO (S3 Compatible)
- **Build**: Gradle 9.4.0

### Security & Performance
- **Authentication**: JWT (Access/Refresh Token) + OAuth2 (Google)
- **Rate Limiting**: Bucket4j + Redisson
- **Circuit Breaker**: Resilience4j
- **Image Processing**: WEBP 전용, S3 SDK

---

## 7. 🗣 Q&A & Issues

개발 중 발생하는 API 오류나 문의 사항은 **[Issues 탭](../../issues)**에 남겨주세요.
급한 건은 메일로 연락 바랍니다. bona.stella.91@gmail.com

### 📝 개발 참고사항
- **Rate Limiting**: 각 API별 제한 있음 (상단 테스트 계정 참고)
- **Image Upload**: WEBP 형식으로 변환 (content-type 또는 .webp 확장자)
- **Search API**: 최소 2자, 최대 15자, 와일드카드(%,*) 차단
- **Pagination**: 모든 목록 조회 API에 페이지네이션 적용됨
