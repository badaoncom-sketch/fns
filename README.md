# FNS (Future Network System)

**Multi-Tenant MLM ERP Platform** — 다단계/네트워크 마케팅(직접판매) 기업을 위한 ERP. FNS 한 회사 전용이 아니라, 다양한 직접판매 기업이 사용할 수 있는 SaaS ERP를 목표로 한다.

> **현재 단계: Design Freeze 완료 → 개발 착수 단계.** 2026-06-26부로 설계가 동결([docs/DESIGN-FREEZE.md](docs/DESIGN-FREEZE.md), D-065)되었다. 코드는 아직 한 줄도 작성되지 않았다. **D-075(한국 공제조합 연동·E-Wallet·글로벌 결제)를 프로젝트의 마지막 보강 라운드로 간주하며, 설계·운영 준비·개발 준비가 모두 종료되었다. 이후 신규 요구사항은 Change Request(CR)로만 관리한다.** 전체 문서 지도는 **[docs/MASTER-INDEX.md](docs/MASTER-INDEX.md)** 를 가장 먼저 읽을 것.

## 프로젝트 소개

MLM(다단계/네트워크 마케팅) 보상플랜·정산은 이 ERP의 **모듈 중 하나**일 뿐이며, 전체 시스템은 일반 이커머스 ERP가 갖춰야 할 쇼핑몰·회원관리·주문·결제·배송·재고·CRM·CMS·마케팅·통계 기능과, 그 모든 모듈이 공유하는 ERP Core(Workflow Engine/API Center/File Manager/Scheduler/Dashboard·Report·Form Builder/System Settings/Audit) 엔진 계층을 포함한다. 자세한 배경은 [docs/PROJECT-CONTEXT.md](docs/PROJECT-CONTEXT.md) 참조.

## FNS 마케팅 플랜 한눈에 보기

1. 회원은 **무료로 가입**하고, 추천인 링크로 추천조직(LINE1~5)에 연결된다.
2. 매월 5만원 이상 구매하면 그 달의 **유니레벨 후원수당** 자격이 생긴다(매월 재판정).
3. 한 번이라도 패키지를 구매하면 **제품 판매수익·페어보너스** 자격을 영구히 얻는다(이후 직추천 회원의 구매·페어 성립마다 지급).
4. 두 자격은 서로 완전히 독립적이며, 모든 산정 결과는 35% 법적 한도 검증을 거쳐 **월정산**으로 지급된다.
5. 상세 흐름도/자격 비교표/실제 예시는 [docs/COMPENSATION-RULES.md](docs/COMPENSATION-RULES.md) §9~§11, 보상플랜 전체 규정은 §1~§8 참조.

## ERP 구조

```
ERP Platform
│
├─ ERP Core (공통 엔진 — 모든 업무 모듈이 사용)
│  ├─ Authentication / Authorization
│  ├─ Workflow Engine        — PRD §5.30
│  ├─ API Center             — PRD §5.31
│  ├─ File Manager           — PRD §5.32
│  ├─ Scheduler Center       — PRD §5.33
│  ├─ Notification Center    — PRD §5.34
│  ├─ Audit Center           — PRD §5.35
│  ├─ Dashboard / Report / Form Builder — PRD §5.36~5.38
│  └─ System Settings        — PRD §5.39
│
└─ 업무 모듈 (ERP Core를 사용)
   ├─ 쇼핑몰 (Shop)         — PRD §5.1.3, §5.21, §5.41, §5.43
   ├─ MLM (Compensation)    — COMPENSATION-RULES.md, PRD §5.1/§5.16
   ├─ 정산 (Settlement)     — SETTLEMENT-RULES.md
   ├─ 회원관리 / CRM / CMS / 마케팅 / 통계 — PRD §5.x
   └─ Multi-Tenant 설정      — PRD §5.26/§5.42
```

전체 모듈 구조는 [docs/PROJECT-CONTEXT.md](docs/PROJECT-CONTEXT.md) §1.1, 전체 메뉴 트리는 [docs/SITEMAP.md](docs/SITEMAP.md) 참조.

## 기술 스택

