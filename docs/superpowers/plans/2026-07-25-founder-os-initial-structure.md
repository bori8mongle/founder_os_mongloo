# Founder OS Initial Structure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the approved private, domain-based Founder OS workspace with safe Git defaults, live external-source references, reusable Figma ideation indexing, and starter templates.

**Architecture:** Git stores Markdown context, decisions, indexes, and templates. Google Workspace, Notion, Figma, Drive, and the separate app repository remain canonical for live collaborative data, design originals, sensitive originals, and source code; this repository stores links and usage rules instead of duplicated exports.

**Tech Stack:** Git, Markdown, Google Drive/Sheets connector, Figma links, shell validation

## Global Constraints

- Keep the repository organized by work domain, not by sharing permissions.
- Use numbered English top-level folder names; Korean document titles and content are allowed.
- Keep one canonical copy of each document and cross-link it from other domains.
- Treat Google Sheets, Google Docs, Notion, Drive, Figma, and the app repository as external sources of truth where specified.
- Never commit passwords, API keys, tokens, certificates, resident registration numbers, account details, `.env` files, or sensitive original evidence.
- Work directly on `main` for routine document updates; use branches only for large restructuring, risky experiments, or future Git-based team collaboration.
- Commit one independently understandable work unit at a time with `area: change` messages.

---

## File Map

### Core repository controls

- Modify: `README.md` — entry point, storage decision guide, and daily Git workflow
- Create: `AGENTS.md` — rules for live external-source reads, canonical files, and sensitive data
- Create: `.gitignore` — Python, secret, build, and macOS exclusions

### Intake and dashboards

- Create: `00_inbox/README.md` — temporary-capture policy
- Create: `01_dashboard/README.md` — dashboard usage
- Create: `01_dashboard/now.md` — current priorities
- Create: `01_dashboard/backlog.md` — deferred tasks
- Create: `01_dashboard/recurring.md` — recurring work

### Domain guides

- Create: `10_meetings/README.md`
- Create: `20_brand_marketing/README.md`
- Create: `30_app/README.md`
- Create: `40_grants/README.md`
- Create: `50_business_admin/README.md`
- Create: `80_resources/README.md`
- Create: `90_templates/README.md`
- Create: `99_archive/README.md`

### Live-source indexes

- Create: `80_resources/external-sources.md` — registry for live Sheets, Docs, Drive, Notion, and Figma sources
- Create: `80_resources/app-repositories.md` — registry for app source repositories used as read-only planning references

### Figma ideation library

- Create: `30_app/ideation/README.md` — capture and reuse workflow
- Create: `30_app/ideation/catalog.md` — searchable idea index
- Create: `30_app/ideation/ideas/README.md` — one-file-per-idea rule
- Create: `90_templates/app-idea.md` — idea analysis template

### Working templates

- Create: `90_templates/meeting-note.md`
- Create: `90_templates/app-plan.md`
- Create: `90_templates/qa-checklist.md`
- Create: `90_templates/grant-program.md`

### Empty planned locations

Create `.gitkeep` in these directories so the approved structure exists before content arrives:

```text
10_meetings/2026/
20_brand_marketing/strategy/
20_brand_marketing/owned/website/
20_brand_marketing/owned/youtube/
20_brand_marketing/owned/instagram/
20_brand_marketing/paid/meta/
20_brand_marketing/paid/google/
30_app/planning/
30_app/design/
30_app/operations/
30_app/reviews/
30_app/qa/
30_app/web/
30_app/references/
40_grants/opportunities/
40_grants/templates/
40_grants/programs/
50_business_admin/evidence/
50_business_admin/tax_finance/
```

---

### Task 1: Add safe repository controls

**Files:**

- Create: `.gitignore`
- Modify: `README.md`
- Create: `AGENTS.md`

**Interfaces:**

- Consumes: approved structure design in `docs/superpowers/specs/2026-07-24-founder-os-structure-design.md`
- Produces: repository-wide storage, safety, and external-source rules used by every later task

- [ ] **Step 1: Create `.gitignore`**

Use this exact content:

