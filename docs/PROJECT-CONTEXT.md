# PROJECT-CONTEXT.md

> 상태: Draft v0.18 (D-046 — §1.1 ERP 모듈 구조를 2계층(ERP Core 엔진 + 업무 모듈)으로 재구성, ERP Core 12개 엔진 추가) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 이 문서는 FNS 프로젝트의 전체 맥락을 제공하는 최상위 문서입니다. 다른 모든 문서는 이 문서를 전제로 작성됩니다.

## 1. 프로젝트 한 줄 정의

**FNS(Future Network System)** 는 **ERP 플랫폼**이다. MLM(다단계/네트워크 마케팅) 보상플랜·정산은 이 ERP의 **모듈 중 하나**이며, 전체 시스템은 일반 이커머스 ERP가 갖춰야 할 쇼핑몰·회원관리·주문·결제·배송·재고·CRM·CMS·마케팅·통계·설정 기능을 모두 포함한다(확정 — [DECISIONS.md](DECISIONS.md) D-037). 회원(파트너/디스트리뷰터) 관리, 조직(스폰서 트리) 관리, 후원수당(보상플랜) 계산, 정산(커미션 지급), 주문/매출 관리, 법적 컴플라이언스를 하나의 시스템에서 처리하는 것을 목표로 한다.

### 1.1 ERP 모듈 구조 (확정 — [DECISIONS.md](DECISIONS.md) D-037, D-046로 ERP Core 계층 추가)

FNS ERP는 **2개 계층**으로 구성된다 — ① 모든 업무 모듈이 공통으로 사용하는 **ERP Core**(엔진 계층), ② 실제 업무 도메인을 다루는 **업무 모듈**(쇼핑몰/MLM/CMS 등). 모듈/엔진은 모두 **논리적 도메인 경계**(NestJS 모듈 단위)이며, §6.1의 5개 물리 서비스(web/api/worker/scheduler/redis) 구조를 대체하지 않는다.

```
ERP Platform
│
├─ ERP Core (공통 엔진 — 모든 업무 모듈이 사용, [DECISIONS.md](DECISIONS.md) D-046)
│  ├─ Authentication       — 기존 Supabase Auth(D-002)를 ERP Core 엔진으로 재인식, PRD §5.29
│  ├─ Authorization        — 기존 RBAC(§5.6.4)를 ERP Core 엔진으로 재인식, PRD §5.29
│  ├─ Workflow Engine      — 신규, PRD §5.30
│  ├─ API Center           — 신규, PRD §5.31
│  ├─ Scheduler Center      — 신규(기존 scheduler 서비스의 관리 UI), PRD §5.33
│  ├─ Notification Center  — 기존(§5.12)을 CMS에서 ERP Core로 재배치 + 보강, PRD §5.34
│  ├─ Audit Center         — 신규(기존 audit_logs의 관리 UI), PRD §5.35
│  ├─ File Manager         — 신규, PRD §5.32
│  ├─ Dashboard Builder    — 신규, PRD §5.36
│  ├─ Report Builder       — 신규, PRD §5.37
│  ├─ Form Builder         — 신규, PRD §5.38
│  └─ System Settings      — 신규(분산된 설정의 허브) + 보안정책, PRD §5.39
│
└─ 업무 모듈 (ERP Core를 사용 — 직접 구현 중복 금지)
   ├─ 쇼핑몰        (Shop)         — PRD §5.1.3, §5.21, §5.41(Phase 2 보강), §5.43(이미지/미디어 관리 보강)
   ├─ 회원관리      (Member)       — PRD §5.1/§5.3/§5.6
   ├─ 주문관리      (Order)        — PRD §5.1.3, §3.3(DATABASE)
   ├─ 결제관리      (Payment)      — PRD §5.1.3.3(자동결제), §5.21(쇼핑몰 결제)
   ├─ 배송관리      (Logistics)    — PRD §5.5
   ├─ 재고관리      (Inventory)    — PRD §5.5
   ├─ CRM          (Customer)     — PRD §5.11(CS Center) → §5.40(CRM Center로 확장)
   ├─ CMS          (Content)      — PRD §5.10, §5.19(대폭 확장)
   ├─ 마케팅        (Marketing)    — PRD §5.1.5/§5.20(Marketing Program Engine), §5.17(추천링크)
   ├─ MLM          (Compensation) — COMPENSATION-RULES.md 전체, PRD §5.1/§5.16
   ├─ 정산          (Settlement)   — SETTLEMENT-RULES.md, PRD §5.7(공제조합)
   └─ 통계          (Analytics)    — PRD §5.18(35%모니터링), §5.25(관리자 통계) — Dashboard/Report Builder(ERP Core)를 사용해 시각화/출력
```

