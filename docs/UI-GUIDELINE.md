# UI-GUIDELINE.md — 시각 디자인 가이드라인
> 상태: 신규 v0.1 ([DECISIONS.md](DECISIONS.md) D-063 — 개발 착수 준비 문서 세트, O-169 라이브러리 선정은 권장안으로 미확정 표기) · 최종 수정일: 2026-06-25 · 단계: 설계(Design)
> 전제 문서: [PRD.md](PRD.md) §5.44, [DESIGN-SYSTEM.md](DESIGN-SYSTEM.md)

## 0. 목적과 범위

이 문서는 [DESIGN-SYSTEM.md](DESIGN-SYSTEM.md)가 정의한 컴포넌트들에 적용할 **시각적 토큰**(타이포그래피/색상/spacing/shadow/radius 등)의 권장값을 제시한다. 모든 구체적 수치는 **권장값**이며, 최종 확정은 디자이너/사업팀 검토가 필요하다. 이 문서는 PRD §5.44의 UX 정책(언제 Confirm을 쓰고 언제 Warning을 쓰는지 등)을 바꾸지 않으며, 그 정책을 화면에 그릴 때 쓰는 색/크기/간격만 다룬다.

## 1. Typography (권장값)

| 레벨 | 크기(권장) | Weight(권장) | 용도 |
|---|---|---|---|
| H1 | 32px | 700 (Bold) | 페이지 타이틀 |
| H2 | 28px | 700 (Bold) | 섹션 타이틀 |
| H3 | 24px | 600 (Semibold) | 카드/패널 타이틀 |
| H4 | 20px | 600 (Semibold) | 서브 섹션 타이틀 |
| H5 | 18px | 600 (Semibold) | 위젯 타이틀 |
| H6 | 16px | 600 (Semibold) | 작은 라벨성 타이틀 |
| Body | 14px | 400 (Regular) | 본문/표 데이터/폼 라벨 |
| Caption | 12px | 400 (Regular) | 부가 설명, 타임스탬프, 메타 정보 |

- 줄간격(line-height) 권장: 헤딩 1.3~1.4배, Body/Caption 1.5배.
- 위 수치는 데스크탑 기준 권장값이며, Responsive(§9)에서 모바일 축소 비율을 별도로 다룬다.
- 폰트 패밀리(한글/영문 조합) 선정은 본 문서 범위 밖이며 디자이너 검토 시 별도 확정 필요.

## 2. Color — 시맨틱 컬러 (권장값)

PRD §5.44.3(Toast 정책)·§5.44.9(Toast 종류: Success/Warning/Error/Info)와 1:1 매핑되도록 시맨틱 컬러를 정의한다. Toast뿐 아니라 Badge(§2.4), Button variant(`danger` 등), Dialog severity 표시에도 동일한 토큰을 재사용한다.

| 시맨틱 | 권장 색상(예시 Hex) | 매핑되는 §5.44 요소 |
|---|---|---|
| Primary | #2563EB (Blue 600) | 기본 액션 버튼, 활성 상태 강조 |
| Secondary | #64748B (Slate 500) | 보조 버튼, 비활성 강조 |
| Success | #16A34A (Green 600) | SuccessToast, 승인/완료 Badge |
| Warning | #D97706 (Amber 600) | WarningDialog 강조색, 경고성 Toast/Badge |
| Error | #DC2626 (Red 600) | ErrorToast, 위험 행동(danger) 버튼/Badge |
| Info | #0891B2 (Cyan 600) | InfoToast, 안내성 Badge |

- 각 시맨틱 컬러는 배경/텍스트/보더용으로 명도가 다른 보조 톤(예: 100/600/700 단계)을 함께 정의해야 하며, 정확한 팔레트(50~900 스케일)는 디자이너 검토 시 확정한다.
- 색상만으로 의미를 전달하지 않는다 — 아이콘/텍스트 라벨을 항상 병행한다([DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) WarningDialog/Badge 접근성 항목과 연결).

## 3. Spacing (권장값)

- 4px 기반 스케일을 권장: `4 / 8 / 12 / 16 / 24 / 32 / 48 / 64`(px).
- 컴포넌트 내부 padding은 4의 배수, 컴포넌트 간 margin(레이아웃 간격)은 8의 배수를 권장 — 즉 "4px 단위 미세 조정, 8px 단위 레이아웃 배치"의 2단 원칙.
- 예: Button 내부 padding 8~16px, Card 내부 padding 16~24px, 섹션 간 간격 32~48px.

## 4. Shadow / Border Radius (권장 토큰 단계)

| 토큰 | Shadow 권장값 | 용도 |
|---|---|---|
| shadow-sm | 0 1px 2px rgba(0,0,0,0.05) | Card 기본 |
| shadow-md | 0 4px 6px rgba(0,0,0,0.1) | Dropdown, Tooltip |
| shadow-lg | 0 10px 15px rgba(0,0,0,0.1) | Modal, Dialog |

| 토큰 | Radius 권장값 | 용도 |
|---|---|---|
| radius-sm | 4px | Badge, Input |
| radius-md | 8px | Button, Card |
| radius-lg | 12px | Modal, Dialog |
| radius-full | 999px | Avatar, Pill형 Badge |

## 5. Icon (선정 기준)

