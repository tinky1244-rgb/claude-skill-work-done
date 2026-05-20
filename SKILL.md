---
name: work-done
description: |
  Use this skill whenever the user wants to save, record, summarize, or log what they
  worked on in the current Claude Code session. Triggers include the slash command
  /work_done, and natural-language phrases like "정리해줘", "오늘 한 일 정리", "업무 기록
  남겨줘", "지금까지 한 거 저장", "work log", "session log", or any request to persist
  what was just accomplished. Generates a structured Korean markdown entry
  (한 일 / 왜 / 의사결정 / 다음) as an independent session file inside
  ~/work-log/_sessions/. Session files are later consolidated into a daily file by
  the companion /day_merge skill. Always invoke this skill when the user wants to
  capture or persist their work, even if they don't explicitly say "work_done" —
  undertriggering this skill leaves the user's work history incomplete, which
  defeats the whole purpose.
---

# work-done — 업무 기록 자동 생성 (세션 파일)

사용자가 Claude Code에서 수행한 작업을 **한 일 / 왜 / 의사결정 / 다음** 구조의 마크다운으로 정리해 **세션 단위 파일**로 저장한다. 같은 날 여러 세션·여러 환경(CLI/VS Code/앱)에서 호출해도 각자 자기 파일에만 쓰므로 절대 충돌하지 않는다.

세션 파일들은 나중에 사용자가 `/day_merge`를 호출하면 그 날의 통합본(`YYYY-MM-DD.md`)으로 합쳐진다.

이 스킬은 사용자가 명시적으로 호출한다 (예: `/work_done`, "정리해줘"). 자동 트리거하지 않는다.

---

## 핵심 원칙

- **사용자의 시간을 아끼는 게 목표** — 사용자가 직접 정리하지 않아도 되도록 Claude가 현재 대화 컨텍스트를 보고 알아서 초안을 작성한다.
- **충돌 없는 동시 호출** — 각 호출이 자기만의 세션 파일을 생성하므로 여러 세션에서 동시에 호출해도 데이터 손실이 발생하지 않는다.
- **나중에 검색·복기·상관관계 발견에 쓰일 자산** — frontmatter의 `topics`와 `projects`가 핵심.
- **OS 중립** — 사용자의 홈 디렉토리(`$env:USERPROFILE` on Windows, `$HOME` on macOS/Linux) 기준으로 동작.

---

## 1단계: 작업 디렉토리 해결

```
1. 설정 파일 확인:
   - 경로: <HOME>/.claude/work-log.config.json
   - 존재하면 JSON 파싱해서 "path" 키의 값을 work-log 디렉토리로 사용
   - 없으면 기본값 사용: <HOME>/work-log/

2. 다음 폴더가 없으면 첫 실행으로 간주하고 모두 생성:
   - <work-log>/
   - <work-log>/_sessions/    ← 세션 파일 임시 저장소
   - <work-log>/_archive/     ← /day_merge 후 원본 보관
   - <work-log>/_index/       ← 태그 사전 + 주제 인덱스

3. 첫 실행이면:
   - <work-log>/README.md 생성 (아래 "첫 실행 README 템플릿" 참고)
   - <work-log>/_index/tags.md 생성 (`references/tags-seed.md` 내용 그대로 복사)
   - 사용자에게 알림:
     "처음 사용이네요. 업무 기록을 <work-log>에 저장합니다.
      다른 경로 원하시면 알려주세요. 그대로 진행할까요?"
   - 사용자가 다른 경로 지정하면:
     - <HOME>/.claude/work-log.config.json 생성
     - {"path": "<지정한 경로>"} 저장
     - 새 경로로 재시작
```

OS 분기:
- Windows: `$env:USERPROFILE`
- macOS/Linux: `$HOME`

---

## 2단계: 세션 식별자 생성

각 호출마다 고유한 세션 파일을 만든다.

```
1. 현재 날짜 (YYYY-MM-DD) — 시스템 currentDate 사용
2. 현재 시각 (HHMM) — 호출 시점
3. 세션 ID — 4자리 랜덤 16진수 (예: a3f2, b9c1, d4e7)
   PowerShell: -join ((0..3) | ForEach-Object { '{0:x}' -f (Get-Random -Max 16) })
   Bash:       printf '%04x' $((RANDOM % 65536))

4. 최종 파일 경로:
   <work-log>/_sessions/YYYY-MM-DD__<세션ID>_HHMM.md
   예: ~/work-log/_sessions/2026-05-18__a3f2_1430.md
```

`_sessions/`에 이미 같은 파일명이 있으면 (드물지만) ID를 재생성한다.

---

## 3단계: 현재 대화에서 초안 작성

현재 대화 컨텍스트(이 대화 전체)를 보고 다음을 추출한다:

