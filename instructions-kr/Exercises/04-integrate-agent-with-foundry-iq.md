---
lab:
    title: 'AI 에이전트를 Foundry IQ와 통합'
    description: 'Azure AI Agent Service를 사용하여 Foundry IQ로 기술 자료를 검색하는 에이전트를 개발합니다.'
    level: 300
    duration: 45
    islab: true
    status: 'released'
---

# AI 에이전트를 Foundry IQ와 통합

이 연습에서는 Microsoft Foundry 포털을 사용하여 Foundry IQ와 통합되는 에이전트를 만들고 기술 자료에서 정보를 검색하고 가져옵니다. 검색 리소스를 만들고, 샘플 데이터로 기술 자료를 구성하고, 포털에서 에이전트를 빌드한 다음, Visual Studio Code에서 연결하여 프로그래밍 방식으로 상호 작용합니다.

> **Tip**: 이 연습에서 사용하는 코드는 Python용 Microsoft Foundry SDK를 기반으로 합니다. Microsoft .NET, JavaScript 및 Java용 SDK를 사용하여 유사한 솔루션을 개발할 수 있습니다. 자세한 내용은 [Microsoft Foundry SDK 클라이언트 라이브러리](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)를 참조하세요.

이 연습을 완료하는 데 약 **45**분이 걸립니다.

> **Note**: 이 연습에서 사용하는 일부 기술은 미리 보기이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

이 연습을 시작하기 전에 다음이 준비되어 있는지 확인합니다.

