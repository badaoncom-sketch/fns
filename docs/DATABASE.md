# DATABASE.md — 데이터 모델

> 상태: Draft v0.32 (D-079 — Reward Policy 고도화 및 운영 시뮬레이션: §3.64 신규(`reward_formula_versions` 1종, append-only Formula 버전 원장) + `reward_policies.active_formula_version_id` 컬럼 추가(§3.63 확장). Reward Simulation/Formula Test는 신규 테이블 없이 계산만 수행(결과 미저장), 실행 이력은 기존 `audit_logs` 재사용. 새 MLM 정책·Business Rule 없음, Settlement·ERP Core·Workflow 구조 변경 없음, O-210 등록. D-078 — Marketing Reward System 및 Lifestyle Wallet 구조 개선: §3.63 신규(`reward_policies` 1종) + §3.60 확장(`member_wallets.wallet_type`/`wallet_transactions.counts_toward_compliance_limit` 컬럼 추가, `transaction_type`에 RESTORE/EXPIRE 허용값 추가). Lifestyle Bonus 적립분의 저장 위치를 `point_transactions`(D-041)에서 `wallet_transactions`(wallet_type=LIFESTYLE_POINT)로 재라우팅 — `lifestyle_bonus_accumulations`(§3.25)의 산정 로직 자체는 변경 없음. 현금(Settlement/CASH Wallet)과 포인트(Lifestyle Point)는 절대 혼합하지 않음. 신규 MLM 정책·Business Rule 없음, O-208/O-209 등록. D-077 — ERD 동기화·API Sequence·Event Flow 완성: 신규 기능/테이블/컬럼 없음 — 본 문서는 이미 Source of Truth이므로 내용 변경 없이, [ERD.md](ERD.md)가 본 문서(§3.1~§3.62, 154개+ 엔터티)와 100% 일치하도록 동기화되었음을 확인·명시한다. [EVENT-FLOW.md](EVENT-FLOW.md)/[API-SEQUENCE.md](API-SEQUENCE.md)(신규)가 본 문서의 테이블/컬럼을 인용해 시각화했으며 충돌 없음 확인. D-076 — ERP 운영 생산성 및 관리자 UX 완성: §3.62 신규(`admin_favorite_menus`/`saved_filters`/`notification_inbox_states`/`admin_notes`/`approval_delegations` 5종뿐 — Global Search/Approval Center/Recent Activity/Personal Workspace 등 대부분은 기존 테이블 federated 조회). **O-199 해소**(즐겨찾기 메뉴/저장된 검색조건). 신규 Engine 없음, Business Rule·MLM·Settlement·ERP Core·Workflow 구조 변경 없음. D-075 — 한국 공제조합 연동·E-Wallet·글로벌 결제: §3.59~§3.61 신규(전부 Tenant별 선택 기능)) · 최종 수정일: 2026-06-27 · 단계: 설계(Design)
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
| `payment_methods` / `recurring_orders` / `recurring_order_items` / `recurring_order_payment_attempts` | 쇼핑몰 — 정기배송(구독형 반복주문)·자동결제 수단·매 주기 결제 시도 이력 (§3.30, D-031/D-033) |
| `packages` / `package_commission_policies` | 패키지 엔진 일반화 — 무제한 패키지 카탈로그 및 패키지·국가별 추천수당/페어보너스/유니레벨포함/자격부여 설정 (§3.24.1, D-033) |
| `tenants` | Multi-Tenant 구조 준비 — 테넌트(회사) 마스터, 활성화 보류 (§3.31, D-035) |
| `banners` / `lifestyle_programs` | 쇼핑몰 CMS 배너(범용) / ~~Lifestyle Program~~(§3.34로 일반화) — 쇼핑몰 메인·회원몰 메인 마케팅 노출 (§3.32, D-036) |
| `cms_pages` / `faq_categories`·`faq_items` / `popups` / `cms_translations` | CMS 대폭 확장 — 콘텐츠·공지·페이지 CMS / FAQ CMS / 팝업 CMS / 다국어 번역 오버레이 (§3.33, D-038) |
| `marketing_programs` / `marketing_program_products` / `marketing_program_applications` | Marketing Program Engine — 무제한 프로그램(Lifestyle 포함) 카탈로그·관련상품 연결·신청 워크플로우 (§3.34, D-039/D-042) |
| `product_reviews` / `product_inquiries` / `product_wishlists` / `recently_viewed_products` / `coupons` / `search_query_logs` / `content_view_events`·`content_click_events` | 쇼핑몰 기능 보강 — 리뷰/상품문의/찜/최근본상품/쿠폰/검색로그/조회·클릭 이벤트 (§3.35, D-040) |
| `point_accounts` / `point_transactions` / `point_policies` | 포인트 시스템 생애주기 — 적립/차감/사용신청/승인/취소/복원/만료 (§3.36, D-041) |
| `workflow_definitions`·`workflow_steps`·`workflow_instances`·`workflow_step_actions` | ERP Core — Workflow Engine(신규 승인 프로세스 범용 엔진, 기존 전용 구조는 유지) (§3.37, D-047) |
| `external_api_connections` / `external_api_call_logs` | ERP Core — API Center(외부 연동 일원화) (§3.38, D-048) |
| `files` / `file_versions` / `file_folders` / `file_access_permissions` | ERP Core — File Manager(통합 파일 관리) (§3.39, D-049) |
| `scheduled_job_definitions` / `scheduled_job_run_logs` | ERP Core — Scheduler Center(기존 scheduler 관리 UI) (§3.40, D-050) |
| `dashboard_definitions` / `dashboard_widgets` | ERP Core — Dashboard Builder (§3.43, D-053) |
| `report_definitions` / `report_generation_logs` | ERP Core — Report Builder (§3.44, D-054) |
| `form_definitions` / `form_submissions` | ERP Core — Form Builder (§3.45, D-055) |
| `system_security_policies` | ERP Core — System Settings(보안/운영 정책) (§3.46, D-056) |
| `crm_consultations` / `crm_consultation_reservations` / `crm_member_notes` | CRM Center(CS Center 보완) (§3.47, D-057) |
| `product_options`·`product_option_combinations` / `related_products` / `restock_notifications` / `shipping_fee_policies` | 쇼핑몰 기능 보강 Phase 2 (§3.48, D-058) |

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
- `products`의 이미지/미디어 컬럼(대표 썸네일/동영상 URL/PDF 첨부) 및 이미지 행 단위 테이블 `product_images`는 §3.50(D-060) 참조.

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

