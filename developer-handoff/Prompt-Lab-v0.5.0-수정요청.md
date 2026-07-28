---
type: Developer Change Request
title: Prompt Lab v0.5.0 최종 수정 요청
description: feature/venting-tab의 현재 구현을 보존하면서 v0.5.0 안전·생성·실패 계약과 평가 원본을 일치시키기 위한 개발 요청.
tags: [pebbling, prompt-lab, developer-handoff, evaluation, v0.5.0]
resource: https://github.com/hux2/pebbling-expo/tree/feature/venting-tab/tools/prompt-lab
timestamp: 2026-07-28T12:00:00+09:00
status: ready_for_development
audience: [developer, product_manager]
source_branch: feature/venting-tab
source_commit: fba64161d4f8ba5c78fedc8d5ec2c34416e55b66
implementation_status: partial_implementation
---

# Prompt Lab v0.5.0 최종 수정 요청

## 1. 전달 결론

이 문서는 `hux2/pebbling-expo`의 `feature/venting-tab`,
커밋 `fba64161d4f8ba5c78fedc8d5ec2c34416e55b66`을 기준으로 작성했다.

현재 Prompt Lab과 운영 코드는 핵심 기반이 이미 구현돼 있다. 이번 작업은 도구를
새로 만드는 일이 아니라, **운영 v0.4.1과 Prompt Lab 평가기 v0.4.0을
v0.5.0으로 맞추고, Prompt Lab의 안전 도움만 반환하는 평가 경로와 실패 처리를
정리하는 작업**이다.

완료 전 가장 중요한 결과는 다음 네 가지다.

1. 생성 자격을 통과한 모든 요청은 `일반 속마음` 또는 `속마음+안전 도움`을 반환한다.
2. 두 결과 모두 캐릭터 말풍선 3~7개를 가진다.
3. 안전 도움 결과도 정상 생성·저장 성공이면 일반 결과와 동일하게 1회 차감되고 분석 기준이 갱신된다.
4. 생성·검수·저장 전에 실패하면 개인화 결과를 확정하지 않고 미차감·분석 기준 미갱신으로 끝난다.

## 2. 현재 코드에서 이미 구현된 기반

아래는 유지하고 회귀만 확인한다. 같은 기능을 새로 만들지 않는다.

| 영역 | 현재 확인된 구현 |
|---|---|
| 서버 생성 자격 | 권한·횟수·새 기록 스냅샷·80자 게이트 |
| 입력 범위 | 최근 24시간 기록, 최대 20,000자 입력 구성 |
| 운영 정상 결과 | `THOUGHT`, `THOUGHT_WITH_SAFETY_SUPPORT` |
| 말풍선 구조 | 두 결과 모두 3~7개, 개별 90자·전체 420자 검사 |
| 출력 검수 | 구조 검사, 질문 수, 직접 식별정보 반복 검사, 출력 Moderation |
| 저장·사용량 | 두 정상 결과가 공통 finalize 경로를 거쳐 저장·차감·분석 기준 확정 |
| 기술 오류 재시도 | 일시적 네트워크·429·일부 5xx에 한해 답변 생성 호출 최대 2회 |
| 운영 활성 프롬프트 | `observation_prompt_v0.4.1`; 안전 결과도 말풍선을 생성하도록 이미 변경됨 |
| Prompt Lab 원본 | CSV 240개와 실행·비용·지연 추적 |
| 검수 저장 | Google Sheets의 Runs·실행 결과 시트, 자동 판정·사람 판정·메모·golden 관련 필드 |

정적 검수 당시 테스트는 41개 중 38개가 통과했다. 나머지 3개는 코드 실패가 아니라
읽기 전용 점검 환경에 `@supabase/supabase-js`, `typescript` 의존성이 설치되지 않아 로드하지
못했다. 개발 환경에서 의존성을 설치한 뒤 전체 테스트를 다시 실행해야 한다.

## 3. 현재 코드와 v0.5.0의 정확한 차이

