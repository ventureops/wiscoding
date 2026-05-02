# claude-forge 도입 사례 — 외부 Claude Code 번들 검토 체크리스트

> 한국어 친화적인 GitHub 배포 번들 [`sangrokjung/claude-forge`](https://github.com/sangrokjung/claude-forge)를 깔았다가 5분 만에 롤백한 경험을 정리한 사례. 마켓플레이스 외 번들을 도입할 때 **무엇을 먼저 봐야 하는지**의 실전 점검 항목.

## 상황

`~/.claude/`에 본인이 만든 슬래시 커맨드·스킬·세팅이 이미 있는 사용자가 외부 Claude Code 번들을 시험 삼아 깔았더니 다음 셋이 한꺼번에 발생했다.

- **이름 충돌**: 번들 측 `/plan` 슬래시 커맨드가 Claude Code 내장 plan 모드와 충돌
- **발자국**: `install.sh`가 `~/.claude/{agents,commands,skills,hooks,rules,scripts,settings.json}` 7개를 자기 리포로 가는 심볼릭 링크로 일괄 덮어씀. 사용자의 기존 파일은 별도 backup 폴더로 밀려남
- **광고 vs 실체 간극**: skills 디렉토리 44개 중 21개가 `./.agents/skills/...` 를 가리키는 dangling 심볼릭 링크였고, `.agents` 디렉토리는 클론에도 없고 `.gitmodules`나 `install.sh`도 만들지 않음

배포자는 AI 강사이고 superpowers 위에 자체 워크플로우 레이어를 올리는 강의용 자료로 만든 것으로 추정된다. **강사가 직접 부트스트랩을 도와주는 라이브 강의 환경에서는 작동하지만, README만 보고 따라하는 학습자에게는 첫 5분 안에 깨지는 경험이 된다.**

이 사례를 바탕으로, **마켓플레이스 외 GitHub 배포 번들을 도입하기 전 점검할 6항목**을 정리한다.

## 방법

### 점검 체크리스트 6항목

#### 1. 설치 발자국(footprint)을 미리 그려본다

확인할 것:
- 어떤 디렉토리를, 어떤 방식(복사·심볼릭 링크)으로 건드리는가
- install 스크립트가 dry-run 모드를 제공하는가
- 사용자의 기존 파일이 있을 때의 처리(덮어쓰기 / 자동 백업 / 머지 / 중단)가 명시돼 있는가
- 설치 후 결과물의 위치가 사용자에게 즉시 보이는가

**claude-forge 사례**: `--dry-run` 플래그는 제공되지만 README Quick Start의 기본 경로(`./install.sh`)가 dry-run을 권장하지 않음. 기존 파일이 있을 때 덮어쓰기 전 사용자에게 어떤 충돌이 있는지를 사전 미리보기로 보여주지 않고, 실행 후 backup 폴더로 옮겨졌다는 사실을 사용자가 나중에 발견해야 한다.

#### 2. 이름공간(namespace) 충돌을 확인한다

확인할 것:
- 호스트 플랫폼(Claude Code)의 빌트인 명령(`/init`, `/review`, `/plan`, `/config`, `/help` 등)과 같은 이름이 있는가
- 이미 깔려 있는 다른 플러그인(superpowers 등)과 스킬·에이전트 이름이 겹치는가
- 같은 이름이 두 곳에 존재할 때 어느 쪽이 이기는지 명시적으로 결정되는가

**claude-forge 사례**: `/plan`이 plan 모드와 정면 충돌. `using-superpowers`, `writing-plans`, `executing-plans`, `brainstorming`, `test-driven-development`, `systematic-debugging` 등 **superpowers 스킬과 동일한 이름**을 14개 가까이 자기 skills 디렉토리에 또 두고 있어, 호출 시 어느 정의가 적용되는지가 환경 의존적이 된다.

#### 3. 전제 의존성을 명시했는지 본다

확인할 것:
- "X를 먼저 깔아야 한다"는 prerequisite가 README의 Quick Start 안에 있는가
- 의존성을 만족하지 못한 상태로 install이 실행됐을 때 실패하는가, 조용히 깨진 상태로 통과하는가
- 설치 후 self-check(헬스체크) 명령이 있는가

**claude-forge 사례**: 동작상 superpowers 선설치가 전제이지만 README Quick Start에는 명시 없음. 의존성 미충족 상태로 설치하면 실패하지 않고 dangling 심볼릭 링크가 그대로 자리 잡는다. 사용자는 깨진 스킬을 호출해야 비로소 인지하게 된다.

#### 4. 광고 카운트와 실체를 표본 점검한다

확인할 것:
- 광고된 숫자(예: "11 agents, 24 skills")가 실체 있는 항목만 세는지, placeholder도 포함하는지
- 가장 끌리는 1~2개 항목을 골라 실제 파일까지 따라가본다
- 심볼릭 링크가 있다면 `readlink`로 타깃을 확인해 dangling이 아닌지 검증한다

**claude-forge 사례**: 광고된 24 skills 중 약 21개가 dangling 심볼릭 링크 — 절반 가까이가 빈 상자. 광고 카운트를 신뢰할 수 없다.

빠른 확인 명령:
```bash
# skills 디렉토리에서 dangling 심볼릭 링크 일괄 검출
find ~/projects/<번들-이름>/skills -maxdepth 1 -type l ! -exec test -e {} \; -print
```

#### 5. 공식 플러그인 모델과의 정합성을 확인한다

확인할 것:
- Claude Code 마켓플레이스 플러그인으로 제공되는가, 별도 install 스크립트가 필수인가
- 별도 스크립트 경로일 경우, Anthropic의 플러그인 sandboxing을 우회해 `~/.claude/`를 직접 수정하는 방식인가
- 미래 Claude Code 업데이트와의 호환 위험이 어느 정도인가 (settings.json 스키마 변경 등)

**claude-forge 사례**: 메인테이너 본인이 README에서 인정하듯, 마켓플레이스 플러그인 경로(`/plugin install`)는 commands + skills의 일부만 로드한다. agents/hooks/rules/MCP/statusLine은 `install.sh`로 직접 `~/.claude/`에 심볼릭 링크를 거는 경로로만 작동한다. 즉 풀 기능을 쓰려면 공식 모델을 우회해야 한다.

#### 6. distribution(자동 설치형) vs reference(참고 자료실) 포지셔닝을 구분한다

자동 설치형 distribution을 표방하는데 첫 5분 사용자 경험이 깨지면, **실제로는 reference distribution이다**. 둘은 사용 방식이 완전히 다르다.

| 포지셔닝 | 사용 방식 | 적합한 형태 |
|---|---|---|
| **distribution** (유통 가능한 프레임워크) | 통째로 install → 즉시 작동 | 마켓플레이스 플러그인, 명확한 prerequisite 검증, dry-run, 충돌 사전 알림 |
| **reference** (참고 자료실) | clone 해두고 골라서 발췌 | GitHub 리포 그 자체, 설치 스크립트 불필요 |

자동 설치를 표방하지만 실제로는 reference에 가깝다고 판단되면, **install 스크립트를 돌리지 말고 `git clone`만 한 뒤 필요한 부분만 발췌**하는 게 안전하다.

### 안전하게 발췌해서 쓰는 방법

발췌는 다음 순서로 한다.

```bash
# 1. 별도 디렉토리에 클론 (~/.claude/ 가 아닌 곳)
git clone https://github.com/<owner>/<repo>.git ~/projects/<repo>

# 2. 인벤토리 훑기 — 카테고리별 description 추출
cd ~/projects/<repo>
for f in agents/*.md; do
  awk '/^description:/{sub(/^description: */,""); print FILENAME": "$0; exit}' "$f"
done
```

3. 후보 1~2개를 정한다. **prefix를 자기 컨벤션으로 리네임**(예: `cf-` 접두) 해서 충돌을 사전 차단.
4. 의존성 점검: 그 파일이 다른 스킬·hook 라이브러리·설정을 참조하는지 `grep`으로 확인.
5. 자기 `~/.claude/` 구조에 맞게 복사하고 경로 수정.
6. **hook을 발췌한 경우 `settings.json`의 매칭 엔트리도 함께 옮긴다** — 이걸 빠뜨리면 hook이 작동하지 않는다.
7. MCP 서버 설정은 별도 파일(`mcp-servers.json` 또는 `.mcp.json`)이라 따로 챙긴다.

발췌 후 정리:
```bash
# 발췌가 끝나면 클론과 옛 설치 자리 모두 삭제 (선택)
rm -rf ~/projects/<repo>
rm -rf ~/.claude.<repo>-installed   # install.sh가 만든 옛 자리가 있다면
```

## 주의사항

- 본 사례는 **2026-05-03 시점의 v3.0.1 스냅샷**이다. 이후 개선될 수 있고, 특히 마켓플레이스 정식 등재나 prerequisite 명시가 추가되면 평가는 달라질 수 있다.
- 메인테이너의 강의 자료로서의 가치는 별도로 평가되어야 한다. 라이브 코칭이 함께라면 학습 효율이 클 수 있다.
- 본 노트는 특정 패키지 비방을 목적으로 하지 않는다. **외부 번들을 받아들이는 사용자 측 점검 절차**가 핵심이다.
- 이름 충돌을 막는 가장 단순한 방법은 발췌 시 prefix(`cf-` 등)로 리네임하는 것이다. 호스트·다른 플러그인과의 충돌을 사전에 차단한다.
- hook을 발췌할 때는 반드시 `settings.json`의 hook 등록 엔트리도 함께 옮겨야 한다.

## 검증환경

- macOS (Darwin 25.4.0)
- Claude Code (VSCode native extension)
- 사용자 사전 셋업: 본인 작성 슬래시 커맨드·스킬 다수, [Anthropic Superpowers](https://github.com/anthropics/skills) 사용 중
- 검증 시점: 2026-05-03 (claude-forge README 기준 v3.0.1)

## 출처·참고

- claude-forge GitHub: <https://github.com/sangrokjung/claude-forge>
- 관련 문서 (claude-forge 리포 내): `docs/PLUGIN-VS-INSTALL-SH.md`, `docs/MIGRATION.md`
- 관련 wiscoding 문서: [Claude Code 슬래시 커맨드 만들기](../claude-code/custom-slash-commands.md), [Obsidian CLI 활용](obsidian-cli.md)
- 본 검토 세션 기록 (2026-05-03)
