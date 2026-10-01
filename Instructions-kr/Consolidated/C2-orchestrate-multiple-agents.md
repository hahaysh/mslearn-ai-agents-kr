---
title: '작업 2 – 여러 에이전트를 순서대로 오케스트레이션'
lab:
    title: '작업 2 – 여러 에이전트를 순서대로 오케스트레이션'
    description: 'Microsoft Agent Framework를 사용해 여러 에이전트를 순서대로 오케스트레이션합니다. 요약 에이전트, 분류 에이전트, 작업 에이전트가 사이트 피드백 하나를 단계별로 분류하며, 각 단계는 이전 단계의 결과를 기반으로 합니다.'
    type: 'task'
    parent: 'C'
    order: 2
    section: 'optional'
    difficulty: 3
    duration: 30
    access: 'open'
    level: 300
    concepts: 'Microsoft Agent Framework, 다중 에이전트 오케스트레이션, 순차 워크플로'
    status: 'draft'
---

# 작업 2 — 여러 에이전트를 순서대로 오케스트레이션

***Agent Framework로 다중 에이전트 솔루션 빌드** 랩의 일부입니다. 처음이라면 [시작하기](C0-getting-started.md)부터 시작하세요.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트와 시작 코드가 필요합니다. 아직 완료하지 않았다면 [시작하기](C0-getting-started.md)를 완료하여 프로젝트를 만들고, 코드를 복제하고, `PROJECT_ENDPOINT` 및 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에서 설정합니다.
> 그런 다음 VS Code에서 연 `Python` 폴더에서 준비되었는지 확인합니다.

