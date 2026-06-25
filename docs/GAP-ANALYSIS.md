# GAP-ANALYSIS.md — 상용 ERP 수준 Gap Analysis (1차)

> 상태: 3차 보강 완료(2026-06-26, D-064 — §11 문서 중복 제거 보고서 추가, 개발 Blocker를 [DECISIONS.md](DECISIONS.md) §2.2로 단일화) · 점검일: 2026-06-25~26 · 단계: 설계(Design) · [DECISIONS.md](DECISIONS.md) D-062~D-064
> 목적: FNS를 "FNS 전용 시스템"이 아니라 "다양한 직접판매 기업이 사용할 수 있는 상용 Multi-Tenant MLM ERP Platform" 수준으로 완성하기 위해, 현재까지의 설계(PROJECT-CONTEXT/PRD/ARCHITECTURE/DATABASE/COMPENSATION-RULES/SETTLEMENT-RULES/LEGAL-CHECKLIST/TASK-SPEC/DO-NOT-TOUCH/DECISIONS/CHANGELOG)를 상용 ERP/이커머스 관리자 기준으로 점검한 결과를 기록한다.
> **이 문서는 신규 기능을 직접 설계하지 않는다.** 발견된 갭은 모두 [DECISIONS.md](DECISIONS.md)의 Open Decision(O-121~O-169)으로 등록했으며, 실제 설계(스키마 추가/신규 PRD 섹션 작성)는 사용자가 우선순위를 정해 별도 라운드로 진행한다(D-062).

## 0. 점검 방법론

- 5개 영역(① 회원관리/CRM/Multi-Tenant ② 쇼핑몰/상품/주문/배송/재고 ③ CMS/Marketing Program/Notification/Dashboard·Report·Form Builder ④ ERP Core 운영(Workflow/API Center/Scheduler/File Manager/System Settings/Audit) + 개발 산출물 점검 ⑤ MLM/정산/법무 운영기능 + 성능/확장성 + UX)으로 나눠 각 영역의 실제 문서를 읽고, SAP S/4HANA·Microsoft Dynamics 365·Oracle NetSuite·Odoo·ERPNext·Salesforce CRM(운영기능)·Shopify Admin·Adobe Commerce(Magento)·Cafe24·NHN Commerce·Shopline 등이 **일반적으로 제공하는 기능 카테고리**(특정 제품의 화면을 그대로 모방하지 않음)와 비교했다.
- 모든 갭은 제안 전 [DECISIONS.md](DECISIONS.md)의 D-001~D-061, O-002~O-120을 grep으로 확인해 중복 여부를 검증했다.
- **다음은 점검 범위에서 제외했다(중요 원칙 준수)**: MLM 보상플랜 산정 로직/비율, 정산 계산 로직, Workflow Engine·ERP Core의 기존 구조, 기존 기능의 삭제. 이 영역들은 "운영/조회 기능의 누락" 여부만 점검했다.

## 1. 현재 설계 완성도 평가

| 관점 | 평가 | 근거 |
|---|---|---|
| **기능 설계(Functional Design) 완성도** | 높음 (정성 평가, 약 90% 내외) | D-001~D-061 61건의 확정 결정과 PRD §5.1~§5.44(44개 절)가 MLM 핵심 도메인뿐 아니라 ERP Core 12개 엔진·CMS·CRM·쇼핑몰 보강까지 상용 ERP의 주요 도메인을 대부분 커버한다. 본 점검에서 49건의 보완 영역을 추가로 식별했으나, 이는 "없는 영역"이 아니라 "이미 있는 설계의 깊이(depth)를 더하는" 성격이 대부분이다. |
| **구현 착수 준비도(Implementation-readiness)** | 낮음~중간 (정성 평가, 약 40~50%) | §3 "개발 준비도 점검" 참조 — ERD 다이어그램, API Specification, Wireframe, Coding Standard, Test Plan, Deployment Guide가 전혀 없거나 텍스트 수준의 부분 정보만 존재한다. 기능 설계는 앞서 있으나, 개발자가 바로 착수할 수 있는 산출물은 아직 부족하다. |
| **법무/컴플라이언스** | 중간 | 다단계판매법 핵심 규정(후원수당/청약철회/금지행위)은 상세히 다뤄졌으나, 일반 전자상거래법(고객몰 운영 시)·개인정보 self-service 권리 관련 항목이 상대적으로 얕다(O-123/O-154/O-155). |
| **Multi-Tenant 성숙도** | 중간 | 구조(도메인/브랜드/설정 분리)는 준비되어 있으나(D-035/D-044/D-059), 활성화 시 리소스 격리(O-159)·과금 모델이 아직 설계되지 않았다. "구조 준비, 활성화는 보류" 원칙(D-035) 자체는 유지한다. |

## 2. 영역별 상세 점검 결과 (기능 갭)

### 2.1 회원관리 / CRM / Multi-Tenant / Auth

