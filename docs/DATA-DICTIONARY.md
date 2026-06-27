# DATA-DICTIONARY.md — Data Dictionary

> 상태: v0.7 (D-078 — Marketing Reward System 및 Lifestyle Wallet 구조 개선: `reward_policies`(신규 1종) + `member_wallets.wallet_type`/`wallet_transactions.counts_toward_compliance_limit`(신규 컬럼) + `transaction_type` 허용값 추가(RESTORE/EXPIRE)를 §13으로 반영, Dictionary Gaps §13→§14로 재배치. **새 MLM 정책 아님** — Lifestyle 보상의 저장 위치만 point_transactions(D-041)에서 wallet_transactions로 재라우팅. O-208/O-209 등록. D-076 — ERP 운영 생산성 및 관리자 UX 완성: `admin_favorite_menus`/`saved_filters`/`notification_inbox_states`/`admin_notes`/`approval_delegations`(신규 5종)를 §12로 반영, Dictionary Gaps §12→§13으로 재배치. **O-199 해소**(즐겨찾기 메뉴/저장된 검색조건). 신규 Business Rule·MLM·Settlement·ERP Core·Workflow 구조 변경 없음. D-075 — 한국 공제조합 연동·E-Wallet·글로벌 결제: `compliance_member_registrations`/`compliance_transmission_items`/`member_wallets`/`wallet_transactions`/`wallet_withdrawal_requests`/`payment_webhook_events`(신규 6종) + `compliance_report_definitions`/`external_api_connections` 컬럼 추가를 §11로 반영. **MLM 보상플랜·정산 계산 로직·기존 Business Rule·ERP Core 구조 무변경.** D-074 — Dynamic Board Engine: `boards`/`board_categories`/`board_posts`/`board_post_comments`/`board_post_likes`(신규 5종)를 §9로 반영. **기존 CMS(`cms_pages`/FAQ/팝업/배너) 무변경.** D-072 — 쇼핑몰 UX·알림·운영자 대시보드 완성: `carts`/`cart_items`/`product_price_alerts`(신규 3종) + `shipments`/`notification_templates` 컬럼 명료화를 §8로 반영) · 최종 수정일: 2026-06-27 · 단계: 설계(Design)
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

## 11. 한국 공제조합 연동·E-Wallet·글로벌 결제 (D-075)

