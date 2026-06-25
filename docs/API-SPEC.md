# API-SPEC.md — REST API 명세

> 상태: v0.2 (D-064 — 개발착수 전 최종 안정화: 버전 deprecation/Idempotency Open Decision 연계 보강) · 최종 수정일: 2026-06-26 · 단계: 설계(Design)
> 전제 문서: [ARCHITECTURE.md](ARCHITECTURE.md), [DATABASE.md](DATABASE.md)
> 본 문서는 OpenAPI 산출물을 대체하지 않으며, 구현 단계에서 실제 OpenAPI yaml로 구체화하기 위한 설계 단계 명세다.

## 0. 범위와 원칙

- 본 문서가 다루는 대상은 [ARCHITECTURE.md](ARCHITECTURE.md) §2.2의 `api`(NestJS, 요청 처리 전용) 서비스가 노출하는 REST 엔드포인트다. `worker`/`scheduler`는 외부에 API를 노출하지 않는다 — `api`가 Job을 생성하고 상태/결과만 조회한다.
- **api는 계산하지 않는다(확정 — 위반 금지, [ARCHITECTURE.md](ARCHITECTURE.md) §1.1/§2.2/§3, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3).** 후원수당/정산/세금/프로모션 판정/보고서 생성 등 연산량이 큰 작업을 다루는 모든 엔드포인트는 본 문서에서 **"Job 생성 → 202 Accepted + Job id → 별도 상태조회 엔드포인트"** 패턴으로만 기술한다. 동기 계산 응답(200 + 계산결과)을 반환하는 것처럼 쓰지 않는다.
- 실제 값(필드 상세 타입, 정확한 enum, 정렬·필터 가능 필드 전체 목록 등)은 다수가 [DECISIONS.md](DECISIONS.md)에서 **미확정** 상태다. 본 문서는 ARCHITECTURE.md/DATABASE.md에 이미 정의된 모듈·테이블·흐름을 REST 형태로 옮기는 것이며, 존재하지 않는 필드/모듈을 새로 만들지 않는다.

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
| `/v1/notification-templates` | 표준 CRUD 패턴 적용 | 관리자(CMS) | `channel`/`locale`/`country_code`/`body` | 템플릿 목록 | - |
| `/v1/notification-logs` | GET | 관리자/본인(자기 수신내역) | `status`(성공/실패)/기간 | 발송/실패 이력 | 실패 시 재시도는 Redis Retry([ARCHITECTURE.md](ARCHITECTURE.md) §2.5) |

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
| `/v1/tickets` | 표준 CRUD 패턴 적용 | 본인(생성)/관리자(처리) | `ticket_type`(1:1문의/정산문의/수당문의/회원문의/명의변경문의/이의신청/불만접수) | 티켓 목록/상세 | - |
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
| `/v1/marketing-program-applications/{id}/approve` | POST | 관리자 | - | `status=APPROVED` | 승인 처리는 경량이라 **api에서 직접 수행**([ARCHITECTURE.md](ARCHITECTURE.md) §2.2 MarketingProgram) |
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
| `/v1/coupons` | 표준 CRUD 패턴 적용 | 관리자(마케팅) | `code`/`discount_type`/`discount_value`/유효기간 | 쿠폰 목록 | - |
| `/v1/coupons/{id}/issue` | POST | 관리자/시스템(프로그램 완료 트리거) | `member_ids[]` | 발급 결과 | `coupon_issuances` — 대량 발급은 Bulk Action 패턴(§5.44.6, 기존 단건 API 재사용) |
| `/v1/coupons/issuances/{id}/use` | POST | 본인(결제 시) | `order_id` | `status=USED` | - |

### 2.22 Analytics

| Resource Path | Method | Permission | 주요 요청 필드 | 주요 응답 필드 | 비고 |
|---|---|---|---|---|---|
| `/v1/analytics/dashboards/{type}` | GET | 관리자(국가스코프) | `period`/`country_code` | 집계 결과 | 실시간 쿼리 또는 스냅샷 캐시 조회, 방식 미확정([ARCHITECTURE.md](ARCHITECTURE.md) §2.2 Analytics) |
| `/v1/analytics/reports/jobs` | POST | 관리자 | `report_type`/`period` | `202 Accepted` + `job_id` | 집계 자체가 무거우면 worker로 위임 |
| `/v1/analytics/reports/jobs/{jobId}` | GET | 관리자 | - | Job 상태/결과 | - |

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