| 항목 | 현재 상태 | 갭 | 우선순위 | O-번호 |
|---|---|---|---|---|
| 중복가입 검증 | `members`(§3.1)에 unique 제약 명시 없음 | 이메일/휴대폰 중복가입 및 재가입 시 식별정보 충돌 처리 미정 | High | O-121 |
| 회원 세그먼트/태그 | 직급(Rank) 폐기(D-030) 이후 대체 분류체계 없음 | 세그먼트/태그 체계 부재 — 알림 대상그룹(O-109)·회원할인 그룹과 중복 설계 위험 | High | O-122 |
| 개인정보 self-service | LEGAL-CHECKLIST §7은 보관/파기만 체크 | 열람·이동·삭제권 self-service 워크플로우 없음 | High | O-123 |
| 관리자 SSO | Auth는 Supabase Auth 그대로(D-002) | SAML/OIDC 연동 여부 미정 | Medium | O-124 |
| 권한 임시 위임 | 역할별 권한만 정의(§5.6.4) | 대리 승인 메커니즘 없음 | Medium | O-125 |
| 대량 Import/Export | File Manager는 저장소일 뿐 | 마스터데이터 대량 등록/내보내기 기능 없음 | Medium | O-126 |

### 2.2 쇼핑몰 / 상품관리 / 주문관리 / 배송관리 / 재고관리

| 항목 | 현재 상태 | 갭 | 우선순위 | O-번호 |
|---|---|---|---|---|
| 재고 예약(Hold) | `inventory_ledger`(append-only 원장)만 존재 | 주문시점 재고 예약·자동해제·창고 우선순위 로직 없음 | High | O-127 |
| 카테고리 계층구조 | `products`는 이름/가격/활성여부만(§3.3), 카테고리 체계 "미확정"으로만 서술 | 트리 구조·다중 소속 여부 미정 | High | O-128 |
| 반품/교환 상태머신 | `returns`는 "접수/검수/환불·재입고"로만 서술 | 정확한 상태값, 교환과 반품의 분리 여부 미정 | High | O-129 |
| 백오더(선주문) | 재입고알림(`restock_notifications`)만 존재 | 품절 상태에서의 주문 허용 여부 미정 | Medium | O-130 |
| 일반 번들/세트 상품 | 패키지 엔진(§3.24.1)은 보상플랜 전용 | 보상플랜 무관 묶음판매 모델 부재 | Medium | O-131 |
| 자동 할인 규칙 | 쿠폰(§3.35)만 존재 | 수량/금액 조건 기반 자동 할인 엔진 없음 | Medium | O-132 |
| 분할배송 | `shipments`는 단수 개념으로 서술 | 주문:배송 = 1:N 구조 명시 필요 | Medium | O-133 |
| 안전재고 알림 | 재입고알림은 고객 대상 | 관리자 대상 안전재고 임계치 알림 없음 | Low | O-134 |
| 부분 취소/환불 | 주문 단위 취소/환불만 서술(§3.3) | order_item 단위 부분 처리 미정 | Medium | O-135 |

### 2.3 CMS / Marketing Program / Notification / Dashboard·Report·Form Builder

| 항목 | 현재 상태 | 갭 | 우선순위 | O-번호 |
|---|---|---|---|---|
| SEO 메타필드 | `cms_pages`/`marketing_programs`에 메타필드 없음 | 메타타이틀/디스크립션/OG태그 부재 | High | O-136 |
| 콘텐츠 승인/예약발행 | `published_at`만 존재, 상태머신 없음 | DRAFT/IN_REVIEW/SCHEDULED/PUBLISHED 등 상태 부재 | High | O-137 |
| A/B 테스트 | 배너/팝업/프로그램 모두 단일 버전 | 트래픽 분할 노출·성과비교 기능 없음 | Medium | O-138 |
| 캠페인 시퀀스 | `notification_send_rules`는 단발성 조건발송만(O-109) | 다단계 드립 캠페인 구조 없음 | Medium | O-139 |
| 번역 상태 추적 | `cms_translations`는 값만 저장 | 번역 진행상태(미번역/번역중 등) 추적 불가 | Medium | O-140 |
| Form 제출 연계 | `form_submissions`는 적재만 | CRM/Notification으로의 연계 경로 없음 | Medium | O-141 |
| 데이터소스 카탈로그 | Dashboard/Report Builder 각자 `data_source` 자유 문자열 | 공유 카탈로그/API 계약 없음 | Medium | O-142 |
| 팝업 동시노출 정책 | 배너만 동시노출수 정책 논의(O-094) | 팝업에는 동일 정책 미적용 | Low | O-143 |

### 2.4 ERP Core 운영(Workflow/API Center/Scheduler/File Manager/System Settings/Audit)

