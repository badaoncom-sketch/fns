# ARCHITECTURE.md — 시스템 아키텍처

> 상태: Draft v0.17 (D-063 — 개발 착수 준비 문서 세트: [API-SPEC.md](API-SPEC.md)/[DEPLOYMENT.md](DEPLOYMENT.md)/[CODING-STANDARD.md](CODING-STANDARD.md)가 본 문서를 전제로 작성됨, §10에 Open Decision 3건 추가) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 전제 문서: [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md), [PRD.md](PRD.md)

## 1. 아키텍처 개요

FNS의 최종 서버 구조는 **Railway 프로젝트 1개 안의 5개 서비스**로 확정한다 ([DECISIONS.md](DECISIONS.md) D-010). 각 서비스는 모듈형 모놀리스 코드베이스를 공유하되, **"요청을 받는 것"과 "무겁게 계산하는 것"을 물리적으로 분리된 프로세스로 둔다.**

> **물리 구조 vs 논리 모듈(D-037)**: 위 5개 서비스는 **물리적 배포 단위**이고, [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) §1.1의 ERP 13개 모듈(쇼핑몰/회원관리/.../MLM/CMS/마케팅/통계/설정 등)은 **논리적 도메인 경계**(api/worker 내부의 NestJS 모듈)다. 둘은 서로 다른 축이며, ERP 모듈이 늘어나도(예: CMS 대폭 확장, D-038) 5개 서비스 구조 자체는 변경되지 않는다 — §2.2의 모듈 목록이 ERP 모듈 경계와 대응된다.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ Railway Project: FNS                                                     │
│                                                                            │
│  ┌─────────┐      ┌─────────┐      ┌──────────┐      ┌────────────┐     │
│  │  web    │─REST→│  api    │      │  worker  │      │ scheduler  │     │
│  │ Next.js │      │ NestJS  │      │ NestJS   │      │ NestJS     │     │
│  │(UI만)   │      │(요청처리/│      │(Job 실제 │      │(Job 트리거 │     │
│  │         │      │ Job생성) │      │  처리)   │      │  만 수행)  │     │
│  └─────────┘      └────┬────┘      └────┬─────┘      └─────┬──────┘     │
│                        │  enqueue        │ dequeue          │ enqueue    │
│                        └────────►┌───────┴───────┐◄─────────┘            │
│                                  │     redis      │                      │
│                                  │ BullMQ Queue /  │                      │
│                                  │ Cache / Job     │                      │
│                                  │ Tracking/Retry  │                      │
│                                  └────────┬────────┘                      │
│                                  (source of truth 아님 — 휘발성 가정)      │
└───────────────────────────────────┬──────────────────────────────────────┘
                                     │  모든 서비스가 최종 데이터를 읽고/쓰는 곳
                              ┌──────▼──────────────────┐
                              │   Supabase (외부)        │
                              │ PostgreSQL(Source of     │
                              │ Truth) / Auth / Storage /│
                              │ Backup                   │
                              └───────────────────────────┘
