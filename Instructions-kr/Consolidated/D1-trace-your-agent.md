---
title: '작업 1 – 에이전트 추적'
lab:
    title: '작업 1 – 에이전트 추적'
    description: 'OpenTelemetry로 에이전트를 계측하고, 추적을 Azure Monitor로 내보내고, 직접 만든 span과 특성을 추가한 다음 Foundry portal에서 결과를 읽습니다.'
    type: 'task'
    parent: 'D'
    order: 1
    section: 'core'
    difficulty: 3
    duration: 25
    access: 'open'
    level: 300
    concepts: '추적, OpenTelemetry, Azure Monitor, Application Insights'
    islab: true
    status: 'draft'
---

# 작업 1 — 에이전트 추적

***에이전트 관찰, 평가 및 보안** 랩의 일부입니다. 처음이라면 [시작하기](D0-getting-started.md)부터 시작하세요.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트, 여기에 연결된 **Application Insights
> 리소스**, 추적할 grounded agent, 시작 코드가 필요합니다. 아직 완료하지 않았다면
> [시작하기](D0-getting-started.md)를 완료해 프로젝트를 만들고, 코드를 복제하고,
> `PROJECT_ENDPOINT`와 `AGENT_NAME`을 `Python/.env`에 설정합니다. 이 값을
> [Lab B](B-integrate-agents-with-enterprise-knowledge-and-m365.md) 에이전트로 지정하거나
> `python ../setup/bootstrap_agent.py`로 새로 만든 다음 Application Insights를 연결합니다.
> 그런 다음 VS Code에서 연 `Python` 폴더에서 준비되었는지 확인합니다.