| 영역 | 선택 | 비고 |
|---|---|---|
| Frontend | Next.js | UI/조회 표시만, 계산 로직 없음 |
| Backend | NestJS | `api`(요청처리)/`worker`(실제 계산)/`scheduler`(Job 트리거)로 분리 |
| Database | Supabase (PostgreSQL) | Auth/Storage/Backup 포함, 유일한 source of truth |
| Queue/Cache | Redis (BullMQ) | source of truth 아님 — 소실되어도 비즈니스 데이터 손실 없어야 함 |
| Hosting | Railway | 1개 프로젝트 안에 5개 서비스(web/api/worker/scheduler/redis) |
| ORM/마이그레이션 | **미확정** | [DECISIONS.md](docs/DECISIONS.md) O-022 |
| UI 컴포넌트 라이브러리 | **권장안: shadcn/ui**(확정 아님) | [docs/DESIGN-SYSTEM.md](docs/DESIGN-SYSTEM.md) O-169 |

상세는 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) 참조. **핵심 원칙: 계산(수당/정산/세금/프로모션)은 `worker`만 수행하며, `web`/`api`/`scheduler`에는 절대 계산 로직을 두지 않는다** ([docs/DO-NOT-TOUCH.md](docs/DO-NOT-TOUCH.md)).

## 폴더 구조 (예정 — 구현 착수 시 적용, [docs/CODING-STANDARD.md](docs/CODING-STANDARD.md) 참조)

```
fns/
├─ docs/              # 설계 문서 전체 (현재 유일하게 존재하는 디렉터리)
├─ apps/
│  ├─ web/            # Next.js — Admin Console + Partner Portal + 쇼핑몰
│  ├─ api/            # NestJS — 요청 처리, Job 생성만
│  ├─ worker/         # NestJS — 수당/정산/세금/프로모션 등 실제 계산
│  └─ scheduler/      # NestJS — Job 트리거(cron)만
└─ packages/          # 공통 타입/유틸 (모노레포 권장안, 미확정)
```

## 문서 구조

`docs/` 34개 문서의 목적·상태·읽는 순서·의존관계는 **[docs/MASTER-INDEX.md](docs/MASTER-INDEX.md)** 에 모두 정리되어 있다. 빠른 링크:

- **설계 종료**: [DESIGN-FREEZE.md](docs/DESIGN-FREEZE.md)(Scope/변경원칙) · [RELEASE-ROADMAP.md](docs/RELEASE-ROADMAP.md)(v1.0/v1.1/v2.0)
- **개발 착수**: [DEVELOPMENT-KICKOFF.md](docs/DEVELOPMENT-KICKOFF.md)(시작기준/Phase 1~5) · [IMPLEMENTATION-GUIDE.md](docs/IMPLEMENTATION-GUIDE.md)(문서 읽기 순서)

- **비즈니스/도메인 규칙**: [PROJECT-CONTEXT.md](docs/PROJECT-CONTEXT.md) · [PRD.md](docs/PRD.md) · [COMPENSATION-RULES.md](docs/COMPENSATION-RULES.md)(MLM) · [SETTLEMENT-RULES.md](docs/SETTLEMENT-RULES.md) · [LEGAL-CHECKLIST.md](docs/LEGAL-CHECKLIST.md)
- **개발 산출물**: [ARCHITECTURE.md](docs/ARCHITECTURE.md) · [DATABASE.md](docs/DATABASE.md) · [ERD.md](docs/ERD.md) · [API-SPEC.md](docs/API-SPEC.md) · [SITEMAP.md](docs/SITEMAP.md) · [ROLE-MATRIX.md](docs/ROLE-MATRIX.md) · [WIREFRAME.md](docs/WIREFRAME.md) · [DESIGN-SYSTEM.md](docs/DESIGN-SYSTEM.md) · [UI-GUIDELINE.md](docs/UI-GUIDELINE.md) · [CODING-STANDARD.md](docs/CODING-STANDARD.md)
- **운영/검증**: [DO-NOT-TOUCH.md](docs/DO-NOT-TOUCH.md)(가드레일) · [TEST-PLAN.md](docs/TEST-PLAN.md) · [DEPLOYMENT.md](docs/DEPLOYMENT.md)
- **표준화 카탈로그**: [BUSINESS-RULE-CATALOG.md](docs/BUSINESS-RULE-CATALOG.md) · [EVENT-CATALOG.md](docs/EVENT-CATALOG.md) · [ERROR-CODE.md](docs/ERROR-CODE.md) · [STATE-MACHINE.md](docs/STATE-MACHINE.md) · [DATA-DICTIONARY.md](docs/DATA-DICTIONARY.md)
- **의사결정 추적**: [DECISIONS.md](docs/DECISIONS.md)(D-001~D-075 확정 결정 + O-002~O-205 Open Decision) · [CHANGELOG.md](docs/CHANGELOG.md) · [GAP-ANALYSIS.md](docs/GAP-ANALYSIS.md)(상용 ERP 대비 점검)
- **AI 작업 위임**: [TASK-SPEC.md](docs/TASK-SPEC.md) — Claude Code/Codex에게 작업을 지시할 때의 표준 형식

