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
| **Cache/NoSQL** | **Redis + Redisson** | 7.2 (Caching, Refresh Token, Rate Limiting) |
| **ORM** | **Spring Data JPA** | + **QueryDSL 5.0.0** (jakarta classifier) |
| **Auth** | **JWT + OAuth2** | jjwt 0.13.0 (Access/Refresh Token) + Google OAuth2 |
| **Security** | **Spring Security** | 7.0.2 + Bucket4j + Resilience4j |
| **Storage** | **MinIO** | S3 Compatible Object Storage |
| **Docs** | **Swagger/OpenAPI** | Springdoc-openapi-ui 3.0.2 (API 문서 자동화) |
| **Testing** | **Testcontainers** | PostgreSQL, Redis 통합 테스트 |
| **Deploy** | **Docker** | Docker Compose (PWA, API, MinIO 오케스트레이션) |

---

## 2. 📂 System Architecture (Package Structure)
유지보수와 도메인 응집도를 높이기 위한 **도메인형 패키지 구조**를 따릅니다.

```text
com.github.stella.welog
├── domain              // 핵심 비즈니스 로직 (도메인별 분리)
│   ├── admin           // 관리자 기능
│   ├── analysis        // 분석 알고리즘 (Risk Calculation, Search)
│   ├── auth            // 인증 및 인가 (Login, JWT, OAuth2)
│   ├── feed            // 피드 조회
│   ├── meal            // 식사 기록 (Meal, MealDetail)
│   ├── member          // 사용자 (User, MemberDisease)
│   └── symptom         // 증상/발병 기록 (Symptom)
├── global              // 전역 공통 모듈
│   ├── config          // 설정 (Security, Swagger, Redis, Rate Limiting 등)
│   ├── exception       // Global Exception Handler
│   ├── ratelimit       // Rate Limiting (Bucket4j, Redisson)
│   ├── security       // JWT Provider, Security Filter, OAuth2
│   └── util            // DateUtil, ImageUtil, S3Service 등
└── infra               // 외부 인프라 연동
    └── s3              // 이미지 스토리지 (MinIO)
```

## 3. 🧠 Core Algorithm (Risk Scoring Logic)
WeLog의 핵심 기능인 '발병 확률'을 계산하는 로직입니다.
단순 합산 방식이 아닌, **여집합의 확률(Probability of Union)**을 사용하여 수학적 정합성을 보장합니다.

### 4.1 Basic Concept
> **Logic:** "현재 소화 중인 모든 음식/태그/식당을 독립적인 위험 인자로 간주하고, 이들로부터 **모두 살아남을(발병하지 않을) 확률을 구해 1에서 뺀 값**을 발병 확률로 정의한다."

이 방식을 통해 위험 요소가 많아질수록 확률은 100%에 수렴하며, 단순 합산 시 100%를 초과하는 오류를 방지합니다.

### 4.2 Calculation Formula  
$$ P(TotalRisk) = 1 - \prod_{i=1}^{n} (1 - P(Factor_i)) $$  

**[최종 발병 확률 공식]**  
```text
Final_Risk = 1 - ( (1 - Risk_Factor_1) * (1 - Risk_Factor_2) * ... * (1 - Risk_Factor_n) )
```

1. **Scope:** `NOW()` 기준 `User.digestion_time`(Default 18h) 이내에 섭취한 모든 `FoodLog`, `Tag`, `Restaurant` 수집.
2. **Individual Risk (개별 위험도):** 각 요소의 과거 데이터 조회.
    * `Risk_Factor = (해당 요소 먹고 아픈 횟수) / (해당 요소 먹은 총 횟수)`
    * *데이터가 없거나 적을 경우(Cold Start), 가중치를 0으로 처리하거나 기본값 적용.*
3. **Aggregation:** 위 공식을 적용하여 최종 확률(%) 도출.

### 4.3 Simulation Example
**상황:** 사용자가 최근 18시간 내에 **[스타벅스]**에서 **[매운 라떼]**를 섭취함.
*   요소 1 (**매운맛**): 과거 기록상 위험도 **0.5 (50%)** -> 생존확률 0.5
*   요소 2 (**라떼**): 과거 기록상 위험도 **0.2 (20%)** -> 생존확률 0.8
*   요소 3 (**스타벅스**): 과거 기록상 위험도 **0.1 (10%)** -> 생존확률 0.9

