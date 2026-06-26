# GIT-WORKFLOW.md — GitHub 개발 Workflow

> 목적: FNS ERP 개발을 GitHub Issue, Project, Branch, Pull Request, Release 단위로 운영하기 위한 절차를 정의한다.
> 범위: GitHub 운영 방식만 다룬다. 코드, Repository 구조, 설계, Business Rule, MLM, 정산, Workflow, ERP Core 구조는 변경하지 않는다.

## 1. 기본 원칙

- 모든 개발 작업은 GitHub Issue에서 시작한다.
- 모든 구현은 `feature/*` 브랜치에서 진행한다.
- Pull Request는 `develop`을 대상으로 생성한다.
- `main`은 릴리스된 안정 버전만 유지한다.
- Design Freeze 이후 설계 변경은 [DESIGN-FREEZE.md](DESIGN-FREEZE.md)의 Bug / Change Request / New Feature 절차를 따른다.
- 구현 전에는 [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md)의 문서 읽기 순서를 먼저 따른다.
- 금지 항목은 [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)를 따른다.

## 2. Branch Strategy

| Branch | 목적 | Merge 대상 | 규칙 |
|---|---|---|---|
| `main` | 운영/릴리스 기준 브랜치 | 없음 | 직접 Push 금지, Pull Request 필수, Review 필수 |
| `develop` | 통합 개발 브랜치 | `main` 또는 `release/*` | Pull Request 기반 Merge |
| `feature/*` | Issue 단위 기능/작업 브랜치 | `develop` | Issue 번호를 브랜치명에 포함 |
| `release/*` | 릴리스 후보 안정화 브랜치 | `main`, `develop` | 버그 수정과 릴리스 문서 정리만 허용 |
| `hotfix/*` | 운영 긴급 수정 브랜치 | `main`, `develop` | 긴급 수정 후 양쪽에 반영 |

### Branch Naming

```text
feature/{issue-number}-{short-name}
release/v{version}
hotfix/{issue-number}-{short-name}
```

예:

```text
feature/12-member-approval
release/v1.0.0-alpha
hotfix/88-settlement-export
```

## 3. GitHub 개발 절차

```text
Issue 생성
↓
Feature Branch 생성
↓
Codex 개발
↓
Pull Request
↓
Review
↓
Merge develop
↓
Release
↓
Merge main
```

### 3.1 Issue 생성

- 모든 작업은 Issue로 추적한다.
- Issue에는 Type, Area, Priority label을 붙인다.
- Epic 하위 작업은 관련 Epic Issue를 본문에 연결한다.
- 구현 Issue는 참조 문서와 완료 조건을 명시한다.

### 3.2 Feature Branch 생성

- 대상 Issue가 `Ready` 상태일 때 `feature/*` 브랜치를 만든다.
- 브랜치는 최신 `develop`에서 생성한다.
- 하나의 브랜치는 하나의 Issue 해결을 기본 단위로 한다.

### 3.3 Codex 개발

- Codex는 Issue 본문, 참조 문서, [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md), [DO-NOT-TOUCH.md](DO-NOT-TOUCH.md)를 확인한 뒤 작업한다.
- MLM/정산/Workflow/ERP Core 구조 변경이 필요하면 바로 구현하지 않고 Change Request로 전환한다.
- Open Decision을 임의로 확정하거나 삭제하지 않는다.

### 3.4 Pull Request

- Pull Request 대상은 기본적으로 `develop`이다.
- PR 본문에는 관련 Issue, 변경 요약, 검증 결과, 남은 리스크를 적는다.
- 코드 변경 PR은 테스트 또는 검증 로그를 포함한다.

### 3.5 Review

- Review는 설계 위반, Business Rule 변경, 계산 로직 위치, 보안, 테스트 누락을 우선 확인한다.
- Review 중 설계 변경이 발견되면 [DESIGN-FREEZE.md](DESIGN-FREEZE.md)의 Change Request 절차를 따른다.

### 3.6 Merge develop

- Review 승인 후 `develop`에 병합한다.
- 병합 전 충돌, 테스트 실패, 문서 누락이 없어야 한다.