> ✅ **관리자 설정화 스키마 확정(D-032, [DECISIONS.md](DECISIONS.md) §5.10.5 gap 해소) — `package` 하위 객체는 D-033으로 제거됨**: D-005 시점 placeholder를 대체하여, D-018/D-028에서 확정된 유니레벨/자격/+알파 수치를 `plan_definition` 필드로 명시한다 — 아래 필드 외의 수치를 코드/마이그레이션에 하드코딩하지 않는다(§15 관리자 설정 원칙).
>
> ```
> plan_definition:
>   unilevel:                                  # COMPENSATION-RULES.md §3.3
>     line_rates: [3, 3, 3, 4, 5]               # 라인 도달 깊이별 단일 비율(%) — index 0=깊이1~3(3%), 3=깊이4(4%), 4=깊이5(5%)
>     max_depth: 5                              # LINE6 이상은 산정 제외
>   eligibility:                                 # COMPENSATION-RULES.md §3.5
>     unilevel_maintenance_amount: 50000        # 유니레벨 후원수당 자격 — 매월 누적 구매 기준액(원). 국가 단위 단일값(패키지와 무관)
>   lifestyle_bonus:                             # COMPENSATION-RULES.md §4.2 (옵션, 세부 수치 미확정)
>     travel_rate_range: [0.001, 0.005]
>     car_rate: 0.002
>     self_dev_rate: 0.002
> ```
>
> - ~~`package: { price, sales_profit_rate, pair_bonus_amount, pair_window_days }`~~ — **D-033으로 제거**. 패키지가 1종이라는 전제(D-024)가 깨지면서, 국가 단위 단일 객체로는 "어떤 패키지의 정책인지" 표현할 수 없게 되었다 — 이 값들은 이제 **패키지 단위**로 `package_commission_policies`(§3.24.1)에 저장된다. 마이그레이션 시 기존 `plan_definition.package` 값을 FNS의 첫 `packages`/`package_commission_policies` 행으로 옮기는 것을 권고한다.
> - 이 스키마는 `country_code`+`effective_from/to`로 버전 관리되는 `marketing_plan_versions` 행의 한 컬럼이므로, **국가별·시점별로 다른 값을 가질 수 있다** — 예를 들어 TH 시장이 다른 LINE 비율을 채택해도 코드 변경 없이 새 버전 행을 추가하는 것으로 반영한다. (D-035 Multi-Tenant 활성화 시에는 `tenant_id`도 함께 버전 키에 추가될 것으로 예상 — §3.31)
> - **35% 한도 임계치(30/33/35%)는 이 스키마에 포함하지 않는다** — 이미 별도 테이블 `compliance_thresholds`(§3.29, D-027)로 분리 설계되어 있으며, 두 설정의 변경 승인 절차가 다를 수 있기 때문이다(한도는 법적 강제값, 마케팅 플랜은 사업적 선택값).
> - `lifestyle_bonus`의 실제 수치는 여전히 **미확정**([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.2)이므로, 위 값은 원본 문서상 범위를 그대로 옮긴 placeholder다 — 사업팀 확정 후 갱신한다.

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
>
> ⚠️ **일반화 안내(D-033)**: 아래 본문은 작성 당시("패키지 1종, 400만원") 기준 설명을 그대로 보존한다(append-only). **패키지가 여러 종류로 확장된 현재는 §3.24.1을 함께 참조** — `amount`/`bonus_amount`는 고정값이 아니라 구매 시점의 상품 가격·정책 스냅샷이며, 자격 검증과 페어 큐는 패키지별로 분리된다.

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

#### 3.24.1 패키지 엔진 일반화 — 무제한 패키지 지원 ([PRD.md](PRD.md) §5.1.4, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.0~§4.1.3, [DECISIONS.md](DECISIONS.md) D-033)

> §3.24 본문은 "패키지 1종(400만원)"을 전제로 작성되었다. FNS가 다른 MLM 사업자도 사용할 수 있는 ERP 플랫폼을 지향함에 따라, **패키지 개수·정책을 무제한으로 확장 가능한 구조로 일반화**한다. §3.24의 3개 테이블은 폐기되지 않고 아래 두 테이블을 참조하도록 갱신된다.

**`packages`** — 패키지 카탈로그 마스터. 일반 쇼핑몰에 진열되는 상품 중 패키지로 분류된 것([PRD.md](PRD.md) §5.1.3.1 "패키지는 별도 쇼핑몰이 아니다") — `products`와 1:1.

| 컬럼(개념) | 설명 |
|---|---|
| product_id | `products`(§3.3) 참조, unique — 패키지명/판매가는 `products.name`/`price`를 그대로 사용하며 중복 컬럼을 두지 않는다 |
| sales_start_at / sales_end_at | 판매기간 (nullable = 상시 판매) |
| is_active | 활성 여부 |
| country_codes | 이 패키지를 판매 가능한 국가 목록 — nullable/빈 배열 = 전체 활성국 |

**`package_commission_policies`** — 패키지별 후원수당 정책. `marketing_plan_versions`(§3.13)와 동일한 국가·시점별 버전 관리 패턴을 따른다(`package_id` + `country_code` + `effective_from/to`) — 같은 패키지라도 국가별로 다른 정책을 가질 수 있다(예: 법정 한도가 다른 국가에서는 페어보너스 비활성화).

| 컬럼(개념) | 설명 |
|---|---|
| package_id | `packages` 참조 |
| country_code | 적용 국가 |
| effective_from / effective_to | 적용 기간 |
| referral_bonus_enabled | 이 패키지 구매 시 추천인에게 제품 판매수익을 지급할지 여부 |
| referral_bonus_rate / referral_bonus_amount | 비율형(가격×rate) 또는 고정금액형 — 하나만 설정. rate가 설정되어 있으면 우선 적용 |
| pair_bonus_enabled | 이 패키지 구매자끼리 페어보너스를 적용할지 여부 |
| pair_bonus_amount | Pair당 지급액 |
| pair_bonus_window_days | 페어 판정 윈도우(일) |
| **counts_toward_unilevel_line_revenue** | 이 패키지의 매출을 유니레벨 라인 매출(§3.3, §3.5.4)에 포함할지 여부 — 기본값 `false`(FNS 패키지와 동일 동작) |
| counts_toward_maintenance_purchase | 이 패키지 구매액을 구매자 본인의 그 달 유니레벨 자격(§3.27.1) 누적 구매액에 포함할지 여부 |
| **grants_qualification** | 이 패키지를 구매하면 §3.5.5 제품판매수익·페어보너스 자격을 얻는지 여부 |
| created_by / approved_by | 설정 작성자/승인자 |

> §8 Open Decision **O-088**: 한 자격 인정 패키지(`grants_qualification=true`)의 구매로 얻은 §3.5.5 자격이 **다른 모든** 자격 인정 패키지 구매자에 대한 보너스 수령에 동일하게 적용되는지(현재 가정: 그렇다 — 자격은 패키지 종류를 구분하지 않는 회원 단위 EXISTS 플래그), 아니면 패키지별로 자격 범위를 좁혀야 하는지는 **미확정**.

**§3.24 테이블의 갱신 사항(폐기 아님, 보강)**:

- `package_purchases.amount`: "구매 금액(400만원, 고정값)"이 아니라 **구매 시점 `products.price` 스냅샷**으로 재정의. `package_id`(`packages` 참조) 컬럼을 추가해 어떤 패키지의 구매인지 식별한다.
- `package_sales_profits`/`package_pair_bonuses.bonus_amount`: 고정 "100만원"/"200만원"이 아니라, **구매 시점 해당 패키지의 `package_commission_policies` 값으로 계산한 결과의 스냅샷**이다 — 이후 정책이 바뀌어도 과거 산정 결과는 변경되지 않는다(append-only 유지).
- **자격 검증 일반화**: §3.27.2의 "패키지 구매 1건 이상"은 이제 **`grants_qualification=true`인 패키지의 구매 1건 이상**으로 좁혀진다.
- **페어 큐 범위 변경**: D-026의 "추천인별 대기 후보 최대 1건"은 **추천인 + 패키지(`referrer_member_id` + `package_id`) 단위**로 좁혀진다 — 서로 다른 패키지는 페어보너스 금액·기간이 다를 수 있으므로 섞어서 매칭하지 않는다([DECISIONS.md](DECISIONS.md) O-089).
- 패키지를 추가/변경해도 위 갱신된 산정 로직은 **변경되지 않는다** — 모든 패키지가 동일한 일반화된 쿼리(해당 패키지의 `package_commission_policies` 조회)를 공유한다(D-033 확정 원칙).

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

### 3.31 Multi-Tenant 구조 준비 ([DECISIONS.md](DECISIONS.md) D-035 — 구조 설계, 활성화는 보류)

> 목적: 향후 FNS 외 다른 직접판매 회사가 동일 ERP를 사용하는 상황(예: 회사A/회사B/회사C)에 대응할 수 있는지 점검한다. **이번 라운드는 구조 준비까지만 다루며, 실제 다회사 운영 활성화는 별도 사업 결정(D-035 Open Decision)을 거친다** — Center/전자서명/Rule Designer(D-012)와 동일한 "메커니즘 설계, 활성화는 보류" 패턴을 따른다.

**`tenants`** — 테넌트(회사) 마스터. FNS는 이 테이블에 **단일 행**으로 존재하는 것으로 시작한다.

| 컬럼(개념) | 설명 |
|---|---|
| id | 테넌트 식별자 |
| name | 회사명(예: "FNS") |
| status | ACTIVE / SUSPENDED |
| created_at | 생성 시각 |

- **`tenant_id`는 `country_code`와 독립된 별개의 차원이다** — 한 테넌트(회사)가 여러 국가에서 운영할 수 있고, 이론적으로 여러 테넌트가 같은 국가에서 운영할 수도 있다. 둘을 하나로 합치지 않는다.
- **`tenant_id` 적용이 필요한 테이블 범위(향후 활성화 시)**: 회사별로 달라지는 모든 마스터/설정/트랜잭션 데이터 — `members`, `products`/`packages`/`package_commission_policies`(§3.24.1), `orders`/`order_items`, `marketing_plan_versions`, `country_tax_rules`/`country_promotions`/`country_settlement_configs`, `compliance_thresholds`, `commission_records`, `settlement_batches`/`settlement_items`, `recurring_orders` 등 §3.30 쇼핑몰 테이블, `referral_link_clicks`, `organization_transfer_logs` 등 §3.26 조직 이동 테이블, `admin_roles`/`admin_role_assignments`. 반대로 **테넌트와 무관한 순수 카탈로그성 코드 테이블**(예: `organization_transfer_reason_codes`처럼 회사를 초월한 고정 코드)은 tenant_id가 불필요할 수 있다 — 활성화 시점에 테이블별로 재검토.
- **권장 아키텍처 패턴**: 테넌트별 별도 DB/스키마가 아니라 **공유 스키마 + `tenant_id` 컬럼 + Supabase RLS(Row Level Security)** 를 권장한다 — FNS는 이미 Supabase(D-002)를 인증/DB로 채택했고 RLS 적용 범위가 Open Decision(O-020)으로 이미 추적되고 있어, 멀티테넌트 RLS 정책이 이 작업에 자연스럽게 합류한다. 테넌트별 DB 분리는 격리수준이 가장 높지만 회사 수가 늘어날 때 운영비용(Railway/Supabase 프로젝트 증가)이 선형으로 늘어나 권장하지 않는다.
- **FNS 단독 운영과의 관계**: 활성화 전까지 모든 행은 암묵적으로 단일 테넌트("FNS")에 속한다고 가정해도 무방하다 — 즉 지금 당장 모든 테이블에 `tenant_id` 컬럼을 추가하는 마이그레이션을 강제하지 않는다. 다만 **나중에 컬럼을 추가하는 것보다 처음부터 schema에 자리를 마련해 두는 것이 비용이 낮으므로**, 구현 착수 시점에 위 목록 테이블에 `tenant_id`(nullable 또는 기본값="FNS" 테넌트 id)를 함께 만드는 것을 권고한다.
- **활성화 시 추가로 필요한 것(현재는 설계하지 않음, 활성화 결정 후 별도 라운드)**: 테넌트 온보딩 플로우, 테넌트별 과금/구독 모델, 테넌트 관리자(예: SuperAdmin 위의 "Platform Admin") 역할, 테넌트 간 데이터 격리 자동 테스트.

#### 3.31.1 `tenant_settings` — 회사별 설정 ([PRD.md](PRD.md) §5.26, D-044 — "설정만으로 새 MLM 회사 생성" 목표)

| 컬럼(개념) | 설명 |
|---|---|
| tenant_id | `tenants` 참조 |
| company_name / company_logo_ref | 브랜딩 |
| domain | 회사별 접속 도메인(서브도메인 포함) |
| brand_color | UI 테마 색상 |
| default_language / default_currency / active_countries | 기본 언어/통화/활성 국가 범위(`countries`, §3.13의 부분집합) |
| terms_document_set_id | 약관 CMS(§3.18/§3.33)의 테넌트별 버전 묶음 |
| shop_config_ref / cms_config_ref / mlm_config_ref / settlement_config_ref | 쇼핑몰/CMS/MLM/정산 설정의 테넌트별 인스턴스 참조 — 각각 기존 `marketing_plan_versions`/`package_commission_policies`/`country_settlement_configs`/CMS 테이블에 `tenant_id`가 추가된 이후에는 이 컬럼들이 별도로 필요하지 않을 수 있음(중복 가능성, **미확정** — 활성화 설계 시 재검토) |

> 본 절은 D-035의 "구조 준비, 활성화는 보류" 원칙을 그대로 따른다 — 지금 `tenant_settings` 테이블을 생성하지 않으며, 활성화 결정 시 위 항목을 기준으로 스키마를 확정한다.

### 3.32 쇼핑몰 CMS 배너 / Lifestyle Program 콘텐츠 ([PRD.md](PRD.md) §5.1.5, [DECISIONS.md](DECISIONS.md) D-036)

> 목적: "+알파" 보너스(Lifestyle Bonus, §3.25)를 마이오피스 적립 현황 화면에서만 보여주던 것을 쇼핑몰 메인/회원몰 메인까지 노출하는 마케팅 콘텐츠 구조로 확장한다. §3.25(회원별 적립 원장)는 변경하지 않고, 그 위에 콘텐츠/배너 계층을 추가한다.

**`banners`** — 범용 CMS 배너. Lifestyle Program 전용이 아니라 쇼핑몰 전반의 프로모션/이벤트 배너에도 재사용한다([PRD.md](PRD.md) §5.1.5.2 "재사용성").

| 컬럼(개념) | 설명 |
|---|---|
| id | 배너 식별자 |
| name | 배너명(관리자 식별용) |
| thumbnail_image_ref | 썸네일 — Supabase Storage 참조 |
| pc_image_ref / mobile_image_ref | PC/모바일 이미지 — Supabase Storage 참조 |
| **placement** | 노출 위치 — `SHOPPING_MALL_MAIN_SLIDE`(쇼핑몰 메인 슬라이드) / `SHOPPING_MALL_PROMOTION`(프로모션 배너) / `SHOPPING_MALL_EVENT`(이벤트 배너) / `SHOPPING_MALL_CATEGORY`(카테고리 배너, 신규 — D-038) / `MEMBER_MALL_PROGRAM_BANNER`(회원몰 메인 프로그램 배너, Marketing Program 전반으로 일반화 — D-039) |
| exposure_start_at / exposure_end_at | 노출 시작일/종료일 — 조회 시점에 `now() BETWEEN exposure_start_at AND exposure_end_at`로 필터링(쿼리 타임, 별도 배치/Job 불필요) |
| link_url | 클릭 시 이동 경로 — 내부 경로(예: `lifestyle_programs` 상세페이지) 또는 외부 URL 모두 허용 |
| sort_order | 동일 `placement` 내 표시 순서 |
| is_active | 활성 여부 — 기간과 무관하게 즉시 노출 차단 가능한 수동 스위치 |
| created_by / updated_by | 관리 책임 추적 |

> 노출 조건은 **`is_active = true` AND `now()`가 노출기간 이내**일 때만 충족된다 — 둘 중 하나라도 거짓이면 노출되지 않는다.

**`lifestyle_programs`** — ~~Lifestyle Program 상세페이지 콘텐츠(CMS)~~ — **§3.34로 일반화됨(D-039)**.

> ⚠️ **일반화 안내(D-039)**: 본 테이블은 ERP 완성도 보강 라운드에서 `marketing_programs`(§3.34)로 일반화되었다 — Lifestyle Program은 무제한 마케팅 프로그램 종류 중 `category = LIFESTYLE`인 인스턴스일 뿐이라는 점이 확인되었기 때문이다(패키지가 D-033으로 일반화된 것과 동일한 이유). 아래 원본 스키마는 append-only 보존 원칙에 따라 그대로 남기되, 신규 구현은 §3.34를 따른다.
>
> | 컬럼(개념) | 설명 |
> |---|---|
> | id | 프로그램 식별자 |
> | bonus_type | `lifestyle_bonus_accumulations.bonus_type`(§3.25)과 동일한 값(TRAVEL/CAR/SELF_DEVELOPMENT) |
> | name / description / image_gallery_refs / participation_conditions / attachment_refs / is_active | §3.34의 동일 컬럼으로 대체 |
>
> - 포인트 정책 섹션은 `plan_definition.lifestyle_bonus`(D-032)를 그대로 참조한다는 원칙, FAQ는 별도 컬럼 없이 외부 참조한다는 원칙, 누적 현황은 §3.27.1과 동일한 파생 패턴으로 `lifestyle_bonus_accumulations`를 실시간 조회한다는 원칙은 **모두 §3.34에서도 변경 없이 유지**된다.
> - FAQ 연결은 D-038로 **FAQ CMS**(§3.33 `faq_items`, scope 지정)로 해소되었다(O-091).
> - bonus_type 고정 3종을 무제한으로 일반화할지의 Open Decision(O-092)은 — **본 라운드로 일반화 자체는 실행되었으나**, "+알파" 보상(MLM 후원수당류)과 연결되는 카테고리는 여전히 원본 보상플랜 문서 기준 3종으로 한정한다(§3.34 `links_to_compensation` 참조) — 보상과 무관한 신규 카테고리(Seminar/Golf 등)는 자유 생성 가능하므로 O-092는 "절반 해소"로 본다.

### 3.33 CMS 확장 ([PRD.md](PRD.md) §5.19, [DECISIONS.md](DECISIONS.md) D-038)

**`cms_pages`** — 콘텐츠 CMS / 공지 CMS / 페이지 CMS 통합 테이블.

| 컬럼(개념) | 설명 |
|---|---|
| id | 페이지 식별자 |
| page_type | `COMPANY_INTRO`/`BRAND_INTRO`/`CEO_GREETING`/`HISTORY`/`VISION`/`BUSINESS_INTRO`(콘텐츠) / `NOTICE`/`EVENT`/`NEWS`(공지) / `CUSTOM`(페이지 CMS — 관리자 자유생성) |
| slug | URL 경로 — `CUSTOM` 타입일 때 관리자가 직접 지정 |
| title | 제목 |
| body | 본문(리치 텍스트/HTML) |
| thumbnail_image_ref | 목록형(공지/이벤트/뉴스) 노출용 썸네일 — nullable |
| country_codes | 노출 국가 범위 — nullable = 전체 |
| sort_order | 정렬 순서 |
| is_active | 활성 여부 |
| published_at | 게시 시각 |
| created_by / updated_by | 작성자/수정자 |

> 기존 `documents`(§3.18)는 약관/보상플랜안내/교육자료 등 **파일 기반 버전관리 문서**에 한정한다. "공지사항"은 본 라운드부터 `documents`가 아니라 `cms_pages`(page_type=NOTICE)로 재분류한다 — 공지는 법적 버전관리가 필요한 문서가 아니라 블로그형 콘텐츠이기 때문이다. 기존 `documents`에 남아있던 공지사항 행은 마이그레이션 시 `cms_pages`로 이전을 권고한다(코드 작업이므로 구현 단계 과제).

**`faq_categories`** / **`faq_items`** — FAQ CMS(신규, O-091 해소).

| 테이블 | 컬럼(개념) | 설명 |
|---|---|---|
| faq_categories | id / name / sort_order / country_codes | 카테고리 마스터 |
| faq_items | id / category_id / question / answer / sort_order / is_active | FAQ 항목 |
| faq_items | **scope_type** | `GENERAL`(전체 공개) / `MARKETING_PROGRAM`(특정 프로그램 종속) |
| faq_items | **scope_id** | `scope_type=MARKETING_PROGRAM`일 때 `marketing_programs.id`(§3.34) 참조, `GENERAL`이면 null |

**`popups`** — 팝업 CMS(신규).

| 컬럼(개념) | 설명 |
|---|---|
| id | 팝업 식별자 |
| name | 팝업명 |
| placement | `MAIN`(메인 팝업) / `MEMBER`(회원 팝업) / `COUNTRY`(국가별 팝업) |
| country_codes | `placement=COUNTRY`일 때 적용 국가 |
| image_ref | 팝업 이미지 |
| link_url | 클릭 시 이동 경로 |
| exposure_start_at / exposure_end_at | 노출 기간 |
| display_frequency | 노출 빈도(매 방문/하루 1회/평생 1회 등) — 정확한 옵션 범위는 **미확정** |
| sort_order | 동시 노출 시 우선순위 |
| is_active | 활성 여부 |

**`cms_translations`** — 다국어 CMS(신규, 전체 CMS 콘텐츠 공통 적용).

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| content_type | 번역 대상 테이블명(예: `cms_pages`/`faq_items`/`banners`/`popups`/`notification_templates`/`marketing_programs`) |
| content_id | 대상 행의 id |
| locale | 언어 코드(예: ko/en/th/ja) |
| field_name | 번역 대상 필드명(예: title/body/question/answer) |
| translated_value | 번역된 값 |

- 기본 언어(원본 행)는 그대로 두고, 추가 언어만 본 테이블에 오버레이로 저장한다 — 번역이 없는 locale은 기본 언어로 폴백한다. 이 패턴은 신규/기존 CMS 테이블 전체에 동일하게 적용되며, 테이블마다 언어별 컬럼을 추가하지 않는다.
- **약관 CMS**는 신규 테이블이 필요 없다 — 이용약관/개인정보처리방침/국가별 약관은 기존 `documents`/`document_versions`(§3.18)를 그대로 사용하며, **마케팅 수신동의**는 `consent_history`(§3.21)의 `consent_type`에 `MARKETING_OPT_IN`을 추가하는 것으로 충족한다.
- **이메일/SMS/Push CMS**는 신규 테이블이 필요 없다 — 기존 `notification_templates`(§3.20)를 그대로 사용하며, ERP 모듈 분류상 CMS 산하로 재배치할 뿐이다.

### 3.34 Marketing Program Engine ([PRD.md](PRD.md) §5.20/§5.22/§5.24, [DECISIONS.md](DECISIONS.md) D-039/D-040/D-042 — `lifestyle_programs`(§3.32) 대체)

**`marketing_programs`** — 무제한 마케팅 프로그램 카탈로그(CMS). Lifestyle Program을 포함한 모든 프로그램 종류의 통합 테이블.

| 컬럼(개념) | 설명 |
|---|---|
| id | 프로그램 식별자 |
| program_code | 고유 코드(URL 슬러그 겸용) |
| name | 프로그램명 |
| **category** | 자유 텍스트/관리자 관리 카테고리 — 예시: Lifestyle/Promotion/Campaign/Event/Seminar/Travel/Golf/Education/VIP/Mission/Coupon. 시스템이 강제하는 고정 목록 없음 |
| country_codes / language(다국어 CMS 연동, §3.33) | 노출 범위 |
| thumbnail_image_ref / main_image_ref | 썸네일/대표 이미지 |
| detail_image_refs | 상세 이미지(이미지 갤러리) — 배열 |
| video_ref | 동영상(신규) |
| attachment_refs | 첨부 PDF |
| intro_text / detail_description | 소개글/상세 설명 |
| participation_method | 참여 방법(신규) |
| precautions | 유의 사항(신규) |
| (FAQ는 컬럼 없음) | `faq_items.scope_type=MARKETING_PROGRAM, scope_id=이 프로그램`으로 연결(§3.33) |
| exposure_start_at / exposure_end_at | 노출 기간 |
| sort_order | 정렬 순서 |
| is_active | 활성 여부 |
| **links_to_compensation** | 이 프로그램이 MLM 보상("+알파" 보너스 등)과 연결되는지 — `true`면 `plan_definition.lifestyle_bonus`/`lifestyle_bonus_accumulations`(§3.25)와 연동, `false`면 순수 마케팅/참여 콘텐츠(보상 없음 또는 §3.36 포인트 시스템으로만 연동) |
| **requires_application** | 신청 워크플로우(§3.34.1) 필요 여부 — `false`면 조회만 가능 |

**`marketing_program_products`** — 프로그램 ↔ 상품(N:N) 연결([PRD.md](PRD.md) §5.22 "관련 상품").

| 컬럼(개념) | 설명 |
|---|---|
| program_id | `marketing_programs` 참조 |
| product_id | `products` 참조(§3.3, 패키지 포함 — §3.24.1) |
| display_label | 노출 시 분류(추천상품/패키지/정기배송상품 등, 자유 텍스트) |
| sort_order | 표시 순서 |

#### 3.34.1 프로그램 신청 프로세스 ([PRD.md](PRD.md) §5.24, D-042)

**`marketing_program_applications`** — 신청 원장(append-only, 상태 전이는 행 갱신이 아니라 `status` 컬럼 업데이트 — 금액이 결부되지 않아 §4 append-only 원칙의 "금전 데이터" 수준 엄격함은 적용하지 않되, 상태변경 이력은 `audit_logs`로 추적).

| 컬럼(개념) | 설명 |
|---|---|
| id | 신청 식별자 |
| program_id | `marketing_programs` 참조(`requires_application=true`인 프로그램만 신청 가능) |
| member_id | 신청 회원 |
| status | `APPLIED`(신청) / `PENDING_APPROVAL`(승인대기) / `APPROVED`(승인) / `REJECTED`(반려) / `IN_PROGRESS`(참여중) / `COMPLETED`(완료) |
| rejection_reason | 반려 시 사유 |
| approved_by / approved_at | 승인자/승인 시각 — 관리자만 가능 |
| completed_at | 완료 시각 — 완료 시 §3.36 포인트 적립 트리거 가능(프로그램 정책에 따름) |

### 3.35 쇼핑몰 기능 보강 ([PRD.md](PRD.md) §5.21, [DECISIONS.md](DECISIONS.md) D-040)

| 테이블 | 컬럼(개념) | 설명 |
|---|---|---|
| `product_reviews` | id/product_id/member_id/order_id/rating/content/image_refs/created_at | 리뷰 — `order_id`로 구매 인증 리뷰만 허용할지는 **미확정** |
| `product_inquiries` | id/product_id/member_id/question/answer/is_secret/created_at | 상품문의 — CS Center(§3.19) 티켓과는 별개로 상품 단위로 연결 |
| `product_wishlists` | member_id/product_id/created_at | 관심상품(찜) |
| `recently_viewed_products` | member_id/product_id/viewed_at | 최근 본 상품 — 비회원 처리는 클라이언트 저장(서버 테이블 없음)을 기본값으로 권고, **미확정** |
| `coupons` / `coupon_issuances` | id/code/discount_type/discount_value/valid_from/valid_to + (issuance: member_id/coupon_id/status/used_at) | 쿠폰 — Marketing Program의 한 카테고리("Coupon")로 발행 트리거를 둘 수 있으나, 쿠폰 자체는 발급/사용/만료라는 독립 생애주기를 가지므로 **독립 엔티티로 설계**(권고, 최종 확정 필요) |
| `search_query_logs` | id/keyword/member_id(nullable)/searched_at | 검색어 순위 집계용 원시 로그 |
| `content_view_events` / `content_click_events` | id/content_type(`PRODUCT`/`MARKETING_PROGRAM`/`BANNER`)/content_id/member_id(nullable)/occurred_at | 조회수/클릭수 통계(§5.25)의 원시 이벤트 — `referral_link_clicks`(§3.28)와 동일한 "원시 이벤트 로그 + 파생 집계" 패턴 |

- 베스트/인기/추천/신상품/타임세일/브랜드관 구현에는 `products`(§3.3) 스키마 확장(브랜드/카테고리/태그/할인가/판매수량)이 선행되어야 한다 — 현재 `products`는 최소 스키마만 정의되어 있어 **후속 라운드에서 제품 카탈로그 스키마를 구체화** 필요(신규 Open Decision).

### 3.36 포인트 시스템 ([PRD.md](PRD.md) §5.23, [DECISIONS.md](DECISIONS.md) D-041)

**`point_accounts`** — 회원별 포인트 잔액(파생 캐시 — `point_transactions` 합계와 항상 일치해야 함).

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| available_balance | 사용 가능 잔액 |
| pending_balance | 사용신청 중(승인 대기)이라 사용 가능 잔액에서 분리된 금액 |

**`point_transactions`** — 포인트 원장(append-only).

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| member_id | 대상 회원 |
| transaction_type | `EARN`(적립)/`DEDUCT`(차감)/`USE_REQUEST`(사용신청)/`USE_APPROVED`(승인,사용완료)/`USE_REJECTED`(반려)/`CANCEL`(취소)/`RESTORE`(복원)/`EXPIRE`(만료) |
| amount | 금액(부호로 가감 표현 또는 별도 sign 컬럼 — 세부 미확정) |
| source_type | `LIFESTYLE_BONUS`(§3.25 확정분 전환)/`SHOPPING_REWARD`(구매적립)/`PROGRAM_COMPLETION`(§3.34.1 프로그램 완료 적립)/`MANUAL_ADJUSTMENT`(관리자 수동) |
| source_id | source_type에 따른 참조(예: `lifestyle_bonus_accumulations.id`, `orders.id`, `marketing_program_applications.id`) |
| **counts_toward_compliance_limit** | 이 포인트가 MLM 35% 법적 한도 산정에 포함되는지 — `source_type=LIFESTYLE_BONUS`는 `true`(§4.2 기존 보수적 기본값 유지), `SHOPPING_REWARD`는 `false` |
| approved_by / approved_at | `USE_APPROVED`/`USE_REJECTED`인 경우의 승인자/시각 |
| expires_at | 만료 예정일(EARN 시점에 설정, nullable=무기한) |
| related_transaction_id | `CANCEL`/`RESTORE` 시 원본 거래 참조 — 보정 엔트리 추적([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) append-only 원칙) |

**`point_policies`** — 국가·테넌트별 포인트 정책(버전 관리, `marketing_plan_versions`와 동일 패턴) — §5.27에서 발견한 하드코딩 위험 해소.

| 컬럼(개념) | 설명 |
|---|---|
| country_code / tenant_id(§3.31, D-044 활성화 시) | 적용 범위 |
| effective_from / effective_to | 적용 기간 |
| default_expiry_months | 적립 후 기본 만료 기간 |
| requires_approval_for_use | 사용신청에 관리자 승인이 필요한지(전체 공통 기본값 — 프로그램별 예외는 미확정) |

- `lifestyle_bonus_accumulations`(§3.25)는 적립의 **산정 엔진**으로 유지되며, 누적 기간 종료로 금액이 확정되는 시점에 `point_transactions`에 `EARN`(source_type=LIFESTYLE_BONUS) 행을 생성해 본 시스템으로 넘어온다 — 두 테이블의 책임이 분리된다(산정 vs 생애주기 관리).

## ERP Core (공통 엔진 계층 — [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) §1.1, [DECISIONS.md](DECISIONS.md) D-046)

> 아래 §3.37~§3.46은 모든 업무 모듈이 공통으로 사용하는 ERP Core 엔진의 데이터 모델이다. 업무 모듈 고유 테이블(§3.1~§3.36)과는 의존 방향이 반대다 — ERP Core 테이블은 업무 모듈 테이블을 참조하지 않으며, 업무 모듈이 ERP Core 테이블을 참조한다.

### 3.37 Workflow Engine ([PRD.md](PRD.md) §5.30, [DECISIONS.md](DECISIONS.md) D-047)

**`workflow_definitions`** — 관리자가 생성하는 워크플로우 템플릿.

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| name | Workflow명 |
| module_scope | 자유 텍스트 분류(예: REFUND/EXCHANGE/RETURN/GENERIC_APPROVAL) — 관리자 관리, 하드코딩 강제 없음 |
| is_active | 활성 여부 |

**`workflow_steps`** — 워크플로우 단계.

| 컬럼(개념) | 설명 |
|---|---|
| workflow_definition_id | `workflow_definitions` 참조 |
| step_order | 단계 순서 |
| step_name | |
| approver_type | `SPECIFIC_USER`/`ROLE`(Authorization 엔진의 역할 참조, §3.15) |
| approver_ref | 대상 사용자/역할 id |
| auto_approval_condition | 자동승인 조건(표현식, 문법 미확정) |
| notification_template_id | `notification_templates`(§3.20) 참조 — 단계 진입/승인/반려 알림 |

**`workflow_instances`** — 실제 신청/실행 건.

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| workflow_definition_id | 참조 |
| subject_type / subject_id | 어떤 업무 객체에 대한 워크플로우인지 — 범용 참조(예: `REFUND_REQUEST`/id) — **Workflow Engine은 subject_type이 가리키는 업무 테이블의 의미를 알지 못한다**(§5.28.1 의존방향 원칙) |
| current_step_id | |
| status | `IN_PROGRESS`/`APPROVED`/`REJECTED`/`CANCELLED` |
| initiated_by | |

**`workflow_step_actions`** — 단계별 승인/반려 이력(append-only).

| 컬럼(개념) | 설명 |
|---|---|
| workflow_instance_id / step_id | |
| actor_id | |
| decision | 승인/반려 |
| comment | |
| decided_at | |

> **기존 전용 구조(조직이동/회원변경/프로그램신청/포인트사용/정산승인)는 위 테이블로 이전하지 않는다** — [PRD.md](PRD.md) §5.30.3 참조. `organization_transfer_logs`(§3.26)/`member_change_requests`(§3.9)/`marketing_program_applications`(§3.34.1)/`point_transactions`(§3.36)는 그대로 유지된다.

### 3.38 API Center ([PRD.md](PRD.md) §5.31, [DECISIONS.md](DECISIONS.md) D-048)

**`external_api_connections`** — 외부 API 연동 등록.

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| api_name | 자유 입력 |
| category | PG/택배/3PL/SMS/EMAIL/KAKAO/PUSH/AI/COMPLIANCE_REPORTING/ERP_INTEGRATION/WEBHOOK/OTHER — 관리자 관리, 고정 enum 아님 |
| status | ACTIVE/INACTIVE/TESTING |
| **auth_key_ref** | **평문 저장 금지** — Supabase Vault 등 암호화 저장소의 참조 키만 보관([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) 원칙 확장) |
| endpoint_url | |
| is_enabled | |
| tenant_id(§3.31, D-044 활성화 시) | 테넌트별로 다른 연동을 쓸 수 있음 — `tenant_settings`(§3.31.1)가 SMTP/SMS/PG/3PL "값"을 직접 갖지 않고 본 테이블의 행을 참조하도록 일원화(O-103 해소) |

**`external_api_call_logs`** — 호출 로그(append-only).

| 컬럼(개념) | 설명 |
|---|---|
| connection_id | 참조 |
| request_summary / response_summary | 요약(전문 저장은 PII/보안 고려, 미확정) |
| status_code / is_success | |
| failed_reason | |
| retry_count | |
| called_at | |

- 기존 모듈(공제조합 보고센터 §3.16, 결제 §3.30, 물류 §3.10, Notification §3.20)이 외부 연동을 호출하던 **업무 로직 자체는 변경하지 않는다** — 연동 등록·키 관리만 API Center로 일원화하고, 각 모듈은 `external_api_connections.id`를 참조한다.

### 3.39 File Manager ([PRD.md](PRD.md) §5.32, [DECISIONS.md](DECISIONS.md) D-049)

**`files`** — 중앙 파일 레지스트리.

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| file_name / file_ref | Supabase Storage 참조 |
| folder_id | `file_folders` 참조 |
| tags | 검색용 태그 배열 |
| category | 상품/회원/법인/배너/팝업/Marketing Program/첨부파일/정산자료/원천징수/계약서 — 자유 카테고리 |
| related_entity_type / related_entity_id | 선택적 — 어떤 업무 객체에 연결되는지 |
| mime_type / file_size | |
| uploaded_by / uploaded_at | |
| is_active | |

**`file_versions`** — 버전 관리(append-only). `file_id`/`version_label`/`file_ref`/`uploaded_by`/`uploaded_at`.

**`file_folders`** — 계층형 폴더. `id`/`name`/`parent_folder_id`.

**`file_access_permissions`** — 권한. `file_id` 또는 `folder_id` / `scope_type`(ROLE/COUNTRY/MEMBER_TYPE — `document_visibility_rules` §3.18과 동일 패턴) / `scope_value`.

- **기존 `*_ref` 컬럼(`documents.file_ref`, `member_identity_profiles.document_refs`, `banners.*_image_ref` 등)은 변경하지 않는다** — File Manager는 신규 통합 등록처이며, 기존 컬럼의 마이그레이션은 강제하지 않는다([PRD.md](PRD.md) §5.32).

### 3.40 Scheduler Center ([PRD.md](PRD.md) §5.33, [DECISIONS.md](DECISIONS.md) D-050)

**`scheduled_job_definitions`** — 기존 `scheduler` 서비스가 코드로 등록한 cron Job의 메타데이터(관리자 가시성용).

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| job_name | 정산/정기배송/포인트만료/공제조합보고/Email/SMS/Push/Cache/Backup/Report 등([ARCHITECTURE.md](ARCHITECTURE.md) §2.3의 12종 Job과 대응) |
| cron_expression | 실행주기 |
| next_run_at / last_run_at / last_run_status | |
| is_enabled | 관리자가 끄고 켤 수 있음 — **단, cron 표현식 자체의 관리자 수정 가능 여부는 미확정** |

**`scheduled_job_run_logs`** — 실행 로그(append-only). `job_definition_id`/`started_at`/`completed_at`/`status`/`error_message`/`triggered_retry`.

- 본 절은 기존 Job의 **실행 로직을 변경하지 않는다** — 가시성·수동 재실행 기능만 추가한다.

### 3.41 Notification Center 보강 ([PRD.md](PRD.md) §5.34, [DECISIONS.md](DECISIONS.md) D-051 — §3.20 확장)

**`notification_templates`(§3.20) 컬럼 추가**: `version`(템플릿 버전), `version_history_ref`(이전 버전 보존).

**`notification_send_rules`** — 신규 발송 규칙.

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| template_id | `notification_templates` 참조 |
| scheduled_at | 예약발송 시각(nullable) |
| trigger_condition | 조건발송 트리거(표현식, 문법 미확정) |
| target_filter | 국가별/회원유형별/대상그룹 필터(JSON) |
| is_auto_send | 자동발송 여부 |

**`notification_logs`(§3.20) 컬럼 추가**: `failure_reason`, `retry_count`, `resent_from_log_id`(재발송 시 원본 참조).

- 실제 발송 로직(worker가 Email/SMS/KakaoTalk/Push API Center 연동을 호출)은 변경하지 않는다 — 위 컬럼/테이블은 **발송 대상·시점을 정교화하는 설정 계층**이다.

### 3.42 Audit Center ([PRD.md](PRD.md) §5.35, [DECISIONS.md](DECISIONS.md) D-052 — §3.8 점검)

- `audit_logs`(§3.8)의 기존 컬럼(행위자/모듈/행위/변경내용/시각)이 검색 요구사항(검색/필터/기간/사용자/모듈/IP/행위/변경내용)을 충분히 커버하는지 점검했다 — **IP 컬럼이 없으면 추가가 필요**하다(현재 §3.8 스키마에 IP 기록 여부는 미확정으로 남아있던 부분 — 본 라운드에서 명시적으로 플래그).
- 신규 테이블은 두지 않는다 — 다운로드/감사보고는 Report Builder(§3.44)를 재사용한다(§5.28.1 원칙 3).

### 3.43 Dashboard Builder ([PRD.md](PRD.md) §5.36, [DECISIONS.md](DECISIONS.md) D-053)

**`dashboard_definitions`** — `id`/`name`/`owner_admin_id`/`is_shared`.

**`dashboard_widgets`** — `id`/`dashboard_id`/`widget_type`(CHART/TABLE/KPI)/`data_source`(회원/매출/주문/배송/정산/수당/포인트/재고/국가/Marketing Program 등 — 업무 모듈이 노출하는 집계 엔드포인트 식별자)/`config`(JSON)/`position`(Drag & Drop 좌표, JSON).

- §5.18(35% 모니터링)·§5.25(관리자 통계)의 **고정형 대시보드는 대체하지 않는다** — 본 테이블은 그 외의 자유 구성형 대시보드용.

### 3.44 Report Builder ([PRD.md](PRD.md) §5.37, [DECISIONS.md](DECISIONS.md) D-054)

**`report_definitions`** — `id`/`name`/`output_format`(EXCEL/PDF/CSV)/`data_source`/`filters`(JSON)/`sort_config`/`schedule_cron`(예약 생성, nullable)/`schedule_email_recipients`(예약 메일 발송 대상).

**`report_generation_logs`** — 생성 이력(append-only). `report_definition_id`/`generated_at`/`status`/`file_ref`(생성된 파일은 File Manager §3.39에 등록 권고).

- Audit Center(§3.42)의 다운로드/감사보고, 공제조합 보고센터(§3.16)의 보고서 생성이 본 엔진을 재사용한다 — 각자 별도 export 로직을 두지 않는다.

### 3.45 Form Builder ([PRD.md](PRD.md) §5.38, [DECISIONS.md](DECISIONS.md) D-055)

**`form_definitions`** — `id`/`name`/`purpose`(자유 — 회원가입보조/문의/체험단/MarketingProgram/교육/이벤트/승인요청)/`fields`(JSON 배열, 각 필드: `type`(Text/Textarea/Select/Checkbox/Radio/File/Image/Date)/`label`/`validation_rules`/`required`).

**`form_submissions`** — 제출 데이터(append-only). `form_id`/`submitted_by`(member_id, nullable)/`data`(JSON)/`submitted_at`.

- **기존 회원가입 핵심 플로우(§3.14 `member_identity_profiles`)는 변경하지 않는다** — Form Builder는 보조 항목/별도 신청서용.
- Marketing Program 신청(§3.34.1)의 입력 UI로 쓰일 수 있으나, 제출 데이터는 `marketing_program_applications`에 연결되며 승인 워크플로우 자체는 변경되지 않는다.

### 3.46 System Settings ([PRD.md](PRD.md) §5.39, [DECISIONS.md](DECISIONS.md) D-056)

**`system_security_policies`** — 보안/운영 정책(국가·테넌트별 버전관리 패턴 재사용 가능).

| 컬럼(개념) | 설명 |
|---|---|
| two_factor_auth_required | 관리자 계정 2FA 강제 여부 |
| ip_whitelist | 관리자 콘솔 접근 허용 IP 배열 |
| password_policy | 최소길이/복잡도(JSON) |
| session_timeout_minutes | |
| file_upload_policy | 허용 확장자/최대 용량(JSON) — File Manager(§3.39) 업로드에 적용 |
| log_retention_days | 로그 보존 기간 |
| backup_schedule_ref | 백업 주기 — Scheduler Center(§3.40) Job 참조 |

- 회사정보/SMTP/SMS/Push/PG/3PL/API는 **신규 테이블을 두지 않는다** — API Center(§3.38)/Notification Center(§3.41)/`tenant_settings`(§3.31.1)가 이미 관리하며, System Settings는 관리자 화면의 진입점만 통합 제공한다([PRD.md](PRD.md) §5.39).

### 3.47 CRM Center ([PRD.md](PRD.md) §5.40, [DECISIONS.md](DECISIONS.md) D-057 — §3.19 CS Center 보완)

| 테이블 | 컬럼(개념) |
|---|---|
| `crm_consultations` | id/member_id/consultant_admin_id/consultation_type(전화/방문/채팅)/content/consulted_at/status(진행중/완료/Follow-up필요) |
| `crm_consultation_reservations` | id/member_id/requested_at/scheduled_at/status |
| `crm_member_notes` | id/member_id/note/created_by/created_at |

- **관심상품/회원활동은 신규 테이블을 만들지 않는다** — `product_wishlists`(§3.35)/`member_activity_logs`(§3.22)를 그대로 재사용한다(§5.28.1 원칙 3).
- `tickets`(§3.19)는 변경하지 않는다 — CRM Center는 비-티켓 상호작용(상담/예약/메모)을 보완한다.

### 3.48 쇼핑몰 기능 보강 Phase 2 ([PRD.md](PRD.md) §5.41, [DECISIONS.md](DECISIONS.md) D-058 — §3.35 Phase 1 보강)

| 테이블/컬럼 | 설명 |
|---|---|
| `products.brand_id` / `manufacturer_id`(컬럼 추가) | 브랜드/제조사 — O-098 일부 해소 |
| `product_options` / `product_option_combinations` | 상품옵션/옵션조합 |
| `related_products` | 연관상품/추천상품 — `product_id_a`/`product_id_b`/`relation_type`. `marketing_program_products`(§3.34)와는 별개(마케팅 종속 아님) |
| `restock_notifications` | 재입고알림 신청 — member_id/product_id/notified_at |
| `shipping_fee_policies` | 배송비/무료배송 정책 — country_code/free_shipping_threshold/base_fee |

- `product_inquiries`/`product_reviews`/`coupons`(§3.35, D-040)는 변경하지 않는다. 프로모션/기획전/이벤트관은 신규 엔티티 없이 `marketing_programs`(§3.34, category=Promotion/Campaign/Event)로 구현한다.

### 3.49 `tenant_settings`(§3.31.1) 확장 ([PRD.md](PRD.md) §5.42, [DECISIONS.md](DECISIONS.md) D-059)

| 컬럼(개념) | 설명 |
|---|---|
| favicon_ref | 신규 |
| smtp_connection_id / sms_connection_id / pg_connection_id / threepl_connection_id | **API Center(§3.38) `external_api_connections` 참조로 통합** — 자격증명을 `tenant_settings`가 직접 갖지 않음(O-103 해소) |
| timezone | 신규 |
| admin_email | 신규 — 테넌트 운영 담당자 연락처 |

- domain/brand_color/default_language·currency·active_countries/terms_document_set_id/company_logo_ref는 §3.31.1(D-044)에 이미 존재 — 변경 없음.

### 3.50 상품 이미지/미디어 관리 ([PRD.md](PRD.md) §5.43, [DECISIONS.md](DECISIONS.md) D-060 — §3.3 `products` 보강)

**`products`(§3.3) 컬럼 추가**

| 컬럼(개념) | 설명 |
|---|---|
| thumbnail_image_ref | 대표 썸네일 — Supabase Storage 참조, 목록/카드/검색결과 공용 |
| video_url | 동영상 URL(외부 호스팅, YouTube/Vimeo 등) — `marketing_programs.video_ref`(§3.34, 업로드형)와 달리 URL 문자열. 자체 업로드 지원 여부는 미확정(O-115) |
| attachment_refs | PDF 첨부(배열) — `marketing_programs.attachment_refs`(§3.34)와 동일 패턴 |

**`product_images`**(신규) — 정렬순서/Alt Text/PC·모바일 구분이 필요한 이미지를 행 단위로 관리한다. `marketing_programs.detail_image_refs`(§3.34, 배열 컬럼)와 달리 별도 테이블로 설계한 이유는 이 메타데이터를 배열 컬럼으로 표현할 수 없기 때문이다.

| 컬럼(개념) | 설명 |
|---|---|
| id | 식별자 |
| product_id | `products`(§3.3) 참조 |
| image_type | `LIST`(목록 보조이미지) / `DETAIL`(상세) / `INTRO`(제품소개) / `INGREDIENT`(성분표) / `USAGE`(사용방법) / `CERTIFICATE`(인증서) |
| device_type | `PC` / `MOBILE` / `COMMON` — PC·모바일 이미지 구분 노출, COMMON은 공용 |
| image_ref | Supabase Storage 참조 |
| alt_text | 접근성/SEO용 대체 텍스트 |
| sort_order | 같은 `product_id`+`image_type` 범위 내 정렬 순서 |
| is_active | 논리 삭제(소프트 삭제) — `audit_logs`(§3.8) 기록 대상 |
| created_at / updated_at | |

- 파일 용량/형식 제한은 전용 컬럼·테이블을 신설하지 않는다 — `system_security_policies.file_upload_policy`(§3.46, D-056)의 적용 범위를 상품 이미지 업로드까지 확장한다. 이미지 종류별 별도 기준(예: 인증서 PDF와 일반 이미지의 용량 기준 분리) 필요 여부는 미확정(O-116).
- 이미지 미리보기/드래그앤드롭 정렬 UI는 프론트엔드 동작이며 본 절의 스키마에 영향을 주지 않는다.
- 기존 `banners`(§3.32)/`marketing_programs`(§3.34) 이미지 컬럼 구조는 변경하지 않는다.

### 3.51 ERP UX Standard 데이터 연계 ([PRD.md](PRD.md) §5.44, [DECISIONS.md](DECISIONS.md) D-061)

본 표준은 프론트엔드(web) 정책이며 신규 핵심 테이블을 거의 요구하지 않는다 — 기존 테이블을 재사용한다.

| 항목 | 데이터 모델 |
|---|---|
| Confirm/Warning/Success/Error Dialog, Toast, Button 상태, Loading | DB 영향 없음 — web 컴포넌트 상태로만 존재 |
| Unsaved Changes Guard | DB 영향 없음 — 클라이언트 폼 상태 |
| Activity Log 연계 | 기존 `member_activity_logs`(§3.22) 재사용 — 신규 컬럼 없음 |
| Audit Log 연계 | 기존 `audit_logs`(§3.8) 재사용 — 신규 컬럼 없음 |
| Notification 연계 | 기존 Notification Center(§3.41) 재사용. 연동 트리거 기준은 미확정(O-120) |
| Undo | 기존 `is_active` 소프트 삭제 컬럼 재사용(§4 원칙 3). 복구 가능 시간(윈도우) 및 영구삭제 전환 시점은 미확정(O-117). **append-only 원장(`commission_records`/`settlement_items`)에는 적용하지 않는다** |
| Bulk Action | 기존 단건 API를 N회 호출하거나 기존 batch 엔드포인트를 재사용 — 결과는 건별로 기존 `audit_logs`에 기록. 별도 요약 테이블(`bulk_action_logs`, 대상건수/성공/실패) 신설 여부 및 동기/비동기(Job) 처리 전환 임계값은 미확정(O-118) |
| 파일 업로드 진행률/형식·용량 검증 | 기존 `system_security_policies.file_upload_policy`(§3.46) 재사용 — 신규 정책 테이블 없음 |

- 기존 5개 전용 승인 구조(조직이동/회원변경/프로그램신청/포인트사용/정산승인, D-046)의 승인 권한·순서·데이터 모델은 변경하지 않는다 — Confirm Dialog는 그 앞에 확인 단계를 추가할 뿐이다.

### 3.52 쇼핑몰 운영 고도화 — 상품/옵션 ([PRD.md](PRD.md) §5.45, [DECISIONS.md](DECISIONS.md) D-069)

> Design Freeze(D-065) 이후 New Feature 트랙으로 진행하는 쇼핑몰 운영 보강이다. MLM 패키지 엔진(§3.24.1)·정산·Workflow 기존 구조는 변경하지 않는다.

**`products`(§3.3) 컬럼 추가**

| 컬럼(개념) | 설명 |
|---|---|
| publish_start_at / publish_end_at | 예약 공개/종료 — `packages.sales_start_at/sales_end_at`(§3.24.1)와 동일 패턴 |
| sales_status | 판매상태(DRAFT/ON_SALE/SOLD_OUT/SUSPENDED/ENDED 등) — 자동전이 규칙은 BR-045 참조 |
| display_order | 진열순서(수동 정렬값) |
| search_boost_score | 검색 노출순위 가중치 |

- 상품 복사·일괄 등록/수정/삭제·Import/Export(CSV/Excel)는 신규 테이블이 필요 없다 — 복사는 애플리케이션 레벨 행 복제, 대량 처리는 기존 **O-126**(마스터데이터 대량 Import/Export, PRD §5.28/§5.32)에 종속되며 본 라운드에서 재등록하지 않는다.
- 상품 승인은 신규 전용 구조가 아니라 **Workflow Engine을 그대로 재사용**한다 — `workflow_instances.subject_type = 'PRODUCT_APPROVAL'`(§3.37)로 충분하다(D-047/BR-036 "범용 승인용"). 기존 5개 전용 승인 구조에는 추가하지 않는다.
- 상품 변경이력은 신규 테이블 없이 **`audit_logs`(§3.8)를 재사용**한다.

**`product_option_combinations`(§3.48) 컬럼 추가 — 핵심 갭**

| 컬럼(개념) | 설명 |
|---|---|
| sku_code | 옵션조합 SKU — **기존 설계 공백**: `inventory_items`/`inventory_ledger`는 개념상 SKU 단위 연결을 전제하나 옵션조합 테이블에 SKU 컬럼 자체가 없었다. 옵션-재고 연결모델 최종 확정은 미확정(O-176) |
| barcode | 옵션별 바코드 |
| price_delta 또는 price_override | 옵션별 가격(차액형/절대값형 중 택1 — `package_commission_policies`의 "비율형 또는 고정금액형, 하나만 설정" 패턴과 동일) |
| discount_price / discount_start_at / discount_end_at | 옵션별 할인 — 상품 차원 타임세일(§3.54)과 정책 통합 필요 여부는 미확정(O-185) |
| is_active | 옵션별 판매중지 |
| shipping_fee_override | 옵션별 배송비 예외(nullable) — `shipping_fee_policies`(§3.48) 기본 정책에 대한 옵션 단위 예외 |

**`product_images`(§3.50) 컬럼 추가**: `product_option_combination_id`(신규, nullable FK) — 옵션별 이미지. alt_text는 이미 존재하므로 재사용(중복 추가 없음).

### 3.53 쇼핑몰 운영 고도화 — 재고/주문/결제/배송 ([PRD.md](PRD.md) §5.45, [DECISIONS.md](DECISIONS.md) D-069)

**재고 — `inventory_ledger`(§3.10) 확장**: 신규 테이블 대신 **변동유형(type) 확장**으로 대부분 해결한다 — `INBOUND_EXPECTED`(입고예정), `ADJUSTMENT`(재고조정, `reason` 컬럼 동반), `TRANSFER_OUT`/`TRANSFER_IN`(창고간 이동, `related_transfer_id`로 두 행 연결— 쌍 일관성은 BR-047). 재고 예약(Hold)/안전재고/백오더는 이미 **O-127/O-134/O-130**으로 등록되어 있어 재등록하지 않는다.

**`inventory_lots`**(신규) — LOT(로트) 관리. 도입 여부 자체는 미확정(O-177).

| 컬럼(개념) | 설명 |
|---|---|
| warehouse_id | `warehouses`(§3.10) 참조 |
| product_option_combination_id 또는 sku_code | 대상 SKU(§3.52 O-176 해소 후 확정) |
| lot_number | 로트 번호 |
| manufactured_at / expires_at | 제조일/유통기한 |
| quantity | 로트 단위 수량 |

- `inventory_ledger.lot_id`(신규, nullable) — 로트 단위 출고 추적. LOT 우선 출고(FEFO) 규칙은 BR-048 참조.
- 입고예정(발주)을 단순 "예정 수량 표시"를 넘어 공급처/발주서 단위로 관리할지(`purchase_orders` 신설 여부)는 미확정(O-178).

**주문 운영(신규)**

| 테이블 | 설명 |
|---|---|
| `order_admin_notes` | 주문 메모/관리자 메모 — `related_ticket_id`(nullable, CS Center §3.19 티켓 참조)로 CS 메모 중복 없이 연결 |
| `shipment_change_logs` | 송장 변경/배송사 변경 통합 로그(§3.55에서도 참조) |
| `order_merge_logs` / `order_split_logs` | 주문 병합/분리 — 매출 합계 일치 규칙은 BR-049. 허용 범위는 미확정(O-179) |
| `exchange_requests` / `exchange_items` | 부분교환 — `returns`(§3.10, O-129 반품 상태머신)와 통합할지 분리할지는 미확정(O-180), 재고정합성 트랜잭션 규칙은 BR-050 |

> 부분배송/부분취소/부분환불/부분반품은 이미 **O-133/O-135/O-129**로 등록되어 있어 재등록하지 않는다.

**결제 운영(신규)**

| 테이블/컬럼 | 설명 |
|---|---|
| `order_payment_attempts` | 일반 주문 PG 실패/재결제 시도 로그(`order_id`/`attempt_no`/`pg_code`/`status`/`failure_reason`) — 정기배송 자동결제 실패/재시도는 이미 **O-086**으로 등록되어 재등록하지 않음. 가상계좌/무통장입금 도입 여부는 미확정(O-181) |
| `virtual_account_issuances` | 가상계좌 발급 내역(도입 시) |
| `bank_transfer_payments` | 무통장입금 매칭 내역(도입 시) |
| `order_payment_splits` | 복합결제(포인트+카드 등) 분할 내역 — 부분실패 처리 정책은 미확정(O-182) |
| `point_transactions.transaction_type`(§3.36) | 포인트의 결제수단화는 신규 값(`PAYMENT_USE`) 추가로 충분 — 기존 적립/사용 승인 절차(§5.23)와의 관계는 미확정(O-182에 포함) |

**배송 운영(`shipments`/`shipment_items`, §3.10 컬럼 추가)**: `bundle_group_id`(묶음배송), `scheduled_dispatch_at`(예약배송), `hold_reason`/`hold_at`(출고보류). 배송사 변경/송장 일괄 업로드는 위 `shipment_change_logs` 및 O-126(Import 종속)을 재사용한다. 묶음배송/예약배송의 데이터모델 확정 및 O-133(1주문:N배송)과의 경계는 미확정(O-183).

**`shipping_fee_settlements`**(신규) — 배송비 정산 실행/내역(`shipping_fee_policies`는 요율 정책만 정의, 실행 기록은 부재 확인). 정산 시점의 정책 버전을 스냅샷으로 고정하는 규칙은 BR-051. **MLM/후원수당 정산과는 무관**(명칭의 "정산"은 배송비 결제 영역). 도입 여부/실행주기는 미확정(O-184).

### 3.54 쇼핑몰 운영 고도화 — 리뷰/고객쇼핑/검색/프로모션 ([PRD.md](PRD.md) §5.45, [DECISIONS.md](DECISIONS.md) D-069)

**`product_reviews`(§3.35) 컬럼 추가**: `video_url`(리뷰 동영상), `admin_reply`/`admin_replied_at`/`admin_replied_by`(관리자 답변 — 스레드형으로 확장할지는 미확정 O-186), `is_best`/`best_selected_at`/`best_selected_by`(베스트 리뷰 — 선정기준은 미확정 O-187). 리뷰 이미지는 이미 `image_refs`로 존재(재사용). 리뷰 포인트는 `point_transactions.source_type`(§3.36)에 `REVIEW_REWARD` 값 추가로 충분(신규 테이블 불필요).

**`product_comparisons`**(신규) — 상품 비교(`member_id`/`product_id`/`added_at`). 비회원 처리(서버 vs 클라이언트 저장)는 미확정(O-188, 기존 O-099와 동일 패턴).

- 최근 검색은 신규 테이블 없이 `search_query_logs`(§3.35)를 회원 단위로 조회하는 것으로 충분. 재구매는 `orders`/`order_items`(§3.3) 조회 후 재주문이라 데이터 모델 변경이 필요 없다(애플리케이션 기능).
- 연관검색어는 `search_query_logs` 동시발생 분석(파생) 또는 별도 `related_search_keywords` 마스터 테이블 중 택1 — 미확정(O-189). 오타교정은 데이터 모델 불필요(애플리케이션 레벨 유사도 매칭, 정확도 임계값은 관리자 설정값 — BR-052).
- 검색어별 전환율: `search_query_logs`에 `resulted_in_purchase`/`resulting_order_id`(신규, `referral_link_clicks.resulting_member_id`(§3.28)와 동일한 "전환 연결" 패턴) 또는 §5.25 통계 모듈에서 조인 — 미확정(O-190).
- 추천상품은 이미 §5.21에서 "관리자 수동 큐레이션 또는 알고리즘(미확정)"으로 다뤄지고 있어 재등록하지 않는다.

**`product_bundles`(신규) / `product_bundle_items`(신규)** — One+One/Bundle(번들) 묶음판매. **MLM 패키지 엔진(`packages`/§3.24.1)과 명확히 별개**다(보상플랜·페어보너스와 무관한 일반 쇼핑몰 번들). Bundle 할인과 쿠폰/회원할인의 중복 적용 규칙은 미확정(O-191).

- 쿠폰/기획전/타임세일/회원할인은 이미 존재하거나 기존 Open Decision(O-099/O-113)으로 등록되어 재등록하지 않는다. 첫구매 할인/생일쿠폰은 `coupons`/`coupon_issuances`(§3.35)의 발급조건 종류 확장으로 충분 — 자동발급 트리거(Scheduler Center vs 쇼핑몰 모듈 책임)는 미확정(O-192).

### 3.55 상품/사이트 SEO 및 공유 이미지 관리 ([PRD.md](PRD.md) §5.46, [DECISIONS.md](DECISIONS.md) D-069 — O-136 구체화)

> 1차 Gap Analysis(D-062)에서 O-136으로 "SEO 메타필드 추가 여부"만 식별되어 있었다 — 본 절은 그 구체적 스키마를 설계한다. 최종 "신규 테이블 분리 vs 컬럼 추가" 확정은 여전히 미확정(O-193, 아래 권고안 참조).

**`product_seo`**(신규, `products` 1:1) — 상품은 가격/재고/리뷰평점 등 Schema.org 특화 정보가 있어 범용 테이블과 분리 권고.

| 컬럼(개념) | 설명 |
|---|---|
| product_id | PK 겸 FK, `products` 1:1 |
| seo_title / seo_description / seo_keywords | 미입력 시 자동생성(BR-053) |
| slug | 상품 고유 URL slug |
| canonical_url | 중복 URL 정규화 |
| meta_robots | INDEX_FOLLOW/NOINDEX_FOLLOW 등 |
| og_title / og_description / og_image_ref | 미입력 시 `products.thumbnail_image_ref`(§3.50) 자동 매핑 |
| twitter_title / twitter_description / twitter_image_ref | |
| schema_brand_override / schema_enabled | Schema.org Product 구조화데이터 — 가격/재고/리뷰평점은 **저장하지 않고 조회 시 `products.price`/`inventory_items`/`product_reviews` 실시간 조합**(원장 중복 방지). 실시간 조합 vs 캐시 컬럼은 미확정(O-194) |
| sitemap_priority / sitemap_change_frequency | sitemap.xml 힌트(선택, 미입력 시 시스템 기본값) |

- 이미지 alt text는 이미 `product_images.alt_text`(§3.50)로 존재 — 재사용(중복 컬럼 없음).
- robots.txt는 상품 단위가 아니라 **tenant 전역 설정**(아래)을 따른다.

**`tenant_settings`(§3.31.1/§3.49) 컬럼 추가** — Site 단위 SEO/공유 기본값.

| 컬럼(개념) | 설명 |
|---|---|
| site_title / site_description / site_keywords | |
| default_og_image_ref / default_twitter_image_ref | 상품/페이지 SEO 미입력 시 최종 fallback |
| apple_touch_icon_ref | favicon(`favicon_ref`, §3.49)과 별개 규격 |
| canonical_base_url | 멀티 도메인 운영 시 정규 도메인 |
| default_robots_txt | 사이트 전역 robots.txt — meta robots 기본값과는 별개 |

- favicon/기본언어·통화·국가는 이미 `tenant_settings`(§3.31.1/§3.49)에 존재 — 재사용.

**`tenant_share_images`**(신규) — 카카오톡/공유 이미지(용도별 다수, `tenant_settings` 컬럼 무한 추가 대신 1:N 테이블).

| 컬럼(개념) | 설명 |
|---|---|
| tenant_id | `tenants`(§3.31) 참조 |
| share_type | `KAKAO_DEFAULT`/`MAIN_URL_DEFAULT`/`PRODUCT_DEFAULT`/`EVENT_DEFAULT`/`PROMOTION_DEFAULT`/`BRAND_DEFAULT` — `marketing_programs.category`(§3.34)처럼 자유 확장 가능한 분류값, 고정 enum 강제 아님 |
| image_ref | File Manager(§3.39) 업로드 결과 참조 |
| label | 관리자 식별용 메모 |
| is_active | |

- 이미지 권장 규격(OG 1200×630px, 카카오톡 1200×630 또는 800×400, 정사각 800×800)은 **DB 제약이 아니라 UI 안내 문구로만 처리**한다(BR-054) — `system_security_policies.file_upload_policy`(§3.46)의 확장자/용량 제한과는 별개 차원이며 그 정책에 규격을 추가하지 않는다.

**`content_seo_metadata`**(신규, 범용 1:N) — 상품 외 페이지(쇼핑몰 메인/카테고리/브랜드관/기획전/이벤트/Lifestyle Program/공지사항/FAQ/회사소개/회원가입/로그인 등)의 SEO. File Manager(§3.39)의 `related_entity_type`/`related_entity_id` 패턴을 그대로 재사용한다(기존 패턴 재사용 원칙).

| 컬럼(개념) | 설명 |
|---|---|
| related_entity_type | `CMS_PAGE`/`MARKETING_PROGRAM`/`FAQ_CATEGORY` 등 |
| related_entity_id | 대상 행 id |
| seo_title / seo_description / og_image_ref / slug / canonical_url / meta_robots | `product_seo`와 동일 필드셋(상품 특화 Schema.org 필드 제외) |
| is_indexable | 콘텐츠 노출(`is_active`)과 별개로 검색엔진 노출만 제어할 필요가 있는지는 미확정 — 회원가입/로그인 페이지처럼 "콘텐츠는 노출되지만 검색엔진 비노출"이 필요한 사례 존재 |

- 다국어 SEO 메타데이터는 기존 `cms_translations`(§3.33)의 `field_name` 확장으로 흡수 가능 — 언어별로 SEO 키워드 전략 자체가 달라야 하는지는 미확정(O-193에 포함, 테이블 구조 확정과 함께 정리).
- SEO Preview(Google/Naver/KakaoTalk/Facebook/X/모바일 미리보기)는 **순수 프론트엔드(web) 렌더링이며 DB 영향이 없다** — 위 컬럼 값을 읽어 화면에 렌더링만 한다.
- sitemap.xml/robots.txt는 별도 원장 없이 위 컬럼(`is_active`/`meta_robots`/`slug`)을 조합해 **요청 시점에 동적 생성**한다(BR-054, `banners`의 노출기간 쿼리타임 필터링과 동일 패턴).

### 3.56 쇼핑몰 Dashboard/관리자 운영 보강 — 데이터 연계 ([PRD.md](PRD.md) §5.45, [DECISIONS.md](DECISIONS.md) D-069)

본 영역은 **신규 핵심 테이블을 거의 요구하지 않는다** — 기존 Dashboard Builder(§3.43)/Report Builder(§3.44)/Audit Center(§3.42)/Bulk Action 패턴(§3.51)을 재사용한다.

| 항목 | 데이터 모델 |
|---|---|
| 상품/주문/결제/배송/재고 Dashboard | Dashboard Builder(§3.43) data_source에 위젯만 추가 — 기존 트랜잭션 테이블 집계, 신규 테이블 없음 |
| 검색/SEO Dashboard, 검색어 통계, 전환율 | `search_query_logs`(§3.35) 재사용, §3.54의 전환 연결 컬럼(O-190) 확정 시 함께 위젯화 |
| 공유 클릭 통계 | `content_click_events.click_type`(§3.35)에 `SOCIAL_SHARE` 추가 검토 — 미확정 |
| 상품별 유입 경로(referrer) | `content_view_events`(§3.35)에 `referrer_source`(신규, nullable) 확장 검토 — `referral_link_clicks`(§3.28, MLM 추천링크 전용)와는 별개 트랙임을 명확히 한다. 미확정 |
| 상품 승인/히스토리/운영 로그 | `audit_logs`(§3.8) + Audit Center(§3.42) 재사용 |
| SEO 일괄수정/OG이미지 일괄업로드/alt text 일괄수정 | Bulk Action 패턴(§3.51) 재사용 — §3.55 컬럼 확정 후 그 컬럼에 대한 일괄 PATCH, 신규 계산 로직 없음 |
| 상품 Import(파일→DB 반영) | File Manager(§3.39) 업로드 + Bulk Action 조합으로 가능해 보이나, 검증 실패 행 처리 등은 **O-126**(대량 Import/Export 도입 여부)에 종속 — 별도 Job 추적 테이블 필요 여부는 O-118과 연계해 후속 확정 |

> `content_click_events`/`content_view_events`(§3.35) 컬럼 확장(공유클릭/유입경로)은 §3.54(검색/프로모션)·§3.55(SEO/공유이미지) 설계와 동일 테이블을 다루므로, 실제 컬럼명/값 도메인은 한 번에 확정한다(중복 설계 방지) — 본 절에서 통합 인지만 하고 별도 컬럼 후보를 추가하지 않는다.

### 3.57 쇼핑몰 UX/알림/운영자 대시보드 완성 ([PRD.md](PRD.md) §5.56~§5.61, [DECISIONS.md](DECISIONS.md) D-072)

> 가능한 모든 곳에서 기존 테이블을 재사용했다 — 신규 테이블은 **`carts`/`cart_items`(불가피, MVP 갭이었던 장바구니 영속화)** 와 **`product_price_alerts`(가격인하 알림, 기존 `restock_notifications`과 동일 패턴이나 트리거 종류가 달라 별도 테이블)** 2종뿐이다.

**재사용 매핑(신규 테이블 없음)**

| 기능 | 기존 테이블/엔진 |
|---|---|
| 최근 본 상품 | `recently_viewed_products`(§3.35) |
| 관심상품/찜하기 | `product_wishlists`(§3.35) |
| 상품 비교 | `product_comparisons`(§3.54, D-070) |
| 재입고 알림 | `restock_notifications`(§3.48) |
| 고객 알림 전체(주문/배송/반품/정기배송/MLM 등 이벤트 카탈로그) | `notification_templates`/`notifications`/`notification_send_rules`(§3.20/§3.41)의 `event_type` 값 목록 확장 — **신규 테이블/엔진 없음**, 신규 알림은 모두 기존 `event_type`의 카탈로그 항목 추가일 뿐 |
| Notification Template 다국어 | `cms_translations`(§3.33, `content_type='notification_templates'` 이미 지원, §3.33 비고 참조) |
| 취소/환불/교환/반품 | `orders.status`/`returns`·`return_items`(§3.10)/`exchange_requests`·`exchange_items`(§3.53, D-069) |
| 운영자 대시보드(오늘 지표/긴급 처리 지표/쇼핑몰 운영 지표) | Dashboard Builder(§3.43)/Report Builder(§3.44) 위젯 추가 — 기존 트랜잭션 테이블 집계, 신규 테이블 없음 |
| 관리자 업무 Queue | **신규 테이블을 두지 않는다.** Dashboard Builder의 위젯 종류로 "Task Queue"를 추가해, `workflow_instances`(승인대기)/`order_payment_attempts`(결제실패)/`exchange_requests`·`return_items`(승인대기)/`product_reviews`(신고)/Workflow `PRODUCT_APPROVAL`(상품승인대기)/`inventory_items`(재고부족)/CS 티켓(미답변) 등 기존 테이블의 상태값을 횡단 조회하는 **파생 뷰**로 구현한다 |
| 관리자 UX(즐겨찾기 메뉴/최근 작업/최근 본 회원·주문·상품) | `member_activity_logs`(§3.22) 재사용 — 관리자 행위도 이미 이 테이블에 기록되는 행위자 로그 패턴과 동일 |

**신규 — `carts` / `cart_items`** (불가피, 기존 설계 공백 — PRD.md §5.1.3.5가 D-034 시점에 이미 "장바구니: MVP에 추가 필요"로 식별했으나 데이터 모델이 만들어지지 않았던 항목)

| 컬럼(개념) | 설명 |
|---|---|
| `carts.id` / `member_id` | 회원 장바구니(1:1). 비회원 장바구니는 클라이언트 저장을 기본값으로 권고 — **미확정**(O-099/O-188과 동일한 "비회원 처리" 패턴) |
| `carts.updated_at` | 마지막 변경 시각 |
| `cart_items.cart_id` | `carts` 참조 |
| `cart_items.product_id` / `product_option_combination_id` | 상품/옵션(§3.52) 참조 |
| `cart_items.quantity` | 수량 |
| `cart_items.added_at` | 담은 시각 — "나중에 구매하기"는 `cart_items`를 삭제하지 않고 상태 플래그로 구분할지, `product_wishlists`로 이동시킬지는 **미확정**(O-197) |
| 품절/가격변경 표시 | 신규 컬럼 없음 — 조회 시점에 `product_option_combinations.is_active`/`price_delta`(§3.52)와 조합해 파생 표시 |
| 배송비 예상 표시 | 신규 컬럼 없음 — `shipping_fee_policies`(§3.48) 조회 결과를 파생 표시 |
| 쿠폰 적용 가능 여부 | 신규 컬럼 없음 — `coupons`(§3.35) 발급조건 조회 결과를 파생 표시 |

**신규 — `product_price_alerts`** (가격인하 알림 — `restock_notifications`와 동일한 "상품 알림 신청" 패턴이나 트리거가 재고가 아닌 가격이라 별도 테이블)

| 컬럼(개념) | 설명 |
|---|---|
| member_id / product_id | 알림 신청 대상 |
| target_price 또는 알림 기준(현재가 대비 %) | 트리거 조건 — **미확정**(O-198) |
| notified_at | 알림 발송 시각(nullable) |

**`shipments`/`shipment_items`(§3.10) 컬럼 명료화 — 배송 추적 UX 지원**: 기존 절은 "배송 상태(추적)"라고만 개념 기술되어 있었다. 배송 추적 UX(택배사/송장번호/배송상태/배송 타임라인)를 위해 `courier_name`/`tracking_no`/`status`(준비중/출고완료/배송중/배송완료/지연/보류)/`status_updated_at`을 명시적으로 확인한다 — **신규 테이블이 아니라 기존 테이블의 컬럼 존재를 명문화**하는 것이다. 택배사 배송조회 링크는 `courier_name` 기준 URL 패턴(고정 매핑, DB 불필요).

**`notification_templates`(§3.20) 컬럼 추가**: `subject_template`(제목, EMAIL 채널용 — SMS/PUSH는 nullable), `is_active`(활성/비활성 토글). 변수({{member_name}} 등)는 `content_template`/`subject_template` 문자열 내 placeholder 문법으로 처리 — 별도 변수 테이블 불필요(어떤 변수가 사용 가능한지는 `event_type`별 안내 문서로 관리, DB 강제 아님).

- **저장된 검색조건(관리자 UX)은 본 라운드에서 테이블을 만들지 않는다** — 빈도·필요성이 상대적으로 낮아 "불가피한 경우에만 신규 테이블"(D-072 원칙) 기준에 못 미친다고 판단했다. 도입 여부는 **O-199**로 등록한다.

### 3.58 Dynamic Board Engine ([PRD.md](PRD.md) §5.67, [DECISIONS.md](DECISIONS.md) D-074)

> **기존 CMS(`cms_pages`/`faq_categories`·`faq_items`/`popups`/`banners`/`documents`, §3.33)는 변경하지 않는다.** 본 절은 그 옆에 병렬로 존재하는 **신규 범용 게시판 엔진**을 설계한다 — 관리자가 새로운 게시판(보도자료/갤러리/자료실/이벤트/홍보영상/교육자료/제품자료/인증자료/사용후기/회사소개/CSR/채용/IR 등)을 코드 배포 없이 직접 생성할 수 있도록 한다. **게시판 유형(Board Type)마다 별도 테이블을 만들지 않는다** — `boards`/`board_posts` 공통 구조 + `board_type` 분류값 + `metadata`(JSON) 확장으로 모든 유형을 수용한다.

**`boards`** — 게시판 정의(신규).

| 컬럼(개념) | 설명 |
|---|---|
| id | 게시판 식별자 |
| name | 게시판명 |
| code | 게시판 코드(URL/API 식별자, 관리자 지정, unique) |
| description | 게시판 설명 |
| board_type | `GENERAL`/`NOTICE`/`GALLERY`/`FAQ`/`ARCHIVE`(자료실)/`PRESS`(보도자료)/`EVENT`/`VIDEO` 등 — `marketing_programs.category`(§3.34)와 동일한 **자유 확장 분류값**(고정 enum 강제 아님, 관리자가 새 유형명을 추가할 수 있음) |
| layout_type | `LIST`/`CARD`/`GALLERY`/`FAQ`/`VIDEO` — 목록 화면 레이아웃(§5.67.6), board_type과 독립적으로 선택 가능 |
| is_active | 활성/비활성 |
| menu_exposure | 메뉴 노출 여부 |
| shop_exposure / shop_member_exposure / my_office_exposure / main_exposure | 쇼핑몰/회원몰/마이오피스/메인 노출 여부 — 각각 독립 토글 |
| menu_group | 어느 정적 메뉴 그룹(예: "회사소개") 하위에 노출되는지 — 참조 무결성 없는 자유 텍스트(기존 SITEMAP.md 메뉴 트리와의 매핑은 운영 단계 설정) |
| sort_order | 노출 순서 |
| country_codes | 사용 국가 — nullable = 전체(기존 패턴) |
| language_codes | 사용 언어 — nullable = 전체 |
| feature_flags | JSON — 댓글/답글/파일첨부/이미지/대표이미지/동영상/다운로드/조회수/좋아요/공유/SEO/OG/예약게시/승인후게시/카테고리/태그/검색/RSS 각각의 ON/OFF(§5.67.4). 컬럼을 20개 추가하는 대신 JSON 1개로 묶어 신규 기능 토글 추가 시 스키마 변경 없이 확장 가능 |
| created_by / updated_by / created_at / updated_at | |

**`board_categories`** — 게시판별 카테고리(신규, `boards.feature_flags.category=true`일 때만 사용).

| 컬럼(개념) | 설명 |
|---|---|
| id / board_id | 소속 게시판 |
| name / sort_order | |

**`board_posts`** — 게시글 공통 구조(신규). `cms_pages`(§3.33)와 의도적으로 유사한 형태를 취하되 별도 테이블이다(기존 CMS 비변경 원칙).

| 컬럼(개념) | 설명 |
|---|---|
| id / board_id | 소속 게시판 |
| category_id | `board_categories` 참조 — nullable |
| title / content | 제목/본문(리치 텍스트) |
| thumbnail_image_ref | 대표 이미지 — File Manager(`files`, §3.39) 참조 |
| slug | URL 경로(관리자 지정 또는 자동생성) |
| status | `DRAFT`/`SCHEDULED`/`PENDING_APPROVAL`/`PUBLISHED`/`UNPUBLISHED` |
| scheduled_publish_at | 예약게시 시각 — nullable, `feature_flags.예약게시=true`일 때 사용 |
| published_at | 실제 게시 시각 |
| is_public | 공개/비공개 |
| tags | JSON 배열 — 태그(전용 마스터/조인 테이블 대신 배열로 단순화, 태그클라우드·분석이 필요해지면 후속 라운드에서 전용 테이블 검토) |
| metadata | JSON, nullable — **유형별 특수 필드 확장 포인트**(§14 원칙). 예: VIDEO 유형의 `video_url`, FAQ 유형의 `answer`(질문은 title 재사용) — 신규 컬럼 추가 대신 이 안에 담는다 |
| view_count_cache / like_count_cache | 조회수/좋아요 캐시(파생값, 원본은 `content_view_events`/`board_post_likes`) — 목록 화면 성능을 위한 비정규화, 신뢰 가능한 값은 항상 원본 집계 |
| created_by / updated_by / created_at / updated_at | |

**`board_post_comments`** — 댓글(신규, `boards.feature_flags.댓글=true`일 때만 사용). 답글은 별도 테이블이 아니라 **자기참조**로 표현한다(§5.67.5).

| 컬럼(개념) | 설명 |
|---|---|
| id / post_id | 소속 게시글 |
| parent_comment_id | nullable — 값이 있으면 답글, null이면 최상위 댓글 |
| member_id | 작성 회원 |
| content | |
| is_active | 소프트 삭제 |
| created_at | |

**`board_post_likes`** — 좋아요(신규, `boards.feature_flags.좋아요=true`일 때만 사용).

| 컬럼(개념) | 설명 |
|---|---|
| post_id / member_id | 복합 unique(회원당 게시글당 1회) |
| created_at | |

**기존 구조 재사용 매핑(신규 테이블 최소화)**

| 기능 | 재사용 대상 |
|---|---|
| 첨부파일/이미지/다중이미지/PDF/동영상 파일 | File Manager `files`(§3.39, `related_entity_type='board_posts'`/`related_entity_id`) — 이미 다형 참조 패턴이라 신규 테이블 불필요 |
| SEO/OG/Canonical | `content_seo_metadata`(§3.55, D-069, `related_entity_type='board_posts'`) — 이미 범용 SEO 테이블 |
| 다국어 | `cms_translations`(§3.33, `content_type='board_posts'`/`'boards'`) — 이미 범용 번역 오버레이 |
| 조회수 | `content_view_events`(§3.35, `content_type='board_posts'`) — 이미 범용 조회 이벤트, `board_posts.view_count_cache`는 이 집계의 파생 캐시일 뿐 |
| 공유/다운로드 통계 | `content_click_events`(§3.35, `click_type` 확장) — 공유클릭(SOCIAL_SHARE)/다운로드 구분은 이미 미확정으로 추적 중(**O-194**, 재등록하지 않음) |
| 검색 | `search_query_logs`(§3.35) 패턴 재사용 또는 구현 단계 전문검색(풀텍스트 인덱스) — 신규 테이블 불필요 |
| RSS | 신규 저장 없음 — `boards.feature_flags.RSS=true`인 게시판의 `board_posts`(status=PUBLISHED)를 요청 시점에 RSS XML로 동적 생성(BR-054의 sitemap.xml과 동일한 쿼리타임 파생 패턴) |
| 승인후게시 | Workflow Engine(§3.37) 재사용 — `workflow_instances.subject_type='BOARD_POST_APPROVAL'`(기존 자유 카탈로그 값 추가, PRODUCT_APPROVAL과 동일 패턴) — 신규 전용 승인 구조 없음 |
| 예약게시 실행 | `board_posts.scheduled_publish_at` 도달 시 `status: SCHEDULED → PUBLISHED` 전이 — Scheduler Center(§3.40) 기존 Job 패턴 재사용(products.publish_start_at, §3.52와 동일 원리) |

- **권한(§5.67.7)은 신규 테이블이 필요 없다** — [ROLE-MATRIX.md](ROLE-MATRIX.md)의 기존 역할 체계(7역할×11액션)를 게시판 모듈에 그대로 적용한다.
- **기존 5개 CMS 콘텐츠(공지사항/FAQ/팝업/배너)를 Board Engine으로 마이그레이션할지 여부는 본 라운드에서 결정하지 않는다** — "기존 CMS를 변경하지 않는다"는 본 라운드 원칙에 따라 현재 구조를 그대로 유지하며, 통합 여부는 **O-200**으로 등록한다.

### 3.59 한국 공제조합 연동 보강 ([PRD.md](PRD.md) §5.68, [DECISIONS.md](DECISIONS.md) D-075 — §3.16 확장)

> 기존 §3.16(`compliance_report_definitions`/`compliance_report_submissions`)은 "보고서 단위" 생성/제출만 다뤘다 — 본 절은 **회원/후원관계/매출/수당/환불/반품/취소를 항목 단위로 전송·추적**하는 운영 레이어를 추가한다. **Tenant별 선택 기능**이며, 신규 연동 엔진을 만들지 않고 API Center(§3.38)/Scheduler Center(§3.40)/Audit Center(§3.42)/Report Builder(§3.44)를 재사용한다.

**연동 등록**: 신규 테이블 없음 — `external_api_connections`(§3.38, `category='COMPLIANCE_REPORTING'`)의 행으로 등록한다. `api_name`에 "직접판매공제조합"/"한국특수판매공제조합" 등을 자유 입력(고정 enum 아님, 동시에 둘 다 등록 가능). `tenant_id`(§3.38 기존 컬럼)로 고객사별 사용/미사용·연동 대상 선택을 표현한다 — 테스트/운영 모드는 기존 `status`(TESTING/ACTIVE/INACTIVE)를 그대로 사용, API Key/인증정보는 기존 `auth_key_ref`(Vault 참조, 평문 금지) 재사용.

**`compliance_report_definitions`(§3.16) 컬럼 추가**: `tenant_id`(nullable — 테넌트별 보고서 정의 차등), `auto_transmit`(자동 전송 여부)/`manual_transmit_allowed`(수동 전송 허용 여부). 보고 주기(`frequency`)는 기존 컬럼 그대로(정확한 옵션 범위는 기존부터 미확정).

**`compliance_member_registrations`**(신규) — 회원별 공제조합 등록 정보.

| 컬럼(개념) | 설명 |
|---|---|
| member_id | 대상 회원 |
| connection_id | `external_api_connections` 참조(어느 공제조합인지) |
| registration_number | 공제번호 |
| certificate_file_ref | 공제증서 — File Manager(`files`, §3.39) 참조, 신규 파일 테이블 없음 |
| status | 등록대기/등록완료/등록실패/해지 |
| registered_at | |

**`compliance_transmission_items`**(신규) — 전송 대상 항목별 추적(회원/후원관계/매출/수당/환불/반품/취소).

| 컬럼(개념) | 설명 |
|---|---|
| connection_id | 어느 공제조합 연동인지 |
| item_type | MEMBER/SPONSOR_RELATION/REVENUE/COMMISSION/REFUND/RETURN/CANCEL — 자유 확장값 |
| source_type / source_id | 범용 참조(예: `members`/`orders`/`commission_records`/`returns`) — File Manager와 동일한 다형 참조 패턴 |
| period | 대상 기간(해당하는 경우) |
| status | 대기/전송중/성공/실패 |
| failure_reason | |
| retry_count / last_attempted_at | 재전송 추적 |
| call_log_id | `external_api_call_logs`(§3.38) 참조 — 실제 HTTP 호출 1건과 연결(재시도마다 새 호출 로그) |

- **전송 로그/PG 응답과 동일한 패턴**: 실제 호출 시도·성공/실패·재시도 횟수는 기존 `external_api_call_logs`(§3.38)를 그대로 사용한다 — `compliance_transmission_items`는 "이 비즈니스 항목이 전송됐는가"를, `external_api_call_logs`는 "그 전송 시도가 어떻게 됐는가"를 각각 책임진다(1:N).
- **관리 기능**(전송대상조회/전송상태/성공실패/재전송/수동전송/전송로그/보고서 다운로드)은 모두 위 두 테이블 + 기존 Audit Center/Report Builder 조회 화면으로 충분하다 — 신규 화면 엔진 없음.
- **자동 전송**은 Scheduler Center(§3.40)의 기존 Job 패턴(`job_name`에 "공제조합 항목전송" 추가)을 재사용한다.

### 3.60 전자지갑 (E-Wallet) ([PRD.md](PRD.md) §5.69, [DECISIONS.md](DECISIONS.md) D-075)

> Tenant별 선택 기능. **반드시 append-only Ledger 구조** — 잔액은 항상 원장에서 파생하며 직접 UPDATE하지 않는다(`commission_records`/`settlement_items`와 동일한 원칙, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)). 포인트(`point_transactions`, §3.36)와는 **별개의 시스템**이다 — 포인트는 그대로 유지하고, 지갑은 신규 병행 구조로 둔다(전환 여부는 정책 미확정, 아래 참조).