| 항목 | 현재 상태 | 갭 | 우선순위 | O-번호 |
|---|---|---|---|---|
| RTO/RPO·DR 절차 | O-037은 "백업 주기"만 추적 | 복구목표시간/시점, 리전 장애 대응 절차 없음 | High | O-144 |
| 아웃바운드 Webhook | API Center는 인바운드(외부 API 호출) 패턴만 | 외부 시스템의 이벤트 구독/콜백 수신 기능 없음 | Medium | O-145 |
| API 호출 쿼터 추적 | Rate Limiting은 NestJS 레벨 한 줄만(ARCHITECTURE §4) | 외부 연동별 쿼터 추적/경고 없음 | Medium | O-146 |
| Workflow SLA/에스컬레이션 | 단계/승인자/자동승인조건만 정의(§5.30.1) | 처리기한·기한초과 에스컬레이션 없음 | Medium | O-147 |
| 환경 분리(dev/staging/prod) | ARCHITECTURE §10에 비공식 한 줄만 존재 | 정식 Open Decision으로 등록되지 않음 | Medium | O-148 |
| Audit Log PII/보존 | 변경 전/후 JSON은 이미 확보(§3.8) | 마스킹/암호화, 보존기간 만료 후 처리방식 미정 | Low | O-149 |
| Scheduler 가시성 | `is_enabled`만 존재 | 동시실행/분산락 상태 가시화 없음 | Low | O-150 |

### 2.5 MLM/정산/법무 운영기능 (※ 계산 로직·비율은 점검 대상 아님 — 운영/조회 기능만)

| 항목 | 현재 상태 | 갭 | 우선순위 | O-번호 |
|---|---|---|---|---|
| 수당 내역서 다운로드 | 회원용 라인별 노출 자체가 미확정(§5.1.1) | PDF/Excel 다운로드 기능 미언급 | Medium | O-151 |
| 세무서식 자동생성 | "법적 의무 확정, 구현방식 미확정"(SETTLEMENT-RULES §4) | 표준서식/발급주기 미정 | Medium | O-152 |
| 정산 이의신청 연결 | CS의 이의신청은 처분 이의로 한정(O-027) | 산정결과 이의 → 재계산·보정엔트리 연결 구조 없음 | High | O-153 |
| 전자상거래법 표시의무 | 통신판매업 신고/청약철회만 체크(§6) | 사업자정보·거래조건 표시의무 체크리스트 부재 | Medium | O-154 |
| 결제대금예치(에스크로) | 언급 없음 | 구매안전서비스 가입 의무 검토 누락 | Medium | O-155 |
| 패키지 변경 시 고지 갱신 | 사전고지는 일반론만(§3) | 패키지별 변경 시 고지자료 갱신 절차 없음 | Low | O-156 |
| 공제조합 보고서 정정 | 제출 이력 조회만(§5.7) | 정정·재제출 절차 없음 | Low | O-157 |

### 2.6 성능/확장성

| 항목 | 현재 상태 | 갭 | 우선순위 | O-번호 |
|---|---|---|---|---|
| 이미지/동영상 CDN | Storage 참조만 정의 | CDN·리사이징/트랜스코딩 전략 없음 | High | O-158 |
| Tenant Job 격리 | 활성화 시점만 논의(O-090) | 테넌트별 리소스 쿼터/노이즈 네이버 방지 없음 | High | O-159 |
| 로그 아카이빙 | 파티셔닝(O-055)만 논의 | 장기보존·콜드스토리지 정책 없음 | Medium | O-160 |
| BullMQ 장애복구 | Redis↔Postgres 정합화(O-078)는 컴플라이언스 한정 | Job 손실 감지·DLQ 재처리 일반 정책 없음 | Medium | O-161 |
| 다국어 검색 인덱싱 | 국가는 계산 파라미터로만 다룸(ARCHITECTURE §8) | 언어별 형태소분석 전략 없음 | Medium | O-162 |
| closure table 쓰기비용 | `member_ancestors` 도입 권고만 존재(§7) | 대규모 조직이동 시 갱신비용 처리방식 없음 | Low | O-163 |

### 2.7 UX 추가 점검 (ERP UX Standard, D-061 보강)

| 항목 | 현재 상태 | 갭 | 우선순위 | O-번호 |
|---|---|---|---|---|
| 대량 리스트 UX | §5.44는 Confirm/Toast/Bulk Action 등만 다룸 | 페이지네이션 방식·그리드 가상화 기준 없음 | High | O-164 |
| 검색/필터 표준 | 적용 화면만 명시(§5.44.12) | 고급검색·필터 프리셋 표준 없음 | Medium | O-165 |
| 테이블 컬럼 커스터마이즈 | 언급 없음 | 표시/순서/고정 공통 컴포넌트 없음 | Medium | O-166 |
| 접근성(WCAG) | 언급 없음 | 키보드/ARIA 기준 없음 | Medium | O-167 |
| 다중 파일 업로드 UX | 이미지 순서변경만 다룸(O-119) | 동시 업로드 개별 진행률 표시 패턴 없음 | Low | O-168 |

### 2.8 추가 벤치마킹 결과 (개발준비도 관점, D-063 라운드 — O-121/O-009/O-062/O-113 일부는 정리되어 본 절에서 갱신)

"개발 착수 준비 문서 세트" 라운드(D-063)에서 §2.1~2.7과 중복되지 않는 운영 패턴 갭을 추가로 찾았다 — 권한 운영/Multi-Tenant 운영/API 거버넌스/시스템 헬스/다국어 운영/컴플라이언스 보고 관점.