```gitignore
# Python virtual environments
.venv/
venv/

# Python caches
__pycache__/
*.py[cod]
.pytest_cache/
.mypy_cache/
.ruff_cache/

# Python build output
build/
dist/
*.egg-info/

# Local secrets and environment values
.env
.env.*
!.env.example

# macOS
.DS_Store
```

- [ ] **Step 2: Verify ignored files**

Run:

```bash
git check-ignore -v .DS_Store .env .venv/bin/python __pycache__/sample.pyc
```

Expected: four output lines pointing to matching `.gitignore` rules.

- [ ] **Step 3: Replace `README.md` with the workspace entry point**

Use this exact content:

```markdown
# Founder OS Mongloo

창업자의 회의, 브랜딩·마케팅, 앱 기획·운영, 정부지원사업, 경영지원 업무를
정리하는 비공개 운영 저장소입니다.

## 어디에 저장할까?

| 자료 | 저장 위치 |
| --- | --- |
| 아직 분류하지 않은 메모 | `00_inbox` |
| 현재 할 일과 반복 업무 | `01_dashboard` |
| 회의록과 결정사항 | `10_meetings` |
| 브랜드·콘텐츠·광고 | `20_brand_marketing` |
| 앱 아이디어·기획·QA·운영 | `30_app` |
| 정부지원사업 | `40_grants` |
| 증빙·세무·재무 업무의 메모와 원본 위치 | `50_business_admin` |
| 외부 Sheets·Docs·Drive·Notion·Figma·앱 저장소 링크 | `80_resources` |
| 반복 문서 양식 | `90_templates` |
| 완료되거나 비활성화된 자료 | `99_archive` |

## 원본 관리 원칙

- Git에는 Markdown 문서, 결정의 이유, 체크리스트, 템플릿과 외부 원본 링크를 저장합니다.
- 계속 갱신되는 표는 Google Sheets를 원본으로 유지합니다.
- 협업용 완성 문서는 Google Docs 또는 Notion을 원본으로 유지합니다.
- 디자인 원본은 Figma, 앱 코드는 별도 앱 저장소에 유지합니다.
- 계약서, 세금계산서, 증빙 원본과 민감정보는 Git에 저장하지 않습니다.
- 하나의 문서는 한 곳에만 두고 다른 영역에서는 링크로 참조합니다.

## 평소 Git 사용

```bash
git pull
git status
git add 수정한-파일
git commit -m "영역: 변경 내용"
git push
```

일상적인 문서 변경은 `main`에 직접 커밋합니다. 큰 구조 변경이나 위험한
실험에만 별도 브랜치를 사용합니다.
```

- [ ] **Step 4: Create `AGENTS.md`**

Use this exact content:

```markdown
# Founder OS 작업 규칙

## 작업 시작

1. 요청과 관련된 업무 영역의 `README.md`를 확인한다.
2. `80_resources/external-sources.md`에서 관련 외부 원본을 찾는다.
3. 최신 확인 규칙이 있는 Google Sheets·Docs·Notion 자료는 연결된 원본을 직접 읽는다.
4. 외부 원본에 접근할 수 없으면 오래된 로컬 자료를 최신 자료처럼 사용하지 않고 사용자에게 알린다.

## 파일 배치

- 새 자료의 위치가 불분명하면 `00_inbox`에 둔다.
- 하나의 원본 문서만 유지하고 다른 업무 영역에서는 상대 링크로 참조한다.
- Figma 화면 원본은 복제하지 않고 `30_app/ideation`에 직접 노드 링크와 분석을 기록한다.
- 앱 소스코드는 이 저장소에 복제하지 않고 별도 저장소를 읽기 전용 참고자료로 사용한다.

## 보안

- 비밀번호, API 키, 토큰, 인증서, 실제 `.env` 값을 기록하지 않는다.
- 주민등록번호, 계좌정보, 계약서, 세금계산서, 증빙 원본을 Git에 추가하지 않는다.
- 민감 원본은 접근이 제한된 Drive에 두고 여기에는 위치와 처리 상태만 기록한다.
- 커밋 전 `git status`로 모든 대상 파일을 확인한다.

## 외부 문서 변경

- 외부 원본은 사용자가 명시적으로 수정을 요청하지 않는 한 읽기 전용으로 취급한다.
- 외부 문서를 수정한 경우 대상 파일, 시트 또는 페이지와 변경 범위를 결과에 명시한다.
```

