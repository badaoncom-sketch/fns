# DATA-DICTIONARY.md — Data Dictionary

> 상태: 신규 v0.1 (문서 표준화 — DATABASE.md 기반 컬럼 사전) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 목적: [DATABASE.md](DATABASE.md)에 흩어진 테이블/컬럼 개념을 구현자가 빠르게 찾을 수 있도록 정리한다. 본 문서는 DB 스키마를 변경하지 않는다.

## 0. 작성 원칙

- 실제 DB 타입, nullable, default는 대부분 아직 확정되지 않았다. 아래 타입은 문서상 개념을 구현 후보 타입으로 정리한 것이다.
- `PK/FK/Nullable/Default`가 원문에 명시되지 않은 경우 `미정`으로 둔다.
- `Source of Truth`는 반드시 [DATABASE.md](DATABASE.md)의 해당 섹션이다. 관련 Business Rule은 [BUSINESS-RULE-CATALOG.md](BUSINESS-RULE-CATALOG.md)를 참조한다.

## 1. Core Member / Organization

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| members | id | 회원 식별자 | uuid | Y | N | N | 미정 | DATABASE §3.1 | BR-001 |
| members | auth_user_id | Supabase Auth 사용자 연결 | uuid | N | Supabase Auth | 미정 | 미정 | DATABASE §3.1 | BR-001 |
| members | sponsor_id | 상위 후원자 현재값 | uuid | N | members.id | Y | 최상위 nullable | DATABASE §3.1 | BR-005/BR-023 |
| members | status | 회원 상태 | enum | N | N | N | PENDING_VERIFICATION/ACTIVE/DORMANT/WITHDRAWN/FORCED_WITHDRAWN | DATABASE §3.1/§3.12 | BR-002/BR-022 |
| members | member_type | 회원 유형 | enum | N | N | N | INDIVIDUAL/BUSINESS/CORPORATION/FOREIGN | DATABASE §3.1 | BR-003 |
| members | country_code | 소속 국가 | string | N | countries.country_code | N | KR/TH/JP/US 등 | DATABASE §3.1/§3.13 | BR-004 |
| members | center_id | 소속 센터 | uuid | N | centers.id | Y | 센터 도입 미확정 | DATABASE §3.1/§3.17 | BR-043 |
| members | joined_at | 가입일 | timestamp | N | N | 미정 | 미정 | DATABASE §3.1 | BR-001 |
| members | previous_member_id | 재가입 시 이전 회원 참조 | uuid | N | members.id | Y | 미정 | DATABASE §3.1 | BR-002 |
| member_sponsor_history | member_id | 대상 회원 | uuid | 미정 | members.id | N | 미정 | DATABASE §3.9 | BR-026 |
| member_sponsor_history | previous_sponsor_id | 변경 전 스폰서 | uuid | N | members.id | 미정 | 미정 | DATABASE §3.9 | BR-026 |
| member_sponsor_history | new_sponsor_id | 변경 후 스폰서 | uuid | N | members.id | 미정 | 미정 | DATABASE §3.9 | BR-026 |
| member_sponsor_history | transfer_log_id | 조직 이동 원장 참조 | uuid | N | organization_transfer_logs.id | N | 미정 | DATABASE §3.9 | BR-025 |
| member_sponsor_history | effective_from | 변경 적용 시점 | timestamp | N | N | N | 미정 | DATABASE §3.9 | BR-026 |
| member_identity_profiles | member_id | 대상 회원 | uuid | 미정 | members.id | N | 미정 | DATABASE §3.14 | BR-003 |
| member_identity_profiles | member_type | 유형별 정보 구분 | enum | N | N | N | INDIVIDUAL/BUSINESS/CORPORATION/FOREIGN | DATABASE §3.14 | BR-003 |
| member_identity_profiles | document_refs | 제출 서류 Storage 참조 | json | N | Storage/File | 미정 | 미정 | DATABASE §3.14 | BR-002 |