| 항목 | 현재 상태 | 갭 | 우선순위 | O-번호 |
|---|---|---|---|---|
| Multi-Tenant 온보딩/모니터링/격리검증 | D-035 §4에 "이번 라운드 설계 범위 밖"으로 서술만, O-번호 미등록 상태였음 | 신규 테넌트 온보딩 초기 데이터 셋업, 테넌트별 사용량 모니터링, 데이터 격리 자동검증 3가지 모두 부재 | High | O-170 |
| 역할 템플릿/이력 | `admin_roles`(§3.15)는 매핑만 정의 | 역할 템플릿 복제, 권한 변경 전용 이력 조회 없음 | Medium | O-171 |
| API 버전/Deprecation | ARCHITECTURE §3 "REST+OpenAPI" 한 줄만 | 버전관리 방식·deprecation 공지 절차 없음 | Medium | O-172 |
| 서비스 상태/장애공지 | ARCHITECTURE §6 미확정 | 상태페이지·Incident Communication 없음 | Medium | O-173 |
| 다국어 용어집 | `cms_translations`는 값만 저장 | 용어 일관성 관리(Glossary) 없음 | Low | O-174 |
| 정기 컴플라이언스 리포트 | 공제조합 보고센터(§5.7)는 법정 제출용 한정 | 경영진용 내부 정기 요약 리포트 표준 산출물 없음 | Low | O-175 |

- 이 라운드에서 [DECISIONS.md](DECISIONS.md) §2 Open Decisions 전체(O-002~O-169)를 점검해 **해소 1건**(O-062 — D-030으로 PV 계층 자체 폐기되어 소멸) **병합 3건**(O-009→O-010 동일쟁점, O-113 브랜드/제조사 부분→O-098로 통합, O-121→O-028 동일쟁점)을 정리하고, 전체에 **High 37 / Medium 111 / Low 14** 우선순위를 부여했다(상세는 [DECISIONS.md](DECISIONS.md) §2.1). O-121이 사라진 자리는 통합된 **O-028**(재가입 식별자+중복가입 정합성)이 대신하며, 본 문서가 원래 O-121에 부여했던 **High** 우선순위를 O-028이 승계한다.

## 3. 개발 준비도 점검 (Sitemap / Role Matrix / Wireframe / ERD / API Spec / UI Guideline / Design System / Coding Standard / Test Plan / Deployment Guide)

> **2026-06-25 갱신(D-063)**: 1차 점검(아래 "1차 평가" 컬럼) 이후, 같은 날 "개발 착수 준비" 라운드에서 이 10종을 모두 신규 문서로 작성했다. "2차 현황" 컬럼이 현재 상태다 — 모두 기존 PRD/ARCHITECTURE/DATABASE 텍스트를 그대로 옮기거나(MLM/정산/Workflow/ERP Core 로직 변경 없음) "권장안(미확정)"으로 명시했을 뿐 새 정책을 확정하지 않았다.

| 산출물 | 1차 평가(2026-06-25 오전) | 2차 현황(2026-06-25, D-063) |
|---|---|---|
| Sitemap | **전혀 없음** | **신설** — [SITEMAP.md](SITEMAP.md) (7개 메뉴그룹, 1~3Depth) |
| Role Matrix | **부분적**(O-042/O-066 미확정과 연동) | **신설** — [ROLE-MATRIX.md](ROLE-MATRIX.md) (22개 모듈별 표, 미확정 셀은 O-042/O-066 그대로 인용) |
| ERD(다이어그램) | **부분적(텍스트만)** | **신설** — [ERD.md](ERD.md) (13개 클러스터, Mermaid 다이어그램 15개, 117개 엔터티) |
| API Specification | **부분적** | **신설** — [API-SPEC.md](API-SPEC.md) (22개 모듈, 비동기 Job 패턴 일관 적용, 3개 모듈 완전 예시) |
| UI Guideline | **부분적** | **신설** — [UI-GUIDELINE.md](UI-GUIDELINE.md) (Typography/Color/Spacing 등, 수치는 권장값) |
| Design System | **전혀 없음**(O-169 미확정) | **신설** — [DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) (컴포넌트 30종, 라이브러리 선정은 여전히 O-169 미확정·권장안만 제시) |
| Coding Standard | **전혀 없음** | **신설** — [CODING-STANDARD.md](CODING-STANDARD.md) (폴더구조/네이밍/NestJS·Next.js 규칙/에러처리/로깅) |
| Test Plan | **전혀 없음** | **신설** — [TEST-PLAN.md](TEST-PLAN.md) (7개 영역: MLM/정산/35%게이트/쇼핑몰/Workflow/API분리검증/Idempotency) |
| Deployment Guide | **부분적** | **신설** — [DEPLOYMENT.md](DEPLOYMENT.md) (환경/배포순서/Migration/Rollback/Backup/HealthCheck/Monitoring, O-037/O-144/O-148 미확정 부분은 권장안) |
| Wireframe | **전혀 없음** | **신설** — [WIREFRAME.md](WIREFRAME.md) (6개 레이아웃 아키타입 × 28개 모듈 매핑) |

