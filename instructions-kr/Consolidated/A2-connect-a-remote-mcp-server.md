---
title: '작업 2 – 원격 MCP 서버 연결'
lab:
    title: '작업 2 – 원격 MCP 서버 연결'
    description: '원격 Model Context Protocol (MCP) 서버에 연결하여 도구로 에이전트를 확장하고 코드에서 도구 승인 요청을 처리합니다.'
    type: 'task'
    parent: 'A'
    order: 2
    section: 'core'
    difficulty: 3
    duration: 20
    access: 'open'
    level: 300
    concepts: '도구, Model Context Protocol (MCP), 승인'
    status: 'draft'
---

# 작업 2 — 원격 MCP 서버 연결

***AI 에이전트 빌드 및 확장** 랩의 일부입니다. 처음 오셨나요? [시작하기](A0-getting-started.md)부터 진행합니다.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트와 스타터 코드가 필요합니다. 아직
> 완료하지 않았다면 [시작하기](A0-getting-started.md)를 완료하여 프로젝트를 만들고,
> 코드를 복제하고, `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에 설정합니다.
> 그런 다음 VS Code에서 연 `Python` 폴더에서 준비 상태를 확인합니다.

```
python ../setup/check_env.py --task 2
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 이전 작업을 막 완료했다면
> 프로젝트, 가상 환경, `.env`가 이미 설정되어 있습니다. 아래 **에이전트를 MCP 서버에 연결**로
> 바로 이동합니다.

---

**Model Context Protocol (MCP)**을 사용하면 에이전트가 서버에서 호스팅되는 도구를 검색하고
호출할 수 있습니다. Caldova의 플랫폼 팀은 Azure에서 공급망 시스템을 다시 빌드하고
있습니다. 이 별도의 지원 연습에서는 플랫폼 설명서 에이전트를 만들고 **Microsoft Learn
Docs** 원격 MCP 서버에 연결합니다. 그러면 에이전트가 필요할 때 신뢰할 수 있는 최신 Azure
문서를 검색할 수 있습니다.

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
<summary>MCP란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

**Model Context Protocol (MCP)**은 에이전트가 런타임에 도구를 검색할 수 있게 하여 이
문제를 해결합니다. MCP에서 도구는 실시간 카탈로그 역할을 하는 **server**에 있습니다.
에이전트는 **client**를 통해 서버에 사용 가능한 도구를 물어보고 필요할 때 호출합니다.

로컬 함수는 애플리케이션 내부에서 실행됩니다. MCP 도구는 다른 서버에서 실행될 수 있으며
여러 클라이언트가 공유할 수 있습니다. 이 작업에서 문서 도구는 Microsoft Learn에서 원격으로
호스팅됩니다.

