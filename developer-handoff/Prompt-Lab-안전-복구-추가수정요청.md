---
type: Developer Change Request
title: Prompt Lab 안전·복구 추가 수정 요청
description: 현재 Prompt Lab 구현을 보존하면서 운영 안전·복구 변경을 검증할 수 있게 만드는 차이 중심 작업지시서.
tags: [pebbling, prompt-lab, safety, moderation, recovery, evaluation]
status: ready_for_development
audience: [developer, product_manager]
owner: 김세인
created_at: 2026-07-30
source_branch: feature/venting-tab
source_commit: 3a5cd9cdc06a85ed377a8b514b800c6fb9f65c0a
---

# Prompt Lab 안전·복구 추가 수정 요청

> **HOLD — 프롬프트·평가 내용**
>
> 새 운영 프롬프트, few-shot, 2·3차 생성 지시문, 문체·품질 평가 루브릭과 기본
> 모델은 재설계 후 별도 전달한다. 이번에는 프롬프트·예시·모델을 교체할 수 있는
> 설정 구조, `attempts[]`, 모의 응답 기반 최대 3회 흐름과 fallback 검증까지만
> 구현한다. 기존 v0.5.0의 의미 평가 기준을 새 기준으로 간주하지 않는다.

## 0. 이 문서의 범위

이 문서는 **현재 구현된 Prompt Lab에서 추가로 바꿀 것만** 설명한다.

다음 기능은 최근 개발 버전에 이미 있으므로 다시 만들지 않는다.

- Google Sheets `TestCases` 단일 원본
- 실행별 결과 시트와 `Runs`
- 테스트 케이스 ID·사용자 입력을 고정하고 실행별 답변을 오른쪽으로 쌓는 `AnswerHistory`
- `AnswerHistory` 상단의 실행 ID·프롬프트 버전·답변 모델
- 답변 모델과 평가 모델의 별도 선택
- 평가 AI 사용 여부 선택
- 답변·평가 비용 분리
- 사용자 입력의 여러 줄 편집과 동적 `test_tier`

Prompt Lab을 다시 설계하지 않는다. 운영 안전·복구 변경이 제대로 작동하는지
확인하기 위한 최소 변경만 한다.

