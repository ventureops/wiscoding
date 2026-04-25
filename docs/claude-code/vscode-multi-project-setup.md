# Claude Code 다중 프로젝트 환경에서 VS Code 창 관리

> 여러 프로젝트를 Claude Code로 동시에 다룰 때, 프로젝트당 VS Code 창 1개를 권장하는 이유와 셋업 방법.

## 상황

Claude Code로 여러 프로젝트를 병행할 때 IDE 구성에 두 가지 패턴이 있다.

**방식 A: 프로젝트당 VS Code 창 1개**
```
[VS Code 창 1] project-a
[VS Code 창 2] project-b
[VS Code 창 3] project-c
```

**방식 B: VS Code 1개 + 멀티 루트 워크스페이스**
```
[VS Code 창 1]
├── project-a
├── project-b
└── project-c
```

대부분의 경우 **방식 A**가 더 안전하고 실수가 적다. 그 이유와 운영 방법을 정리한다.

## 비교표

| 항목 | 방식 A (창 여러 개) | 방식 B (멀티 루트) |
|---|---|---|
| 메모리 사용 | 프로젝트당 약 500MB–1GB | 합쳐서 1.5–2GB |
| 컨텍스트 분리 | ✅ 완벽 분리 | 폴더 경로로만 분리 |
| Claude Code 세션 | ✅ 프로젝트별 독립 | 어디서 실행됐는지 모호 |
| 프로젝트 전환 | `Cmd+Tab` (창 전환) | 사이드바 폴더 클릭 |
| 터미널 작업 디렉토리 | ✅ 자동으로 프로젝트 루트 | 명시적 `cd` 필요 |
| CLAUDE.md 인식 | ✅ 명확 | 어느 CLAUDE.md를 읽는지 모호 |
| Git 상태 표시 | 프로젝트별 명확 | 멀티 리포 표시 (혼동 가능) |
| 통합 검색 | 프로젝트 내로 한정 | 여러 프로젝트 통합 검색 가능 (방식 B의 장점) |
| 창 관리 부담 | 창이 많아짐 | 한 창에 집중 |

## Claude Code 관점에서의 결정적 차이

Claude Code는 **실행한 디렉토리를 프로젝트 루트로 인식**한다. 구체적으로:

- 현재 작업 디렉토리(`pwd`)를 루트로 가정
- 그 루트에서 `CLAUDE.md`, `.claude/settings.local.json`을 찾음
- `docs/`, `tasks/` 같은 상대 경로를 그 루트 기준으로 해석

### 방식 A에서

VS Code 창마다 통합 터미널이 자동으로 해당 프로젝트 루트를 작업 디렉토리로 사용한다.

```bash
# project-a 창의 터미널
$ pwd
/Users/me/projects/project-a

$ claude
# CLAUDE.md, settings.local.json 모두 정확히 인식
```

### 방식 B에서

멀티 루트 워크스페이스의 통합 터미널은 기본 디렉토리가 **첫 번째 폴더**로 고정된다. 다른 프로젝트로 전환하려면 명시적 `cd`가 필요하다.

```bash
# 멀티 루트 워크스페이스의 터미널
$ pwd
/Users/me/projects/project-a   # 첫 번째 폴더

$ cd ../project-b
$ claude
```

이 과정에서 실수로 **project-a 디렉토리에서 project-b 작업을 지시**하면, Claude Code가 잘못된 `CLAUDE.md`를 읽고 엉뚱한 작업을 할 위험이 있다. 컨텍스트 격리가 핵심인 운영 모델에서는 이 한 가지 이유만으로도 방식 A가 압도적으로 유리하다.

## 방법: 방식 A 운영 패턴

### 기본 흐름

```
1. project-a 작업 시작
   → VS Code 창 1 열기
   → 터미널 자동으로 project-a 디렉토리
   → claude 실행

2. project-b도 병행
   → 별도 VS Code 창 2 열기
   → 자동으로 project-b 디렉토리
   → claude 실행 (별도 세션)

3. 두 작업이 완전히 격리됨
```

`Cmd+Tab`으로 창을 전환하는 것이 사이드바에서 폴더를 클릭하는 것보다 훨씬 명확하다.

### 창 색상으로 시각적 구분

VS Code 창 색상 커스터마이징을 활용하면 한눈에 어느 프로젝트인지 알 수 있다. 각 프로젝트 루트의 `.vscode/settings.json`에 추가:

```json
{
  "workbench.colorCustomizations": {
    "activityBar.background": "#1a3a52",
    "titleBar.activeBackground": "#1a3a52"
  }
}
```

프로젝트별 다른 색상 예시:

| 프로젝트 | 색상 |
|---|---|
| project-a | 파랑 `#1a3a52` |
| project-b | 초록 `#2d5a3d` |
| project-c | 보라 `#3d2d5a` |

> 참고: `.vscode/settings.json`은 개인 환경 설정이므로 보통 `.gitignore`에 넣어 커밋에서 제외한다.

### 셸 별칭으로 진입 자동화

`~/.zshrc` 또는 `~/.bashrc`에:

```bash
alias proja="cd ~/projects/project-a && code . && claude"
alias projb="cd ~/projects/project-b && code . && claude"
```

터미널에서 `proja` 한 단어로 폴더 이동 + VS Code 실행 + Claude Code 실행이 한 번에 끝난다. 진입 자동화로 디렉토리 실수 가능성이 차단된다.

## 주의사항

- **메모리 부담**: VS Code 창 3개 이상이면 메모리 사용량이 커진다. 작업 안 하는 프로젝트는 창을 닫고, 다음에 열면 VS Code가 마지막 상태를 복원한다. 또는 현재 다루는 1–2개 프로젝트만 창을 띄워두고 나머지는 필요할 때 연다.
- **방식 B가 적합한 경우**:
  - 프론트엔드+백엔드처럼 한 제품의 여러 리포가 묶일 때
  - 같은 키워드를 여러 리포에서 자주 통합 검색할 때
  - 모노레포의 일부 폴더만 다룰 때
  - Claude Code를 거의 안 쓰거나 한 프로젝트에서만 쓸 때
- **그 외 — 별개의 프로젝트들**: 방식 A 권장.
- **권한 모드**: `claude` 실행 시 권한 모드는 본인 환경에 맞게 선택. 자동 승인 모드는 신뢰하는 환경에서만 신중히 사용.

## 검증환경

- OS: macOS 14
- VS Code: 1.x
- Claude Code: 2.x
- 마지막 확인일: 2026-04-26

## 출처·참고

- 직접 경험 (다중 프로젝트 운영 중 `CLAUDE.md` 오인식 사고로부터)
- [VS Code Multi-root Workspaces](https://code.visualstudio.com/docs/editor/multi-root-workspaces)