| 항목 | 현재 기준 커밋 | 최종 v0.5.0 |
|---|---|---|
| 운영 실행 프롬프트 | `observation_prompt_v0.4.1` | 동봉한 `observation_prompt_v0.5.0` |
| 평가기 | `venting-evaluator-v0.4.0` | v0.5.0 제품·안전 계약 |
| 운영 JSON Schema | 정상 결과 2종 | 현재 2종 유지 |
| Prompt Lab 행동 | `generate_standard`, `generate_with_safety_support`, `safety_only`, `server_no_call` | 앞의 2종과 `server_no_call`만 |
| 안전 시나리오 | `safety_only` 19개가 말풍선 0개를 기대 | 19개 모두 `generate_with_safety_support`, 말풍선 3~7개 |
| 평가 CSV | 240개지만 구정책 | 동봉 CSV 240개로 교체 |
| 구조 실패 | 일반 실패 처리 | 원본 입력 전체로 계약 제한 재생성 1회 |
| 출력 안전·제품 실패 | 일반 실패 처리 | 원본 입력 전체로 안전 제한 재생성 1회 |
| 최종 실패 | 내부 재시도 가능 오류 중심 | 대체 안내, 안전 관련이면 고정 안전 도움도 표시, 미차감 |

현재 코드의 시나리오 ID 240개와 동봉 CSV의 ID 240개는 같다. 행 내용이 다른 것은
총 24개이며, 그중 핵심은 `safety_only` 19개를 정상 안전 지원 생성으로 바꾸는 것이다.

동봉 CSV의 기대 행동 분포:

- `generate_standard`: 217개
- `generate_with_safety_support`: 22개
- `server_no_call`: 1개

### 수정 위치를 찾기 위한 코드 지도

| 확인할 내용 | 현재 코드 위치 |
|---|---|
| 운영 정상 결과 2종·출력 제한 | `supabase/functions/generate-venting-observation/contract.ts` |
| 활성 프롬프트 조회·버전 오류 문구 | `supabase/functions/generate-venting-observation/config.ts` |
| 운영 생성·재시도·replay·finalize | `supabase/functions/generate-venting-observation/index.ts` |
| 운영 v0.4.1 등록·활성화 | `supabase/migrations/20260727103346_activate_observation_prompt_v0_4_1.sql` |
| 과거 `safety_only` 요청 상태 DB 제약 | `supabase/migrations/20260724073851_align_venting_observation_v040_contract.sql` |
| Prompt Lab 사례 허용값 | `tools/prompt-lab/server/case-contract.mjs`, `public/case-options.mjs` |
| 구조·평가기 | `tools/prompt-lab/server/logic-evaluator.mjs`, `ai-evaluator.mjs`, `evaluation-criteria.mjs` |
| 평가 원본 | `tools/prompt-lab/evals/evaluation_scenario_bank.csv` |
| Google Sheets 저장 | `tools/prompt-lab/server/google-sheets-repository.mjs`, `google-sheets-rows.mjs` |

파일명은 기준 커밋의 현재 위치다. 리팩터링할 수 있지만 운영과 Prompt Lab의 계약이
다시 갈라지지 않도록 공통 원본 또는 명시적인 동기화 테스트를 남긴다.

## 4. P0 — 이번 전달에서 반드시 반영할 것

### P0-1. v0.5.0 프롬프트를 발행하고 운영·Prompt Lab에서 같은 계약 사용

기준 원본은
[`implementation/observation_prompt_v0.5.0.txt`](implementation/observation_prompt_v0.5.0.txt)다.

- 변경 가능한 초안과 발행 후 변경하지 않는 버전 아티팩트의 기존 구조를 유지한다.
- 운영 코드와 Prompt Lab이 같은 v0.5.0 프롬프트 원본 또는 같은 빌드 아티팩트를 사용한다.
- 평가기 또한 v0.5.0 결과 유형·질문·안전·조언 금지 규칙으로 갱신한다.
- v0.5.0 프롬프트에는 `안전 화면만`, 말풍선 0개, 제3의 정상 결과 의미를 두지 않는다.
- 운영 JSON Schema의 정상 결과 2종 구조는 유지한다.
- `config.ts`처럼 v0.4.0이 오류 문구에 고정된 곳은 활성 버전과 무관한 일반 문구로
  바꾸거나 실제 활성 버전에서 가져온다.
