# DATABASE.md — 데이터 모델

> 상태: Draft v0.17 (D-031/D-032 — §3.30 쇼핑몰(정기배송/자동결제) 테이블 신설, §3.13 `plan_definition` 스키마 확정(하드코딩 위험 gap 해소), `member_rank_history` 잔여 참조 정리) · 최종 수정일: 2026-06-24 · 단계: 설계(Design)
> 전제 문서: [ARCHITECTURE.md](ARCHITECTURE.md), [COMPENSATION-RULES.md](COMPENSATION-RULES.md), [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)
> 본 문서는 테이블 구조의 **개념 설계**이며, 실제 마이그레이션 파일/스키마는 구현 단계에서 작성한다. 코드/마이그레이션은 생성하지 않는다.

## 1. DB 엔진 및 컨벤션

- 엔진: Supabase (PostgreSQL)
- 네이밍: 테이블/컬럼은 `snake_case`, 테이블명은 복수형(예: `members`, `orders`)을 기본으로 한다 (확정 필요).
- 금액(money) 컬럼: 정수(원 단위, KRW 기준 소수점 없음) 저장을 기본으로 검토 — 확정 필요.
- 식별자: UUID 기본키를 기본으로 검토 (Supabase 기본값과 호환).
- 시각: 모든 timestamp는 UTC로 저장, 표시 시점에 KST 변환.

## 2. 핵심 엔터티 개요

| 엔터티 | 설명 |
|---|---|
| `members` | 회원(파트너) 마스터. 스폰서 관계(self-reference) 포함 |
| `products` | 제품 마스터 |
| `orders` / `order_items` | 주문 및 주문 상세 |
| `commission_records` | 후원수당 산정 결과 원장 (append-only) |
| `settlement_batches` / `settlement_items` | 정산 배치 및 회원별 정산 상세 |
| `tax_withholdings` | 세금 원천징수 내역 |
| `audit_logs` | 모든 도메인 변경에 대한 감사로그 |
| `member_change_requests` | 회원이 **직접 신청 가능한** 민감 변경(명의/탈퇴/휴면/강제탈퇴/재가입/계좌/센터이동 등) 요청·승인·적용 이력 (append-only). **추천인 변경(조직 이동)은 포함하지 않음** — §3.26 참조 |
| `member_sponsor_history` | 추천인(스폰서) 변경 이력 (append-only). Unilevel Sponsor Plan 확정([DECISIONS.md](DECISIONS.md) D-008)에 따라 조직 구조 변경은 이 테이블 하나로 처리하며, 별도 포지션/레그 테이블은 없다. **변경 출처는 `organization_transfers`(관리자 전용, §3.26)로 일원화** ([DECISIONS.md](DECISIONS.md) D-020) |
| `member_bank_accounts` | 정산 수령 계좌 정보 및 변경 이력 (append-only 이력 포함) |
| `change_impact_analyses` | 민감 변경 요청에 대한 수당/정산 영향 분석(시뮬레이션) 결과 |
| `organization_transfer_reason_codes` | 조직 이동(추천인 변경) 허용 사유 8종 카탈로그 (관리자 전용 — §3.26) |
| `organization_transfer_logs` | 조직 이동 요청·승인·적용 원장 (append-only, 관리자 전용) |
| `organization_transfer_attachments` | 조직 이동 신청의 증빙 첨부파일 |
| `organization_transfer_approvals` | 조직 이동 승인/반려 이력 |
| (파생) '내 조직' 조회 데이터 | LINE1~LINE5/조직매출/조직수당/조직성장 — 별도 테이블이 아닌 `order_items`/`commission_records` 기반 파생 데이터 (§3.11) |
| `warehouses` / `inventory_items` | 창고(자사/3PL)별 SKU 재고 현황 |
| `inventory_ledger` | 입고/출고/반품/환수 등 재고 변동 원장 (append-only) |
| `shipments` / `shipment_items` | 배송 송장(운송장) 및 배송 상세, 3PL/택배사 연동 |
| `returns` / `return_items` | 반품 접수/검수/환불 연계 내역 |
| `inventory_recoveries` | 탈퇴·강제탈퇴 시 재고 환수, 리콜 등 회수 내역 |
| `countries` | 지원 국가 마스터 (KR/TH/JP/US 활성 + CN Reserved, [DECISIONS.md](DECISIONS.md) D-023) |
| `member_identity_profiles` | 회원 유형(개인/사업자/법인/외국인)별 본인확인 정보 및 검증 상태 |
| `marketing_plan_versions` | 국가별·시점별 마케팅 플랜(보상플랜 파라미터) 버전 |
| `country_tax_rules` / `country_promotions` / `country_settlement_configs` | 국가별 세금규칙 / 프로모션 / 정산규칙 설정 (시점별 버전 포함) |
| `admin_roles` / `admin_role_assignments` | 관리자 역할 카탈로그 및 회원-역할-국가스코프 매핑 |
| `compliance_report_definitions` / `compliance_report_submissions` | 공제조합(국가별 규제기관) 보고서 정의 및 제출 이력 |
| `centers` | 국가 하위 지역 거점(센터) 마스터 — 도입 여부 미확정 |
| `member_center_history` | 회원의 소속 센터 변경 이력 (append-only) |
| `documents` / `document_versions` / `document_visibility_rules` | Document Center — 문서 마스터, 버전, 공개범위 |
| `tickets` / `ticket_messages` / `ticket_attachments` | Customer Service Center — 문의/이의신청 티켓 |
| `notifications` / `notification_templates` / `notification_logs` | Notification Center — 알림 발송 요청, 템플릿, 발송/실패 이력 |
| `e_signatures` / `consent_history` | 전자서명 기록, 약관/개인정보 동의 이력 |
| `member_activity_logs` | 회원 자기열람용 최근 활동(로그인/주문/수당/정산/프로모션) |
| `rule_designer_drafts` / `rule_versions` / `rule_publish_history` | Rule Designer — 규칙 초안, 버전, 발행 이력 (도입 여부 미확정) |
| `package_purchases` / `package_sales_profits` / `package_pair_bonuses` | 400만원 패키지 구매 내역, 추천인에게 지급되는 제품 판매수익(25%, D-028 개칭), 동일 추천인 산하 구매자 간 페어보너스 — 둘 다 **수령자 본인의 패키지 구매 이력**이 자격 조건(D-028, §3.27.2) |
| `lifestyle_bonus_accumulations` | "+알파" 보너스(여행/자동차/자기계발) 누적 적립 |
| (파생) 회원별 후원수당 자격(월 단위) | `members.status`와 독립된 월 단위 자격 플래그 — 별도 테이블이 아닌 `orders` 기반 파생 데이터 (§3.27, D-024) |
| `referral_link_clicks` | 추천 링크 클릭 이벤트(가입 전 방문자 포함) — 가입 수/구매회원 수는 `members`/`orders`/`package_purchases` 파생값 (§3.28, D-025) |
| `compliance_thresholds` / `compliance_ratio_snapshots` | 법적 한도(35%) 모니터링 — 국가별 임계치 설정 및 실시간/월별/연도누적 비율 스냅샷 (§3.29, D-027) |
| `payment_methods` / `recurring_orders` / `recurring_order_items` / `recurring_order_payment_attempts` | 쇼핑몰 — 회원몰 정기배송(구독형 반복주문)·자동결제 수단·매 주기 결제 시도 이력 (§3.30, D-031) |

## 3. 엔터티 상세 (개념 정의)

### 3.1 `members`

| 컬럼(개념) | 설명 |
|---|---|
| id | 회원 식별자 |
| auth_user_id | Supabase Auth 사용자와의 연결 |
| sponsor_id | 상위 후원자(self-reference, nullable — 최상위 제외). **현재값만 보관** — 변경 이력은 `member_sponsor_history`에 별도 기록. **회원 탈퇴(WITHDRAWN) 시에도 이 값을 자동으로 변경하지 않는다** — 조직 구조는 관리자의 명시적 조직 이동(§3.26)을 통해서만 바뀐다 ([DECISIONS.md](DECISIONS.md) D-021) |
| status | 가입심사중/활성/휴면/탈퇴/강제탈퇴 등 — 공식 상태 체계는 §3.12 참조 (세부 값 집합·전환 조건 확정 필요) |
| member_type | 회원 유형 enum: INDIVIDUAL(개인) / BUSINESS(사업자) / CORPORATION(법인) / FOREIGN(외국인) — 확정([DECISIONS.md](DECISIONS.md) D-011), 유형별 상세 정보는 §3.14 `member_identity_profiles` |
| country_code | 소속 국가 (`countries` 참조 — §3.13). 해당 국가의 마케팅 플랜 버전/세금규칙/정산규칙이 적용됨 |
| center_id | 소속 센터 (`centers` 참조 — §3.17, nullable). **센터는 집계/관리 단위일 뿐 수당 계산에는 영향을 주지 않는다** — 센터 구조 도입 여부 자체가 미확정이므로 nullable로 설계 |
| joined_at | 가입일 |
| previous_member_id | 재가입 시 직전(탈퇴한) 회원 레코드 참조, nullable. 재가입 시 신규 식별자 발급 vs 기존 레코드 재활성화 중 어느 정책을 채택할지에 따라 사용 여부가 달라짐 — **미확정** (§7) |

> 스폰서 트리(=추천조직, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.2)는 `sponsor_id`의 재귀 관계로 표현하며, 이는 FNS의 **유일한** 조직 구조다(Unilevel Sponsor Plan 확정, D-008 — 별도 포지션/레그 테이블 없음). LINE1~LINE5(추천 깊이 5단계) 조회가 빈번하므로, closure table(예: `member_ancestors`, 각 조상까지의 depth 포함) 도입을 권장한다 — 도입 시점은 [DECISIONS.md](DECISIONS.md)에서 확정. 스폰서의 시점별 이력은 §3.9의 `member_sponsor_history`에서 관리하며, `members` 테이블 자체는 항상 "현재" 상태만 가진다.

### 3.2 ~~`ranks` / `member_rank_history`~~ — 폐기됨 (D-030)

> 직급(Rank) 체계 자체가 FNS의 실제 마케팅 플랜에 존재하지 않는다는 정합성 점검 결과에 따라, `ranks`/`member_rank_history` 테이블과 `members.current_rank_id` 컬럼을 **폐기**한다([DECISIONS.md](DECISIONS.md) D-030, O-083 해소). 섹션 번호는 후속 섹션(§3.3 이후)의 교차 참조 보존을 위해 비우지 않고 유지한다. 폐기 배경과 영향 범위 분석은 [DECISIONS.md](DECISIONS.md) §5.10.2 참조.

### 3.3 `products` / `orders` / `order_items`

- `products`: 제품명, 가격, 활성 여부.
- `orders`: 구매자(회원 또는 고객), 주문 상태(결제완료/취소/환불), 주문일, 매출 금액.
- `order_items`: 제품, 수량, 단가.
- 주문 취소/환불 시 매출 차감 처리 방식(별도 음수 차감 엔트리 생성 등)은 [COMPENSATION-RULES.md](COMPENSATION-RULES.md)와 정합성을 맞춰 확정.

### 3.4 ~~`pv_ledger`~~ — 폐기됨 (D-030)

> PV(Point Value) 추상화 계층을 **폐기**한다([DECISIONS.md](DECISIONS.md) D-030, O-084 해소) — 모든 실제 산정 규칙(§3.3 라인매출, §3.5.2/§3.5.5 자격조건, §4.1 패키지)은 PV가 아니라 매출(KRW)을 직접 사용하므로, `pv_ledger`는 실질적으로 어떤 계산에도 입력값으로 쓰이지 않았다. "조직매출"·"후원수당 자격 판정" 등 기존에 `pv_ledger`를 참조하던 파생 데이터는 모두 `orders`/`order_items`(매출 금액 직접 합산)로 대체한다. 섹션 번호는 후속 섹션 교차 참조 보존을 위해 유지한다. 영향 범위는 [DECISIONS.md](DECISIONS.md) §5.10.2 참조.

