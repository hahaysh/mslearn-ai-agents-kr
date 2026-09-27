---
title: '작업 4 – 지원 티켓 분류 및 라우팅'
lab:
    title: '작업 4 – 지원 티켓 분류 및 라우팅'
    description: 'Microsoft Agent Framework를 사용해 triage 에이전트로 Caldova 지원 티켓을 분류한 다음, 범주와 신뢰도를 기반으로 코드에서 각 티켓을 라우팅합니다.'
    type: 'task'
    parent: 'C'
    order: 4
    section: 'optional'
    difficulty: 3
    duration: 30
    access: 'open'
    level: 300
    concepts: 'Microsoft Agent Framework, 구조화된 출력, 분류, 조건부 라우팅'
    status: 'draft'
---

# 작업 4 — 지원 티켓 분류 및 라우팅

***Agent Framework로 다중 에이전트 솔루션 빌드** 랩의 일부입니다. 처음이라면 [시작하기](C0-getting-started.md)부터 시작하세요.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트와 시작 코드가 필요합니다. 아직 완료하지 않았다면 [시작하기](C0-getting-started.md)를 완료하여 프로젝트를 만들고, 코드를 복제하고, `PROJECT_ENDPOINT` 및 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에서 설정합니다.
> 그런 다음 VS Code에서 연 `Python` 폴더에서 준비되었는지 확인합니다.

