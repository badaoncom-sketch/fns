# DESIGN-SYSTEM.md — 공통 컴포넌트 라이브러리
> 상태: 신규 v0.1 ([DECISIONS.md](DECISIONS.md) D-063 — 개발 착수 준비 문서 세트, O-169 라이브러리 선정은 권장안으로 미확정 표기) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 전제 문서: [PRD.md](PRD.md) §5.44, [UI-GUIDELINE.md](UI-GUIDELINE.md)

## 0. 목적과 범위

이 문서는 [PRD.md](PRD.md) §5.44(ERP UX Standard)가 정의한 **정책**(Confirm/Warning Dialog, Toast, Button 상태, Bulk Action, Undo, Activity Log 연계 등)을 실제로 화면에 구현할 때 쓰는 **공통 컴포넌트 15종 + 확장 컴포넌트 15종**을 정의한다. 본 문서는 §5.44의 정책을 변경하지 않으며, 그 위에 컴포넌트 단위의 시각적/구조적 정의를 더한다.

- §5.44.9 기존 15종: 정책 문서에 이미 등록된 Dialog/Toast/Button/Uploader 계열 컴포넌트.
- 확장 15종: 기존 15종만으로는 다루지 않는 데이터 표시/탐색/입력 계열 컴포넌트(Table, Card, Modal 등) — ERP 화면 전반(§5.44.12 적용 범위)에 반복적으로 필요하므로 신규로 정의한다.
- 시각적 토큰(색상/타이포/spacing 등)은 본 문서가 아니라 [UI-GUIDELINE.md](UI-GUIDELINE.md)에서 정의한다. 이 문서는 "어떤 컴포넌트가 있고 무엇을 하는지"만 다룬다.
- 코드(실제 React 구현)는 다루지 않는다 — props 개념과 접근성 고려사항만 정의한다.

## 1. 기존 컴포넌트 15종 (PRD §5.44.9)

PRD §5.44.9 표에 등록된 컴포넌트를 그대로 가져온다. "PRD 연결"은 각 컴포넌트가 구현해야 하는 §5.44 정책 절을 가리킨다.

### 1.1 ConfirmDialog
- **역할**: 저장/수정/삭제/승인/반려/상태변경 등 일반 행동에 대한 확인 레이어.
- **언제 쓰는지**: 위험도가 낮은 행동(되돌리기 쉬운 행동) 전반. "삭제하시겠습니까?"류의 고위험 행동은 WarningDialog를 쓴다.
- **주요 props 개념**: `title`, `description`, `confirmLabel`/`cancelLabel`, `onConfirm`(Loading→완료→Success Toast 흐름 트리거), `onCancel`(화면 변경 없이 닫힘).
- **접근성**: 열릴 때 포커스가 Dialog 내부(기본 액션 버튼 또는 제목)로 이동하고, 닫히면 트리거 요소로 복귀해야 한다(focus trap).
- **PRD 연결**: §5.44.1 Confirm Dialog 정책.

### 1.2 WarningDialog
- **역할**: 삭제/탈퇴/비활성화 등 위험 행동, 그리고 정산승인/포인트승인 등 append-only 원장 기록 행동에 대한 확인 레이어.
- **언제 쓰는지**: 복구가 어렵거나(소프트 삭제 포함) Undo 대상이 아닌(원장 기록) 행동.
- **주요 props 개념**: ConfirmDialog와 동일한 기본 props + `severity`(예: `destructive`) + `ledgerNotice`(불리언 — true면 "이 작업은 원장에 기록되며 되돌리기로 취소할 수 없습니다" 문구를 강제 표시).
- **접근성**: 위험 행동임을 색상에만 의존하지 않고 아이콘+텍스트로 함께 전달(색약 대응), 기본 포커스는 "취소" 버튼에 두어 의도치 않은 확정을 방지.
- **PRD 연결**: §5.44.2 Warning Dialog 정책, §5.44.7 Undo 정책(원장 기록 행동은 Undo 제외 고지).