> 전부 **Tenant별 선택 기능**이며, 신규 연동/정산 엔진을 만들지 않고 기존 API Center(§3.38)/Scheduler Center(§3.40)/Audit Center(§3.42)/Report Builder(§3.44)/Workflow Engine(§3.37)/Payment(§3.53) 구조를 재사용한다. 신규 테이블은 `compliance_member_registrations`/`compliance_transmission_items`/`member_wallets`/`wallet_transactions`/`wallet_withdrawal_requests`/`payment_webhook_events` 6종뿐이다. **MLM 보상플랜·정산 계산 로직·기존 Business Rule·ERP Core 구조는 전혀 변경하지 않았다.** [DATABASE.md](DATABASE.md) §3.59~§3.61 참조.

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| compliance_member_registrations | member_id / connection_id | 대상 회원 / 어느 공제조합 연동인지 | uuid | 미정 | members.id / external_api_connections.id | N | 미정 | DATABASE §3.59 | - |
| compliance_member_registrations | registration_number | 공제번호 | string | N | N | Y | 미정 | DATABASE §3.59 | - |
| compliance_member_registrations | certificate_file_ref | 공제증서 — File Manager(`files`, §3.39) 참조, 신규 파일 테이블 없음 | ref | N | files.id(논리참조) | Y | 미정 | DATABASE §3.59 | - |
| compliance_member_registrations | status | 등록 상태 | enum | N | N | N | 등록대기/등록완료/등록실패/해지 | DATABASE §3.59 | - |
| compliance_member_registrations | registered_at | 등록 완료 시각 | timestamp | N | N | Y | 미정 | DATABASE §3.59 | - |
| compliance_transmission_items | connection_id | 어느 공제조합 연동인지 | uuid | N | external_api_connections.id | N | 미정 | DATABASE §3.59 | - |
| compliance_transmission_items | item_type | 전송 항목 유형 | string | N | N | N | MEMBER/SPONSOR_RELATION/REVENUE/COMMISSION/REFUND/RETURN/CANCEL — 자유 확장값(`marketing_programs.category`와 동일 패턴), 고정 enum 아님 | DATABASE §3.59 | - |
| compliance_transmission_items | source_type / source_id | 범용 다형 참조(예: `members`/`orders`/`commission_records`/`returns`) — File Manager와 동일한 패턴 | string/uuid | N | 다형(미정) | N | 미정 | DATABASE §3.59 | - |
| compliance_transmission_items | period | 대상 기간(해당하는 경우) | date range | N | N | Y | 미정 | DATABASE §3.59 | - |
| compliance_transmission_items | status | 전송 상태 | enum | N | N | N | 대기/전송중/성공/실패 | DATABASE §3.59 | - |
| compliance_transmission_items | failure_reason / retry_count / last_attempted_at | 실패 사유 / 재전송 횟수 / 마지막 시도 시각 | text/integer/timestamp | N | N | Y/N/Y | 미정 | DATABASE §3.59 | - |
| compliance_transmission_items | call_log_id | `external_api_call_logs`(§3.38) 참조 — 실제 HTTP 호출 1건과 연결(재시도마다 새 호출 로그, 1:N) | uuid | N | external_api_call_logs.id | Y | 미정 | DATABASE §3.59 | - |
| compliance_report_definitions | tenant_id | 테넌트별 보고서 정의 차등(§3.16 기존 테이블에 컬럼 추가) | uuid | N | tenants.id | Y | 미정 | DATABASE §3.59 | - |
| compliance_report_definitions | auto_transmit / manual_transmit_allowed | 자동 전송 여부 / 수동 전송 허용 여부 | boolean | N | N | N | 미정 | DATABASE §3.59 | - |
| member_wallets | id / member_id | 회원별 × 통화별 지갑(1:N) | uuid | Y(id) | members.id | N | 미정 | DATABASE §3.60 | - |
| member_wallets | currency_code | 지갑 통화 | string | N | N | N | KRW/USD/THB/JPY 등 — `marketing_programs.category`와 동일한 자유 확장값, 고정 enum 아님 | DATABASE §3.60 | - |
| member_wallets | status | 지갑 상태(관리자 잠금/해제) | enum | N | N | N | ACTIVE/LOCKED | DATABASE §3.60 | - |
| member_wallets | available_balance_cache / pending_balance_cache / withdrawable_balance_cache / used_balance_cache / hold_balance_cache | **파생 캐시** — 원본은 항상 `wallet_transactions` 집계, 직접 UPDATE 금지(`board_posts.view_count_cache`와 동일 원칙) | decimal | N | N | N | 미정, Source of Truth = wallet_transactions | DATABASE §3.60 | - |
| member_wallets | created_at | | timestamp | N | N | N | 미정 | DATABASE §3.60 | - |
| wallet_transactions | id / wallet_id | append-only 원장 — 잔액 직접 수정 금지, 모든 변경은 새 행 추가로만 표현 | uuid | Y(id) | member_wallets.id | N | 미정 | DATABASE §3.60 | - |
| wallet_transactions | transaction_type | 거래 유형 | enum | N | N | N | CHARGE/EARN/USE/CANCEL/REFUND/WITHDRAWAL_REQUEST/WITHDRAWAL_COMPLETED/ADJUSTMENT/HOLD/RELEASE | DATABASE §3.60 | - |
| wallet_transactions | amount | 부호로 증감 표현(양수=증가, 음수=감소) | decimal | N | N | N | 미정 | DATABASE §3.60 | - |
| wallet_transactions | balance_type_affected | 영향받는 잔액 종류 | enum | N | N | N | available/pending/withdrawable/used/hold | DATABASE §3.60 | - |
| wallet_transactions | source_type / source_id | 범용 다형 참조 — EARN이면 `commission_records`/`settlement_items` 참조(정산 결과 인용, 정산 계산 로직 자체는 불변경), USE면 `orders` 참조 등 | string/uuid | N | 다형(미정) | Y | 미정 | DATABASE §3.60 | - |
| wallet_transactions | reason | ADJUSTMENT/HOLD/RELEASE일 때 사유 기록 | text | N | N | 조건부(해당 유형일 때 필수) | 미정 | DATABASE §3.60 | - |
| wallet_transactions | created_by / created_at | 시스템(배치) 또는 관리자(수동 보정) | uuid/timestamp | N | N | N | 미정 | DATABASE §3.60 | - |
| wallet_withdrawal_requests | id / wallet_id / member_id | 출금 신청(신규 승인 구조 아님, Workflow Engine 재사용) | uuid | Y(id) | member_wallets.id / members.id | N | 미정 | DATABASE §3.60 | - |
| wallet_withdrawal_requests | amount | 신청 금액 | decimal | N | N | N | 미정 | DATABASE §3.60 | - |
| wallet_withdrawal_requests | bank_account_ref | 출금 계좌 — 기존 회원 계좌 정보 참조 또는 1회성 입력 | string | N | N | Y | 정확한 방식 미확정(O-201 — 은행송금/지갑적립 분배 정책과 연계) | DATABASE §3.60 | - |
| wallet_withdrawal_requests | status | 출금 상태 | enum | N | N | N | REQUESTED/APPROVED/REJECTED/COMPLETED | DATABASE §3.60 | - |
| wallet_withdrawal_requests | workflow_instance_id | `workflow_instances`(§3.37, `subject_type='WALLET_WITHDRAWAL'`) 참조 — 신규 승인 구조를 만들지 않고 Workflow Engine 재사용 | uuid | N | workflow_instances.id | N | 미정 | DATABASE §3.60 | - |
| wallet_withdrawal_requests | requested_at / processed_at / processed_by | | timestamp/timestamp/uuid | N | N | N/Y/Y | 미정 | DATABASE §3.60 | - |
| external_api_connections | country_code | PG 연동을 국가별로 스코프(예: 태국 PromptPay는 `country_code='TH'`, §3.38 기존 테이블에 컬럼 추가, 기존 `tenant_id`와 결합) | string | N | N | Y | 미정 | DATABASE §3.61 | - |
| payment_webhook_events | connection_id | 어느 PG 연동인지 | uuid | N | external_api_connections.id | N | 미정 | DATABASE §3.61 | - |
| payment_webhook_events | event_type | PG사별 원본 이벤트명을 정규화 | string | N | N | N | 결제승인/결제실패/취소/부분취소/환불/부분환불/정기결제성공/정기결제실패 — 자유 확장값, 매핑 규칙 미확정 | DATABASE §3.61 | - |
| payment_webhook_events | raw_payload_ref | 원문 페이로드 참조 | ref | N | N | Y | 저장 메커니즘 미확정(O-205 — Webhook 서명검증과 연계) | DATABASE §3.61 | - |
| payment_webhook_events | signature_verified | Webhook 서명 검증 통과 여부 | boolean | N | N | N | 검증 방식/실패 시 처리 미확정(O-205) | DATABASE §3.61 | - |
| payment_webhook_events | related_order_id | 매칭된 주문(매칭 실패 시 null, 수동 매칭 필요) | uuid | N | orders.id | Y | 미정 | DATABASE §3.61 | - |
| payment_webhook_events | status | 처리 상태 | enum | N | N | N | 수신/처리중/처리완료/처리실패 | DATABASE §3.61 | - |
| payment_webhook_events | received_at / processed_at | | timestamp | N | N | N/Y | 미정 | DATABASE §3.61 | - |

