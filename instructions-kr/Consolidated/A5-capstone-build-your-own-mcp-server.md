---
title: '작업 5 – 종합 과제: 나만의 MCP 서버 빌드'
lab:
    title: '작업 5 – 종합 과제: 나만의 MCP 서버 빌드'
    description: '종합 과제: 나만의 MCP 서버를 빌드하고 함수 도구와 결합하여 하나의 Caldova 도우미를 만듭니다.'
    type: 'task'
    parent: 'A'
    order: 5
    section: 'optional'
    difficulty: 4
    duration: 35
    access: 'open'
    level: 400
    concepts: 'MCP 서버, 도구 오케스트레이션, Microsoft Agent Framework'
    status: 'draft'
---

# 작업 5 — 종합 과제: 나만의 MCP 서버 빌드

***AI 에이전트 빌드 및 확장** 랩의 일부입니다. 처음 오셨나요? [시작하기](A0-getting-started.md)부터 진행합니다.*

> **Set up (start here):** 이것은 **종합 과제**입니다. Foundry 프로젝트와 스타터 코드가
> 필요합니다. 아직 완료하지 않았다면 [시작하기](A0-getting-started.md)를 완료하여 프로젝트를
> 만들고, 코드를 복제하고, `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에
> 설정합니다. 이 작업은 [작업 4](A4-add-custom-function-tools.md)의 `functions.py`를
> 재사용합니다. 해당 파일은 이미 스타터 폴더에 있으므로 작업 4를 완료하지 않아도 됩니다.
> 그런 다음 VS Code에서 연 `Python` 폴더에서 확인합니다.

```
python ../setup/check_env.py --task 5
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 이전 작업을 막 완료했다면
> 프로젝트, 가상 환경, `.env`가 이미 설정되어 있습니다. 아래 **설정**으로 바로 이동하여
> `server.py`와 `client.py` 편집을 시작합니다.

---

**목표**: **나만의** 도구를 MCP 서버에서 호스팅한 다음, 랩의 내용을 하나의
**Caldova Supply Chain Assistant**로 통합합니다. 즉 **생산 능력을 계획하고 이전 비용을 추정**하는
에이전트(작업 4의 함수 도구)와, 여기에서 호스팅하는 도구로 **실시간 자재 재고와 소비량을
확인**하는 에이전트를 하나로 결합합니다.

이 작업은 로컬 Python 함수와 MCP 서버가 제공하는 도구를 결합합니다. `respond()`에서 각 호출을
올바른 위치로 라우팅합니다.

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
<summary>도구는 언제 로컬에서 실행하고 언제 MCP 서버에서 실행해야 하나요?</summary>
<div class="concept-body" markdown="1">

로컬 함수는 애플리케이션과 같은 프로세스에서 실행됩니다. 애플리케이션이 소유한 로직에
적합합니다. MCP 서버는 표준 프로토콜을 통해 도구를 노출하므로, 여러 에이전트와 클라이언트가
도구를 검색하고 재사용할 수 있습니다. 이 작업에서는 생산 능력 함수가 로컬에서 실행되고,
공유 재고 함수는 MCP 서버에서 실행됩니다.

</div>
</details>

Caldova의 세 공장은 Ashford, Brightwater, Calderwood입니다. 코드는 공장 매개 변수 이름으로
`site`를 사용합니다.

> **How this builds on Task 4**: 이 종합 과제는 작업 4의 생산 능력 플래너 도구와 새 MCP 서버를
> *결합*합니다. 작업 4를 완료하지 않아도 됩니다. 해당 도구
> (`next_available_slot`, `calculate_transfer_cost`, `generate_capacity_report`)는 `client.py`에
> 준비된 상태로 제공되므로, 새 작업인 MCP 서버 호스팅과 하나의 에이전트에서 두 도구 세트를
> *결합*하는 데 집중할 수 있습니다. (이미 작업 4를 완료했다면 더 좋습니다. 익숙할 것입니다.)

**설정:**

1. `Labfiles/A-build-and-extend-ai-agents/Python` 폴더에서 가상 환경
    (`.\labenv\Scripts\Activate.ps1`)을 활성화하고 **.env**에 `PROJECT_ENDPOINT`와
    `MODEL_DEPLOYMENT_NAME`이 있는지 확인합니다([시작하기](A0-getting-started.md) 참조).
    **server.py**와 **client.py**를 편집합니다.