### 1.3 SuccessToast / ErrorToast / InfoToast
- **역할**: 행동 결과 피드백(성공/오류/정보·경고성 안내).
- **언제 쓰는지**: Confirm/Warning Dialog 처리 완료 후, 또는 Dialog 없이 즉시 처리되는 경량 행동의 결과 통지.
- **주요 props 개념**: `message`, `variant`(`success`/`error`/`info`/`warning` — §5.44.9의 Toast 4종과 동일 어휘), `duration`(자동 닫힘 시간), `action`(예: Undo 버튼).
- **접근성**: `role="status"`(success/info) 또는 `role="alert"`(error)로 스크린리더에 능동적으로 알려야 하며, 자동 닫힘 시간이 너무 짧지 않아야 한다.
- **PRD 연결**: §5.44.3 Toast 정책.

### 1.4 LoadingOverlay
- **역할**: 화면 전체가 처리 중임을 표시하고 그 동안 입력을 막는 차단 레이어.
- **언제 쓰는지**: 페이지 단위의 무거운 처리(예: 대량 데이터 로딩, 일괄 작업 처리 중) — 버튼 단위 로딩은 LoadingButton을 쓴다.
- **주요 props 개념**: `visible`, `message`(선택적 안내 텍스트), `blocking`(입력 차단 여부).
- **접근성**: `aria-busy="true"`와 함께 포커스를 오버레이 내(또는 안내 텍스트)로 유지해 배경 요소 조작을 막아야 한다.
- **PRD 연결**: §5.44.4 Button 상태/Loading 정책(화면 단위 확장), §5.44.6 Bulk Action UX(대량 처리 중 표시).

### 1.5 LoadingButton
- **역할**: Loading 상태를 자체적으로 갖는 버튼 — 클릭 시 처리 중 비활성화.
- **언제 쓰는지**: 모든 행동 버튼의 기본형. PRD가 "모든 버튼은 Loading 상태를 가진다"고 명시.
- **주요 props 개념**: `state`(`default`/`hover`/`disabled`/`loading`/`success`/`error` 6종), `onClick`, Loading 중 `disabled` 강제.
- **접근성**: Loading 중 `aria-disabled`와 `aria-busy`를 함께 설정하고, 스피너만으로 상태를 전달하지 않도록 텍스트 라벨도 함께 유지(예: "저장 중...").
- **PRD 연결**: §5.44.4 Button 상태/Loading 정책(공통 Button 상태 6종, 중복 클릭 방지).

### 1.6 ActionButton / ConfirmActionButton
- **역할**: ActionButton은 일반 행동 버튼, ConfirmActionButton은 ConfirmDialog가 내장되어 클릭 즉시 확인 단계로 이어지는 버튼.
- **언제 쓰는지**: ActionButton은 확인이 필요 없는 즉시 행동, ConfirmActionButton은 §5.44.1 대상 행동(저장/삭제/승인 등)에 직결되는 버튼.
- **주요 props 개념**: `variant`(`primary`/`secondary`/`ghost`/`danger`), `size`, `state`(1.5의 6종과 공유), ConfirmActionButton은 추가로 `confirmConfig`(ConfirmDialog/WarningDialog 중 어느 것을 띄울지).
- **접근성**: 아이콘 전용 버튼은 `aria-label` 필수, `danger` variant는 시각적 색상 외에 텍스트로도 위험도 전달.
- **PRD 연결**: §5.44.1 Confirm Dialog 정책, §5.44.4 Button 상태 정책.

### 1.7 BulkActionDialog
- **역할**: 다건 선택 후 일괄 행동(승인/삭제/지급/발송 등)을 확인하고 결과를 요약하는 레이어.
- **언제 쓰는지**: 체크박스 등으로 다건을 선택한 리스트/그리드 화면의 일괄 처리.
- **주요 props 개념**: `selectedCount`, `actionLabel`, `onConfirm`, 처리 후 `resultSummary`(성공/실패 건수 — Toast로 전달).
- **접근성**: 선택된 건수를 시각 표시(배지 등)뿐 아니라 라이브 리전(`aria-live`)으로도 변경을 알려야 함.
- **PRD 연결**: §5.44.6 Bulk Action UX(문구 패턴 "선택한 {N}건을 {행동}하시겠습니까?" / 결과 "{N}건 중 {성공}건 처리 완료, {실패}건 실패").