```

**처리 흐름 예시 (정산)**:
1. `scheduler`가 정산 주기에 맞춰 "정산 Job 생성" 요청만 `api`(또는 직접 Redis)로 보낸다 — 계산하지 않는다.
2. `api`는 Job을 **생성만** 하여 `redis`(BullMQ)에 적재하고 즉시 응답(202 Accepted 등)한다.
3. `worker`가 Job을 꺼내 실제 정산 계산을 수행하고, 결과를 **Supabase PostgreSQL**에 기록한다.
4. `web`(Admin Console/Partner Portal)은 `api`를 통해 Job 상태/결과를 Supabase에서 조회해 보여준다.

### 1.1 계산과 요청처리의 분리 원칙 (확정 — 위반 금지)

| 원칙 | 내용 |
|---|---|
| web | 수당 계산을 하지 않는다. UI 렌더링과 api 호출만 담당한다 |
| api | request handler에서 무거운 계산을 하지 않는다. **Job 생성만** 담당한다 |
| worker | api/scheduler가 만든 Job을 꺼내 실제 처리(수당/정산/세금/프로모션/보고서)를 수행한다 |
| scheduler | Job을 **트리거만** 한다. 직접 계산하지 않는다 |
| redis | BullMQ Queue/Cache/Job Tracking/Retry 용도이며 **source of truth가 아니다** |
| Supabase PostgreSQL | 모든 최종 데이터가 저장되는 **유일한 source of truth**다 |

위 원칙은 [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)에도 동일하게 명시되어 있다.

## 2. 구성 요소

### 2.1 web — Next.js (Railway 서비스)

| 영역 | 설명 |
|---|---|
| Admin Console | 본사 운영자용. 회원/제품/보상플랜/정산 관리, 전체 추천조직 트리를 **'조직도'**(관리자 전용 용어)로 조회 |
| Partner Portal | 파트너용. **'내 조직'** 메뉴에서 LINE1~LINE5, 조직매출, 조직수당, 조직성장 조회 — "조직도"/"추천조직도" 용어는 사용하지 않음 ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.4) |
| 고객몰(B2C) | 포함 여부 미확정 — 포함 시 별도 Next.js 앱 또는 동일 앱 내 라우트로 분리 검토 |

- 인증은 Supabase Auth 세션을 Next.js 미들웨어에서 검증.
- API 통신은 `api` 서비스(REST)를 통해서만 수행 (Next.js에서 Supabase DB 직접 접근 금지).
- **수당/정산 등 어떤 계산도 web에서 수행하지 않는다 (확정 — 위반 금지).** UI는 `api`가 반환한 결과를 표시만 한다.

#### 2.1.1 ERP UX Standard — 공통 UX 컴포넌트 계층 (신규 — [DECISIONS.md](DECISIONS.md) D-061)

Admin Console/Partner Portal/쇼핑몰/CMS가 공유하는 공통 UX 컴포넌트 라이브러리([PRD.md](PRD.md) §5.44)는 `web` 서비스 내부에만 존재하는 **순수 프레젠테이션 계층**이다.

- ConfirmDialog/WarningDialog/SuccessToast·ErrorToast·InfoToast/LoadingOverlay/LoadingButton/ActionButton/ConfirmActionButton/BulkActionDialog/UnsavedChangesGuard/ImageUploader/FileUploader/ImagePreview/ProgressBar는 화면마다 새로 구현하지 않고 공통 라이브러리를 재사용한다([PRD.md](PRD.md) §5.44.9/§5.44.12).
- **§1.1 원칙(계산과 요청처리의 분리)을 그대로 따른다** — Confirm Dialog/Loading/Toast는 `api` 호출 전후의 UI 상태일 뿐이며, 어떤 계산·검증 로직도 포함하지 않는다.
- **Bulk Action**(§5.44.6)은 기존 단건 `api` 엔드포인트를 N회 호출하거나 기존 batch 엔드포인트를 재사용한다 — 신규 계산 로직이나 신규 워크플로우 분기를 추가하지 않는다. 대규모 건수에서 동기 처리가 비효율적이면 §2.3 `worker`의 기존 Job 패턴을 재사용할 수 있으나, 정확한 전환 임계값은 미확정(O-118, §10).
- **Undo**(§5.44.7)는 `api`의 기존 소프트 삭제(`is_active`) 토글 엔드포인트를 재호출하는 것으로 구현되며, append-only 원장(`commission_records`/`settlement_items`)을 다루는 모듈에는 적용하지 않는다([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2).
- Activity Log/Audit Log/Notification Center 연계(§5.44.8)는 §2.2 `Audit`/`Notification` 모듈의 기존 경로를 그대로 호출한다 — 신규 모듈을 추가하지 않는다.

### 2.2 api — NestJS (Railway 서비스, 요청 처리 전용)

모듈 경계(초안) 및 책임 범위:

| 모듈 | api에서의 책임 |
|---|---|
| Auth | Supabase Auth 연동, 역할기반접근제어(RBAC) — SuperAdmin/Compliance Admin/지정 조직관리 관리자/CountryAdmin/CenterAdmin/기능별 관리자/Partner, **국가 스코프(country_scope) 검사 포함** ([PRD.md](PRD.md) §5.6.4). **조직 이동 권한(SuperAdmin/Compliance Admin/지정 조직관리 관리자 한정)은 일반 RBAC와 별도의 전용 role guard로 검사** ([PRD.md](PRD.md) §5.16.5) |
| Member | 회원 CRUD, 스폰서 트리 조회(읽기), 회원 유형(개인/사업자/법인/외국인)별 가입 정보·서류 접수 ([PRD.md](PRD.md) §5.6.1) |
| Catalog | 제품/가격 관리 |
| Order | 주문 생성/취소 **요청** 접수, 즉시 반영 가능한 매출 기록 |
| Compensation / Settlement | **계산을 수행하지 않는다.** 계산 Job 생성, 계산 결과 조회만 담당 — 실제 계산은 §2.3 `worker`. 국가별 마케팅 플랜 버전·세금규칙·정산규칙(§8)을 입력 파라미터로 받는다 |
| MemberLifecycle | 회원이 신청 가능한 민감 변경(명의/탈퇴/휴면/강제탈퇴/재가입/계좌, [PRD.md](PRD.md) §5.3) 요청 접수, Snapshot 생성, **영향 분석 Job 생성**(계산은 worker가 수행). 회원 가입 심사(§5.6.3 "가입심사중")도 동일한 요청/승인 패턴을 따른다. **추천인 변경(조직 이동)은 더 이상 이 모듈을 통해 회원이 신청할 수 없다 — OrganizationTransfer 모듈 참조** ([DECISIONS.md](DECISIONS.md) D-020) |
| OrganizationTransfer | **조직 이동(= 추천인 변경, 관리자 전용)**([PRD.md](PRD.md) §5.16) — 사유코드·증빙첨부 접수, **role guard(SuperAdmin/Compliance Admin/지정 조직관리 관리자만 허용)**, Snapshot 생성, 영향 분석 Job 생성. 일반 관리자(CountryAdmin 등)의 호출은 거부 |
| Compliance | **공제조합 보고센터**([PRD.md](PRD.md) §5.7) — 보고서 생성 **요청** 접수(계산은 worker), 제출 이력 조회. **법적 한도(35%) 모니터링 대시보드**([PRD.md](PRD.md) §5.18) — `compliance_ratio_snapshots`([DATABASE.md](DATABASE.md) §3.29) 조회만 담당, 계산은 worker(§8.1) |
| Logistics | 재고/배송/반품 관련 CRUD, 3PL 연동 호출, 무거운 정합성 대조는 Job으로 위임 |
| Audit | 도메인 변경에 대한 감사로그 기록(경량 — api에서 직접 기록) |
| Notification | **Notification Center**([PRD.md](PRD.md) §5.12) — 알림 발송 **요청**(Job 생성)만 담당, 실제 발송은 `worker` |
| Center | **센터 구조**([PRD.md](PRD.md) §5.9, 도입 미확정) — 센터 CRUD, 센터 이동 요청 접수(MemberLifecycle과 동일 패턴) |
| DocumentCenter | **Document Center**([PRD.md](PRD.md) §5.10) — 문서/버전/공개범위 CRUD, 회원 개인 문서(정산자료 등)는 생성 요청만(실제 생성은 worker) |
| CustomerService | **CS Center**([PRD.md](PRD.md) §5.11) — 티켓 CRUD, 변경요청/이의신청 연결 |
| ESignature | **전자서명**([PRD.md](PRD.md) §5.13) — 서명 요청/검증(경량 — api에서 직접 처리), `member_change_requests`의 승인 선행조건 검사 |
| ActivityLog | **회원 활동 이력**([PRD.md](PRD.md) §5.14) — 경량 기록(api에서 직접 기록), 회원 자기열람 조회 |
| RuleDesigner | **Rule Designer**([PRD.md](PRD.md) §5.15, 도입 미확정) — 규칙 초안 CRUD, **시뮬레이션/발행 Job 생성**(계산·검증은 worker) |
| CMS | **CMS 대폭 확장**([PRD.md](PRD.md) §5.19, D-038) — `cms_pages`/`faq_categories`·`faq_items`/`popups`/`banners`/`cms_translations` CRUD. 기존 DocumentCenter/Notification 모듈과는 별개로, 콘텐츠·FAQ·팝업·배너·다국어 번역을 전담 |
| MarketingProgram | **Marketing Program Engine**([PRD.md](PRD.md) §5.20/§5.22, D-039) — `marketing_programs`/`marketing_program_products` CRUD, 신청 접수(`marketing_program_applications` 생성 — 승인 처리는 경량이라 api에서 직접 수행, 무거운 통계 집계만 worker로 위임) |
| Point | **포인트 시스템**([PRD.md](PRD.md) §5.23, D-041) — 사용신청 접수, `point_accounts` 조회. 적립/만료 등 배치성 처리는 worker(§2.3) |
| Shop | **쇼핑몰 기능 보강**([PRD.md](PRD.md) §5.21, D-040) — 리뷰/상품문의/위시리스트 CRUD, 쿠폰 발급/사용 요청 접수 |
| Analytics | **관리자 통계**([PRD.md](PRD.md) §5.25, D-043) — 통계 조회 API(실시간 쿼리 또는 스냅샷 캐시 조회, 방식은 미확정). 집계 자체가 무거우면 worker로 위임(§2.3) |

- **api request handler는 무거운 계산을 하지 않는다 (확정 — 위반 금지).** 후원수당/정산/세금/프로모션 판정/보고서 생성 등 연산량이 큰 작업은 모두 **Job을 생성**해 `redis`(BullMQ)에 적재하고, 즉시 응답(202 Accepted + Job id)한다.
- 클라이언트는 Job 상태/결과를 폴링 또는 webhook으로 조회하며, 결과는 `worker`가 Supabase PostgreSQL에 기록한 값을 읽는다.
- 모듈 간 호출은 명확한 인터페이스를 통해서만 이루어지며, Compensation/Settlement 관련 모듈은 입력 스냅샷을 받아 순수 계산을 수행하도록 설계되어 `worker`로의 이전이 자연스럽다.

### 2.3 worker — NestJS 워커 (Railway 서비스, Job 실제 처리)

- `api`/`scheduler`가 `redis`(BullMQ)에 적재한 Job을 꺼내 **실제 무거운 처리**를 수행하는 단일 서비스다.
- 처리 대상(확정 — 12종, 직급 체계 폐기로 "프로모션" Job 제거, 조직이동 적용·정기배송 처리·포인트 만료 처리 Job 추가 — [DECISIONS.md](DECISIONS.md) D-030/D-022/D-031/D-041):
  - **수당**: Compensation 모듈의 LINE1~LINE5 후원수당(조직수당) 계산 배치
  - **정산**: Settlement 모듈의 정산 배치, 법적 한도(35%) 검증 — 검증 로직은 §8.1 **법적 한도 모니터링 엔진**을 호출한다(중복 계산 로직을 두지 않는다)
  - **법적 한도 모니터링(§8.1)**: ① 이벤트 기반 실시간 캐시 갱신(매출/후원수당 발생 시마다, Redis), ② 일/월/연도누적 배치 정합화(Postgres 확정 — scheduler 트리거)
  - **세금**: 원천징수(3.3%) 계산, 지급조서 산출
  - **보고서**: 관리자 대시보드/리포트용 집계 생성 및 **공제조합 보고센터**(§8)의 규제 보고서 생성
  - **알림 발송**: Notification Center(§2.2 Notification)의 Email/SMS/KakaoTalk/Push 실제 발송, 실패 시 Redis Retry
  - **회원 개인 문서 생성**: Document Center(§2.2 DocumentCenter)의 정산자료/지급명세서/원천징수자료 생성
  - **규칙 시뮬레이션/발행 검증**: Rule Designer(§2.2 RuleDesigner) 초안에 대한 dry-run 시뮬레이션 및 법적 한도 등 하드 제약 검증 (도입 시)
  - **3PL 정합성 대조**: §2.7 참조
  - **조직이동 적용(D-022)**: scheduler가 트리거한 Job을 받아, `effective_date`가 도달한 승인된 조직 이동을 찾아 `members.sponsor_id` 갱신 + `member_sponsor_history` 기록 + Audit Log를 수행한다 (§7.1.1)
  - **정기배송 처리(D-031, 기존 목록에 누락되어 있던 것을 보강)**: scheduler가 `next_delivery_date` 도달 건을 트리거하면, 자동결제 시도 → 성공 시 `orders` 생성 → 배송 트리거를 수행한다 ([DATABASE.md](DATABASE.md) §3.30, [PRD.md](PRD.md) §5.1.3.3)
  - **포인트 만료 처리(신규, D-041)**: scheduler가 매일 트리거하면, `point_transactions.expires_at`이 도달한 미사용 포인트를 찾아 `EXPIRE` 보정 엔트리를 생성한다 ([DATABASE.md](DATABASE.md) §3.36)
- 수당/정산/프로모션 계산 Job은 **회원의 `country_code`** 를 기준으로 해당 국가에 활성화된 마케팅 플랜 버전·세금규칙·정산규칙을 조회해 적용한다 ([DATABASE.md](DATABASE.md) §3.13) — 국가별로 동일 계산 로직이 다른 파라미터를 사용한다.
- **수령/판정 대상 필터링과 트리 순회는 분리한다 (확정, D-021)**: WITHDRAWN/FORCED_WITHDRAWN 회원은 수당 수령·정산 생성의 **대상에서는 제외**하지만, 추천조직 트리(`sponsor_id`) 순회 자체는 상태와 무관하게 수행한다 — 다른 회원의 라인 깊이 계산에 영향을 주지 않기 위함이다 ([DATABASE.md](DATABASE.md) §4 원칙 10).
- 회원 민감 변경의 **수당/정산 영향 분석(dry-run 시뮬레이션, §7)** 도 무거운 연산이므로 worker가 처리한다 — api가 직접 계산하지 않는다.
- 계산 결과는 **Supabase PostgreSQL**에만 기록한다 (`commission_records`, `settlement_items` 등 append-only 원장 — [DATABASE.md](DATABASE.md)). Redis에는 Job 진행상태/재시도 정보만 남긴다.
- worker는 수평 확장(인스턴스 추가)이 가능하도록, 내부에서 다른 서비스(web/api)의 인메모리 상태에 의존하지 않는다.

### 2.4 scheduler — NestJS 스케줄러 (Railway 서비스, Job 트리거 전용)

- **자동정산, 자동프로모션, 자동보고서, 조직이동 적용 배치(신규, D-022)** 를 정해진 주기(cron)에 따라 **Job 생성만** 하여 `redis`에 적재하는 서비스다.
- **scheduler는 계산을 직접 수행하지 않는다 (확정 — 위반 금지).** 생성된 Job은 `worker`가 처리한다.
- **조직이동 적용 배치**: `effective_date`가 도달한 승인된 조직 이동을 찾아 적용하는 Job을 주기적으로(매일 등, 정확한 주기는 [DECISIONS.md](DECISIONS.md) O-070) 트리거한다 — §7.1.1 참조. scheduler는 "언제 검사할지"만 결정하며, 실제로 어떤 건을 적용할지는 worker가 판단한다.
- cron 구현 방식(Railway 자체 cron vs BullMQ repeatable job)은 **미확정** ([DECISIONS.md](DECISIONS.md)).
- 정산 주기([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §2)가 확정되면 scheduler의 cron 설정에 그대로 반영한다. (직급 체계 폐기로 "프로모션 판정 주기"는 N/A — D-030)

### 2.5 redis — Redis (Railway 서비스)

- 역할(확정 — 4종): **BullMQ Queue**(Job 적재/분배), **Cache**(조회 캐시), **Job Tracking**(Job 상태 추적), **Retry**(실패 Job 재시도).
- **Redis는 source of truth가 아니다 (확정 — 위반 금지).** Redis 데이터가 사라져도 비즈니스 데이터는 손실되지 않아야 하며, 모든 최종 데이터는 Supabase PostgreSQL에 저장한다.
- 세션/조회 캐시 용도로도 사용하되, 캐시 무효화 실패 시에도 정합성은 PostgreSQL 조회로 복구 가능해야 한다.

### 2.6 Supabase — PostgreSQL / Auth / Storage / Backup (외부 관리형 서비스)

- **PostgreSQL**: FNS의 **유일한 source of truth**. web/api/worker/scheduler 모든 서비스가 최종적으로 이곳을 읽고 쓴다.
- **Auth**: 회원 인증 처리, `api`에서 JWT 검증.
- **Storage**: 파일 저장 — 회원 유형별(개인/사업자/법인/외국인) 본인확인 서류, 공제조합 보고센터의 생성된 보고서 파일 등.
- **Backup**: Supabase의 자동 백업/PITR(Point-in-Time Recovery) 기능을 1차 백업 수단으로 사용한다. 백업 주기·보존기간·별도 백업 정책 필요 여부는 **미확정** ([DECISIONS.md](DECISIONS.md)).
- Row Level Security(RLS) 사용 여부는 [DECISIONS.md](DECISIONS.md)의 Open Decision — 비즈니스 로직이 api/worker에 집중되는 구조이므로 RLS는 보조적 방어선으로 검토.

### 2.7 외부 연동 — 3PL(제3자 물류)

- 자체 물류 인력 없이 시작하는 것을 전제로, 창고/배송을 외부 3PL 업체에 위탁한다. `api`가 주문 전달·재고 동기화 요청을 접수하고, 정합성 대조 등 무거운 배치는 `worker`가 Job으로 처리한다.
- 연동 대상 3PL 업체, 연동 방식(REST API vs 파일 배치 연동)은 **미확정** ([DECISIONS.md](DECISIONS.md)).
- 3PL 측 재고와 FNS 내부 재고 데이터의 정합성 검증(주기적 대조)은 `scheduler`가 Job을 트리거하고 `worker`가 수행한다.
- 반품/환수 물류(반송 접수, 검수, 재입고 또는 폐기 처리)도 동일 연동 경로를 사용한다.

## 3. API 설계 원칙

- 기본은 **REST API** (OpenAPI 스펙으로 문서화). GraphQL 전환 필요성은 추후 검토.
- **`api`는 Job 생성과 조회만 담당한다 (확정 — §1.1, §2.2).** 수당/정산/세금/프로모션/보고서 등 무거운 연산은 request handler 내에서 절대 직접 수행하지 않고, "Job 생성 → `worker` 처리 → 상태/결과 조회" 비동기 패턴(202 Accepted + Job id, polling 또는 webhook)을 따른다.
- 모든 API는 역할기반 권한 검사를 통과해야 하며, 파트너는 본인 및 본인 하위 조직 데이터만 조회 가능.

## 4. 보안 아키텍처

- PII(주민번호, 계좌번호, 사업자/법인등록번호, 외국인등록번호·여권정보 등 회원 유형별 본인확인 정보)는 암호화 저장, 접근 로그 기록.
- 정산 금액 변경은 모두 append-only 이력으로 남기고 직접 UPDATE/DELETE 금지 ([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) 참조).
- **국가 스코프(country_scope) 검사**: CountryAdmin/기능별 관리자의 모든 요청은 RBAC 검사에 country_scope 비교를 포함한다 — 본인 국가 외 회원/정산/보고서 데이터에 대한 조회·수정 요청은 거부한다 (SuperAdmin 제외). 위반 시도는 `audit_logs`에 기록한다.
- **조직 이동 전용 role guard(확정, D-020)**: 조직 이동 API는 일반 RBAC 검사 외에 호출자가 SuperAdmin/Compliance Admin/지정 조직관리 관리자 중 하나인지 추가로 검사한다 — 셋 중 하나가 아니면 요청이 생성되기 전 단계(인가)에서 즉시 거부하며, 거부 시도도 `audit_logs`에 기록한다. 회원 계정으로의 호출은 애초에 해당 엔드포인트가 노출되지 않는다(§7.1).
- Rate limiting, 입력 검증은 NestJS 레벨(Guard/Pipe)에서 일괄 적용.
- 시크릿/자격증명은 Railway 환경변수로 관리, 레포에 커밋하지 않음.

## 5. 확장성 고려사항

- Compensation/Settlement/Logistics(정합성 대조)는 처음부터 `worker` 서비스로 분리되어 있으므로(§2.3), 처리량이 늘어나면 **worker 인스턴스 수를 늘리는 것**만으로 확장한다 — 코드 구조 변경이 필요하지 않다.
- `worker`가 처리하는 Job 종류(수당/정산/세금/프로모션/보고서)가 늘어나 단일 worker로 부족해지면, Job 종류별로 worker를 추가 분리하는 것을 검토한다 (현재는 5개 서비스 중 worker 1개로 통합 — [DECISIONS.md](DECISIONS.md) D-010).
- 회원/조직 규모가 커질 경우 스폰서 트리 조회는 재귀 쿼리 대신 closure table 등 구조 검토 ([DATABASE.md](DATABASE.md) 참조).

## 6. 모니터링/로깅 (미확정)

- 1차는 Railway 기본 로그 활용.
- APM/에러트래킹(Sentry 등) 도입 여부는 구현 단계에서 결정.

## 7. 민감 변경(Sensitive Change) 처리 아키텍처

회원 변경(명의 변경, 탈퇴, 휴면, 강제탈퇴, 재가입, 계좌 변경 — [PRD.md](PRD.md) §5.3)은 후원수당/정산 계산에 직접 영향을 주므로, 다음 공통 흐름을 따른다.

```
[변경 요청] → [Snapshot 생성] → [수당/정산 영향 분석 요청]
   → [운영자 승인/반려] → [승인 시 반영 + Audit Log] → [반려 시 폐기 + Audit Log]
