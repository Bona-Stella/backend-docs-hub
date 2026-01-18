# 🛠️ WeLog - V1.0 Backend Technical Spec

> **WeLog** 백엔드 서버의 기술 스택, 아키텍처, 데이터베이스 스키마 및 API 명세를 정의합니다.  
> **Target:** High-spec Private Server Deployment (Ubuntu Server 24.04)

---

## 1. 🏗️ Tech Stack (기술 스택)
안정성과 생산성을 최우선으로 고려한 **Java 21 (LTS) & Spring Boot 3.4** 기반의 모던 스택입니다.

| Category | Technology | Version / Note |
| :--- | :--- | :--- |
| **Language** | **Java** | **21.0.9 Eclipse Adoptium (LTS)** (Virtual Threads 활성화) |
| **Framework** | **Spring Boot** | **4.0.1** (Latest Stable) |
| **Build Tool** | **Gradle** | 2.2.20/4.0.28 (Kotlin/Groovy DSL) |
| **Database** | **PostgreSQL** | 16.x or 17.x (Main DB) |
| **Cache/NoSQL** | **Redis** | 7.x or 8.x (Caching, Refresh Token) |
| **ORM** | **Spring Data JPA** | + **QueryDSL 5.0.0** (jakarta classifier) |
| **Auth** | **JWT** | jjwt 0.12.5 (Access/Refresh Token Strategy) |
| **Docs** | **Swagger** | Springdoc-openapi-ui 6.4.x (API 문서 자동화) |
| **Deploy** | **Docker** | Docker Compose (DB, Redis, App 오케스트레이션) |

---

## 2. 📂 System Architecture (Package Structure)
유지보수와 도메인 응집도를 높이기 위한 **도메인형 패키지 구조**를 따릅니다.

```text
com.welog.server
├── common              // 전역 공통 모듈
│   ├── config          // 설정 (Security, Swagger, QueryDSL, Redis 등)
│   ├── exception       // Global Exception Handler
│   └── response        // 공통 응답 포맷 (ApiResponse)
├── domain              // 핵심 비즈니스 로직 (도메인별 분리)
│   ├── member          // 사용자 (User)
│   ├── disease         // 질병 (Disease)
│   ├── food            // 음식 기록 (FoodLog, Tag, FoodTagMap)
│   ├── symptom         // 증상/발병 기록 (SymptomLog)
│   └── analysis        // 분석 알고리즘 (Risk Calculation Service)
├── global              // 유틸리티 및 보안
│   ├── auth            // JWT Provider, Security Filter
│   └── util            // DateUtil, ImageUtil 등
└── infra               // 외부 인프라 연동
    └── s3              // 이미지 스토리지 (Optional)
```
## 3. 🗄️ Database Schema (ERD & DDL)
PostgreSQL 기준 DDL입니다. 네이밍 컨벤션은 snake_case를 따릅니다.
```SQL
-- 👤 1. 회원 (Members)
CREATE TABLE members (
    member_id           BIGSERIAL PRIMARY KEY,
    email               VARCHAR(100) NOT NULL UNIQUE,
    password            VARCHAR(255) NOT NULL,
    nickname            VARCHAR(50) NOT NULL,
    profile_image_url   VARCHAR(512),
    risk_criteria_time  INTEGER NOT NULL DEFAULT 18,
    role                VARCHAR(20) NOT NULL DEFAULT 'USER',
    status              VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    provider            VARCHAR(20),
    provider_id         VARCHAR(255),
    fcm_token           VARCHAR(512),
    created_at          TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMP NOT NULL DEFAULT NOW()
);

-- 🏥 2. 관리 질병 (Member Diseases)
CREATE TABLE member_diseases (
    member_disease_id   BIGSERIAL PRIMARY KEY,
    member_id           BIGINT NOT NULL,
    disease_name        VARCHAR(100) NOT NULL,
    is_primary          BOOLEAN NOT NULL DEFAULT FALSE,
    created_at          TIMESTAMP NOT NULL DEFAULT NOW(),
    CONSTRAINT fk_member_disease_user FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE
);

-- ⭐ 3. 즐겨찾기 (Favorites)
CREATE TABLE favorites (
    favorite_id         BIGSERIAL PRIMARY KEY,
    member_id           BIGINT NOT NULL,
    name                VARCHAR(100) NOT NULL,
    factor_type         VARCHAR(20) NOT NULL,
    CONSTRAINT fk_favorite_user FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE
);

-- 🍽 4. 식사 기록 (Meals)
CREATE TABLE meals (
    meal_id             BIGSERIAL PRIMARY KEY,
    member_id           BIGINT NOT NULL,
    image_url           VARCHAR(512),
    eaten_at            TIMESTAMP NOT NULL,
    created_at          TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMP NOT NULL DEFAULT NOW(),
    CONSTRAINT fk_meal_user FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE
);
-- Index for analysis
CREATE INDEX idx_meals_eaten_at ON meals(eaten_at);

-- 🏷 5. 식사 상세 요인 (Meal Details)
CREATE TABLE meal_details (
    meal_detail_id      BIGSERIAL PRIMARY KEY,
    meal_id             BIGINT NOT NULL,
    factor_name         VARCHAR(100) NOT NULL,
    factor_type         VARCHAR(20) NOT NULL,
    CONSTRAINT fk_meal_detail_meal FOREIGN KEY (meal_id) REFERENCES meals(meal_id) ON DELETE CASCADE
);
-- Index for search & stats
CREATE INDEX idx_meal_details_name ON meal_details(factor_name);

-- ⚡ 6. 증상 기록 (Symptoms)
CREATE TABLE symptoms (
    symptom_id          BIGSERIAL PRIMARY KEY,
    member_id           BIGINT NOT NULL,
    disease_name        VARCHAR(100) NOT NULL,
    occurred_at         TIMESTAMP NOT NULL,
    created_at          TIMESTAMP NOT NULL DEFAULT NOW(),
    CONSTRAINT fk_symptom_user FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE
);
-- Index for analysis
CREATE INDEX idx_symptoms_occurred_at ON symptoms(occurred_at);

-- 📊 7. 위험도 점수표 (Factor Scores - Batch Result)
CREATE TABLE factor_scores (
    factor_score_id     BIGSERIAL PRIMARY KEY,
    member_id           BIGINT NOT NULL,
    factor_name         VARCHAR(100) NOT NULL,
    eat_count           INTEGER NOT NULL DEFAULT 0,
    sick_count          INTEGER NOT NULL DEFAULT 0,
    risk_score          DOUBLE PRECISION NOT NULL DEFAULT 0.0,
    updated_at          TIMESTAMP NOT NULL DEFAULT NOW(),
    CONSTRAINT fk_factor_score_user FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE,
    CONSTRAINT uk_factor_score UNIQUE (member_id, factor_name)
);

-- 📅 8. 통계 리포트 (Daily Reports)
CREATE TABLE daily_reports (
    report_id           BIGSERIAL PRIMARY KEY,
    member_id           BIGINT NOT NULL,
    report_date         DATE NOT NULL,
    daily_risk_score    DOUBLE PRECISION,
    symptom_count       INTEGER NOT NULL DEFAULT 0,
    most_risk_factor    VARCHAR(100),
    CONSTRAINT fk_daily_report_user FOREIGN KEY (member_id) REFERENCES members(member_id) ON DELETE CASCADE,
    CONSTRAINT uk_daily_report UNIQUE (member_id, report_date)
);
-- Index for calendar
CREATE INDEX idx_report_date ON daily_reports(report_date);
```

