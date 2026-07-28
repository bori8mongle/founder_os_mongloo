# 속마음 실사용·장문 시나리오 확장 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 기존 240개 단문 중심 시나리오를 유지하면서, 실제 Z세대 생활 기록·분할 기록·30분/1시간 세션·20,000자 경계를 검증하는 추가 시나리오 60개를 근거 기반으로 제작하고 Prompt Lab 및 서버 E2E 검증에 연결한다.

**Architecture:** 문장 품질을 보는 `모델 평가 시나리오`와 기록 선택·80자·20,000자·절단을 보는 `서버 입력 fixture`를 분리한다. AI Hub 원문과 실제 사용자 원문은 Git에 넣지 않고 제한된 원본 보관소에서만 다루며, Git에는 비식별화한 사건 골격·변형 이력·검수 완료 시나리오만 저장한다.

**Tech Stack:** Markdown, CSV, JSON, Node.js Prompt Lab, Node test runner, Google Sheets 결과 시트, AI Hub 공개 데이터

**Plan Status:** planned — 시나리오 제작을 시작하기 전 기준 문서

## Global Constraints

- 이번 계획 문서는 실행 중 컨텍스트 보존용이며, 완료 후 제품 기준 문서로 사용하지 않는다.
- 현재 `evaluation_scenario_bank.csv` 240개는 삭제하거나 의미를 바꾸지 않는다.
- AI Hub·실제 사용자 원문·개인정보·민감정보·다운로드 파일은 Git에 추가하지 않는다.
- AI Hub 원문은 데이터별 이용정책과 출처 표시 기준을 확인한 뒤 사용한다.
- 원문을 그대로 복제하지 않고 사건 구조·말투 특성·기록 흐름을 추출해 비식별 합성 시나리오로 전환한다.
- 실제 사용자 기록을 사용하려면 별도 고지·명시적 동의·보유 기간·접근 권한이 먼저 확정되어야 한다.
- 20,000자는 목표 분량이 아니라 절대 입력 상한이다.
- 30분·1시간 사례는 억지로 글자 수를 채운 단일 장문이 아니라 시간순 기록 묶음으로 만든다.
- 장문 결과의 정답은 완성 문장 하나가 아니라 필수 사실·장면 관계·금지 추론으로 정의한다.
- `draft → reviewed → approved` 상태를 유지한다. 입력과 기대값 검토 전에는 모델 품질 판정에 사용하지 않는다.
- `pebbling-expo/`는 이 저장소에서 읽기 전용 기준 코드로 취급한다. 코드 반영은 개발자 저장소의 별도 커밋으로 수행한다.
- `safety_only | SAFETY_ONLY`는 최신 v0.5.0 정책의 신규 시나리오에 사용하지 않는다.

---

## 1. 완료 정의

다음 조건을 모두 만족하면 이 계획을 완료한 것으로 본다.

- 추가 시나리오 60개의 입력·기대 행동·출처 변형 방식이 작성되어 있다.
- 모델 평가용 40개가 기존 CSV 계약에 맞게 추가되어 유효성 검사를 통과한다.
- 서버 입력 선택·경계 fixture 20개가 자동 테스트에서 통과한다.
- 60개 모두 개인정보·출처·현실성 검수를 통과해 `reviewed`가 된다.
- 20,000자 사례에서 포함 기록 ID·제외 기록 ID·부분 절단·최종 글자 수를 확인할 수 있다.
- 30분·1시간 사례가 시간순 기록 묶음으로 실행된다.
- Prompt Lab에서 `실사용`과 `입력 한계` 묶음을 구분해 선택할 수 있다.
- 실행 결과에 입력 글자 수·토큰·비용·지연·선택 장면을 검토할 수 있다.
- 실제 AI 결과까지 승인된 대표 사례만 `approved` 골든으로 승격된다.
- 실제 사용자 원문과 AI Hub 원본이 Git 및 Google Sheets 결과에 포함되지 않았음을 확인한다.

