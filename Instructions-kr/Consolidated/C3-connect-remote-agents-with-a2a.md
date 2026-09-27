---
title: '작업 3 – A2A로 원격 에이전트 연결'
lab:
    title: '작업 3 – A2A로 원격 에이전트 연결'
    description: 'Agent-to-Agent (A2A) 프로토콜을 사용해 별도 프로세스에서 실행되는 에이전트를 연결합니다. 라우팅 에이전트가 Caldova 기술 이전 계획을 위해 협업하는 transfer-title 에이전트와 transfer-outline 에이전트를 검색하고 위임합니다.'
    type: 'task'
    parent: 'C'
    order: 3
    section: 'optional'
    difficulty: 4
    duration: 30
    access: 'open'
    level: 400
    concepts: 'A2A 프로토콜, 원격 에이전트, 다중 에이전트 오케스트레이션'
    status: 'draft'
---

# 작업 3 — A2A로 원격 에이전트 연결

***Agent Framework로 다중 에이전트 솔루션 빌드** 랩의 일부입니다. 처음이라면 [시작하기](C0-getting-started.md)부터 시작하세요.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트와 시작 코드가 필요합니다. 아직 완료하지 않았다면 [시작하기](C0-getting-started.md)를 완료하여 프로젝트를 만들고, 코드를 복제하고, `PROJECT_ENDPOINT` 및 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에서 설정합니다.
> 그런 다음 VS Code에서 연 `Python` 폴더에서 준비되었는지 확인합니다.