- **잔여 갭**: 문서가 생겼다고 모든 미확정이 해소된 것은 아니다 — Design System(O-169)의 라이브러리 선정, Role Matrix의 다수 셀(O-042/O-066), Deployment의 환경분리(O-148)·DR목표(O-144)는 **여전히 사업팀/기술 책임자 확정이 필요**하다. 전체 문서 목록·읽는 순서는 [MASTER-INDEX.md](MASTER-INDEX.md) 참조.

## 4. 신규 Open Decision

1차(5개 영역) 점검에서 **49건**(O-121~O-169), 2차(개발준비도) 점검에서 **6건**(O-170~O-175)을 [DECISIONS.md](DECISIONS.md) §2 Open Decisions 표에 등록했다(총 55건). 분량상 PRD.md/ARCHITECTURE.md/DATABASE.md 각 절에 개별 인용하지 않고, 해당 문서의 Open Questions/Open Decisions 섹션에는 본 문서로의 요약 링크만 추가했다(§6 참조). 전체 목록과 관련 문서는 [DECISIONS.md](DECISIONS.md) §2 표, 우선순위 분류는 §2.1을 참조.

## 5. 충돌/중복 발견

1차 점검에서는 기존 결정(D-001~D-061)·Open Decision(O-002~O-120)과의 직접적 충돌이 발견되지 않았다. 다만 ARCHITECTURE.md §10의 "환경 단계 수 및 명칭" 항목이 그 문서의 비공식 목록에만 있고 [DECISIONS.md](DECISIONS.md) 중앙 표에는 등록되지 않았던 **문서 관리 프로세스 갭**을 O-148로 정식화했다.

**2차 점검(D-063)에서 Open Decision 표 전체(O-002~O-169)를 재검토해 다음을 정리했다** (상세는 [DECISIONS.md](DECISIONS.md) §2.1):
- **해소 1건**: O-062(PV 추상화 계층 재검토) — D-030으로 PV 계층 자체가 폐기되어 질문이 소멸. 취소선 처리.
- **병합 3건**: O-009→O-010(정산 cut-off 시각/타임존, 동일쟁점 중복), O-113 브랜드/제조사 부분→O-098(제품 카탈로그 확장 범위에 이미 포함), O-121→O-028(재가입 식별자 정책과 이메일/휴대폰 중복가입이 동일한 "회원 식별 정합성" 쟁점).
- O-128(O-098 세분화), O-132(O-099 연계) 등은 중복이 아니라 후속 정밀화로 판단해 유지했다(1차 판단 유지).

## 6. 수정 대상 문서

| 문서 | 변경 내용 |
|---|---|
| [GAP-ANALYSIS.md](GAP-ANALYSIS.md) | 신규 생성(본 문서) |
| [DECISIONS.md](DECISIONS.md) | D-062 추가, O-121~O-169(49건) 등록 |
| [CHANGELOG.md](CHANGELOG.md) | 신규 버전 항목 추가 |
| [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) | 본 문서 참조 추가, 완성도 평가 요약 반영 |
| [PRD.md](PRD.md) | §8 Open Questions에 본 문서 요약 링크 추가 |
| [ARCHITECTURE.md](ARCHITECTURE.md) | §10 Open Decisions에 본 문서 요약 링크 추가, "환경 단계" 항목을 O-148로 정식화 |
| [DATABASE.md](DATABASE.md) | §7 Database Review에 본 문서 요약 링크 추가 |
| [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) | §6에 체크 항목 2건 추가, §13 Open Items에 O-154/O-155 추가 |
| [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) | §4/§10에 O-152/O-153 연계 표기 추가 |
| COMPENSATION-RULES.md, TASK-SPEC.md, DO-NOT-TOUCH.md | **1차 변경 없음** — 점검 결과 이 라운드에서 추가할 내용이 없음을 확인 |
| [SITEMAP.md](SITEMAP.md), [ROLE-MATRIX.md](ROLE-MATRIX.md), [ERD.md](ERD.md), [API-SPEC.md](API-SPEC.md), [WIREFRAME.md](WIREFRAME.md), [DESIGN-SYSTEM.md](DESIGN-SYSTEM.md), [UI-GUIDELINE.md](UI-GUIDELINE.md), [CODING-STANDARD.md](CODING-STANDARD.md), [TEST-PLAN.md](TEST-PLAN.md), [DEPLOYMENT.md](DEPLOYMENT.md) | **2차(D-063) 신규 생성** — §3 참조 |
| [MASTER-INDEX.md](MASTER-INDEX.md) | **2차(D-063) 신규 생성** — 전체 문서 색인/읽는 순서/최종 개발준비도 평가 |
| README.md(저장소 루트) | **2차(D-063) 개선** — 프로젝트 진입 문서로 보강 |

## 7. 우선순위 종합

