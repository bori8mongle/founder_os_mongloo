---
type: Handoff Index
title: Pebbling 속마음 앱 개발 인계
description: 앱 개발자가 속마음 생성 런타임 변경 요청과 제외 범위를 확인하는 시작 문서.
tags: [pebbling, ai, developer-handoff, index]
timestamp: 2026-08-13T00:00:00+09:00
status: active
audience: [developer, product_manager]
source_repository: https://github.com/hux2/pebbling-expo.git
source_branch: dev
baseline_commit: 44bc01c4a7b75f16e867c8848a90b33c1405ecbe
---

# Pebbling 속마음 앱 개발 인계

## 현우님이 읽을 문서

현재 구현 기준은 `dev@44bc01c4a7b75f16e867c8848a90b33c1405ecbe`다.

아래 문서 하나만 이번 구현 기준으로 읽는다.

1. [속마음 앱 런타임 변경 요청](속마음-출력계약-Moderation-저장-변경요청.md)

이 문서는 다음을 정의한다.

- 두 결과 유형 공통 `2~10개 / 개별 200자 / 전체 1,000자`
- 질문 개수 런타임 제한 제거
- Responses inline input/output Moderation
- flagged·Moderation error 상태여도 유효한 속마음은 표시하고 상세 결과는 기록
- 실패·flagged 후보와 검사 결과의 서버 전용 저장
- 구조·개수·길이 오류에 한정한 교정 생성 최대 1회
- 정상 안전 결과의 `속마음 + 고정 안전 도움` 표시
- 새 DB migration과 v060 finalize
- 앱·서버 수용 테스트와 UI QA

## 이번 작업에서 제외

현우님은 다음을 수정하지 않는다.

- `tools/prompt-lab/**`
- Google Sheets의 TestCases·Runs·AnswerHistory·결과 시트
- 평가 AI와 평가 프롬프트
- Prompt Lab UI·실행 로직·테스트
- 운영 프롬프트 본문·few-shot·문체
- 테스트 케이스 CSV
- Prompt Lab용 Supabase 테이블
- 실패 후보의 보관 기간·자동 삭제 정책

Prompt Lab은 제품 책임자가 별도 비공개 저장소에서 테스트·비교 전용으로 관리한다.
확정된 운영 프롬프트는 이후 별도 패키지로 전달한다.

## 문서 상태

| 문서 | 상태 | 사용 방법 |
|---|---|---|
| [속마음 앱 런타임 변경 요청](속마음-출력계약-Moderation-저장-변경요청.md) | **ACTIVE** | 이번 앱·서버·DB 구현의 단일 기준 |
| [속마음 결정적 출력 검사 P0](속마음-결정적-출력검사-P0-변경요청.md) | SUPERSEDED | 과거 논의 근거만 참고 |
| [속마음 D안 런타임 변경 요청](속마음-D안-런타임-변경요청.md) | SUPERSEDED IN PART | 배경만 참고. 충돌하면 ACTIVE 우선 |
| [속마음 운영 안전·복구 추가 수정 요청](속마음-운영-안전-복구-추가수정요청.md) | SUPERSEDED | 구현 기준으로 사용하지 않음 |
| Prompt Lab 관련 문서 | OUT OF SCOPE | 이번 현우님 작업에서 읽거나 수정할 필요 없음 |
| `implementation/`, `validation/` | REFERENCE | 배경 확인용. 수치·흐름은 ACTIVE 우선 |
| PPT·inspect 산출물 | SNAPSHOT | 갱신하거나 구현 기준으로 사용하지 않음 |

> 과거 문서의 `1~20개 / 300자 / 3,000자`, SDK 추가, standalone
> Moderation, flagged 미노출, Moderation 기반 재생성, 최대 3회 의미 생성은
> 이번 구현 기준이 아니다.

## 개발 시작 전 첫 회신

코드를 수정하기 전에 다음만 먼저 알려준다.

1. 현재 작업 브랜치와 기준 커밋
2. ACTIVE 문서 기준 current → changed 요약
3. 예상 변경 파일
4. 새 migration과 RPC 이름
5. 모호하거나 구현을 막는 항목

문서에 확정되지 않은 프롬프트 내용이나 UI를 임의로 추가하지 않는다.

## 참고 문서

필요할 때만 읽는다.

| 목적 | 참고 문서 |
|---|---|
| 기존 제품 상태와 사용자 흐름 | [제품·구현 명세](implementation/01-제품-구현-명세.md) |
| 기존 AI 호출 구조의 배경 | [AI 호출 구조 결정 기록](implementation/02-AI-호출-구조-결정-기록.md) |
| 기존 평가·안전 원칙 | [AI 평가 판정 기준](validation/01-AI-평가-판정-기준.md) |
