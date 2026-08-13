---
type: implementation-change-request
title: 속마음 앱 런타임 변경 요청 — 출력 계약·inline Moderation·실패 결과 저장
status: ready-for-developer
date: 2026-08-13
audience: [developer, product_manager]
source_repository: https://github.com/hux2/pebbling-expo.git
source_branch: dev
baseline_commit: 44bc01c4a7b75f16e867c8848a90b33c1405ecbe
scope: app-runtime-only
supersedes:
  - 속마음-결정적-출력검사-P0-변경요청.md
  - 속마음-D안-런타임-변경요청.md의 출력 한도·Moderation·재생성·실패 원문 저장 규칙
---

# 속마음 앱 런타임 변경 요청

## 1. 이 문서만 구현 기준으로 읽어주세요

이 문서는 `dev@44bc01c4a7b75f16e867c8848a90b33c1405ecbe`를 기준으로
실제 앱의 속마음 생성 서버·DB·앱 표시 경로에서 바꿀 내용만 정리한다.
문서의 `현재`는 이 커밋에 이미 있는 동작을, `변경 후`는 이번에
새로 구현할 목표를 뜻한다.

이번 작업에는 Prompt Lab, Google Sheets, 평가 AI, 테스트 케이스 화면이 포함되지
않는다. 해당 도구는 제품 책임자가 별도 저장소에서 관리한다.

과거 문서와 충돌하면 이 문서가 우선한다. 다음 과거 기준은 구현하지 않는다.

- `1~20개 / 개별 300자 / 전체 3,000자`
- standalone `/v1/moderations` 재검사
- Moderation flagged 결과 미노출
- Moderation을 이유로 한 재생성
- 최대 3회의 의미 생성
- OpenAI 공식 JavaScript SDK 신규 설치

## 2. 확정된 변경사항

| 항목 | 최종 기준 |
|---|---|
| 결과 유형 | `THOUGHT` 또는 `THOUGHT_WITH_SAFETY_SUPPORT` |
| 말풍선 | 두 유형 모두 2~10개 |
| 개별 길이 | 공백·문장부호 포함 최대 200자 |
| 전체 길이 | 공백·문장부호 포함 최대 1,000자 |
| 질문 | 런타임 개수 제한 없음. `questionCount`는 분석 지표로만 저장 |
| Moderation | Responses 생성 요청 안에서 input·output inline Moderation 수행 |
| `flagged=true` | 참고·분석 신호로 저장하고, 이것만으로 숨김·재생성·안전 route 변경 안 함 |
| `type=error` | Moderation 검사 자체가 실패한 상태. 오류를 저장하되 유효한 속마음은 숨기거나 다시 생성하지 않음 |
| standalone Moderation | 제거 |
| 실패 후보 | 실패 사유와 생성 후보를 서버 전용 DB에 저장 |
| 교정 생성 | 구조·개수·길이 오류에만 최대 1회 |
| Moderation 재생성 | 없음 |
| PII·안전 조합 오류 재생성 | 없음 |
| 기술 재시도 | 현재의 timeout·일시 오류 재시도만 유지. 새 재시도 정책 추가 안 함 |
| SDK | 현재 REST `fetch()` 유지 |

Moderation은 이번 버전에서 자동 차단 장치가 아니라 개선을 위한 관찰 신호다.
다만 JSON·출력 계약·PII·안전 상태 조합 검사는 계속 실제 노출 조건으로 사용한다.

### 읽기 전에 알아둘 뜻

- **생성 후보:** AI가 만든 하나의 응답이다. 검사를 통과해 앱에 저장되기
  전의 결과도 포함한다.
- **inline Moderation:** 별도 `/v1/moderations` API를 호출하는 방식이 아니다.
  속마음을 생성하는 Responses 요청에 `moderation` 옵션을 넣고, 그 응답에서
  사용자 입력과 생성 출력의 검사 결과를 각각 받는다.
- **`flagged=true`:** 위험 가능성을 나타낸 분석 신호일 뿐, 이번 버전에서
  자동 미노출·재생성 조건이 아니다.
