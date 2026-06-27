# API-SPEC.md — REST API 명세

> 상태: v0.10 (D-079 — Reward Policy 고도화 및 운영 시뮬레이션: Marketing Reward System(D-078)을 완성하는 보강 — **새 MLM 정책이 아니다.** DATABASE.md §3.64/PRD.md §5.87~§5.90 기반으로 §2.39 Marketing Reward(Reward Policy)에 Reward Formula Version 관리·Reward Simulation·Formula Test 엔드포인트를 추가 — `/v1/reward-policies/{id}/formula-versions` GET(목록/이력)·POST(신규 버전 생성, append-only — 기존 버전은 PUT/PATCH/DELETE 불가)/`/v1/reward-policies/{id}/formula-versions/{versionId}/activate`(POST, `is_active` 전환)/`/v1/reward-policies/{id}/simulate`(POST, 가상 입력값→예상 Point 미리보기, 결과 미저장)/`/v1/reward-policies/formula-test`(POST, 저장 전 Formula 초안 테스트, 결과 미저장)를 신설했다. 기존 `/v1/reward-policies` CRUD 응답의 `accrual_basis`/`accrual_method`/`accrual_rate`는 본 라운드부터 활성 Formula Version(§3.64)의 파생 캐시로 의미가 바뀐다(엔드포인트 경로/메서드 변경 없음). Simulation/Formula Test 모두 신규 테이블 없이 계산만 수행하며 실행 메타데이터는 기존 `audit_logs`에만 기록한다. §2.5 Compensation/Settlement는 본 라운드에서 전혀 변경하지 않는다. 신규 Business Rule 없음, 새 MLM 정책 없음, 신규 Open Decision 없음(O-210은 DATABASE.md §3.64/PRD.md §5.87 설계 시 이미 등록됨). D-078 — Marketing Reward System 및 Lifestyle Wallet 구조 개선: 새 MLM 정책이 아니라 기존 "+알파" 보너스(Travel/Car/자기계발)가 현금성 수당이 아닌 Marketing Reward Program 지급분임을 데이터/API 구조로 명확히 하는 라운드. DATABASE.md §3.63/PRD.md §5.83~§5.86 기반으로 §2.30 E-Wallet의 기존 지갑조회/거래내역/출금신청 엔드포인트에 `wallet_type`(CASH/LIFESTYLE_POINT) 응답 필드·`RESTORE`/`EXPIRE` 거래유형 비고를 추가하고 출금 계열 엔드포인트가 `LIFESTYLE_POINT` 지갑을 거부하도록 명시(엔드포인트 경로/메서드 변경 없음), §2.19 MarketingProgram의 기존 프로그램 신청 승인(`/v1/marketing-program-applications/{id}/approve`) 비고에 활성 Reward Policy 연결 시 `wallet_transactions`(USE) 부수효과 생성 비고를 추가(승인 절차 자체 불변), §2.39 Marketing Reward(Reward Policy)를 신설 — `/v1/reward-policies` CRUD·`/activate`·`/deactivate`(`reward_policies` 신규 1종 테이블)와 `/v1/lifestyle-wallet/balance`·`/transactions`(§2.30 지갑조회/거래내역 패턴을 `wallet_type=LIFESTYLE_POINT` 필터로 재사용하는 전용 진입점)·`/v1/reward-policies/{id}/accrual-report`(Report Builder Job 재사용)를 추가했다. §2.5 Compensation/Settlement는 본 라운드에서 전혀 변경하지 않으며 현금과 Lifestyle Point는 어떤 엔드포인트에서도 혼합되지 않는다. 신규 Business Rule 없음, 신규 Open Decision 없음(O-208/O-209는 DATABASE.md §3.63/PRD.md §5.83~§5.86 설계 시 이미 등록됨). D-077 — ERD 동기화·API Sequence·Event Flow 완성: 신규 기능 없음, 순수 문서 동기화/시각화 라운드. §0에 [API-SEQUENCE.md](API-SEQUENCE.md)(신규, 19개 Mermaid Sequence Diagram) 교차참조 추가. 본 문서 자체의 엔드포인트 표면(§2.1~§2.38)은 변경하지 않았다 — Source of Truth 검증 결과 기존 엔드포인트와 신규 API-SEQUENCE.md/EVENT-FLOW.md 간 충돌 없음 확인. D-076 — ERP 운영 생산성 및 관리자 UX 완성: 새 ERP 기능이 아니라 운영자(본사/Tenant 관리자/CS/물류/마케팅)의 일상 생산성 보강이며, **신규 Engine을 만들지 않고** Workflow Engine/Notification Center/Dashboard Builder/Report Builder/Scheduler Center/API Center/File Manager/Audit Center를 최대한 재사용한다. DATABASE.md §3.62에서 신설된 `admin_favorite_menus`/`saved_filters`/`notification_inbox_states`/`admin_notes`/`approval_delegations` 5종 테이블을 기반으로 §2.32 Global Search(`/v1/search`·`/v1/search/recent`, 신규 검색 인덱스 테이블 없는 federated 조회, `search_query_logs` 재사용)/§2.33 Approval Center(기존 §2.27 `/v1/admin/task-queue`를 ERP 전체 승인 대상으로 일반화 확장 + 신규 `/v1/approval-delegations` CRUD·`/v1/approval-sla-policies`)/§2.34 Approval History(`/v1/approval-history`·`/export`, 기존 승인 테이블의 approved_by/approved_at/decision federated 조회 + Report Builder Job)/§2.35 Favorite Menu·Saved Filter(`/v1/admin/favorite-menus`·`/v1/admin/saved-filters` CRUD, **O-199 해소**)/§2.36 Notification Inbox(`/v1/notification-inbox`·`/{id}`·`/bulk-read`, 기존 `notifications`/`notification_logs`(§2.11)에 대한 상태 오버레이)/§2.37 Operator Notes(`/v1/admin-notes` CRUD, 기존 주문 전용 `order_admin_notes`(§2.21)와 병렬·통합여부 O-206)/§2.38 Recent Activity 관리자(`/v1/admin/recent-activity`, §2.2 Customer Timeline과 동일 원리의 관리자 버전)를 신설했다. Global Search/Approval Center/Approval History/Recent Activity/Tenant Usage Dashboard(§2.22.1에 `tenant-usage` `type` 값 추가)/Personal Workspace/Command Palette(§2.32 재사용)/Universal Clipboard(DB 영향 없음)는 모두 기존 테이블 federated 조회이며 신규 테이블이 없다. §2.5 Compensation/Settlement와 Workflow Engine 자체 CRUD(`workflow_definitions`/`workflow_instances`/`workflow_step_actions`)는 본 라운드에서 전혀 변경하지 않는다. 신규 Business Rule 없음, 신규 Open Decision 없음(O-206/O-207은 DATABASE.md §3.62 설계 시 이미 등록됨). D-075 — 한국 공제조합 연동·E-Wallet·글로벌 결제(전부 Tenant별 선택 기능): DATABASE.md §3.59~§3.61 기반으로 §2.29 Compliance Transmission(공제조합 연동, `compliance_member_registrations`/`compliance_transmission_items` CRUD·전송대상조회·재전송·수동전송 Job)/§2.30 E-Wallet(`member_wallets`/`wallet_transactions`/`wallet_withdrawal_requests` 기반 지갑조회·거래내역·출금신청·승인반려·관리자보정·잠금해제, 잔액은 항상 원장에서 파생)/§2.31 Global Payment(기존 `external_api_connections`에 `country_code` 컬럼 확장 인용, 인바운드 Webhook 수신용 신규 `payment_webhook_events` 조회/재처리)를 신설. 연동 등록은 기존 `external_api_connections`/`external_api_call_logs`(§2.25)를 재사용하고 한국 결제수단(신용카드/계좌이체/가상계좌/무통장입금)은 D-069(§2.21) 기존 엔드포인트를 그대로 인용(재문서화하지 않음), Idempotency(O-054)/PG 토큰화(O-087)도 기존 Open Decision 재사용. §2.5 Compensation/Settlement는 본 라운드에서 변경하지 않으며 지갑 EARN 적립은 `settlement_items`를 읽기 전용으로 인용. 신규 Business Rule 없음, 신규 Open Decision 없음(O-201~O-205는 DATABASE.md §3.59~§3.61 설계 시 이미 등록됨). D-074 — Dynamic Board Engine: 코드 배포 없이 신규 게시판 종류를 추가할 수 있는 범용 게시판/게시물 엔진을 §2.28 Board Engine으로 신설 — DATABASE.md §3.58에서 신설된 `boards`/`board_categories`/`board_posts`/`board_post_comments`/`board_post_likes` 5종 테이블에 대한 CRUD/게시물/댓글/좋아요/RSS 엔드포인트를 추가하고, 첨부·이미지·동영상은 File Manager(`files`)/SEO는 `content_seo_metadata`/다국어는 `cms_translations`/조회수는 `content_view_events`/승인후게시는 Workflow Engine/예약게시는 Scheduler Center 기존 패턴을 재사용. 기존 CMS(§2.18, `cms_pages`/FAQ/popups/banners)는 본 라운드에서 변경하지 않으며 병렬 구조로 신설(통합 여부는 O-200 미확정). D-073 — 운영 UX 및 고객 경험 완성(최종 설계 라운드): Database 구조 변경 없이(신규 테이블/컬럼 0건) 기존 테이블 재사용만으로 §2.2 Member에 Customer Timeline 엔드포인트 신설, §2.14 CustomerService에 티켓 상세-Timeline 연계 비고 추가, §2.22/§2.22.1 Analytics에 Abandoned Cart `type` 값 및 SEO/Feed 오류 위젯 비고 보강, §2.22에 Dashboard Builder `/v1/dashboards` 신규 CRUD 진입점 추가(My Dashboard), §2.26 Cart에 Saved Cart 비고 추가, §0에 Quick Action 비고 추가. D-072 — 쇼핑몰 UX·알림·운영자 대시보드 완성: DATABASE.md §3.57에서 신설된 `carts`/`cart_items`/`product_price_alerts`와 기존 테이블 재사용 기능을 §2.11/§2.21/§2.22에 보강하고, §2.26 Cart/§2.27 AdminTaskQueue를 신설 — 장바구니/최근 본 상품/가격 알림/알림 템플릿 테스트발송·미리보기·재발송/배송 추적/운영자 대시보드 위젯/관리자 업무 Queue 엔드포인트 추가. D-070 — 쇼핑몰 운영 Phase 2 및 문서 동기화: D-069(쇼핑몰 운영 고도화/SEO·공유이미지, DATABASE.md §3.52~§3.56)와 D-070(Digital Marketing 연동/SEO Dashboard/이미지 최적화/상품 Feed, PRD.md §5.47~§5.50)에서 신설된 기능에 대한 개념 수준(설계 단계, 미구현) 엔드포인트를 §2.21~§2.25에 보강) · 최종 수정일: 2026-06-27 · 단계: 설계(Design)
> 전제 문서: [ARCHITECTURE.md](ARCHITECTURE.md), [DATABASE.md](DATABASE.md)
> 본 문서는 OpenAPI 산출물을 대체하지 않으며, 구현 단계에서 실제 OpenAPI yaml로 구체화하기 위한 설계 단계 명세다.

## 0. 범위와 원칙

- 본 문서가 다루는 대상은 [ARCHITECTURE.md](ARCHITECTURE.md) §2.2의 `api`(NestJS, 요청 처리 전용) 서비스가 노출하는 REST 엔드포인트다. `worker`/`scheduler`는 외부에 API를 노출하지 않는다 — `api`가 Job을 생성하고 상태/결과만 조회한다.
- **api는 계산하지 않는다(확정 — 위반 금지, [ARCHITECTURE.md](ARCHITECTURE.md) §1.1/§2.2/§3, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3).** 후원수당/정산/세금/프로모션 판정/보고서 생성 등 연산량이 큰 작업을 다루는 모든 엔드포인트는 본 문서에서 **"Job 생성 → 202 Accepted + Job id → 별도 상태조회 엔드포인트"** 패턴으로만 기술한다. 동기 계산 응답(200 + 계산결과)을 반환하는 것처럼 쓰지 않는다.
- 실제 값(필드 상세 타입, 정확한 enum, 정렬·필터 가능 필드 전체 목록 등)은 다수가 [DECISIONS.md](DECISIONS.md)에서 **미확정** 상태다. 본 문서는 ARCHITECTURE.md/DATABASE.md에 이미 정의된 모듈·테이블·흐름을 REST 형태로 옮기는 것이며, 존재하지 않는 필드/모듈을 새로 만들지 않는다.
- **Quick Action(관리자 대시보드 바로가기 메뉴, [PRD.md](PRD.md) §5.61, D-073)은 신규 API 표면이 아니다** — 주문조회/회원조회/상품등록/패키지등록/송장등록/환불승인/공지등록 등 Quick Action 메뉴 항목은 모두 본 문서에 이미 문서화된 기존 엔드포인트(§2.2/§2.3/§2.4/§2.9/§2.18 등)로 1:1 매핑되는 UI 바로가기일 뿐이며, 별도의 `/v1/quick-actions` 엔드포인트를 두지 않는다. 환불승인 Quick Action도 기존 `returns` 승인 흐름(§2.9)을 그대로 호출하며 우회 경로를 만들지 않는다.
- **본 문서가 정의하는 엔드포인트의 실제 호출 순서(누가 언제 무엇을 호출하는지)는 [API-SEQUENCE.md](API-SEQUENCE.md)(D-077, Mermaid Sequence Diagram)에 시각화되어 있다.** 본 문서는 "무엇이 존재하는지"(엔드포인트 표면)를, API-SEQUENCE.md는 "어떤 순서로 호출되는지"를 다룬다 — 두 문서는 서로 다른 관점이며 내용이 충돌하면 본 문서(엔드포인트 정의의 원천)가 우선한다.

## 1. 전역 컨벤션

### 1.1 Base URL 및 버저닝

```
https://api.fns.example.com/v1/{module-path}
```

- 모든 엔드포인트는 `/v1` 접두사를 가진다. 메이저 버전(브레이킹 체인지) 변경 시에만 버전을 올리며, 마이너 변경은 하위호환을 유지한다. **`/v1`→`/v2` 전환 시 구버전 sunset 기한·사전고지 절차는 미확정**([DECISIONS.md](DECISIONS.md) O-172) — 본 문서는 버전 prefix 자체만 정의하며, deprecation 운영 절차는 별도 확정이 필요하다.
- Railway 서비스 구조상 `api` 서비스 하나가 모든 모듈 경로를 단일 NestJS 앱(모듈형 모놀리스)으로 호스팅한다 ([ARCHITECTURE.md](ARCHITECTURE.md) §1).
- 국가별 동작이 달라지는 리소스(정산/수당/세금/프로모션 관련)는 URL에 국가를 넣지 않고, 대상 회원의 `country_code`를 서버가 조회해 적용한다 ([ARCHITECTURE.md](ARCHITECTURE.md) §8). 관리자 API는 RBAC의 `country_scope` 검사를 통해 접근 범위가 제한된다.

### 1.2 인증

- **Supabase Auth** 세션 기반. 클라이언트(`web`)는 Supabase Auth로 로그인 후 발급된 JWT를 `Authorization: Bearer {token}` 헤더로 `api`에 전달한다.
- `api`는 모든 요청에서 JWT를 검증하고(Supabase Auth 연동, [ARCHITECTURE.md](ARCHITECTURE.md) §2.2 Auth 모듈), 검증된 사용자의 역할(SuperAdmin/Compliance Admin/지정 조직관리 관리자/CountryAdmin/CenterAdmin/기능별 관리자/Partner)과 `country_scope`를 RBAC Guard에서 사용한다.
- **국가 스코프 검사**: CountryAdmin/기능별 관리자의 모든 요청은 RBAC 검사에 `country_scope` 비교를 포함하며, 본인 국가 외 데이터 요청은 거부(403)한다(SuperAdmin 제외) — 위반 시도는 `audit_logs`에 기록한다 ([ARCHITECTURE.md](ARCHITECTURE.md) §4).
- **조직 이동(OrganizationTransfer) 전용 role guard**: 일반 RBAC 통과 여부와 무관하게, 호출자가 SuperAdmin/Compliance Admin/지정 조직관리 관리자 중 하나가 아니면 인가 단계에서 즉시 403으로 거부한다 — 회원 계정에는 해당 엔드포인트 자체가 노출되지 않는다 ([ARCHITECTURE.md](ARCHITECTURE.md) §4, §7.1).
- `api`는 상태 비저장(stateless)이다 — 세션/토큰 검증 결과를 인스턴스 로컬 메모리에 캐시하지 않으며, 필요한 캐시는 Redis를 통해 공유한다 ([ARCHITECTURE.md](ARCHITECTURE.md) §9.2).

### 1.3 페이지네이션 (권장안 제시 — 미확정)

- ARCHITECTURE.md/DATABASE.md에 페이지네이션 방식이 명시적으로 확정되어 있지 않다. **미확정.**
- **권장안**: 목록 조회 엔드포인트는 offset 기반(`page`/`limit`)을 기본으로 채택한다. append-only 원장(`commission_records`/`settlement_items`/`audit_logs`/`organization_transfer_logs` 등)처럼 신규 행이 끝없이 추가되며 일관된 순서로 무한 스크롤이 필요한 조회는 cursor 기반(`cursor`/`limit`)을 권장한다.

| 파라미터 | 방식 | 설명 |
|---|---|---|
| `page` / `limit` | offset | 일반 CRUD 리소스(Member/Product/Order 등). `limit` 기본값 20, 최대값은 미확정 |
| `cursor` / `limit` | cursor | append-only 원장 조회, 무한 스크롤형 목록 |

응답 메타(offset 방식 예시):

```json
{
  "data": [ ... ],
  "meta": { "page": 1, "limit": 20, "total": 134, "total_pages": 7 }
}
```

응답 메타(cursor 방식 예시):

```json
{
  "data": [ ... ],
  "meta": { "next_cursor": "eyJpZCI6MTIzfQ==", "has_more": true }
}
```

- 최종 채택 방식, `limit` 상한, 두 방식의 혼용 허용 여부는 **미확정**([DECISIONS.md](DECISIONS.md) O-164) — 대량 리스트 그리드 가상화 기준(O-164)과 함께 확정해야 화면/API 계약이 어긋나지 않는다.

### 1.4 정렬

- 형식: `sort=field:asc` 또는 `sort=field:desc`. 복수 정렬 키는 콤마로 구분(`sort=created_at:desc,name:asc`).
- 정렬 가능 필드는 리소스마다 화이트리스트로 제한한다(임의 컬럼 정렬 금지 — SQL Injection/성능 보호). 리소스별 정렬 허용 필드 목록은 OpenAPI 구체화 단계에서 확정.
- 기본 정렬은 리소스마다 다르며 보통 `created_at:desc`.

### 1.5 필터링

- Query parameter 기반 등호 필터를 기본으로 한다: `?status=ACTIVE&country_code=KR`.
- 범위 필터는 접미사 컨벤션을 사용한다: `created_at_from`, `created_at_to` (또는 `created_at[gte]`/`created_at[lte]` 형태 — 최종 표기 방식은 **미확정**, 구현 단계에서 OpenAPI로 확정).
- 다중값 필터는 콤마 구분: `status=ACTIVE,DORMANT`.
- 검색어 필터는 `q` 파라미터를 공통으로 사용한다(리소스별 검색 대상 컬럼은 다름).
- 국가 스코프가 있는 관리자는 `country_code` 필터를 자신의 `country_scope` 밖 값으로 지정할 수 없다(요청 자체가 422/403으로 거부됨).

### 1.6 에러 응답 포맷 (공통 에러 스키마)

