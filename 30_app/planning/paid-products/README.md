---
title: 페블링 유료상품 기획 기준
type: product-planning
status: baseline
last_verified: 2026-07-25
tags:
  - app
  - monetization
  - product-baseline
---

# 페블링 유료상품 기획 기준

기존 아이디어 아카이브를 유료상품 관점에서 수정·재구조화하기 전에, 실제 서비스와
개발 중 상태를 혼동하지 않기 위한 기준 문서다. 이 단계에서는 가격이나 상품을
확정하지 않는다.

## 1~3단계 결과

1. [배포 제품 기준선](01-deployed-product-baseline.md)
2. [개발·기획 차이](02-development-and-planning-gap.md)
3. [약관·개인정보 기준선](03-legal-privacy-baseline.md)
4. [첫 유료상품 출시 작업 맥락](05-first-paid-release-work-context.md)

최초 질문 목록은 [유료화 전 핵심 질문](04-next-interview-questions.md)에
보존한다. 현재의 열린 질문과 채택된 답은
[페블링 제품 컨텍스트](../product-context.md)를 우선한다.

## 외부 레퍼런스 근거

[유사 앱 결정 근거](../../references/app-reference-research/index.md)는 Q-001,
Q-003~Q-006과 무료·유료 경계에 필요한 외부 사례를 결정 단위로 보존한다.
이 묶음의 임시 권고는 새로운 제품 기준선이 아니며, 김세인이 채택한 결정만
[페블링 제품 컨텍스트](../product-context.md)에 반영한다.

## 상태를 판단할 때의 원본 우선순위

| 판단하려는 것 | 우선 원본 |
| --- | --- |
| 사용자가 지금 받을 수 있는 기능 | App Store 공개 버전, `pebbling-expo/main` |
| 개발 브랜치에 구현된 기능 | `pebbling-expo/origin/dev` |
| `속마음 엿보기`가 앞으로 따라야 할 계약 | `planing/main`의 최신 구현 명세와 프롬프트 |
| UI 탐색과 재활용 가능한 아이디어 | Figma 원본과 `30_app/ideation` 분석 |
| 사용자에게 현재 고지된 내용 | 공개 이용약관·개인정보처리방침 |

한 원본이 다른 원본을 자동으로 덮어쓰지 않는다. 예를 들어 테스트를 통과한 개발
코드라도 최신 기획 계약과 다르면, “구현 완료”가 아니라 “구버전 계약 구현”으로
분류한다.

## 확인 시점과 기준 커밋

- App Store 공개 버전: `1.2.6` (2026-07-25 확인)
- 앱 기준 저장소 `main`: `0dac89b` (`app.config.ts` 버전 `1.2.8`, 제출 준비 커밋)
- 앱 개발 브랜치 `origin/dev`: `0805e98`
- 기획 저장소 `planing/main`: `9adadfb`
- 이용약관·개인정보처리방침 시행일: `2026-07-15`

App Store의 공개 버전과 로컬 `main`의 버전이 다르므로 `1.2.8`을 이미 배포된
버전으로 단정하지 않는다.

## 이번 단계에서 하지 않은 것

- 기존 Figma 아이디어에 유료/무료 등급을 부여하지 않았다.
- 가격, 구독 주기, 무료 체험과 일일 횟수를 확정하지 않았다.
- 앱 코드와 외부 정책 문서를 수정하지 않았다.
- 법률 적합성을 판정하지 않았다. 문서와 구현의 정합성 위험만 표시했다.