```
python ../setup/check_env.py --task 1
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 방금 다른 작업을 마쳤다면 프로젝트,
> 가상 환경, `.env`가 이미 설정되어 있습니다. 아래 **에이전트 계측**으로 바로 이동합니다.

---

Caldova의 Ashford 현장 월요일 아침입니다. 계획 담당자들은 회의 사이에 도우미에게 질문을 쏟아내고,
계획 책임자는 일부 답변이 "너무 오래 걸린다"고 말합니다. 어떤 답변인지, 왜 그런지 알 수 없습니다.
터미널에는 답변만 표시되고, 그 답변이 어떻게 만들어졌는지는 아무것도 보이지 않습니다.

**Tracing**은 이 문제를 해결합니다. 코드는 **span**을 내보냅니다. span은 작업을 시간, 이름, 중첩 구조로
기록한 항목입니다. 그리고 이를 **Application Insights**로 보내면 Foundry portal이 단계별로 살펴볼 수
있는 waterfall로 렌더링합니다.

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
<summary>OpenTelemetry란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

**OpenTelemetry**는 trace, metric, log를 내보내기 위한 공급업체 중립 표준입니다.
**span**은 시작, 종료, 특성을 가진 하나의 작업 단위이며, span들이 중첩되어 전체 작업의 **trace**를
형성합니다. 표준이기 때문에 Azure SDK, OpenAI 클라이언트, 사용자의 코드가 모두 같은 waterfall에
정렬되는 span을 생성합니다. 또한 계측을 다시 작성하지 않고도 내일 다른 백 엔드로 보낼 수 있습니다.

</div>
</details>

> **Server-side traces come free.** 이제 Application Insights가 프로젝트에 연결되었으므로,
> Foundry는 자신이 호스트하는 에이전트의 trace를 이미 기록합니다. 코드가 필요하지 않습니다. 여기서
> 추가하는 것은 **client-side** 계측입니다. 즉 *사용자* 코드 주위의 span입니다. 둘 다 같은
> Application Insights 리소스에 저장되지만, Foundry의 **Agents > Traces** 페이지는 자체
> server-side 보기(`Invoke Agent` / `Execute tool` / `Chat`)만 렌더링합니다. 사용자가 만든 span과
> 특성을 함께 보려면 Application Insights를 직접 확인합니다.

[시작하기](D0-getting-started.md)의 `Python` 폴더를 열고 가상 환경(`.\labenv\Scripts\Activate.ps1`)을 활성화한 다음 아래를 계속 진행합니다.

### 에이전트 계측

**traced_agent.py**를 열고 주석 처리된 각 자리 표시자에 코드를 추가합니다.

> **Tip**: 코드를 추가할 때 들여쓰기를 주석과 맞춥니다.

1. **참조 추가**:

    ```python
    # Add references
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.monitor.opentelemetry import configure_azure_monitor
    from opentelemetry import trace
    ```

1. **GenAI tracing 켜기** — 모델 호출을 캡처하는 span은 기본적으로 꺼져 있으며, 프롬프트에 개인 데이터가
    포함될 수 있으므로 메시지 콘텐츠도 별도로 꺼져 있습니다. 이 랩에서는 둘 다 켭니다.

    ```python
    # Turn on GenAI tracing
    os.environ.setdefault("AZURE_EXPERIMENTAL_ENABLE_GENAI_TRACING", "true")
    os.environ.setdefault("OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT", "true")
    ```

    > 클라이언트가 만들어지기 **전에** 이 값을 설정해야 하므로 파일 맨 위에 배치합니다.
    > 프로덕션에서는 메시지 콘텐츠를 켜기 전에 신중하게 고려합니다.

1. **프로젝트에 연결**:

    ```python
    # Connect to the project
    with (
        DefaultAzureCredential() as credential,
        AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
        project_client.get_openai_client() as openai_client,
    ):
    ```

1. **Application Insights 연결 문자열을 읽고 trace 내보내기 시작** — 프로젝트는 설정에서 연결한
    리소스의 연결 문자열을 제공하며, `configure_azure_monitor`가 exporter를 연결합니다.

    ```python
    # Read the Application Insights connection string and start exporting traces
    try:
        connection_string = project_client.telemetry.get_application_insights_connection_string()
    except Exception as error:
        raise SystemExit(
            "Could not read an Application Insights connection string from this project.\n"
            "In the Foundry portal, open your project, select Agents > Traces, and select\n"
            f"Connect to create or connect an Application Insights resource.\n\nDetails: {error}"
        )
    configure_azure_monitor(connection_string=connection_string)
    ```

1. **이 스크립트용 tracer 가져오기** — tracer는 사용자 고유 span을 만드는 데 사용합니다.

    ```python
    # Get a tracer for this script
    tracer = trace.get_tracer(__name__)
    ```

1. **에이전트 조회** — Foundry portal에서 이 특정 에이전트와 trace를 상호 연결하려면 이름뿐 아니라
    해당 `id`가 필요합니다.

    ```python
    # Look up the agent so its id can be included in agent_reference
    agent = project_client.agents.get(agent_name=agent_name)
    ```

1. **각 질문을 자체 span 안에서 묻기** — 이 부분이 효과를 발휘합니다. 바깥쪽 span은 검토를 나타내고,
    각 질문은 하위 span을 갖습니다. portal에서 구분할 수 있도록 선택한 특성으로 태그를 지정합니다.
    이 작업 전용으로 별도 에이전트를 세우지 않고 `caldova-knowledge-agent`를 다시 사용합니다.

    ```python
    # Ask each question inside its own span
    with tracer.start_as_current_span("morning-planning-review") as shift_span:
        shift_span.set_attribute("caldova.site", "ashford")
        conversation = openai_client.conversations.create()

        for number, question in enumerate(QUESTIONS, start=1):
            with tracer.start_as_current_span("planner-question") as question_span:
                question_span.set_attribute("caldova.question_number", number)
                response = openai_client.responses.create(
                    conversation=conversation.id,
                    input=question,
                    extra_body={"agent_reference": {"name": agent.name, "id": agent.id, "type": "agent_reference"}},
                )
                question_span.set_attribute("caldova.answer_length", len(response.output_text))
                print(f"\nQ{number}: {question}")
                print(f"A{number}: {response.output_text}")
    ```

1. 파일을 저장합니다(**Ctrl+S**).

### 실행 및 테스트

1. 터미널에서 로그인하고 앱을 실행합니다.

    ```
    az login
    ```

    ```
    python traced_agent.py
    ```

1. 세 개의 답변이 출력되는 것을 볼 수 있습니다.

    ```
    Q1: How long does review take for a capacity request with a complete brief?
    A1: ...
    ```

    > 연결 문자열 오류가 발생하면 Application Insights가 아직 프로젝트에 연결되지 않은 것입니다.
    > [시작하기](D0-getting-started.md)로 돌아가 연결합니다.

### trace 읽기

**두 보기, 두 목적.** Foundry의 **Agents > Traces** 페이지는 에이전트 자체 턴의 자동
**server-side** trace(`Invoke Agent` > `Execute tool` / `Chat`)를 보여 줍니다. trace가 작동 중임은
확인할 수 있지만, 스크립트가 방금 추가한 **client-side** span은 표시하지 않습니다.
`morning-planning-review`, `planner-question`, 그리고 해당 사용자 지정 특성을 보려면 Application Insights
리소스를 직접 확인합니다.

1. [Foundry portal](https://ai.azure.com)에서 프로젝트를 열고 **Agents**, **caldova-knowledge-agent**,
    **Traces**를 차례로 선택해 실행이 도착했는지 확인합니다(telemetry는 1~2분 정도 걸립니다. 아직 보이지
    않으면 기다린 뒤 새로 고칩니다). 여기에서 trace를 선택하면 사용자의 사용자 지정 span이 아니라
    Foundry 자체 `Invoke Agent` / `Execute tool` / `Chat` 보기가 표시됩니다.

1. [Azure portal](https://portal.azure.com)에서 Application Insights 리소스를 엽니다. 이 프로젝트용으로
    만든 리소스 그룹을 열고 그 안의 Application Insights 리소스를 선택합니다.

1. Application Insights의 왼쪽 탐색에서 **Investigate**를 확장하고 **Search**를 선택한 다음
    `morning-planning-review`를 검색합니다.

1. 일치하는 결과를 선택해 **end-to-end transaction details**를 엽니다. 이것이 원시 span 트리입니다.
    맨 위에는 `morning-planning-review` span이 있고, 세 개의 `planner-question` 자식이 있으며,
    각 자식 안에는 SDK가 내보낸 모델 호출이 있습니다.

1. `planner-question` span을 선택하고 속성을 살펴봅니다. 사용자의 `caldova.question_number`와
    `caldova.answer_length`가 표준 GenAI 특성과 함께 표시됩니다.

1. 세 질문의 기간을 비교합니다. 계획 책임자의 불만이 데이터로 답변됩니다. 또한 span 세부 내역은
    느린 질문의 *어느 부분*이 느렸는지 알려 줍니다.

> **Try it**: `QUESTIONS`에 훨씬 더 어려운 네 번째 질문을 추가하고 다시 실행합니다. 추가 시간은 모델 호출에
> 나타나나요, 아니면 다른 곳에 나타나나요?

> ✅ **Checkpoint**: 실행 중인 에이전트 내부를 볼 수 있습니다. SDK 자체 span과 직접 만든 사용자 지정
> span이 선택한 특성과 함께 하나의 타임라인에 표시됩니다.

완료되면 `deactivate`를 입력해 가상 환경을 종료합니다.

---

**다음:** [작업 2 — 답변 품질 평가](D2-evaluate-answer-quality.md)
