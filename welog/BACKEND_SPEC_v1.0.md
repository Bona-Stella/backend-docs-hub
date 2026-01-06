# 🛠️ WeLog - V1.0 Backend Technical Spec

> **WeLog** 백엔드 서버의 기술 스택, 아키텍처, 데이터베이스 스키마 및 API 명세를 정의합니다.  
> **Target:** High-spec Private Server Deployment (Ubuntu Server 24.04)

---

## 1. 🏗️ Tech Stack (기술 스택)
안정성과 생산성을 최우선으로 고려한 **Java 21 (LTS) & Spring Boot 3.4** 기반의 모던 스택입니다.

| Category | Technology | Version / Note |
| :--- | :--- | :--- |
| **Language** | **Java** | **21 (LTS)** (Virtual Threads 활성화) |
| **Framework** | **Spring Boot** | **3.4.x** (Latest Stable) |
| **Build Tool** | **Gradle** | 8.x (Kotlin or Groovy DSL) |
| **Database** | **PostgreSQL** | 16.x or 17.x (Main DB) |
| **Cache/NoSQL** | **Redis** | 7.x or 8.x (Caching, Refresh Token) |
| **ORM** | **Spring Data JPA** | + **QueryDSL 5.0.0** (jakarta classifier) |
| **Auth** | **JWT** | jjwt 0.12.x (Access/Refresh Token Strategy) |
| **Docs** | **Swagger** | Springdoc-openapi-ui (API 문서 자동화) |
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
-- 1. Users (사용자)
CREATE TABLE users (
    user_id             BIGSERIAL PRIMARY KEY,
    email               VARCHAR(100) NOT NULL UNIQUE,
    password            VARCHAR(255) NOT NULL,
    nickname            VARCHAR(50) NOT NULL,
    digestion_time      INTEGER DEFAULT 18, -- 소화 시간 설정 (단위: 시간)
    role                VARCHAR(20) DEFAULT 'USER',
    profile_image_url   VARCHAR(255),
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Diseases (관리 질병)
CREATE TABLE diseases (
    disease_id          BIGSERIAL PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    name                VARCHAR(100) NOT NULL,
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 3. Tags (태그 사전 - 재료/맛/상황 등)
CREATE TABLE tags (
    tag_id              BIGSERIAL PRIMARY KEY,
    name                VARCHAR(50) NOT NULL UNIQUE,
    tag_type            VARCHAR(20) DEFAULT 'INGREDIENT', 
    -- Enum: INGREDIENT(재료), TASTE(맛), TEMP(온도), TEXTURE(식감), SITUATION(상황), ETC
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 4. FoodLogs (음식 섭취 기록)
CREATE TABLE food_logs (
    food_id             BIGSERIAL PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    menu_name           VARCHAR(100) NOT NULL,
    restaurant_name     VARCHAR(100),
    eat_date            TIMESTAMP NOT NULL, -- 실제 섭취 시간
    is_excluded         BOOLEAN DEFAULT FALSE, -- 분석 제외 여부
    image_url           VARCHAR(255),
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 5. FoodTagMap (음식-태그 연결)
CREATE TABLE food_tag_map (
    map_id              BIGSERIAL PRIMARY KEY,
    food_id             BIGINT NOT NULL REFERENCES food_logs(food_id) ON DELETE CASCADE,
    tag_id              BIGINT NOT NULL REFERENCES tags(tag_id) ON DELETE CASCADE
);

-- 6. SymptomLogs (발병 기록)
CREATE TABLE symptom_logs (
    symptom_id          BIGSERIAL PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    disease_id          BIGINT NOT NULL REFERENCES diseases(disease_id) ON DELETE CASCADE,
    symptom_date        TIMESTAMP NOT NULL, -- 발병 시간
    severity            INTEGER DEFAULT 3, -- 통증 강도 (1~5)
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for Performance
CREATE INDEX idx_food_logs_user_date ON food_logs(user_id, eat_date);
CREATE INDEX idx_symptom_logs_user_date ON symptom_logs(user_id, symptom_date);
```

## 4. 🧠 Core Algorithm (Risk Scoring Logic)
WeLog의 핵심 기능인 '발병 확률'을 계산하는 로직입니다.
단순 합산 방식이 아닌, **여집합의 확률(Probability of Union)**을 사용하여 수학적 정합성을 보장합니다.

### 4.1 Basic Concept
> **Logic:** "현재 소화 중인 모든 음식/태그/식당을 독립적인 위험 인자로 간주하고, 이들로부터 **모두 살아남을(발병하지 않을) 확률을 구해 1에서 뺀 값**을 발병 확률로 정의한다."

이 방식을 통해 위험 요소가 많아질수록 확률은 100%에 수렴하며, 단순 합산 시 100%를 초과하는 오류를 방지합니다.

### 4.2 Calculation Formula  
$$ P(TotalRisk) = 1 - \prod_{i=1}^{n} (1 - P(Factor_i)) $$  

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
