---
name: work-done
description: |
  Use this skill whenever the user wants to save, record, summarize, or log what they
  worked on in the current Claude Code session. Triggers include the slash command
  /work_done, and natural-language phrases like "정리해줘", "오늘 한 일 정리", "업무 기록
  남겨줘", "지금까지 한 거 저장", "work log", "session log", or any request to persist
  what was just accomplished. Generates a structured Korean markdown entry
  (한 일 / 왜 / 의사결정 / 다음) inside the user's work-log directory
  (default ~/work-log/) and auto-maintains a topic index. Always invoke this skill
  when the user wants to capture or persist their work, even if they don't explicitly
  say "work_done" — undertriggering this skill leaves the user's work history
  incomplete, which defeats the whole purpose.
---

# work-done — 업무 기록 자동 생성

사용자가 Claude Code에서 수행한 작업을 **한 일 / 왜 / 의사결정 / 다음** 구조의 마크다운으로 정리해 일자별 파일에 누적 저장하고, 주제 인덱스를 자동 유지한다.

이 스킬은 사용자가 명시적으로 호출한다 (예: `/work_done`, "정리해줘"). 자동 트리거하지 않는다.

---

## 핵심 원칙

- **사용자의 시간을 아끼는 게 목표** — 사용자가 직접 정리하지 않아도 되도록 Claude가 현재 대화 컨텍스트를 보고 알아서 초안을 작성한다.
- **나중에 검색·복기·상관관계 발견에 쓰일 자산** — frontmatter의 `topics`와 `projects`가 핵심. 추후 "지난 3개월 SBR 관련 작업 모아줘" 같은 질의에 답하는 근거가 된다.
- **OS 중립** — 사용자의 홈 디렉토리(`$env:USERPROFILE` on Windows, `$HOME` on macOS/Linux) 기준으로 동작한다. 절대 경로 하드코딩 금지.

---

## 1단계: 작업 디렉토리 해결

```
1. 설정 파일 확인:
   - 경로: <HOME>/.claude/work-log.config.json  (HOME = $env:USERPROFILE 또는 $HOME)
   - 존재하면 JSON 파싱해서 "path" 키의 값을 work-log 디렉토리로 사용
   - 없으면 기본값 사용: <HOME>/work-log/

2. 디렉토리가 없으면 첫 실행으로 간주:
   - <work-log>/ 및 <work-log>/_index/ 폴더 생성
   - <work-log>/README.md 생성 (아래 "첫 실행 README 템플릿" 참고)
   - 사용자에게 알림:
     "처음 사용이네요. 업무 기록을 <work-log>에 저장합니다.
      다른 경로 원하시면 알려주세요. 그대로 진행할까요?"
   - 사용자가 다른 경로 지정하면:
     - <HOME>/.claude/work-log.config.json 생성
     - {"path": "<지정한 경로>"} 저장
     - 새 경로로 재시작
```

OS 분기:
- Windows (PowerShell): `$env:USERPROFILE`
- macOS/Linux (Bash): `$HOME`

현재 OS는 환경 정보에서 확인 가능 (예: `Platform: win32` → Windows).

---

## 2단계: 오늘 날짜 파일 확인

```
1. 현재 날짜를 YYYY-MM-DD 형식으로 얻는다 (시스템에서 제공된 currentDate 활용 가능)
2. 파일 경로: <work-log>/YYYY-MM-DD.md
3. 파일이 있으면 Read로 읽고 frontmatter 파싱
   - 기존 topics, projects, sessions 카운트 보존
4. 없으면 새로 생성할 준비
```

---

## 3단계: 현재 대화에서 초안 작성

현재 대화 컨텍스트(이 대화 전체)를 보고 다음을 추출한다:

- **한 일 (What):** 실제로 수행한 작업·변경·산출물. 동사로 시작하는 불릿. 파일명·함수명 등 구체적인 식별자를 포함.
- **왜 (Why):** 그 작업이 필요했던 동기, 해결하려던 문제, 사용자가 명시한 맥락.
- **의사결정·트레이드오프:** 두 개 이상의 대안 중 무엇을 골랐고 왜 그랬는지. 사용자와 의논한 선택이 있으면 그것을 우선 기록.
- **다음 (Next):** 미완료 작업, 후속으로 할 일, 사용자가 미루기로 한 것.

추가로 추론:
- **topics:** 작업의 주제 태그 (예: `SBR`, `python-script`, `refactor`, `bug-fix`, `docs`). 짧고 검색하기 쉬운 영문/한글 키워드 2-5개.
- **projects:** 현재 작업 폴더 경로에서 추론 (예: 경로에 `1. WAI Design`이 있으면 `WAI-Design`). 환경 정보의 "Primary working directory"를 참고.

---

## 4단계: 사용자 확인

