# DESIGN-FREEZE.md — 설계 종료 선언

> 상태: **Frozen** ([DECISIONS.md](DECISIONS.md) D-065) · Freeze Date: 2026-06-26 · 단계: 설계 종료 → 개발 착수
> 목적: FNS의 설계(Design) 단계를 공식 종료하고, 이후 모든 변경이 Bug/Change Request/New Feature 3가지 트랙으로만 관리되도록 기준을 정의한다.

## 1. Design Version

**Design v1.0-freeze** — [DECISIONS.md](DECISIONS.md) D-001~D-065 확정 결정 전체를 기준으로 한다. 이후 D-번호가 추가되는 것은 본 문서가 정의하는 Change Request/New Feature 절차를 통해서만 허용된다.

## 2. Freeze Date

**2026-06-26**

## 3. Scope — 현재 설계 범위

FNS는 Multi-Tenant MLM ERP Platform이다. 현재 설계가 커버하는 범위:

| 영역 | 범위 |
|---|---|
| MLM(보상플랜) | Unilevel Sponsor Plan(LINE1~5), 패키지 엔진(무제한 패키지+정책), 제품 판매수익/페어보너스, "+알파" 보너스, 35% 법적 한도 게이트 — [COMPENSATION-RULES.md](COMPENSATION-RULES.md) |
| 정산 | 월정산 단일 주기, append-only 원장, 세금 원천징수, 운영자 승인 — [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) |
| 회원관리 | 가입/심사/상태체계/민감변경(8종)/조직이동(관리자전용) — [PRD.md](PRD.md) §5.1~§5.6/§5.16 |
| 쇼핑몰 | 단일 통합 카탈로그 + 회원몰(유지구매/정기배송/자동결제), 상품/주문/배송/재고 보강 — [PRD.md](PRD.md) §5.1.3/§5.21/§5.41/§5.43 |
| ERP Core | Workflow Engine/API Center/File Manager/Scheduler/Notification/Audit/Dashboard·Report·Form Builder/System Settings — [PRD.md](PRD.md) §5.28~§5.39 |
| CMS / CRM / Marketing Program | 페이지/팝업/배너/FAQ/다국어, CRM Center, 무제한 Marketing Program 카탈로그 — [PRD.md](PRD.md) §5.10/§5.19/§5.20/§5.40 |
| Multi-Tenant | 구조 준비 완료, 활성화는 보류(D-035) |
| 개발 산출물 | Sitemap/Role Matrix/ERD/API Spec/Wireframe/Design System/UI Guideline/Coding Standard/Test Plan/Deployment Guide + 표준화 카탈로그 5종(Business Rule/Event/Error/State/Data Dictionary) |

Scope 밖(이번 Freeze가 다루지 않는 것): 실제 코드/마이그레이션/CI 파이프라인, 법률·세무 최종 자문 결과, 사업 수치(원본 외 4개국 마케팅 플랜 세부값), AI/BI 기능, Multi-Tenant 실제 활성화, 모바일 앱 — [RELEASE-ROADMAP.md](RELEASE-ROADMAP.md) v1.1/v2.0 참조.

## 4. Freeze 대상 문서 (29종)

[MASTER-INDEX.md](MASTER-INDEX.md) §1의 전체 목록과 동일하다 — PROJECT-CONTEXT/PRD/ARCHITECTURE/DATABASE/COMPENSATION-RULES/SETTLEMENT-RULES/LEGAL-CHECKLIST/PROMOTION-RULES(폐기)/TASK-SPEC/DO-NOT-TOUCH/DECISIONS/CHANGELOG/GAP-ANALYSIS/SITEMAP/ROLE-MATRIX/ERD/API-SPEC/WIREFRAME/UI-GUIDELINE/DESIGN-SYSTEM/CODING-STANDARD/TEST-PLAN/DEPLOYMENT/BUSINESS-RULE-CATALOG/EVENT-CATALOG/ERROR-CODE/STATE-MACHINE/DATA-DICTIONARY/MASTER-INDEX, 그리고 본 문서와 [RELEASE-ROADMAP.md](RELEASE-ROADMAP.md).

## 5. 변경 금지 항목

다음은 Design Freeze 이후 **Change Request 승인 없이는 변경할 수 없다**(기존 [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)와 별개로, 이 항목은 "설계 자체"의 동결을 다룬다):