- **교정 생성:** 속마음의 방향을 새로 정하는 재생성이 아니라, 직전 후보의
  JSON·개수·길이만 계약에 맞게 고치는 두 번째이자 마지막 AI 호출이다.
- **고정 안전 도움:** AI가 작성하는 말풍선이 아니라, `safety_route`에 따라
  서버·앱이 붙이는 승인된 문구·연락처·행동 버튼이다.
- **attempt:** 최초 생성 또는 교정 생성 한 번의 기록이다. timeout 같은
  기술 재시도는 새 attempt로 계산하지 않는다.
- **finalize:** observation 저장, 횟수 차감, entry 소비, request 완료를
  하나의 DB 트랜잭션으로 끝내는 처리다.

## 3. 현재 구현과 변경 후 차이

| 항목 | 현재 | 변경 후 |
|---|---|---|
| 출력 한도 | 3~7개, 90자, 420자 | 2~10개, 200자, 1,000자 |
| 질문 검사 | 일반 최대 1개, 안전 결과 0개 | 개수 오류 제거, 지표만 유지 |
| Moderation | 생성 뒤 출력만 standalone 검사 | Responses 안에서 입력·출력 동시 검사 |
| Moderation 저장 | 상세 결과를 DB에 저장하지 않음 | input/output의 type·flagged·categories·scores·applied input types를 저장 |
| flagged 처리 | 새로운 후보 생성, 반복 시 실패 | 같은 후보를 저장·차감·표시 |
| Moderation error | 생성 실패로 이어질 수 있음 | 오류만 저장하고 정상 후보 계속 처리 |
| 실패 후보 원문 | 저장하지 않음 | 실패 코드와 함께 서버 전용 DB에 보관 |
| DB 배열 제약 | 3~7개 | 2~10개 |

인증, quota 사전검사, 요청 스냅샷, `store:false`, 구조화 출력, 요청 멱등성,
observation·차감·entry 소비의 원자적 finalize는 유지한다.

## 4. 변경 후 처리 흐름

```text
앱이 속마음 생성을 요청
→ 서버 사전검사와 입력 스냅샷 고정
→ Responses API로 구조화 후보 생성
   · moderation: { model: "omni-moderation-latest" } 포함
→ 생성 후보와 input/output Moderation 상태 기록
→ 결정적 검사
   ├─ 통과
   │   └─ Moderation 상태와 관계없이 저장·1회 차감·표시
   ├─ trim·빈 배열 항목 제거만으로 수정 가능
   │   └─ 코드 수정 후 전체 검사 재실행
   ├─ JSON·필드 타입·개수·길이 실패
   │   └─ 직전 후보와 정확한 오류 코드로 교정 생성 최대 1회
   ├─ 직접 식별정보 반복
   │   └─ 고확신 값만 코드 마스킹 후 재검사
   └─ 안전 상태 조합 오류 또는 최종 실패
       └─ 추가 생성 없이 기존 실패 API 응답
```

### 상황별 동작

| 상황 | 사용자에게 보이는 결과 | AI 교정 호출 | 저장·횟수 차감 |
|---|---|---:|---|
| 계약 통과 + unflagged | 같은 후보 | 없음 | 정상 저장·1회 차감 |
| 계약 통과 + input/output flagged | 같은 후보 | 없음 | 정상 저장·차감 + Moderation 기록 |
| 계약 통과 + input/output `type:error` | 같은 후보 | 없음 | 정상 저장·차감 + 오류 기록 |
| 빈 말풍선 일부 | 빈 항목 제거 후 후보 | 없음 | 재검사 통과 시 정상 처리 |
| JSON·개수·길이 실패 | 교정 후보 | 최대 1회 | 교정 통과 시 정상 처리 |
| 교정 후보도 실패 | 기존 생성 실패 화면 | 없음 | 미차감·entry 미소비 |
| PII를 안전하게 마스킹하지 못함 | 기존 생성 실패 화면 | 없음 | 미차감·entry 미소비 |
| 유효하지 않은 안전 조합 | 기존 실패 API 계약 | 없음 | 미차감·entry 미소비 |
| Responses 요청 전체 실패 | 기존 실패 화면 | 없음. 기존 기술 재시도만 별도 유지 | 최종 실패 시 미차감 |