## 2. 파일 구조와 책임

### Founder OS 작업 자료

- Create: `30_app/qa/inner-thought-realistic-scenarios/README.md`
  - 작업 범위, 원본 금지 규칙, 상태 흐름과 파일 지도를 기록한다.
- Create: `30_app/qa/inner-thought-realistic-scenarios/source-registry.md`
  - AI Hub 데이터명·공식 URL·버전·접근일·사용 목적·변형 원칙·이용 제한을 기록한다.
- Create: `30_app/qa/inner-thought-realistic-scenarios/coverage-matrix.csv`
  - 60개 시나리오의 연령·주제·길이·세션·흐름·감정·출처·검수 상태를 관리한다.
- Create: `30_app/qa/inner-thought-realistic-scenarios/model-scenario-drafts.csv`
  - Prompt Lab에 합치기 전 40개 모델 평가 시나리오의 작업본이다.
- Create: `30_app/qa/inner-thought-realistic-scenarios/server-input-fixtures.json`
  - 80자·시간순 기록·20,000자 선택과 절단을 검증하는 20개 구조 fixture다.
- Create: `30_app/qa/inner-thought-realistic-scenarios/review-log.md`
  - 현실성·제품 계약·개인정보 검수 결과와 수정 이력을 기록한다.
- Modify: `80_resources/external-sources.md`
  - 실제 사용하기로 확정한 AI Hub 원본만 등록한다.

### 개발 인계 원본

- Modify: `developer-handoff/validation/evaluation_scenario_bank.csv`
  - 검수 완료된 모델 평가 시나리오 40개만 `G241`부터 추가한다.
- Modify: `developer-handoff/validation/02-AI-평가-실행-가이드.md`
  - 실사용·장문·입력 한계 묶음의 실행 순서와 판정 방법을 추가한다.
- Modify: `developer-handoff/validation/03-시나리오뱅크-구조검증-결과.md`
  - 추가 후 총개수·분포·중복·유효성 검사 결과를 갱신한다.
- Modify: `developer-handoff/README.md`
  - 모든 검수가 끝난 시점에만 시나리오 총개수와 필수 실행 묶음을 갱신한다.

### Pebbling 앱 저장소에서 개발자가 반영할 파일

- Modify: `pebbling-expo/tools/prompt-lab/evals/evaluation_scenario_bank.csv`
- Modify: `pebbling-expo/tools/prompt-lab/evals/case.schema.json`
- Modify: `pebbling-expo/tools/prompt-lab/server/case-contract.mjs`
- Modify: `pebbling-expo/tools/prompt-lab/public/case-options.mjs`
- Modify: `pebbling-expo/tools/prompt-lab/tests/case-files.test.mjs`
- Create: `pebbling-expo/tools/prompt-lab/evals/server-input-fixtures.json`
- Create: `pebbling-expo/tools/prompt-lab/tests/input-selection-fixtures.test.mjs`
- Modify: `pebbling-expo/docs/dev/prompt-lab.md`

## 3. 시나리오 포트폴리오

### 3.1 모델 평가용 40개

| 묶음 | 개수 | 길이·형태 | 핵심 검증 |
|---|---:|---|---|
| 현실적 짧은 기록 | 8 | 80~150자 | 80자를 넘긴 일반 사용 입력의 자연스러운 반응 |
| 현실적 중간 기록 | 8 | 151~500자 | 구체적 사건에서 핵심을 고르는지 |
| 현실적 장문 기록 | 4 | 501~1,500자 | 장문을 요약문처럼 되풀이하지 않는지 |
| 같은 사건 분할 | 6 | 3~8개 기록 | 설명·정정·결과를 같은 장면으로 묶는지 |
| 무관한 사건 분할 | 5 | 3~8개 기록 | 서로 다른 일을 인과나 감정 변화로 합치지 않는지 |
| 혼합 긍정·부정 | 4 | 2~6개 기록 | 부정 장면만 기계적으로 고르지 않는지 |
| 30분 세션 | 3 | 8~15개 기록 | 반복·주제 전환·후속 결과를 선별하는지 |
| 1시간 세션 | 2 | 15~30개 기록 | 열람가치가 있는 소수 장면에 집중하는지 |
| **합계** | **40** |  |  |

