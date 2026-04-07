# 💩 WeLog Backend Guide

**WeLog 프로젝트**의 프론트엔드 연동을 위한 백엔드 가이드 문서입니다.
서버 접속 정보, 테스트 계정, 그리고 **API 변경 내역(Changelog)**을 이곳에서 관리합니다.

> **Current Version**: `v1.0.0`
> **Last Updated**: 2026-01-05

---

## 1. 🌐 Server Environments (서버 접속 정보)

API 요청을 보낼 기본 Base URL입니다.

| 환경 (Env) | 상태 (Status) | Base URL | 비고 |
| :--- | :---: | :--- | :--- |
| **Local** | 💻 My PC | `http://localhost:8080` | 로컬 개발용 |
| **Dev** | 🟡 Running | `http://welog.comma.my` | 개발 서버 (AWS EC2) |
| **Prod** | 🔴 Stop | `https://api.welog.com` | 운영 서버 (준비 중) |

---

## 2. 🔑 Test Accounts (테스트 계정)

개발 및 테스트 시 아래 계정을 사용해주세요. **비밀번호는 주기적으로 변경될 수 있습니다.**

| Role | Username (ID) | Password | 설명 |
| :--- | :--- | :--- | :--- |
| **Admin** | `admin@welog.com` | `admin1234!` | 관리자 권한 (모든 글 삭제 가능) |
| **User A** | `user1@welog.com` | `Passowrd1234!` | 일반 사용자 (게시글 작성용) |
| **User B** | `user2@welog.com` | `Passowrd1234!` | 일반 사용자 (게시글 작성용) |
| **User C** | `user3@welog.com` | `Passowrd1234!` | 일반 사용자 (게시글 작성용) |
| **User D** | `user4@welog.com` | `Passowrd1234!` | 일반 사용자 (게시글 작성용) |

> ⚠️ **주의**: `test01` 계정의 데이터는 매일 자정에 초기화됩니다.

---

## 3. 🚀 API Change Log (변경 내역)

프론트엔드 연동에 영향을 주는 **API 변경 사항(CRUD)**을 최신순으로 기록합니다.

#### [v1.0.0] - 2026-01-05
- **✨ New Features**
    - `POST /api/logs`: 로그 작성 기능 구현 완료
    - `GET /api/logs`: 로그 목록 조회 (페이지네이션 적용, `page`, `size` 파라미터 필요)
- **🛠 Updates**
    - `POST /auth/login`: 응답 값에 `refreshToken` 필드 추가됨
- **🔥 Removals**
    - `GET /api/temp`: 임시 테스트용 API 삭제

#### [v0.9.0] - 2026-01-01
- **✨ New Features**
    - 회원가입 (`/auth/signup`), 로그인 (`/auth/login`) 기본 로직 구현
- **🐛 Bug Fixes**
    - 이메일 중복 체크 시 500 에러 발생하던 문제 수정

---

## 4. 📚 API Specification (상세 명세)

요청 파라미터(Request Body) 및 응답 값(Response)에 대한 상세 스펙은 아래 링크를 참고하세요.

- **Swagger UI**: [http://dev-api.welog.com/swagger-ui.html](http://localhost:8080) (접속 불가 시 요청 바람)
- **Postman**: (초대 링크가 있다면 여기에 첨부)

---

## 5. 🗣 Q&A & Issues

개발 중 발생하는 API 오류나 문의 사항은 **[Issues 탭](../../issues)**에 남겨주세요.
급한 건은 메일/카톡으로 연락 바랍니다. bona.stella.91@gmail.com
