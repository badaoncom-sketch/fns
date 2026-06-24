# DO-NOT-TOUCH.md — 절대 임의 변경 금지 대상

> 상태: Draft v0.3 (D-031 반영 — 폐기된 `pv_ledger`/`member_rank_history`를 append-only 보호 목록에서 제거) · 최종 수정일: 2026-06-24 · 단계: 설계(Design)
> 목적: 사람/AI(Claude Code, Codex) 모두에게 적용되는, 명시적 승인 없이 변경·삭제·실행해서는 안 되는 대상을 정의한다.

## 1. 현재(설계 단계) 기준 원칙

이 시점에는 아직 코드/운영 환경이 없으므로, 아래는 **앞으로 적용될 원칙의 선언**이며 구현 단계 진입 시 그대로 유효하다.

### 1.1 문서 관련

- [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md), [PRD.md](PRD.md) 등 합의된 문서의 내용을 **사업팀 논의 없이 임의로 변경하지 않는다.** 특히 [COMPENSATION-RULES.md](COMPENSATION-RULES.md), [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)의 수치(비율, 한도, 조건)는 [DECISIONS.md](DECISIONS.md)에 결정 기록이 남기 전까지 "확정"으로 표시하지 않는다. ([PROMOTION-RULES.md](PROMOTION-RULES.md)는 D-030으로 폐기되어 본 원칙의 대상에서 제외)
- [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md)의 법적 판단은 변호사/세무사 검토 없이 "확정"으로 전환하지 않는다.

### 1.2 코드/데이터 관련 (구현 단계 진입 시 적용)

- **정산(커미션) 계산 로직**: 법적·금전적 영향이 크므로, [COMPENSATION-RULES.md](COMPENSATION-RULES.md)/[SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)에 명시적으로 확정되지 않은 수치나 로직을 임의로 구현하지 않는다.
- **append-only 원장 테이블** (`commission_records`, `settlement_items`, `audit_logs` — [DATABASE.md](DATABASE.md)): 기존 행을 `UPDATE`/`DELETE`로 직접 수정하지 않는다. 정정은 반드시 보정(역분개) 엔트리 추가로 처리한다. (`pv_ledger`/`member_rank_history`는 D-030으로 폐기되어 더 이상 존재하지 않음 — 목록에서 제외)
- **실 회원 PII / 실 정산 금액 데이터**: 개발/테스트 환경에서는 가상/목업 데이터만 사용한다. 실 데이터를 비프로덕션 환경으로 복제하지 않는다.
- **프로덕션 데이터베이스**: 마이그레이션/배포 파이프라인을 거치지 않은 직접 수정(콘솔 등)을 하지 않는다.
- **`.env`, 시크릿, DB 자격증명, Railway/Supabase API 키**: 레포에 커밋하지 않으며, 코드/문서에 평문으로 기록하지 않는다.

### 1.3 서버 구조 관련 (구현 단계 진입 시 적용 — [DECISIONS.md](DECISIONS.md) D-010)

FNS의 서버는 Railway 프로젝트 1개 안의 **web/api/worker/scheduler/redis** 5개 서비스로 확정되어 있다 ([ARCHITECTURE.md](ARCHITECTURE.md) §1~§2). 다음을 위반하지 않는다.

- **web(Next.js)에 후원수당/정산/세금/프로모션 등 계산 로직을 작성하지 않는다.** web은 UI 렌더링과 api 호출만 한다.
- **api(NestJS)의 request handler에 무거운 계산(후원수당/정산/세금/프로모션 판정/보고서 생성)을 직접 구현하지 않는다.** api는 Job 생성과 상태/결과 조회만 담당한다.
- **scheduler에 계산 로직을 작성하지 않는다.** scheduler는 Job 트리거(cron 등록)만 담당하며, 실제 처리는 worker에 위임한다.
- **Redis를 source of truth로 취급하지 않는다.** Redis 데이터(큐/캐시/Job 상태)가 소실되어도 비즈니스 데이터가 손실되지 않아야 하며, 최종 데이터는 항상 Supabase PostgreSQL에 기록한다.
- **worker가 아닌 서비스(web/api/scheduler)에서 `commission_records`/`settlement_items` 등 핵심 append-only 원장에 직접 쓰지 않는다.** 이 원장들은 worker만 기록한다.
- **5개 서비스 구조(web/api/worker/scheduler/redis)를 임의로 합치거나 쪼개지 않는다.** 구조 변경이 필요하면 먼저 [DECISIONS.md](DECISIONS.md)에 결정을 기록하고 [ARCHITECTURE.md](ARCHITECTURE.md)를 갱신한 뒤에 진행한다.

### 1.4 Git/배포 관련

- `main`(또는 기본) 브랜치에 대한 **force push, 히스토리 재작성**을 하지 않는다.
- 명시적 요청 없이 **git hook을 우회**(`--no-verify` 등)하지 않는다.

## 2. 위반 시 처리

- 위 원칙을 위반해야 하는 상황이 발생하면, 먼저 사용자(프로젝트 오너)에게 보고하고 명시적 승인을 받은 뒤에만 진행한다.

## 3. 향후 추가 예정 (구현 단계 진입 시 보강)

- 프로덕션 환경 변수 목록 및 접근 권한 정책
- 배포 파이프라인에서 보호해야 할 브랜치/환경 목록
- 데이터 백업/복구 정책과 연계된 추가 제약