```
python ../setup/check_env.py --task 2
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 이전 작업을 방금 마쳤다면 프로젝트, 가상 환경 및 `.env`가 이미 설정되어 있습니다. 아래 **에이전트 만들기**로 바로 이동합니다.

---

어떤 작업은 한 단계씩 처리하고 결과를 다음 단계로 넘기는 전문가 **팀**이 수행하는 것이 가장 좋습니다. Microsoft Agent Framework의 **순차 오케스트레이션**이 바로 이 작업을 수행합니다. 에이전트 목록을 순서대로 실행하고 각 에이전트의 출력을 수집합니다. 이 작업에서는 Caldova **피드백 분류** 파이프라인을 빌드합니다. *요약 에이전트*가 사이트 의견을 압축하고, *분류 에이전트*가 레이블을 지정하며, *작업 에이전트*가 다음 단계를 권장합니다.

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
<summary>순차 오케스트레이션이란?</summary>
<div class="concept-body" markdown="1">

**순차 오케스트레이션**은 여러 에이전트를 차례로 실행하면서 각 에이전트의 진행 중인 대화를 다음 에이전트에 전달합니다. 작업이 요약, 분류, 결정처럼 순서가 있는 단계로 명확히 나뉘고 각 단계가 이전 단계의 출력을 활용할 때 적합합니다. Agent Framework에서는 `SequentialBuilder`를 사용해 만들며, 참여 에이전트를 실행할 순서대로 나열합니다.

[자세히 알아보기 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

`Python` 폴더를 열고 [시작하기](C0-getting-started.md)에서 만든 가상 환경(`.\labenv\Scripts\Activate.ps1`)을 활성화한 다음 아래를 계속 진행합니다.

### 에이전트 만들기

**feedback_agents.py**를 열고 주석 처리된 각 자리 표시자에 코드를 추가합니다.

1. 파일에 이미 있는 코드를 검토합니다. `main` 함수에서 세 가지 에이전트 **instructions**(요약, 분류, 작업)를 잠시 읽어 봅니다. 이 지침은 각 에이전트가 수행할 작업을 정의합니다.

    > **Tip**: 코드를 추가할 때 들여쓰기를 주석과 맞춥니다.

1. 파일 맨 위에서 **Add references** 주석을 찾아 필요한 네임스페이스를 추가합니다.

    ```python
    # Add references
    from agent_framework import Message
    from agent_framework.foundry import FoundryChatClient
    from agent_framework.orchestrations import SequentialBuilder
    from azure.identity import AzureCliCredential
    ```

1. **Create the chat client** 주석을 찾아 다음을 추가합니다(들여쓰기 수준 유지).

    ```python
    # Create the chat client
    credential = AzureCliCredential()
    chat_client = FoundryChatClient(
        credential=credential,
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),
    )
    ```

    **AzureCliCredential**를 사용하면 코드가 `az login` 세션을 사용해 Azure에 인증할 수 있으며, **FoundryChatClient**는 Foundry 프로젝트에 연결합니다. 세 에이전트는 모두 이 하나의 클라이언트를 공유합니다.

1. **Create agents** 주석을 찾아 공유 클라이언트에서 세 에이전트를 만들도록 다음을 추가합니다.

    ```python
    # Create agents
    summarizer_agent = chat_client.as_agent(
        name="summarizer",
        instructions=summarizer_instructions,
    )

    classifier_agent = chat_client.as_agent(
        name="classifier",
        instructions=classifier_instructions,
    )

    action_agent = chat_client.as_agent(
        name="action",
        instructions=action_instructions,
    )
    ```

1. **Initialize the current feedback** 주석을 찾아 파이프라인이 분류할 샘플 사이트 피드백을 추가합니다.

    ```python
    # Initialize the current feedback
    feedback="""
    I use the line-scheduling app before every changeover, and it works well overall.
    But when I'm checking the schedule at night on the floor, the bright screen is really harsh on my eyes.
    If you added a dark mode option, it would make it much more comfortable to use in low light.
    """
    ```

### 순차 오케스트레이션 만들기

1. **Build sequential orchestration** 주석을 찾아 파이프라인을 정의하도록 다음을 추가합니다.

    ```python
    # Build sequential orchestration
    workflow = SequentialBuilder(
        participants=[summarizer_agent, classifier_agent, action_agent],
        output_from="all",
    ).build()
    ```

    에이전트는 나열된 순서대로 피드백을 처리합니다. `output_from="all"`은 마지막 에이전트뿐 아니라 *모든* 에이전트의 출력이 수집되도록 보장합니다.

1. **Run and collect outputs** 주석을 찾아 다음을 추가합니다.

    ```python
    # Run and collect outputs
    result = await workflow.run(f"Site feedback: {feedback}")
    outputs = result.get_outputs()
    ```

    이 코드는 오케스트레이션을 실행하고 각 참여 에이전트의 출력을 수집합니다.

1. **Display outputs** 주석을 찾아 다음을 추가합니다.

    ```python
    # Display outputs
    i = 1
    for response in outputs:
        for msg in cast(list[Message], response.messages):
            name = msg.author_name or ("assistant" if msg.role == "assistant" else "user")
            print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
            i += 1
    ```

    이 코드는 오케스트레이션에서 수집한 각 메시지에 해당 메시지를 생성한 에이전트 레이블을 붙여 형식화하고 출력합니다.

1. 파일을 저장합니다(**Ctrl+S**).

### 실행 및 테스트

1. 터미널에서 로그인하고 앱을 실행합니다.

    ```
    az login
    ```

    ```
    python feedback_agents.py
    ```

1. 출력을 검토합니다. 각 에이전트가 한 단계씩 기여하며, 다음과 비슷한 출력이 표시됩니다.

    ```
    Site team requests a dark mode option for comfortable night-shift use.
    Feature request
    Log as an enhancement request to add a dark mode for night-shift use.
    ------------------------------------------------------------
    01 [summarizer]
    Site team requests a dark mode option for comfortable night-shift use.
    ------------------------------------------------------------
    02 [classifier]
    Feature request
    ------------------------------------------------------------
    03 [action]
    Log as an enhancement request to add a dark mode for night-shift use.
    ```

    > **Tip**: 속도 제한을 초과하여 앱이 실패하면 몇 초 기다린 후 다시 시도합니다. `feedback` 문자열을 불만이나 칭찬으로 편집하고 다시 실행하여 분류와 권장 작업이 어떻게 바뀌는지 확인해 보세요.

> ✅ **Checkpoint**: Microsoft Agent Framework를 사용해 세 에이전트를 순서대로 오케스트레이션하고, 한 전문가에서 다음 전문가로 작업을 전달하며, 모든 에이전트의 출력을 수집했습니다.

완료했으면 `deactivate`를 입력하여 가상 환경을 종료합니다.

---

**다음(선택 사항):** [작업 3 — A2A로 원격 에이전트 연결](C3-connect-remote-agents-with-a2a.md)