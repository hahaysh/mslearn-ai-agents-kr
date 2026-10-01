---
title: '작업 4 – Work IQ: Microsoft 365 신호를 에이전트에 가져오기'
lab:
    title: '작업 4 – Work IQ: Microsoft 365 신호를 에이전트에 가져오기'
    description: '회의 준비, 프로젝트 추적, 작업 항목을 위해 Work IQ와 Model Context Protocol을 사용하여 Microsoft 365 업무 데이터에 액세스하는 에이전트를 빌드합니다.'
    type: 'task'
    parent: 'B'
    order: 4
    section: 'optional'
    difficulty: 4
    duration: 40
    access: 'gated'
    requires: 'A Microsoft 365 Copilot licence, IT admin consent for Work IQ, and Node.js 18 or later'
    verify: 'Run the command below. If it returns your calendar you''re ready; if it reports missing consent or no Copilot licence, skip this task.'
    verify_command: 'npm install -g @microsoft/workiq && workiq accept-eula && workiq ask -q "What meetings do I have today?"'
    level: 400
    concepts: 'Work IQ, Microsoft 365, Model Context Protocol (MCP), 함수 도구'
    status: 'draft'
---

# 작업 4 — Work IQ: Microsoft 365 신호를 에이전트에 가져오기

*이 작업은 **엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합** 랩의 일부입니다. 처음 오셨나요? [시작하기](B0-getting-started.md)부터 시작하세요.*

<!-- BEGIN GENERATED: gated-notice - do not edit by hand; run: python tools/generate_lab_blocks.py -->

> ### 시작하기 전에 액세스 확인
>
> **This task needs:** A Microsoft 365 Copilot licence, IT admin consent for Work IQ, and Node.js 18 or later.
>
> 아래 명령을 실행합니다. 일정이 반환되면 준비된 것입니다. 동의 누락 또는 Copilot 라이선스 없음이 보고되면 이 작업을 건너뜁니다.

```
npm install -g @microsoft/workiq && workiq accept-eula && workiq ask -q "What meetings do I have today?"
```

> **Don't have it?** 이 작업을 건너뜁니다. 이 랩의 다른 어떤 내용도 이 작업에 의존하지 않으며, 작동 방식을 알아보기 위해 단계를 읽어볼 수는 있습니다.

<!-- END GENERATED: gated-notice -->

