---
lab:
    title: 'Work IQ - AI 에이전트를 위한 업무 환경 인텔리전스(선택 사항)'
    description: 'Work IQ 및 Model Context Protocol을 사용하여 Microsoft 365 업무 데이터를 활용하는 AI 에이전트를 빌드하고, 회의 준비, 프로젝트 추적 및 작업 항목을 처리합니다.'
    level: 300
    duration: 40
    islab: true
    status: 'released'
---

# Work IQ - AI 에이전트를 위한 업무 환경 인텔리전스

이 랩에서는 **Work IQ**를 사용하여 Microsoft 365 업무 데이터에 액세스하는 AI 에이전트를 빌드합니다. Work IQ는 Model Context Protocol(MCP)을 기반으로 구축된 Microsoft의 컨텍스트 인텔리전스 계층입니다. 실제 M365 데이터를 사용하여 회의를 준비하고, 프로젝트를 추적하고, 작업 항목을 추출하고, 업무 관련 질문에 답할 수 있는 업무 환경 인텔리전스 에이전트를 만듭니다.

이 랩은 약 **40**분이 걸립니다.

> **Note:** 이 랩은 Microsoft 365 Copilot 라이선스가 필요한 **선택 사항/고급 랩**입니다. 엔터프라이즈 학습자, Microsoft 직원 또는 M365 Copilot 액세스 권한이 있는 사용자를 위해 설계되었습니다. Copilot이 없는 표준 M365 계정으로는 작동하지 않습니다.

## 필수 구성 요소

이 랩을 시작하기 전에 다음이 준비되어 있는지 확인합니다.