## ERP Core

Workflow/API Center/File Manager/Scheduler/Notification/Audit/Dashboard·Report·Form Builder/System Settings 12개 공통 엔진. 모든 업무 모듈이 사용하며, ERP Core는 업무 모듈에 의존하지 않는다. 상세: [PRD.md](docs/PRD.md) §5.28~§5.39, [API-SPEC.md](docs/API-SPEC.md).

## MLM Engine

Unilevel Sponsor Plan(추천조직 트리, LINE1~5). 후원수당·페어보너스·패키지 정책은 모두 관리자 설정 가능(하드코딩 없음). append-only 원장(`commission_records`)에 입력 스냅샷을 함께 저장해 사후 재현 가능. 상세: [COMPENSATION-RULES.md](docs/COMPENSATION-RULES.md), [DATABASE.md](docs/DATABASE.md) §3.5.

## 쇼핑몰

단일 통합 카탈로그(일반 쇼핑몰) + 회원몰(유지구매·정기배송·자동결제 센터). 패키지 상품도 별도 채널이 아니라 카탈로그 안의 상품 종류 중 하나다. 옵션/재고(LOT)/주문 병합·분리/부분교환/결제 재시도/리뷰/Bundle/검색/SEO·공유이미지 등 운영 고도화는 PRD §5.45~§5.46(D-069) 참조. 장바구니(`carts`/`cart_items`, D-072 — 오랜 MVP 데이터모델 갭 해소)/배송추적/취소·환불·교환·반품 UX는 §5.58 참조. 상세: [PRD.md](docs/PRD.md) §5.1.3, [SITEMAP.md](docs/SITEMAP.md).

## CMS

페이지/팝업/배너/FAQ/다국어 번역을 다루는 콘텐츠 관리. Marketing Program Engine(무제한 프로그램 카탈로그)과 연동. 상세: [PRD.md](docs/PRD.md) §5.19~§5.20. **Dynamic Board Engine**(D-074) — 공지사항/보도자료/갤러리/자료실/이벤트/홍보영상 등을 관리자가 코드 배포 없이 직접 생성하는 범용 게시판 엔진, 기존 CMS는 무변경. 상세: §5.67.

## API

REST 기준, 무거운 계산은 항상 `Job 생성 → 202 Accepted → 별도 상태조회` 패턴을 따른다(`api`에 계산 로직 금지). 31개 모듈의 엔드포인트 목록과 전역 컨벤션(페이지네이션/정렬/필터/에러포맷)은 [API-SPEC.md](docs/API-SPEC.md) 참조.

## ERD

149개+ 엔터티를 17개 도메인 클러스터로 묶어 Mermaid ER 다이어그램으로 정리했다(D-069 쇼핑몰 운영/SEO 신규 테이블 18종은 클러스터14; D-072의 `carts`/`cart_items`/`product_price_alerts` 3종/D-074의 Board Engine 5종/D-075의 공제조합·E-Wallet·글로벌결제 6종은 클러스터15~17로 D-075 라운드에서 일괄 catch-up 반영 완료). [ERD.md](docs/ERD.md) 참조 — 원본 텍스트 정의는 [DATABASE.md](docs/DATABASE.md) §3.

## Wireframe

List/Detail/Form/Dashboard/Workflow보드/쇼핑몰 6개 레이아웃 아키타입과 28개 모듈의 매핑. [WIREFRAME.md](docs/WIREFRAME.md) 참조.

## Roadmap

상세 릴리스 범위는 **[docs/RELEASE-ROADMAP.md](docs/RELEASE-ROADMAP.md)** 참조.

