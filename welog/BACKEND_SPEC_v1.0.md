# 🛠️ WeLog - V1.0 Backend Technical Spec

> **WeLog** 백엔드 서버의 기술 스택, 아키텍처, 데이터베이스 스키마 및 API 명세를 정의합니다.
> **Target:** High-spec Private Server Deployment (Ubuntu Server 24.04)

---

## 1. 🏗️ Tech Stack (기술 스택)
안정성과 생산성을 최우선으로 고려한 **Java 25 (LTS) & Spring Boot 4.0** 기반의 모던 스택입니다.

| Category | Technology | Version / Note |
| :--- | :--- | :--- |
| **Language** | **Java** | **25.0.2 Eclipse Adoptium (LTS)** (Virtual Threads 활성화) |
| **Framework** | **Spring Boot** | **4.0.5** (Latest Stable) |
| **Build Tool** | **Gradle** | 9.4.0 (Groovy DSL) |
| **Database** | **PostgreSQL** | 17 (Main DB) |
| **Cache/NoSQL** | **Redis + Redisson** | 7.2 (Caching, Refresh Token, Rate Limiting, Throttling) |
| **ORM** | **Spring Data JPA** | Hibernate 6.6+ |
| **Auth** | **JWT + OAuth2** | jjwt 0.13.0 + Google OAuth2 |
| **Resilience** | **Resilience4j** | Circuit Breaker & Retry (Email Service) |
| **Security** | **Spring Security** | 7.0.2 + Bucket4j (Rate Limiting) |
| **Storage** | **MinIO** | S3 Compatible Object Storage |
| **Docs** | **Swagger/OpenAPI** | Springdoc-openapi-ui 3.0.2 |
| **Testing** | **Testcontainers** | PostgreSQL, Redis 통합 테스트 |

---

## 2. 📂 System Architecture (Package Structure)
유지보수와 도메인 응집도를 높이기 위한 **도메인형 패키지 구조**를 따릅니다.

```text
com.github.stella.welog
├── domain              // 핵심 비즈니스 로직 (도메인별 분리)
│   ├── admin           // 관리자 기능
│   ├── analysis        // 분석 알고리즘 (Risk Calculation, Batch, Search)
│   ├── auth            // 인증 및 인가 (Login, JWT, OAuth2)
│   ├── feed            // 피드 조회 (식사 + 증상 통합 피드)
│   ├── meal            // 식사 기록 (Meal, MealDetail, Image)
│   ├── member          // 사용자 (User, MemberDisease, Password)
│   └── symptom         // 증상/발병 기록 (Symptom)
├── global              // 전역 공통 모듈
│   ├── config          // 설정 (Security, Swagger, Redis, Rate Limiting 등)
│   ├── exception       // Global Exception Handler & ErrorResponse
│   ├── ratelimit       // Rate Limiting (Bucket4j, Redisson Aspect)
│   ├── security        // JWT Provider, Security Filter, OAuth2
│   └── util            // DateUtil, ImageUtil, S3Service, ServletUtil
└── infra               // 외부 인프라 연동
    └── s3              // 이미지 스토리지 (MinIO/S3 SDK)
```

---

## 3. 🧠 Core Algorithm (Analysis & Risk Logic)
WeLog는 비동기 배치 분석과 실시간 데이터 결합을 통해 정밀한 위험도를 산출합니다.

### 3.1 Hybrid Analysis Strategy
*   **Batch Layer:** 1시간 간격으로 모든 회원의 전체 로그를 재집계(`StatsAggregator`)하여 `FactorScore`(요인별 위험도)를 계산하고 DB에 저장합니다.
*   **Real-time Layer:** 메인 화면 조회 시, 사용자의 `riskCriteriaTime`(기본 18시간) 내 식사 기록들과 DB에 저장된 `FactorScore`를 결합하여 현재 위험도를 즉시 산출합니다.

### 3.2 Calculation Formula (Probability of Union)
$$ P(TotalRisk) = 1 - \prod_{i=1}^{n} (1 - P(MealRisk_i)) $$

1.  **Individual Factor Risk (개별 요인 위험도):**
    `Risk = (해당 요인 섭취 후 증상 발생 횟수) / (전체 섭취 횟수)` (0.0 ~ 1.0)
2.  **Meal Risk (식사별 위험도):**
    한 식사에 포함된 여러 요인 중 **가장 높은 위험도(Max)**를 가진 요인을 해당 식사의 대표값으로 사용합니다.
    $P(MealRisk) = \max(Risk_{factor1}, Risk_{factor2}, ...)$
3.  **Total Risk (종합 위험도):**
    분석 윈도우 내 모든 식사들이 각각 "안전할 확률"($1 - MealRisk$)을 모두 곱한 뒤, 이를 1에서 빼서 최종 발병 확률을 구합니다.

### 3.3 Simulation Example
**상황:** 최근 18시간 내에 두 번의 식사를 함.
*   **식사 1 (점심):** [제육볶음(0.5)], [공기밥(0.01)], [식당A(0.1)] 섭취
    *   대표 위험도: $\max(0.5, 0.01, 0.1) = \mathbf{0.5}$
