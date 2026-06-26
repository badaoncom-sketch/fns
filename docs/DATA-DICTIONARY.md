# DATA-DICTIONARY.md — Data Dictionary

> 상태: v0.4 (D-074 — Dynamic Board Engine: `boards`/`board_categories`/`board_posts`/`board_post_comments`/`board_post_likes`(신규 5종)를 §9로 반영, Dictionary Gaps §10으로 재배치. **기존 CMS(`cms_pages`/FAQ/팝업/배너) 무변경.** D-072 — 쇼핑몰 UX·알림·운영자 대시보드 완성: `carts`/`cart_items`/`product_price_alerts`(신규 3종) + `shipments`/`notification_templates` 컬럼 명료화를 §8로 반영) · 최종 수정일: 2026-06-26 · 단계: 설계(Design)
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

## 7. 쇼핑몰 운영 Phase 2 (Shop Operations) / SEO

> D-069(DATABASE.md §3.52~§3.56)에서 신설/확장되었으나 본 사전에 반영되지 않았던 항목을 D-070에서 동기화한다. 신규 Business Rule/Open Decision은 추가하지 않으며, 기존 BR-045~BR-054/O-176~O-194만 인용한다.

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| products | publish_start_at / publish_end_at | 예약 공개/종료 — `packages.sales_start_at/sales_end_at`와 동일 패턴 | timestamp | N | N | Y | 미정 | DATABASE §3.52 | - |
| products | sales_status | 판매상태 | enum | N | N | N | DRAFT/ON_SALE/SOLD_OUT/SUSPENDED/ENDED | DATABASE §3.52 | BR-045 |
| products | display_order | 진열순서(수동 정렬값) | integer | N | N | 미정 | 미정 | DATABASE §3.52 | - |
| products | search_boost_score | 검색 노출순위 가중치 | decimal | N | N | 미정 | 미정 | DATABASE §3.52 | - |
| product_option_combinations | sku_code | 옵션조합 SKU — 옵션-재고 연결모델 최종 확정 미확정 | string | N | N | 미정 | 연결모델 미확정(O-176) | DATABASE §3.52 | - |
| product_option_combinations | barcode | 옵션별 바코드 | string | N | N | Y | 미정 | DATABASE §3.52 | - |
| product_option_combinations | price_delta 또는 price_override | 옵션별 가격(차액형/절대값형 중 택1) | decimal | N | N | 조건부 | 하나만 설정 | DATABASE §3.52 | - |
| product_option_combinations | discount_price / discount_start_at / discount_end_at | 옵션별 할인 — 상품 차원 타임세일과 정책 통합 필요 여부 미확정(O-185) | decimal/timestamp | N | N | Y | 미정 | DATABASE §3.52 | - |
| product_option_combinations | is_active | 옵션별 판매중지 | boolean | N | N | N | 미정 | DATABASE §3.52 | BR-046 |
| product_option_combinations | shipping_fee_override | 옵션별 배송비 예외 — `shipping_fee_policies` 기본 정책에 대한 예외 | decimal | N | N | Y | 미정 | DATABASE §3.52 | - |
| product_images | product_option_combination_id | 옵션별 이미지 연결(신규) | uuid | N | product_option_combinations.id | Y | 미정 | DATABASE §3.52 | - |
| inventory_ledger | type(변동유형) 신규 값 | 입고예정/재고조정/창고간 이동 | enum | N | N | N | INBOUND_EXPECTED/ADJUSTMENT/TRANSFER_OUT/TRANSFER_IN | DATABASE §3.53 | BR-047 |
| inventory_lots | warehouse_id | 대상 창고 | uuid | 미정 | warehouses.id | N | 미정 | DATABASE §3.53 | - |
| inventory_lots | product_option_combination_id 또는 sku_code | 대상 SKU(O-176 해소 후 확정) | uuid/string | 미정 | product_option_combinations.id | N | 연결모델 미확정(O-176) | DATABASE §3.53 | - |
| inventory_lots | lot_number | 로트 번호 | string | N | N | N | 미정 | DATABASE §3.53 | BR-048 |
| inventory_lots | manufactured_at / expires_at | 제조일/유통기한 | timestamp | N | N | 미정 | LOT 도입 자체 미확정(O-177) | DATABASE §3.53 | BR-048 |
| inventory_lots | quantity | 로트 단위 수량 | integer | N | N | N | 미정 | DATABASE §3.53 | - |
| order_admin_notes | order_id | 대상 주문 | uuid | 미정 | orders.id | N | 미정 | DATABASE §3.53 | - |
| order_admin_notes | related_ticket_id | CS Center 티켓 참조(중복 메모 방지) | uuid | N | tickets.id | Y | 미정 | DATABASE §3.53 | - |
| shipment_change_logs | shipment_id | 대상 배송 | uuid | 미정 | shipments.id | N | 미정 | DATABASE §3.53 | - |
| order_merge_logs / order_split_logs | order_id 관련 참조 | 주문 병합/분리 로그 — 매출 합계 일치 규칙, 허용 범위 미확정(O-179) | uuid | 미정 | orders.id | N | 미정 | DATABASE §3.53 | BR-049 |
| exchange_requests / exchange_items | order_id 관련 참조 | 부분교환 — 반품(returns)과 통합 여부 미확정(O-180) | uuid | 미정 | orders.id / returns.id | N | 미정 | DATABASE §3.53 | BR-050 |
| order_payment_attempts | order_id / attempt_no / pg_code / status / failure_reason | 일반 주문 PG 실패/재결제 시도 로그 | uuid/integer/string/enum/text | 미정 | orders.id | N | 미정 | DATABASE §3.53 | - |
| virtual_account_issuances / bank_transfer_payments | (도입 시 정의) | 가상계좌 발급/무통장입금 매칭 — 도입 여부 자체 미확정 | 미정 | 미정 | 미정 | 도입 미확정(O-181) | DATABASE §3.53 | - |
| order_payment_splits | order_id 관련 참조 | 복합결제(포인트+카드 등) 분할 내역 — 부분실패 처리 정책 미확정(O-182) | uuid | 미정 | orders.id | N | 미정 | DATABASE §3.53 | - |
| point_transactions | transaction_type 신규 값 | 포인트의 결제수단화 | enum | N | N | N | PAYMENT_USE | DATABASE §3.53 | - |
| shipments | bundle_group_id | 묶음배송 그룹 | uuid | N | N | Y | 미정 | DATABASE §3.53 | - |
| shipments | scheduled_dispatch_at | 예약배송 시각 | timestamp | N | N | Y | 미정 | DATABASE §3.53 | - |
| shipments | hold_reason / hold_at | 출고보류 사유/시각 | text/timestamp | N | N | Y | 미정 | DATABASE §3.53 | - |
| shipping_fee_settlements | (배송비 정산 실행/내역) | `shipping_fee_policies`는 요율 정책만 정의, 실행 기록 신설 | 미정 | 미정 | 미정 | 도입 미확정(O-184) | DATABASE §3.53 | BR-051 |
| product_reviews | video_url | 리뷰 동영상 | string | N | N | Y | 미정 | DATABASE §3.54 | - |
| product_reviews | admin_reply / admin_replied_at / admin_replied_by | 관리자 답변 — 컬럼형 vs 스레드형 미확정 | text/timestamp/uuid | N | N | Y | 스레드 확장 여부 미확정(O-186) | DATABASE §3.54 | - |
| product_reviews | is_best / best_selected_at / best_selected_by | 베스트 리뷰 — 선정기준 미확정 | boolean/timestamp/uuid | N | N | Y | 선정기준 미확정(O-187) | DATABASE §3.54 | - |
| product_comparisons | member_id | 비교 등록 회원 — 비회원 처리(서버/클라이언트) 미확정 | uuid | 미정 | members.id | Y | 비회원 처리 미확정(O-188) | DATABASE §3.54 | - |
| product_comparisons | product_id / added_at | 비교 대상 상품/등록 시각 | uuid/timestamp | 미정 | products.id | N | 미정 | DATABASE §3.54 | - |
| product_bundles / product_bundle_items | (번들 구성) | One+One/Bundle 묶음판매 — MLM `packages`와 명확히 별개 | 미정 | products.id | N | 쿠폰 중복적용 미확정(O-191) | DATABASE §3.54 | - |
| product_seo | product_id | PK 겸 FK, products 1:1 | uuid | Y | products.id | N | 미정 | DATABASE §3.55 | - |
| product_seo | seo_title / seo_description / seo_keywords | 미입력 시 자동생성 | string | N | N | Y | 자동생성 우선순위 BR-053 | DATABASE §3.55 | BR-053 |
| product_seo | slug / canonical_url | 상품 고유 URL slug / 중복 URL 정규화 | string | N | N | Y | 미정 | DATABASE §3.55 | - |
| product_seo | meta_robots | 검색엔진 노출 제어 | enum | N | N | Y | INDEX_FOLLOW/NOINDEX_FOLLOW 등 | DATABASE §3.55 | - |
| product_seo | og_title / og_description / og_image_ref | 미입력 시 `products.thumbnail_image_ref` 자동 매핑 | string/ref | N | N | Y | BR-053 자동매핑 | DATABASE §3.55 | BR-053 |
| product_seo | twitter_title / twitter_description / twitter_image_ref | OG와 별도 필드셋 | string/ref | N | N | Y | 미정 | DATABASE §3.55 | - |
| product_seo | schema_brand_override / schema_enabled | Schema.org Product 구조화데이터 — 가격/재고/리뷰평점은 저장하지 않고 조회 시 실시간 조합 | boolean/string | N | N | Y | 실시간조합 vs 캐시 미확정(O-194) | DATABASE §3.55 | - |
| tenant_settings | site_title | Site 단위 SEO 기본값 | string | N | tenants.id | Y | 미정 | DATABASE §3.55 | - |
| tenant_settings | default_og_image_ref | SEO 미입력 시 최종 fallback | ref | N | tenants.id | Y | 미정 | DATABASE §3.55 | - |
| tenant_settings | apple_touch_icon_ref | favicon(favicon_ref)과 별개 규격 | ref | N | tenants.id | Y | 미정 | DATABASE §3.55 | - |
| tenant_settings | canonical_base_url | 멀티 도메인 운영 시 정규 도메인 | string | N | tenants.id | Y | 미정 | DATABASE §3.55 | - |
| tenant_settings | default_robots_txt | 사이트 전역 robots.txt | text | N | tenants.id | Y | 미정 | DATABASE §3.55 | - |
| tenant_share_images | tenant_id | 소속 테넌트 | uuid | 미정 | tenants.id | N | 미정 | DATABASE §3.55 | - |
| tenant_share_images | share_type | 공유 이미지 용도 분류 | enum/string | N | N | N | KAKAO_DEFAULT/MAIN_URL_DEFAULT/PRODUCT_DEFAULT/EVENT_DEFAULT/PROMOTION_DEFAULT/BRAND_DEFAULT, 자유 확장 가능 | DATABASE §3.55 | BR-054 |
| content_seo_metadata | related_entity_type / related_entity_id | 범용 polymorphic 참조 — File Manager(§3.39) 패턴 재사용 | string/uuid | N | 대상 콘텐츠 테이블 | N | 미정 | DATABASE §3.55 | - |
| content_seo_metadata | seo_title / seo_description / og_image_ref / slug / canonical_url / meta_robots | product_seo와 동일 필드셋(상품 특화 Schema.org 필드 제외) | string/ref/enum | N | N | Y | 미정 | DATABASE §3.55 | - |