- **v1.0** — Core ERP(KR): 회원/MLM/정산/쇼핑몰/ERP Core/CMS/CRM. 착수 전 [DECISIONS.md](docs/DECISIONS.md) §2.2/§2.3 BLOCKER 15건 확정 필요.
- **v1.1** — BI/AI 보강(Dashboard 확장, Rule Designer, AI 기능 — 범위는 New Feature로 후속 정의)
- **v2.0** — Global(TH/JP/US 실제 출시)/Multi-Tenant 활성화/Marketplace/Open API/Mobile App

전체 Open Decision은 [DECISIONS.md](docs/DECISIONS.md) §2.3에서 BLOCKER/POST v1/FUTURE로 재분류되어 있다. 개발 준비도 최종 평가는 [MASTER-INDEX.md](docs/MASTER-INDEX.md) §5 참조.

## 현재 단계

✔ 기획 완료
✔ 설계 완료
✔ 개발 문서 완료
✔ **Design Freeze 완료** ([docs/DESIGN-FREEZE.md](docs/DESIGN-FREEZE.md), [DECISIONS.md](docs/DECISIONS.md) D-065)
✔ **개발 준비 완료** ([docs/DEVELOPMENT-KICKOFF.md](docs/DEVELOPMENT-KICKOFF.md), [DECISIONS.md](docs/DECISIONS.md) D-066)
✔ **쇼핑몰 운영/SEO 고도화 + 문서 동기화 완료** ([DECISIONS.md](docs/DECISIONS.md) D-069/D-070)
✔ **운영 완성도 향상 + 프로젝트 마무리 완료** ([DECISIONS.md](docs/DECISIONS.md) D-071)
✔ **쇼핑몰 UX·알림·운영자 대시보드 완성** ([DECISIONS.md](docs/DECISIONS.md) D-072)
✔ **운영 UX 및 고객 경험 완성** ([DECISIONS.md](docs/DECISIONS.md) D-073)
✔ **Dynamic Board Engine 설계 완료** ([DECISIONS.md](docs/DECISIONS.md) D-074)
✔ **한국 공제조합 연동·E-Wallet·글로벌 결제 설계 완료(설계 마지막 라운드)** ([DECISIONS.md](docs/DECISIONS.md) D-075)
⬜ 개발 진행
⬜ QA
⬜ 운영

Design Freeze 이후 모든 설계 변경은 Bug / Change Request / New Feature 3가지 트랙으로만 진행한다 — [docs/DESIGN-FREEZE.md](docs/DESIGN-FREEZE.md) §7~§9 참조.

## 개발 시작 안내

현재 프로젝트는 설계가 종료되었으며, 다음 단계는 **Codex를 이용한 구현 단계**다.

**구현 전 반드시 [docs/IMPLEMENTATION-GUIDE.md](docs/IMPLEMENTATION-GUIDE.md)를 먼저 읽는다.** 개발 시작 기준(Design Freeze 버전/Source of Truth/개발 원칙)과 Phase 1~5 구현 순서는 [docs/DEVELOPMENT-KICKOFF.md](docs/DEVELOPMENT-KICKOFF.md)에 정의되어 있다. 해당 Phase 착수 전 [DECISIONS.md](docs/DECISIONS.md) §2.2/§2.3의 BLOCKER 15건이 확정되어 있는지 먼저 확인한다.

이 시점 이후 Claude의 역할은 Business Rule 관리·법률 검토·Change Request 영향분석·문서 관리·Release Note 작성으로 전환된다 — 실제 구현은 Codex가 담당한다.

## GitHub 개발 Workflow

Issue, Project, Branch, Pull Request, Review, Release 운영 절차는 [docs/GIT-WORKFLOW.md](docs/GIT-WORKFLOW.md)를 따른다.

## 쇼핑몰 운영 / SEO / Analytics / Feed (D-069/D-070)

상용 ERP 수준의 쇼핑몰 운영 기능까지 보강을 완료했다 — 신규 Business Rule/MLM 정책 변경 없이 기존 구조(Workflow/Dashboard Builder/File Manager/Scheduler Center/External API Center) 재사용을 우선했다.