```
python ../setup/check_env.py --task 3
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 이전 작업을 방금 마쳤다면 프로젝트, 가상 환경 및 `.env`가 이미 설정되어 있습니다. 아래 **검색 가능한 에이전트 만들기**로 바로 이동합니다.

---

지금까지 만든 에이전트는 단일 프로세스 안에 있었습니다. 실제 시스템은 여러 서비스로 분리되는 경우가 많습니다. 각 에이전트는 자체적으로 실행되고 네트워크를 통해 협업합니다. **Agent-to-Agent (A2A) 프로토콜**은 에이전트가 자신이 수행할 수 있는 작업을 알리고 서로에게 작업을 보낼 수 있게 하는 표준 방식입니다. 이 작업에서는 세 원격 에이전트로 Caldova 이전 계획 시스템을 빌드합니다. **transfer-title 에이전트**는 제목을 제안하고, **transfer-outline 에이전트**는 계획 초안을 작성하며, **routing 에이전트**는 두 에이전트를 검색하고 각 요청을 적절한 에이전트에 위임합니다.

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
<summary>A2A 프로토콜이란?</summary>
<div class="concept-body" markdown="1">

**Agent-to-Agent (A2A) 프로토콜**을 사용하면 별도 프로세스의 에이전트가 서로를 검색하고 호출할 수 있습니다. 각 에이전트는 이름, 기술 및 엔드포인트를 설명하는 작은 문서인 **agent card**를 게시하므로 다른 에이전트가 런타임에 이를 찾을 수 있습니다. 한 에이전트(여기서는 routing 에이전트)가 이러한 카드를 읽고 요청을 누가 처리해야 하는지 결정한 뒤 HTTP로 메시지를 보냅니다. 원격 에이전트는 작업을 수행하고 응답을 반환합니다.

[자세히 알아보기 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

`Python` 폴더를 열고 [시작하기](C0-getting-started.md)에서 만든 가상 환경(`.\labenv\Scripts\Activate.ps1`)을 활성화한 다음 아래를 계속 진행합니다.

이 작업의 시작 코드는 에이전트별 폴더 하나씩과 클라이언트 및 실행기로 구성되어 있습니다.

```output
Python
├── outline_agent/       # remote agent: drafts a transfer plan outline (provided complete)
│   ├── agent.py
│   ├── agent_executor.py
│   └── server.py
├── routing_agent/       # orchestrator that discovers and delegates to the other agents
│   ├── agent.py
│   └── server.py
├── title_agent/         # remote agent: suggests a transfer brief title
│   ├── agent.py
│   ├── agent_executor.py
│   └── server.py
├── client.py            # sends your prompt to the routing agent
└── run_all.py           # launches all three agent servers
```

각 에이전트 폴더에는 Foundry 에이전트 코드와 이를 호스트할 서버가 포함되어 있습니다. **routing 에이전트**는 **transfer-title** 및 **transfer-outline** 에이전트를 검색하고 통신합니다. **client**를 사용하면 프롬프트를 routing 에이전트에 제출할 수 있습니다. `run_all.py`는 모든 서버를 시작합니다.

> `outline_agent`(transfer-outline 에이전트)는 참고용으로 **완성된 상태**로 제공됩니다. `title_agent`에서 이에 해당하는 코드를 빌드한 다음 `routing_agent`를 연결합니다.

### 검색 가능한 에이전트 만들기

이 작업에서는 Caldova 기술 이전을 위한 제목을 제안하는 transfer-title 에이전트를 완성합니다. 또한 A2A 프로토콜이 에이전트를 검색 가능하게 만드는 데 사용하는 에이전트의 기술과 카드를 정의합니다.

> **Tip**: 코드를 추가할 때 들여쓰기를 주석과 맞춥니다.

1. **title_agent/agent.py**를 엽니다.

1. **Create the agents client** 주석을 찾아 Foundry 프로젝트에 연결하는 코드를 추가합니다.

    ```python
    # Create the agents client
    self.client = AgentsClient(
        endpoint=os.environ['PROJECT_ENDPOINT'],
        credential=DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True
        )
    )
    ```

1. **Create the title agent** 주석을 찾아 에이전트를 만드는 코드를 추가합니다.

    ```python
    # Create the title agent
    self.agent = self.client.create_agent(
        model=os.environ['MODEL_DEPLOYMENT_NAME'],
        name='transfer-title-agent',
        instructions="""
        You are a helpful planning assistant for Caldova.
        Given a site or capability the planner names, suggest a single clear transfer brief title.
        """,
    )
    ```

1. **Create a thread for the chat session** 주석을 찾아 다음을 추가합니다.

    ```python
    # Create a thread for the chat session
    thread = self.client.threads.create()
    ```

1. **Send user message** 주석을 찾아 다음을 추가합니다.

    ```python
    # Send user message
    self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. **Create and run the agent** 주석을 찾아 다음을 추가합니다.

    ```python
    # Create and run the agent
    run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    파일의 나머지 부분은 에이전트의 응답을 처리하고 반환합니다.

1. 파일을 저장합니다(**Ctrl+S**). 이제 A2A 프로토콜과 에이전트의 기술 및 카드를 공유합니다.

1. **title_agent/server.py**를 엽니다.

1. **Define agent skills** 주석을 찾아 다음을 추가합니다.

    ```python
    # Define agent skills
    skills = [
        AgentSkill(
            id='generate_trip_title',
            name='Generate Transfer Title',
            description='Generates a transfer brief title based on a site or capability',
            tags=['title'],
            examples=[
                'Can you give me a title for a packaging transfer at Ashford?',
            ],
        ),
    ]
    ```

1. **Create agent card** 주석을 찾아 에이전트를 검색 가능하게 만드는 메타데이터를 추가합니다.

    ```python
    # Create agent card
    agent_card = AgentCard(
        name='Caldova Transfer Title Agent',
        description='An intelligent title generator agent powered by Foundry. '
        'I can help you generate clear titles for Caldova tech transfers.',
        url=f'http://{host}:{port}/',
        version='1.0.0',
        default_input_modes=['text'],
        default_output_modes=['text'],
        capabilities=AgentCapabilities(),
        skills=skills,
    )
    ```

1. **Create agent executor** 주석을 찾아 다음을 추가합니다.

    ```python
    # Create agent executor
    agent_executor = create_foundry_agent_executor(agent_card)
    ```

1. **Create request handler** 주석을 찾아 다음을 추가합니다.

    ```python
    # Create request handler
    request_handler = DefaultRequestHandler(
        agent_executor=agent_executor, task_store=InMemoryTaskStore()
    )
    ```

1. **Create A2A application** 주석을 찾아 다음을 추가합니다.

    ```python
    # Create A2A application
    a2a_app = A2AStarletteApplication(
        agent_card=agent_card, http_handler=request_handler
    )
    ```

    이렇게 하면 transfer-title 에이전트의 정보를 공유하고 에이전트 실행기를 사용해 들어오는 요청을 처리하는 A2A 서버가 만들어집니다.

1. 파일을 저장합니다(**Ctrl+S**).

### 에이전트 간 메시지 사용

이 작업에서는 A2A 프로토콜을 사용해 routing 에이전트가 다른 에이전트에 메시지를 보낼 수 있게 하고, transfer-title 에이전트가 agent executor를 완성하여 해당 메시지를 받을 수 있게 합니다.

1. **routing_agent/agent.py**를 엽니다.

    routing 에이전트는 시스템을 오케스트레이션합니다. 사용자 메시지가 도착하면 스레드를 시작하고, `create_and_process`를 사용해 요청을 처리할 원격 에이전트를 결정하며, `send_message` 함수를 사용해 HTTP를 통해 해당 에이전트로 메시지를 라우팅합니다. `send_message` 메서드는 비동기이며 실행이 완료되려면 await해야 합니다.

1. **Retrieve the remote agent's A2A client using the agent name** 주석을 찾아 다음을 추가합니다.

    ```python
    # Retrieve the remote agent's A2A client using the agent name 
    client = self.remote_agent_connections[agent_name]
    ```

1. **Construct the payload to send to the remote agent** 주석을 찾아 다음을 추가합니다.

    ```python
    # Construct the payload to send to the remote agent
    payload: dict[str, Any] = {
        'message': {
            'role': 'user',
            'parts': [{'kind': 'text', 'text': task}],
            'messageId': message_id,
        },
    }
    ```

1. **Wrap the payload in a SendMessageRequest object** 주석을 찾아 다음을 추가합니다.

    ```python
    # Wrap the payload in a SendMessageRequest object
    message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. **Send the message to the remote agent client and await the response** 주석을 찾아 다음을 추가합니다.

    ```python
    # Send the message to the remote agent client and await the response
    send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```