- 기존 DB RPC 이름의 `_v040`처럼 내부 호환을 위해 유지할 이름은 기능이 같다면
  단순히 버전 표기를 맞추려는 이유만으로 새 RPC를 만들지 않는다.
- 기준 브랜치의 마이그레이션은 v0.4.1을 활성화하도록 정의하지만, 실제 배포 DB의
  활성 config는 개발 시작 전에 읽기 전용으로 확인해 회신한다.

완료 증거:

- 운영 실행과 Prompt Lab 실행에 기록된 프롬프트·정책 버전
- 같은 입력과 모델에서 두 경로의 파서·구조 검사 결과가 일치하는 테스트

### P0-2. `safety_only`를 활성 계약에서 제거

Prompt Lab의 타입, 선택지, 파서, 자동 평가, 집계와 테스트에서 `safety_only`와
`SAFETY_ONLY`를 정상 동작으로 인정하지 않는다.

허용되는 시스템 동작은 다음 세 가지다.

| 시스템 동작 | AI 호출 | 캐릭터 말풍선 | 앱 결합 |
|---|---:|---:|---|
| `generate_standard` | 실행 | 3~7개 | 없음 |
| `generate_with_safety_support` | 실행 | 3~7개 | 고정 안전 도움 |
| `server_no_call` | 미실행 | 없음 | 서버의 고정 제품 안내 |

급박한 위험 단서가 있어도 개인화 말풍선을 생략하는 제3의 정상 결과를 만들지 않는다.
대신 `generate_with_safety_support` 안에서 앱이 고정 안전 도움 영역을 더 먼저 또는 더
강하게 보여줄 수 있는 신호를 사용한다. 필드 이름은 기존 구조에 맞게 개발자가 정한다.

운영 요청 테이블과 `respondToReplay`에는 과거 `safety_only` 행을 읽기 위한 호환 코드가
남아 있다. 이를 새 생성 경로와 혼동하지 않는다.

- 새 요청이 `safety_only` 상태나 `SAFETY_ONLY` 결과를 저장하는 경로는 0개여야 한다.
- 기존 행이 실제로 존재하는지 배포 DB에서 먼저 확인한다.
- 기존 행이 있으면 읽기 전용 호환을 유지하거나 명시적으로 마이그레이션한다.
- 기존 행이 없고 지원 클라이언트에도 의존성이 없다는 근거가 있을 때만 DB 허용값과
  replay 분기를 제거한다.
- 선택한 호환·마이그레이션 방식과 확인 쿼리 결과를 회신한다. 실제 사용자 원문은
  포함하지 않는다.

### P0-3. 평가 원본을 동봉 CSV 240개로 교체

단일 원본은
[`validation/evaluation_scenario_bank.csv`](validation/evaluation_scenario_bank.csv)다.

- 현재 브랜치의 240개 CSV를 동봉 CSV로 교체한다.
- 240개 ID는 유지하고 달라진 24개 행의 기대 동작·제약·문구를 반영한다.
- 이전 `safety_only` 19개는 `generate_with_safety_support`로 바꾼다.
- `expected_output_type`은 `expected_system_action`에서 파생하고 이중 수정하지 않는다.
- `server_no_call`만 답변 모델과 평가 AI의 호출·토큰·비용이 0이어야 한다.
- 80자 미만 언어 사례는 게이트를 통과했다고 가정해 출력 품질을 검사한다.

완료 증거:

- 행 수 240, 고유 ID 240
- 기대 행동 분포 `217 / 22 / 1`
- 허용되지 않은 `safety_only` 값 0개
- CSV 구조 검증 명령과 결과

### P0-4. 정상 안전 결과의 기존 저장·차감 경로를 보존

현재 운영 코드는 `THOUGHT_WITH_SAFETY_SUPPORT`도 공통 finalize 경로에서 처리한다.
이 경로를 없애거나 안전 도움 결과를 미차감으로 바꾸지 않는다.