## 2. Member Lifecycle / Organization Transfer

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| member_change_requests | id | 요청 식별자 | uuid | Y | N | N | 미정 | DATABASE §3.9 | BR-024 |
| member_change_requests | member_id | 대상 회원 | uuid | N | members.id | N | 미정 | DATABASE §3.9 | BR-024 |
| member_change_requests | change_type | 변경 유형 | enum | N | N | N | PROFILE_UPDATE/IDENTITY_TRANSFER/WITHDRAWAL/DORMANT/FORCED_WITHDRAWAL/REINSTATEMENT/BANK_ACCOUNT_CHANGE/CENTER_CHANGE | DATABASE §3.9 | BR-024 |
| member_change_requests | reason | 변경 사유 | text | N | N | 미정 | 미정 | DATABASE §3.9 | BR-024 |
| member_change_requests | before_snapshot | 변경 전 상태 스냅샷 | json | N | N | 미정 | 미정 | DATABASE §3.9 | BR-024 |
| member_change_requests | requested_state | 변경 후 제안 상태 | json | N | N | 미정 | 미정 | DATABASE §3.9 | BR-024 |
| member_change_requests | compensation_impact_id | 수당 영향 분석 결과 | uuid | N | change_impact_analyses.id | Y | 미정 | DATABASE §3.9 | BR-024 |
| member_change_requests | settlement_impact_id | 정산 영향 분석 결과 | uuid | N | change_impact_analyses.id | Y | 미정 | DATABASE §3.9 | BR-024 |
| member_change_requests | e_signature_id | 전자서명 참조 | uuid | N | e_signatures.id | 조건부 | 명의/탈퇴/강제탈퇴 필수 | DATABASE §3.9 | BR-024 |
| member_change_requests | status | 요청 상태 | enum | N | N | N | 요청/검토중/승인/반려/적용완료 | DATABASE §3.9 | BR-024 |
| member_change_requests | requested_by / approved_by | 요청자/승인자 | uuid | N | users/admins | 미정 | 미정 | DATABASE §3.9 | BR-024 |
| member_change_requests | requested_at / approved_at / applied_at | 단계별 시각 | timestamp | N | N | 미정 | 미정 | DATABASE §3.9 | BR-024 |
| change_impact_analyses | change_request_id | 변경 요청 참조 | uuid | 미정 | member_change_requests.id | N | 미정 | DATABASE §3.9 | BR-024 |
| change_impact_analyses | analysis_type | 분석 유형 | enum | N | N | N | COMPENSATION_IMPACT/SETTLEMENT_IMPACT | DATABASE §3.9 | BR-024 |
| change_impact_analyses | affected_member_ids | 영향받는 회원 목록 | json | N | members.id | 미정 | 미정 | DATABASE §3.9 | BR-024 |
| change_impact_analyses | estimated_delta | 예상 변동액 | json | N | N | 미정 | 미정 | DATABASE §3.9 | BR-024 |
| change_impact_analyses | input_snapshot | 시뮬레이션 입력 스냅샷 | json | N | N | 미정 | 미정 | DATABASE §3.9 | BR-024 |
| organization_transfer_reason_codes | code | 조직 이동 사유 코드 | enum/string | 미정 | N | N | 9종 사유, 정확한 enum O-067 | DATABASE §3.26 | BR-025 |
| organization_transfer_reason_codes | is_emergency_eligible | 긴급 이동 가능 여부 | boolean | N | N | N | 사유별 true/false | DATABASE §3.26 | BR-027 |
| organization_transfer_logs | id | 조직 이동 요청 식별자 | uuid | Y | N | N | 미정 | DATABASE §3.26 | BR-025 |
| organization_transfer_logs | member_id | 대상 회원 | uuid | N | members.id | N | 미정 | DATABASE §3.26 | BR-025 |
| organization_transfer_logs | previous_sponsor_id / new_sponsor_id | 변경 전/후 스폰서 | uuid | N | members.id | N | 미정 | DATABASE §3.26 | BR-026 |
| organization_transfer_logs | is_emergency | 긴급 조직 이동 여부 | boolean | N | N | N | false 후보 | DATABASE §3.26 | BR-027 |
| organization_transfer_logs | status | 조직 이동 상태 | enum | N | N | N | REQUESTED/UNDER_REVIEW/APPROVED_SCHEDULED/APPLIED/REJECTED | DATABASE §3.26 | BR-026 |
| organization_transfer_logs | effective_date | 적용 예정 일시 | timestamp | N | N | N | 기준시점 O-068/O-069 | DATABASE §3.26 | BR-026 |
| organization_transfer_logs | approved_at / applied_at | 승인/적용 시각 | timestamp | N | N | 미정 | 서로 다른 시각 | DATABASE §3.26 | BR-026 |

