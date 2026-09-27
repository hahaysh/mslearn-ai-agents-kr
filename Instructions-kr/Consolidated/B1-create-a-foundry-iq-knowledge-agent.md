---
title: '작업 1 – Foundry IQ 지식 에이전트를 만들고 코드에서 연결'
lab:
    title: '작업 1 – Foundry IQ 지식 에이전트를 만들고 코드에서 연결'
    description: 'Microsoft Foundry 포털에서 엔터프라이즈 지식 에이전트를 만들고, Foundry IQ를 사용하여 Caldova 지식 베이스를 기반으로 응답하게 하며, 지식 조회 전에 승인을 요구하도록 설정한 다음 코드에서 연결하고 승인 흐름을 처리합니다.'
    type: 'task'
    parent: 'B'
    order: 1
    section: 'core'
    difficulty: 3
    duration: 35
    access: 'open'
    level: 300
    concepts: 'Foundry IQ, 엔터프라이즈 지식 기반화, 도구 승인, conversations API'
    status: 'draft'
---

# 작업 1 — Foundry IQ 지식 에이전트를 만들고 코드에서 연결

*이 작업은 **엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합** 랩의 일부입니다. 처음 오셨나요? [시작하기](B0-getting-started.md)부터 시작하세요.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트(배포된 모델 포함)와 시작 코드가 필요합니다. 아직 완료하지 않았다면 [시작하기](B0-getting-started.md)를 완료하여 프로젝트를 만들고, 코드를 복제하고, `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에서 설정합니다. 그런 다음 VS Code에서 연 `Python` 폴더에서 준비 상태를 확인합니다.

```
python ../setup/check_env.py --task 1
```

> **Continuing from a previous task?** 프로젝트, 가상 환경, `.env`가 이미 설정되어 있다면 설정을 건너뛰고 아래의 **에이전트 만들기**로 바로 이동할 수 있습니다.

---

이 작업에서는 **Caldova 직원 지식 도우미**를 빌드합니다. 이 에이전트는 **Foundry IQ**를 사용하여 회사 내부 문서(사이트 운영, 공장 용량, CMO 디렉터리, 기술 이전, 공급업체)를 기반으로 응답하며, 각 지식 조회를 승인 단계로 제어하는 Python 앱에서 연결합니다.

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
<summary>Foundry IQ란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

**Foundry IQ**는 에이전트를 **지식 베이스**에 연결합니다. 이 지식 베이스는 Azure AI Search가 지원하는, 자체 문서에서 만들어진 검색 가능한 인덱스입니다. 에이전트에 사실 정보가 필요하면 해당 지식 베이스를 대상으로 *에이전트형 검색*을 수행하고 찾은 내용을 인용합니다. 각 조회 전에 **승인**을 요구하여 애플리케이션이 모든 지식 베이스 액세스를 검토하고 제어하도록 할 수 있습니다.

[자세히 알아보기 →](https://learn.microsoft.com/azure/ai-foundry/)

</div>
</details>

## 에이전트 만들기

시작하기 중에 `caldova-knowledge-agent`를 만들었다면 지금 엽니다(**Build** → **Agents** → **caldova-knowledge-agent**). 그런 다음 **데이터 및 Foundry IQ 구성**으로 건너뜁니다. 그렇지 않으면 다음을 수행합니다.

1. 홈 페이지에서 **Build** 탭을 선택한 다음 **Agents** 탭에서 **Create agent**를 선택합니다.
1. 이름이 `caldova-knowledge-agent`인 에이전트를 만듭니다.

에이전트를 만들면 기본 모델(예: `gpt-5`)이 배포됩니다. 에이전트가 만들어지면 해당 기본 모델이 자동으로 선택된 에이전트 플레이그라운드가 표시됩니다.

## 데이터 및 Foundry IQ 구성

이제 Foundry IQ를 사용하여 Caldova 지식 베이스를 검색하도록 에이전트를 구성합니다.

1. 먼저 에이전트에 다음 지침을 제공합니다.

    ```
    You are the Caldova staff knowledge assistant, specializing in plant capacity,
    contract manufacturers, tech transfer, site operations, and suppliers. You must
    ALWAYS search the knowledge base to answer questions about our capacity, policies,
    or procedures. Provide detailed, accurate information and always cite your sources.
    If you don't find relevant information in the knowledge base, say so clearly.
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    당신은 공장 용량, 계약 제조업체, 기술 이전, 사이트 운영, 공급업체를 전문으로 하는 Caldova 직원 지식 도우미입니다. 당사의 용량, 정책 또는 절차에 대한 질문에 답할 때는 항상 지식 베이스를 검색해야 합니다. 자세하고 정확한 정보를 제공하고 항상 출처를 인용하세요. 지식 베이스에서 관련 정보를 찾지 못하면 그 사실을 명확히 말하세요.
    ```

1. **Save**를 선택하여 현재 에이전트 구성을 저장합니다.
1. 그런 다음 **Knowledge** 섹션에서 **Add** 드롭다운을 확장하고 **Connect to Foundry IQ**를 선택합니다.
1. Foundry IQ 설정 창에서 **Connect to an AI Search resource**를 선택한 다음 **Create new resource**를 선택합니다. 그러면 리소스를 만드는 대화 상자가 열립니다.
1. 기본 설정으로 검색 리소스를 만듭니다.
    - **Resource name**: *전역적으로 고유한 이름*
    - **Subscription**: *사용자의 Azure 구독*
    - **Resource group**: *프로젝트와 동일한 리소스 그룹 사용*
    - **Region**: *프로젝트와 동일한 위치*
    - **Pricing tier**: 가능하면 Free, 그렇지 않으면 Basic 선택

이제 Foundry IQ로 연결할 Caldova 지식 문서를 업로드합니다.

1. 샘플 지식 문서를 다운로드합니다. 시작 코드의 `Labfiles/B-integrate-agents-with-enterprise-knowledge-and-m365/Python/data/` 아래에 있는 Markdown 파일입니다.
    - `caldova-site-operations.md`
    - `caldova-plant-capacity.md`
    - `caldova-cmo-directory.md`
    - `caldova-tech-transfer-playbook.md`
    - `caldova-capacity-booking-policy.md`
    - `caldova-supplier-guide.md`

    > **Tip**: 시작하기에서 이미 이 파일들을 로컬에 받았습니다. 직접 다운로드하려면 [리포지토리](https://github.com/MicrosoftLearning/mslearn-ai-agents/tree/main/Labfiles/B-integrate-agents-with-enterprise-knowledge-and-m365/Python/data)의 `data` 폴더로 이동하여 각 파일을 저장합니다.

1. 새 탭을 열고 `https://portal.azure.com`의 Azure 포털로 이동합니다. 위쪽 검색 창에서 **Storage accounts**를 검색하고 서비스 섹션에서 **Storage accounts**를 선택합니다.
1. 다음 설정으로 스토리지 계정을 만듭니다.
    - **Subscription**: *사용자의 Azure 구독*
    - **Resource group**: *프로젝트와 동일한 리소스 그룹 사용*
    - **Storage account name**: *고유한 스토리지 계정 이름*
    - **Region**: *프로젝트와 동일한 위치*
    - **Primary service**: *Azure Blob Storage 또는 Azure Data Lake Storage*
    - **Performance**: *Standard*
    - **Redundancy**: *Locally-redundant storage (LRS)*