운영 분기의 단일 기준은
[앱·서버 변경요청서의 전체 처리 플로우와 분기 정의표](속마음-운영-안전-복구-추가수정요청.md#01-전체-처리-플로우--구현-분기의-단일-기준)다.
Prompt Lab은 같은 모의 응답 순서에서 운영과 동일한 최종 상태, 차감 여부와
분석 기준 갱신 여부를 내야 한다. 이 문서에서 분기 정의를 별도로 만들지 않는다.

## 1. 기준

- 확인 코드: 로컬 `pebbling-expo`
- 브랜치: `feature/venting-tab`
- 확인 커밋: `3a5cd9c`
- 확인일: 2026-07-30
- 현재 자동 테스트:
  - `npm run test:prompt-lab`: 97개 통과
  - `npm run test:venting-observation`: 77개 통과

## 2. 현재 확인된 구현

| 영역 | 현재 구현 |
| --- | --- |
| 운영 계약 공유 | 운영 `contract.ts`를 불러 JSON schema·파서·말풍선 검사를 사용 |
| 생성 호출 | 최초 포함 최대 2회. 이번 변경에서 운영 계약과 함께 최대 3회로 확장 |
| Moderation | `omni-moderation-latest`, 카테고리·점수·응답 모델·응답 ID·지연 파싱 |
| 복구 이유 | `transient_transport`, `contract_violation`, `safety_product_violation` |
| 결과 시트 | 최종 출력·Moderation·AI 평가·호출 수·재생성 이유·비용 저장 |
| AnswerHistory | 동일 ID의 입력과 최종 답변을 실행별 열로 누적 |
| 평가 AI | 답변 생성과 분리된 오프라인 평가 |

## 3. 현재 부족한 점

| 항목 | 현재 | 필요한 변경 |
| --- | --- | --- |
| 1차·2차 후보 | 최종 `rawOutput`과 누적 사용량 중심 | 최대 3회 시도의 후보·계약·Moderation을 별도로 보존 |
| Moderation 확인 | 점수는 `moderation_json` 안에서 확인 가능 | 주요 카테고리·점수를 사람이 읽기 쉬운 열로 표시 |
| 재생성 이유 | 모든 출력 안전 문제를 `safety_product_violation`으로 묶음 | `output_moderation`과 실제 감지 카테고리를 기록 |
| 재생성 지시 | 모든 안전 문제에 같은 일반 지시 | 카테고리별 제한 지시 |
| 반복 실패 | flagged 최종 후보를 반환해 로직 실패로 표시 | 3차 최소 안전 모드까지 실패하면 개인화 결과로 반환하지 않고 고정 fallback으로 종료 |
| Moderation 연속 오류 | 실패 상태의 Moderation을 붙인 후보를 반환 | 후보 미노출·고정 fallback |
| 최종 처리 | 정상 후보 또는 생성 오류 중심 | 정상 / 고정 fallback / 안전 도움 포함 fallback 구분 |
| 운영과 일치 | 파서·계약은 공유하지만 복구 로직은 별도 구현 | 같은 테스트 벡터로 복구 결과까지 일치시킴 |

최대 3회는 서로 다른 의미 후보를 만드는 상한이다. 네트워크 타임아웃처럼 같은
요청을 다시 보내는 기술 재시도는 별도 로그로 남기고, 네 번째 의미 후보처럼
집계하거나 결과를 중복 저장하지 않는다.

## 4. P0-1 — 시도별 결과를 보존한다

하나의 테스트 케이스 결과에 다음 구조를 보존한다.

```text
attempts[]
  attempt_no
  generation_mode
  generation_response_id
  raw_output
  parsed_output
  contract_error_codes
  moderation
    response_id
    model
    flagged
    categories
    category_scores
    latency_ms
  input_tokens
  cached_input_tokens
  output_tokens
  generation_latency_ms
  regeneration_reason

final_disposition
  generated
  fixed_fallback
  fixed_fallback_with_safety_support
  server_no_call
```

- 앞 시도 후보를 다음 시도 후보가 덮어쓰지 않는다.
- 계약 파싱 전의 원본 응답도 시도 단위로 보존한다.
- 여러 Moderation 카테고리가 true면 모두 보존한다.
- 점수는 비교용이며 Prompt Lab이 임의 임계값으로 허용 여부를 뒤집지 않는다.

## 5. P0-2 — 운영과 같은 복구 결과를 만든다

### 계약 실패

구체적인 오류 코드와 함께 같은 사용자 원문으로 전체 재생성한다.

```text
invalid_json
invalid_schema
invalid_safety_combination
invalid_bubble_count
empty_bubble
bubble_too_long
total_too_long
pii_echo
```

새 코드 계약은 말풍선 1~20개·개별 300자·전체 3,000자이며 질문 수를 기계적
탈락 기준으로 사용하지 않는다. Prompt Lab은 운영 `contract.ts`를 그대로 공유한다.
과거 결과의 `too_many_questions` 값은 읽을 수 있지만 새 실행에서 만들지 않는다.

### 출력 Moderation flag

- `regeneration_reason = output_moderation`
- 감지된 카테고리를 재생성 지시에 전달
- 첫 후보나 일부 문장을 두 번째 입력에 넣지 않음
- 카테고리에 맞게 공격·위협·혐오·폭력·자해·성적·불법 묘사를 제한

### 최종 처리

| 상황 | Prompt Lab 최종 처리 |
| --- | --- |
| 1차 통과 | `generated` |
| 1차 실패, 2차 통과 | `generated`와 두 시도 비교 |
| 1·2차 실패, 3차 통과 | `generated`와 세 시도 비교 |
| 세 후보 모두 flag 또는 계약 실패 | `fixed_fallback` |
| 어느 유효 후보에서든 안전 경로가 확인됨 | 뒤 시도에서 없애거나 낮추지 않고 `fixed_fallback_with_safety_support`. 같은 경로는 가장 높은 priority 유지 |
| Moderation 연속 오류 | 후보 미노출·고정 fallback |
| `server_no_call` | 기존처럼 모든 AI 호출 0 |

답변 생성은 최대 3회다. Moderation의 일시적 기술 오류 재시도는 생성 횟수와 따로
기록한다.

## 6. P0-3 — Google Sheets 결과 시트만 확장한다

기존 검수 열과 숨김 원본 열을 삭제하거나 이름을 바꾸지 않는다. 기존 시트는 데이터
손실 없이 새 열을 추가한다.

사람이 먼저 볼 열:

| 열 | 내용 |
| --- | --- |
| `1차 후보` | 첫 번째 말풍선 또는 계약 파싱 오류 |
| `1차 검사` | 계약 오류, Moderation 카테고리와 주요 점수 |
| `재생성 이유` | 계약 오류 또는 출력 Moderation 범주 |
| `2차 후보` | 원인별 복구 말풍선 또는 미실행 |
| `2차 검사` | 계약 오류, Moderation 카테고리와 주요 점수 |
| `3차 후보` | 최소 안전 모드 말풍선 또는 미실행 |
| `3차 검사` | 계약 오류, Moderation 카테고리와 주요 점수 |
| `최종 처리` | 정상 생성 / 고정 fallback / 안전 도움 포함 fallback |
| `최종 안전 경로` | route와 priority 또는 없음 |
| `총 호출` | 생성·Moderation·평가 호출 수 |

오른쪽 숨김 원본 열:

```text
attempts_json
final_disposition
fallback_safety_route
fallback_safety_priority
```

`AnswerHistory`는 구조를 바꾸지 않는다.

- 정상 생성: 현재처럼 최종 말풍선 표시
- 고정 fallback: `[고정 fallback]`
- 안전 도움 포함 fallback: `[고정 fallback + 안전 도움: route / priority]`
- 생성 호출 없음: 기존처럼 `[답변 생성 없음]`

시도별 세부 내용은 실행별 결과 시트에서만 확인한다.

## 7. P0-4 — 운영과 Prompt Lab의 일치 테스트

최소한 다음 계약을 공통 모듈로 사용하거나 동일한 테스트 벡터로 일치시킨다.

- 출력 JSON schema
- 구조·제품 계약 검사
- Moderation 응답 정규화
- 카테고리별 재생성 지시
- 생성 호출 상한
- 최종 disposition 결정

Prompt Lab이 운영 코드와 별도 복구 로직을 계속 가진다면, 같은 모의 응답 순서를
넣었을 때 양쪽 최종 결과와 시도 이력이 같다는 테스트를 추가한다.

### 실제 API를 쓰지 않아도 되는 것과 꼭 써야 하는 것

최대 3회 상태 흐름 자체는 실제 API 비용을 쓰지 않고 모의 응답으로 검증한다.

| 검증 | 실행 방식 |
| --- | --- |
| 1→2→3차 전환, 호출 상한, fallback, 미차감 | 자동 테스트의 고정 모의 응답 |
| JSON·개수·길이·빈 값·PII 반복 검사 | 자동 테스트 |
| 결과 시트 열·시도 이력·비용 합산 | 자동 테스트 |
| 프롬프트 초안의 어조·실패 사례 탐색 | Codex 채팅과 사람 검토를 보조로 사용 가능 |
| 실제 API 모델의 한국어 문장·구조화 출력·Moderation·토큰·지연 | Prompt Lab의 실제 API 실행 필요 |

Codex 채팅은 제품 프롬프트 외에 Codex 자체 지시·도구·대화 맥락이 함께 작동하므로
운영 API와 동일한 회귀 결과로 간주하지 않는다. 비용을 줄이기 위해 실제 API 실행
범위를 다음처럼 단계화한다.

1. `smoke`: 연결·구조 확인용 10개
2. `core`: 품질·안전 골든 후보 30개
3. `release`: 출시 직전 최종 후보만 100개 이상 또는 전체 회귀 세트

프롬프트·모델 후보를 만들 때마다 전체 세트를 실행하지 않는다. Codex와 사람 검토로
명백한 실패 후보를 먼저 제거하고, 실제 운영 후보만 `core`를 통과시킨 뒤
`release`를 실행한다.

## 8. 필수 자동 테스트

실제 OpenAI 호출 없이 모의 응답으로 검증한다.

- 1차 정상 통과 → 시도 1개, `generated`
- 1차 계약 실패·2차 통과 → 시도 2개와 오류 코드 보존
- 1차 `violence` flag·2차 통과 → 카테고리 기반 재생성 이유 보존
- 1차 다중 카테고리 flag → 모든 카테고리·점수 보존
- 1·2차 실패·3차 통과 → 3차 `minimal_safe` 모드와 세 시도 보존
- 세 후보 모두 flag → flagged 후보를 최종 답변으로 반환하지 않고 fallback
- Moderation 일시 오류·재시도 성공 → 호출 수 정확
- Moderation 연속 오류 → 후보 미노출·fallback
- 어느 유효 후보에서든 안전 경로가 확인됨 → 뒤 시도에서 없애거나 낮추지 않고 안전 도움 포함 fallback
- 결과 시트에 1차·2차·최종 처리 열 저장
- 기존 결과 시트 열과 사람 검수값 보존
- `AnswerHistory`의 고정 ID·입력·실행별 열 구조 유지
- 답변 모델과 평가 모델 선택 구조 유지
- `server_no_call` 비용과 호출 수 0 유지

실행:

```bash
npm run test:prompt-lab
npm run test:venting-observation
```

## 9. 수정 후보 파일

| 목적 | 파일 |
| --- | --- |
| 시도별 실행 결과 | `tools/prompt-lab/server/eval-case-runner.mjs` |
| Moderation 정규화 | `tools/prompt-lab/server/openai-client.mjs` |
| 결과 행·읽기 쉬운 열 | `tools/prompt-lab/server/google-sheets-rows.mjs` |
| 실행별 시트 레이아웃 | `tools/prompt-lab/server/google-sheets-layout.mjs` |
| AnswerHistory fallback 표시 | `tools/prompt-lab/server/google-sheets-answer-history.mjs` |
| 복구 계약 테스트 | `tools/prompt-lab/tests/recovery-contract.test.mjs` |
| Sheets 무손실 변경 테스트 | `tools/prompt-lab/tests/google-sheets-repository.test.mjs` |
| 사용 설명 | `docs/dev/prompt-lab.md` |

## 10. 완료 조건

- [ ] 최대 세 후보·계약·Moderation이 각각 보인다.
- [ ] 13개 카테고리와 점수가 시도별로 보존된다.
- [ ] 출력 안전 재생성에 실제 카테고리가 사용된다.
- [ ] 두 번째 flagged 후보나 Moderation 미확인 후보를 정상 답변으로 취급하지 않는다.
- [ ] 최종 fallback 종류와 안전 경로를 확인할 수 있다.
- [ ] 기존 `AnswerHistory`, 모델 비교, 사람 검수, 비용 구조가 유지된다.
- [ ] 운영과 같은 모의 입력에서 같은 최종 disposition을 낸다.
- [ ] 기존 테스트와 새 테스트가 모두 통과한다.
- [ ] 모의 응답 자동 테스트와 실제 API 실행 범위가 구분되어 있다.

## 11. 이번 작업에서 하지 않을 것

- `AnswerHistory` 재구축
- 새로운 부모 비교 시트 추가
- 기존 `TestCases` 240개 일괄 교체
- 답변 모델·평가 모델 메뉴 변경
- 평가 AI를 운영 요청에 추가
- Prompt Lab에서 운영 사용자의 실패 원문 조회
- 네 번째 답변 생성 호출
- 임의 점수 임계값으로 Moderation 결과 재판정

## 12. 개발 완료 회신

1. 기준 브랜치와 커밋
2. 이미 구현돼 유지한 항목
3. 실제 변경 파일
4. 최대 3회의 시도별 결과 데이터 구조
5. 결과 시트의 추가 열
6. 세 후보 모두 실패했을 때 최종 disposition
7. 생성·Moderation·평가 최대 호출 수
8. 운영과 일치시키는 공통 모듈 또는 테스트 위치
9. 전체 자동 테스트 결과
10. 아직 제품 책임자가 확인해야 하는 문구나 화면

API key, 서비스 role key, 실제 사용자 기록은 회신이나 Git에 포함하지 않는다.