- **"설정" 모듈은 폐지하지 않고 ERP Core의 System Settings로 흡수한다(D-046)** — 기존 §5.27(관리자 설정 일원화)·§5.6.4(권한)는 그대로 유지되며, System Settings가 이들을 위한 **허브 화면**을 추가로 제공한다. 중복이 아니라 재배치다.
- **MLM은 여전히 ERP의 한 업무 모듈이다(D-037, 변경 없음)** — 다른 업무 모듈은 MLM 없이도 독립적으로 동작할 수 있어야 한다(일반 이커머스 ERP로도 사용 가능). MLM 모듈은 쇼핑몰(주문/결제) 모듈에 의존한다.
- **ERP Core는 업무 모듈에 의존하지 않는다(D-046, 확정 원칙)** — Workflow Engine/API Center/File Manager 등은 어떤 업무 모듈(쇼핑몰/MLM/CMS)이 존재하지 않아도 동작할 수 있는 순수 엔진이다. 의존 방향은 항상 "업무 모듈 → ERP Core"이며 반대 방향은 금지한다.
- **모듈/엔진 간 연결은 독립성을 유지한 채 API·설정으로 이루어진다(D-037 원칙 유지)** — 식별자(예: `package_id`)와 정책 설정으로 연결하며, 내부 테이블을 직접 참조하지 않는다.
- **ERP UX Standard(신규, D-061)는 위 다이어그램과 별개의 계층이다** — 위 ERP Core/업무 모듈 구조는 **백엔드(api/worker) 모듈 경계**를 다루지만, ERP UX Standard는 **프론트엔드(web) 전용 공통 UX 컴포넌트**(Confirm/Warning Dialog, Toast, Loading, Bulk Action, Undo 등)로 관리자/회원/쇼핑몰/CMS 화면 전체가 공유한다. 어떤 업무 모듈·MLM/정산/Workflow의 기존 로직도 변경하지 않는다. 상세는 [PRD.md](PRD.md) §5.44, [ARCHITECTURE.md](ARCHITECTURE.md) §2.1.1 참조.
- **상용 ERP 수준 Gap Analysis(D-062)를 1차로 수행했다** — "FNS 전용이 아닌 다양한 직접판매 기업이 사용할 Multi-Tenant MLM ERP Platform" 목표 대비 점검한 결과는 [GAP-ANALYSIS.md](GAP-ANALYSIS.md)에 기록한다. 결론: **기능 설계 완성도는 높음**(정성 평가 약 90% 내외, MLM 핵심 도메인뿐 아니라 ERP Core/CMS/CRM/쇼핑몰까지 폭넓게 커버)이나 **구현 착수 준비도는 낮음~중간**(ERD 다이어그램/API Specification/Coding Standard/Test Plan/Deployment Guide 등 개발 산출물이 아직 없거나 텍스트 수준에 불과). 신규 발견 갭 49건은 [DECISIONS.md](DECISIONS.md) O-121~O-169로 등록했으며, 이번 라운드에서 기능을 직접 추가하지는 않았다.
- **개발 착수 준비 문서 세트(D-063)를 후속으로 완성했다** — Gap Analysis(D-062)가 식별한 산출물 공백 9종(Sitemap/Role Matrix/ERD/API Spec/Wireframe/Design System/UI Guideline/Coding Standard/Test Plan/Deployment Guide)을 모두 신규 문서로 작성하고, [MASTER-INDEX.md](MASTER-INDEX.md)(전체 문서 색인·읽는순서·의존관계도·최종 개발준비도 평가)를 신설했다. 모든 신규 문서는 기존 1차 설계(PRD/ARCHITECTURE/DATABASE/DECISIONS)를 재구성한 것이며 새 정책을 만들지 않았다. Open Decision 전체(O-002~O-169)도 이번에 정리했다(해소 1건, 병합 3건, 우선순위 분류 — [DECISIONS.md](DECISIONS.md) §2.1) 및 추가 갭 6건(O-170~O-175)을 등록했다. **최종 개발 준비도 평가는 [MASTER-INDEX.md](MASTER-INDEX.md) §5 참조.**
- **개발 착수 전 최종 안정화(D-064)를 완료했다** — [BUSINESS-RULE-CATALOG.md](BUSINESS-RULE-CATALOG.md)/[EVENT-CATALOG.md](EVENT-CATALOG.md)/[ERROR-CODE.md](ERROR-CODE.md)/[STATE-MACHINE.md](STATE-MACHINE.md)/[DATA-DICTIONARY.md](DATA-DICTIONARY.md) 5개 표준화 카탈로그를 신설해 "어느 문서가 정본인지" 찾는 비용을 낮췄고, 흩어져 있던 개발 Blocker를 **[DECISIONS.md](DECISIONS.md) §2.2 단일 목록(17건, 3-Tier)** 으로 통합했다. 문서 중복 점검 결과 심각한 중복 산문은 없었음을 확인했다([GAP-ANALYSIS.md](GAP-ANALYSIS.md) §11). 신규 기능/Open Decision은 추가하지 않았다. **Tier 1 8건(O-022/O-028/O-127/O-128/O-144/O-148/O-164/O-169)이 코드 작성 전 최우선 확정 대상이다.**
- **기존 전용 구조는 강제로 ERP Core로 이전하지 않는다(D-046)** — 조직 이동(`organization_transfer_logs`, D-020 — 9개 사유코드·제한된 승인권자라는 법적 요구사항이 있어 범용 Workflow Engine으로 단순화하면 위험), 회원 생애주기 변경(`member_change_requests`, D-006) 등은 이미 같은 패턴(요청→Snapshot→영향분석→승인)을 선구현한 사례로 인정하되, 데이터 모델은 그대로 유지한다. ERP Core는 **신규 워크플로우**(환불/반품/교환/전자결재 등)부터 적용한다.
- 본 모듈 구조는 기존 문서의 섹션 번호를 재배치하지 않는다 — 각 모듈/엔진의 상세 스펙은 계속 해당 문서(주로 [PRD.md](PRD.md))의 기존/신규 섹션에 위치하며, 본 절은 그 위치를 안내하는 **색인**이다.