1. 만들어지면 만든 스토리지 계정으로 이동하고 위쪽 표시줄에서 **Upload**를 선택합니다.
1. **Upload blob** 블레이드에서 `caldovaproducts`라는 새 컨테이너를 만듭니다.
1. `data` 폴더에서 여섯 개의 Caldova Markdown 파일을 찾아 모두 선택한 다음 **Upload**를 선택합니다.
1. 파일 업로드가 완료되면 만든 검색 서비스로 이동합니다.
1. 왼쪽 창의 **Security + networking** > **Keys** 아래에서 API Access control로 **Both**를 선택하고 선택을 확인합니다. 완료되면 Azure Portal 탭은 열린 상태로 두고 Foundry 포털 탭으로 돌아가 페이지를 새로 고칩니다.
1. **Knowledge** 페이지에 있는지 확인하고 **Create a knowledge base**를 선택합니다. 지식 원본으로 **Azure Blob Storage**를 선택한 다음 **Connect**를 선택합니다.
1. 다음 설정으로 지식 원본을 구성합니다.
    - **Name**: `ks-caldovaproducts`
    - **Description**: `Caldova staff knowledge base`(Caldova 직원 지식 베이스)
    - **Storage account name**: *스토리지 계정 선택*
    - **Container name**: `caldovaproducts`
    - **Authentication type**: *API Key*
    - **Content extraction mode**: *minimal*
    - **Embedding model**: *사용 가능한 배포 모델 선택, 아마 text-embedding-3-small*
    - **Chat completions model**: *사용 가능한 배포 모델 선택, 아마 gpt-5*