- **쇼핑몰 운영**: 상품옵션(SKU)/재고(LOT)/주문 병합·분리/부분교환/PG 재시도·가상계좌/묶음·예약배송/배송비정산/리뷰·Bundle — [PRD.md](docs/PRD.md) §5.45
- **SEO**: 상품별 SEO(자동생성 + 관리자 수동값 우선, BR-053)/사이트·페이지별 SEO/공유 이미지/SEO Preview — §5.46, SEO 운영 Dashboard(검색노출·클릭·CTR·순위) — §5.48
- **Digital Marketing 연동**: GA4/GTM/Search Console/Naver Search Advisor/Meta·TikTok Pixel/Google Ads Conversion 등 — 기존 API Center(`external_api_connections`) 재사용, **운영 설계만(실제 연동 미구현)** — §5.47
- **Feed**: Google Shopping/Facebook/Naver Shopping Feed 생성·다운로드·자동갱신 — §5.50

상세는 [PRD.md](docs/PRD.md) §5.45~§5.50, 동기화된 산출물은 [API-SPEC.md](docs/API-SPEC.md) §2.21~§2.25 / [ERD.md](docs/ERD.md) 클러스터14 / [DATA-DICTIONARY.md](docs/DATA-DICTIONARY.md) §7 / [ROLE-MATRIX.md](docs/ROLE-MATRIX.md) §24~§25 / [TEST-PLAN.md](docs/TEST-PLAN.md) §2.10 참조.

## 운영 완성도 보강 (D-071, 프로젝트 마지막 라운드)

새 기능이 아니라 **운영 편의성·유지보수성**을 높이는 설계만 추가했다 — 전부 기존 구조 재사용, 신규 Database 구조 없음.

- **Audit 운영 보고**: 사용자별/관리자별/기간별 변경 이력, PDF/Excel 다운로드, 검색/필터 — 기존 `audit_logs`+Report Builder 재사용([PRD.md](docs/PRD.md) §5.51)
- **Tenant 운영**: Backup/Restore/Clone/Export/Import — Supabase 백업/PITR·기존 Import/Export(O-126) 재사용, 실제 Backup 시스템 미구현(§5.52)
- **Feature Flag**: 국가별/Tenant별 기능 ON/OFF — 저장 방식은 신규 Open Decision **O-196**(유일한 신규 항목)으로만 추적(§5.53)
- **System Health Dashboard / Monitoring**: API/Worker/Scheduler/Redis/DB/Storage/Notification/Queue 상태 — Dashboard Builder 재사용(§5.54)
- **License 관리**: Tenant Plan/License Expire/사용량 — Multi-Tenant 활성화 시점(D-035)으로 이미 deferred된 영역(§5.55)

## 쇼핑몰 UX·알림·운영자 대시보드 완성 (D-072, 프로젝트 마지막 라운드)

개발 후 수정하면 복잡해지는 항목(장바구니/결제실패/택배추적/관심상품/상품비교/취소·환불·교환·반품/알림/운영자 대시보드/관리자 업무 Queue)을 개발 전에 설계로 고정했다 — 거의 전부 기존 구조 재사용, Database는 불가피한 곳에서만 최소 추가했다.

- **고객 알림**: 주문/배송/반품·교환/정기배송·자동결제/회원·MLM 전 영역 약 40종 알림을 기존 Notification Center `event_type` 카탈로그로 등록 — 신규 알림 엔진 없음([PRD.md](docs/PRD.md) §5.56)
- **Notification Template 관리**: 제목/변수/다국어/활성·비활성/테스트발송/미리보기/재발송 보강(§5.57)
- **장바구니**: `carts`/`cart_items`(신규 — 2026-06-26 이전부터 식별되어 있던 MVP 데이터모델 갭을 해소) + 옵션·수량변경/선택삭제/관심상품이동(§5.58.2)
- **배송 추적**: 택배사/송장번호/배송상태/타임라인 — 기존 `shipments` 컬럼 명료화([STATE-MACHINE.md](docs/STATE-MACHINE.md) §17)
- **취소/환불/교환/반품 UX**: 가능여부·예상금액 표시(파생), 진행 상태 타임라인(§5.58.5)
- **운영자 대시보드**: 오늘 지표/긴급 처리 지표/쇼핑몰 운영 지표 — Dashboard Builder 재사용(§5.59)
- **관리자 업무 Queue**: 결제실패/무통장입금확인/송장등록/승인대기 등 — 기존 테이블 횡단 조회, 신규 큐 테이블 없음(§5.60)