모든 에러 응답은 아래 공통 스키마를 따른다.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "요청 값이 유효하지 않습니다.",
    "details": [
      { "field": "email", "reason": "유효한 이메일 형식이 아닙니다." }
    ],
    "request_id": "req_8f1c2a3b",
    "timestamp": "2026-06-25T09:00:00Z"
  }
}
```

| HTTP Status | `error.code` 예시 | 설명 |
|---|---|---|
| 400 | `BAD_REQUEST` | 요청 형식 오류 |
| 401 | `UNAUTHENTICATED` | 인증 토큰 누락/만료/무효 |
| 403 | `FORBIDDEN` | RBAC/국가스코프/조직이동 role guard 위반 |
| 404 | `NOT_FOUND` | 대상 리소스 없음 |
| 409 | `CONFLICT` | 상태 충돌(예: 이미 처리된 변경 요청 재승인 시도) |
| 422 | `VALIDATION_ERROR` | 필드 검증 실패 |
| 429 | `RATE_LIMITED` | Rate limit 초과 (NestJS Guard 레벨, [ARCHITECTURE.md](ARCHITECTURE.md) §4) |
| 500 | `INTERNAL_ERROR` | 서버 오류 |
| 503 | `SERVICE_UNAVAILABLE` | Redis 등 의존 인프라 장애 — Job 생성 실패 시 포함 가능 |

- `request_id`는 분산 로그 추적용으로 모든 응답(성공/에러 공통)에 포함을 권장한다(헤더 `X-Request-Id`로도 노출 검토).
- 인가 거부(국가 스코프 위반, 조직이동 role guard 위반)는 `audit_logs`에 기록 대상이다 ([ARCHITECTURE.md](ARCHITECTURE.md) §4) — 이 기록은 api가 직접 수행하는 경량 기록이며 Job 생성 대상이 아니다(§2.2 Audit 모듈, "경량 — api에서 직접 기록").

### 1.7 비동기 Job 패턴 (핵심 — 모든 무거운 연산에 일관 적용)

> 근거: [ARCHITECTURE.md](ARCHITECTURE.md) §1.1, §2.2, §2.3, §3 — "**api는 request handler에서 무거운 계산을 하지 않는다. Job 생성만 담당한다**." [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3 "api(NestJS)의 request handler에 무거운 계산(후원수당/정산/세금/프로모션 판정/보고서 생성)을 직접 구현하지 않는다."

**적용 대상**: 후원수당(Compensation) 계산, 정산(Settlement) 배치 처리, 세금(원천징수) 계산, 법적 한도(35%) 정합화 배치, 프로모션/규칙 판정, 보고서 생성(공제조합 보고센터), 회원 민감 변경의 수당/정산 영향 분석(dry-run), 조직 이동 영향 분석, Rule Designer 시뮬레이션/발행, Notification 실제 발송, 회원 개인 문서(정산자료/지급명세서/원천징수자료) 생성, 3PL 정합성 대조 등 — [ARCHITECTURE.md](ARCHITECTURE.md) §2.3에 열거된 worker 처리 대상 전체.

**공통 흐름**:

```
1) 클라이언트 → api: POST .../jobs (또는 무거운 연산을 트리거하는 액션 엔드포인트)
2) api: 요청 검증(권한/입력) → Job 메타 생성 → redis(BullMQ)에 적재 → 즉시 응답
   응답: 202 Accepted + { "job_id": "...", "status": "QUEUED" }
3) worker: Job을 꺼내 실제 계산 수행 → 결과를 Supabase PostgreSQL에 기록
   (commission_records/settlement_items 등 append-only 원장은 worker만 기록 — DO-NOT-TOUCH.md §1.3)
4) 클라이언트 → api: GET .../jobs/{job_id} 로 상태/결과를 폴링
   (또는 webhook 콜백 — Job 등록 시 callback_url을 선택적으로 받는 것을 검토, 아웃바운드 Webhook 자체는 O-145로 미확정)
```

**공통 Job 상태조회 응답 스키마**:

```json
{
  "job_id": "job_5f8e2c1a",
  "job_type": "COMMISSION_CALCULATION",
  "status": "PROCESSING",
  "requested_by": "admin_user_id",
  "requested_at": "2026-06-25T01:00:00Z",
  "started_at": "2026-06-25T01:00:05Z",
  "completed_at": null,
  "result_ref": null,
  "error": null
}
```

| 필드 | 설명 |
|---|---|
| `status` | `QUEUED` / `PROCESSING` / `COMPLETED` / `FAILED` / `RETRYING` — Redis Job Tracking 기준([ARCHITECTURE.md](ARCHITECTURE.md) §2.5) |
| `result_ref` | 완료 시 결과를 조회할 수 있는 리소스 경로 또는 식별자(예: `settlement_batches/{id}`) — Redis는 source of truth가 아니므로 최종 결과는 항상 Supabase 리소스를 가리킨다 |
| `error` | `FAILED` 시 에러 요약 — 상세는 worker 로그(미확정 영역, [ARCHITECTURE.md](ARCHITECTURE.md) §6) |

**Idempotency**: Job 생성 요청(`POST .../jobs`)은 네트워크 재시도·중복 클릭으로 동일 요청이 두 번 도착할 수 있다. 클라이언트는 `Idempotency-Key` 헤더(요청당 고유값)를 전달하고, `api`는 동일 키로 재요청 시 새 Job을 생성하지 않고 기존 `job_id`를 그대로 반환해야 한다. **멱등성 키 저장 방식(Redis TTL vs Postgres 테이블)과 적용 대상 범위(전체 Job vs 금전 영향 Job만)는 미확정**([DECISIONS.md](DECISIONS.md) O-054) — 정산/수당 계산 Job처럼 중복 실행 시 중복 지급으로 이어질 수 있는 Job은 이 정책이 확정되기 전까지 worker 자체의 재계산 결과가 기존 `commission_records`/`settlement_items` 행과 동일한지 사후 대조하는 것을 최소 안전장치로 권장한다.

- Job 상태/결과 조회 엔드포인트(`GET /v1/jobs/{job_id}`)는 모듈 공통으로 둘지, 모듈별 하위 경로(`GET /v1/settlements/jobs/{job_id}`)로 둘지는 본 문서 작성 시점에 **미확정** — 아래 모듈 표/예시에서는 가독성을 위해 모듈별 하위 경로로 표기했다. 실제 구현은 OpenAPI 구체화 단계에서 공통/개별 여부를 확정한다.
- 클라이언트가 결과를 받는 방식(polling 전용 vs webhook 지원)은 [ARCHITECTURE.md](ARCHITECTURE.md) §3 "polling 또는 webhook"으로 양쪽 다 언급되어 있으나, 아웃바운드 Webhook 인프라 자체는 [GAP-ANALYSIS.md](GAP-ANALYSIS.md)/[DECISIONS.md](DECISIONS.md) O-145로 **미확정**이다. 본 문서는 polling을 1차 확정 패턴으로, webhook은 향후 확장 옵션으로 기술한다.

## 2. 모듈별 엔드포인트 표

표기 규칙:
- **Permission**은 [PRD.md](PRD.md) §5.6.4 역할 체계(SuperAdmin/Compliance Admin/지정 조직관리 관리자/CountryAdmin/CenterAdmin/기능별 관리자/Partner)를 기준으로 표기한다. "관리자(국가스코프)"는 CountryAdmin 등 country_scope 검사가 적용되는 역할을 의미한다.
- "**표준 CRUD 패턴 적용**"이라고 표기된 모듈/리소스는 `GET(목록)/GET(단건)/POST/PATCH/DELETE(소프트삭제)`가 §1의 전역 컨벤션(페이지네이션/정렬/필터/에러포맷)을 그대로 따르며 특이사항이 없음을 의미한다.
- 계산/판정/보고서 생성이 포함된 행은 모두 "Job 생성"으로 표기하며 동기 계산 응답을 암시하는 표현을 쓰지 않는다.

### 2.1 Auth

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/auth/session` | GET | 인증된 모든 사용자 | - | `member_id`/`role`/`country_scope` | Supabase Auth JWT 검증 결과 조회 |
| `/v1/auth/logout` | POST | 인증된 모든 사용자 | - | - | Supabase Auth 세션 종료 |
| `/v1/admin-roles` | 표준 CRUD 패턴 적용 | SuperAdmin | `role_name` | `id`/`role_name` | `admin_roles` 카탈로그([DATABASE.md](DATABASE.md) §3.15) |
| `/v1/admin-role-assignments` | 표준 CRUD 패턴 적용 | SuperAdmin | `admin_user_id`/`role_id`/`country_scope` | - | 역할 부여/회수, `audit_logs` 기록 필수([PRD.md](PRD.md) §5.6.4) |

### 2.2 Member

상세 예시는 §3.1 참조.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/members` | GET | 관리자(국가스코프) | - | 목록(페이지네이션) | 회원 CRUD — 회원 유형/상태/국가 필터 |
| `/v1/members` | POST | 공개(가입) 또는 관리자 | `member_type`/`country_code`/`sponsor_ref` | `member_id`/`status=PENDING_VERIFICATION` | 가입 — 추천 링크(§5.17)로 `sponsor_id` 자동 연결 |
| `/v1/members/{id}` | GET | 본인/관리자(국가스코프) | - | 회원 상세 | - |
| `/v1/members/{id}` | PATCH | 본인(제한적)/관리자 | 일반 정보 필드 | 갱신된 회원 | 민감 변경(상태/계좌/스폰서 등)은 본 엔드포인트가 아니라 §2.6 MemberLifecycle/§2.7 OrganizationTransfer를 통해서만 처리 — `members` 현재값 테이블 직접 수정 경로는 두지 않는다([DATABASE.md](DATABASE.md) §4 원칙 5) |
| `/v1/members/{id}/identity-profile` | GET/POST | 본인/관리자 | `member_type`/`identity_fields`/`document_refs` | `verification_status` | 회원 유형별 본인확인 서류 접수([DATABASE.md](DATABASE.md) §3.14) |
| `/v1/members/{id}/identity-profile/review` | POST | 관리자(국가스코프) | `decision`(승인/반려) | `verification_status` | 가입심사 — 승인 시 `status: PENDING_VERIFICATION → ACTIVE` |
| `/v1/members/{id}/sponsor-tree` | GET | 본인(읽기)/관리자 | `depth`(기본 5) | LINE1~LINE5 트리 | **읽기 전용** — Member 모듈은 스폰서 트리 조회만 담당([ARCHITECTURE.md](ARCHITECTURE.md) §2.2). 변경은 OrganizationTransfer만 가능 |
| `/v1/members/{id}/my-organization` | GET | 본인 | `period` | LINE1~5/조직매출/조직수당/조직성장 | '내 조직' 메뉴([PRD.md](PRD.md) §5.1.1, [DATABASE.md](DATABASE.md) §3.11) — 파생 데이터 조회, 실시간 계산 여부는 미확정 |
| `/v1/members/{id}/qualification-status` | GET | 본인/관리자 | - | 유니레벨 자격/패키지 자격 현황 | §5.1.2 — 2개의 독립 자격을 구분 표시(D-028) |
| `/v1/members/{id}/referral-link` | GET | 본인 | - | 추천링크/QR/클릭수/가입수 | §5.17.2 마이오피스 |
| `/v1/members/{id}/timeline` | GET | 본인/관리자(국가스코프) | `type`(필터, 다중 지정 가능)/`from`/`to` | 시간순 통합 타임라인 목록 — 각 항목 `type`(가입/주문/결제/배송/문의/반품/패키지구매/정산/Lifestyle/로그인/알림)/`occurred_at`/`summary`/`detail_ref` | **Customer Timeline**([PRD.md](PRD.md) §5.64, D-073) — **읽기 전용 federated 조회, 신규 테이블 없음.** `members.created_at`(가입)/`orders`(주문)/`order_payment_attempts`(결제)/`shipments`(배송)/`tickets`(§2.14, 문의)/`returns`·`exchange_requests`(반품/패키지 관련 교환)/`settlement_items`(§2.5, 정산)/`point_transactions`(§2.20, Lifestyle/포인트)/`member_activity_logs`(§3.22, `activity_type=LOGIN`인 로그인 이력)/`notifications`·`notification_logs`(§2.11, 알림)를 `occurred_at` 기준으로 시간 역순 병합한 결과를 반환한다. 각 기존 테이블에 대한 UNION 조회이며 응답 즉시 조립(실시간) vs 캐시 여부는 §2.22 운영자 대시보드와 동일하게 미확정 — "구현 단계에서 보정 필요". CS Center 티켓 상세 화면에서도 본 엔드포인트를 그대로 재사용한다(§2.14 비고 참조) |

### 2.3 Catalog

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/products` | 표준 CRUD 패턴 적용 | 관리자(상품관리)/공개(목록·상세 조회) | `name`/`price`/`is_active` | 상품 목록/상세 | §2 상세 예시는 §3.2 참조 |
| `/v1/products/{id}/images` | 표준 CRUD 패턴 적용 | 관리자(상품관리) | `image_type`/`device_type`/`image_ref`/`alt_text`/`sort_order` | `product_images` 행 | §3.50(D-060) — 상세는 §3.2 예시 |
| `/v1/products/{id}/images/{imageId}/reorder` | PATCH | 관리자(상품관리) | `sort_order` | - | 같은 `product_id`+`image_type` 범위 내 정렬만 허용 |
| `/v1/packages` | 표준 CRUD 패턴 적용 | 관리자(상품관리) | `name`/`price`/판매기간/국가범위 | 패키지 목록/상세 | `packages`([DATABASE.md](DATABASE.md) §3.24.1, D-033) |
| `/v1/packages/{id}/commission-policies` | 표준 CRUD 패턴 적용 | SuperAdmin/Compliance Admin | 추천수당비율/페어보너스금액·기간/유니레벨포함여부/자격부여여부 | 정책 상세 | `package_commission_policies` — 계산 로직 자체는 worker, 여기는 파라미터 CRUD만 |

### 2.4 Order

