---
type: implementation-change-request
title: 속마음 PII 최소 치환 지시서
status: ready-for-developer
date: 2026-08-14
audience: [developer, product_manager]
source_repository: https://github.com/hux2/pebbling-expo.git
source_branch: dev
baseline_commit: 44bc01c4a7b75f16e867c8848a90b33c1405ecbe
scope: app-runtime-pii-minimum
companion_to: 속마음-출력계약-Moderation-저장-변경요청.md
---

# 속마음 PII 최소 치환 지시서

## 1. 변경 목적

현재 코드는 사용자 입력의 직접 식별정보가 AI 답변에 그대로 반복되면 후보를
실패시킨다. 이번에는 실패시키지 않고 해당 문자열만 치환한 뒤 같은 답변을 사용한다.
완전한 개인정보 탐지기를 만드는 작업이 아니라, 현재 검사를 저비용 안전장치로
바꾸는 최소 범위다.

```text
PII exact match 발견
→ 코드로 문자열 치환
→ 같은 답변 저장·표시
→ 재생성 없음·정상 1회 차감
```

## 2. 이번에 포함할 세 유형

| 입력과 출력에 동일하게 반복된 값 | 치환 결과 |
|---|---|
| 이메일 | `이메일 주소` |
| 한국 휴대전화번호 | `전화번호` |
| 주민·외국인등록번호형 | `개인 식별번호` |

예: `010-1234-5678로 연락했어` → `전화번호로 연락했어`

다음은 이번에 하지 않는다.

- OpenAI 전송 전 입력 마스킹
- 하이픈·공백·대소문자가 달라진 유사값 비교
- 이름·주소·계좌·카드·긴 숫자 추정
- 입력에 없고 출력에만 생긴 개인정보 추정
- PII 전용 DB 필드·migration·분석 지표
- PII 전용 재생성·실패 코드

## 3. 코드 변경

대상:

```text
supabase/functions/generate-venting-observation/contract.ts
```

현재 `findEchoedDirectIdentifier()`의 실패 처리를 아래처럼 바꾼다. 별도 파일이나
새 상태값은 만들지 않는다.

```ts
const PII_RULES = [
  {
    pattern: /\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b/giu,
    replacement: "이메일 주소",
  },
  {
    pattern: /\b01[016789][- ]?\d{3,4}[- ]?\d{4}\b/gu,
    replacement: "전화번호",
  },
  {
    pattern: /\b\d{6}[- ]?[1-8]\d{6}\b/gu,
    replacement: "개인 식별번호",
  },
] as const;

export function replaceEchoedPii(inputText: string, candidateText: string) {
  let replaced = candidateText;

  for (const rule of PII_RULES) {
    const inputMatches = inputText.match(rule.pattern) ?? [];

    for (const value of new Set(inputMatches)) {
      replaced = replaced.replaceAll(value, rule.replacement);
    }
  }

  return replaced;
}
```

적용 순서:

```ts
const inputText = input?.newEntries.map((entry) => entry.text).join("\n") ?? "";
const replacedText = replaceEchoedPii(inputText, text);
const parsed = parseJsonObject(stripJsonFence(replacedText));

// 이후 기존 normalize·계약 검사·저장 진행
```

즉, `parseObservationResultJson()`에서 JSON 파싱 전에 provider의 text에 한 번 적용한다.
inline Moderation 결과는 Responses API가 이미 반환한 값을 그대로 저장하며, PII
치환 때문에 Moderation이나 생성 API를 다시 호출하지 않는다.

기존 동작에서 제거할 것:

- `Output repeats a direct identifier from the input` 오류 추가
- PII 때문에 `safety_product` 실패로 분류하는 처리
- PII 때문에 AI를 다시 생성하는 처리

## 4. 저장·로그

- 최종 observation과 진단 후보에는 치환된 결과만 저장한다.
- 기존 `metrics.echoedIdentifier` 필드는 제거한다. 실제 탐지 문자열은 새 필드로
  만들거나 저장·로그하지 않는다.
- 치환 전 provider 원응답을 콘솔·Sentry/Firebase·Sheets에 남기지 않는다.

## 5. 필수 테스트

- [ ] 동일한 이메일이 반복되면 `이메일 주소`로 치환되고 재생성하지 않음
- [ ] 동일한 휴대전화번호가 반복되면 `전화번호`로 치환되고 재생성하지 않음
- [ ] 동일한 개인식별번호형이 반복되면 `개인 식별번호`로 치환되고 재생성하지 않음
- [ ] 입력과 정확히 같은 값이 없으면 기존 답변을 변경하지 않음
- [ ] 치환 결과가 정상 저장·표시되고 정확히 1회 차감됨
- [ ] 생성 결과·진단 후보·일반 로그에 실제 식별정보가 남지 않음

## 6. 완료 회신

개발자는 다음만 회신한다.

1. 변경 파일
2. 위 테스트 결과
3. PII 치환 경로에서 AI 재호출이 없다는 확인