- AI 에이전트 및 Model Context Protocol(MCP)에 대한 기본 이해
- **Microsoft 365 with Copilot License**
- Work IQ에 대한 IT 관리자 승인(조직 계정만 해당)
- 설치된 [Node.js 18](https://nodejs.org/en/download/) 이상
- 설치된 [Python 3.13](https://www.python.org/downloads/)
- 설치된 [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)(`az login`으로 인증됨)
- 쿼리할 활성 M365 데이터(메일, 모임, Teams 채팅)

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에 3.14 빌드가 없습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

> **Important:** Work IQ는 Microsoft 365 Copilot 사용 계정에서만 **작동합니다**. Copilot 없이는 이 랩을 완료할 수 없습니다.

## Work IQ 설치

1. 터미널 또는 명령 프롬프트를 엽니다.

2. npm을 통해 Work IQ를 전역으로 설치합니다.

    ```bash
   npm install -g @microsoft/workiq
    ```

3. 최종 사용자 사용권 계약에 동의합니다.

    ```bash
   workiq accept-eula
    ```

4. Work IQ 설치를 테스트합니다.

    ```bash
   workiq ask -q "What meetings do I have today?"
    ```

5. **테스트가 성공하는 경우** - M365 일정의 모임 정보가 표시됩니다. 다음 작업으로 계속 진행합니다.

6. **"Admin consent required"가 표시되는 경우:**

   - 명령이 동의 URL을 표시합니다.
   - IT 관리자에게 이 URL과 함께 다음 메시지를 보냅니다. "Microsoft Learn AI Agents 랩을 위해 Work IQ 액세스가 필요합니다."
   - 관리자 승인을 기다린 다음 테스트 명령을 다시 시도합니다.

7. **"No M365 Copilot license"가 표시되는 경우:**

   - 안타깝지만 Copilot 라이선스가 없으면 이 랩을 완료할 수 없습니다.
   - 그래도 지침을 읽어 개념을 이해할 수 있습니다.
   - 이 랩은 선택 사항으로 간주하고 Copilot 액세스 권한이 있을 때 다시 진행합니다.

## Visual Studio Code에서 앱 개발 준비

이제 Visual Studio Code를 사용하여 앱을 개발하겠습니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되어 있습니다.

1. Visual Studio Code를 시작하고 터미널 창을 엽니다.

2. 명령을 입력하여 리포지토리를 로컬 폴더(어느 폴더든 상관없음)에 복제합니다.

    ```bash
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

3. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **Note**: Visual Studio Code에서 열고 있는 코드를 신뢰할지 묻는 팝업 메시지가 표시되면 **Yes, I trust the authors**를 선택하여 계속합니다.

4. 리포지토리의 Python 코드 프로젝트를 지원하기 위한 추가 파일이 설치되는 동안 기다립니다(메시지가 표시되는 경우).

    > **Note**: 빌드 및 디버그에 필요한 자산을 설치하라는 메시지가 표시되면 **Not Now**를 선택합니다.

5. **Explorer** 창에서 **Labfiles/05b-work-iq-integration/Python** 폴더를 확장합니다.

    제공된 파일에는 애플리케이션 코드, 구성 설정 및 에이전트 클라이언트 시작 코드가 포함되어 있습니다.

6. 터미널에서 Python 가상 환경을 만드는 명령을 입력합니다.

    ```bash
   python -m venv venv
    ```

7. 가상 환경을 활성화합니다.

   **Windows:**

    ```bash
   venv\Scripts\activate
    ```

   **macOS/Linux:**

    ```bash
   source venv/bin/activate
    ```

8. 필요한 Python 패키지를 설치합니다.

    ```bash
   pip install -r requirements.txt
    ```

9. `.env` 파일을 구성합니다.

   랩 폴더에서 `.env` 파일을 열고 Foundry 프로젝트 엔드포인트로 업데이트합니다.

    ```env
   PROJECT_ENDPOINT=https://your-project.services.ai.azure.com/api/projects/your-id
   MODEL_DEPLOYMENT_NAME=gpt-5
    ```

   > **Tip:** 엔드포인트를 가져오려면 VS Code에서 **Foundry Toolkit** 확장을 열고 활성 프로젝트를 마우스 오른쪽 단추로 클릭한 다음 **Copy Endpoint**를 선택합니다. Foundry Toolkit은 Foundry Toolkit for VS Code 확장에 포함되어 있습니다.

### 설정 확인

다음이 준비되어 있는지 확인합니다.

- Work IQ가 설치되어 있고 액세스 가능함(`workiq --version` 작동)
- 관리자 동의가 승인됨(또는 Copilot이 포함된 개인 M365 계정)
- `workiq_lab.py` - 기본 대화형 애플리케이션
- `requirements.txt` - 설치된 Python 종속성
- 프로젝트 엔드포인트로 구성된 `.env` 파일

## 업무 환경 인텔리전스 시나리오 살펴보기

이 연습에서는 Work IQ 도구가 있는 단일 AI 에이전트를 사용하여 다섯 가지 업무 환경 인텔리전스 시나리오를 보여 주는 통합 대화형 애플리케이션을 실행합니다.

### 랩 애플리케이션 시작

1. 가상 환경이 활성화된 상태로 랩 디렉터리에 있는지 확인합니다.

2. 랩 애플리케이션을 실행합니다.

    ```bash
   python workiq_lab.py
    ```

3. 애플리케이션은 다음을 수행합니다.
   - Work IQ 설정 유효성 검사
   - Microsoft Foundry 프로젝트에 연결
   - Work IQ MCP 클라이언트 초기화
   - 업무 환경 인텔리전스 에이전트 만들기
   - 5개 시나리오가 있는 대화형 메뉴 표시

### 회의 준비 시나리오

이 시나리오는 관련 컨텍스트를 수집하여 회의를 준비하는 데 도움을 줍니다.

1. 기본 메뉴에서 **1 - Meeting Prep**을 선택합니다.

2. 메시지가 표시되면 다음과 같은 모임 주제 또는 시간을 입력합니다.
   - "my 2pm meeting"
   - "Q4 Planning session"
   - "team standup"

3. 에이전트는 다음을 수행합니다.
   - 모임 세부 정보(시간, 참석자, 의제)를 찾습니다.
   - 해당 주제에 대한 최근 메일을 검색합니다.
   - 이 주제와 관련된 이전 모임을 찾습니다.
   - 핵심 사항과 결정을 요약합니다.
   - 토론 포인트를 제안합니다.

4. 출력을 검토하고 다음을 확인합니다.
   - 출처가 인용되는 방식(메일, 모임, 날짜)
   - 에이전트가 여러 원본의 정보를 종합하는 방식
   - 수동 검색과 비교해 절약되는 시간

**Reflection:** 이메일과 일정에서 직접 검색하는 것과 어떻게 다른가요?

### 프로젝트 상태 시나리오

이 시나리오는 업무 도구 전반에서 프로젝트 업데이트를 추적합니다.

1. 기본 메뉴에서 **2 - Project Status**를 선택합니다.

2. 다음과 같이 작업 중인 프로젝트 이름을 입력합니다.
   - "Website redesign"
   - "Q1 OKRs"
   - "Customer onboarding"

3. 에이전트는 다음을 수행합니다.
   - 프로젝트와 관련된 메일 및 Teams 메시지를 검색합니다.
   - 관련 모임과 그 결과를 찾습니다.
   - 최근 결정 및 변경 사항을 식별합니다.
   - 언급된 차단 요소 또는 문제를 나열합니다.
   - 다음 단계와 기한을 요약합니다.

4. 결과를 분석합니다.
   - 상태 업데이트가 얼마나 포괄적인가요?
   - 에이전트가 어떤 원본을 사용했나요?
   - 기존 API로도 이를 빌드할 수 있을까요? 개발 노력의 차이는 무엇일까요?

### 작업 항목 시나리오

이 시나리오는 다양한 원본에서 미해결 작업을 추출합니다.

1. 기본 메뉴에서 **3 - Action Items**를 선택합니다.

2. 시간 범위를 선택합니다(또는 "this week"의 경우 Enter를 누름).
   - "today"
   - "last 3 days"
   - "this month"

3. 에이전트는 다음을 수행합니다.
   - 모임 메모에서 할당된 작업 항목을 검색합니다.
   - 사용자에게 전송된 작업 관련 메일을 찾습니다.
   - 사용자가 언급된 Teams 메시지를 확인합니다.
   - 기한이 있는 항목을 식별합니다.
   - 가능한 경우 긴급도에 따라 우선순위를 지정합니다.

4. 출력을 검토합니다.
   - 모든 작업 항목이 캡처되었나요?
   - 우선순위 지정은 얼마나 정확한가요?
   - 작업 항목은 어디에서 발견되었나요(모임, 메일, Teams)?

### 결합 인텔리전스 시나리오

이 시나리오는 Work IQ(업무 데이터)와 Foundry IQ(기술 자료)를 **함께** 사용하는 방법을 보여 줍니다.

> **Note:** 이 시나리오에는 Foundry 프로젝트에서 인덱싱된 기술 자료가 포함된 Azure AI Search 구성이 필요합니다.

1. 기본 메뉴에서 **4 - Combined Intelligence**를 선택합니다.

2. 업무 환경 논의와 공식 문서 모두에 존재하는 주제를 입력합니다.
   - "remote work policy"
   - "expense reporting"
   - "security guidelines"

3. 에이전트는 다음을 수행합니다.
   - 업무 데이터(Work IQ) 검색: 메일, 모임, Teams 논의
   - 기술 자료(Foundry IQ) 검색: 공식 문서, 정책, 절차
   - 업무 환경 논의와 공식 문서를 비교합니다.
   - 차이 또는 불일치를 식별합니다.
   - 레이블이 지정된 출처와 함께 포괄적인 요약을 제공합니다.

4. 두 관점을 비교합니다.
   - 공식적으로 문서화된 내용과 비공식적으로 논의된 내용은 무엇인가요?
   - 모순이 있나요?
   - 어느 원본이 더 최신인가요?

**Key Insight:**

- **Work IQ**는 사람들이 실제로 무엇을 하고 말하는지 알려 줍니다.
- **Foundry IQ**는 공식적으로 문서화된 내용을 알려 줍니다.
- **Together**는 의사 결정을 위한 완전한 컨텍스트를 제공합니다.

### 사용자 지정 쿼리 시나리오

이 시나리오에서는 직접 질문을 사용하여 업무 데이터를 탐색할 수 있습니다.

1. 기본 메뉴에서 **5 - Custom Query**를 선택합니다.

2. 다양한 유형의 업무 관련 질문을 시도합니다.

   **Email searches:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Find emails about the budget from my manager
    ```

    ```prompt
   내 관리자가 보낸 예산 관련 메일을 찾아줘
    ```

   **Meeting summaries:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What was decided in yesterday's standup?
    ```

    ```prompt
   어제 스탠드업에서 무엇이 결정되었나요?
    ```

   **Team activity:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What did the engineering team discuss this week?
    ```

    ```prompt
   이번 주 엔지니어링 팀은 무엇을 논의했나요?
    ```

   **Document discovery:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Show me shared documents about security policies
    ```

    ```prompt
   보안 정책에 대한 공유 문서를 보여줘
    ```

3. 다음을 실험합니다.
   - 다양한 시간 범위
   - 다양한 데이터 원본(메일 vs. 모임 vs. Teams)
   - 다양한 구체성 수준
   - 결과를 구체화하기 위한 후속 질문

4. 잘 작동하는 부분을 기록합니다.
   - 일반적으로 구체적인 쿼리가 모호한 쿼리보다 더 잘 작동합니다.
   - 시간 범위를 포함하면 관련성이 향상됩니다.
   - 이름과 키워드는 결과 범위를 좁히는 데 도움이 됩니다.

## 탐색 및 실험

이제 모든 시나리오를 완료했으므로 5~10분 동안 직접 탐색해 봅니다.

### 에지 사례 테스트

1. 가지고 있지 않은 데이터에 대한 쿼리를 시도합니다. 에이전트가 어떻게 응답하나요?

2. 모호한 질문을 합니다. 에이전트가 어떻게 처리하나요?

3. 매우 오래된 정보를 검색합니다. 제한은 무엇인가요?

### 다양한 쿼리 스타일 살펴보기

1. **Very specific**: "Find the email from John about Q3 budget sent on January 15th"

2. **Very broad**: "Tell me about recent developments"

3. **Comparative**: "Compare this week's discussions to last week's"

### Work IQ 기능 보기

기본 메뉴에서 **6 - View Work IQ Capabilities**를 선택하여 다음을 검토합니다.

- 아키텍처 개요
- 사용 가능한 데이터 원본
- 보안 및 개인 정보 보호 모델
- Work IQ vs. Foundry IQ 비교
- 일반적인 사용 사례

## 코드 이해

이 랩에서 사용되는 핵심 패턴을 살펴보겠습니다.

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

영구 연결을 유지하는 대신, 작업마다 새 MCP 세션이 열립니다. `StdioServerParameters`는 매번 Work IQ MCP 서버 하위 프로세스를 시작하는 데 사용되는 명령과 인수를 저장합니다.

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
    agent_name="workplace-intelligence-agent",
    definition=PromptAgentDefinition(
        model=self.model_deployment,
        instructions="You are a workplace intelligence assistant...",
        tools=workiq_tools  # Work IQ tools added here
    )
)

# Keep a map of raw tools for lookup during execution
self.raw_tools_map = {tool.name: tool for tool in raw_tools}
```

각 MCP 도구는 `FunctionTool` 개체로 래핑되어 `PromptAgentDefinition`에 전달됩니다. 원시 도구 맵을 사용하면 에이전트가 이름으로 도구를 호출할 때 효율적으로 조회할 수 있습니다.

### 패턴 3: Responses API로 쿼리 실행

```python
# Create conversation
conversation = self.openai_client.conversations.create(
    items=[{"type": "message", "role": "user", "content": query}]
)

# Create response with agent
response = self.openai_client.responses.create(
    conversation=conversation.id,
    extra_body={"agent_reference": {"name": self.agent.name, "type": "agent_reference"}}
)
```

이는 더 깔끔한 에이전트 실행을 위해 Responses API 패턴(이전 Runs/Threads 패턴이 아님)을 사용합니다.

### 패턴 4: 도구 호출 루프

초기 응답 후 에이전트가 하나 이상의 Work IQ 도구 호출을 요청할 수 있습니다. 이러한 호출을 실행하고 다시 전달하여 대화를 계속해야 합니다.

```python
from openai.types.responses.response_input_param import FunctionCallOutput

while True:
    if response.status == "failed":
        break

    input_list = []
    for item in response.output:
        if item.type == "function_call":
            kwargs = json.loads(item.arguments)

            # Call the Work IQ tool via MCP
            async def _execute():
                async with stdio_client(self.workiq_server_params) as (read, write):
                    async with ClientSession(read, write) as session:
                        await session.initialize()
                        return await session.call_tool(item.name, kwargs)

            result = asyncio.run(_execute())
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=result.content[0].text,
                )
            )

    if input_list:
        # Send tool results back and continue
        response = self.openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": self.agent.name, "type": "agent_reference"}}
        )
    else:
        break  # No more tool calls - final response ready
```

에이전트가 대기 중인 함수 호출이 없는 응답을 생성할 때까지 루프가 계속되며, 이때 `response.output_text`에 최종 답변이 포함됩니다.

## 정리

랩은 종료할 때 에이전트를 자동으로 정리합니다.

```python
self.openai_client.agents.delete_version(
    agent_name=self.agent.name,
    version=self.agent.version
)
```

이 랩에서는 Azure 리소스가 만들어지지 않으므로(Work IQ는 M365 라이선스를 사용함) 추가 정리가 필요하지 않습니다.

## 문제 해결

### "Work IQ command not found"

**Solution:** Work IQ를 설치합니다.

```bash
npm install -g @microsoft/workiq
```

### "Admin consent required"

**Solution:**

1. `workiq mcp`를 실행하여 동의 URL을 가져옵니다.
2. 승인을 위해 IT 관리자에게 보냅니다.
3. 또는 Copilot이 포함된 개인 M365 계정을 사용합니다.

### "No M365 Copilot license"

**Solution:** 이 랩에는 Copilot이 필요합니다. 다음 중 하나를 수행합니다.

- M365 Copilot 라이선스 구매($30/월)
- Copilot이 있는 조직 계정 사용
- 실습 없이 개념을 이해하기 위해 랩 읽기

### "MCP server not responding"

**Solution:** Work IQ를 직접 테스트합니다.

```bash
workiq ask -q "What meetings do I have?"
```

실패하면 다시 설치합니다.

```bash
npm install -g @microsoft/workiq
```

### "No data returned"

**Solution:**

- M365 계정에 메일, 모임, Teams 활동이 있는지 확인합니다.
- 더 넓은 쿼리를 시도합니다.
- 쿼리가 실제 데이터와 일치하는지 확인합니다.
