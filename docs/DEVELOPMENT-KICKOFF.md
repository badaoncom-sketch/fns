# DEVELOPMENT-KICKOFF.md — 개발 착수 안내

> 상태: 신규 v0.1 ([DECISIONS.md](DECISIONS.md) D-066 — Development Kickoff 준비) · 작성일: 2026-06-26 · 단계: Design Freeze 완료 → 개발 착수
> 목적: 설계 단계를 공식 종료하고, Codex가 바로 개발에 착수할 수 있도록 시작 기준과 구현 순서를 정의한다. 본 문서는 새 기능/정책을 만들지 않으며, 이미 Frozen된 설계([DESIGN-FREEZE.md](DESIGN-FREEZE.md))를 실행 순서로 재구성한다.

## 1. 프로젝트 현재 상태

- ✔ 기획 완료
- ✔ 설계 완료
- ✔ Design Freeze 완료([DESIGN-FREEZE.md](DESIGN-FREEZE.md), [DECISIONS.md](DECISIONS.md) D-065)
- ✔ **개발 준비 완료** — 본 문서 + [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md)로 확정

## 2. 개발 시작 기준

| 항목 | 값 |
|---|---|
| **Design Freeze 버전** | Design v1.0-freeze ([DESIGN-FREEZE.md](DESIGN-FREEZE.md) §1) |
| **Freeze Date** | 2026-06-26 |
| **Git Tag** | 권장: `design-freeze-v1.0` — 첫 구현 커밋 전에 현재 `docs/` 상태에 태그를 부여할 것을 권장한다(실제 태그 생성/커밋은 본 라운드의 범위가 아니며, 개발 착수 시점에 수행). 이후 모든 구현 커밋은 이 태그 이후의 변경으로 추적된다. |
| **문서 기준** | [DESIGN-FREEZE.md](DESIGN-FREEZE.md) §4 Freeze 대상 31개 문서 — 이 문서들이 코드의 유일한 근거다 |
| **Source of Truth** | 의사결정: [DECISIONS.md](DECISIONS.md) · 비즈니스 규칙: [BUSINESS-RULE-CATALOG.md](BUSINESS-RULE-CATALOG.md)(원본은 각 BR이 가리키는 문서) · 스키마: [DATABASE.md](DATABASE.md)/[ERD.md](ERD.md)/[DATA-DICTIONARY.md](DATA-DICTIONARY.md) · API 계약: [API-SPEC.md](API-SPEC.md) · 절대 금지: [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) |
| **개발 원칙** | (1) 계산(수당/정산/세금/프로모션)은 `worker`만 수행 — `web`/`api`/`scheduler` 금지(BR-039~BR-041) (2) append-only 원장은 직접 수정 금지, 정정은 보정 엔트리(BR-018) (3) 기존 5개 전용 승인구조는 Workflow Engine으로 흡수하지 않음(BR-037) (4) 모든 정책 수치는 관리자 설정 가능해야 함(하드코딩 금지) (5) [DECISIONS.md](DECISIONS.md) §2.2/§2.3 **BLOCKER 15건**은 해당 모듈 착수 전 확정 필요 |

## 3. 구현 순서

각 Phase는 [RELEASE-ROADMAP.md](RELEASE-ROADMAP.md) v1.0 Scope 안에서의 **착수 순서**다(릴리스 자체의 단계가 아니라 개발 순서). Phase 간 의존관계가 있으므로 순서를 역행하지 않는다.

### Phase 1 — Repository / Monorepo / Framework / Database / ERP Core

기반 인프라와 ERP Core 엔진 골격을 먼저 만든다 — 이후 모든 업무 모듈이 이 위에서 동작한다(D-046 "업무 모듈 → ERP Core" 의존 방향).