```
python ../setup/check_env.py --task 4
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 이전 작업을 방금 마쳤다면 프로젝트, 가상 환경 및 `.env`가 이미 설정되어 있습니다. 아래 **triage 에이전트 빌드**로 바로 이동합니다.

---

모든 다중 에이전트 작업에 파이프라인이 필요한 것은 아닙니다. 때로는 하나의 에이전트가 메시지를 읽고 결정을 내리는 *사고*를 담당하고, 사용자의 **코드**가 그 결정에 따라 동작합니다. 이 작업에서는 Caldova **지원 데스크 triage**를 빌드합니다. 단일 에이전트가 각 지원 티켓을 신뢰도 점수와 함께 범주로 분류하고, Python 코드가 그에 따라 티켓을 라우팅합니다. 청구 문제는 에스컬레이션하고, 낮은 신뢰도의 티켓은 더 자세한 정보를 요청하도록 되돌리며, 나머지는 자동으로 처리합니다.

이 방식을 가능하게 하는 핵심은 **구조화된 출력**입니다. 에이전트가 산문 대신 작은 JSON 개체로 답하도록 요청하면 코드가 이를 안정적으로 분기 처리할 수 있습니다.

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
<summary>하나의 큰 프롬프트 대신 코드에서 라우팅하는 이유는?</summary>
<div class="concept-body" markdown="1">

단일 에이전트에게 분류와 다음 작업 결정을 모두 요청할 *수도* 있습니다. 하지만 **결정**(에이전트의 판단)과 **라우팅**(비즈니스 규칙)을 분리하면 시스템을 테스트, 감사, 변경하기가 더 쉬워집니다. 에이전트는 작고 예측 가능한 분류를 반환하고, 코드는 다음에 일어날 일을 소유합니다. 모델에 **구조화된 출력**(여기서는 `category` 및 `confidence`가 있는 JSON 개체)을 요청하면 코드가 결과에 따라 결정론적으로 분기할 수 있습니다.

[자세히 알아보기 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

`Python` 폴더를 열고 [시작하기](C0-getting-started.md)에서 만든 가상 환경(`.\labenv\Scripts\Activate.ps1`)을 활성화한 다음 아래를 계속 진행합니다.

### triage 에이전트 빌드

**ticket_triage.py**를 열고 주석 처리된 각 자리 표시자에 코드를 추가합니다.

1. 파일에 이미 있는 코드를 검토합니다. `TRIAGE_INSTRUCTIONS`(에이전트가 `customer_issue`, `category`, `confidence`가 포함된 JSON 개체를 반환하도록 지시), `parse_classification` 도우미(응답에서 해당 JSON을 읽음), `route_ticket`(비즈니스 규칙)을 확인합니다. 샘플 티켓은 `sample_tickets.json`에서 로드됩니다.

    > **Tip**: 코드를 추가할 때 들여쓰기를 주석과 맞춥니다.

1. 파일 맨 위에서 **Add references** 주석을 찾아 필요한 네임스페이스를 추가합니다.

    ```python
    # Add references
    from agent_framework import Agent
    from agent_framework.foundry import FoundryChatClient
    from azure.identity import AzureCliCredential
    ```

1. **Create a foundry chat client** 주석을 찾아 다음을 추가합니다(들여쓰기 수준 유지).

    ```python
    # Create a foundry chat client
    client = FoundryChatClient(
        project_endpoint=os.getenv("PROJECT_ENDPOINT"),
        model=os.getenv("MODEL_DEPLOYMENT_NAME"),
        credential=AzureCliCredential(),
    )
    ```

    **AzureCliCredential**를 사용하면 코드가 `az login` 세션을 사용해 Azure에 인증할 수 있으며, **FoundryChatClient**는 Foundry 프로젝트에 연결합니다.

1. **Create the triage agent** 주석을 찾아 다음을 추가합니다.

    ```python
    # Create the triage agent
    agent = Agent(
        client=client,
        name="TicketTriageAgent",
        instructions=TRIAGE_INSTRUCTIONS,
    )
    ```

    공유 클라이언트가 지원하는 단일 에이전트가 모든 분류를 수행합니다. 에이전트의 동작은 전적으로 `TRIAGE_INSTRUCTIONS`에서 비롯됩니다.

### 각 티켓 분류 및 라우팅

1. `for` 루프 안에서 **Create a session, classify the ticket, then parse and route the result** 주석을 찾아 다음을 추가합니다(`pass` 자리 표시자를 대체).

    ```python
        # Create a session, classify the ticket, then parse and route the result
        session = agent.create_session()
        response = await agent.run(ticket, session=session)

        try:
            classification = parse_classification(response.text)
        except (ValueError, json.JSONDecodeError):
            print("  [review] Could not parse the classification. Send for manual review.")
            continue

        category = classification.get("category", "unknown")
        confidence = float(classification.get("confidence", 0))
        print(f"  Category:   {category} (confidence {confidence:.2f})")
        print(f"  Decision:   {route_ticket(classification)}")
    ```

    각 티켓에 대해 새 세션을 만들고, 에이전트를 실행하여 분류를 가져오고, JSON을 구문 분석한 다음 결과를 `route_ticket`에 전달합니다. 이는 더 큰 워크플로에서 노드별로 빌드하던 것과 같은 **분류한 다음 분기** 패턴입니다.

1. 파일을 저장합니다(**Ctrl+S**).

### 실행 및 테스트

1. 터미널에서 로그인하고 앱을 실행합니다.

    ```
    az login
    ```

    ```
    python ticket_triage.py
    ```

1. 출력을 검토합니다. 각 티켓이 분류되고 라우팅됩니다. 다음과 비슷한 출력이 표시됩니다.

    ```
    Ticket 1: The batch record terminal on packaging line B keeps losing its connection even after a full restart.
      Category:   Equipment (confidence 0.95)
      Decision:   [auto] Equipment issue: send troubleshooting steps and raise a maintenance job.

    Ticket 2: Is there a way to see all of our past capacity requests and export them as a report?
      Category:   General (confidence 0.90)
      Decision:   [auto] General question: reply with a help-center answer.

    Ticket 3: We were invoiced twice for the same transfer week last Friday and the statement shows two payments. Can someone fix this?
      Category:   Billing (confidence 0.97)
      Decision:   [escalated] Billing issue routed to the Caldova orders team.
    ```

    > **Tip**: 모호한 티켓(예: `"It's not working"`)을 `sample_tickets.json`에 추가해 보세요. 낮은 신뢰도 점수는 `CONFIDENCE_THRESHOLD`를 트리거하여 추측하는 대신 더 자세한 정보를 요청하도록 라우팅해야 합니다.

> ✅ **Checkpoint**: 단일 에이전트의 **구조화된 출력**을 사용해 코드에서 **조건부 라우팅**을 구동했습니다. 시각적 워크플로 디자이너 없이 각 티켓을 분류하고 범주와 신뢰도에 따라 분기했습니다.

완료했으면 `deactivate`를 입력하여 가상 환경을 종료합니다.

---

**다음:** 선택 작업을 완료했습니다. 요약 및 정리 단계는 [랩 개요](C-build-multi-agent-solutions-with-agent-framework.md)로 돌아가 확인합니다.