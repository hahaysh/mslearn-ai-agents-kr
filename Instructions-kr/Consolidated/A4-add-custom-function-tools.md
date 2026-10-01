---
title: '작업 4 – 사용자 지정 함수 도구 추가'
lab:
    title: '작업 4 – 사용자 지정 함수 도구 추가'
    description: '직접 만든 Python 함수로 지원되는 도구를 에이전트에 제공하고 함수 호출 루프를 처리합니다.'
    type: 'task'
    parent: 'A'
    order: 4
    section: 'optional'
    difficulty: 3
    duration: 25
    access: 'open'
    level: 300
    concepts: '함수 도구, 함수 호출, Microsoft Agent Framework'
    status: 'draft'
---

# 작업 4 — 사용자 지정 함수 도구 추가

***AI 에이전트 빌드 및 확장** 랩의 일부입니다. 처음 오셨나요? [시작하기](A0-getting-started.md)부터 진행합니다.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트와 스타터 코드가 필요합니다. 아직
> 완료하지 않았다면 [시작하기](A0-getting-started.md)를 완료하여 프로젝트를 만들고,
> 코드를 복제하고, `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에 설정합니다.
> 도우미 파일 `functions.py`는 이미 스타터 폴더에 있습니다. 그런 다음 VS Code에서 연
> `Python` 폴더에서 준비 상태를 확인합니다.

```
python ../setup/check_env.py --task 4
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 이전 작업을 막 완료했다면
> 프로젝트, 가상 환경, `.env`가 이미 설정되어 있습니다. 아래 **설정**의 **functions.py**
> 검토로 바로 이동합니다.

---

**목표**: **직접 만든 Python 함수**로 지원되는 도구를 에이전트에 제공하고, 에이전트가 만드는
함수 호출을 처리합니다.

각 함수를 도구로 설명한 다음, 에이전트가 선택한 함수를 실행하는 루프를 작성합니다.

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
<summary>에이전트는 Python 함수를 어떻게 호출하나요?</summary>
<div class="concept-body" markdown="1">

에이전트는 각 함수의 목적과 매개 변수를 설명하는 JSON 스키마를 받습니다. 그러면 최종
답변 대신 함수 이름과 인수를 반환할 수 있습니다. 애플리케이션은 일치하는 Python 함수를
실행하고, 그 출력을 다시 보내며, 에이전트가 결과를 응답에 사용할 수 있게 합니다. 모델이
호출을 선택하지만, 실제로 무엇이 실행되는지는 코드가 제어합니다.

</div>
</details>

**설정:**

1. `Labfiles/A-build-and-extend-ai-agents/Python` 폴더에서 가상 환경
    (`.\labenv\Scripts\Activate.ps1`)을 활성화하고, **.env**에 `PROJECT_ENDPOINT`와
    `MODEL_DEPLOYMENT_NAME`이 설정되어 있는지 확인합니다([시작하기](A0-getting-started.md) 참조).
    그런 다음 생산 능력 플래너의 도우미 함수가 들어 있는 **functions.py**를 검토합니다.

    Caldova의 세 공장은 Ashford, Brightwater, Calderwood입니다. 코드는 공장 매개 변수 이름으로
    `site`를 사용합니다.

> **Try it first**: **functions.py**에서 `next_available_slot(site)`를 살펴봅니다. 모델이 언제
> 어떻게 호출해야 하는지 알 수 있도록 단일 `site` 매개 변수를 어떻게 설명하겠습니까?
> 솔루션을 보기 전에 JSON 스키마를 작성합니다.

<details markdown="1">
<summary>솔루션 보기</summary>

**functions_agent.py**의 주석을 따라 진행합니다. 참조를 추가하고 프로젝트에 연결합니다(작업 2와
같은 패턴). 이 파일은 에이전트 설정이 한 번 실행된 다음, `respond()` 함수가 각 채팅
메시지를 처리하고 응답을 `run_chat_app()`에 전달하도록 구성되어 있습니다.

1. **세 가지 함수 도구를 정의합니다.** 각 스키마는 모델에 Python 함수를 호출하는 방법을 알려 줍니다. 예를 들어 슬롯 조회 도구는 다음과 같습니다.

    ```python
    # Define the slot lookup function tool
    slot_tool = FunctionTool(
        name="next_available_slot",
        description="Get the next open production slot at a given site.",
        parameters={
            "type": "object",
            "properties": {
                "site": {
                    "type": "string",
                    "description": "site to find the next open production slot at (e.g. 'ashford', 'brightwater', 'calderwood')",
                },
            },
            "required": ["site"],
            "additionalProperties": False,
        },
        strict=True,
    )
    ```

    각 함수의 매개 변수와 일치하도록 같은 방식으로 `cost_tool`(`calculate_transfer_cost`)과
    `report_tool`(`generate_capacity_report`)을 정의합니다.

2. **세 가지 도구가 모두 포함된 에이전트를 만듭니다.**

    ```python
    agent = project_client.agents.create_version(
        agent_name="capacity-planner-agent",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="""You are a capacity planning assistant for Caldova that helps
                planners find open production slots and estimate contract manufacturing costs.
                Use the available tools to assist users with their inquiries.""",
            tools=[slot_tool, cost_tool, report_tool],
        ),
    )
    ```