1. **Create**를 선택합니다.
1. 지식 베이스 만들기 페이지에서 **Chat completions model** 드롭다운에서 `gpt-5` 모델을 선택하고 나머지 필드는 기본값으로 둡니다.
1. **Save knowledge base**를 선택한 다음 브라우저를 새로 고쳐 지식 원본 상태가 *active*인지 확인합니다. 아직 그렇지 않다면 1분 정도 기다린 뒤 상태가 활성화될 때까지 페이지를 새로 고칩니다.
1. 뒤로 단추를 선택하여 **Knowledge** 페이지로 돌아간 다음 *Connection* 드롭다운 옆의 **Manage** 링크를 선택합니다.
1. **Connected resources**까지 아래로 스크롤합니다. 검색 서비스가 표시되어야 합니다. 해당 행을 선택하고 **Authentication** 섹션을 찾습니다.
1. **Key authentication**을 선택한 다음 **Edit authentication**을 선택합니다.
1. 대화 상자를 열린 상태로 두고, 검색 서비스 **Keys** 페이지가 열려 있어야 하는 Azure 포털 탭으로 돌아갑니다. 해당 키 중 하나를 Foundry의 대화 상자에 복사하고 **Save**를 선택합니다.

이제 Foundry IQ 설정이 완료되었습니다.

## 플레이그라운드에서 에이전트 테스트

코드에서 연결하기 전에 포털 플레이그라운드에서 에이전트를 테스트합니다.

1. **Build** > **Agents** 페이지에서 에이전트로 다시 이동하고 만든 에이전트를 선택합니다.
2. 에이전트 페이지에 플레이그라운드 탭이 선택되어 있어야 합니다. 지식 섹션을 찾고 만든 연결 및 지식 베이스를 선택하여 Foundry IQ를 추가합니다.
1. 에이전트가 지식 베이스에서 정보를 검색할 수 있는지 확인하기 위해 다음 테스트 쿼리를 시도합니다.
    - `Which sites can make oral solid dose product?`
    - `Tell me which contract manufacturers are qualified for sterile work.`
    - `How much headroom does Calderwood have?`

1. 응답을 검토하고 다음을 확인합니다.
    - 에이전트가 지식 베이스의 특정 정보를 제공합니다.
    - 원본 문서에 대한 인용 또는 참조가 포함될 수 있습니다.
    - 에이전트가 Caldova 정보에 집중합니다.

1. 에이전트 세부 정보 페이지에서 다음 정보를 찾아 메모장에 복사합니다(나중에 필요합니다).
    - **Agent name**: 만든 이름입니다(`caldova-knowledge-agent`).
    - **Project endpoint**: 프로젝트 설정 또는 홈 페이지에서 찾을 수 있습니다.

### 도구 호출에 승인이 필요하도록 에이전트 구성

포털에서 에이전트를 만들면 해당 Foundry IQ(지식) 도구는 기본적으로 승인을 요청하지 **않고** 실행됩니다. 앱이 각 지식 베이스 조회를 검토하고 제어할 수 있도록 Foundry Toolkit for VS Code 확장을 사용하여 도구를 사용하기 전에 승인을 요구하도록 에이전트를 변경합니다.