### 3.2 서버 입력·경계 fixture 20개

| 묶음 | 개수 | 핵심 검증 |
|---|---:|---|
| 79·80·81자 | 3 | 공백·문장부호·Unicode를 포함한 생성 자격 |
| 여러 기록 합산 80자 | 3 | 새 자유 텍스트만 합산하고 느낌 라벨·보조 맥락은 제외 |
| 30분 시간순 세션 | 3 | 다수 기록의 ID·시각·순서 유지 |
| 1시간 시간순 세션 | 3 | 생성 중 추가·수정·삭제 스냅샷 |
| 19,999·20,000·20,001자 | 3 | 상한 경계 |
| 단일 최신 기록 20,000자 초과 | 1 | 뒤쪽 원문만 포함하고 앞부분 생략 표시 |
| 여러 기록 합계 20,000자 초과 | 2 | 가장 오래된 포함 기록의 부분 절단 |
| 새 기록과 보조 맥락 경쟁 | 1 | 새 기록을 먼저 채우고 남는 범위에만 보조 맥락 포함 |
| 상한 밖 중요 장면 | 1 | 제외 기록을 이번 결과에 사용하지 않고 다음 속마음으로 이월하지 않음 |
| **합계** | **20** |  |

### 3.3 교차 분포 목표

- 연령: `17–19`, `20–22`, `23–25`, `26–29`가 각각 최소 10개
- 주제: 공부·평가, 취업·직장, 관계·가족, 돈·주거, 건강·몸, 일상·휴식, 취미·즐거움이 각각 최소 6개
- 톤: 긍정 20%, 부정 35%, 혼합 30%, 중립·불명확 15% 내외
- 기록 흐름: 단일·복수·정정·변화 사례를 모두 포함한다. 기존 속마음 뒤 새 기록은
  모델 평가 CSV의 현재 계약에 없는 항목이므로 서버 fixture에서 최소 3개를 별도 검증한다.
- 안전 사례: 전체 60개 중 6~8개. 일반 결과와 안전 도움의 과잉·누락을 모두 포함
- 말투 특성: 오타·줄임말·이모지·반복·주어 생략을 각각 최소 5개에 포함하되 모든 사례에 강제하지 않음

## 4. 출처 전략

### 우선 사용할 AI Hub 원본

1. `감정이 태깅된 자유대화 (성인)`
   - 20대 자유대화의 말투·주제 전환·감정 태그에 사용
2. `감정이 태깅된 자유대화 (청소년)`
   - 17~19세의 교육·취미·게임·고민 Q&A·생활 주제에 사용
3. `주제별 텍스트 일상 대화 데이터`
   - 감정 중심이 아닌 음식·교통·주거·교육·가족 등 일상 주제에 사용
4. `한국어 감정 정보가 포함된 연속적 대화`
   - 멀티턴·정정·주제 재진입·긴 흐름에 사용
5. `감성 대화 말뭉치`
   - 세부 감정 누락을 보완하되 실제 일기 말투의 근거로 단독 사용하지 않음
6. `웰니스 대화 스크립트`
   - 안전·과잉 상담·의존 표현의 반례에만 제한적으로 사용

### 출처별 사용 비중