| 우선순위 | 건수(1차 49건 + 2차 6건 = 55건) | 비중이 큰 영역 |
|---|---|---|
| High | 14건 (O-028(O-121 병합), O-127, O-128, O-129, O-136, O-137, O-144, O-153, O-158, O-159, O-164, O-170 + 개발 산출물 6종 — Sitemap/Role Matrix/ERD/API Spec/Coding Standard/Test Plan/Deployment Guide는 §3에서 이미 문서화 완료, 단 내부 미확정 항목은 잔존) | 회원 식별 정합성, 재고·카테고리·반품 기초 모델, CMS 상태관리, DR, Multi-Tenant 온보딩/격리, 개발 산출물 |
| Medium | 31건 | 대부분의 운영기능 보강(자동화/연계/표준화) |
| Low | 10건 | 운영 편의성 향상(알림/가시성/절차 정비) |

> 위 55건은 [DECISIONS.md](DECISIONS.md) §2.1의 ERP 전체 Open Decision 우선순위 분류(High 37/Medium 111/Low 14, O-002~O-175 전체 대상)의 부분집합이다 — 본 표는 GAP-ANALYSIS 출신 항목만, §2.1은 프로젝트 전체 Open Decision을 다룬다.

## 8. 개발 착수 전 반드시 보완해야 하는 항목 (Blocking)

> **2026-06-26 갱신(D-064)**: 본 절의 Blocker 후보가 [MASTER-INDEX.md](MASTER-INDEX.md) §5와 거의 같은 내용을 각자 다르게 나열하고 있어, 이 자체가 "문서 간 중복"이었다. 이제 **단일 출처는 [DECISIONS.md](DECISIONS.md) §2.2**다 — 17건을 Tier 1(착수 전 필수)/Tier 2(모듈별 필수)/Tier 3(Multi-Tenant 활성화 시 필수)로 분류해 두었다. 본 절과 MASTER-INDEX.md §5는 그 목록을 반복하지 않고 요약만 남긴다.

~~**개발 산출물 6종 작성**~~ — **완료(D-063)**: Sitemap/Role Matrix/ERD/API Spec/Coding Standard/Test Plan/Deployment Guide §3 참조. 단, 그 안의 미확정 항목은 [DECISIONS.md](DECISIONS.md) §2.2에서 Blocker로 추적한다 — 문서의 "존재"와 "내용 확정"은 다르다.

[DECISIONS.md](DECISIONS.md) §2.2 Tier 1(8건: O-022/O-028/O-127/O-128/O-144/O-148/O-164/O-169)이 "어느 모듈을 먼저 만들든" 영향을 주는 진짜 착수 전 블로커다. Tier 2(7건)는 해당 모듈 착수 전에만, Tier 3(2건)는 Multi-Tenant 활성화 시점에만 필요하다.

## 9. 구현 단계에서 추가 검토할 항목 (Non-blocking)

[DECISIONS.md](DECISIONS.md) §2.2 Blocker 목록(17건)에 포함되지 않은 나머지 Open Decision은 해당 모듈을 실제로 구현하는 시점에 함께 확정해도 전체 일관성에 영향을 주지 않는다 — 각 항목의 정확한 위치는 [DECISIONS.md](DECISIONS.md) §2 표, 항목별 난이도/선행조건/후속작업은 §10을 참조.

## 10. 항목별 영향도 / 난이도 / 개발시점 / 선행조건 / 후속작업

기준 — **영향도**: 다른 모듈/항목에 미치는 파급력(상=다수 모듈 영향, 중=해당 모듈 한정, 하=국지적). **난이도**: 구현 복잡도(상=신규 인프라·스키마 광범위 변경, 중=기존 테이블 확장, 하=설정값/문구 수준). **개발시점**: MVP전제(착수 전 확정 필요)/MVP(1차 출시 범위)/Phase2(후속)/후순위.