**`member_wallets`** — 회원별 × 통화별 지갑(1:N, 회원 1명이 통화별로 여러 지갑을 가질 수 있음).

| 컬럼(개념) | 설명 |
|---|---|
| id / member_id | |
| currency_code | KRW/USD/THB/JPY 등 — `marketing_programs.category`와 동일한 **자유 확장값**(고정 enum 아님, 신규 통화 추가 시 스키마 변경 불필요) |
| status | ACTIVE/LOCKED — 관리자 지갑 잠금/해제(§5.69 관리자 기능) |
| available_balance_cache / pending_balance_cache / withdrawable_balance_cache / used_balance_cache / hold_balance_cache | **파생 캐시** — 신뢰 가능한 원본은 항상 `wallet_transactions` 집계. 목록 화면 성능을 위한 비정규화(§3.57 `board_posts.view_count_cache`와 동일 원칙) |
| created_at | |

**`wallet_transactions`** — append-only 원장(Ledger). 잔액 직접 수정 금지, 모든 변경은 새 행 추가로만 표현한다.

| 컬럼(개념) | 설명 |
|---|---|
| id / wallet_id | |
| transaction_type | CHARGE(충전)/EARN(적립)/USE(사용)/CANCEL(취소)/REFUND(환불)/WITHDRAWAL_REQUEST(출금신청)/WITHDRAWAL_COMPLETED(출금완료)/ADJUSTMENT(보정)/HOLD(보류)/RELEASE(해제) |
| amount | 부호로 증감 표현(양수=증가, 음수=감소) |
| balance_type_affected | available/pending/withdrawable/used/hold 중 영향받는 잔액 종류 |
| source_type / source_id | 범용 참조 — EARN이면 `commission_records`/`settlement_items` 참조(수당 적립 시 정산 결과를 그대로 인용, **정산 계산 로직 자체는 건드리지 않음**), USE면 `orders` 참조 등 |
| reason | ADJUSTMENT/HOLD/RELEASE일 때 사유 기록(필수) |
| created_by | 시스템(배치) 또는 관리자(수동 보정) |
| created_at | |

