# EVENT-CATALOG.md — Event Catalog

> 상태: 신규 v0.1 (문서 표준화 — 기존 문서 기반 이벤트 색인) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 목적: ERP 전체에서 기존 문서에 이미 등장하는 "발생/요청/승인/처리/발송" 흐름을 이벤트 관점으로 찾을 수 있게 정리한다. 본 문서는 이벤트 버스 도입이나 신규 기능을 확정하지 않는다.

## 0. 작성 원칙

- Event 이름은 개발자가 대화할 때 쓰기 위한 **카탈로그명**이다. 구현 이벤트명/토픽명은 아직 확정하지 않는다.
- Trigger/Publisher/Subscriber는 [ARCHITECTURE.md](ARCHITECTURE.md), [API-SPEC.md](API-SPEC.md), [DATABASE.md](DATABASE.md)에 명시된 책임 경계를 따른다.
- `Worker 여부`가 Yes인 이벤트는 계산/집계/발송/문서 생성 등 비동기 Job 패턴을 따르는 흐름이다.

## 1. Event Catalog

| Event | Trigger | Publisher | Subscriber | Worker 여부 | API 여부 | 관련 문서 |
|---|---|---|---|---|---|---|
| MemberRegistered | 회원 가입 요청 생성 | api: Member | Supabase/Auth, Member | No | Yes | [API-SPEC.md](API-SPEC.md) §2.2, [PRD.md](PRD.md) §5.6 |
| MemberIdentitySubmitted | 회원 유형별 본인확인 정보 제출 | api: Member | MemberIdentity/Profile Review | No | Yes | [DATABASE.md](DATABASE.md) §3.14, [API-SPEC.md](API-SPEC.md) §2.2 |
| MemberApproved | 가입심사 승인 | api: Member | Member, Audit | No | Yes | [PRD.md](PRD.md) §5.6.3, [API-SPEC.md](API-SPEC.md) §2.2 |
| MemberChangeRequested | 회원 민감 변경 요청 | api: MemberLifecycle | worker 영향분석, Audit | Yes | Yes | [PRD.md](PRD.md) §5.3/§5.4, [ARCHITECTURE.md](ARCHITECTURE.md) §7 |
| MemberChangeImpactAnalyzed | 민감 변경 영향분석 완료 | worker | MemberLifecycle, Audit | Yes | 조회 API | [DATABASE.md](DATABASE.md) §3.9, [API-SPEC.md](API-SPEC.md) §2.6 |
| MemberChangeApproved | 민감 변경 승인 | api: MemberLifecycle | Member, Audit, Logistics(탈퇴/강제탈퇴 시) | No | Yes | [ARCHITECTURE.md](ARCHITECTURE.md) §7, [PRD.md](PRD.md) §5.4 |
| MemberChangeRejected | 민감 변경 반려 | api: MemberLifecycle | Audit | No | Yes | [API-SPEC.md](API-SPEC.md) §2.6 |
| OrganizationTransferRequested | 조직 이동 요청 | api: OrganizationTransfer | worker 영향분석, Audit | Yes | Yes | [PRD.md](PRD.md) §5.16, [DATABASE.md](DATABASE.md) §3.26 |
| OrganizationTransferApproved | 조직 이동 승인 및 적용일 예약 | api: OrganizationTransfer | scheduler/worker 적용 배치, Audit | 적용은 Yes | Yes | [ARCHITECTURE.md](ARCHITECTURE.md) §7.1, [API-SPEC.md](API-SPEC.md) §2.7 |
| OrganizationTransferApplied | effective_date 도달 또는 긴급 적용 | worker | Member, SponsorHistory, Audit | Yes | No(결과 조회만) | [ARCHITECTURE.md](ARCHITECTURE.md) §7.1.1, [DATABASE.md](DATABASE.md) §3.26 |
| OrganizationTransferRejected | 조직 이동 반려 | api: OrganizationTransfer | Audit | No | Yes | [API-SPEC.md](API-SPEC.md) §2.7 |
| ReferralLinkClicked | 추천 링크 클릭 | web/api | Analytics, ReferralTracking | No | Yes/Tracking | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6, [DATABASE.md](DATABASE.md) §3.28 |
| OrderCreated | 주문 생성 | api: Order | Order, Compensation Job trigger | 부분 Yes | Yes | [API-SPEC.md](API-SPEC.md) §2.4/§3.3, [ARCHITECTURE.md](ARCHITECTURE.md) §2.2 |
| OrderPaid | 주문 결제 완료/매출 기록 | api: Order | worker Compensation, Compliance cache | Yes | Yes | [API-SPEC.md](API-SPEC.md) §2.4, [ARCHITECTURE.md](ARCHITECTURE.md) §8.1 |
| OrderCancelled | 주문 취소 | api: Order | Order, Commission/Settlement adjustment path | 미확정 | Yes | [API-SPEC.md](API-SPEC.md) §2.4, [DATABASE.md](DATABASE.md) §3.3 |
| ReturnRequested | 반품 접수 | api: Logistics | Logistics/3PL | 상황별 | Yes | [PRD.md](PRD.md) §5.5, [API-SPEC.md](API-SPEC.md) §2.9 |
| InventoryReconciliationRequested | 3PL 재고 정합성 대조 요청 | scheduler/api | worker Logistics | Yes | Yes/Internal | [ARCHITECTURE.md](ARCHITECTURE.md) §2.7, [API-SPEC.md](API-SPEC.md) §2.9 |
| RecurringOrderDue | 정기배송 처리일 도달 | scheduler | worker RecurringOrder | Yes | No/Internal | [ARCHITECTURE.md](ARCHITECTURE.md) §2.3, [DATABASE.md](DATABASE.md) §3.30 |
| PackagePurchased | 패키지 상품 구매 | api: Order | worker Compensation, Package policy lookup | Yes | 주문 API | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1, [DATABASE.md](DATABASE.md) §3.24/§3.24.1 |
| PairCandidateQueued | 페어 대기 후보 등록/만료 판단 | worker Compensation | Package Pair logic | Yes | No | [DATABASE.md](DATABASE.md) §3.24, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.3 |
| PairBonusMatched | 페어 성립 | worker Compensation | CommissionRecords | Yes | No | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.3 |
| CommissionCalculationRequested | 후원수당 계산 Job 요청 | scheduler/admin via api | worker Compensation | Yes | Yes | [API-SPEC.md](API-SPEC.md) §2.5, [ARCHITECTURE.md](ARCHITECTURE.md) §2.3 |
| CommissionCalculated | 수당 계산 완료 | worker | commission_records, Compliance ratio | Yes | 결과 조회 | [DATABASE.md](DATABASE.md) §3.5, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §5 |
| ComplianceRatioUpdated | 35% 비율 스냅샷 갱신 | worker | Compliance dashboard, Settlement gate | Yes | 조회 API | [ARCHITECTURE.md](ARCHITECTURE.md) §8.1, [DATABASE.md](DATABASE.md) §3.29 |
| SettlementJobRequested | 정산 Job 요청 | scheduler/admin via api | worker Settlement | Yes | Yes | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9, [API-SPEC.md](API-SPEC.md) §2.5 |
| SettlementCreated | 정산 배치 생성/검증 단계 진입 | worker Settlement | settlement_batches/items | Yes | 결과 조회 | [DATABASE.md](DATABASE.md) §3.6 |
| SettlementBlockedByCompliance | 35% 한도 Hard Gate | worker Settlement/Compliance | Settlement, Notification candidate | Yes | 조회 API | [ARCHITECTURE.md](ARCHITECTURE.md) §8.1.3 |
| SettlementApproved | 운영자 정산 승인 | api: Settlement | Settlement, Audit | No | Yes | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9, [API-SPEC.md](API-SPEC.md) §2.5 |
| SettlementPaid | 지급 실행 및 지급완료 확정 | Settlement flow | settlement_items, tax_withholdings | 부분 Yes | 미확정 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 |
| TaxWithholdingCreated | 원천징수 내역 생성 | worker Settlement | tax_withholdings | Yes | 조회 API | [DATABASE.md](DATABASE.md) §3.7 |
| ComplianceReportRequested | 공제조합 보고서 생성 요청 | scheduler/admin via api | worker Compliance | Yes | Yes | [ARCHITECTURE.md](ARCHITECTURE.md) §8, [API-SPEC.md](API-SPEC.md) §2.8 |
| ComplianceReportGenerated | 공제조합 보고서 생성 완료 | worker | Supabase Storage/File, submissions | Yes | 조회 API | [DATABASE.md](DATABASE.md) §3.16 |
| ComplianceReportSubmitted | 보고서 제출완료 상태 전이 | api: Compliance | ComplianceSubmission, Audit | No | Yes | [API-SPEC.md](API-SPEC.md) §2.8 |
| NotificationCreated | 알림 발송 요청 생성 | api: Notification | worker Notification | Yes | Yes | [ARCHITECTURE.md](ARCHITECTURE.md) §2.2/§2.3, [API-SPEC.md](API-SPEC.md) §2.13 |
| NotificationSent | 알림 발송 성공 | worker Notification | notification_logs | Yes | 결과 조회 | [DATABASE.md](DATABASE.md) §3.20/§3.41 |
| NotificationFailed | 알림 발송 실패 | worker Notification | Redis Retry, notification_logs | Yes | 결과 조회 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.5 |
| DocumentGenerated | 회원 개인 문서/리포트 생성 | worker Document/Report | Storage/File Manager | Yes | 조회 API | [ARCHITECTURE.md](ARCHITECTURE.md) §2.3, [DATABASE.md](DATABASE.md) §3.18/§3.39 |
| WorkflowStarted | 범용 워크플로우 인스턴스 시작 | api: Workflow | Workflow, Notification rule | No/상황별 | Yes | [PRD.md](PRD.md) §5.30, [DATABASE.md](DATABASE.md) §3.37 |
| WorkflowApproved | 워크플로우 승인 | api: Workflow | workflow_step_actions, subject module | No | Yes | [DATABASE.md](DATABASE.md) §3.37 |
| WorkflowRejected | 워크플로우 반려 | api: Workflow | workflow_step_actions, subject module | No | Yes | [DATABASE.md](DATABASE.md) §3.37 |
| ProgramJoined | Marketing Program 신청 | api: MarketingProgram | marketing_program_applications | No | Yes | [PRD.md](PRD.md) §5.24, [API-SPEC.md](API-SPEC.md) §2.19 |
| ProgramApproved | Marketing Program 신청 승인 | api: MarketingProgram | Program application, Audit | No | Yes | [API-SPEC.md](API-SPEC.md) §2.19 |
| ProgramCompleted | Marketing Program 완료 | api: MarketingProgram | Point trigger candidate | 부분 Yes | Yes | [PRD.md](PRD.md) §5.24 |
| PointEarned | 포인트 적립 | worker/api depending source | point_transactions | 경우별 | 조회 API | [PRD.md](PRD.md) §5.23, [DATABASE.md](DATABASE.md) §3.36 |
| PointUseRequested | 포인트 사용 신청 | api: Point | point_transactions | No | Yes | [API-SPEC.md](API-SPEC.md) §2.20 |
| PointUseApproved | 포인트 사용 승인 | api: Point | point_transactions | No | Yes | [DATABASE.md](DATABASE.md) §3.36 |
| PointExpired | 포인트 만료 | scheduler | worker Point | Yes | No/Internal | [ARCHITECTURE.md](ARCHITECTURE.md) §2.3 |
| ExternalApiCalled | 외부 API 호출 기록 | api/worker module | API Center log | No/상황별 | 내부 | [DATABASE.md](DATABASE.md) §3.38 |
| ScheduledJobRunStarted | Scheduler Job 실행 시작 | scheduler | scheduled_job_run_logs | No | 내부 | [DATABASE.md](DATABASE.md) §3.40 |
| ScheduledJobRunCompleted | Scheduler Job 실행 종료 | scheduler/worker | scheduled_job_run_logs | 경우별 | 내부 | [DATABASE.md](DATABASE.md) §3.40 |

## 2. Responsibility Boundary

| 경계 | 원칙 | Source |
|---|---|---|
| web | UI 렌더링과 api 호출만 담당 | [ARCHITECTURE.md](ARCHITECTURE.md) §1.1 |
| api | 요청 검증, 상태 전이, Job 생성/조회 담당. 무거운 계산 금지 | [ARCHITECTURE.md](ARCHITECTURE.md) §1.1/§2.2 |
| worker | 수당/정산/보고서/알림/문서/정합성 대조 등 무거운 처리 담당 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.3 |
| scheduler | 시간 기반 Job 트리거만 담당 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.4 |
| Redis | Queue/Cache/Job Tracking/Retry. source of truth 아님 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.5 |
| Supabase PostgreSQL | 최종 데이터 source of truth | [ARCHITECTURE.md](ARCHITECTURE.md) §2.6 |

