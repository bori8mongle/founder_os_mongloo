---
type: developer-handoff
title: VentChat 사용자 기록 시간 묶음 변경 요청
status: ready-for-developer
scope: vent-chat-ui-only
source_repository: https://github.com/hux2/pebbling-expo.git
source_branch: dev
baseline_commit: 44bc01c4a7b75f16e867c8848a90b33c1405ecbe
updated_at: 2026-08-14
---

# VentChat 사용자 기록 시간 묶음 변경 요청

## 한 문장 요약

사용자 기록은 **직전 사용자 기록과 30분 이상 떨어졌거나 날짜가 바뀐 경우에만**
새 시간 묶음을 시작하고, 그 묶음의 첫 기록 앞에 시간을 한 번 표시한다.

이 변경은 `VentChat`의 화면 계산·표시만 다룬다. 서버 API, DB 스키마,
돌멩이 생각 생성 로직은 바꾸지 않는다.

## 현재 구현에서 확인한 내용

기준 코드는 `dev@44bc01c4a7b75f16e867c8848a90b33c1405ecbe`다.

- `src/venting/chatTimeline.ts`
  - 현재는 같은 분에 연속된 사용자 기록 중 마지막 기록에만 `showTime=true`가 된다.
  - 다음 항목이 돌멩이 생각이면 현재 사용자 기록에 시간이 표시되므로, 돌멩이
    생각이 시간 표시 판단에 영향을 줄 수 있다.
- `src/screens/venting/VentChatScreen.tsx`
  - 현재 시간은 사용자 말풍선 아래에 렌더링된다.
- `src/api/venting/entries.ts`
  - 사용자 기록은 `created_at` 오름차순으로 조회된다.
- 기존 데이터에는 `createdAt`, `recordDate`, `localCreatedAt`이 이미 있다.

따라서 새 세션 필드나 타이머를 추가할 필요 없이 기존 타임라인 계산을 바꾸면 된다.

## 최종 정책

### 적용 대상

시간 묶음 계산과 시간 표시는 `entry`, 즉 사용자가 직접 작성한 기록에만 적용한다.

다음 항목은 시간을 표시하지 않고 시간 묶음 계산에서도 제외한다.

- 돌멩이의 생각(`observation`)
- 돌멩이 생각 요청·생성 완료 상태
- 로딩, 실패, 재시도 상태

이들의 시각은 필요한 경우 내부 성능·운영 데이터로만 사용한다.

### 새 시간 묶음을 시작하는 조건

현재 사용자 기록에 아래 조건 중 하나라도 해당하면 `showTime=true`다.

1. 타임라인의 첫 번째 사용자 기록이다.
2. 직전 사용자 기록과 `recordDate`가 다르다.
3. 현재 `createdAt - 직전 사용자 기록 createdAt >= 30분`이다.

그 외에는 `showTime=false`다. 정확히 30분 차이도 새 묶음이다.

비교 대상은 항상 **직전 사용자 기록**이다. 두 기록 사이에 돌멩이 생각이나 상태
항목이 있어도 무시한다.

### 날짜와 시간 표시

- 날짜 헤더와 날짜 변경 판단은 VentChat이 이미 사용하는 `recordDate`를 유지한다.
- 기록 정렬과 30분 간격 계산은 서버 저장 시각인 `createdAt`을 사용한다.
- 시간 문구는 묶음 첫 기록의 `createdAt`을 기존
  `formatVentingEntryTime()`으로 표시한다.
- 표시 형식은 기존의 `오전/오후 h:mm`을 유지한다.
- 날짜 헤더가 있으면 `날짜 헤더 → 시간 → 첫 사용자 말풍선` 순서로 표시한다.
- 시간은 묶음의 첫 사용자 말풍선 **앞에** 한 번만 표시한다.

`localCreatedAt`의 저장 방식이나 과거 기록의 시간대 정책은 이번 범위에서 바꾸지
않는다.

