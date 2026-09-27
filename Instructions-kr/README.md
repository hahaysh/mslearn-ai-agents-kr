# Develop AI Agents in Azure 한국어 실습 번역 및 유지관리 지침

## 목적

이 디렉터리는 `Instructions`에 있는 영문 콘텐츠의 디렉터리 구조를 그대로 반영한 한국어 번역본을 관리합니다.

- 영문 원본은 수정하지 않습니다.
- 한국어 파일은 영문 원본과 동일한 파일명과 문서 구조를 유지합니다.
- 자연스러운 한국어를 사용하고 필요한 UI 이름은 영문과 함께 표기합니다.
- 영문 원본이 변경되면 이 문서의 동기화 절차에 따라 번역본을 갱신합니다.

## 번역 경로

| 구분 | 경로 |
|---|---|
| 영문 원본 | `Instructions` |
| 한국어 번역 | `Instructions-kr` |
| 영문 실습(Exercises) | `Instructions/Exercises` |
| 한국어 실습(Exercises) | `Instructions-kr/Exercises` |
| 영문 워크숍(Consolidated) | `Instructions/Consolidated` |
| 한국어 워크숍(Consolidated) | `Instructions-kr/Consolidated` |
| 영문 미디어 | `Instructions/Media` |
| 한국어 미디어 | `Instructions-kr/Media` |

> **참고**: 이 저장소는 기본 레이아웃(`Instructions/Labs`)이 아니라 `Instructions/Exercises`(개별 실습)와 `Instructions/Consolidated`(워크숍 태스크) 두 트리로 구성되어 있습니다. 한국어 번역본은 두 트리와 `Media` 폴더를 동일한 상대 경로로 그대로 반영합니다.

## 번역 원칙

- 수행 단계는 `~합니다` 형식으로 작성합니다.
- Microsoft 제품명은 원문을 유지합니다(예: Microsoft Foundry, Azure AI Agent Service, Microsoft Agent Framework, Visual Studio Code, GitHub, Microsoft Teams, Microsoft 365 Copilot).
- 영어 UI를 사용하는 경우 주요 항목을 `한국어(English)` 형식으로 표기합니다.
- 원문에 없는 절차를 추측하여 추가하지 않습니다.
- 원문의 오류는 임의로 수정하지 않고 아래 "알려진 원문 문제"에 기록합니다.
- 제목 수준, 단계 번호, 목록, 표, 콜아웃, 링크, 이미지, 코드 블록의 구조를 그대로 유지합니다.
- 스키마 이름, 변수 이름, URL, GUID, 파일 경로, Power Fx/Power Automate 식, OData 필터, API/커넥터 식별자 등 기술 식별자는 변경하지 않습니다.

## 프롬프트

영문과 한국어 프롬프트를 각각 복사 가능한 `prompt` 코드 블록으로 제공합니다. 영문 원본 블록은 그대로 두고, 그 뒤에 한국어 프롬프트 블록을 추가합니다.

````markdown
다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

```prompt
You are an agent that analyzes tasks.
```

```prompt
작업을 분석하는 에이전트입니다.
```
````

- `한국어 의미:`와 같은 레이블을 사용하지 않습니다.
- 한국어 프롬프트 안에 불필요한 Markdown 장식(`**`, 백틱 등)을 넣지 않습니다.
- URL, 자리표시자, 슬래시 참조 및 필수 식별자를 보존합니다.

## 사람 친화적인 입력값

실제 영문 입력값은 그대로 유지하고 한국어 설명을 코드 범위 밖에 표기합니다.

```markdown
**`US Benefits Assistant`(미국 복리후생 지원 담당자)**
```

학습자는 `US Benefits Assistant`만 입력하며, 한국어 텍스트는 설명용입니다. 스키마 이름, 변수 이름, 식, 필터, URL, 파일 경로, Excel 열 이름 및 종속 선택값에는 이 방식을 적용하지 않습니다.

## 동기화 절차

1. 마지막 동기화 커밋과 현재 원본을 비교합니다.
2. 추가, 삭제, 이름 변경 및 내용 변경을 분류합니다.
3. 영문 diff를 의미 단위로 한국어 문서에 반영합니다.
4. 기존 한국어 파일을 영문 파일로 덮어쓰지 않습니다.
5. 변경된 이미지와 필수 자산을 동일한 상대 경로로 복사합니다.
6. 전체 상대 경로 구조와 링크를 검증합니다.
7. 모든 변경을 반영한 후에만 기준 커밋과 검토일을 갱신합니다.

```powershell
git diff 7eafc3c339d31a8fd125af71ee7fd2465c083be8..HEAD -- Instructions
python .github\skills\mslearn-korean-localization\scripts\validate_translation.py --repo .
```

## 동기화 상태

| 항목 | 값 |
|---|---|
| 저장소 | `hahaysh/mslearn-ai-agents-kr` |
| 기준 커밋 | `7eafc3c339d31a8fd125af71ee7fd2465c083be8` |
| 마지막 검토일 | 2026-09-27 |

## 파일별 번역 상태

### Instructions/Exercises

