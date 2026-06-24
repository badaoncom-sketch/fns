# CHANGELOG.md

> 문서 체계 변경 이력을 기록한다. [Keep a Changelog](https://keepachangelog.com/) 형식을 따른다.
> 의사결정 자체의 배경/근거는 [DECISIONS.md](DECISIONS.md)에 기록한다. 본 문서는 "무엇이 바뀌었는가"만 기록한다.

## [v0.20.0] - 2026-06-24 (FNS 마케팅 플랜 재정의 최종 점검 — 잔여 정합성 정리, 쇼핑몰/정기배송 구조 신설, 관리자 설정화 완료)

> 사용자가 "기존 문서에 일반 MLM 구조(PV/직급/주정산)가 섞여 들어간 것 같다"는 우려로 마케팅 플랜 전체를 FNS 고유 구조 기준으로 재점검 요청. 점검 결과 핵심 구조는 이미 D-024~D-030으로 반영되어 있었음을 확인했고, D-030 폐기 작업이 남긴 잔여 참조 불일치를 정리했으며, 빠져 있던 쇼핑몰 구조를 신설했다. [DECISIONS.md](DECISIONS.md) D-031/D-032, §5.11. 코드는 생성하지 않음.

### Fixed (D-031 — 잔여 정합성 정리, 문서 충돌 해소)

- [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2 — D-030으로 폐기된 `pv_ledger`/`member_rank_history`가 "수정 금지 append-only 원장" 보호 목록에 잘못 남아 있던 것을 제거(2곳)
- [ARCHITECTURE.md](ARCHITECTURE.md) §2.3 — 같은 두 테이블이 worker 계산결과 기록 예시에 남아 있던 것을 제거. worker Job 처리 대상 개수 "9종" → 실제 나열 항목 수에 맞춰 "10종"으로 정정(조직이동 적용 Job 포함, D-022)
- [ARCHITECTURE.md](ARCHITECTURE.md) §2.4/§7, [PRD.md](PRD.md) §5.4 — 폐기 안내로 대체되어 더 이상 "§6"을 갖지 않는 [PROMOTION-RULES.md](PROMOTION-RULES.md)를 "§6" 단위로 가리키던 깨진 참조를 D-021/D-022 직접 참조로 교체

### Added (D-031 — 쇼핑몰 구조 신설)

- [PRD.md](PRD.md) §5.1.3(신규) — 일반 쇼핑몰 / 회원몰 / **정기배송(핵심 기능, MVP)** / 자동결제 / 유지구매 관리 구조 확정. 기존 "제품/주문" 모듈 한 줄 정의를 대체
  - 정기배송 처리 흐름(scheduler 트리거 → worker 결제·주문생성·배송), 회원 셀프서비스(민감 변경 승인 대상 아님) 명시
  - §5.1.2 자격 현황 화면에 정기배송 등록 현황/당월 결제 이력/유지구매 부족분 안내 추가
- [DATABASE.md](DATABASE.md) §3.30(신규) — `payment_methods`/`recurring_orders`/`recurring_order_items`/`recurring_order_payment_attempts` 4개 테이블. 정기배송으로 생성된 주문은 일반 주문과 동일하게 §3.27.1 유니레벨 자격 매출에 합산
- 신규 Open Decision **O-086**(자동결제 실패 처리), **O-087**(PG사 선정·토큰화 방식, 배송주기 옵션)

### Added (D-032 — 관리자 설정화 완료)

- [DATABASE.md](DATABASE.md) §3.13 — `marketing_plan_versions.plan_definition` 스키마를 v0.18.0에서 권고 스켈레톤으로만 남겼던 상태에서 **확정 스키마로 전환**: `unilevel.line_rates`/`max_depth`, `package.price`/`sales_profit_rate`/`pair_bonus_amount`/`pair_window_days`, `eligibility.unilevel_maintenance_amount`, `lifestyle_bonus.*` 필드를 확정. 35% 한도 임계치는 `compliance_thresholds`(D-027)로 분리 유지(법적 강제값과 사업적 선택값의 변경 승인 절차 분리)
- [DECISIONS.md](DECISIONS.md) §5.10.6 6번 항목("관리자 설정화")을 완료로 갱신

### Added (§5.11 — 사용자 요청 7개 산출물 종합 보고서)

- [DECISIONS.md](DECISIONS.md) §5.11에 FNS 최종 마케팅 플랜(자격 흐름 다이어그램)/수당 종류 정의/자격 구조 정의/관리자 설정 가능 항목/삭제 가능한 MLM 일반 구조/유지해야 할 FNS 구조/문서 충돌 항목(해소 내역) 7개 산출물을 정리
  - 결론: 마케팅 플랜의 실체 규칙 자체에서는 추가 MLM 템플릿 잔재가 발견되지 않았다(D-030이 이미 직급/PV를 제거 완료) — 발견된 문제는 모두 "빠르게 반복된 정정 라운드 간의 문서 동기화 지연"이었다

### Notes

- 본 라운드는 새 비즈니스 규칙을 추가하지 않았다 — 모든 수치(라인비율/패키지금액/제품판매수익비율/페어보너스금액/페어기간/유지구매금액)는 D-018/D-024/D-028에서 이미 확정된 값을 그대로 스키마에 반영했을 뿐이다.
- 쇼핑몰 구조(정기배송 포함)는 사용자가 이번 라운드에서 처음 명시적으로 지시한 신규 범위이며, 기존 "제품/주문" 모듈의 누락을 보완한 것이다.

## [v0.19.0] - 2026-06-24 (직급/PV 폐기 실행 — D-030)

> 사용자가 v0.18.0의 정합성 보고서(삭제 후보: 직급 체계, PV 추상화 계층)를 검토한 뒤 **"둘 다 제거 진행"을 확정**해 실제로 실행했다. [DECISIONS.md](DECISIONS.md) D-030. 코드는 생성하지 않음.

### Removed

- **직급(Rank) 체계 전면 폐기**: 5단계 직급(회원/실버/골드/플래티넘/다이아), 승급조건 4요소, 유지/강급 규정 폐기
  - [PROMOTION-RULES.md](PROMOTION-RULES.md) 내용을 폐기 안내로 대체(파일은 보존, 사유·영향범위 명시)
  - [DATABASE.md](DATABASE.md) §3.2 `ranks`/`member_rank_history` 테이블, `members.current_rank_id` 컬럼 폐기(섹션 번호는 후속 교차참조 보존을 위해 유지)
  - [ARCHITECTURE.md](ARCHITECTURE.md) api "Promotion" 모듈, worker "프로모션" Job 제거(처리 대상 9종으로 조정)
  - [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4 "직급수당(Rank Bonus)" 행 폐기
  - [PRD.md](PRD.md) "직급 관리" MVP 모듈, 조직성장의 직급변화 집계, Document Center 직급별 공개범위, 활동이력의 "최근 프로모션(직급변경)" 등 제거
  - [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) §1 프로젝트 한 줄 정의·§2 배경에서 "직급 승급" 제거, §3 용어표 "직급(등급)" 행 제거, §8 문서체계안내에서 PROMOTION-RULES.md를 폐기 표시
- **PV(Point Value) 추상화 계층 전면 폐기**: `pv_ledger` 테이블, `products`/`order_items`의 PV 관련 컬럼 폐기
  - [DATABASE.md](DATABASE.md) §3.4 `pv_ledger` 폐기, '내 조직'(§3.11) 조직매출/조직성장 산출식을 `orders`/`order_items` 기반으로 전환
  - [ARCHITECTURE.md](ARCHITECTURE.md) Catalog/Order 모듈의 PV 관리 항목 제거
  - [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §2 PV 정의 섹션 폐기(번호는 유지)
  - [PRD.md](PRD.md)/[PROJECT-CONTEXT.md](PROJECT-CONTEXT.md)의 PV 표기 전부 제거 — 모든 매출/구매 기준은 KRW로 통일
- [DECISIONS.md](DECISIONS.md) D-015(직급체계 확정) 폐기 표시, Open Decisions에서 O-007/O-008/O-083/O-084 해소 처리, §5.10 보고서에 "분석 후 D-030으로 실행됨" 업데이트 추가

### Notes

- 데이터/문서를 물리적으로 삭제하지 않고 "폐기됨" 안내로 대체하는 방식을 택했다 — 본 저장소가 git 등으로 백업되지 않아, 과거 설계의 유일한 기록이 문서 자체이기 때문(append-only 원칙의 연장).
- 섹션 번호([DATABASE.md](DATABASE.md) §3.2/§3.4, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §2)는 의도적으로 비우지 않고 유지했다 — 수십 곳에서 번호로 교차 참조하는 문서 체계 특성상, 재번호화 시 깨질 위험이 너무 크다고 판단.
- 남은 후속 작업(§5.10.6 참조): `marketing_plan_versions.plan_definition` 스키마 구체화, "제품 판매수익" 명칭 최종 확정(O-085), O-081 법률 자문 의뢰.

## [v0.18.0] - 2026-06-24 (마케팅 플랜 정합성 전면 점검 — 정산 주기 정정(월정산 단일), 직급/PV 삭제 후보 식별, 명칭 비교, O-081 심화, 관리자 설정화 gap 발견)

> 새 기능 추가가 목적이 아니라, **실제 FNS 마케팅 플랜 기준으로 문서를 재정렬**하고 불필요한 일반 MLM 구조를 식별하는 라운드. 정산 주기만 사용자가 명확히 정정을 지시해 실행했고(D-029), 직급/PV 삭제는 **후보 식별까지만** 진행했다(실행 안 함). 코드는 생성하지 않음.

### Changed (중요 — 정산 주기 정정, D-029)

- **D-016("후원수당=주정산/직급수당=월정산" 혼합형) 정정 → D-029(월정산 단일)**: FNS는 주정산을 사용하지 않으며, 모든 채택 수당을 매월 1회 단일 월정산한다.
  - [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §2/§3/§9/§10 전면 정정 — 두 트랙 구조 폐기, `settlement_batches`는 국가별 월 1개만 생성
  - [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §5(산정 주기 월 단위로 수정)/§6(35% 한도 검증을 트랙 구분 없이 단일 누적 기준으로 단순화)
  - [DATABASE.md](DATABASE.md) §3.27.1 — 정산 주기와 자격 판정 주기가 모두 월 단위로 일치해 캐시 테이블 필요성이 낮아짐(우선순위 하향)
  - 이벤트 기반 즉시 산정(제품 판매수익/페어보너스)은 **산정**은 즉시, **지급**은 그 달 월정산 배치에 포함되는 구조를 유지

### Added (정합성 보고서 — [DECISIONS.md](DECISIONS.md) §5.10, 6개 섹션)

- **충돌 항목**: D-016 vs 실제 운영(해소), 직급 체계가 PROJECT-CONTEXT.md §1 비전 정의에까지 들어가 있으나 실제 마케팅 플랜(D-018 이후 4축 구조)에는 없음, PV가 COMPENSATION-RULES §2에 "확정"으로 표시되어 있으나 모든 실제 계산은 KRW를 직접 사용
- **삭제 후보(실행 안 함, 영향도 분석만)**:
  - **직급(Rank) 체계** — `PROMOTION-RULES.md` 전체, `ranks`/`member_rank_history`/`members.current_rank_id`. D-015 이후 구체 임계값이 한 번도 확정된 적 없고, 4축 보상엔진의 어떤 공식에도 입력값으로 쓰이지 않음. 영향 범위: PROJECT-CONTEXT/PRD/DATABASE/ARCHITECTURE/COMPENSATION-RULES 전반(상세는 §5.10.2 표)
  - **PV 추상화 계층** — `pv_ledger`, `products.PV환산값`. 모든 실제 산정 규칙이 PV 대신 KRW를 직접 사용 — 이미 O-062로 자체 플래그된 사안과 연속
  - 신규 Open Decision **O-083**(직급 폐기 여부), **O-084**(PV 제거 여부)로 추적 — **사용자의 명시적 실행 승인 전까지 보류**
- **유지 항목**: 유니레벨/제품판매수익/페어보너스/+알파 4축, 35% 모니터링 엔진, 추천 링크, 회원 생애주기, 조직 이동, 월정산, 다국가 구조 — 모두 직급/PV와 독립적
- **법률 검토 필요 — O-081 심화**(판단 아님, 리스크 신호만): 가입 강제는 아니나 "보너스 수령 조건으로서의 400만원 구매 유도"가 별도 금지행위 패턴에 해당할 가능성, 자격유지비용(400만원)이 일반적 Active Qualification 수준(5만원)의 80배라는 점, pass-up 없는 소멸 구조가 구매 압박을 키우는 점 등을 [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §5/[DECISIONS.md](DECISIONS.md) §5.10.4에 정리
- **관리자 설정화 gap**: `marketing_plan_versions.plan_definition`이 D-005 시점 placeholder 그대로 남아 있어, D-018~D-028에서 확정된 실제 수치(LINE비율/25%/400만원/200만원/30일/5만원)가 스키마에 전혀 반영되지 않음 — 구현 시 하드코딩 위험. [DATABASE.md](DATABASE.md) §3.13에 권고 스키마 스켈레톤 추가. 35%/30%/33% 임계치(`compliance_thresholds`, D-027)는 이미 올바르게 설계되어 있음을 확인
- **명칭 비교(O-085)**: "제품 판매수익"을 한국 MLM 업계 용어(추천수당/판매수당/소개수당/패키지추천수당)와 비교 — 권고: **현재 명칭 유지**("추천"을 다시 쓰면 D-028의 자격조건 분리 논리·O-059 법적분류 논의와 충돌). 업계 친숙도 절충 대안: "패키지 판매수당"
- 신규 Open Decision **O-082**(자격 미충족 미지급 로그 여부 — 직전 라운드에서 식별했으나 마스터 표에 등록이 누락되어 있던 것을 이번에 보완)

### Notes

- 코드는 생성하지 않음(사용자 명시적 요청). "단순 삭제가 아니라 전체 문서 영향도를 분석해달라"는 요청에 따라, 직급/PV는 실행하지 않고 후보·영향맵만 작성했다 — 실제 제거는 사용자 확인 후 별도 라운드 권고.
- DECISIONS.md §5(Design Freeze Report)는 기존 컨벤션(과거 스냅샷 보존, 새 라운드는 새 서브섹션)에 따라 §5.10을 신규 추가했다 — §5.1~§5.9는 그대로 유지.

## [v0.17.0] - 2026-06-24 (마케팅 플랜 자격 구조 정정 — 유니레벨/제품판매수익 자격 분리, "패키지 추천보너스"를 "제품 판매수익"으로 개칭)

> D-024의 "5만원 일반제품 구매 또는 400만원 패키지 구매를 하나의 후원수당 자격으로 묶은" 설계가 잘못되었다는 사용자 지적에 따른 정정. 신규 결정 번호 **D-028**로 기록(D-024 본문은 보존, 해당 부분만 취소선+주석 처리).

### Changed (중요 — 자격 구조 정정)

- **자격을 2개의 완전히 독립된 차원으로 분리** — [DECISIONS.md](DECISIONS.md) D-028 (D-024 §2/§5/§6 정정)
  1. **유니레벨 후원수당 자격**: 매월 5만원 상당 제품 구매로 그 달의 수령 자격을 판정한다("최초 충족" 단계 폐지, 매월 독립 판정). 400만원 패키지 구매는 더 이상 이 자격의 별도 충족 경로가 아니다 — 다만 구매액이 5만원을 넘으므로 그 달의 자격은 산술적으로 함께 충족된다.
  2. **제품 판매수익·페어보너스 자격**: 본인이 먼저 400만원 패키지를 구매한 적이 있어야 한다(1회로 영구 획득, 유지조건 없음). 본인이 미구매 상태면, 직추천 회원의 구매/페어 성립이 발생해도 **지급하지 않으며 누구에게도 이전되지 않고 소멸**한다(pass-up 없음).
  3. 예시: 무료가입 후 5만원 제품을 구매한 "나"는 유니레벨 자격은 있으나, 본인이 패키지를 구매하지 않았다면 직추천 A의 패키지 구매에 대한 제품 판매수익(100만원)과 이후 페어보너스(200만원)를 받을 수 없다.
- **명칭 정정**: "패키지 추천보너스(Package Referral Bonus)" → **"제품 판매수익(Package Sales Profit / 패키지 판매수익)"**. 수령자·트리거·금액(25%/100만원, Pair당 200만원)은 변경 없음 — 자격 조건과 명칭만 정정.
  - [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4/§4.1, [DATABASE.md](DATABASE.md) §3.24의 테이블명 `package_referral_bonuses` → `package_sales_profits` 변경
- [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5 재구성: §3.5.2(유니레벨 자격, 단순화) / §3.5.3(삭제·통합 표시) / §3.5.4(유니레벨 산정매출기준, 변경 없음) / §3.5.5(신규, 제품판매수익·페어보너스 자격)
- [DATABASE.md](DATABASE.md) §3.27을 §3.27.1(유니레벨 자격)/§3.27.2(제품판매수익·페어보너스 자격)로 분리, §3.24에 자격 검증 로직(미충족 시 행 생성 안 함) 추가

### Added (Open Decisions — 신규 발견)

- **O-081**: 제품 판매수익·페어보너스 수령을 위한 "본인 패키지 구매" 자격 조건이 사실상 구매를 강요하는 효과를 낳는지 — 가입 자체는 무료이나, 다운라인을 둔 회원이 보너스를 받으려면 사실상 본인도 패키지를 사야 하는 경제적 압박이 발생. [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §5에 신규 체크리스트 항목 추가
- **O-082**: 자격 미충족으로 지급되지 않은 건을 별도 로그로 남길지(회원 화면에서 "왜 못 받았는지" 설명하기 위함) — [DATABASE.md](DATABASE.md) §3.24

### Notes

- 코드는 생성하지 않음(사용자 명시적 요청). D-027(35% 모니터링 엔진, 같은 날 이전 작성)의 O-059 시나리오 분석에서 사용된 "패키지 추천보너스" 표현은 본 라운드 이후 "제품 판매수익"으로 읽는다 — 계산식 자체는 변경 없음.
- O-059(법적 분류)에 새로운 시사점 추가: "제품 판매수익"이라는 명칭과 본인 구매 자격조건이 "판매이익" 해석에도 무게를 더해, 분류 판단이 기존보다 더 미묘해졌다 — 여전히 법무 최종 확인 필요.
- PRD.md §5.1.2/§5.17.2에 회원이 "2개의 자격이 다르다"는 점과 "본인이 자격 미충족이면 직추천 구매 통계가 0보다 커도 수당이 지급되지 않는다"는 점을 명확히 안내하도록 반영.

## [v0.16.0] - 2026-06-24 (법률 리스크 최소화 — 35% 한도 모니터링/차단 엔진 설계, O-059 시나리오 분석)

### Added

- **법적 한도(35%) 모니터링 엔진 설계 확정** — [DECISIONS.md](DECISIONS.md) D-027
  - 3개 시간 단위 계산: 실시간(Redis 추정치)/월별(추세용)/연도누적(법적 판단 기준, [ARCHITECTURE.md](ARCHITECTURE.md) §8.1)
  - 임계치 3단계 확정(KR): **30% 주의 / 33% 경고 / 35% 차단**, 국가별 설정 가능(`compliance_thresholds`, [DATABASE.md](DATABASE.md) §3.29)
  - "차단"을 정산 배치의 검증→승인 전이를 보류하는 **Hard Gate**로 설계(영구 미지급 아님) — 초과분의 최종 처리 방식은 O-004로 계속 추적
  - 공제조합 보고센터와 **단일 소스 원칙**으로 연계 — 대시보드와 규제 보고서가 같은 `compliance_ratio_snapshots` 값을 인용 ([PRD.md](PRD.md) §5.7/§5.18)
- **관리자 대시보드 신설**([PRD.md](PRD.md) §5.18): 현재/예상/국가별 비율, 임계치 표시, 패키지 매출 비중·페어 성립률 리딩 인디케이터
- **DATABASE.md §3.29 신설**: `compliance_thresholds`(국가별 임계치), `compliance_ratio_snapshots`(실시간정합화/월별/연도누적 스냅샷)
- **ARCHITECTURE.md §8.1 신설**: 모니터링 엔진 구조, worker Job 확장, "차단"의 법적 충돌 검토(COMPENSATION-RULES §1과의 관계)
- **O-059 시나리오 분석(D-027)**: 패키지 추천보너스·페어보너스의 법적 분류에 따라 35% 계산식이 어떻게 달라지는지 수치로 분석 — **패키지 매출만 따로 보면 커미션 비율 = 25%×(1+p)(p=페어 성립률), p≥40%에서 이미 패키지 매출만으로 35% 도달**. 시나리오 A(둘 다 후원수당, 현재 임시 기본값) vs B(둘 다 판매이익) vs C(혼합) 비교
- Open Decisions 4건 추가: O-078(정합화 주기), O-079(차단-지급거부금지 충돌 법률확인), O-080(예상비율 산출 방법론) — O-004/O-059는 영향분석으로 보강(신규 번호 아님)
- LEGAL-CHECKLIST.md §3/§9에 "35% 차단이 정당한 사유 없는 지급거부 금지 조항과 충돌하지 않는다"는 해석과 그 법률 확인 필요성 명시

### Notes

- 코드는 생성하지 않음(사용자 명시적 요청). 수정 대상은 ARCHITECTURE/PRD/DATABASE/LEGAL-CHECKLIST/DECISIONS/CHANGELOG로 한정 — COMPENSATION-RULES.md(§6의 35% 하드 제약은 이미 확정되어 있으나 30/33% 단계는 미반영)와 SETTLEMENT-RULES.md(§9 ③단계가 본 엔진을 호출하도록 갱신 필요, §10의 "주/월 2단계 검증" Open Decision과 직접 연결됨)는 이번 라운드 범위 밖이라 **참조만 하고 수정하지 않았다** — 다음 라운드에서 두 문서를 갱신해 일관성을 맞출 것을 권고.
- 모니터링 엔진은 "감지·게이트"까지만 확정했고, 초과분의 정확한 처리 로직(O-004)은 여전히 미확정이다 — 이 부분이 확정되어야 차단 기능을 실제로 구현할 수 있다.

## [v0.15.0] - 2026-06-24 (마케팅 플랜 최종 검증 — 누락 2건 발견, 추천코드 비교/권고, Open Decision 우선순위 재정렬)

> 본 라운드는 **분석/검증 라운드**다 — 사용자의 최종 확정 지시가 아니라 "점검 후 필요 시 Open Decision 생성"을 요청받아 진행했다. 따라서 신규 D-번호(확정 결정)는 없으며, 모두 Open Decision(O-076/O-077) 추가 또는 기존 항목 보강으로 기록한다.

### Added (Open Decisions — 신규 발견)

- **O-076 (패키지 구매 선행조건)**: 무료회원이 5만원 일반제품 구매 없이 바로 400만원 패키지를 구매할 수 있는지 — 점검 결과 현재 문서(§3.5.2/§4.1.1)는 이미 "OR" 조건으로 일관되어 있어 모순은 없으나, 이 선행조건 없음이 사업적으로 의도된 결정인지 확인된 적이 없어 신규 추적. 권고: 현재 상태(선행조건 없음) 유지
- **O-077 (패키지 재구매 정책)**: 패키지가 1회성 상품인지 재구매 가능 상품인지 어떤 문서에도 명시되어 있지 않음을 발견 — `package_purchases`(DATABASE.md §3.24)에 `buyer_member_id` 유니크 제약이 없어 스키마상 재구매가 암묵적으로 허용됨. **추가로 더 중요한 누락을 발견**: 현재 페어보너스 매칭 규칙이 동일 구매자의 두 구매가 서로 페어를 이루는 것(자기매칭/Self-pair)을 배제하지 않음 — 재구매 허용 시 한 사람이 패키지를 두 번 사는 것만으로 인위적인 200만원 Pair Bonus를 발생시킬 수 있는 구조적 허점. 권고: 자기매칭 방지는 재구매 정책과 무관하게 즉시 반영, 재구매 허용 여부 자체는 사업 결정 필요

### Added (비교분석/권고안)

- **O-073 (추천 코드 형식)**: PK 직접노출 / 랜덤 추천코드 / 닉네임 기반 slug 3가지 방식의 장단점을 [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6.1에 비교표로 정리 — **권고: 랜덤 추천코드(B안)**. PK 노출(A안)은 보안 모범사례에 반하고, 닉네임 slug(C안)는 개인정보 노출·중복처리 부담이 있음. 최종 확정은 사업팀 확인 필요(자동 해소 아님)

### Changed (Open Decision 우선순위 재정렬)

- [DECISIONS.md](DECISIONS.md) §3을 "§3.1 마케팅 플랜/보상엔진 개발착수 전 필수 우선순위"(Tier 0~3)와 "§3.2 전체 백로그 우선순위"로 분리·재정렬
  - Tier 0: O-059(법적 분류) — 다른 모든 한도 관련 결정의 전제
  - Tier 1: O-004(한도 초과 처리), 한도산정 스위치 — O-059에 직접 종속
  - Tier 2: O-077(재구매·자기매칭), O-076(구매 선행조건) — 법무 불필요, 즉시 확정 가능, 자금 무결성 직결
  - Tier 3: O-073/O-074/O-075(추천 링크 세부) — 법적 영향 적은 그로스 기능
- O-059 항목에 **영향 범위 분석**을 추가: 35% 한도 base → O-004, 한도산정 스위치(DATABASE.md §3.24), 사전고지 문구(LEGAL-CHECKLIST.md §3), 공제조합 보고 정확성까지 연쇄 영향

### Notes

- 코드는 생성하지 않음(사용자 명시적 요청). 충돌분석 결과 §3.5.2/§4.1.1(패키지 구매 선행조건)은 **모순 없음**으로 확인됨 — 다만 의도 확인은 별도 필요(O-076).
- LEGAL-CHECKLIST.md §5에 "패키지 재구매 무제한 허용 시 사재기/순환매출 조장 우려" 검토 항목 신설, §3 O-059 항목에 영향범위 보강.

## [v0.14.0] - 2026-06-24 (마케팅 플랜 최종 점검 및 보강 — 추천인 링크 구조 신설, 페어보너스 실패 규칙 최종 확정)

### Added

- **추천인 링크 구조 신설** (원본 PDF 8페이지 "링크 공유 핵심" 반영) — [DECISIONS.md](DECISIONS.md) D-025
  - 모든 회원이 고유 추천 링크(`fns.com/r/{memberid}`, `fns.com/join?ref={memberid}`)를 가짐
  - 가입자가 추천 링크로 가입하면 링크 소유 회원이 추천인으로 자동 연결 — 가입 시점에 `members.sponsor_id` 자동 설정([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.6 신설)
  - 회원("마이오피스") 제공 항목: 추천 링크, QR 코드, 링크 클릭 수, 회원가입 수, 5만원 구매회원 수, 400만원 패키지 구매회원 수 ([PRD.md](PRD.md) §5.17 신설)
  - 관리자 제공 항목: 전체 추천 링크 통계, 회원별 전환율, 링크별 가입 통계
  - DATABASE.md §3.28 `referral_link_clicks` 신설 — 가입 전 비회원의 클릭도 추적, 회원가입/구매 통계는 기존 테이블의 파생값
  - **D-020과 충돌 아님을 명시**: 본 기능은 신규 가입 시점의 `sponsor_id` 최초 설정이며, 가입 후 추천인 변경은 여전히 D-020(관리자 전용 조직 이동)으로만 가능
- **기존 설계 공백 해소**: "신규 가입 시 추천인을 어떻게 지정하는가"에 대한 명시적 메커니즘이 이전까지 문서화되어 있지 않았음을 발견·보완

### Changed (중요 — O-072 해소)

- **페어보너스 실패 규칙 최종 확정** — [DECISIONS.md](DECISIONS.md) D-026
  - 추천인별 "대기 후보"는 최대 1건. 30일 Pair Window가 새 구매자 없이 만료되면 그 Pair Bonus는 소멸하고 해당 구매는 종료(closed) 처리
  - **재사용 금지·자동 이월 없음·자동 재매칭 없음**: 만료된 구매는 이후 어떤 Pair 계산에도 다시 쓰이지 않음 (예: A 실패 → 종료, 이후 B+C가 Pair, A는 미포함)
  - [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.3에 "Pair 실패 규칙" 신설, [DATABASE.md](DATABASE.md) §3.24에 `package_purchases.pair_status`(파생: PENDING/PAIRED/EXPIRED_NO_PAIR) 개념 추가

### Resolved

- O-072(페어보너스 큐 재정렬 규칙) — D-026으로 해소

### Added (Open Decisions)

- O-073(추천 링크 `memberid` 노출 정책), O-074(클릭 봇/중복 필터링), O-075(비회원 클릭데이터 개인정보 처리 근거) — LEGAL-CHECKLIST.md §7에 비회원 추적 데이터 체크리스트 항목 신설(기존에 누락되어 있었음)

### Notes

- 코드는 생성하지 않음(사용자 명시적 요청). 문서 간 참조 관계 및 모순·중복·충돌·누락 전수 점검 완료.
- 추천 링크를 통한 `sponsor_id` 자동 설정(가입 시점 최초 1회)과 D-020의 조직 이동(가입 후 변경, 관리자 전용)은 서로 다른 이벤트이며 충돌하지 않음을 양쪽 문서에 명시했다.

## [v0.13.0] - 2026-06-24 (마케팅 플랜 최종 정책 — 무료 회원가입 허용, 후원수당 자격/유지 신설, "제품 팩 판매 수익" → "패키지 추천보너스"+"페어보너스" 정정)

### Changed (중요 — 정책 반전 및 정정)

- **이전 지시 "무료 회원가입 없음" 취소**: 회원가입은 무료이며, 가입 자체에는 구매가 필수 조건이 아니다 — [DECISIONS.md](DECISIONS.md) D-024
- **"제품 팩 판매 수익(Pack Sales Bonus)"(D-019) 폐기 → "패키지 추천보너스"+"페어보너스"로 대체**: 수령자가 "직접판매한 회원 본인"에서 "구매자의 추천인"으로 바뀜 — 25%(100만원)는 패키지 구매자의 직추천인에게, 페어보너스(Pair당 200만원)는 동일 추천인의 직추천 구매자끼리 구매순서(1-2번째, 3-4번째 ...)로 페어가 성립할 때 그 추천인에게 지급
- 페어보너스 판정 기준을 "회원가입일"이 아니라 "패키지 구매일"로 명확화, 30일 Pair Window는 구매일부터 시작
- 400만원/25%(100만원)/200만원/30일 수치는 이번 결정으로 **확정값**임을 명시 — O-060 해소

### Added

- **후원수당 자격/유지 조건 신설** ([COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.5, [DECISIONS.md](DECISIONS.md) D-024)
  - 최초 자격: 5만원 상당 일반제품 구매 또는 400만원 패키지 구매 중 하나
  - 유지: 매월 5만원 이상 구매. 미달 시 회원 자격은 유지되나 그 달의 수령 대상에서만 제외, 다음 달 충족 시 자동 복구
  - `members.status`와 독립된 별도 월 단위 자격 차원 — 자격 없는 회원도 다른 회원의 LINE 깊이 판정에는 그대로 노드로 포함됨(D-021 탈퇴회원 처리 원칙과 동일 패턴)
- **유니레벨 후원수당 산정 매출 기준 명확화**: 매월 5만원 이상 유지구매 매출 기준으로 계산하며, 400만원 패키지 매출은 산정 대상에서 제외 (LINE 비율 3%/3%/3%/4%/5% 자체는 D-014/D-018 변경 없음)
- **마케팅 플랜 4축 구조 확정**: 유니레벨 후원수당 / 패키지 추천보너스 / 페어보너스 / "+알파" 보너스 — [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4
- DATABASE.md §3.24를 `package_purchases`/`package_referral_bonuses`/`package_pair_bonuses`로 재설계(`pack_sales`/`pack_sales_pair_bonuses` 대체), 신규 §3.27 "후원수당 자격/유지" 파생 데이터 정의 추가
- PRD.md §5.1.2를 "패키지 추천보너스/페어보너스/+알파 보너스 현황"으로 재정의(후원수당 자격 현황 항목 추가)
- LEGAL-CHECKLIST.md §3에 패키지 추천보너스·페어보너스 법적분류 체크리스트 항목 신설(기존에 누락되어 있었음), §5 "가입조건 사재기 강요 금지" 항목을 무료가입 확정과 부합하는 것으로 확인 처리
- Open Decisions: O-072(페어보너스 큐 재정렬 규칙) 신규 추가, O-059 설명을 새 구조 기준으로 갱신

### Resolved

- O-060(제품 세트 가격/마진율/페어기준기간 확정값 여부) — 사용자가 최종 정책으로 확정값임을 명시해 해소

### Notes

- 코드는 생성하지 않음(사용자 명시적 요청). 문서 간 참조 관계 전수 점검 완료.
- O-059(패키지 추천보너스/페어보너스의 법적 분류)는 여전히 Open Decision이나, 수령자가 추천인으로 바뀌면서 "후원수당" 쪽으로 분류될 가능성이 한층 높아졌다 — 법무 최종 확인 전까지는 35% 한도 산정에 포함하는 것을 보수적 기본값으로 둔다.
- DECISIONS.md §5(Design Freeze Report, 2026-06-23 시점 스냅샷)는 문서 자체의 컨벤션("과거 스냅샷은 보존, 새 라운드는 새 보고서로")에 따라 이번 라운드에서는 갱신하지 않았다 — D-024는 §1(확정된 결정) 및 §2(Open Decisions)/§3(우선순위 권고)에만 반영했다.

## [v0.12.0] - 2026-06-23 (중국(CN) 제외 — 4개국 구조 확정: KR/TH/JP/US)

### Changed

- **지원(활성) 국가를 5개국 → 4개국으로 축소**: KR / TH / JP / US — [DECISIONS.md](DECISIONS.md) D-023 (D-011 부분 수정)
- **CN(중국)은 삭제하지 않고 "Reserved Country" 상태로 전환** — 1차 및 중기 계획에서 제외, 현재 활성 국가 구조에서 제외. 향후 재활성화에는 별도 명시적 의사결정 필요
- `countries.is_active`(boolean) → **`countries.status`(ACTIVE/PLANNED/RESERVED enum)로 대체** — [DATABASE.md](DATABASE.md) §3.13. KR=ACTIVE, TH/JP/US=PLANNED, CN=RESERVED
- CN 데이터 현지화(PIPL) 법률 리스크(O-053)는 **삭제하지 않고 우선순위만 하향** — CN 활성화 검토 시점에 재상향 (DECISIONS.md §2/§3, [DATABASE.md](DATABASE.md) §7.5)
- LEGAL-CHECKLIST.md §12 **"해외 진출(Reserved Country) 관련 법적 고려사항 — 향후 검토"** 신설, PIPL/중국 직접판매 규제(直销许可证) 항목 보관

### Fixed

- PROJECT-CONTEXT.md/PRD.md/ARCHITECTURE.md/DATABASE.md/DECISIONS.md 전반의 "KR/TH/JP/US/CN 5개국" 표기를 "KR/TH/JP/US 4개국 + CN Reserved"로 전수 수정
- DECISIONS.md §5.2(확정항목 목록)에 누락되어 있던 D-020/D-021/D-022 행 추가, §5.3(미확정요약)/§5.5(개발불가능영역)/§3(우선순위)의 CN 관련 서술을 D-023 기준으로 갱신

### Notes

- 코드는 생성하지 않음. 문서 간 참조 관계 전수 점검 완료 — 헤딩 번호 순서, 교차 참조, 링크 무결성 모두 확인.
- D-011은 원문을 보존하고 "국가 5개국" 부분만 취소선+주석으로 D-023을 가리키도록 처리 (회원 유형 4종 부분은 변경 없음).

## [v0.11.0] - 2026-06-23 (조직 이동 — 승인≠적용 분리, 긴급 조직 이동 도입)

### Added

- **조직 이동 "승인"과 "적용"을 분리** — [DECISIONS.md](DECISIONS.md) D-022
  - 모든 조직 이동은 `approved_at`(승인일)과 `effective_date`/`applied_at`(적용일)을 별도로 가짐
  - 기본 정책: 승인 후 **익월 1일 00:00** 적용 (기준 시점은 승인일로 가정 — O-068 확인 필요)
  - 적용 전까지 기존 조직구조/수당/직급/정산 유지, 적용 후 신규 구조 적용. 과거 기록은 항상 불변(append-only)
- **긴급 조직 이동** 신설: 회원사망/법원판결/법적조치/운영오류수정/**중대한 컴플라이언스 이슈(신규 9번째 사유)**에 한해 승인 즉시 적용 가능. **SuperAdmin만 승인** 가능(Compliance Admin/지정 조직관리 관리자보다 좁음), Audit Log·변경사유 필수
- 처리 절차 7단계 → **9단계**로 확장: 요청→증빙첨부→영향분석→Snapshot→승인→**적용일예약**→**적용일도달**→실제적용→Audit Log
- `organization_transfer_logs`에 `effective_date`/`is_emergency` 컬럼 추가, `organization_transfer_reason_codes`에 `is_emergency_eligible` 컬럼 및 9번째 사유 추가
- ARCHITECTURE.md §7.1.1(적용일 예약/배치 처리)·§7.1.2(엔진과의 관계) 신설, scheduler/worker에 "조직이동 적용 배치" Job 추가
- COMPENSATION-RULES.md/PROMOTION-RULES.md/SETTLEMENT-RULES.md에 "계산 엔진은 `sponsor_id` 현재값만 읽으면 자동으로 적용완료 구조만 보게 된다"는 공통 원칙 추가
- Open Decisions 4건 추가 (O-068~O-071)

### Fixed

- PRD.md §5.16 하위 섹션 번호 정정 (긴급 조직 이동 삽입 과정에서 §5.16.7/§5.16.8 순서가 뒤바뀐 것을 발견해 수정), 관련 모든 교차 참조(ARCHITECTURE.md/DATABASE.md/DECISIONS.md) 동기화

### Notes

- 코드는 생성하지 않음. 문서 간 참조 관계 전수 점검 완료.

## [v0.10.0] - 2026-06-23 (회원 탈퇴 시 조직 구조 처리 원칙 확정 — 누락 보완)

### Added

- **회원 탈퇴 시 조직 구조는 자동 변경하지 않는다** 확정 — [DECISIONS.md](DECISIONS.md) D-021
  - `sponsor_id`/하위 조직 구조는 탈퇴 후에도 그대로 유지
  - 탈퇴 회원은 신규 후원수당/직급/정산/프로모션 대상에서 제외(과거 기록은 유지)
  - 탈퇴 회원도 다른 회원의 LINE 깊이 판정(트리 순회)에는 그대로 포함됨 — "수령 자격 제외"와 "트리 구조 보존"을 분리
  - 조직 재배치가 필요하면 관리자가 조직 이동(§3.26)을 별도로, 수동으로 수행 — 탈퇴 승인이 조직 이동을 자동 트리거하지 않음
- PRD.md §5.3(탈퇴 행)·§5.16.3(조직이동 사유 설명) 보강
- COMPENSATION-RULES.md §3.3/§5에 탈퇴회원 노드/수령자 구분 원칙 추가
- PROMOTION-RULES.md §6에 탈퇴회원 직급판정 제외 원칙 추가
- SETTLEMENT-RULES.md §9에 탈퇴회원 정산제외 원칙 추가
- DATABASE.md members.sponsor_id/§3.12/§3.5/§4(신규 원칙 10)에 상태변경↔조직구조변경 독립성 명시
- ARCHITECTURE.md §7/§2.3에 MemberLifecycle↔OrganizationTransfer 모듈 독립성, worker의 필터링/순회 분리 원칙 명시

### Notes

- 코드는 생성하지 않음. 이 결정은 직전 대화에서 사용자에게 "탈퇴 시 다운라인 처리가 문서화되어 있지 않다"고 보고한 것에 대한 직접적인 답변이며, D-020(조직 이동 관리자 전용화)의 자연스러운 후속 보완이다.

## [v0.9.0] - 2026-06-23 (추천인 변경 정책 전환 — 회원 자기서비스 폐지, 관리자 전용으로)

### Changed (중요 — 정책 반전)

- **D-017("추천인 변경 전면 허용") 폐기 → D-020으로 대체**: 회원은 추천인을 스스로 변경할 수 없다. 회원 화면에는 "추천인 변경"/"조직 이동"/"변경 신청" 기능을 제공하지 않으며, **조회만 가능**하다 — [DECISIONS.md](DECISIONS.md) D-020
- 추천인 변경은 **관리자 전용 "조직 이동"** 기능으로 전환. 8개 제한 사유(탈퇴/사망/명의변경/운영오류수정/회사정책상재배치/법원판결/법적조치/기타승인사유)에 한해서만 가능
- 권한은 **SuperAdmin / Compliance Admin / 지정 조직관리 관리자**로 한정 — 일반 관리자(CountryAdmin 등)는 수행 불가
- 7단계 절차 의무화: 사유입력 → 증빙첨부 → 영향분석 → Snapshot → 승인 → 적용 → Audit Log
- **O-024(추천인변경 법률위험) 해소**: 회원 신청 자체가 불가능해져 위험 전제가 소멸. 잔여 경량 확인 항목(O-064, "기타 승인사유" 오남용 방지)만 남음

### Added

- PRD.md §5.16 "조직 이동(Organization Transfer) — 관리자 전용" 신설, §5.6.4에 Compliance Admin/지정 조직관리 관리자 역할 추가
- ARCHITECTURE.md §7.1 "조직 이동 — 별도 흐름" 추가, OrganizationTransfer 모듈(role guard 포함)
- DATABASE.md §3.26 신설 — `organization_transfer_reason_codes`/`organization_transfer_logs`/`organization_transfer_attachments`/`organization_transfer_approvals` 4개 테이블. `member_change_requests.change_type`에서 SPONSOR_CHANGE 제거, `member_sponsor_history`의 참조를 `organization_transfer_logs`로 변경
- LEGAL-CHECKLIST.md §10 — "조직 이동은 회원 자유 변경 기능이 아니라 회사 운영목적상 관리자 권한으로만 수행되는 기능"으로 재정의, 법적 위험 재평가(크게 감소) 반영
- 관리자 메뉴 "조직관리"에 조직 병합/분리/복구 항목 추가 (메뉴만 확정, 기능 정의는 O-065로 보류)

### Notes

- 코드는 생성하지 않음(사용자 명시적 요청).
- 문서 간 참조 관계 전수 점검 완료 — PRD/ARCHITECTURE/DATABASE/LEGAL-CHECKLIST/DECISIONS 상호 참조 및 헤딩 번호 일관성 확인.

## [v0.8.0] - 2026-06-23 (회사 보상플랜 원본 문서 반영 — LINE 산정방식 정정)

### Fixed (중요)

- **후원수당(조직수당) LINE 산정 방식 정정**: D-014("LINE별 독립 계산 후 합산, 최대 18%")를 **D-018로 폐기·정정**. 실제 방식은 "직추천자별 라인이 도달한 최대 깊이(1~3/4/5)에 따라 단일 비율(3/4/5%)을 그 라인 전체 매출에 적용"하는 것 — [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3
- 사용자가 제공한 `COMPENSATIONAL MODEL_V2_260622_130715.pdf`(회사 실제 보상플랜 원본) 검토로 발견. 코드 작성 전 단계에서 발견되어 구현 오류로 이어지지 않음

### Added

- **제품 팩 판매 수익(Pack Sales Bonus)** 추가: 제품 세트 직접판매 25% + 30일 내 페어(2건) 달성 시 2배 보너스 — [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1, [DATABASE.md](DATABASE.md) §3.24, D-019
- **"+알파" 보너스(Lifestyle Bonus)** 추가: 여행(0.1~0.5%/24개월)/자동차(0.2%/1개월)/자기계발(0.2%/6개월) 누적 보너스 — §4.2, [DATABASE.md](DATABASE.md) §3.25
- `commission_records` 스키마를 라인 단위 산정(`line_root_member_id`/`qualified_depth`/`applied_rate`) 구조로 재정의
- PRD.md §5.1.2 "제품 팩 판매 수익/+알파 현황" 회원용 화면 영역 추가
- Open Decisions 5건 추가 (O-059~O-063) — **O-059(제품 팩 판매 수익의 후원수당/판매이익 법적 분류)는 O-024와 동급의 최우선 법률 검토 항목**

### Notes

- 코드는 생성하지 않음(사용자 명시적 요청). 이번 정정으로 Phase 1(KR) 설계 종료를 위한 법률 검토 항목이 1건(O-024)에서 2건(O-024, O-059)으로 늘었다 — [DECISIONS.md](DECISIONS.md) §5.8.
- 교훈: 사업팀 보유 원본 기획 문서는 설계 초기에 공유받을수록 재작업 비용을 줄인다.

## [v0.7.0] - 2026-06-23 (Phase 1 KR 설계 종료 — 최종 5개 결정 확정)

### Added

- **LINE1~LINE5 후원수당(조직수당) 비율 확정**: 3%/3%/3%/4%/5% (백로딩, 합계 18%) — [DECISIONS.md](DECISIONS.md) D-014, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3
- **직급 체계 5단계 확정**: 회원/실버/골드/플래티넘/다이아, 승급조건 4요소(개인PV/그룹PV/직추천수/하위직급보유) 구조 확정 — D-015, [PROMOTION-RULES.md](PROMOTION-RULES.md) §2/§3 (임계값은 미확정으로 잔여)
- **정산 주기 혼합형 확정**: 후원수당=주정산, 직급/리더십수당=월정산, 35% 한도는 두 트랙 합산 기준으로 검증 — D-016, [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §2/§9
- **추천인(스폰서) 변경 전면허용 확정**: 자격 제한 없음(승인 워크플로우는 유지), 어뷰징 모니터링·법률자문 필요사항 추가 — D-017, [LEGAL-CHECKLIST.md](LEGAL-CHECKLIST.md) §10
- DECISIONS.md §5.7 "최종 Design Freeze 합의" 추가 — 사용자의 95% 완성도 보고서와 자체평가(58%)를 방법론 차이로 reconcile, Phase 1(KR) 설계 종료 선언

### Resolved

- O-002(일부)/O-006/O-007(일부)/O-009/O-024(허용여부) Open Decisions 표에서 해소 → 확정된 결정으로 이동

### Notes

- 코드는 생성하지 않음. 남은 선결 과제는 2가지뿐: (1) 직급 승급조건 PV 임계값(제품가격표 필요), (2) 추천인 변경 전면허용 정책의 법률 자문(O-024) — 나머지는 구현 착수 가능.
- 사용자가 4개 결정 중 4개 모두에서 제 추천안과 다른 선택을 함 (백로딩/5단계/혼합정산/전면허용) — 추천은 참고용이며 최종 결정권은 사업팀에 있음을 보여주는 기록.

## [v0.6.0] - 2026-06-23 (FNS 최종 설계 검토 — Phase Design Freeze)

### Added

- **센터(Center) 구조** 검토 — `centers`/`member_center_history` 설계, CenterAdmin 역할 추가, 도입 여부는 미확정 ([PRD.md](PRD.md) §5.9, [DATABASE.md](DATABASE.md) §3.17)
- **Document Center** — 약관/보상플랜안내/교육자료/공지사항 + 회원개인문서(정산자료/지급명세서/사업자자료/원천징수자료), 버전관리/공개범위/국가별 문서 ([PRD.md](PRD.md) §5.10, [DATABASE.md](DATABASE.md) §3.18)
- **Customer Service Center** — 1:1문의/정산문의/수당문의/명의변경문의/이의신청/불만접수 티켓 체계, `tickets`/`ticket_messages`/`ticket_attachments` ([PRD.md](PRD.md) §5.11, [DATABASE.md](DATABASE.md) §3.19)
- **Notification Center** — Email/SMS/KakaoTalk/Push, 템플릿/이력/실패이력, `notifications`/`notification_templates`/`notification_logs` ([PRD.md](PRD.md) §5.12, [DATABASE.md](DATABASE.md) §3.20)
- **전자서명(E-Signature)** 검토 결과 도입 권고 — `e_signatures`/`consent_history`, 민감 변경 승인의 선행조건으로 연결 ([PRD.md](PRD.md) §5.13, [DATABASE.md](DATABASE.md) §3.21)
- **회원 활동 이력** — `member_activity_logs` (로그인/주문/수당/정산/프로모션), audit_logs와 목적 구분 ([PRD.md](PRD.md) §5.14, [DATABASE.md](DATABASE.md) §3.22)
- **Rule Designer** 검토 결과 도입 권고 — `rule_designer_drafts`/`rule_versions`/`rule_publish_history`, 시뮬레이션+법적한도 검증 후 발행 ([PRD.md](PRD.md) §5.15, [DATABASE.md](DATABASE.md) §3.23)
- **Database Review** — 누락 테이블/컬럼, 성능/확장성/법률/운영 위험 점검, CN 데이터 현지화 법규 위험 신규 식별 ([DATABASE.md](DATABASE.md) §7)
- **Architecture Review** — 유니콘 스케일 확장 시 병목 가능성·서비스 분리 필요성 점검 ([ARCHITECTURE.md](ARCHITECTURE.md) §9)
- **Design Freeze Report** — 설계 완성도(~58%), 확정/미확정 항목, 즉시개발가능/불가능 영역 정리 ([DECISIONS.md](DECISIONS.md) §5)
- D-012, D-013 결정 기록, Open Decisions 12건 추가 (O-047~O-058)

### Notes

- 코드는 생성하지 않음. Center/전자서명/Rule Designer는 메커니즘만 설계되었으며 최종 도입 여부는 사업팀 확정 필요.
- CN(중국) 데이터 현지화 법규 위험은 이번 라운드에서 식별된 가장 중대한 신규 리스크로, 다음 우선순위 1번으로 격상됨.

## [v0.5.0] - 2026-06-23

### Added

- 회원 유형(개인/사업자/법인/외국인) 4종, 지원 국가(KR/TH/JP/US/CN) 5개국 구조 확정 — [DECISIONS.md](DECISIONS.md) D-011
- PROJECT-CONTEXT.md §4 타겟시장을 다국가 구조로 재정의, §3 도메인 용어에 회원유형/국가/마케팅플랜버전/공제조합 추가
- PRD.md §5.6(회원유형·국가구조·회원상태체계·관리자권한체계), §5.7(공제조합 보고센터), §5.8(국가별 규칙 확장 placeholder) 신규 섹션 추가
- ARCHITECTURE.md §8 "다국가·컴플라이언스 아키텍처" 추가 (국가 스코프 계산, Compliance 모듈, country_scope RBAC), Compliance 모듈을 api 모듈 표에 추가
- DATABASE.md §3.12(회원 상태 체계) ~ §3.16(공제조합 보고센터) 추가: `countries`, `member_identity_profiles`, `marketing_plan_versions`, `country_tax_rules`/`country_promotions`/`country_settlement_configs`, `admin_roles`/`admin_role_assignments`, `compliance_report_definitions`/`compliance_report_submissions`. `members`에 `member_type`/`country_code` 추가, `commission_records`에 `plan_version_id` 추가
- Open Decisions 8건 추가 (O-039~O-046)

### Notes

- 코드는 생성하지 않음. 국가별 세금규칙/마케팅플랜/프로모션/정산규칙의 실제 내용은 이번 라운드 범위 밖이며 COMPENSATION-RULES.md/SETTLEMENT-RULES.md/LEGAL-CHECKLIST.md 후속 라운드에서 정의한다.

## [v0.4.0] - 2026-06-23

### Added

- 서버 구조를 Railway 프로젝트 1개 + **web/api/worker/scheduler/redis** 5개 서비스로 확정 — [DECISIONS.md](DECISIONS.md) D-010
- ARCHITECTURE.md §1~§2 전면 재구성: 5개 서비스별 역할/책임, "계산과 요청처리의 분리 원칙"(§1.1), 처리 흐름 예시 추가. §3/§5/§7/§8도 worker 중심으로 갱신
- PROJECT-CONTEXT.md §6.1에 서버 구조 핵심 원칙(web 계산금지/api Job생성만/worker가 처리/scheduler는 트리거만/Redis는 SoT 아님/Supabase가 최종 SoT) 추가
- TASK-SPEC.md §6 "서버 구조 배치 원칙" 추가, 작업 지시서 템플릿에 "서비스 배치" 항목 추가
- DO-NOT-TOUCH.md §1.3 "서버 구조 관련" 금지 항목 추가 (web/api/scheduler 계산 금지, Redis를 SoT로 취급 금지, 5개 서비스 구조 임의 변경 금지 등)
- Open Decisions 3건 추가 (O-036~O-038: cron 구현 방식, Supabase 백업 정책, worker Job 동시성 정책)

### Notes

- 코드는 생성하지 않음. 본 개정은 서버 인프라 구조에 대한 설계 결정이며, 구현 단계 진입 전까지는 문서 상의 원칙으로만 존재한다.

## [v0.3.0] - 2026-06-23

### Added

- 보상플랜 모델을 **Unilevel Sponsor Plan**(추천조직 기반)으로 확정 — [DECISIONS.md](DECISIONS.md) D-008
- LINE1~LINE5(추천 깊이 5단계) 개념 확정, 회원용 메뉴명 **"내 조직"** 확정, "조직도"는 관리자 전용 용어, "추천조직도" 미사용 확정 — D-009
- COMPENSATION-RULES.md §3에 보상플랜 모델/조직 구조/LINE 정의/용어 규칙 반영, §4 수당 종류에 후원수당(조직수당) 확정 표시
- PRD.md §5.1.1 '내 조직' 메뉴 명세(LINE1~5/조직매출/조직수당/조직성장) 추가
- ARCHITECTURE.md Partner Portal/Admin Console/Compensation 모듈 설명에 확정 용어 반영
- DATABASE.md §3.11 '내 조직' 조회용 파생 데이터 정의 추가, `commission_records`에 `line_depth`/`source_member_id` 개념 추가

### Changed

- "조직 이동(레그 변경)"을 "추천인(스폰서) 변경"과 동일 기능으로 통합 (Unilevel 확정에 따른 직접 귀결)
- DATABASE.md `member_position_history` 테이블 제거 → `member_sponsor_history`로 통합

### Resolved

- O-001(보상플랜 모델), O-031(포지션/레그 개념) — D-008로 해소되어 Open Decisions 목록에서 제거

## [v0.2.0] - 2026-06-23

### Added

- 회원 변경/생애주기 관리(Member Lifecycle & Change Management) 설계 추가: 회원정보변경, 조직이동(레그변경), 추천인변경, 명의변경, 탈퇴, 휴면, 강제탈퇴, 재가입, 계좌변경 9개 기능과 공통 처리 원칙(승인 프로세스/Snapshot/Audit Log/수당 영향 분석/정산 영향 분석) — [PRD.md](PRD.md) §5.3~5.4
- ARCHITECTURE.md에 `MemberLifecycle` 모듈, 민감 변경(Sensitive Change) 처리 아키텍처(§7), 3PL 외부 연동(§2.6) 추가
- DATABASE.md에 `member_change_requests`, `member_sponsor_history`, `member_position_history`, `member_bank_accounts`, `change_impact_analyses` 및 재고/물류 테이블(`warehouses`, `inventory_items`, `inventory_ledger`, `shipments`, `returns`, `inventory_recoveries`) 추가 (§3.9, §3.10)
- 물류/재고/3PL 관리 범위 초안 추가 — [PRD.md](PRD.md) §5.5
- LEGAL-CHECKLIST.md에 회원 변경/생애주기 관련 법적 고려사항(§11), 물류/재고(반품·환수) 관련 법적 고려사항(§12) 추가
- Open Decisions 10건 추가 (O-026~O-035), D-006/D-007 결정 기록 추가

### Notes

- 본 개정은 사용자 요청에 따른 설계 점검(누락 기능 15개 항목) 결과 반영. 코드는 생성하지 않음.

## [v0.1.0] - 2026-06-23

### Added

- `docs/` 디렉터리 및 초기 문서 세트 11종 생성:
  - PROJECT-CONTEXT.md
  - PRD.md
  - ARCHITECTURE.md
  - DATABASE.md
  - COMPENSATION-RULES.md
  - PROMOTION-RULES.md
  - SETTLEMENT-RULES.md
  - LEGAL-CHECKLIST.md
  - TASK-SPEC.md
  - DO-NOT-TOUCH.md
  - DECISIONS.md
- 기술 스택 확정 기록: Next.js, NestJS, Supabase, Railway, Redis / 개발도구: VS Code, Claude Code, Codex
- 프로젝트 단계를 "설계(Design)"로 명시, 코드 생성 금지 원칙 선언

### Notes

- 본 버전은 초안(Draft)이며, 보상플랜/직급/정산의 핵심 수치는 다수 미확정 상태 ([DECISIONS.md](DECISIONS.md)의 Open Decisions 참조).