## 3. Commerce / Package / Compensation

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| products | id | 상품 식별자 | uuid | Y | N | N | 미정 | DATABASE §3.3 | BR-029 |
| products | name | 제품명 | string | N | N | N | 미정 | DATABASE §3.3 | BR-029 |
| products | price | 가격 | decimal | N | N | N | KRW 등 | DATABASE §3.3 | BR-029 |
| products | is_active | 활성 여부 | boolean | N | N | N | 미정 | DATABASE §3.3 | BR-029 |
| orders | id | 주문 식별자 | uuid | Y | N | N | 미정 | DATABASE §3.3 | BR-029 |
| orders | buyer | 구매자 | uuid/string | N | members/customers | 미정 | 회원 또는 고객 | DATABASE §3.3 | BR-029 |
| orders | status | 주문 상태 | enum | N | N | N | 결제완료/취소/환불 | DATABASE §3.3 | BR-029 |
| orders | ordered_at | 주문일 | timestamp | N | N | 미정 | 미정 | DATABASE §3.3 | BR-029 |
| orders | revenue_amount | 매출 금액 | decimal | N | N | 미정 | 미정 | DATABASE §3.3 | BR-019 |
| order_items | order_id | 주문 참조 | uuid | 미정 | orders.id | N | 미정 | DATABASE §3.3 | BR-029 |
| order_items | product_id | 상품 참조 | uuid | 미정 | products.id | N | 미정 | DATABASE §3.3 | BR-029 |
| order_items | quantity | 수량 | integer | N | N | N | 미정 | DATABASE §3.3 | BR-029 |
| order_items | unit_price | 단가 | decimal | N | N | N | 주문시점 스냅샷 | DATABASE §3.3 | BR-029 |
| packages | product_id | 패키지 상품 참조 | uuid | 후보 | products.id | N | unique | DATABASE §3.24.1 | BR-010 |
| packages | sales_start_at / sales_end_at | 판매기간 | timestamp | N | N | Y | 상시 판매 가능 | DATABASE §3.24.1 | BR-010 |
| packages | is_active | 활성 여부 | boolean | N | N | N | 미정 | DATABASE §3.24.1 | BR-010 |
| packages | country_codes | 판매 가능 국가 목록 | array | N | countries | Y | 빈 배열=전체 활성국 후보 | DATABASE §3.24.1 | BR-010 |
| package_commission_policies | package_id | 패키지 참조 | uuid | 미정 | packages.product_id | N | 미정 | DATABASE §3.24.1 | BR-010 |
| package_commission_policies | country_code | 적용 국가 | string | N | countries.country_code | N | 미정 | DATABASE §3.24.1 | BR-010 |
| package_commission_policies | effective_from / effective_to | 적용 기간 | timestamp | N | N | N/Y | to nullable | DATABASE §3.24.1 | BR-010 |
| package_commission_policies | referral_bonus_enabled | 제품 판매수익 사용 여부 | boolean | N | N | N | 미정 | DATABASE §3.24.1 | BR-011 |
| package_commission_policies | referral_bonus_rate / referral_bonus_amount | 비율/고정 추천수당 | decimal | N | N | 조건부 | 하나만 설정 | DATABASE §3.24.1 | BR-011 |
| package_commission_policies | pair_bonus_enabled | 페어보너스 사용 여부 | boolean | N | N | N | 미정 | DATABASE §3.24.1 | BR-012 |
| package_commission_policies | pair_bonus_amount | Pair당 지급액 | decimal | N | N | 조건부 | 미정 | DATABASE §3.24.1 | BR-012 |
| package_commission_policies | pair_bonus_window_days | 페어 판정 기간 | integer | N | N | 조건부 | 미정 | DATABASE §3.24.1 | BR-012 |
| package_commission_policies | counts_toward_unilevel_line_revenue | 유니레벨 라인매출 포함 여부 | boolean | N | N | N | 기본 false | DATABASE §3.24.1 | BR-008/BR-010 |
| package_commission_policies | grants_qualification | 제품판매수익·페어보너스 자격 부여 여부 | boolean | N | N | N | 미정 | DATABASE §3.24.1 | BR-009 |
| commission_records | recipient_member_id | 수령 회원 | uuid | 미정 | members.id | N | 원문은 개념명 | DATABASE §3.5 | BR-006~BR-014 |
| commission_records | commission_type | 수당 종류 | enum | N | N | N | 후원수당/제품 판매수익/페어보너스/+알파 등 | DATABASE §3.5 | BR-014 |
| commission_records | line_root_member_id | 라인 시작점 | uuid | N | members.id | 조건부 | 후원수당일 때 | DATABASE §3.5 | BR-007 |
| commission_records | plan_version_id | 적용 마케팅 플랜 버전 | uuid | N | marketing_plan_versions.id | N | 미정 | DATABASE §3.5 | BR-014 |
| commission_records | period | 산정 기간 | date/range | N | N | N | 월 단위 | DATABASE §3.5 | BR-014 |
| commission_records | amount | 산정 금액 | decimal | N | N | N | 미정 | DATABASE §3.5 | BR-014 |
| commission_records | input_snapshot | 계산식/입력 스냅샷 | json | N | N | N | append-only 재현성 | DATABASE §3.5 | BR-014 |