## 구현 지시

상수는 한 곳에서 관리한다.

```ts
export const TIME_GROUP_GAP_MINUTES = 30;
```

타이머나 별도 세션 ID는 만들지 않는다. 타임라인을 만들 때 사용자 기록만 순서대로
비교한다.

```text
previousUserEntry = null

for item in timeline ordered by createdAt:
    if item.type != entry:
        continue

    if previousUserEntry == null:
        item.showTime = true
    else if item.entry.recordDate != previousUserEntry.recordDate:
        item.showTime = true
    else if item.entry.createdAt - previousUserEntry.createdAt >= 30 minutes:
        item.showTime = true
    else:
        item.showTime = false

    previousUserEntry = item.entry
```

구현 시 다음을 지킨다.

- 입력 배열을 직접 변경하지 않는다.
- 기존 `createdAt` 오름차순을 유지한다.
- `observation`이 중간에 있어도 `previousUserEntry`를 초기화하지 않는다.
- 기록을 추가·삭제·복원하면 타임라인 재계산 결과가 즉시 반영되어야 한다.
- 첫 기록이 삭제되면 같은 묶음의 다음 사용자 기록이 새 첫 기록이 되어 시간을
  표시한다.
- `showTime` 타입과 기존 시간 포맷터는 유지해도 된다.

## 예상 변경 파일

| 파일 | 변경 내용 |
|---|---|
| `src/venting/chatTimeline.ts` | 기존 분 단위 마지막 기록 로직을 30분 단위 첫 기록 로직으로 교체 |
| `src/screens/venting/VentChatScreen.tsx` | 시간 라벨을 해당 사용자 말풍선 앞에 렌더링 |
| `tests/venting-chat-timeline.test.mjs` | 아래 경계값·돌멩이 생각·날짜 변경 테스트로 교체·추가 |

서버 API, migration, Supabase 테이블 변경은 필요하지 않다. 날짜·시간 포맷을 그대로
사용한다면 `src/venting/date.ts`도 수정할 필요가 없다.

## 필수 수용 테스트

| 입력 | 기대 결과 |
|---|---|
| 사용자 10:00 → 사용자 10:05 | 10:00 앞에만 시간 표시 |
| 사용자 10:00 → 사용자 10:29 | 10:00 앞에만 시간 표시 |
| 사용자 10:00 → 사용자 10:30 | 10:00과 10:30 앞에 시간 표시 |
| 사용자 10:00 → 돌멩이 생각 10:10 → 사용자 10:25 | 10:00 앞에만 시간 표시 |
| 사용자 10:00 → 돌멩이 생각 10:10 → 사용자 10:35 | 10:00과 10:35 앞에 시간 표시 |
| 사용자 23:50 → 다음 `recordDate`의 사용자 00:05 | 새 날짜 헤더와 00:05 시간 표시 |
| 돌멩이 생각만 있음 | 사용자 기록 시간 표시 없음 |
| 10:00, 10:05 중 10:00 기록 삭제 | 10:05가 묶음 첫 기록이 되어 시간 표시 |

추가로 기존 검증도 유지한다.

- 돌멩이 생각은 마지막 원본 사용자 기록 뒤에 정상적으로 배치된다.
- 사용자 기록 추가·수정·삭제·복원 동작은 바뀌지 않는다.
- 빈 기록 화면, 로딩, 생성 중, 실패 상태에 불필요한 시간이 표시되지 않는다.

## 완료 조건

- 위 필수 수용 테스트가 자동 테스트로 통과한다.
- iOS와 Android에서 날짜 헤더, 시간 라벨, 첫 말풍선의 간격을 확인한다.
- 시간 때문에 말풍선 편집·삭제 터치 영역이나 자동 스크롤 위치가 달라지지 않는다.
- 서버·DB·돌멩이 생각 생성 코드에 변경이 없다.