| 출처 | 목표 시나리오 수 | 사용 방식 |
|---|---:|---|
| 성인 자유대화 | 18 | 20대 화자의 발화 흐름에서 사건 골격과 말투 특성 추출 |
| 청소년 자유대화 | 10 | 17~19세 생활 주제와 표현 특성 추출 |
| 주제별 일상 대화 | 10 | 중립·긍정 일상 주제 보완 |
| 감정 연속 대화 | 8 | 복수 기록·정정·주제 전환 구조 보완 |
| 감성 대화 말뭉치 | 8 | 세부 감정·혼합 감정 보완 |
| 웰니스 스크립트 | 6 | 안전 경계와 잘못된 응답 반례 |
| **합계** | **60** | 한 시나리오가 주출처 하나만 갖도록 관리 |

### 원본 보관 원칙

- AI Hub 압축 파일·음성·전사 원문은 Git 밖의 접근 제한 저장소에 둔다.
- `source-registry.md`에는 데이터명·공식 URL·버전·다운로드일·이용정책 확인일만 기록한다.
- 작업 시트에는 원문 대신 `source_record_id`, 사건 골격, 변형 수준을 기록한다.
- 외부 개발자에게는 원본을 전달하지 않고 검수된 합성 시나리오만 전달한다.
- AI Hub 데이터로부터 만든 결과라는 출처 표시는 AI Hub 최신 이용정책을 따른다.

## 5. 변형 방법

각 시나리오는 아래 다섯 단계를 거친다.

1. **원본 단위 선택**
   - 동일 화자·동일 세션·시간순 관계가 확인되는 범위만 선택한다.
2. **사건 골격 추출**
   - `누가/어디서/무슨 일/사용자가 직접 말한 느낌/후속 변화`만 구조화한다.
3. **비식별화**
   - 실명·학교·회사·지역·연락처·계정·희귀 사건을 일반화하거나 교체한다.
4. **페블링 기록화**
   - 대화 상대의 말은 제거하고 사용자 1인의 짧은 기록 여러 개로 재구성한다.
5. **기대값 작성**
   - 반드시 알아차릴 사실, 같은 장면 ID, 분리할 장면 ID, 금지 추론, 질문·안전 조건을 기록한다.

### 허용되는 변형

- 등장 인물 관계를 `친구`, `팀원`, `가족`처럼 일반화
- 구체 기관·지역·금액을 흔한 범주와 규모로 변경
- 같은 화자의 발화를 시간순 기록 조각으로 분할
- 직접 식별정보와 희귀 사건 제거
- 길이·오타·이모지·줄임말을 목표 분포에 맞게 조정
- 동일 사건의 정정·결과를 뒤 기록으로 재배치

### 금지되는 변형

- 원문에 없는 자해·범죄·질병·트라우마를 추가
- 실제 사람의 희귀 경험을 알아볼 수 있게 유지
- 여러 사람의 사건을 섞어 한 사람의 실제 경험처럼 제시
- 감성 말뭉치의 챗봇 답변을 좋은 속마음 예시로 복사
- 20,000자를 맞추기 위해 무관한 문장을 무작위 반복
- 모델 출력 품질을 쉽게 통과시키려고 기대값을 결과에 맞춰 사후 변경

## 6. 시나리오 데이터 계약

### 모델 평가 CSV

기존 열 계약을 유지하고 다음 값을 사용한다.

- `scenario_id`: `G241`부터 연속 부여
- `entries`: `[새 기록 입력]`, `HH:MM`, 읽기 전용 보조 맥락 표기를 기존 파서 형식으로 작성
- `source_type`: `AIHUB_TRANSFORMED`, `PEBBLING_FEEDBACK_SYNTHETIC`, `PRODUCT_EDGE_SYNTHETIC` 중 하나
- `source_id`: `데이터셋 번호/비식별 원본 ID/변형 버전` 형식
- `status`: 처음에는 `draft`; 사람 검수 후 `reviewed`
- `test_tier`: 모델 품질 사례는 `realistic`, 장문 품질 사례는 `stress`

### 서버 fixture JSON

각 fixture는 다음 필드를 가진다.