## 2. 배경 및 목적

- 다단계/네트워크 마케팅(직접판매) 사업자는 일반 이커머스와 달리 **조직 구조(스폰서 트리)**, **후원수당 계산**, **정산 시 세금 처리**, **방문판매 등에 관한 법률(이하 방문판매법) 준수**라는 도메인 특화 요구사항을 갖는다.
- 기존에는 엑셀, 레거시 시스템, 외부 솔루션 조합으로 운영되는 경우가 많아 정확성·확장성·법적 추적성에 한계가 있다.
- FNS는 이 도메인 요구사항을 코드와 문서로 명확히 정의하고, 작은 단위로 검증 가능한 방식으로 구축하는 것을 목표로 한다.
- **목표 시장은 FNS 한 회사가 아니다(확정, D-035/D-037)** — FNS를 포함한 **다양한 직접판매 기업이 사용할 수 있는 Multi-Tenant SaaS ERP**를 지향한다. 이 때문에 모든 정책(수당/패키지/포인트/프로그램/쇼핑몰/CMS)은 코드에 하드코딩하지 않고 관리자 설정으로 제어해야 한다는 원칙이 문서 전반에 일관되게 적용된다(§1.1, §5.27 참조).

## 3. 비즈니스 도메인 요약

| 개념 | 설명 |
|---|---|
| 회원(파트너/디스트리뷰터) | 제품을 구매/판매하고 신규 회원을 후원(추천)할 수 있는 주체 |
| 추천조직 (= 스폰서 트리) | 회원 간 추천(후원) 관계로 형성되는 단일 조직 구조 (상위-하위). FNS는 **Unilevel Sponsor Plan**(확정, [DECISIONS.md](DECISIONS.md) D-008)을 채택하여 이 트리가 보상플랜의 유일한 조직 구조이며, 별도의 포지션/레그 개념은 없다 |
| LINE1~LINE5 | 특정 회원을 기준으로 추천조직을 추천 깊이별로 구분한 단위. LINE1=본인이 직접 추천한 회원, LINE5=5단계 하위까지 ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3) |
| 내 조직 (회원용 메뉴명) | 회원이 본인의 LINE1~LINE5, 조직매출, 조직수당, 조직성장을 확인하는 화면의 고정 명칭 (확정 — [PRD.md](PRD.md) §5.1.1) |
| 조직도 (관리자 전용 용어) | 관리자(Admin Console)가 전체 추천조직 트리를 조회하는 기능/용어. **회원 화면에는 사용하지 않음.** "추천조직도"라는 용어는 어디에서도 사용하지 않는다 |
| 후원수당 (= 조직수당) | LINE1~LINE5 추천조직 실적을 기준으로 회원에게 지급되는 수당. 회원 화면에는 "조직수당"으로 표시한다 |
| 자격 (2종, 서로 독립) | 회원가입은 무료이며 구매가 가입 조건이 아니다. **① 유니레벨 후원수당 자격**: 매월 5만원 상당 제품 구매로 그 달의 수령 자격이 결정된다(`members.status`와 독립된 월 단위 자격, 미달 시 회원 자격은 유지되고 그 달의 수령 대상에서만 제외). **② 제품 판매수익·페어보너스 자격**: 본인이 먼저 "자격 인정 패키지"를 구매한 적이 있어야 하며, 1회 충족으로 영구 획득한다(D-033 — 어떤 패키지가 자격을 부여하는지는 패키지별 정책으로 설정, FNS 현재 패키지는 자격 인정으로 설정됨). **하나를 충족해도 다른 하나가 자동으로 충족되지 않는다** (확정 — [DECISIONS.md](DECISIONS.md) D-028/D-033, D-024 정정, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5) |
| 패키지 | 회원가입 후 선택적으로 구매 가능한 고가 제품 세트 — **개수 제한 없이 무제한 등록 가능**하며, 패키지마다 추천수당/페어보너스/유니레벨포함/자격부여 여부를 독립적으로 설정한다(확정 — [DECISIONS.md](DECISIONS.md) D-033, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.0). **FNS 현재 운영 패키지(1종, 명칭 미정)**: 가격 400만원, 구매자의 추천인이 본인도 자격 인정 패키지를 구매한 적이 있는 경우에만 구매 시 **제품 판매수익**(25%, 100만원)을 받으며, 동일 추천인 산하 동일 패키지 구매자끼리 30일 내 구매 순서로 짝지어지면 **페어보너스**(Pair당 200만원)가 추가 지급된다 — 추천인 본인이 자격 인정 패키지 구매 이력이 없으면 둘 다 지급되지 않고 소멸한다(D-028). 이 패키지의 매출 자체는 유니레벨 후원수당 산정에서는 제외된다 (확정 — D-024/D-028/D-033, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1) |
| 회원몰 / 일반 쇼핑몰 | **일반 쇼핑몰**: 건강기능식품/화장품/생활용품/프로모션 상품/패키지 상품 등 모든 상품이 존재하는 **단일 통합 카탈로그**(회원·일반 고객 공통, 비회원 접근 허용 여부만 미확정). **회원몰**: 상품 판매 채널이 아니라 유지구매센터×정기배송센터×자동결제센터로 구성된 회원 전용 대시보드 — 실제 구매는 일반 쇼핑몰에서 일어난다 (확정 — [DECISIONS.md](DECISIONS.md) D-034, D-031 정정, [PRD.md](PRD.md) §5.1.3) |
| Lifestyle Program (= "+알파" 보너스) | "+알파" 보너스(Lifestyle Bonus, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.2)의 **회원/쇼핑몰 노출용 명칭** — 마이오피스 적립 현황뿐 아니라 쇼핑몰 메인(슬라이드/프로모션/이벤트 배너)·회원몰 메인(프로그램 배너·포인트 현황·진행률)·전용 상세페이지(소개/이미지갤러리/포인트정책/누적현황/참여조건/첨부파일/FAQ)로 노출되는 마케팅·동기부여 콘텐츠다. 관리자/엔진 내부 용어는 그대로 "+알파" 보너스를 사용한다 (확정 — [DECISIONS.md](DECISIONS.md) D-036, [PRD.md](PRD.md) §5.1.5) |
| 정기배송 (Recurring Delivery) | 회원몰에서 특정 제품을 일정 주기로 자동결제·자동주문받는 구독형 주문. §3.5.2 유니레벨 자격(매월 5만원 구매)을 안정적으로 유지시키는 핵심 기능(MVP)으로 확정 ([PRD.md](PRD.md) §5.1.3, [DECISIONS.md](DECISIONS.md) D-031) |
| 추천 링크 | 모든 회원이 갖는 고유 가입 링크(`fns.com/r/{memberid}`, `fns.com/join?ref={memberid}`). 이 링크로 가입하면 링크 소유 회원이 추천인으로 자동 연결되어 `members.sponsor_id`가 설정된다 — 신규 가입 시점의 최초 1회 설정이며, 가입 후 추천인 변경은 여전히 관리자 전용 조직 이동(D-020)을 통해서만 가능하다 (확정 — [DECISIONS.md](DECISIONS.md) D-025, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6) |
| 정산 | 계산된 후원수당을 실제 금액으로 확정하고 세금을 처리하여 지급하는 절차 |
| 회원 유형 | 회원의 법적 주체 유형 — **개인 / 사업자 / 법인 / 외국인** 4종으로 확정([DECISIONS.md](DECISIONS.md) D-011). 유형에 따라 가입 시 필요한 본인확인 정보, 세무 처리 방식이 달라진다 |
| 국가 | 회원/조직이 속한 국가 시장. **KR/TH/JP/US** 4개국을 지원 대상으로 확정([DECISIONS.md](DECISIONS.md) D-011/D-023) — 실제 운영 시작 국가·순서는 미확정 |
| Reserved Country | 향후 확장을 위해 보관하지만 현재 활성 국가 구조에서는 제외된 국가. **CN(중국)이 현재 유일한 Reserved Country**다 — 데이터/문서는 삭제하지 않고 보관하며, 활성화하려면 별도 의사결정이 필요하다 ([DECISIONS.md](DECISIONS.md) D-023) |
| 마케팅 플랜 (버전) | 후원수당 산정에 쓰이는 LINE별 비율 등 파라미터의 집합. 국가별로 별도 버전을 가지며, 시간에 따라 새 버전이 발효될 수 있다 (과거 정산에는 당시 발효된 버전이 적용됨) |
| 공제조합 (= 국가별 규제 보고 대상 기관) | 한국 방문판매법상 다단계판매업자가 가입해야 하는 소비자피해보상 기구. FNS에서는 이 기관(및 타 국가의 동등 기관)에 제출할 보고서를 관리하는 기능을 **'공제조합 보고센터'** 라는 화면명으로 제공한다 ([PRD.md](PRD.md) §5.7) |
| 센터 (Center) | 국가 하위의 지역 거점(예: 서울센터, 부산센터, 방콕센터, 도쿄센터). 후원수당 계산(LINE1~5)에는 영향을 주지 않는 **집계·관리 단위**다. 도입 여부 자체는 미확정 ([PRD.md](PRD.md) §5.9) |
| Document Center | 약관/보상플랜안내/교육자료/공지사항 및 회원 개인 문서(정산자료/지급명세서/사업자자료/원천징수자료)를 관리하는 화면명 ([PRD.md](PRD.md) §5.10) |
| Customer Service Center | 1:1문의/정산문의/수당문의/명의변경문의/이의신청/불만접수를 처리하는 화면명. 강제탈퇴 이의제기 절차(O-027)를 구현하는 창구이기도 하다 ([PRD.md](PRD.md) §5.11) |
| Notification Center | Email/SMS/KakaoTalk/Push 알림 발송·템플릿·이력을 관리하는 화면명 ([PRD.md](PRD.md) §5.12) |
| 전자서명 / 동의 이력 | 명의변경·탈퇴·강제탈퇴 승인의 선행조건이 되는 전자서명, 그리고 약관/개인정보 동의를 시점별로 기록하는 별도 이력 ([PRD.md](PRD.md) §5.13). 조직 이동(관리자 전용, §5.16)은 회원 행위가 아니므로 전자서명 대상이 아니다 |
| 회원 활동 이력 | 회원이 본인 화면에서 보는 최근 로그인/주문/수당/정산/프로모션 — 관리자용 `audit_logs`와는 목적이 다른 별도 로그 ([PRD.md](PRD.md) §5.14) |
| Rule Designer | 코드 배포 없이 관리자가 LINE 비율 등 국가별 규칙을 안전하게(시뮬레이션·승인 거쳐) 변경할 수 있는 도구. 도입을 권고했으나 최종 채택 여부는 미확정 ([PRD.md](PRD.md) §5.15) |