### 3.5 `commission_records`

- 후원수당 계산 배치가 산출한 회원별 수당 산정 결과를 기록하는 **append-only 원장**.
- 컬럼(개념): 수령자 회원 id, 수당 종류(후원수당(조직수당)/제품 판매수익/페어보너스/+알파/소개/매칭 등 — [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4 확정 후 enum 확정), **line_root_member_id**(후원수당의 경우 — 이 산정이 속한 라인의 시작점인 LINE1 직추천자, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3 정정(D-018)에 따라 라인 단위로 산정하므로 단일 `line_depth` 대신 **라인 식별자 + 그 라인이 도달한 최대 깊이(qualified_depth) + 적용 비율(applied_rate)** 을 함께 기록), source_member_ids(해당 라인에 속한 하위 회원 목록 또는 집계 범위), **plan_version_id**(`marketing_plan_versions` 참조 — 계산 시점에 적용된 국가별 마케팅 플랜 버전, §3.13), 산정 기간, 산정 금액, 계산식/입력 스냅샷(JSON), 계산 배치 실행 id.
- **재계산 시에도 기존 行은 수정하지 않고 새로운 行을 추가**하며, 정정은 보정 엔트리(역분개)로 처리 ([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)).
- ⚠️ **계산 로직 주의(D-018)**: 후원수당은 LINE1~5를 독립적으로 비율 계산해 합산하는 것이 아니라, **회원의 직추천자별 라인마다 도달 깊이를 먼저 판정하고, 그 라인 전체 매출에 단일 비율을 적용**한다 ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3). `commission_records`는 이 산정을 라인 단위로 기록해야 하며, "LINE1행/LINE2행/.../LINE5행"을 별도로 5개 만들어 합산하는 구조로 구현하면 잘못된 금액이 산출된다.
- ⚠️ **계산 로직 주의(D-021)**: 수령자(recipient) 후보를 추릴 때는 `members.status`가 WITHDRAWN/FORCED_WITHDRAWN인 회원을 제외해야 하지만, **라인의 도달 깊이(qualified_depth)를 판정하기 위한 트리 순회 자체는 status와 무관하게 `sponsor_id` 구조 전체를 그대로 사용**해야 한다 (§4 원칙 10). 트리 순회 단계에서 탈퇴 회원을 걸러내면 도달 깊이가 실제보다 낮게 잘못 계산된다.

### 3.6 `settlement_batches` / `settlement_items`

- `settlement_batches`: 정산 회차(기간, 상태: 생성/검증/승인/지급완료, 처리 시각).
- `settlement_items`: 회원별 정산 상세 — 해당 회차에 포함된 commission_records 합계, 세금 차감액, 실지급액, 지급 상태.
- 정산 취소/정정은 `settlement_items`를 직접 수정하지 않고 보정 항목을 추가 ([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)).

### 3.7 `tax_withholdings`

- 정산 항목별 원천징수 내역 (사업소득세 3.3% 등 — [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) 확정 기준 적용).
- 연말 소득 신고/증빙 발급 근거 데이터로 사용.

### 3.8 `audit_logs`

- 모든 모듈의 생성/수정/삭제(논리적) 이벤트를 기록.
- 컬럼(개념): 행위자(시스템/사용자), 대상 엔터티/id, 액션, 변경 전/후 값(JSON), 발생 시각.
- 다음 §3.9의 회원 신청형 민감 변경 8종(회원정보/명의/탈퇴/휴면/강제탈퇴/재가입/계좌/센터이동) 및 §3.26의 관리자 전용 조직 이동은 **반드시** 감사로그 기록 대상이며, `actor`(요청자)와 `approver`(승인자)를 모두 남긴다.

### 3.9 회원 민감 변경(Sensitive Change) 관련 테이블

[PRD.md](PRD.md) §5.3~5.4, [ARCHITECTURE.md](ARCHITECTURE.md) §7에서 정의한 회원 변경/생애주기 관리 기능의 데이터 모델이다.

**`member_change_requests`** — 회원이 **직접 신청 가능한** 민감 변경(회원정보 변경, 명의 변경, 탈퇴, 휴면, 강제탈퇴, 재가입, 계좌 변경, 센터 이동)의 단일 요청/승인 원장 (append-only). **추천인 변경(조직 이동)은 회원이 신청할 수 없으므로 이 테이블의 대상이 아니다** — §3.26 참조.

| 컬럼(개념) | 설명 |
|---|---|
| id | 요청 식별자 |
| member_id | 대상 회원 |
| change_type | 변경 유형 enum (PROFILE_UPDATE / IDENTITY_TRANSFER / WITHDRAWAL / DORMANT / FORCED_WITHDRAWAL / REINSTATEMENT / BANK_ACCOUNT_CHANGE / CENTER_CHANGE). **SPONSOR_CHANGE는 더 이상 이 enum에 포함하지 않는다** — 추천인 변경(조직 이동)은 회원이 신청할 수 없으며, 관리자 전용 `organization_transfer_logs`(§3.26)로 완전히 분리되었다 ([DECISIONS.md](DECISIONS.md) D-020, D-017 대체). CENTER_CHANGE는 센터 구조 도입 시에만 사용 ([PRD.md](PRD.md) §5.9) |
| reason | 변경 사유 (강제탈퇴의 경우 사유 코드 필수) |
| before_snapshot | 변경 전 상태 스냅샷(JSON) — 조직 위치, 계좌, 센터 등 변경 유형에 따라 다름 |
| requested_state | 변경 후 제안 상태(JSON) |
| compensation_impact_id | `change_impact_analyses` 참조 (수당 영향 분석 결과) |
| settlement_impact_id | `change_impact_analyses` 참조 (정산 영향 분석 결과) |
| e_signature_id | `e_signatures` 참조 (§3.21) — IDENTITY_TRANSFER/WITHDRAWAL/FORCED_WITHDRAWAL은 승인 전 전자서명 필수 ([PRD.md](PRD.md) §5.13). 조직 이동(organization_transfer)은 회원 행위가 아니므로 전자서명 대상이 아니다 |
| status | 요청/검토중/승인/반려/적용완료 |
| requested_by / approved_by | 요청자 / 승인자 |
| requested_at / approved_at / applied_at | 각 단계 시각 |

> 승인 후 실제 적용은 `members`의 현재값(sponsor_id, status 등)과 아래 이력 테이블에 새 행을 추가하는 것으로 반영하며, `member_change_requests` 행 자체는 이후 수정하지 않는다.

**`member_sponsor_history`** — 추천인(스폰서) 변경 이력. **변경의 유일한 출처는 관리자 전용 조직 이동(§3.26)이다** ([DECISIONS.md](DECISIONS.md) D-020) — 회원이 직접 발생시키는 경로는 없다.

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| previous_sponsor_id / new_sponsor_id | 변경 전/후 스폰서 |
| transfer_log_id | `organization_transfer_logs` 참조 (§3.26) — **이전에는 `member_change_requests`를 참조했으나 D-020에 따라 변경됨** |
| effective_from | 변경 적용 시점 (이후 계산에만 적용) |

> ~~`member_position_history`~~ — **사용하지 않음.** 바이너리/매트릭스처럼 추천관계와 별도인 포지션(레그) 개념이 있는 모델을 위해 검토했던 테이블이나, 보상플랜 모델이 **Unilevel Sponsor Plan**으로 확정되면서([DECISIONS.md](DECISIONS.md) D-008) 조직 구조는 추천관계(`member_sponsor_history`) 하나뿐이 되어 불필요해졌다. "조직 이동"은 `member_sponsor_history`로 처리한다(기록 출처는 §3.26).

**`member_bank_accounts`** — 정산 수령 계좌. 변경 시 기존 행을 덮어쓰지 않고 새 행을 추가하며 `is_current` 플래그로 현재 계좌를 표시 (append-only + 현재값 플래그 패턴).

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| bank_code / account_number_encrypted / account_holder_name | 계좌 정보 — 암호화 저장 |
| is_current | 현재 사용 계좌 여부 |
| change_request_id | `member_change_requests` 참조 |
| activated_at | 적용 시각 — 사기 방지를 위한 보류기간 적용 시 보류 종료 시점 |

**`change_impact_analyses`** — 민감 변경 요청에 대한 수당/정산 영향 분석(시뮬레이션) 결과.

| 컬럼(개념) | 설명 |
|---|---|
| change_request_id | `member_change_requests` 참조 |
| analysis_type | COMPENSATION_IMPACT / SETTLEMENT_IMPACT |
| affected_member_ids | 영향받는 회원 목록(본인 + 상/하위 조직) |
| estimated_delta | 예상 변동액(JSON, 회원별) |
| input_snapshot | 시뮬레이션에 사용된 입력 데이터 스냅샷 |
| generated_at | 분석 생성 시각 |

> 이 테이블의 결과는 **참고용 시뮬레이션**이며, 실제 `commission_records`/`settlement_items`에는 영향을 주지 않는다 — 승인 후 다음 정규 계산 배치에서 실제 반영된다.

### 3.10 재고/물류 관련 테이블 (Phase 2 — [PRD.md](PRD.md) §5.5)