## 4. 🧠 Core Algorithm (Risk Scoring Logic)
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
## 5. 📡 API Specification Strategy
Restful API 표준을 준수하며, 모든 응답은 공통 포맷(ApiResponse)을 사용합니다.
### 5.1 Common Response Format
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
### 5.2 Key Endpoints
| Domain | Method | URI | Description |
| :--- | :--- | :--- | :--- |
| **Auth** | `POST` | `/api/v1/auth/signup` | 회원가입 |
| | `POST` | `/api/v1/auth/login` | 로그인 (Access + Refresh Token 발급) |
| **User** | `GET` | `/api/v1/users/me` | 내 정보 및 설정 조회 (닉네임, 프로필, 소화시간 등) |
| | `PUT` | `/api/v1/users/me/setting` | 사용자 설정 변경 (소화 시간, 목표 질병 등) |
| **Food** | `POST` | `/api/v1/foods` | 음식 기록 저장 (이미지, 태그 포함) |
| | `GET` | `/api/v1/foods` | 날짜별 음식 기록 조회 (Feed/Calendar 필터링) |
| | `GET` | `/api/v1/foods/recent` | 최근 입력한 태그/메뉴 추천 (입력 편의성 제공) |
| **Tag** | `GET` | `/api/v1/tags/categories` | UI 렌더링용 태그 버튼 목록 (맛, 온도, 상황, 식감 등) |
| | `GET` | `/api/v1/tags/search` | 재료 태그 검색 (자동완성) |
| **Symptom** | `POST` | `/api/v1/symptoms` | 질병 발병 기록 저장 (통증 강도 포함) |
| **Analysis** | `GET` | `/api/v1/analysis/risk` | **[Main]** 오늘/내일 발병 확률 예측 및 Top3 위험 요인 |
| | `GET` | `/api/v1/analysis/chart` | 상세 분석 차트 데이터 (메뉴별/태그별/식당별 위험도) |
## 6. 🚀 Deployment Strategy

* **Target Environment:** Private Ubuntu Server (High Spec / 32GB RAM)
* **Orchestration Tool:** Docker Compose
* **Container Stack:**
  1.  **`welog-app`**: Spring Boot Web Application (Port: 8080)
  2.  **`welog-db`**: PostgreSQL Persistence Storage (Port: 5432)
  3.  **`welog-redis`**: In-memory Cache & Session Store (Port: 6379)
  4.  **`welog-nginx`**: Reverse Proxy & SSL Termination (Port: 80/443)