> **Note**: Foundry 포털은 현재 이 승인 동작을 변경하는 설정을 노출하지 않으므로, 대신 Foundry Toolkit 확장에서 구성합니다.

1. Visual Studio Code의 왼쪽 창에서 **Extensions**를 선택하거나(**Ctrl+Shift+X** 누르기), 마켓플레이스에서 Microsoft의 `Foundry Toolkit for VS Code` 확장을 검색하고 아직 설치되어 있지 않다면 **Install**을 선택합니다.

    > **Note**: 확장은 현재 **Foundry Toolkit**으로 표시되지만, 일부 VS Code 레이블, 명령 또는 오래된 스크린샷에는 여전히 **AI Toolkit**이라고 표시될 수 있습니다. 이 랩에서는 두 이름이 같은 확장 환경을 가리킨다고 생각하면 됩니다.

1. 사이드바에서 **Foundry Toolkit** 아이콘을 선택하고, 메시지가 표시되면 Azure 계정에 로그인합니다.

    > **Note**: Foundry Toolkit 확장으로 로그인할 수 없는 경우 Azure 확장을 선택해야 할 수 있습니다. 거기에서 로그인한 다음 Foundry Toolkit으로 돌아가 리소스에 액세스합니다.

1. **Microsoft Foundry Resources** 아래에서 **Set Default Project**를 선택하고 앞서 만든 프로젝트를 선택합니다.
1. 프로젝트 섹션을 확장합니다. **Prompt Agents** 아래에서 `caldova-knowledge-agent` 에이전트를 선택하여 **Agent Builder** 창을 엽니다.
1. **Tools** 섹션에서 `kb-knowledgebase` 접두사 뒤에 고유 ID가 붙은 도구(예: `kb-knowledgebase677-7w5fj`)를 찾습니다. 이것이 Foundry IQ 지식 베이스 도구이며, 포털에서 Foundry IQ를 연결했을 때 자동으로 추가되었습니다.

    > **Note**: 에이전트에는 둘 이상의 도구가 표시됩니다. Foundry 포털은 새 에이전트에 기본적으로 **Web search** 도구를 추가하며, 독립 실행형 **Azure AI Search** 도구도 표시될 수 있습니다. 에이전트가 지식 베이스를 검색할 때 실제로 호출하는 도구는 `kb-knowledgebase...` 도구이므로, 다른 도구에 승인을 설정해도 효과가 없습니다.
1. `kb-knowledgebase...` 도구의 점 세 개를 선택합니다. 그런 다음 **Require approval before using tools** 드롭다운에서 **Ask for approval for all tools**를 선택하고, 메시지가 표시되면 변경 내용을 저장합니다.

이제 에이전트는 Foundry IQ를 사용하여 지식 베이스를 검색할 때마다 승인을 요청하며, 다음에 완료할 클라이언트 앱이 이를 처리합니다.

## 코드에서 에이전트에 연결

이제 에이전트와 통신하고 승인 흐름을 처리하는 Python 콘솔 클라이언트를 완성합니다. 시작 파일은 `Python` 폴더에 제공됩니다.

`Python` 폴더를 열고 [시작하기](B0-getting-started.md)의 가상 환경을 활성화한 다음(`.\labenv\Scripts\Activate.ps1`) 아래를 계속 진행합니다.

1. **Python/.env**에서 `AGENT_NAME`이 `caldova-knowledge-agent`(`.env.example`의 기본값)로 설정되어 있는지 확인합니다. 파일을 저장합니다.

1. **knowledge_agent.py**를 열고 다음을 포함한 시작 코드를 검토합니다.
    - 가져오기 문 및 구성 로드
    - `send_message_to_agent()` 함수 구조
    - `display_conversation_history()` 함수
    - 기본 프로그램 루프