> **Set up (start here):** 이 작업에는 Foundry 프로젝트(배포된 모델 포함)와 시작 코드가 필요합니다. 아직 완료하지 않았다면 [시작하기](B0-getting-started.md)를 완료하여 프로젝트를 만들고, 코드를 복제하고, `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에서 설정합니다. 그런 다음 VS Code에서 연 `Python` 폴더에서 준비 상태를 확인합니다.

```
python ../setup/check_env.py --task 4
```

> **Continuing from a previous task?** 이전 작업에서 프로젝트, 가상 환경, `.env`가 이미 설정되어 있다면 Work IQ(아래)만 설치한 뒤 **업무 인텔리전스 시나리오 살펴보기**로 바로 이동합니다.

---

작업 1~3에서는 에이전트를 *문서* 기반으로 만들었습니다. 이 작업에서는 **Work IQ**를 사용하여 에이전트를 **라이브 Microsoft 365 신호**(전자 메일, 회의, Teams 메시지)에 연결합니다. 회의를 준비하고, 프로젝트를 추적하고, 실제 M365 데이터에서 작업 항목을 추출할 수 있는 Caldova Traders 업무 인텔리전스 에이전트를 빌드합니다.

<style>
/* "Ask Anton" just-in-time concept blocks */
details.concept { margin:.6rem 0 1rem; }
details.concept > summary { display:inline-block; cursor:pointer; list-style:none;
  font-size:.85em; font-weight:600; color:#6b4ba1; background:#6b4ba112;
  border:1px solid #6b4ba133; border-radius:999px; padding:.2em .7em; }
details.concept > summary::-webkit-details-marker { display:none; }
details.concept > summary::before { content:"Ask Anton: "; font-weight:700;
  padding-left:1.5em;
  background:url("../Media/anton-avatar.png") left center / 1.25em 1.25em no-repeat; }
details.concept > summary:hover { background:#6b4ba1; color:#fff; border-color:#6b4ba1; }
details.concept[open] > summary { border-bottom-left-radius:0; border-bottom-right-radius:0; }
details.concept .concept-body { border:1px solid #6b4ba133; border-top:none;
  border-radius:0 8px 8px 8px; padding:.6rem .9rem; background:#6b4ba108; font-size:.95em; }
</style>

<details markdown="1" class="concept">
<summary>Work IQ란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

**Work IQ**는 **Model Context Protocol (MCP)** 서버로 노출되는 Microsoft 365용 Microsoft의 컨텍스트 인텔리전스 계층입니다. 에이전트가 전자 메일, 일정, Teams 메시지, 문서와 같은 업무 데이터에 권한 인식 방식으로 액세스할 수 있게 하여, 사람들이 실제로 무엇을 하고 말하는지에 대해 추론할 수 있도록 합니다. 이는 **Foundry IQ**(선별된 지식)를 **라이브 업무 신호**로 보완합니다.

</div>
</details>

## Work IQ 설치

1. 터미널 또는 명령 프롬프트를 엽니다.

2. npm을 통해 Work IQ를 전역으로 설치합니다.

   ```
   npm install -g @microsoft/workiq
   ```

3. 최종 사용자 사용권 계약에 동의합니다.

   ```
   workiq accept-eula
   ```

4. Work IQ 설치를 테스트합니다.

   ```
   workiq ask -q "What meetings do I have today?"
   ```

5. **테스트가 성공하는 경우** - M365 일정의 회의 정보가 표시됩니다. 다음 섹션으로 계속 진행합니다.

6. **"Admin consent required"가 표시되는 경우:**

   - 명령이 동의 URL을 표시합니다.
   - IT 관리자에게 이 URL을 보내고 다음 메시지를 함께 전달합니다. "Microsoft Learn AI Agents lab을 위해 Work IQ 액세스가 필요합니다."
   - 관리자 승인을 기다린 다음 테스트 명령을 다시 시도합니다.

7. **"No M365 Copilot license"가 표시되는 경우:**

   - 안타깝지만 Copilot 라이선스 없이는 이 작업을 완료할 수 없습니다.
   - 개념을 이해하기 위해 지침을 계속 읽을 수는 있습니다.

## 앱 준비

Work IQ 앱은 시작 코드에 **완성된 상태**로 제공됩니다. 그대로 실행합니다.

1. `Python` 폴더를 열고 [시작하기](B0-getting-started.md)의 가상 환경을 활성화합니다.

    ```
    .\labenv\Scripts\Activate.ps1
    ```

1. `.env`에 `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`이 설정되어 있는지 확인합니다(Work IQ는 모델 배포를 사용하여 에이전트를 실행합니다).

1. **workiq_lab.py**를 검토합니다. 이 파일은 다음을 수행합니다.
    - Work IQ 설치 유효성 검사
    - Microsoft Foundry 프로젝트에 연결
    - Work IQ MCP 클라이언트(`npx -y @microsoft/workiq mcp`) 초기화
    - Work IQ 도구가 포함된 `caldova-workplace-agent` 만들기
    - 다섯 가지 시나리오가 있는 대화형 메뉴 표시

## 업무 인텔리전스 시나리오 살펴보기

1. Azure에 로그인한 다음 앱을 실행합니다.

    ```
    az login
    ```

    ```
    python workiq_lab.py
    ```

애플리케이션이 Work IQ와 Foundry 프로젝트에 연결된 다음 다섯 가지 시나리오 메뉴를 표시합니다.

### 회의 준비 시나리오

1. 기본 메뉴에서 **1 - Meeting Prep**을 선택합니다.

2. 메시지가 표시되면 다음과 같은 회의 주제 또는 시간을 입력합니다.
   - "my 2pm meeting"
   - "Spring Catalog Planning session"
   - "site operations standup"

3. 에이전트가 회의 세부 정보를 찾고, 주제와 관련된 최근 전자 메일을 검색하고, 이전 회의를 찾아보고, 핵심 사항을 요약하고, 논의할 내용을 제안합니다.

4. 출력을 검토하고 원본(전자 메일, 회의, 날짜)이 인용되는 방식과 에이전트가 여러 원본의 정보를 종합하는 방식을 확인합니다.

### 프로젝트 상태 시나리오

1. 기본 메뉴에서 **2 - Project Status**를 선택합니다.

2. 작업 중인 프로젝트 이름을 입력합니다. 예시는 다음과 같습니다.
   - "Spring Catalog Launch"
   - "Capacity Review"
   - "Supplier onboarding"

3. 에이전트가 전자 메일과 Teams 메시지를 검색하고, 관련 회의를 찾고, 최근 결정과 차단 요소를 식별하고, 다음 단계와 마감일을 요약합니다.

### 작업 항목 시나리오

1. 기본 메뉴에서 **3 - Action Items**를 선택합니다.

2. 시간 범위를 선택합니다(또는 "this week"의 경우 Enter 키를 누릅니다). 예: "today", "last 3 days", "this month".

3. 에이전트가 회의록, 작업 관련 전자 메일, Teams 멘션을 검색하고, 마감일이 있는 항목을 식별하고, 긴급도에 따라 우선순위를 지정합니다.

### 결합 인텔리전스 시나리오

이 시나리오는 Work IQ(업무 데이터)와 Foundry IQ(지식 베이스)를 **둘 다** 함께 사용하는 방법을 보여줍니다.

> **Note**: 이 시나리오에는 인덱싱된 지식 베이스로 구성된 Foundry IQ(Azure AI Search)가 프로젝트에 필요합니다. 예를 들어 [작업 1](B1-create-a-foundry-iq-knowledge-agent.md)의 Caldova 지식 베이스가 해당됩니다.

1. 기본 메뉴에서 **4 - Combined Intelligence**를 선택합니다.

2. 업무 대화와 공식 문서 모두에 존재하는 주제를 입력합니다.
   - "capacity request and transfer policies"
   - "supplier lead times"
   - "contract manufacturing transfers"

3. 에이전트가 업무 데이터(Work IQ) **및** 지식 베이스(Foundry IQ)를 검색하고, 비공식 논의와 공식 문서를 비교하고, 차이를 식별하고, 레이블이 지정된 원본과 함께 포괄적인 요약을 제공합니다.

**핵심 인사이트:**

- **Work IQ**는 사람들이 실제로 무엇을 하고 말하는지 알려줍니다.
- **Foundry IQ**는 공식적으로 문서화된 내용을 알려줍니다.
- **함께 사용하면** 의사 결정을 위한 완전한 컨텍스트를 제공합니다.

### 사용자 지정 쿼리 시나리오

1. 기본 메뉴에서 **5 - Custom Query**를 선택합니다.

2. 다양한 유형의 업무 질문을 시도합니다.

    ```
    Find emails about the spring catalog from my manager
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    내 관리자가 보낸 봄 카탈로그 관련 전자 메일을 찾아줘
    ```

    ```
    What was decided in yesterday's site operations standup?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    어제 사이트 운영 스탠드업에서 무엇이 결정되었나요?
    ```

    ```
    Show me shared documents about supplier lead times
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    공급업체 리드 타임에 대한 공유 문서를 보여줘
    ```

3. 다양한 시간 범위, 데이터 원본, 후속 질문을 실험하여 결과를 구체화합니다.

### Work IQ 기능 보기

기본 메뉴에서 **6 - View Work IQ Capabilities**를 선택하여 아키텍처, 데이터 원본, 보안 모델, Work IQ와 Foundry IQ 비교를 검토합니다. 종료하려면 **0**을 선택합니다. 앱은 나가는 과정에서 `caldova-workplace-agent` 버전을 삭제합니다.

## 코드 이해

`workiq_lab.py`에서 사용되는 주요 패턴을 살펴보겠습니다.

### 패턴 1: Work IQ MCP 클라이언트 초기화

```python
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

