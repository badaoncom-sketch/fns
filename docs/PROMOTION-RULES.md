# PROMOTION-RULES.md — 직급 승급/유지/강급 규정 (폐기됨)

> 상태: **폐기 (Deprecated)** — [DECISIONS.md](DECISIONS.md) D-030 · 최종 수정일: 2026-06-24 · 단계: 설계(Design)

## 폐기 사유

2026-06-24 마케팅 플랜 정합성 전면 점검([DECISIONS.md](DECISIONS.md) §5.10) 결과, **FNS의 실제 마케팅 플랜(D-018 이후 원본 PDF 기반 4축 구조)에는 직급·승급·강급·직급수당 개념이 존재하지 않는다는 점이 확인되었다.**

본 문서가 정의했던 직급 체계(5단계: 회원/실버/골드/플래티넘/다이아, [DECISIONS.md](DECISIONS.md) D-015)와 승급조건 4요소(개인PV/그룹PV/직추천수/하위직급보유)는 D-001~D-017(원본 PDF 검토 이전) 시점에 일반적인 MLM 템플릿을 가정해 설계된 것이며, D-015 확정 이후 단 하나의 구체적 임계값도 정해진 적이 없었다. 4축 보상엔진(유니레벨 후원수당/제품 판매수익/페어보너스/"+알파" 보너스)의 어떤 계산 공식에도 직급은 입력값으로 쓰이지 않는다.

사용자(프로젝트 오너) 확인 후, 직급 체계 전체를 폐기한다([DECISIONS.md](DECISIONS.md) D-030, O-083 해소).

## 영향 범위

직급 체계 폐기에 따라 다음이 함께 변경되었다 — 상세 영향 맵은 [DECISIONS.md](DECISIONS.md) §5.10.2 참조:

- [DATABASE.md](DATABASE.md): `ranks`/`member_rank_history` 테이블, `members.current_rank_id` 컬럼 폐기 (§3.2)
- [ARCHITECTURE.md](ARCHITECTURE.md): api의 "Promotion" 모듈, worker의 "프로모션" Job 제거 (10종→9종)
- [COMPENSATION-RULES.md](COMPENSATION-RULES.md): §4 수당 종류 표의 "직급수당(Rank Bonus)" 행 폐기
- [PRD.md](PRD.md): "직급 관리" MVP 모듈, "조직성장"의 직급변화 집계, Document Center의 직급별 공개범위 옵션 등 제거
- [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md): §1 프로젝트 한 줄 정의에서 "직급 승급" 제거

## 본 파일을 보존하는 이유

FNS는 모든 설계 결정에 append-only 원칙(과거 기록 보존)을 적용한다([DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)). 이 저장소는 git 등 버전관리 시스템으로 백업되지 않으므로, 과거에 어떤 구조가 검토·확정되었는지에 대한 유일한 기록은 문서 자체다 — 파일을 삭제하는 대신 폐기 사실과 사유를 명시한 이 안내문으로 대체한다.

직급 체계의 원래 정의(5단계 구조, 승급조건 4요소, 유지/강급 규정의 상세 내용)는 [DECISIONS.md](DECISIONS.md) D-015 및 §5.10.2의 영향 분석에 요약되어 있다.