상세는 [PRD.md](docs/PRD.md) §5.56~§5.61, 동기화된 산출물은 [STATE-MACHINE.md](docs/STATE-MACHINE.md) §17~§20 / [API-SPEC.md](docs/API-SPEC.md) §2.26~§2.27 / [DATA-DICTIONARY.md](docs/DATA-DICTIONARY.md) §8 / [ROLE-MATRIX.md](docs/ROLE-MATRIX.md) §26 / [TEST-PLAN.md](docs/TEST-PLAN.md) §2.11 / [SITEMAP.md](docs/SITEMAP.md) / [WIREFRAME.md](docs/WIREFRAME.md) §2 참조. 신규 Open Decision은 O-197~O-199 3건뿐이다.

## 운영 UX 및 고객 경험 완성 (D-073, 프로젝트 마지막 라운드)

실제 서비스 운영에서 고객 경험(CX)과 운영 편의성을 높이는 마지막 보강 — **이번 라운드는 Database 구조를 전혀 변경하지 않았고(신규 테이블/컬럼 0건) 신규 Open Decision도 만들지 않았다(0건)**, 전부 기존 구조 재사용.

- **Abandoned Cart**: 24시간 미결제 장바구니 → 자동 알림 → 관리자 Dashboard → 자동 쿠폰 → 재방문 — 기존 Notification Center/`coupons`/Scheduler Center 재사용([PRD.md](docs/PRD.md) §5.62)
- **Saved Cart**: `carts`/`cart_items`(D-072)가 이미 영속 테이블이므로 별도 기능이 아니라 명명만 추가(§5.63)
- **Customer Timeline**: 회원 상세 화면에 가입·주문·결제·배송·문의·반품·패키지구매·정산·Lifestyle·로그인·알림을 시간순 표시 — 신규 테이블 없이 기존 데이터 federated 조회(§5.64)
- **My Dashboard**: 운영자 개인화 Dashboard — 기존 `dashboard_definitions.owner_admin_id`/`dashboard_widgets.position`이 이미 완전히 지원(§5.65)
- **Customer Service 보강**: CS Center 화면에 Customer Timeline 임베드(§5.66)
- **Dashboard 위젯**: 오늘 지표에 재고부족/SEO오류/Feed오류 추가, 전부 기존 테이블 파생 조회(§5.59)
- **Quick Action**: 주문조회/회원조회/상품등록/패키지등록/송장등록/환불승인/공지등록 — 기존 화면 바로가기, 승인 절차 우회 없음(§5.61)

상세는 [PRD.md](docs/PRD.md) §5.59/§5.61~§5.66, 동기화된 산출물은 [WIREFRAME.md](docs/WIREFRAME.md) §2/§3 / [API-SPEC.md](docs/API-SPEC.md) §2.2/§2.14/§2.22/§2.26 / [ROLE-MATRIX.md](docs/ROLE-MATRIX.md) §1/§10/§16/§22/§26 / [TEST-PLAN.md](docs/TEST-PLAN.md) §2.12 참조.

## Dynamic Board Engine (D-074, 프로젝트 마지막 라운드)

공지사항/보도자료/갤러리/FAQ/자료실/이벤트/홍보영상/교육자료/제품자료/인증자료/사용후기/회사소개/CSR/채용/IR 등을 관리자가 **코드 배포 없이 직접 생성**할 수 있는 범용 게시판 엔진 — **기존 CMS(`cms_pages`/FAQ/팝업/배너)는 변경하지 않았다.** 게시판 유형마다 별도 테이블을 만들지 않고 `boards`/`board_posts`(신규 5테이블) 공통 구조 + `board_type`/`metadata`(JSON) 확장으로 모든 유형을 수용한다.

- **신규 테이블 5종**: `boards`(게시판 정의 + `feature_flags` JSON으로 댓글/답글/파일첨부/이미지/동영상/다운로드/조회수/좋아요/공유/SEO/예약게시/승인후게시/카테고리/태그/검색/RSS ON-OFF) / `board_categories` / `board_posts`(공통 게시글 구조) / `board_post_comments`(자기참조로 답글 표현) / `board_post_likes`
- **기존 구조 재사용**: 첨부/이미지 → File Manager `files` / SEO·OG → `content_seo_metadata` / 다국어 → `cms_translations` / 조회수 → `content_view_events` / 승인후게시 → Workflow Engine(`BOARD_POST_APPROVAL`) / 예약게시 → Scheduler Center
- **게시판 권한**: 신규 권한 테이블 없이 기존 [ROLE-MATRIX.md](docs/ROLE-MATRIX.md) 7역할×11액션 체계를 그대로 적용(§27)