| O-번호 | 영향도 | 난이도 | 개발시점 | 선행조건 | 후속작업 |
|---|---|---|---|---|---|
| O-028(O-121 병합) | 상 | 중 | MVP전제 | 없음 | `members` 유니크 제약/식별자 정책 확정 → DATABASE.md §3.1 갱신 |
| O-122 | 중 | 중 | MVP | 직급폐기(D-030, 완료) | 세그먼트/태그 테이블 설계, O-109/§5.41 그룹과 통합 검토 |
| O-123 | 중 | 중 | Phase2 | 고객몰 도입여부(O-017) | `member_change_requests` change_type 추가 |
| O-124 | 하 | 중 | Phase2 | 없음 | SSO 연동방식 확정 후 System Settings 스키마 추가 |
| O-125 | 하 | 중 | Phase2 | O-042/O-066 | 위임 데이터모델 설계 |
| O-126 | 중 | 중 | Phase2 | File Manager(완료, D-049) | Import/Export 포맷·검증 규칙 정의 |
| O-127 | 상 | 상 | MVP전제 | 창고/3PL 연동 방식(O-033) | `inventory_ledger` 예약상태 컬럼/Job 설계 |
| O-128 | 상 | 중 | MVP전제 | O-098 | `product_categories` 트리 테이블 설계 |
| O-129 | 상 | 중 | MVP전제 | 청약철회 법적기준(LEGAL-CHECKLIST §4) | `returns` 상태 enum 확정 |
| O-130 | 중 | 하 | MVP | O-127 | `restock_notifications` 흐름과 통합 |
| O-131 | 중 | 중 | Phase2 | 패키지 엔진 변경 없음 확인(완료) | 신규 `product_bundles` 테이블 검토 |
| O-132 | 중 | 중 | Phase2 | O-099(쿠폰 구조) | 할인규칙 엔진 스키마 |
| O-133 | 중 | 중 | MVP | O-127 | `shipments` 1:N 구조 반영 |
| O-134 | 하 | 하 | 후순위 | O-127 | 알림 임계치 컬럼 추가 |
| O-135 | 중 | 중 | MVP | 정산 영향 분석 | order_item 단위 환불 흐름 |
| O-136 | 상 | 하 | MVP전제 | 없음 | `cms_pages`/`marketing_programs`에 메타필드 컬럼 추가 |
| O-137 | 상 | 중 | MVP전제 | Workflow Engine 연동범위 결정 | 상태 enum + 전이 규칙 정의 |
| O-138 | 중 | 상 | 후순위 | 트래픽 분할 인프라 | A/B 테스트 엔진 별도 검토 |
| O-139 | 중 | 상 | Phase2 | O-109 | 시퀀스 진행상태 테이블 설계 |
| O-140 | 중 | 하 | MVP | 없음 | `cms_translations` 상태 컬럼 추가 |
| O-141 | 중 | 중 | Phase2 | Webhook 여부(O-145) | 연계 메커니즘 확정 |
| O-142 | 중 | 중 | Phase2 | O-111 | 데이터소스 카탈로그 공유 설계 |
| O-143 | 하 | 하 | 후순위 | O-094 | 팝업 정책에 동일 규칙 적용 |
| O-144 | 상 | 상 | MVP전제 | Supabase 플랜/리전 정책 | RTO/RPO 목표 합의 → DEPLOYMENT.md 갱신 |
| O-145 | 중 | 상 | Phase2 | 보안 검토 | Webhook 서명/재전송 설계 |
| O-146 | 중 | 중 | Phase2 | 없음 | 연동별 쿼터 추적 컬럼 추가 |
| O-147 | 중 | 중 | Phase2 | 없음 | SLA/에스컬레이션 필드 추가 |
| O-148 | 상 | 하 | MVP전제 | 없음 | 환경 수·명명 확정 → DEPLOYMENT.md/ARCHITECTURE.md 갱신 |
| O-149 | 하 | 중 | Phase2 | 법무 검토 | 마스킹 정책 정의 |
| O-150 | 하 | 하 | 후순위 | 없음 | Scheduler 화면에 상태표시 추가 |
| O-151 | 중 | 하 | Phase2 | §5.1.1 라인노출 확정 | Report Builder 템플릿 추가 |
| O-152 | 중 | 중 | MVP | 세무사 확인 | 표준서식 템플릿 확정 |
| O-153 | 상 | 상 | MVP전제 | 보정엔트리 정책(완료) | 이의신청 워크플로우 연결 설계 |
| O-154 | 중 | 하 | Phase2 | 고객몰 도입여부 | LEGAL-CHECKLIST 체크 보강(완료, 본 라운드) |
| O-155 | 중 | 중 | Phase2 | 고객몰 도입여부 | 에스크로 업체 검토 |
| O-156 | 하 | 하 | 후순위 | 없음 | 고지자료 갱신 트리거 정의 |
| O-157 | 하 | 하 | 후순위 | 없음 | 정정 절차 문서화 |
| O-158 | 상 | 상 | MVP전제 | 없음 | CDN/리사이징 서비스 선정 |
| O-159 | 상 | 상 | MVP전제(Multi-Tenant 활성화 시) | O-090 | Job 큐 격리·쿼터 설계 |
| O-160 | 중 | 중 | Phase2 | O-055 | 아카이빙 정책 정의 |
| O-161 | 중 | 중 | Phase2 | 없음 | DLQ 설계 |
| O-162 | 중 | 상 | Phase2 | TH/JP 출시 시점(O-039) | 형태소분석기 선정 |
| O-163 | 하 | 중 | Phase2 | `member_ancestors` 도입 | 갱신 배치 설계 |
| O-164 | 상 | 중 | MVP전제 | 없음 | API-SPEC.md 페이지네이션 확정과 동기화 |
| O-165 | 중 | 중 | Phase2 | 없음 | 검색/필터 공통 컴포넌트 설계 |
| O-166 | 중 | 중 | Phase2 | 없음 | 테이블 컬럼 설정 저장 모델 |
| O-167 | 중 | 중 | MVP | 없음 | UI-GUIDELINE.md 접근성 기준 확정 |
| O-168 | 하 | 하 | 후순위 | 없음 | 업로드 컴포넌트 진행률 UI |
| O-169 | 상 | 하 | MVP전제 | 없음 | DESIGN-SYSTEM.md 권장안(shadcn/ui) 승인 |
| O-170 | 상 | 상 | MVP전제(Multi-Tenant 활성화 시) | O-090 | 온보딩 스크립트/모니터링 대시보드 설계 |
| O-171 | 하 | 중 | Phase2 | O-042 | 역할 이력 테이블 설계 |
| O-172 | 중 | 중 | MVP | 없음 | API-SPEC.md 버전정책 확정 |
| O-173 | 하 | 중 | Phase2 | 없음 | 상태페이지 도구 선정 |
| O-174 | 하 | 하 | 후순위 | 없음 | 용어집 테이블/엑셀 관리 검토 |
| O-175 | 하 | 하 | 후순위 | Report Builder 완료(D-054) | 정기 리포트 템플릿 정의 |