상세 예시는 §3.3 참조.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/orders` | GET | 본인/관리자(국가스코프) | 상태/기간 필터 | 주문 목록 | - |
| `/v1/orders` | POST | 본인(쇼핑몰 결제)/관리자 | `items[]`/`payment_method` | `order_id`/`status=PAID` + `202 Accepted`(수당 Job) | 주문 생성 — 매출 기록은 즉시 반영, 후원수당 계산은 비동기 Job([ARCHITECTURE.md](ARCHITECTURE.md) §2.2 Order) |
| `/v1/orders/{id}` | GET | 본인/관리자 | - | 주문 상세 | - |
| `/v1/orders/{id}/cancel` | POST | 본인(제한적 기간)/관리자 | `reason` | `status=CANCELLED` | 매출 차감은 별도 음수 차감 엔트리(확정 필요, [DATABASE.md](DATABASE.md) §3.3) |
| `/v1/orders/{id}/shipment` | GET | 본인/관리자 | - | 배송 상태/운송장 | §5.5 물류 연동 — 3PL 조회 |
| `/v1/orders/{id}/returns` | POST | 본인 | `items[]`/`reason` | `return_id` | 반품 접수 — 검수/환불은 Logistics 모듈이 처리 |
| `/v1/recurring-orders` | 표준 CRUD 패턴 적용 | 본인(정기배송센터) | `product_id`/`cycle`/`payment_method_id` | 정기배송 목록/상태 | 등록은 일반 쇼핑몰 상품상세에서, 조회/변경/해지/재개는 본 엔드포인트(§5.1.3.2/§5.1.3.3) — 실제 처리는 scheduler Job 트리거 + worker 수행 |
| `/v1/payment-methods` | 표준 CRUD 패턴 적용 | 본인(자동결제센터) | `pg_token` 등 | 결제수단 목록 | PG 토큰화 방식은 미확정(O-087) |

### 2.5 Compensation / Settlement

> **api는 계산을 수행하지 않는다 — 계산 Job 생성, 계산 결과 조회만 담당** ([ARCHITECTURE.md](ARCHITECTURE.md) §2.2). 상세 흐름(주문 → 수당 Job)은 §3.3 Order 예시에서 시연한다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/compensation/jobs` | POST | scheduler(내부)/관리자(수동 트리거) | `period`/`job_type`(COMMISSION_CALCULATION) | `202 Accepted` + `job_id` | worker가 LINE1~5 후원수당/제품판매수익/페어보너스/+알파 산정 배치 수행([ARCHITECTURE.md](ARCHITECTURE.md) §2.3) |
| `/v1/compensation/jobs/{jobId}` | GET | 관리자 | - | Job 상태/결과 | §1.7 공통 스키마 |
| `/v1/commission-records` | GET | 본인(자기 항목만)/관리자(국가스코프) | `member_id`/`period`/`commission_type` | `commission_records` 목록 | **조회만** — append-only 원장, worker만 기록([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3) |
| `/v1/settlement/jobs` | POST | scheduler(내부)/관리자(수동) | `period` | `202 Accepted` + `job_id` | 정산 배치 생성 Job — [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 ①~⑦ 흐름을 worker가 수행 |
| `/v1/settlement/jobs/{jobId}` | GET | 관리자 | - | Job 상태/결과 | - |
| `/v1/settlement-batches` | GET | 관리자(국가스코프) | `period`/`status` | 배치 목록 | `status`: 생성/검증/승인/지급완료 — 법적 한도 차단 시 "검증" 단계에 머무름([ARCHITECTURE.md](ARCHITECTURE.md) §8.1.3) |
| `/v1/settlement-batches/{id}` | GET | 관리자 | - | 배치 상세 | - |
| `/v1/settlement-batches/{id}/approve` | POST | SuperAdmin/Compliance Admin | - | `status=APPROVED` | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 ⑤단계 — 경량 상태 전이로 api가 직접 처리(계산 아님), 단 ③ 법적한도검증 통과 후에만 호출 가능 |
| `/v1/settlement-items` | GET | 본인/관리자 | `batch_id`/`member_id` | 정산 상세 목록 | append-only — worker만 기록 |
| `/v1/tax-withholdings` | GET | 본인/관리자 | `member_id`/`period` | 원천징수 내역 | 세금 계산 자체는 worker(§2.3) — 본 엔드포인트는 결과 조회만 |
| `/v1/compliance/ratio-snapshots` | GET | 관리자 | `period_type`(REALTIME/MONTHLY/ANNUAL_CUMULATIVE) | 비율/임계 단계 | §2.10 Compliance와 데이터 공유([ARCHITECTURE.md](ARCHITECTURE.md) §8.1.4) |

### 2.6 MemberLifecycle

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/member-change-requests` | POST | 본인(8종 신청)/관리자 | `change_type`(PROFILE_UPDATE/IDENTITY_TRANSFER/WITHDRAWAL/DORMANT/FORCED_WITHDRAWAL/REINSTATEMENT/BANK_ACCOUNT_CHANGE/CENTER_CHANGE)/`reason`/`requested_state` | `request_id`/`status=REQUESTED` + `202 Accepted`(영향분석 Job) | 요청 접수·Snapshot 생성은 api, **영향 분석은 Job**([ARCHITECTURE.md](ARCHITECTURE.md) §7) — SPONSOR_CHANGE는 enum에 없음(D-020) |
| `/v1/member-change-requests` | GET | 본인/관리자 | `status`/`change_type` | 목록 | - |
| `/v1/member-change-requests/{id}` | GET | 본인/관리자 | - | 상세(스냅샷/영향분석 결과 참조 포함) | - |
| `/v1/member-change-requests/{id}/impact-analysis` | GET | 관리자 | - | `change_impact_analyses` 결과 | 영향분석 Job 완료 후 결과 조회(§1.7 패턴) |
| `/v1/member-change-requests/{id}/approve` | POST | 관리자 | `e_signature_id`(IDENTITY_TRANSFER/WITHDRAWAL/FORCED_WITHDRAWAL 필수) | `status=APPROVED` → 적용 | 승인은 경량 상태전이(api 직접 처리), 승인 후 `members` 현재값 갱신 + 이력 추가 |
| `/v1/member-change-requests/{id}/reject` | POST | 관리자 | `reason` | `status=REJECTED` | - |
| `/v1/members/{id}/bank-accounts` | 표준 CRUD 패턴 적용 | 본인 | `bank_code`/`account_number`/`account_holder_name` | 계좌 목록(append-only + `is_current`) | 변경은 `member_change_requests`(BANK_ACCOUNT_CHANGE) 경유, 보류기간 적용 |

### 2.7 OrganizationTransfer

상세는 [ARCHITECTURE.md](ARCHITECTURE.md) §7.1, [PRD.md](PRD.md) §5.16 참조. **role guard로 SuperAdmin/Compliance Admin/지정 조직관리 관리자만 허용** — 그 외 호출은 인가 단계에서 즉시 거부(요청 자체가 생성되지 않음).

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/organization-transfers/reason-codes` | GET | 관리자(조직이동 권한) | - | 9종 사유 코드 목록 | `organization_transfer_reason_codes` |
| `/v1/organization-transfers` | POST | SuperAdmin/Compliance Admin/지정 조직관리 관리자 (role guard) | `member_id`/`new_sponsor_id`/`reason_code`/`attachment_refs[]` | `transfer_id`/`status=REQUESTED` + `202 Accepted`(영향분석 Job) | 사유코드·증빙 없이는 요청 생성 불가 |
| `/v1/organization-transfers/{id}/impact-analysis` | GET | 관리자(조직이동 권한) | - | 영향분석 결과 | Job 결과 조회(§1.7) |
| `/v1/organization-transfers/{id}/approve` | POST | SuperAdmin/Compliance Admin/지정 조직관리 관리자 | - | `status=APPROVED_SCHEDULED`/`effective_date` | 승인≠적용(D-022) — `sponsor_id`는 변경되지 않음 |
| `/v1/organization-transfers/{id}/approve-emergency` | POST | **SuperAdmin만** | `reason_code`(5종 긴급 사유만 허용) | `status=APPLIED`(즉시) | 더 좁은 권한(§5.16.5) — 적용일 예약 생략 |
| `/v1/organization-transfers/{id}/reject` | POST | 관리자(조직이동 권한) | `reason` | `status=REJECTED` | - |
| `/v1/organization-transfers/pending-applications` | GET | 관리자(조직이동 권한) | - | 승인됐으나 미적용 건 목록 | §5.16.6 "적용 예정 조직 이동 조회" |
| `/v1/organization-transfers/{id}` | GET | 관리자(조직이동 권한) | - | 상세(전후 비교 포함) | 실제 `sponsor_id` 갱신은 api가 아니라 scheduler 트리거 + worker 배치 수행([ARCHITECTURE.md](ARCHITECTURE.md) §7.1.1) — 본 엔드포인트는 조회만 |

### 2.8 Compliance

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/compliance/reports/jobs` | POST | 관리자(수동 생성)/scheduler(내부) | `report_type`/`period` | `202 Accepted` + `job_id` | 공제조합 보고센터 — worker가 회원/매출/후원수당 집계해 생성([PRD.md](PRD.md) §5.7) |
| `/v1/compliance/reports/jobs/{jobId}` | GET | 관리자 | - | Job 상태/결과 | - |
| `/v1/compliance/report-submissions` | GET | 관리자(국가스코프) | `period`/`status` | `compliance_report_submissions` 목록 | 제출 이력 조회 |
| `/v1/compliance/report-submissions/{id}/submit` | POST | 관리자 | - | `status=제출완료` | 경량 상태전이(api 직접 처리) |
| `/v1/compliance/dashboard` | GET | 관리자 | `country_code` | 현재/예상 비율, 임계단계, 리딩 인디케이터 | §5.18 — `compliance_ratio_snapshots` 조회만, 계산은 worker(§8.1) |
| `/v1/compliance/thresholds` | 표준 CRUD 패턴 적용 | SuperAdmin/Compliance Admin | `country_code`/30·33·35% 등 | 국가별 임계치 | `compliance_thresholds` — KR 확정, TH/JP/US는 미설정(O-045) |

### 2.9 Logistics

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/warehouses` | 표준 CRUD 패턴 적용 | 관리자(물류담당) | `name`/`type`(자사/3PL) | 창고 목록 | - |
| `/v1/inventory-items` | GET | 관리자(물류담당) | `warehouse_id`/`product_id` | 현재 재고(파생값) | `inventory_ledger` 합산 파생 — 직접 수정 불가 |
| `/v1/inventory-ledger` | GET/POST | 관리자(물류담당) | `type`(입고/출고/반품/환수/폐기)/`quantity` | 원장 행 | append-only, 무거운 3PL 정합 대조는 별도 Job |
| `/v1/inventory-reconciliation/jobs` | POST | scheduler(내부) | `warehouse_id` | `202 Accepted` + `job_id` | 3PL 재고-내부 재고 정합성 대조([ARCHITECTURE.md](ARCHITECTURE.md) §2.7) — worker 수행 |
| `/v1/shipments` | 표준 CRUD 패턴 적용 | 본인(조회)/관리자(물류담당) | `order_id`/`carrier` | 운송장/배송상태 | 3PL 연동 호출은 api가 접수만(경량 호출), 정합성 대조는 Job |
| `/v1/returns` | 표준 CRUD 패턴 적용 | 본인(접수)/관리자(검수) | `order_id`/`items[]`/`reason` | `return_id`/`status` | 청약철회권 연계([LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §4) |
| `/v1/inventory-recoveries` | GET/POST | 관리자(물류담당) | `member_id`/`change_request_id` | 환수 내역 | 탈퇴/강제탈퇴 승인 시 MemberLifecycle이 트리거 |

### 2.10 Audit

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/audit-logs` | GET | 관리자(국가스코프) | `actor`/`entity`/`action`/기간 | `audit_logs` 목록(cursor 권장) | 경량 — api가 도메인 변경 시 직접 기록(Job 아님, [ARCHITECTURE.md](ARCHITECTURE.md) §2.2 Audit) |
| `/v1/audit-logs/{id}` | GET | 관리자 | - | 상세(변경 전/후 JSON) | - |

### 2.11 Notification

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/notifications/jobs` | POST | 관리자/시스템(내부 트리거) | `channel`(Email/SMS/KakaoTalk/Push)/`template_id`/`recipient_ids[]` | `202 Accepted` + `job_id` | 발송 **요청**만 접수, 실제 발송은 worker([PRD.md](PRD.md) §5.12) |
| `/v1/notifications/jobs/{jobId}` | GET | 관리자 | - | 발송 Job 상태 | - |
| `/v1/notification-templates` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `channel`/`locale`/`country_code`/`subject_template`/`body`(`content_template`)/`is_active` | 템플릿 목록 | `subject_template`(제목, EMAIL 채널용 — SMS/PUSH는 nullable)/`is_active`(활성/비활성 토글)은 신규 컬럼([DATABASE.md](DATABASE.md) §3.57, D-072) |
| `/v1/notification-templates/{id}/test-send` | POST | 관리자(CMS) | `channel`/`recipient`(테스트 수신처)/`sample_variables`(placeholder 치환값) | `202 Accepted` + `job_id` | 실제 발송 채널로 테스트 1건만 발송 — §1.7 Job 패턴 재사용(경량이나 외부 발송 채널 호출이라 Job으로 통일), 발송 이력은 `notification_logs`에 일반 발송과 동일하게 기록되되 테스트 여부 구분 컬럼은 미확정(O-199와 별개 — 후속 확인 필요) |
| `/v1/notification-templates/{id}/preview` | GET | 관리자(CMS) | `sample_variables`(선택, 미지정 시 기본 placeholder 표기) | 렌더링된 `subject`/`body` | **순수 렌더링, 부수효과 없음** — `notification_logs`/실제 발송 채널 호출 없이 `subject_template`/`content_template`의 placeholder만 치환해 반환 |
| `/v1/notification-logs` | GET | 관리자/본인(자기 수신내역) | `status`(성공/실패)/기간 | 발송/실패 이력 | 실패 시 재시도는 Redis Retry([ARCHITECTURE.md](ARCHITECTURE.md) §2.5) |
| `/v1/notification-logs/{id}/resend` | POST | 관리자 | - | `202 Accepted` + `job_id` + `resent_from_log_id` | 기존 로그를 그대로 재발송 — 신규 발송 Job을 생성하고 결과 로그의 `notification_logs.resent_from_log_id`가 원본 로그를 참조([DATABASE.md](DATABASE.md) §3.57) |

### 2.12 Center

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/centers` | 표준 CRUD 패턴 적용 | SuperAdmin/CountryAdmin | `country_id`/`name` | 센터 목록 | 도입 여부 미확정([PRD.md](PRD.md) §5.9) |
| `/v1/members/{id}/center-change-requests` | POST | 본인 | `target_center_id` | `request_id` + `202 Accepted` | MemberLifecycle과 동일 패턴(CENTER_CHANGE) — 영향분석은 수당 계산에 영향 없음(센터는 집계 차원, [DATABASE.md](DATABASE.md) §4 원칙 8)이므로 경량 승인일 가능성이 높으나 최종 처리 절차는 미확정 |

### 2.13 DocumentCenter

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/documents` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `category`(정책/보상플랜안내/교육자료)/`country_codes`/`visibility` | 문서 목록/버전 | 약관/교육자료 등 업로드형([PRD.md](PRD.md) §5.10) |
| `/v1/documents/{id}/versions` | GET | 관리자/본인(공개 문서) | - | 버전 이력 | `document_versions` |
| `/v1/members/{id}/personal-documents/jobs` | POST | 본인/관리자 | `document_type`(정산자료/지급명세서/원천징수자료) | `202 Accepted` + `job_id` | **생성 요청만** — 실제 생성은 worker가 `settlement_items`/`tax_withholdings` 기반으로 수행 |
| `/v1/members/{id}/personal-documents` | GET | 본인 | `document_type`/기간 | 생성된 문서 목록 | Job 완료 후 결과 조회 |

### 2.14 CustomerService

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/tickets` | 표준 CRUD 패턴 적용 | 본인(생성)/관리자(처리) | `ticket_type`(1:1문의/정산문의/수당문의/회원문의/명의변경문의/이의신청/불만접수) | 티켓 목록/상세 | 티켓 상세(단건 GET) 응답에는 해당 회원의 `/v1/members/{id}/timeline`(§2.2, D-073) 결과가 함께 임베드된다 — **신규 엔드포인트/데이터 모델 아님**, 상담원이 문의 처리 중 회원의 가입/주문/결제/배송/반품/정산/로그인/알림 이력을 한 화면에서 볼 수 있도록 기존 Customer Timeline을 그대로 참조([PRD.md](PRD.md) §5.66, D-073) |
| `/v1/tickets/{id}/messages` | 표준 CRUD 패턴 적용 | 본인/관리자 | `content`/`attachment_refs[]` | 메시지 목록 | - |
| `/v1/tickets/{id}/link-change-request` | POST | 관리자 | `change_request_id` | 연결됨 | 명의변경 문의/이의신청을 `member_change_requests`와 연결([DATABASE.md](DATABASE.md) §3.18) |

### 2.15 ESignature

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/e-signatures` | POST | 본인 | `target_type`(IDENTITY_TRANSFER/WITHDRAWAL/FORCED_WITHDRAWAL 등)/`target_id`/`signature_payload` | `signature_id` | 경량 처리(api 직접 — 무거운 계산 아님, [ARCHITECTURE.md](ARCHITECTURE.md) §2.2 ESignature) |
| `/v1/e-signatures/{id}/verify` | GET | 관리자 | - | 검증 결과 | `member_change_requests` 승인 선행조건 검사 |
| `/v1/consent-history` | GET/POST | 본인/관리자 | `consent_type`(약관/개인정보/MARKETING_OPT_IN) | 동의 이력 | - |

### 2.16 ActivityLog

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/members/{id}/activity-logs` | GET | 본인 | `activity_type`(로그인/주문/수당/정산)/기간 | 활동 목록 | 경량 기록(api 직접) — `audit_logs`와는 목적이 다른 별도 로그([PRD.md](PRD.md) §5.14) |

### 2.17 RuleDesigner

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/rule-drafts` | 표준 CRUD 패턴 적용 | SuperAdmin/Compliance Admin | `target_table`(marketing_plan_versions 등)/`draft_definition` | 초안 목록/상세 | 도입 여부 미확정([PRD.md](PRD.md) §5.15) |
| `/v1/rule-drafts/{id}/simulate` | POST | SuperAdmin/Compliance Admin | - | `202 Accepted` + `job_id` | **시뮬레이션(dry-run)은 Job** — worker가 법적 한도 등 하드 제약 검증 수행 |
| `/v1/rule-drafts/{id}/simulation-results/{jobId}` | GET | 관리자 | - | 시뮬레이션 결과 | Job 결과 조회 |
| `/v1/rule-drafts/{id}/publish` | POST | SuperAdmin/Compliance Admin | - | `202 Accepted` + `job_id` | **발행도 Job** — 시뮬레이션 통과 + 승인 후에만 `marketing_plan_versions` 등에 반영, 검증 실패 시 발행 거부 |
| `/v1/rule-publish-history` | GET | 관리자 | - | 발행 이력 | `rule_publish_history` |

### 2.18 CMS

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/cms/pages` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `page_type`/`slug`/`title`/`body`/`country_codes` | 페이지 목록/상세 | `cms_pages`(§3.33) |
| `/v1/cms/faq-categories` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `name`/`sort_order` | 카테고리 목록 | - |
| `/v1/cms/faq-items` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `category_id`/`question`/`answer`/`scope_type`/`scope_id` | FAQ 목록 | `scope_type=MARKETING_PROGRAM`이면 `scope_id`로 프로그램 연결 |
| `/v1/cms/popups` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `placement`/`image_ref`/노출기간 | 팝업 목록 | 노출 빈도 옵션 미확정 |
| `/v1/cms/banners` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `placement`(쇼핑몰메인슬라이드/프로모션/이벤트/회원몰프로그램)/노출기간/`link_url` | 배너 목록 | 범용 배너 — Lifestyle Program 전용 아님([PRD.md](PRD.md) §5.1.5.2) |
| `/v1/cms/translations` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `content_type`/`content_id`/`locale`/`field_name`/`translated_value` | 번역 오버레이 | 번역 없는 locale은 기본 언어 폴백 |

### 2.19 MarketingProgram

상세 흐름은 [PRD.md](PRD.md) §5.20/§5.22/§5.24 참조.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/marketing-programs` | 표준 CRUD 패턴 적용 | 관리자(CMS/마케팅) | `program_code`/`category`/`links_to_compensation`/`requires_application` | 프로그램 목록/상세 | `marketing_programs`(§3.34) |
| `/v1/marketing-programs/{id}/products` | 표준 CRUD 패턴 적용 | 관리자(CMS/마케팅) | `product_id`/`display_label`/`sort_order` | 연결 상품 목록 | `marketing_program_products`(N:N) |
| `/v1/marketing-programs/{id}/applications` | POST | 본인 | - | `application_id`/`status=APPLIED` | `requires_application=true`인 프로그램만 신청 가능 |
| `/v1/marketing-programs/{id}/applications` | GET | 본인/관리자 | `status` | 신청 목록 | - |
| `/v1/marketing-program-applications/{id}/approve` | POST | 관리자 | - | `status=APPROVED` | 승인 처리는 경량이라 **api에서 직접 수행**([ARCHITECTURE.md](ARCHITECTURE.md) §2.2 MarketingProgram). **D-078**: 해당 Program에 연결된 활성(`is_active=true`) `reward_policies`(§2.39, §3.63)가 있으면, 본 승인 처리의 부수효과로 `wallet_transactions`(`USE`, `wallet_type=LIFESTYLE_POINT`, `source_type=PROGRAM_APPLICATION`, `source_id`=본 신청 id) 1건이 함께 생성된다 — 승인 엔드포인트의 경로/메서드/승인 절차 자체는 변경 없음 |
| `/v1/marketing-program-applications/{id}/complete` | POST | 관리자 | - | `status=COMPLETED` | 완료 시 §2.20 Point 적립 트리거 가능(프로그램 정책에 따름) — 트리거 자체는 경량, 무거운 통계 집계만 Job |
| `/v1/marketing-programs/stats/jobs` | POST | 관리자 | `program_id`/`period` | `202 Accepted` + `job_id` | 무거운 통계 집계는 worker로 위임 |

### 2.20 Point

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/members/{id}/point-account` | GET | 본인/관리자 | - | `available_balance`/`pending_balance` | 조회만(파생 캐시) |
| `/v1/point-transactions` | GET | 본인/관리자 | `transaction_type`/`source_type`/기간 | 원장 목록(cursor 권장) | append-only |
| `/v1/point-transactions/use-requests` | POST | 본인 | `amount`/`purpose` | `transaction_id`/`status=USE_REQUEST` | 사용신청 접수는 api(경량), 적립/만료 등 배치성 처리는 worker(§2.3) |
| `/v1/point-transactions/{id}/approve` | POST | 관리자 | - | `status=USE_APPROVED` | `point_policies.requires_approval_for_use` 기준 |
| `/v1/point-policies` | 표준 CRUD 패턴 적용 | SuperAdmin | `country_code`/`default_expiry_months` | 정책 목록 | `point_policies`(버전관리) |

### 2.21 Shop

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/products/{id}/reviews` | 표준 CRUD 패턴 적용 | 본인(작성)/공개(조회) | `rating`/`content`/`image_refs[]`/`order_id` | 리뷰 목록 | 구매인증 리뷰 여부 미확정 |
| `/v1/products/{id}/inquiries` | 표준 CRUD 패턴 적용 | 본인(작성)/관리자(답변) | `question`/`is_secret` | 문의 목록 | CS Center 티켓과 별개 |
| `/v1/wishlists` | 표준 CRUD 패턴 적용 | 본인 | `product_id` | 찜 목록 | - |
| `/v1/recently-viewed-products` | GET | 본인 | `limit` | 최근 본 상품 목록(조회순) | `recently_viewed_products`(기존 테이블, [DATABASE.md](DATABASE.md) §3.35) — 본 라운드(D-072)에서 추가되는 것은 엔드포인트뿐, 테이블은 기존. 비회원 처리(클라이언트 저장 권고)는 미확정 |
| `/v1/products/{id}/price-alerts` | POST/DELETE | 본인(회원) | `target_price` 또는 할인율 기준 | `alert_id` | `product_price_alerts`([DATABASE.md](DATABASE.md) §3.57, D-072) — `restock_notifications`와 동일한 "상품 알림 신청" 패턴, 트리거가 가격이라는 점만 다름. 트리거 기준이 절대가 vs 할인율인지는 **미확정**(O-198) — 확정 전까지 요청 필드는 `target_price`(절대가)를 잠정 기본으로 받되, 알림 발송 자체는 §2.11 `notifications/jobs`와 동일한 비동기 패턴(스케줄러가 가격 변동 감지 시 트리거)을 따른다 |
| `/v1/coupons` | 표준 CRUD 패턴 적용 | 관리자(마케팅) | `code`/`discount_type`/`discount_value`/유효기간 | 쿠폰 목록 | - |
| `/v1/coupons/{id}/issue` | POST | 관리자/시스템(프로그램 완료 트리거) | `member_ids[]` | 발급 결과 | `coupon_issuances` — 대량 발급은 Bulk Action 패턴(§5.44.6, 기존 단건 API 재사용) |
| `/v1/coupons/issuances/{id}/use` | POST | 본인(결제 시) | `order_id` | `status=USED` | - |
| `/v1/product-bundles` | 표준 CRUD 패턴 적용 | 관리자(상품관리) | `name`/`bundle_type`/판매기간 | 번들 목록/상세 | `product_bundles`([DATABASE.md](DATABASE.md) §3.54, D-069) — MLM 패키지 엔진(`packages`)과 별개, 보상플랜·페어보너스 무관 |
| `/v1/product-bundles/{id}/items` | 표준 CRUD 패턴 적용 | 관리자(상품관리) | `product_id`/`option_combination_id`/`quantity`/`sort_order` | 번들 구성품 목록 | `product_bundle_items` — 번들 할인과 쿠폰/회원할인 중복 적용 규칙은 미확정(O-191) |
| `/v1/products/{id}/comparisons` | POST/DELETE | 본인(회원) | `product_id` | `comparison_id` | `product_comparisons`([DATABASE.md](DATABASE.md) §3.54) — 비회원 처리(서버 vs 클라이언트 저장)는 미확정(O-188) |
| `/v1/members/{id}/comparisons` | GET | 본인 | - | 비교함 상품 목록 | - |
| `/v1/inventory-items/{id}/hold` | POST/DELETE | 관리자(물류담당)/시스템(주문 생성·결제완료 내부 트리거) | `order_id`/`quantity` | hold 상태 | 재고 예약(Hold) 개념은 이미 §2.9 Logistics `inventory-ledger`에 내포 — 본 행은 Shop 모듈에서의 진입점만 추가. 예약→확정차감→실패시 해제 절차/창고 우선순위는 미확정(O-127) |
| `/v1/orders/{id}/exchange-requests` | POST | 본인 | `order_item_ids[]`/`reason`/`exchange_to_option_combination_id` | `exchange_id`/`status=REQUESTED` | 부분교환 — `exchange_requests`+`exchange_items`([DATABASE.md](DATABASE.md) §3.53, D-069). 반품 상태머신(O-129)과 통합할지 별도 분리할지는 미확정(O-180). 재고 정합성은 반품입고확인+신규출고 단일 트랜잭션(BR-050) |
| `/v1/exchange-requests/{id}` | GET | 본인/관리자 | - | 교환 요청 상세 | - |
| `/v1/exchange-requests/{id}/approve` | POST | 관리자(물류담당) | - | `status=APPROVED` | 경량 상태전이(api 직접 처리) — 실제 재고 트랜잭션 수행은 BR-050 규칙을 따름 |
| `/v1/orders/{id}/merge` / `/v1/orders/{id}/split` | POST | 관리자 | `target_order_ids[]` 또는 `split_items[]` | `order_merge_logs`/`order_split_logs` 행 | 매출 합계 일치 규칙은 BR-049. 허용 범위(병합/분리 가능 상태·시점)는 미확정(O-179) |
| `/v1/orders/{id}/admin-notes` | 표준 CRUD 패턴 적용 | 관리자 | `content`/`related_ticket_id` | 메모 목록 | `order_admin_notes` — CS Center 티켓(§2.14)과 `related_ticket_id`로 연결, 중복 메모 구조 아님 |
| `/v1/orders/{id}/shipment/change-logs` | GET | 본인/관리자(물류담당) | - | 송장/배송사 변경 이력 | `shipment_change_logs`([DATABASE.md](DATABASE.md) §3.53) |
| `/v1/shipments/{id}/tracking` | GET | 본인/관리자(물류담당) | - | `courier_name`/`tracking_no`/`status`(`PREPARING`준비중/`DISPATCHED`출고완료/`IN_TRANSIT`배송중/`DELAYED`지연/`HOLD`보류/`DELIVERED`배송완료)/`timeline`(상태 변경 이력) | §2.4 `/v1/orders/{id}/shipment`의 배송 추적 상세 진입점 — `shipments`/`shipment_items`(§3.10) 컬럼 명료화([DATABASE.md](DATABASE.md) §3.57)와 [STATE-MACHINE.md](STATE-MACHINE.md) §17(D-072 신규)을 반영. 택배사 추적페이지 URL은 `courier_name` 기준 고정 URL 패턴으로 응답 시점에 조합(저장 데이터 아님). 지연 판정 기준(시간/단계)은 미확정 |
| `/v1/payment-methods/{id}/payment-attempts` | GET | 본인/관리자 | `order_id`/`status` | 결제 시도/재시도 이력 | `order_payment_attempts`(일반 주문 PG 실패/재결제) — 정기배송 자동결제 재시도는 이미 O-086으로 별도 등록. 가상계좌/무통장입금 도입 여부는 미확정(O-181) |
| `/v1/orders/{id}/payment-splits` | GET | 본인/관리자 | - | 복합결제 분할 내역 | `order_payment_splits`(포인트+카드 등) — 부분실패 처리 정책은 미확정(O-182) |
| `/v1/virtual-account-issuances` | GET | 본인/관리자 | `order_id` | 가상계좌 발급 내역 | `virtual_account_issuances` — 도입 자체는 미확정(O-181), 도입 전제 시 사용할 경로만 선반영 |
| `/v1/bank-transfer-payments` | GET | 관리자 | `order_id`/`match_status` | 무통장입금 매칭 내역 | `bank_transfer_payments` — 도입 자체는 미확정(O-181) |
| `/v1/shipping-fee-settlements` | GET | 관리자(물류담당) | `period`/`warehouse_id` | 배송비 정산 내역 | `shipping_fee_settlements` — 정산 시점 정책 버전 스냅샷 고정(BR-051). MLM/후원수당 정산과 무관. 도입 여부/실행주기는 미확정(O-184) |

### 2.22 Analytics

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/analytics/dashboards/{type}` | GET | 관리자(국가스코프) | `period`/`country_code` | 집계 결과 | 실시간 쿼리 또는 스냅샷 캐시 조회, 방식 미확정([ARCHITECTURE.md](ARCHITECTURE.md) §2.2 Analytics). **운영자 대시보드(D-072)도 본 엔드포인트를 그대로 재사용** — 신규 엔드포인트 형태 없이 `{type}` 값만 아래 §2.22.1 표처럼 추가, Dashboard Builder의 기존 `data_source`/`widget_type` 패턴([DATABASE.md](DATABASE.md) §3.43) 그대로 재사용. 신규 테이블 없음([DATABASE.md](DATABASE.md) §3.57). **D-073에서 `abandoned-cart` `type` 값 추가**(§2.22.1 표 참조) |
| `/v1/analytics/reports/jobs` | POST | 관리자 | `report_type`/`period` | `202 Accepted` + `job_id` | 집계 자체가 무거우면 worker로 위임 |
| `/v1/analytics/reports/jobs/{jobId}` | GET | 관리자 | - | Job 상태/결과 | - |
| `/v1/dashboards` | 표준 CRUD 패턴 적용 | 관리자(본인) | `name`/`is_shared`/`widgets[]`(`widget_type`/`data_source`/`config`/`position`) | 대시보드 목록/상세 | Dashboard Builder([DATABASE.md](DATABASE.md) §3.43) `dashboard_definitions`/`dashboard_widgets`에 대한 최초 CRUD 진입점 — 본 문서에 그동안 §2.22/§2.27이 위젯 종류만 보강해왔을 뿐 정의 자체의 CRUD 엔드포인트가 없었기에 본 라운드(D-073)에서 추가. 신규 컬럼 없음 — 기존 `owner_admin_id`/`is_shared`/`position`(JSON) 그대로 노출 |
| `/v1/dashboards` | GET | 관리자(본인) | `mine`(기본 true) | 본인 소유 대시보드 목록 우선 | **My Dashboard**([PRD.md](PRD.md) §5.65, D-073) — 신규 엔드포인트가 아니라 위 CRUD 목록 조회가 기존 `dashboard_definitions.owner_admin_id`로 자동 필터링되고 기본 뷰로 채택되는 것뿐, 위젯 배치는 기존 `dashboard_widgets.position`(JSON) 그대로 사용 — 신규 테이블/컬럼 없음 |
| `/v1/analytics/dashboards/seo` | GET | 관리자(상품관리/CMS) | `product_id`/`period` | 상품별 SEO 점수(파생)/OG·Meta·Schema 설정여부(파생)/공유클릭·SNS공유횟수/검색유입·클릭·노출수·CTR·검색어순위 | SEO 운영 Dashboard([PRD.md](PRD.md) §5.48, D-070) — Dashboard Builder(§3.43)/Report Builder(§3.44) 재사용, 신규 위젯 종류만 추가. SEO 점수/설정여부는 `product_seo` 필드 채움률·NULL 여부를 조회 시점에 파생 계산(별도 점수 컬럼 없음). 공유클릭/SNS공유횟수는 `content_click_events.click_type` SOCIAL_SHARE 후보값 사용 여부가 미확정(O-194)이라 그 값이 비어있으면 해당 지표만 "미확정(후속 확인 필요)"으로 응답. 검색유입/클릭/노출수/CTR/순위는 §2.25 Digital Marketing의 GSC/Naver Search Advisor 연동 결과 위젯화 — 이 지표를 DB 캐시 vs 실시간 외부 API 호출로 가져올지는 미확정(O-195) |

#### 2.22.1 운영자 대시보드(D-072, D-073, D-076) — `/v1/analytics/dashboards/{type}` 신규 `{type}` 값

> [PRD.md](PRD.md) §5.56~§5.58(D-072)/§5.62(D-073)/§5.76(D-076), [DATABASE.md](DATABASE.md) §3.57 기반. 신규 엔드포인트 형태 없음 — 위 `/v1/analytics/dashboards/{type}` 한 경로에 `{type}` 값만 추가하고, Dashboard Builder의 `dashboard_widgets.data_source`/`widget_type`(§3.43)을 기존 트랜잭션 테이블에 대해 조합한 결과를 반환한다. **D-076에서 `tenant-usage` `type` 값 추가**(아래 표 참조, 신규 테이블 없음).

| `{type}` 값 | 위젯 그룹 | 포함 지표(요약) | 데이터 소스(기존 테이블) |
|---|---|---|---|
| `today-summary` | 오늘 지표 | 오늘 매출/주문/결제실패/배송준비/출고/배송완료/취소/환불/반품/교환/문의/가입/패키지구매/자동결제실패/정기배송예정/**SEO 오류**(D-073)/**Feed 오류**(D-073) | `orders`/`order_items`/`order_payment_attempts`/`shipments`/`returns`/`exchange_requests`/`tickets`/`members`/`recurring-orders`(§2.4). SEO 오류는 `product_seo`(§3.55) 필수 필드 미입력/NULL 건수를 조회 시점에 파생 카운트(§2.22 `dashboards/seo`와 동일한 파생 계산 방식, 별도 오류 컬럼 없음). Feed 오류는 `scheduled_job_run_logs`(§3.40)를 Feed 종류 Job(§2.24 `product-feeds`)으로 필터링한 `status=FAILED` 건수 — 둘 다 신규 컬럼 없는 파생 카운트 |
| `urgent-tasks` | 긴급 처리 지표 | 결제실패/자동결제실패/송장미등록/배송지연/환불대기/반품검수대기/CS미응답/재고부족/품절임박/정산보류/시스템장애 | `order_payment_attempts`/`shipments`(§3.57 `status`)/`return_items`/`tickets`/`inventory_items`/`settlement_batches`/시스템 모니터링(§5.54, O-196 연계) |
| `shop-operation` | 쇼핑몰 운영 지표 | 상품별/카테고리별/검색어별 매출, 장바구니 전환율, 주문/결제 전환율, 재구매율 | `order_items`/`products`/`search_query_logs`/`carts`·`cart_items`(§3.57, 신규)/`orders` |
| `abandoned-cart` | Abandoned Cart 지표(D-073) | 미결제 장바구니 수/알림발송건수/쿠폰전환율/재방문율 | 미결제 장바구니 수 = `carts`/`cart_items`(§3.57)를 `cart_items.added_at` 기준 미결제 상태로 일정 시간 경과한 건 카운트(파생). 알림발송건수 = `notification_logs`(§2.11)를 `event_type=ABANDONED_CART_REMINDER`로 필터(아래 비고 참조). 쿠폰전환율 = `coupon_issuances`(§3.35)에서 해당 트리거로 발급된 쿠폰의 `status=USED` 비율(파생). 재방문율 = `member_activity_logs`(§3.22, `activity_type=LOGIN`)로 알림 발송 후 재방문 여부 파생 집계. 4개 지표 모두 기존 테이블 조회 결과의 파생값이며 신규 컬럼 없음 |
| `tenant-usage` | Tenant Usage Dashboard(D-076) | 회원수/주문수/매출, Storage/API 호출, 메일/SMS/Push 발송량, Queue 사용량, License/Plan/사용률 | [PRD.md](PRD.md) §5.76(D-076) — 신규 테이블 없음. 회원수/주문수/매출은 기존 트랜잭션 테이블 집계, Storage/API 호출은 `external_api_call_logs`(§3.38, §2.25), 메일/SMS/Push는 `notification_logs`(§3.20, §2.11), Queue 사용량은 `scheduled_job_run_logs`(§3.40)를 데이터 소스로 사용. License/Plan/사용률은 이미 O-170(Multi-Tenant 활성화 시 테넌트별 사용량 모니터링)/O-146(API 호출 쿼터)/§5.55(D-071, License 관리)가 다루는 영역으로 본 위젯은 그 결과를 노출만 할 뿐 신규 Open Decision을 만들지 않음 |

- 위 5개 그룹 모두 **신규 테이블·신규 컬럼 없음** — 기존 트랜잭션 테이블 집계이며, Dashboard Builder의 위젯 추가만으로 구현([DATABASE.md](DATABASE.md) §3.57 재사용 매핑).
- 응답 스키마(위젯별 정확한 필드명/단위)는 OpenAPI 구체화 단계에서 확정 — 본 절은 어떤 위젯 그룹이 존재하는지와 데이터 소스만 명시한다.

**Abandoned Cart 감지/알림([PRD.md](PRD.md) §5.62, D-073)**:
- 장바구니 방치 감지 자체는 **고객향 엔드포인트가 아니다** — Scheduler Center(§3.40)가 주기적으로 트리거하는 내부 Job이 `cart_items.added_at`/`carts.updated_at`(§3.57)을 조회해 방치 장바구니를 판정한다. `api`는 이 내부 Job에 대한 신규 엔드포인트 형태를 두지 않는다(§1 원칙 — scheduler/worker는 외부에 API를 노출하지 않음). 수동 트리거가 필요하면 기존 `/v1/notifications/jobs`(§2.11) 패턴을 그대로 사용한다.
- 알림 발송은 `notification_templates.event_type`에 `ABANDONED_CART_REMINDER` 값을 추가하는 것으로 처리한다 — D-072 §5.56에서 채택한 "신규 알림은 모두 기존 `event_type` 카탈로그 항목 추가" 패턴과 동일하며, `notification_templates`/`notifications`/`notification_logs`(§3.20/§2.11) 구조 자체는 변경하지 않는다.
- 중복 발송 방지는 신규 컬럼 없이 발송 전 `notification_logs`(§2.11)를 `member_id`+`event_type=ABANDONED_CART_REMINDER`+기간 조건으로 조회해 이미 발송 이력이 있는지 확인하는 조회 로직으로 처리한다.
- 자동 쿠폰 발급은 기존 `/v1/coupons/{id}/issue`(§2.21)를 그대로 호출 — 발급 트리거 주체(Scheduler Center vs Shop 모듈)는 신규 Open Decision을 만들지 않고 기존 **O-192**(첫구매/생일쿠폰 자동발급 트리거 시점·책임 모듈)와 동일한 미확정 사항으로 묶어 처리한다.

### 2.23 SEO

> [DATABASE.md](DATABASE.md) §3.55(D-069)/[PRD.md](PRD.md) §5.46·§5.48(D-070) 기반 — 본 절의 모든 엔드포인트는 **설계 단계 개념 정의**이며 실제 OpenAPI 구체화·구현은 하지 않는다. SEO 자동생성 필드라도 관리자 수동 입력값이 항상 우선한다(BR-053).

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/products/{id}/seo` | GET/PATCH | 관리자(상품관리)/공개(메타태그 렌더링용 GET) | `seo_title`/`seo_description`/`seo_keywords`/`slug`/`canonical_url`/`meta_robots`/`og_*`/`twitter_*`/`schema_brand_override`/`schema_enabled`/`sitemap_priority`/`sitemap_change_frequency` | `product_seo` 행(1:1) | 미입력 필드는 BR-053 우선순위(상품명→Title, 요약설명→Description, 대표이미지→OG Image, 브랜드→Schema Brand, 가격→Offer, 재고→Availability)로 자동매핑된 값을 응답에 포함하되 원본 컬럼 값(수동입력 여부)도 함께 노출 — 자동매핑 vs 수동입력 구분 표시 형식은 미확정 |
| `/v1/content-seo-metadata` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `related_entity_type`(`CMS_PAGE`/`MARKETING_PROGRAM`/`FAQ_CATEGORY` 등)/`related_entity_id`/`seo_title`/`seo_description`/`og_image_ref`/`slug`/`canonical_url`/`meta_robots`/`is_indexable` | `content_seo_metadata` 목록/상세 | 상품 외 페이지(쇼핑몰 메인/카테고리/브랜드관/기획전/이벤트/공지사항/FAQ/회사소개/회원가입/로그인 등) SEO — File Manager의 `related_entity_type`/`related_entity_id` 패턴 재사용. 다국어 SEO는 `cms_translations` 확장으로 흡수 여부 미확정(O-193) |
| `/v1/tenant-share-images` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `share_type`(`KAKAO_DEFAULT`/`MAIN_URL_DEFAULT`/`PRODUCT_DEFAULT`/`EVENT_DEFAULT`/`PROMOTION_DEFAULT`/`BRAND_DEFAULT` 등 자유확장)/`image_ref`/`label` | `tenant_share_images` 목록/상세 | 이미지 권장 규격(OG 1200×630 등)은 DB 제약이 아닌 UI 안내 문구로만 처리(BR-054) — 본 엔드포인트는 규격 검증을 하지 않으며, 업로드 자체의 확장자/용량 검증은 기존 `system_security_policies.file_upload_policy`(§3.46) 경로를 그대로 따른다 |
| `/v1/tenant-settings/seo` | GET/PATCH | 관리자(SuperAdmin/CountryAdmin) | `site_title`/`site_description`/`site_keywords`/`default_og_image_ref`/`default_twitter_image_ref`/`apple_touch_icon_ref`/`canonical_base_url`/`default_robots_txt` | `tenant_settings` SEO 관련 컬럼 | 상품/콘텐츠 SEO 미입력 시 최종 fallback 값 |
| `/v1/sitemap.xml` | GET | 공개 | - | sitemap XML | 별도 원장 없이 `product_seo`/`content_seo_metadata`의 `is_active`/`slug`/`sitemap_priority`/`sitemap_change_frequency`를 조합해 **요청 시점에 동적 생성**(BR-054, `banners` 노출기간 쿼리타임 필터링과 동일 패턴) — 캐싱 여부는 미확정 |
| `/v1/robots.txt` | GET | 공개 | - | robots.txt 텍스트 | `tenant_settings.default_robots_txt` + 상품/콘텐츠 `meta_robots` 조합 동적 생성(BR-054) — 캐싱 여부는 미확정 |
| `/v1/seo/sitemap-regenerate` | POST | 관리자(상품관리/CMS) | - | `202 Accepted` + `job_id` | sitemap.xml/robots.txt는 쿼리타임 파생이라 원칙적으로 "재생성"이 불필요하나, CDN/캐시 계층 도입 시 캐시 무효화 트리거 용도로 개념상 자리만 선반영 — 무거운 연산이면 §1.7 Job 패턴, 경량이면 즉시 처리(캐시 계층 도입 여부 자체가 미확정, O-148 연계) |
| `/v1/products/seo/bulk-update` | POST | 관리자(상품관리) | `product_ids[]`/변경 필드 | `202 Accepted` + `job_id`(Bulk Action) | SEO 일괄수정/OG이미지 일괄업로드/alt text 일괄수정([DATABASE.md](DATABASE.md) §3.56) — Bulk Action 패턴(§3.51) 재사용, 신규 계산 로직 없음. 동기/비동기 전환 임계값은 미확정(O-118) |

### 2.24 Feed

> [PRD.md](PRD.md) §5.50(D-070) 기반 — Bulk Action(§3.51)/File Manager(§3.39, category=FEED)/Scheduler Center(§3.40, `scheduled_job_definitions`) 기존 패턴을 재사용한다. Feed 포맷별 신규 테이블은 없다(동일 데이터의 출력 포맷만 다름).

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/product-feeds/jobs` | POST | 관리자(상품관리/마케팅) | `feed_type`(`GOOGLE_SHOPPING`/`FACEBOOK`/`NAVER_SHOPPING`)/`country_code` | `202 Accepted` + `job_id` | Feed 생성 — `products`/`product_seo`/`product_option_combinations` 조회 결과를 파생 생성(저장하지 않음), 생성 자체는 무거운 연산으로 간주해 Job 패턴 적용 |
| `/v1/product-feeds/jobs/{jobId}` | GET | 관리자 | - | Job 상태/결과(`result_ref`로 생성된 `files` 행 참조) | §1.7 공통 스키마 |
| `/v1/product-feeds` | GET | 관리자(상품관리/마케팅) | `feed_type`/기간 | 생성된 Feed 목록(다운로드 링크 포함) | File Manager(`files`, category=FEED) 재사용 — 생성된 파일을 등록·목록화한 것을 조회만, 별도 Feed 전용 테이블 없음 |
| `/v1/product-feeds/{id}/download` | GET | 관리자 | - | 파일 스트림 또는 signed URL | `files.file_ref`(Supabase Storage) 참조 |
| `/v1/product-feeds/auto-refresh-config` | GET/PATCH | 관리자(상품관리/마케팅) | `feed_type`/`is_enabled`/`cron_expression` | `scheduled_job_definitions` 행("Feed 갱신" Job 종류) | Scheduler Center 재사용 — 관리자가 on/off, 주기는 기존 cron 관리 범위([DATABASE.md](DATABASE.md) §3.40). Google Merchant Center/Facebook Catalog 연동은 §2.25의 `external-api-connections` 등록을 전제로 한다 |

### 2.25 Digital Marketing 연동

> [PRD.md](PRD.md) §5.47(D-070) 기반 — **운영 설계만, 실제 외부 API 호출/스크립트 삽입은 본 라운드에서 구현하지 않는다.** 기존 API Center `external_api_connections`(§3.38, §2.x 표에 전용 섹션 없이 [DATABASE.md](DATABASE.md)에만 정의되어 있었음)를 그대로 재사용 — `category`가 이미 자유 입력 필드라 신규 엔드포인트 형태를 만들지 않는다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/external-api-connections` | 표준 CRUD 패턴 적용 | SuperAdmin/관리자(API Center) | `api_name`/`category`/`auth_key_ref`/`endpoint_url`/`is_enabled` | `external_api_connections` 목록/상세 | 본 라운드에서 `category`에 추가되는 값: `GA4`/`GTM`/`GSC`/`GOOGLE_MERCHANT_CENTER`/`NAVER_SEARCH_ADVISOR`/`NAVER_ANALYTICS`/`META_PIXEL`/`TIKTOK_PIXEL`/`GOOGLE_ADS_CONVERSION`/`FACEBOOK_CATALOG`/`RSS_FEED`/`INDEXNOW`/`SITEMAP_PING` — 신규 컬럼/엔드포인트 형태 추가 없음, 기존 자유 카테고리 값 사용 |
| `/v1/external-api-connections/{id}/status` | GET | SuperAdmin/관리자(API Center) | - | `status`(`TESTING`/`ACTIVE`/`INACTIVE`) | 신규 상태값 없음 — [STATE-MACHINE.md](STATE-MACHINE.md) §12 기존 enum 그대로 재사용 |
| `/v1/external-api-call-logs` | GET | SuperAdmin/관리자(API Center) | `connection_id`/기간 | `external_api_call_logs` 목록(append-only) | 호출 로그 조회 — GSC/Naver Search Advisor 등 §2.22 SEO Dashboard 지표의 데이터원 추적용 |

### 2.26 Cart

> [PRD.md](PRD.md) §5.56(D-072), [DATABASE.md](DATABASE.md) §3.57 기반 — `carts`/`cart_items`는 MVP 시점부터 식별되어 있던 갭(장바구니 영속화 테이블 부재)을 해소하는 **불가피한 신규 테이블**이다([DATABASE.md](DATABASE.md) §3.57). 그 외 상태 표시(품절/가격변경/배송비/쿠폰적용가능)는 신규 컬럼 없이 조회 시점 파생값이다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/cart` | GET | 본인(회원) | - | `cart_id`/`items[]`(상품/옵션/수량/단가/품절여부/가격변경여부/배송비예상/쿠폰적용가능여부)/`subtotal` | `carts`/`cart_items`(신규, [DATABASE.md](DATABASE.md) §3.57) — 현재 장바구니 조회. 비회원 장바구니는 클라이언트 저장이 기본 권고이나 **미확정**(O-197과 동일 계열의 "비회원 처리" 패턴, [DATABASE.md](DATABASE.md) §3.57 O-099/O-188 참조). **Saved Cart([PRD.md](PRD.md) §5.63, D-073)는 별도 기능이 아니다** — `carts`/`cart_items`가 D-072부터 이미 영속 테이블이므로 본 엔드포인트가 곧 "저장된 장바구니" 조회이며, 신규 경로 없이 메뉴 명칭만 "저장된 장바구니"로 노출할 수 있다(순수 UX/메뉴-네이밍 정리) |
| `/v1/cart/items` | POST | 본인(회원) | `product_id`/`product_option_combination_id`/`quantity` | `cart_item_id` | 담기 — `cart_items`(신규) 행 생성, `carts.updated_at` 갱신 |
| `/v1/cart/items` | PATCH | 본인(회원) | `cart_item_id`/`quantity` 또는 `product_option_combination_id`(옵션변경) | 갱신된 `cart_items` 행 | 수량·옵션 변경 — 옵션변경은 기존 행의 `product_option_combination_id`를 교체(별도 이력 테이블 없음) |
| `/v1/cart/items` | DELETE | 본인(회원) | `cart_item_ids[]` | - | 선택삭제 — 다중 삭제 지원 |
| `/v1/cart/items/{id}/move-to-wishlist` | POST | 본인(회원) | - | `wishlist_id` | `cart_items` 행을 삭제하고 `wishlists`(§2.21)에 동일 `product_id`로 등록 — "나중에 구매하기"를 `cart_items` 상태 플래그로 둘지 본 방식(위시리스트 이동)으로 처리할지는 **미확정**(O-197, [DATABASE.md](DATABASE.md) §3.57) — 본 엔드포인트는 후자(위시리스트 이동)를 잠정 채택 |

> **파생 지표(저장 컬럼 아님)**: 장바구니 아이템의 품절/가격변경/배송비예상/쿠폰적용가능 표시는 모두 조회(`GET /v1/cart`) 시점에 `product_option_combinations`(재고·`is_active`·`price_delta`, §3.52)/`shipping_fee_policies`(§3.48)/`coupons`(§2.21, §3.35) 테이블과 조인해 파생 계산한다 — `cart_items`에 해당 값을 저장하는 컬럼은 없다([DATABASE.md](DATABASE.md) §3.57).

### 2.27 AdminTaskQueue

> [PRD.md](PRD.md) §5.58(D-072), [DATABASE.md](DATABASE.md) §3.57 기반. **신규 Task Queue 테이블 없음** — Dashboard Builder(§3.43)의 "Task Queue" 위젯 종류로, `workflow_instances`/`order_payment_attempts`/`exchange_requests`/`return_items`/`product_reviews`/재고(`inventory_items`)/CS 티켓(`tickets`) 등 기존 테이블의 상태값을 횡단 조회하는 **파생 뷰**다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/admin/task-queue` | GET | 관리자(국가스코프) | `task_type`(필터, 다중 지정 가능)/`country_code` | 항목별 federated 목록(`task_type`/`count`/`detail_ref`) | **파생/집계 조회(읽음 전용) — 신규 Task Queue 테이블이 아니다.** 아래 `task_type` 표의 13종을 각각 기존 테이블/상태값 조회로 federated하여 단일 응답으로 묶어 반환([DATABASE.md](DATABASE.md) §3.57) |

**`task_type` 값과 데이터 소스**

| `task_type` | 의미 | 데이터 소스(기존 테이블/상태) |
|---|---|---|
| `NEW_ORDER_CONFIRM` | 신규 주문확인 | `orders`(§2.4, 신규 상태) |
| `PAYMENT_FAILED_CONFIRM` | 결제실패확인 | `order_payment_attempts`(§2.21) |
| `BANK_TRANSFER_CONFIRM` | 무통장입금확인 | `bank_transfer_payments`(§2.21, 도입 자체 미확정 O-181) |
| `INVOICE_REGISTER_NEEDED` | 송장등록필요 | `shipments`(§3.57 `status=PREPARING`) |
| `SHIPMENT_DELAY_CONFIRM` | 배송지연확인 | `shipments`(§3.57 `status=DELAYED`, [STATE-MACHINE.md](STATE-MACHINE.md) §17) |
| `RETURN_APPROVAL_PENDING` | 반품승인대기 | `return_items`(§2.9) |
| `EXCHANGE_APPROVAL_PENDING` | 교환승인대기 | `exchange_requests`(§2.21, §3.53) |
| `REFUND_APPROVAL_PENDING` | 환불승인대기 | `returns`/`return_items`(§2.9) 환불 대기 상태 |
| `REVIEW_REPORT_PENDING` | 리뷰신고처리 | `product_reviews`(§2.21) 신고 상태 |
| `PRODUCT_APPROVAL_PENDING` | 상품승인대기 | `workflow_instances`(`PRODUCT_APPROVAL` 워크플로우) |
| `LOW_STOCK_HANDLING` | 재고부족처리 | `inventory_items`(§2.9, 파생 현재고) |
| `RECURRING_PAYMENT_FAILED` | 자동결제실패처리 | `order_payment_attempts`(정기배송 자동결제 재시도, O-086) |
| `CS_TICKET_UNANSWERED` | CS문의미답변 | `tickets`(§2.14) 미답변 상태 |

- 응답의 `detail_ref`는 각 항목의 상세 목록 조회 경로(예: `PAYMENT_FAILED_CONFIRM` → `/v1/payment-methods/{id}/payment-attempts`)를 가리키며, AdminTaskQueue 자체는 건수/요약만 federated 응답으로 제공하고 상세 처리는 각 기존 엔드포인트(§2.4/§2.9/§2.14/§2.21)에서 수행한다.
- 정렬/우선순위 기준, 실시간 쿼리 vs 캐시 조회 여부는 §2.22 `/v1/analytics/dashboards/{type}`와 동일하게 미확정이다.

### 2.28 Board Engine

> [DATABASE.md](DATABASE.md) §3.58(D-074) 기반 — 코드 배포 없이 관리자가 새 게시판 종류(보도자료/갤러리/자료실/이벤트/홍보영상/교육자료/제품자료/인증자료/사용후기/회사소개/CSR/채용/IR 등)를 만들 수 있는 **범용 게시판/게시물 엔진**이다. **본 라운드는 기존 CMS(§2.18, `cms_pages`/FAQ/popups/banners)를 변경하지 않는다** — Board Engine은 그와 별개인 신규 병렬 구조이며, 두 구조의 통합/마이그레이션 여부는 미확정(O-200)이다. 신규 테이블은 `boards`/`board_categories`/`board_posts`/`board_post_comments`/`board_post_likes` 5종뿐이며, 첨부파일/이미지/동영상은 File Manager(`files`, §3.39)의 `related_entity_type='board_posts'` 재사용, SEO/OG는 `content_seo_metadata`(§3.55, §2.23) 재사용, 다국어는 `cms_translations`(§3.33) 재사용, 조회수는 `content_view_events`(§3.35) 재사용, 승인후게시는 Workflow Engine(§3.37) 재사용, 예약게시는 Scheduler Center(§3.40) 재사용이다 — 아래 표에서 이 영역은 신규 엔드포인트 형태가 아니라 기존 엔드포인트에 대한 **비고 cross-reference**로만 표기한다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/boards` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `name`/`code`/`description`/`board_type`/`layout_type`(`LIST`/`CARD`/`GALLERY`/`FAQ`/`VIDEO`)/`menu_exposure`/`shop_exposure`/`shop_member_exposure`/`my_office_exposure`/`main_exposure`/`menu_group`/`sort_order`/`country_codes`/`language_codes`/`feature_flags` | 게시판 정의 목록/상세 | `boards`(§3.58, D-074) — `board_type`은 `marketing_programs.category`(§2.19)와 동일하게 자유 확장 분류값, 코드 배포 없이 신규 게시판 종류 추가 가능. `feature_flags`(JSON)는 댓글/답글/파일첨부/이미지/대표이미지/동영상/다운로드/조회수/좋아요/공유/SEO/OG/예약게시/승인후게시/카테고리/태그/검색/RSS 토글 — 각 토글의 정확한 키 이름/기본값은 OpenAPI 구체화 단계에서 확정 |
| `/v1/boards/{id}/categories` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `name`/`sort_order` | 카테고리 목록 | `board_categories`(§3.58) — `feature_flags.카테고리=true`인 게시판에만 의미 있음(`false`면 카테고리 없이 운영, 엔드포인트 자체는 막지 않되 UI 노출만 조건부) |
| `/v1/boards/{boardId}/posts` | GET | 공개(목록, `is_public=true`/`status=PUBLISHED`만)/관리자(전체 상태 조회) | `category_id`/`status`/`tags`/`q` | 게시물 목록 | `board_posts`(§3.58) — 공개 조회는 `status=PUBLISHED`+`is_public=true`+`published_at<=now()`만 노출(예약게시 미도달 건 제외) |
| `/v1/boards/{boardId}/posts` | POST | 관리자(CMS) 또는 본인(게시판 정책에 따라 회원 작성 허용 시) | `category_id`/`title`/`content`/`thumbnail_image_ref`/`slug`/`tags[]`/`metadata`/`is_public`/`scheduled_publish_at` | `post_id`/`status` | `scheduled_publish_at` 지정 시 `status=SCHEDULED`로 생성되고 실제 게시 전환은 Scheduler Center(§3.40, `products.publish_start_at`와 동일 패턴) 재사용. 대상 게시판의 `feature_flags.승인후게시=true`이면 `status=PENDING_APPROVAL`로 생성되며 Workflow Engine(§3.37) 인스턴스가 `subject_type='BOARD_POST_APPROVAL'`(기존 `PRODUCT_APPROVAL`과 동일 패턴, §2.27 `PRODUCT_APPROVAL_PENDING` 참조)로 생성되어 승인 완료 후에만 `PUBLISHED`로 전환 — 승인후게시이면서 예약게시도 함께 지정된 경우의 우선순위/순서는 미확정(구현 단계 결정) |
| `/v1/posts/{id}` | GET/PATCH/DELETE | 공개(GET, 공개 게시물만)/관리자 또는 작성자 본인(PATCH/DELETE) | `title`/`content`/`thumbnail_image_ref`/`tags[]`/`metadata`/`is_public`/`status` | 게시물 상세 | `board_posts` 표준 CRUD(DELETE는 소프트삭제, §1 표기 규칙) — `metadata`(JSON)는 게시판 유형별 확장 필드(예: VIDEO 게시판의 `video_url`, FAQ형 게시판의 `answer`)를 담는 용도이며 컬럼 추가 없이 흡수 |
| `/v1/posts/{id}/comments` | GET/POST | 공개(GET, 공개 게시물)/본인(회원, POST) | `content`/`parent_comment_id`(선택) | 댓글 목록(트리 구조)/`comment_id` | `board_post_comments`(§3.58) — 대상 게시판의 `feature_flags.댓글=true`일 때만 노출/허용. 답글은 별도 엔드포인트 없이 본 엔드포인트에 `parent_comment_id`를 채워 POST(self-referencing, §3.58) — `parent_comment_id=null`이면 최상위 댓글 |
| `/v1/posts/{id}/likes` | POST/DELETE | 본인(회원) | - | `liked`(boolean)/`like_count_cache` | `board_post_likes`(§3.58, `post_id`+`member_id` 복합 unique) — POST는 토글(이미 좋아요 상태면 취소, 아니면 등록) 또는 POST=등록/DELETE=취소로 분리할지는 미확정(구현 단계 결정), 대상 게시판의 `feature_flags.좋아요=true`일 때만 노출/허용. `board_posts.like_count_cache`는 파생 캐시(원장은 `board_post_likes`) |
| `/v1/boards/{id}/rss.xml` | GET | 공개 | - | RSS XML | 별도 저장 없이 `board_posts`에서 `status=PUBLISHED`+`is_public=true`인 게시물을 **요청 시점에 동적 생성**(§2.23 `sitemap.xml`/BR-054와 동일 패턴) — 대상 게시판의 `feature_flags.RSS=true`일 때만 노출, 캐싱 여부는 미확정 |
| (cross-ref) 첨부/이미지/대표이미지/동영상/다운로드 파일 | - | - | - | - | File Manager `files`(§3.39) 기존 업로드/조회 엔드포인트를 `related_entity_type='board_posts'`/`related_entity_id={post_id}`로 그대로 사용 — Board Engine 전용 파일 엔드포인트를 신설하지 않음. 대상 게시판의 `feature_flags.파일첨부`/`이미지`/`대표이미지`/`동영상`/`다운로드` 토글에 따라 클라이언트가 업로드 UI 노출만 분기 |
| (cross-ref) SEO/OG | - | - | - | - | `/v1/content-seo-metadata`(§2.23) 기존 엔드포인트를 `related_entity_type='board_posts'`로 그대로 사용 — Board Engine 전용 SEO 엔드포인트를 신설하지 않음. 대상 게시판의 `feature_flags.SEO`/`OG=true`일 때만 입력 UI 노출 |
| (cross-ref) 다국어(게시판/게시물) | - | - | - | - | `cms_translations`(§3.33) 패턴 재사용 — `content_type='boards'` 또는 `'board_posts'`로 적용. §2.18 CMS에 이미 노출된 `/v1/cms/translations` 표준 CRUD 엔드포인트와 동일한 `content_type`/`content_id`/`locale`/`field_name`/`translated_value` 형태를 그대로 따르되, 본 라운드에서 Board Engine용 신규 엔드포인트는 추가하지 않는다 |
| (cross-ref) 조회수 | - | - | - | - | `content_view_events`(§3.35)에 `content_type='board_posts'`로 조회 이벤트 적재(기존 패턴 재사용) — `board_posts.view_count_cache`는 이를 집계한 파생 캐시일 뿐, 별도 조회수 기록 엔드포인트를 신설하지 않음 |
| (cross-ref) 공유/다운로드 통계 | - | - | - | - | `content_click_events`(§3.35) `click_type` 확장으로 흡수 — 이미 §2.22 `dashboards/seo`에서 동일하게 미확정으로 등록된 기존 Open Decision(O-194, SOCIAL_SHARE 후보값 채택 여부)에 묶이며, 본 라운드에서 별도 Open Decision을 추가하지 않는다 |

- 게시판별로 어떤 `feature_flags` 키가 어떤 엔드포인트의 노출/허용을 좌우하는지는 위 표의 비고에 1:1로 정리했다 — 실제 키 이름/기본값/하위호환(신규 키 추가 시 기존 게시판 기본값) 정책은 OpenAPI 구체화 단계에서 확정한다.
- 기존 CMS(`cms_pages`/`faq_categories`/`faq_items`/`popups`/`banners`, §2.18)의 콘텐츠를 Board Engine으로 이전할지(예: 공지사항을 `boards`로 전환)는 본 라운드의 범위가 아니며 O-200으로 미확정 — §2.18은 본 라운드에서 변경하지 않는다.

### 2.29 Compliance Transmission(공제조합 연동)

> [PRD.md](PRD.md) §5.68(D-075), [DATABASE.md](DATABASE.md) §3.59 기반 — **전부 Tenant별 선택 기능**이다. 연동 등록(직접판매공제조합/한국특수판매공제조합) 자체는 신규 엔드포인트를 두지 않고 기존 `/v1/external-api-connections`(§2.25)를 그대로 재사용한다 — `category=COMPLIANCE_REPORTING`/`api_name`(자유 입력, "직접판매공제조합"/"한국특수판매공제조합")만 본 라운드에서 사용하는 값이며 신규 컬럼/엔드포인트 형태를 추가하지 않는다. 보고서 단위 흐름(`compliance_report_definitions`/`compliance_report_submissions`, §2.8)도 변경하지 않으며, 본 절은 그 아래 **항목 단위** 전송·추적 레이어만 보강한다. `compliance_report_definitions`에 신설된 `tenant_id`/`auto_transmit`/`manual_transmit_allowed` 컬럼은 §2.8 `/v1/compliance/report-submissions` 응답에 자연 포함되는 필드일 뿐 별도 엔드포인트를 두지 않는다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/compliance/member-registrations` | 표준 CRUD 패턴 적용 | 본인(신청)/관리자(Compliance) | `connection_id`/`registration_number`(공제번호)/`certificate_file_ref`(공제증서) | `compliance_member_registrations` 목록/상세 | `certificate_file_ref`는 File Manager(`files`, §3.39) 업로드 결과 참조를 그대로 받는다 — Compliance 전용 업로드 엔드포인트를 신설하지 않음(§2.28 Board Engine의 File Manager cross-ref와 동일 패턴). `status`(등록대기/등록완료/등록실패/해지)는 표준 CRUD의 PATCH로 전이, 실제 공제조합 측 등록 확인은 §2.29 `transmission-items`의 `item_type=MEMBER` 전송 결과에 연동되나 자동 동기화 여부는 미확정(O-201~O-205 범위 밖이라 본 라운드는 수동 PATCH로만 설계) |
| `/v1/compliance/transmission-items` | GET | 관리자(Compliance, 국가스코프) | `status`(대기/전송중/성공/실패)/`item_type`(MEMBER/SPONSOR_RELATION/REVENUE/COMMISSION/REFUND/RETURN/CANCEL, 자유확장)/`connection_id`/`period` | `compliance_transmission_items` 목록(전송대상조회) | `source_type`+`source_id`(polymorphic)로 원본 회원/매출/수당/환불/반품/취소 레코드를 참조 — 원본 데이터 자체는 조회하지 않고 전송 상태 레이어만 조회. `call_log_id`로 기존 `external_api_call_logs`(§2.25) 호출 1건과 1:1 연결 |
| `/v1/compliance/transmission-items/{id}/resend` | POST | 관리자(Compliance) | - | `202 Accepted` + `job_id`(또는 `call_log_id`, Job 패턴 적용 여부는 단건이라 경량 처리 가능 — §1.7 기준 외부 API 호출이라 Job으로 통일) | 재전송 — `compliance_transmission_items.retry_count` 증가 + 신규 `external_api_call_logs` 행 생성, `status=전송중`으로 전이. `failure_reason`/`last_attempted_at`은 worker가 호출 결과로 갱신 |
| `/v1/compliance/transmission-items/bulk-send` | POST | 관리자(Compliance) | `item_ids[]` 또는 `status`/`item_type`/`period` 필터 | `202 Accepted` + `job_id` | 수동 전송(대량) — 잠재적으로 무거운 연산이라 §1.7 Job 패턴 적용(Bulk Action, §3.51과 동일 계열). worker가 대상 항목을 순회하며 각각 `external_api_call_logs`를 적재하고 `compliance_transmission_items.status`를 갱신 |

### 2.30 E-Wallet

> [PRD.md](PRD.md) §5.69(D-075), [DATABASE.md](DATABASE.md) §3.60 기반 — **Tenant별 선택 기능**이며, **잔액은 항상 `wallet_transactions`(append-only Ledger)에서 파생하고 직접 UPDATE하지 않는다**(D-075 결정). `member_wallets`의 `available_balance_cache`/`pending_balance_cache`/`withdrawable_balance_cache`/`used_balance_cache`/`hold_balance_cache`는 모두 파생 캐시이며 본 절의 어떤 응답 필드도 직접 PATCH로 설정 가능한 값이 아니다 — 잔액을 바꾸는 모든 엔드포인트는 내부적으로 `wallet_transactions`에 행을 추가한 결과로만 캐시가 갱신된다. 출금 승인/반려는 Workflow Engine(`workflow_instances`, `subject_type='WALLET_WITHDRAWAL'`, §3.37) 기존 패턴을 재사용 — 신규 승인 구조를 만들지 않는다. **후원수당 1차 산정·35% 법적 한도 검증·세금 계산(§2.5 Compensation/Settlement)은 본 라운드에서 전혀 변경하지 않는다** — 지갑 EARN 적립은 정산이 이미 확정한 `commission_records`/`settlement_items`를 읽기 전용으로 인용하는 추가 지급 채널일 뿐이다. 정산↔지갑 적립 분배 정책(전액/회원선택/병행)은 **미확정**(O-201).
>
> **D-078 확장(§3.63) — `wallet_type` 컬럼 추가**: `member_wallets`/`wallet_transactions`에 `wallet_type`(`CASH`/`LIFESTYLE_POINT`) 컬럼이 추가되어, 아래 모든 목록·조회 엔드포인트의 응답 항목에 `wallet_type`이 포함된다(기존 `CASH` 지갑은 응답상 `wallet_type=CASH`로 표시 — 의미 변경 없음). `transaction_type` 허용값에 `RESTORE`/`EXPIRE`가 추가된다. **출금 관련 엔드포인트(`/v1/wallets/{id}/withdrawal-requests`, `/v1/withdrawal-requests/{id}/approve`·`/reject`)는 `wallet_type=LIFESTYLE_POINT` 지갑을 대상에서 제외/거부한다** — 신청 시 대상 지갑이 `LIFESTYLE_POINT`이면 422, 목록 조회 시에도 `CASH` 지갑만 노출된다. Lifestyle Point 잔액/거래내역 조회는 본 절의 엔드포인트를 그대로 재사용하지 않고 §2.39(`/v1/lifestyle-wallet/balance`·`/transactions`)가 `wallet_type=LIFESTYLE_POINT` 필터를 고정해 별도 진입점으로 제공한다(권한·응답 모양은 동일 패턴). 상세는 §2.39 참조.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/members/{id}/wallets` | GET | 본인/관리자(국가스코프) | `currency_code`(필터)/`wallet_type`(필터, 신규: CASH/LIFESTYLE_POINT, D-078) | `member_wallets` 목록(통화별) — `status`/5종 잔액 캐시 필드/`wallet_type`(신규) 포함 | 회원당 통화별로 복수 지갑이 존재할 수 있음(`currency_code` 자유확장: KRW/USD/THB/JPY). 잔액 필드는 모두 파생 캐시 — 직접 설정 불가(아래 비고 일괄 참조). `wallet_type=LIFESTYLE_POINT` 지갑은 §2.39에서 전용 진입점(`/v1/lifestyle-wallet/balance`)으로도 조회 가능(D-078) |
| `/v1/wallets/{id}/transactions` | GET | 본인/관리자 | `transaction_type`(CHARGE/EARN/USE/CANCEL/REFUND/WITHDRAWAL_REQUEST/WITHDRAWAL_COMPLETED/ADJUSTMENT/HOLD/RELEASE/**RESTORE/EXPIRE**(신규, D-078))/기간 | `wallet_transactions` 목록(append-only 원장, cursor 페이지네이션 권장 — §1.3) — `wallet_type`(신규, D-078) 포함 | **조회만** — `source_type`+`source_id`(polymorphic)로 `commission_records`/`settlement_items`(EARN 시)/`reward_policies`(EARN, wallet_type=LIFESTYLE_POINT 시, D-078) 등 원본을 참조. 원장 행 자체의 생성/수정 엔드포인트는 본 절에 없음 — 모두 아래 액션형 엔드포인트(출금/보정/잠금 등)의 부수효과로만 worker/api가 기록 |
| `/v1/wallets/{id}/withdrawal-requests` | POST | 본인(회원) | `amount`/`bank_account_ref` | `wallet_withdrawal_requests` 행 — `status=REQUESTED` + `202 Accepted`(Workflow Engine 인스턴스 생성) | `workflow_instance_id`가 `subject_type='WALLET_WITHDRAWAL'` 인스턴스를 참조 — 신청 시점에 `wallet_transactions`에 `WITHDRAWAL_REQUEST`(또는 `HOLD`) 행이 함께 생성되어 가용 잔액에서 분리 표시. **대상 지갑이 `wallet_type=LIFESTYLE_POINT`이면 422 거부(D-078, §3.63) — Lifestyle Point는 출금(은행송금) 대상이 아님** |
| `/v1/wallets/{id}/withdrawal-requests` | GET | 본인/관리자 | `status`(REQUESTED/APPROVED/REJECTED/COMPLETED) | 출금신청 목록 | `wallet_type=CASH` 지갑만 대상 — `LIFESTYLE_POINT` 지갑은 본 목록에 출현하지 않음(D-078) |
| `/v1/withdrawal-requests/{id}/approve` | POST | 관리자(Compliance/정산담당) | - | `status=APPROVED` | Workflow Engine 인스턴스 진행 + `wallet_transactions`에 `WITHDRAWAL_COMPLETED` 행 생성(잔액 차감은 이 원장 행을 통해서만 반영) — DATABASE.md §3.60 규칙대로 잔액 직접 UPDATE 금지, 본 엔드포인트는 경량 상태전이+원장 기록을 트리거하는 것으로 표기(실제 지급 실행이 무거운 외부 송금 연동을 포함하는지는 미확정, O-204 PG사 선정과 별개로 후속 확인 필요) |
| `/v1/withdrawal-requests/{id}/reject` | POST | 관리자(Compliance/정산담당) | `reason` | `status=REJECTED` | Workflow Engine 인스턴스 종료 + `wallet_transactions`에 `RELEASE` 행 생성(신청 시 분리된 금액을 가용 잔액으로 복원) |
| `/v1/wallets/{id}/adjustments` | POST | 관리자(Compliance/정산담당) | `amount`(부호 포함)/`reason`(필수) | `wallet_transactions`에 `ADJUSTMENT` 행 생성 | 관리자 수동 보정 — `reason` 누락 시 422. 보정 사유는 `audit_logs`(§2.10)에도 별도 기록 대상(금전 영향 변경) |
| `/v1/wallets/{id}/lock` | POST | 관리자(Compliance) | `reason` | `status=LOCKED` | 잠금 중에는 USE/WITHDRAWAL_REQUEST 계열 신규 원장 생성을 거부(409) — 정확한 거부 대상 트랜잭션 타입 범위는 미확정(구현 단계 결정) |
| `/v1/wallets/{id}/unlock` | POST | 관리자(Compliance) | - | `status=ACTIVE` | - |

### 2.31 Global Payment(글로벌 결제)

> [PRD.md](PRD.md) §5.70(D-075), [DATABASE.md](DATABASE.md) §3.61 기반 — **Tenant별 선택 기능**. 한국 결제수단(신용카드/계좌이체/가상계좌/무통장입금)은 이미 D-069(§3.53, `order_payment_attempts`/`virtual_account_issuances`/`bank_transfer_payments`/`order_payment_splits`, §2.21에 엔드포인트 기존 등록)가 다루며 본 라운드는 이를 재문서화하지 않는다 — 존재만 인용한다. 정기/자동결제의 PG 토큰 재사용은 신규 컬럼 없이 기존 `payment_methods.pg_token`(§3.30, §2.4)이 PG-agnostic하게 이미 확장 가능한 구조라 본 라운드에서 스키마 변경이 없다(PG 토큰화 방식 자체는 기존 미확정 O-087 그대로). Idempotency는 기존 O-054(§1.7)를 재사용한다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/external-api-connections` | (기존 엔드포인트 확장, §2.25 인용 — 본 절에서 CRUD 재정의 없음) | SuperAdmin/관리자(API Center) | 기존 필드 + `country_code`(신규 컬럼, nullable) | `external_api_connections` 목록/상세 | `country_code`는 PG 연동을 국가로 스코프하기 위한 신규 컬럼(예: 태국 PromptPay=`TH`) — `category=PG`인 행에 한해 의미를 가지며, 그 외 카테고리(COMPLIANCE_REPORTING/GA4 등)는 nullable로 비워둠. 신규 엔드포인트를 추가하지 않고 §2.25 표의 기존 행에 필드만 추가되는 것으로 처리. 글로벌 PG사 구체 선정은 미확정(O-204) |
| `/v1/payment-webhooks/{connectionId}` | POST | **공개**(PG가 호출하는 인바운드 엔드포인트, 별도 인증 대신 서명 검증) | PG사별 raw payload(스키마는 PG마다 다름, `raw_payload_ref`로 원본 보존) | `202 Accepted` | 인바운드 Webhook 수신 — 기존 `external_api_call_logs`(아웃바운드, 우리가 PG로 호출)와 방향이 반대인 신규 `payment_webhook_events` 테이블에 적재. 서명 검증(`signature_verified`) 후 실제 처리는 Job 생성(202 Accepted) 패턴 — api는 수신·서명검증·Job 생성까지만 담당하고 무거운 처리(주문/지갑/정산 영향 반영)는 worker가 수행(§1.7). 서명 검증 방식 및 실패 시 처리(거부/격리 후 검토)는 미확정(O-205) — 검증 실패 시에도 `payment_webhook_events`에 `signature_verified=false`로 격리 기록할지 즉시 거부할지는 O-205 확정 후 결정 |
| `/v1/payment-webhook-events` | GET | SuperAdmin/관리자(API Center) | `connection_id`/`event_type`/`status`(수신/처리중/처리완료/처리실패)/기간 | `payment_webhook_events` 목록 | 관리자 수신 로그 조회 — `related_order_id`로 §2.4 주문과 연결, `raw_payload_ref`로 원본 payload 조회(File Manager 또는 Storage 참조, 정확한 보존 방식은 미확정) |
| `/v1/payment-webhook-events/{id}/reprocess` | POST | SuperAdmin/관리자(API Center) | - | `202 Accepted` + `job_id` | 재처리 — `처리실패` 건에 대해 동일 payload로 Job을 재생성(§1.7), `status=처리중`으로 전이. 재처리 시 멱등성 보장은 기존 O-054(Idempotency Key) 정책 범위에 포함되나 적용 대상에 Webhook 재처리가 명시적으로 포함되는지는 O-054 확정 시 함께 결정 |

### 2.32 Global Search

> [PRD.md](PRD.md) §5.71(D-076), [DATABASE.md](DATABASE.md) §3.62 기반 — **신규 Engine/신규 검색 인덱스 테이블 없음.** `members`/`products`/`orders`/`shipments`/`workflow_instances`/`board_posts`/`cms_pages`/`faq_items`/`marketing_programs`/`files`/`audit_logs`/`notifications`/`external_api_connections`/`scheduled_job_definitions` 등 기존 테이블에 대한 federated 쿼리이며, PostgreSQL Full-Text 등 전문검색 인덱스 적용 여부는 구현 단계 최적화 사항이다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/search` | GET | 인증된 관리자(모듈별 기존 조회 권한 적용) | `q`(검색어, 필수)/`modules[]`(검색 대상 모듈 필터, 선택) | 모듈(카테고리)별로 그룹화된 결과 목록 — 각 결과 항목 `module`/`title`/`snippet`/`deep_link`(해당 모듈 기존 상세 화면 경로) | 별도 검색 권한 모델 없음 — 호출자의 기존 모듈별 조회 권한([ROLE-MATRIX.md](ROLE-MATRIX.md))을 결과 필터링에 그대로 적용한다. 권한이 없는 모듈의 데이터는 검색 결과 자체에 노출되지 않는다(검색 자체를 차단하는 것이 아니라 결과 단계에서 federated 쿼리 대상이 호출자 권한 범위로 제한됨). Command Palette(§5.81, D-076)는 본 엔드포인트를 그대로 호출하는 프론트엔드 컴포넌트일 뿐 별도 엔드포인트가 없다. 자동완성/Highlight도 DB 영향 없는 순수 프론트엔드 렌더링이라 별도 엔드포인트가 없다 |
| `/v1/search/recent` | GET | 관리자(본인) | `limit` | 본인 최근 검색어 목록(검색 시각순) | 신규 테이블 없음 — 기존 `search_query_logs`(§3.35, 쇼핑몰 검색용)와 동일한 패턴을 관리자 검색에도 그대로 적용해 `member_id` 대신 호출 관리자 식별자로 적재·조회한다. 저장된 검색조건(자주 쓰는 필터 저장)은 본 엔드포인트가 아니라 §2.35 `saved_filters`를 재사용한다(별도 검색 전용 저장구조 없음) |

### 2.33 Approval Center

> [PRD.md](PRD.md) §5.72(D-076), [DATABASE.md](DATABASE.md) §3.62 기반 — **관리자 업무 Queue(§2.27, §5.60)의 ERP 전체 확장판이며 신규 승인 엔진/신규 승인 테이블이 없다.** 기존 `/v1/admin/task-queue`(§2.27)가 쇼핑몰 운영 항목(13종 `task_type`)에 한정했던 것을, 회원변경/조직이동/환불/반품/교환/E-Wallet 출금/Workflow/게시글 승인/프로그램 신청까지 federated 대상으로 일반화한다. 각 모듈의 기존 승인 권한·절차(정산 운영자 승인, 조직 이동 승인 등)는 변경하지 않는다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/admin/task-queue` | GET | 관리자(국가스코프) | `task_type`(필터, 다중 지정 가능, 아래 신규 값 포함)/`country_code` | 항목별 federated 목록(`task_type`/`count`/`detail_ref`) | **기존 엔드포인트 확장 — §2.27에서 CRUD 재정의 없음.** §2.27의 기존 13종 `task_type`에 더해 본 라운드에서 다음 `task_type` 값을 추가한다: `MEMBER_CHANGE_REQUEST_PENDING`(`member_change_requests`, §2.6) / `ORGANIZATION_TRANSFER_PENDING`(`organization_transfer_logs`, §2.7) / `RETURN_REFUND_PENDING`(`returns`, §2.9 — §2.27 `REFUND_APPROVAL_PENDING`과 동일 소스, ERP 전체 관점 별칭) / `EXCHANGE_PENDING`(`exchange_requests`, §2.21 — §2.27 `EXCHANGE_APPROVAL_PENDING`과 동일) / `WALLET_WITHDRAWAL_PENDING`(`wallet_withdrawal_requests`, §2.30) / `WORKFLOW_STEP_PENDING`(`workflow_instances`+`workflow_step_actions`, 범용 Workflow 단계) / `BOARD_POST_APPROVAL_PENDING`(`board_posts`, `status=PENDING_APPROVAL`, §2.28) / `PROGRAM_APPLICATION_PENDING`(`marketing_program_applications`, §2.19) — 신규 테이블 없음, 모두 기존 테이블의 federated 조회 |
| `/v1/approval-delegations` | 표준 CRUD 패턴 적용 | 관리자(본인 위임 설정)/SuperAdmin(전체 조회) | `delegator_id`/`delegate_id`/`start_date`/`end_date`/`reason` | `approval_delegations` 목록/상세 | `approval_delegations`([DATABASE.md](DATABASE.md) §3.62, 신규) — Workflow Engine 자체 테이블(`workflow_definitions`/`workflow_instances`/`workflow_step_actions`)은 변경하지 않으며, 본 위성 테이블은 Workflow Engine이 승인자를 결정할 때 참조만 한다. 위임 가능 범위(동일 역할 내 한정 여부 등)는 미확정 — **O-207** |
| `/v1/approval-sla-policies` | GET/PATCH | SuperAdmin/관리자(모듈별 설정 권한) | `module`(자유확장)/`sla_hours` | 모듈별 SLA 설정 목록 | 신규 테이블 없음 — System Settings(§3.46) 패턴의 관리자 설정값으로 저장(별도 SLA 전용 테이블을 두지 않음). SLA 초과 항목의 지연 알림은 Scheduler Center(§2.x, `scheduled_job_definitions`) Job이 주기 점검 후 §2.11 `notifications/jobs`로 발송하는 기존 패턴을 재사용. **정확한 기본 SLA 시간(24시간/48시간 등)은 미확정(구현 단계 결정)** — 이는 사소한 파라미터값이라 별도 Open Decision 번호를 신설하지 않는다 |

### 2.34 Approval History

> [PRD.md](PRD.md) §5.77(D-076) 기반 — Audit Center(§2.10)와는 별개다. Audit Center는 시스템 전체 변경의 포렌식 기록이고, 본 절은 **승인/반려 이력만** 모아 보여주는 운영 조회 화면이다. 신규 테이블 없음 — 기존 승인 테이블들의 `approved_by`/`approved_at`/`decision`/반려 사유 컬럼을 federated 조회한다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/approval-history` | GET | 관리자(국가스코프) | `module`(필터)/`approver`(승인자)/`from`/`to`(기간) | 승인/반려 이력 목록(`module`/`approver`/`decision`/`decided_at`/`reason`/`detail_ref`) | `member_change_requests`/`organization_transfer_logs`/`returns`/`exchange_requests`/`wallet_withdrawal_requests`(§2.30)/`workflow_instances`+`workflow_step_actions`(`decision` 컬럼)/`board_posts`(승인후게시 이력)/`marketing_program_applications`의 `approved_by`/`approved_at`/반려사유 컬럼을 federated 조회 — §2.33 Approval Center가 "대기 중" 항목을 보여주는 것과 짝을 이루는 "완료된 결정" 조회. 신규 컬럼 없음 |
| `/v1/approval-history/export` | POST | 관리자(국가스코프) | `format`(PDF/Excel)/§ 위 필터와 동일 | `202 Accepted` + `job_id` | Report Builder Job 패턴(§2.22 `analytics/reports/jobs`와 동일 계열) 재사용 — 무거운 집계/문서 생성은 worker가 수행, api는 Job 생성·상태조회만 담당(§1.7) |

### 2.35 Favorite Menu / Saved Filter

> [PRD.md](PRD.md) §5.73(D-076), [DATABASE.md](DATABASE.md) §3.62 기반 — **O-199(관리자 저장된 검색조건/즐겨찾기 메뉴 테이블 도입 여부)를 본 라운드에서 해소한다.** D-072 시점에 빈도·필요성이 낮다는 이유로 테이블 신설을 미뤘던 항목을 `admin_favorite_menus`/`saved_filters`(DATABASE.md §3.62, 신규) 2종으로 해소한다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/admin/favorite-menus` | 표준 CRUD 패턴 적용 | 관리자(본인) | `menu_key`(SITEMAP.md 메뉴 트리 노드 키)/`sort_order` | `admin_favorite_menus` 목록 | **O-199 해소.** `pinned_at`은 생성 시각으로 자동 기록. 최근 메뉴/자주 사용하는 메뉴/추천은 신규 테이블 없이 `member_activity_logs`(§3.22, 기존)의 메뉴 진입 빈도를 파생 집계한 것이라 본 엔드포인트와 별개다 |
| `/v1/admin/saved-filters` | 표준 CRUD 패턴 적용 | 관리자(본인 작성, `is_shared=true`인 경우 타 관리자 조회 가능) | `name`/`target_module`(자유 문자열)/`filter_criteria`(JSON)/`is_default`/`is_shared` | `saved_filters` 목록/상세 | **O-199 해소.** `is_default=true`인 필터는 대상 모듈(`target_module`) 진입 시 기본 적용 — 모듈당 `is_default` 복수 허용 여부 등 세부 규칙은 OpenAPI 구체화 단계에서 확정. `is_shared=true`이면 다른 관리자도 조회 가능(공유), 수정/삭제는 작성자(`admin_id`)만 가능 |

### 2.36 Notification Inbox

> [PRD.md](PRD.md) §5.75(D-076), [DATABASE.md](DATABASE.md) §3.62 기반 — §2.11 Notification의 확장. 기존 `notifications`/`notification_logs`(§3.20, §2.11)는 변경하지 않으며, `notification_inbox_states`(신규)가 읍음/중요/보관 등 **상태만 오버레이**한다(`cms_translations`가 `cms_pages`를 오버레이하는 것과 동일 패턴). 마이오피스 알림함(회원용, D-070)과는 별개이며 `recipient_type=ADMIN`으로 구분한다.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/notification-inbox` | GET | 관리자(본인) | `is_read`/`is_important`/`is_archived`/`tags`/`event_type`/기간/`q`(§2.32 검색 패턴 재사용) | `notifications`(§3.20)와 `notification_inbox_states`(신규)를 `recipient_type=ADMIN`+`recipient_id`로 join한 목록 | 관리자 본인의 알림함 — 회원 알림(`notifications.member_id`)과는 별도로 `notification_inbox_states.recipient_type=ADMIN`인 행만 대상. 첨부파일은 File Manager(§3.39) 기존 패턴 재사용 |
| `/v1/notification-inbox/{id}` | PATCH | 관리자(본인) | `is_read`/`is_important`/`is_archived`/`tags`(JSON 배열) | 갱신된 `notification_inbox_states` 행 | 읍음/안읍음/중요/보관/태그 변경 — 즐겨찾기는 별도 컬럼 없이 `tags` 또는 `is_important`로 표현(§5.75). 삭제는 `is_archived=true`로 처리(소프트 삭제, §1 표기 규칙과 동일 원칙) |
| `/v1/notification-inbox/bulk-read` | POST | 관리자(본인) | `notification_ids[]`(선택, 미지정 시 전체 대상) | `202 Accepted` + `job_id` 또는 처리 건수 | 전체 읍음 — Bulk Action 패턴(§2.21 `coupons/{id}/issue` 비고, §3.51) 재사용. 대상 건수가 적으면 경량 동기 처리, 많으면 Job 생성 — 동기/비동기 전환 기준은 §2.23 `products/seo/bulk-update`와 동일하게 미확정(O-118 범위) |

### 2.37 Operator Notes

> [PRD.md](PRD.md) §5.79(D-076), [DATABASE.md](DATABASE.md) §3.62 기반 — `admin_notes`(신규, 범용 운영자 메모)는 File Manager(§3.39)의 `related_entity_type`/`related_entity_id` 패턴을 재사용한다. **기존 `/v1/orders/{id}/admin-notes`(§2.21 Shop, `order_admin_notes` 기반, 주문 전용)는 변경하지 않는다** — 본 절은 그와 별개인 범용 메모 엔드포인트이며, 두 구조의 통합/마이그레이션 여부는 본 라운드에서 결정하지 않는다(**O-206**).

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/admin-notes` | 표준 CRUD 패턴 적용 | 관리자 | `related_entity_type`(MEMBER/ORDER/PRODUCT/BOARD_POST/WORKFLOW_INSTANCE/SETTLEMENT_BATCH 등 자유확장)/`related_entity_id`/`content`/`is_important`/`is_internal` | `admin_notes` 목록/상세 | `admin_notes`(DATABASE.md §3.62, 신규) — File Manager의 polymorphic 참조 패턴 재사용. `is_internal`은 항상 true 전제(고객 노출 화면에 절대 노출 안 함)이나 컬럼으로 명시. 주문 메모는 기존 `/v1/orders/{id}/admin-notes`(§2.21, `order_admin_notes`)를 그대로 사용하며 본 엔드포인트로 중복 작성하지 않는다 — 두 구조 통합 여부는 **O-206** |

### 2.38 Recent Activity (관리자)

> [PRD.md](PRD.md) §5.74(D-076) 기반 — Customer Timeline(§2.2 `/v1/members/{id}/timeline`, D-073)과 동일한 federated 조회 원리를 **관리자 본인**에게 적용한 버전이다. 신규 테이블 없음.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/admin/recent-activity` | GET | 관리자(본인) | `type`(필터, 다중 지정 가능)/`from`/`to` | 시간순 통합 타임라인 목록 — 각 항목 `type`(조회/수정/승인/Workflow/파일/로그인)/`occurred_at`/`summary`/`detail_ref` | **읽기 전용 federated 조회, 신규 테이블 없음** — §2.2 `/v1/members/{id}/timeline`과 동일한 응답 형태를 관리자 본인 행위로 스코프해 반환한다. `audit_logs`(§2.10, 조회/수정)/`member_activity_logs`(§3.22, 관리자 행위 기록·`activity_type=LOGIN` 로그인 포함)/`workflow_step_actions`(§3.37, `decision` 컬럼 — 최근 승인)/`workflow_instances`(§3.37, 최근 Workflow)/`files`(§3.39, `uploaded_by` — 최근 파일)를 `occurred_at` 기준 시간 역순으로 병합한 결과다. Personal Workspace(§5.78, D-076)는 본 엔드포인트를 My Dashboard(§2.22 `/v1/dashboards`)/Favorite Menu(§2.35)/Quick Action(§0)과 함께 한 화면에 모으는 순수 UI 집약이며 별도 엔드포인트가 없다 |

### 2.39 Marketing Reward (Reward Policy)

> [PRD.md](PRD.md) §5.83~§5.86(D-078), [DATABASE.md](DATABASE.md) §3.63 기반 — **새 MLM 정책이 아니다.** 기존 "+알파" 보너스(Travel/Car/자기계발)는 현금성 수당이 아니라 Marketing Reward Program 지급분이라는 점을 명확히 하기 위한 구조 정리이며, `Reward Policy → Lifestyle Point Ledger → Lifestyle Wallet → 사용` 체인을 표준화한다. **현금(Settlement/E-Wallet `wallet_type=CASH`)과 Lifestyle Point는 절대 혼합하지 않는다.** Reward Policy는 Marketing Program Engine(§2.19, 기존 `marketing_programs`) 산하 설정이며 신규 Engine이 아니다 — 신규 테이블은 `reward_policies` 1종뿐이고, Lifestyle Wallet 자체는 §2.30 E-Wallet 구조(`member_wallets`/`wallet_transactions`)를 `wallet_type=LIFESTYLE_POINT`로 재사용한다(중복 문서화 방지를 위해 잔액/거래 조회는 §2.30과 동일 패턴만 명시하고 필드 표는 반복하지 않음). Lifestyle Point의 적립 산정 로직 자체(Travel/Car/자기계발의 적립률·누적기간, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.2)는 변경하지 않으며, 본 절이 바꾸는 것은 적립 결과의 저장 위치(라우팅)뿐이다. 쇼핑몰 결제 사용 여부는 Tenant별 ON/OFF 토글이며 기본값은 **미확정(O-208)**. 기존 `point_transactions.source_type=LIFESTYLE_BONUS` 과거 적립분의 마이그레이션 여부도 **미확정(O-209)**.

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/reward-policies` | 표준 CRUD 패턴 적용 | 관리자(마케팅/SuperAdmin) | `program_id`/`reward_type`(현재 LIFESTYLE_POINT 단일값)/`accrual_basis`/`accrual_method`/`accrual_rate`/`accumulation_period`/`accumulation_duration`/`min_condition`/`max_limit`/`start_date`/`end_date`/`usable_period`/`target_wallet_type`/`is_active` | `reward_policies` 목록/상세 | `reward_policies`(§3.63, 신규) — Program명은 `program_id`로 `marketing_programs`(§2.19)를 참조해 가져오며 중복 입력하지 않는다. Travel/Car/자기계발 3종은 본 엔드포인트로 조회는 가능하나 `accrual_rate`/`accumulation_duration` 수치는 `plan_definition.lifestyle_bonus.*`(D-032)를 읽기 전용으로 참조 표시 — PATCH로 수치를 직접 바꿀 수 없음(Golf 등 신규 Program만 관리자가 직접 입력). **D-079**: 응답의 `accrual_basis`/`accrual_method`/`accrual_rate`는 `active_formula_version_id`(§3.64)가 가리키는 `reward_formula_versions` 행의 값을 그대로 복제한 파생 캐시다 — 신뢰 가능한 원본은 항상 `reward_formula_versions`(아래 `/formula-versions`)이며, 본 CRUD 엔드포인트의 경로/메서드는 변경되지 않았다 |
| `/v1/reward-policies/{id}/activate` | POST | 관리자(마케팅/SuperAdmin) | - | `is_active=true` | 경량 상태 전이 — api에서 직접 처리(계산 아님) |
| `/v1/reward-policies/{id}/deactivate` | POST | 관리자(마케팅/SuperAdmin) | - | `is_active=false` | 비활성화 이후에는 본 정책 기준 신규 EARN 적립이 생성되지 않음(기존 적립분/잔액에는 영향 없음) |
| `/v1/lifestyle-wallet/balance` | GET | 본인/관리자(국가스코프) | - | `member_wallets` 중 `wallet_type=LIFESTYLE_POINT` 행 — §2.30 `/v1/members/{id}/wallets`와 동일한 잔액 캐시 필드 구성 | §2.30 E-Wallet 잔액조회 패턴을 `wallet_type=LIFESTYLE_POINT` 필터로 고정한 전용 진입점 — 잔액은 §2.30과 동일하게 `wallet_transactions` 원장에서만 파생, 직접 UPDATE 없음. 신규 응답 스키마 없음(§2.30 재사용) |
| `/v1/lifestyle-wallet/transactions` | GET | 본인/관리자 | `transaction_type`(EARN/USE/CANCEL/RESTORE/EXPIRE/ADJUSTMENT)/기간 | `wallet_transactions` 중 `wallet_type=LIFESTYLE_POINT` 목록(append-only, cursor 페이지네이션 권장 — §1.3) | §2.30 `/v1/wallets/{id}/transactions` 패턴을 `wallet_type=LIFESTYLE_POINT` 필터로 고정한 전용 진입점. `source_type=REWARD_POLICY`(EARN, source_id=`reward_policies.id`)/`source_type=PROGRAM_APPLICATION`(USE, source_id=`marketing_program_applications.id`)로 원본 인용 |
| `/v1/reward-policies/{id}/accrual-report` | POST | 관리자(마케팅/SuperAdmin) | `period`/`format`(PDF/Excel, 선택) | `202 Accepted` + `job_id` | Program별 적립 현황(§5.86) — `reward_policies`+`wallet_transactions` 조인 집계, Report Builder Job 패턴(§2.22 `analytics/reports/jobs`, §2.34 `/v1/approval-history/export`와 동일 계열) 재사용. 무거운 집계는 worker가 수행, api는 Job 생성·상태조회만 담당(§1.7) |
| `/v1/reward-policies/{id}/formula-versions` | GET | 관리자(마케팅/SuperAdmin) | `version_number`(정렬, 기본 내림차순) | `reward_formula_versions`(§3.64) 목록 — Formula 변경 이력(§5.90 Reward History) | 정책별 전체 버전을 `version_number` 순으로 조회 — append-only 원장이므로 과거 버전도 그대로 보존·조회된다. 신규 이력 테이블 없음(`reward_formula_versions` 자체가 이력) |
| `/v1/reward-policies/{id}/formula-versions` | POST | 관리자(마케팅/SuperAdmin) | `formula_type`(REVENUE_RATE/PV_RATE/BV_RATE/FIXED_POINT/CUSTOM 등 자유 확장값)/`formula_definition`(JSON)/`accrual_rate`/`effective_from`(선택) | `reward_formula_versions` 신규 행(`version_number` 자동 증가) | **append-only — 항상 새 버전 행을 추가하며 기존 버전은 절대 PUT/PATCH/DELETE하지 않는다.** 생성 직후 자동으로 활성화되지는 않으며, 활성화는 아래 `/activate`를 별도 호출해야 한다(명시적 2단계). `formula_type=CUSTOM`(관리자 정의 계산식)의 표현 방식·안전한 평가 메커니즘은 미확정 — **O-210** |
| `/v1/reward-policies/{id}/formula-versions/{versionId}/activate` | POST | 관리자(마케팅/SuperAdmin) | - | 대상 버전 `is_active=true`(기존 활성 버전은 자동으로 `is_active=false`로 전환), `reward_policies.active_formula_version_id` 갱신 | 경량 상태 전이 — api에서 직접 처리(계산 아님). 정책당 활성 버전은 항상 정확히 1개이며, 활성화 전환은 기존 버전 행을 수정하지 않고 `is_active` 플래그만 전환(버전 행 자체는 append-only 원칙 유지) |
| `/v1/reward-policies/{id}/simulate` | POST | 관리자(마케팅/SuperAdmin) | 가상 입력값(예: `revenue`/`pv`/`bv` 등 — 대상 정책의 활성 Formula Version `formula_type`에 따라 다름) | 계산된 예상 Point 미리보기 값 | **Reward Simulation**(§5.88) — 이미 저장된 정책의 활성 `reward_formula_versions`를 읽어 계산만 수행. **응답은 어떤 테이블에도 저장되지 않음, audit_logs에 실행 메타데이터만 기록**(요청자/시각/입력값). Simulation과 Formula Test(아래)는 동일 계산 로직(worker 또는 경량 api)을 재사용하며 신규 계산 엔진을 만들지 않는다 |
| `/v1/reward-policies/formula-test` | POST | 관리자(마케팅/SuperAdmin) | 아직 저장하지 않은 Formula 초안(`formula_type`/`formula_definition`/`accrual_rate`) + 테스트 입력값(`revenue`/`pv`/`bv` 등) | 계산된 예상 Point 미리보기 값 | **Formula Test**(§5.89) — `reward_formula_versions`에 아직 커밋되지 않은 초안을 대상으로 테스트. **응답은 어떤 테이블에도 저장되지 않음, audit_logs에 실행 메타데이터만 기록**(요청자/시각/입력값). Simulation은 "이미 등록된 정책"을, 본 엔드포인트는 "아직 저장하지 않은 Formula 초안"을 대상으로 한다는 점만 다르다 — `reward_policies`/`reward_formula_versions`/`wallet_transactions` 등 실제 데이터는 전혀 변경하지 않음 |

> §2.39는 [DECISIONS.md](DECISIONS.md) D-078/**D-079** 참조. 신규 Business Rule 없음, 신규 MLM 정책 없음. §2.5 Compensation/Settlement는 본 절에서 전혀 변경하지 않으며, Lifestyle Point는 어떤 엔드포인트에서도 현금 Settlement와 합산·교환되지 않는다. D-078 신규 Open Decision은 **O-208(쇼핑몰 사용 기본값)/O-209(과거 적립분 마이그레이션 여부)** 2건, D-079 신규 Open Decision은 **O-210(Custom Formula 표현·안전한 평가 방식)** 1건이며 모두 DATABASE.md §3.63~§3.64/PRD.md §5.83~§5.90 설계 시 이미 등록됨 — 본 절에서 신규로 추가하지 않음. Formula Version 관련 엔드포인트는 모두 **append-only 원칙**을 따른다 — 기존 버전 행에 대한 PUT/PATCH/DELETE는 존재하지 않으며, 모든 변경은 새 버전 행 추가(`POST /formula-versions`) + 활성 플래그 전환(`POST /formula-versions/{versionId}/activate`)으로만 이루어진다.

## 3. 상세 예시 3종

아래 3개 모듈은 비동기 Job 패턴, Validation, Error 케이스, Pagination/Sorting/Filter를 포함한 완전한 명세를 제공한다.

### 3.1 Member — 회원 CRUD

#### 3.1.1 `GET /v1/members` — 목록 조회

**Query Parameters**

| 파라미터 | 타입 | 설명 |
|---|---|---|
| `page` / `limit` | integer | 페이지네이션(§1.3, offset 방식 권장안) — 기본 `page=1&limit=20` |
| `status` | string | `PENDING_VERIFICATION`/`ACTIVE`/`DORMANT`/`WITHDRAWN`/`FORCED_WITHDRAWN`(§3.12). 콤마로 다중 지정 가능 |
| `member_type` | string | `INDIVIDUAL`/`BUSINESS`/`CORPORATION`/`FOREIGN` |
| `country_code` | string | 국가 필터 — 관리자의 `country_scope` 밖 값 지정 시 403 |
| `sponsor_id` | uuid | 특정 스폰서의 직추천 회원만 조회 |
| `q` | string | 이름/이메일/회원ID 검색 |
| `sort` | string | 예: `joined_at:desc` (§1.4) |

**요청 권한**: CountryAdmin 이상(국가스코프 적용), Partner는 본인 정보만(`/v1/members/{id}`로 대체).

**Response 200**

```json
{
  "data": [
    {
      "id": "mem_01HXA...",
      "member_type": "INDIVIDUAL",
      "country_code": "KR",
      "status": "ACTIVE",
      "sponsor_id": "mem_01HX9...",
      "center_id": null,
      "joined_at": "2026-01-15T02:00:00Z"
    }
  ],
  "meta": { "page": 1, "limit": 20, "total": 4821, "total_pages": 242 }
}
```

#### 3.1.2 `POST /v1/members` — 가입

**Request**

```json
{
  "member_type": "INDIVIDUAL",
  "country_code": "KR",
  "sponsor_ref": "fns.com/r/mem_01HX9...",
  "email": "partner01@example.com",
  "name": "홍길동",
  "phone": "010-0000-0000"
}
```

**Validation 규칙**

| 필드 | 규칙 |
|---|---|
| `member_type` | 필수, enum(`INDIVIDUAL`/`BUSINESS`/`CORPORATION`/`FOREIGN`) |
| `country_code` | 필수, `countries`의 활성 국가만 허용(KR/TH/JP/US) — `CN`은 Reserved이므로 거부([DECISIONS.md](DECISIONS.md) D-023) |
| `sponsor_ref` | 선택 — 추천 링크 형식(`/r/{memberid}` 또는 `?ref={memberid}`) 파싱, 미지정 시 `sponsor_id=null`(최상위) |
| `email` | 필수, 형식 검증, 중복 가입 거부 |

**Response 202 Accepted**

> 가입 자체(레코드 생성, `status=PENDING_VERIFICATION`)는 무거운 계산이 아니므로 동기 처리하되, 본인확인 서류 검증(§3.1.3)은 별도 단계다. 가입 직후에는 과거 실적이 없어 수당/정산 영향 분석 대상이 아니다([ARCHITECTURE.md](ARCHITECTURE.md) §8).

```json
{
  "id": "mem_01HXB...",
  "status": "PENDING_VERIFICATION",
  "member_type": "INDIVIDUAL",
  "country_code": "KR",
  "sponsor_id": "mem_01HX9...",
  "joined_at": "2026-06-25T01:23:45Z"
}
```

**Error 케이스**

| 상황 | Status | `error.code` |
|---|---|---|
| 이메일 중복 | 409 | `CONFLICT` |
| `country_code=CN` 지정 | 422 | `VALIDATION_ERROR` (`details: [{"field":"country_code","reason":"Reserved Country는 가입 대상국이 아닙니다"}]`) |
| `sponsor_ref` 파싱 실패(존재하지 않는 추천인) | 422 | `VALIDATION_ERROR` |
| `member_type=BUSINESS`인데 필수 서류 누락 | 422 | `VALIDATION_ERROR` |

#### 3.1.3 `POST /v1/members/{id}/identity-profile` — 본인확인 서류 접수 → 심사

가입심사(§5.6.3)는 "요청→검증→승인/반려" 패턴([ARCHITECTURE.md](ARCHITECTURE.md) §8)을 따르되, 서류 검증 자체는 관리자의 수동 심사이므로 Job이 아니라 상태 전이로 처리한다.

**Request**

```json
{
  "member_type": "BUSINESS",
  "identity_fields": { "business_registration_number": "***-**-*****", "representative_name": "홍길동" },
  "document_refs": ["storage://identity-docs/mem_01HXB/biz-reg.pdf"]
}
```

**Response 201**

```json
{ "member_id": "mem_01HXB...", "verification_status": "심사대기" }
```

심사 승인: `POST /v1/members/{id}/identity-profile/review` → `{"decision":"APPROVE"}` → `members.status: PENDING_VERIFICATION → ACTIVE`.

#### 3.1.4 `GET /v1/members/{id}/my-organization` — 내 조직 조회 (페이지네이션/정렬/필터 시연)

**Query**: `period=2026-06&line=1,2,3` (LINE1~3만 조회), `sort=order_revenue:desc`, `page=1&limit=20`.

**Response 200**

```json
{
  "data": {
    "lines": [
      { "line": 1, "members": [{ "member_id": "mem_02...", "joined_at": "2026-02-01" }] }
    ],
    "organization_revenue": 125000000,
    "organization_commission": 4250000,
    "organization_growth": [{ "month": "2026-05", "new_members": 12 }]
  },
  "meta": { "page": 1, "limit": 20, "total_lines_members": 87 }
}
```

> 조회 전용([PRD.md](PRD.md) §5.1.1, D-020) — 본 엔드포인트에서 "추천인 변경" 등 쓰기 액션은 제공하지 않는다. 실시간 계산 vs 캐시 테이블 사용 여부는 미확정([DATABASE.md](DATABASE.md) §3.11).

### 3.2 Catalog — Product / 상품 이미지

#### 3.2.1 `POST /v1/products` — 상품 등록

**Request**

```json
{
  "name": "비즈니스 패키지",
  "price": 4000000,
  "is_active": true,
  "thumbnail_image_ref": "storage://products/biz-pkg/thumb.jpg",
  "video_url": "https://youtube.com/watch?v=...",
  "attachment_refs": ["storage://products/biz-pkg/brochure.pdf"]
}
```

**Validation 규칙**

| 필드 | 규칙 |
|---|---|
| `name` | 필수, 최대 길이 제약은 미확정 |
| `price` | 필수, 정수(원 단위, [DATABASE.md](DATABASE.md) §1) |
| `thumbnail_image_ref` | 선택, Supabase Storage 참조 형식 |
| `video_url` | 선택, URL 형식(외부 호스팅 — 자체 업로드는 O-115로 미확정) |
| `attachment_refs` | 선택, 배열, 각 항목 Storage 참조 형식 |

**Response 201**

```json
{ "id": "prod_01HXC...", "name": "비즈니스 패키지", "price": 4000000, "is_active": true }
```

#### 3.2.2 `POST /v1/products/{id}/images` — 상품 이미지 등록 ([PRD.md](PRD.md) §5.43, [DATABASE.md](DATABASE.md) §3.50, D-060)

**Request**

```json
{
  "image_type": "DETAIL",
  "device_type": "PC",
  "image_ref": "storage://products/biz-pkg/detail-01-pc.jpg",
  "alt_text": "비즈니스 패키지 상세 이미지 1",
  "sort_order": 1
}
```

**Validation 규칙**

| 필드 | 규칙 |
|---|---|
| `image_type` | 필수, enum(`LIST`/`DETAIL`/`INTRO`/`INGREDIENT`/`USAGE`/`CERTIFICATE`) |
| `device_type` | 필수, enum(`PC`/`MOBILE`/`COMMON`) |
| `image_ref` | 필수, Storage 참조 |
| `sort_order` | 선택(미지정 시 같은 `product_id`+`image_type` 범위 내 마지막 순서로 자동 배정) |
| 파일 용량/형식 | `system_security_policies.file_upload_policy`(§3.46) 정책에 위반 시 거부 — 본 엔드포인트가 아니라 업로드 단계(별도 Storage 업로드 API 또는 사전 서명 URL 발급)에서 1차 검증, 본 엔드포인트는 메타데이터 등록만 수행 |

**Response 201**

```json
{
  "id": "img_01HXD...",
  "product_id": "prod_01HXC...",
  "image_type": "DETAIL",
  "device_type": "PC",
  "image_ref": "storage://products/biz-pkg/detail-01-pc.jpg",
  "alt_text": "비즈니스 패키지 상세 이미지 1",
  "sort_order": 1,
  "is_active": true
}
```

**Error 케이스**

| 상황 | Status | `error.code` |
|---|---|---|
| `image_type` 미지정/잘못된 enum | 422 | `VALIDATION_ERROR` |
| 파일 용량 초과(전역 업로드 정책 위반) | 422 | `VALIDATION_ERROR` (`details: [{"field":"image_ref","reason":"파일 용량이 허용 기준을 초과합니다"}]`) |
| 존재하지 않는 `product_id` | 404 | `NOT_FOUND` |

#### 3.2.3 `GET /v1/products/{id}/images` — 목록/정렬/필터 시연

**Query**: `image_type=DETAIL&device_type=PC&sort=sort_order:asc&is_active=true`

**Response 200**

```json
{
  "data": [
    { "id": "img_01HXD...", "image_type": "DETAIL", "device_type": "PC", "sort_order": 1, "image_ref": "...", "is_active": true },
    { "id": "img_01HXE...", "image_type": "DETAIL", "device_type": "PC", "sort_order": 2, "image_ref": "...", "is_active": true }
  ],
  "meta": { "page": 1, "limit": 20, "total": 6, "total_pages": 1 }
}
```

#### 3.2.4 `DELETE /v1/products/{id}/images/{imageId}` — 소프트 삭제

`product_images.is_active = false`로 전환(물리 삭제 아님, [DATABASE.md](DATABASE.md) §4 원칙 3) — `audit_logs` 기록 대상.

**Response 200**: `{ "id": "img_01HXD...", "is_active": false }`

### 3.3 Order — 주문 생성 및 Compensation Job 트리거 (비동기 패턴 시연)

#### 3.3.1 `POST /v1/orders` — 주문 생성

**Request**

```json
{
  "items": [
    { "product_id": "prod_01HXC...", "quantity": 1, "unit_price": 4000000 }
  ],
  "payment_method": "CARD_TOKEN_xxxx",
  "shipping_address_ref": "addr_01HXF..."
}
```

**Validation 규칙**

| 필드 | 규칙 |
|---|---|
| `items` | 필수, 최소 1개, 각 `product_id`는 활성 상품만 허용(`is_active=true`) |
| `quantity` | 필수, 양의 정수 |
| `payment_method` | 필수 — PG 토큰화 방식은 미확정(O-087) |

**처리 흐름 (api 책임 범위 — 계산 위임 시연)**

1. `api`가 재고/상품 활성 여부 등 경량 검증을 수행하고, `orders`/`order_items`에 즉시 매출을 기록한다 — "주문 생성/취소 요청 접수, **즉시 반영 가능한 매출 기록**"은 Order 모듈의 책임([ARCHITECTURE.md](ARCHITECTURE.md) §2.2 Order)이며 여기까지는 동기 처리한다.
2. 매출 기록이 완료되면 `api`는 **후원수당(Compensation) 계산을 직접 수행하지 않고**, `redis`(BullMQ)에 `COMMISSION_CALCULATION` Job을 적재한다.
3. 주문 생성 응답에는 주문 결과(`201`에 준하는 본문)와 함께, 비동기로 트리거된 수당 계산 Job 정보를 **함께 반환**한다 — 최종 HTTP status는 `202 Accepted`로 응답해 "주문은 생성되었으나 그에 따른 수당 계산은 아직 완료되지 않았음"을 명확히 한다.

**Response 202 Accepted**

```json
{
  "order": {
    "id": "ord_01HXG...",
    "status": "PAID",
    "total_amount": 4000000,
    "created_at": "2026-06-25T01:30:00Z"
  },
  "triggered_jobs": [
    {
      "job_id": "job_7c3e1f2a",
      "job_type": "COMMISSION_CALCULATION",
      "status": "QUEUED",
      "status_url": "/v1/compensation/jobs/job_7c3e1f2a"
    }
  ]
}
```

4. 클라이언트(`web`)는 `triggered_jobs[0].status_url`을 폴링해 계산 완료 여부를 확인한다.

#### 3.3.2 `GET /v1/compensation/jobs/{jobId}` — 수당 계산 Job 상태 조회

**Response 200 (처리 중)**

```json
{
  "job_id": "job_7c3e1f2a",
  "job_type": "COMMISSION_CALCULATION",
  "status": "PROCESSING",
  "requested_at": "2026-06-25T01:30:00Z",
  "started_at": "2026-06-25T01:30:02Z",
  "completed_at": null,
  "result_ref": null,
  "error": null
}
```

**Response 200 (완료)**

```json
{
  "job_id": "job_7c3e1f2a",
  "job_type": "COMMISSION_CALCULATION",
  "status": "COMPLETED",
  "requested_at": "2026-06-25T01:30:00Z",
  "started_at": "2026-06-25T01:30:02Z",
  "completed_at": "2026-06-25T01:30:11Z",
  "result_ref": "/v1/commission-records?source_order_id=ord_01HXG...",
  "error": null
}
```

> `worker`가 [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3(라인 단위 도달 깊이 판정, D-018)에 따라 LINE1~5 후원수당, 패키지 상품인 경우 제품 판매수익·페어보너스([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1)까지 계산해 `commission_records`(append-only)에 직접 기록한다 — `api`는 이 계산식에 관여하지 않으며, 오직 Job 생성과 상태/결과 조회만 수행한다([ARCHITECTURE.md](ARCHITECTURE.md) §2.2/§2.3, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3).

**Response 200 (실패)**

```json
{
  "job_id": "job_7c3e1f2a",
  "job_type": "COMMISSION_CALCULATION",
  "status": "FAILED",
  "error": { "code": "PLAN_VERSION_NOT_FOUND", "message": "해당 국가에 활성화된 마케팅 플랜 버전을 찾을 수 없습니다." }
}
```

#### 3.3.3 `GET /v1/commission-records` — 계산 결과 조회 (Pagination/Sorting/Filter 시연)

**Query**: `source_order_id=ord_01HXG...&sort=created_at:desc&page=1&limit=20`

**Response 200**

```json
{
  "data": [
    {
      "id": "cr_01HXH...",
      "recipient_member_id": "mem_02...",
      "commission_type": "PRODUCT_SALES_PROFIT",
      "line_root_member_id": "mem_02...",
      "amount": 1000000,
      "plan_version_id": "plan_KR_2026H1",
      "period": "2026-06",
      "source_order_id": "ord_01HXG...",
      "created_at": "2026-06-25T01:30:09Z"
    }
  ],
  "meta": { "page": 1, "limit": 20, "total": 1, "total_pages": 1 }
}
```

> append-only 원장 — `recipient_member_id`는 본인 조회 시 자기 항목으로 제한, 관리자는 `country_scope`로 제한된다.

#### 3.3.4 `POST /v1/orders/{id}/cancel` — 주문 취소 (Error 케이스 시연)

**Request**: `{ "reason": "고객 요청 취소" }`

| 상황 | Status | `error.code` |
|---|---|---|
| 이미 배송 완료된 주문 취소 시도 | 409 | `CONFLICT` |
| 본인 소유가 아닌 주문 취소 시도 | 403 | `FORBIDDEN` |
| `reason` 누락 | 422 | `VALIDATION_ERROR` |
| 정상 취소 | 202 | - (`status=CANCELLED` + 매출 차감 Job 또는 즉시 음수 차감 엔트리 — 정확한 처리 방식은 [DATABASE.md](DATABASE.md) §3.3에서 확정 필요) |

## 4. 미확정 사항 — 구현 단계 확인 필요

- 페이지네이션 방식(offset vs cursor) 최종 채택 및 `limit` 상한 (§1.3)
- 범위 필터 표기법(`_from`/`_to` 접미사 vs `[gte]`/`[lte]`) (§1.5)
- Job 상태조회 엔드포인트를 전역 공통(`/v1/jobs/{id}`)으로 둘지 모듈별 하위 경로로 둘지
- 아웃바운드 Webhook 인프라 도입 여부 및 시점 (O-145, [GAP-ANALYSIS.md](GAP-ANALYSIS.md))
- API 호출 쿼터/Rate Limit 정책 (O-146)
- Multi-Tenant Job 격리 방식 (O-159)
- Redis 장애 시 DLQ(Dead Letter Queue) 재처리 절차 (O-161)
- 주문 취소/환불 시 매출 차감 처리 방식(별도 음수 엔트리 vs 다른 방식) ([DATABASE.md](DATABASE.md) §3.3)
- Bulk Action의 동기/비동기 전환 임계값 (O-118)
- API 버전 deprecation 공지 절차·sunset 기한 (O-172, §1.1)
- Job 생성 Idempotency Key 저장 방식 및 적용 범위 (O-054, §1.7)
- 검색엔진 지표(노출/클릭/CTR/순위)를 DB 캐시 vs 매 조회 시 외부 API 실시간 호출로 가져올지 — SEO Dashboard 조회 엔드포인트(O-195, §2.22)
- SEO/공유이미지/Feed 관련 그 외 미확정 항목은 §2.23~§2.25 각 행의 비고에 인용된 기존 O-176/O-177/O-180~O-194/O-118/O-148을 그대로 참조 — 본 라운드에서 신규 Open Decision은 O-195 외 추가하지 않았다
- 장바구니 "나중에 구매하기" 처리 방식(상태 플래그 vs 위시리스트 이동) — O-197, §2.26
- 가격 알림 트리거 기준(절대가 vs 할인율) — O-198, §2.21
- 장바구니/가격알림 비회원 처리(서버 저장 vs 클라이언트 저장) — O-099/O-188과 동일 계열, §2.21/§2.26
- 배송 지연 판정 기준(시간/단계) — [STATE-MACHINE.md](STATE-MACHINE.md) §17, §2.21 `/v1/shipments/{id}/tracking`
- 테스트발송 발송이력 구분 컬럼 여부, 관리자 업무 Queue 정렬/우선순위 기준, 운영자 대시보드 실시간 쿼리 vs 캐시 — 모두 본 라운드(D-072)에서 신규 O-번호를 추가하지 않고 "미확정(후속 확인 필요)"으로만 표기(§2.11/§2.22/§2.27)
- 기존 CMS 콘텐츠를 Board Engine으로 이전할지 여부 — O-200, §2.28
- Board Engine 승인후게시+예약게시 동시 지정 시 우선순위/처리 순서, 좋아요 토글 방식(단일 토글 엔드포인트 vs POST/DELETE 분리) — 본 라운드(D-074)에서 신규 O-번호를 추가하지 않고 "미확정(구현 단계 결정)"으로만 표기(§2.28)
- 정산↔E-Wallet 적립 분배 정책(전액/회원선택/병행) — O-201, §2.30
- 포인트↔E-Wallet 상호 전환 허용 여부 — O-202, §2.30
- 쇼핑몰 결제 시 포인트/E-Wallet 차감 우선순위 — O-203, §2.30/§2.21
- 글로벌 결제 PG사 구체 선정(국가별 실제 업체명) — O-204, §2.31
- 결제 Webhook 서명 검증 방식 및 검증 실패 시 처리(거부/격리) — O-205, §2.31
- 공제조합 회원등록(`compliance_member_registrations`)과 항목 전송(`compliance_transmission_items`) 간 자동 상태 동기화 여부, E-Wallet 잠금 시 거부 대상 트랜잭션 타입 범위, 출금 승인 시 실제 지급 실행의 동기/비동기(Job) 여부, Webhook 재처리 시 Idempotency Key 적용 포함 여부 — 모두 본 라운드(D-075)에서 신규 O-번호를 추가하지 않고 "미확정(구현 단계 결정)"으로만 표기(§2.29/§2.30/§2.31)