- 두 정상 결과 모두 구조·Moderation·제품 검수와 저장에 성공하면 즉시 1회 차감한다.
- 두 정상 결과 모두 속마음 분석 기준을 갱신한다.
- 화면을 실제로 열었는지는 차감 조건이 아니다.
- 고정 안전 도움은 AI가 쓰지 않고 승인된 서버·앱 콘텐츠를 결합한다.

완료 증거:

- 일반 결과와 안전 도움 결과의 저장·차감·분석 기준 회귀 테스트
- 결과 유형별 finalize 경로가 분기 때문에 달라지지 않는다는 코드 위치

### P0-5. 원인별 한정 재생성과 최종 실패 처리

현재 일시적 기술 오류의 1회 재시도는 유지한다. 첫 답변 생성 결과가 저장되기 전에
실패했을 때만 아래 원칙을 적용한다.

| 최초 실패 원인 | 허용하는 두 번째 답변 생성 |
|---|---|
| 일시적 네트워크·429·재시도 가능한 5xx | 같은 요청 재시도 |
| 파싱·JSON Schema·결정적 출력 계약 실패 | 원본 입력 전체로 계약 제한 재생성 |
| 출력 Moderation·결정적 안전/제품 규칙 실패 | 원본 입력 전체로 안전 제한 재생성 |

공통 제한:

- 최초 호출을 포함한 답변 생성 모델 호출은 최대 2회다.
- 첫 실패 원인에 해당하는 한 경로만 선택하며 재시도·계약 재생성·안전 재생성을 연쇄하지 않는다.
- 문장 일부 repair나 주관적 품질을 이유로 한 무작위 재생성은 하지 않는다.
- 출력 Moderation 분류 호출은 답변 생성 호출 수와 구분해서 기록한다.
- 두 번째 결과도 실패하면 추가 AI 호출 없이 생성 실패 대체 안내를 표시한다.
- 안전 관련 검수 실패라면 대체 안내에 고정 안전 도움도 함께 표시한다.
- 개인화 결과 저장 전 실패는 횟수 미차감·분석 기준 미갱신이다.
- 저장 성공 뒤 조회·표시만 실패했다면 새 AI 호출·추가 차감 없이 같은 저장 결과를 복구한다.

완료 증거:

- 각 실패 원인별 답변 생성 호출 수와 최종 상태 자동 테스트
- 세 번째 답변 생성 호출이 없다는 테스트
- 저장 전 실패와 저장 후 조회 실패의 차감·복구 테스트

### P0-6. 기존 Google Sheets 검수 흐름을 유지하고 증거를 완성

Google Sheets 실행·검수 저장은 이미 구현돼 있으므로 로컬 SQLite나 별도 검수 화면으로
되돌리지 않는다.

각 실행 결과에서 다음 네 층을 구분해 확인할 수 있어야 한다.

1. 결정적 구조 검사
2. OpenAI 출력 Moderation
3. 평가 AI 판정과 근거
4. 사람의 최종 판정·수정 이유

함께 보존할 실행 메타데이터:

- 시나리오 ID와 `case_hash`
- 답변·평가기 모델
- 프롬프트·정책·평가기 버전
- 답변 생성 호출 수와 Moderation 호출
- 토큰·비용·지연
- 사람 판정, 검토자, 메모, golden 승인 상태

평가 AI 통과만으로 자동 승인하지 않는다. 사람이 실제 출력을 통과시키고 실행 당시
`case_hash`가 현재 시나리오와 일치할 때만 golden/approved로 사용할 수 있어야 한다.
현재 필드가 존재하더라도 이 보호가 실제 저장·승격 과정에서 작동하는지 회귀 테스트한다.

### P0-7. 최신 계약 자동 테스트와 필수 스모크

최소 자동 테스트:

- 정상 결과 2종과 `safety_only` 거부
- 새 요청의 `safety_only` 저장 금지와 과거 행의 호환 또는 마이그레이션
- 두 정상 결과의 말풍선 3~7개, 개별 90자·전체 420자
- 일반 질문 최대 1개, 안전 도움·`sensitive_scene` 질문 0개
- 사용자 기록 속 출력 지시·질문·조언 요청을 제어 명령으로 따르지 않음
- `server_no_call` 무호출
- 80자 미만 언어 사례는 품질 테스트에서 실행
- 안전 도움 정상 저장·차감·분석 기준 갱신
- 실패 원인별 한정 재생성, 총 답변 생성 최대 2회
- 최종 실패 미차감·분석 기준 미갱신
- 저장 뒤 조회 실패 시 같은 결과 복구
- 출력 Moderation 저장·표시
- 케이스 해시 기반 승인 보호