- 아이콘 라이브러리(예: Lucide, Heroicons 등) 자체는 **미확정**이며, [DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) §3(O-169)의 컴포넌트 라이브러리 결정과 함께 정해질 가능성이 높다.
- 라이브러리 선정과 무관하게 지켜야 할 **일관성 원칙**:
  - 동일한 stroke 두께/스타일(outline 또는 filled 중 하나)을 전체 화면에서 통일.
  - 동일한 의미는 항상 동일한 아이콘을 사용(예: 삭제는 항상 같은 휴지통 아이콘).
  - 아이콘 전용 버튼은 [DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) ActionButton 접근성 항목대로 `aria-label`을 항상 동반.
  - 시맨틱 컬러(§2)와 아이콘 색상을 혼용할 때는 의미가 충돌하지 않도록(예: 삭제 아이콘에 Success 컬러 사용 금지) 매핑 규칙을 따른다.

## 6. Table 스타일 (권장값)

[DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) §2.1 Table 컴포넌트의 시각 기준이다. 대량 데이터 그리드의 처리 방식(페이지네이션/가상화)은 [DECISIONS.md](DECISIONS.md) O-164(대량 리스트 화면의 페이지네이션 방식(offset vs cursor) 및 그리드 가상화 적용 기준)와 직결되며 **미확정**이므로, 아래는 O-164 확정 여부와 무관하게 적용 가능한 시각 스타일 권장값만 다룬다.

- **행 hover**: 마우스 오버 시 배경색을 Secondary 톤의 가장 옅은 단계(예: 옅은 회색)로 표시해 현재 행을 식별.
- **줄무늬(striped)**: 데이터 행이 많을 때 짝수/홀수 행에 미세한 배경 대비를 권장(가독성), 단 hover 색상과 충돌하지 않는 명도 차이 유지.
- **고정 헤더(sticky header)**: 스크롤 시 헤더가 상단에 고정되어 컬럼 라벨을 항상 유지 — 특히 O-164에서 가상화/대량 스크롤이 채택될 경우 필수 요건이 된다.
- **선택 행 강조**: BulkActionDialog([DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) §1.7)와 연동되는 체크박스 선택 행은 Primary 톤의 옅은 배경으로 구분.

## 7. Card / Badge / Form / Upload / Editor / Chart 스타일 기준

| 컴포넌트 | 스타일 기준(한 줄) |
|---|---|
| Card | shadow-sm + radius-md 기본, hover 시 shadow-md로 승격(clickable Card에 한정). |
| Badge | radius-full(Pill) 또는 radius-sm, 시맨틱 컬러(§2)의 옅은 배경 + 진한 텍스트 조합으로 대비 확보. |
| Form | 라벨은 Body 크기·입력 위 또는 좌측 고정 배치, 에러 상태는 Error 컬러 보더 + 인라인 에러 텍스트(Caption 크기)로 표시. |
| Upload | ImageUploader/FileUploader([DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) §1.9) 드롭존은 점선 보더(radius-md), 드래그 오버 시 Primary 컬러로 보더 강조. |
| Editor | 툴바는 고정 높이의 상단 바로 구분, 본문 영역은 Body 타이포 기준을 그대로 사용해 미리보기와 실제 출력 간 괴리를 줄임. |
| Chart | 시맨틱 컬러(§2)를 데이터 시리즈 색상의 1차 후보로 사용하고, 그 외 시리즈는 보조 팔레트(디자이너 검토 시 확정)를 사용. |

## 8. Responsive (권장 breakpoint)

| 구분 | 권장 기준 |
|---|---|
| 모바일 | ~767px |
| 태블릿 | 768px ~ 1023px |
| 데스크탑 | 1024px 이상 |

- ERP 관리자 화면(Admin Console)은 데스크탑 우선 설계를 권장하되, Partner Portal(회원용)과 쇼핑몰 화면은 모바일 우선 고려가 필요 — 화면 그룹별 우선순위는 디자이너/사업팀 검토 시 확정.
- Table(§6)처럼 컬럼이 많은 컴포넌트는 모바일에서 카드형으로 재배치하는 패턴을 권장(컬럼 수가 많을수록 가로 스크롤보다 카드 전환이 가독성에 유리).

## 9. 접근성 (O-167 연계)

[DECISIONS.md](DECISIONS.md)는 O-167을 다음과 같이 기록한다:

> O-167 | ERP UX Standard의 WCAG 접근성 준수 수준 및 키보드/ARIA 기준 | PRD.md §5.44 | D-062 신규(Gap Analysis)

O-167은 미확정이며, 아래는 그 결정이 내려지기 전까지 적용을 권장하는 **잠정 기준**이다.

- **색상 대비**: 본문 텍스트와 배경 간 대비비 **4.5:1 이상**(WCAG AA 권장), 큰 텍스트(18px 이상 또는 14px Bold 이상)는 **3:1 이상**을 권장. §2의 시맨틱 컬러 팔레트 확정 시 이 기준을 충족하는지 함께 검증 필요.
- **키보드 포커스 표시**: 모든 인터랙티브 요소(Button/Dropdown/Tab/Table 행 등)는 마우스 hover 스타일과 별도로 **키보드 포커스 전용 outline**(예: 2px Primary 컬러 outline + 약간의 offset)을 가져야 하며, `outline: none`으로 포커스 표시를 제거하지 않는다.
- **포커스 순서**: Modal/Dialog 계열([DESIGN-SYSTEM.md](DESIGN-SYSTEM.md) §1.1~1.2, §2.3)은 열릴 때 포커스를 내부로 이동시키고 닫히면 트리거로 복귀하는 focus trap을 기본 원칙으로 한다.
- 위 수치(4.5:1, 3:1, outline 2px 등)는 WCAG AA 일반 권장값을 참고한 잠정안이며, O-167에서 준수 수준(AA 전체 적용 여부, 일부 예외 등)이 확정되면 본 절을 갱신해야 한다.
