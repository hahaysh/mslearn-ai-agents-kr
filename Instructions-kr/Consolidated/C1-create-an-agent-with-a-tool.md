---
title: '작업 1 – 도구가 있는 에이전트 빌드'
lab:
    title: '작업 1 – 도구가 있는 에이전트 빌드'
    description: 'Microsoft Agent Framework를 사용해 사용자 지정 도구를 호출하는 단일 에이전트를 빌드합니다. Python 함수에 @tool을 데코레이트하고, Agent에 연결한 다음, agent.run()이 도구 호출 루프를 진행하도록 합니다.'
    type: 'task'
    parent: 'C'
    order: 1
    section: 'core'
    difficulty: 3
    duration: 30
    access: 'open'
    level: 300
    concepts: 'Microsoft Agent Framework, 도구, 에이전트'
    status: 'draft'
---

# 작업 1 — 도구가 있는 에이전트 빌드

***Agent Framework로 다중 에이전트 솔루션 빌드** 랩의 일부입니다. 처음이라면 [시작하기](C0-getting-started.md)부터 시작하세요.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트와 시작 코드가 필요합니다. 아직 완료하지 않았다면 [시작하기](C0-getting-started.md)를 완료하여 프로젝트를 만들고, 코드를 복제하고, `PROJECT_ENDPOINT` 및 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에서 설정합니다.
> 그런 다음 VS Code에서 연 `Python` 폴더에서 준비되었는지 확인합니다.

