# BUSINESS-RULE-CATALOG.md — Business Rule Catalog

> 상태: 신규 v0.1 (문서 표준화 — 기존 24개 문서의 Business Rule 색인) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
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

## 2. Open/Mixed 상태 Rule 추적

| Rule ID | 미확정 성격 | 추적 위치 |
|---|---|---|
| BR-011/BR-012/BR-019 | 제품 판매수익·페어보너스의 법적 분류와 35% 포함 여부 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §6/§8, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §3, [DECISIONS.md](DECISIONS.md) O-059 |
| BR-026 | 조직 이동 effective_date 기준시점/타임존 | [DATABASE.md](DATABASE.md) §3.26, [DECISIONS.md](DECISIONS.md) O-068/O-069 |
| BR-031 | 정기배송 결제 재시도 정책 | [PRD.md](PRD.md) §5.27, [DECISIONS.md](DECISIONS.md) O-104 |
| BR-043 | 기능별 관리자 권한 매핑 | [ROLE-MATRIX.md](ROLE-MATRIX.md), [DECISIONS.md](DECISIONS.md) O-042/O-066 |

