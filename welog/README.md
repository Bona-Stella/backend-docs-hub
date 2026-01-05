# 📚 Backend Docs Hub

이 저장소는 프론트엔드 팀과의 협업을 위해 **모든 프로젝트의 백엔드 API 명세, 변경 내역, 접속 정보**를 통합 관리하는 곳입니다.

소스 코드는 포함되어 있지 않으며, 아래 표에서 프로젝트별 **최신 릴리즈 버전**과 **문서**를 확인하실 수 있습니다.

---

## 🚦 Project Status & Release

각 프로젝트의 배포 상태와 API 버전을 한눈에 확인하세요.
`Docs` 링크를 클릭하면 상세 가이드(계정 정보, 변경 로그)로 이동합니다.

| Project Name | Current Version | Stage | Last Updated | Link |
| :--- | :---: | :---: | :---: | :---: |
| **🛍️ 쇼핑몰 (Mall)** | `v1.2.0` | 🟢 Live | 2024-01-05 | [Go to Docs](./Shopping-Mall/README.md) |
| **🔧 관리자 (Admin)** | `v0.8.5` | 🟡 Dev | 2024-01-03 | [Go to Docs](./Admin-Dashboard/README.md) |
| **💬 채팅 서버 (Chat)** | `v0.1.0` | 🔴 Local | 2023-12-28 | [Go to Docs](./Chat-Server/README.md) |
| **🎫 예매 시스템** | `v1.0.1` | 🟢 Live | 2024-01-02 | [Go to Docs](./Ticket-System/README.md) |

> **Stage 설명:**
> - 🟢 **Live**: 운영 서버 배포 완료 (안정적)
> - 🟡 **Dev**: 개발 서버 배포 완료 (테스트 중)
> - 🔴 **Local**: 로컬 개발 중 (서버 불안정 할 수 있음)

---

## 📂 Quick Links (프로젝트별 바로가기)

### 1. [쇼핑몰 프로젝트 (Shopping-Mall)](./Shopping-Mall/README.md)
- **주요 기능**: 상품 조회, 장바구니, 결제 모듈
- **비고**: 결제 검증 로직이 v1.2.0에서 변경되었습니다.

### 2. [관리자 대시보드 (Admin-Dashboard)](./Admin-Dashboard/README.md)
- **주요 기능**: 회원 관리, 매출 통계, 배너 관리
- **비고**: 현재 `auth` 관련 API 리팩토링 중입니다.

### 3. [채팅 서버 (Chat-Server)](./Chat-Server/README.md)
- **주요 기능**: 실시간 소켓 통신, 채팅방 CRUD
- **비고**: 소켓 포트가 `8080`에서 `3000`으로 변경될 예정입니다.

---

## 📢 공통 공지사항 (Notice)

- **[2024-01-01]** 모든 개발 서버의 DB 패스워드가 변경되었습니다. 각 프로젝트 문서 내 `Credential` 항목을 확인해주세요.
- **[2023-12-25]** 매주 수요일 오전 04:00 ~ 05:00 정기 점검이 있습니다.

---

### 📞 Contact
API 관련 문의나 이슈는 **Issues** 탭을 이용해주시거나 아래로 연락주세요.
- **Backend Dev**: `your_email@example.com`