- [ ] **Step 5: Validate and commit**

Run:

```bash
git diff --check
git status --short
```

Expected: only `.gitignore`, `README.md`, and `AGENTS.md` are changed; `.DS_Store` is absent.

Commit:

```bash
git add .gitignore README.md AGENTS.md
git commit -m "chore: Founder OS 기본 운영 규칙 추가"
```

---

### Task 2: Create intake, dashboard, and domain navigation

**Files:**

- Create: `00_inbox/README.md`
- Create: `01_dashboard/README.md`
- Create: `01_dashboard/now.md`
- Create: `01_dashboard/backlog.md`
- Create: `01_dashboard/recurring.md`
- Create: domain `README.md` files listed in the file map
- Create: planned `.gitkeep` files listed in the file map

**Interfaces:**

- Consumes: storage decision rules from `README.md`
- Produces: navigable domain boundaries and empty approved locations for later tasks

- [ ] **Step 1: Create inbox and dashboard files**

`00_inbox/README.md`:

```markdown
# Inbox

위치를 바로 판단하기 어려운 메모와 아이디어를 임시로 둡니다.
매주 한 번 검토해 적절한 업무 영역으로 옮기거나 삭제합니다.
완성 문서를 장기간 보관하는 장소로 사용하지 않습니다.
```

`01_dashboard/README.md`:

```markdown
# Dashboard

- `now.md`: 현재 집중하는 일
- `backlog.md`: 아직 시작하지 않은 일
- `recurring.md`: 반복 업무와 점검 주기

세부 실행 자료는 해당 업무 영역에 두고 여기에서는 링크로 연결합니다.
```

`01_dashboard/now.md`:

```markdown
# Now

## 이번 주 핵심 결과

## 진행 중

## 막힌 일

## 관련 문서
```

`01_dashboard/backlog.md`:

```markdown
# Backlog

## 다음에 검토

## 언젠가 할 일

## 하지 않기로 한 일
```

`01_dashboard/recurring.md`:

```markdown
# Recurring

| 주기 | 업무 | 관련 문서 | 마지막 확인 | 다음 확인 |
| --- | --- | --- | --- | --- |
```

- [ ] **Step 2: Create domain guides**

`10_meetings/README.md`:

```markdown
# Meetings

회의록을 연도별로 보관합니다. 파일명은 `YYYY-MM-DD-주제.md` 형식을 사용하고,
논의 내용뿐 아니라 결정사항, 담당자, 기한과 후속 작업을 기록합니다.
```

`20_brand_marketing/README.md`:

```markdown
# Brand & Marketing

- `strategy`: 브랜드 원칙과 포지셔닝
- `owned`: 홈페이지, YouTube, Instagram 등 자체 채널
- `paid`: Meta, Google 등 유료 광고

성과 수치의 원본이 Google Sheets에 있으면 `80_resources/external-sources.md`에 등록합니다.
```

`30_app/README.md`:

```markdown
# App

- `ideation`: Figma 아이디어의 링크, 분석과 재활용 상태
- `planning`: 채택된 제품 기획
- `design`: 디자인 원칙과 검토 의견
- `operations`: 앱 운영 절차
- `reviews`: 사용자 리뷰 분석과 대응
- `qa`: 테스트 계획과 결과
- `web`: 약관, 공지 등 웹 페이지 기획
- `references`: 별도 앱 저장소를 포함한 참고자료

실제 앱 코드는 이 저장소에 넣지 않습니다.
```

`40_grants/README.md`:

```markdown
# Grants

- `opportunities`: 검토 중인 지원사업
- `templates`: 공통 지원서 작성 자료
- `programs`: 지원사업별 계획, 체크리스트와 결과

지원사업 목록이 Google Sheets에 있으면 작업 전에 최신 원본을 확인합니다.
```

`50_business_admin/README.md`:

