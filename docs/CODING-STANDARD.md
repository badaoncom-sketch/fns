# CODING-STANDARD.md — 개발 표준

> 상태: 신규 v0.1 ([DECISIONS.md](DECISIONS.md) D-063 — 개발 착수 준비 문서 세트) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 전제 문서: [ARCHITECTURE.md](ARCHITECTURE.md), [DATABASE.md](DATABASE.md), [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)

본 문서는 [ARCHITECTURE.md](ARCHITECTURE.md)에서 이미 확정된 5개 서비스 구조(`web`/`api`/`worker`/`scheduler`/`redis`, §1~§2)와 [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3의 "계산과 요청처리 분리" 원칙을 **코드 구조 차원에서 강제**하기 위한 폴더 구조·네이밍·에러/로깅 컨벤션을 정의한다. 아직 코드가 존재하지 않는 설계 단계 문서이므로, 여기 제시된 폴더 트리/클래스 시그니처는 모두 **컨벤션 설명용 예시**이며 실제 FNS 기능을 구현하지 않는다.

## 목차

1. [폴더 구조](#1-폴더-구조)
2. [파일명 규칙](#2-파일명-규칙)
3. [NestJS 규칙 (api/worker/scheduler)](#3-nestjs-규칙-apiworkerscheduler)
4. [Next.js 규칙 (web)](#4-nextjs-규칙-web)
5. [Database 규칙](#5-database-규칙)
6. [Naming Rule](#6-naming-rule)
7. [Error Handling](#7-error-handling)
8. [Logging](#8-logging)

---

## 1. 폴더 구조

> **권장안, 미확정** — monorepo 채택 여부 자체가 [DECISIONS.md](DECISIONS.md)에 결정 기록이 없다. 아래는 [ARCHITECTURE.md](ARCHITECTURE.md) §1~§2의 5서비스 구조(물리적 배포 단위)를 그대로 반영한 monorepo 가정 하의 제안이며, 구현 착수 시점에 polyrepo(서비스별 별도 레포)로 재확정될 수 있다. 어느 쪽이든 **서비스 간 경계(§3.1.1 의존 규칙)는 동일하게 적용**되어야 한다.

### 1.1 최상위 구조 (monorepo 가정)

```
fns/
├── apps/
│   ├── web/          # Next.js — Admin Console / Partner Portal (§2.1)
│   ├── api/           # NestJS — 요청 처리, Job 생성 전용 (§2.2)
│   ├── worker/         # NestJS — Job 실제 처리 전용 (§2.3)
│   └── scheduler/      # NestJS — Job 트리거 전용 (§2.4)
├── packages/
│   ├── shared-types/    # 서비스 간 공유 DTO/타입 (계산 로직 없음)
│   ├── shared-constants/  # enum, 에러 코드 등 순수 상수
│   └── shared-config/    # 환경변수 스키마, lint/tsconfig base
└── docs/                # 본 설계 문서 세트
```

- `redis`/`Supabase`는 외부 관리형 서비스이므로 레포 내 별도 앱 디렉터리를 갖지 않는다([ARCHITECTURE.md](ARCHITECTURE.md) §2.5~§2.6).
- `packages/`는 **계산 로직을 두지 않는 순수 공유 자산**(타입/상수/설정)만 포함한다. 계산 로직을 `packages/`에 두면 `api`/`web`이 그것을 import해 간접적으로 계산을 수행하게 되는 우회 경로가 생기므로 금지한다(§3.1.1).

### 1.2 서비스 내부 구조 — 공통 패턴 (NestJS: api/worker/scheduler)

```
apps/api/
└── src/
    ├── modules/
    │   ├── auth/
    │   ├── member/
    │   ├── catalog/
    │   ├── order/
    │   ├── compensation/
    │   ├── organization-transfer/
    │   ├── compliance/
    │   └── ...           # ARCHITECTURE.md §2.2 모듈 목록과 1:1 대응 (§3.2)
    ├── common/
    │   ├── guards/        # RBAC, country_scope, role guard
    │   ├── filters/        # 공통 예외 필터 (§7)
    │   ├── interceptors/
    │   └── pipes/
    ├── config/
    └── main.ts
```

- 모듈 목록은 [ARCHITECTURE.md](ARCHITECTURE.md) §2.2 표와 **이름까지 동일하게** 유지한다 — 표에 없는 모듈을 api에 임의로 추가하지 않는다.

### 1.3 서비스 내부 구조 — worker

```
apps/worker/
└── src/
    ├── processors/        # BullMQ Job consumer (모듈별 1:1, §3.3)
    │   ├── compensation/
    │   ├── settlement/
    │   ├── tax/
    │   ├── report/
    │   └── ...           # ARCHITECTURE.md §2.3의 12종 Job과 대응
    ├── calculators/        # 실제 계산 로직 — api/web/scheduler에는 존재하지 않음 (§3.1.1)
    ├── common/
    └── main.ts
```

- `calculators/`는 **worker에만 존재**하는 디렉터리다. 다른 서비스에 동일 이름의 디렉터리가 생기면 그 자체가 DO-NOT-TOUCH.md §1.3 위반 신호다.

### 1.4 서비스 내부 구조 — scheduler

```
apps/scheduler/
└── src/
    ├── triggers/          # cron 등록만 수행, 계산 없음
    │   ├── settlement.trigger.ts
    │   ├── organization-transfer-apply.trigger.ts
    │   └── ...
    ├── common/
    └── main.ts
```

- scheduler에는 `calculators/`나 `processors/`(Job 실제 처리)에 해당하는 디렉터리가 존재해서는 안 된다 — `triggers/`만 둔다.

### 1.5 서비스 내부 구조 — web (Next.js)

§4에서 상세 기술.

---

## 2. 파일명 규칙

| 컨텍스트 | 규칙 | 예시 |
|---|---|---|
| 폴더명 | kebab-case | `organization-transfer/`, `member-lifecycle/` |
| NestJS 클래스 파일 | kebab-case + 역할 접미사 | `member.controller.ts`, `member.service.ts`, `member.module.ts` |
| NestJS DTO 파일 | kebab-case + `.dto.ts` | `create-member.dto.ts`, `update-bank-account.dto.ts` |
| NestJS Guard/Pipe/Filter | kebab-case + 역할 접미사 | `country-scope.guard.ts`, `http-exception.filter.ts` |
| BullMQ Processor (worker) | kebab-case + `.processor.ts` | `compensation.processor.ts` |
| 클래스/타입 선언 내부 | PascalCase | `class MemberService`, `interface CreateMemberDto` |
| Next.js 페이지 파일 | Next.js App Router 규약(`page.tsx`/`layout.tsx`) 그대로 | `app/admin/members/page.tsx` |
| React 컴포넌트 파일 | PascalCase | `ConfirmDialog.tsx`, `MemberStatusBadge.tsx` |
| React 컴포넌트 외 유틸/훅 파일 | kebab-case (훅은 `use-` 접두사) | `use-member-list.ts`, `format-currency.ts` |
| 상수/enum 전용 파일 | kebab-case + `.constants.ts` / `.enum.ts` | `change-type.enum.ts` |
| 테스트 파일 | 대상 파일명 + `.spec.ts`(unit) / `.e2e-spec.ts`(e2e) | `member.service.spec.ts` |

---

## 3. NestJS 규칙 (api/worker/scheduler)

### 3.1 모듈/컨트롤러/서비스/DTO 네이밍

| 요소 | 네이밍 패턴 | 예시 |
|---|---|---|
| Module | `{Domain}Module` | `MemberModule`, `OrganizationTransferModule` |
| Controller | `{Domain}Controller` | `MemberController` |
| Service | `{Domain}Service` | `MemberService` |
| Request DTO | `{Action}{Domain}Dto` | `CreateMemberDto`, `RequestOrganizationTransferDto` |
| Response DTO | `{Domain}ResponseDto` 또는 `{Domain}Dto` | `MemberResponseDto` |
| Job Producer 메서드 | `enqueue{JobName}` | `enqueueCompensationCalculation()` |
| Job Payload 타입 | `{JobName}JobPayload` | `CompensationCalculationJobPayload` |
| Processor 클래스 (worker) | `{JobName}Processor` | `CompensationCalculationProcessor` |
| Calculator 클래스 (worker) | `{Domain}Calculator` | `CompensationCalculator`, `SettlementCalculator` |
| Guard | `{Purpose}Guard` | `CountryScopeGuard`, `OrganizationTransferRoleGuard` |

### 3.1.1 모듈 경계 — ARCHITECTURE.md §2.2 와 1:1 대응

`api`의 `modules/` 하위 디렉터리명·모듈명은 [ARCHITECTURE.md](ARCHITECTURE.md) §2.2 표의 모듈명과 **정확히 1:1로 대응**시킨다(Auth/Member/Catalog/Order/Compensation·Settlement/MemberLifecycle/OrganizationTransfer/Compliance/Logistics/Audit/Notification/Center/DocumentCenter/CustomerService/ESignature/ActivityLog/RuleDesigner/CMS/MarketingProgram/Point/Shop/Analytics). 표에 없는 모듈이 코드에 생기면 ARCHITECTURE.md를 먼저 갱신해야 한다(임의 추가 금지, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3 마지막 항목).

### 3.2 api에 계산 로직을 두지 않는다 — 코드 구조 차원의 강제 규칙

[DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3은 "api의 request handler에 무거운 계산을 직접 구현하지 않는다"는 원칙을 선언했다. 본 절은 이를 **리뷰어의 주의가 아니라 디렉터리/클래스 구조로 강제**한다.

1. **`Calculator` 접미사를 가진 클래스는 `apps/worker/src/calculators/` 디렉터리에만 존재할 수 있다.** `apps/api`, `apps/web`, `apps/scheduler` 어디에도 `*.calculator.ts` 파일이 있으면 안 된다.
2. **api의 Compensation/Settlement/Compliance/RuleDesigner 등 모듈은 `{Domain}Service`만 가지며, 이 서비스의 책임은 Job 생성·Job 상태 조회·결과 조회로 한정한다.** 예시 시그니처:

   ```typescript
   // apps/api/src/modules/compensation/compensation.service.ts
   // api 쪽 — 계산하지 않고 Job만 만든다
   class CompensationService {
     enqueueCalculation(payload: CompensationCalculationJobPayload): Promise<{ jobId: string }> {
       // redis(BullMQ)에 적재만 한다 — 여기서 수당을 계산하지 않는다
     }

     getCalculationResult(jobId: string): Promise<CommissionRecordDto[]> {
       // Supabase에 worker가 이미 기록한 결과를 조회만 한다
     }
   }
   ```

   ```typescript
   // apps/worker/src/calculators/compensation.calculator.ts
   // worker 쪽 — 실제 계산은 여기만 존재한다
   class CompensationCalculator {
     calculate(snapshot: CompensationInputSnapshot): CommissionRecordDraft[] {
       // LINE1~LINE5 라인 단위 산정 로직 (COMPENSATION-RULES.md 확정 후 구현)
     }
   }
   ```

3. **의존 방향 강제**: `apps/api`는 `apps/worker/src/calculators/**`를 import할 수 없다(빌드 의존성 자체를 분리 — 같은 패키지에 두지 않는다, §1.1). lint 규칙(예: `eslint-plugin-boundaries` 또는 monorepo 도구의 모듈 경계 설정)으로 이 import를 차단하는 것을 권장한다 — 구체적 lint 도구 선정은 미확정.
4. **append-only 원장 쓰기 권한도 동일하게 제한**: `commission_records`/`settlement_items`에 쓰는 Repository/Service 클래스는 `apps/worker`에만 존재한다([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3). api의 해당 모듈은 조회용 Repository만 가진다.
5. **scheduler도 동일 원칙**: scheduler의 `triggers/`는 `enqueue{JobName}()` 호출만 하며, `Calculator` 클래스를 import하지 않는다.

### 3.3 worker의 Processor ↔ Calculator 분리

- `processors/`(BullMQ consumer, Job 수신/재시도/결과 기록 오케스트레이션)와 `calculators/`(순수 계산 함수/클래스)를 분리한다 — Processor는 Calculator를 호출하는 얇은 어댑터로 유지한다.
- 이렇게 분리하면 Calculator는 Job/Queue를 모르는 순수 함수로 단위테스트가 가능해지고, [ARCHITECTURE.md](ARCHITECTURE.md) §7의 영향 분석(dry-run 시뮬레이션)도 동일 Calculator를 재호출하는 것으로 구현할 수 있다(계산 로직 중복 금지).

---

## 4. Next.js 규칙 (web)

### 4.1 페이지/컴포넌트 구조 (권장안, 미확정)

```
apps/web/
└── src/
    ├── app/
    │   ├── admin/           # Admin Console 라우트
    │   │   └── members/page.tsx
    │   └── partner/          # Partner Portal 라우트 ('내 조직' 등)
    │       └── my-organization/page.tsx
    ├── components/
    │   ├── ui-standard/       # ERP UX Standard 공통 컴포넌트 — 화면별 재구현 금지 (§4.2)
    │   └── domain/            # 화면/도메인 전용 컴포넌트 (재사용 범위가 좁은 것)
    ├── lib/
    │   ├── api-client/        # api 서비스 호출 래퍼 (계산 없음, §4.3)
    │   └── auth/             # Supabase Auth 세션 검증 미들웨어 연동
    └── middleware.ts
```

### 4.2 ERP UX Standard 공통 컴포넌트 디렉터리 — 화면별 재구현 금지 강제

[PRD.md](PRD.md) §5.44, [DECISIONS.md](DECISIONS.md) D-061에서 정의한 공통 UX 컴포넌트(ConfirmDialog/WarningDialog/SuccessToast·ErrorToast·InfoToast/LoadingOverlay/LoadingButton/ActionButton/ConfirmActionButton/BulkActionDialog/UnsavedChangesGuard/ImageUploader/FileUploader/ImagePreview/ProgressBar)는 **`apps/web/src/components/ui-standard/` 디렉터리 하나에만** 존재한다([ARCHITECTURE.md](ARCHITECTURE.md) §2.1.1).

- 다른 디렉터리(`components/domain/**`, 각 `app/**` 라우트 폴더)에 `ConfirmDialog`, `*Toast`, `*Dialog` 등 동일 역할의 컴포넌트를 새로 만들지 않는다 — 화면마다 재구현하지 않고 `ui-standard/`에서 import해 재사용한다([PRD.md](PRD.md) §5.44.9/§5.44.12).
- `ui-standard/` 디렉터리 내부 컴포넌트는 **계산·검증 로직을 포함하지 않는다** — props로 받은 콜백을 호출하는 순수 프레젠테이션 컴포넌트로만 작성한다([ARCHITECTURE.md](ARCHITECTURE.md) §2.1.1, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.1 ERP UX Standard 원칙).
- 신규 화면 PR에서 `ui-standard/`와 동일한 역할의 컴포넌트를 다른 경로에 추가하는 경우, 리뷰에서 반려하고 기존 공통 컴포넌트 사용으로 교체하는 것을 표준 절차로 한다.

```typescript
// apps/web/src/components/ui-standard/ConfirmDialog.tsx
// 예시 시그니처 — 계산 없음, 콜백만 노출
interface ConfirmDialogProps {
  open: boolean;
  title: string;
  onConfirm: () => void;
  onCancel: () => void;
}
```

### 4.3 api 호출 경계

- `web`은 Supabase DB에 직접 접근하지 않고 `lib/api-client/`를 통해서만 `api` 서비스를 호출한다([ARCHITECTURE.md](ARCHITECTURE.md) §2.1).
- `lib/api-client/` 내부 함수는 fetch 래퍼·응답 타입 매핑만 수행하며, 수당/정산 등 계산 로직을 포함하지 않는다.

---

## 5. Database 규칙

[DATABASE.md](DATABASE.md) §1의 컨벤션을 그대로 인용한다(임의 수정 금지, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.1 문서 관련 원칙):

> - 엔진: Supabase (PostgreSQL)
> - 네이밍: 테이블/컬럼은 `snake_case`, 테이블명은 복수형(예: `members`, `orders`)을 기본으로 한다 (확정 필요).
> - 금액(money) 컬럼: 정수(원 단위, KRW 기준 소수점 없음) 저장을 기본으로 검토 — 확정 필요.
> - 식별자: UUID 기본키를 기본으로 검토 (Supabase 기본값과 호환).
> - 시각: 모든 timestamp는 UTC로 저장, 표시 시점에 KST 변환.

- **ORM/마이그레이션 도구는 미확정(O-022)** — [DECISIONS.md](DECISIONS.md)에 결정이 기록되기 전까지 특정 ORM(Prisma/TypeORM/Drizzle 등)을 가정한 코드를 작성하지 않는다. 도구가 확정되면 본 절을 갱신하고, NestJS Repository 계층의 디렉터리 구조(예: `*.repository.ts`)도 함께 확정한다.
- append-only 원장 테이블(`commission_records`/`settlement_items`/`audit_logs`)에 대한 Repository는 `update`/`delete` 메서드를 노출하지 않는다 — 보정(역분개) 엔트리 추가만 가능한 `insertCorrection()` 류의 메서드만 제공한다([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2).

---

## 6. Naming Rule

| 대상 | 규칙 | 예시 |
|---|---|---|
| 변수 | camelCase | `memberId`, `commissionAmount` |
| 함수/메서드 | camelCase, 동사로 시작 | `calculateCommission()`, `enqueueSettlementJob()` |
| 불리언 변수/함수 | `is`/`has`/`should` 접두사 | `isActive`, `hasPendingApproval` |
| 클래스/인터페이스/타입 | PascalCase | `MemberService`, `interface CreateMemberDto` |
| Enum 타입명 | PascalCase | `enum ChangeType` |
| Enum 멤버 | UPPER_SNAKE_CASE ([DATABASE.md](DATABASE.md) §3.9 `change_type` 값 표기와 통일) | `ChangeType.WITHDRAWAL`, `ChangeType.BANK_ACCOUNT_CHANGE` |
| 상수(모듈 레벨) | UPPER_SNAKE_CASE | `MAX_COMPLIANCE_RATIO`, `DEFAULT_PAGE_SIZE` |
| 환경 변수 | UPPER_SNAKE_CASE | `SUPABASE_URL`, `REDIS_URL` |
| DB 식별자(테이블/컬럼) | snake_case ([DATABASE.md](DATABASE.md) §1 그대로) | `commission_records`, `sponsor_id` |

- 도메인 용어는 [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md)/[PRD.md](PRD.md)의 한글 용어를 코드 네이밍에 영문으로 그대로 매핑한다 — 예: "조직 이동" → `OrganizationTransfer`(Admin Console 전용 용어), Partner Portal 노출 문자열에서는 "조직도" 대신 "내 조직"을 쓰되 변수/클래스명은 도메인 모델 기준(`OrganizationTransfer`)을 유지한다([ARCHITECTURE.md](ARCHITECTURE.md) §2.1, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.4 — UI 문구 규칙과 코드 네이밍 규칙은 별개 축).

---

## 7. Error Handling

- 공통 에러 클래스/응답 포맷은 **[API-SPEC.md](API-SPEC.md) 참조** — API 에러 포맷은 그 문서에서 정의하며 본 문서는 중복 정의하지 않는다.
- NestJS 레벨에서는 공통 예외 필터(`common/filters/`, §1.2)를 `api`/`worker`/`scheduler` 모두에 동일하게 적용해, [API-SPEC.md](API-SPEC.md)가 정의한 에러 포맷으로 일관되게 변환한다.
- worker의 Job 처리 실패는 HTTP 에러가 아니라 BullMQ의 재시도/실패(Failed) 상태로 표현되며, 최종 실패 시 Redis Job Tracking에 실패 사유를 남기고 Supabase에는 쓰지 않는다([ARCHITECTURE.md](ARCHITECTURE.md) §2.5).
- 도메인 예외(예: 법적 한도 차단 — [ARCHITECTURE.md](ARCHITECTURE.md) §8.1.3)는 일반 HTTP 4xx/5xx와 구분되는 도메인 예외 클래스로 표현하고, [API-SPEC.md](API-SPEC.md)의 에러 코드 체계에 맞춰 매핑한다.

---

## 8. Logging

### 8.1 로그 레벨

| 레벨 | 용도 |
|---|---|
| `error` | 처리 실패, 예외 — 즉시 조치가 필요한 상태 |
| `warn` | 법적 한도 "주의/경고" 단계 도달([ARCHITECTURE.md](ARCHITECTURE.md) §8.1.2) 등 잠재적 문제 |
| `info` | Job 생성/완료, 요청 처리 완료 등 정상 흐름의 주요 이벤트 |
| `debug` | 개발 단계 상세 추적 — 프로덕션에서는 기본 비활성 |

### 8.2 구조화 로깅

- 모든 서비스(api/worker/scheduler)는 구조화(JSON) 로그를 출력한다 — 사람이 읽는 문자열 로그가 아니라 `{ level, message, service, module, jobId?, memberId?, timestamp }` 형태의 필드를 갖는다.
- 로그에 PII(주민번호/계좌번호/사업자번호 등 [DATABASE.md](DATABASE.md) §2 `member_identity_profiles` 대상 정보)를 평문으로 남기지 않는다([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2).
- 구체적 로깅 라이브러리/APM(Sentry 등) 도입 여부는 **미확정** — [ARCHITECTURE.md](ARCHITECTURE.md) §6 참조.

### 8.3 audit_logs 와 일반 애플리케이션 로그의 구분

| 구분 | `audit_logs` ([DATABASE.md](DATABASE.md) §3.8) | 일반 애플리케이션 로그 |
|---|---|---|
| 목적 | 도메인 변경의 법적/감사 증적 — 누가 무엇을 언제 바꿨는지 | 운영/디버깅을 위한 기술적 추적 |
| 저장 위치 | Supabase PostgreSQL (append-only, source of truth) | 로그 수집기/콘솔(Railway 기본 로그 등, 미확정) |
| 기록 대상 | 민감 변경 요청/승인/반려, 조직 이동, 권한 위반 시도 등 [DATABASE.md](DATABASE.md) §3.8/§3.9/§3.26이 정의한 도메인 이벤트 | HTTP 요청/응답, Job 시작/종료, 예외 스택트레이스 등 모든 기술적 이벤트 |
| 기록 위치(코드) | 각 모듈의 `{Domain}Service`가 도메인 트랜잭션의 일부로 직접 기록(api에서 직접 기록 — [ARCHITECTURE.md](ARCHITECTURE.md) §2.2 Audit 모듈, 경량) | NestJS 공통 인터셉터/로거(`common/interceptors/`)가 횡단 관심사로 일괄 기록 |
| 삭제/수정 | 불가 (append-only, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2) | 보존 기간 정책에 따라 로테이션/삭제 가능 |

- 두 로그를 같은 저장소에 섞어 쓰지 않는다 — `audit_logs`에 일반 디버깅 로그를 남기거나, 애플리케이션 로그만으로 감사 추적을 대체하지 않는다.
