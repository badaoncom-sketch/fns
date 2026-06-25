# RELEASE-ROADMAP.md — 릴리스 로드맵

> 상태: 신규 v0.1 ([DECISIONS.md](DECISIONS.md) D-065 — Design Freeze) · 작성일: 2026-06-26 · 단계: 설계 종료 → 개발 착수
> 목적: 현재까지 확정된 설계([DESIGN-FREEZE.md](DESIGN-FREEZE.md) Scope) 기준으로 현실적인 출시 단계를 정의한다. 본 문서는 새 기능을 만들지 않으며, 이미 설계된 범위를 출시 시점별로 묶는다.

## v1.0 — Core ERP (KR, 회원 중심)

[DESIGN-FREEZE.md](DESIGN-FREEZE.md) §3 Scope 전체. **개발 착수 전 [DECISIONS.md](DECISIONS.md) §2.2/§2.3 BLOCKER 15건 확정 필요.**

| 모듈 | 포함 기능 | 근거 |
|---|---|---|
| 회원 | 가입/가입심사/상태체계/민감변경(8종)/조직이동(관리자전용) | [PRD.md](PRD.md) §5.1~§5.6/§5.16, BR-001~BR-004/BR-022~BR-027 |
| MLM | Unilevel(LINE1~5), 패키지 엔진(무제한), 제품판매수익/페어보너스, "+알파", 35% 게이트 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md), BR-005~BR-021 |
| 정산 | 월정산, 세금원천징수, append-only, 운영자 승인 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md), BR-014~BR-018 |
| 쇼핑몰 | 통합카탈로그, 회원몰(유지구매/정기배송/자동결제), 상품이미지, 재고·배송·반품 기초모델(O-127/128/129 확정 후) | [PRD.md](PRD.md) §5.1.3/§5.21/§5.41/§5.43, BR-029~BR-032 |
| ERP Core | Workflow Engine, API Center, File Manager, Scheduler Center, Notification Center, Audit Center, System Settings | [PRD.md](PRD.md) §5.28~§5.35/§5.39, BR-036~BR-041 |
| CMS | 페이지/팝업/배너/FAQ/다국어(KR 기준) | [PRD.md](PRD.md) §5.19 |
| CRM | CRM Center, CS Center | [PRD.md](PRD.md) §5.40, §5.11 |
| 통계 | Dashboard Builder/Report Builder/Form Builder(기본형) | [PRD.md](PRD.md) §5.36~§5.38 |
| 법적 컴플라이언스 | 공제조합 보고센터, 35% 모니터링 엔진 | [ARCHITECTURE.md](ARCHITECTURE.md) §8 |

**Out of scope(v1.0이 아님, 아래 v1.1/v2.0 참조)**: B2C 고객몰(O-017), 해외 운영 실제 개시, Multi-Tenant 활성화, 모바일 앱, AI 기능, Rule Designer/Center 구조 등 선택적 고급기능.

## v1.1 — BI/AI 보강

| 모듈 | 포함 기능 | 근거/관련 Open Decision |
|---|---|---|
| BI/Dashboard 확장 | Dashboard Builder 위젯 다양화, Report Builder 고도화 | [PRD.md](PRD.md) §5.36/§5.37 기반 확장 — 구체 범위는 New Feature로 별도 정의 필요 |
| Rule Designer | 보상플랜 규칙 no-code 편집기(권고, 도입여부 미확정) | O-052(FUTURE) |
| Center 구조 | 지역 허브 도입(메커니즘 설계됨, 도입여부 미확정) | O-047(FUTURE) |
| 전자서명 고도화 | 국가별 법적 효력 확장 | O-051 |
| AI 기능 | 범위 미정 — 예: 35% 한도 추세 예측 고도화([PRD.md](PRD.md) §5.18 "예상 후원수당 비율" O-080), CS 자동응답, 이상거래 탐지 등 **구체 기능은 본 라운드에서 확정하지 않으며, v1.1 진입 시 New Feature 절차로 개별 정의한다** | 신규 영역 — 현재 설계 문서에 구체 사양 없음 |

**전제조건**: v1.0 출시 및 안정화 이후 착수. 신규 기능은 [DESIGN-FREEZE.md](DESIGN-FREEZE.md) §9 New Feature 절차를 따른다.

## v2.0 — Global / Multi-Tenant / Marketplace

| 모듈 | 포함 기능 | 근거/관련 Open Decision |
|---|---|---|
| Global | TH/JP/US 실제 운영 개시, 국가별 세금/정산규칙 구체화, 다국어 검색 | O-039/O-044/O-045/O-050/O-057/O-162(모두 FUTURE) |
| Multi-Tenant 활성화 | 신규 테넌트 온보딩, 사용량 모니터링, Job 격리, 과금 모델 | O-090/O-103/O-114/O-159/O-170(모두 FUTURE), 구조는 D-035/D-044/D-059로 이미 준비됨 |
| Marketplace | B2C 고객몰(일반 고객 접근 허용) 검토 | O-017(FUTURE) |
| Open API | 외부 개발자 대상 API 공개(현재 [API-SPEC.md](API-SPEC.md)는 내부용) — 공개 시 인증/쿼터/문서화 거버넌스 별도 필요 | 신규 영역, API Center(O-145/O-146)와 연계 검토 |
| Mobile App | 네이티브/하이브리드 앱 | O-019(FUTURE) |
| CN 활성화 | Reserved Country 해제 검토 시(현재 비활성) | O-053 |

**전제조건**: v1.0/v1.1이 KR 단일 테넌트로 안정 운영된 이후 착수. Multi-Tenant 활성화는 "구조 준비, 활성화는 보류"(D-035) 원칙에 따라 별도의 명시적 사업 결정이 선행되어야 한다.

## 우선순위 권고

1. v1.0 BLOCKER 15건([DECISIONS.md](DECISIONS.md) §2.2) 확정 → 2. v1.0 개발 착수(Codex) → 3. v1.0 출시 전 POST v1 129건 중 해당 모듈 관련 항목 순차 확정 → 4. v1.0 안정화 → 5. v1.1(BI/AI) → 6. v2.0(Global/Multi-Tenant/Marketplace).

이 순서를 거꾸로 진행하지 않는다 — 예를 들어 Multi-Tenant 활성화(v2.0)를 v1.0 개발 중에 끌어와 구현하면 D-035 "구조 준비, 활성화는 보류" 원칙과 충돌하므로 Change Request 절차를 거쳐야 한다.
