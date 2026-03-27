# 📊 WeLog (위로그) V1.0 - Database Schema (ERD)

> **Version:** 1.0  
> **Date:** 2026.01  
> **Description:** WeLog 서비스의 핵심 데이터 모델링 명세서입니다. 회원, 기록, 분석 도메인으로 구성되어 있습니다.

---

### 2.1 members (회원)
사용자 기본 정보 및 설정을 관리합니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| member_id | BIGINT | PK, AI | 회원 고유 식별자 |
| uuid | VARCHAR(36) | UK, NN | S3 경로 및 외부 식별용 식별자 |
| email | VARCHAR(100) | UK, NN | 사용자 이메일 |
| password | VARCHAR(255) | NN | 암호화된 비밀번호 |
| nickname | VARCHAR(50) | NN | 사용자 닉네임 |
| profile_image_url | VARCHAR(512) | | 프로필 이미지 URL |
| risk_criteria_time | INT | NN | 위험 분석 기준 시간 (분) |
| member_role | VARCHAR(20) | NN | 권한 (USER, ADMIN) |
| member_status | VARCHAR(20) | NN | 상태 (ACTIVE, PENDING 등) |
| provider | VARCHAR(255) | | OAuth 제공자 (google 등) |
| provider_id | VARCHAR(255) | | OAuth 제공자 식별값 |
| fcm_token | VARCHAR(255) | | FCM 디바이스 토큰 |
| created_at | DATETIME | NN | 생성 일시 |
| updated_at | DATETIME | NN | 수정 일시 |

### 2.2 member_consents (약관 동의)
사용자의 서비스 이용 약관 동의 이력을 관리합니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| consent_id | BIGINT | PK, AI | 동의 기록 식별자 |
| member_id | BIGINT | FK, UK, NN | 해당 회원 식별자 |
| privacy_policy_agreed | BOOLEAN | NN | 개인정보 처리방침 동의 여부 |
| terms_of_service_agreed | BOOLEAN | NN | 이용약관 동의 여부 |
| data_reset_agreed | BOOLEAN | NN | 데이터 초기화 동의 여부 |
| agreed_version | VARCHAR(20) | NN | 동의 시점의 정책 버전 |
| agreed_ip | VARCHAR(45) | | 동의 시점의 IP 주소 |
| agreed_at | DATETIME | NN | 동의 일시 |

### 2.3 member_diseases (회원 질병)
사용자가 관리 중인 질병 목록입니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| member_disease_id | BIGINT | PK, AI | 질병 식별자 |
| member_id | BIGINT | FK, NN | 회원 식별자 |
| disease_name | VARCHAR(255) | NN | 질병 이름 |
| emoji | VARCHAR(255) | NN | 질병 이모지 |
| is_primary | BOOLEAN | NN | 대표 질병 여부 |
| created_at | DATETIME | NN | 생성 일시 |
| updated_at | DATETIME | NN | 수정 일시 |

### 2.4 meals (식사 기록)
사용자가 기록한 식사 데이터의 상위 엔티티입니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| meal_id | BIGINT | PK, AI | 식사 식별자 |
| member_id | BIGINT | FK, NN | 회원 식별자 |
| image_url | VARCHAR(255) | | 식사 사진 URL |
| eaten_at | DATETIME | NN | 섭취 일시 |
| created_at | DATETIME | NN | 생성 일시 |
| updated_at | DATETIME | NN | 수정 일시 |

### 2.5 meal_details (식사 상세)
하나의 식사에 포함된 개별 요인들(음식, 장소 등)입니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| meal_detail_id | BIGINT | PK, AI | 상세 식별자 |
| meal_id | BIGINT | FK, NN | 부모 식사 식별자 |
| factor_name | VARCHAR(255) | NN | 요인 이름 (예: 마라탕) |
| factor_type | VARCHAR(50) | NN | 요인 타입 (MENU, PLACE 등) |

### 2.6 symptoms (증상 기록)
발생한 증상과 관련 질병을 기록합니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| symptom_id | BIGINT | PK, AI | 증상 식별자 |
| member_id | BIGINT | FK, NN | 회원 식별자 |
| member_disease_id | BIGINT | FK, NN | 관련 질병 식별자 |
| occurred_at | DATETIME | NN | 증상 발병 일시 |
| created_at | DATETIME | NN | 기록 생성 일시 |
| updated_at | DATETIME | NN | 기록 수정 일시 |

### 2.7 daily_reports (일일 분석 리포트)
사용자별 일일 통계 리포트입니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| report_id | BIGINT | PK, AI | 리포트 식별자 |
| member_id | BIGINT | FK, NN | 회원 식별자 |
| report_date | DATE | NN | 리포트 대상 날짜 |
| daily_risk_score | DOUBLE | | 당일 종합 위험도 점수 |
| symptom_count | INT | | 당일 발생 증상 횟수 |
| most_risk_factor | VARCHAR(255) | | 주요 위험 요인 이름 |
| created_at | DATETIME | NN | 생성 일시 |
| updated_at | DATETIME | NN | 수정 일시 |

### 2.8 factor_scores (요인별 점수)
개별 음식/장소 요인별 누적 통계 및 위험도입니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| factor_score_id | BIGINT | PK, AI | 점수 식별자 |
| member_id | BIGINT | FK, NN | 회원 식별자 |
| factor_name | VARCHAR(255) | NN | 요인 이름 |
| factor_type | VARCHAR(50) | NN | 요인 타입 |
| eat_count | INT | NN | 총 섭취 횟수 |
| sick_count | INT | NN | 증상 연관 횟수 |
| risk_score | DOUBLE | NN | 계산된 위험 점수 |
| created_at | DATETIME | NN | 생성 일시 |
| updated_at | DATETIME | NN | 수정 일시 |