- **한 일 (What):** 실제로 수행한 작업·변경·산출물. 동사로 시작하는 불릿. 파일명·함수명 등 구체적인 식별자를 포함.
- **왜 (Why):** 그 작업이 필요했던 동기, 해결하려던 문제, 사용자가 명시한 맥락.
- **의사결정·트레이드오프:** 두 개 이상의 대안 중 무엇을 골랐고 왜 그랬는지. 사용자와 의논한 선택을 우선 기록.
- **다음 (Next):** 미완료 작업, 후속으로 할 일, 사용자가 미루기로 한 것.

추가로 추론:
- **topics:** 작업의 주제 태그 (예: `SBR`, `python-script`, `refactor`). 2-5개.
- **projects:** 현재 작업 폴더 경로에서 추론. 환경 정보의 "Primary working directory" 참고.

---

## 3.5단계: 태그 사전 정규화

3단계에서 추론한 태그를 `<work-log>/_index/tags.md` 사전과 대조해 일관성을 맞춘다.

```
1. <work-log>/_index/tags.md를 Read.
2. 사전을 파싱: {표준명: [동의어 리스트], category}

3. 추론된 각 태그에 대해:
   a. 사전의 표준명·동의어와 정확 매칭 → 표준명으로 자동 변환 (조용히)
   b. 퍼지 매칭 (편집 거리 ≤ 2 또는 부분 문자열) → 사용자에게 통합 여부 확인
   c. 매칭 실패 → 신규 태그. "카테고리는? [공정/작업유형/프로젝트/도구/기타]" 물어봄

4. 정규화 결과를 사전에 반영:
   - 신규 표준명 추가: 해당 카테고리 끝에 한 줄
   - 신규 동의어 추가: 기존 줄 괄호 안에 추가
```

사전 손상 시 정규화 단계 건너뛰고 작성은 진행. 사용자에게 알림.

자세한 매칭 규칙은 `references/format.md`의 "태그 사전" 섹션 참고.

---

## 4단계: 사용자 확인

초안을 사용자에게 보여주고 추가/수정 의사를 묻는다:

```
다음 내용으로 세션 파일에 저장할게요.

📌 한 일
- ...

🎯 왜
- ...

⚖️ 의사결정·트레이드오프
- ...

➡️ 다음
- ...

🏷️ topics: [...]
📁 projects: [...]

💾 저장 위치: ~/work-log/_sessions/YYYY-MM-DD__<ID>_HHMM.md
```

`/work_done --quick` 또는 "그냥 저장" → 이 단계 스킵.

---

## 5단계: 세션 파일 작성

`<work-log>/_sessions/YYYY-MM-DD__<ID>_HHMM.md`에 Write:

```markdown
---
date: YYYY-MM-DD
session_id: <ID>
start_time: HH:MM
topics: [태그1, 태그2]
projects: [프로젝트1]
---

# <YYYY-MM-DD HH:MM> <세션 제목>

## 한 일 (What)
- ...

## 왜 (Why)
- ...

## 의사결정·트레이드오프
- ...

## 다음 (Next)
- ...
```

세션 제목은 한 일의 핵심을 한 줄로.

**주의:** 절대로 기존 파일에 append하지 않는다. 이 단계는 새 파일 하나를 작성하는 것일 뿐. `_sessions/` 안에 같은 세션 ID·시각이 있으면 ID 재생성.

---

## 6단계: 결과 보고

```
✓ 세션 파일 저장: _sessions/YYYY-MM-DD__<ID>_HHMM.md

📊 오늘 _sessions/에 N개 세션 누적 중.
```

`N`은 `Glob`으로 `<work-log>/_sessions/YYYY-MM-DD__*.md` 개수.

파일 경로는 markdown 링크로.

---

## 7단계: 인덱스 자동 갱신 (선택)

