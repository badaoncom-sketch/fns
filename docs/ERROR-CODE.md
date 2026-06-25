# ERROR-CODE.md — Error Code Standard

> 상태: 신규 v0.1 (문서 표준화 — 기존 API/Error 원칙 기반 코드 체계) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 목적: ERP 전체 에러 코드를 일관되게 부여하기 위한 네임스페이스와 예시를 정리한다. 본 문서는 실제 구현, 문구, HTTP 응답 본문을 확정하지 않는다.

## 0. 작성 원칙

- 실제 API 에러 포맷은 [API-SPEC.md](API-SPEC.md) §1.6이 우선한다.
- 본 문서는 코드 체계의 **카탈로그**이며, 신규 비즈니스 정책을 만들지 않는다.
- 코드의 의미는 기존 PRD/DATABASE/ARCHITECTURE/LEGAL/COMPENSATION/SETTLEMENT 문서의 규칙에 종속된다.

## 1. Code Format

```text
{DOMAIN}-{NNN}
```

| 요소 | 의미 |
|---|---|
| DOMAIN | 업무/플랫폼 영역 prefix |
| NNN | 001부터 증가하는 3자리 식별자 |

## 2. Domain Prefix

| Prefix | 영역 | Source |
|---|---|---|
| AUTH | 인증/권한/RBAC | [PRD.md](PRD.md) §5.6.4, [API-SPEC.md](API-SPEC.md) §2.1 |
| MEMBER | 회원/가입/상태/민감 변경 | [PRD.md](PRD.md) §5.3~§5.6 |
| ORG | 조직 이동/스폰서 트리 | [PRD.md](PRD.md) §5.16 |
| SHOP | 쇼핑몰/카탈로그/회원몰 | [PRD.md](PRD.md) §5.1.3 |
| PRODUCT | 상품/패키지/이미지 | [PRD.md](PRD.md) §5.1.4/§5.43 |
| ORDER | 주문/결제/취소/반품 접수 | [API-SPEC.md](API-SPEC.md) §2.4 |
| LOGISTICS | 배송/반품/재고/3PL | [PRD.md](PRD.md) §5.5 |
| MLM | 후원수당/자격/패키지 보상 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) |
| SETTLEMENT | 정산/세금/지급 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) |
| COMPLIANCE | 35% 한도/공제조합/법적 검증 | [ARCHITECTURE.md](ARCHITECTURE.md) §8 |
| WORKFLOW | Workflow Engine | [PRD.md](PRD.md) §5.30 |
| NOTIFICATION | Notification Center | [PRD.md](PRD.md) §5.34 |
| CMS | CMS/배너/FAQ/번역 | [PRD.md](PRD.md) §5.19 |
| PROGRAM | Marketing Program | [PRD.md](PRD.md) §5.20/§5.24 |
| POINT | 포인트 | [PRD.md](PRD.md) §5.23 |
| API | API Center/외부 연동/쿼터 | [PRD.md](PRD.md) §5.31 |
| FILE | File Manager/Storage | [PRD.md](PRD.md) §5.32 |
| SYSTEM | Scheduler/System Settings/공통 장애 | [PRD.md](PRD.md) §5.33/§5.39 |

## 3. Standard Error Catalog