**`wallet_withdrawal_requests`** — 출금 신청 워크플로우(신규 승인 구조 아님, Workflow Engine 재사용).

| 컬럼(개념) | 설명 |
|---|---|
| id / wallet_id / member_id | |
| amount | 신청 금액 |
| bank_account_ref | 출금 계좌(기존 회원 계좌 정보 참조 또는 1회성 입력 — 정확한 방식 미확정) |
| status | REQUESTED/APPROVED/REJECTED/COMPLETED |
| workflow_instance_id | `workflow_instances`(§3.37, `subject_type='WALLET_WITHDRAWAL'`) 참조 — **신규 승인 구조를 만들지 않고 Workflow Engine 재사용** |
| requested_at / processed_at / processed_by | |

- **상태 전이마다 `wallet_transactions`에 대응 원장 행이 생긴다**: REQUESTED → HOLD(가용잔액 차감, 보류잔액 증가) / REJECTED → RELEASE(보류 해제, 가용잔액 복원) / COMPLETED → WITHDRAWAL_COMPLETED(보류잔액 소진).
- **정산(Settlement)과의 경계**: 후원수당의 1차 산정·법적 한도 검증·세금 계산(③~④단계, [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9)은 **전혀 변경하지 않는다.** 지갑 적립(EARN)은 정산이 이미 확정한 `settlement_items` 금액을 그대로 인용하는 **추가 지급 채널(은행송금 대신 또는 더불어)**일 뿐이다 — 정산 금액 계산 자체에는 영향을 주지 않는다. 은행송금/지갑적립 분배 정책(전액/선택/병행)은 **O-201**(아래)로 미확정 등록.
- **포인트(`point_transactions`)와 지갑(`wallet_transactions`) 간 전환 여부**는 정책 미확정 — **O-202**.
- **쇼핑몰 결제 시 지갑/포인트 우선순위**는 정책 미확정 — **O-203**.
- **`member_wallets`/`wallet_transactions`는 D-078(§3.63)에서 `wallet_type`(CASH/LIFESTYLE_POINT) 컬럼이 추가되어 Lifestyle Point도 같은 구조로 수용한다 — 본 절(현금성 E-Wallet)의 기존 컬럼/의미는 변경되지 않는다.** 상세는 §3.63 참조.

### 3.61 글로벌 결제 ([PRD.md](PRD.md) §5.70, [DECISIONS.md](DECISIONS.md) D-075)

> Tenant별 선택 기능. 한국 결제(신용카드/계좌이체/가상계좌/무통장입금)는 이미 §3.53(D-069, `order_payment_attempts`/`virtual_account_issuances`/`bank_transfer_payments`/`order_payment_splits`)이 다룬다 — **재등록하지 않는다.** 본 절은 (1) PG를 국가별로 스코프하는 컬럼 1개 추가, (2) 글로벌 PG(태국 PromptPay/일본 신용카드/Stripe/PayPal) 등록, (3) 인바운드 Webhook 수신만 신규로 다룬다.

**`external_api_connections`(§3.38) 컬럼 추가**: `country_code`(nullable) — PG 연동을 국가별로 스코프(예: 태국 PromptPay 연동은 `country_code='TH'`). 기존 `tenant_id`와 결합해 "어느 테넌트가 어느 국가에서 어느 PG를 쓰는지"를 표현한다. Test/Live 모드는 기존 `status`, API Key/Secret Key는 기존 `auth_key_ref`(복수 키가 필요하면 JSON으로 확장 검토— 미확정), Webhook URL은 기존 `endpoint_url` 재사용.

**`payment_webhook_events`**(신규) — PG가 보내는 **인바운드** Webhook 수신 로그(append-only). 기존 `external_api_call_logs`(§3.38)는 **우리가 PG를 호출한** 아웃바운드 기록이라 인바운드 수신과는 방향이 반대 — 별도 테이블이 불가피하다.

| 컬럼(개념) | 설명 |
|---|---|
| connection_id | 어느 PG 연동인지 |
| event_type | 결제승인/결제실패/취소/부분취소/환불/부분환불/정기결제성공/정기결제실패 등 — PG사별로 다른 원본 이벤트명을 정규화(매핑 규칙은 미확정) |
| raw_payload_ref | 원문 페이로드 — PII/보안 고려해 File Manager 또는 별도 암호화 저장 검토(미확정) |
| signature_verified | Webhook 서명 검증 통과 여부 |
| related_order_id | 매칭된 주문(매칭 실패 시 null, 수동 매칭 필요) |
| status | 수신/처리중/처리완료/처리실패 |
| received_at / processed_at | |

- **자동결제(정기결제)**: 기존 `recurring_order_payment_attempts`(§3.30)/`payment_methods.pg_token`(PG 무관 토큰화 패턴, §3.30)을 그대로 재사용 — 글로벌 PG 토큰도 동일 컬럼에 저장한다. 정확한 토큰화 방식은 기존 **O-087**로 이미 추적 중(재등록하지 않음).
- **Idempotency Key**: 기존 **O-054**(`idempotency_key` 도입 범위 미확정)에 결제 영역도 포함된다 — 재등록하지 않음.
- **부분취소/부분환불**: 주문 레벨 처리는 이미 §3.53(O-133/O-135/O-129)이 다룬다. 본 절은 PG로 보내는 **요청** 자체만 다루며, 그 요청·응답은 `external_api_call_logs`(아웃바운드) 재사용.
- **PG사 구체 선정**(한국 신용카드/계좌이체 PG사명, 일본 신용카드 PG사명 등)은 미확정 — **O-204**.
- **Webhook 서명 검증 방식 및 검증 실패 시 처리**(거부/격리 후 검토 등)는 미확정 — **O-205**.

### 3.62 ERP 운영 생산성 및 관리자 UX 완성 ([PRD.md](PRD.md) §5.71~§5.78, [DECISIONS.md](DECISIONS.md) D-076)

> 새 ERP 기능이 아니라 **운영자(본사/Tenant 관리자/CS/물류/마케팅)의 일상 생산성**을 높이는 보강이다. **신규 Engine을 만들지 않는다** — Workflow Engine/Notification Center/Dashboard Builder/Report Builder/Form Builder/Scheduler Center/API Center/File Manager/Audit Center를 최대한 재사용한다. 다수 항목은 신규 테이블 없이 **기존 테이블 federated 조회**(Customer Timeline §3.57, 관리자 업무 Queue §3.57과 동일 패턴)이며, 진짜 신규 테이블은 5종뿐이다.

**`admin_favorite_menus`**(신규) — 메뉴 즐겨찾기/Pin. **O-199(저장된 검색조건/즐겨찾기 메뉴 테이블 도입 여부)를 본 라운드에서 해소한다.**

| 컬럼(개념) | 설명 |
|---|---|
| admin_id | 관리자 |
| menu_key | 메뉴 식별자(SITEMAP.md 메뉴 트리의 노드 키) |
| sort_order | 즐겨찾기 내 순서 |
| pinned_at | |

**`saved_filters`**(신규) — 검색조건/필터 저장. **O-199의 나머지 절반("저장된 검색조건")을 해소한다.**

| 컬럼(개념) | 설명 |
|---|---|
| admin_id | 작성자 |
| name | 필터명(예: "승인 대기"/"환불 대기"/"배송 지연"/"재고 부족"/"신규 가입"/"정산 보류") |
| target_module | 적용 대상 모듈(자유 확장값) |
| filter_criteria | 필터 조건(JSON) |
| is_default | 해당 모듈 진입 시 기본 적용 여부 |
| is_shared | 다른 관리자에게 공유 여부 |
| created_at | |

**`notification_inbox_states`**(신규) — Notification Inbox의 읍음/중요/보관 등 **상태 오버레이**. 기존 `notifications`/`notification_logs`(§3.20)는 변경하지 않는다 — `cms_translations`(§3.33)가 `cms_pages`를 오버레이하는 것과 동일한 패턴.

| 컬럼(개념) | 설명 |
|---|---|
| notification_id | `notifications` 참조 |
| recipient_type / recipient_id | MEMBER/ADMIN 등 — 기존 `notifications.member_id`는 회원 전용이라, 관리자 수신 알림(예: 결제실패/배송지연 운영 알림)까지 다루기 위한 범용 참조 |
| is_read / is_important / is_archived | |
| tags | 자유 태그(JSON 배열) |

**`admin_notes`**(신규, 범용 운영자 메모) — 회원/주문/상품/게시글/Workflow/정산 등 **여러 대상에 적용되는 범용 메모**. 기존 `order_admin_notes`(§3.53, 주문 전용)는 변경하지 않는다 — File Manager(§3.39)의 `related_entity_type`/`related_entity_id` 패턴 재사용.

| 컬럼(개념) | 설명 |
|---|---|
| related_entity_type / related_entity_id | 범용 참조(MEMBER/ORDER/PRODUCT/BOARD_POST/WORKFLOW_INSTANCE/SETTLEMENT_BATCH 등) |
| content | |
| is_important | 중요 메모 표시 |
| is_internal | 내부 전용(고객 노출 화면에 절대 노출 안 함, 항상 true 전제이나 컬럼으로 명시) |
| created_by / created_at | |

- **`admin_notes`(범용)와 `order_admin_notes`(주문 전용, 기존)의 관계는 본 라운드에서 통합/마이그레이션하지 않는다** — 기존 구조를 변경하지 않는다는 원칙에 따라 두 구조가 병렬로 존재한다. 통합 여부는 **O-206**.

**`approval_delegations`**(신규) — 승인 위임/휴가 중 대리 승인자. Workflow Engine(§3.37) 자체 구조는 변경하지 않는다 — 위임 정보는 Workflow Engine이 승인자를 결정할 때 **참조하는 위성 테이블**일 뿐이다.

| 컬럼(개념) | 설명 |
|---|---|
| delegator_id | 원래 승인자 |
| delegate_id | 대리 승인자 |
| start_date / end_date | 위임 기간 |
| reason | |
| created_by | |

- 위임 가능 범위(동일 역할 내에서만 위임 가능한지, 권한 레벨 검증 방식)는 미확정 — **O-207**.
- **승인 SLA**(모듈별 처리 기한 기준값)는 신규 테이블 없이 System Settings(§3.46) 패턴을 따르는 관리자 설정값으로 둔다 — 정확한 기본값(24시간/48시간 등)은 미확정. 지연 알림은 Scheduler Center(§3.40) Job이 SLA 초과 항목을 주기 점검해 Notification Center로 발송하는 기존 패턴 재사용.

**기존 구조 재사용 매핑(신규 테이블 최소화)**

| 기능 | 재사용 대상 |
|---|---|
| Global Search | 신규 검색 인덱스 테이블 없음 — `members`/`products`/`orders`/`shipments`/`workflow_instances`/`board_posts`/`cms_pages`/`faq_items`/`marketing_programs`/`files`/`audit_logs`/`notifications`/`external_api_connections`/`scheduled_job_definitions` 등 기존 테이블에 대한 federated 쿼리. 검색 통계는 `search_query_logs`(§3.35, 기존)를 관리자 검색에도 동일 패턴으로 재사용. 검색 권한은 ROLE-MATRIX.md 기존 모듈별 조회 권한을 그대로 적용(검색 결과는 권한 없는 모듈을 노출하지 않음) |
| Approval Center / Approval History | 신규 승인 테이블 없음 — `member_change_requests`/`organization_transfer_logs`/`returns`/`exchange_requests`/`wallet_withdrawal_requests`(§3.60)/`workflow_instances`+`workflow_step_actions`/`board_posts`(status=PENDING_APPROVAL)/`marketing_program_applications`를 federated 조회하는 **관리자 업무 Queue(§3.57)의 일반화**. 이력 조회는 `workflow_step_actions.decision`(기존, 승인/반려)+각 테이블의 approved_by/approved_at 조회, PDF/Excel은 Report Builder 재사용 |
| Recent Activity / Activity Timeline(관리자) | `audit_logs`(§3.8)/`member_activity_logs`(§3.22, 관리자 행위 기록 패턴 기존 확인)/`workflow_step_actions`/`external_api_call_logs`/`scheduled_job_run_logs` federated 조회 — Customer Timeline(§3.57)과 동일 원리의 관리자용 버전 |
| Notification Inbox | `notifications`/`notification_logs`(§3.20, 기존) + `notification_inbox_states`(신규, 위 참조). 첨부파일은 File Manager 재사용 |
| Tenant Usage Dashboard | 신규 테이블 없음 — 회원수/주문수/매출은 기존 트랜잭션 테이블 집계, Storage/API호출은 `external_api_call_logs`(§3.38), 메일/SMS/Push는 `notification_logs`(§3.20), Queue 사용량은 `scheduled_job_run_logs`(§3.40) — Dashboard Builder 위젯. License/Plan/사용률은 이미 O-170/O-146/§5.55(D-071, Multi-Tenant 활성화 시점으로 deferred)로 추적 중 |
| Personal Workspace | 신규 데이터 모델 없음 — My Dashboard(§3.57 `dashboard_definitions.owner_admin_id`)/Favorite Menu(위 신규)/Recent Activity/Saved Filter(위 신규)를 한 화면에 모은 것일 뿐 |
| Command Palette | DB 영향 없음 — Global Search와 동일한 검색 API를 빠른 진입 UI로 노출하는 프론트엔드 컴포넌트 |
| Universal Clipboard | DB 영향 없음 — 순수 프론트엔드 유틸리티(복사 후 기존 화면으로 이동) |
| 관리자 Dashboard(로그인 첫 화면) | 신규 테이블 없음 — My Dashboard(§3.57)의 시스템 기본 템플릿 + §3.56(D-069, 오늘 지표)/§3.60(D-075, 공제조합 전송실패·E-Wallet 출금대기 위젯 추가) + §3.57(D-071, System Health) 위젯 묶음 |
| System Health Widget | 신규 테이블 없음 — System Health Dashboard(PRD §5.54, D-071)를 별도 페이지가 아니라 관리자 Dashboard의 위젯으로도 노출 |
| Quick Action(보강) | 신규 데이터 모델 없음 — 기존 Quick Action(PRD §5.61)에 정산 조회/게시글 등록/Workflow 생성 항목만 추가, 각 기존 화면 바로가기 |

### 3.63 Marketing Reward System 및 Lifestyle Wallet 구조 개선 ([PRD.md](PRD.md) §5.83~§5.86, [DECISIONS.md](DECISIONS.md) D-078)

> **새 MLM 정책이 아니다.** 목적은 "Lifestyle 보상은 현금성 수당이 아니라 Marketing Reward Program"이라는 점을 데이터 구조로 명확히 하는 것이다 — `Reward Policy → Lifestyle Point Ledger → Lifestyle Wallet` 구조를 표준화하고, **현금(Settlement/E-Wallet CASH)과 포인트(Lifestyle Point)를 절대 혼합하지 않는다.** 신규 테이블은 `reward_policies` 1종뿐이며, 나머지는 §3.60(E-Wallet)을 확장 재사용한다.

**`member_wallets`(§3.60 확장) — 컬럼 추가**

| 컬럼(개념) | 설명 |
|---|---|
| **wallet_type**(신규) | `CASH`(§3.60 기존 5개 통화 지갑, 출금 가능) / `LIFESTYLE_POINT`(신규, 본 절) — 자유 확장값(Wallet Engine으로 향후 확장 가능, 현재는 두 값만 활성화). 기존 행은 `CASH`로 간주(개념상 backfill, 실제 마이그레이션은 구현 단계) |

- `wallet_type='LIFESTYLE_POINT'`인 지갑은 `currency_code` 대신 "POINT" 등 비-화폐 식별값을 쓰거나 nullable로 둔다(정확한 값은 구현 단계 결정) — `currency_code`는 화폐 단위 개념이고 `wallet_type`은 그보다 상위의 "지갑 종류" 구분이라 의미를 혼용하지 않는다.

**`wallet_transactions`(§3.60 확장) — 컬럼 추가 + 허용값 확장**

| 컬럼(개념) | 설명 |
|---|---|
| **counts_toward_compliance_limit**(신규) | 이 거래가 MLM 35% 법적 한도 산정에 포함되는지 — `point_transactions.counts_toward_compliance_limit`(§3.36, D-041)와 동일한 패턴을 그대로 포팅. `wallet_type=CASH`는 기존 §3.60 동작과 동일(EARN 시 항상 정산 인용이므로 사실상 산정에 이미 포함된 값), `wallet_type=LIFESTYLE_POINT`는 원천 `marketing_programs.links_to_compensation`(§3.34) 값을 그대로 따른다 |

- `transaction_type` 허용값에 **RESTORE(복원)**/**EXPIRE(만료)**를 추가한다(기존 CHARGE/EARN/USE/CANCEL/REFUND/WITHDRAWAL_REQUEST/WITHDRAWAL_COMPLETED/ADJUSTMENT/HOLD/RELEASE는 그대로 유지) — `point_transactions.transaction_type`(§3.36)이 이미 가진 RESTORE/EXPIRE와 동일한 개념이며, Lifestyle Point의 "사용 가능 기간 만료"/"주문 취소 시 사용분 복원"을 표현하기 위해 필요하다. 이는 컬럼 추가가 아니라 기존 자유 확장값 컬럼의 허용값 추가다.
- **`wallet_withdrawal_requests`(§3.60)는 `wallet_type=CASH`에만 적용된다 — `LIFESTYLE_POINT` 지갑은 출금(은행송금) 대상이 아니며, §3.63 §10/§11(쇼핑몰 사용/Program 신청 사용)으로만 차감(USE)된다.** 출금 신청 화면/API에 LIFESTYLE_POINT 지갑이 노출되지 않도록 한다.

**`reward_policies`(신규)** — Marketing Program 산하 보상 정책. 관리자가 직접 설정하며 하드코딩하지 않는다.

| 컬럼(개념) | 설명 |
|---|---|
| id | |
| program_id | `marketing_programs`(§3.34) 참조 — Program명은 이 참조로 가져오며 중복 저장하지 않는다 |
| reward_type | 자유 확장값, 현재는 `LIFESTYLE_POINT` 단일값(향후 Wallet Engine 확장 시 다른 값 추가 가능) |
| accrual_basis | 적립 기준 — 매출/PV/BV/주문금액/지급수당/기타 계산식, 자유 확장값(관리자 선택) |
| accrual_method | 적립 방식 — %/고정POINT/누적/단계별/조건부, 자유 확장값(관리자 선택) |
| accrual_rate | 적립률(%, 고정POINT 방식이면 고정 적립액으로 사용) — **Travel/Car/자기계발(§3.25, 기존) 3종은 본 컬럼에 수치를 별도 저장하지 않고 `plan_definition.lifestyle_bonus.*`(D-032)를 그대로 참조 표시한다** — 동일 수치를 두 곳에 중복 저장해 불일치 위험을 만들지 않는다. Golf 등 신규 Program만 본 컬럼에 실제 값을 관리자가 직접 입력한다 |
| accumulation_period | 누적 방식 — 월/분기/반기/연/사용자지정, 자유 확장값 |
| accumulation_duration | 누적 기간(예: 24개월/1개월/6개월/관리자설정) — Travel/Car/자기계발은 `plan_definition.lifestyle_bonus.*`(D-032) 참조와 동일하게 표시만 하고 별도 저장하지 않음 |
| min_condition | 최소 조건(자유 텍스트/JSON) |
| max_limit | 최대 한도 |
| start_date / end_date | 정책 시작일/종료일 |
| usable_period | 적립 후 사용 가능 기간(만료 계산 기준, `wallet_transactions.transaction_type=EXPIRE`와 연계) |
| target_wallet_type | `wallet_type`(위) 참조 — 현재는 `LIFESTYLE_POINT`만 유효 |
| is_active | 활성/비활성 |

- **기존 `lifestyle_bonus_accumulations`(§3.25)는 변경하지 않는다** — Travel/Car/자기계발 3종의 **적립 산정(누적 계산) 자체는 그대로 §3.25가 수행**한다. 본 라운드가 바꾸는 것은 **산정 완료 후의 저장 위치(라우팅)** 뿐이다: D-041은 누적 기간 종료 시 `point_transactions`(EARN, source_type=LIFESTYLE_BONUS)로 넘겼으나, 본 라운드 이후 신규 적립분은 **`wallet_transactions`(EARN, wallet_type=LIFESTYLE_POINT, source_type=REWARD_POLICY, source_id=`reward_policies.id`)로 라우팅한다.** D-041의 나머지(일반 쇼핑몰 구매 적립금의 `point_transactions` 경유 자체)는 전혀 변경하지 않는다 — Lifestyle Bonus 적립분의 목적지만 바뀐다.
- Golf 등 `lifestyle_bonus_accumulations`에 해당 `bonus_type`이 없는 신규 Program은 §3.25를 거치지 않고 worker가 `reward_policies` 설정을 직접 읽어 계산 후 `wallet_transactions`(EARN)을 생성한다 — 신규 산정 엔진을 만들지 않고 기존 worker 계산 패턴([ARCHITECTURE.md](ARCHITECTURE.md) §2.3과 동일한 "관리자 설정값을 입력 파라미터로 받는 계산" 원칙)을 그대로 따른다.
- **기존 `point_transactions.source_type=LIFESTYLE_BONUS`로 이미 적립된 과거 데이터를 Lifestyle Wallet으로 이전할지 여부는 미확정 — O-209.**

**Program 신청 사용 ([PRD.md](PRD.md) §5.85)** — `marketing_program_applications`(§3.34.1, 기존) 승인 시 `wallet_transactions`(USE, wallet_type=LIFESTYLE_POINT, source_type=PROGRAM_APPLICATION, source_id=`marketing_program_applications.id`) 1건을 생성한다. 승인 절차 자체(Workflow Engine 또는 §3.34.1의 경량 api 직접 승인)는 변경하지 않는다.

**쇼핑몰 사용 ([PRD.md](PRD.md) §5.84)** — Lifestyle Point의 쇼핑몰 사용 가능 여부는 Tenant별 설정이다. `tenant_settings`(§3.31.1)는 Multi-Tenant 활성화 보류 상태로 아직 생성되지 않으므로, 활성화 전까지는 System Settings(§3.46, 기존 활성 테이블)의 키-값 패턴을 재사용한다. **기본값(ON/OFF)은 미확정 — O-208.**

**Settlement와의 경계(재확인)** — Settlement(`settlement_batches`/`settlement_items`, §3.6)는 Unilevel Sponsor Bonus/Product Sales Bonus/Pair Bonus(현금성 3종)만 처리하며 본 절로 전혀 변경되지 않는다. Lifestyle Point/Wallet은 Settlement Ledger를 절대 참조하거나 합산하지 않는다 — 두 원장은 구조적으로 완전히 분리된다.

### 3.64 Reward Policy 고도화 및 운영 시뮬레이션 ([PRD.md](PRD.md) §5.87~§5.90, [DECISIONS.md](DECISIONS.md) D-079)

> Marketing Reward System(§3.63, D-078)을 완성하는 보강 — **새 MLM 정책이 아니다.** Reward Policy에 계산식(Reward Formula)을 추가하고 append-only로 버전 관리하며, 저장하지 않는 Simulation/Test 기능을 제공한다. 신규 테이블은 `reward_formula_versions` 1종뿐이다.

**`reward_policies`(§3.63 확장) — 컬럼 추가**

| 컬럼(개념) | 설명 |
|---|---|
| **active_formula_version_id**(신규) | `reward_formula_versions`(아래) 참조 — 현재 활성 Formula Version. 정책 등록 시점에는 nullable(Formula 미설정 상태로 시작 가능) |

- 기존 `accrual_basis`/`accrual_method`/`accrual_rate`(§3.63) 컬럼은 그대로 유지하되, 본 라운드 이후로는 **`active_formula_version_id`가 가리키는 행의 값을 그대로 복제한 파생 캐시**로 의미가 바뀐다(목록 화면 성능을 위한 비정규화 — `member_wallets.*_balance_cache`와 동일 원칙) — Formula Version이 바뀌면 이 캐시도 함께 갱신된다. **신뢰 가능한 원본은 항상 `reward_formula_versions`다.**

**`reward_formula_versions`(신규)** — Reward Formula의 append-only 버전 원장. 정책 수정 시 기존 행을 수정하지 않고 새 행을 추가하며, 기존 버전은 조회만 가능하다(append-only 원칙, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)와 동일 정신).

| 컬럼(개념) | 설명 |
|---|---|
| id | |
| reward_policy_id | `reward_policies`(§3.63) 참조 |
| version_number | 정책당 순번(1부터 증가) |
| formula_type | 자유 확장값 — `REVENUE_RATE`(매출×%)/`PV_RATE`/`BV_RATE`/`FIXED_POINT`/`CUSTOM`(관리자 정의 계산식) 등, 시스템이 강제하는 고정 목록 없음 |
| formula_definition | 계산식 정의(JSON, 구조는 `formula_type`에 따라 다름) — `marketing_plan_versions.plan_definition`(D-032)과 동일한 "관리자 설정값 JSON" 패턴, 하드코딩 없음. **`CUSTOM`(관리자 정의 계산식)의 정확한 표현 방식(자유 수식 문자열 파싱 vs 구조화된 규칙) 및 안전한 평가 방식은 미확정 — O-210** |
| accrual_rate | 이 버전의 적립률(% 또는 고정 Point) — `reward_policies.accrual_rate` 캐시의 원본 |
| is_active | 이 정책의 현재 활성 버전인지(정책당 단 1개만 true) |
| created_by / created_at | |
| effective_from | 이 버전이 적용되기 시작하는 시점(nullable=즉시 적용) |

- **Reward Simulation**([PRD.md](PRD.md) §5.88) — 관리자가 매출/PV/BV 등 가상 입력값을 넣어 예상 Point를 미리 계산한다. **신규 테이블 없음** — `reward_formula_versions`(임시 미저장 초안 포함) 또는 활성 버전을 worker/api가 그대로 읽어 계산만 수행하고 결과를 어떤 테이블에도 쓰지 않는다("순수 계산"). 실행 이력(요청자/시각/입력값)만 `audit_logs`(§3.8, 기존)에 기록하고, **계산 결과(가상의 Point 값)는 저장하지 않는다.**
- **Formula Test**([PRD.md](PRD.md) §5.89) — Formula Version을 저장(새 버전 생성)하기 전에 입력값으로 미리 테스트한다. Simulation과 동일하게 **신규 테이블 없음, 계산 결과 미저장**이며, 실행 이력만 `audit_logs`에 기록한다. Simulation은 "이미 등록된 정책"을 대상으로, Formula Test는 "아직 저장하지 않은 Formula 초안"을 대상으로 한다는 점만 다르다 — 둘 다 동일한 계산 로직(worker 또는 경량 api)을 재사용하며 신규 계산 엔진을 만들지 않는다.
- **Reward History**([PRD.md](PRD.md) §5.90) — Formula 변경 이력은 `reward_formula_versions`를 정책별로 시간순 조회, Policy 변경 이력은 `audit_logs`(기존, 관리자 설정 변경 추적 패턴 재사용)로 조회, Simulation/Formula Test 실행 이력도 `audit_logs`로 조회. **신규 이력 테이블 없음.**
- 본 절은 Settlement(`settlement_batches`/`settlement_items`)·MLM 보상플랜 계산 로직·ERP Core·Workflow 구조를 전혀 변경하지 않는다.

## 4. 설계 원칙

1. **정산/수당 관련 테이블은 append-only.** 금액이 걸린 데이터는 직접 수정하지 않고 새로운 행으로 정정한다 ([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)).
2. **모든 계산 결과는 입력 스냅샷을 함께 저장**하여 사후 검증 가능하게 한다.
3. **소프트 삭제 우선** — 회원/제품 등 참조되는 엔터티는 물리 삭제 대신 상태값으로 비활성화. **ERP UX Standard(D-061)의 Undo 기능도 이 소프트 삭제를 되돌리는 것이며, append-only 원장(원칙 1)에는 적용하지 않는다** — `commission_records`/`settlement_items`의 정정은 Undo가 아니라 원칙 1의 보정 엔트리로만 처리한다(§3.51).
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

### 7.7 상용 ERP 수준 Gap Analysis 연계 (신규 — [DECISIONS.md](DECISIONS.md) D-062)

[GAP-ANALYSIS.md](GAP-ANALYSIS.md) 1차 점검에서 데이터 모델에 영향을 주는 항목을 식별했다 — 재고 예약/Hold(O-127), 상품 카테고리 계층구조(O-128), 반품/교환 상태머신(O-129), 분할배송 구조(O-133), 로그 장기보존·아카이빙(O-160), closure table(`member_ancestors`) 쓰기비용(O-163) 등. 본 라운드는 스키마를 직접 변경하지 않으며, 전체 목록은 [DECISIONS.md](DECISIONS.md) §2 표를 참조.

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
- **(신규, D-033)** `package_commission_policies.grants_qualification`이 패키지 종류를 구분하지 않는 회원 단위 자격인지, 패키지별로 범위를 좁혀야 하는지(O-088), 패키지별 국가 판매범위(`packages.country_codes`)의 정확한 표현 방식(배열 컬럼 vs 조인 테이블)
- **(신규, D-035)** Multi-Tenant 활성화 시점 및 범위 — `tenant_id` 컬럼을 지금 미리 추가할지(§3.31 권고) 활성화 결정 시점에 일괄 마이그레이션할지
- **(신규, D-036)** Lifestyle Program FAQ를 Document Center(§3.18) 콘텐츠와 통합할지(태그/참조) `lifestyle_programs`에 별도 `faq_refs`를 신설할지, `lifestyle_programs.bonus_type`을 패키지(D-033)처럼 무제한 커스텀으로 일반화할지, 배너 클릭 통계 추적(`referral_link_clicks`, D-025와 동일 패턴) 필요 여부
- **(신규, D-038)** 페이지 CMS(`cms_pages` CUSTOM 타입)의 URL 라우팅 방식, 팝업 `display_frequency` 옵션 범위 및 저장 방식(쿠키 vs 회원데이터), `cms_translations`의 폴백 캐싱 전략(매 요청 조인 vs 빌드타임 생성)
- **(신규, D-039)** `marketing_programs.category`를 사전 정의 목록으로 제한할지 자유 텍스트로 둘지, `links_to_compensation=true` 프로그램 카테고리를 원본 보상플랜 3종(여행/자동차/자기계발) 이상으로 확장 허용할지
- **(신규, D-040)** `products` 스키마 확장 범위(브랜드/카테고리/태그/할인가/판매수량 집계) — 베스트·인기·추천·신상품·타임세일·브랜드관 구현의 선행조건, `coupons`를 독립 엔티티로 둘지 Marketing Program 카테고리로 흡수할지
- **(신규, D-041)** `point_transactions.amount`의 부호 표현 방식(양/음수 vs 별도 sign 컬럼), 사용신청 승인이 모든 `source_type`에 공통 필요한지 일부 자동승인 허용할지(`point_policies.requires_approval_for_use`의 세분화 여부)
- **(신규, D-042)** `marketing_program_applications`에 신청 정원(capacity) 관리 컬럼이 필요한지, 신청 마감일과 `marketing_programs.exposure_end_at`의 관계
- **(신규, D-043)** 관리자 통계(조회수/전환율/유지구매율 등)를 실시간 쿼리로 계산할지 `compliance_ratio_snapshots`(D-027)와 동일한 스냅샷 캐시 테이블을 둘지 — 회원/거래 규모 확정 후 재검토
- **(신규, D-044)** `tenant_settings`의 `shop_config_ref`/`cms_config_ref`/`mlm_config_ref`/`settlement_config_ref`가 기존 정책 테이블의 `tenant_id` 컬럼과 중복되는지(활성화 설계 시 재검토 필요)
- **(신규, D-047)** 기존 5개 전용 승인 구조를 Workflow Engine으로 통합할지(현재는 통합하지 않음), `workflow_steps.auto_approval_condition`의 표현 문법
- **(신규, D-048)** `external_api_connections.auth_key_ref`의 암호화 저장소 선정(Supabase Vault vs 별도 KMS), `category`를 사전정의 목록으로 제한할지
- **(신규, D-049)** 기존 `*_ref` 컬럼들의 File Manager 마이그레이션 범위/시점(전체 vs 신규 업로드만)
- **(신규, D-050)** `scheduled_job_definitions.cron_expression`을 관리자가 직접 수정 가능하게 할지(Rule Designer §3.23와 유사한 위험)
- **(신규, D-051)** `notification_send_rules.trigger_condition`/`target_filter`의 표현 문법(고정 세그먼트 vs 동적 쿼리)
- **(신규, D-052)** `audit_logs`(§3.8)에 IP 주소 컬럼이 이미 존재하는지 확인 필요 — 없으면 추가
- **(신규, D-053)** `dashboard_widgets.data_source`가 참조할 각 업무 모듈의 집계 API 계약 정의
- **(신규, D-057)** CRM Center를 CS Center(§3.19)와 하나의 화면으로 통합할지
- **(신규, D-058)** `products` 브랜드/제조사 마스터 테이블 분리 여부(현재는 컬럼으로 단순화), 회원할인 기준(회원유형 vs 별도 그룹)
- **(신규, D-059)** `tenant_settings` → `external_api_connections` 참조의 정확한 FK 구조 및 활성화 시점

> ~~포지션(레그) 개념의 구체적 구조~~ — **N/A로 해소** ([DECISIONS.md](DECISIONS.md) D-008, Unilevel Sponsor Plan에는 별도 포지션 개념 없음).