```markdown
# Business Admin

- `evidence`: 증빙 업무의 목록, 기한, 처리 상태와 Drive 원본 위치
- `tax_finance`: 세무·재무 업무의 메모, 의사결정과 원본 위치

계약서, 세금계산서, 증빙 원본, 계좌정보와 개인정보는 Git에 저장하지 않습니다.
```

`80_resources/README.md`:

```markdown
# Resources

외부 원본과 별도 저장소의 색인입니다.

- `external-sources.md`: Sheets, Docs, Drive, Notion, Figma
- `app-repositories.md`: 앱 소스 저장소와 참고 방법

링크만 등록하지 말고 용도와 최신 확인 규칙을 함께 기록합니다.
```

`90_templates/README.md`:

```markdown
# Templates

회의록, 앱 기획, QA, 지원사업, 아이디어 분석처럼 반복되는 문서의 시작 양식입니다.
템플릿을 복사한 뒤 각 업무 영역에서 실제 문서를 작성합니다.
```

`99_archive/README.md`:

```markdown
# Archive

완료됐거나 더 이상 활성 상태가 아닌 자료를 보관합니다.
재활용 가능한 앱 아이디어는 이곳이 아니라 `30_app/ideation`에 유지합니다.
```

- [ ] **Step 3: Create planned empty directories**

Create these exact files with a single blank line as their content:

```text
10_meetings/2026/.gitkeep
20_brand_marketing/strategy/.gitkeep
20_brand_marketing/owned/website/.gitkeep
20_brand_marketing/owned/youtube/.gitkeep
20_brand_marketing/owned/instagram/.gitkeep
20_brand_marketing/paid/meta/.gitkeep
20_brand_marketing/paid/google/.gitkeep
30_app/planning/.gitkeep
30_app/design/.gitkeep
30_app/operations/.gitkeep
30_app/reviews/.gitkeep
30_app/qa/.gitkeep
30_app/web/.gitkeep
30_app/references/.gitkeep
40_grants/opportunities/.gitkeep
40_grants/templates/.gitkeep
40_grants/programs/.gitkeep
50_business_admin/evidence/.gitkeep
50_business_admin/tax_finance/.gitkeep
```

- [ ] **Step 4: Validate navigation**

Run:

```bash
find 00_inbox 01_dashboard 10_meetings 20_brand_marketing 30_app 40_grants 50_business_admin 80_resources 90_templates 99_archive -maxdepth 4 -print
```

Expected: every directory in the approved tree is present.

- [ ] **Step 5: Commit**

Stage only the directories created in this task:

```bash
git add 00_inbox 01_dashboard 10_meetings 20_brand_marketing 30_app 40_grants 50_business_admin 80_resources/README.md 90_templates/README.md 99_archive
git commit -m "chore: 업무 영역별 기본 폴더 구성"
```

---

### Task 3: Add live external-source indexes

**Files:**

- Create: `80_resources/external-sources.md`
- Create: `80_resources/app-repositories.md`

**Interfaces:**

- Consumes: live-read rules in `AGENTS.md`
- Produces: a stable registry that later work uses to resolve current Google and Figma sources

- [ ] **Step 1: Create the external-source registry**

Use this exact content:

```markdown
# External Sources

이 파일은 외부 원본의 위치와 확인 규칙을 관리합니다. 실제 데이터는 복제하지 않습니다.

| 이름 | 유형 | 관련 영역 | 원본 링크 | 용도 | 최신 확인 규칙 | 민감도 |
| --- | --- | --- | --- | --- | --- | --- |

## 등록 규칙

- Google Sheets는 관련 업무를 시작할 때 최신 범위를 직접 조회합니다.
- Google Docs와 Notion은 완성 문서 또는 협업 문서의 원본으로 사용합니다.
- Drive는 PDF, 이미지, 계약서와 증빙 원본을 보관합니다.
- Figma는 디자인과 아이디에이션 화면의 원본으로 사용합니다.
- 접근할 수 없는 원본은 추측하거나 오래된 내보내기 파일로 대체하지 않습니다.
- 비밀번호, API 키, 인증 토큰은 이 파일에 기록하지 않습니다.
```

