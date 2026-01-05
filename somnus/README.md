# 😴 Somnus Backend Guide

**Somnus(수면 사이클)** 프로젝트의 프론트엔드 연동을 위한 백엔드 가이드 문서입니다.
서버 접속 정보, 테스트 계정, 그리고 **API 변경 내역(Changelog)**을 이곳에서 관리합니다.

> **Current Version**: `v0.1.0`
> **Last Updated**: 2026-01-05

---

## 1. 🌐 Server Environments (서버 접속 정보)

API 요청을 보낼 기본 Base URL입니다.

| 환경 (Env) | 상태 (Status) | Base URL | 비고 |
| :--- | :---: | :--- | :--- |
| **Local** | 💻 My PC | `http://localhost:8080` | 로컬 개발용 |
| **Dev** | 🟡 Running | `http://dev-api.somnus.com` | 개발 서버 (기능 테스트 중) |
| **Prod** | 🔴 Stop | `https://api.somnus.com` | 운영 서버 (준비 중) |

---

## 2. 🔑 Test Accounts (테스트 계정)

개발 및 테스트 시 아래 계정을 사용해주세요. **비밀번호는 주기적으로 변경될 수 있습니다.**

| Role | Username (ID) | Password | 설명 |
| :--- | :--- | :--- | :--- |
| **Admin** | `admin@somnus.com` | `admin1234!` | 관리자 (유저 통계 조회) |
| **User A** | `sleeper01` | `test1234` | 일반 사용자 (수면 기록 보유) |
| **User B** | `newbie` | `test1234` | 일반 사용자 (기록 없음/초기화면용) |
| **Guest** | - | - | 비로그인 상태 (단순 계산기 이용) |

> ⚠️ **주의**: `sleeper01` 계정의 수면 로그는 매주 월요일 초기화됩니다.

---

## 3. 🚀 API Change Log (변경 내역)

프론트엔드 연동에 영향을 주는 **API 변경 사항(CRUD 및 계산 로직)**을 최신순으로 기록합니다.

#### [v0.1.0] - 2026-01-05
- **✨ New Features (Core Logic)**
    - `GET /api/calculator/wake-up`: 기상 시간 추천 API (현재 시각 기준)
        - *Logic*: 현재 시간 + (90분 * 사이클 횟수) + 잠드는 시간(15분)
    - `GET /api/calculator/bed-time`: 취침 시간 추천 API (목표 기상 시간 기준)
    - `POST /api/records`: 수면 시간(입면~기상) 기록 저장
- **🛠 Updates**
    - `Member` 엔티티에 `targetWakeUpTime` (목표 기상 시간) 필드 추가
- **🐛 Bug Fixes**
    - 자정(00:00) 넘어가는 시간 계산 시 날짜가 하루 밀리는 버그 수정

---

## 4. 📚 API Specification (상세 명세)

요청 파라미터(Request Body) 및 응답 값(Response)에 대한 상세 스펙은 아래 링크를 참고하세요.

- **Swagger UI**: [http://dev-api.somnus.com/swagger-ui.html](http://localhost:8080) (접속 불가 시 요청 바람)
- **Postman**: (초대 링크가 있다면 여기에 첨부)

---

## 5. 🗣 Q&A & Issues

개발 중 발생하는 API 오류나 문의 사항은 **[Issues 탭](../../issues)**에 남겨주세요.
급한 건은 메일/카톡으로 연락 바랍니다. bona.stella.91@gmail.com
