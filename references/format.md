# 파일 포맷 상세 (work-done + day-merge)

work-log 폴더 내에서 사용되는 모든 파일 포맷을 한곳에 정리.

```
~/work-log/
├── README.md
├── _sessions/                  ← /work_done의 출력 (세션 파일)
├── 2026-05-18.md               ← /day_merge의 출력 (일일 통합본)
├── _archive/                   ← /day_merge 후 원본 세션 보관
└── _index/
    ├── topics.md               ← /day_merge가 갱신 (주제 인덱스)
    └── tags.md                 ← /work_done이 갱신 (태그 사전)
```

---

## 세션 파일 (`_sessions/YYYY-MM-DD__<ID>_HHMM.md`)

`/work_done` 한 번이 만드는 단위. 한 세션 = 한 파일.

### 파일명 규칙

```
YYYY-MM-DD__<4-hex-ID>_HHMM.md

예: 2026-05-18__a3f2_1430.md
   └─ 날짜 = 2026-05-18
   └─ 세션 ID = a3f2 (랜덤 4자리 16진수)
   └─ 시작 시각 = 14:30
```

이중 언더스코어(`__`)로 날짜와 ID-시각을 구분 — Glob·파싱 시 명확.

### Frontmatter

```yaml
---
date: 2026-05-18
session_id: a3f2
start_time: 14:30
topics: [SBR, python-script]
projects: [WAI-Design]
---
```

- `date` — ISO 형식
- `session_id` — 4자리 hex
- `start_time` — `HH:MM` (24시간제)
- `topics`, `projects` — 배열

### 본문

```markdown
# 2026-05-18 14:30 — <세션 제목>

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

### 풀 예시

```markdown
---
date: 2026-05-18
session_id: a3f2
start_time: 14:30
topics: [SBR, python-script, refactor]
projects: [WAI-Design]
---

# 2026-05-18 14:30 — SBR 공정 스크립트 r2 리팩토링

## 한 일 (What)
- `251117_Process_SBR_jyj.py`의 가동시간 계산 로직을 `utils/operation_time.py`로 분리
- 공통 유틸 함수 3개 추출

## 왜 (Why)
- AMX 공정 스크립트도 같은 로직을 중복 사용 중 → DRY 위반

## 의사결정·트레이드오프
- 모듈 분리 vs 인라인 유지: 분리 선택 (재사용 + 단위 테스트 가능)

## 다음 (Next)
- AMX 공정 스크립트도 동일 리팩토링
```

---

## 일일 통합본 (`YYYY-MM-DD.md`)

`/day_merge`의 출력. 그 날의 모든 세션 파일을 시간순으로 병합한 결과.

### Frontmatter

```yaml
---
date: 2026-05-18
topics: [SBR, python-script, refactor, 회의록]   # 모든 세션의 union
projects: [WAI-Design, Generative-Water-Design]  # 모든 세션의 union
sessions: 3                                       # 통합된 세션 파일 개수
merged_at: 2026-05-18 23:05                       # 마지막 /day_merge 시각
---
```

`merged_at`은 멱등 merge 시 갱신.

### 본문 구조

```markdown
# YYYY-MM-DD 업무 기록

## [HH:MM] <세션 1 제목>

### 한 일 (What)
- ...

### 왜 (Why)
- ...

### 의사결정·트레이드오프
- ...

### 다음 (Next)
- ...

---

## [HH:MM] <세션 2 제목>
...
```

세션 사이 구분선 `---` 한 줄. 시간순 정렬.

### 풀 예시

```markdown
---
date: 2026-05-18
topics: [SBR, python-script, refactor, 회의록]
projects: [WAI-Design]
sessions: 2
merged_at: 2026-05-18 23:05
---

# 2026-05-18 업무 기록

## [14:30] SBR 공정 스크립트 r2 리팩토링

### 한 일 (What)
- ...

### 왜 (Why)
- ...

---

## [21:00] 팀 회의록 정리

### 한 일 (What)
- ...
```

---

## 주제 인덱스 (`_index/topics.md`)

`/day_merge`가 매번 통합본들을 스캔해서 재생성. append 아님 — 전체 재생성으로 drift 방지.

### 구조

```markdown
# 주제별 인덱스

> 이 파일은 `/day_merge` 스킬이 자동 생성합니다. 직접 수정해도 다음 호출 시 덮어쓰여집니다.

마지막 갱신: YYYY-MM-DD HH:MM

## <topic 1>
- [YYYY-MM-DD](../YYYY-MM-DD.md) — 그 날의 첫 세션 제목
- [YYYY-MM-DD](../YYYY-MM-DD.md) — 그 날의 첫 세션 제목

## <topic 2>
- ...

---

## 📁 프로젝트별

### <project 1>
- [YYYY-MM-DD](../YYYY-MM-DD.md) — 첫 세션 제목
```

### 정렬 규칙

- topics와 projects 각각 알파벳·가나다 순
- 각 topic 안의 항목은 날짜 내림차순 (최신 먼저)

---

## 태그 사전 (`_index/tags.md`)

`/work_done`이 3.5단계 정규화에서 읽고 쓴다. 사용자도 직접 편집 가능.

### 구조

```markdown
# 태그 사전

> 이 파일은 `/work_done` 스킬이 자동 유지합니다.
> 직접 편집해도 됩니다 — 다음 호출 시 그대로 사용됩니다.
> 형식: `- <표준명> (<동의어1>, <동의어2>, ...)`
> 동의어가 없으면 괄호 생략 가능.

## <카테고리 1>
- 표준명 (동의어1, 동의어2, ...)
- 표준명만

## <카테고리 2>
- ...
```

### 카테고리 (고정 5개)

- **공정** — 수처리 공정 (SBR, AMX 등)
- **작업유형** — 일의 종류 (refactor, docs 등)
- **프로젝트** — 사용자의 프로젝트 (WAI-Design 등)
- **도구** — 라이브러리·툴 (python, excel 등)
- **기타**

### 한 줄 파싱 규칙

```
- SBR (sbr, SBR-process, 연속회분식)
  └ 표준명: "SBR"
  └ 동의어: ["sbr", "SBR-process", "연속회분식"]

- python
  └ 표준명: "python"
  └ 동의어: [] (괄호 없음)
```

### 매칭 우선순위 (3.5단계에서 사용)

1. 표준명 정확 일치 (대소문자 무시)
2. 동의어 정확 일치 (대소문자 무시)
3. 퍼지 매칭 (편집 거리 ≤ 2 또는 부분 문자열) → 사용자 확인
4. 매칭 실패 → 신규 태그로 카테고리 묻기

### 자동 갱신

- **신규 표준명:** 해당 카테고리 끝에 `- <표준명>` 추가
- **신규 동의어:** 기존 줄 괄호 안에 추가 (괄호 없었으면 새로 만듦)
- **카테고리 이동·삭제·이름 변경:** 사용자 수동 편집

---

## 아카이브 (`_archive/YYYY-MM-DD/`)

`/day_merge`가 통합 완료 후 원본 세션 파일을 이동시키는 곳.

```
_archive/2026-05-18/
├── a3f2_1430.md      ← 원본 세션 파일 (이름에서 날짜 접두사 제거)
├── b9c1_2100.md
└── d4e7_2300.md
```

이름 규칙: `<세션ID>_HHMM.md` (날짜는 부모 폴더 이름에 들어가 있으므로 중복 제거).

복원이 필요하면 사용자가 수동으로 `_sessions/`로 옮기고 `/day_merge` 재실행.
