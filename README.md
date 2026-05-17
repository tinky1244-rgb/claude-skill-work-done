# work-done Skill

Claude Code에서 수행한 업무를 자동으로 마크다운 파일에 정리·누적·인덱싱하는 스킬.

## 무엇을 하나

`/work_done`을 호출하면 Claude가 현재 대화를 보고:

1. **한 일 / 왜 / 의사결정·트레이드오프 / 다음** 4개 섹션으로 초안 작성
2. 사용자 확인 받기 (또는 `--quick`으로 스킵)
3. `~/work-log/YYYY-MM-DD.md`에 저장 (같은 날 여러 번 호출하면 append)
4. `~/work-log/_index/topics.md` 주제 인덱스 자동 갱신

자연어로도 호출 가능: "정리해줘", "오늘 한 일 정리", "업무 기록" 등.

## 왜 만들었나

업무가 끝났을 때 본인이 직접 정리하는 부담을 없애고,
나중에 자연어로 "지난 3개월간 X 관련 작업 모아줘"처럼 회수·복기·상관관계 발견에 쓸 수 있는 검색 가능한 기록을 쌓기 위함.

## 설치

이 폴더(`work-done/`)를 다음 위치에 놓는다:

- **Windows:** `C:\Users\<유저명>\.claude\skills\work-done\`
- **macOS/Linux:** `~/.claude/skills/work-done/`

Claude Code를 재시작하면 자동으로 인식한다.

## 사용

```
/work_done                    ← 확인 받고 저장
/work_done --quick            ← 확인 없이 바로 저장
정리해줘                       ← 자연어 호출도 가능
```

## 저장 위치

기본값: 사용자 홈 디렉토리 아래 `work-log/`
- Windows: `C:\Users\<유저명>\work-log\`
- macOS/Linux: `~/work-log/`

다른 경로를 원하면 첫 실행 시 지정 가능. 설정은 `~/.claude/work-log.config.json`에 저장됨:

```json
{
  "path": "D:\\내 업무 기록\\"
}
```

## 폴더 구조

```
~/work-log/
├── README.md           ← 첫 실행 시 자동 생성
├── 2026-05-17.md       ← 날짜별 기록
├── 2026-05-18.md
└── _index/
    └── topics.md       ← 주제 태그 자동 인덱스
```

## 회수하기

기록이 쌓이면 Claude에게 자연어로 물어보면 된다:

- "지난 주 SBR 관련 작업 모아줘"
- "WAI-Design 프로젝트에서 했던 의사결정 보여줘"
- "최근에 리팩토링한 거 정리해줘"

Claude가 `_index/topics.md`와 일일 파일들을 읽어 답한다.

## 백업

`~/work-log/` 폴더를 통째로 드롭박스/원드라이브/git에 넣으면 자동 백업·동기화.

## 파일

| 파일 | 역할 |
|---|---|
| `SKILL.md` | 트리거·워크플로우 정의 (Claude가 읽음) |
| `references/format.md` | 일일 파일·인덱스의 상세 템플릿 |
| `references/index-rebuild.md` | 인덱스 재생성 로직 |
| `README.md` | 이 파일 |

## 라이선스

자유롭게 수정·배포 가능.