### 1.8 UnsavedChangesGuard
- **역할**: 작성 중 이탈(뒤로가기/브라우저 종료/메뉴 이동/취소) 시 저장되지 않은 변경을 감지하고 안내.
- **언제 쓰는지**: 폼/에디터 등 입력 상태를 갖는 모든 작성 화면.
- **주요 props 개념**: `isDirty`(변경 여부), `onStay`("계속 작성"), `onLeave`("나가기").
- **접근성**: 브라우저 네이티브 이탈 확인(beforeunload)과 인앱 라우팅 가드 양쪽에서 동일한 메시지 의미를 유지해야 함.
- **PRD 연결**: §5.44.5 Unsaved Changes Guard.

### 1.9 ImageUploader / FileUploader
- **역할**: 이미지/일반 파일 업로드 입력 컴포넌트.
- **언제 쓰는지**: 상품 이미지, 첨부파일, 배너 이미지 등 파일 입력이 필요한 모든 화면.
- **주요 props 개념**: `accept`(허용 형식), `maxSize`, `multiple`, `onUpload`(ProgressBar 연동), 오류 시 형식/용량 안내 메시지.
- **접근성**: 드래그앤드롭만 제공하지 않고 키보드로 접근 가능한 파일 선택 버튼을 항상 병행 제공.
- **PRD 연결**: §5.44.11 파일 업로드 UX(`system_security_policies.file_upload_policy` 기준).

### 1.10 ImagePreview
- **역할**: 업로드된 이미지의 미리보기 표시.
- **언제 쓰는지**: ImageUploader와 함께 사용, 또는 이미지 순서 변경/대표 이미지 지정 화면(§5.44.10).
- **주요 props 개념**: `src`, `alt`(필수), `onRemove`, `isPrimary`(대표 이미지 표시).
- **접근성**: `alt` 텍스트 미입력 시 업로드를 막거나 자동 대체 텍스트를 강제해야 함.
- **PRD 연결**: §5.44.10 상품 이미지 관리 UX 보강, §5.44.11 파일 업로드 UX.

### 1.11 ProgressBar
- **역할**: 업로드 등 진행률 표시.
- **언제 쓰는지**: ImageUploader/FileUploader의 업로드 진행, 대량 Bulk Action의 비동기 처리 진행(임계값 미확정, O-118).
- **주요 props 개념**: `value`(0~100), `indeterminate`(진행률을 알 수 없을 때).
- **접근성**: `role="progressbar"` + `aria-valuenow`/`aria-valuemin`/`aria-valuemax` 필수.
- **PRD 연결**: §5.44.11 파일 업로드 UX, §5.44.6 Bulk Action UX.

## 2. 확장 컴포넌트 15종

§5.44가 다루는 "행동 전후 확인/피드백 레이어"를 벗어나, ERP 전체 화면(§5.44.12 적용 범위: 회원관리/상품관리/주문관리/정산관리/CMS/Dashboard 등)에 공통으로 필요한 데이터 표시·탐색·입력 컴포넌트를 추가로 정의한다. 이 컴포넌트들은 PRD §5.44에 명시적으로 등록되어 있지 않지만, 동일한 "화면마다 다른 UX를 만들지 않는다"는 원칙(§5.44.12)을 따른다.

### 2.1 Table
- **역할**: 목록형 데이터를 행/열로 표시하는 기본 그리드.
- **언제 쓰는지**: 회원/상품/주문/정산 등 거의 모든 관리자 리스트 화면.
- **주요 props 개념**: `columns`, `data`, `sortable`, `selectable`(BulkActionDialog와 연동되는 행 선택), `pagination` 또는 `virtualized`(대량 데이터).
- **접근성**: `<table>` 시맨틱 구조 또는 동등한 ARIA(`role="grid"`) 사용, 정렬 상태를 `aria-sort`로 전달.
- **PRD 연결**: §5.44.6 Bulk Action UX(행 선택→일괄 처리), 대량 리스트 처리 방식은 [DECISIONS.md](DECISIONS.md) O-164(페이지네이션 방식·그리드 가상화 적용 기준)와 연계 — 미확정.