- MLM 보상플랜 산정 로직·비율([COMPENSATION-RULES.md](COMPENSATION-RULES.md) — BR-005~BR-021)
- 정산 계산 로직·주기·append-only 원칙([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) — BR-014~BR-018)
- Workflow Engine 구조 및 기존 5개 전용 승인구조 비흡수 원칙(D-046, D-047 — BR-036/BR-037)
- ERP Core 12개 엔진의 모듈 경계와 "업무 모듈 비의존" 원칙(D-046)
- DATABASE.md §3의 확정 테이블/컬럼 구조([ERD.md](ERD.md)/[DATA-DICTIONARY.md](DATA-DICTIONARY.md)에 반영된 것)
- ARCHITECTURE.md의 5서비스 구조(web/api/worker/scheduler/redis) 및 계산/요청처리 분리 원칙(BR-039/BR-040/BR-041)
- 쇼핑몰 구조(단일 통합 카탈로그, 회원몰=구매채널 아님 — BR-029/BR-030)
- 이미 확정된 D-001~D-065 전체

## 6. 허용 변경 항목

다음은 Design Freeze 이후에도 **별도 Change Request 없이** 갱신 가능하다:

- [DECISIONS.md](DECISIONS.md) §2 Open Decision의 **상태 갱신**(미확정 → 확정, 단 확정 시점에는 일반 Change Request 절차로 D-번호를 새로 발급)
- 오타/링크 깨짐/포맷 수정 등 내용 변경이 없는 문서 정정
- [CHANGELOG.md](CHANGELOG.md)에 변경 이력 기록
- 신규 Bug Report/Change Request/New Feature 요청서 자체의 작성(본 문서 §9~§11의 양식)
- Test Plan 실행 결과 반영(테스트 자체가 새 설계를 의미하지 않는 한)

## 7. Bug 처리 원칙

**Bug = 구현 결과가 Frozen 설계 문서와 다르게 동작하는 경우.** 설계를 바꾸지 않는다 — 구현을 설계에 맞게 고친다. 예: COMPENSATION-RULES.md §3.3의 라인 판정 로직과 다르게 계산되면 Bug(코드 수정), 그 로직 자체를 바꾸자는 요청이면 Bug가 아니라 Change Request다.

## 8. Change Request 처리 원칙

**Change Request(CR) = 이미 Frozen된 설계 자체(정책/구조/Rule/Database/Architecture)를 바꾸는 요청.** 절차: ① CR 작성(어떤 BR-XXX/D-XXX/테이블을 바꾸는지 명시) → ② 영향 분석(BUSINESS-RULE-CATALOG.md/ERD.md/EVENT-CATALOG.md 기준으로 영향받는 다른 Rule·테이블·이벤트 식별) → ③ 사업팀 승인 → ④ 승인 시 새 D-번호 발급, 관련 문서(BR-XXX 원본 등) 갱신, CHANGELOG 기록 → ⑤ 구현 반영. **DO-NOT-TOUCH.md의 가드레일(append-only, worker 계산 전담 등)은 CR로도 변경할 수 없다** — 그 자체를 바꾸려면 별도의 더 높은 수준의 의사결정(사업 오너 명시적 승인)이 필요하다.

## 9. New Feature 처리 원칙

**New Feature = v1.0 Scope(§3) 밖의 신규 기능, 또는 [RELEASE-ROADMAP.md](RELEASE-ROADMAP.md) v1.1/v2.0 범위의 기능을 앞당겨 도입하는 요청.** 기존 BR/D-번호를 변경하지 않고 **새로운 BR-XXX/D-XXX를 신규 발급**해 추가한다. 기존 Frozen 설계와 충돌하면(예: 기존 테이블 구조 변경이 필요) New Feature가 아니라 Change Request로 재분류한다.

## 10. Open Decision 처리 원칙

[DECISIONS.md](DECISIONS.md) §2의 Open Decision은 **삭제하지 않는다.** §2.3에서 BLOCKER/POST v1/FUTURE로 재분류했다 — BLOCKER는 v1.0 개발 착수 전 확정, POST v1은 v1.0 출시 전 확정, FUTURE는 v1.1/v2.0 시점에 재검토한다. 확정되면 해당 행에 취소선 + 해소시킨 D-번호를 기록하는 기존 컨벤션을 그대로 따른다(번호 재사용 금지).

## 11. 개발 단계 변경 절차

1. 본 Freeze 이후 신규 코드 작업은 [TASK-SPEC.md](TASK-SPEC.md) 표준 형식의 작업 지시서로만 발행한다.
2. 모든 작업 지시서는 BLOCKER 목록([DECISIONS.md](DECISIONS.md) §2.2)에 속한 항목이 자신의 작업에 영향을 주는지 먼저 확인한다 — 영향을 준다면 그 Open Decision이 먼저 해소되어야 작업을 발행할 수 있다.
3. 개발은 Codex를 중심으로 진행하며, Claude Code는 설계 일관성 검증·문서 갱신·Change Request 영향분석 역할을 담당한다.
4. 설계 문서(`docs/`)는 이후 직접 점진적으로 고쳐나가지 않는다 — Bug/CR/New Feature 트랙을 거친 변경만 반영한다.