# Store server parameters for reuse
self.workiq_server_params = StdioServerParameters(
    command="npx",
    args=["-y", "@microsoft/workiq", "mcp"]
)

# Fetch available tools from Work IQ MCP server
async def _fetch():
    async with stdio_client(self.workiq_server_params) as (read, write):
        async with ClientSession(read, write) as session:
            await session.initialize()
            tools_result = await session.list_tools()
            return tools_result.tools

raw_tools = asyncio.run(_fetch())
```

지속적인 연결을 유지하는 대신, 작업마다 새 MCP 세션을 엽니다. `StdioServerParameters`는 매번 Work IQ MCP 서버 하위 프로세스를 시작하는 데 사용되는 명령과 인수를 저장합니다.

### 패턴 2: Work IQ 도구로 에이전트 만들기

```python
from azure.ai.projects.models import PromptAgentDefinition, FunctionTool

# Convert MCP tools to FunctionTool objects
workiq_tools = [
    FunctionTool(
        name=tool.name,
        description=tool.description,
        parameters=tool.inputSchema,
    )
    for tool in raw_tools
]

# Create agent with Work IQ tools
self.agent = self.project_client.agents.create_version(
    agent_name="caldova-workplace-agent",
    definition=PromptAgentDefinition(
        model=self.model_deployment,
        instructions="You are a workplace intelligence assistant for Caldova staff...",
        tools=workiq_tools  # Work IQ tools added here
    )
)
```

각 MCP 도구는 `FunctionTool`로 래핑되어 `PromptAgentDefinition`에 전달됩니다.

### 패턴 3: 도구 호출 루프

초기 응답 후 에이전트가 하나 이상의 Work IQ 도구 호출을 요청할 수 있습니다. 이러한 호출을 실행하고 다시 전달하여 대화를 계속합니다.

```python
from openai.types.responses.response_input_param import FunctionCallOutput