```

- **MemberLifecycle 모듈**이 위 흐름을 오케스트레이션하되, **요청 접수·Snapshot 생성은 `api`**, **영향 분석 계산은 `worker`**가 수행한다 (§1.1, §2.2, §2.3) — api가 직접 계산하지 않는다.
- 영향 분석은 Compensation/Settlement의 **읽기 전용 시뮬레이션(dry-run) 인터페이스**를 worker 내에서 호출하여 수행한다 — 실제 `commission_records`/`settlement_items`에는 쓰지 않고, 변경이 반영되었다고 가정한 입력으로 계산만 수행해 결과를 Supabase에 기록한다.
- 모든 변경 요청·승인/반려·영향 분석 결과는 `audit_logs` 및 [DATABASE.md](DATABASE.md) §3.9 `member_change_requests`에 기록한다.
- 승인된 변경은 **변경 시점 이후의 계산에만 적용**되며, 과거 확정된 `commission_records`/`settlement_items`는 재작성하지 않는다 (D-021/D-022와 동일한 append-only 원칙).
- 회원 탈퇴/강제탈퇴 승인 시, Logistics 모듈에 **재고 환수(반품) 프로세스 시작**을 트리거한다 ([PRD.md](PRD.md) §5.5, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §11).
- **탈퇴/강제탈퇴 승인은 조직 이동(§7.1)을 트리거하지 않는다 (확정, [DECISIONS.md](DECISIONS.md) D-021).** MemberLifecycle 모듈은 `members.status`만 변경하며 `sponsor_id`에는 관여하지 않는다 — 두 모듈(MemberLifecycle/OrganizationTransfer)은 서로 독립적으로 동작하고, 조직 재배치가 필요하면 관리자가 OrganizationTransfer를 별도로 호출해야 한다.

### 7.1 조직 이동(Organization Transfer) — 별도 흐름 (확정 — [DECISIONS.md](DECISIONS.md) D-020, D-022)

추천인 변경(= 조직 이동, [PRD.md](PRD.md) §5.16)은 위 일반 민감 변경 흐름과 **세 가지 점에서 다르다**:

1. **개시 주체**: 회원이 아니라 **관리자**만 요청을 개시할 수 있다. `api`는 OrganizationTransfer 모듈(§2.2) 호출 시 호출자의 역할이 SuperAdmin/Compliance Admin/지정 조직관리 관리자인지 **role guard**로 먼저 검증하며, 그 외 역할의 호출은 인가 단계에서 즉시 거부한다(요청 자체가 생성되지 않음).
2. **추가 필수 입력**: 일반 흐름의 "[변경 요청]" 단계가 **"[사유 코드 선택] → [증빙 첨부] → [영향 분석]"** 으로 세분화된다 — 사유 코드(§5.16.3, 9종)와 증빙 첨부 없이는 요청 자체를 생성할 수 없다.
3. **승인과 적용의 분리(D-022, 신규)**: 일반 민감 변경은 "[승인] → [반영]"이 한 단계지만, 조직 이동은 **"[승인] → [적용일 예약] → [적용일 도달] → [실제 적용]"** 으로 더 세분화된다. 승인 시점에는 `organization_transfer_logs.status`만 `APPROVED_SCHEDULED`로 바뀌고, **`members.sponsor_id`는 전혀 변경되지 않는다.** 실제 변경은 적용일에 별도 배치(아래 7.1.1)가 수행한다.

나머지(Snapshot/영향분석/Audit Log)는 일반 흐름과 동일한 패턴(worker가 영향분석 수행, append-only 기록)을 따른다. 데이터 모델은 [DATABASE.md](DATABASE.md) §3.26 참조.

#### 7.1.1 적용일 예약 및 배치 처리 (확정, D-022)

```
[승인] → status=APPROVED_SCHEDULED, effective_date 확정(기본: 승인일 기준 익월 1일 00:00)
            │
            │  (긴급 조직 이동의 5개 사유인 경우: effective_date = 승인 시각, 즉시 다음 단계로)
            ▼