| 항목 | 내용 | 선결 확정 필요(BLOCKER) |
|---|---|---|
| Repository | Git 구조, 브랜치 전략 | — |
| Monorepo | `apps/{web,api,worker,scheduler}` + `packages/` (권장안, [CODING-STANDARD.md](CODING-STANDARD.md)) | — |
| Framework | Next.js(web) / NestJS(api·worker·scheduler) 초기화 ([ARCHITECTURE.md](ARCHITECTURE.md) §2) | O-169(컴포넌트 라이브러리) |
| Database | Supabase 프로젝트, ORM/마이그레이션 도구 적용, [DATABASE.md](DATABASE.md)/[ERD.md](ERD.md) 기준 스키마 생성 | O-022(ORM), O-148(환경 구조) |
| ERP Core | Workflow Engine/API Center/File Manager/Scheduler Center/Notification Center/Audit Center/System Settings 엔진 골격(PRD §5.28~§5.39) | — |

### Phase 2 — Member / Shop / Order / Inventory / Payment

회원과 쇼핑몰 핵심 트랜잭션 — MLM 계산(Phase 4)이 입력으로 사용할 데이터(회원, 주문)가 먼저 존재해야 한다.

| 항목 | 내용 | 선결 확정 필요(BLOCKER) |
|---|---|---|
| Member | 가입/심사/상태체계/민감변경/조직이동([PRD.md](PRD.md) §5.1~§5.6/§5.16) | O-028(회원 식별 정합성), O-042/O-066(권한 매핑) |
| Shop | 통합 카탈로그, 회원몰, 상품 이미지([PRD.md](PRD.md) §5.1.3/§5.43) | O-128(카테고리 구조) |
| Order | 주문/결제/취소 ([DATABASE.md](DATABASE.md) §3.3) | — |
| Inventory | 재고/배송/반품 ([DATABASE.md](DATABASE.md) §3.10) | O-127(재고 예약), O-129(반품 상태머신) |
| Payment | 자동결제/PG 연동(§5.1.3.3) | — |

### Phase 3 — CMS / CRM / Marketing / Notification / Workflow

업무 모듈이 Phase 1의 ERP Core 엔진을 실제로 사용하는 단계.

| 항목 | 내용 | 선결 확정 필요(BLOCKER) |
|---|---|---|
| CMS | 페이지/팝업/배너/FAQ([PRD.md](PRD.md) §5.19) | O-136(SEO 메타), O-137(콘텐츠 상태) |
| CRM | CRM Center/CS Center(§5.40/§5.11) | — |
| Marketing | Marketing Program Engine(§5.20/§5.22) | — |
| Notification | Notification Center 실제 발송 연동(§5.34) | — |
| Workflow | 신규 워크플로우(환불/반품/전자결재 등) 연동(§5.30) | — |

### Phase 4 — MLM / Settlement / Compliance

가장 민감한 금전/법적 영역 — Phase 2의 회원·주문 데이터가 누적된 후에 계산 엔진을 완성한다.

| 항목 | 내용 | 선결 확정 필요(BLOCKER) |
|---|---|---|
| MLM | Unilevel/패키지/페어보너스/+알파 계산 (worker, [COMPENSATION-RULES.md](COMPENSATION-RULES.md)) | — |
| Settlement | 월정산/세금/append-only ([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)) | O-153(이의신청 연결) |
| Compliance | 35% 법적 한도 게이트, 공제조합 보고센터([ARCHITECTURE.md](ARCHITECTURE.md) §8) | O-144(DR/RTO·RPO) |

### Phase 5 — QA / Beta / Release

| 항목 | 내용 |
|---|---|
| QA | [TEST-PLAN.md](TEST-PLAN.md) 9개 영역 실행(MLM/정산/35%게이트/쇼핑몰/Workflow/API분리/Idempotency/ERP Core/Multi-Tenant) |
| Beta | KR 단일 테넌트로 제한 운영 — Multi-Tenant 활성화는 v2.0(D-035 "구조 준비, 활성화는 보류") |
| Release | v1.0 출시 — [RELEASE-ROADMAP.md](RELEASE-ROADMAP.md) |

## 4. 다음 단계

구현 착수 전 반드시 [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md)의 문서 읽기 순서를 먼저 따른다. BLOCKER 15건([DECISIONS.md](DECISIONS.md) §2.2/§2.3)은 해당 Phase 착수 직전까지 확정되어 있어야 한다.