> 위 표는 GAP-ANALYSIS 출신 O-121~O-175(O-121은 O-028로 표기)만 다룬다 — 그 이전 O-002~O-120은 [DECISIONS.md](DECISIONS.md) §2.1의 우선순위 분류만 적용했고 영향도/난이도/개발시점까지 개별 평가하지는 않았다(범위가 너무 넓어 본 라운드에서는 GAP-ANALYSIS 항목에 한정).

## 11. 문서 중복 제거 보고서 (2026-06-26, D-064)

### 점검 방법

사용자가 예시로 든 4개 주제(패키지 자격조건/worker만 계산/append-only/조직이동)를 포함해, PRD/ARCHITECTURE/DATABASE/COMPENSATION-RULES/DO-NOT-TOUCH/API-SPEC/TEST-PLAN/SITEMAP/WIREFRAME/LEGAL-CHECKLIST 전반에서 동일 주제가 등장하는 위치를 모두 찾아 비교했다.

### 결론 — 심각한 중복 산문 재설명은 발견되지 않았다

| 주제 | 정식 정의(Single Source of Truth) | 다른 문서의 인용 방식 | 평가 |
|---|---|---|---|
| 패키지 자격조건 | [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.5/§4.1, [DATABASE.md](DATABASE.md) §3.24/§3.27.2 | PRD.md의 여러 화면(자격현황/제품판매수익내역/페어보너스현황)에서 짧게 언급 — 각각 다른 화면 맥락이라 정당한 반복, 원본 링크 포함 | 정상 |
| worker만 계산 | [ARCHITECTURE.md](ARCHITECTURE.md) §1.1, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3 | CODING-STANDARD/TEST-PLAN/API-SPEC/TASK-SPEC에서 1줄 + 링크로 반복 | **의도된 반복**(안전 가드레일은 여러 곳에서 상기시키는 것이 정상) |
| append-only | [DATABASE.md](DATABASE.md) §4 원칙 1, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2 | SETTLEMENT-RULES/PRD §5.44.7(Undo 예외)에서 1줄 + 링크로 반복 | 정상 |
| 조직 이동 | [PRD.md](PRD.md) §5.16, [ARCHITECTURE.md](ARCHITECTURE.md) §7.1 | DATABASE/API-SPEC/TEST-PLAN/WIREFRAME/LEGAL-CHECKLIST/SITEMAP에서 1줄 + 링크로 반복 | 정상 |

기존 인용들은 이미 "1줄 요약 + 원본 링크" 형태를 따르고 있었다 — 같은 내용을 문단 단위로 베껴 쓴 사례는 발견되지 않았다. 즉 이 프로젝트의 실제 위험은 "중복된 산문"이 아니라 **"어느 문서가 정본인지 빠르게 찾을 색인이 없었다"** 는 것이었고, 이는 D-064의 5개 표준화 카탈로그([BUSINESS-RULE-CATALOG.md](BUSINESS-RULE-CATALOG.md) 등)로 구조적으로 해결했다.

### 적용한 조치

- [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1에 "정식 정의와 전체 목록은 BUSINESS-RULE-CATALOG.md 참조" 안내 1건 추가(반복되는 가드레일 원칙들이 서로 다른 표현으로 보일 때의 기준점 제공).
- [DECISIONS.md](DECISIONS.md) §2.2에 개발 Blocker 통합 목록 신설 — 이전에는 본 문서 §8과 [MASTER-INDEX.md](MASTER-INDEX.md) §5가 Blocker 후보를 각자 따로 나열해 그 자체가 문서 간 중복이었다(메타 차원의 중복). 이제 단일 출처로 통합했다.

### 향후 권고

- 신규 라운드에서 새 규칙/원칙을 추가할 때는 "정식 정의 문서"를 먼저 정하고, 다른 문서에는 1줄 요약 + 링크만 추가하는 기존 패턴을 계속 따른다.
- [BUSINESS-RULE-CATALOG.md](BUSINESS-RULE-CATALOG.md)에 새 BR이 필요해지면 거기에만 추가하고, 다른 문서를 거슬러 올라가 일일이 갱신할 필요는 없다(Source Locator 패턴 유지).