> 본 문서에서 사용하는 용어의 확정 정의는 [COMPENSATION-RULES.md](COMPENSATION-RULES.md), [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)를 따른다. (PROMOTION-RULES.md는 D-030으로 폐기됨)

## 4. 타겟 시장 (다국가 구조 — [DECISIONS.md](DECISIONS.md) D-011/D-023)

- FNS는 **KR(대한민국) / TH(태국) / JP(일본) / US(미국)** 4개 국가를 지원하는 것을 전제로 아키텍처·데이터 모델을 설계한다.
- **1차 운영(라이브) 시장은 KR**이며, "정산", "후원수당" 등 용어 및 방문판매법은 KR 기준으로 우선 정의되어 있다. TH/JP/US는 데이터 모델·아키텍처 수준에서 확장 가능하도록 설계하되, 각국의 실제 법규(세금/직접판매 관련 법/마케팅 플랜 한도 등)는 **미확정**이다.
- **CN(중국)은 1차 및 중기 계획에서 제외**하며, **"Reserved Country"** 상태로 보관한다 ([DECISIONS.md](DECISIONS.md) D-023) — 데이터/문서를 삭제하지 않되, 현재 활성 국가 구조(4개국)에는 포함하지 않는다. 향후 CN을 다시 활성화하려면 별도의 명시적 의사결정이 필요하다.
- 실제 운영 시작 국가(TH/JP/US)의 순서·시점, 다통화 지원 범위는 **미확정**이다 → [DECISIONS.md](DECISIONS.md)의 Open Decisions 참조.