1. 첫 번째 **TODO** 주석을 찾아 다음 코드를 추가하여 프로젝트에 연결하고, OpenAI 클라이언트를 가져오고, 에이전트를 검색하고, 새 대화를 만듭니다.

    > **Tip**: 올바른 들여쓰기 수준을 유지하도록 주의하세요.

    ```python
    # Connect to the project and agent
    credential = DefaultAzureCredential(
        exclude_environment_credential=True,
        exclude_managed_identity_credential=True
    )
    project_client = AIProjectClient(
        credential=credential,
        endpoint=project_endpoint
    )

    # Get the OpenAI client
    openai_client = project_client.get_openai_client()

    # Get the agent
    agent = project_client.agents.get(agent_name=agent_name)
    print(f"Connected to agent: {agent.name} (id: {agent.id})\n")

    # Create a new conversation
    conversation = openai_client.conversations.create(items=[])
    print(f"Created conversation (id: {conversation.id})\n")
    ```

1. `send_message_to_agent()` 함수 내부의 두 번째 **TODO** 주석을 찾아 다음 코드를 추가하여 Foundry IQ 승인 요청을 포함해 메시지를 보내고 응답을 처리합니다.

    ```python
    # Add user message to the conversation
    openai_client.conversations.items.create(
        conversation_id=conversation.id,
        items=[{"type": "message", "role": "user", "content": user_message}],
    )

    # Store in conversation history (client-side)
    conversation_history.append({
        "role": "user",
        "content": user_message
    })

    # Create a response using the agent
    response = openai_client.responses.create(
        conversation=conversation.id,
        extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
        input=""
    )

    # Check if the response output contains an MCP approval request
    approval_request = None
    if hasattr(response, 'output') and response.output:
        for item in response.output:
            if hasattr(item, 'type') and item.type == 'mcp_approval_request':
                approval_request = item
                break

    # Handle approval request if present
    if approval_request:
        print(f"[Approval required for: {approval_request.name}]\n")
        print(f"Server: {approval_request.server_label}")

        # Parse and display the arguments (optional, for transparency)
        import json
        try:
            args = json.loads(approval_request.arguments)
            print(f"Arguments: {json.dumps(args, indent=2)}\n")
        except Exception:
            print(f"Arguments: {approval_request.arguments}\n")

        # Prompt user for approval
        approval_input = input("Approve this action? (yes/no): ").strip().lower()

        if approval_input in ['yes', 'y']:
            print("Approving action...\n")

            # Create approval response item
            approval_response = {
                "type": "mcp_approval_response",
                "approval_request_id": approval_request.id,
                "approve": True
            }
        else:
            print("Action denied.\n")

            # Create denial response item
            approval_response = {
                "type": "mcp_approval_response",
                "approval_request_id": approval_request.id,
                "approve": False
            }

        # Add the approval response to the conversation
        openai_client.conversations.items.create(
            conversation_id=conversation.id,
            items=[approval_response]
        )

        # Get the actual response after approval/denial
        response = openai_client.responses.create(
            conversation=conversation.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
            input=""
        )
    ```

1. 코드를 추가한 후 파일을 저장합니다.

1. 코드가 conversations API를 사용하여 에이전트와의 상호 작용을 관리하는 방식을 검토합니다.
    - 대화가 만들어지고 해당 ID로 추적됩니다.
    - `conversations.items.create()`를 사용하여 사용자 메시지가 대화에 추가됩니다.
    - 에이전트 참조와 함께 `responses.create()`를 사용하여 응답이 생성됩니다.
    - **승인 처리**: 에이전트가 Foundry IQ에 액세스해야 할 때 응답 출력에 `mcp_approval_request`가 반환됩니다.
    - 계속하기 전에 코드는 작업을 승인할지 거부할지 묻습니다.
    - 승인/거부 후 `mcp_approval_response`가 대화에 추가되고 새 응답이 생성됩니다.

## 통합 테스트

이제 애플리케이션을 실행하고 에이전트가 지식 베이스에서 정보를 검색하는 기능을 테스트합니다.

1. 터미널(`Python` 폴더)에서 Azure에 로그인합니다.

    ```
    az login
    ```

    > **Note**: 대부분의 시나리오에서는 *az login*만 사용해도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트를 지정해야 할 수 있습니다.

1. 메시지가 표시되면 로그인 프로세스를 완료하고, 메시지가 표시되면 Foundry 리소스가 포함된 구독을 선택합니다.