| 파일 | 상태 |
|---|---|
| `01-build-agent-portal-and-vscode.md` | 완료 |
| `02-agent-custom-tools.md` | 완료 |
| `03-mcp-integration.md` | 완료 |
| `04-integrate-agent-with-foundry-iq.md` | 완료 |
| `05a-m365-teams-integration.md` | 완료 |
| `05b-work-iq-integration.md` | 완료 |
| `06-build-workflow-ms-foundry.md` | 완료 |
| `07-agent-framework.md` | 완료 |
| `08-agent-framework-multi-agents.md` | 완료 |
| `09-multi-remote-agents-with-a2a.md` | 완료 |

### Instructions/Consolidated

| 파일 | 상태 |
|---|---|
| `A-build-and-extend-ai-agents.md` | 완료 |
| `A0-getting-started.md` | 완료 |
| `A1-create-and-ground-an-agent.md` | 완료 |
| `A2-connect-a-remote-mcp-server.md` | 완료 |
| `A3-call-your-agent-from-a-client-app.md` | 완료 |
| `A4-add-custom-function-tools.md` | 완료 |
| `A5-capstone-build-your-own-mcp-server.md` | 완료 |
| `A6-promote-your-assistant-to-a-hosted-agent.md` | 완료 |
| `B-integrate-agents-with-enterprise-knowledge-and-m365.md` | 완료 |
| `B0-getting-started.md` | 완료 |
| `B1-create-a-foundry-iq-knowledge-agent.md` | 완료 |
| `B2-publish-to-microsoft-teams.md` | 완료 |
| `B3-publish-to-microsoft-365-copilot.md` | 완료 |
| `B4-work-iq-workplace-intelligence.md` | 완료 |
| `C-build-multi-agent-solutions-with-agent-framework.md` | 완료 |
| `C0-getting-started.md` | 완료 |
| `C1-create-an-agent-with-a-tool.md` | 완료 |
| `C2-orchestrate-multiple-agents.md` | 완료 |
| `C3-connect-remote-agents-with-a2a.md` | 완료 |
| `C4-classify-and-route-a-ticket.md` | 완료 |
| `D-observe-evaluate-and-secure-agents.md` | 완료 |
| `D0-getting-started.md` | 완료 |
| `D1-trace-your-agent.md` | 완료 |
| `D2-evaluate-answer-quality.md` | 완료 |
| `D3-red-team-your-agent.md` | 완료 |

## 검증 체크리스트

- [x] 영문과 한국어 파일명이 대응하는가?
- [x] 영문과 한국어 파일의 상대 경로가 대응하는가?
- [x] YAML과 제목 구조가 유지되었는가?
- [x] 단계 번호가 유지되었는가?
- [x] 이미지와 링크 경로가 유지되었는가?
- [x] 참조되는 이미지와 필수 자산이 한국어 디렉터리에 존재하는가?
- [x] 원문의 코드 블록이 보존되었는가?
- [x] 영문과 한국어 프롬프트를 각각 복사할 수 있는가?
- [x] 기술 식별자와 식이 변경되지 않았는가?
- [x] 기존 영문 원본과 관련 없는 파일이 변경되지 않았는가?

## GitHub Pages

기본 공개 URL 형식:

```text
https://<owner>.github.io/<repository>/Instructions-kr/Exercises/<lab>.html
```

게시 완료: 이 저장소의 한국어 실습은 다음 사이트에 게시되어 있습니다.

- 사이트 홈: <https://hahaysh.github.io/mslearn-ai-agents-kr/>
- 실습(Exercises) 예: <https://hahaysh.github.io/mslearn-ai-agents-kr/Instructions-kr/Exercises/01-build-agent-portal-and-vscode.html>
- 워크숍(Consolidated) 예: <https://hahaysh.github.io/mslearn-ai-agents-kr/Instructions-kr/Consolidated/A1-create-and-ground-an-agent.html>

게시 구성:

- GitHub Pages 원본: `main` 브랜치 루트(`/`).
- 루트 `index.md`의 실습 목록 쿼리를 `/Instructions-kr/Exercises`로 지정하여 홈페이지에 한국어 실습이 표시됩니다.
- 마지막 확인: 홈페이지 및 대표 실습·워크숍 페이지, 미디어 이미지 모두 HTTP 200 응답을 확인했습니다(2026-09-27).

## 알려진 원문 문제

- **`A5-capstone-build-your-own-mcp-server.md` — 검증기 오탐(false positive)**: 검증 스크립트(`validate_translation.py`)의 링크 정규식이 Python 예제 코드 `local_functions[item.name](**kwargs)` 및 `functions_dict[item.name](**kwargs)`의 `[item.name](**kwargs)` 부분을 Markdown 링크로 잘못 인식하여 `broken local target: **kwargs` 오류 2건을 보고합니다. 이 코드는 영문 원본과 완전히 동일하며(원본에서도 같은 오탐이 발생), 번역으로 인한 문제가 아닙니다. 예제 코드를 훼손하지 않고는 해결할 수 없으므로 원문 코드 블록을 그대로 유지했습니다. 그 외 모든 검증 항목은 통과합니다.
- 위 항목을 제외하고, 번역을 가로막는 원문 오류는 발견되지 않았습니다. 추가로 발견 시 파일명과 함께 이 섹션에 기록합니다.