## 12. ERP 운영 생산성 및 관리자 UX 완성 (D-076)

> 신규 테이블은 5종(`admin_favorite_menus`/`saved_filters`/`notification_inbox_states`/`admin_notes`/`approval_delegations`)뿐이다 — Global Search/Approval Center/Recent Activity/Tenant Usage Dashboard/Personal Workspace/Command Palette/Universal Clipboard/관리자 Dashboard/System Health Widget/Quick Action 보강 등 나머지는 모두 기존 테이블 federated 조회이거나 DB 영향이 없는 프론트엔드 컴포넌트이므로 본 절에 재등록하지 않는다. **`admin_favorite_menus`/`saved_filters`가 O-199(관리자 저장된 검색조건/즐겨찾기 메뉴 테이블 도입 여부)를 해소한다 — 즐겨찾기 메뉴는 `admin_favorite_menus`, 저장된 검색조건은 `saved_filters`.** [DATABASE.md](DATABASE.md) §3.62 참조.

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| admin_favorite_menus | admin_id | 관리자 — O-199(해소) 전반부: 즐겨찾기 메뉴 | uuid | 미정 | admins/members | N | 미정 | DATABASE §3.62 | - |
| admin_favorite_menus | menu_key | 메뉴 식별자(SITEMAP.md 메뉴 트리의 노드 키) | string | N | N | N | 미정 | DATABASE §3.62 | - |
| admin_favorite_menus | sort_order | 즐겨찾기 내 순서 | integer | N | N | N | 미정 | DATABASE §3.62 | - |
| admin_favorite_menus | pinned_at | | timestamp | N | N | 미정 | 미정 | DATABASE §3.62 | - |
| saved_filters | admin_id | 작성자 — O-199(해소) 후반부: 저장된 검색조건 | uuid | 미정 | admins/members | N | 미정 | DATABASE §3.62 | - |
| saved_filters | name | 필터명(예: "승인 대기"/"환불 대기"/"배송 지연"/"재고 부족"/"신규 가입"/"정산 보류") | string | N | N | N | 미정 | DATABASE §3.62 | - |
| saved_filters | target_module | 적용 대상 모듈 | string | N | N | N | 자유 확장값(`marketing_programs.category`와 동일 패턴) | DATABASE §3.62 | - |
| saved_filters | filter_criteria | 필터 조건 | json | N | N | N | 미정 | DATABASE §3.62 | - |
| saved_filters | is_default | 해당 모듈 진입 시 기본 적용 여부 | boolean | N | N | N | 미정 | DATABASE §3.62 | - |
| saved_filters | is_shared | 다른 관리자에게 공유 여부 | boolean | N | N | N | 미정 | DATABASE §3.62 | - |
| saved_filters | created_at | | timestamp | N | N | N | 미정 | DATABASE §3.62 | - |
| notification_inbox_states | notification_id | `notifications`(§3.20) 참조 — 상태 오버레이, 원본 테이블 무변경 | uuid | 미정 | notifications.id | N | 미정 | DATABASE §3.62 | - |
| notification_inbox_states | recipient_type / recipient_id | MEMBER/ADMIN 등 — `notifications.member_id`(회원 전용)와 별개로 관리자 수신 알림까지 다루는 범용 참조 | string/uuid | N | 다형(미정) | N | 미정 | DATABASE §3.62 | - |
| notification_inbox_states | is_read / is_important / is_archived | | boolean | N | N | N | 미정 | DATABASE §3.62 | - |
| notification_inbox_states | tags | 자유 태그 | json | N | N | Y | 배열 | DATABASE §3.62 | - |
| admin_notes | related_entity_type / related_entity_id | 범용 polymorphic 참조(MEMBER/ORDER/PRODUCT/BOARD_POST/WORKFLOW_INSTANCE/SETTLEMENT_BATCH 등) — File Manager(§3.39) 패턴 재사용. 기존 `order_admin_notes`(§3.53, 주문 전용)는 무변경, 통합 여부는 **O-206** | string/uuid | N | 다형(미정) | N | 미정 | DATABASE §3.62 | - |
| admin_notes | content | | text | N | N | N | 미정 | DATABASE §3.62 | - |
| admin_notes | is_important | 중요 메모 표시 | boolean | N | N | N | 미정 | DATABASE §3.62 | - |
| admin_notes | is_internal | 내부 전용(고객 노출 화면에 절대 노출 안 함) | boolean | N | N | N | 항상 true 전제이나 컬럼으로 명시 | DATABASE §3.62 | - |
| admin_notes | created_by / created_at | | uuid/timestamp | N | N | N | 미정 | DATABASE §3.62 | - |
| approval_delegations | delegator_id | 원래 승인자 — Workflow Engine(§3.37)이 승인자 결정 시 참조하는 위성 테이블, Workflow Engine 자체 구조는 무변경 | uuid | N | admins/members | N | 미정 | DATABASE §3.62 | - |
| approval_delegations | delegate_id | 대리 승인자 — 위임 가능 범위(동일 역할 내 한정 여부, 권한 레벨 검증 방식) 미확정, **O-207** | uuid | N | admins/members | N | 미정 | DATABASE §3.62 | - |
| approval_delegations | start_date / end_date | 위임 기간 | date | N | N | N | 미정 | DATABASE §3.62 | - |
| approval_delegations | reason | | text | N | N | 미정 | 미정 | DATABASE §3.62 | - |
| approval_delegations | created_by | | uuid | N | N | N | 미정 | DATABASE §3.62 | - |