```json
{
  "fixture_id": "S001",
  "context": "여러 새 기록의 합이 정확히 80자인 경우",
  "request_at": "2026-07-28T22:00:00+09:00",
  "timezone": "Asia/Seoul",
  "records": [
    {
      "id": "r1",
      "created_at": "2026-07-28T21:10:00+09:00",
      "text": "사용자가 쓴 자유 텍스트",
      "already_analyzed": false,
      "deleted": false
    }
  ],
  "expected": {
    "should_call_ai": true,
    "included_record_ids": ["r1"],
    "excluded_record_ids": [],
    "prefix_truncated_record_id": null,
    "new_text_char_count": 80,
    "model_input_char_count": 80
  }
}
```

fixture의 정답은 서버 코드의 실제 결과와 직접 비교한다. 모델 출력 문장 품질은 이 fixture에서 합격 조건으로 사용하지 않는다.

## 7. 실행 작업

### Task 1: 작업 공간과 출처 원장 생성

**Files:**
- Create: `30_app/qa/inner-thought-realistic-scenarios/README.md`
- Create: `30_app/qa/inner-thought-realistic-scenarios/source-registry.md`
- Modify: `80_resources/external-sources.md`

- [x] AI Hub 6개 후보의 공식 URL·데이터셋 번호·최신 버전·접근 조건을 기록한다.
- [ ] 각 데이터셋의 이용정책과 원본 재배포 제한을 확인한다.
- [x] 원본 파일의 접근 제한 저장 위치를 정하고, 실제 경로 대신 접근 방법만 기록한다.
- [x] AI Hub 원본·실제 사용자 원본을 Git에서 제외하는 규칙을 README에 기록한다.
- [x] `git status --short`로 원본 데이터가 추가되지 않았는지 확인한다.

**Acceptance:** 출처마다 `용도`, `사용하지 않을 용도`, `변형 원칙`, `최신 확인일`이 비어 있지 않다.

### Task 2: 커버리지 매트릭스 확정

**Files:**
- Create: `30_app/qa/inner-thought-realistic-scenarios/coverage-matrix.csv`

- [x] `R001`부터 `R060`까지 60행을 만든다.
- [x] 각 행에 연령·주제·톤·흐름·길이·세션 시간·주출처·모델/서버 구분을 입력한다.
- [x] 3.3의 최소 분포를 수식 또는 피벗으로 확인한다.
- [x] 중복되는 조합은 사건 주제나 기록 흐름을 바꿔 차이를 만든다.
- [x] 20,000자 사례를 일반 사용자 대표 사례가 아니라 `입력 한계`로 표시한다.

**Acceptance:** 모든 행에 주출처 하나와 1차 검증 목적 하나가 있고, 분포 목표의 빈 셀이 없다.

### Task 3: 사건 골격과 비식별 변형

**Files:**
- Modify: `30_app/qa/inner-thought-realistic-scenarios/coverage-matrix.csv`
- Create: `30_app/qa/inner-thought-realistic-scenarios/review-log.md`

- [ ] 출처별 목표 개수만큼 동일 화자·동일 세션 후보를 선별한다.
- [ ] 각 후보에서 사건·명시 감정·후속 변화·말투 특성만 추출한다.
- [ ] 실명·학교·회사·지역·연락처·희귀 사건을 제거한다.
- [ ] 원문과 변형본의 유사성이 높은 경우 표현과 세부 사건을 한 번 더 일반화한다.
- [ ] 웰니스 자료는 안전 입력과 잘못된 응답 반례에만 사용했는지 확인한다.

**Acceptance:** Git에 원문이 없고, 각 시나리오가 원본 없이도 독립적인 합성 입력으로 읽힌다.

### Task 4: 모델 평가 시나리오 40개 작성

**Files:**
- Create: `30_app/qa/inner-thought-realistic-scenarios/model-scenario-drafts.csv`

