# BUSINESS-RULE-CATALOG.md — Business Rule Catalog

> 상태: v0.5 (D-070 — 쇼핑몰 운영 Phase 2 및 문서 동기화: §4 쇼핑몰/SEO Rule Cross Reference 신설(BR-045~054 × API-SPEC/ERD/DATA-DICTIONARY/ROLE-MATRIX/TEST-PLAN 동기화 반영). 신규 Business Rule 없음, 기존 BR 변경 없음. D-069 — BR-045~BR-054 신규 추가, 모두 쇼핑몰·CMS 운영 규칙이며 MLM Rule 아님) · 최종 수정일: 2026-06-26 · 단계: 설계(Design)
> 목적: PRD/DATABASE/COMPENSATION/SETTLEMENT/LEGAL/DECISIONS에 흩어진 Business Rule을 한곳에서 찾을 수 있게 정리한다. 본 문서는 새 정책을 만들지 않고, 기존 문서의 위치와 상태만 색인한다.

## 0. 작성 원칙

- 본 문서는 **Single Source of Truth가 아니라 Source Locator**다. 실제 규칙 문장은 아래 "Source of Truth" 문서가 우선한다.
- "확정/미확정/권고/폐기" 상태는 기존 문서 표현을 그대로 따른다.
- 본 문서에 없는 규칙을 구현 근거로 삼지 않는다. 구현 전 반드시 관련 원문 문서를 확인한다.

## 1. Rule Catalog

