# SITEMAP.md — 전체 메뉴/사이트 구조

> 상태: v0.3 ([DECISIONS.md](DECISIONS.md) D-074 — Dynamic Board Engine: §5.3(게시판 관리, 관리자 메뉴 신규) 추가, §2/§3에 admin-created board 노출 cross-reference 보강(신규 메뉴 행 나열 없음, 게시판 유형은 개방형). D-072 — 쇼핑몰 UX·알림·운영자 대시보드 완성: §1.3(관리자 업무 Queue)/§3(장바구니·상품비교·가격인하알림·배송추적 상세)/§4.5(알림 템플릿 상세)/§4.7(Task Queue·운영자 대시보드 위젯) 보강. D-069 — 쇼핑몰 운영 고도화/SEO 메뉴 추가: §1.2~1.4/§1.12 신규 메뉴) · 최종 수정일: 2026-06-26 · 단계: 설계(Design)
> 전제 문서: [PRD.md](PRD.md), [PROJECT-CONTEXT.md](PROJECT-CONTEXT.md)

본 문서는 [PRD.md](PRD.md) §5.1~§5.44에서 언급된 화면/기능을 메뉴 그룹별로 1Depth > 2Depth > 3Depth 트리로 재구성한 것이다. 각 항목에는 근거가 된 PRD 섹션 번호를 인용한다. 화면명이 PRD상 "미확정"으로 명시된 경우 그대로 표기한다.

## 목차