- [ ] 3.1의 개수대로 40개 입력을 작성한다.
- [ ] 분할 기록에는 각 기록의 시각과 순서를 넣는다.
- [ ] 장문에는 반복·주제 전환·정정 중 필요한 특성만 포함한다.
- [ ] 각 행에 `must_do`를 관찰 가능한 사실로 작성한다.
- [ ] 각 행에 `must_not_do`를 구체적인 잘못된 연결·추론으로 작성한다.
- [ ] 완성 답변 문구를 정답으로 강제하지 않는다.
- [ ] 신규 안전 시나리오는 `generate_with_safety_support` 또는 일반 생성 중 하나만 사용한다.

**Acceptance:** 40개 모두 입력과 기대 행동을 읽은 사람이 같은 사실관계 지도를 재구성할 수 있다.

### Task 5: 서버 입력 fixture 20개 작성

**Files:**
- Create: `30_app/qa/inner-thought-realistic-scenarios/server-input-fixtures.json`

- [ ] 79·80·81자의 실제 Unicode 글자 수를 계산해 고정한다.
- [ ] 30분·1시간 fixture에 기록 ID·작성 시각·분석 완료 여부를 명시한다.
- [ ] 20,000자 경계 fixture에 포함·제외·부분 절단 정답을 명시한다.
- [ ] 새 기록이 먼저 채워지고 보조 맥락이 나중에 들어가는 사례를 포함한다.
- [ ] 생성 중 수정·삭제된 기록이 요청 스냅샷에 미치는 기대를 명시한다.
- [ ] 상한 밖 기록이 다음 속마음에는 이월되지 않고 주간 편지 원본에는 유지되는 기대를 명시한다.

**Acceptance:** 모델을 호출하지 않고도 각 fixture의 합격·실패를 서버 결과만으로 판정할 수 있다.

### Task 6: 3단계 사람 검수

**Files:**
- Modify: `30_app/qa/inner-thought-realistic-scenarios/model-scenario-drafts.csv`
- Modify: `30_app/qa/inner-thought-realistic-scenarios/server-input-fixtures.json`
- Modify: `30_app/qa/inner-thought-realistic-scenarios/review-log.md`

- [ ] 1차 현실성 검수: 타깃이 실제로 쓸 법한 표현·길이·시간 흐름인지 판정한다.
- [ ] 2차 제품 계약 검수: 장면 묶기·선택·80자·20,000자·안전 기대가 현재 명세와 맞는지 판정한다.
- [ ] 3차 개인정보·출처 검수: 원문 복제·식별 가능성·출처 누락이 없는지 판정한다.
- [ ] 한 단계라도 실패한 행은 `draft`로 유지하고 실패 이유를 기록한다.
- [ ] 세 단계가 모두 통과한 모델 사례만 `reviewed`로 변경한다.

**Acceptance:** 60개 모두 검수자·검수일·판정·수정 내용을 추적할 수 있다.

### Task 7: 개발 인계 CSV로 승격

**Files:**
- Modify: `developer-handoff/validation/evaluation_scenario_bank.csv`
- Modify: `developer-handoff/validation/02-AI-평가-실행-가이드.md`
- Modify: `developer-handoff/validation/03-시나리오뱅크-구조검증-결과.md`

- [ ] 검수 완료된 모델 사례 40개를 `G241~G280`으로 추가한다.
- [ ] 기존 G001~G240의 입력·기대값이 바뀌지 않았는지 diff로 확인한다.
- [ ] `realistic`, `stress` 실행 묶음과 목적을 평가 실행 가이드에 추가한다.
- [ ] 단문 품질·실사용 품질·서버 한계 결과를 한 점수로 합치지 않는다고 명시한다.
- [ ] 구조검증 문서의 총개수·연령·주제·흐름·출처 분포를 갱신한다.

**Acceptance:** 기존 240개를 보존한 280개 CSV가 생성되고, 신규 40개만 diff에 나타난다.

### Task 8: Prompt Lab과 서버 테스트 반영