## 4. Settlement / Compliance / Tax

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| settlement_batches | id | 정산 회차 | uuid | Y | N | N | 미정 | DATABASE §3.6 | BR-015 |
| settlement_batches | period | 정산 기간 | date/range | N | N | N | 월 단위 | DATABASE §3.6 | BR-015 |
| settlement_batches | status | 배치 상태 | enum | N | N | N | 생성/검증/승인/지급완료 | DATABASE §3.6 | BR-017 |
| settlement_batches | processed_at | 처리 시각 | timestamp | N | N | 미정 | 미정 | DATABASE §3.6 | BR-017 |
| settlement_items | batch_id | 정산 배치 참조 | uuid | 미정 | settlement_batches.id | N | 미정 | DATABASE §3.6 | BR-018 |
| settlement_items | member_id | 회원 참조 | uuid | 미정 | members.id | N | 미정 | DATABASE §3.6 | BR-018 |
| settlement_items | commission_records_sum | 수당 합계 | decimal | N | commission_records | N | 미정 | DATABASE §3.6 | BR-018 |
| settlement_items | tax_amount | 세금 차감액 | decimal | N | tax_withholdings | 미정 | 미정 | DATABASE §3.6 | BR-016 |
| settlement_items | net_amount | 실지급액 | decimal | N | N | N | 미정 | DATABASE §3.6 | BR-017 |
| settlement_items | payout_status | 지급 상태 | enum | N | N | 미정 | 미정 | DATABASE §3.6 | BR-017 |
| tax_withholdings | settlement_item_id | 정산 항목 참조 | uuid | 미정 | settlement_items.id | N | 미정 | DATABASE §3.7 | BR-016 |
| tax_withholdings | amount | 원천징수 금액 | decimal | N | N | N | 3.3% 기준 후보 | DATABASE §3.7 | BR-016 |
| compliance_thresholds | country_code | 대상 국가 | string | N | countries.country_code | N | KR 등 | DATABASE §3.29 | BR-020 |
| compliance_thresholds | caution_threshold | 주의 임계치 | decimal | N | N | N | KR 30% | DATABASE §3.29 | BR-020 |
| compliance_thresholds | warning_threshold | 경고 임계치 | decimal | N | N | N | KR 33% | DATABASE §3.29 | BR-020 |
| compliance_thresholds | block_threshold | 차단 임계치 | decimal | N | N | N | KR 35% | DATABASE §3.29 | BR-020 |
| compliance_ratio_snapshots | period_type | 계산 단위 | enum | N | N | N | REALTIME/MONTHLY/ANNUAL_CUMULATIVE | DATABASE §3.29 | BR-019 |
| compliance_ratio_snapshots | ratio | 후원수당/매출 비율 | decimal | N | N | N | 미정 | DATABASE §3.29 | BR-019 |
| compliance_ratio_snapshots | status | 임계 단계 | enum | N | N | N | SAFE/CAUTION/WARNING/BLOCKED | DATABASE §3.29 | BR-020/BR-021 |

## 5. Country / Rules / Tenant

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| countries | country_code | 국가 코드 | string | Y | N | N | KR/TH/JP/US/CN | DATABASE §3.13 | BR-004 |
| countries | status | 국가 상태 | enum | N | N | N | ACTIVE/PLANNED/RESERVED | DATABASE §3.13 | BR-004 |
| countries | currency | 통화 | string | N | N | 미정 | 미정 | DATABASE §3.13 | BR-004 |
| countries | default_language | 기본 언어 | string | N | N | 미정 | 미정 | DATABASE §3.13 | BR-004 |
| marketing_plan_versions | country_code | 적용 국가 | string | N | countries.country_code | N | 미정 | DATABASE §3.13 | BR-014 |
| marketing_plan_versions | status | 버전 상태 | enum | N | N | N | DRAFT/ACTIVE/EXPIRED | DATABASE §3.13 | BR-014 |
| marketing_plan_versions | plan_definition | 보상플랜 파라미터 | json | N | N | N | D-032/D-033 구조 | DATABASE §3.13 | BR-014 |
| tenants | id | 테넌트 식별자 | uuid | Y | N | N | 활성화 보류 | DATABASE §3.31 | BR-042 |
| tenants | status | 테넌트 상태 | enum | N | N | 미정 | ACTIVE/SUSPENDED | DATABASE §3.31 | BR-042 |
| tenant_settings | company_name/logo/domain | 회사별 설정 | string/ref | N | tenants.id | 미정 | 활성화 보류 | DATABASE §3.31.1 | BR-042 |