**계산:**
$Risk = 1 - (0.5 \times 0.8 \times 0.9) = 0.64$, 결과값은 **64%**
```text
1. 생존 확률 곱셈 (모두 통과할 확률)
   Survive_Prob = 0.5 * 0.8 * 0.9 = 0.36 (36%)

2. 최종 발병 확률 (1에서 뺌)
   Final_Risk = 1.0 - 0.36 = 0.64

3. 결과
   Result = 64%
```

---
## 4. 📡 API Specification Strategy
Restful API 표준을 준수하며, 모든 응답은 공통 포맷(ApiResponse)을 사용합니다.
### 4.1 Common Response Format
```JSON
// Success
{
  "success": true,
  "status": 200,
  "code": "OK",
  "message": "Success",
  "data": {
    "id": 1,
    "title": "게시글 제목"
  },
  "timestamp": "2025-01-01T12:30:02Z",
  "path": "/api/v1/posts/1"
}

// Fail
{
  "success": false,
  "status": 400,
  "code": "INVALID_INPUT",
  "message": "Request validation failed.",
  "timestamp": "2025-01-01T12:30:02Z",
  "path": "/api/v1/users"
}
```
### 4.2 Key Endpoints
| Domain | Method | URI | Description | Rate Limit |
| :--- | :--- | :--- | :--- | :--- |
| **Auth** | `POST` | `/api/v1/auth/signup` | 회원가입 | - |
| | `POST` | `/api/v1/auth/login` | 로그인 (Access + Refresh Token 발급) | - |
| | `POST` | `/api/v1/auth/reissue` | 토큰 재발급 | - |
| | `POST` | `/api/v1/auth/oauth2/google` | Google OAuth2 로그인 | - |
| **User** | `GET` | `/api/v1/members/me` | 내 정보 및 설정 조회 | - |
| | `PUT` | `/api/v1/members/me` | 사용자 정보 수정 | - |
| **Meal** | `POST` | `/api/v1/meals` | 식사 기록 저장 (이미지, 태그 포함) | 3회/분 (사용자당) |
| | `GET` | `/api/v1/meals` | 날짜별 식사 기록 조회 (Feed/Calendar) | - |
| | `GET` | `/api/v1/meals/{id}` | 식사 상세 조회 | - |
| | `PATCH` | `/api/v1/meals/{id}` | 식사 기록 수정 | 3회/분 (사용자당) |
| | `DELETE` | `/api/v1/meals/{id}` | 식사 기록 삭제 | - |
| **Symptom** | `POST` | `/api/v1/symptoms` | 증상 기록 저장 (10분 throttle + 일일 10회) | 10회/일 (사용자당) |
| | `DELETE` | `/api/v1/symptoms/{id}` | 증상 기록 삭제 | - |
| **Search** | `GET` | `/api/v1/search/suggest` | 검색어 자동완성 (최소 2자, 최대 15자) | 40회/분 (IP당) |
| | `GET` | `/api/v1/search` | 검색 결과 조회 (페이징, Redis 캐시) | 10회/분 (IP당) |
| | `POST` | `/api/v1/search/recent` | 최근 검색어 저장 | - |
| | `DELETE` | `/api/v1/search/recent` | 최근 검색어 삭제 | - |
| **Analysis** | `GET` | `/api/v1/analysis/home` | **[Main]** 오늘/내일 발병 확률 예측 및 Top3 위험 요인 | - |
| | `GET` | `/api/v1/analysis/reports` | 상세 분석 차트 데이터 (메뉴별/태그별/식당별 위험도) | - |
| **Feed** | `GET` | `/api/v1/feed` | 통합 피드 조회 (식사 + 증상) | - |
## 5. 🚀 Deployment Strategy

* **Target Environment:** Private Ubuntu Server (High Spec / 32GB RAM)
* **Orchestration Tool:** Docker Compose
* **Container Stack:**
  1.  **`welog-pwa`**: Frontend PWA Application (Port: 80)
  2.  **`welog-api`**: Spring Boot Web Application (Port: 8080)
  3.  **`welog-postgres`**: PostgreSQL Persistence Storage (Port: 5432)
  4.  **`welog-redis`**: In-memory Cache & Session Store (Port: 6379)
  5.  **`welog-minio`**: Object Storage for Images (Port: 9000)
  6.  **`welog-minio-console`**: MinIO Web Console (Port: 9001)
* **Reverse Proxy:** Nginx with SSL Termination (Port: 80/443)
* **Health Checks:** Spring Boot Actuator (`/actuator/health`)
* **Rate Limit:** Resilience4j - Circuit Breaker & Bucket4j