| Rule ID | Rule 이름 | 상태 | Source of Truth | 관련 구현/검증 문서 |
|---|---|---|---|---|
| BR-001 | 무료 회원가입 | 확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.1, [PRD.md](PRD.md) §5.1/§5.6 | [DATABASE.md](DATABASE.md) §3.1/§3.12, [API-SPEC.md](API-SPEC.md) §2.2 |
| BR-002 | 가입심사 후 활성화 | 구조 제안, 세부 전환 미확정 | [PRD.md](PRD.md) §5.6.1~§5.6.3 | [DATABASE.md](DATABASE.md) §3.12/§3.14, [API-SPEC.md](API-SPEC.md) §2.2 |
| BR-003 | 회원 유형 4종 | 확정 | [PRD.md](PRD.md) §5.6.1, [DECISIONS.md](DECISIONS.md) D-011 | [DATABASE.md](DATABASE.md) §3.1/§3.14 |
| BR-004 | 지원 국가 KR/TH/JP/US, CN Reserved | 확정 | [PRD.md](PRD.md) §5.6.2, [DECISIONS.md](DECISIONS.md) D-011/D-023 | [DATABASE.md](DATABASE.md) §3.13 |
| BR-005 | Unilevel Sponsor Plan | 확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.1~§3.2, [DECISIONS.md](DECISIONS.md) D-008 | [DATABASE.md](DATABASE.md) §3.1/§3.9 |
| BR-006 | LINE1~LINE5 깊이 제한 | 확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3 | [DATABASE.md](DATABASE.md) §3.5/§3.11, [TEST-PLAN.md](TEST-PLAN.md) |
| BR-007 | 라인 도달 깊이별 단일 비율 산정 | 확정, D-014 정정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3, [DECISIONS.md](DECISIONS.md) D-018 | [DATABASE.md](DATABASE.md) §3.5 |
| BR-008 | 유니레벨 후원수당 자격 | 확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.2, [DECISIONS.md](DECISIONS.md) D-028 | [DATABASE.md](DATABASE.md) §3.27.1 |
| BR-009 | 제품 판매수익·페어보너스 자격 | 확정, 패키지 범위 일부 미확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.5, [DECISIONS.md](DECISIONS.md) D-028/D-033 | [DATABASE.md](DATABASE.md) §3.24/§3.27.2 |
| BR-010 | 패키지 엔진 일반화 | 확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.0~§4.1.3, [DECISIONS.md](DECISIONS.md) D-033 | [DATABASE.md](DATABASE.md) §3.24.1, [PRD.md](PRD.md) §5.1.4 |
| BR-011 | 제품 판매수익 산정 | 확정 구조, 법적 분류 미확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.2 | [DATABASE.md](DATABASE.md) §3.24, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §3 |
| BR-012 | 페어보너스 산정 | 확정 구조, 세부 재구매/자기매칭 이슈 미확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.3, [DECISIONS.md](DECISIONS.md) D-026 | [DATABASE.md](DATABASE.md) §3.24 |
| BR-013 | +알파/Lifestyle Bonus | 옵션, 세부 미확정 포함 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.2 | [DATABASE.md](DATABASE.md) §3.25/§3.36 |
| BR-014 | 후원수당 월 단위 산정 | 확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §5, [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §2 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.3/§2.4 |
| BR-015 | 월정산 단일 주기 | 확정, D-016 정정 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §2, [DECISIONS.md](DECISIONS.md) D-029 | [DATABASE.md](DATABASE.md) §3.6 |
| BR-016 | 세금 원천징수 3.3% | 법령 기준, 세부 적용 확인 필요 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §4 | [DATABASE.md](DATABASE.md) §3.7, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §8 |
| BR-017 | 정산 승인은 전용 구조 유지 | 확정 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9, [PRD.md](PRD.md) §5.30.3 | [DATABASE.md](DATABASE.md) §3.6/§3.37 |
| BR-018 | 정산 취소/정정 append-only | 확정 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §8, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1 | [DATABASE.md](DATABASE.md) §3.6 |
| BR-019 | 35% 법적 한도 | 확정 법령 제약 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §1/§6, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §3 | [ARCHITECTURE.md](ARCHITECTURE.md) §8.1, [DATABASE.md](DATABASE.md) §3.29 |
| BR-020 | 35% 모니터링 임계치 KR 30/33/35 | 확정값은 KR, 타국 미확정 | [ARCHITECTURE.md](ARCHITECTURE.md) §8.1.2, [DECISIONS.md](DECISIONS.md) D-027 | [DATABASE.md](DATABASE.md) §3.29 |
| BR-021 | 35% 차단은 Hard Gate | 확정 구조, 초과분 처리 미확정 | [ARCHITECTURE.md](ARCHITECTURE.md) §8.1.3 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 |
| BR-022 | 탈퇴/강제탈퇴 회원 신규 수당·정산 제외 | 확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3/§5, [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 | [DATABASE.md](DATABASE.md) §3.12/§4 |
| BR-023 | 탈퇴해도 sponsor_id 자동 변경 금지 | 확정 | [PRD.md](PRD.md) §5.3, [DECISIONS.md](DECISIONS.md) D-021 | [DATABASE.md](DATABASE.md) §3.1/§3.12, [ARCHITECTURE.md](ARCHITECTURE.md) §7 |
| BR-024 | 민감 변경 요청→영향분석→승인 | 확정 프로세스 | [PRD.md](PRD.md) §5.4, [ARCHITECTURE.md](ARCHITECTURE.md) §7 | [DATABASE.md](DATABASE.md) §3.9 |
| BR-025 | 추천인 변경은 관리자 전용 조직 이동 | 확정 | [PRD.md](PRD.md) §5.16, [DECISIONS.md](DECISIONS.md) D-020 | [DATABASE.md](DATABASE.md) §3.26, [API-SPEC.md](API-SPEC.md) §2.7 |
| BR-026 | 조직 이동 승인과 적용 분리 | 확정 | [ARCHITECTURE.md](ARCHITECTURE.md) §7.1, [DECISIONS.md](DECISIONS.md) D-022 | [DATABASE.md](DATABASE.md) §3.26 |
| BR-027 | 긴급 조직 이동은 SuperAdmin만 승인 | 확정 | [PRD.md](PRD.md) §5.16, [ARCHITECTURE.md](ARCHITECTURE.md) §7.1 | [ROLE-MATRIX.md](ROLE-MATRIX.md) §6 |
| BR-028 | 추천 링크 최초 sponsor 연결 | 확정, 코드 형식 미확정 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6 | [DATABASE.md](DATABASE.md) §3.28, [API-SPEC.md](API-SPEC.md) §2.2 |
| BR-029 | 쇼핑몰은 단일 통합 카탈로그 | 확정 | [PRD.md](PRD.md) §5.1.3, [DECISIONS.md](DECISIONS.md) D-034 | [DATABASE.md](DATABASE.md) §3.3/§3.24.1 |
| BR-030 | 회원몰은 구매 채널이 아니라 상태 관리 대시보드 | 확정 | [PRD.md](PRD.md) §5.1.3 | [SITEMAP.md](SITEMAP.md), [WIREFRAME.md](WIREFRAME.md) |
| BR-031 | 정기배송 처리 | 구조 확정, 세부 정책 미확정 | [PRD.md](PRD.md) §5.1.3, [DATABASE.md](DATABASE.md) §3.30 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.3 |
| BR-032 | 재고/물류는 3PL 전제 | 구조 확정, 업체·방식 미확정 | [PRD.md](PRD.md) §5.5, [ARCHITECTURE.md](ARCHITECTURE.md) §2.7 | [DATABASE.md](DATABASE.md) §3.10 |
| BR-033 | Marketing Program 카테고리 일반화 | 확정 | [PRD.md](PRD.md) §5.20, [DECISIONS.md](DECISIONS.md) D-039 | [DATABASE.md](DATABASE.md) §3.34 |
| BR-034 | Marketing Program 신청 프로세스 | 확정 구조 | [PRD.md](PRD.md) §5.24, [DECISIONS.md](DECISIONS.md) D-042 | [DATABASE.md](DATABASE.md) §3.34.1 |
| BR-035 | 포인트 생애주기 | 확정 구조, 세부 정책 미확정 | [PRD.md](PRD.md) §5.23, [DECISIONS.md](DECISIONS.md) D-041 | [DATABASE.md](DATABASE.md) §3.36 |
| BR-036 | Workflow Engine은 신규 범용 승인용 | 확정 | [PRD.md](PRD.md) §5.30, [DECISIONS.md](DECISIONS.md) D-047 | [DATABASE.md](DATABASE.md) §3.37 |
| BR-037 | 기존 전용 승인 구조는 Workflow Engine으로 이전하지 않음 | 확정 | [PRD.md](PRD.md) §5.30.3, [DATABASE.md](DATABASE.md) §3.37 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 |
| BR-038 | API Center 인증키 평문 저장 금지 | 확정 | [PRD.md](PRD.md) §5.31, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2 | [DATABASE.md](DATABASE.md) §3.38 |
| BR-039 | web/api/scheduler 계산 금지, worker 계산 전담 | 확정, 위반 금지 | [ARCHITECTURE.md](ARCHITECTURE.md) §1.1, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3 | [CODING-STANDARD.md](CODING-STANDARD.md) §3 |
| BR-040 | Redis는 source of truth가 아님 | 확정 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.5, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3 | [DEPLOYMENT.md](DEPLOYMENT.md) §6/§7 |
| BR-041 | Supabase PostgreSQL이 최종 source of truth | 확정 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.6 | [DATABASE.md](DATABASE.md) §1 |
| BR-042 | Multi-Tenant 구조 준비, 활성화 보류 | 확정 | [DECISIONS.md](DECISIONS.md) D-035/D-044/D-059 | [DATABASE.md](DATABASE.md) §3.31/§3.31.1 |
| BR-043 | CountryAdmin 국가 스코프 | 확정 구조, 세부 액션 미확정 | [PRD.md](PRD.md) §5.6.4, [ARCHITECTURE.md](ARCHITECTURE.md) §4 | [ROLE-MATRIX.md](ROLE-MATRIX.md) |
| BR-044 | Promotion/Rank/PV 체계 폐기 | 확정 폐기 | [PROMOTION-RULES.md](PROMOTION-RULES.md), [DECISIONS.md](DECISIONS.md) D-030 | [DATABASE.md](DATABASE.md) §3.2/§3.4 |
| BR-045 | 상품 판매상태(sales_status) 자동전이 규칙 | 신규(쇼핑몰 운영, MLM Rule 아님) | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.52, [PRD.md](PRD.md) §5.45.1 |
| BR-046 | 옵션별 품절 시 상품 노출 유지·해당 옵션만 선택 불가 처리 | 신규 | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.52 |
| BR-047 | 창고간 이동(`TRANSFER_OUT`/`TRANSFER_IN`) 쌍 일관성 — 두 행 수량 항상 일치, 한쪽만 생성 금지 | 신규 | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.53 |
| BR-048 | LOT 우선 출고(FEFO) — 유통기한이 짧은 LOT을 먼저 출고 | 신규, LOT 도입 자체는 미확정(O-177) | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.53 |
| BR-049 | 주문 병합/분리 시 매출 합계 일치(append-only 로그) | 신규 | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.53 |
| BR-050 | 부분교환은 반품 입고 확인과 신규 출고를 단일 트랜잭션으로 묶어 재고 정합성 보장 | 신규 | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.53 |
| BR-051 | 배송비 정산은 정산 시점의 정책 버전을 스냅샷으로 고정 — 사후 정책변경이 과거 정산에 영향 없음 | 신규, 도입 자체는 미확정(O-184) | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.53 |
| BR-052 | 검색어 오타교정은 유사도 매칭, 정확도 임계값은 관리자 설정값(하드코딩 금지 원칙 적용) | 신규 | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.54, [PRD.md](PRD.md) §5.45.3 |
| BR-053 | **SEO 필드 자동생성 우선순위** — 상품명→SEO Title, 요약설명→Description, 대표이미지→OG Image, 브랜드→Schema Brand, 가격→Offer, 재고→Availability 자동 매핑하되 **관리자 수동 입력값이 항상 우선** | 신규(핵심) | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.55, [PRD.md](PRD.md) §5.46.1 |
| BR-054 | SEO 이미지 권장 규격은 DB 제약이 아니라 UI 안내 문구로만 처리. sitemap.xml/robots.txt는 별도 원장 없이 요청 시점 동적 생성(쿼리타임 파생) | 신규 | [DECISIONS.md](DECISIONS.md) D-069 | [DATABASE.md](DATABASE.md) §3.55 |

## 2. Open/Mixed 상태 Rule 추적

| Rule ID | 미확정 성격 | 추적 위치 |
|---|---|---|
| BR-011/BR-012/BR-019 | 제품 판매수익·페어보너스의 법적 분류와 35% 포함 여부 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §6/§8, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §3, [DECISIONS.md](DECISIONS.md) O-059 |
| BR-026 | 조직 이동 effective_date 기준시점/타임존 | [DATABASE.md](DATABASE.md) §3.26, [DECISIONS.md](DECISIONS.md) O-068/O-069 |
| BR-031 | 정기배송 결제 재시도 정책 | [PRD.md](PRD.md) §5.27, [DECISIONS.md](DECISIONS.md) O-104 |
| BR-043 | 기능별 관리자 권한 매핑 | [ROLE-MATRIX.md](ROLE-MATRIX.md), [DECISIONS.md](DECISIONS.md) O-042/O-066 |


## 3. MLM/마케팅 플랜 Rule Cross Reference (가독성 보강, 신규)

> 마케팅 플랜(BR-001~BR-021)에 한정해, 각 Rule이 다른 산출물 어디에 반영되어 있는지 한 번에 보여준다. "없음"은 해당 문서에 그 Rule을 다루는 전용 절이 아직 없다는 뜻이며(누락이 아니라 현재 상태의 정확한 기록), 새 절을 만들지 않는다 — 필요해지면 별도 라운드에서 채운다.

| BR-ID | PRD.md | DATABASE.md | STATE-MACHINE.md | TEST-PLAN.md | API-SPEC.md | ERD.md |
|---|---|---|---|---|---|---|
| BR-001 (무료회원가입) | §5.1/§5.6 | §3.1/§3.12 | §1(회원상태) | 없음(일반 CRUD) | §2.2(Member) | 클러스터1 |
| BR-005 (Unilevel Sponsor Plan) | §5.1/§5.16 | §3.1/§3.9 | 없음(트리 구조, 상태전이 아님) | §2.1 | §2.2/§2.7 | 클러스터1/3 |
| BR-006 (LINE1~5 깊이제한) | §5.1.1 | §3.5/§3.11 | 없음 | §2.1 | §2.5 | 클러스터3 |
| BR-007 (라인 단일비율 산정, D-018) | §5.1.1 | §3.5 | 없음 | §2.1 | §2.5 | 클러스터3 |
| BR-008 (유니레벨 자격) | §5.1.2 | §3.27.1 | 없음(자격은 derived flag) | §2.1 | §2.5 | 클러스터3 |
| BR-009 (제품판매수익·페어보너스 자격) | §5.1.2 | §3.24/§3.27.2 | 없음 | §2.1 | §2.5 | 클러스터3 |
| BR-010 (패키지 엔진 일반화) | §5.1.4 | §3.24.1 | 없음 | §2.1 | §2.3/§2.5 | 클러스터3 |
| BR-011 (제품판매수익 산정) | §5.1.2 | §3.24 | 없음(이벤트는 [EVENT-CATALOG.md](EVENT-CATALOG.md) PackagePurchased) | §2.1/§2.3 | §2.5 | 클러스터3 |
| BR-012 (페어보너스 산정) | §5.1.2 | §3.24 | 없음(이벤트는 [EVENT-CATALOG.md](EVENT-CATALOG.md) PairBonusMatched) | §2.1 | §2.5 | 클러스터3 |
| BR-013 (+알파/Lifestyle Bonus) | §5.1.5 | §3.25/§3.36 | §10(Point Transaction, 부분) | 없음(전용 테스트 절 없음) | §2.20(Point) | 클러스터9 |
| BR-014 (월 단위 산정) | §5.1.1 | §3.5 | §6(정산 배치, 부분) | §2.1 | §2.5 | 클러스터3/4 |
| BR-015 (월정산 단일주기) | §5.7 | §3.6 | §6(정산 배치) | §2.2 | §2.5 | 클러스터4 |
| BR-016 (세금 원천징수) | — | §3.7 | 없음 | §2.2 | §2.5 | 클러스터4 |
| BR-017 (정산승인 전용구조 유지) | §5.30.3 | §3.6/§3.37 | §6/§11(Workflow) | §2.5 | §2.5 | 클러스터4/12 |
| BR-018 (append-only 정정) | — | §3.6 | 없음 | §2.2 | §1.7(Job 패턴) | 클러스터4 |
| BR-019 (35% 법적 한도) | §5.18 | §3.29 | §7(Compliance Ratio) | §2.3 | §2.8(Compliance) | 클러스터4 |
| BR-020 (35% 임계치 KR 30/33/35) | §5.18 | §3.29 | §7 | §2.3 | §2.8 | 클러스터4 |
| BR-021 (35% Hard Gate) | §5.18 | — | §7 | §2.3 | §2.8 | 클러스터4 |

> STATE-MACHINE.md에 페어보너스 대기열(PairCandidateQueued→PairBonusMatched/Expired) 전용 상태도가 아직 없다는 점이 본 표에서 드러난다 — 이는 [EVENT-CATALOG.md](EVENT-CATALOG.md)의 이벤트 정의로만 다뤄지고 있다. 신규 Rule이 아니므로 본 라운드에서 추가하지 않으며, 필요 시 별도 Change Request로 진행한다.

> **관리자 설정 화면 단위로 그룹화한 BR 연결(Package Manager/Compensation Policy/Lifestyle Program 등)은 [WIREFRAME.md](WIREFRAME.md) §4.9 참조** — 본 표의 부분집합을 화면 관점으로 재정리한 것이며 중복 유지하지 않는다(가독성 보강, D-068).

## 4. 쇼핑몰/SEO Rule Cross Reference (D-070, 가독성 보강 — 신규 Rule 없음)

> BR-045~BR-054(D-069, 쇼핑몰·CMS 운영 규칙)에 한정해 §3과 동일한 형식으로 산출물 간 연결을 보여준다. 이번 라운드(D-070, "쇼핑몰 운영 Phase 2 및 문서 동기화")는 §2.21~§2.25(API-SPEC.md)/클러스터14(ERD.md)/§7(DATA-DICTIONARY.md)/§24~§25(ROLE-MATRIX.md)/§2.10(TEST-PLAN.md) 동기화를 반영해 갱신했을 뿐, **새 Business Rule은 추가하지 않았다.**

| BR-ID | PRD.md | DATABASE.md | STATE-MACHINE.md | TEST-PLAN.md | API-SPEC.md | ERD.md | ROLE-MATRIX.md |
|---|---|---|---|---|---|---|---|
| BR-045 (sales_status 자동전이) | §5.45.1 | §3.52 | §15(상품 판매상태) | §2.10 | §2.21(Shop) | 클러스터14-A | §24 |
| BR-046 (옵션 품절시 상품 노출 유지) | §5.45.1 | §3.52 | §15 | §2.10 | §2.21 | 클러스터14-A | §24 |
| BR-047 (창고간 이동 쌍 일관성) | §5.45.2 | §3.53 | 없음 | §2.10 | §2.21 | 클러스터14-A | 없음(전용 화면 미정) |
| BR-048 (LOT 우선출고 FEFO) | §5.45.2 | §3.53 | 없음 | §2.10 | §2.21 | 클러스터14-A | 없음 |
| BR-049 (주문 병합/분리 매출 일치) | §5.45.2 | §3.53 | 없음 | §2.10 | §2.21 | 클러스터14-A | 없음 |
| BR-050 (부분교환 단일 트랜잭션) | §5.45.2 | §3.53 | §16(반품/교환 통합, 권장안) | §2.10 | §2.21 | 클러스터14-A | 없음 |
| BR-051 (배송비 정산 스냅샷 고정) | §5.45.2 | §3.53 | 없음 | §2.10 | §2.21 | 클러스터14-A | 없음 |
| BR-052 (검색 오타교정 유사도/관리자 임계값) | §5.45.3 | §3.54 | 없음 | 없음(애플리케이션 레벨) | 없음(검색은 별도 모듈) | 클러스터14-B | 없음 |
| BR-053 (SEO 자동생성 — 관리자 수동값 우선) | §5.46.1 | §3.55 | 없음 | §2.10 | §2.23(SEO) | 클러스터14-C | §24 |
| BR-054 (SEO 이미지규격 UI안내, sitemap/robots 쿼리타임 파생) | §5.46.2 | §3.55 | 없음 | §2.10 | §2.23 | 클러스터14-C | §24 |

**D-070에서 신설된 운영 기능(§5.47~§5.50)과 BR의 관계**: Digital Marketing 연동 관리(§5.47)/이미지 최적화 운영(§5.49)/상품 Feed 관리(§5.50)는 기존 구조(`external_api_connections`/File Manager/Scheduler Center/Bulk Action)를 재사용하는 **운영·인프라 설정**이며, 계산·판정 로직이 없어 Business Rule 영역에 해당하지 않는다 — **연결되는 BR이 없다는 것이 누락이 아니라 정확한 현황 기록이다.** SEO 운영 Dashboard(§5.48)는 표시 대상 데이터(`product_seo` 필드 채움 여부, 공유클릭 등)에 한해 BR-053과 간접적으로 연결되며, 신규 Open Decision은 O-195(검색엔진 지표 캐시 저장 여부) 하나만 추가되었다 — [DECISIONS.md](DECISIONS.md) D-070 참조.