- AI 리소스를 만들 수 있는 권한이 있는 [Azure 구독](https://azure.microsoft.com/free/)
- 로컬 컴퓨터에 설치된 [Visual Studio Code](https://code.visualstudio.com/)
- 설치된 [Python 3.13](https://www.python.org/downloads/)
- 로컬 컴퓨터에 설치된 [Git](https://git-scm.com/downloads)
- Microsoft Foundry 포털 및 Python 프로그래밍에 대한 기본 지식

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에 3.14 빌드가 없습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

## Foundry 프로젝트 만들기

새 Foundry 환경에서 Foundry 프로젝트를 만드는 것부터 시작하겠습니다.

1. 웹 브라우저에서 [Foundry 포털](https://ai.azure.com)을 `https://ai.azure.com`에서 열고 Azure 자격 증명으로 로그인합니다. 처음 로그인할 때 열리는 팁 또는 빠른 시작 창을 모두 닫습니다.

    > **Important**: 이 랩에서 업데이트된 사용자 인터페이스를 사용하려면 **New Foundry** 토글이 *On*인지 확인합니다.

1. **New Foundry**로 전환하면 프로젝트를 선택하라는 메시지가 표시됩니다. 드롭다운에서 **Create a new project**를 선택합니다.
1. **Create a project** 대화 상자에서 프로젝트에 사용할 유효한 이름을 입력합니다(예: *agent-iq-lab*).
1. 프로젝트에 대해 다음 설정을 확인하거나 구성합니다.
    - **Foundry resource**: *Create a new Foundry resource or select an existing one*
    - **Subscription**: *Your Azure subscription*
    - **Resource group**: *Create or select a resource group*
    - **Location**: *Select any available region*\*

    > \* 일부 Azure AI 리소스는 지역별 모델 할당량의 제약을 받습니다. 연습 후반에 할당량 제한을 초과하는 경우 다른 지역에 다른 리소스를 만들어야 할 수도 있습니다.

1. **Create**를 선택하고 프로젝트가 만들어질 때까지 기다립니다. 몇 분 정도 걸릴 수 있습니다.
1. 프로젝트가 만들어지면 프로젝트 홈 페이지가 표시됩니다.

## 에이전트 만들기

1. 홈 페이지에서 **Build** 탭을 선택한 다음 **Agents** 탭에서 **Create agent**를 선택합니다.
1. `product-expert-agent`와 같이 설명이 포함된 이름으로 에이전트를 만듭니다.

에이전트를 만들면 기본 모델(예: `gpt-5`)이 배포됩니다. 에이전트가 만들어지면 해당 기본 모델이 자동으로 선택된 에이전트 플레이그라운드가 표시됩니다.

## 데이터 및 Foundry IQ 구성

이제 Foundry IQ를 사용하여 기술 자료를 검색하는 에이전트를 구성합니다.

1. 먼저 에이전트에 다음 지침을 제공합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   You are a helpful AI assistant for Contoso, specializing in outdoor camping and hiking products. 
   You must ALWAYS search the knowledge base to answer questions about our products or product 
   catalog. Provide detailed, accurate information and always cite your sources.
   If you don't find relevant information in the knowledge base, say so clearly.
    ```

    ```prompt
   당신은 야외 캠핑 및 하이킹 제품을 전문으로 하는 Contoso의 유용한 AI 도우미입니다.
   당사 제품이나 제품 카탈로그에 대한 질문에 답할 때는 항상 기술 자료를 검색해야 합니다.
   자세하고 정확한 정보를 제공하고 항상 출처를 인용하세요.
   기술 자료에서 관련 정보를 찾지 못하면 이를 명확하게 말하세요.
    ```

1. **Save**를 선택하여 현재 에이전트 구성을 저장합니다.
1. 그런 다음 **Knowledge** 섹션에서 **Add** 드롭다운을 확장하고 **Connect to Foundry IQ**를 선택합니다.
1. Foundry IQ 설정 창에서 **Connect to an AI Search resource**를 선택한 다음 **Create new resource**를 선택합니다. 그러면 리소스를 만드는 대화 상자가 열립니다.
1. 기본 설정으로 검색 리소스를 만듭니다.
    - **Resource name**: *A globally unique name*
    - **Subscription**: *Your Azure subscription*
    - **Resource group**: *Use the same resource group as your project*
    - **Region**: *The same location as your project*
    - **Pricing tier**: Free *if available, otherwise choose Basic*
    - **Foundry IQ Knowledge base capabilities**: Pause til next month

    > **Note**: 여기에서 리소스를 만드는 데 문제가 발생하면 양식 아래쪽의 링크를 선택하여 대신 Azure 포털에서 만듭니다.

이제 Foundry IQ에 연결할 샘플 제품 정보 문서를 업로드합니다.

1. 새 브라우저 탭을 열고 `https://github.com/MicrosoftLearning/mslearn-ai-agents/raw/main/Labfiles/04-integrate-agent-with-foundry-iq/data/contoso-products.zip`으로 이동하여 샘플 제품 정보 파일을 다운로드합니다.
1. zip에서 파일을 추출합니다. Contoso 제품을 자세히 설명하는 PDF 3개가 있어야 합니다.
1. 새 탭을 열고 Azure 포털 `https://portal.azure.com`으로 이동합니다. 위쪽 검색 창에서 **Storage accounts**를 검색하고 서비스 섹션에서 **Storage accounts**를 선택합니다.
1. 다음 설정으로 스토리지 계정을 만듭니다.
    - **Subscription**: *Your Azure subscription*
    - **Resource group**: *Use the same resource group as your project*
    - **Storage account name**: *A unique storage account name*
    - **Region**: *The same location as your project*
    - **Primary service**: *Azure Blob Storage or Azure Data Lake Storage*
    - **Performance**: *Standard*
    - **Redundancy**: *Locally-redundant storage (LRS)*
1. 만들어지면 만든 스토리지 계정으로 이동하고 위쪽 표시줄에서 **Upload**를 선택합니다.
1. **Upload blob** 블레이드에서 `contosoproducts`라는 새 컨테이너를 만듭니다.
1. zip 파일에서 추출한 파일을 찾아 PDF 파일 3개를 모두 선택한 다음 **Upload**를 선택합니다.
1. 파일이 업로드되면 만든 검색 서비스로 이동합니다.
1. 왼쪽 창의 **Security + networking** > **Keys** 아래에서 API Access control로 **Both**를 선택하고 선택을 확인합니다. 완료되면 Azure Portal 탭을 열어 둔 채 Foundry 포털 탭으로 돌아가 페이지를 새로 고칩니다.
1. **Knowledge** 페이지에 있는지 확인하고 **Create a knowledge base**를 선택한 다음 지식 원본으로 **Azure Blob Storage**를 선택하고 **Connect**를 선택합니다.
1. 다음 설정으로 지식 원본을 구성합니다.
    - **Name**: `ks-contosoproducts`
    - **Description**: `Contoso product catalog items`
    - **Storage account name**: *Select your storage account*
    - **Container name**: `contosoproducts`
    - **Authentication type**: *API Key*
    - **Content extraction mode**: *minimal*
    - **Embedding model**: *Select the available deployed model, likely text-embedding-3-small*
    - **Chat completions model**: *Select the available deployed model, likely gpt-5*
1. **Create**를 선택합니다.
1. 기술 자료 만들기 페이지에서 **Chat completions model** 드롭다운에서 `gpt-5` 모델을 선택하고 나머지 필드는 기본값 그대로 둡니다.
1. **Save knowledge base**를 선택한 다음 브라우저를 새로 고쳐 지식 원본 상태가 *active*인지 확인합니다. 아직 active가 아니면 1분 정도 기다린 후 active가 될 때까지 페이지를 새로 고칩니다.
1. 뒤로 단추를 선택하여 **Knowledge** 페이지로 돌아간 다음 *Connection* 드롭다운 옆에 있는 **Manage** 링크를 선택합니다.
1. **Connected resources**까지 아래로 스크롤합니다. 여기에서 검색 서비스가 표시됩니다. 해당 행을 선택하고 **Authentication** 섹션을 찾습니다.
1. **Key authentication**을 선택한 다음 **Edit authentication**을 선택합니다.
1. 대화 상자를 열린 상태로 두고, 검색 서비스 **Keys** 페이지가 계속 열려 있어야 하는 Azure 포털 탭으로 돌아갑니다. 해당 키 중 하나를 Foundry의 대화 상자에 복사한 다음 **Save**를 선택합니다.

이제 Foundry IQ 설정이 완료되었습니다.

## 플레이그라운드에서 에이전트 테스트

코드에서 연결하기 전에 포털 플레이그라운드에서 에이전트를 테스트합니다.

1. **Build** > **Agents** 페이지에서 에이전트로 돌아가서 만든 에이전트를 선택합니다.
2. 에이전트 페이지에는 플레이그라운드 탭이 선택되어 있어야 합니다. 지식 섹션을 찾아 Foundry IQ를 추가하고, 만든 연결 및 기술 자료를 선택합니다.
1. 에이전트가 기술 자료에서 정보를 검색할 수 있는지 확인하려면 다음 테스트 쿼리를 시도합니다.
    - `What types of tents does Contoso offer?`
    - `Tell me about which backpacks are available in XL.`
    - `What camping accessories are available?`

1. 응답을 검토하고 다음을 확인합니다.
    - 에이전트가 기술 자료에서 구체적인 정보를 제공합니다.
    - 원본 문서에 대한 인용 또는 참조가 포함될 수 있습니다.
    - 에이전트가 제품 정보에 집중합니다.

1. 더 다듬어진 웹앱 환경을 위해 **Preview agent**에서 에이전트와 상호 작용해 볼 수도 있습니다.

1. 에이전트 세부 정보 페이지에서 다음 정보를 찾아 메모장에 복사합니다(나중에 필요합니다).
    - **Agent name**: 만든 이름입니다(`product-expert-agent`).
    - **Project endpoint**: 프로젝트 설정 또는 홈 페이지에서 찾을 수 있습니다.

### 도구 호출에 승인이 필요하도록 에이전트 구성

포털에서 에이전트를 만들면 Foundry IQ(지식) 도구는 기본적으로 승인을 요청하지 **않고** 실행됩니다. 앱에서 각 기술 자료 조회를 검토하고 제어할 수 있도록, Foundry Toolkit for VS Code 확장을 사용하여 도구를 사용하기 전에 승인을 요구하도록 에이전트를 변경합니다.

> **Note**: Foundry 포털은 현재 이 승인 동작을 변경하는 설정을 노출하지 않으므로, 대신 Foundry Toolkit 확장에서 구성합니다.

1. Visual Studio Code의 왼쪽 창에서 **Extensions**를 선택하거나 **Ctrl+Shift+X**를 누른 다음, 마켓플레이스에서 Microsoft의 `Foundry Toolkit for VS Code` 확장을 검색하고 아직 설치되어 있지 않으면 **Install**을 선택합니다.

    > **Note**: 이 확장은 현재 **Foundry Toolkit**으로 표시되지만, 일부 VS Code 레이블, 명령 또는 이전 스크린샷에서는 여전히 **AI Toolkit**으로 표시될 수 있습니다. 이 랩에서는 이러한 이름이 동일한 확장 환경을 가리키는 것으로 간주합니다.

1. 사이드바에서 **Foundry Toolkit** 아이콘을 선택하고, 메시지가 표시되면 Azure 계정에 로그인합니다.
   
    > **Note**: Foundry Toolkit 확장으로 로그인할 수 없는 경우 Azure 확장을 선택해야 할 수 있습니다. 그곳에서 로그인한 다음 Foundry Toolkit으로 돌아가 리소스에 액세스합니다.

1. **Microsoft Foundry Resources** 아래에서 **Set Default Project**를 선택하고 앞에서 만든 프로젝트를 선택합니다.
1. 프로젝트 섹션을 확장합니다. **Prompt Agents** 아래에서 `product-expert-agent` 에이전트를 선택하여 **Agent Builder** 창을 엽니다.
1. **Tools** 섹션에는 고유 ID가 뒤따르는 `kb-knowledgebase` 접두사로 이름이 지정된 도구(예: `kb-knowledgebase677-7w5fj`)가 이미 표시되어 있어야 합니다. 이것이 Foundry IQ 기술 자료 도구이며, 포털에서 Foundry IQ를 연결할 때 자동으로 추가되었습니다.

    > **Note**: 에이전트에는 둘 이상의 도구가 나열됩니다. Foundry 포털은 새 에이전트에 기본적으로 **Web search** 도구를 추가하며, 독립 실행형 **Azure AI Search** 도구가 표시될 수도 있습니다. 에이전트는 실제로 기술 자료를 검색할 때 `kb-knowledgebase...` 도구를 호출하므로 다른 도구에 승인을 설정해도 효과가 없습니다.

1. `kb-knowledgebase...` 도구에서 줄임표(**...**) 아이콘을 선택한 다음 **Ask for approval for all tools**를 선택하고, 메시지가 표시되면 변경 내용을 저장합니다.

이제 에이전트는 Foundry IQ를 사용하여 기술 자료를 검색할 때마다 승인을 요청합니다. 다음에 완성할 클라이언트 앱이 이 요청을 처리합니다.

## 앱에서 에이전트에 연결

이제 에이전트와 프로그래밍 방식으로 상호 작용하는 Python 애플리케이션을 만듭니다. 빠르게 시작할 수 있도록 GitHub 리포지토리에 시작 파일이 제공되어 있습니다.

### Visual Studio Code에서 앱 개발 준비

이제 Visual Studio Code를 사용하여 앱을 개발하겠습니다. 앱의 코드 파일은 GitHub 리포지토리에 제공되어 있습니다.

1. Visual Studio Code를 시작하고 명령 팔레트(Shift+Ctrl+P)를 엽니다. 그런 다음 **Git: Clone** 명령을 검색하고 실행하여 `https://github.com/MicrosoftLearning/mslearn-ai-agents` 리포지토리를 로컬 폴더(어느 폴더든 상관없음)에 복제합니다.
1. 리포지토리가 복제되면 Visual Studio Code에서 폴더를 엽니다.

    > **Note**: Visual Studio Code에서 열고 있는 코드를 신뢰할지 묻는 팝업 메시지가 표시되면 **Yes, I trust the authors** 옵션을 클릭하여 계속합니다.

1. 리포지토리의 Python 코드 프로젝트를 지원하기 위한 추가 파일이 설치되는 동안 기다립니다(메시지가 표시되는 경우).

    > **Note**: 빌드 및 디버그에 필요한 자산을 설치하라는 메시지가 표시되면 **Not Now**를 선택합니다.

1. **Explorer** 창에서 **Labfiles/04-integrate-agent-with-foundry-iq/Python** 폴더를 확장합니다.

    제공된 파일에는 애플리케이션 코드, 구성 설정 및 에이전트 클라이언트 시작 코드가 포함되어 있습니다.

### 애플리케이션 설정 구성

1. Visual Studio Code의 **Labfiles/04-integrate-agent-with-foundry-iq/Python** 폴더에서 **.env** 구성 파일을 엽니다.
1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트 엔드포인트(Foundry 포털의 프로젝트 **Home** 페이지에서 복사한 값)로 바꾸고 AGENT_NAME 변수가 에이전트 이름( *product-expert-agent*이어야 함)으로 설정되어 있는지 확인합니다.
1. 자리 표시자를 바꾼 후 파일을 저장합니다.

### 에이전트 클라이언트 코드 완성

> **Tip**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다. 주석 들여쓰기 수준을 기준으로 사용하세요.

1. Visual Studio Code의 **Labfiles/04-integrate-agent-with-foundry-iq/Python** 폴더에서 **agent_client.py** 코드 파일을 엽니다.
1. 다음을 포함하여 제공된 시작 코드를 검토합니다.
    - Import 문 및 구성 로드
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

1. `send_message_to_agent()` 함수 내부의 두 번째 **TODO** 주석을 찾아 다음 코드를 추가하여 MCP 승인 요청을 포함한 메시지 전송 및 응답 처리를 수행합니다.

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

   # Loop until a response has no pending approval requests (zero, one, or many)
   while True:
       approval_requests = [
           item for item in (getattr(response, "output", None) or [])
           if getattr(item, "type", None) == "mcp_approval_request"
       ]

       if not approval_requests:
           break

       approval_items = []
       for approval_request in approval_requests:
           print(f"[Approval required for: {approval_request.name}]\n")
           print(f"Server: {approval_request.server_label}")

           # Show the tool call arguments for transparency
           import json
           try:
               args = json.loads(approval_request.arguments)
               print(f"Arguments: {json.dumps(args, indent=2)}\n")
           except Exception:
               print(f"Arguments: {approval_request.arguments}\n")

           approval_input = input("Approve this action? (yes/no): ").strip().lower()
           approved = approval_input in ['yes', 'y']
           print("Approving action...\n" if approved else "Action denied.\n")

           approval_items.append({
               "type": "mcp_approval_response",
               "approval_request_id": approval_request.id,
               "approve": approved
           })

       # Send the approval decisions and fetch the next response
       openai_client.conversations.items.create(
           conversation_id=conversation.id,
           items=approval_items
       )

       response = openai_client.responses.create(
           conversation=conversation.id,
           extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
           input=""
       )

    ```

    > **Note**: 에이전트가 항상 승인을 요청하는 것은 아니며, 같은 턴에서 둘 이상의 도구 호출에 대해 승인을 요청하는 경우도 있습니다. `approval_requests`가 비어 있을 때까지 반복하면 두 경우를 모두 올바르게 처리할 수 있습니다.

1. 코드를 추가한 후 파일을 저장합니다.

1. 이제 코드가 conversations API를 사용하여 에이전트와의 상호 작용을 관리한다는 점을 검토합니다. 여기서는 다음이 수행됩니다.
    - 대화가 만들어지고 ID로 추적됩니다.
    - `conversations.items.create()`를 사용하여 사용자 메시지가 대화에 추가됩니다.
    - 에이전트 참조와 함께 `responses.create()`를 사용하여 응답이 생성됩니다.
    - **MCP approval handling**: 에이전트가 Foundry IQ에 액세스해야 할 때 응답 출력에 하나 이상의 `mcp_approval_request` 항목을 반환하여 승인을 요청합니다.
    - 코드는 대기 중인 각 요청을 승인하거나 거부하라는 메시지를 표시하면서, 에이전트가 미해결 승인 요청이 없는 응답을 반환할 때까지 반복합니다(처음부터 승인이 필요하지 않았던 경우 포함).
    - 각 승인/거부 후 `mcp_approval_response`가 대화에 추가되고 새 응답이 생성됩니다.
    - 에이전트는 사용자의 승인 결정에 따라 Foundry IQ에서 정보를 검색합니다.

## 통합 테스트

이제 애플리케이션을 실행하고 기술 자료에서 정보를 검색하는 에이전트의 기능을 테스트합니다.

1. Visual Studio Code에서 **Labfiles/04-integrate-agent-with-foundry-iq/Python** 폴더를 마우스 오른쪽 단추로 클릭하고 **Open in Integrated Terminal**을 선택하여 해당 폴더의 통합 터미널을 엽니다.
1. 먼저 가상 환경을 만들고 종속성을 설치합니다.

    ```
   python -m venv labenv
   ./labenv/Scripts/activate
   pip install -r requirements.txt
    ```

1. 터미널 창에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```
   az login
    ```

    > **Note**: 대부분의 시나리오에서는 *az login*만 사용해도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트를 지정해야 할 수 있습니다. 자세한 내용은 [Azure CLI를 사용하여 대화형으로 Azure에 로그인](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)을 참조하세요.

1. 메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드와 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고, 메시지가 표시되면 Foundry 리소스가 포함된 구독을 선택합니다.

1. 터미널 창에서 애플리케이션을 실행합니다.

    ```
   python agent_client.py
    ```

1. 애플리케이션이 시작되면 다음 쿼리로 에이전트를 테스트합니다.

    **쿼리 1 - 제품 범주:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What types of outdoor products does Contoso offer?
    ```

    ```prompt
   Contoso는 어떤 종류의 야외 제품을 제공하나요?
    ```

    승인을 묻는 메시지가 표시되면 에이전트가 기술 자료를 검색할 수 있도록 **yes**를 입력합니다. 에이전트가 기술 자료의 여러 문서에서 정보를 검색하는 방식을 관찰합니다.

    **쿼리 2 - 특정 제품 세부 정보:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Tell me about the weatherproof features of your tents.
    ```

    ```prompt
   텐트의 방수 및 방풍 기능에 대해 알려주세요.
    ```

    요청을 승인하고 에이전트가 텐트 카탈로그에서 구체적인 세부 정보를 제공하는 방식을 확인합니다.

    **쿼리 3 - 제품 비교:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What's the difference between your daypacks and expedition backpacks?
    ```

    ```prompt
   데이팩과 원정용 배낭의 차이점은 무엇인가요?
    ```

    요청을 승인하고 에이전트가 배낭 가이드의 정보를 종합하는 방식을 확인합니다.

    **쿼리 4 - 액세서리 및 추가 제품:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What camping accessories would you recommend for a weekend hiking trip?
    ```

    ```prompt
   주말 하이킹 여행에 어떤 캠핑 액세서리를 추천하시나요?
    ```

    요청을 승인하고 에이전트가 기술 자료를 바탕으로 추천을 제공하는 기능을 관찰합니다.

    **쿼리 5 - 후속 질문:**

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   How much do those items typically cost?
    ```

    ```prompt
   해당 품목들은 보통 가격이 얼마인가요?
    ```

    에이전트가 이전 쿼리의 대화 컨텍스트를 유지하는 방식을 확인합니다.

1. 전체 대화 기록을 보려면 `history`를 입력합니다.

1. 테스트를 마치면 `quit`를 입력합니다.

### 결과 검토

에이전트 응답의 다음 측면을 고려합니다.

- **MCP Approval Flow**: 에이전트가 기술 자료에 액세스해야 할 때마다 승인을 요청하므로 외부 도구 사용을 제어할 수 있습니다.
- **Accuracy**: 에이전트가 기술 자료 문서에서 직접 정보를 제공합니다.
- **Citations**: 에이전트가 원본 참조 또는 문서 ID를 포함할 수 있습니다.
- **Context awareness**: 에이전트가 대화의 이전 메시지를 기억합니다.
- **Grounding**: 에이전트가 기술 자료에서 관련 정보를 찾을 수 없을 때 이를 표시합니다.
- **Error handling**: 애플리케이션이 오류와 연결 문제를 원활하게 처리합니다.

## 요약

이 연습에서는 다음을 수행했습니다.

- 새 Foundry UI로 Foundry 프로젝트 및 에이전트를 만들었습니다.
- 제품 정보 문서로 기술 자료를 빌드했습니다.
- Foundry IQ가 사용되도록 포털에서 에이전트를 구성했습니다.
- Python SDK를 사용하여 Visual Studio Code에서 에이전트에 연결했습니다.
- MCP 승인 처리, 대화 기록 및 오류 처리가 포함된 클라이언트 애플리케이션을 구현했습니다.
- 외부 도구 액세스에 대한 사용자 제어 승인을 통해 기술 자료에서 정보를 검색하고 종합하는 에이전트의 기능을 테스트했습니다.

이는 대화 컨텍스트를 유지하면서 엔터프라이즈 기술 자료에서 정보를 검색하고 가져올 수 있는 지능형 애플리케이션을 만들기 위해 AI 에이전트를 Foundry IQ와 통합하는 방법을 보여 줍니다.

## 정리

Azure AI Agent Service 및 Foundry IQ 탐색을 마쳤다면 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. 웹 브라우저에서 [Azure 포털](https://portal.azure.com)을 `https://portal.azure.com`에서 엽니다.
1. Foundry 리소스와 AI Search 리소스가 포함된 리소스 그룹으로 이동합니다.
1. 도구 모음에서 **Delete resource group**을 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