```
python ../setup/check_env.py --task 1
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 이전 작업을 방금 마쳤다면 프로젝트, 가상 환경 및 `.env`가 이미 설정되어 있습니다. 아래 **사용자 지정 도구로 에이전트 빌드**로 바로 이동합니다.

---

유용한 모든 에이전트는 채팅을 넘어 무언가를 *수행*할 수 있습니다. **Microsoft Agent Framework (MAF)**에서는 일반 Python 함수를 작성하고 `@tool`로 표시한 다음 에이전트에 전달하여 기능을 제공합니다. 프레임워크가 도구의 스키마를 생성하고 전체 도구 호출 루프를 실행합니다. 이 작업에서는 Caldova **사이트 방문 경비 에이전트**를 빌드합니다. 이 에이전트는 엔지니어의 사이트 방문 경비 데이터를 읽고 항목별로 정리한 뒤, 도구를 호출하여 재무 데스크로 경비 상환 청구를 "이메일"로 보냅니다.

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
<summary>도구란?</summary>
<div class="concept-body" markdown="1">

**도구**는 에이전트가 모델 자체 지식을 넘어 작업을 수행하거나 정보를 가져올 수 있도록 제공하는 함수입니다. Agent Framework에서는 일반 Python 함수를 작성하고 `@tool` 데코레이터를 추가합니다. 프레임워크는 함수 시그니처(매개 변수 설명 포함)를 읽어 모델에 필요한 스키마를 빌드합니다. 모델이 도구가 필요하다고 판단하면 `agent.run()`이 함수를 호출하고, 결과를 모델에 다시 전달한 다음 계속 진행합니다. 이 모든 과정이 자동으로 수행됩니다.

[자세히 알아보기 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

`Python` 폴더를 열고 [시작하기](C0-getting-started.md)에서 만든 가상 환경(`.\labenv\Scripts\Activate.ps1`)을 활성화한 다음 아래를 계속 진행합니다.

### 사용자 지정 도구로 에이전트 빌드

**expense_agent.py**를 열고 주석 처리된 각 자리 표시자에 코드를 추가합니다.

1. 파일에 이미 있는 코드를 검토합니다. 여기에는 다음이 포함되어 있습니다.
    - 몇 가지 **import** 문.
    - `main` 함수. 이 함수는 `data.txt`(사이트 방문 경비 데이터)를 로드하고, 이를 어떻게 처리할지 묻고, 그런 다음 호출합니다...
    - 에이전트를 만들고 실행할 `process_expenses_data` 함수.

    > **Tip**: 코드를 추가할 때 들여쓰기를 주석과 맞춥니다.

1. 파일 맨 위에서 **Add references** 주석을 찾아 필요한 네임스페이스를 추가합니다.

    ```python
    # Add references
    from agent_framework import tool, Agent
    from agent_framework.foundry import FoundryChatClient
    from azure.identity import AzureCliCredential
    from pydantic import Field
    ```

1. 파일 아래쪽에서 **Create a tool function for the email functionality** 주석을 찾아 에이전트가 청구를 보내는 데 사용할 도구를 추가합니다.

    ```python
    # Create a tool function for the email functionality
    @tool(approval_mode="never_require")
    def submit_claim(
        to: Annotated[str, Field(description="Who to send the email to")],
        subject: Annotated[str, Field(description="The subject of the email.")],
        body: Annotated[str, Field(description="The text body of the email.")],
    ):
        """Submit a Caldova site-visit expense claim by sending an email."""
        print("\nTo:", to)
        print("Subject:", subject)
        print(body, "\n")
    ```

    > **Note**: 이 함수는 이메일을 실제로 보내는 대신 콘솔에 출력하여 이메일 전송을 *시뮬레이션*합니다. 실제 애플리케이션에서는 SMTP 서비스나 유사한 서비스를 사용해 이메일을 실제로 보냅니다. `approval_mode="never_require"`를 사용하면 에이전트가 매번 승인을 요청하기 위해 멈추지 않고 도구를 호출할 수 있습니다.

1. `process_expenses_data` 함수로 다시 올라가 **Create a foundry chat client** 주석을 찾고 다음을 추가합니다(들여쓰기 수준 유지).

    ```python
    # Create a foundry chat client
    client = FoundryChatClient(
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),
        credential=AzureCliCredential(),
    )
    ```

    **AzureCliCredential** 개체를 사용하면 코드가 `az login` 세션을 사용해 Azure에 인증할 수 있습니다. **FoundryChatClient**는 `.env`의 엔드포인트와 모델 배포 이름을 사용해 Foundry 프로젝트에 연결합니다.

1. **Initialize an agent with the tool and instructions** 주석을 찾아 다음을 추가합니다.

    ```python
    # Initialize an agent with the tool and instructions
    agent = Agent(
        client=client,
        name="SiteVisitExpenseAgent",
        instructions="""You are an AI assistant for Caldova site-visit expense claims.
                    At the user's request, create an expense claim and use the submit_claim tool to send an email to expenses@caldova.example with the subject 'Site Visit Expense Claim' and a body that contains the itemized expenses with a total.
                    Then confirm to the user that you've done so. Don't ask for any more information from the user, just use the data provided to create the email.""",
        tools=[submit_claim],
    )
    ```

    **Agent** 개체는 클라이언트, 동작 방식을 알려 주는 지침, 호출이 허용된 `submit_claim` 도구로 초기화됩니다.

1. 에이전트 뒤에 이어지는 코드(이미 제공됨)를 검토합니다. 이 코드는 대화를 보관할 **session**을 만들고 `await agent.run(...)`을 호출합니다. 이 호출은 전체 도구 호출 루프를 실행하고 최종 응답을 `response.text`로 반환합니다.

    ```python
    # Create a session and use the agent to process the expenses data
    try:
        # A session keeps the conversation history across the agent run
        session = agent.create_session()
        # Invoke the agent with the prompt and the site-visit expenses data
        response = await agent.run(f"{prompt}: {expenses_data}", session=session)
        # Display the response
        print(f"\n# Agent:\n{response.text}")
    except Exception as e:
        # Something went wrong
        print(e)
    ```

1. 파일을 저장합니다(**Ctrl+S**).

### 실행 및 테스트

1. 터미널에서 로그인하고 앱을 실행합니다.

    ```
    az login
    ```

    ```
    python expense_agent.py
    ```

    `az login`을 사용하면 `AzureCliCredential`이 Azure 계정에 인증할 수 있습니다.

1. 경비 데이터로 수행할 작업을 묻는 메시지가 표시되면 다음을 입력합니다.

    ```
    Submit an expense claim
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    경비 청구를 제출합니다
    ```

1. 출력을 검토합니다. 에이전트는 `submit_claim` 도구가 출력하는 항목별 경비 청구 이메일을 작성한 다음 완료되었음을 확인해야 합니다. 다음과 비슷한 출력이 표시됩니다.

    ```
    To: expenses@caldova.example
    Subject: Site Visit Expense Claim
    ...itemized expenses with a total...

    # Agent:
    I've submitted your site-visit expense claim to expenses@caldova.example.
    ```

    > **Tip**: 속도 제한을 초과하여 앱이 실패하면 몇 초 기다린 후 다시 시도합니다. 구독에서 사용 가능한 할당량이 충분하지 않으면 모델이 응답하지 못할 수 있습니다.

> ✅ **Checkpoint**: Microsoft Agent Framework를 사용해 사용자 지정 도구가 있는 단일 에이전트를 빌드했습니다. 모델이 도구 호출 시점을 결정했고 `agent.run()`이 루프를 처리했습니다.
> 이것이 이 랩의 Core입니다. 아래 선택 작업에서는 이를 다중 에이전트 솔루션으로 확장합니다.

완료했으면 `deactivate`를 입력하여 가상 환경을 종료합니다.

---

**다음(선택 사항):** [작업 2 — 여러 에이전트를 순서대로 오케스트레이션](C2-orchestrate-multiple-agents.md) · [작업 3 — A2A로 원격 에이전트 연결](C3-connect-remote-agents-with-a2a.md)