# TEST-PLAN.md — 테스트 전략

> 상태: v0.3 (D-070 — 쇼핑몰 운영 Phase 2 및 문서 동기화: §2.10 쇼핑몰 운영 Phase 2/SEO/Digital Marketing 테스트 추가, §3/§4 갱신. D-064 — 개발착수 전 최종 안정화: §2.8 ERP Core 엔진 테스트, §2.9 Multi-Tenant 격리 테스트 추가) · 최종 수정일: 2026-06-26 · 단계: 설계(Design)
> 전제 문서: [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md), [ARCHITECTURE.md](ARCHITECTURE.md), [COMPENSATION-RULES.md](COMPENSATION-RULES.md), [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md), [DATABASE.md](DATABASE.md)

## 0. 문서의 목적과 범위

본 문서는 FNS(Multi-Tenant MLM ERP) 구현 단계에서 **무엇을 어떤 레벨로 테스트할지**를 정의한다. [COMPENSATION-RULES.md](COMPENSATION-RULES.md)/[SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)에 정의된 계산 로직 자체를 변경하거나 재정의하지 않으며, 오직 "그 로직이 설계대로 동작하는지 어떻게 검증할지"만 다룬다. 아직 코드가 없는 설계 단계이므로, 본 문서의 테스트 케이스는 **구현 착수 시 그대로 테스트 스위트로 옮길 수 있는 명세** 역할을 한다.