1. [관리자 메뉴 (Admin Console)](#1-관리자-메뉴-admin-console)
2. [회원 메뉴 (Partner Portal)](#2-회원-메뉴-partner-portal)
3. [쇼핑몰 메뉴 (일반 쇼핑몰)](#3-쇼핑몰-메뉴-일반-쇼핑몰)
4. [ERP Core 메뉴](#4-erp-core-메뉴)
5. [CMS 메뉴](#5-cms-메뉴)
6. [CRM 메뉴](#6-crm-메뉴)
7. [Marketing Program 메뉴](#7-marketing-program-메뉴)

---

## 1. 관리자 메뉴 (Admin Console)

### 1.1 회원관리

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 회원관리 | 회원 목록/검색 | 회원 유형별 필터(개인/사업자/법인/외국인) | [PRD.md](PRD.md) §5.6.1 |
| 회원관리 | 가입심사 | 본인확인 서류 검증 대기 목록 | [PRD.md](PRD.md) §5.6.1, §5.6.3 |
| 회원관리 | 회원 상태 관리 | 가입심사중/활성/휴면/탈퇴/강제탈퇴 전환 | [PRD.md](PRD.md) §5.6.3 |
| 회원관리 | 회원 변경 요청 | 회원 정보 변경 | [PRD.md](PRD.md) §5.3 |
| 회원관리 | 회원 변경 요청 | 회원 명의 변경(IDENTITY_TRANSFER) | [PRD.md](PRD.md) §5.3, §5.13 |
| 회원관리 | 회원 변경 요청 | 회원 탈퇴 처리(WITHDRAWN) | [PRD.md](PRD.md) §5.3 |
| 회원관리 | 회원 변경 요청 | 휴면 처리 | [PRD.md](PRD.md) §5.3 |
| 회원관리 | 회원 변경 요청 | 강제 탈퇴 | [PRD.md](PRD.md) §5.3 |
| 회원관리 | 회원 변경 요청 | 재가입 처리 | [PRD.md](PRD.md) §5.3 |
| 회원관리 | 회원 변경 요청 | 계좌 변경 | [PRD.md](PRD.md) §5.3 |
| 회원관리 | 회원 변경 요청 | 센터 이동 (도입 여부 미확정) | [PRD.md](PRD.md) §5.3, §5.9 |
| 회원관리 | 민감 변경 승인 처리 | 영향분석(수당/정산 시뮬레이션) | [PRD.md](PRD.md) §5.4 |
| 회원관리 | 민감 변경 승인 처리 | Snapshot/Audit Log 조회 | [PRD.md](PRD.md) §5.4 |
| 회원관리 | 회원 활동 이력 조회 | 최근 로그인/주문/수당/정산 | [PRD.md](PRD.md) §5.14 |
| 회원관리 | 추천 링크 통계 | 전체 추천 링크 통계(클릭/가입/전환율) | [PRD.md](PRD.md) §5.17.3 |
| 회원관리 | 추천 링크 통계 | 회원별 전환율 | [PRD.md](PRD.md) §5.17.3 |
| 회원관리 | 추천 링크 통계 | 링크별 가입 통계 | [PRD.md](PRD.md) §5.17.3 |

### 1.2 상품관리

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 상품관리 | 상품 목록/등록/수정 | 상품 대표 썸네일 등록 | [PRD.md](PRD.md) §5.43 |
| 상품관리 | 상품 목록/등록/수정 | 상품 목록/상세/제품소개/성분표/사용방법/인증서 이미지 등록 | [PRD.md](PRD.md) §5.43 |
| 상품관리 | 상품 목록/등록/수정 | PDF 첨부 / 동영상 URL 등록 | [PRD.md](PRD.md) §5.43 |
| 상품관리 | 상품 목록/등록/수정 | 이미지 순서 변경 / 미리보기 / 삭제 | [PRD.md](PRD.md) §5.43, §5.44.10 |
| 상품관리 | 상품옵션 관리 | 옵션/옵션조합 | [PRD.md](PRD.md) §5.41 |
| 상품관리 | 브랜드/제조사 관리 | 브랜드별 상품 모아보기(브랜드관) 연동 | [PRD.md](PRD.md) §5.21, §5.41 |
| 상품관리 | 연관상품/추천상품 관리 | | [PRD.md](PRD.md) §5.41 |
| 상품관리 | 재입고알림 관리 | | [PRD.md](PRD.md) §5.41 |
| 상품관리 | 배송비정책 관리 | 무료배송 기준 | [PRD.md](PRD.md) §5.41 |
| 상품관리 | 패키지 관리 | 패키지 목록/생성/수정/비활성화 | [PRD.md](PRD.md) §5.1.4 |
| 상품관리 | 패키지 관리 | 패키지별 정책 설정(추천수당/페어보너스/유니레벨 포함 여부/자격 부여 여부) | [PRD.md](PRD.md) §5.1.4 |
| 상품관리 | 패키지 관리 | 패키지별 판매 통계 | [PRD.md](PRD.md) §5.1.4 |
| 상품관리 | 상품문의/리뷰 관리 | | [PRD.md](PRD.md) §5.21, §5.41 |
| 상품관리 | 쿠폰정책 관리 | (구조 후속 확정 필요) | [PRD.md](PRD.md) §5.21, §5.41 |
| 상품관리 | 회원할인 관리 | 회원 유형별/그룹별 할인 | [PRD.md](PRD.md) §5.41 |
| 상품관리(신규, D-069) | 상품 일괄 작업 | 상품 복사 / 일괄 등록·수정·삭제 / Import·Export(CSV/Excel) | [PRD.md](PRD.md) §5.45.1, O-126 종속 |
| 상품관리(신규, D-069) | 상품 운영 설정 | 예약 공개/종료 · 판매중지 · 노출순위 · 진열순서 · 검색순위 · 판매기간 · 판매상태 · 승인 · 변경이력 | [PRD.md](PRD.md) §5.45.1 |
| 상품관리(신규, D-069) | 옵션/재고 고도화 | 옵션별 SKU/바코드/이미지/가격/할인/판매중지/배송비, 안전재고/Hold/백오더/LOT/유통기한 | [PRD.md](PRD.md) §5.45.1~5.45.2, O-176/O-177 |
| 상품관리(신규, D-069) | 상품 SEO 관리 | SEO Title/Description/Keywords/Slug/OG/Twitter Card/Schema.org, 자동생성 + 수동우선(BR-053) | [PRD.md](PRD.md) §5.46.1 |
| 상품관리(신규, D-069) | 리뷰 운영 | 리뷰 이미지/동영상/신고/답변/베스트리뷰/포인트 | [PRD.md](PRD.md) §5.45.3 |
| 상품관리(신규, D-069) | 프로모션 — Bundle(One+One) | MLM 패키지 엔진과 별개의 일반 묶음판매 | [PRD.md](PRD.md) §5.45.3, O-191 |

### 1.3 주문관리

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 주문관리 | 주문 목록/검색 | 일반 주문 / 정기배송 주문 구분 | [PRD.md](PRD.md) §5.1.3.3, §5.1.3.5 |
| 주문관리 | 결제 내역 조회 | | [PRD.md](PRD.md) §5.1.3.5 |
| 주문관리 | 반품/교환/환불 처리 | 청약철회권 연계 | [PRD.md](PRD.md) §5.5, §5.1.3.5 |
| 주문관리 | 정기배송 관리(관리자) | 자동결제 처리 이력 | [PRD.md](PRD.md) §5.1.3.3 |
| 주문관리(신규, D-069) | 주문 운영 보강 | 메모/CS메모/송장변경/병합/분리/부분교환, PG실패·재결제·가상계좌·무통장입금·복합결제 | [PRD.md](PRD.md) §5.45.2 |
| 주문관리(신규, D-072) | 관리자 업무 Queue | 결제실패/무통장입금확인/송장등록필요/배송지연/반품·교환·환불 승인대기/CS미응답 등 — 기존 테이블 횡단 조회(신규 큐 테이블 아님) | [PRD.md](PRD.md) §5.60 |

### 1.4 배송관리

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 배송관리 | 3PL 연동 관리 | 연동 대상 3PL 업체(미확정) | [PRD.md](PRD.md) §5.5 |
| 배송관리 | 재고 관리 | SKU별 본사/3PL 창고 재고 현황 | [PRD.md](PRD.md) §5.5 |
| 배송관리 | 송장(배송) 처리 | 택배사 연동, 배송 상태 추적 | [PRD.md](PRD.md) §5.5 |
| 배송관리 | 반품 처리 | 접수/검수/환불 연계 | [PRD.md](PRD.md) §5.5 |
| 배송관리 | 환수(재고 회수) | 탈퇴/강제탈퇴 시 재고 환수, 불량/리콜 회수 | [PRD.md](PRD.md) §5.5 |
| 배송관리(신규, D-069) | 배송 운영 보강 | 묶음/예약배송, 출고보류, 배송사변경, 송장 일괄업로드, 배송비 정산 | [PRD.md](PRD.md) §5.45.2 |

### 1.5 정산관리

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 정산관리 | 정산 배치 생성/조회 | [SETTLEMENT-RULES.md](SETTLEMENT-RULES.md) 기준 | [PRD.md](PRD.md) §5.1(5.1 MVP 범위 표) |
| 정산관리 | 세금 처리 | 원천징수, 사업자/법인 세무 처리 | [PRD.md](PRD.md) §5.1, §5.6.1 |
| 정산관리 | 지급 내역서 | | [PRD.md](PRD.md) §5.1 |
| 정산관리 | 운영자 승인 | (Workflow Engine으로 이전하지 않음 — 기존 구조 유지) | [PRD.md](PRD.md) §5.30.3 |

### 1.6 MLM(후원수당 계산) 관리

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| MLM관리 | 후원수당 계산 배치 | Unilevel Sponsor Plan(LINE별) | [PRD.md](PRD.md) §5.1 |
| MLM관리 | 제품 판매수익/페어보너스 산출 | 이벤트 기반, 본인 패키지구매 자격 검증 | [PRD.md](PRD.md) §5.1 |
| MLM관리 | "+알파" 보너스 적립 산출 | 배치 처리 | [PRD.md](PRD.md) §5.1 |
| MLM관리 | Rule Designer (도입 권고, 최종 미확정) | LINE 비율/프로모션/정산 조건 편집(Draft) | [PRD.md](PRD.md) §5.15 |
| MLM관리 | Rule Designer | rule_versions 시뮬레이션(dry-run) | [PRD.md](PRD.md) §5.15 |
| MLM관리 | Rule Designer | 발행(Publish) 및 발행 이력 | [PRD.md](PRD.md) §5.15 |

### 1.7 조직관리 (§5.16)

```
조직관리                                                          [PRD.md §5.16.8]
├─ 조직 조회
├─ 조직 이동
├─ 조직 이동 이력
├─ 조직 이동 영향 분석
├─ 조직 병합        ※ 기능 정의 미확정 — DECISIONS.md O-065
├─ 조직 분리        ※ 기능 정의 미확정 — O-065
├─ 조직 복구        ※ 기능 정의 미확정 — O-065
└─ 조직 통계
```

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 조직관리 | 조직 조회 | 전체 추천조직 트리 조회("조직도") | [PRD.md](PRD.md) §5.1(조직 항목), §5.16.8 |
| 조직관리 | 조직 이동 | 조직 이동 요청(9개 사유 코드 선택) | [PRD.md](PRD.md) §5.16.3, §5.16.4 |
| 조직관리 | 조직 이동 | 증빙 첨부 | [PRD.md](PRD.md) §5.16.4 |
| 조직관리 | 조직 이동 | 영향 분석(수당/정산) | [PRD.md](PRD.md) §5.16.4 |
| 조직관리 | 조직 이동 | Snapshot 생성/관리자 승인 | [PRD.md](PRD.md) §5.16.4 |
| 조직관리 | 조직 이동 | 적용일 예약(기본값: 승인일 익월 1일) | [PRD.md](PRD.md) §5.16.4 |
| 조직관리 | 조직 이동 | 긴급 조직 이동(5개 사유, 승인 즉시 적용, SuperAdmin만) | [PRD.md](PRD.md) §5.16.7 |
| 조직관리 | 조직 이동 이력 | 과거 모든 조직 이동 내역 | [PRD.md](PRD.md) §5.16.6 |
| 조직관리 | 조직 이동 이력 | 조직 이동 전후 비교(스냅샷) | [PRD.md](PRD.md) §5.16.6 |
| 조직관리 | 조직 이동 이력 | 조직 이동 승인 이력(승인/반려, 승인자) | [PRD.md](PRD.md) §5.16.6 |
| 조직관리 | 조직 이동 이력 | 조직 이동 첨부파일 관리 | [PRD.md](PRD.md) §5.16.6 |
| 조직관리 | 조직 이동 이력 | 조직 이동 사유 코드 관리(9종 카탈로그) | [PRD.md](PRD.md) §5.16.6 |
| 조직관리 | 조직 이동 이력 | 적용 예정 조직 이동 조회(대기 목록) | [PRD.md](PRD.md) §5.16.6 |
| 조직관리 | 조직 이동 영향 분석 | §5.16.4 ③ 단계 결과 조회 | [PRD.md](PRD.md) §5.16.6 |
| 조직관리 | 조직 병합 | (정의 미확정 — O-065) | [PRD.md](PRD.md) §5.16.8 |
| 조직관리 | 조직 분리 | (정의 미확정 — O-065) | [PRD.md](PRD.md) §5.16.8 |
| 조직관리 | 조직 복구 | (정의 미확정 — O-065) | [PRD.md](PRD.md) §5.16.8 |
| 조직관리 | 조직 통계 | | [PRD.md](PRD.md) §5.16.8 |

### 1.8 법적 한도 모니터링 (§5.18)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 법적한도모니터링 | 35% 모니터링 대시보드 | 현재 후원수당 비율(실시간/배치 정합화) | [PRD.md](PRD.md) §5.18 |
| 법적한도모니터링 | 35% 모니터링 대시보드 | 예상 후원수당 비율(연말 예측) | [PRD.md](PRD.md) §5.18 |
| 법적한도모니터링 | 35% 모니터링 대시보드 | 국가별 비율 | [PRD.md](PRD.md) §5.18 |
| 법적한도모니터링 | 35% 모니터링 대시보드 | 임계치 표시(30%/33%/35%) | [PRD.md](PRD.md) §5.18 |
| 법적한도모니터링 | 35% 모니터링 대시보드 | 리딩 인디케이터(패키지 매출 비중, 페어 성립률) | [PRD.md](PRD.md) §5.18 |
| 법적한도모니터링 | 차단 이력 | 정산 배치 보류 이력, 검토/해제 처리 | [PRD.md](PRD.md) §5.18 |

### 1.9 공제조합 보고센터 (§5.7)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 공제조합보고센터 | 보고서 생성 | 국가별 규제기관 형식 보고서 | [PRD.md](PRD.md) §5.7 |
| 공제조합보고센터 | 매출/후원수당/비율 자동 집계 | compliance_ratio_snapshots 인용(§5.18과 동일 값) | [PRD.md](PRD.md) §5.7 |
| 공제조합보고센터 | 보고 일정 관리 | 국가별 보고 주기/마감일, 마감 임박 알림 | [PRD.md](PRD.md) §5.7 |
| 공제조합보고센터 | 제출 이력 관리 | 제출 상태(생성/검토/제출완료) | [PRD.md](PRD.md) §5.7 |

### 1.10 국가/센터/권한 설정

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 국가관리 | 국가 활성화/비활성화 | 신규 국가 추가, Reserved(CN) 처리 | [PRD.md](PRD.md) §5.6.2 |
| 국가관리 | 국가별 규칙 설정 | 마케팅 플랜 버전/세금규칙/프로모션/정산규칙 | [PRD.md](PRD.md) §5.8 |
| 센터관리 (도입 여부 미확정) | 센터별 집계 | 매출/조직/수당 집계 | [PRD.md](PRD.md) §5.9 |
| 센터관리 (도입 여부 미확정) | 센터 이동 처리 | | [PRD.md](PRD.md) §5.3, §5.9 |
| 권한관리 | 관리자 역할 관리 | SuperAdmin/Compliance Admin/지정 조직관리 관리자/CountryAdmin/CenterAdmin/기능별 관리자 | [PRD.md](PRD.md) §5.6.4 |
| 권한관리 | 역할 부여/회수 | 감사로그 기록 | [PRD.md](PRD.md) §5.6.4 |

### 1.11 고객지원(관리자)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 고객지원 | CS Center | 1:1문의/정산문의/수당문의/회원문의/명의변경문의/이의신청/불만접수 처리 | [PRD.md](PRD.md) §5.11 |

### 1.12 SEO/공유 이미지 관리 (신규, D-069)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| SEO/공유이미지 관리 | 사이트 기본 SEO | Site Title/Description/Keywords, Default Meta/OG/Twitter Image, Favicon/Apple Touch Icon, Canonical Base URL, Default Robots | [PRD.md](PRD.md) §5.46.2 |
| SEO/공유이미지 관리 | 공유 이미지 업로드 | 카카오톡/메인주소/상품/이벤트/프로모션/브랜드 공유 이미지(권장 규격 안내) | [PRD.md](PRD.md) §5.46.2 |
| SEO/공유이미지 관리 | 페이지별 SEO | 쇼핑몰 메인/카테고리/브랜드관/기획전/이벤트/Lifestyle Program/공지사항/FAQ/회사소개/회원가입/로그인 | [PRD.md](PRD.md) §5.46.3 |
| SEO/공유이미지 관리 | SEO Preview | Google/Naver 검색결과, KakaoTalk/Facebook/X 공유, 모바일 미리보기 | [PRD.md](PRD.md) §5.46.4 |
| SEO/공유이미지 관리 | SEO 일괄 작업 | SEO 일괄 수정, OG 이미지 일괄 업로드, alt text 일괄 수정 | [PRD.md](PRD.md) §5.45.4 |

> CMS(§5.10/§5.19), Notification Center(§5.12/§5.34), CRM Center(§5.40), ERP Core 10개 엔진(§5.30~§5.39)은 별도 메뉴 그룹(4장/5장/6장)에서 다룬다 — 본 장과 중복 정의하지 않는다.

---

## 2. 회원 메뉴 (Partner Portal)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 내 조직 | LINE1~5 조직 조회 | 추천 깊이별 하위 조직(조회 전용) | [PRD.md](PRD.md) §5.1.1 |
| 내 조직 | 조직매출 | LINE1~5 매출 합계 | [PRD.md](PRD.md) §5.1.1 |
| 내 조직 | 조직수당 | 직추천 라인 후원수당(Sponsor Bonus) 합계 | [PRD.md](PRD.md) §5.1.1 |
| 내 조직 | 조직성장 | 신규가입 추이 | [PRD.md](PRD.md) §5.1.1 |
| 마이오피스 (화면명 미확정) | 추천 링크/QR코드 | 2종 형식 추천 링크 | [PRD.md](PRD.md) §5.17.2 |
| 마이오피스 | 추천 활동 통계 | 링크 클릭 수/회원가입 수/5만원 구매 회원 수/400만원 패키지 구매 회원 수 | [PRD.md](PRD.md) §5.17.2 |
| 마이오피스 | 자격 현황 (화면명 미확정) | 유니레벨 후원수당 자격 현황(당월 5만원 진행률) | [PRD.md](PRD.md) §5.1.2 |
| 마이오피스 | 자격 현황 | 제품 판매수익·페어보너스 자격 현황(400만원 패키지 구매 여부) | [PRD.md](PRD.md) §5.1.2 |
| 마이오피스 | 제품 판매수익 내역 | 직추천 패키지 구매 시 발생한 100만원 수익 목록 | [PRD.md](PRD.md) §5.1.2 |
| 마이오피스 | 페어보너스 현황 | 페어 진행 현황(30일 Pair Window), 200만원 내역 | [PRD.md](PRD.md) §5.1.2 |
| 마이오피스 | "+알파" 적립 현황 | 여행/자동차/자기계발 보너스 누적액/진행률 | [PRD.md](PRD.md) §5.1.2 |
| 회원몰 — 유지구매센터 | 유니레벨 자격 진행률 | 당월 5만원 이상 구매 진행률, 부족분 안내 | [PRD.md](PRD.md) §5.1.3.2, §5.1.3.4 |
| 회원몰 — 유지구매센터 | 정기배송 등록 현황 | 제품/주기/다음 배송일/결제수단 목록 | [PRD.md](PRD.md) §5.1.3.4 |
| 회원몰 — 유지구매센터 | 당월 자동결제 처리 이력 | 결제 성공/실패 내역(자동결제센터와 공유) | [PRD.md](PRD.md) §5.1.3.4 |
| 회원몰 — 정기배송센터 | 정기배송 조회 | | [PRD.md](PRD.md) §5.1.3.2, §5.1.3.3 |
| 회원몰 — 정기배송센터 | 정기배송 변경 | 주기/제품/일시정지 변경 | [PRD.md](PRD.md) §5.1.3.3 |
| 회원몰 — 정기배송센터 | 정기배송 해지/재개 | | [PRD.md](PRD.md) §5.1.3.3 |
| 회원몰 — 자동결제센터 | 결제수단 등록/조회/변경 | | [PRD.md](PRD.md) §5.1.3.2 |
| 회원몰 — 자동결제센터 | 당월 자동결제 처리 이력 조회 | | [PRD.md](PRD.md) §5.1.3.2 |
| 회원몰 메인 | 프로그램 배너 | Marketing Program 배너(로그인 회원 대상) | [PRD.md](PRD.md) §5.1.5.1 |
| 회원몰 메인 | 포인트 현황/진행률 | "+알파" 보너스 개인화 데이터(§5.1.2와 동일 소스) | [PRD.md](PRD.md) §5.1.5.1 |
| Lifestyle Program 상세 | 프로그램 소개/이미지 갤러리 | | [PRD.md](PRD.md) §5.1.5.3 |
| Lifestyle Program 상세 | 포인트 정책/누적 현황 | 본인 누적액/진행률(로그인 회원) | [PRD.md](PRD.md) §5.1.5.3 |
| Lifestyle Program 상세 | 참여 조건/첨부파일/FAQ | | [PRD.md](PRD.md) §5.1.5.3 |
| 회원 변경 신청 | 회원 정보 변경 신청 | | [PRD.md](PRD.md) §5.3 |
| 회원 변경 신청 | 명의 변경 신청 | 전자서명 필요 | [PRD.md](PRD.md) §5.3, §5.13 |
| 회원 변경 신청 | 탈퇴 신청 | 전자서명 필요 | [PRD.md](PRD.md) §5.3, §5.13 |
| 회원 변경 신청 | 계좌 변경 신청 | | [PRD.md](PRD.md) §5.3 |
| 회원 활동 이력 | 최근 로그인/주문/수당/정산 | 회원 자기열람용(조회 성능 최적화) | [PRD.md](PRD.md) §5.14 |
| Document Center (회원 화면) | 정책 문서 | 약관, 개인정보처리방침 | [PRD.md](PRD.md) §5.10 |
| Document Center (회원 화면) | 보상플랜 안내 | 공개용 설명 자료 | [PRD.md](PRD.md) §5.10 |
| Document Center (회원 화면) | 교육자료 | 제품/사업 교육 콘텐츠 | [PRD.md](PRD.md) §5.10 |
| Document Center (회원 화면) | 회원 개인 문서 | 정산자료/지급명세서/사업자자료/원천징수자료(시스템 생성) | [PRD.md](PRD.md) §5.10 |
| Customer Service Center (회원 화면) | 1:1문의/정산문의/수당문의/회원문의/명의변경문의/이의신청/불만접수 | | [PRD.md](PRD.md) §5.11 |
| 프로그램 신청 | 신청 내역 조회 | 신청/승인대기/승인/반려/참여중/완료 상태 | [PRD.md](PRD.md) §5.24 |
| 포인트 | 사용 신청 | 실물/서비스 전환 신청(승인 대기) | [PRD.md](PRD.md) §5.23 |
| 포인트 | 사용 이력 | 적립/사용/취소/만료 타임라인 | [PRD.md](PRD.md) §5.23 |

> "내 조직"/"마이오피스"/회원몰의 화면 통합 여부(탭 구조로 합칠지)는 PRD상 미확정이다([PRD.md](PRD.md) §5.17.2, §8 Open Questions) — 본 트리는 PRD가 구분한 기능 단위 그대로 별도 1Depth로 나열했다.

> **마이오피스 노출(신규, D-074)**: 관리자가 생성한 게시판은 `boards.my_office_exposure=true`로 설정 시 마이오피스 영역에 동적으로 노출될 수 있다 — 게시판 유형이 개방형(admin-defined)이므로 본 트리에 가상의 게시판명을 별도 메뉴 행으로 나열하지 않는다. 관리 화면은 §5.3 참조.

---

## 3. 쇼핑몰 메뉴 (일반 쇼핑몰)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 쇼핑몰 메인 | 메인 슬라이드 배너 | 비회원 노출 포함 | [PRD.md](PRD.md) §5.1.5.1, §5.21 |
| 쇼핑몰 메인 | 메인 팝업 | | [PRD.md](PRD.md) §5.21 |
| 쇼핑몰 메인 | 프로모션 배너 / 이벤트 배너 | | [PRD.md](PRD.md) §5.1.5.1 |
| 상품목록 | 카테고리 배너 | | [PRD.md](PRD.md) §5.21 |
| 상품목록 | 베스트상품/인기상품/추천상품/신상품/타임세일 | | [PRD.md](PRD.md) §5.21 |
| 상품목록 | 브랜드관 | 브랜드별 상품 모아보기 | [PRD.md](PRD.md) §5.21 |
| 상품목록 | 이벤트관/기획전 | marketing_programs 관련 상품 연결로 구현 | [PRD.md](PRD.md) §5.21 |
| 상품목록 | 검색 / 검색어 순위 | | [PRD.md](PRD.md) §5.21 |
| 상품상세 | 상품 이미지(대표/목록/상세/제품소개/성분표/사용방법/인증서) | | [PRD.md](PRD.md) §5.1.3.5, §5.43 |
| 상품상세 | 상품문의 / 리뷰 | | [PRD.md](PRD.md) §5.21 |
| 상품상세 | 최근 본 상품 / 관심상품(찜) | | [PRD.md](PRD.md) §5.21 |
| 상품상세 | "정기배송으로 구매" 옵션 | 배송 주기/결제수단 지정 | [PRD.md](PRD.md) §5.1.3.3 |
| 상품상세 | 재입고알림 신청 / 가격인하 알림 신청(신규, D-072) | `product_price_alerts` | [PRD.md](PRD.md) §5.41, §5.58.1 |
| 상품상세 | 상품 비교 | `product_comparisons`(D-070) | [PRD.md](PRD.md) §5.45.3, §5.58.1 |
| 장바구니(D-072 — `carts`/`cart_items` 데이터 모델 신설) | 옵션·수량 변경 / 선택 삭제 | 품절·가격변경 표시(파생), 배송비 예상 표시(파생), 쿠폰 적용 가능 여부(파생) | [PRD.md](PRD.md) §5.58.2 |
| 장바구니 | 관심상품으로 이동 / 나중에 구매하기 | 처리방식 미확정(O-197) | [PRD.md](PRD.md) §5.58.2 |
| 주문서(결제 전) | 쿠폰 적용 / 포인트 사용 / 복합결제 | | [PRD.md](PRD.md) §5.1.3.5, §5.21, §5.58.3 |
| 결제 | PG 연동 / 결제 실패 후 재시도 | 자동결제와 동일 정책 공유 예정(미확정), 재시도는 §3.18(STATE-MACHINE) | [PRD.md](PRD.md) §5.1.3.5, §5.58.3 |
| 주문완료 | | | [PRD.md](PRD.md) §5.1.3.5 |
| 배송조회 | 배송 추적(택배사/송장번호/배송상태/타임라인) | `shipments` 컬럼 명료화(D-072), [STATE-MACHINE.md](STATE-MACHINE.md) §17 | [PRD.md](PRD.md) §5.1.3.5, §5.58.4 |
| 반품/교환/환불 신청 | 진행 상태 타임라인 | 청약철회권 연계, [STATE-MACHINE.md](STATE-MACHINE.md) §16 | [PRD.md](PRD.md) §5.1.3.5, §5.5, §5.58.5 |
| 공지사항 | | Document Center 연동 | [PRD.md](PRD.md) §5.1.3.5, §5.10 |
| FAQ | | 신규 콘텐츠 종류 | [PRD.md](PRD.md) §5.1.3.5, §5.19.2 |
| 1:1문의 | | Customer Service Center 연동 | [PRD.md](PRD.md) §5.1.3.5, §5.11 |
| 브랜드소개 / 회사소개 | | 정적 콘텐츠 페이지 | [PRD.md](PRD.md) §5.1.3.5 |
| 이용약관 / 개인정보처리방침 | | Document Center 연동 | [PRD.md](PRD.md) §5.1.3.5, §5.10 |
| 마이페이지 (로그인 회원) | 위시리스트 / 최근 본 상품 / 상품 비교함 | | [PRD.md](PRD.md) §5.21, §5.58.1 |
| 마이오피스 알림함 | 알림 이력 조회 | 기존 `notifications` 조회 — 신규 채널 아님 | [PRD.md](PRD.md) §5.56 |

> 비회원(End Customer)의 일반 쇼핑몰 접근 허용 여부는 미확정(O-017, §5.2) — 본 트리는 카탈로그 자체의 메뉴 구조만 다룬다.

> **쇼핑몰/회원몰 노출(신규, D-074)**: 관리자가 생성한 게시판은 `boards.shop_exposure`/`shop_member_exposure=true` 설정에 따라 일반 쇼핑몰 또는 회원몰 영역에 동적으로 노출될 수 있다 — 게시판 유형이 개방형(admin-defined)이므로 본 트리에 가상의 게시판명을 별도 메뉴 행으로 나열하지 않는다. 관리 화면은 §5.3 참조.

---

## 4. ERP Core 메뉴

ERP Core는 모든 업무 모듈이 공통으로 사용하는 엔진 계층이다([PRD.md](PRD.md) §5.28). Authentication/Authorization은 기존 구조의 재인식(§5.29, 변경 없음)이므로 본 트리에서는 제외하고, 신규/보강된 10개 엔진만 메뉴로 나열한다.

### 4.1 Workflow

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Workflow | Workflow 목록/생성 | Workflow명/단계/담당자/승인자/자동승인 여부/알림/조건 설정 | [PRD.md](PRD.md) §5.30.1 |
| Workflow | 적용 대상 관리 | 환불승인/반품승인/교환승인/전자결재승인 등 신규 워크플로우 | [PRD.md](PRD.md) §5.30.2 |
| Workflow | 기존 전용 승인구조 (참고, 변경 없음) | 조직이동/회원변경/프로그램신청/포인트사용신청/정산승인 — Workflow Engine 미대체 | [PRD.md](PRD.md) §5.30.3 |

### 4.2 API Center

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| API Center | 외부 연동 목록 | API명/상태(ACTIVE/INACTIVE/TESTING)/인증키(Vault 참조)/Endpoint | [PRD.md](PRD.md) §5.31 |
| API Center | 연동 테스트 | ping/sandbox 호출 | [PRD.md](PRD.md) §5.31 |
| API Center | 로그 | 호출 이력 | [PRD.md](PRD.md) §5.31 |
| API Center | 실패이력 | 실패 호출 필터링 | [PRD.md](PRD.md) §5.31 |
| API Center | 재시도 정책 | | [PRD.md](PRD.md) §5.31 |

### 4.3 File Manager

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| File Manager | 폴더 | 계층형 폴더 구조 | [PRD.md](PRD.md) §5.32 |
| File Manager | 태그 / 검색 | 파일명/태그/카테고리 기준 | [PRD.md](PRD.md) §5.32 |
| File Manager | 버전 관리 | 재업로드 시 이전 버전 보존 | [PRD.md](PRD.md) §5.32 |
| File Manager | 다운로드 / 미리보기 | 이미지/PDF 등 | [PRD.md](PRD.md) §5.32 |
| File Manager | 권한 관리 | 폴더/파일 단위 접근 제어 | [PRD.md](PRD.md) §5.32 |

### 4.4 Scheduler

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Scheduler Center | Job 목록 | 실행주기(cron)/다음실행/최근실행 | [PRD.md](PRD.md) §5.33 |
| Scheduler Center | 실행 결과 | 성공/실패 표시 | [PRD.md](PRD.md) §5.33 |
| Scheduler Center | 재실행 | 실패 Job 수동 재트리거 | [PRD.md](PRD.md) §5.33 |
| Scheduler Center | 로그 | 실행 이력 | [PRD.md](PRD.md) §5.33 |

### 4.5 Notification

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Notification Center | 알림 템플릿 관리 | 채널/언어/국가별 템플릿(템플릿 버전관리), 제목·본문·변수·활성/비활성(D-072), 테스트발송/미리보기(DB 영향 없음) | [PRD.md](PRD.md) §5.12, §5.34, §5.57 |
| Notification Center | 발송 설정 | 예약발송/조건발송/국가별/회원유형별/대상그룹 발송 | [PRD.md](PRD.md) §5.34 |
| Notification Center | 자동발송 | 이벤트 기반(주문완료 등) — 이벤트 카탈로그는 [PRD.md](PRD.md) §5.56(D-072) | [PRD.md](PRD.md) §5.34, §5.56 |
| Notification Center | 발송 이력 | 발송로그 / 재발송 / 실패관리 | [PRD.md](PRD.md) §5.12, §5.34 |

### 4.6 Audit

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Audit Center | 검색/필터 | 기간/사용자/모듈/IP/행위/변경내용 | [PRD.md](PRD.md) §5.35 |
| Audit Center | 다운로드 | Report Builder 재사용 | [PRD.md](PRD.md) §5.35 |
| Audit Center | 감사보고 | 정기 감사보고서 생성(Report Builder 재사용) | [PRD.md](PRD.md) §5.35 |

### 4.7 Dashboard Builder

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Dashboard Builder | 위젯 배치 | Drag & Drop | [PRD.md](PRD.md) §5.36 |
| Dashboard Builder | 위젯 데이터 소스 선택 | 회원/매출/주문/배송/정산/수당/포인트/재고/국가/Marketing Program | [PRD.md](PRD.md) §5.36 |
| Dashboard Builder | 위젯 유형 선택 | 차트/테이블/KPI/Task Queue(D-072) | [PRD.md](PRD.md) §5.36 |
| Dashboard Builder(신규, D-072) | 운영자 대시보드 위젯 | 오늘 지표/긴급 처리 지표/쇼핑몰 운영 지표 — 기존 트랜잭션 테이블 집계, 신규 테이블 없음 | [PRD.md](PRD.md) §5.59 |

### 4.8 Report Builder

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Report Builder | 보고서 구성 | 조건검색/필터/정렬 | [PRD.md](PRD.md) §5.37 |
| Report Builder | 출력 | Excel/PDF/CSV, 출력 대상(회원/주문/매출/배송/정산/수당/포인트/재고/CRM/Marketing Program) | [PRD.md](PRD.md) §5.37 |
| Report Builder | 예약 생성/발송 | Scheduler Center 연동, Notification Center 발송 | [PRD.md](PRD.md) §5.37 |

### 4.9 Form Builder

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Form Builder | 신청서/설문 생성 | 회원가입(보조 항목)/문의/체험단/Marketing Program 신청/교육/이벤트/승인요청 | [PRD.md](PRD.md) §5.38 |
| Form Builder | 입력 필드 유형 | Text/Textarea/Select/Checkbox/Radio/File/Image/Date | [PRD.md](PRD.md) §5.38 |
| Form Builder | Validation 설정 | 필드별 필수/형식 검증 | [PRD.md](PRD.md) §5.38 |

### 4.10 System Settings

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| System Settings | 허브(통합 진입점) | 회사정보/SMTP/SMS/Push/PG/3PL/API (API Center·Notification Center·Multi-Tenant가 실제 관리) | [PRD.md](PRD.md) §5.39 |
| System Settings | 로그 정책 | 보존기간(Audit Center 연동) | [PRD.md](PRD.md) §5.39 |
| System Settings | 백업 정책 | 주기/보존기간 | [PRD.md](PRD.md) §5.39 |
| System Settings | 보안 정책 | 2차인증(2FA) 강제 여부 | [PRD.md](PRD.md) §5.39 |
| System Settings | 보안 정책 | IP 제한(관리자 콘솔 화이트리스트) | [PRD.md](PRD.md) §5.39 |
| System Settings | 보안 정책 | 비밀번호 정책(최소 길이/복잡도) | [PRD.md](PRD.md) §5.39 |
| System Settings | 보안 정책 | 세션 정책(타임아웃) | [PRD.md](PRD.md) §5.39 |
| System Settings | 파일업로드 정책 | 허용 확장자/최대 용량(File Manager 적용) | [PRD.md](PRD.md) §5.39 |
| System Settings | Multi-Tenant 설정 | 회사명/로고/도메인/컬러/언어/통화/국가/약관/쇼핑몰/CMS/MLM/정산 설정 | [PRD.md](PRD.md) §5.26, §5.42 |

---

## 5. CMS 메뉴

### 5.1 Document Center (§5.10)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Document Center | 문서 업로드/등록 | 정책 문서/보상플랜 안내/교육자료/공지사항/회원 개인 문서 | [PRD.md](PRD.md) §5.10 |
| Document Center | 문서 버전관리 | 변경 시 이전 버전 보존 | [PRD.md](PRD.md) §5.10 |
| Document Center | 문서 공개범위 설정 | 전체/특정 국가/특정 회원유형 | [PRD.md](PRD.md) §5.10 |
| Document Center | 국가별 문서 관리 | 동일 문서 종류, 국가별 다른 파일 | [PRD.md](PRD.md) §5.10 |

### 5.2 페이지/팝업/배너/FAQ (§5.19)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 콘텐츠 CMS | 회사소개/브랜드소개/CEO인사말/연혁/비전/사업소개 | 고정 슬러그(`cms_pages`, page_type) | [PRD.md](PRD.md) §5.19.1 |
| 공지 CMS | 공지사항/이벤트/뉴스 | 목록형, 작성일 기준 정렬 | [PRD.md](PRD.md) §5.19.1 |
| 페이지 CMS | CUSTOM 페이지 생성 | 관리자가 슬러그 직접 지정(캠페인 랜딩 등) | [PRD.md](PRD.md) §5.19.1 |
| FAQ CMS | FAQ 카테고리 관리 | | [PRD.md](PRD.md) §5.19.2 |
| FAQ CMS | FAQ 항목 관리 | 일반 FAQ / Marketing Program 종속 FAQ(scope 구분) | [PRD.md](PRD.md) §5.19.2 |
| 팝업 CMS | 메인 팝업 | 쇼핑몰 메인 진입 시 노출 | [PRD.md](PRD.md) §5.19.3 |
| 팝업 CMS | 회원 팝업 | 로그인 회원 전용 | [PRD.md](PRD.md) §5.19.3 |
| 팝업 CMS | 국가별 팝업 | 특정 국가 한정 | [PRD.md](PRD.md) §5.19.3 |
| 팝업 CMS | 노출 설정 | 팝업명/이미지/링크URL/노출기간/노출빈도(미확정)/정렬순서/활성여부 | [PRD.md](PRD.md) §5.19.3 |
| 배너 CMS | 쇼핑몰 메인 슬라이드 배너 | | [PRD.md](PRD.md) §5.1.5.2, §5.19.4 |
| 배너 CMS | 쇼핑몰 프로모션 배너 | | [PRD.md](PRD.md) §5.1.5.2, §5.19.4 |
| 배너 CMS | 쇼핑몰 이벤트 배너 | | [PRD.md](PRD.md) §5.1.5.2, §5.19.4 |
| 배너 CMS | 회원몰 프로그램 배너 | Marketing Program 배너로 통합 | [PRD.md](PRD.md) §5.1.5.2, §5.19.4 |
| 배너 CMS | 카테고리 배너 | 상품 카테고리 페이지 상단(신규) | [PRD.md](PRD.md) §5.19.4 |
| 배너 CMS | 배너 설정 | 배너명/썸네일/PC이미지/모바일이미지/노출시작일/노출종료일/링크URL/정렬순서/활성여부 | [PRD.md](PRD.md) §5.1.5.2 |
| 약관 CMS | 이용약관/개인정보처리방침/국가별 약관 | Document Center 기존 관리 | [PRD.md](PRD.md) §5.19.5 |
| 약관 CMS | 마케팅 수신동의 | 신규 약관 유형, consent_history 기록 | [PRD.md](PRD.md) §5.19.5 |
| 이메일/SMS/Push CMS | 템플릿 관리 | Notification Center(§4.5)와 동일 데이터, CMS 분류상 재배치 | [PRD.md](PRD.md) §5.19.6 |
| 다국어 CMS | 번역 오버레이 관리 | 전체 CMS 공통, 언어 탭 입력(번역 없으면 기본 언어 폴백) | [PRD.md](PRD.md) §5.19.7 |

### 5.3 Board Management (게시판 관리, 신규, D-074)

> 관리자가 코드 배포 없이 새 게시판 유형(보도자료/갤러리/자료실/이벤트/홍보영상/교육자료/제품자료/인증자료/사용후기/회사소개/CSR/채용/IR 등)을 직접 생성하는 범용 게시판 엔진이다(`boards`/`board_categories`/`board_posts`/`board_post_comments`/`board_post_likes`, [DATABASE.md](DATABASE.md) §3.58). **§5.1(Document Center)/§5.2(페이지/팝업/배너/FAQ)의 기존 CMS 메뉴는 변경하지 않는다** — Board Engine은 그 옆에 신설되는 병렬 관리자 모듈이다. 기존 CMS 콘텐츠를 Board Engine으로 통합·마이그레이션할지 여부는 미확정(O-200).

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 게시판 관리(신규, D-074) | 게시판 목록 | 게시판명/코드/유형/활성여부/노출범위 한눈에 조회 | [PRD.md](PRD.md) §5.67.2, [DATABASE.md](DATABASE.md) §3.58 |
| 게시판 관리(신규, D-074) | 게시판 생성 | 게시판명/코드/설명/유형(`board_type`, 자유 확장 분류값)/사용 국가/사용 언어 | [PRD.md](PRD.md) §5.67.1~5.67.3, [DATABASE.md](DATABASE.md) §3.58 |
| 게시판 관리(신규, D-074) | 게시판 설정(유형·레이아웃·노출범위·기능 ON/OFF) | 레이아웃(리스트/카드/갤러리/FAQ/영상, `layout_type`) · 노출범위(메뉴/쇼핑몰/회원몰/마이오피스/메인 각각 독립 토글, `menu_exposure`/`shop_exposure`/`shop_member_exposure`/`my_office_exposure`/`main_exposure`) · 메뉴그룹·노출순서(`menu_group`/`sort_order`) · 기능 ON/OFF(댓글/답글/파일첨부/이미지/대표이미지/동영상/다운로드/조회수/좋아요/공유/SEO/OG/예약게시/승인후게시/카테고리/태그/검색/RSS, `feature_flags` JSON) | [PRD.md](PRD.md) §5.67.2, §5.67.4, §5.67.6, §5.67.8, [DATABASE.md](DATABASE.md) §3.58 |
| 게시판 관리(신규, D-074) | 게시글 관리 | 게시글 목록/등록/수정/삭제, 카테고리·태그·예약게시·공개범위 설정(`board_posts`) | [PRD.md](PRD.md) §5.67.5, [DATABASE.md](DATABASE.md) §3.58 |
| 게시판 관리(신규, D-074) | 게시글 승인(조건부) | `feature_flags.승인후게시=true`인 게시판만 노출 — Workflow Engine 재사용(`subject_type='BOARD_POST_APPROVAL'`), 신규 승인 구조 아님 | [PRD.md](PRD.md) §5.67.7, [DATABASE.md](DATABASE.md) §3.58 |

---

## 6. CRM 메뉴

### 6.1 CRM Center (§5.40)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| CRM Center | 회원상담 | 상담 등록/조회 | [PRD.md](PRD.md) §5.40 |
| CRM Center | 상담예약 | | [PRD.md](PRD.md) §5.40 |
| CRM Center | 전화기록 | | [PRD.md](PRD.md) §5.40 |
| CRM Center | 메모 | | [PRD.md](PRD.md) §5.40 |
| CRM Center | 상담이력 | | [PRD.md](PRD.md) §5.40 |
| CRM Center | 상담상태 관리 | 진행중/완료/Follow-up필요 | [PRD.md](PRD.md) §5.40 |
| CRM Center | Follow-up 추적 | 후속 조치 필요 상담(Workflow Engine 또는 단순 상태값) | [PRD.md](PRD.md) §5.40 |
| CRM Center | 관심상품 (재사용) | product_wishlists 참조 | [PRD.md](PRD.md) §5.40 |
| CRM Center | 회원활동 (재사용) | member_activity_logs 참조 | [PRD.md](PRD.md) §5.40 |

### 6.2 Customer Service Center (§5.11)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| CS Center | 1:1 문의 | 일반 문의 | [PRD.md](PRD.md) §5.11 |
| CS Center | 정산 문의 | | [PRD.md](PRD.md) §5.11 |
| CS Center | 수당 문의 | | [PRD.md](PRD.md) §5.11 |
| CS Center | 회원 문의 | 회원정보/가입 관련 | [PRD.md](PRD.md) §5.11 |
| CS Center | 명의변경 문의 | §5.3 IDENTITY_TRANSFER 절차 연계 | [PRD.md](PRD.md) §5.11 |
| CS Center | 이의신청 | 강제탈퇴 등 처분에 대한 이의(O-027 해소) | [PRD.md](PRD.md) §5.11 |
| CS Center | 불만 접수 | 일반 불만/클레임 | [PRD.md](PRD.md) §5.11 |

> CRM Center와 CS Center를 하나의 화면(탭 구조)으로 통합할지는 미확정([PRD.md](PRD.md) §5.40, §8 Open Questions D-057) — CS Center는 CRM Center가 대체하지 않는다.

---

## 7. Marketing Program 메뉴

### 7.1 Marketing Program Engine (§5.20)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Marketing Program | 프로그램 목록/생성/수정 | 프로그램명/코드/카테고리(자유 생성)/국가/언어 | [PRD.md](PRD.md) §5.20.2, §5.20.3 |
| Marketing Program | 프로그램 콘텐츠 관리 | 썸네일/대표이미지/상세이미지/동영상/첨부PDF/소개글/참여방법/유의사항 | [PRD.md](PRD.md) §5.20.2 |
| Marketing Program | 프로그램 노출 설정 | 노출기간/정렬순서/활성여부, 배너 연결(§5.19.4) | [PRD.md](PRD.md) §5.20.2, §5.20.3 |
| Marketing Program | 카테고리 관리 | 자유 생성(Lifestyle/Promotion/Campaign/Event/Seminar/Travel/Golf/Education/VIP/Mission/Coupon 등) | [PRD.md](PRD.md) §5.20.1, §5.20.3 |
| Marketing Program | MLM 보상 연동 설정 | links_to_compensation 플래그(Lifestyle 등 보상 결부 프로그램 구분) | [PRD.md](PRD.md) §5.20.2 |
| Marketing Program | 프로그램별 통계 | 조회수/클릭수/신청수/승인수/완료수/포인트 지급액/사용액/참여율 | [PRD.md](PRD.md) §5.20.3, §5.25 |
| Marketing Program | 관련 상품 연결 | marketing_program_products(N:N) | [PRD.md](PRD.md) §5.22 |

### 7.2 프로그램 신청 관리 (§5.24)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| 프로그램 신청 관리 | 신청 목록 | 신청/승인대기/승인/반려/참여중/완료 | [PRD.md](PRD.md) §5.24 |
| 프로그램 신청 관리 | 승인/반려 처리 | 사유 기록, 관리자 전용 승인 권한 | [PRD.md](PRD.md) §5.24 |
| 프로그램 신청 관리 | 신청서 설정 | requires_application 여부(프로그램별), Form Builder 연동 가능 | [PRD.md](PRD.md) §5.24, §5.38 |

### 7.3 Lifestyle Program ("+알파" 보너스 마케팅 노출, §5.1.5)

| 1Depth | 2Depth | 3Depth | 근거 |
|---|---|---|---|
| Lifestyle Program | 배너 관리 | 쇼핑몰 메인/프로모션/이벤트/회원몰 프로그램 배너(4종) | [PRD.md](PRD.md) §5.1.5.1, §5.1.5.2 |
| Lifestyle Program | 상세페이지 관리 | 프로그램 소개/이미지 갤러리/포인트 정책/참여조건/첨부파일/FAQ | [PRD.md](PRD.md) §5.1.5.3 |

> Lifestyle Program은 §5.20의 Marketing Program Engine 일반화에 따라 `category = LIFESTYLE`인 프로그램 인스턴스로 재배치된다([PRD.md](PRD.md) §5.20.1) — 엔진·관리자 내부 용어는 "+알파" 보너스를 그대로 유지한다.