### 3.7 Release

- 릴리스 후보는 `release/*` 브랜치에서 안정화한다.
- 릴리스 범위는 [RELEASE-ROADMAP.md](RELEASE-ROADMAP.md)를 따른다.
- 릴리스 노트는 CHANGELOG와 GitHub Release에 남긴다.

### 3.8 Merge main

- 검증이 끝난 `release/*` 브랜치만 `main`에 병합한다.
- `main` 병합 후 version tag를 생성한다.
- hotfix는 `main` 반영 후 반드시 `develop`에도 되돌려 반영한다.

## 4. GitHub Project 운영

Project 이름:

```text
FNS ERP Development
```

Project Status 컬럼:

```text
Backlog
Ready
In Progress
Code Review
Testing
Done
Blocked
```

운영 기준:

- 새 Epic은 기본적으로 `Backlog`에 둔다.
- 착수 준비가 끝난 Feature/Task는 `Ready`로 이동한다.
- 작업 브랜치가 생성되면 `In Progress`로 이동한다.
- PR이 생성되면 `Code Review`로 이동한다.
- Review 후 검증 단계는 `Testing`으로 이동한다.
- 완료된 작업은 `Done`으로 이동한다.
- 외부 결정, 권한, 법무, 미확정 Open Decision 때문에 진행할 수 없으면 `Blocked`로 이동한다.

## 5. Milestone

| Milestone | 용도 |
|---|---|
| `v1.0.0-alpha` | 초기 통합 개발 및 내부 검증 |
| `v1.0.0-beta` | v1.0 후보 기능 안정화 |
| `v1.0.0` | Core ERP KR 릴리스 |
| `v1.1.0` | BI/AI 보강 릴리스 |
| `v2.0.0` | Global/Multi-Tenant 활성화 릴리스 |

## 6. Label

### Type

```text
epic
feature
task
bug
cr
release
documentation
```

### Area

```text
erp-core
member
shop
order
payment
inventory
logistics
cms
crm
marketing
mlm
settlement
multi-tenant
api
database
worker
scheduler
frontend
backend
security
performance
```

### Priority

```text
priority: critical
priority: high
priority: medium
priority: low
```

## 7. Branch Protection 설정

### 7.1 `main`

필수 설정:

- 직접 Push 금지
- Pull Request 필수
- Review 필수
- 병합 전 최신 상태 요구
- force push 금지
- branch deletion 금지

GitHub 설정 경로:

```text
Repository → Settings → Branches → Branch protection rules → Add rule
```

설정값:

```text
Branch name pattern: main
Require a pull request before merging: enabled
Require approvals: enabled
Require status checks to pass before merging: enabled
Require branches to be up to date before merging: enabled
Do not allow bypassing the above settings: enabled
Restrict who can push to matching branches: enabled
Allow force pushes: disabled
Allow deletions: disabled
```

### 7.2 `develop`

필수 설정:

- Pull Request 기반 Merge
- 직접 Push 제한 권장
- CI 도입 후 status check 필수화

GitHub 설정 경로:

```text
Repository → Settings → Branches → Branch protection rules → Add rule
```

설정값:

```text
Branch name pattern: develop
Require a pull request before merging: enabled
Require status checks to pass before merging: enabled after CI setup
Allow force pushes: disabled
Allow deletions: disabled
```

## 8. GitHub Project 생성 권한

GitHub Projects v2를 API로 생성하거나 상태 필드를 수정하려면 토큰에 Project 권한이 필요하다.

필요 scope:

```text
read:project
project
repo
```

토큰 권한이 없으면 Milestone, Label, Issue는 생성할 수 있어도 Project board 생성과 Status 컬럼 설정은 실패한다.

## 9. 하지 말아야 할 것

- GitHub 운영 준비 중 코드 작성 금지
- Next.js/NestJS/Monorepo/package.json 생성 금지
- Database Migration 작성 금지
- 설계 문서 내용 변경 금지
- Business Rule 변경 금지
- Open Decision 확정/삭제 금지
- MLM/정산/Workflow/ERP Core 구조 변경 금지
