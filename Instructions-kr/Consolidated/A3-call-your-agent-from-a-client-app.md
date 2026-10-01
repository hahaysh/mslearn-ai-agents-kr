---
title: '작업 3 – 클라이언트 앱에서 에이전트 호출'
lab:
    title: '작업 3 – 클라이언트 앱에서 에이전트 호출'
    description: 'Foundry SDK와 Responses API를 사용하여 작은 웹 채팅 앱에서 그라운딩된 포털 에이전트를 구동하고 인라인 차트를 표시합니다.'
    type: 'task'
    parent: 'A'
    order: 3
    section: 'optional'
    difficulty: 3
    duration: 20
    access: 'open'
    level: 300
    concepts: 'Foundry SDK, Responses API, 코드 인터프리터'
    status: 'draft'
---

# 작업 3 — 클라이언트 앱에서 에이전트 호출

***AI 에이전트 빌드 및 확장** 랩의 일부입니다. 처음 오셨나요? [시작하기](A0-getting-started.md)부터 진행합니다.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트와 스타터 코드가 필요합니다. 아직
> 완료하지 않았다면 [시작하기](A0-getting-started.md)를 완료하여 프로젝트를 만들고,
> 코드를 복제하고, `PROJECT_ENDPOINT`를 `Python/.env`에 설정합니다.

이 작업은 **그라운딩된 에이전트**를 구동합니다. 가장 빠른 방법은 코드에서 하나를 만드는
것입니다. VS Code에서 연 `Python` 폴더에서 다음을 실행합니다.

```
python ../setup/bootstrap_agent.py
```

그러면 출력 데이터가 이미 연결된 **Code Interpreter** 도구를 포함하여 `caldova-agent`를
만들고 그라운딩한 다음, `AGENT_NAME`을 `.env`에 작성합니다. 그런 다음 준비 상태를 확인합니다.

```
python ../setup/check_env.py --task 3
```

> **Already built the agent in [Task 1](A1-create-and-ground-an-agent.md)?** 스크립트 대신
> 해당 에이전트를 사용합니다. 포털에서 `caldova-agent`를 열고, 출력 데이터가 포함된
> **Code interpreter** 도구(아래 1단계)를 추가한 다음 `AGENT_NAME=caldova-agent`를 `.env`에 설정합니다.

---

**목표**: 플레이그라운드 대신 작은 **웹 채팅 앱**에서 그라운딩된 포털 에이전트와 상호
작용합니다. 에이전트가 생성하는 차트(코드 인터프리터에서 생성됨)도 포함되며, 차트는 채팅
창에 **인라인**으로 렌더링됩니다.

기존 에이전트를 이름으로 로드하고 제공된 웹 인터페이스에서 메시지를 보냅니다.

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
<summary>앱은 Foundry 에이전트를 어떻게 호출하나요?</summary>
<div class="concept-body" markdown="1">

**Foundry SDK**는 Python 코드에서 프로젝트와 에이전트에 액세스할 수 있게 합니다.
**Responses API**는 사용자의 입력을 보내고 도구 출력을 포함한 에이전트의 응답을 반환합니다.
`agent_reference`는 어떤 저장된 에이전트가 요청을 처리해야 하는지 API에 알려 주고,
conversation은 여러 턴의 메시지를 함께 유지합니다.

</div>
</details>

<details markdown="1" class="concept">
<summary>Code Interpreter는 무엇을 하나요?</summary>
<div class="concept-body" markdown="1">

Code Interpreter는 에이전트가 첨부 파일에 대해 코드를 실행할 수 있는 관리형 환경을
제공합니다. 여기에서 에이전트는 `weekly_output.csv`를 읽고, 결과를 계산하며, 차트를
만듭니다. 클라이언트는 생성된 이미지를 감지하고 채팅에 표시합니다.

</div>
</details>

**설정:**

위에서 `python ../setup/bootstrap_agent.py`를 실행했다면 에이전트, 에이전트의
**Code Interpreter** 도구, `AGENT_NAME`이 이미 구성되어 있습니다. 가상 환경
(`.\labenv\Scripts\Activate.ps1`)을 활성화하고 **직접 시도해 보기**로 건너뜁니다.

**작업 1에서 에이전트를 직접 만든 경우**, 다음 연결 작업을 완료합니다.

1. 포털에서 `caldova-agent`를 열고 **Code interpreter** 도구를 추가한 다음 분석할 데이터 파일을 업로드합니다. 다음을 다운로드하여 첨부합니다.

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/A-build-and-extend-ai-agents/Python/weekly_output.csv
    ```

    에이전트를 저장합니다.

1. `Labfiles/A-build-and-extend-ai-agents/Python` 폴더에서 가상 환경(`.\labenv\Scripts\Activate.ps1`)을 활성화합니다. 그런 다음 **.env**를 열고 `AGENT_NAME=caldova-agent`를 이미 설정한 `PROJECT_ENDPOINT`와 함께 추가합니다. 파일을 저장합니다.

> **Try it first**: `agent_with_functions.py` 파일에는 웹 채팅 창을 여는 완전한 클라이언트가
> 이미 포함되어 있습니다. 실행하기 전에 예측해 봅니다. 어떤 SDK 호출이 기존 포털 에이전트를
> *이름으로* 로드하나요? 클라이언트는 Responses API에 해당 에이전트를 사용하라고 어떻게
> 알려 주나요? `respond()` 함수는 하나의 메시지를 UI에 표시할 수 있는 답변으로 어떻게
> 바꾸나요?

<details markdown="1">
<summary>솔루션 보기</summary>

제공된 `agent_with_functions.py`는 이미 클라이언트를 구현하고, 해당 `respond()` 함수를 공유
`run_chat_app()` 셸에 전달합니다. 중요한 줄은 다음과 같습니다.

1. **이름으로 포털 에이전트 로드**(**.env**의 `AGENT_NAME` 사용):

    ```python
    agent = project_client.agents.get(agent_name=agent_name)
    ```

2. `respond()` 내부에서 Responses API를 통해 **각 요청을 해당 에이전트로 라우팅**:

    ```python
    response = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        input="",
    )
    ```

3. **인라인 차트**: 도우미 함수는 이미지 출력과 `container_file_citation`
    주석을 감지하고, 이를 `agent_outputs/` 아래에 저장한 다음, UI가 채팅에 **인라인**으로
    렌더링할 수 있도록 `AgentReply`에 반환합니다.

4. **앱 시작**: 파일 끝에서 브라우저 채팅 창을 시작합니다.

    ```python
    run_chat_app(respond, title="Caldova Supply Chain Assistant")
    ```

로그인하고 실행합니다.

```
az login
python agent_with_functions.py
```

브라우저가 `http://localhost:7860`의 채팅 창을 엽니다. 코드 인터프리터를 사용하는 내용을 요청합니다.

```
Analyze the weekly production output data and create a chart of output over time.
```
다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
```prompt
주간 생산 산출량 데이터를 분석하고 시간에 따른 산출량 차트를 만들어 주세요.
```

에이전트의 분석이 채팅에 표시되고 **차트가 인라인으로 표시됩니다**. 브라우저 탭을 닫고
터미널에서 **Ctrl+C**를 눌러 앱을 중지합니다.

</details>

**Stretch**: 각 응답 후 에이전트의 토큰 사용량을 표시합니다.

---

**다음(선택 사항):** [작업 4 — 사용자 지정 함수 도구 추가](A4-add-custom-function-tools.md)