## 5. 주요 이용자 그룹

| 그룹 | 설명 |
|---|---|
| 본사 운영자 (SuperAdmin) | 전체 국가·전체 기능에 접근 가능한 최상위 관리자 |
| 국가 관리자 (CountryAdmin) | 특정 국가에 스코프가 한정된 관리자 — 본인 국가의 회원/정산/보고서만 관리 ([PRD.md](PRD.md) §5.6) |
| 센터 관리자 (CenterAdmin) | 특정 센터에 스코프가 한정된 관리자 — 센터 구조 도입 시에만 존재 ([PRD.md](PRD.md) §5.9) |
| 기능별 관리자 (정산담당/법무담당/고객지원 등) | 특정 업무 영역에 한정된 권한을 가진 관리자 — 세부 역할 목록은 미확정 |
| 파트너 (Member/Distributor) | 제품을 구매·판매하고 후원수당을 받는 회원. 개인/사업자/법인/외국인 4개 유형 중 하나로 가입 |
| 고객 (End Customer) | 파트너를 통하거나 직접 제품을 구매하는 일반 소비자 (포함 여부 미확정) |

## 6. 기술 스택 (확정)

| 영역 | 선택 |
|---|---|
| 프론트엔드 | Next.js |
| 백엔드 | NestJS |
| 데이터베이스 / 인증 / 스토리지 | Supabase (PostgreSQL 기반) |
| 호스팅/배포 | Railway — **1개 프로젝트, 5개 서비스 구조로 확정** (§6.1) |
| 캐시 / 큐 | Redis |
| 개발 도구 | VS Code, Claude Code, Codex |