## 8. 쇼핑몰 UX/알림/운영자 대시보드 완성 (D-072)

> 신규 테이블은 `carts`/`cart_items`/`product_price_alerts` 3종뿐이다 — 나머지(최근본상품/관심상품/상품비교/재입고알림/알림템플릿/반품/교환)는 기존 테이블을 그대로 재사용하므로 본 절에 재등록하지 않는다. [DATABASE.md](DATABASE.md) §3.57 참조.

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| carts | id / member_id | 회원 장바구니(1:1) | uuid | Y | members.id | 미정(비회원 O-099/O-188 패턴) | 미정 | DATABASE §3.57 | - |
| carts | updated_at | 마지막 변경 시각 | timestamp | N | N | N | 미정 | DATABASE §3.57 | - |
| cart_items | cart_id | 소속 장바구니 | uuid | 미정 | carts.id | N | 미정 | DATABASE §3.57 | - |
| cart_items | product_id / product_option_combination_id | 상품/옵션 참조 | uuid | N | products.id / product_option_combinations.id | 조건부 | 미정 | DATABASE §3.57 | - |
| cart_items | quantity | 수량 | integer | N | N | N | 미정 | DATABASE §3.57 | - |
| cart_items | added_at | 담은 시각 — "나중에 구매하기" 처리방식 미확정 | timestamp | N | N | N | 미확정(O-197) | DATABASE §3.57 | - |
| product_price_alerts | member_id / product_id | 가격인하 알림 신청 대상 | uuid | 미정 | members.id / products.id | N | 미정 | DATABASE §3.57 | - |
| product_price_alerts | target_price 또는 알림 기준 | 트리거 조건 — 절대가 vs 할인율 미확정 | decimal | N | N | N | 미확정(O-198) | DATABASE §3.57 | - |
| product_price_alerts | notified_at | 알림 발송 시각 | timestamp | N | N | Y | 미정 | DATABASE §3.57 | - |
| shipments | courier_name / tracking_no | 배송 추적 — 기존 테이블 컬럼 명료화(신규 테이블 아님) | string | N | N | Y | 미정 | DATABASE §3.10/§3.57 | - |
| shipments | status | 준비중/출고완료/배송중/배송완료/지연/보류 | enum | N | N | N | [STATE-MACHINE.md](STATE-MACHINE.md) §17 | DATABASE §3.10/§3.57 | - |
| notification_templates | subject_template | 제목(EMAIL 채널용) | string | N | N | Y | 미정 | DATABASE §3.57 | - |
| notification_templates | is_active | 활성/비활성 토글 | boolean | N | N | N | 미정 | DATABASE §3.57 | - |

