# STATE-MACHINE.md — State Machine Catalog

> 상태: 신규 v0.1 (문서 표준화 — 기존 상태값 색인) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 목적: 기존 문서에 이미 정의된 상태값과 전이를 Mermaid 다이어그램으로 모아본다. 본 문서는 새로운 상태값을 만들지 않는다.

## 0. 작성 원칙

- 상태값은 기존 문서에 명시된 값만 사용한다.
- 상태값/전환 조건이 미확정인 영역은 `미확정`으로 표기하고 임의 상태를 추가하지 않는다.
- 상태 전이의 구현 권한/API는 [API-SPEC.md](API-SPEC.md), 상태 source of truth는 [DATABASE.md](DATABASE.md)를 따른다.

## 1. 회원 상태

Source: [PRD.md](PRD.md) §5.6.3, [DATABASE.md](DATABASE.md) §3.12.

```mermaid
stateDiagram-v2
    PENDING_VERIFICATION: 가입심사중
    ACTIVE: 활성
    DORMANT: 휴면
    WITHDRAWN: 탈퇴
    FORCED_WITHDRAWN: 강제탈퇴
    REJECTED: 가입 불승인

    [*] --> PENDING_VERIFICATION: 신규 가입/재가입
    PENDING_VERIFICATION --> ACTIVE: 심사 승인
    PENDING_VERIFICATION --> REJECTED: 심사 반려
    ACTIVE --> DORMANT: 장기 미활동(기준 미확정)
    DORMANT --> ACTIVE: 재활동
    ACTIVE --> WITHDRAWN: 자발적 탈퇴 승인
    DORMANT --> WITHDRAWN: 자발적 탈퇴 승인
    ACTIVE --> FORCED_WITHDRAWN: 약관/법령 위반
    DORMANT --> FORCED_WITHDRAWN: 약관/법령 위반
    WITHDRAWN --> PENDING_VERIFICATION: 재가입
```

> `SUSPENDED` 등 추가 상태 필요 여부는 미확정이다. 탈퇴/강제탈퇴는 `sponsor_id`를 자동 변경하지 않는다.

## 2. 회원 민감 변경 요청

Source: [PRD.md](PRD.md) §5.4, [DATABASE.md](DATABASE.md) §3.9.

```mermaid
stateDiagram-v2
    REQUESTED: 요청
    UNDER_REVIEW: 검토중/영향분석
    APPROVED: 승인
    REJECTED: 반려
    APPLIED: 적용완료

    [*] --> REQUESTED
    REQUESTED --> UNDER_REVIEW: Snapshot/영향분석 요청
    UNDER_REVIEW --> APPROVED: 운영자 승인
    UNDER_REVIEW --> REJECTED: 운영자 반려
    APPROVED --> APPLIED: members 현재값/이력 반영
```

## 3. 조직 이동

Source: [DATABASE.md](DATABASE.md) §3.26, [ARCHITECTURE.md](ARCHITECTURE.md) §7.1.

```mermaid
stateDiagram-v2
    REQUESTED: REQUESTED
    UNDER_REVIEW: UNDER_REVIEW
    APPROVED_SCHEDULED: APPROVED_SCHEDULED
    APPLIED: APPLIED
    REJECTED: REJECTED

    [*] --> REQUESTED
    REQUESTED --> UNDER_REVIEW: 증빙첨부/영향분석
    UNDER_REVIEW --> APPROVED_SCHEDULED: 승인 및 effective_date 예약
    UNDER_REVIEW --> REJECTED: 반려
    APPROVED_SCHEDULED --> APPLIED: effective_date 도달 후 worker 적용
    UNDER_REVIEW --> APPLIED: 긴급 조직 이동 승인(SuperAdmin)
```

## 4. 주문

Source: [DATABASE.md](DATABASE.md) §3.3, [API-SPEC.md](API-SPEC.md) §2.4.

```mermaid
stateDiagram-v2
    PAID: 결제완료/PAID
    CANCELLED: 취소/CANCELLED
    REFUNDED: 환불

    [*] --> PAID: 주문 생성/결제 완료
    PAID --> CANCELLED: 주문 취소
    PAID --> REFUNDED: 환불
```

> 주문 취소/환불 시 매출 차감 처리 방식은 미확정이다. 배송 세부 상태는 3PL 연동과 함께 미확정이므로 이 다이어그램에 임의 상태를 추가하지 않는다.

## 5. 정기배송

Source: [DATABASE.md](DATABASE.md) §3.30.

```mermaid
stateDiagram-v2
    ACTIVE: ACTIVE
    PAUSED: PAUSED
    CANCELLED: CANCELLED

    [*] --> ACTIVE: 정기배송 등록
    ACTIVE --> PAUSED: 일시중지
    PAUSED --> ACTIVE: 재개
    ACTIVE --> CANCELLED: 해지
    PAUSED --> CANCELLED: 해지
```

## 6. 정산 배치

