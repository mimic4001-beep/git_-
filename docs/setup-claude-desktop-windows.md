# Claude Desktop(Windows)에서 한/글 문서 다루기

대상 환경: **Windows + 한/글 설치 + Claude Desktop**, 주 사용 포맷 `.hwpx`

---

## 1. 전제 조건

- **Node.js 20 이상**이 설치되어 있어야 함. 명령 프롬프트에서 확인함.

```cmd
node -v
```

  `v20.x` 이상이 출력되지 않으면 [nodejs.org](https://nodejs.org)에서 LTS 버전을 설치함.

- 한컴오피스 설치는 **필요하지 않음**. `hwp-mcp`는 rhwp(Rust + WebAssembly) 파서로 파일을 직접 해석하므로 한/글이 실행되지 않아도 동작함. (다만 `.hwp` 원본을 직접 편집하려는 경우에는 6절 참고)

---

## 2. 설정 파일 위치

| 항목 | 경로 |
|---|---|
| 설정 파일 | `%APPDATA%\Claude\claude_desktop_config.json` |
| 로그 디렉터리 | `%APPDATA%\Claude\logs` |

앱에서 여는 방법: **설정(Settings) → 개발자(Developer) → 설정 편집(Edit Config)**

파일이 없으면 위 경로에 새로 생성하면 됨.

---

## 3. 설정 내용

아래 내용을 `claude_desktop_config.json`에 작성함. 이미 다른 MCP 서버가 등록되어 있다면 `mcpServers` 객체 안에 `hwp-mcp` 항목만 추가함.

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

### Windows에서 서버가 기동되지 않는 경우

Windows 환경에서는 `npx`가 직접 실행되지 않는 사례가 보고되고 있음. 이 경우 `cmd /c`로 감싸서 등록함.

```json
{
  "mcpServers": {
    "hwp-mcp": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "hwp-mcp"]
    }
  }
}
```

---

## 4. 연결 확인

1. Claude Desktop을 **완전히 종료**함. 창을 닫는 것만으로는 부족하며, 작업 표시줄 트레이 아이콘에서 종료해야 함.
2. 앱을 다시 실행함.
3. 입력창 하단의 도구(MCP) 표시에서 `hwp-mcp`가 나타나는지 확인함.
4. 표시되지 않으면 `%APPDATA%\Claude\logs\mcp.log`를 확인함.

---

## 5. 사용 방법

파일 경로를 함께 제시하면 됨. 대화창에서 아래와 같이 요청함.

### 읽기

```
C:\Users\사용자\Documents\보고서.hwpx 파일을 읽고 목차를 정리해줘
```

내부적으로 `read_hwp`, `read_hwp_text`, `read_hwp_tables` 등이 사용됨. 본문·머리말·꼬리말·각주·수식·표·이미지 목록을 추출함.

### 수정

```
보고서.hwpx에서 "2025년"을 "2026년"으로 모두 바꿔줘
3번 표의 2행 3열 값을 1,250으로 수정해줘
```

`replace_hwp_text`, `set_hwp_cell_text`, `apply_hwp_text_style` 등이 사용됨.

### 생성

```
아래 내용으로 회의록 .hwpx 문서를 새로 만들어줘
```

`create_hwpx_document`가 사용됨.

### Markdown 변환

```
보고서.hwpx를 Markdown으로 변환해서 저장해줘
```

`convert_hwp_markdown`이 문서 흐름 순서를 보존하여 변환하고, 내장 이미지를 상대경로 링크로 내보냄. 상세 절차는 [HWP → Markdown 변환 계획](superpowers/plans/2026-07-08-convert-hwp-markdown.md) 참고.

### 경로 작성 시 주의

설정 파일이나 프롬프트에 Windows 경로를 JSON으로 넣을 때는 역슬래시를 두 번 쓰거나(`C:\\Users\\...`) 슬래시(`C:/Users/...`)를 사용함. 단일 역슬래시는 JSON 이스케이프 문자로 해석되어 오류를 유발함.

---

## 6. 제약 사항

| 항목 | 내용 |
|---|---|
| `.hwp` 쓰기 | **미지원**. 읽기만 가능함. 수정하려면 한/글에서 `다른 이름으로 저장 → HWPX`로 변환함 |
| 교차 포맷 저장 | `.hwpx` 입력은 `.hwpx`로만 저장됨 |
| 표 생성 | `create_hwpx_document`로 만든 신규 문서의 표는 현재 버전에서 텍스트 행으로 평면화됨. 정식 OWPML 표는 v0.3 예정 |
| 미지원 요소 | 텍스트박스 내용, 미주, 차트는 v0.3으로 이연됨 |
| 검색 한계 | 검색어가 복수 XML 텍스트 노드에 걸치면 매칭되지 않음 |

한글 2024부터 기본 저장 형식이 HWPX이므로, 최신 문서 위주로 작업하는 경우 `.hwp` 제약의 실질적 영향은 크지 않음.

### `.hwp` 원본을 직접 편집해야 하는 경우

Windows에 한/글이 설치되어 있으므로, COM 방식 서버를 **추가로** 등록하여 병행 사용할 수 있음. 한/글을 직접 조종하므로 `.hwp`를 원본 그대로 열고 저장함.

```json
{
  "mcpServers": {
    "hwp-mcp": {
      "command": "npx",
      "args": ["-y", "hwp-mcp"]
    },
    "hwp-com": {
      "command": "python",
      "args": ["C:/경로/hwp-mcp/hwp_mcp_stdio_server.py"]
    }
  }
}
```

사전에 저장소를 클론하고 의존성을 설치해야 함(Python 3.7 이상 필요).

```cmd
git clone https://github.com/jkf87/hwp-mcp.git
cd hwp-mcp
pip install -r requirements.txt
```

---

## 7. 문제 해결

| 증상 | 원인 및 조치 |
|---|---|
| MCP 서버가 목록에 아예 없음 | JSON 문법 오류일 가능성이 높음. Claude Desktop은 설정 파일이 잘못되어도 오류 대화상자 없이 조용히 무시함. JSON 검증기로 확인함 |
| 서버는 보이나 연결 실패 | 3절의 `cmd /c` 방식으로 변경 후 재시도함 |
| 앱 재시작 후에도 반영 안 됨 | 창만 닫은 경우임. 트레이에서 완전히 종료 후 재실행함 |
| `Edit Config`가 엉뚱한 파일을 엶 | Windows MSIX 설치본에서 보고된 사례임. `%APPDATA%\Claude\claude_desktop_config.json`을 직접 열어 편집함 |
| 파일을 못 찾는다고 함 | 경로의 역슬래시 처리를 확인함(`\\` 또는 `/` 사용) |
| 원인 불명 | `%APPDATA%\Claude\logs\mcp.log` 확인함 |

---

## 참고 자료

- [treesoop/hwp-mcp](https://github.com/treesoop/hwp-mcp)
- [jkf87/hwp-mcp (COM 방식)](https://github.com/jkf87/hwp-mcp)
- [HWPX 관련 자주 묻는 질문 — 한컴 지원센터](https://www.hancom.com/support/faqCenter/faq/detail/2784)
- [한/글 문서 파일 형식: HWPX 포맷 구조 — 한컴테크](https://tech.hancom.com/hwpxformat/)