상세는 [PRD.md](docs/PRD.md) §5.67, 동기화된 산출물은 [DATABASE.md](docs/DATABASE.md) §3.58 / [API-SPEC.md](docs/API-SPEC.md) §2.28 / [ROLE-MATRIX.md](docs/ROLE-MATRIX.md) §27 / [WIREFRAME.md](docs/WIREFRAME.md) §2 / [SITEMAP.md](docs/SITEMAP.md) §5.3 / [TEST-PLAN.md](docs/TEST-PLAN.md) §2.13 참조. 신규 Open Decision은 **O-200**(기존 CMS의 Board Engine 통합 마이그레이션 여부) 1건뿐이다.

상세는 [PRD.md](docs/PRD.md) §5.67, 동기화된 산출물은 [DATABASE.md](docs/DATABASE.md) §3.58 / [API-SPEC.md](docs/API-SPEC.md) §2.28 / [ROLE-MATRIX.md](docs/ROLE-MATRIX.md) §27 / [WIREFRAME.md](docs/WIREFRAME.md) §2 / [SITEMAP.md](docs/SITEMAP.md) §5.3 / [TEST-PLAN.md](docs/TEST-PLAN.md) §2.13 참조. 신규 Open Decision은 **O-200**(기존 CMS의 Board Engine 통합 마이그레이션 여부) 1건뿐이다.

## 한국 공제조합 연동·E-Wallet·글로벌 결제 (D-075, 프로젝트 마지막 라운드)

글로벌 Multi-Tenant MLM ERP에 필수적인 3대 확장 기능 — **전부 Tenant별 선택 기능**이며, MLM 보상플랜·정산 계산 로직·기존 Business Rule·ERP Core 구조는 전혀 변경하지 않았다.

- **한국 공제조합 연동**: 직접판매공제조합/한국특수판매공제조합 동시 등록 가능 — 회원/후원관계/매출/수당/환불/반품/취소를 항목 단위로 전송·추적(`compliance_member_registrations`/`compliance_transmission_items`, 신규). 연동 등록 자체는 기존 API Center(`external_api_connections`) 재사용 — [PRD.md](docs/PRD.md) §5.68
- **전자지갑(E-Wallet)**: 회원별·통화별 지갑(KRW/USD/THB/JPY, 확장 가능), **반드시 append-only Ledger**(`wallet_transactions`) — 잔액은 항상 원장에서 파생, 직접 UPDATE 금지. 출금 승인은 기존 Workflow Engine 재사용. 후원수당 1차 산정·35% 법적 한도 검증·세금 계산은 전혀 변경 없음 — 지갑 적립은 정산이 확정한 금액을 인용하는 추가 지급 채널일 뿐 — §5.69
- **글로벌 결제**: 한국 결제(기존, D-069 그대로)/태국 PromptPay/일본 신용카드/Stripe/PayPal — `external_api_connections`에 `country_code` 컬럼만 추가. 인바운드 Webhook 수신은 기존 아웃바운드 로그와 방향이 반대라 `payment_webhook_events` 1개 테이블만 신규 — §5.70

상세는 [PRD.md](docs/PRD.md) §5.68~§5.70, 동기화된 산출물은 [DATABASE.md](docs/DATABASE.md) §3.59~§3.61 / [STATE-MACHINE.md](docs/STATE-MACHINE.md) §21~§24 / [API-SPEC.md](docs/API-SPEC.md) §2.29~§2.31 / [DATA-DICTIONARY.md](docs/DATA-DICTIONARY.md) §11 / [ERD.md](docs/ERD.md) 클러스터17 / [ROLE-MATRIX.md](docs/ROLE-MATRIX.md) §28~§30 / [TEST-PLAN.md](docs/TEST-PLAN.md) §2.14 참조. 신규 Open Decision은 **O-201~O-205 5건**이다(정산↔지갑 분배/포인트↔지갑 전환/결제 우선순위/PG사 선정/Webhook 서명검증).

**D-075를 끝으로 설계를 최종 종료하며, 이후 신규 요구사항은 Change Request(CR)로만 관리한다.**