방문판매법상 회원 탈퇴 시 재고 반품 의무가 있어([LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §11), 최소한의 재고/물류 데이터 모델을 개념 설계 단계에서 함께 정의한다. 3PL(제3자 물류) 연동을 전제로 한다 ([ARCHITECTURE.md](ARCHITECTURE.md) §2.6).

| 엔터티 | 설명 |
|---|---|
| `warehouses` | 창고 마스터 (자사/3PL 구분) |
| `inventory_items` | 창고별 SKU 재고 현황(파생값 — `inventory_ledger` 합으로 도출 검토) |
| `inventory_ledger` | 입고/출고/반품 재입고/환수/폐기 등 모든 재고 변동을 기록하는 **append-only 원장** |
| `shipments` / `shipment_items` | 주문에 대한 배송 송장(운송장) 발행 내역, 택배사/3PL 연동, 배송 상태(추적) |
| `returns` / `return_items` | 반품 접수, 검수 결과, 환불·재입고 처리 연계 (청약철회권 — [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §4) |
| `inventory_recoveries` | 회원 탈퇴/강제탈퇴 시 보유 재고 환수, 또는 불량/리콜에 의한 회수 — 환수 수량, 환급액, 관련 `member_change_requests` 참조 |

- `inventory_ledger`/`returns`/`inventory_recoveries`는 금전(환급액) 영향이 있으므로 **append-only** 원칙을 동일하게 적용한다.
- 3PL 측 재고와의 동기화 방식, 정합성 대조 주기는 **미확정** — 구현 단계에서 3PL 연동 방식 확정 후 결정.

### 3.11 '내 조직' 메뉴 조회용 파생 데이터 (LINE1~5 / 조직매출 / 조직수당 / 조직성장)

[PRD.md](PRD.md) §5.1.1에서 정의한 회원용 '내 조직' 메뉴의 4개 항목은 **별도 테이블이 아니라 기존 원장의 파생(derived) 데이터**다.

| 항목 | 산출 방식 |
|---|---|
| LINE1~LINE5 | `sponsor_id`(또는 closure table `member_ancestors`, §3.1)를 추천 깊이 5단계까지 순회하여 산출 |
| 조직매출 | 위에서 산출한 LINE1~5 회원들의 `order_items`(또는 `orders`) 매출 합계 |
| 조직수당 | 본인의 `commission_records` 중 수당 종류가 후원수당(Sponsor Bonus)이고 `source_member_id`가 LINE1~5에 속하는 행의 합계 (§3.5) |
| 조직성장 | LINE1~5 회원들의 `joined_at`(신규가입) 추이를 기간별로 집계 |

- 위 항목을 매 요청마다 실시간 계산할지, 별도 집계/캐시 테이블(예: `member_organization_summary`)로 미리 계산해 둘지는 **미확정** — 회원 수·LINE 조회 빈도에 따라 결정 ([DECISIONS.md](DECISIONS.md)).
- 어떤 방식이든, 이 파생 데이터는 **읽기 전용 조회 결과**이며 `commission_records`/`orders` 원장 자체를 대체하지 않는다.

### 3.12 회원 상태 체계 (구조 제안 — [PRD.md](PRD.md) §5.6.3)

`members.status`가 가질 수 있는 값과 의미. 명칭/세부 전환 조건은 미확정이며, 아래는 현재까지 식별된 구조다.

| 상태 | 의미 | 진입 경로 |
|---|---|---|
| PENDING_VERIFICATION (가입심사중) | 회원유형별 본인확인 서류 검증 대기 — §3.14 `member_identity_profiles` | 신규 가입, 재가입 |
| ACTIVE (활성) | 정상 활동 중 | 가입심사 승인, 휴면 회원의 재활동 |
| DORMANT (휴면) | 장기 미활동으로 전환 (기준 미확정) | 활성 상태에서 일정 기간 미활동 |
| WITHDRAWN (탈퇴) | 자발적 탈퇴 ([DATABASE.md](DATABASE.md) §3.9 `member_change_requests`, change_type=WITHDRAWAL) | 활성/휴면에서 탈퇴 신청·승인 |
| FORCED_WITHDRAWN (강제탈퇴) | 본사에 의한 탈퇴 (change_type=FORCED_WITHDRAWAL) | 약관/법령 위반 적발 |

- "정지(SUSPENDED)" 등 추가 상태 필요 여부는 **미확정**.
- 상태 전환 자체는 모두 §3.9의 `member_change_requests` 워크플로우(요청→승인→적용)를 통하며, `members.status`를 직접 UPDATE하는 경로는 두지 않는다 — 단, PENDING_VERIFICATION ↔ ACTIVE 전환(가입심사)은 기존 9종 change_type에 추가로 정의 필요 (예: `SIGNUP_VERIFICATION`).
- **WITHDRAWN/FORCED_WITHDRAWN 전환은 `sponsor_id`/조직 구조에 어떤 영향도 주지 않는다 (확정, D-021)** — 상태값만 바뀌고 트리 구조는 그대로다. 수당(§[COMPENSATION-RULES.md](COMPENSATION-RULES.md) §5)/정산([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9) 계산 배치는 이 두 상태를 신규 산정/판정 대상에서 제외하는 필터를 반드시 적용해야 한다.

### 3.13 국가 구조 및 국가별 규칙 설정 ([PRD.md](PRD.md) §5.6.2, §5.8)

**`countries`** — 지원 국가 마스터.

| 컬럼(개념) | 설명 |
|---|---|
| country_code | ISO 3166-1 alpha-2 코드. **확정 대상: KR / TH / JP / US (활성) + CN (Reserved)** — [DECISIONS.md](DECISIONS.md) D-011/D-023 |
| name / currency / default_language | 국가명, 통화, 기본 언어 |
| **status** (신규, D-023 — `is_active` 대체) | **ACTIVE**(실제 운영 중 — 현재 KR만) / **PLANNED**(지원 대상이나 미출시 — TH/JP/US) / **RESERVED**(향후 확장을 위해 보관, 현재 활성 구조 제외 — CN). RESERVED 국가는 회원 가입·정산·보고 등 어떤 활성 기능에도 노출되지 않는다 |
| launched_at | 실제 운영 시작일 (nullable — 미출시국은 null) |

> ~~`is_active`(boolean)~~ — **`status`(enum) 3단계로 대체** (D-023). 단순 활성/비활성 2분류로는 "지원하지만 아직 미출시"(PLANNED)와 "지원 대상에서 제외되어 보관 중"(RESERVED)을 구분할 수 없었기 때문이다. CN을 RESERVED로 두는 것은 "비활성 처리"가 아니라 "활성 구조 자체에서 분리"를 의미한다 — 예를 들어 관리자가 국가를 단순히 `is_active=false`로 토글하는 것과는 달리, RESERVED→ACTIVE 전환은 별도의 명시적 의사결정(신규 D-번호)을 요구한다 (확정, D-023).

**`marketing_plan_versions`** — 국가별·시점별 마케팅 플랜(보상플랜 파라미터) 버전.

| 컬럼(개념) | 설명 |
|---|---|
| id | 버전 식별자 |
| country_code | 적용 국가 (`countries` 참조) |
| version_label | 버전 명칭 (예: "KR v1") |
| status | DRAFT / ACTIVE / EXPIRED |
| effective_from / effective_to | 적용 기간 (to는 nullable — 현재 유효 버전은 null) |
| plan_definition | LINE별 비율 등 후원수당/패키지/자격/+알파 파라미터(JSON) — **확정 스키마(D-032, 아래)** |
| created_by / approved_by | 작성자/승인자 |

> ✅ **관리자 설정화 스키마 확정(D-032, [DECISIONS.md](DECISIONS.md) §5.10.5 gap 해소)**: D-005 시점 placeholder를 대체하여, D-018~D-028에서 확정된 실제 수치를 모두 `plan_definition` 필드로 명시한다 — 아래 필드 외의 수치를 코드/마이그레이션에 하드코딩하지 않는다(§15 관리자 설정 원칙).
>
> ```
> plan_definition:
>   unilevel:                                  # COMPENSATION-RULES.md §3.3
>     line_rates: [3, 3, 3, 4, 5]               # 라인 도달 깊이별 단일 비율(%) — index 0=깊이1~3(3%), 3=깊이4(4%), 4=깊이5(5%)
>     max_depth: 5                              # LINE6 이상은 산정 제외
>   package:                                     # COMPENSATION-RULES.md §4.1
>     price: 4000000                            # 패키지 가격(원) — 1종
>     sales_profit_rate: 0.25                   # 제품 판매수익 비율 — amount = price * rate
>     pair_bonus_amount: 2000000                # 페어보너스(Pair당, 원)
>     pair_window_days: 30                      # 페어 판정 윈도우(일)
>   eligibility:                                 # COMPENSATION-RULES.md §3.5
>     unilevel_maintenance_amount: 50000        # 유니레벨 후원수당 자격 — 매월 누적 구매 기준액(원)
>   lifestyle_bonus:                             # COMPENSATION-RULES.md §4.2 (옵션, 세부 수치 미확정)
>     travel_rate_range: [0.001, 0.005]
>     car_rate: 0.002
>     self_dev_rate: 0.002
> ```
>
> - 이 스키마는 `country_code`+`effective_from/to`로 버전 관리되는 `marketing_plan_versions` 행의 한 컬럼이므로, **국가별·시점별로 다른 값을 가질 수 있다** — 예를 들어 TH 시장이 다른 LINE 비율을 채택해도 코드 변경 없이 새 버전 행을 추가하는 것으로 반영한다.
> - **35% 한도 임계치(30/33/35%)는 이 스키마에 포함하지 않는다** — 이미 별도 테이블 `compliance_thresholds`(§3.29, D-027)로 분리 설계되어 있으며, 두 설정의 변경 승인 절차가 다를 수 있기 때문이다(한도는 법적 강제값, 마케팅 플랜은 사업적 선택값).
> - `lifestyle_bonus`의 실제 수치는 여전히 **미확정**([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.2)이므로, 위 값은 원본 문서상 범위를 그대로 옮긴 placeholder다 — 사업팀 확정 후 갱신한다.
> - 패키지 종류가 향후 2종 이상으로 늘어날 경우(현재는 1종 확정, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.1) `package`는 배열로 확장이 필요하다 — 현재는 1종 기준 단일 객체로 둔다.

> `commission_records.plan_version_id`(§3.5)가 이 테이블을 참조하여, 과거 산정 결과가 "그 당시 어떤 버전으로 계산되었는지"를 항상 추적 가능하게 한다. 플랜 버전이 바뀌어도 과거 `commission_records`는 재계산하지 않는다 (기존 append-only/비소급 원칙과 동일).

**`country_tax_rules` / `country_promotions` / `country_settlement_configs`** — 국가별 세금규칙 / 프로모션 / 정산규칙. `marketing_plan_versions`와 동일한 패턴(country_code + effective_from/to + 버전 상태)을 따르는 설정 테이블로, 실제 컬럼 스키마는 각각 [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md), [COMPENSATION-RULES.md](COMPENSATION-RULES.md) 후속 라운드에서 확정한다. 현재는 "국가+시점별로 달라질 수 있는 설정"이라는 구조만 정의한다.

### 3.14 회원 유형별 신원 정보 ([PRD.md](PRD.md) §5.6.1)

**`member_identity_profiles`** — 회원 유형(개인/사업자/법인/외국인)별 본인확인 정보 (1:1 with `members`).

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| member_type | INDIVIDUAL / BUSINESS / CORPORATION / FOREIGN |
| identity_fields | 유형별 본인확인 데이터(JSON, 암호화) — 개인: 최소화된 본인확인 정보, 사업자: 사업자등록번호/대표자명, 법인: 법인등록번호/법인명/대표자명, 외국인: 외국인등록번호 또는 여권정보/국적 |
| document_refs | 제출 서류의 Supabase Storage 참조 |
| verification_status | 심사대기 / 승인 / 반려 |
| verified_by / verified_at | 심사자/심사 시각 |

- `identity_fields`의 정확한 필드 목록은 **미확정** — 각 유형별 필요 최소 정보는 [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) 후속 라운드(개인정보보호법 최소수집 원칙)에서 확정한다.

### 3.15 관리자 권한 체계 ([PRD.md](PRD.md) §5.6.4)

**`admin_roles`** — 역할 카탈로그 (SuperAdmin/CountryAdmin/기능별 역할 등 — 세부 목록 미확정).

**`admin_role_assignments`** — 사용자-역할-국가스코프 매핑.

| 컬럼(개념) | 설명 |
|---|---|
| admin_user_id | 대상 관리자 |
| role_id | `admin_roles` 참조 |
| country_scope | 적용 국가 (nullable = 전체 국가, SuperAdmin용) |
| granted_by / granted_at | 부여자/부여 시각 |

- 역할 부여/회수는 `audit_logs`에 필수 기록 ([PRD.md](PRD.md) §5.6.4).
- 세부 역할 목록과 역할별 권한(리소스×액션) 매핑은 **미확정**.

### 3.16 공제조합 보고센터 ([PRD.md](PRD.md) §5.7)

**`compliance_report_definitions`** — 국가별 규제 보고서 정의.

| 컬럼(개념) | 설명 |
|---|---|
| country_code | 대상 국가 |
| report_type | 보고서 종류 (예: KR=공제조합 정기보고 — 타국 동등 보고서 종류는 미확정) |
| frequency | 보고 주기 (월간/분기 등 — 미확정) |
| required_fields | 보고서에 포함되어야 하는 항목 정의(JSON) — 미확정 |

**`compliance_report_submissions`** — 생성/제출된 보고서 이력 (append-only).

| 컬럼(개념) | 설명 |
|---|---|
| definition_id | `compliance_report_definitions` 참조 |
| period | 보고 대상 기간 |
| generated_at | worker가 보고서를 생성한 시각 |
| file_ref | 생성된 보고서 파일의 Supabase Storage 참조 |
| status | 생성됨 / 검토중 / 제출완료 |
| submitted_at / submitted_by | 제출 시각/제출자 |

- 보고서 생성은 `scheduler`가 트리거하고 `worker`가 수행한다 ([ARCHITECTURE.md](ARCHITECTURE.md) §8).
- KR 외 4개국의 실제 보고 요건은 **미확정** — [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) 후속 라운드에서 국가별로 정의한다.
- **매출 총액/후원수당 총액/비율(확정, D-027)**: 보고서가 이 수치를 직접 재집계하지 않고, §3.29 `compliance_ratio_snapshots`(`period_type=ANNUAL_CUMULATIVE`)의 최신 값을 그대로 인용한다 — 모니터링 대시보드와 규제 보고서 간 숫자 불일치를 구조적으로 방지([ARCHITECTURE.md](ARCHITECTURE.md) §8.1.4).

### 3.17 센터(Center) 구조 ([PRD.md](PRD.md) §5.9 — 도입 여부 미확정)

**`centers`** — 국가 하위 지역 거점 마스터.

| 컬럼(개념) | 설명 |
|---|---|
| id | 센터 식별자 |
| country_code | 소속 국가 (`countries` 참조 — 센터는 정확히 하나의 국가에 속함) |
| name | 센터명 (예: 서울센터, 부산센터, 방콕센터, 도쿄센터) |
| is_active | 운영 여부 |

**`member_center_history`** — 회원의 소속 센터 변경 이력 (append-only, `member_sponsor_history`와 동일 패턴).

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| previous_center_id / new_center_id | 변경 전/후 센터 |
| change_request_id | `member_change_requests` 참조 (change_type=CENTER_CHANGE) |
| effective_from | 변경 적용 시점 |

- 센터는 `members.center_id`(§3.1)로 현재값만 보관하고, 변경 이력은 본 테이블로 관리한다 — 스폰서 트리와 동일한 "현재값 테이블 + append-only 이력 테이블" 패턴.
- **센터별 매출/조직/수당**은 별도 집계 테이블이 아니라, `members.center_id` 기준으로 `orders`/`commission_records`를 그룹화한 **파생 조회**다 (§3.11의 '내 조직' 파생 데이터와 동일한 접근).
- 센터 이동이 국가 변경을 수반하는 경우의 처리(센터만 변경 vs 국가도 함께 변경)는 **미확정**.

### 3.18 Document Center ([PRD.md](PRD.md) §5.10)

**`documents`** — 문서 마스터 (약관/보상플랜안내/교육자료/공지사항 등 관리자 업로드 문서).

| 컬럼(개념) | 설명 |
|---|---|
| id | 문서 식별자 |
| document_type | 약관 / 보상플랜안내 / 교육자료 / 공지사항 / (시스템 생성형: 정산자료/지급명세서/사업자자료/원천징수자료) |
| title | 문서명 |
| current_version_id | 현재 게시 버전 (`document_versions` 참조) |

**`document_versions`** — 버전 관리 (append-only).

| 컬럼(개념) | 설명 |
|---|---|
| document_id | `documents` 참조 |
| version_label | 버전 식별자 |
| file_ref | Supabase Storage 참조 |
| country_code | 국가별 문서일 경우 해당 국가 (nullable = 전체 공통) |
| published_by / published_at | 게시자/게시 시각 |

**`document_visibility_rules`** — 공개범위 설정.

| 컬럼(개념) | 설명 |
|---|---|
| document_id | `documents` 참조 |
| scope_type | ALL / COUNTRY / MEMBER_TYPE 등 (세부 미확정) |
| scope_value | 적용 대상 값 (예: country_code, member_type) |

- "회원 개인 문서"(정산자료/지급명세서/원천징수자료)는 `documents`/`document_versions`에 행을 직접 추가하는 것이 아니라, `settlement_items`/`tax_withholdings`로부터 **worker가 생성**하여 회원별 파일 참조를 연결하는 별도 경로를 가진다 — 정확한 연결 테이블/방식은 **미확정**.

### 3.19 Customer Service Center ([PRD.md](PRD.md) §5.11)

**`tickets`**

| 컬럼(개념) | 설명 |
|---|---|
| id | 티켓 식별자 |
| member_id | 문의 회원 |
| category | 1:1문의/정산문의/수당문의/회원문의/명의변경문의/이의신청/불만접수 |
| related_change_request_id | `member_change_requests` 참조 (nullable — 명의변경 문의/이의신청이 실제 변경 요청으로 이어지는 경우 연결) |
| status | 접수/처리중/답변완료/종료 |
| assigned_to | 담당 관리자 (§3.15 `admin_role_assignments`와 연계) |
| created_at | 접수 시각 |

**`ticket_messages`** — 티켓 내 대화 이력 (append-only).

| 컬럼(개념) | 설명 |
|---|---|
| ticket_id | `tickets` 참조 |
| sender_type | MEMBER / ADMIN |
| sender_id | 발신자 |
| body | 본문 |
| created_at | 발신 시각 |

**`ticket_attachments`** — 첨부파일 (Supabase Storage 참조).

| 컬럼(개념) | 설명 |
|---|---|
| ticket_message_id | `ticket_messages` 참조 |
| file_ref | Storage 참조 |

- "이의신청" 카테고리 티켓이 강제탈퇴 재심 등으로 이어지는 경우, `related_change_request_id`로 새로운 `member_change_requests`(REINSTATEMENT 등)와 연결한다.

### 3.20 Notification Center ([PRD.md](PRD.md) §5.12)

**`notification_templates`**

| 컬럼(개념) | 설명 |
|---|---|
| id | 템플릿 식별자 |
| channel | EMAIL / SMS / KAKAOTALK / PUSH |
| country_code / language | 국가/언어별 템플릿 분기 (nullable = 공통) |
| event_type | 발생 이벤트(가입승인/정산완료 등 — 미확정 목록) |
| content_template | 메시지 본문 템플릿 |

**`notifications`** — 발송 요청 (Job 단위).

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 수신 회원 |
| template_id | `notification_templates` 참조 |
| channel | 발송 채널 |
| status | 대기/발송중/발송완료/실패 |
| requested_at | api가 Job을 생성한 시각 |

**`notification_logs`** — 발송/실패 이력 (append-only).

| 컬럼(개념) | 설명 |
|---|---|
| notification_id | `notifications` 참조 |
| attempt_no | 재시도 회차 |
| result | 성공/실패 |
| failure_reason | 실패 시 사유 |
| processed_at | worker가 처리한 시각 |

- `notifications`는 `api`가 생성하고 `worker`가 처리하는 일반적인 Job 패턴을 그대로 따른다 ([ARCHITECTURE.md](ARCHITECTURE.md) §1.1). 재시도는 Redis Retry 메커니즘을 사용하되, 최종 결과는 `notification_logs`(Supabase)에 남는다.

### 3.21 전자서명(E-Signature) 및 동의 이력 ([PRD.md](PRD.md) §5.13)

**`e_signatures`**

| 컬럼(개념) | 설명 |
|---|---|
| id | 서명 식별자 |
| member_id | 서명자 |
| related_change_request_id | `member_change_requests` 참조 (nullable — 회원가입 등 변경요청이 아닌 경우도 있음) |
| signed_payload_hash | 서명 대상 문서/내용의 해시 |
| signed_at | 서명 시각 |
| signature_method | 서명 방식 (미확정) |

**`consent_history`** — 약관/개인정보 동의 이력 (append-only, 변경요청과 독립적).

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 동의자 |
| consent_type | TERMS(약관) / PRIVACY(개인정보) |
| document_version_id | 동의 대상 문서 버전 (`document_versions` 참조 — §3.18) |
| consented_at | 동의 시각 |

- `member_change_requests.e_signature_id`(§3.9)가 `e_signatures`를 참조하여, 명의변경/탈퇴/강제탈퇴 승인 전 서명 완료 여부를 검증한다. (조직 이동은 회원 행위가 아니므로 전자서명 대상이 아니다 — §3.26)
- 전자서명의 법적 요건(서명 방식, 보존 기간 등)은 **미확정** — 국가별로 다를 수 있음.

### 3.22 회원 활동 이력 (Member Activity Log) ([PRD.md](PRD.md) §5.14)

**`member_activity_logs`** — 회원 자기열람용 활동 로그.

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| activity_type | LOGIN / ORDER / COMMISSION / SETTLEMENT / PROMOTION |
| related_id | 관련 레코드 참조(주문 id, commission_records id 등 — activity_type에 따라 다름) |
| occurred_at | 발생 시각 |

- `audit_logs`(§3.8)와 별도 — audit_logs는 모든 시스템 변경에 대한 포렌식 기록이고, 본 테이블은 회원이 본인 화면에서 빠르게 조회할 수 있도록 최적화된 요약 로그다. 동일 이벤트가 두 테이블에 중복 기록될 수 있다(목적이 다르므로 정상).

### 3.23 Rule Designer ([PRD.md](PRD.md) §5.15 — 도입 여부 미확정)

**`rule_designer_drafts`** — 관리자가 작성 중인 규칙 초안.

| 컬럼(개념) | 설명 |
|---|---|
| id | 초안 식별자 |
| rule_target | LINE_RATIO / RANK_CONDITION / PROMOTION_CONDITION / SETTLEMENT_CONDITION / COUNTRY_RULE |
| country_code | 적용 국가 |
| draft_content | 초안 내용(JSON) |
| created_by | 작성자 |

**`rule_versions`** — 시뮬레이션을 거친 발행 후보 버전.

| 컬럼(개념) | 설명 |
|---|---|
| draft_id | `rule_designer_drafts` 참조 |
| simulation_result_id | `change_impact_analyses`(§3.9)와 동일한 패턴의 시뮬레이션 결과 참조 |
| legal_limit_check | 법적 한도(35% 등) 검증 통과 여부 — **위반 시 발행 자체를 시스템이 차단** (확정 — 원칙) |
| status | 시뮬레이션완료/승인/반려 |

**`rule_publish_history`** — 실제 발행 이력 (append-only).

| 컬럼(개념) | 설명 |
|---|---|
| rule_version_id | `rule_versions` 참조 |
| target_table | 발행 결과가 반영된 실제 테이블 (`marketing_plan_versions` 등) |
| published_by / published_at | 발행자/발행 시각 |

- Rule Designer가 도입되더라도, 실제 적용은 항상 `marketing_plan_versions`/`country_tax_rules`/`country_promotions`/`country_settlement_configs`(§3.13)에 새 버전을 추가하는 것으로 이루어진다 — Rule Designer는 그 위의 **편집/검증/발행 워크플로우**이며 별도의 계산 로직을 갖지 않는다.
- 도입 여부 자체와 적용 대상 규칙의 범위는 **미확정**.

### 3.24 제품 판매수익 / 페어보너스 (Package Sales Profit & Pair Bonus) ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1, [DECISIONS.md](DECISIONS.md) D-028 — D-024 "패키지 추천보너스" 명칭·자격 정정)

> 이전 라운드(D-019)의 `pack_sales`/`pack_sales_pair_bonuses`(수령자=판매한 회원)를 폐기·대체한 모델이다. 새 모델의 수령자는 항상 **구매자의 추천인**이며, **D-028로 수령자 본인의 패키지 구매 이력이 자격 조건으로 추가**되었다. 테이블명을 `package_referral_bonuses` → **`package_sales_profits`** 로 변경한다(명칭 정정 반영).

**`package_purchases`** — 회원의 400만원 패키지 구매 내역.

| 컬럼(개념) | 설명 |
|---|---|
| id | 구매 식별자 |
| buyer_member_id | 패키지를 구매한 회원 |
| order_id | 관련 주문 (`orders` 참조) |
| amount | 구매 금액(400만원) |
| **referrer_member_id** | 구매 시점의 `members.sponsor_id` **스냅샷** — 이후 조직 이동으로 구매자의 추천인이 바뀌어도 이 값은 변하지 않는다(§3.26 D-022와 동일한 "계산 시점 스냅샷" 원칙) |
| purchased_at | 구매 시각 — 페어보너스 30일 Pair Window 및 §3.5.5 자격(본인 구매 이력) 판정의 기준 시각 |

> **`pair_status`(파생, 컬럼 아님)**: 이 구매가 `package_pair_bonuses`의 `package_purchase_id_1`/`_2`로 참조되면 **PAIRED**, 참조되지 않은 채 `purchased_at`로부터 30일이 지났으면 **EXPIRED_NO_PAIR**(D-026 — 영구 종료, 이후 어떤 페어 계산에도 재사용되지 않음), 그 외에는 **PENDING**(대기 중)으로 derive한다 — 별도 컬럼으로 저장할지는 §8 Open Decisions에서 추적.

> **회원의 "본인 자격 보유 여부"(파생, §3.5.5)**: `package_purchases`에 `buyer_member_id = 해당 회원`인 행이 1건이라도 존재하면 그 회원은 제품 판매수익·페어보너스 자격을 보유한다 — 별도 플래그 컬럼 없이 존재 여부(EXISTS 쿼리)로 판정 가능하다.

**`package_sales_profits`**(D-028로 `package_referral_bonuses`에서 개칭) — 패키지 구매 시 추천인에게 지급되는 25%(100만원) 보너스 기록 (append-only).

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| package_purchase_id | `package_purchases` 참조 |
| referrer_member_id | 수령자(해당 구매의 `referrer_member_id`와 동일) |
| bonus_amount | 100만원(구매금액의 25%) |
| commission_record_id | `commission_records` 참조 — 실제 지급은 이 원장을 통해 이루어짐 |

> **자격 검증(신규, D-028)**: 이 행은 `referrer_member_id`가 §3.5.5 자격(본인 `package_purchases` 1건 이상)을 보유한 경우에만 생성된다 — **자격 미충족이면 이 테이블에 행 자체를 만들지 않는다**(append-only 원장에 "0원 지급" 행을 남기지 않고, 그냥 생성하지 않는 방식을 기본으로 한다). 자격 미충족 사실 자체를 추적할지(예: 감사 목적의 "미지급 사유" 로그)는 **미확정**(O-082, 신규) — 회원 화면(§5.1.2)에서 "왜 지급되지 않았는지" 설명하려면 최소한의 이벤트 로그가 필요할 수 있다.

**`package_pair_bonuses`** — 동일 추천인의 직추천 패키지 구매자끼리 구매 순서로 페어가 성립할 때 지급되는 200만원 보너스 기록 (append-only).

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| referrer_member_id | 페어가 묶이는 기준이 되는 추천인(수령자) |
| package_purchase_id_1 / package_purchase_id_2 | 페어를 구성하는 두 `package_purchases` 참조 — 둘 다 `referrer_member_id`의 직추천 구매여야 한다 |
| pair_sequence_no | 해당 추천인 산하에서 몇 번째 페어인지(1, 2, 3 ...) — §4.1.3의 "1-2번째, 3-4번째" 순서 매김에 사용 |
| window_start / window_end | 30일 판정 기준 기간(앞선 구매의 `purchased_at` 기준) |
| bonus_amount | 200만원 |
| commission_record_id | `commission_records` 참조 |

> **자격 검증(신규, D-028)**: `package_sales_profits`와 동일한 원칙 — `referrer_member_id`가 §3.5.5 자격을 보유하지 않으면 페어가 성립(2건의 구매가 발생)해도 이 테이블에 행을 생성하지 않는다. 단, **대기 후보 큐 관리(아래) 자체는 자격과 무관하게 정상 동작**해야 한다 — 자격 없는 추천인의 직추천 구매도 페어 큐에는 정상적으로 들어가지만, 페어가 성립하는 순간 지급 자격만 막힌다(큐 소비는 그대로 발생 — 즉 자격 없는 추천인 밑에서도 구매들은 정상적으로 페어 매칭되어 "소비"되며, 나중에 자격을 갖춰도 그 페어가 되살아나지 않는다).

- 25%/200만원의 법적 분류(후원수당 vs 제품 판매이익, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1)가 아직 법무 최종 확인 전이므로, 이 3개 테이블과 `commission_records`를 분리해 두어 **법무 결정에 따라 35% 한도 산정에 포함/제외를 유연하게 전환**할 수 있도록 설계했다 — 현재는 포함을 기본값으로 한다.
- **큐 관리(확정, [DECISIONS.md](DECISIONS.md) D-026 — O-072 해소)**: `referrer_member_id`별로 "짝을 기다리는 대기 후보"는 최대 1건만 존재한다. 새 구매가 들어올 때 대기 후보가 있고 30일 이내면 페어 성립(`package_pair_bonuses` 1행 생성 — 자격 보유 시), 대기 후보가 없거나 이미 30일이 지났으면 새 구매가 새 대기 후보가 된다. **만료된 대기 후보(`pair_status = EXPIRED_NO_PAIR`)는 이후 어떤 페어 계산에도 재사용·이월·재매칭되지 않는다** — 예: A 구매 후 30일간 짝 없음 → A 종료, 이후 B·C가 구매하면 B+C가 페어이며 A는 포함되지 않는다.
- **재구매 제약 없음(누락 발견, [DECISIONS.md](DECISIONS.md) O-077)**: `package_purchases`에는 `buyer_member_id`당 1건으로 제한하는 unique 제약이 없다 — 즉 현재 스키마는 재구매를 암묵적으로 허용한다. 패키지가 1회성 상품으로 확정되면 `(buyer_member_id)` 유니크 제약(또는 애플리케이션 레벨 검증)을 추가해야 한다. **§3.5.5 자격은 "1건 이상 존재"로만 판정하므로 재구매 정책과 무관하게 영향받지 않는다.**
- **자기매칭 방지 제약 누락(O-077, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.3)**: `package_pair_bonuses.package_purchase_id_1`/`_2`는 둘 다 `referrer_member_id`의 직추천 구매여야 한다는 제약만 있고, **두 구매의 `buyer_member_id`가 서로 달라야 한다는 제약이 없다** — 페어 매칭 쿼리/로직에 `package_purchase_id_1.buyer_member_id <> package_purchase_id_2.buyer_member_id` 조건을 반드시 포함해야 한다(재구매 정책과 무관하게 구현 시 반영 권고).

### 3.25 "+알파" 보너스 누적 (Lifestyle Bonus Accumulation) ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.2)

**`lifestyle_bonus_accumulations`** — 회원별 여행/자동차/자기계발 누적 적립 현황.

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| bonus_type | TRAVEL(0.1~0.5%, 24개월) / CAR(0.2%, 1개월) / SELF_DEVELOPMENT(0.2%, 6개월) |
| accumulation_period_start / end | 누적 기간 |
| accumulated_amount | 현재까지 누적된 적립액 |
| status | 적립중/지급완료/소멸 |

- 적립 주기 종료 시 소멸 또는 이월 여부는 **미확정** ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.2).
- 지급(현금 환산 vs 실물/서비스 제공)은 정산(`settlement_items`)과 별도 경로일 가능성이 있음 — **미확정**.

### 3.26 조직 이동 (Organization Transfer) — 관리자 전용 ([PRD.md](PRD.md) §5.16, [DECISIONS.md](DECISIONS.md) D-020, D-021, D-022; D-017 대체)

추천인 변경(= 조직 이동)을 회원 자기서비스(`member_change_requests`)에서 완전히 분리한 **전용 테이블 세트**다. 회원이 생성할 수 있는 경로는 없으며, 모든 행은 관리자(SuperAdmin/Compliance Admin/지정 조직관리 관리자)에 의해서만 생성된다.

**`organization_transfer_reason_codes`** — 허용 사유 카탈로그.

| 컬럼(개념) | 설명 |
|---|---|
| code | 사유 코드 (예: WITHDRAWAL/DEATH/IDENTITY_TRANSFER/OPERATIONAL_ERROR/COMPANY_REORG/COURT_ORDER/LEGAL_ACTION/MAJOR_COMPLIANCE_ISSUE/OTHER_APPROVED — 정확한 코드값은 [DECISIONS.md](DECISIONS.md) O-067 확정 필요) |
| label | 회원 탈퇴 / 회원 사망 / 명의 변경 / 운영 오류 수정 / 회사 정책상 조직 재배치 / 법원 판결 / 법적 조치 / **중대한 컴플라이언스 이슈(신규, D-022)** / 기타 회사 승인 사유 (**9종**, [PRD.md](PRD.md) §5.16.3) |
| requires_attachment | 증빙 첨부 필수 여부 (사유별로 다를 수 있음 — 미확정) |
| **is_emergency_eligible** (신규, D-022) | 이 사유로 **긴급(즉시 적용) 조직 이동**을 신청할 수 있는지 여부. 회원사망/법원판결/법적조치/운영오류수정/중대한컴플라이언스이슈 = `true`, 나머지 4종 = `false` |
| is_active | 사용 가능 여부 (관리자가 카탈로그 관리, §5.16.6 "사유 코드 관리") |

**`organization_transfer_logs`** — 조직 이동 요청·승인·적용 원장 (append-only, `member_change_requests`와 유사하지만 별도 테이블).

| 컬럼(개념) | 설명 |
|---|---|
| id | 요청 식별자 |
| member_id | 대상 회원(추천인이 변경되는 회원) |
| reason_code_id | `organization_transfer_reason_codes` 참조 (필수) |
| reason_detail | 사유 상세 설명(자유 텍스트, "기타 회사 승인 사유" 선택 시 특히 중요) |
| previous_sponsor_id / new_sponsor_id | 변경 전/후 스폰서 |
| before_snapshot | 변경 전 조직 상태 스냅샷(JSON) |
| compensation_impact_id / settlement_impact_id | `change_impact_analyses` 참조 (§3.9와 동일 패턴) |
| **is_emergency** (신규, D-022) | 긴급 조직 이동 여부. `true`인 경우 `reason_code_id`가 가리키는 사유의 `is_emergency_eligible`이 `true`여야 하며, 승인자 역할이 SuperAdmin이어야 한다(애플리케이션 레벨 검증) |
| status | REQUESTED(요청) / UNDER_REVIEW(증빙첨부·영향분석 진행중) / **APPROVED_SCHEDULED(승인됨, 적용일 대기중 — 신규, D-022)** / APPLIED(적용완료) / REJECTED(반려). §5.16.4의 9단계에 대응 — 승인(⑤)과 적용(⑧)이 분리되어 있으므로 "승인" 자체를 별도 완료 상태로 두지 않고, 승인 즉시 `APPROVED_SCHEDULED`로 전이시킨다 |
| initiated_by | 요청을 개시한 관리자 (**SuperAdmin/Compliance Admin/지정 조직관리 관리자만 가능** — role guard, [ARCHITECTURE.md](ARCHITECTURE.md) §7.1) |
| approved_by | 승인자. **`is_emergency=true`인 경우 SuperAdmin만 허용** (확정, D-022) |
| requested_at | 요청 시각 (①) |
| approved_at | 승인 시각 (⑤) — **이 시점에 `members.sponsor_id`는 아직 바뀌지 않는다** |
| **effective_date** (신규, D-022) | 적용 예정 일시 (⑥에서 확정). 기본값: 승인일 기준 익월 1일 00:00([DECISIONS.md](DECISIONS.md) O-068로 기준시점 확인 필요). 긴급 조직 이동은 `approved_at`과 동일한 값 |
| applied_at | 실제로 `members.sponsor_id`가 갱신된 시각 (⑧). 일반 건은 `effective_date` 도달 후 scheduler/worker 배치가 처리한 시각, 긴급 건은 `approved_at`과 거의 동시 |

> **승인(approved_at)과 적용(applied_at)은 서로 다른 시각이다 (확정, D-022).** `members.sponsor_id`와 `member_sponsor_history`(§3.9)는 `applied_at` 시점에만 갱신되며, `approved_at` 시점에는 `organization_transfer_logs.status`만 `APPROVED_SCHEDULED`로 바뀐다. `organization_transfer_logs` 행 자체는 이후 수정하지 않는다 — append-only.

**`organization_transfer_attachments`** — 증빙 첨부파일.

| 컬럼(개념) | 설명 |
|---|---|
| transfer_log_id | `organization_transfer_logs` 참조 |
| file_ref | Supabase Storage 참조 |
| uploaded_by / uploaded_at | 업로더(관리자)/업로드 시각 |
| description | 첨부파일 설명(예: "사망진단서", "법원판결문") |

**`organization_transfer_approvals`** — 승인/반려 이력 (단일 승인자를 넘어 다단계 승인이 필요할 경우를 대비, append-only).

| 컬럼(개념) | 설명 |
|---|---|
| transfer_log_id | `organization_transfer_logs` 참조 |
| approver_id / approver_role | 승인자 및 역할(SuperAdmin/Compliance Admin/지정 조직관리 관리자) |
| decision | 승인 / 반려 |
| comment | 승인/반려 사유 |
| decided_at | 결정 시각 |

- 위 4개 테이블 도입 여부에 대한 사용자 요청은 **"추가 검토"였으나, D-020 정책(회원 자기서비스 폐지 + 사유코드/증빙/제한된 승인권자 의무화)의 구조적 요구사항을 만족하려면 기존 `member_change_requests` 범용 테이블로는 부족하다고 판단해 전용 테이블로 확정한다.**
- 조직 이동도 다른 민감 변경과 동일하게 **과거 `commission_records`/`settlement_items`를 수정하지 않으며, 적용 시점(`applied_at`) 이후의 계산에만 영향을 준다** (확정 — 기존 원칙과 동일, D-022로 "적용 시점"이 "승인 시점"과 명확히 분리됨).
- **계산 엔진은 `organization_transfer_logs`를 직접 조회할 필요가 없다 (확정, D-022)** — `members.sponsor_id`가 `applied_at` 시점에만 갱신되므로, 수당/정산 엔진은 평소처럼 `members.sponsor_id`를 읽기만 하면 자동으로 "적용 완료된 구조만" 보게 된다. `APPROVED_SCHEDULED` 상태의 건은 `sponsor_id`에 어떤 영향도 주지 않으므로 엔진이 별도로 상태를 확인할 필요가 없다.
- **적용 배치**: `effective_date <= 현재시각`이고 `status = APPROVED_SCHEDULED`인 행을 주기적으로(scheduler가 트리거, worker가 처리) 찾아 적용한다 ([ARCHITECTURE.md](ARCHITECTURE.md) §7.1.1). 긴급 조직 이동(`is_emergency=true`)은 승인 즉시 동일한 적용 로직을 호출해 대기 없이 처리한다.
- 조직 병합/분리/복구(§5.16.8)의 데이터 모델은 기능 정의([DECISIONS.md](DECISIONS.md) O-065) 확정 후 별도로 설계한다 — 현재는 메뉴 항목만 존재하며 테이블이 없다.

### 3.27 자격 구조 (Eligibility) — 2개의 독립 자격 ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5, [DECISIONS.md](DECISIONS.md) D-028, D-024 정정)

> ⚠️ D-024 당시 이 절은 "5만원 일반제품 OR 400만원 패키지" 단일 게이트로 설계되어 있었다. **D-028로 두 자격이 완전히 분리**되었다 — 아래 §3.27.1/§3.27.2를 별개로 참조해야 한다.

#### 3.27.1 유니레벨 후원수당 자격 (월 단위, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.2)

- 회원의 월별 유니레벨 후원수당 수령 자격은 **`members`의 별도 컬럼이 아니라, 해당 캘린더월의 `orders`를 합산해 5만원 이상인지를 판정하는 파생(derived) 값**이다 — §3.11 '내 조직' 파생 데이터와 같은 패턴이다. **"최초 충족"이라는 별도 단계는 없다 — 매월 독립적으로 판정한다(D-028).**
- 패키지 구매액(400만원)도 그 달의 `orders` 합계에 포함되므로, 패키지를 구매한 달은 산술적으로 이 자격도 함께 충족된다 — **별도 충족 경로이기 때문이 아니라 단순히 구매 총액이 5만원을 넘기 때문**이다.
- 이 파생 값은 `members.status`(§3.12)와 **독립적인 차원**이다 — 두 값을 하나의 컬럼/필터로 합치지 않는다(§4 설계원칙 10과 동일한 원칙의 연장).
- **`commission_records`(§3.5) 산정 시 입력값으로 사용**: 후원수당 계산 배치는 수령 대상 회원을 선정할 때 `members.status` 필터에 더해 "산정 기간이 속한 달의 유니레벨 자격" 필터를 추가로 적용해야 한다([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §5).
- **유니레벨 라인 매출 집계 시에도 동일하게 사용**: 라인 매출 합계(§3.3, §3.5.4)는 "그 달에 이 자격을 충족한 회원"의 구매 매출만 합산하며, `package_purchases` 매출은 제외한다.
- **캐시 테이블 필요성 — 우선순위 하향(D-029)**: D-016 당시에는 후원수당이 주 단위 배치인 반면 이 자격은 월 단위라 배치마다 재집계가 필요했으나, **D-029로 후원수당 계산 자체가 월정산 단일이 되어 두 주기가 일치**한다 — 월 1회 계산 시점에 그 달의 누적 구매액을 한 번만 합산하면 되므로, 캐시 테이블(예: `member_unilevel_eligibility_monthly`) 필요성은 낮아졌다. 회원 수가 매우 커질 경우에만 재검토(§3.11과 동일 기준).

#### 3.27.2 제품 판매수익·페어보너스 자격 (1회성·영구, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.5, 신규 D-028)

- **§3.27.1과 완전히 독립된 차원**이다 — 매월 판정하지 않으며, **`package_purchases`에 `buyer_member_id = 해당 회원`인 행이 1건이라도 존재하는지**로 영구 판정한다(§3.24 참조). 재구매 여부(O-077)와 무관하게 "1건 이상 존재"만 보므로 정의가 흔들리지 않는다.
- **`package_sales_profits`/`package_pair_bonuses`(§3.24) 생성 시 입력값으로 사용** — 자격이 없으면 그 테이블에 행 자체를 생성하지 않는다(§3.24 참조).
- 캐시가 필요할 만큼 비싼 연산이 아니다(단순 EXISTS 쿼리) — 별도 캐시 테이블은 불필요할 것으로 예상되나, 회원 수 규모가 커지면 §3.27.1과 마찬가지로 재검토.

### 3.28 추천 링크 (Referral Link Tracking) ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6, [PRD.md](PRD.md) §5.17, [DECISIONS.md](DECISIONS.md) D-025)

**`referral_link_clicks`** — 추천 링크 클릭 이벤트 (append-only). 클릭 시점에는 클릭한 사람이 아직 회원이 아닐 수 있으므로(가입 전 방문자), 회원 테이블과 무관하게 독립적으로 기록한다.

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| referrer_member_id | 링크 소유 회원(`members` 참조) |
| link_type | `R_PATH`(`/r/{memberid}`) / `JOIN_QUERY`(`/join?ref={memberid}`) — §5.17.3 "링크별 가입 통계"의 분류 기준 |
| clicked_at | 클릭 시각 |
| visitor_token | 클릭 주체를 가입 전부터 가입 후까지 연결하기 위한 익명 식별자(쿠키/세션 기반) — 봇 필터링·중복 클릭 판별에도 사용 (O-074) |
| resulting_member_id | 이 클릭이 실제 가입으로 이어졌다면 그 신규 회원 id(nullable, 가입 완료 시점에 연결) — "전환율" 산출의 기준 |

- **가입 처리와의 연결**: 가입 폼이 `ref`(또는 `/r/{memberid}` 경로)를 받으면, 가입 트랜잭션 내에서 ① `members.sponsor_id`를 그 추천인으로 설정하고 ② 가능하면 직전 `referral_link_clicks` 행의 `resulting_member_id`를 채운다([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6).
- **회원/관리자 통계는 대부분 파생값**이다(§3.11/§3.27과 동일 패턴) — `members`(가입 수, `sponsor_id` 기준), `orders`/`package_purchases`(5만원 구매회원 수, 패키지 구매회원 수)에서 직접 집계하며, `referral_link_clicks`만 신규 원본 데이터가 필요하다. 전환율 = 가입 수 ÷ 클릭 수.
- **`memberid` 노출 정책(미확정, O-073)**: 링크의 `{memberid}`가 `members.id`(내부 PK)를 그대로 쓰는지, 별도의 비순차 `referral_code`를 발급해 쓰는지는 미확정 — 3가지 방식(PK노출/랜덤코드/닉네임slug) 비교와 권고안(랜덤코드)은 [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6.1 참조. 랜덤코드로 확정되면 `members.referral_code`(또는 별도 테이블) 컬럼이 추가로 필요하다.
- **봇/중복 클릭 필터링(미확정, O-074)**: `visitor_token` 기준 짧은 시간 내 중복 클릭을 어떻게 집계에서 제외할지는 미확정.
- **개인정보(미확정, O-075, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §7)**: 클릭 주체가 회원이 아닌 경우(가입 전 방문자)도 포함되므로, 기존 §7(회원가입 시 수집하는 개인정보) 체크리스트와는 별도로 **비회원 추적 데이터에 대한 처리 근거·쿠키 동의 절차**가 필요한지 검토해야 한다.

### 3.29 법적 한도(35%) 모니터링 (Compliance Ratio Monitoring) ([ARCHITECTURE.md](ARCHITECTURE.md) §8.1, [PRD.md](PRD.md) §5.18, [DECISIONS.md](DECISIONS.md) D-027)

**`compliance_thresholds`** — 국가별 모니터링 임계치 설정 (시점별 버전 관리, `marketing_plan_versions`§3.13와 동일 패턴).

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| country_code | 대상 국가 (`countries` 참조) |
| caution_threshold | 주의 임계치 — KR 확정값 30% |
| warning_threshold | 경고 임계치 — KR 확정값 33% |
| block_threshold | 차단 임계치 — KR 확정값 35%(법정 한도, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §1) |
| effective_from / effective_to | 적용 기간 |

- KR 외 국가는 자체 법정 한도가 미확정(O-045)이므로 행 자체가 없을 수 있다 — 해당 국가는 차단 게이트가 비활성 상태로 동작한다([ARCHITECTURE.md](ARCHITECTURE.md) §8.1.2).

**`compliance_ratio_snapshots`** — 실시간/월별/연도누적 비율 계산 결과 (append-only, Postgres가 source of truth).

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| country_code | 대상 국가 |
| period_type | `REALTIME_RECONCILED` / `MONTHLY` / `ANNUAL_CUMULATIVE` — `REALTIME`(Redis 추정치) 자체는 이 테이블에 저장하지 않는다(캐시는 Redis에만 존재, §8.1.1) |
| period_start / period_end | 집계 대상 기간 (연도누적은 사업연도 시작일~계산 시점) |
| total_revenue | 분모 — 누적 매출액 |
| total_sponsor_bonus_amount | 분자 — 누적 후원수당 총액. **`commission_records`의 어떤 수당 종류를 포함하는지는 O-059 분류 결과를 그대로 따른다**(현재 임시 기본값: 제품 판매수익·페어보너스 포함) |
| ratio | `total_sponsor_bonus_amount / total_revenue` |
| status | SAFE / CAUTION / WARNING / BLOCKED — `compliance_thresholds`와 비교해 도출(파생값이지만 이력 추적을 위해 저장) |
| calculated_at | 계산 시각 |
| calculation_batch_id | 계산을 수행한 worker Job 실행 id — 사후 추적용 |

- `ANNUAL_CUMULATIVE` 타입의 최신 행이 **법적 준수 여부를 판단하는 유일한 권위있는 값**이다([ARCHITECTURE.md](ARCHITECTURE.md) §8.1.1). `MONTHLY`는 추세 모니터링용 보조 지표다.
- 정산배치 검증 단계([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 ③)는 그 배치를 포함했을 때의 `ANNUAL_CUMULATIVE` 비율을 미리 계산해 `status`를 확인하고, `BLOCKED`면 배치를 다음 단계로 진행시키지 않는다([ARCHITECTURE.md](ARCHITECTURE.md) §8.1.3).
- 공제조합 보고센터([PRD.md](PRD.md) §5.7, §3.16)는 이 테이블의 `ANNUAL_CUMULATIVE` 값을 그대로 인용한다 — 별도 재집계 없음(단일 소스 원칙, [ARCHITECTURE.md](ARCHITECTURE.md) §8.1.4).
- **실시간 캐시(Redis)는 이 테이블에 직접 쓰지 않는다** — 주기적 정합화 Job이 캐시값과 실제 Postgres 집계를 비교해, 그 결과를 `period_type=REALTIME_RECONCILED`로 기록한다. 정합화 주기는 **미확정**(O-078).

### 3.30 쇼핑몰 — 정기배송 / 자동결제 ([PRD.md](PRD.md) §5.1.3, [DECISIONS.md](DECISIONS.md) D-031)

**`payment_methods`** — 회원의 자동결제 수단 (PG 토큰화 전제, 카드 원본 정보는 저장하지 않음).

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| pg_token | PG사가 발급한 결제수단 토큰 (실제 카드번호는 FNS DB에 저장하지 않음) |
| masked_card_info | 회원 표시용 마스킹된 카드 정보(예: "**** 1234") |
| is_current | 현재 사용 결제수단 여부 |
| registered_at | 등록 시각 |

**`recurring_orders`** — 정기배송 등록 (현재값 테이블 — `members.sponsor_id`와 동일하게 변경 이력은 별도 관리하지 않고 상태 전환만 기록).

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| member_id | 등록 회원 (**회원몰 전용** — §5.1.3, 일반 쇼핑몰/고객몰에는 적용하지 않음) |
| payment_method_id | `payment_methods` 참조 |
| frequency | 배송 주기(예: 매월 N일 — 정확한 주기 옵션 범위는 미확정) |
| next_delivery_date | 다음 배송(결제) 예정일 |
| status | ACTIVE / PAUSED / CANCELLED |
| created_at / cancelled_at | 등록/해지 시각 |

**`recurring_order_items`** — 정기배송에 포함된 제품 구성.

| 컬럼(개념) | 설명 |
|---|---|
| recurring_order_id | `recurring_orders` 참조 |
| product_id | `products` 참조 |
| quantity | 수량 |

**`recurring_order_payment_attempts`** — 매 주기 자동결제 시도 이력 (append-only, `notification_logs`와 동일한 시도/결과 기록 패턴).

| 컬럼(개념) | 설명 |
|---|---|
| recurring_order_id | `recurring_orders` 참조 |
| attempt_no | 재시도 회차 |
| result | 성공/실패 |
| failure_reason | 실패 시 사유(PG 응답) |
| resulting_order_id | 성공 시 생성된 `orders` 참조(nullable) |
| attempted_at | 시도 시각 |

- **처리 흐름(§1.1 원칙과 동일)**: `scheduler`가 `next_delivery_date`가 도달한 정기배송을 찾아 "정기배송 처리 Job"을 트리거하고, `worker`가 PG 결제 요청 → 성공 시 `orders` 행 생성(일반 주문과 동일 경로, §3.3) → `recurring_order_payment_attempts` 기록 → 다음 `next_delivery_date` 갱신 → 배송 트리거(§3.10 물류)를 수행한다.
- **유니레벨 자격(§3.27.1)과의 관계**: 정기배송으로 생성된 `orders`는 일반 주문과 구분 없이 그 달의 누적 구매액에 합산된다 — 정기배송 전용 별도 자격 경로는 없다. 결제가 실패하면 그 회차의 매출은 발생하지 않아 일반적인 구매 미달과 동일하게 처리된다.
- **회원 자기서비스**: 정기배송 등록/변경/해지는 §5.4의 민감 변경(승인 워크플로우) 대상이 아니다 — 조직/수당 구조에 영향을 주지 않는 일반 구매 설정이므로 회원이 즉시 셀프서비스로 처리한다.
- 결제 실패 시 재시도 정책(횟수/간격), 연속 실패 시 정기배송 자동 해지 여부, PG사 선정·토큰화 연동 방식은 **미확정**([PRD.md](PRD.md) §8 신규 항목).

## 4. 설계 원칙

1. **정산/수당 관련 테이블은 append-only.** 금액이 걸린 데이터는 직접 수정하지 않고 새로운 행으로 정정한다 ([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)).
2. **모든 계산 결과는 입력 스냅샷을 함께 저장**하여 사후 검증 가능하게 한다.
3. **소프트 삭제 우선** — 회원/제품 등 참조되는 엔터티는 물리 삭제 대신 상태값으로 비활성화.
4. **스폰서 트리 변경은 관리자 전용이다(확정, D-020)** — 변경 메커니즘(`member_sponsor_history`, §3.9)은 회원이 아닌 **관리자가 `organization_transfer_logs`(§3.26)를 통해서만** 발생시킬 수 있다. 회원이 직접 호출 가능한 API/UI 경로는 존재하지 않는다.
5. **민감 변경은 항상 요청→영향분석→승인의 3단계를 거친다** — `members` 등 현재값 테이블을 직접 수정하는 경로는 두지 않고, 반드시 `member_change_requests`(§3.9)를 통한다 ([ARCHITECTURE.md](ARCHITECTURE.md) §7).
6. **국가는 모든 계산의 입력 파라미터다** — `members.country_code`를 기준으로 §3.13의 국가별 설정(마케팅 플랜 버전/세금규칙/정산규칙)을 조회해 계산하며, 국가별 규칙을 코드에 하드코딩하지 않는다 ([ARCHITECTURE.md](ARCHITECTURE.md) §8).
7. **국가/시점별 설정은 모두 버전 관리된다** — `marketing_plan_versions` 등은 `effective_from/to`로 적용 기간을 가지며, 과거 계산 결과(`commission_records.plan_version_id` 등)는 그 시점에 유효했던 버전을 그대로 참조해 사후 재현 가능해야 한다.
8. **센터(Center)는 계산 차원이 아니라 집계 차원이다** — `members.center_id`는 후원수당/정산 계산 로직의 입력이 아니며, 오직 조회·리포팅 시 그룹화 기준으로만 사용한다 ([PRD.md](PRD.md) §5.9).
9. **규칙 변경(Rule Designer 포함)도 민감 변경과 동일한 안전장치를 따른다** — 시뮬레이션(영향분석) → 법적 한도 등 하드 제약 검증 → 승인 → 발행의 순서를 거치며, 검증을 통과하지 못한 규칙은 발행될 수 없다 (§3.23).
10. **상태 변경과 조직 구조 변경은 서로 독립적이다(확정, D-021)** — `members.status`가 WITHDRAWN/FORCED_WITHDRAWN으로 바뀌어도 `sponsor_id`는 그대로 유지되며, 둘 사이에 자동 연동(트리거)은 없다. 계산 배치(수당/정산)는 **수령·판정 대상 필터링**(status 기준)과 **트리 구조 순회**(sponsor_id 기준, status와 무관)를 별개의 단계로 구현해야 한다 — 하나의 필터로 합치면 안 된다.

## 5. 마이그레이션/ORM (미확정)

- ORM/쿼리빌더 선택(Prisma / TypeORM / Drizzle / Supabase 클라이언트 직접 사용) — 미확정.
- 마이그레이션 관리 방식(Supabase CLI migration vs ORM migration) — 미확정.

## 6. RLS (Row Level Security)

- 비즈니스 로직이 NestJS에 집중되는 구조이므로, RLS는 "최후 방어선"으로 적용할지, 아니면 비활성화하고 NestJS 권한 체크에 전적으로 의존할지 미확정 ([ARCHITECTURE.md](ARCHITECTURE.md) 참조).
- country_scope 기반 RLS 정책(`country_code = current_setting('app.country_scope')` 형태)은 비교적 단순하게 구현 가능하여, RLS 도입 시 1차 적용 후보로 검토한다 — 최종 여부는 미확정.

## 7. Database Review (설계 검토 — Design Freeze 시점)

현재까지 누적된 전체 데이터 모델(§2~§3.23)을 기준으로 누락/위험 요소를 검토한다.

### 7.1 누락 테이블 (이번 라운드에서 식별, 후속 필요)

| 항목 | 설명 |
|---|---|
| 제품의 국가별 가용성/가격 | `products`(§3.3)는 국가 구분이 없다. 국가별로 판매 가능한 제품/가격이 다를 수 있다면 `product_country_availability` 같은 테이블이 필요 — **미확정**(국가별 마케팅플랜·세금규칙은 설계했으나 제품 자체의 국가 분기는 아직 없음) |
| 환율 | 다통화 지원이 확정될 경우 `currency_exchange_rates`(시점별 환율)가 필요 — 다통화 자체가 Open Decision이라 보류 |
| 알림 수신 설정 | `notification_templates`(§3.20)는 있으나, 회원이 채널별 수신 동의/거부를 설정하는 `notification_preferences`가 없다 — 스팸 관련 법규(정보통신망법 등) 준수에 필요할 가능성 |
| 회원 유형 변경 이력 | `member_identity_profiles`(§3.14)는 현재값 위주로 설계되어 있다. 회원 유형(개인→사업자 전환 등) 자체가 변경 가능한지, 가능하다면 이력 테이블이 필요한지는 미확정 — 현재 §5.3 9종 변경 유형에 "회원유형 변경"이 없음 |

### 7.2 누락 컬럼 (기존 테이블 보강 필요)

| 테이블 | 보강 필요 컬럼 | 이유 |
|---|---|---|
| `tax_withholdings`(§3.7) | `country_code` | 현재 KR 3.3% 기준으로만 정의되어 있어, 국가별 세율(§3.13 `country_tax_rules`)과 연결할 명시적 컬럼이 없다 |
| `orders`/`order_items`(§3.3) | `country_code` | 회원의 country_code로 간접 추정 가능하지만, 주문 시점의 국가를 명시적으로 스냅샷하지 않으면 회원이 이후 국가를 변경할 경우 과거 주문의 국가를 잘못 추정할 위험 |
| `member_change_requests`(§3.9) | `idempotency_key` | Job 재시도 시 동일 요청이 중복 생성되는 것을 막기 위한 멱등성 키 — §7.3 운영 위험과 연계 |

### 7.3 성능 위험 (Performance Risk)

| 위험 | 설명 | 권고 |
|---|---|---|
| 추천조직 트리 순회 | LINE1~5 조회/계산이 회원 수 증가에 따라 재귀 쿼리로는 느려짐 (이미 closure table 권장이 있었으나, 데이터 영역이 7개 추가된 지금 더 시급함) | closure table(`member_ancestors`) 도입을 Design Freeze 이후 **1순위 구현 과제**로 격상 권고 |
| append-only 원장의 무한 증가 | `commission_records`/`audit_logs`/`member_activity_logs`/`notification_logs` 모두 시간이 지날수록 무한히 증가 | 월/연 단위 파티셔닝(PostgreSQL partitioning) 전략을 구현 단계 초기에 설계 — 사후에 적용하면 마이그레이션 비용이 커짐 |
| JSONB 컬럼 과다 사용 | `plan_definition`, `identity_fields`, `before_snapshot`, `draft_content` 등 다수 테이블이 JSON으로 유연성을 확보했으나, 인덱싱/쿼리 성능 저하 가능 | 자주 필터링되는 필드(예: `country_code`, `change_type`)는 JSON 내부가 아닌 별도 컬럼으로 추출 유지 (현재 설계는 대체로 이 원칙을 따르고 있음 — 신규 테이블 구현 시에도 동일 원칙 적용) |

### 7.4 확장성 위험 (Scalability Risk)

| 위험 | 설명 |
|---|---|
| 단일 Supabase Postgres가 OLTP+OLAP 동시 처리 | 회원 CRUD 같은 트랜잭션 처리와 '내 조직'/공제조합 보고서 같은 집계 처리가 같은 DB를 사용 — 집계 쿼리가 늘어나면 일반 서비스 응답 속도에 영향을 줄 수 있음. 리포팅/집계 전용 read replica 분리를 추후 검토 |
| 국가별 데이터 불균형 | KR이 1차로 커지고 나머지 4개국이 나중에 성장하면, 국가별 데이터 규모가 크게 달라짐 — `country_code` 기준 파티셔닝이 장기적으로 유리할 수 있음 |
| 단일 `worker`가 9개 이상의 Job 유형 처리 | 수당/정산/세금/프로모션/보고서/공제조합보고/3PL정합성/알림발송/규칙발행 시뮬레이션까지 누적되어 Job 종류가 많아짐 — [ARCHITECTURE.md](ARCHITECTURE.md) §9에서 재검토 |

### 7.5 법률 위험 (Legal Risk)

| 위험 | 설명 |
|---|---|
| 데이터 현지화(Data Localization) | 중국(CN)은 개인정보의 국외 이전을 엄격히 제한하는 법규(PIPL 등)가 있다 — 현재 설계는 단일 Supabase 리전을 전제로 하므로, CN 회원의 데이터를 동일 DB에 저장하는 것이 CN 법규를 위반할 가능성이 있다. **CN이 Reserved Country로 전환되어([DECISIONS.md](DECISIONS.md) D-023) 현재는 긴급하지 않으나, 위험 자체는 사라지지 않았다 — CN 활성화(Reserved 해제)를 검토하는 시점에 반드시 재점검 필요** |
| 회원 유형별 PII 암호화 범위 | `member_identity_profiles.identity_fields`(§3.14)가 JSON 통합 컬럼이라, 국가별로 다른 개인정보보호 규제(예: 외국인 회원의 여권정보)에 맞춰 필드별 접근 통제를 적용하기 어려울 수 있음 |
| 데이터 보존 기간 | 탈퇴/강제탈퇴 회원의 PII 보존 기간(법령상 일정 기간 보관 후 파기 의무 등)이 테이블 설계에 반영되어 있지 않음 — 파기 배치 작업 필요 |

### 7.6 운영 위험 (Operational Risk)

| 위험 | 설명 |
|---|---|
| Job 재처리 시 멱등성 | Redis Retry(§ARCHITECTURE)로 재시도되는 Job(수당 계산, 알림 발송 등)이 **중복 실행**될 경우, 중복 커미션 지급이나 중복 알림 발송이 발생할 위험. `idempotency_key` 도입을 §7.2에서 제안 |
| 백업/복구 미검증 | Supabase 자동 백업에 의존하고 있으나, 실제 복구 절차가 테스트된 적이 없음 (여전히 [DECISIONS.md](DECISIONS.md) O-037) |
| 다국가 운영 중 장애 격리 | 한 국가(예: KR)의 대량 배치 작업이 다른 국가(TH 등)의 처리를 지연시키지 않도록 격리하는 메커니즘이 현재 설계에는 없음 — worker Job 우선순위/큐 분리(O-038)와 연계 |

> 위 항목들은 모두 [DECISIONS.md](DECISIONS.md) Open Decisions 및 Design Freeze Report에 반영한다.

## 8. Open Decisions

- 금액 컬럼 타입(정수 vs decimal) 및 화폐 단위 정책
- closure table 등 트리 조회 최적화 도입 시점
- ORM/마이그레이션 도구 선정
- RLS 적용 범위
- `organization_transfer_reason_codes`의 정확한 코드값/명칭 및 사유별 필수 증빙 종류 ([DECISIONS.md](DECISIONS.md) O-067)
- 조직 병합/분리/복구 기능 정의 및 데이터 모델 (O-065)
- `organization_transfer_approvals`가 다단계 승인을 실제로 요구하는지, 단일 승인으로 충분한지
- 재가입 시 `previous_member_id` 연결 정책(신규 식별자 발급 vs 기존 레코드 재활성화)
- 계좌 변경 시 적용 보류기간(`activated_at` 지연 정책)
- 3PL 재고 동기화 방식 및 정합성 대조 주기
- `inventory_items` 파생값 캐시 필요 여부
- LINE1~5 조회를 위한 closure table(`member_ancestors`) 도입 시점 (깊이 5 고정으로 강하게 권장됨)
- '내 조직'(조직매출/조직수당/조직성장) 집계를 실시간 계산할지, 캐시/배치 집계 테이블을 둘지
- `member_identity_profiles.identity_fields`의 유형별 정확한 필드 목록 (개인정보 최소수집 원칙 검토 필요)
- 가입심사(PENDING_VERIFICATION → ACTIVE) 전환을 위한 change_type(예: `SIGNUP_VERIFICATION`) 추가 여부
- "정지(SUSPENDED)" 등 회원 상태 추가 필요 여부 및 전환 조건
- `country_tax_rules`/`country_promotions`/`country_settlement_configs`의 실제 컬럼 스키마 (각 RULES 문서 확정 후)
- `admin_roles`의 세부 역할 목록 및 권한(리소스×액션) 매핑 스키마
- `compliance_report_definitions`의 국가별 실제 보고 항목/주기
- 센터(Center) 구조 도입 여부, 도입 시 센터 변경↔국가 변경의 관계
- `documents`/회원 개인 문서(정산자료 등)의 생성-연결 방식 (시스템 생성 문서를 `documents` 테이블과 어떻게 연결할지)
- 알림 수신 동의/거부(`notification_preferences`) 테이블 필요 여부
- 회원 유형(개인/사업자/법인/외국인) 자체의 변경 가능 여부 및 변경 이력 관리 필요성
- 제품의 국가별 가용성/가격 분기(`product_country_availability`) 필요 여부
- Job 재처리 멱등성(`idempotency_key`) 도입 범위
- CN(중국) 회원 데이터의 현지화(Data Localization) 법규 준수 방안 — 단일 Supabase 리전 사용이 PIPL 등과 충돌할 가능성. **CN이 Reserved 상태(D-023)인 동안은 우선순위 낮음, 활성화 검토 시점에 재점검 필요**
- Rule Designer 도입 여부 및 적용 대상 규칙 범위
- 전자서명 방식 및 법적 효력 (국가별 전자서명법)
- append-only 원장 테이블(`commission_records`/`audit_logs` 등)의 파티셔닝 전략 및 도입 시점
- PII 보존기간 및 파기 배치 절차
- `commission_records`의 라인 단위 산정 스키마(`line_root_member_id`/`qualified_depth`/`applied_rate`/`source_member_ids`) 정확한 컬럼 설계 — 구현 단계에서 확정
- `package_sales_profits`/`package_pair_bonuses`(제품 판매수익·페어보너스, D-028 개칭)의 법적 분류(후원수당 vs 제품 판매이익)에 따른 35% 한도 산정 포함/제외 스위치 설계 (O-059)
- 월 단위 유니레벨 자격(§3.27.1)을 매 배치마다 실시간 집계할지, 캐시 테이블(`member_unilevel_eligibility_monthly`)을 둘지
- `package_purchases.pair_status`(PAIRED/EXPIRED_NO_PAIR/PENDING)를 파생값으로 둘지, 조회 성능을 위해 실제 컬럼으로 둘지 (§3.24)
- **(O-076)** 패키지 구매 자체의 선행조건(5만원 일반제품 우선구매 요구 여부) — 권고: 현재 상태(선행조건 없음) 유지
- **(O-077)** 패키지 1회성 vs 재구매 가능 여부, `package_purchases`에 `(buyer_member_id)` 유니크 제약 추가 필요성, `package_pair_bonuses` 자기매칭 방지 제약(`buyer_member_id` 상호 불일치 검증) 추가 — 권고: 자기매칭 방지는 재구매 정책과 무관하게 즉시 반영
- **(O-082, 신규)** 제품 판매수익·페어보너스 자격 미충족으로 지급되지 않은 건을 별도 로그로 남길지(회원 화면 안내용) — §3.24 참조
- **(O-081)** 제품 판매수익·페어보너스 자격(본인 패키지 구매)이 사실상 구매 강요로 작동하는지 법률 검토 — [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §5
- 추천 링크 `memberid`가 내부 PK 노출인지 별도 `referral_code`인지 (O-073 — 권고: 랜덤코드, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6.1), `referral_link_clicks` 봇/중복 클릭 필터링 정책 (O-074)
- **(O-078, 신규)** 실시간 Redis 캐시 ↔ Postgres `compliance_ratio_snapshots` 정합화(reconciliation) Job의 실행 주기 (§3.29, [ARCHITECTURE.md](ARCHITECTURE.md) §8.1.1)
- 법적 한도 초과 시 정확한 처리 방식(배치 전체보류/비례축소/초과분만보류) 확정 후 `settlement_batches`/`settlement_items`에 미칠 스키마 영향 (O-004 연계)
- TH/JP/US `compliance_thresholds` 값 확정 시점 (O-045 연계)
- `lifestyle_bonus_accumulations`의 소멸/이월 정책 및 지급 방식(현금 vs 실물)
- `organization_transfer_logs.effective_date` 익월 1일 기준 시점(승인일 vs 신청일, [DECISIONS.md](DECISIONS.md) O-068) 및 타임존(O-069)
- 조직이동 적용 배치의 실행 주기(O-070) — 매일/매시간 등 정확한 cron 설계
- 9개 사유 코드의 정확한 enum 값 및 `is_emergency_eligible` 매핑 확정 (O-067)
- **(신규, D-031)** 정기배송 결제 실패 시 재시도 정책(횟수/간격), 연속 실패 시 자동 해지 기준, PG사 선정 및 `payment_methods.pg_token` 연동 방식 (§3.30)
- **(신규, D-031)** 정기배송 배송 주기(`recurring_orders.frequency`)의 선택 가능 옵션 범위(매월 고정일 vs 회원 지정일 등)

> ~~포지션(레그) 개념의 구체적 구조~~ — **N/A로 해소** ([DECISIONS.md](DECISIONS.md) D-008, Unilevel Sponsor Plan에는 별도 포지션 개념 없음).