### 2.2 Card
- **역할**: 정보를 그룹화해 표시하는 박스형 컨테이너.
- **언제 쓰는지**: Dashboard 요약 지표, 상품 카드, 회원 프로필 요약 등.
- **주요 props 개념**: `variant`(`default`/`outlined`/`elevated`), `header`/`footer` 슬롯, `clickable`.
- **접근성**: 클릭 가능한 Card는 버튼/링크와 동일한 키보드 동작(Enter/Space)을 제공해야 함.
- **PRD 연결**: §5.44.12 적용 범위(Dashboard 등 전체 화면에 공통 적용).

### 2.3 Modal
- **역할**: ConfirmDialog/WarningDialog보다 더 복잡한 컨텐츠(폼, 상세 뷰 등)를 띄우는 범용 오버레이 컨테이너.
- **언제 쓰는지**: 단순 확인이 아니라 입력 폼이나 상세 정보를 모달로 띄울 때 — ConfirmDialog/WarningDialog는 Modal 위에 구축되는 특화 형태로 간주.
- **주요 props 개념**: `size`(`sm`/`md`/`lg`/`fullscreen`), `closeOnOverlayClick`, `isDirty` 연동(UnsavedChangesGuard).
- **접근성**: focus trap, `Esc` 닫힘, `aria-modal="true"`, 배경 스크롤 잠금.
- **PRD 연결**: §5.44.5 Unsaved Changes Guard(모달 내 작성 중 이탈 감지)와 함께 사용.

### 2.4 Badge
- **역할**: 상태/카테고리를 짧은 라벨로 표시.
- **언제 쓰는지**: 주문 상태, 회원 등급, 승인 상태 등 짧은 상태 표시 전반.
- **주요 props 개념**: `variant`(시맨틱 컬러와 매핑 — success/warning/error/info/neutral), `size`.
- **접근성**: 색상만으로 상태를 구분하지 않고 텍스트 라벨을 항상 포함.
- **PRD 연결**: §5.44.3 Toast 정책의 시맨틱 컬러 어휘(Success/Warning/Error/Info)를 상태 표시에도 동일하게 적용.

### 2.5 Dropdown
- **역할**: 옵션 목록 중 선택(단일/다중) 또는 행위 메뉴(More actions) 트리거.
- **언제 쓰는지**: 셀렉트 박스, 행별 액션 메뉴(수정/삭제/상태변경 트리거 — 클릭 시 ConfirmActionButton 흐름으로 이어짐).
- **주요 props 개념**: `options`, `multiple`, `searchable`, `placement`.
- **접근성**: `role="listbox"`/`role="menu"`, 방향키 탐색, `aria-expanded`.
- **PRD 연결**: §5.44.1 Confirm Dialog 정책(메뉴에서 트리거되는 행동의 확인 단계로 연결).

### 2.6 Tabs
- **역할**: 동일 화면 내 여러 뷰를 전환.
- **언제 쓰는지**: 상세 화면의 정보/이력/설정 구분 등.
- **주요 props 개념**: `items`, `activeKey`, `onChange`.
- **접근성**: `role="tablist"`/`role="tab"`/`role="tabpanel"`, 방향키로 탭 전환.
- **PRD 연결**: §5.44.5 Unsaved Changes Guard(탭 전환도 "메뉴 이동"에 준하는 이탈로 취급해야 하는 경우 연계).

### 2.7 Accordion
- **역할**: 콘텐츠를 접고 펼치는 구조.
- **언제 쓰는지**: FAQ, 설정 그룹, 긴 폼의 섹션 구분.
- **주요 props 개념**: `items`, `multiple`(여러 패널 동시 펼침 허용 여부), `defaultExpanded`.
- **접근성**: `aria-expanded`, 헤더는 버튼 시맨틱.
- **PRD 연결**: §5.44.12 적용 범위(System Settings 등 설정 화면에서 자주 사용).

