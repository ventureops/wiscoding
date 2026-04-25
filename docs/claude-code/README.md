# Claude Code

Anthropic의 공식 CLI인 **Claude Code** 자체에 관한 자료를 모읍니다.

## 다루는 주제

- **설치·설정** — 설치 방법, `~/.claude/settings.json`, 프로젝트별 `.claude/`
- **훅 (Hooks)** — `PreToolUse`, `PostToolUse`, `SessionStart`, `Stop` 등 이벤트 훅 작성법
- **슬래시 명령** — 내장 명령(`/help`, `/clear` 등)과 커스텀 명령 만들기
- **MCP 서버** — Model Context Protocol 서버 연결, 인증, 자체 MCP 만들기
- **Skills** — Skill 작성, 트리거 조건, 외부 스킬 가져오기
- **단축키·키바인딩** — `~/.claude/keybindings.json` 커스터마이징
- **권한·보안** — 자동 승인 규칙, 위험 명령 차단, 권한 모드
- **IDE 통합** — VSCode 확장, JetBrains 연동
- **하위 에이전트** — Agent 도구, subagent 작성, 병렬 실행 패턴
- **트러블슈팅** — 흔한 에러와 해결, 캐시 문제, 인증 이슈

## 자료 추가하기

이슈 템플릿 **"새로운 팁"** 에서 분류를 `Claude Code`로 골라 제출하시거나,
직접 PR로 이 폴더에 마크다운 파일을 추가해주세요.

파일명 예시:
- `hooks-pre-commit-format.md`
- `mcp-server-setup-mac.md`
- `custom-slash-commands.md`

자세한 작성 규칙은 루트의 [CONTRIBUTING.md](../../CONTRIBUTING.md) 참고.