while True:
    if response.status == "failed":
        break

    input_list = []
    for item in response.output:
        if item.type == "function_call":
            kwargs = json.loads(item.arguments)
            result = self._call_workiq_tool(item.name, kwargs)
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=result.content[0].text,
                )
            )

    if input_list:
        response = self.openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": self.agent.name, "type": "agent_reference"}}
        )
    else:
        break  # No more tool calls - final response ready
```

에이전트가 보류 중인 함수 호출이 없는 응답을 생성할 때까지 루프가 계속됩니다. 이 시점에서 `response.output_text`에는 최종 답변이 포함됩니다.

> ✅ **Checkpoint**: **라이브 Microsoft 365 신호**를 Work IQ를 통해 추론에 가져오는 에이전트를 빌드했고, 이 에이전트가 작업 1의 문서 기반 에이전트를 어떻게 보완하는지 확인했습니다.

## 정리

종료하면 앱이 `caldova-workplace-agent` 버전을 삭제합니다. Work IQ는 Azure 리소스를 만드는 대신 M365 라이선스를 사용하므로, 이 작업에서 제거할 다른 항목은 없습니다. 완료되면 `deactivate`를 입력하여 가상 환경을 종료합니다.

## 문제 해결

**"Work IQ command not found"** — Work IQ를 설치합니다: `npm install -g @microsoft/workiq`

**"Admin consent required"** — `workiq mcp`를 실행하여 동의 URL을 가져오고 IT 관리자에게 보내거나, Copilot이 있는 개인 M365 계정을 사용합니다.

**"No M365 Copilot license"** — 이 작업에는 Copilot이 필요합니다. M365 Copilot 라이선스가 있는 계정을 사용하거나 랩을 읽으며 개념을 이해합니다.

**"MCP server not responding"** — `workiq ask -q "What meetings do I have?"`로 Work IQ를 직접 테스트합니다. 실패하면 `npm install -g @microsoft/workiq`로 다시 설치합니다.

**"No data returned"** — M365 계정에 전자 메일, 회의, Teams 활동이 있는지 확인하고 더 광범위한 쿼리를 시도합니다.

---

**[랩 개요](B-integrate-agents-with-enterprise-knowledge-and-m365.md)로 돌아가기.**

