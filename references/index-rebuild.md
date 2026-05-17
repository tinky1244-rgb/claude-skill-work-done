# 주제 인덱스 재생성 로직

`<work-log>/_index/topics.md`는 매 `/work_done` 호출 시 처음부터 다시 만든다. append 방식이 아니라 **전체 재생성**이다.

## 왜 매번 재생성하는가

- **Drift 방지:** 일일 파일의 frontmatter가 진실의 원천. 인덱스는 파생물.
- **삭제·수정 반영:** 사용자가 일일 파일의 태그를 수정하거나 파일을 삭제해도 다음 호출 시 자동 반영.
- **유지비용 0:** "추가/삭제 시 인덱스도 같이 갱신" 같은 규칙 없음. 항상 재생성.
- **규모 부담 없음:** 일일 파일이 수백 개라도 frontmatter만 읽으면 충분히 빠름 (`Read`의 `limit: 15`).

## 단계별 절차

### 1. 파일 수집

```
Glob 패턴: <work-log>/*.md
제외 대상:
  - <work-log>/README.md
  - <work-log>/_index/ 안의 모든 파일
```

남는 것은 `YYYY-MM-DD.md` 형식의 일일 파일들.

### 2. Frontmatter 추출

각 파일에 대해:

```
Read(file_path, limit: 15)
```

처음 15줄 안에 frontmatter (`---`로 시작/종료) + 본문 첫 섹션 제목이 모두 들어옴. 다음을 파싱:

- `date` — ISO 날짜
- `topics` — 배열
- `projects` — 배열
- 본문 첫 `## [HH:MM] <제목>` 행에서 `<제목>` 부분 추출

### 3. 집계

두 개의 매핑을 만든다:

```
topic_map: {topic_name: [{date, summary, path}, ...]}
project_map: {project_name: [{date, summary, path}, ...]}
```

각 일일 파일을 순회하면서, 그 파일의 모든 topic·project에 대해 항목을 추가.

### 4. 정렬

- **상위 키 (topic/project 이름):** 알파벳·가나다 순
  - 영문은 대소문자 무시, 한글은 유니코드 순서
  - 같은 카테고리 안에서 영문이 한글보다 먼저 와도 OK (유니코드 정렬 결과)
- **각 항목:** date 내림차순 (최신 먼저)

### 5. 출력 작성

`references/format.md`의 "주제 인덱스" 섹션 템플릿대로 `Write`.

상단에 `마지막 갱신:` 줄 포함 (현재 일시).

### 6. 빈 인덱스 처리

- 일일 파일이 0개면 인덱스 파일은 다음 내용만:
  ```markdown
  # 주제별 인덱스

  > 아직 기록된 작업이 없습니다. `/work_done`을 호출하면 자동으로 채워집니다.
  ```

## 의사 코드

```
files = Glob("<work-log>/*.md")
files = [f for f in files if not f.endswith("README.md")]

topic_map = {}
project_map = {}

for f in files:
    head = Read(f, limit=15)
    fm = parse_frontmatter(head)        # date, topics, projects
    summary = parse_first_section(head) # "## [HH:MM] <제목>" 에서 <제목>
    rel_path = f"../{basename(f)}"

    entry = {"date": fm.date, "summary": summary, "path": rel_path}

    for t in fm.topics:
        topic_map.setdefault(t, []).append(entry)
    for p in fm.projects:
        project_map.setdefault(p, []).append(entry)

# 정렬
for k in topic_map: topic_map[k].sort(key=lambda e: e["date"], reverse=True)
for k in project_map: project_map[k].sort(key=lambda e: e["date"], reverse=True)

# 렌더링
md = render_index_md(topic_map, project_map, now())
Write("<work-log>/_index/topics.md", md)
```

## 주의

- frontmatter가 누락된 파일은 건너뛴다 (조용히, 에러 아님)
- `topics`나 `projects`가 빈 배열이어도 OK — 해당 파일은 인덱스에 안 들어감
- 본문 첫 섹션 제목을 못 찾으면 요약을 `(제목 없음)`으로