초안을 사용자에게 보여주고 추가/수정 의사를 묻는다. 한 번에 보여주기 위해 다음과 같이 간결하게:

```
다음 내용으로 저장할게요. 추가하거나 고칠 것 있나요?

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
```

사용자가 빠르게 저장하고 싶을 때:
- `/work_done --quick` 또는 "그냥 저장" → 이 단계 스킵하고 바로 저장

---

## 5단계: 파일 저장

### 새 파일인 경우

`<work-log>/YYYY-MM-DD.md`에 다음 구조로 Write:

```markdown
---
date: YYYY-MM-DD
topics: [태그1, 태그2]
projects: [프로젝트1]
sessions: 1
---

# YYYY-MM-DD 업무 기록

## [HH:MM] <세션 제목>

### 한 일 (What)
- ...

### 왜 (Why)
- ...

### 의사결정·트레이드오프
- ...

### 다음 (Next)
- ...
```

세션 제목은 한 일의 핵심을 한 줄로 (예: "SBR 공정 스크립트 r2 리팩토링").

`HH:MM`은 현재 시각.

### 기존 파일에 append하는 경우

`Read`로 기존 내용을 읽은 뒤 `Edit`로:
1. frontmatter의 `topics`·`projects`를 합집합으로 업데이트 (중복 제거)
2. `sessions` 값을 +1
3. 파일 끝에 새 `## [HH:MM] <제목>` 섹션 추가 (구분선 `---` 사이에 두기)

자세한 포맷 예시는 `references/format.md` 참고.

---

## 6단계: 주제 인덱스 재생성

`<work-log>/_index/topics.md`를 매번 새로 만든다 (append하지 않음 — drift 방지).

절차:
1. `Glob`으로 `<work-log>/*.md` 파일 모두 찾기 (단, `README.md`와 `_index/` 안의 파일 제외)
2. 각 파일의 frontmatter만 빠르게 읽기 (`Read`의 `limit` 옵션으로 처음 ~15줄만 충분)
3. 각 파일의 첫 `## [HH:MM]` 섹션 제목을 그 날의 요약으로 사용 (여러 세션이면 첫 번째만)
4. `{topic: [{date, summary, path}]}` 형태로 집계
5. 알파벳/가나다 순으로 정렬해서 마크다운으로 작성

자세한 로직은 `references/index-rebuild.md` 참고.

---

## 7단계: 결과 보고

사용자에게 1-2줄로 보고:

```
✓ <work-log>/YYYY-MM-DD.md 에 "<세션 제목>" 추가했습니다 (오늘 N번째 세션).
✓ topics 인덱스 갱신: <변경된 태그>
```

파일 경로는 markdown 링크로 (`[YYYY-MM-DD.md](경로)`).

---

## 첫 실행 README 템플릿

`<work-log>/README.md`에 다음을 작성:

```markdown
# 업무 기록 (Work Log)

이 폴더는 `/work_done` 스킬이 자동으로 관리합니다.

## 구조

- `YYYY-MM-DD.md` — 날짜별 일일 기록. 한 일/왜/의사결정/다음 구조.
- `_index/topics.md` — 주제 태그별 자동 인덱스. 직접 수정하지 마세요 (덮어쓰기됨).
- `README.md` — 이 파일.

## 사용법

Claude Code에서:
- `/work_done` — 현재 대화를 정리해서 오늘 파일에 추가
- `/work_done --quick` — 확인 없이 바로 저장
- "정리해줘", "오늘 한 일 정리" 등 자연어로도 호출 가능

## 회수하기

나중에 과거 작업을 찾으려면 Claude에게 자연어로 물어보세요:
- "지난 주에 SBR 관련해서 뭐 했지?"
- "WAI-Design 프로젝트에서 의사결정 했던 거 모아줘"

Claude가 `topics.md` 인덱스와 일일 파일들을 읽어 답합니다.

## 백업

이 폴더 전체를 드롭박스/원드라이브/git 등에 넣으면 자동 백업됩니다.
```

---

## 엣지 케이스

- **빈 대화에서 호출:** "정리할 작업 내용이 없어 보입니다. 무엇을 기록할지 알려주세요." 라고 묻기.
- **사용자가 topics 명시:** 사용자가 `/work_done #SBR #refactor`처럼 태그를 주면 추론값 대신 그것을 사용.
- **여러 날에 걸친 세션:** 자정을 넘긴 작업이라도 호출 시점의 날짜에 기록 (사용자가 "어제 작업으로 기록" 요청 시만 예외).
- **설정 파일 손상:** JSON 파싱 실패 시 기본 경로로 폴백하고 사용자에게 알림.

---

## 참고 파일

- `references/format.md` — 일일 파일 / 주제 인덱스의 상세 템플릿과 예시
- `references/index-rebuild.md` — 주제 인덱스 재생성의 단계별 의사 코드
