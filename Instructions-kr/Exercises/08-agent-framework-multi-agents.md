---
lab:
    title: 'Microsoft Agent Framework로 다중 에이전트 솔루션 개발'
    description: 'Microsoft Agent Framework SDK를 사용하여 여러 에이전트가 협업하도록 구성하는 방법을 알아봅니다.'
    level: 300
    duration: 30
    islab: true
    status: 'released'
---

# Microsoft Agent Framework로 다중 에이전트 솔루션 개발

이 연습에서는 Microsoft Agent Framework SDK의 순차 오케스트레이션 패턴 사용을 연습합니다. 함께 작업하여 고객 피드백을 처리하고 다음 단계를 제안하는 세 에이전트의 간단한 파이프라인을 만듭니다. 다음 에이전트를 만들게 됩니다.

- Summarizer 에이전트는 원시 피드백을 짧고 중립적인 문장으로 요약합니다.
- Classifier 에이전트는 피드백을 Positive, Negative 또는 Feature request로 분류합니다.
- 마지막으로 Recommended Action 에이전트는 적절한 후속 단계를 권장합니다.

Microsoft Agent Framework SDK를 사용하여 문제를 나누고, 적절한 에이전트로 라우팅하고, 실행 가능한 결과를 생성하는 방법을 알아봅니다. 시작해 보겠습니다.

이 연습을 완료하는 데 약 **30**분이 걸립니다.

> **Note**: 이 연습에서 사용하는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

이 연습을 시작하기 전에 다음 항목이 있는지 확인합니다.