> ℹ️ 이 단계는 자매 스킬 [`rebuild-index`](https://github.com/tinky1244-rgb/claude-skill-rebuild-index)가 설치된 경우에만 동작한다. 미설치 시 조용히 스킵 — 에러를 발생시키지 않는다.
> `/day_merge` 통합 흐름을 선호하면 본 단계를 통째로 삭제해도 무방.

6단계 보고가 끝난 직후, `rebuild-index` 스킬을 자동으로 호출해 `_index/topics.md`를 최신 상태로 갱신한다.

```
Skill 도구 호출:
  skill: rebuild-index
  args: (없음 — 전체 재생성)
```

호출 트리거:
- 기본: 5단계 Write 성공 → 6단계 출력 → 7단계 자동 호출
- 사용자가 `/work_done --no-index`로 호출한 경우 → 7단계 스킵
- `rebuild-index` 스킬 미설치 → 7단계 조용히 스킵 (Skill 도구 호출 실패를 silent fallback)
- 5단계 실패(파일 작성 안 됨) → 7단계 스킵

rebuild-index의 리포트는 6단계 출력 뒤에 이어서 사용자에게 표시된다. 두 리포트가 하나의 응답으로 합쳐지는 형태.

이렇게 분리하는 이유:
- work_done은 가볍게 유지 — 한 번에 한 파일만 책임
- 인덱스 통합은 멱등하게 따로 — 언제든 안전하게 재실행 가능
- rebuild-index의 자가치유(`tags.md` 변경의 소급 적용) 효과를 매 호출마다 누리기

---

## 토픽 인덱스 작성권

`/work_done`은 `_index/topics.md`를 **직접 손대지 않는다**. 갱신은 7단계에서 호출하는 `rebuild-index` 스킬의 단독 책임. 본 스킬은:
- `_sessions/<신규>.md` 생성 (5단계)
- `_index/tags.md` 갱신 (3.5단계 — 신규 태그/동의어 추가)

까지만 담당한다.

---

## 첫 실행 README 템플릿

`<work-log>/README.md`에 다음을 작성:

```markdown
# 업무 기록 (Work Log)

이 폴더는 `/work_done`과 `/day_merge` 스킬이 자동으로 관리합니다.

## 구조

- `_sessions/` — 아직 통합되지 않은 개별 세션 파일들 (`/work_done`의 출력)
- `YYYY-MM-DD.md` — 그 날의 통합본 (`/day_merge`의 출력)
- `_archive/YYYY-MM-DD/` — 통합 후 원본 세션 파일 보관소
- `_index/topics.md` — 주제 태그별 자동 인덱스 (편집 금지, 덮어쓰기됨)
- `_index/tags.md` — 태그 사전 (편집 가능)

## 사용법

매 세션 끝나기 전 (또는 잠시 멈출 때):
- `/work_done` — 현재 대화를 정리해 세션 파일로 저장
- `/work_done --quick` — 확인 없이 바로 저장
- "정리해줘", "오늘 한 일 정리" 등 자연어로도 호출 가능

하루 마무리 (퇴근 전 / 자기 전 / 주말):
- `/day_merge` — 오늘자 세션 파일들을 통합본으로 합치고 인덱스 갱신
- `/day_merge 2026-05-17` — 특정 날짜만 통합 (지난 날 미통합 정리 시)

## 회수하기

나중에 과거 작업을 찾으려면 Claude에게 자연어로 물어보세요:
- "지난 주에 SBR 관련해서 뭐 했지?"
- "WAI-Design 프로젝트에서 의사결정 했던 거 모아줘"

Claude가 `_index/topics.md`와 통합본들을 읽어 답합니다.

## 여러 PC에서 사용 / 백업

이 폴더 전체를 private GitHub 레포로 관리하면 사무실·집 등 여러 PC에서 같은 기록을 공유할 수 있습니다. 별도 도구·스킬 불필요 — Claude한테 자연어로 시키면 됩니다.

**최초 셋업 (한 번):** Claude한테 "이 ~/work-log 폴더를 private GitHub 레포로 푸시해줘"

**다른 PC 시작:** "그 레포 ~/work-log에 클론해줘"

**작업 끝:** /day_merge 후 "git 커밋 푸시해줘"

GitHub이 부담스러우면 OneDrive/Dropbox 폴더 안에 두는 것도 동일하게 동작 (경로는 `~/.claude/work-log.config.json`에서 변경).

⚠️ 어느 방식이든 회사 데이터 정책 먼저 확인.
```

---

## 엣지 케이스

- **빈 대화에서 호출:** "정리할 작업 내용이 없어 보입니다. 무엇을 기록할지 알려주세요."
- **사용자가 topics 명시:** `/work_done #SBR #refactor` → 추론값 대신 사용자 지정값 사용. 새 태그는 카테고리만 한 번 물어봄.
- **자정 넘긴 작업:** 호출 시점의 날짜에 기록 (사용자가 "어제 작업으로" 요청 시만 예외).
- **설정 파일 손상:** JSON 파싱 실패 시 기본 경로로 폴백 + 사용자 알림.
- **동시 호출:** 각 호출이 고유 세션 ID로 다른 파일을 작성하므로 영향 없음.

---

## 참고 파일

- `references/format.md` — 세션 파일 / 통합본 / 태그 사전의 상세 템플릿과 예시
- `references/tags-seed.md` — 첫 실행 시 `_index/tags.md`에 복사할 초기 태그 사전
- 자매 스킬 `day-merge`가 통합본 생성과 인덱스 갱신을 담당. 본 스킬은 세션 파일 생성까지만.
