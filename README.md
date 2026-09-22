# git_-

HWP(한글) 문서 작업을 위한 MCP 서버 설정을 관리하는 저장소임.

## MCP 서버 설정

프로젝트 스코프 설정 파일 `.mcp.json`에 `hwp-mcp` 서버가 등록되어 있음.

```json
{
  "mcpServers": {
    "hwp-mcp": {
      "command": "npx",
      "args": ["-y", "hwp-mcp"]
    }
  }
}
```

### 적용 방법

1. 이 저장소를 클론한 뒤 해당 디렉터리에서 Claude Code를 실행함.
2. 프로젝트 스코프 MCP 서버 사용 승인 여부를 묻는 프롬프트에 동의함.
3. `/mcp` 명령으로 `hwp-mcp` 서버의 연결 상태를 확인함.

### 전제 조건

- Node.js 20 이상이 필요함(`-y` 옵션으로 최초 실행 시 패키지를 자동 설치함).
- macOS·Windows·Linux를 지원하며, 한컴오피스(한/글) 설치는 필요하지 않음. rhwp(Rust + WebAssembly) 파서를 사용함.

### 기능 범위

| 연산 | `.hwp` | `.hwpx` |
|---|:---:|:---:|
| 읽기 | 지원 | 지원 |
| 쓰기 | 미지원 | 지원 |
| SVG 렌더링 | 지원 | 지원 |

### 문서

- [Claude Desktop(Windows) 설정 가이드](docs/setup-claude-desktop-windows.md) — 설치, 설정 파일 위치, 사용 예시, 문제 해결
- [HWP → Markdown 변환 계획](docs/superpowers/plans/2026-07-08-convert-hwp-markdown.md) — 변환 절차 및 검증 기준

### 설정 스코프 참고

- `.mcp.json`(프로젝트 스코프): 저장소를 공유하는 모든 사용자에게 동일하게 적용됨.
- 개인 환경에만 적용하려는 경우 `claude mcp add hwp-mcp -s user -- npx -y hwp-mcp` 형태로 사용자 스코프에 등록함.