*   **식사 2 (간식):** [우유(0.3)], [초코쿠키(0.1)] 섭취
    *   대표 위험도: $\max(0.3, 0.1) = \mathbf{0.3}$

**최종 계산:**
1.  각 식사가 안전할 확률: $(1 - 0.5) = 0.5$, $(1 - 0.3) = 0.7$
2.  모두 안전할 확률 (생존 확률): $0.5 \times 0.7 = 0.35$ (35%)
3.  최종 발병 확률: $1 - 0.35 = 0.65$ (**65%**)

---

## 4. 📡 API Specification Strategy
Restful API 표준을 준수하며, 모든 응답은 공통 포맷(`ApiResponse` / `ErrorResponse`)을 사용합니다.

### 4.1 Common Response Format
#### [Success Response]
```JSON
{
  "success": true,
  "status": 200,
  "code": "OK",
  "message": "Success",
  "data": {
    "id": 123,
    "name": "WeLog User"
  },
  "timestamp": "2025-01-01T12:30:00+09:00"
}
```

#### [Error Response]
```JSON
{
  "success": false,
  "status": 429,
  "code": "TOO_MANY_REQUESTS",
  "message": "너무 많은 요청이 발생했습니다. 59초 후에 다시 시도해주세요.",
  "timestamp": "2025-01-01T12:30:05+09:00",
  "path": "/api/v1/meals",
  "traceId": "a1b2c3d4e5f6g7h8"
}
```

### 4.2 Key Endpoints & Rate Limits
| Domain | Method | URI | Description | Rate Limit Policy |
| :--- | :--- | :--- | :--- | :--- |
| **Auth** | `POST` | `/api/v1/auth/login` | 로그인 (Access/Refresh) | 10회 실패 시 30분 잠금 |
| | `POST` | `/api/v1/auth/password/forgot` | 비밀번호 찾기 메일 | 이메일별 2분 쿨다운 / IP별 일 10회 / 이메일별 일 3회 |
| | `POST` | `/api/v1/auth/reissue` | 토큰 재발급 | 분당 10회 / 일일 100회 |
| **Member** | `GET` | `/api/v1/members/me` | 내 정보 조회 | - |
| | `POST` | `/api/v1/members/check-nickname` | 닉네임 확인 | - |
| | `POST` | `/api/v1/members/password/verify` | 비밀번호 확인 | 분당 3회 / 5회 실패 시 15분 잠금 |
| | `DELETE` | `/api/v1/members/withdraw` | 회원 탈퇴 | 일일 1회 (엄격) |
| **Meal** | `POST` | `/api/v1/meals` | 식사 기록 저장 | 분당 3회 / 일일 30회 |
| **Symptom** | `POST` | `/api/v1/symptoms` | 증상 기록 ("아파요") | 10분 1회 (스로틀) / 일일 10회 |
| **Feed** | `GET` | `/api/v1/feeds` | 통합 피드 조회 | 분당 60회 (IP 기준) |
| **Search** | `GET` | `/api/v1/search/suggest` | 검색어 자동완성 | 분당 40회 (IP 기준) |
| | `GET` | `/api/v1/search` | 검색 결과 조회 | 분당 10회 (IP 기준) |
| **Analysis** | `GET` | `/api/v1/analysis/home` | 실시간 발병 확률 예측 | 분당 60회 (IP 기준) |
| | `GET` | `/api/v1/analysis/reports` | 도메인별 통계 리포트 | 분당 60회 (IP 기준) |

---

## 5. 🚀 Deployment & Resilience Strategy

### 5.1 Infrastructure Stack (Docker Compose)
* **Target Environment:** Private Ubuntu Server (High Spec / 32GB RAM)
* **Orchestration Tool:** Docker Compose
* **Reverse Proxy:** Nginx with SSL Termination (Port: 80/443)

#### [Container Stack]
1.  **`welog-pwa`**: Frontend PWA Application (Port: 80)
2.  **`welog-api`**: Spring Boot 서버 (Port: 8080). **Java 25 Virtual Threads** 활성화로 고성능 I/O 처리.
3.  **`welog-postgres`**: PostgreSQL 17. 모든 비즈니스 데이터 저장.
4.  **`welog-redis`**: Redis 7. JWT Refresh Token, Rate Limit 버킷, 분석 캐시 관리.
5.  **`welog-minio`**: S3 호환 오브젝트 스토리지. 식사 사진 및 프로필 이미지 저장 (Port: 9000).
6.  **`welog-minio-console`**: MinIO Web Console (Port: 9001).

### 5.2 Resilience (Resilience4j)
*   **Email Circuit Breaker:** 외부 메일 전송 API(Resend) 장애 시 시스템 전체 지연을 방지하기 위해 50% 실패율 도달 시 30초간 차단.
*   **Email Retry:** 네트워크 일시 오류 대응을 위해 최대 3회 재시도 (지수 백오프: 2s, 4s, 8s).
*   **Actuator Monitoring:** `/actuator/health` 엔드포인트를 통해 각 인프라 요소의 상태를 실시간 체크.