기술 스택 선정 이유 및 대안 검토는 [DECISIONS.md](DECISIONS.md), 구조적 적용 방식은 [ARCHITECTURE.md](ARCHITECTURE.md)를 참조한다.

### 6.1 서버 구조 원칙 (확정 — [DECISIONS.md](DECISIONS.md) D-010)

FNS의 서버는 Railway 프로젝트 1개 안에 **web / api / worker / scheduler / redis** 5개 서비스로 구성하고, Supabase는 PostgreSQL/Auth/Storage/Backup을 담당한다. 핵심 원칙(위반 금지):

- **web**(Next.js)은 수당 등 어떤 계산도 하지 않는다 — UI만 담당한다.
- **api**(NestJS)는 request handler에서 무거운 계산을 하지 않는다 — **Job 생성만** 담당한다.
- **worker**(NestJS)가 Job을 실제로 처리한다 (수당/정산/세금/프로모션/보고서).
- **scheduler**(NestJS)는 Job을 **트리거만** 한다 (자동정산/자동프로모션/자동보고서) — 직접 계산하지 않는다.
- **redis**는 BullMQ Queue/Cache/Job Tracking/Retry 용도이며 **source of truth가 아니다.**
- **최종 데이터는 Supabase PostgreSQL에만 저장한다.**

