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
