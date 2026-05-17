# 태그 사전 시드

`/work_done` 스킬이 처음 실행될 때 이 파일의 **본문 전체**를 `<work-log>/_index/tags.md`로 그대로 복사한다. 사용자 도메인(수처리·환경공학·Claude Code 자동화)을 미리 반영한 초기 태그 셋이다.

이 파일 자체는 시드일 뿐이고, 실제 사전은 `<work-log>/_index/tags.md`에서 자라난다. 시드를 수정해도 이미 생성된 사전에는 영향이 없다 (덮어쓰지 않음).

---

## 복사할 본문 시작

```markdown
# 태그 사전

> 이 파일은 `/work_done` 스킬이 자동 유지합니다.
> 직접 편집해도 됩니다 — 다음 호출 시 그대로 사용됩니다.
> 형식: `- <표준명> (<동의어1>, <동의어2>, ...)`
> 동의어가 없으면 괄호 생략 가능.

## 공정
- SBR (sbr, SBR-process, 연속회분식, Sequencing-Batch-Reactor)
- AMX (anammox, Anammox, 아나목스)
- vDAF (vdaf, vacuum-DAF)
- Partial-Nitritation (PN, 부분질산화, partial-nitritation)
- Equalization (균등조, 유량조정조, pH-tank)
- Dehydrator (탈수기, dehydrator)
- Intermediate-Tank (중간조)
- Sludge-Storage-Tank (슬러지저장조)
- Treatment-Tank (처리조)
- Screenings (스크린, integrated-screenings)

## 작업유형
- refactor (리팩토링, refactoring)
- bug-fix (버그수정, bugfix, fix)
- docs (문서, documentation, 문서화)
- review (검토, code-review)
- design (설계)
- 계산서 (calculation-sheet)
- 회의록 (meeting-notes)
- 분석 (analysis)
- 자동화 (automation)
- 테스트 (test, testing)

## 프로젝트
- WAI-Design (WAI, WAI design)
- Generative-Water-Design (GWD, Generative)
- work-done-skill (작업 기록 스킬 자체)

## 도구
- python
- pandas
- numpy
- excel (xlsx)
- wai (wai-lib)
- claude-code (claude)
- git
- github

## 기타
```

## 복사할 본문 끝

---

## 사용자에게 안내할 점

첫 실행 후 사용자가 `_index/tags.md`를 열어보면 위 시드 그대로 들어가 있다. 사용 중 자연스럽게 자기 도메인에 맞게 자라남:
- 새 공정 등장 → `## 공정` 섹션에 자동 추가
- "DRACO" 같은 새 시스템 → 사용자가 카테고리만 골라주면 추가
- 동의어 변경·항목 삭제 등 정리는 사용자가 직접 편집 가능