## 9. Dynamic Board Engine (D-074)

> 신규 테이블 5종(`boards`/`board_categories`/`board_posts`/`board_post_comments`/`board_post_likes`)만 등록한다 — 첨부/이미지/SEO/다국어/조회수는 기존 File Manager `files`/`content_seo_metadata`/`cms_translations`/`content_view_events`를 재사용하므로 본 절에 재등록하지 않는다. **기존 CMS(`cms_pages`/`faq_categories`/`faq_items`/`popups`)는 본 라운드에서 변경하지 않아 §3.33 그대로다.** [DATABASE.md](DATABASE.md) §3.58 참조.

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| boards | id / name / code | 게시판 식별자/명/코드(unique) | uuid/string | Y(id) | N | N | 미정 | DATABASE §3.58 | - |
| boards | board_type | 게시판 유형 | string | N | N | N | 자유 확장값(`marketing_programs.category`와 동일 패턴), 고정 enum 아님 | DATABASE §3.58 | - |
| boards | layout_type | 목록 화면 레이아웃 | enum | N | N | N | LIST/CARD/GALLERY/FAQ/VIDEO | DATABASE §3.58 | - |
| boards | menu_exposure / shop_exposure / shop_member_exposure / my_office_exposure / main_exposure | 노출 위치 토글 | boolean | N | N | N | 미정 | DATABASE §3.58 | - |
| boards | menu_group / sort_order | 메뉴 연결/순서 | string/integer | N | N | Y/N | 참조 무결성 없는 자유 텍스트 | DATABASE §3.58 | - |
| boards | country_codes / language_codes | 사용 국가/언어 | array | N | N | Y | nullable=전체(기존 패턴) | DATABASE §3.58 | - |
| boards | feature_flags | 댓글/답글/파일첨부/이미지/대표이미지/동영상/다운로드/조회수/좋아요/공유/SEO/OG/예약게시/승인후게시/카테고리/태그/검색/RSS ON/OFF | json | N | N | N | 미정 | DATABASE §3.58 | - |
| board_categories | board_id / name / sort_order | 게시판별 카테고리 | uuid/string/integer | 미정 | boards.id | N | `feature_flags.카테고리=true`일 때만 사용 | DATABASE §3.58 | - |
| board_posts | board_id / category_id | 소속 게시판/카테고리 | uuid | 미정 | boards.id / board_categories.id | 조건부 | 미정 | DATABASE §3.58 | - |
| board_posts | title / content / thumbnail_image_ref / slug | 제목/본문/대표이미지/URL경로 | string/text/ref/string | N | N | 조건부 | 미정 | DATABASE §3.58 | - |
| board_posts | status | 게시 상태 | enum | N | N | N | DRAFT/SCHEDULED/PENDING_APPROVAL/PUBLISHED/UNPUBLISHED | DATABASE §3.58 | - |
| board_posts | scheduled_publish_at | 예약게시 시각 | timestamp | N | N | Y | `feature_flags.예약게시=true`일 때만 사용 | DATABASE §3.58 | - |
| board_posts | is_public / tags | 공개여부/태그 | boolean/json | N | N | N/Y | tags는 배열, 전용 마스터 테이블 미도입 | DATABASE §3.58 | - |
| board_posts | metadata | 유형별 특수 필드 확장 포인트 | json | N | N | Y | 예: VIDEO의 video_url, FAQ의 answer | DATABASE §3.58 | - |
| board_posts | view_count_cache / like_count_cache | 조회수/좋아요 캐시(파생) | integer | N | N | N | 원본은 `content_view_events`/`board_post_likes` | DATABASE §3.58 | - |
| board_post_comments | post_id / parent_comment_id | 소속 게시글/상위 댓글 | uuid | 미정 | board_posts.id / board_post_comments.id | 조건부(parent는 null=최상위, 값있음=답글) | `feature_flags.댓글=true`일 때만 사용, 답글 전용 테이블 없음 | DATABASE §3.58 | - |
| board_post_comments | member_id / content / is_active | 작성회원/내용/소프트삭제 | uuid/text/boolean | 미정 | members.id | N | 미정 | DATABASE §3.58 | - |
| board_post_likes | post_id / member_id | 복합 unique(회원당 1회) | uuid | Y(복합) | board_posts.id / members.id | N | `feature_flags.좋아요=true`일 때만 사용 | DATABASE §3.58 | - |