### 2.9 favorites (즐겨찾기)
사용자가 등록한 자주 먹는 요인들입니다.

| 컬럼명 | 타입 | 제약조건 | 설명 |
| :--- | :--- | :--- | :--- |
| favorite_id | BIGINT | PK, AI | 즐겨찾기 식별자 |
| member_id | BIGINT | FK, NN | 회원 식별자 |
| name | VARCHAR(255) | NN | 즐겨찾기 이름 |
| factor_type | VARCHAR(50) | NN | 요인 타입 |
| created_at | DATETIME | NN | 생성 일시 |
| updated_at | DATETIME | NN | 수정 일시 |

---

## 4. 데이터 관계 (Relationships)

주요 테이블 간의 참조 무결성 및 카디널리티(Cardinality) 정의입니다.

### 4.1. 회원 (Member) 중심
*   **Member (1) : MemberDisease (N)**
    *   한 명의 회원은 여러 개의 관리 질병을 가질 수 있습니다.
    *   `Cascade: ALL` (회원 탈퇴 시 질병 목록 삭제)
*   **Member (1) : Favorite (N)**
    *   한 명의 회원은 여러 개의 즐겨찾기를 가질 수 있습니다.
    *   `Cascade: ALL`
*   **Member (1) : Meal (N)**
    *   한 명의 회원은 수많은 식사 기록을 남깁니다.
    *   `Cascade: ALL`
*   **Member (1) : Symptom (N)**
    *   한 명의 회원은 수많은 증상 기록을 남깁니다.
    *   `Cascade: ALL`

### 4.2. 식사 (Meal) 중심
*   **Meal (1) : MealDetail (N)**
    *   하나의 식사 기록은 여러 개의 상세 요인(메뉴, 재료, 맛 등)을 포함합니다.
    *   **생명주기:** `Meal`이 생성/삭제될 때 `MealDetail`도 함께 생성/삭제됩니다. (`Cascade: ALL`, `OrphanRemoval: true`)

### 4.3. 분석 (Analysis) 중심
*   **Member (1) : FactorScore (N)**
    *   회원별로 각 요인(Factor)에 대한 위험도 점수가 N개 존재합니다.
    *   `FactorScore`는 배치를 통해 생성/갱신되므로 `Cascade` 설정보다는 배치 로직에서 관리합니다.
*   **Member (1) : DailyReport (N)**
    *   회원별로 날짜마다 하나의 리포트가 생성됩니다.

## 5. ERD Diagram

```mermaid
erDiagram
    MEMBERS ||--o| MEMBER_CONSENTS : "1:1"
    MEMBERS ||--o{ FAVORITES : "1:N"
    MEMBERS ||--o{ MEMBER_DISEASES : "1:N"
    MEMBERS ||--o{ MEALS : "1:N"
    MEMBERS ||--o{ SYMPTOMS : "1:N"
    MEMBERS ||--o{ DAILY_REPORTS : "1:N"
    MEMBERS ||--o{ FACTOR_SCORES : "1:N"

    MEALS ||--o{ MEAL_DETAILS : "1:N"

    MEMBER_DISEASES ||--o{ SYMPTOMS : "1:N"

    MEMBERS {
        bigint member_id PK
        varchar uuid "UK"
        varchar email "UK"
        varchar password
        varchar nickname
        varchar profile_image_url
        int risk_criteria_time
        varchar member_role
        varchar member_status
        varchar provider
        varchar provider_id
        varchar fcm_token
        datetime created_at
        datetime updated_at
    }

    MEMBER_CONSENTS {
        bigint consent_id PK
        bigint member_id FK "UK"
        boolean privacy_policy_agreed
        boolean terms_of_service_agreed
        boolean data_reset_agreed
        varchar agreed_version
        varchar agreed_ip
        datetime agreed_at
    }

    FAVORITES {
        bigint favorite_id PK
        bigint member_id FK
        varchar name
        varchar factor_type
        datetime created_at
        datetime updated_at
    }

    MEMBER_DISEASES {
        bigint member_disease_id PK
        bigint member_id FK
        varchar disease_name
        varchar emoji
        boolean is_primary
        datetime created_at
        datetime updated_at
    }

    MEALS {
        bigint meal_id PK
        bigint member_id FK
        varchar image_url
        datetime eaten_at
        datetime created_at
        datetime updated_at
    }

    MEAL_DETAILS {
        bigint meal_detail_id PK
        bigint meal_id FK
        varchar factor_name
        varchar factor_type
    }

    SYMPTOMS {
        bigint symptom_id PK
        bigint member_id FK
        bigint member_disease_id FK
        datetime occurred_at
        datetime created_at
        datetime updated_at
    }

    DAILY_REPORTS {
        bigint report_id PK
        bigint member_id FK
        date report_date
        double daily_risk_score
        int symptom_count
        varchar most_risk_factor
        datetime created_at
        datetime updated_at
    }

    FACTOR_SCORES {
        bigint factor_score_id PK
        bigint member_id FK
        varchar factor_name
        varchar factor_type
        int eat_count
        int sick_count
        double risk_score
        datetime created_at
        datetime updated_at
    }
```