상세 구조는 [ARCHITECTURE.md](ARCHITECTURE.md) §1~§2, 위반 금지 사항은 [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)를 참조한다.

## 7. 현재 프로젝트 단계

**설계(Design) 단계.** 코드는 아직 생성하지 않는다. 이 단계의 목표는:

1. 도메인 규칙(보상플랜/정산/법무)을 문서로 명확히 정의
2. 시스템 아키텍처와 데이터 모델을 문서로 합의
3. 이후 구현 단계에서 Claude Code / Codex가 참조할 수 있는 "단일 진실 공급원(Source of Truth)" 문서 세트 확보

설계 단계 종료 조건은 [DECISIONS.md](DECISIONS.md)의 Open Decisions가 모두 해소되고, PRD/DATABASE/ARCHITECTURE가 합의된 시점으로 한다.

## 8. 문서 체계 안내

| 문서 | 역할 |
|---|---|
| [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) | (본 문서) 전체 맥락, 모든 문서의 전제 |
| [PRD.md](PRD.md) | 제품 요구사항 — 무엇을 만드는가 |
| [ARCHITECTURE.md](ARCHITECTURE.md) | 시스템 구조 — 어떻게 구성하는가 |
| [DATABASE.md](DATABASE.md) | 데이터 모델 — 무엇을 어떻게 저장하는가 |
| [COMPENSATION-RULES.md](COMPENSATION-RULES.md) | 후원수당(보상플랜) 규정 |
| ~~[PROMOTION-RULES.md](PROMOTION-RULES.md)~~ | ~~직급 승급/유지/강급 규정~~ — **폐기됨 (D-030)**: 직급 체계 자체가 FNS 마케팅 플랜에 존재하지 않음이 확인되어 폐기. 파일은 보존하되 내용은 폐기 안내로 대체됨 |
| [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) | 정산(지급/세금) 규정 |
| [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) | 법적 준수 체크리스트 |
| [TASK-SPEC.md](TASK-SPEC.md) | AI(Claude Code/Codex) 작업 지시서 표준 형식 |
| [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) | 절대 임의 변경 금지 대상과 이유 |
| [CHANGELOG.md](CHANGELOG.md) | 문서 변경 이력 |
| [DECISIONS.md](DECISIONS.md) | 의사결정 기록 및 미확정 사항(Open Decisions) |