후속 변경 시 [DECISIONS.md](DECISIONS.md)에 결정 기록을 남기고 본 문서를 갱신한다 — [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.1의 문서 변경 원칙과 동일하게 취급한다.

## 1. 테스트 레벨과 비율 권장안

| 레벨 | 책임 범위 | 권장 비율 | 근거 |
|---|---|---|---|
| **Unit** | 순수 계산 함수 단위 — 라인 깊이 판정, 비율 적용, 페어링 큐 로직, 35% 비율 계산, 세금 원천징수액 계산 등 입력→출력이 결정적인 로직 | **높음 (전체 테스트의 다수 비중)** | [COMPENSATION-RULES.md](COMPENSATION-RULES.md)/[SETTLEMENT-RULES.md](SETTLEMENT-RULES.md)는 "회사의 실제 자금이 지급되는 핵심 비즈니스 로직"([COMPENSATION-RULES.md](COMPENSATION-RULES.md) 서두 경고)이며, [ARCHITECTURE.md](ARCHITECTURE.md) §2.2는 Compensation/Settlement가 "입력 스냅샷을 받아 순수 계산을 수행하도록 설계"되어 있다고 명시한다 — 순수 함수는 회귀위험이 큰 만큼 Unit 레벨에서 가장 값싸게, 가장 빠르게, 가장 많은 경우의 수로 검증 가능하다 |
| **Integration** | worker가 Supabase PostgreSQL에 실제로 쓰는 흐름 — Job 생성→큐 적재→worker 소비→append-only 원장 기록까지의 연계, api가 Job 상태를 올바르게 조회하는지, 국가별 마케팅 플랜 버전 조회가 올바른 시점의 버전을 가져오는지 | **중간** | [ARCHITECTURE.md](ARCHITECTURE.md) §1.1의 "계산과 요청처리의 분리 원칙"은 서비스 간 경계(api↔worker↔redis↔Postgres)에서만 깨질 수 있으므로, 이 경계를 넘는 흐름은 Unit으로 잡을 수 없고 Integration이 필요하다 |
| **E2E** | 사용자 관점 흐름 전체 — 주문 생성부터 정산 지급까지, 회원가입부터 조직 이동 적용까지 | **낮음 (핵심 critical path만 소수)** | E2E는 느리고 깨지기 쉬워 회귀 탐지 비용이 높다. [ARCHITECTURE.md](ARCHITECTURE.md)가 정의하는 5개 서비스 흐름이 "한 줄로 끝까지 이어지는지"를 확인하는 용도로만 최소한으로 둔다 |

**원칙**: 계산 로직(MLM/정산/35% 게이트)일수록 Unit 비중을 극단적으로 높이고, 서비스 경계를 넘는 흐름일수록 Integration/E2E로 내려간다. 이는 [ARCHITECTURE.md](ARCHITECTURE.md) §1.1이 강제하는 "계산과 요청처리의 물리적 분리" 구조와 정확히 대응된다 — 계산은 worker 내부의 순수 함수이므로 Unit으로 격리해서 검증할 수 있고, 검증해야 한다.

## 2. 영역별 테스트 전략

### 2.1 MLM/수당 계산 테스트

| 항목 | 내용 |
|---|---|
| **무엇을** | LINE1~5 라인별 도달깊이 판정(D-018), 라인 단일비율(3%/4%/5%) 적용, 다중 라인 합산, 자격 판정(§3.5.2 매월 5만원 / §3.5.5 패키지 구매 자격), 패키지 정책별 유니레벨 포함 여부(`counts_toward_unilevel_line_revenue`) 반영 |
| **왜** | [DATABASE.md](DATABASE.md) §4 원칙 2 "모든 계산 결과는 입력 스냅샷을 함께 저장하여 사후 검증 가능하게 한다"는 설계가 이미 "동일 입력 → 동일 출력" 회귀 테스트를 1급 시민으로 지원하도록 만들어져 있다 — `commission_records`에 저장된 입력 스냅샷을 그대로 재생(replay)하면 같은 출력이 나와야 한다. 또한 [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3은 D-018에서 "LINE별로 독립 계산해 합산(최대 18%)" 방식이 회사 원본 문서와 불일치해 폐기되었다고 명시한다 — 이런 회귀가 한 번 실제로 발생했던 만큼, 동일 클래스의 오류가 재발하지 않도록 worked-example 기반 테스트가 필수다 |
| **어떻게(레벨)** | **Unit 다수**: §3.3의 검증 예시(라인 1,555명, 평균 구매 50,000원, 월매출 77,750,000원 × 5% = 3,887,500원)를 그대로 테스트 케이스로 사용 — "라인별로 3%+3%+3%+4%+5%를 합산하면 3,736,500원이 되어 원본과 달라진다"는 D-018의 반례도 함께 negative 테스트로 등록해, 향후 누군가 실수로 D-014 방식(라인별 독립 합산)으로 되돌리면 즉시 실패하도록 한다. **Unit**: 스냅샷 재생 테스트 — 동일한 `commission_records.input_snapshot`을 계산 함수에 다시 넣었을 때 저장된 결과와 100% 일치하는지 확인(회귀 감지용, CI에 상시 포함). **Integration**: 탈퇴(WITHDRAWN) 회원이 "다른 회원의 라인 깊이 판정에는 포함되지만 본인이 수령자가 되는 계산에서는 제외"되는지(D-021, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §3.3) — 수령 대상 필터링과 트리 순회가 서로 다른 단계임을 worker 레벨에서 검증 |

#### Worked-example 테스트 케이스 표 (D-018 핵심 로직)

| 케이스 | 입력 | 기대 출력 | 검증 포인트 |
|---|---|---|---|
| 정상 케이스(원본 문서) | 라인 1개, LINE1~5 가득 찬 6진 트리(1,555명), 평균 구매 50,000원 | 월매출 77,750,000원 × 5% = 3,887,500원 | 라인 전체에 "도달 최대 깊이(LINE5)" 기준 단일비율(5%) 적용 |
| D-014 회귀 방지(negative) | 위와 동일 입력 | **3,736,500원이 나오면 실패** (3%+3%+3%+4%+5% 라인별 합산 방식의 결과값) | 라인별 독립 계산·합산 방식으로 회귀하지 않았는지 |
| LINE1~3만 도달 | 라인이 LINE3까지만 형성 | 적용 비율 3% | "3단계 이하 = 3%" 구간 경계값 |
| LINE6 이상 존재 | 라인이 LINE6까지 형성 | LINE6 매출은 산정에서 제외, 비율은 LINE5 도달 기준 5%로 동일 | "LINE6 이상은 어떤 라인에서도 산정에 포함하지 않는다"(확정) 검증 |
| 다중 라인 | 직추천자 3명 → 3개 독립 라인, 각각 다른 깊이 도달 | 각 라인을 독립적으로 계산 후 합산 | "라인당 최대 5%"이지 "전체 조직 매출의 5%"가 아님을 검증 |
| 자격 미충족 회원 포함 | 라인 중간에 당월 5만원 미달 회원 존재 | 그 회원의 구매 매출은 0원으로 라인 매출에 반영, 라인 깊이 판정 자체에는 영향 없음 | §3.5.2/§3.5.4 자격과 트리 구조 판정의 분리 |

### 2.2 정산(Settlement) 테스트

| 항목 | 내용 |
|---|---|
| **무엇을** | `settlement_items`/`commission_records` 등 append-only 원장에 대한 UPDATE/DELETE 직접 수정 금지 강제, 보정 엔트리(역분개) 방식 정정의 정확성, 정산 프로세스 단계 전이(§9 ①~⑦) |
| **왜** | [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2가 "append-only 원장 테이블(`commission_records`, `settlement_items`, `audit_logs`)... 기존 행을 UPDATE/DELETE로 직접 수정하지 않는다. 정정은 반드시 보정(역분개) 엔트리 추가로 처리한다"고 명시하며, [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §8이 "`settlement_items`는 append-only다... 정정은 반드시 보정 엔트리(역분개)로 처리한다... 모든 정정은 사유와 승인자를 기록한다"고 동일 원칙을 정산 도메인에서 재확인한다. 이는 법적·금전적 영향이 큰 영역([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.2 "정산(커미션) 계산 로직")이므로 원장 불변성이 코드 리뷰가 아니라 **테스트로 강제**되어야 한다 |
| **어떻게(레벨)** | **Integration**: 실제 DB(또는 동등한 테스트 DB)에 대해 `settlement_items`/`commission_records` 행에 UPDATE/DELETE를 시도하는 테스트를 작성하고, 애플리케이션 레벨(repository/service 계층)에 UPDATE/DELETE 메서드 자체가 노출되지 않거나 호출 시 거부되는지 확인 — DB 권한(REVOKE UPDATE/DELETE) 또는 트리거로 강제하는 방식을 택할 경우 그 DB 제약 자체를 Integration 테스트로 검증. **Unit**: 보정 엔트리 생성 로직 — 정정 대상 금액의 음수 보정 엔트리가 원본과 동일한 스키마로 생성되는지, 사유/승인자 필드가 누락 없이 채워지는지. **Integration**: 정정 후 "원본 + 보정 엔트리"를 합산한 값이 올바른 정정 결과와 일치하는지(원본을 지우지 않고 합산으로 정합성을 회복하는 구조이므로) |

### 2.3 35% 법적 한도 게이트 테스트

| 항목 | 내용 |
|---|---|
| **무엇을** | 정상(30% 미만)/주의(30%↑)/경고(33%↑)/차단(35%↑) 3단계 임계치 동작, 차단 시 "③ 법적 한도 검증 → ④ 세금 계산" 전이 보류(Hard Gate), 연도 누적(Postgres) 값만이 법적 판단 근거이고 실시간 캐시(Redis)는 추정치로만 쓰이는지, 모니터링 대시보드 수치와 실제 정산 배치 처리 결과의 정합성 |
| **왜** | [ARCHITECTURE.md](ARCHITECTURE.md) §8.1은 "법적 차단은 항상 연도 누적(Postgres) 값만을 근거로 판단하며, 실시간 캐시는 대시보드 표시용 추정치로만 쓴다"고 명시하고, §8.1.3은 "차단"이 "정산 배치를 영구 폐기하는 것이 아니라... ③→④ 전이를 보류하는 Hard Gate"라고 정의한다. 이 구분이 깨지면(예: 캐시값으로 차단을 판단) 법적 한도를 실제로는 초과한 채 지급이 진행될 위험이 있다 — [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §1의 "사업연도 매출액의 35%를 초과할 수 없다"는 법령상 hard constraint이므로 이 게이트의 오작동은 가장 심각한 등급의 결함이다 |
| **어떻게(레벨)** | **Unit**: 30%/33%/35% 각 경계값(boundary) 케이스 — 29.99%/30.00%/32.99%/33.00%/34.99%/35.00% 등 경계 바로 아래/위 비율 입력에 대해 정확한 단계(정상/주의/경고/차단)가 반환되는지. **Unit**: 분자/분모 정의가 [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §6의 보수적 기본값(모든 패키지의 제품 판매수익·페어보너스를 한도 산정에 포함)과 일치하는지 — O-059 분류 결과가 바뀌어 분자 정의가 패키지별로 달라지는 시나리오도 케이스로 준비해 둔다. **Integration**: 캐시(Redis 실시간 카운터)와 Postgres(`compliance_ratio_snapshots`) 값에 의도적으로 drift를 주입한 뒤, 차단 게이트가 캐시값이 아니라 Postgres 연도 누적값으로 판단하는지 확인 — 이것이 "모니터링↔실제 정산 정합성"의 핵심 검증 지점. **Integration**: 35% 이상 시나리오에서 `settlement_batches`가 실제로 "검증" 단계에 머무르고 ④(세금 계산)/⑤(운영자 승인)로 자동 전이하지 않는지, 지정 관리자(SuperAdmin/Compliance Admin) 알림이 발생하는지(33% 경고 단계 포함). **Integration**: 개별 회원 단위 보류([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §6)와 배치 단위 한도 차단(§8.1.3)이 동시에 걸리는 케이스 — 개별 보류된 항목이 한도 계산의 분자에서 제외되는지(ARCHITECTURE.md §8.1.3이 "구현 시 주의"로 명시한 지점) |

#### 임계치 경계값 테스트 매트릭스

| 비율 | 기대 단계 | 기대 동작 |
|---|---|---|
| 29.99% | 정상 | 표시만 |
| 30.00% | 주의 | 대시보드 강조 |
| 32.99% | 주의 | 대시보드 강조 |
| 33.00% | 경고 | 강조 + 관리자 알림 |
| 34.99% | 경고 | 강조 + 관리자 알림 |
| 35.00% | 차단 | ③→④ 전이 보류, 검토 요청 |
| 40.00%(초과) | 차단 | 동일(차단은 35% 이상 전부 동일 처리) |

### 2.4 쇼핑몰/주문 테스트

| 항목 | 내용 |
|---|---|
| **무엇을** | 주문 생성→재고 차감→배송 트리거 흐름, 패키지 구매 플로우(구매 확정 시점에 이벤트 기반으로 제품 판매수익/페어보너스 즉시 산정), 환불/취소 시 재고 환수 및 클로백 연계 |
| **왜** | [ARCHITECTURE.md](ARCHITECTURE.md) §2.2는 Order 모듈이 "주문 생성/취소 요청 접수, 즉시 반영 가능한 매출 기록"을 담당한다고 정의하고, [COMPENSATION-RULES.md](COMPENSATION-RULES.md) §4.1.2는 패키지 구매수익이 "배치 주기를 기다리지 않고 패키지 구매 확정 시점에 이벤트 기반으로 즉시 산정"된다고 명시한다 — 일반 후원수당(월배치)과 다른 타이밍 모델이므로, 주문 흐름과 수당 산정 트리거가 정확히 맞물리는지가 핵심 위험 지점이다 |
| **어떻게(레벨)** | **Unit**: 재고 차감 계산, 패키지 정책(추천수당 비율/금액, 페어 활성화 여부)에 따른 산정 분기. **Integration**: 주문 생성 API(`api`) 호출 → Job 생성 → worker 처리 → `orders`/`inventory` 갱신까지의 흐름, 패키지 구매 확정 이벤트가 실제로 `package_sales_profits`/`package_pair_bonuses` 생성을 즉시 트리거하는지(월배치를 기다리지 않는지). **E2E**: 회원가입→패키지 구매→추천인에게 제품 판매수익 지급까지의 critical path 1~2개 |

### 2.5 Workflow Engine 테스트

| 항목 | 내용 |
|---|---|
| **무엇을** | 신규 워크플로우(환불/반품/교환/전자결재 등)의 승인 단계 전이(순차 다단계, 자동승인 조건, 알림 연동), **그리고 기존 5개 전용 승인구조가 Workflow Engine으로 흡수되지 않았음을 검증하는 회귀 테스트** |
| **왜** | [PRD.md](PRD.md) §5.30.3은 조직 이동 승인(D-020/D-022)·회원 생애주기 변경 승인(D-006)·프로그램 신청 승인(D-042)·포인트 사용신청 승인(D-041)·정산 운영자 승인([SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) §9)의 5개를 "유지 — 변경하지 않음"으로 명시하고, [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.1은 "ERP Core 엔진(Workflow Engine 등, D-046)을 도입한다고 해서 기존 전용 구조를... 임의로 그 엔진으로 재구현하지 않는다"고 못박는다. 이는 "기능이 동작하는지"보다 "**누군가 실수로 통합하지 않았는지**"를 지키는 회귀 테스트가 핵심이라는 의미다 — 일반적인 정상 동작 테스트와 반대로, 통합이 발생하면 실패해야 하는 negative 테스트 성격이 강하다 |
| **어떻게(레벨)** | **Unit**: Workflow Engine 자체의 단계 전이 로직(순차 승인, 자동승인 조건 평가, 분기 조건) — 신규 워크플로우 기준. **Integration**: 5개 전용 구조 각각이 **여전히 독립된 테이블/엔드포인트를 사용**하는지의 구조적 검증 — 예를 들어 조직 이동 승인 호출이 `organization_transfer_logs`를 경유하고 Workflow Engine의 공용 테이블(workflow_instances 등)을 전혀 거치지 않는지, 조직 이동의 "9개 사유코드·3개 역할 제한"이라는 전용 role guard([ARCHITECTURE.md](ARCHITECTURE.md) §4)가 Workflow Engine의 범용 역할 지정 방식으로 대체되지 않았는지. **Integration**: 정산 운영자 승인(§9 ⑤단계)이 Workflow Engine을 호출하지 않고 기존 경로로만 처리되는지 — 이 회귀 테스트는 Workflow Engine 관련 코드가 변경될 때마다(다른 워크플로우 기능 추가 시에도) CI에서 상시 실행해, "5개 전용 구조 비흡수" 원칙이 미래의 무관한 변경에 의해 우연히 깨지지 않도록 가드레일 역할을 하게 한다 |

### 2.6 API 테스트 — "api는 계산하지 않는다" 원칙 자체의 검증

| 항목 | 내용 |
|---|---|
| **무엇을** | api(NestJS) request handler가 무거운 계산(후원수당/정산/세금/프로모션 판정/보고서 생성)을 직접 수행하지 않고 Job 생성만 하는지 — 이 아키텍처 원칙 자체를 테스트로 신호화 |
| **왜** | [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md) §1.3은 "api(NestJS)의 request handler에 무거운 계산... 을 직접 구현하지 않는다. api는 Job 생성과 상태/결과 조회만 담당한다"고 명시하고, [ARCHITECTURE.md](ARCHITECTURE.md) §1.1/§3도 동일하게 "api request handler는 무거운 계산을 하지 않는다 (확정 — 위반 금지)"고 반복한다. 이 원칙은 코드 리뷰만으로는 시간이 지나며 서서히 깨지기 쉬운(누군가 "임시로" 계산 로직을 api에 넣는) 종류의 위반이므로, **응답 시간을 신호(proxy metric)로 삼는 자동화된 가드**가 필요하다 |
| **어떻게(레벨)** | **Integration**: 정산/수당/세금/프로모션/보고서 생성 관련 모든 api 엔드포인트에 대해 응답이 **202 Accepted + Job id** 패턴을 따르는지 구조적으로 확인([ARCHITECTURE.md](ARCHITECTURE.md) §3) — 200 OK로 계산 결과를 직접 반환하면 그 자체로 실패. **Integration(가드레일, 성능 신호)**: api 응답시간이 일정 임계치(예: p95 200ms~500ms 등 — 정확한 값은 구현 단계에서 운영 데이터로 보정 필요, 본 문서는 "신호로 삼는다"는 원칙만 제시하고 정확한 임계값은 미정 사항으로 남김)를 초과하는 엔드포인트가 발견되면 "계산이 새어들어갔다"는 가설을 세우고 코드 리뷰를 트리거하는 모니터링/알림을 CI 또는 운영 환경에 둔다 — 응답시간 자체가 버그를 증명하지는 않지만(네트워크/DB 지연 등 다른 원인도 있음), Job 생성만 하는 엔드포인트는 본질적으로 빨라야 하므로 이상 신호로는 유효하다. **Unit**: api 모듈(Compensation/Settlement/Compliance 등)의 컨트롤러·서비스 코드에 대해 정적 분석(예: 특정 계산 함수/라이브러리 import 금지 룰)을 두어, worker 전용으로 분리된 계산 함수가 api 패키지에서 import되면 빌드 단계에서 실패하게 하는 린트 규칙 — 이는 테스트라기보다 빌드 가드이지만 동일한 목적(api 계산 금지)을 더 이른 단계에서 강제 |

### 2.7 Idempotency 테스트

| 항목 | 내용 |
|---|---|
| **무엇을** | Job 재시도(Redis Retry) 시 동일 커미션/동일 알림이 중복 생성되지 않는지 |
| **왜** | [DATABASE.md](DATABASE.md) §7.6 운영 위험이 "Redis Retry로 재시도되는 Job(수당 계산, 알림 발송 등)이 중복 실행될 경우, 중복 커미션 지급이나 중복 알림 발송이 발생할 위험"을 명시하고, `idempotency_key` 도입을 §7.2/§8에서 제안한다(Open Decision O-054, [DECISIONS.md](DECISIONS.md)). 금전이 직접 걸린 영역(중복 커미션 지급)이므로 멱등성 결함은 회귀위험이 가장 높은 카테고리 중 하나다 — O-054가 아직 "도입 범위 미확정"이라는 점은 설계 공백이 있다는 의미이므로, 테스트는 구현 단계 진입 시 이 공백이 메워졌는지를 확인하는 역할도 겸한다 |
| **어떻게(레벨)** | **Unit**: 동일 `idempotency_key`로 두 번 호출된 계산 함수/서비스 메서드가 두 번째 호출에서 새 행을 생성하지 않고 기존 결과를 그대로 반환(또는 no-op)하는지. **Integration**: 동일 Job을 의도적으로 두 번(또는 N번) worker에 재투입(실제 Redis Retry 상황을 시뮬레이션)했을 때 `commission_records`/`notification_logs`에 중복 행이 생기지 않는지 — Job 페이로드에 멱등키가 없는 구버전 호환 케이스도 함께 점검(있다면). **Integration**: 알림 발송(Notification Center, worker)의 재시도 시나리오 — 동일 알림이 회원에게 두 번 발송되지 않는지(Email/SMS/KakaoTalk/Push 채널별로 각각 확인, [ARCHITECTURE.md](ARCHITECTURE.md) §2.3). **주의**: O-054(`idempotency_key` 도입 범위)가 미확정이므로, 본 절의 테스트 케이스는 "이 키가 도입된 이후" 전제의 명세다 — 구현 착수 시 O-054가 먼저 확정되어야 본 절의 테스트가 실질적으로 작성 가능하다 |

### 2.8 ERP Core 엔진 테스트 (Workflow/API Center/File Manager/Scheduler/Notification/Audit/Dashboard·Report·Form Builder/System Settings)

| 항목 | 내용 |
|---|---|
| **무엇을** | ERP Core 12개 엔진(Workflow Engine/API Center/File Manager/Scheduler Center/Notification Center/Audit Center/Dashboard·Report·Form Builder/System Settings)이 **업무 모듈(쇼핑몰/MLM/CMS)에 의존하지 않는다**는 D-046 원칙의 검증, 그리고 §2.5에서 다룬 "기존 5개 전용 승인구조 비흡수" 외의 나머지 엔진별 핵심 동작 |
| **왜** | [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md) §1.1 "ERP Core는 업무 모듈에 의존하지 않는다(D-046, 확정 원칙)"는 향후 업무 모듈을 추가/제거해도 ERP Core가 동작해야 한다는 아키텍처 보증이다 — 이 의존 방향이 거꾸로(ERP Core → 업무 모듈) 새지 않았는지는 코드 리뷰만으로 놓치기 쉬워 테스트로 고정할 필요가 있다 |
| **어떻게(레벨)** | **Integration**: 업무 모듈 없이(목업 `subject_type`/`subject_id`만으로) Workflow Engine 인스턴스를 생성·승인·반려할 수 있는지(D-046 의존성 역전 방지). **Integration**: API Center의 외부 연동(`external_api_connections`) CRUD가 PG/3PL/SMS 등 특정 업무 모듈 코드를 import하지 않고 동작하는지. **Unit**: Scheduler Center가 트리거하는 cron이 실제 계산을 포함하지 않고 Job 생성만 하는지(§2.6과 동일한 "계산 분리" 원칙을 scheduler에도 적용). **Integration**: System Settings 허브가 기존 분산 설정(§5.27)을 중복 저장하지 않고 참조만 하는지(`tenant_settings`/`system_security_policies` 등) |

### 2.9 Multi-Tenant 격리 테스트

| 항목 | 내용 |
|---|---|
| **무엇을** | 한 테넌트의 요청/Job/데이터가 다른 테넌트의 데이터에 접근하거나 영향을 주지 않는지 |
| **왜** | [DECISIONS.md](DECISIONS.md) O-159(Multi-Tenant 활성화 시 Job 격리)·O-170(온보딩/모니터링/격리검증)이 아직 미확정이지만, "구조 준비, 활성화는 보류"(D-035) 원칙상 구조 자체는 이미 존재(`tenants`/`tenant_settings`, [DATABASE.md](DATABASE.md) §3.31) — 활성화 이전에도 데이터 격리 테스트 전략을 미리 정의해 두면 활성화 시점에 즉시 검증 가능하다 |
| **어떻게(레벨)** | **Integration**: 테넌트 A의 토큰으로 테넌트 B의 리소스 ID를 직접 호출했을 때 403/404로 거부되는지(RLS 적용 여부와 무관하게 애플리케이션 레벨에서도 검증, O-020 RLS 범위 미확정과는 별개 방어선). **Integration**: 한 테넌트의 대량 배치(Bulk Action/대량 수당계산)가 다른 테넌트의 Job 큐 처리 지연을 유발하지 않는지(O-159 확정 후 구체화 가능 — 확정 전에는 "현재 단일 큐이므로 격리 보장 없음"을 명시적으로 아는 상태로 둔다). **주의**: O-159/O-170이 미확정이므로 본 절도 §2.7과 동일하게 "확정 후 구체화"가 필요한 명세 수준이다 |

### 2.10 쇼핑몰 운영 Phase 2 / SEO / Digital Marketing 테스트 (D-070 신규)

| 항목 | 내용 |
|---|---|
| **무엇을** | `product_seo`(DATABASE.md §3.55) 자동생성 규칙(BR-053)의 우선순위 처리, Schema.org 가격/재고/리뷰평점의 실시간 파생 여부, sitemap.xml/robots.txt의 쿼리타임 생성(BR-054), 카카오/SNS 공유 미리보기 및 Google/Naver Preview의 렌더링 정확성, OG 메타태그 정확성, 상품 Feed(Google Shopping/Facebook/Naver Shopping) 생성·자동갱신, Digital Marketing 연동(`external_api_connections`, PRD.md §5.47) 등록/상태 관리, SEO Dashboard(§5.48) 지표 표시 |
| **왜** | D-069/D-070(쇼핑몰 운영 고도화 및 Phase 2)에서 추가된 기능은 모두 신규 Business Rule(BR-053/BR-054)이거나 기존 ERP Core 패턴(Dashboard Builder/Scheduler Center/External API Center) 재사용이다 — 회귀 위험은 "자동생성값과 관리자 수동값의 우선순위가 뒤바뀌는 것"(BR-053 위반)과 "캐시된 값이 즉시 반영되지 않는 것"(Schema.org 실시간 파생 원칙 위반)에 집중된다. [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)가 강조하는 정산/MLM 계산만큼 금전적으로 치명적이지는 않지만, 검색엔진 노출·소셜 공유 품질에 직접 영향을 주므로 별도 영역으로 추적할 필요가 있다 |
| **어떻게(레벨)** | **SEO 테스트(Unit)**: 관리자가 `product_seo` 필드를 직접 입력하지 않은 경우 상품명→Title/요약설명→Description/대표이미지→OG Image 자동매핑이 정확한지. **SEO 테스트(Integration)**: 관리자가 수동으로 값을 입력한 후에는 그 값이 항상 우선하고 자동생성 로직이 덮어쓰지 않는지(BR-053 핵심 회귀 포인트). **Schema 테스트(Integration)**: 가격/재고/리뷰평점 변경 직후 Schema.org 출력이 즉시 새 값을 반영하는지(저장된 캐시값이 노출되면 실패) — 저장 컬럼이 아니라 매 조회 시 `products`/`product_option_combinations`/리뷰 집계에서 파생되는지 확인. **자동 Sitemap 테스트(Integration)**: 신규 상품 등록/비공개 전환 직후 sitemap.xml에 즉시 반영(추가/제외)되는지, robots.txt가 별도 원장 없이 매 요청 시 동적 생성되는지(BR-054). **OG 테스트(Unit)**: og:title/og:description/og:image이 자동생성값 또는 관리자 입력값 중 BR-053 우선순위대로 정확히 채워지는지. **카카오 공유 테스트 / Google Preview 테스트(E2E, 프론트엔드 렌더링 레벨)**: `tenant_share_images`(§3.55) 업로드 값과 `product_seo` 입력값이 카카오톡/Google/Naver/Facebook/Twitter 미리보기 레이아웃에 정확히 반영되는지 — §5.46.4가 "순수 프론트엔드 렌더링, DB 영향 없음"이라고 명시하므로, 이 테스트는 렌더링 레벨에 한정하고 데이터 저장 레벨 검증과 명확히 분리한다. **Feed 테스트(Unit)**: Google Shopping/Facebook/Naver Shopping 포맷별 스펙 차이(필수 필드/형식)가 정확히 반영되는지. **Feed 테스트(Integration)**: Feed 생성 시 상품/SEO/가격/재고 데이터가 정확히 조합되는지, 자동 갱신 Job이 Scheduler Center 패턴(§2.8과 동일 원칙)대로 동작하는지. **Analytics 설정 테스트(Integration)**: `external_api_connections`에 GA4/GTM/Meta Pixel 등 category 값을 등록했을 때 연동 상태(TESTING/ACTIVE/INACTIVE)가 올바르게 전이하는지 — **실제 GA4/Meta 등으로의 데이터 전송 자체는 본 라운드 설계 범위가 아니므로(PRD.md §5.47, "실제 연동 구현 없음") 이 테스트는 연동 등록/상태 관리 레벨에 한정된다.** **SEO Dashboard 지표 테스트**: O-195(검색엔진 지표를 자체 캐시할지 실시간 API 호출할지)가 미확정이므로, §2.7/§2.9와 동일하게 **본 항목은 O-195 확정 후에야 구체화 가능한 명세 수준**으로 남긴다 |

## 3. 영역 간 우선순위 권고

구현 착수 시 테스트 작성 우선순위는 다음 순서를 권고한다 — 모두 [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)/[ARCHITECTURE.md](ARCHITECTURE.md)가 "위반 금지"로 명시한 영역일수록 우선한다.

1. **35% 법적 한도 게이트**(§2.3) — 법령 hard constraint 위반은 가장 심각한 등급의 리스크
2. **MLM 수당 계산 worked-example 회귀**(§2.1) — 이미 한 차례 실제 회귀(D-014→D-018)가 발생했던 영역
3. **append-only 원장 불변성**(§2.2) — 위반 시 사후 복구가 사실상 불가능한 데이터 손상
4. **Idempotency**(§2.7) — 중복 지급은 직접적 금전 손실
5. **Workflow Engine 비흡수 회귀**(§2.5) — 향후 무관한 기능 추가 중 실수로 깨지기 쉬움
6. **API 계산 분리**(§2.6) — 구조적 원칙이며 직접적 금전 손실보다는 아키텍처 건전성 문제
7. **쇼핑몰/주문**(§2.4) — 일반적인 CRUD/흐름 검증, 다른 영역보다 회귀위험이 상대적으로 낮음
8. **ERP Core 의존성 역전 방지**(§2.8) — D-046 원칙이 깨지면 영향 범위가 넓지만, 발생 빈도는 낮은 구조적 리스크
9. **Multi-Tenant 격리**(§2.9) — Multi-Tenant 활성화(O-090) 전까지는 우선순위가 낮음, 활성화 결정 시 1순위로 재배치
10. **쇼핑몰 운영 Phase 2/SEO/Digital Marketing**(§2.10) — §2.4(쇼핑몰/주문)와 동일한 낮은~중간 우선순위 — 금전적 직접 영향은 없으나 검색엔진 노출/소셜 공유 품질에 영향을 주는 회귀(특히 BR-053 우선순위 위반)는 출시 전 점검 필요

## 4. Open Items (본 문서 범위에서 확정하지 못한 사항)

- O-054(`idempotency_key` 도입 범위) 확정 전까지 §2.7의 테스트는 명세 수준에 머문다.
- §2.6의 "api 응답시간 임계치" 정확한 수치는 운영 데이터 확보 전까지 미정— 구현 단계에서 실측 후 보정 필요.
- O-004(35% 초과 시 정확한 처리 방식 — 배치 전체 보류/비례 축소/마지막 항목만 보류)가 확정되어야 §2.3의 "차단 이후" 동작에 대한 테스트 케이스를 구체화할 수 있다 — 현재는 "③→④ 전이 보류"까지만 테스트 가능.
- O-159/O-170(Multi-Tenant Job 격리/온보딩) 확정 전까지 §2.9는 "단일 테넌트 가정" 테스트만 작성 가능 — 활성화 결정 시 본 절을 구체화해야 한다.
- 재고 예약(O-127)/카테고리 구조(O-128)/반품 상태머신(O-129)이 미확정이므로, §2.4 쇼핑몰 테스트의 재고·반품 관련 케이스는 해당 Open Decision 확정 후 추가한다.
- O-078(실시간 캐시↔Postgres 정합화 Job 주기)이 확정되어야 §2.3의 drift 보정 테스트의 정확한 시간 윈도우를 정할 수 있다.
- O-195(검색엔진 지표 캐시 저장 vs 실시간 API 호출)가 확정되어야 §2.10의 "SEO Dashboard 지표 테스트"를 구체화할 수 있다 — 확정 전에는 등록/상태 관리 레벨 테스트만 작성 가능하다.