## 13. Marketing Reward System 및 Lifestyle Wallet 구조 개선 (D-078)

> **새 MLM 정책이 아니다.** Lifestyle 보상("+알파" 보너스: 여행/자동차/자기계발)이 현금성 후원수당이 아니라 Marketing Reward Program이라는 점을 데이터 구조로 명확히 한다 — `Reward Policy → Lifestyle Point Ledger → Lifestyle Wallet` 구조를 표준화하고, **현금(Settlement/E-Wallet CASH)과 포인트(Lifestyle Point)는 절대 혼합하지 않는다.** 신규 테이블은 `reward_policies` 1종뿐이며, 나머지는 `member_wallets`/`wallet_transactions`(§3.60, 기존)를 확장 재사용한다. 기존 `lifestyle_bonus_accumulations`(§3.25)의 적립 산정 로직 자체는 변경 없음 — 산정 완료 후 저장 위치(라우팅)만 `point_transactions`(D-041)에서 `wallet_transactions`(wallet_type=LIFESTYLE_POINT)로 바뀐다. [DATABASE.md](DATABASE.md) §3.63 참조.

| Table | Column | 설명 | 자료형(개념) | PK | FK | Nullable | Enum/Default | Source of Truth | 관련 Rule |
|---|---|---|---|---|---|---|---|---|---|
| member_wallets | wallet_type(신규) | `CASH`(§3.60 기존 5개 통화 지갑, 출금 가능) / `LIFESTYLE_POINT`(신규, 본 절) — 자유 확장값. 기존 행은 `CASH`로 간주(개념상 backfill) | string | N | N | N | 자유 확장값(현재 CASH/LIFESTYLE_POINT 2종 활성화) | DATABASE §3.63 | - |
| wallet_transactions | counts_toward_compliance_limit(신규) | 이 거래가 MLM 35% 법적 한도 산정에 포함되는지 — `point_transactions.counts_toward_compliance_limit`(§3.36, D-041)와 동일한 패턴을 그대로 포팅. wallet_type=CASH는 기존 §3.60 동작과 동일, wallet_type=LIFESTYLE_POINT는 원천 `marketing_programs.links_to_compensation`(§3.34) 값을 따른다 | boolean | N | N | N | 미정 | DATABASE §3.63 | - |
| wallet_transactions | transaction_type(기존 컬럼, 허용값 추가) | 기존 CHARGE/EARN/USE/CANCEL/REFUND/WITHDRAWAL_REQUEST/WITHDRAWAL_COMPLETED/ADJUSTMENT/HOLD/RELEASE는 그대로 유지하고 **RESTORE(복원)**를 추가 — `point_transactions.transaction_type`(§3.36)의 RESTORE와 동일한 개념, 주문 취소 시 Lifestyle Point 사용분 복원을 표현 | string(enum-like) | N | N | N | 허용값 추가: RESTORE | DATABASE §3.63 | - |
| wallet_transactions | transaction_type(기존 컬럼, 허용값 추가) | **EXPIRE(만료)**를 추가 — `point_transactions.transaction_type`(§3.36)의 EXPIRE와 동일한 개념, Lifestyle Point의 사용 가능 기간(`reward_policies.usable_period`) 만료를 표현 | string(enum-like) | N | N | N | 허용값 추가: EXPIRE | DATABASE §3.63 | - |
| reward_policies | id | | uuid | Y | N | N | 미정 | DATABASE §3.63 | - |
| reward_policies | program_id | `marketing_programs`(§3.34) 참조 — Program명은 이 참조로 가져오며 중복 저장하지 않는다 | uuid | N | marketing_programs.id | N | 미정 | DATABASE §3.63 | - |
| reward_policies | reward_type | 자유 확장값, 현재는 `LIFESTYLE_POINT` 단일값(향후 Wallet Engine 확장 시 다른 값 추가 가능) | string | N | N | N | 자유 확장값 | DATABASE §3.63 | - |
| reward_policies | accrual_basis | 적립 기준 — 매출/PV/BV/주문금액/지급수당/기타, 자유 확장값(관리자 선택) | string | N | N | N | 자유 확장값 | DATABASE §3.63 | - |
| reward_policies | accrual_method | 적립 방식 — %/고정Point/누적/단계별/조건부, 자유 확장값(관리자 선택) | string | N | N | N | 자유 확장값 | DATABASE §3.63 | - |
| reward_policies | accrual_rate | 적립률(%, 고정POINT 방식이면 고정 적립액) — Travel/Car/자기계발(§3.25, 기존) 3종은 `plan_definition.lifestyle_bonus.*`(D-032)를 그대로 참조 표시(중복 저장 안 함). Golf 등 신규 Program만 실제 값을 관리자가 직접 입력 | numeric | N | N | N | 미정 | DATABASE §3.63 | - |
| reward_policies | accumulation_period | 누적 방식 — 월/분기/반기/연/사용자지정, 자유 확장값 | string | N | N | N | 자유 확장값 | DATABASE §3.63 | - |
| reward_policies | accumulation_duration | 누적 기간(예: 24개월/1개월/6개월/관리자설정) — Travel/Car/자기계발은 `plan_definition.lifestyle_bonus.*`(D-032) 참조와 동일하게 표시만 하고 별도 저장하지 않음 | string | N | N | N | 미정 | DATABASE §3.63 | - |
| reward_policies | min_condition | 최소 조건 | text/json | N | N | Y | 미정 | DATABASE §3.63 | - |
| reward_policies | max_limit | 최대 한도 | numeric | N | N | Y | 미정 | DATABASE §3.63 | - |
| reward_policies | start_date / end_date | 정책 시작일/종료일 | date | N | N | 미정 | 미정 | DATABASE §3.63 | - |
| reward_policies | usable_period | 적립 후 사용 가능 기간(만료 계산 기준, `wallet_transactions.transaction_type=EXPIRE`와 연계) | string | N | N | N | 미정 | DATABASE §3.63 | - |
| reward_policies | target_wallet_type | `member_wallets.wallet_type`(위) 참조 — 현재는 `LIFESTYLE_POINT`만 유효 | string | N | N | N | 자유 확장값 | DATABASE §3.63 | - |
| reward_policies | is_active | 활성/비활성 | boolean | N | N | N | 미정 | DATABASE §3.63 | - |