## 5. 구현 요구사항

### 5.1 출력 계약

대상:

- `supabase/functions/generate-venting-observation/contract.ts`

```ts
export const OBSERVATION_OUTPUT_LIMITS = {
  minimumBubbles: 2,
  maximumBubbles: 10,
  maximumBubbleCharacters: 200,
  maximumTotalCharacters: 1000,
} as const;
```

함께 변경한다.

1. 에러 메시지의 `3~7`, `90`, `420` 하드코딩을 상수 참조로 교체한다.
2. `maximumQuestions`와 질문 개수에 따른 계약 오류를 제거한다.
3. `countQuestions()`와 `metrics.questionCount`는 분석 지표로 유지한다.
4. `THOUGHT`와 `THOUGHT_WITH_SAFETY_SUPPORT` 모두 같은 한도를 사용한다.
5. 빈 문자열을 trim 후 제거하고 전체 계약을 다시 검사한다.

### 5.2 Responses inline Moderation

대상:

- `supabase/functions/generate-venting-observation/providers/openai.ts`
- `supabase/functions/generate-venting-observation/generation-pipeline.ts`
- 관련 type

현재 REST Responses 요청에 다음을 추가한다.

```ts
moderation: {
  model: "omni-moderation-latest",
},
```

이는 별도 Moderation API를 한 번 더 호출하는 방식이 아니다. 같은 Responses
응답에 포함된 input·output Moderation 결과를 읽는다.

다음을 input과 output으로 구분해 보존한다.

- 실제 반환된 `type`
- `flagged`
- `categories`
- `category_scores`
- `category_applied_input_types`
- `type:error`일 때 제공된 `message`

규칙:

- 정상·flagged 경로에서 별도 `/v1/moderations`를 호출하지 않는다.
- `flagged` 또는 score로 `safety_route`나 `safety_priority`를 바꾸지 않는다.
- `flagged`와 `type:error`는 재생성 사유가 아니다.
- 여기서 `type:error`는 Responses 요청 전체 실패가 아니라 Moderation 하위
  검사만 정상 완료되지 않은 상태다. 사용할 수 있는 생성 후보가 없는
  Responses 전체 실패는 기존 생성 실패 경로로 처리한다.
- 성공 result의 type 문자열은 실제 fixture를 보고 파싱한다.
- 전체 Responses 요청 실패는 기존 transport 재시도 정책을 따른다.

### 5.3 자동 수정과 교정 생성

고정 실패 코드를 사용한다.

```text
JSON_PARSE_FAILED
INVALID_FIELD_TYPE
BUBBLE_COUNT_INVALID
ALL_BUBBLES_EMPTY
BUBBLE_TOO_LONG
TOTAL_LENGTH_EXCEEDED
INVALID_SAFETY_COMBINATION
PII_ECHO_DETECTED
PROVIDER_REFUSAL
```

교정 생성 가능:

- `JSON_PARSE_FAILED`
- `INVALID_FIELD_TYPE`
- `BUBBLE_COUNT_INVALID`
- `ALL_BUBBLES_EMPTY`
- `BUBBLE_TOO_LONG`
- `TOTAL_LENGTH_EXCEEDED`

교정 생성에는 같은 입력 스냅샷, 직전 후보, 실패 코드와 정확한 한도만 전달한다.
의미와 돌멩이 말투는 유지하고 계약만 고치도록 지시한다. 교정 후보도 inline
Moderation과 전체 검사를 거친다. 실패하면 세 번째 후보를 만들지 않는다.
따라서 AI 의미 생성은 `최초 1회 + 교정 최대 1회 = 최대 2회`다. timeout·일시적
연결 오류의 기존 기술 재시도는 이 횟수와 별도다.