- [ ] **Step 2: Create the app-repository registry**

Use this exact content:

```markdown
# App Repositories

실제 앱 소스코드는 별도 저장소에서 관리합니다. 이 저장소에서는 기획과 검토를
위해 읽기 전용으로 참고합니다.

| 저장소 | URL | 기본 브랜치 | 로컬 경로 | 참고 목적 |
| --- | --- | --- | --- | --- |

## 사용 규칙

- 앱 코드를 수정하거나 커밋하거나 push하지 않고 읽기 전용으로 참고합니다.
- 기획 문서에 코드 근거가 필요하면 저장소, 파일 경로와 커밋을 함께 기록합니다.
- 최신 코드 확인이 필요하면 별도 저장소에서 `git pull`한 뒤 참고합니다.
```

- [ ] **Step 3: Validate and commit**

Run:

```bash
git diff --check
rg -n "원본|최신|읽기 전용" 80_resources
```

Expected: both files explain source-of-truth and freshness behavior.

Commit:

```bash
git add 80_resources/external-sources.md 80_resources/app-repositories.md
git commit -m "docs: 외부 원본과 앱 저장소 색인 추가"
```

---

### Task 4: Build the reusable Figma ideation library

**Files:**

- Create: `30_app/ideation/README.md`
- Create: `30_app/ideation/catalog.md`
- Create: `30_app/ideation/ideas/README.md`
- Create: `90_templates/app-idea.md`

**Interfaces:**

- Consumes: Figma-as-original rule from `AGENTS.md`
- Produces: searchable idea metadata and a consistent analysis template

- [ ] **Step 1: Create the ideation guide**

Use this exact content:

```markdown
# App Ideation

정리되지 않은 Figma 화면을 검색하고 재활용할 수 있는 아이디어 라이브러리입니다.
화면 원본은 Figma에 유지하고 여기에는 직접 노드 링크와 분석만 기록합니다.

## 수집 순서

1. Figma 화면이 해결하려던 사용자 문제를 파악합니다.
2. `catalog.md`에 아이디어와 직접 노드 링크를 등록합니다.
3. 분석이 길면 `ideas`에 `YYYY-MM-DD-아이디어명.md` 문서를 만듭니다.
4. 실제 기획에 채택되면 아이디어를 이동하지 않고 기획 문서에서 링크합니다.

## 상태

- `검토 전`: 링크만 수집한 상태
- `보류`: 지금 사용하지 않지만 재활용 가능
- `기획 반영`: 실제 제품 기획에 사용
- `폐기`: 사용하지 않으며 판단 근거만 보존
```

- [ ] **Step 2: Create the catalog**

Use this exact content:

```markdown
# Idea Catalog

| 아이디어 | Figma 노드 | 수집일 | 관련 영역 | 상태 | 태그 | 분석 문서 |
| --- | --- | --- | --- | --- | --- | --- |

## 작성 규칙

- Figma 파일 첫 화면이 아니라 해당 아이디어의 직접 노드 링크를 사용합니다.
- 상태는 `검토 전`, `보류`, `기획 반영`, `폐기` 중 하나만 사용합니다.
- 태그는 사용자 문제나 흐름을 나타내는 단어를 우선합니다.
- 한 아이디어에 대한 상세 분석 문서는 하나만 유지합니다.
```

- [ ] **Step 3: Create the idea-document guide**

Use this exact content in `30_app/ideation/ideas/README.md`:

```markdown
# Idea Analyses

상세 분석이 필요한 아이디어만 별도 문서로 작성합니다.
파일명은 `YYYY-MM-DD-아이디어명.md` 형식을 사용하고
`90_templates/app-idea.md`를 복사해 시작합니다.
```

- [ ] **Step 4: Create the app-idea template**

Use this exact content:

```markdown
# 아이디어 제목

- Figma 노드:
- 수집일:
- 관련 영역:
- 상태: 검토 전
- 태그:

## 해결하려던 문제

## 핵심 아이디어

## 활용 가능한 요소

## 적용하지 않은 이유

## 다시 검토할 조건

## 실제 기획 연결
```

- [ ] **Step 5: Validate and commit**

