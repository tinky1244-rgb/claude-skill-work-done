# work-done Skill

Claude Code 세션의 작업을 **세션 단위 마크다운 파일**로 자동 정리하는 스킬.

## 무엇을 하나

`/work_done` 호출 시:

1. 현재 대화 컨텍스트에서 **한 일 / 왜 / 의사결정 / 다음** 4섹션 초안 작성
2. 태그 사전(`_index/tags.md`)과 대조해 일관성 있게 정규화
3. 사용자 확인 (또는 `--quick`으로 스킵)
4. `~/work-log/_sessions/YYYY-MM-DD__<ID>_HHMM.md`에 **독립 세션 파일**로 저장

각 호출이 자기만의 파일을 만들기 때문에 **여러 세션·환경에서 동시에 호출해도 절대 충돌하지 않음**.

세션 파일들은 자매 스킬 `/day_merge`로 그 날의 통합본 `~/work-log/YYYY-MM-DD.md`에 합쳐진다.

## 왜 만들었나

업무가 끝났을 때 본인이 직접 정리하는 부담을 없애고, 나중에 자연어로 "지난 3개월간 X 관련 작업 모아줘"처럼 회수·복기·상관관계 발견에 쓸 수 있는 검색 가능한 기록을 쌓기 위함.

## 설치

이 폴더(`work-done/`)를 다음 위치에 놓는다:

- **Windows:** `C:\Users\<유저명>\.claude\skills\work-done\`
- **macOS/Linux:** `~/.claude/skills/work-done/`

### 자매 스킬 (둘 중 하나 또는 둘 다)

워크플로우 한 쪽만 골라 설치하거나, 둘 다 설치해서 상황별로 골라써도 됨:

**`day-merge`** — 하루 끝에 통합본 1개로 합치는 흐름
- 레포: <https://github.com/tinky1244-rgb/claude-skill-day-merge>
- 위치: `~/.claude/skills/day-merge/`

**`rebuild-index`** — 통합 안 하고 세션 파일을 atomic으로 영구 유지, 토픽 인덱스만 멱등 재생성 (자가치유)
- 레포: <https://github.com/tinky1244-rgb/claude-skill-rebuild-index>
- 위치: `~/.claude/skills/rebuild-index/`
- 본 SKILL.md 7단계가 `/work_done` 끝마다 이걸 자동 호출하게 작성돼 있음. 미설치 시 조용히 스킵.

Claude Code 재시작 후 자동 인식.

## 사용

```
/work_done                    ← 확인 받고 저장
/work_done --quick            ← 확인 없이 바로 저장
/work_done #SBR #refactor     ← 태그 직접 지정
정리해줘                       ← 자연어 호출
```

`/day_merge`로 마무리:
```
/day_merge                    ← 오늘자 세션 통합
```

## 저장 위치

기본값: `~/work-log/`
- Windows: `C:\Users\<유저명>\work-log\`
- macOS/Linux: `~/work-log/`

다른 경로를 원하면 `~/.claude/work-log.config.json`:

```json
{
  "path": "D:\\내 업무 기록\\"
}
```

여러 PC에서 쓰려면 클라우드 동기화 폴더(OneDrive/Dropbox 등) 안에 두는 것을 추천.

## 폴더 구조

```
~/work-log/
├── README.md                            ← 첫 실행 시 자동 생성
├── _sessions/                            ← 통합 전 세션 파일들
│   └── 2026-05-18__a3f2_1430.md
├── 2026-05-17.md                         ← 통합본 (day_merge 출력)
├── _archive/                             ← 통합 후 원본 보관
│   └── 2026-05-17/
└── _index/
    ├── topics.md                         ← 주제 인덱스 (day_merge 갱신)
    └── tags.md                           ← 태그 사전 (work_done 갱신)
```

## 추천 흐름

자매 스킬 어느 쪽을 쓰느냐에 따라 두 가지 흐름.

### A. day-merge 흐름 (통합본 만들기)

```
세션 작업 끝       → /work_done
다른 세션 작업 끝  → /work_done
                       ⋮
하루 마무리        → /day_merge
```

모든 /work_done 호출이 충돌 없이 자기 세션 파일을 만들고, /day_merge 한 번이 모두 통합. 통합본(`YYYY-MM-DD.md`) 기반 회수에 적합.

### B. atomic 흐름 (rebuild-index, 통합 안 함)

```
세션 작업 끝       → /work_done   (7단계가 rebuild-index 자동 호출)
다른 세션 작업 끝  → /work_done
                       ⋮
(별도 마무리 단계 없음)
```

세션 파일이 영원히 atomic 유지, `_index/topics.md`만 매번 멱등 재생성. grep 친화 / 환각 위험 낮음 / `tags.md`에 동의어 추가 시 옛 세션도 자동 정정.

둘 다 설치해두고 그날 기분에 따라 골라써도 무방 — 두 스킬은 서로 충돌하지 않음.

## 회수하기

기록이 쌓이면 Claude에게 자연어로:

- "지난 주 SBR 관련 작업 모아줘"
- "WAI-Design 프로젝트에서 했던 의사결정 보여줘"
- "최근에 리팩토링한 거 정리해줘"

Claude가 `_index/topics.md`와 통합본들을 읽어 답변 + 인용 제공.

## 여러 PC에서 동기화 (GitHub 사용)

`~/work-log/` 폴더 전체를 **private GitHub 레포로 관리**하면 사무실·집·노트북에서 같은 기록을 공유할 수 있습니다. 별도 스킬·도구 불필요 — Claude한테 자연어로 시키면 됩니다.

### 최초 셋업 (한 번만)

첫 `/work_done` 호출로 `~/work-log/`가 생성된 뒤, Claude한테:

> "내 ~/work-log 폴더를 private GitHub 레포(이름: work-log)로 푸시해줘"

→ Claude가 `gh repo create` + `git init` + `git push` 모두 처리.

### 다른 PC에서 시작할 때

> "https://github.com/<유저명>/work-log.git 클론해서 ~/work-log 에 둬"

또는 직접:
```powershell
git clone https://github.com/<유저명>/work-log.git "$env:USERPROFILE\work-log"
```

### 작업 끝나고 동기화

```
/work_done × N         (세션 작업)
/day_merge             (통합)
```
그리고 Claude한테:
> "오늘 작업 git에 커밋 푸시해줘"

### 주의사항

- ⚠️ **반드시 Private 레포** — 업무 내용이 들어가니까
- ⚠️ **회사 데이터 정책 확인** — 개인 GitHub에 업무 기록 올리는 게 금지일 수 있음
- 두 PC에서 동시에 push하면 한 쪽이 거부됨 → pull 먼저 받고 다시 push (Claude한테 시키면 됨)

## 단순 백업만 원할 때

GitHub 동기화가 부담스럽다면 폴더를 OneDrive/Dropbox/iCloud 폴더 안에 두는 것만으로도 자동 백업·동기화 가능 (`~/.claude/work-log.config.json`에서 경로 변경).

## 파일

| 파일 | 역할 |
|---|---|
| `SKILL.md` | 트리거·7단계 워크플로우 (Claude가 읽음. 7단계는 rebuild-index 연동용 선택 단계) |
| `references/format.md` | 세션 파일·통합본·태그 사전 상세 템플릿 |
| `references/tags-seed.md` | 첫 실행 시 `_index/tags.md`에 복사할 시드 |
| `README.md` | 이 파일 |

## 라이선스

자유롭게 수정·배포 가능.