**Files:**
- Modify: `pebbling-expo/tools/prompt-lab/evals/evaluation_scenario_bank.csv`
- Modify: `pebbling-expo/tools/prompt-lab/evals/case.schema.json`
- Modify: `pebbling-expo/tools/prompt-lab/server/case-contract.mjs`
- Modify: `pebbling-expo/tools/prompt-lab/public/case-options.mjs`
- Modify: `pebbling-expo/tools/prompt-lab/tests/case-files.test.mjs`
- Create: `pebbling-expo/tools/prompt-lab/evals/server-input-fixtures.json`
- Create: `pebbling-expo/tools/prompt-lab/tests/input-selection-fixtures.test.mjs`
- Modify: `pebbling-expo/docs/dev/prompt-lab.md`

- [ ] `test_tier`에 `realistic`, `stress`를 추가한다.
- [ ] Prompt Lab 실행 화면에서 두 묶음을 별도로 선택할 수 있게 한다.
- [ ] 시나리오 총개수 검사를 240 고정값에서 280과 연속 ID 검사로 갱신한다.
- [ ] server fixture 20개를 입력 선택 함수에 연결한다.
- [ ] 각 fixture에서 호출 여부·글자 수·포함·제외·부분 절단을 단정적으로 검사한다.
- [ ] `npm run test:prompt-lab`을 실행한다.
- [ ] 실패가 있으면 기존 240개 회귀와 신규 40개 계약 실패를 구분해 수정한다.

**Acceptance:** Prompt Lab 전체 테스트가 통과하고 실사용·입력 한계 묶음을 독립 실행할 수 있다.

### Task 9: 실제 모델 실행과 골든 승격

**Files:**
- Modify: `30_app/qa/inner-thought-realistic-scenarios/review-log.md`
- Modify: `developer-handoff/validation/evaluation_scenario_bank.csv`

- [ ] `reviewed` 40개를 같은 모델·프롬프트·생성 설정으로 실행한다.
- [ ] 출력 계약·안전·근거 위반을 먼저 제외한다.
- [ ] 통과 결과에서 장면 선택·자연스러움·재열람 가치·캐릭터 균형을 검토한다.
- [ ] 30분·1시간 사례의 입력 토큰·출력 토큰·비용·지연을 별도로 기록한다.
- [ ] 20,000자 사례는 모델 문장보다 서버 입력 선택 정답을 먼저 판정한다.
- [ ] 서로 다른 성공 동작을 대표하는 8~12개만 `approved`로 승격한다.

**Acceptance:** `approved`가 단순히 점수가 높은 결과가 아니라 제품 책임자가 실제 문장까지 승인한 사례다.

### Task 10: 종료 정리

**Files:**
- Modify: `developer-handoff/README.md`
- Modify: `30_app/qa/inner-thought-realistic-scenarios/README.md`
- Move after approval: `docs/superpowers/plans/2026-07-28-inner-thought-realistic-scenario-expansion.md` → `99_archive/plans/`

- [ ] 최종 시나리오 수·골든 수·필수 실행 묶음을 developer-handoff README에 반영한다.
- [ ] 작업용 초안과 최종 인계 원본을 구분한다.
- [ ] 원본 데이터·임시 추출물·중간 결과가 Git에 없는지 확인한다.
- [ ] 완료 후 이 문서를 `status: completed`로 표시하고 사용자 확인 뒤 보관한다.

**Acceptance:** 이후 작업자는 이 계획 없이도 developer-handoff의 활성 문서만으로 실행할 수 있다.

## 8. 예상 일정

AI Hub 다운로드 승인이 완료되어 있다는 전제의 작업 시간이다.

| 구간 | 예상 시간 |
|---|---:|
| 출처 원장·커버리지 설계 | 2~3시간 |
| 원본 선별·사건 골격 추출 | 4~6시간 |
| 모델 시나리오 40개 작성 | 6~8시간 |
| 서버 fixture 20개 작성 | 4~5시간 |
| 현실성·계약·개인정보 검수 | 5~7시간 |
| Prompt Lab·자동 테스트 반영 | 4~6시간 |
| 실제 모델 실행·골든 검토 | 3~5시간 |
| **합계** | **28~40시간** |