- 로컬 컴퓨터에 [Visual Studio Code](https://code.visualstudio.com/)가 설치되어 있어야 합니다.
- 활성 [Azure 구독](https://azure.microsoft.com/free/)이 있어야 합니다.
- [Python 3.13](https://www.python.org/downloads/)이 설치되어 있어야 합니다.
- 로컬 컴퓨터에 [Git](https://git-scm.com/downloads)이 설치되어 있어야 합니다.

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에는 3.14 빌드가 없습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

## Foundry Toolkit for VS Code 확장을 사용하여 Foundry 프로젝트 만들기

개발자는 Foundry 포털에서 작업하는 시간이 있을 수 있지만, Visual Studio Code에서 많은 시간을 보낼 가능성도 높습니다. Foundry Toolkit for VS Code 확장은 개발 환경을 벗어나지 않고 Foundry 프로젝트 리소스로 작업할 수 있는 편리한 방법을 제공합니다.

1. Visual Studio Code를 엽니다.

2. 왼쪽 창에서 **Extensions**를 선택합니다(또는 **Ctrl+Shift+X**를 누릅니다).

3. 확장 마켓플레이스에서 Microsoft의 `Foundry Toolkit` 확장을 검색하고 **Install**을 선택합니다.

    > **Note**: 이 확장은 현재 **Foundry Toolkit**으로 표시되지만, 일부 VS Code 레이블, 명령 또는 이전 스크린샷에서는 여전히 **AI Toolkit**으로 표시될 수 있습니다. 이 랩에서는 이러한 이름이 동일한 확장 환경을 가리키는 것으로 간주합니다.

4. 확장을 설치한 후 사이드바에서 해당 아이콘을 선택하여 Foundry Toolkit 보기를 엽니다.

    아직 로그인하지 않았다면 Azure 계정에 로그인하라는 메시지가 표시됩니다.

5. **Microsoft Foundry Resources** 아래에서 **Create Project**를 선택합니다.

    기본 프로젝트가 이미 활성 상태이면 프로젝트 이름이 **My Resources** 아래에 표시됩니다. 활성 프로젝트를 마우스 오른쪽 단추로 클릭하고 **Switch Default Project in Azure Extension**을 선택하여 새 프로젝트를 만들 수 있습니다.

6. Azure 구독과 리소스 그룹을 선택한 다음, 이 연습에 사용할 새 프로젝트를 만들기 위해 Foundry 프로젝트 이름을 입력합니다.

    배포가 완료되면 Foundry Toolkit 창에 프로젝트가 기본 프로젝트로 표시됩니다.

## 모델 배포

모든 생성형 AI 프로젝트의 중심에는 하나 이상의 생성형 AI 모델이 있습니다. 이 작업에서는 에이전트에서 사용할 모델을 Model Catalog에서 배포합니다.

1. "Project deployed successfully" 팝업이 나타나면 **Deploy a new model** 단추를 선택합니다. 그러면 Model Catalog가 열립니다.

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

이 연습에서는 Foundry 프로젝트에 연결하고 고객 피드백을 처리할 수 있는 다중 에이전트 솔루션을 만드는 데 도움이 되는 스타터 코드를 사용합니다. GitHub 리포지토리에서 이 코드를 복제합니다.

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P** 또는 **View > Command Palette**)를 엽니다.

1. **Git: Clone**을 입력하고 목록에서 선택합니다.

1. 리포지토리 URL을 입력합니다.

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. 리포지토리를 복제할 로컬 컴퓨터의 위치를 선택합니다.

1. 메시지가 표시되면 **Open**을 선택하여 복제한 리포지토리를 VS Code에서 엽니다.

1. 리포지토리가 열리면 **File > Open Folder**를 선택하고 `mslearn-ai-agents/Labfiles/08-agent-orchestration`으로 이동한 다음 **Select Folder**를 선택합니다.

1. Explorer 창에서 **Python** 폴더를 확장하여 이 연습의 코드 파일을 봅니다.

1. **requirements.txt** 파일을 마우스 오른쪽 단추로 클릭하고 **Open in Integrated Terminal**을 선택합니다.

1. 터미널에서 다음 명령을 입력하여 필요한 Python 패키지를 가상 환경에 설치합니다.

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. **.env** 파일을 열고 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Foundry Toolkit 확장에서 프로젝트 배포 리소스에서 복사한 값)로 바꾸고 MODEL_DEPLOYMENT_NAME 변수가 모델 배포 이름으로 설정되어 있는지 확인합니다. 이러한 변경을 한 후 **Ctrl+S**를 사용하여 파일을 저장합니다.

## AI 에이전트 만들기

이제 다중 에이전트 솔루션을 위한 에이전트를 만들 준비가 되었습니다. 시작해 보겠습니다.

1. 코드 편집기에서 **agents.py** 파일을 엽니다.

1. 파일 맨 위의 **Add references** 주석 아래에, 에이전트를 구현하는 데 필요한 라이브러리의 네임스페이스를 참조하도록 다음 코드를 추가합니다.

    ```python
   # Add references
   from agent_framework import Message
   from agent_framework.foundry import FoundryChatClient
   from agent_framework.orchestrations import SequentialBuilder
   from azure.identity import AzureCliCredential
    ```

1. **main** 함수에서 에이전트 지침을 잠시 검토합니다. 이러한 지침은 오케스트레이션에서 각 에이전트의 동작을 정의합니다.

1. **Create the chat client** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Create the chat client
   credential = AzureCliCredential()
   chat_client = FoundryChatClient(
       credential=credential,
       project_endpoint=os.getenv("AZURE_AI_PROJECT_ENDPOINT"),
       model=os.getenv("AZURE_AI_MODEL_DEPLOYMENT_NAME"),
   )
    ```

    **AzureCliCredential** 개체를 사용하면 코드가 Azure 계정에 인증할 수 있습니다. **FoundryChatClient** 개체는 .env 구성의 엔드포인트와 모델 배포 이름을 사용하여 Foundry 프로젝트에 연결합니다.

1. **Create agents** 주석 아래에 다음 코드를 추가합니다.

    (들여쓰기 수준을 유지해야 합니다.)

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

## 순차 오케스트레이션 만들기

1. **main** 함수에서 **Initialize the current feedback** 주석을 찾고 다음 코드를 추가합니다.

    (들여쓰기 수준을 유지해야 합니다.)

    ```python
   # Initialize the current feedback
   feedback="""
   I use the dashboard every day to monitor metrics, and it works well overall. 
   But when I'm working late at night, the bright screen is really harsh on my eyes. 
   If you added a dark mode option, it would make the experience much more comfortable.
   """
    ```

1. **Build a sequential orchestration** 주석 아래에 다음 코드를 추가하여 정의한 에이전트로 순차 오케스트레이션을 정의합니다.

    ```python
   # Build sequential orchestration
   workflow = SequentialBuilder(
       participants=[summarizer_agent, classifier_agent, action_agent],
       output_from="all",
   ).build()
    ```

    에이전트는 오케스트레이션에 추가된 순서대로 피드백을 처리합니다. `output_from="all"` 매개 변수는 모든 에이전트의 출력이 수집되도록 합니다.

1. **Run and collect outputs** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Run and collect outputs
   result = await workflow.run(f"Customer feedback: {feedback}")
   outputs = result.get_outputs()
    ```

    이 코드는 오케스트레이션을 실행하고 참여 에이전트 각각의 출력을 수집합니다.

1. **Display outputs** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Display outputs
   i = 1
   for response in outputs:
       for msg in cast(list[Message], response.messages):
           name = msg.author_name or ("assistant" if msg.role == "assistant" else "user")
           print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
           i += 1
    ```

    이 코드는 오케스트레이션에서 수집한 워크플로 출력의 메시지를 형식화하여 표시합니다.

1. **CTRL+S** 명령을 사용하여 코드 파일의 변경 내용을 저장합니다.

## 애플리케이션 테스트

이제 코드를 실행하고 AI 에이전트가 협업하는 모습을 볼 준비가 되었습니다.

1. 통합 터미널에서 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
   az login
    ```

    ```
   python agents.py
    ```

1. 다음과 유사한 출력이 표시됩니다.

    ```output
   User requests a dark mode option for more comfortable nighttime use.
   Feature request
   Log as enhancement request to add dark mode for improved user comfort during nighttime use.
   ------------------------------------------------------------
   01 [summarizer]
   User requests a dark mode option for more comfortable nighttime use.
   ------------------------------------------------------------
   02 [classifier]
   Feature request
   ------------------------------------------------------------
   03 [action]
   Log as enhancement request to add dark mode for improved user comfort during nighttime use.
    ```

1. 선택적으로 다음과 같은 다른 피드백 입력을 사용하여 코드를 실행해 볼 수 있습니다.

    ```output
   I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
    ```

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