- **쇼핑몰 사용**: Lifestyle Point의 쇼핑몰 사용 가능 여부는 Tenant별 설정이다. `tenant_settings`(§3.31.1)는 Multi-Tenant 활성화 보류 상태로 미생성이라, 활성화 전까지는 System Settings(§3.46, 기존)의 키-값 패턴을 재사용한다. **기본값(ON/OFF)은 미확정 — O-208.**
- **마이그레이션**: 기존 `point_transactions.source_type=LIFESTYLE_BONUS`로 이미 적립된 과거 데이터를 Lifestyle Wallet으로 이전할지 여부는 **미확정 — O-209.**
- Settlement(`settlement_batches`/`settlement_items`, §3.6)는 본 절로 전혀 변경되지 않으며, Lifestyle Point/Wallet은 Settlement Ledger를 절대 참조·합산하지 않는다.

## 14. Dictionary Gaps

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
| 후원수당 지급 채널 분배 정책 | 은행송금(기존 정산) vs E-Wallet 적립 — 전액/회원선택/병행 미확정(O-201) — `wallet_transactions`(EARN) 도입의 전제조건 |
| 포인트↔E-Wallet 상호 전환 | `point_transactions`(§3.36)와 `wallet_transactions`(§3.60) 간 전환 허용 여부 미확정(O-202) — 현재는 별개 병행 시스템 |
| 쇼핑몰 결제 시 포인트/지갑 차감 우선순위 | 포인트 먼저 / 지갑 먼저 / 회원 선택 미확정(O-203) |
| 글로벌 결제 PG사 구체 선정 | 한국 신용카드/계좌이체 PG사명, 일본 신용카드 PG사명 등 실제 업체 미확정(O-204) |
| 결제 Webhook 서명 검증 방식 | 검증 방식 및 검증 실패 시 처리(거부/격리 후 검토 등) 미확정(O-205) — `payment_webhook_events.signature_verified` 의미와 직결 |