[자세히 알아보기 →](https://review.learn.microsoft.com/en-us/training/modules/build-extend-ai-agents/5-connect-agents-to-mcp?branch=pr-en-us-55509)

</div>
</details>

[시작하기](A0-getting-started.md)에서 만든 `Python` 폴더를 열고 가상 환경(`.\labenv\Scripts\Activate.ps1`)을 활성화한 다음 아래를 계속 진행합니다.

### 에이전트를 MCP 서버에 연결

**remote_mcp_agent.py**를 열고 각 주석 처리된 자리 표시자에 코드를 추가합니다.

> **Tip**: 코드를 추가할 때 들여쓰기를 주석과 맞춥니다.

1. **참조 추가**:

    ```python
    # Add references
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.projects.models import PromptAgentDefinition, MCPTool
    from openai.types.responses.response_input_param import McpApprovalResponse, ResponseInputParam
    ```

1. **에이전트 클라이언트에 연결**:

    ```python
    # Connect to the agents client
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
        project_client.get_openai_client() as openai_client,
    ):
    ```

1. **에이전트 MCP 도구 초기화** — 에이전트가 Microsoft Learn Docs MCP 서버를 가리키도록 합니다.

    ```python
    # Initialize agent MCP tool
    mcp_tool = MCPTool(
        server_label="api-specs",
        server_url="https://learn.microsoft.com/api/mcp",
        require_approval="always",
    )
    ```

1. **MCP 도구가 있는 새 에이전트 만들기**:

    ```python
    # Create a new agent with the MCP tool
    agent = project_client.agents.create_version(
        agent_name="platform-docs-agent",
        definition=PromptAgentDefinition(
            model=model_deployment,
            instructions="You are a platform engineering assistant for Caldova. Use the available MCP tools to look up trusted Azure documentation and help the team build and operate the supply chain platform.",
            tools=[mcp_tool],
        ),
    )
    print(f"Agent created (id: {agent.id}, name: {agent.name}, version: {agent.version})")
    ```

1. **대화 스레드 만들기**:

    ```python
    # Create a conversation thread
    conversation = openai_client.conversations.create()
    print(f"Created conversation (id: {conversation.id})")
    ```

1. **MCP 도구를 트리거할 초기 요청 보내기**:

    ```python
    # Send initial request that will trigger the MCP tool
    response = openai_client.responses.create(
        conversation=conversation.id,
        input="Give me the Azure CLI commands to deploy our product catalog API to an Azure Container App with a managed identity.",
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
    )
    ```

1. **MCP 승인 요청 처리** — 도구에 승인이 필요하므로 에이전트는 각 호출 전에 일시 중지하고
    권한을 요청합니다. 이 루프는 각 요청을 자동 승인합니다.

    <details markdown="1" class="concept">
    <summary>MCP 도구에는 왜 승인이 필요한가요?</summary>
    <div class="concept-body" markdown="1">

    MCP 호출은 다른 서비스로 정보를 보내거나 외부 작업을 트리거할 수 있습니다.
    승인은 신뢰 경계를 만듭니다. 애플리케이션은 호출을 허용하기 전에 요청된 서버와
    도구를 검사할 수 있습니다. 이 샘플은 사용자 인터페이스 없이 승인 흐름을 연습할 수
    있도록 예상되는 `api-specs` 서버의 요청만 자동 승인합니다.

    </div>
    </details>

    ```python
    # Process any MCP approval requests that were generated
    while True:
        input_list: ResponseInputParam = []
        for item in response.output:
            if item.type == "mcp_approval_request":
                if item.server_label == "api-specs" and item.id:
                    input_list.append(
                        McpApprovalResponse(
                            type="mcp_approval_response",
                            approve=True,
                            approval_request_id=item.id,
                        )
                    )

        # No more approvals needed -> the agent has produced its final response
        if not input_list:
            break

        response = openai_client.responses.create(
            input=input_list,
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        )

    print(f"\nAgent response: {response.output_text}")
    ```

1. 테스트 에이전트를 남기지 않도록 **에이전트 버전 정리**:

    ```python
    # Clean up resources by deleting the agent version
    project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
    print("Agent deleted")
    ```

1. 파일을 저장합니다(**Ctrl+S**).

### 실행 및 테스트

1. 터미널에서 로그인하고 앱을 실행합니다.

    ```
    az login
    ```

    ```
    python remote_mcp_agent.py
    ```

1. 에이전트가 자신을 만들고, MCP 도구를 호출하고(루프에서 자동 승인됨), 실시간 문서를 사용하여 답변하는지 확인합니다. 다음과 유사한 출력이 표시됩니다.

    ```
    Agent created (id: platform-docs-agent:2, name: platform-docs-agent, version: 2)
    Created conversation (id: conv_...)

    Agent response: Here are Azure CLI commands to create an Azure Container App with a managed identity:
    ...
    Agent deleted
    ```

1. `input` 문자열을 다른 Azure 서비스에 대한 질문으로 변경하고 다시 실행해 봅니다.

> ✅ **Checkpoint**: 그라운딩된 에이전트를 빌드했고, 승인 처리를 포함하여 원격 MCP 서버를
> 통한 외부 도구로 에이전트를 확장했습니다. 이것이 이 랩의 핵심(Core)입니다. 아래의 모든
> 항목은 선택 사항입니다.

완료했으면 `deactivate`를 입력하여 가상 환경을 종료합니다.

---

**다음(선택 사항):** [작업 3 — 클라이언트 앱에서 에이전트 호출](A3-call-your-agent-from-a-client-app.md) · [작업 4 — 사용자 지정 함수 도구 추가](A4-add-custom-function-tools.md)
