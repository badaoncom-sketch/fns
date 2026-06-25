# IMPLEMENTATION-GUIDE.md — 구현 착수 전 문서 읽기 순서

> 상태: 신규 v0.1 ([DECISIONS.md](DECISIONS.md) D-066 — Development Kickoff 준비) · 작성일: 2026-06-26 · 단계: Design Freeze 완료 → 개발 착수
> 목적: 개발자(Codex 포함)가 구현에 착수하기 전 **반드시 읽어야 하는 문서 순서**를 정의한다. 본 문서가 "읽는 순서"의 단일 출처다 — [MASTER-INDEX.md](MASTER-INDEX.md) §4는 더 이상 별도의 순서를 나열하지 않고 본 문서를 참조한다(D-064에서 겪었던 "Blocker 목록 중복 나열" 문제를 다시 반복하지 않기 위함).

## 1. 핵심 16단계 (모든 개발자 공통, 착수 전 1회)

| # | 문서 | 왜 이 순서인지 / 무엇을 확인할지 |
|---|---|---|
| 1 | [MASTER-INDEX.md](MASTER-INDEX.md) | 전체 31개 문서의 지도. 본인 역할에 추가로 필요한 문서는 §2.2의 "역할별 보강 문서"에서 확인 |
| 2 | [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) | FNS가 무엇인지, Multi-Tenant MLM ERP Platform이라는 목표, 모듈 구조 |
| 3 | [PRD.md](PRD.md) | 기능 요구사항 전체(§5.1~§5.44) — 본인이 맡을 모듈 절을 미리 표시해 둘 것 |
| 4 | [ARCHITECTURE.md](ARCHITECTURE.md) | 5서비스 구조(web/api/worker/scheduler/redis)와 **"계산은 worker만 수행한다"** 원칙(§1.1) — 이 프로젝트에서 가장 위반하기 쉬운 원칙이므로 가장 먼저 각인 |
| 5 | [DATABASE.md](DATABASE.md) | 데이터 모델 개념 정의(§3, 117개+ 엔터티). 전체를 외울 필요는 없고 구조와 §4 설계원칙(append-only 등)을 이해 |
| 6 | [ERD.md](ERD.md) | DATABASE.md를 Mermaid로 시각화한 것 — 본인이 다룰 도메인 클러스터의 다이어그램을 확인 |
| 7 | [API-SPEC.md](API-SPEC.md) | §1 전역 컨벤션(페이지네이션/에러포맷/비동기 Job 패턴)은 전원 필수, §2 모듈별 표는 본인 모듈만 |
| 8 | [BUSINESS-RULE-CATALOG.md](BUSINESS-RULE-CATALOG.md) | BR-001~BR-044 — 본인 모듈과 관련된 Rule의 "정식 정의가 어디 있는지" 색인. 여기서 설명을 다시 찾지 말고 원본 문서로 이동 |
| 9 | [STATE-MACHINE.md](STATE-MACHINE.md) | 본인이 다룰 엔터티(회원/주문/정산/Workflow 등)의 상태 전이 다이어그램 — 어떤 상태값이 확정/미확정인지 구분해서 볼 것 |
| 10 | [ROLE-MATRIX.md](ROLE-MATRIX.md) | 본인 모듈에서 어떤 역할이 어떤 액션을 할 수 있는지 — 미확정 셀(O-042/O-066)은 BLOCKER이므로 먼저 확인 |
| 11 | [SITEMAP.md](SITEMAP.md) | 본인이 만들 화면이 전체 메뉴 트리에서 어디에 위치하는지 |
| 12 | [WIREFRAME.md](WIREFRAME.md) | 본인 화면이 어느 레이아웃 아키타입(List/Detail/Form/Dashboard 등)을 따르는지 |
| 13 | [CODING-STANDARD.md](CODING-STANDARD.md) | 폴더구조/네이밍/NestJS·Next.js 규칙 — 첫 PR 전에 반드시 |
| 14 | [TEST-PLAN.md](TEST-PLAN.md) | 본인 모듈에 해당하는 테스트 전략(영역 9개 중 어디에 해당하는지) |
| 15 | [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) | 절대 금지 목록 — 13단계 이전 내용과 충돌하면 본 문서가 항상 우선 |
| 16 | [DESIGN-FREEZE.md](DESIGN-FREEZE.md) | 지금 보고 있는 모든 문서가 Frozen 상태이며, 변경하려면 Bug/Change Request/New Feature 중 무엇에 해당하는지부터 판단해야 한다는 것을 이해 |

> 소요시간 추정치는 의도적으로 표기하지 않는다 — 문서 분량이 계속 늘어 추정치가 곧 stale해지기 때문이다. 1~7단계는 한 번에 읽고, 8~16단계는 본인 모듈 작업 착수 직전에 다시 펼쳐보는 "참조용"으로 다뤄도 된다.

## 2. 역할별 보강 문서 (핵심 16단계 완료 후, 해당하는 사람만)

### 2.1 MLM/정산 개발자(Phase 4)

[COMPENSATION-RULES.md](COMPENSATION-RULES.md) → [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) → [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §1~§5 → [ARCHITECTURE.md](ARCHITECTURE.md) §8(35% 게이트) → [EVENT-CATALOG.md](EVENT-CATALOG.md)(Commission/Settlement 이벤트) → [ERROR-CODE.md](ERROR-CODE.md)(MLM-/SETTLEMENT- prefix)

### 2.2 비즈니스/기획 담당자

[BUSINESS-RULE-CATALOG.md](BUSINESS-RULE-CATALOG.md) → [DECISIONS.md](DECISIONS.md) §2.3(Open Decision BLOCKER/POST v1/FUTURE) → [GAP-ANALYSIS.md](GAP-ANALYSIS.md) §7~§8 → [RELEASE-ROADMAP.md](RELEASE-ROADMAP.md)

### 2.3 디자이너/프론트엔드

[UI-GUIDELINE.md](UI-GUIDELINE.md) → [DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) → [PRD.md](PRD.md) §5.44(ERP UX Standard) — WIREFRAME/SITEMAP은 §1의 11~12단계에서 이미 다룸

### 2.4 백엔드/DB 심화

[DATA-DICTIONARY.md](DATA-DICTIONARY.md)(컬럼 메타데이터) → [EVENT-CATALOG.md](EVENT-CATALOG.md) → [ERROR-CODE.md](ERROR-CODE.md) — ERD/API-SPEC은 §1의 6~7단계에서 이미 다룸

### 2.5 QA/테스트 심화

[TEST-PLAN.md](TEST-PLAN.md) 전체 9개 영역 → [COMPENSATION-RULES.md](COMPENSATION-RULES.md)/[SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)(계산 검증 대상 worked-example)

### 2.6 DevOps/운영

[DEPLOYMENT.md](DEPLOYMENT.md) → [ARCHITECTURE.md](ARCHITECTURE.md) §2.6/§9 → [DECISIONS.md](DECISIONS.md)에서 O-037/O-144/O-148(백업/DR/환경) 진행 상태 확인

## 3. 읽지 않아도 되는 경우

[PROMOTION-RULES.md](PROMOTION-RULES.md)는 폐기(D-030)되어 읽지 않아도 된다. [CHANGELOG.md](CHANGELOG.md)는 과거 변경 이력 추적용이며 신규 합류 시 필수는 아니다(필요할 때 검색용으로만 사용).