재생성하지 않는 항목:

- input/output Moderation flagged 또는 error
- `PII_ECHO_DETECTED`
- `INVALID_SAFETY_COMBINATION`
- 사용자 부정 피드백이나 별도 품질 점수

PII는 기존에 검출하는 고확신 전화번호·이메일·상세 주소·주민등록번호 형태·긴 숫자
식별자만 코드로 마스킹한다. 안전하게 마스킹하지 못하면 기존 생성 실패 응답을
사용한다.

### 5.4 실패 후보와 Moderation 저장

새 migration에 `journal.venting_observation_attempts`를 추가한다. 최초 생성과
교정 생성 각각을 한 행으로 저장한다. 이 테이블은 사용자 기능이 아니라,
왜 특정 후보가 flagged되거나 검사에 실패했는지 팀이 확인해 프롬프트와
검수 로직을 개선하기 위한 **서버 전용 진단 기록**이다.

P0 필수 정보:

| 그룹 | 필드 |
|---|---|
| 연결 | `id`, `user_id`, `request_id`, `attempt_no`, `attempt_type` |
| 재현 | `model_id`, `prompt_version`, `policy_version`, `provider_response_id` |
| 후보 | `raw_output`, 파싱 결과 유형·bubbles·route·priority |
| 검사 | `validator_passed`, `failure_stage`, `failure_codes`, 개수·길이·질문 지표 |
| PII | `pii_echo_detected`, `pii_masked` |
| Moderation | `input_moderation`, `output_moderation` JSONB |
| 결과 | `candidate_decision` (`selected` / `rejected` / `provider_failed`) |
| 보관 | `created_at` |

`input_moderation`과 `output_moderation` JSONB에는 앞 절의 필드를 누락 없이
보존한다.

저장 규칙:

- `attempt_no=1`, `attempt_type=initial`은 최초 생성을, `attempt_no=2`,
  `attempt_type=correction`은 교정 생성을 뜻한다.
- timeout 같은 동일 호출의 기술 재시도는 새 `attempt_no`를 만들지 않는다.
- provider가 실제로 반환한 input/output Moderation 상태와 전체 검사 결과를
  저장한다. provider 전체 실패로 받은 후보가 없으면 해당 필드는 null일 수
  있다.
- `raw_output`은 flagged 후보 또는 내부 검사 실패 후보에 저장한다.
- PII 실패 후보는 식별정보를 마스킹한 텍스트만 저장한다.
- 전체 API 요청 body, API key, header, 시스템 프롬프트와 사용자 입력 원문을
  attempts에 복제하지 않는다.
- `(request_id, attempt_no)`를 unique로 두어 replay·기술 재시도 중복을 방지한다.
- 첫 후보와 교정 후보는 별도 행으로 남기며 서로 덮어쓰지 않는다.
- `candidate_decision=selected`는 서버가 finalize할 후보로 선택했다는 뜻이며,
  실제 화면 열람을 의미하지 않는다.
- 최종 observation은 기존 request 행의 `observation_id`를 `request_id`로 조인해
  찾는다. attempts 테이블에 observation FK를 중복 저장하지 않는다.
- `user_id`와 `request_id`는 사용자·요청 삭제 시 cascade한다.
- 테이블은 service role만 접근할 수 있고 앱 사용자 role에는 권한을 주지 않는다.
- 원문을 콘솔, Sentry/Firebase, Google Sheets에 출력하지 않는다.
- attempts 저장 실패 때문에 계약을 통과한 사용자 결과를 숨기거나 다시 생성하지
  않는다. 비민감 저장 오류 코드만 남긴다.

실패·flagged 후보 원문의 **보관 기간, `delete_at`, 자동 삭제 Cron은 이번
구현 범위에 넣지 않는다.** 사용자 또는 원본 request가 삭제되면 cascade로
함께 삭제되게 한다. 기간 기반 삭제는 실제 검토 주기와 개인정보처리방침을
확정한 뒤 별도로 결정한다.

