# 🌙 Sleep Cycle Calculator (PWA)

개인별 수면 주기와 기상/취침 시간을 계산해주는 iOS 스타일의 웹 앱(PWA) 개발 명세서입니다.

## 🎨 UI/UX Design (Figma)
UI 상세 디자인과 에셋(아이콘, 컬러 등)은 아래 피그마 링크를 참고해주세요.
- **Figma URL**: [https://www.figma.com/design/7WSWFOlEL5dMgBewYOflgr/Somnus-UI-UX?m=auto&t=Sk5vhxeL6ALMkgax-1]

---

## 1. 프로젝트 개요 (Overview)

- **목표**: 사용자가 설정한 시간과 **수면 주기(Cycle)**, **잠드는 시간(Latency)**을 기반으로 최적의 수면 시간을 계산.
- **핵심 가치**: 복잡한 분 단위가 아닌, **10분 단위로 정돈된(Rounding)** 직관적인 시간 제안.
- **개발 방식**: Serverless / Frontend Only (React, Vue 등 자유).
- **배포 형태**: 모바일 웹 및 **PWA (홈 화면 추가 지원)**.

---

## 2. 기술 요구사항 (Technical Specs)

### 2.1 데이터 저장 (Persistence)
DB 없이 브라우저의 **LocalStorage**를 사용하여 사용자 설정을 유지합니다.

| Key | Type | Default | 설명 |
| :--- | :--- | :--- | :--- |
| `calcMode` | String | `'SLEEP'` | 계산 모드 ('SLEEP' / 'WAKE') |
| `latency` | Number | `15` | 잠드는 데 걸리는 시간 (0~60) |
| `cycleDuration` | Number | `90` | **사용자 설정 수면 주기** (70~120) |

> **Note:** `targetTime`(기준 시간)은 저장하지 않으며, 앱 실행 시 항상 **현재 시간(`new Date()`)**으로 초기화합니다.

### 2.2 PWA 설정
아이폰에서 네이티브 앱처럼 보이도록 설정합니다.
- **Manifest:** `standalone` 모드 (주소창 제거).
- **Status Bar:** `black-translucent` (배경과 일체감).
- **Touch:** 롱프레스 시 시스템 메뉴/텍스트 선택 방지 (`user-select: none`).

---

## 3. UI 및 기능 (Features)

### 3.1 메인 디스플레이 (Main Display)
- **Suggested**: 가장 추천하는 시간대 2개 (상단 강조).
- **Other Options**: 나머지 시간대 리스트.
- **표시 형식**: `HH:MM` (10분 단위 반올림됨) + `(N cycles)`.

#### ⭐️ Peek Interaction (포스 터치 로직)
사용자가 **상단 결과 영역(검은 배경)을 꾹 누르고 있을 때**의 동작입니다.
- **Trigger**: `TouchStart` (누름) ~ `TouchEnd` (뗌).
- **Logic**: 누르고 있는 동안 **`latency`를 0분으로 강제 적용**하여 결과를 실시간 재계산.
- **UI**: 별도의 그래픽 효과는 불필요하며, 숫자가 즉시 변경되면 됨.

### 3.2 설정 및 컨트롤 (Controls)
- **Cycle Duration (모달)**: 우측 상단 `≡` 또는 `⚙️` 버튼 클릭 시 노출. (범위: 70~120분, 10분 단위).
- **Time Picker**: iOS 스타일 휠 피커 (현재 시간 기준).
- **Latency Slider**: 5단계 스텝 (0, 15, 30, 45, 60분).
- **Toggle**: `Sleep` vs `Wake`.

---

## 4. 계산 알고리즘 (Core Logic) ⭐️ 중요

결과값이 깔끔하게 보이도록 **모드에 따라 올림/내림 처리**를 다르게 적용합니다.

### 변수 정의
- `CYCLE` = `cycleDuration` (사용자 설정값, 기본 90)
- `LATENCY` = 슬라이더 값 (Peek 동작 시 0)
- `RAW` = 수식으로 계산된 날것의 시간(분)

### A. Sleep 모드 (취침 기준 → 기상 추천)
> "지금 자면 언제 일어나는 게 개운할까?"

- **공식**: `TargetTime + LATENCY + (CYCLE × N)`
- **Rounding**: 미래 시간을 넉넉히 확보하기 위해 **10분 단위 올림(Ceil)**.
- **Code Logic**: `Math.ceil(RAW / 10) * 10`
- *예시: 19:54 계산됨 → **20:00** 표시*

### B. Wake 모드 (기상 기준 → 취침 추천)
> "이때 일어나려면 언제 잠들어야 할까?" (UI 텍스트는 Wake 표기)

- **공식**: `TargetTime - (CYCLE × N) - LATENCY`
- **Rounding**: 늦지 않게 안전하게 눕기 위해 **10분 단위 내림(Floor)**.
- **Code Logic**: `Math.floor(RAW / 10) * 10`
- *예시: 22:45 계산됨 → **22:40** 표시*

---

## 5. 디자인 가이드 (Style)

- **Theme**: Dark Mode Only.
- **Colors**: Figma 참조

---

## 6. 개발자 전달 사항
1. **이미지 참고**: 시안상의 터치 포인트(보라색 원)는 인터랙션 예시일 뿐 실제 구현할 필요는 없습니다.
2. **날짜 처리**: 계산 결과가 자정을 넘어가는 경우(다음 날/전날) 날짜 처리를 정확히 해주세요.