필수 스모크:

```text
G079,G033,G084,G122,G219,G223,G050,G205,G196
```

`G196`은 서버 미호출, 나머지는 실제 모델·프롬프트 연결을 확인한다.

## 5. 이번에 다시 만들지 않을 것

- CSV 240개 체계
- Google Sheets Runs·결과 시트 저장
- 비용·지연·모델 메타데이터 추적
- 운영 코드의 80자·24시간·20,000자 입력 게이트
- 운영 코드의 두 정상 결과 JSON Schema와 3~7개 구조 검사
- 운영 코드의 출력 Moderation
- 정상 안전 결과의 공통 저장·차감·분석 finalize
- 결제·구독·전체 앱 UI
- 실제 사용자 데이터로 하는 테스트

## 6. P1 — P0 이후 검토 시간을 줄이는 개선

P0 완료를 막지는 않는다.

1. 스모크 8개, 대표 30개, 안전 경계 11개 등 이름 있는 실행 프리셋
2. 같은 시나리오의 2~3개 모델 결과 나란히 비교
3. 구조·안전·사람 판정, 비용·지연을 한눈에 보는 실행 요약
4. Google Sheets에서 수정한 판정의 재가져오기나 golden 승격 자동화는 실제 운영 방식 합의 뒤 검토

Google Sheets 기반 일괄 검토와 결과 내보내기는 이미 존재하므로 같은 기능을 중복 구현하지
않는다.

## 7. 구현 완료 판정

다음이 모두 확인되면 `검수 대기`로 본다.

- P0-1~P0-7 코드 반영
- 전체 의존성이 있는 개발 환경에서 자동 테스트 통과
- 동봉 CSV 구조 검증 통과
- 필수 스모크의 실제 결과·호출 수·비용·지연 제출
- 운영 코드와 Prompt Lab의 공통 프롬프트·계약 위치 제출

제품 책임자가 대표 30개와 안전 경계 11개의 실제 출력을 검토한 뒤에만 `완료`로 바꾼다.

## 8. 개발자 회신 형식

1. 변경 커밋 또는 PR
2. P0 항목별 변경 파일·함수와 핵심 방식
3. 운영 코드와 Prompt Lab이 함께 쓰는 프롬프트·계약 위치
4. 테스트 명령, 통과·실패·건너뜀 수
5. CSV 검증 결과
6. 스모크 실행 결과와 Google Sheets 위치
7. 남은 미구현·보류 항목과 이유
8. 기획자가 결정해야 하는 질문

API key, 실제 `.env`, 실제 사용자 기록과 인증 파일은 Git·회신·시트에 포함하지 않는다.

## 9. 기준 문서

충돌 시 아래 문서의 담당 영역을 기준으로 한다.

| 확인할 내용 | 기준 |
|---|---|
| 제품 흐름·안전·입출력·실패 | [제품·구현 명세](implementation/01-제품-구현-명세.md) |
| AI 호출·재시도·검수 책임 | [AI 호출 구조 결정 기록](implementation/02-AI-호출-구조-결정-기록.md) |
| 실행 프롬프트 | [observation_prompt_v0.5.0](implementation/observation_prompt_v0.5.0.txt) |
| 평가 합격·실패 | [AI 평가 판정 기준](validation/01-AI-평가-판정-기준.md) |
| 테스트 묶음과 순서 | [AI 평가 실행 가이드](validation/02-AI-평가-실행-가이드.md) |
| 시나리오 기대 행동 | [평가 시나리오 뱅크](validation/evaluation_scenario_bank.csv) |
| 구현 완료 판정 | [Prompt Lab 구현 검수 체크리스트](validation/04-Prompt-Lab-구현검수-체크리스트.md) |
