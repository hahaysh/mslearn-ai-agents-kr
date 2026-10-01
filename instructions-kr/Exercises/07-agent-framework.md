---
lab:
    title: 'Microsoft Agent Framework SDK로 Azure AI 에이전트 개발'
    description: 'Microsoft Agent Framework SDK를 사용하여 Azure AI 채팅 에이전트를 만들고 사용하는 방법을 알아봅니다.'
    level: 300
    duration: 30
    islab: true
    status: 'released'
---

# Microsoft Agent Framework SDK로 Azure AI 채팅 에이전트 개발

이 연습에서는 Azure AI Agent Service와 Microsoft Agent Framework를 사용하여 경비 청구를 처리하는 AI 에이전트를 만듭니다.

이 연습을 완료하는 데 약 **30**분이 걸립니다.

> **Note**: 이 연습에서 사용하는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

이 연습을 시작하기 전에 다음 항목이 있는지 확인합니다.

- 로컬 컴퓨터에 [Visual Studio Code](https://code.visualstudio.com/)가 설치되어 있어야 합니다.
- 활성 [Azure 구독](https://azure.microsoft.com/free/)이 있어야 합니다.
- [Python 3.13](https://www.python.org/downloads/)이 설치되어 있어야 합니다.
- 로컬 컴퓨터에 [Git](https://git-scm.com/downloads)이 설치되어 있어야 합니다.

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에는 3.14 빌드가 없습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

## Foundry Toolkit VS Code 확장을 사용하여 Foundry 프로젝트 만들기

개발자는 Foundry 포털에서 작업하는 시간이 있을 수 있지만, Visual Studio Code에서 많은 시간을 보낼 가능성도 높습니다. Foundry Toolkit 확장은 개발 환경을 벗어나지 않고 Foundry 프로젝트 리소스로 작업할 수 있는 편리한 방법을 제공합니다.

1. Visual Studio Code를 엽니다.

2. 왼쪽 창에서 **Extensions**를 선택합니다(또는 **Ctrl+Shift+X**를 누릅니다).

3. 확장 마켓플레이스에서 Microsoft의 `Foundry Toolkit` 확장을 검색하고 **Install**을 선택합니다.

    > **Note**: 이 확장은 현재 **Foundry Toolkit**으로 표시되지만, 일부 VS Code 레이블, 명령 또는 이전 스크린샷에서는 여전히 **AI Toolkit**으로 표시될 수 있습니다. 이 랩에서는 이러한 이름이 동일한 확장 환경을 가리키는 것으로 간주합니다.

4. 확장을 설치한 후 사이드바에서 해당 아이콘을 선택하여 Foundry Toolkit 보기를 엽니다.

    아직 로그인하지 않았다면 Azure 계정에 로그인하라는 메시지가 표시됩니다.

5. **Microsoft Foundry Resources** 아래에서 **Create Project**를 선택합니다.

    기본 프로젝트가 이미 활성 상태이면 프로젝트 이름이 **My Resources** 아래에 표시됩니다. 활성 프로젝트를 마우스 오른쪽 단추로 클릭하고 **Switch Default Project**를 선택하여 새 프로젝트를 만들 수 있습니다.

6. Azure 구독과 리소스 그룹을 선택한 다음, 이 연습에 사용할 새 프로젝트를 만들기 위해 Foundry 프로젝트 이름을 입력합니다.

    배포가 완료되면 Foundry Toolkit 창에 프로젝트가 기본 프로젝트로 표시됩니다.

## 모델 배포

모든 생성형 AI 프로젝트의 중심에는 하나 이상의 생성형 AI 모델이 있습니다. 이 작업에서는 에이전트에서 사용할 모델을 Model Catalog에서 배포합니다.

1. "Project deployed successfully" 팝업이 나타나면 **Deploy a model** 단추를 선택합니다. 그러면 Model Catalog가 열립니다.

   > **Tip**: Resources 섹션의 **Models** 옆에 있는 **+** 아이콘을 선택하거나 **F1**을 누르고 **Foundry Toolkit: Show model catalog** 명령을 실행하여 Model Catalog에 액세스할 수도 있습니다.

1. Model Catalog에서 **gpt-5** 모델을 찾습니다(검색 창을 사용하면 빠르게 찾을 수 있습니다).

1. gpt-5 모델 옆의 **Deploy**를 선택합니다.

1. 배포 설정을 구성합니다.
   - **Deployment name**: "gpt-5"와 같은 이름을 입력합니다.
   - **Deployment type**: **Global Standard**를 선택합니다(Global Standard를 사용할 수 없는 경우 **Standard** 선택).
   - **Model version**: 기본값으로 둡니다.
   - **Tokens per minute**: 기본값으로 둡니다.

1. 왼쪽 아래 모서리에서 **Deploy to Microsoft Foundry**를 선택합니다.

1. 배포가 완료될 때까지 기다립니다. 배포된 모델은 Resources 보기의 **Models** 섹션 아래에 표시됩니다.

1. 프로젝트 배포의 이름을 마우스 오른쪽 단추로 클릭하고 **Copy Project Endpoint**를 선택합니다. 다음 단계에서 에이전트를 Foundry 프로젝트에 연결하려면 이 URL이 필요합니다.

    ![Foundry Toolkit VS Code 확장에서 프로젝트 엔드포인트를 복사하는 스크린샷.](../Media/vs-code-endpoint.png)

## 스타터 코드 리포지토리 복제

이 연습에서는 Foundry 프로젝트에 연결하고 경비 데이터를 처리할 수 있는 에이전트를 만드는 데 도움이 되는 스타터 코드를 사용합니다. GitHub 리포지토리에서 이 코드를 복제합니다.

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P** 또는 **View > Command Palette**)를 엽니다.

1. **Git: Clone**을 입력하고 목록에서 선택합니다.

1. 리포지토리 URL을 입력합니다.

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. 리포지토리를 복제할 로컬 컴퓨터의 위치를 선택합니다.

1. 메시지가 표시되면 **Open**을 선택하여 복제한 리포지토리를 VS Code에서 엽니다.

1. 리포지토리가 열리면 **File > Open Folder**를 선택하고 `mslearn-ai-agents/Labfiles/07-agent-framework`로 이동한 다음 **Select Folder**를 선택합니다.

1. Explorer 창에서 **Python** 폴더를 확장하여 이 연습의 코드 파일을 봅니다.

1. **requirements.txt** 파일을 마우스 오른쪽 단추로 클릭하고 **Open in Integrated Terminal**을 선택합니다.

1. 터미널에서 다음 명령을 입력하여 필요한 Python 패키지를 가상 환경에 설치합니다.

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. **.env** 파일을 열고 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Foundry Toolkit 확장에서 프로젝트 배포 리소스에서 복사한 값)로 바꾸고 MODEL_DEPLOYMENT_NAME 변수가 모델 배포 이름으로 설정되어 있는지 확인합니다. 이러한 변경을 한 후 **Ctrl+S**를 사용하여 파일을 저장합니다.

이제 사용자 지정 도구를 사용하여 경비 데이터를 처리하는 AI 에이전트를 만들 준비가 되었습니다.

## 사용자 지정 도구가 있는 에이전트 만들기

> **Tip**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다. 기존 주석을 가이드로 사용하여 새 코드를 같은 들여쓰기 수준에 입력합니다.

1. 코드 편집기에서 **agent-framework.py** 파일을 엽니다.

1. 파일의 코드를 검토합니다. 이 파일에는 다음이 포함되어 있습니다.
    - 자주 사용하는 네임스페이스에 대한 참조를 추가하는 몇 가지 **import** 문
    - 경비 데이터가 포함된 파일을 로드하고 사용자에게 지침을 요청한 다음 호출하는 *main* 함수
    - 에이전트를 만들고 사용하는 코드를 추가해야 하는 **process_expenses_data** 함수

1. 파일 맨 위의 기존 **import** 문 뒤에서 **Add references** 주석을 찾고, 에이전트를 구현하는 데 필요한 라이브러리의 네임스페이스를 참조하도록 다음 코드를 추가합니다.

    ```python
   # Add references
   from agent_framework import tool, Agent
   from agent_framework.foundry import FoundryChatClient
   from azure.identity import AzureCliCredential
   from pydantic import Field
   from typing import Annotated
    ```

1. 파일 아래쪽에서 **Create a tool function for the email functionality** 주석을 찾고, 에이전트가 이메일을 보내는 데 사용할 함수를 정의하도록 다음 코드를 추가합니다(도구는 에이전트에 사용자 지정 기능을 추가하는 방법입니다).

    ```python
   # Create a tool function for the email functionality
   @tool(approval_mode="never_require")
   def submit_claim(
       to: Annotated[str, Field(description="Who to send the email to")],
       subject: Annotated[str, Field(description="The subject of the email.")],
       body: Annotated[str, Field(description="The text body of the email.")]):
           print("\nTo:", to)
           print("Subject:", subject)
           print(body, "\n")
    ```

    > **Note**: 이 함수는 이메일을 콘솔에 출력하여 보내는 동작을 *시뮬레이션*합니다. 실제 애플리케이션에서는 SMTP 서비스 또는 유사한 서비스를 사용하여 실제로 이메일을 보냅니다!

1. **process_expenses_data** 함수에서 **send_email** 코드보다 위로 돌아가 **Create a foundry chat client** 주석을 찾고 다음 코드를 추가합니다.

    (들여쓰기 수준을 유지해야 합니다.)

    ```python
   # Create a foundry chat client 
   client = FoundryChatClient(
       project_endpoint=os.getenv("PROJECT_ENDPOINT"),
       model=os.getenv("MODEL_DEPLOYMENT_NAME"),
       credential=AzureCliCredential()
   )
    ```

    **AzureCliCredential** 개체를 사용하면 코드가 Azure 계정에 인증할 수 있습니다. 이 클라이언트는 Foundry 에이전트 서비스와 상호 작용하는 데 사용됩니다.

2. **Initialize an agent with the tool and instructions** 주석을 찾고 다음 코드를 추가합니다.

    (들여쓰기 수준을 유지해야 합니다.)

    ```python
   # Initialize an agent with the tool and instructions
   async with (
       Agent(
           client=client,
           name="ExpenseClaimAgent",
           instructions="""You are an AI assistant for expense claim submission.
                       At the user's request, create an expense claim and use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                       Then confirm to the user that you've done so. Don't ask for any more information from the user, just use the data provided to create the email.""",
           tools=[submit_claim],
       ) as agent,
   ):
    ```
    이 코드에서는 **Agent** 개체가 클라이언트, 에이전트에 대한 지침, 이메일을 보내기 위해 정의한 도구 함수로 초기화됩니다.

1. **Use the agent to process the expenses data** 주석을 찾고, 에이전트가 실행될 스레드를 만든 다음 채팅 메시지로 호출하도록 다음 코드를 추가합니다.

    (들여쓰기 수준을 유지해야 합니다.)

    ```python
   # Use the agent to process the expenses data
   try:
       # Add the input prompt to a list of messages to be submitted
       prompt_messages = [f"{prompt}: {expenses_data}"]
       # Invoke the agent for the specified thread with the messages
       response = await agent.run(prompt_messages)
       # Display the response
       print(f"\n# Agent:\n{response}")
   except Exception as e:
       # Something went wrong
       print (e)
    ```

1. 주석을 사용하여 각 코드 블록의 동작을 이해하면서 에이전트에 대한 완성된 코드를 검토한 다음 코드 변경 내용을 저장합니다(**CTRL+S**).

## 애플리케이션 테스트

1. 통합 터미널에서 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
   az login
    ```

    ```
   python agent-framework.py
    ```

    `az login`을 사용하면 AzureCliCredential이 Azure 계정에 인증할 수 있습니다.

1. 경비 데이터로 무엇을 할지 묻는 메시지가 표시되면 다음 프롬프트를 입력합니다.

    ```
   Submit an expense claim
    ```

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```prompt
   경비 청구를 제출합니다.
    ```

1. 애플리케이션이 완료되면 출력을 검토합니다. 에이전트는 제공된 데이터를 기반으로 경비 청구 이메일을 작성해야 합니다.

    > **Tip**: 속도 제한을 초과하여 앱이 실패하면 몇 초 기다린 후 다시 시도합니다. 구독에서 사용할 수 있는 할당량이 부족하면 모델이 응답하지 못할 수 있습니다.

1. 완료되면 터미널에 `deactivate`를 입력하여 Python 가상 환경을 종료합니다.

## 정리

Azure AI Agent Service 탐색을 마쳤으면 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

### 모델 삭제

1. VS Code에서 **Azure Resources** 보기를 새로 고칩니다.

1. **Models** 하위 섹션을 확장합니다.

1. 배포된 모델을 마우스 오른쪽 단추로 클릭하고 **Delete**를 선택합니다.

### 리소스 그룹 삭제

1. [Azure portal](https://portal.azure.com)을 엽니다.

1. Microsoft Foundry 리소스가 포함된 리소스 그룹으로 이동합니다.

1. **Delete resource group**을 선택하고 삭제를 확인합니다.