### 2.8 Tooltip
- **역할**: 호버/포커스 시 부가 설명 표시.
- **언제 쓰는지**: 아이콘 버튼 설명, 축약된 텍스트의 전체 내용 표시, 정책 보조 설명(예: Undo 가능 시간 안내).
- **주요 props 개념**: `content`, `placement`, `trigger`(`hover`/`focus`/`click`).
- **접근성**: 마우스 호버에만 의존하지 않고 키보드 포커스에서도 노출되어야 함.
- **PRD 연결**: §5.44.7 Undo 정책(복구 가능 시간 등 보조 설명 노출 시).

### 2.9 Avatar
- **역할**: 사용자/회원을 식별하는 이미지 또는 이니셜 표시.
- **언제 쓰는지**: 회원 리스트, 프로필, Activity Log의 행위자 표시.
- **주요 props 개념**: `src`, `fallback`(이니셜), `size`, `status`(온라인 등 부가 표시 — 선택).
- **접근성**: `alt` 텍스트 필수, 이미지 로드 실패 시 텍스트 fallback 보장.
- **PRD 연결**: §5.44.8 Activity Log/Notification 연계(행위자 표시).

### 2.10 Search
- **역할**: 텍스트 기반 검색 입력.
- **언제 쓰는지**: 리스트/Table 상단의 검색, 전역 검색.
- **주요 props 개념**: `value`, `onSearch`, `debounce`, `placeholder`.
- **접근성**: 입력 라벨을 시각적으로 숨기더라도 `aria-label`로 제공.
- **PRD 연결**: §5.44.12 적용 범위(리스트 화면 전반의 탐색 보조).

### 2.11 Filter
- **역할**: 조건 기반으로 리스트 데이터를 좁히는 컨트롤(드롭다운/체크박스/기간 조합).
- **언제 쓰는지**: Table과 함께 사용되는 다중 조건 필터링.
- **주요 props 개념**: `filters`(조건 정의), `value`, `onApply`/`onReset`.
- **접근성**: 적용된 필터 개수/내용을 스크린리더가 인지할 수 있도록 상태 텍스트 제공.
- **PRD 연결**: §5.44.6 Bulk Action UX(필터로 좁힌 대상에 대한 일괄 작업과 연계).

### 2.12 DatePicker
- **역할**: 날짜/기간 선택 입력.
- **언제 쓰는지**: 정산 기간 조회, 주문 기간 검색, 예약 발송 일정 설정 등.
- **주요 props 개념**: `value`, `range`(단일/기간), `minDate`/`maxDate`, `locale`.
- **접근성**: 키보드만으로 달력 탐색 가능해야 하며(방향키), 선택된 날짜를 `aria-live`로 알림.
- **PRD 연결**: §5.44.7 Undo 정책(복구 가능 시간 윈도우 설정 등 날짜·기간 입력이 필요한 정책과 연계).

### 2.13 Editor
- **역할**: 서식 있는 텍스트(리치 텍스트) 입력.
- **언제 쓰는지**: CMS 콘텐츠 작성, 공지/이메일 템플릿 작성.
- **주요 props 개념**: `value`(HTML/JSON 콘텐츠), `toolbarConfig`, `onChange`(UnsavedChangesGuard의 `isDirty` 트리거).
- **접근성**: 툴바 버튼에 `aria-label`, 키보드로 서식 적용 가능(단축키).
- **PRD 연결**: §5.44.5 Unsaved Changes Guard(에디터 작성 중 이탈 감지의 대표 사용처).

### 2.14 Skeleton
- **역할**: 데이터 로딩 중 레이아웃 자리 표시(Placeholder).
- **언제 쓰는지**: 페이지/리스트 최초 로딩 시 LoadingOverlay 대신 콘텐츠 모양을 미리 보여주고자 할 때.
- **주요 props 개념**: `variant`(`text`/`card`/`table-row`), `count`(반복 개수).
- **접근성**: `aria-busy="true"`와 함께 사용, 스크린리더에는 "로딩 중" 텍스트로 대체 전달.
- **PRD 연결**: §5.44.4 Button 상태/Loading 정책(화면 단위 로딩 표현의 대안적 형태).