### 5.5 DB 제약과 finalize

기존 migration을 수정하지 않고 새 migration을 만든다.

1. `venting_observations_bubbles_check`를 2~10개로 확장한다.
2. ready 상태의 배열 길이 제약을 2~10개로 확장한다.
3. 기존 v040/v050을 덮어쓰지 않고 `finalize_venting_observation_v060()`을 만든다.
4. v060은 v050의 안전 조합 검사와 v040의 observation 저장·quota 차감·entry
   소비·request 완료를 동일 트랜잭션으로 처리하되 배열 길이만 2~10개로 사용한다.
5. Edge Function은 v060을 호출한다.
6. 새 함수는 public·anon·authenticated 권한을 제거하고 service role만 실행한다.

`moderation_call_count`는 standalone 호출 수이므로 변경 후 0으로 기록한다.
`generation_call_count`는 최초+교정 후보 기준 최대 2를 유지한다. Moderation
flagged/error는 `regeneration_reason`으로 기록하지 않는다.

### 5.6 정상 안전 도움 결과와 최종 실패 화면

정상 생성과 최종 실패를 다음과 같이 구분한다.

| 결과 | 사용자에게 보일 것 | 저장·차감 |
|---|---|---|
| 정상 `THOUGHT` | 2~10개 속마음 말풍선 | 저장·1회 차감 |
| 정상 `THOUGHT_WITH_SAFETY_SUPPORT` | 2~10개 말풍선 + route에 맞는 고정 안전 도움 | 저장·1회 차감 |
| 교정까지 실패한 최종 생성 실패 | 현재의 일반 생성 실패 화면 | observation 미저장·미차감·entry 미소비 |

- 정상 `THOUGHT_WITH_SAFETY_SUPPORT`의 안전 도움은 이번에 앱 화면에 연결해야
  하는 **변경 후 목표**다. 현재 `dev` 화면에 이미 노출되고 있다는 뜻이 아니다.
- 안전 도움 문구·연락처·버튼은 AI가 만들지 않고, 기존 서버 자원을
  `safety_route`로 조회해 말풍선 밖에 표시한다.
- `safety_priority=standard`는 안전 도움을 접힌 상태로, `urgent`는 첫
  화면에서 펼친 상태로 표시한다.
- 최종 생성 실패에는 새 실패 화면이나 `generation_failed_with_safety_support`
  분기를 추가하지 않는다. 현재 `generation_failed`, `retryAllowed:true`를 유지한다.

현재 UI에서 10개 말풍선과 최대 1,000자가 잘리거나 겹치지 않는지만 확인한다.
필요한 최소 레이아웃 보정 외에는 디자인을 변경하지 않는다.

## 6. 예상 변경 파일

필수:

- `supabase/functions/generate-venting-observation/contract.ts`
- `supabase/functions/generate-venting-observation/providers/openai.ts`
- `supabase/functions/generate-venting-observation/generation-pipeline.ts`
- `supabase/functions/generate-venting-observation/types.ts`
- `supabase/functions/generate-venting-observation/index.ts`
- 새 `supabase/migrations/*.sql`
- 관련 Edge Function·migration 테스트
- `src/screens/venting/VentingObservationDetailScreen.tsx`
- 정상 안전 결과의 고정 안전 도움 표시 테스트

변경이 필요한 경우에만:

- `src/types/venting.ts`
- `src/api/venting/observations.ts`
- `src/screens/venting/detail/VentingThoughtBubbleList.tsx`
- 관련 앱 UI 테스트

다음은 이번 작업에서 수정하지 않는다.

- `tools/prompt-lab/**`
- Google Sheets 구조·데이터
- Prompt Lab 관련 문서·테스트
- 운영 프롬프트 본문과 평가 프롬프트
- 테스트 케이스 CSV

## 7. 필수 수용 테스트

### 계약과 생성