> **Try it first**: 각 파일의 주석을 사용하여 **server.py**와 **client.py**를 연결합니다.
> 진행하면서 생각해 봅니다. 진단 출력은 왜 `stderr`로 보내거나(또는 억제하고) `stdout`으로는
> 보내지 않아야 하나요? 에이전트에 **두** 도구 세트가 모두 생기면, 주어진 `function_call`을
> 로컬 함수로 실행할지 MCP 도구로 실행할지 코드는 어떻게 알 수 있나요?

<details markdown="1" class="concept">
<summary>MCP 서버는 왜 stdout을 깨끗하게 유지해야 하나요?</summary>
<div class="concept-body" markdown="1">

이 MCP 서버는 표준 입력과 표준 출력을 통해 JSON-RPC 메시지를 교환합니다. 클라이언트는
`stdout`의 모든 줄을 해당 프로토콜의 일부로 해석하므로, 로그 메시지나 시작 배너가 연결을
손상시킬 수 있습니다. 진단은 `stderr`로 보내거나 억제합니다. 그래서 서버가
`show_banner=False`로 시작됩니다.

</div>
</details>

<details markdown="1">
<summary>솔루션 보기</summary>

**`server.py`에서** — 서버를 만들고 제공된 두 함수를 도구로 노출합니다.

```python
# Add references
from fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP(name="Inventory")

@mcp.tool()
def get_inventory_levels() -> dict:
    ...  # returns the sample inventory dict already in the file

@mcp.tool()
def get_weekly_consumption() -> dict:
    ...  # returns the sample consumption dict already in the file

# Run the MCP server
mcp.run(show_banner=False)
```

**`client.py`에서** — 서버에 연결하고, 도구를 검색하고, 하나의 에이전트에 생산 능력 플래너
도구와 **함께** 등록한 다음, `respond()`에서 각 호출을 라우팅합니다. 채팅 UI가 비동기
이벤트 루프에서 실행되므로 연결 코드는 첫 번째 메시지에서 한 번 실행되는 비동기 `setup()`에
있습니다.

1. 파일 맨 위에 MCP 참조를 추가합니다.

    ```python
    from mcp import ClientSession, StdioServerParameters
    from mcp.client.stdio import stdio_client
    ```

    `capacity_planner_tools` 목록과 `local_functions` 디스패치 딕셔너리(작업 4 도구)는
    파일 위쪽에 이미 제공되어 있으므로 다시 작성할 필요가 없습니다.

2. `setup()` 내부에서 stdio를 통해 서버를 시작하고 세션을 연 다음, 사용 가능한 도구를 나열하고 각각을 호출 가능 항목으로 래핑합니다.

    ```python
    stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
    stdio, write = stdio_transport
    session = await exit_stack.enter_async_context(ClientSession(stdio, write))
    await session.initialize()
    tools = (await session.list_tools()).tools

    def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            return await session.call_tool(tool_name, kwargs)
        tool_func.__name__ = tool_name
        return tool_func

    functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}

    mcp_function_tools = [
        FunctionTool(
            name=tool.name,
            description=tool.description,
            parameters={"type": "object", "properties": {}, "additionalProperties": False},
            strict=True,
        )
        for tool in tools
    ]
    ```

3. 생산 능력 플래너와 자재 도구라는 **두** 도구 세트가 모두 포함된 에이전트를 만듭니다.

    ```python
    agent = project_client.agents.create_version(
        agent_name="caldova-assistant",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="""
            You are the Caldova supply chain assistant. You help planners find open
            production capacity and estimate contract manufacturing costs, and you help
            the materials team check live stock and consumption.

            Capacity planning and transfers:
            - Use the slot and transfer tools to find open capacity, estimate cost, and draft capacity requests.

            Material inventory:
            - Recommend reorder if material inventory < 10 and weekly consumption > 15
            - Flag for review if material inventory > 20 and weekly consumption < 5
            """,
            tools=[*capacity_planner_tools, *mcp_function_tools],
        ),
    )
    ```