1. 파일을 저장합니다(**Ctrl+S**). 이제 routing 에이전트가 원격 에이전트를 검색하고 메시지를 보낼 수 있습니다.
    다음으로, 들어오는 메시지를 처리할 수 있도록 transfer-title 에이전트의 executor를 완성합니다.

1. **title_agent/agent_executor.py**를 엽니다.

    `AgentExecutor` 클래스는 `execute` 및 `cancel`을 구현해야 합니다. `cancel` 메서드는 제공되어 있습니다. `execute` 메서드는 `TaskUpdater` 개체를 사용해 이벤트를 관리하고 작업이 완료되었음을 알립니다. 아래 실행 논리를 추가합니다.

1. `execute` 메서드에서 **Process the request** 주석을 찾아 다음을 추가합니다.

    ```python
    # Process the request
    await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. `_process_request` 메서드에서 **Get the title agent** 주석을 찾아 다음을 추가합니다.

    ```python
    # Get the title agent
    agent = await self._get_or_create_agent()
    ```

1. **Update the task status** 주석을 찾아 다음을 추가합니다.

    ```python
    # Update the task status
    await task_updater.update_status(
        TaskState.working,
        message=new_agent_text_message('Title Agent is processing your request...', context_id=context_id),
    )
    ```

1. **Run the agent conversation** 주석을 찾아 다음을 추가합니다.

    ```python
    # Run the agent conversation
    responses = await agent.run_conversation(user_message)
    ```

1. **Update the task with the responses** 주석을 찾아 다음을 추가합니다.

    ```python
    # Update the task with the responses
    for response in responses:
        await task_updater.update_status(
            TaskState.working,
            message=new_agent_text_message(response, context_id=context_id),
        )
    ```

1. **Mark the task as complete** 주석을 찾아 다음을 추가합니다.

    ```python
    # Mark the task as complete
    final_message = responses[-1] if responses else 'Task completed.'
    await task_updater.complete(
        message=new_agent_text_message(final_message, context_id=context_id)
    )
    ```

    이제 transfer-title 에이전트가 A2A 프로토콜이 메시지를 처리하는 데 사용하는 executor로 래핑되었습니다.

1. 파일을 저장합니다(**Ctrl+S**).

### 실행 및 테스트

1. 터미널에서 로그인하고 세 에이전트 서버를 모두 시작합니다.

    ```
    az login
    ```

    ```
    python run_all.py
    ```

    서버는 인증된 Azure 세션을 사용해 시작됩니다. 각 서버가 준비되었다고 보고할 때까지 기다립니다.

1. 가상 환경이 활성화된 **두 번째** 터미널에서 클라이언트를 실행합니다.

    ```
    python client.py
    ```

1. 메시지가 표시되면 다음과 같은 프롬프트를 입력합니다.

    ```
    Create a title and outline for a packaging transfer at Ashford.
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    Ashford의 포장 이전을 위한 제목과 개요를 만들어 주세요.
    ```

    잠시 후 routing 에이전트가 transfer-title 및 transfer-outline 에이전트에 위임하고, 응답에서 제안된 제목과 계획 개요를 볼 수 있습니다.

    > **Tip**: 포트가 이미 사용 중이어서 서버 시작에 실패하면 이전 실행을 중지한 후(`run_all.py` 터미널에서 Ctrl+C) 다시 시도하거나 `*_PORT` 값을 `.env`에서 변경합니다.

1. 완료되면 `run_all.py` 터미널에서 **Ctrl+C**를 눌러 모든 서버를 중지한 다음, 각 터미널에서 `deactivate`를 입력하여 가상 환경을 종료합니다.

> ✅ **Checkpoint**: A2A 프로토콜을 사용해 별도 프로세스에서 실행되는 에이전트를 연결했습니다. agent card를 게시하고, 요청을 올바른 원격 에이전트로 라우팅하고, 결과를 반환했습니다.

---

**다음(선택 사항):** [작업 4 — 지원 티켓 분류 및 라우팅](C4-classify-and-route-a-ticket.md)