- [ ] 두 result type 모두 말풍선 2개와 10개 통과
- [ ] 1개와 11개 실패
- [ ] 개별 200자 통과·201자 실패
- [ ] 전체 1,000자 통과·1,001자 실패
- [ ] 복수 질문이 계약 오류를 만들지 않고 지표에는 남음
- [ ] 빈 항목 일부 제거 후 2개 이상 남으면 재생성 없이 통과
- [ ] 구조·개수·길이 실패만 교정 생성 최대 1회
- [ ] 교정 실패 뒤 세 번째 후보가 생성되지 않음

### Moderation과 저장

- [ ] inline input/output unflagged 결과 전체 필드 저장
- [ ] inline input/output flagged 결과가 같은 후보로 저장·표시
- [ ] `type:error`가 같은 후보로 저장·표시되고 오류 message가 기록됨
- [ ] flagged/error 때문에 generation call count가 증가하지 않음
- [ ] standalone `/v1/moderations` 호출이 없음
- [ ] flagged·내부 검사 실패 후보의 원문과 실패 코드 저장
- [ ] PII 실패 후보 원문은 마스킹 후 저장
- [ ] 동일 request replay가 attempt 중복 행을 만들지 않음
- [ ] authenticated 앱 사용자가 attempts를 직접 조회하지 못함

### DB와 앱

- [ ] 실제 DB가 2개·10개 저장을 허용하고 1개·11개를 거부
- [ ] v060이 observation·quota·entry·request를 원자적으로 완료
- [ ] 정상 결과만 정확히 1회 차감·entry 소비
- [ ] 최종 실패는 미차감·entry 미소비
- [ ] 10개·1,000자 결과가 작은 iPhone과 Android에서 스크롤 가능
- [ ] 정상 `THOUGHT`는 말풍선만 표시
- [ ] 정상 `THOUGHT_WITH_SAFETY_SUPPORT`는 말풍선과 route별 고정 안전 도움을 함께 표시
- [ ] `standard`는 안전 도움을 접힌 상태, `urgent`는 펼친 상태로 표시
- [ ] 최종 생성 실패는 현재 일반 실패 화면을 사용하고 미차감
- [ ] 별도 `generation_failed_with_safety_support` 실패 분기가 추가되지 않음

실행:

```bash
npm run test:venting-observation
npm run typecheck
```

DB 경계·권한·원자성은 소스 문자열 검사만으로 통과 처리하지 말고
migration을 적용한 DB 통합 테스트 결과를 회신한다.

## 8. 배포 순서

1. 새 migration: attempts, 2~10 제약, v060
2. Edge Function
3. 앱: 정상 안전 결과의 고정 안전 도움 연결과 최대 출력 UI QA
4. 제품 책임자가 별도로 전달하는 확정 프롬프트는 회귀 확인 후 새 immutable
   버전으로 등록·활성화

DB보다 Edge Function을 먼저 배포하면 2개 또는 8~10개 결과가 DB 저장에서 실패할
수 있다. 롤백할 때는 이전 Edge Function을 먼저 복구하고, 이미 저장된 2개·8~10개
데이터를 확인하기 전 DB 제약을 다시 줄이지 않는다.

## 9. 개발 완료 회신

다음만 회신한다.

1. 변경 파일과 migration 이름
2. current → changed 차이
3. inline input/output Moderation fixture: unflagged·flagged·error
4. flagged/error가 재생성 없이 표시된 증거
5. 실패 후보와 실패 코드가 저장된 증거
6. 교정 생성이 최대 1회인 증거
7. DB 2개·10개 경계와 원자성·권한 테스트
8. 전체 자동 테스트와 typecheck 결과
9. 최대 출력의 iOS·Android QA 결과
10. 배포·롤백 방법

## 10. 공식 참고

- [OpenAI inline Moderation](https://developers.openai.com/api/docs/guides/moderation#moderate-generated-content)
- [OpenAI Moderation 결과 필드](https://developers.openai.com/api/docs/guides/moderation#understand-moderation-results)
- [OpenAI Responses create](https://developers.openai.com/api/reference/resources/responses/methods/create)

OpenAI 공식 문서 확인 기준일: 2026-08-13.