3. `respond()` 내부의 **도구 호출 루프를 채웁니다**. response에서 각 `function_call`을 읽고,
    일치하는 Python 함수를 실행한 다음 `FunctionCallOutput`을 수집합니다.

    ```python
    # Process function calls
    for item in response.output:
        if item.type == "function_call":
            result = None
            if item.name == "next_available_slot":
                result = next_available_slot(**json.loads(item.arguments))
            elif item.name == "calculate_transfer_cost":
                result = calculate_transfer_cost(**json.loads(item.arguments))
            elif item.name == "generate_capacity_report":
                result = generate_capacity_report(**json.loads(item.arguments))
            input_list.append(
                FunctionCallOutput(
                    type="function_call_output",
                    call_id=item.call_id,
                    output=result,
                )
            )
    ```

    `respond()`의 나머지 부분(이미 제공됨)은 출력을 다시 보내고 최종 답변을 채팅 창에
    반환합니다. 도구 호출이 conversation 상태에서 해결되도록 같은 **conversation**에 출력을
    첨부한다는 점에 유의합니다. 대신 `previous_response_id`로 다시 보내면 *다음* 메시지가
    *"No tool output found for function call"* 오류로 실패합니다.

    ```python
    # Send function call outputs back to the model and retrieve a response
    if input_list:
        response = openai_client.responses.create(
            conversation=conversation.id,
            input=input_list,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        )

    return AgentReply(text=response.output_text)
    ```

`python functions_agent.py`를 실행합니다. 브라우저에서 채팅 창이 열립니다. **두 개**의 도구가
동시에 필요한 프롬프트를 시도합니다.

```
Find me the next open slot at Brightwater and give me the cost for 5 weeks of premium contract capacity at expedited priority.
```
다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
```prompt
Brightwater에서 다음으로 비어 있는 슬롯을 찾아 주고, 긴급 우선순위로 프리미엄 계약 생산 능력 5주분의 비용을 알려 주세요.
```

에이전트는 한 턴에서 두 함수를 모두 호출하고 결과를 결합합니다. 예를 들면 다음과 같습니다.

```
The next open slot at Brightwater is the Line 3 Changeover on March 3rd.
The cost for 5 weeks of premium contract capacity at expedited priority is $1,875K.
```

브라우저 탭을 닫고 터미널에서 **Ctrl+C**를 눌러 앱을 중지합니다(에이전트는 종료 시 자동으로 삭제됩니다).

</details>

**Stretch**: 네 번째 함수 도구를 직접 추가하고 지침을 업데이트하여 해당 도구를 언급합니다.

<details markdown="1">
<summary>비교: Microsoft Agent Framework를 사용하는 동일한 에이전트</summary>

방금 도구마다 두 개의 스키마를 작성하고, 각 `function_call`을 Python 함수에 매칭하는
디스패치 루프를 작성했습니다. **Microsoft Agent Framework**는 두 가지를 모두 제거합니다.
**functions_agent_maf.py**(완성본 제공)를 열고 `python functions_agent_maf.py`로 실행합니다.
그러면 *동일한* 생산 능력 플래너 도우미가 생성됩니다.

차이는 도구 정의와 루프입니다. 직접 작성한 `FunctionTool` 스키마 대신, 함수에 `@tool`을
데코레이트하고 각 매개 변수를 인라인으로 설명합니다.

```python
from agent_framework import tool, Agent
from agent_framework.foundry import FoundryChatClient
from azure.identity import AzureCliCredential
from pydantic import Field
from typing import Annotated

@tool(approval_mode="never_require")
def next_available_slot(
    site: Annotated[str, Field(description="Site to find the next open production slot at (e.g. 'ashford', 'brightwater', 'calderwood')")],
) -> str:
    """Get the next open production slot at a given site."""
    return functions.next_available_slot(site)
```

그런 다음 데코레이트된 함수로 에이전트를 만들고 `agent.run()`이 전체 도구 호출 루프를
처리하게 합니다. `response.output`을 읽지 않고, 이름을 매칭하지 않으며, 출력을 다시 보내지
않습니다.

```python
agent = Agent(
    client=FoundryChatClient(
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),
        credential=AzureCliCredential(),
    ),
    name="capacity-planner-agent",
    instructions="You are a capacity planning assistant for Caldova...",
    tools=[next_available_slot, calculate_transfer_cost, generate_capacity_report],
)

# agent.run() decides which tools to call, runs them, and returns the final answer
result = await agent.run(user_message, session=session)
```

프레임워크가 위에서 직접 작성한 배관 작업을 처리하므로 같은 결과를 훨씬 적은 코드로
얻을 수 있습니다. 먼저 직접 작성해 보았기 때문에 `agent.run()`이 무엇을 대신 해 주는지
명확하게 이해할 수 있습니다.

</details>

---

**다음:** [작업 5 — 종합 과제: 나만의 MCP 서버 빌드](A5-capstone-build-your-own-mcp-server.md)