빠른 1차 검증이 필요하면 60개 전체를 기다리지 않고 다음 16개를 먼저 완성한다.

- 80~150자 현실 기록 4개
- 같은 사건 분할 3개
- 무관한 사건 분할 3개
- 30분 세션 2개
- 1시간 세션 1개
- 79·80·81자 3개

이 16개는 구조를 검증하기 위한 선행 묶음이며 전체 커버리지 완료로 간주하지 않는다.

## 9. 추후 확장 전략

### 9.1 실패 기반 확장

- 프롬프트 변경 후 반복해서 실패한 실제 패턴만 새 회귀 사례로 추가한다.
- 기존 사례와 같은 실패를 검증하면 새로 만들지 않고 기존 사례를 강화한다.
- 새 규칙을 추가하기 전에 승인 예시 교체로 해결되는지 확인한다.

### 9.2 베타 사용 통계 기반 재균형

원문을 수집하지 않고 다음 집계만 먼저 본다.

- 기록당 글자 수 분포
- 하루 기록 횟수
- 한 세션의 기록 간격
- 80자 미만 비율
- 20,000자 도달 비율
- 단일·복수 기록 비율
- 생성 지연·입력 토큰·비용
- 안전 도움·재생성·실패 비율

분기마다 실제 분포와 테스트 분포를 비교해 과대표현된 장문·안전 사례와 부족한 일반 사례를 조정한다.

### 9.3 동의 기반 실제 기록 검증

- 별도 베타 동의 절차가 만들어진 뒤에만 진행한다.
- 8~12명에게 3일 동안 기록하게 하고 원문 검수 범위·보유 기간을 명확히 고지한다.
- 실제 원문은 제한된 환경에서만 확인하고 사건 골격과 비식별 합성본만 회귀셋에 남긴다.
- 참여 철회 시 원본과 연결 키를 삭제할 수 있어야 한다.

### 9.4 기능 확장에 따른 별도 세트

- 음성 기록: 전사 오류·반복·말 끊김·1시간 전사
- 가치·태도: 사용자가 확정한 수첩 정보가 직접 근거가 있을 때만 약하게 연결
- 주간 리포트: 7일 기록 범위·통계·AI 결과를 속마음 세트와 분리
- 영어권: 한국어 사례를 번역하지 않고 해당 문화권 원자료로 새로 제작
- 청소년 안전: 성인 사례의 나이만 바꾸지 않고 학교·가족·그루밍 문맥을 별도 설계

### 9.5 유지보수 주기

- 매 프롬프트 변경: `core + approved` 실행
- 모델 변경: `core + realistic + safety` 실행
- 입력 선택 코드 변경: 서버 fixture 20개 전부 실행
- 월 1회: 신규 실패·중복·사용 분포 검토
- 분기 1회: 출처 버전·이용정책·시나리오 분포 감사
- 6개월 이상 가치가 없는 중복 시나리오는 삭제하지 않고 `retired` 후보 목록으로 이동한 뒤 제품 책임자 확인 후 정리

## 10. 실행 중 의사결정 원칙

1. 실제성보다 테스트 편의성을 우선하지 않는다.
2. 길이 자체보다 장면 관계와 시간 흐름을 먼저 설계한다.
3. AI Hub 감정 라벨을 사용자의 확정 감정처럼 무조건 전달하지 않는다.
4. 모델이 알아야 할 정보와 평가자만 알아야 할 정답을 분리한다.
5. 장문 하나가 실패했다고 장문 전체에 새 프롬프트 규칙을 추가하지 않는다.
6. 서버 입력 선택 실패와 모델 문장 품질 실패를 별도 이슈로 등록한다.
7. 원문 보안·출처 조건이 불명확하면 해당 원본 사용을 중단하고 합성 사건 골격만 사용한다.