[scheduler] 매일(또는 적용일 변경 가능성을 고려한 주기, [DECISIONS.md](DECISIONS.md) O-070) "조직이동 적용 배치" Job 생성
            │
            ▼
[worker] effective_date <= 현재 시각이고 status=APPROVED_SCHEDULED인 건을 조회 →
         members.sponsor_id 갱신 + member_sponsor_history 기록 + status=APPLIED + applied_at 기록 + Audit Log
```

- **scheduler**는 "조직이동 적용 배치"를 **트리거만** 한다 (§1.1 원칙과 동일) — 실제로 어떤 건을 적용할지 판단하고 `sponsor_id`를 갱신하는 것은 **worker**가 수행한다.
- 이 배치가 `members.sponsor_id`를 갱신하는 **유일한 경로**다. 승인 자체는 이 컬럼에 어떤 영향도 주지 않는다.
- 긴급 조직 이동은 승인 즉시 이 적용 로직을 동기적으로(또는 즉시 큐에 적재해) 수행하여, 별도 적용일 대기 없이 곧바로 `APPLIED` 상태로 전환한다.

#### 7.1.2 수당/정산 엔진과의 관계 (확정, D-022)

Compensation/Settlement 엔진(worker)은 계산 시점에 **`members.sponsor_id`(현재값)를 그대로 조회**하기만 하면 된다 — 별도로 "이 조직 이동이 승인됐는지/적용됐는지"를 확인하는 추가 로직이 필요 없다. 그 이유는 7.1.1에서 보장하듯 **`sponsor_id`는 적용(applied) 시점에만 갱신**되기 때문이다 — 승인되었지만 미적용인 조직 이동은 `sponsor_id`에 반영되어 있지 않으므로, 엔진이 평소처럼 현재값을 읽기만 해도 자동으로 "적용 완료된 구조만" 보게 된다.

## 8. 다국가·컴플라이언스 아키텍처 (확정 — [DECISIONS.md](DECISIONS.md) D-011)

FNS는 국가(KR/TH/JP/US, CN은 Reserved — [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) §4, [DECISIONS.md](DECISIONS.md) D-023)를 모든 비즈니스 로직의 입력 파라미터로 다룬다.

- **국가 스코프 계산**: `worker`가 후원수당/정산/프로모션을 계산할 때, 대상 회원의 `country_code`를 기준으로 그 국가에 활성화된 **마케팅 플랜 버전·세금규칙·정산규칙**([DATABASE.md](DATABASE.md) §3.13)을 조회해 적용한다. 같은 계산 로직이라도 국가에 따라 다른 파라미터가 사용된다.
- **관리자 권한의 국가 스코프**: §4에서 정의한 country_scope 검사를 모든 관리자 API에 적용한다.
- **공제조합 보고센터(Compliance 모듈)**: `scheduler`가 국가별 보고 주기에 맞춰 "보고서 생성 Job"을 트리거하고, `worker`가 해당 국가의 회원/매출/후원수당 데이터를 집계해 보고서를 생성한 뒤 Supabase Storage에 저장한다. `api`는 관리자의 수동 생성 요청을 Job으로 변환하고 생성된 보고서 목록/상태만 조회한다 (§1.1 원칙과 동일 — api는 계산하지 않음).
- **회원 가입 심사**: 회원 유형(개인/사업자/법인/외국인 — [PRD.md](PRD.md) §5.6.1)별 본인확인 서류는 Supabase Storage에 저장하고, 검증은 §7과 유사한 "요청→검증→승인/반려" 패턴을 따른다. 단, 신규 가입이므로 수당/정산 영향 분석 단계는 해당되지 않는다(과거 실적이 없음).
- KR 외 4개국의 실제 세금규칙/마케팅플랜/프로모션/정산규칙/규제 보고 요건은 **미확정** — [COMPENSATION-RULES.md](COMPENSATION-RULES.md), [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md), [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) 후속 라운드에서 국가별로 정의한다.

### 8.1 법적 한도(35%) 모니터링 엔진 (Compliance Monitoring Engine) — 확정 — [DECISIONS.md](DECISIONS.md) D-027

방문판매법상 후원수당 지급 총액은 **해당 사업연도 매출액의 35%를 초과할 수 없다**([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §1, §6). 본 절은 이 한도를 **사전에 감지·경고·차단**하는 시스템 구조를 정의한다. 법령상 판단 단위는 "사업연도"이지만, 실무적으로는 더 짧은 주기의 추정치도 필요하므로 **3개 시간 단위**로 계산을 지원한다.

#### 8.1.1 3종 계산 단위

| 단위 | 계산 방식 | 저장 위치 | 용도 |
|---|---|---|---|
| **실시간(Real-time)** | `commission_records`/`orders` 생성 이벤트가 발생할 때마다 누적 매출·후원수당 카운터를 증감 — `api`가 이벤트를 발행하고 `worker`가 Redis 카운터를 갱신 | **Redis(캐시, source of truth 아님)** | 대시보드의 "현재 비율" — 추정치, 오차 허용 |
| **월별(Monthly)** | `scheduler`가 매월 마감 후 트리거, `worker`가 그 달의 `orders`/`commission_records`를 집계 | Postgres `compliance_ratio_snapshots`([DATABASE.md](DATABASE.md) §3.29) | 추세 모니터링 — **법적 판단 기준 아님**(법령은 사업연도 기준) |
| **연도 누적(Annual cumulative)** | 사업연도 시작일부터 현재까지 `commission_records`/`orders`를 누적 합산 — 매 정산배치 검증 시점([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 ③단계)마다 갱신 | Postgres `compliance_ratio_snapshots` | **법적 준수 여부를 판단하는 유일한 권위있는 지표** |

- **실시간 캐시는 Redis 원칙(§2.5, source of truth 아님)을 그대로 따른다** — 캐시가 사라져도 비즈니스 판단에는 영향이 없어야 하므로, 법적 차단(아래 8.1.3)은 **항상 연도 누적(Postgres) 값만을 근거로 판단**하며, 실시간 캐시는 대시보드 표시용 추정치로만 쓴다. 캐시와 Postgres 값은 `worker`의 정기 정합화 Job(예: 매시간)으로 drift를 보정한다.
- 세 단위 모두 **동일한 집계 로직(분자/분모 정의)** 을 공유한다 — 단위만 다를 뿐 계산식이 달라지지 않는다. 분자/분모 정의(특히 패키지 추천보너스·페어보너스 포함 여부)는 [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §6, [DECISIONS.md](DECISIONS.md) O-059의 분류 결과를 그대로 입력받는다 — **이 엔진은 분류를 직접 결정하지 않고, 분류 결과를 적용해 계산만 한다.**

#### 8.1.2 임계치 (3단계 — 확정값은 KR, 국가별 설정 가능)

| 단계 | 임계치 | 동작 |
|---|---|---|
| 정상 | 30% 미만 | 표시만 |
| **주의** | 30% 이상 | 대시보드 표시 강조 |
| **경고** | 33% 이상 | 대시보드 강조 + 지정 관리자 알림(Notification Center) |
| **차단** | 35% 이상 | 아래 8.1.3 게이트 동작 |

- 임계치는 `compliance_thresholds`([DATABASE.md](DATABASE.md) §3.29)에 **국가별로 설정**한다 — KR은 30/33/35%로 확정([DECISIONS.md](DECISIONS.md) D-027)되었으나, TH/JP/US는 자체 법정 한도가 미확정(O-045)이므로 해당 국가는 임계치가 설정되기 전까지 차단 게이트가 비활성 상태다.

#### 8.1.3 "차단"의 정확한 의미 (설계 검토 결과)

> ⚠️ "지급 차단"을 **영구적 미지급**으로 구현하면 [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §1의 "정당한 사유 없는 지급 보류/거부 금지" 원칙과 충돌하는 것처럼 보일 수 있다. 그러나 **법정 한도 준수 자체가 "정당한 사유"** 이므로 충돌이 아니다 — 다만 이 점을 시스템·약관 문구에 명확히 해야 한다 ([LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §3).

- "차단"은 **정산 배치를 영구 폐기하는 것이 아니라, [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9의 "③ 법적 한도 검증 → ④ 세금 계산 → ⑤ 운영자 승인" 흐름에서 ③→④ 전이를 보류**하는 **Hard Gate**로 설계한다. 연도 누적 비율이 35% 이상으로 계산되면, 해당 정산배치는 `settlement_batches`의 "검증" 단계에 머무르며 ④/⑤ 단계로 자동 진행하지 않고, 지정 관리자(SuperAdmin/Compliance Admin)에게 검토를 요청한다.
- **초과분을 정확히 어떻게 처리할지(전체 배치 보류 vs 비례 축소(pro-rata) vs 초과를 유발한 마지막 항목만 보류)는 본 절의 범위 밖이며, [DECISIONS.md](DECISIONS.md) O-004로 별도 추적한다.** 본 엔진은 "감지하고 게이트를 거는 것"까지만 책임지고, 게이트 통과 후의 정확한 분배 로직은 O-004 확정 후 Settlement 모듈에 구현한다.
- [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §6의 기존 "정산 보류 조건"(개별 회원 단위 보류 — 환불대기/부정거래의심/계좌미등록)은 **회원 1명 단위**의 보류이며, 본 절의 한도 차단은 **배치/전체 회원 단위**의 보류다 — 두 메커니즘은 서로 다른 차원이며 함께 작동할 수 있다(개별 보류된 항목은 한도 계산의 분자에서도 제외해야 함, 구현 시 주의).

#### 8.1.4 공제조합 보고센터와의 연계 (단일 소스 원칙)

- 공제조합 보고센터([PRD.md](PRD.md) §5.7)가 생성하는 보고서의 "매출 총액/후원수당 총액/비율"은 **본 엔진의 연도 누적 `compliance_ratio_snapshots` 값을 그대로 읽어 사용**한다 — 보고서 생성 Job이 별도로 매출/후원수당을 재집계하지 않는다. 대시보드(§5.18)와 규제 보고서가 서로 다른 숫자를 보여주는 정합성 문제를 구조적으로 방지하기 위함이다.

#### 8.1.5 리딩 인디케이터 (조기 경보, 권고)

- 분류가 "후원수당"(O-059 Scenario A, [DECISIONS.md](DECISIONS.md) D-027 영향분석)으로 확정될 경우, 패키지 추천보너스+페어보너스의 패키지 매출 대비 비율은 **25%×(1+p)** (p=페어 성립률)로, **p가 40%만 넘어도 패키지 매출만 따로 보면 이미 35%를 초과**한다([DECISIONS.md](DECISIONS.md) D-027 참조). 전체 비율은 일반 매출과 블렌딩되어 완화되지만, **패키지 매출 비중**과 **페어 성립률** 자체를 별도 리딩 인디케이터로 대시보드(§5.18)에 노출할 것을 권고한다 — 블렌딩된 비율이 30%를 넘기 전에 이미 위험 신호를 줄 수 있다.

## 9. 아키텍처 검토 — 유니콘 스케일 확장 시 (Design Freeze 시점 검토)

현재 구조(web/api/worker/scheduler/redis 5서비스, [DATABASE.md](DATABASE.md) §7과 연계)를 회원 수·거래량이 매우 큰 규모("유니콘급")로 확장한다고 가정했을 때의 병목 가능성과 분리 필요성을 검토한다.

### 9.1 병목 가능성 (Bottleneck Risk)

| 영역 | 병목 시나리오 | 비고 |
|---|---|---|
| 단일 `worker` 서비스 | 본 라운드에서 Job 유형이 5종→9종으로 늘었다(§2.3). 보고서 생성처럼 느린 Job이 수당 계산처럼 시간에 민감한 Job을 지연시킬 수 있다 | 이미 [DECISIONS.md](DECISIONS.md) O-038에서 추적 중이나, Job 종류가 늘어난 지금 **우선순위 큐 분리가 더 시급**해짐 |
| 단일 `redis` | 캐시(휘발성, 자주 갱신)와 큐(BullMQ, 신뢰성 중요)가 같은 Redis 인스턴스를 공유 — 캐시 트래픽 폭증이 큐 처리 지연을 유발할 수 있음 | 캐시용/큐용 Redis 인스턴스 분리를 확장 시점의 1순위 후보로 권고 |
| 단일 Supabase Postgres | 회원 CRUD(OLTP)와 '내 조직'/공제조합 보고서/Rule Designer 시뮬레이션 같은 무거운 집계(OLAP성 쿼리)가 같은 DB를 공유 | 리포팅 전용 read replica 분리를 권고 ([DATABASE.md](DATABASE.md) §7.4와 동일 이슈) |
| `scheduler` 단일 인스턴스 가정 | cron 트리거가 중복 실행되지 않으려면 scheduler가 항상 단일 인스턴스로 떠 있어야 함 — Railway에서 이를 어떻게 보장할지 명시되어 있지 않음 | 배포 설정에서 인스턴스 수를 1로 고정하거나, 분산 락(leader election) 도입 필요 |
| `api`의 상태 비저장(stateless) 가정 | 현재 명시적으로 기술되어 있지 않으나, 수평 확장을 위해 api는 인스턴스 간 공유 상태(세션 등)를 로컬에 두지 않아야 함 | 본 라운드에서 원칙으로 명문화 (아래 9.2) |

### 9.2 분리 필요성 (Separation Recommendation)

1. **worker의 도메인별 분리**: Job 종류가 9종으로 늘어난 시점에서, 최소한 "계산형 Job(수당/정산/세금/프로모션)"과 "IO형 Job(알림발송/문서생성/보고서/3PL대조)"을 별도 worker 풀(같은 서비스 내 별도 큐든, 별도 Railway 서비스든)로 분리하는 것을 권고한다 — IO형 Job의 지연이 계산형 Job의 SLA에 영향을 주지 않도록.
2. **Redis 캐시/큐 분리**: 확장 시 가장 먼저 분리할 후보. 큐(BullMQ)는 내구성·순서가 중요하고 캐시는 처리량이 중요해 운영 특성이 다르다.
3. **읽기 전용 복제본(Read Replica)**: '내 조직' 집계(§DATABASE §3.11), 공제조합 보고서(§3.16), Rule Designer 시뮬레이션(§3.23) 등 무거운 조회는 read replica로 분리해 OLTP 경로를 보호.
4. **api 상태 비저장 원칙 명문화(확정 — 신규 원칙)**: `api` 인스턴스는 어떤 요청 상태도 로컬 메모리에 유지하지 않으며, 세션/캐시는 반드시 Redis 또는 Supabase를 통해 공유한다 — 수평 확장의 전제조건.
5. **국가 기준 데이터 파티셔닝**: KR이 먼저 커지고 다른 국가가 나중에 성장하는 구조이므로, `country_code` 기준 파티셔닝이 장기적으로 유리하다 ([DATABASE.md](DATABASE.md) §7.4와 동일).

> 위 분리 작업은 **지금 당장 수행할 필요는 없다** — 현재 5서비스 구조는 설계 단계의 "시작 구조"로 충분하며, 실제 트래픽이 병목 징후를 보일 때 단계적으로 분리하는 것을 권고한다. 단, 분리가 쉽도록 지금부터 "api 상태 비저장"과 "worker 내부 모듈 간 느슨한 결합"(§2.2 마지막 bullet)을 지키는 것이 중요하다.

## 10. Open Decisions

- RLS 적용 범위 (국가 스코프 필터링을 위해 RLS 채택 가능성 — country_code 기준 정책이 단순하여 유력 후보)
- REST vs GraphQL 최종 결정
- 환경 단계 수 및 명칭 — **O-148로 정식화([DECISIONS.md](DECISIONS.md) §2)**, Railway 환경 분리 개수·명명 및 Supabase 프로젝트 분리 여부
- 모니터링/로깅 도구 선정
- 3PL 연동 업체 및 연동 방식(API/EDI/파일)
- 조직 병합/분리/복구의 기능 정의 및 OrganizationTransfer 모듈 내 처리 방식 ([PRD.md](PRD.md) §5.16.8, [DECISIONS.md](DECISIONS.md) O-065)
- Compliance Admin / 지정 조직관리 관리자 임명 절차 및 정확한 권한 경계 (O-066)
- 조직이동 적용 배치의 정확한 실행 주기 (O-070), 익월 1일 기준 시점(O-068)/타임존(O-069)
- "중대한 컴플라이언스 이슈" 분류 기준 (O-071)
- scheduler의 cron 구현 방식 (Railway 자체 cron vs BullMQ repeatable job)
- Supabase 백업 주기/보존기간 및 별도 백업 정책 필요 여부
- worker 내 Job 종류(수당/정산/세금/프로모션/보고서)별 동시성·우선순위 처리 정책
- 마케팅 플랜이 국가별로 구조까지 달라질 수 있는지, 파라미터만 달라지는지
- 관리자 역할(기능별) 세부 목록 및 권한 매핑
- 공제조합 외 4개국의 동등 규제기관 및 보고 요건/주기
- Redis 캐시/큐 분리 시점 (§9.2)
- 리포팅용 read replica 도입 시점 (§9.2)
- scheduler 단일 인스턴스 보장 메커니즘 (Railway 설정 vs 분산 락)
- worker 도메인별(계산형/IO형) 분리 시점 — Job 9종으로 증가에 따라 O-038보다 우선순위 상향 검토
- 센터(Center) 구조 도입 여부
- Notification Center 채널별(KakaoTalk 등) 국가 확장 방식
- 전자서명 도입 방식 및 법적 효력
- Rule Designer 도입 여부 및 적용 범위
- CN 데이터 현지화 법규 대응 방안 ([DATABASE.md](DATABASE.md) §7.5) — **CN이 Reserved Country로 전환되어 우선순위 하향(D-023). CN 활성화 검토 시점에 재상향**
- **(O-004, 신규 영향분석)** 법적 한도(35%) 초과 감지 시 정확한 처리 방식 — 배치 전체 보류 / 비례 축소(pro-rata) / 초과를 유발한 마지막 항목만 보류, 3안 중 선택 필요 (§8.1.3)
- **(O-078)** 실시간 캐시(Redis)↔Postgres 정합화(reconciliation) Job의 실행 주기 (§8.1.1)
- TH/JP/US의 법정 후원수당 한도 및 자체 30/33/35%(또는 다른 값) 임계치 확정 시점 (O-045 연계, §8.1.2)
- 대규모 Bulk Action(§2.1.1)의 동기/비동기(Job) 처리 전환 임계값, 일괄작업 결과 추적용 별도 요약 테이블 필요 여부 (O-118)
- **(신규, D-062)** 상용 ERP 수준 Gap Analysis 결과 — RTO/RPO·DR 절차(O-144), 아웃바운드 Webhook(O-145), API 호출 쿼터(O-146), Multi-Tenant Job 격리(O-159), Redis 장애 시 DLQ 재처리(O-161) 등. 상세는 [GAP-ANALYSIS.md](GAP-ANALYSIS.md), [DECISIONS.md](DECISIONS.md) §2 참조
- **(신규, D-063)** API 버전관리/deprecation 정책(O-172), 서비스 상태페이지·장애공지(O-173), Multi-Tenant 온보딩/사용량모니터링/격리검증(O-170) — [API-SPEC.md](API-SPEC.md), [DEPLOYMENT.md](DEPLOYMENT.md) 신설 문서가 권장안을 제시했으나 최종 확정은 아님

> ~~수당/정산 시뮬레이션(dry-run) API의 동기/비동기 처리 방식~~ — **해소 (D-010)**: api는 Job 생성만 하고 worker가 비동기로 처리하는 것으로 확정.