Run:

```bash
rg -n "Figma 노드|검토 전|기획 반영|폐기" 30_app/ideation 90_templates/app-idea.md
git diff --check
```

Expected: direct-node linking and all four statuses appear in the library.

Commit:

```bash
git add 30_app/ideation 90_templates/app-idea.md
git commit -m "app: Figma 아이디어 라이브러리 추가"
```

---

### Task 5: Add reusable working templates

**Files:**

- Create: `90_templates/meeting-note.md`
- Create: `90_templates/app-plan.md`
- Create: `90_templates/qa-checklist.md`
- Create: `90_templates/grant-program.md`

**Interfaces:**

- Consumes: domain boundaries from Task 2
- Produces: consistent starting points for recurring work

- [ ] **Step 1: Create the meeting template**

```markdown
# 회의 제목

- 일시:
- 참석자:
- 목적:

## 논의 내용

## 결정사항

## 후속 작업

| 작업 | 담당자 | 기한 | 상태 |
| --- | --- | --- | --- |

## 관련 문서
```

- [ ] **Step 2: Create the app-plan template**

```markdown
# 기능 또는 개선안

## 배경

## 해결할 사용자 문제

## 목표

## 범위

### 포함

### 포함하지 않음

## 사용자 흐름

## 요구사항

## 성공 기준

## 관련 Figma·코드·데이터 원본

## 결정과 근거
```

- [ ] **Step 3: Create the QA template**

```markdown
# QA 대상

- 빌드 또는 버전:
- 테스트 환경:
- 테스트 일자:

## 출시 차단 기준

## 점검 항목

| 영역 | 시나리오 | 기대 결과 | 실제 결과 | 상태 | 증거 |
| --- | --- | --- | --- | --- | --- |

## 발견된 문제

## 출시 판단
```

- [ ] **Step 4: Create the grant template**

```markdown
# 지원사업명

- 주관기관:
- 공고 링크:
- 마감일:
- 상태:

## 지원 목적과 실제 사업 목표의 차이

## 지원 조건

## 제출 자료

| 자료 | 원본 위치 | 담당 | 기한 | 상태 |
| --- | --- | --- | --- | --- |

## 작성 전략

## 결정사항

## 결과와 회고
```

- [ ] **Step 5: Validate and commit**

Run:

```bash
find 90_templates -maxdepth 1 -type f -print
git diff --check
```

Expected: README plus five templates, including `app-idea.md` from Task 4.

Commit:

```bash
git add 90_templates/meeting-note.md 90_templates/app-plan.md 90_templates/qa-checklist.md 90_templates/grant-program.md
git commit -m "template: 반복 업무 문서 양식 추가"
```

---

### Task 6: Verify the complete workspace and publish it

**Files:**

- Verify: every file and directory in the file map
- Modify only if verification finds a mismatch

**Interfaces:**

- Consumes: deliverables from Tasks 1–5
- Produces: a clean, published `main` branch matching the approved design

- [ ] **Step 1: Verify required files**

Run:

```bash
test -f README.md
test -f AGENTS.md
test -f .gitignore
test -f 80_resources/external-sources.md
test -f 80_resources/app-repositories.md
test -f 30_app/ideation/catalog.md
test -f 90_templates/app-idea.md
test -f 90_templates/meeting-note.md
test -f 90_templates/app-plan.md
test -f 90_templates/qa-checklist.md
test -f 90_templates/grant-program.md
```

Expected: every command exits with status 0.

- [ ] **Step 2: Verify safety and formatting**

Run:

```bash
git check-ignore -q .DS_Store
git check-ignore -q .env
git diff --check
git status --short
```

Expected: ignore checks exit with status 0, `git diff --check` prints nothing, and status contains no unexpected or sensitive files.

- [ ] **Step 3: Verify recent commit boundaries**

Run:

```bash
git log --oneline -7
```

Expected: separate commits exist for repository controls, folder structure, external indexes, Figma ideation, and templates.

- [ ] **Step 4: Push**

Run:

```bash
git push origin main
```

Expected: Git reports `main -> main` or `Everything up-to-date`.
