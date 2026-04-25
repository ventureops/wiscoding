# CLAUDE.md

이 파일은 Claude Code(claude.ai/code)가 이 리포지토리에서 작업할 때 참고하는 안내 문서입니다.

## 프로젝트 개요

**wiscoding**은 약 10명 규모의 커뮤니티가 바이브코딩과 Claude 활용 노하우를 공유하는 한국어 자료실(knowledge base)입니다. 빌드 시스템·테스트·애플리케이션 코드가 없는 **문서 전용 리포지토리**이며, 모든 콘텐츠는 마크다운으로 작성되고 [CC BY 4.0](LICENSE) 라이선스를 따릅니다.

- GitHub 위치: `ventureops/wiscoding`
- 주 언어: 한국어 (커밋 메시지는 한·영 모두 허용)

## 폴더 구조

| 경로 | 용도 |
|---|---|
| [docs/claude-code/](docs/claude-code/) | Claude Code CLI 기능 (훅, 슬래시 명령, MCP, 설정, 단축키) |
| [docs/vibe-coding/](docs/vibe-coding/) | AI 협업 워크플로·프롬프트 패턴 (도구 비종속) |
| [docs/project-ops/](docs/project-ops/) | 소규모 팀 운영 (배포, CI/CD, 모니터링, 협업 규칙) |
| [docs/tools/](docs/tools/) | 외부 도구 통합·자동화 (에디터, CLI, MCP 서버, 플랫폼) |
| [templates/](templates/) | 재사용 프롬프트·문서 골격·설정 템플릿 |
| [.github/ISSUE_TEMPLATE/](.github/ISSUE_TEMPLATE/) | 이슈 템플릿 (팁 제출, 문서 수정, 질문 안내) |

## 기여 흐름

콘텐츠는 세 가지 경로로 들어옵니다.

1. **이슈** — `새로운 팁` 템플릿으로 제출 → 운영자 검토 → 정식 문서로 편입
2. **Discussions** — 가벼운 질문·검증 전 아이디어. 잘 정리된 답변은 문서화 가능
3. **Pull Request** — 오타·작은 수정은 직접 PR

큰 신규 문서는 PR 전에 이슈로 방향 합의 먼저 권장.

## 문서 규칙

**파일명**: 영문 소문자 + 하이픈, 주제가 드러나게 (예: `mcp-server-setup-on-mac.md`).

**필수 구조**:
```markdown
# 제목
> 한 줄 요약
## 상황
## 방법
## 주의사항
## 검증환경
## 출처·참고
```

**형식**: GitHub Flavored Markdown. 코드 블록은 반드시 언어 표기. 이미지는 같은 폴더 `assets/` 하위 상대 경로.

## 콘텐츠 원칙

- 직접 써보고 검증한 내용만 (추측 금지)
- 외부 자료 참고 시 출처 표기
- 민감정보 금지 (API 키, 토큰, 사내 코드, 개인정보)
- 한 문서에 한 주제

## 작업 진행 기록 (PROGRESS.md)

**작업이 있을 때마다 루트의 [PROGRESS.md](PROGRESS.md)를 업데이트할 것.**

- 모든 항목에 **일련번호**를 매긴다 (`#001`, `#002` 형식)
- 한 항목은 **한두 줄로 간략하게**만 작성
- 형식: `#번호 | 날짜 | 한 줄 요약`
- 상세 내용은 커밋 메시지·PR·이슈에 남기고, PROGRESS.md는 색인 역할만
- 작업 단위 = 의미 있는 변경 한 묶음 (커밋 단위가 아니라 논리 단위)

PROGRESS.md 업데이트는 작업 마무리 단계에서 다른 변경과 **같은 커밋에 포함**한다.