4. `respond()`에서 각 `function_call`을 올바른 실행기로 라우팅합니다. 로컬 함수는 직접 실행되고
    문자열을 반환합니다. MCP 도구는 세션을 통해 awaited됩니다.

    ```python
    for item in response.output:
        if item.type == "function_call":
            kwargs = json.loads(item.arguments)

            if item.name in local_functions:
                output_text = local_functions[item.name](**kwargs)          # Task 4 function
            else:
                result = await functions_dict[item.name](**kwargs)          # your MCP tool
                output_text = result.content[0].text

            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=output_text,
                )
            )

    # ...send outputs back, then:
    return AgentReply(text=response.output_text)
    ```

`python client.py`를 실행합니다. 브라우저에서 채팅 창이 열립니다. 첫 번째 메시지에서 서버가
stdio를 통해 자동으로 시작됩니다. 이제 하나의 대화에서 도우미의 **두** 부분을 모두 사용하는
프롬프트를 시도합니다.

```
Plan capacity: find the next open slot at Brightwater and price 5 weeks of premium contract capacity at expedited priority.
```
다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
```prompt
생산 능력을 계획해 주세요. Brightwater에서 다음으로 비어 있는 슬롯을 찾고, 긴급 우선순위로 프리미엄 계약 생산 능력 5주분의 가격을 계산해 주세요.
```
```
Now check materials — are there any we should reorder?
```
다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
```prompt
이제 자재를 확인해 주세요. 재주문해야 할 항목이 있나요?
```

첫 번째 프롬프트는 작업 4의 생산 능력 플래너 함수를 호출하고, 두 번째 프롬프트는 MCP 재고
도구를 호출합니다. 모두 **같은** 에이전트와 **같은** 채팅에서 실행됩니다. 브라우저 탭을
닫고 터미널에서 **Ctrl+C**를 눌러 앱을 중지합니다.

</details>

**Stretch**: 세 번째 MCP 도구(예: `get_reorder_threshold`)를 추가하고, 다른 클라이언트 변경 없이
에이전트가 해당 도구를 검색하는지 확인합니다. 라우팅은 이미 로컬 함수로 인식하지 않는 모든
도구를 처리합니다.

<details markdown="1">
<summary>비교: Microsoft Agent Framework를 사용하는 동일한 종합 과제</summary>

`client.py`에서는 MCP 클라이언트(`ClientSession`, `stdio_client`)를 직접 연결하고, 검색된 각
도구를 래핑하고, `FunctionTool` 스키마를 빌드한 다음, 모든 `function_call`을 직접
*라우팅*했습니다. 로컬 함수인지 MCP 도구인지 구분했습니다. **Microsoft Agent Framework**는
이 모든 작업을 접어 줍니다. **client_maf.py**(완성본 제공)를 열고 `python client_maf.py`로
실행합니다. 같은 종합 과제와 같은 두 도구 세트를 하나의 에이전트에서 사용하는 동작이
나옵니다.

`server.py`는 변경되지 않습니다. 여전히 MCP 서버를 작성합니다. 사라지는 것은 클라이언트
연결과 라우팅 루프입니다. `MCPStdioTool`은 서버를 시작하고 해당 도구를 노출하며, 여러분은
에이전트의 자체 `agent.run()`에 `@tool` 함수와 함께 이를 전달합니다.

```python
from agent_framework import tool, Agent, MCPStdioTool

agent = Agent(
    client=FoundryChatClient(...),
    name="caldova-assistant",
    instructions="You are the Caldova supply chain assistant...",
    tools=[next_available_slot, calculate_transfer_cost, generate_capacity_report],
)

async with MCPStdioTool(name="Inventory", command="python", args=["server.py"]) as mcp_tool:
    # One call handles either tool set — no manual "local vs MCP" routing
    result = await agent.run(user_message, tools=mcp_tool, session=session)
```

`if item.name in local_functions ... else ...` 분기가 없다는 점에 주목합니다. `agent.run()`은
모델이 선택한 도구를 호출합니다. 그 도구가 여러분의 Python 함수 중 하나든 MCP 서버에서
호스팅되는 도구든 상관없습니다. 먼저 라우팅을 직접 빌드해 보았으므로 프레임워크가 정확히
어떤 단계를 대신 처리하는지 확인할 수 있습니다.

</details>

---

**다음(선택 사항):** [작업 6 — 도우미를 호스티드 에이전트로 승격](A6-promote-your-assistant-to-a-hosted-agent.md), 또는 [랩 개요](A-build-and-extend-ai-agents.md)로 **돌아가기**.