## 6. Marketing / Point / Workflow / ERP Core

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| marketing_programs | id | 프로그램 식별자 | uuid | Y | N | N | 미정 | DATABASE §3.34 | BR-033 |
| marketing_programs | category | 프로그램 카테고리 | string | N | N | N | 자유값, 하드코딩 enum 아님 | DATABASE §3.34 | BR-033 |
| marketing_programs | links_to_compensation | MLM 보상 연결 여부 | boolean | N | N | N | 미정 | DATABASE §3.34 | BR-033 |
| marketing_programs | requires_application | 신청 필요 여부 | boolean | N | N | N | 미정 | DATABASE §3.34 | BR-034 |
| marketing_program_applications | status | 신청 상태 | enum | N | N | N | APPLIED/PENDING_APPROVAL/APPROVED/REJECTED/IN_PROGRESS/COMPLETED | DATABASE §3.34.1 | BR-034 |
| point_accounts | member_id | 회원 참조 | uuid | 미정 | members.id | N | 미정 | DATABASE §3.36 | BR-035 |
| point_accounts | available_balance | 사용 가능 잔액 | decimal | N | N | N | 파생/캐시 후보 | DATABASE §3.36 | BR-035 |
| point_accounts | pending_balance | 승인대기 잔액 | decimal | N | N | N | 미정 | DATABASE §3.36 | BR-035 |
| point_transactions | transaction_type | 거래 유형 | enum | N | N | N | EARN/DEDUCT/USE_REQUEST/USE_APPROVED/USE_REJECTED/CANCEL/RESTORE/EXPIRE | DATABASE §3.36 | BR-035 |
| point_transactions | source_type | 원천 구분 | enum/string | N | N | 미정 | LIFESTYLE_BONUS 등 | DATABASE §3.36 | BR-035 |
| point_transactions | counts_toward_compliance_limit | 35% 한도 포함 여부 | boolean | N | N | N | 미정 | DATABASE §3.36 | BR-019/BR-035 |
| workflow_definitions | module_scope | 적용 업무 범위 | string | N | N | N | 자유 텍스트 | DATABASE §3.37 | BR-036 |
| workflow_instances | subject_type / subject_id | 업무 객체 범용 참조 | string/uuid | N | 업무 테이블 | N | 미정 | DATABASE §3.37 | BR-036 |
| workflow_instances | status | Workflow 상태 | enum | N | N | N | IN_PROGRESS/APPROVED/REJECTED/CANCELLED | DATABASE §3.37 | BR-036 |
| workflow_step_actions | decision | 승인/반려 | enum | N | N | N | 승인/반려 | DATABASE §3.37 | BR-036 |
| external_api_connections | category | 외부 API 카테고리 | string | N | N | N | PG/3PL/SMS/EMAIL 등, 고정 enum 아님 | DATABASE §3.38 | BR-038 |
| external_api_connections | auth_key_ref | 인증키 참조 | string | N | Vault/KMS | N | 평문 저장 금지 | DATABASE §3.38 | BR-038 |
| external_api_connections | status | 연결 상태 | enum | N | N | N | ACTIVE/INACTIVE/TESTING | DATABASE §3.38 | BR-038 |
| scheduled_job_run_logs | status | Job 실행 로그 상태 | enum/string | N | N | 미정 | 미정 | DATABASE §3.40 | BR-039 |
| notification_logs | failure_reason / retry_count | 발송 실패/재시도 정보 | text/integer | N | N | Y | 미정 | DATABASE §3.41 | BR-039 |
| report_generation_logs | status | 보고서 생성 상태 | enum/string | N | N | 미정 | 미정 | DATABASE §3.44 | BR-039 |

## 7. Dictionary Gaps

| 영역 | 현재 상태 |
|---|---|
| 물리 DB 타입 | ORM/마이그레이션 도구 미확정(O-022)이므로 실제 타입은 확정하지 않음 |
| PK/FK 컬럼명 | ERD.md에서도 "(컬럼명 미확정)"으로 표시된 항목 존재 |
| Nullable/Default | 대부분 구현 단계에서 확정 필요 |
| 배송/반품 상태 enum | 기존 문서에 세부 상태머신 미정 |
| CMS 콘텐츠 상태 enum | DRAFT/IN_REVIEW/SCHEDULED/PUBLISHED 도입 여부 미확정 |