Source: [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9, [DATABASE.md](DATABASE.md) §3.6, [API-SPEC.md](API-SPEC.md) §2.5.

```mermaid
stateDiagram-v2
    CREATED: 생성
    VERIFIED: 검증
    APPROVED: 승인
    PAID: 지급완료
    BLOCKED: 검증 단계 보류(35% Hard Gate)

    [*] --> CREATED: commission_records 집계
    CREATED --> VERIFIED: 법적 한도 검증 통과
    CREATED --> BLOCKED: 35% Hard Gate
    BLOCKED --> VERIFIED: 검토/해제 방식은 O-004 확정 후
    VERIFIED --> APPROVED: 운영자 승인
    APPROVED --> PAID: 지급 실행
```

## 7. Compliance Ratio

Source: [DATABASE.md](DATABASE.md) §3.29, [ARCHITECTURE.md](ARCHITECTURE.md) §8.1.

```mermaid
stateDiagram-v2
    SAFE: SAFE
    CAUTION: CAUTION
    WARNING: WARNING
    BLOCKED: BLOCKED

    SAFE --> CAUTION: KR 30% 이상
    CAUTION --> WARNING: KR 33% 이상
    WARNING --> BLOCKED: KR 35% 이상
    BLOCKED --> WARNING: 비율 하락/정정 후 재계산
```

## 8. Job 상태

Source: [API-SPEC.md](API-SPEC.md) §1.7.

```mermaid
stateDiagram-v2
    QUEUED: QUEUED
    PROCESSING: PROCESSING
    RETRYING: RETRYING
    COMPLETED: COMPLETED
    FAILED: FAILED

    [*] --> QUEUED
    QUEUED --> PROCESSING
    PROCESSING --> COMPLETED
    PROCESSING --> RETRYING: 실패 후 재시도
    RETRYING --> PROCESSING
    RETRYING --> FAILED: 재시도 소진
```

## 9. Marketing Program 신청

Source: [PRD.md](PRD.md) §5.24, [DATABASE.md](DATABASE.md) §3.34.1.

```mermaid
stateDiagram-v2
    APPLIED: APPLIED/신청
    PENDING_APPROVAL: PENDING_APPROVAL/승인대기
    APPROVED: APPROVED/승인
    REJECTED: REJECTED/반려
    IN_PROGRESS: IN_PROGRESS/참여중
    COMPLETED: COMPLETED/완료

    [*] --> APPLIED
    APPLIED --> PENDING_APPROVAL
    PENDING_APPROVAL --> APPROVED
    PENDING_APPROVAL --> REJECTED
    APPROVED --> IN_PROGRESS
    IN_PROGRESS --> COMPLETED
```

## 10. Point Transaction

Source: [PRD.md](PRD.md) §5.23, [DATABASE.md](DATABASE.md) §3.36.

```mermaid
stateDiagram-v2
    EARN: EARN
    DEDUCT: DEDUCT
    USE_REQUEST: USE_REQUEST
    USE_APPROVED: USE_APPROVED
    USE_REJECTED: USE_REJECTED
    CANCEL: CANCEL
    RESTORE: RESTORE
    EXPIRE: EXPIRE

    [*] --> EARN: 적립
    [*] --> DEDUCT: 관리자 차감
    EARN --> USE_REQUEST: 사용 신청
    USE_REQUEST --> USE_APPROVED: 관리자 승인
    USE_REQUEST --> USE_REJECTED: 반려
    USE_APPROVED --> CANCEL: 사후 취소(보정 엔트리)
    CANCEL --> RESTORE: 복원
    EARN --> EXPIRE: 만료
```

> `point_transactions`는 append-only 원장이다. 위 다이어그램은 원본 행 업데이트가 아니라 관련 거래 유형의 흐름을 나타낸다.

## 11. Workflow Engine

Source: [DATABASE.md](DATABASE.md) §3.37.

```mermaid
stateDiagram-v2
    IN_PROGRESS: IN_PROGRESS
    APPROVED: APPROVED
    REJECTED: REJECTED
    CANCELLED: CANCELLED

    [*] --> IN_PROGRESS
    IN_PROGRESS --> APPROVED: 모든 단계 승인
    IN_PROGRESS --> REJECTED: 반려
    IN_PROGRESS --> CANCELLED: 취소
```

## 12. External API Connection

Source: [DATABASE.md](DATABASE.md) §3.38.

```mermaid
stateDiagram-v2
    TESTING: TESTING
    ACTIVE: ACTIVE
    INACTIVE: INACTIVE

    [*] --> TESTING
    TESTING --> ACTIVE: 검증 완료
    ACTIVE --> INACTIVE: 비활성화
    INACTIVE --> ACTIVE: 재활성화
```

## 13. 국가 상태

Source: [DATABASE.md](DATABASE.md) §3.13, [PRD.md](PRD.md) §5.6.2.

```mermaid
stateDiagram-v2
    PLANNED: PLANNED
    ACTIVE: ACTIVE
    RESERVED: RESERVED

    [*] --> PLANNED: TH/JP/US
    PLANNED --> ACTIVE: 명시적 출시 결정
    [*] --> ACTIVE: KR
    [*] --> RESERVED: CN
```

> RESERVED에서 ACTIVE로의 전환은 일반 토글이 아니라 별도 명시적 의사결정이 필요하다.

## 14. 미확정 상태 영역

| 영역 | 현재 문서 상태 |
|---|---|
| 배송 상세 상태 | 배송 상태/운송장/3PL 추적은 언급되어 있으나 표준 상태 enum 미확정 |
| 반품/교환 상태머신 | Gap Analysis에서 갭으로 식별, 구체 상태 미확정 |
| CMS 콘텐츠 상태 | DRAFT/IN_REVIEW/SCHEDULED/PUBLISHED 도입 여부 미확정(O-137) |
| CRM 상담/예약 상태 | 진행중/완료/Follow-up필요 등 표기 있으나 상세 전이 미확정 |