### 2.15 Empty State
- **역할**: 데이터가 없을 때의 안내 화면(빈 리스트, 검색 결과 없음 등).
- **언제 쓰는지**: Table/리스트에 표시할 데이터가 0건일 때.
- **주요 props 개념**: `icon`/`illustration`, `title`, `description`, `action`(예: "새로 등록" ActionButton).
- **접근성**: 안내 텍스트가 시각 요소(아이콘/일러스트)에만 의존하지 않아야 함.
- **PRD 연결**: §5.44.12 적용 범위(리스트형 화면 전반의 기본 상태 정의).

## 3. 컴포넌트 라이브러리 선정 (O-169)

[DECISIONS.md](DECISIONS.md)는 O-169를 다음과 같이 기록한다:

> O-169 | Design System을 자체 구축할지 기존 컴포넌트 라이브러리(예: shadcn/ui 등)를 채택할지 | PRD.md §5.44 | D-062 신규(Gap Analysis)

**권장안: shadcn/ui(Radix UI + Tailwind CSS) — Next.js 생태계 표준이며 위 컴포넌트 대부분을 기본 제공. 확정 아님, 사용자 최종 승인 필요.**

- 위 30개 컴포넌트(기존 15종 + 확장 15종) 중 Dialog/Modal/Toast/Dropdown/Tabs/Accordion/Tooltip/Avatar/DatePicker 등 다수가 shadcn/ui(Radix UI 프리미티브 기반)에서 기본 제공되거나 약간의 래핑만으로 §5.44 정책에 맞출 수 있다. 단, BulkActionDialog/UnsavedChangesGuard/ConfirmActionButton/Editor 등 ERP 정책에 특화된 조합형 컴포넌트는 어느 라이브러리를 쓰든 자체 래핑이 필요하다.
- **자체 구축 대안의 트레이드오프**: 자체 구축은 초기 개발 속도가 느리고 접근성(O-167)·크로스 브라우저 검증을 처음부터 직접 해야 하는 부담이 있는 반면, 장기적으로는 ERP 특유의 정책(원장 기록 행동 경고, append-only Undo 제외 등)을 컴포넌트 API 레벨에 자유롭게 반영할 수 있는 커스터마이징 자유도가 더 높다. 기존 라이브러리 채택은 반대로 초기 속도가 빠르지만 라이브러리의 설계 철학을 벗어난 요구사항에서는 우회/오버라이드 비용이 발생한다.
- 본 절의 권장안은 최종 결정이 아니며, [DECISIONS.md](DECISIONS.md) O-169가 닫히기 전까지 본 문서의 컴포넌트 정의(역할/props 개념/접근성)는 라이브러리 종류와 무관하게 유효한 추상 정의로 유지한다.

## 4. 접근성 공통 원칙 (O-167)

[DECISIONS.md](DECISIONS.md)는 O-167을 다음과 같이 기록한다:

> O-167 | ERP UX Standard의 WCAG 접근성 준수 수준 및 키보드/ARIA 기준 | PRD.md §5.44 | D-062 신규(Gap Analysis)

O-167은 아직 미확정이며, 준수 수준(WCAG A/AA/AAA)과 세부 키보드/ARIA 기준은 향후 결정 사항이다. 위 1~2절의 "접근성" 항목은 그 결정이 내려지기 전까지 컴포넌트 설계자가 최소한으로 고려해야 할 항목을 나열한 것이며, 구체적인 색상 대비 수치·포커스 표시 기준은 [UI-GUIDELINE.md](UI-GUIDELINE.md) 접근성 절(O-167 연계)에서 권장값으로 제시한다. 본 문서의 접근성 서술은 O-167 확정 이전의 잠정 가이드이며 확정 시 갱신이 필요하다.