| Code | 기본 의미 | HTTP 후보 | Source |
|---|---:|---:|---|
| AUTH-001 | 인증 토큰 없음 또는 형식 오류 | 401 | [API-SPEC.md](API-SPEC.md) §1.6 |
| AUTH-002 | Supabase Auth JWT 검증 실패 | 401 | [API-SPEC.md](API-SPEC.md) §2.1 |
| AUTH-003 | 역할 권한 부족 | 403 | [ROLE-MATRIX.md](ROLE-MATRIX.md) |
| AUTH-004 | 국가 스코프 위반 | 403 | [ARCHITECTURE.md](ARCHITECTURE.md) §4 |
| AUTH-005 | 조직 이동 전용 role guard 실패 | 403 | [ARCHITECTURE.md](ARCHITECTURE.md) §4/§7.1 |
| MEMBER-001 | 회원을 찾을 수 없음 | 404 | [API-SPEC.md](API-SPEC.md) §2.2 |
| MEMBER-002 | 가입심사 상태 전환 불가 | 409 | [PRD.md](PRD.md) §5.6.3 |
| MEMBER-003 | 민감 변경 요청 상태 충돌 | 409 | [API-SPEC.md](API-SPEC.md) §2.6 |
| MEMBER-004 | 필수 전자서명 누락 | 422 | [PRD.md](PRD.md) §5.13 |
| MEMBER-005 | 탈퇴/강제탈퇴 이후 신규 수당/정산 대상 제외 | 422 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §5, [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 |
| ORG-001 | 조직 이동 요청 권한 없음 | 403 | [PRD.md](PRD.md) §5.16.5 |
| ORG-002 | 조직 이동 사유 코드 누락 또는 비활성 | 422 | [DATABASE.md](DATABASE.md) §3.26 |
| ORG-003 | 조직 이동 증빙 첨부 누락 | 422 | [PRD.md](PRD.md) §5.16 |
| ORG-004 | 긴급 조직 이동 권한 부족 | 403 | [ARCHITECTURE.md](ARCHITECTURE.md) §7.1 |
| ORG-005 | 승인 전/적용 전 상태 충돌 | 409 | [DATABASE.md](DATABASE.md) §3.26 |
| SHOP-001 | 쇼핑몰 접근/채널 범위 미확정 영역 | 422 | [PRD.md](PRD.md) §5.1.3 |
| PRODUCT-001 | 상품을 찾을 수 없음 | 404 | [API-SPEC.md](API-SPEC.md) §2.3 |
| PRODUCT-002 | 패키지 정책 적용 기간 충돌 | 409 | [DATABASE.md](DATABASE.md) §3.24.1 |
| PRODUCT-003 | 상품 이미지 업로드 정책 위반 | 422 | [PRD.md](PRD.md) §5.43/§5.44.11 |
| ORDER-001 | 주문을 찾을 수 없음 | 404 | [API-SPEC.md](API-SPEC.md) §2.4 |
| ORDER-002 | 주문 상태 전이 불가 | 409 | [DATABASE.md](DATABASE.md) §3.3 |
| ORDER-003 | 결제수단/PG 토큰 오류 | 422 | [API-SPEC.md](API-SPEC.md) §2.4 |
| ORDER-004 | 취소/환불 정책 미확정 처리 필요 | 422 | [DATABASE.md](DATABASE.md) §3.3 |
| LOGISTICS-001 | 배송/반품 대상을 찾을 수 없음 | 404 | [DATABASE.md](DATABASE.md) §3.10 |
| LOGISTICS-002 | 3PL 연동 실패 | 502 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.7 |
| LOGISTICS-003 | 반품/검수 상태 충돌 | 409 | [DATABASE.md](DATABASE.md) §3.10 |
| MLM-001 | 유니레벨 자격 미충족 | 422 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.2 |
| MLM-002 | 제품 판매수익·페어보너스 자격 미충족 | 422 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.5 |
| MLM-003 | 패키지 정책 없음 또는 비활성 | 422 | [DATABASE.md](DATABASE.md) §3.24.1 |
| MLM-004 | LINE 깊이 산정 입력 오류 | 422 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3 |
| MLM-005 | worker 외 서비스에서 계산 시도 | 500/정책위반 | [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3 |
| SETTLEMENT-001 | 정산 배치를 찾을 수 없음 | 404 | [API-SPEC.md](API-SPEC.md) §2.5 |
| SETTLEMENT-002 | 정산 상태 전이 불가 | 409 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 |
| SETTLEMENT-003 | 법적 한도 검증 전 승인 시도 | 409 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9 |
| SETTLEMENT-004 | 세금 계산/원천징수 정보 부족 | 422 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §4 |
| SETTLEMENT-005 | append-only 원장 직접 수정 시도 | 500/정책위반 | [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) |
| COMPLIANCE-001 | 35% 한도 차단 상태 | 409 | [ARCHITECTURE.md](ARCHITECTURE.md) §8.1.3 |
| COMPLIANCE-002 | 국가별 임계치 미설정 | 422 | [DATABASE.md](DATABASE.md) §3.29 |
| COMPLIANCE-003 | 공제조합 보고서 생성 Job 실패 | 500 | [API-SPEC.md](API-SPEC.md) §2.8 |
| WORKFLOW-001 | Workflow 정의 비활성 | 422 | [DATABASE.md](DATABASE.md) §3.37 |
| WORKFLOW-002 | 현재 단계 승인자 불일치 | 403 | [DATABASE.md](DATABASE.md) §3.37 |
| WORKFLOW-003 | 이미 승인/반려/취소된 Workflow | 409 | [DATABASE.md](DATABASE.md) §3.37 |
| NOTIFICATION-001 | 알림 템플릿 없음 | 404 | [DATABASE.md](DATABASE.md) §3.20/§3.41 |
| NOTIFICATION-002 | 알림 발송 Job 실패 | 500 | [ARCHITECTURE.md](ARCHITECTURE.md) §2.3 |
| CMS-001 | CMS 콘텐츠를 찾을 수 없음 | 404 | [DATABASE.md](DATABASE.md) §3.33 |
| PROGRAM-001 | 신청 불가 프로그램 | 422 | [PRD.md](PRD.md) §5.24 |
| PROGRAM-002 | Marketing Program 신청 상태 충돌 | 409 | [DATABASE.md](DATABASE.md) §3.34.1 |
| POINT-001 | 포인트 계정을 찾을 수 없음 | 404 | [DATABASE.md](DATABASE.md) §3.36 |
| POINT-002 | 사용 가능 잔액 부족 | 422 | [PRD.md](PRD.md) §5.23 |
| POINT-003 | 포인트 사용신청 상태 충돌 | 409 | [DATABASE.md](DATABASE.md) §3.36 |
| API-001 | 외부 API 연결 설정 없음 | 404 | [DATABASE.md](DATABASE.md) §3.38 |
| API-002 | 외부 API 인증키 참조 오류 | 422 | [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2 |
| API-003 | Rate limit 초과 | 429 | [API-SPEC.md](API-SPEC.md) §1.6 |
| FILE-001 | 파일을 찾을 수 없음 | 404 | [DATABASE.md](DATABASE.md) §3.39 |
| FILE-002 | 파일 접근권한 없음 | 403 | [DATABASE.md](DATABASE.md) §3.39 |
| SYSTEM-001 | Job 상태를 찾을 수 없음 | 404 | [API-SPEC.md](API-SPEC.md) §1.7 |
| SYSTEM-002 | Job 실패 | 500 | [API-SPEC.md](API-SPEC.md) §1.7 |
| SYSTEM-003 | Redis 연결/큐 장애 | 503 | [DEPLOYMENT.md](DEPLOYMENT.md) §6/§7 |
| SYSTEM-004 | Supabase 연결 장애 | 503 | [DEPLOYMENT.md](DEPLOYMENT.md) §6 |

## 4. API Error Envelope Mapping

| 필드 | 매핑 |
|---|---|
| `code` | 위 `DOMAIN-NNN` 또는 [API-SPEC.md](API-SPEC.md) §1.6의 전역 코드 |
| `message` | 사용자/관리자 표시용 문구. 실제 문구는 구현 단계에서 i18n과 함께 확정 |
| `details` | 필드별 검증 실패, 원본 Job id, 외부 API 응답 요약 등 |

