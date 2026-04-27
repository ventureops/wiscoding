# Claude Code에서 Obsidian 활용하기

> Claude Code가 Obsidian CLI를 통해 실행 중인 볼트를 직접 읽고 쓰는 방법. 단순 파일 조작을 넘어 링크·백링크·태그·태스크까지 다룰 수 있다.

## 상황

Claude Code에서 Obsidian 볼트 안의 파일을 읽거나 쓰고 싶을 때. 일반 `cat`/`echo`로도 가능하지만, Obsidian CLI를 쓰면:

- **백링크·링크·태그** 조회에 Obsidian 인덱스를 활용해 훨씬 빠름
- **파일 수정이 실행 중인 Obsidian에 즉각 반영**됨
- 현재 열린 노트·워크스페이스 상태를 인지할 수 있음

## 방법

### 1. 사전 조건

1. **Obsidian이 실행 중**이어야 한다 (백그라운드 실행 포함).
2. Obsidian → Settings → General → Advanced → **CLI 활성화 체크** → PATH 등록 버튼 클릭.
3. 터미널에서 `obsidian` 입력 시 로고가 뜨면 준비 완료.

```bash
obsidian version           # 설치 확인
obsidian vaults verbose    # 등록된 볼트 목록 + 경로 확인
```

### 2. 볼트 지정

- 기본 대상: **가장 최근에 포커스된 볼트**.
- 특정 볼트를 지정하려면 `vault=<이름>`을 **첫 번째 인자**로.

```bash
obsidian vault="projects" files
obsidian vault="obsidian" search query="클로드"
```

### 3. 핵심 명령어

**노트 읽기·쓰기**

```bash
obsidian read file="노트명"              # 노트명으로 (위키링크처럼 검색)
obsidian read path="folder/note.md"     # 볼트 루트 기준 정확한 경로로
obsidian append file="노트명" content="추가할 내용"
obsidian prepend file="노트명" content="앞에 붙일 내용"
```

**노트 생성·열기·이동**

```bash
obsidian create name="새 노트" content="# 제목\n\n본문" silent
obsidian open file="노트명"
obsidian move file="노트명" to="이동할폴더/"
obsidian rename file="노트명" name="새이름"
```

**검색**

```bash
obsidian search query="키워드" limit=20
obsidian search query="키워드" path="특정폴더/"   # 폴더 내에서만
obsidian search:context query="키워드"            # 매칭 줄 미리보기 포함
```

**링크·백링크·태그**

```bash
obsidian backlinks file="노트명"         # 이 노트를 가리키는 링크들
obsidian links file="노트명"             # 이 노트에서 나가는 링크들
obsidian tags counts sort=count          # 볼트 전체 태그 + 빈도
obsidian tags file="노트명"              # 특정 노트의 태그
```

**프로퍼티(frontmatter)**

```bash
obsidian property:read name="status" file="노트명"
obsidian property:set name="status" value="done" file="노트명"
obsidian property:remove name="draft" file="노트명"
```

**데일리 노트**

```bash
obsidian daily:read
obsidian daily:append content="- [ ] 새 할 일"
obsidian daily:prepend content="## 오늘의 목표\n"
obsidian daily:path          # 오늘 데일리 노트 경로 반환
```

**태스크**

```bash
obsidian tasks todo                     # 볼트 전체 미완료 태스크
obsidian tasks done                     # 완료된 태스크
obsidian tasks daily todo               # 데일리 노트의 미완료 태스크
obsidian tasks file="노트명" verbose    # 특정 노트, 파일별 그룹 출력
```

**파일 목록**

```bash
obsidian files                          # 볼트 전체 파일
obsidian files folder="특정폴더/"      # 폴더 내
obsidian files ext=md                   # 확장자 필터
obsidian recents                        # 최근 열었던 파일들
```

### 4. JavaScript eval로 vault API 직접 호출

CLI가 지원하지 않는 복잡한 쿼리는 `eval`로 Obsidian 내부 API를 직접 쓸 수 있다.

