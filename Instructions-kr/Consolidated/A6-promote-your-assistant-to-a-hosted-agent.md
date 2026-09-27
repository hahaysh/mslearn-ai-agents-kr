---
title: '작업 6 – 도우미를 호스티드 에이전트로 승격'
lab:
    title: '작업 6 – 도우미를 호스티드 에이전트로 승격'
    description: 'Caldova 도우미를 호스티드 에이전트로 구현하고 배포합니다. Azure Developer CLI를 사용해 Foundry 관리 컨테이너에서 직접 작성한 코드가 실행됩니다.'
    type: 'task'
    parent: 'A'
    order: 6
    section: 'optional'
    difficulty: 3
    duration: 30
    access: 'open'
    level: 300
    concepts: '호스티드 에이전트, 배포, Azure Developer CLI'
    status: 'draft'
---

# 작업 6 — 도우미를 호스티드 에이전트로 승격

***AI 에이전트 빌드 및 확장** 랩의 일부입니다. 처음 오셨나요? [시작하기](A0-getting-started.md)부터 진행합니다.*

> **Set up (start here):** 이 작업은 코드를 배포하므로 Foundry 프로젝트, 배포된 모델,
> **Azure Developer CLI(`azd`)**가 필요합니다. 아직 완료하지 않았다면
> [시작하기](A0-getting-started.md)를 완료하여 프로젝트를 만들고 `PROJECT_ENDPOINT`와
> `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에 설정합니다. 그런 다음 VS Code에서 연
> `Python` 폴더에서 확인합니다.

```
python ../setup/check_env.py --task 6
```

> **`azd` 1.25.3 이상**과 Foundry 확장도 필요합니다. 다음 명령으로 한 번 설치합니다.
>
> ```
> azd ext install microsoft.foundry
> ```

> **Continuing from a previous task?** 호스티드 에이전트는 자체 종속성이 있는 별도 폴더
> (`Python/hosted_agent/`)에 있으므로 공유 `labenv`를 재사용하지 않습니다. 필요한 모든 내용은
> 아래에 있습니다. 이전 작업을 완료하지 않았어도 여기에서 시작할 수 있습니다.

---

**목표**: **Caldova 도우미**를 **호스티드 에이전트**로 구현하고 배포합니다. 직접 작성한 코드는
Foundry Agent Service에서 실행되며, 이전에 빌드한 프롬프트 에이전트처럼 참조로 호출할 수 있습니다.

이는 동일한 도우미 역할과 비즈니스 시나리오를 다른 방식으로 구현한 것입니다. 호스티드 버전은
Caldova 지침과 대화 기록을 유지하며, 이 작업은 요청 처리와 배포에 집중합니다. 이전 작업의
정책 그라운딩과 도구는 이 호스티드 구현 범위에 포함되지 않습니다.

프롬프트 에이전트는 모델, 지침, 도구로 정의됩니다. 호스티드 에이전트는 직접 작성한 코드를
관리형 컨테이너에서 실행합니다.

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
<summary>프롬프트 에이전트와 호스티드 에이전트의 차이는 무엇인가요?</summary>
<div class="concept-body" markdown="1">

**프롬프트 에이전트**는 선언적입니다. 모델, 지침, 도구를 제공하면 Foundry가 에이전트를
실행합니다. **호스티드 에이전트**는 자체 프레임워크 또는 Python 코드를 컨테이너로 패키지하므로
런타임 동작을 더 많이 제어할 수 있습니다. Foundry는 배포, 크기 조정, 세션 상태, ID,
엔드포인트를 관리합니다. 호스티드 에이전트는 로직이 더 이상 프롬프트와 도구 정의에 맞지 않을 때
유용합니다.

</div>
</details>

이 작업에서는 **Responses protocol**을 사용하므로, 호스티드 에이전트는 OpenAI 호환 상태를
유지합니다. 프롬프트 에이전트를 호출했던 동일한 클라이언트 코드로 이 에이전트도 호출할 수
있습니다.

## 에이전트 코드 검토

1. `Labfiles/A-build-and-extend-ai-agents/Python/hosted_agent` 폴더에서 **main.py**를 엽니다.
    `azure-ai-agentserver-responses` 호스팅 라이브러리는 웹 서버, 상태 검사, 대화 기록을 대신
    실행합니다. 여러분은 한 턴에 답변하는 **handler**만 작성합니다.

1. 두 블록이 `TODO`로 표시되어 있습니다. Responses 클라이언트를 만들고 handler에서 모델을 호출합니다. 해당 부분을 채웁니다.

> **Try it first**: handler에는 이미 사용자의 메시지(`user_input`)와 `input_items`로 조립된 대화
> 기록이 있습니다. 이를 모델 배포로 보내고 응답을 반환하려면 어떻게 해야 할까요?
> *(힌트: Responses 클라이언트의 `create(...)`는 동기식이므로, 샘플은 서버를 차단하지 않도록
> `run_in_executor`로 이벤트 루프 밖에서 실행합니다.)*

<details markdown="1">
<summary>솔루션 보기</summary>

파일 위쪽 근처에 Responses 클라이언트를 만듭니다.

```python
_responses_client = (
    AIProjectClient(endpoint=_endpoint, credential=DefaultAzureCredential())
    .get_openai_client()
    .responses
)
```

그런 다음 handler를 완성하여 Caldova 시스템 프롬프트로 모델을 호출하고 응답을 반환합니다.

```python
response = await asyncio.get_running_loop().run_in_executor(
    None,
    lambda: _responses_client.create(
        model=_model,
        instructions=_SYSTEM_PROMPT,
        input=input_items,
        store=False,
    ),
)
return TextResponse(context, request, text=response.output_text)
```

완성된 파일은 `Solution/Python/hosted_agent/main.py`에 있습니다.

</details>

## 모델 구성

1. `hosted_agent/.env.example`을 `hosted_agent/.env`로 복사하고 `AZURE_AI_MODEL_DEPLOYMENT_NAME`을 배포된 모델 이름(예: `gpt-4o`)으로 설정합니다. 호스티드 컨테이너에서는 `FOUNDRY_PROJECT_ENDPOINT`가 자동으로 삽입됩니다. 로컬에서 테스트할 때 `azd ai agent run`이 이를 자동으로 설정합니다.

## azd 프로젝트 초기화

1. `hosted_agent` 폴더에서 에이전트 정의를 스캐폴드합니다. 그러면 `azure.yaml`이 생성되며, 이는 호스티드 **`azure.ai.agent`** 서비스를 설명합니다.

    ```
    azd ai agent init --protocol responses --deploy-mode code
    ```

    프롬프트에 응답합니다. **agent name**(예: `caldova-hosted-agent`)을 선택하고, **Use an existing Foundry project**([시작하기]에서 만든 프로젝트)를 선택한 다음, 구독과 위치를 선택합니다.

> 완성된 `azure.yaml`이 `Solution/Python/hosted_agent/`에 포함되어 있어 도구가 생성하는 내용을
> 확인할 수 있습니다. `--deploy-mode code`는 Foundry가 컨테이너를 빌드한다는 뜻입니다(**원격
> 빌드**). 로컬에 Docker가 설치되어 있을 필요가 없습니다.

## 로컬에서 프로비전 및 테스트

1. Application Insights와 같은 지원 리소스를 프로비전합니다.

    ```
    azd provision
    ```

1. 에이전트를 로컬에서 실행합니다. 이렇게 하면 가상 환경을 만들고, `requirements.txt`를 설치하고, handler를 시작하고, 브라우저에서 에이전트 검사기를 엽니다.

    ```
    azd ai agent run
    ```

1. 검사기에서 채팅하거나 두 번째 터미널에서 호출합니다.

    ```
    azd ai agent invoke --local "What information should a planner gather before moving production to another factory?"
    ```

## Foundry Agent Service에 배포

1. 컨테이너를 빌드하고 Foundry에 배포합니다.

    ```
    azd deploy
    ```

    완료되면 출력에 **agent playground** 링크와 **agent endpoint**가 포함됩니다. 이제 호스티드 에이전트에는 자체 전용 엔드포인트와 ID가 있습니다.

1. 배포된 에이전트를 호출합니다.

    ```
    azd ai agent invoke "Caldova can move work between factories or hire an approved manufacturing partner. Summarize the factors the planning team should consider."
    ```

> **Same reference, your code now**: 호스티드 에이전트는 빌드한 프롬프트 에이전트와 정확히 같은
> 방식으로 호출됩니다. 이름으로, OpenAI 호환 클라이언트를 통해 호출합니다. `agent_reference`를
> 사용하는 앱은 이 호스티드 에이전트를 가리킬 수 있습니다. 이제 각 턴에 답하는 로직은 프롬프트
> 정의가 아니라 컨테이너에서 실행되는 **여러분의 코드**입니다.

> ✅ **Checkpoint**: Caldova 도우미를 호스티드 에이전트로 구현하고, `azd ai agent run`으로 로컬에서
> 테스트하고, `azd deploy`로 배포한 다음, 이름으로 배포된 에이전트를 호출했습니다.

## 정리

완료했으면 이 작업에서 만든 모든 항목을 제거합니다.

```
azd down
```

> **Warning**: `azd down`은 Foundry 프로젝트와 호스티드 에이전트를 포함하여 리소스 그룹의 모든
> 리소스를 삭제합니다. 그룹에 다른 리소스가 있으면 해당 리소스도 함께 삭제됩니다.

---

**[랩 개요](A-build-and-extend-ai-agents.md)로 돌아가기**.