## 9. 협업 방식

- **Docs-first**: 코드 작성 전 관련 문서가 먼저 갱신되어야 한다.
- **AI 협업 개발**: VS Code 환경에서 Claude Code와 Codex를 활용해 구현을 진행할 예정이며, 두 도구 모두 `docs/` 문서를 1차 컨텍스트로 참조한다.
- **변경 추적**: 문서 내용이 바뀌면 [CHANGELOG.md](CHANGELOG.md)에 기록하고, 의사결정이 바뀌면 [DECISIONS.md](DECISIONS.md)에 기록한다.

## 10. 미확정 사항 (요약)

상세 목록은 [DECISIONS.md](DECISIONS.md) 참조. 대표 항목:

- **제품 판매수익(25%, 구 패키지 추천보너스)·페어보너스의 법적 분류(후원수당 vs 제품 판매이익)** — 35% 한도 적용 여부 직결, 유일하게 남은 최우선 법률 검토 항목 (D-024, O-059) — *추천인 변경 관련 법률위험(O-024)은 D-020(관리자 전용 전환)으로 해소됨, 페어보너스 큐 재정렬 규칙(O-072)은 D-026으로 해소됨*
- **(신규) 제품 판매수익·페어보너스 수령을 위한 "본인 패키지 구매" 자격 조건(D-028)이 사실상 구매 강요로 작동하는지** — [DECISIONS.md](DECISIONS.md) O-081, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §5
- 추천 링크 `memberid` 노출 정책(O-073 — 2026-06-24 비교분석 결과 권고: 랜덤 추천코드), 클릭 봇/중복 필터링(O-074), 비회원 클릭데이터 개인정보 처리 근거(O-075) — [DECISIONS.md](DECISIONS.md) D-025 신규
- **(2026-06-24 마케팅 플랜 검증 라운드 신규)** 패키지 구매 선행조건 여부(O-076, 권고: 현재 상태 유지), 패키지 재구매 정책 및 자기매칭(Self-pair) 방지 안전장치(O-077, 권고: 자기매칭 방지는 즉시 반영) — [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5.2/§4.1
- 조직 병합/조직 분리/조직 복구 기능의 정의, "기타 회사 승인 사유" 포괄조항의 오남용 방지 충분성 ([PRD.md](PRD.md) §5.16)
- 실제 운영 시작 국가(TH/JP/US) 순서·시점, 다통화 지원 범위 (지원 대상 4개국 자체는 확정 — D-011/D-023)
- 고객몰(B2C) 포함 여부
- 센터(Center) 구조, Rule Designer, 전자서명 도입 여부 (각각 §3 참조, 최종 사업 확정 필요)
- CN(중국) 데이터 현지화 법규 대응 방안 — **현재는 CN이 Reserved Country(D-023)로 우선순위가 낮으며, CN 활성화를 검토하는 시점에 재상향** ([DATABASE.md](DATABASE.md) §7.5)
- 국가별 세금규칙/마케팅플랜/프로모션/정산규칙의 실제 내용 (KR 외 4개국)
- 회원 유형(개인/사업자/법인/외국인)별 본인확인 항목 및 검증 절차
- 관리자 권한 체계의 세부 역할 목록 및 권한 매핑
- 공제조합(KR) 외 4개국의 동등 규제기관 및 보고 요건