1. 애플리케이션을 실행합니다.

    ```
    python knowledge_agent.py
    ```

1. 애플리케이션이 시작되면 다음 쿼리로 에이전트를 테스트합니다.

    **쿼리 1 - 제품 범주:**

    ```
    Which sites can make oral solid dose product?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    어떤 사이트에서 경구용 고형 제형 제품을 만들 수 있나요?
    ```

    승인을 요청하는 메시지가 표시되면 에이전트가 지식 베이스를 검색하도록 **yes**를 입력합니다. 에이전트가 여러 문서에서 정보를 검색하는 방식을 관찰합니다.

    **쿼리 2 - 용량 정책:**

    ```
    How much headroom does Calderwood have and how are transfer costs calculated?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    Calderwood에는 여유 용량이 얼마나 있으며 이전 비용은 어떻게 계산되나요?
    ```

    요청을 승인하고 에이전트가 용량 요청 정책에서 가져온 구체적인 세부 정보를 제공하는 방식을 확인합니다.

    **쿼리 3 - 계약 제조업체 비교:**

    ```
    What's the difference between Norvent and Halden for a sterile transfer?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    무균 이전과 관련하여 Norvent와 Halden의 차이점은 무엇인가요?
    ```

    요청을 승인하고 에이전트가 CMO 디렉터리의 정보를 종합하는 방식을 확인합니다.

    **쿼리 4 - 공급업체 및 재주문:**

    ```
    When should we reorder sterile vials, and who is our component supplier?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    무균 바이알은 언제 재주문해야 하며 당사의 구성품 공급업체는 누구인가요?
    ```

    요청을 승인하고 에이전트가 공급업체 가이드에 따라 답변하는 방식을 관찰합니다.

    **쿼리 5 - 후속 질문:**

    ```
    What are our site core hours?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    당사 사이트의 핵심 운영 시간은 어떻게 되나요?
    ```

    에이전트가 대화 컨텍스트를 유지하고 사이트 운영 문서에서 답변하는 방식을 확인합니다.

1. 전체 대화 기록을 보려면 `history`를 입력합니다.

1. 테스트가 끝나면 `quit`를 입력합니다.

> ✅ **Checkpoint**: **Foundry IQ**로 엔터프라이즈 지식 에이전트를 만들고 **기반화**했으며, 각 지식 조회 전에 **승인**을 요구하도록 설정하고, 코드에서 연결하여 승인 흐름을 직접 처리했습니다. 이것이 이 랩의 Core입니다. 아래 내용은 모두 선택 사항입니다.

### 선택 사항: 동일한 에이전트를 웹 채팅 앱으로 실행

동일한 기반화된 에이전트를 공유 Caldova 웹 채팅 창을 통해 제공할 수 있습니다. `Python` 폴더에서 다음을 실행합니다.

```
python knowledge_chat_app.py
```

브라우저가 `http://localhost:7860`에서 열리고 **Caldova Staff Knowledge Assistant**가 표시됩니다. 이 변형은 채팅이 원활하게 유지되도록 Foundry IQ 지식 도구를 **자동 승인**합니다. 위와 같은 질문을 해 봅니다. 중지하려면 탭을 닫고 **Ctrl+C**를 누릅니다.

> **Fast-forward**: 포털 대신 코드에서 에이전트를 기반화하려면 `python ../setup/bootstrap_agent.py`를 `Python` 폴더에서 실행합니다. 이 스크립트는 `caldova-knowledge-agent`를 만들고, File Search로 여섯 개의 지식 문서를 기반화하며, `AGENT_NAME`을 `.env`에 씁니다. 이 에이전트에 대해 실행하는 클라이언트 코드는 동일합니다.

완료되면 `deactivate`를 입력하여 가상 환경을 종료합니다.

---

**다음(선택 사항):** [작업 2 — Microsoft Teams에 게시](B2-publish-to-microsoft-teams.md) · [작업 3 — Microsoft 365 Copilot에 게시](B3-publish-to-microsoft-365-copilot.md) · [작업 4 — Work IQ](B4-work-iq-workplace-intelligence.md)