```bash
# 최근 3일간 생성된 파일 목록
obsidian eval code="(() => {
  const cutoff = Date.now() - 3*24*60*60*1000;
  return app.vault.getFiles()
    .filter(f => f.stat.ctime >= cutoff)
    .sort((a,b) => b.stat.ctime - a.stat.ctime)
    .map(f => new Date(f.stat.ctime).toISOString().slice(0,16).replace('T',' ') + '  ' + f.path)
    .join('\n');
})()"

# 볼트 내 전체 파일 수
obsidian eval code="app.vault.getFiles().length"

# 특정 태그가 붙은 파일 목록
obsidian eval code="app.vault.getMarkdownFiles()
  .filter(f => app.metadataCache.getFileCache(f)?.frontmatter?.tags?.includes('project'))
  .map(f => f.path).join('\n')"
```

### 5. 파일 저장 — Write 도구 vs obsidian create

| 방법 | 장점 | 단점 |
|---|---|---|
| `obsidian create name="..." content="..."` | CLI 한 줄 | 긴 content를 `\n`으로 이스케이프해야 해 멀티라인 관리 어려움 |
| **Write 도구 + 볼트 절대경로** | 코드블록·수식·들여쓰기 그대로 | 볼트 경로를 알고 있어야 함 |

**권장**: 구조화된 노트 저장은 Write 도구 + 볼트 절대경로.

```bash
# 볼트 경로 확인
obsidian vaults verbose
```

예시 (경로는 본인 볼트에 맞게 수정):
```
Write → /Users/<이름>/path/to/vault/폴더/새 노트.md
```

### 6. 실제 활용 패턴

**패턴 A — 현재 열린 노트 요약 후 덧붙이기**

```
"현재 노트를 요약해서 노트 끝에 붙여줘"
```

1. `obsidian file` → 현재 파일 경로 확인
2. `obsidian read` → 내용 로드
3. 요약 생성
4. `obsidian append content="..."` → 덧붙이기

**패턴 B — 키워드로 노트 찾아 읽기**

```
"애자일 관련 노트 찾아서 읽어줘"
```

1. `obsidian search query="애자일"` → 후보 목록
2. 여러 개면 사용자에게 번호 선택 받기
3. `obsidian read path="..."` → 내용 로드

**패턴 C — 슬래시 커맨드로 반복 작업 자동화**

자주 하는 작업은 슬래시 커맨드로 묶으면 한 줄로 실행:

```
/clipping-summarize 국민연금공단 주식
```

→ [Claude Code 슬래시 커맨드 만들기](../claude-code/custom-slash-commands.md) 참고

**패턴 D — 데일리 노트에 자동 기록**

```
"오늘 한 작업 요약을 데일리 노트에 추가해줘"
```

```bash
obsidian daily:append content="## 작업 요약\n- ...\n- ..."
```

### 7. 빠른 참조 모음

```bash
# 볼트 상태 확인
obsidian vaults verbose
obsidian vault info=name
obsidian vault info=files

# 노트 다루기
obsidian read file="노트명"
obsidian append file="노트명" content="내용"
obsidian create name="새 노트" content="내용" silent

# 검색·탐색
obsidian search query="키워드" limit=20
obsidian backlinks file="노트명"
obsidian links file="노트명"
obsidian tags counts sort=count

# 태스크
obsidian tasks todo
obsidian tasks daily todo

# 데일리 노트
obsidian daily:read
obsidian daily:append content="- [ ] 할 일"

# vault API 직접 접근
obsidian eval code="app.vault.getFiles().length"

# 열기
obsidian open file="노트명"
```

## 주의사항

- **Obsidian이 꺼져 있으면** 모든 CLI 명령이 실패한다. 반드시 실행 중이어야 함.
- **`file=` vs `path=`**: `file="노트명"`은 이름으로 퍼지 검색, `path="folder/note.md"`는 볼트 루트 기준 정확한 경로. 동명 파일이 있을 때는 `path=` 사용.
- **eval 결과 크기**: 파일이 수백 개인 볼트에서 eval 출력이 3.5MB를 초과하면 Claude Code가 파일로 내보낸다. `.slice(0, 50)` 등으로 결과를 제한할 것.
- **속도**: 도구 왕복(검색 → 읽기 → 저장)이 누적되면 느려진다. 정확한 경로를 직접 주면 검색 단계를 스킵해 빠르다. 여러 파일을 읽을 때는 병렬 read 호출이 순차보다 빠름.

## 검증환경

- OS: macOS 14
- Obsidian: 최신 버전 (CLI 기능 포함)
- Claude Code: 2.x
- 마지막 확인일: 2026-04-27

## 출처·참고

- 직접 경험 (2026-04-27 세션 기반)
- [Obsidian 공식 사이트](https://obsidian.md)