## 10. Dictionary Gaps

| 영역 | 현재 상태 |
|---|---|
| 물리 DB 타입 | ORM/마이그레이션 도구 미확정(O-022)이므로 실제 타입은 확정하지 않음 |
| PK/FK 컬럼명 | ERD.md에서도 "(컬럼명 미확정)"으로 표시된 항목 존재 |
| Nullable/Default | 대부분 구현 단계에서 확정 필요 |
| 배송/반품 상태 enum | [STATE-MACHINE.md](STATE-MACHINE.md) §16/§17에 권장안 추가(D-069/D-072) — 최종 확정은 아님 |
| CMS 콘텐츠 상태 enum | DRAFT/IN_REVIEW/SCHEDULED/PUBLISHED 도입 여부 미확정 |
| 옵션↔SKU↔재고 연결모델 | `product_option_combinations.sku_code`와 `inventory_items`/`inventory_ledger`/`inventory_lots`의 최종 연결모델 미확정(O-176) — §3.52/§3.53 보강 전체의 전제조건 |
| LOT(로트) 관리 도입 여부 | `inventory_lots` 신설 자체 및 적용 범위(유통기한 카테고리 한정 vs 전체) 미확정(O-177) |
| SEO 테이블 구조 | `product_seo`/`content_seo_metadata`의 신규 테이블 분리 vs 각 콘텐츠 테이블 컬럼 직접 추가 최종 확정 미확정(O-193) |
| 장바구니 "나중에 구매하기" 처리 | `cart_items` 보존 vs `product_wishlists` 이동 미확정(O-197) |
| 가격인하 알림 트리거 기준 | 절대가 vs 할인율 미확정(O-198) |
| 관리자 저장된 검색조건/즐겨찾기 메뉴 | 테이블 도입 여부 미확정(O-199) |
| 기존 CMS ↔ Board Engine 통합 | `cms_pages`/FAQ/팝업/배너를 Board Engine으로 마이그레이션할지 여부 미확정(O-200) — 결정 전까지 두 구조가 병렬로 유지된다 |
| 게시판/게시글 삭제 시 하위 데이터 처리 | 게시판 삭제 시 `board_posts`/`board_categories`/`board_post_comments`/`board_post_likes` cascade vs 소프트삭제 정책 미확정 — 기존 소프트삭제 원칙(§4 설계원칙 3)과의 정합 필요 |

