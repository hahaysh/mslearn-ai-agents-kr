---
lab:
    title: 'A2A 프로토콜로 원격 에이전트에 연결'
    description: 'A2A 프로토콜을 사용하여 원격 에이전트와 협업합니다.'
    level: 300
    duration: 30
    islab: true
    status: 'released'
---

# A2A 프로토콜로 원격 에이전트에 연결

이 연습에서는 Azure AI Agent Service와 A2A 프로토콜을 사용하여 서로 상호 작용하는 간단한 원격 에이전트를 만듭니다. 이러한 에이전트는 기술 작성자가 개발자 블로그 게시물을 준비하는 데 도움을 줍니다. 제목 에이전트는 헤드라인을 생성하고, 개요 에이전트는 제목을 사용하여 문서의 간결한 개요를 작성합니다. 시작해 보겠습니다.

> **Tip**: 이 연습에서 사용하는 코드는 Python용 Microsoft Foundry SDK를 기반으로 합니다. Microsoft .NET, JavaScript 및 Java용 SDK를 사용하여 유사한 솔루션을 개발할 수 있습니다. 자세한 내용은 [Microsoft Foundry SDK 클라이언트 라이브러리](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)를 참조하세요.

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

이 연습에서는 Foundry 프로젝트에 연결하고 경비 데이터를 처리할 수 있는 에이전트를 만드는 데 도움이 되는 스타터 코드를 사용합니다. GitHub 리포지토리에서 이 코드를 복제합니다.

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P** 또는 **View > Command Palette**)를 엽니다.

1. **Git: Clone**을 입력하고 목록에서 선택합니다.

1. 리포지토리 URL을 입력합니다.

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. 리포지토리를 복제할 로컬 컴퓨터의 위치를 선택합니다.

1. 메시지가 표시되면 **Open**을 선택하여 복제한 리포지토리를 VS Code에서 엽니다.

1. 리포지토리가 열리면 **File > Open Folder**를 선택하고 `mslearn-ai-agents/Labfiles/09-build-remote-agents-with-a2a`로 이동한 다음 **Select Folder**를 선택합니다.

1. Explorer 창에서 **Python** 폴더를 확장하여 이 연습의 코드 파일을 봅니다.

1. Explorer 보기에서 **Labfiles/09-build-remote-agents-with-a2a/Python** 폴더로 이동하여 이 연습의 스타터 코드를 찾습니다.

    제공된 파일은 다음과 같습니다.

    ```output
   python
   ├── outline_agent/
   │   ├── agent.py
   │   ├── agent_executor.py
   │   └── server.py
   ├── routing_agent/
   │   ├── agent.py
   │   └── server.py
   ├── title_agent/
   │   ├── agent.py
   |   ├── agent_executor.py
   │   └── server.py
   ├── client.py
   └── run_all.py
    ```

    각 에이전트 폴더에는 Azure AI 에이전트 코드와 에이전트를 호스트하는 서버가 포함되어 있습니다. **routing agent**는 **title** 및 **outline** 에이전트를 검색하고 통신하는 역할을 담당합니다. **client**를 통해 사용자는 라우팅 에이전트에 프롬프트를 제출할 수 있습니다. `run_all.py`는 모든 서버를 시작하고 클라이언트를 실행합니다.

1. **requirements.txt** 파일을 마우스 오른쪽 단추로 클릭하고 **Open in Integrated Terminal**을 선택합니다.

1. 터미널에서 다음 명령을 입력하여 필요한 Python 패키지를 가상 환경에 설치합니다.

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. **.env** 파일을 열고 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Foundry Toolkit 확장에서 프로젝트 배포 리소스에서 복사한 값)로 바꾸고 MODEL_DEPLOYMENT_NAME 변수가 모델 배포 이름으로 설정되어 있는지 확인합니다. 이러한 변경을 한 후 **Ctrl+S**를 사용하여 파일을 저장합니다.

## 검색 가능한 에이전트 만들기

이 작업에서는 작성자가 문서의 트렌디한 헤드라인을 만들 수 있도록 돕는 제목 에이전트를 만듭니다. 또한 A2A 프로토콜에 필요한 에이전트의 기술과 카드를 정의하여 에이전트를 검색 가능하게 만듭니다.

> **Tip**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다. 기존 주석을 가이드로 사용하여 새 코드를 같은 들여쓰기 수준에 입력합니다.

1. 코드 편집기에서 **title_agent/agent.py** 파일을 엽니다.

1. **Create the agents client** 주석을 찾고 다음 코드를 추가하여 Azure AI 프로젝트에 연결합니다.

    > **Tip**: 올바른 들여쓰기 수준을 유지하도록 주의합니다.

    ```python
   # Create the agents client
   self.client = AgentsClient(
       endpoint=os.environ['PROJECT_ENDPOINT'],
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       )
   )
    ```

1. **Create the title agent** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Create the title agent
   self.agent = self.client.create_agent(
       model=os.environ['MODEL_DEPLOYMENT_NAME'],
       name='title-agent',
       instructions="""
       You are a helpful writing assistant.
       Given a topic the user wants to write about, suggest a single clear and catchy blog post title.
       """,
   )
    ```

1. **Create a thread for the chat session** 주석을 찾고 다음 코드를 추가하여 채팅 스레드를 만듭니다.

    ```python
   # Create a thread for the chat session
   thread = self.client.threads.create()
    ```

1. **Send user message** 주석을 찾고 다음 코드를 추가하여 사용자의 프롬프트를 제출합니다.

    ```python
   # Send user message
   self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. **Create and run the agent** 주석 아래에 다음 코드를 추가하여 에이전트의 응답 생성을 시작합니다.

    ```python
   # Create and run the agent
   run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    파일의 나머지 부분에 제공된 코드는 에이전트의 응답을 처리하고 반환합니다.

1. 코드 파일을 저장합니다(*CTRL+S*). 이제 A2A 프로토콜과 에이전트의 기술 및 카드를 공유할 준비가 되었습니다.

1. 코드 편집기에서 **title_agent/server.py** 파일을 엽니다.

1. **Define agent skills** 주석을 찾고 다음 코드를 추가하여 에이전트의 기능을 지정합니다.

    ```python
   # Define agent skills
   skills = [
       AgentSkill(
           id='generate_blog_title',
           name='Generate Blog Title',
           description='Generates a blog title based on a topic',
           tags=['title'],
           examples=[
               'Can you give me a title for this article?',
           ],
       ),
   ]
    ```

1. **Create agent card** 주석을 찾고 다음 코드를 추가하여 에이전트를 검색 가능하게 만드는 메타데이터를 정의합니다.

    ```python
   # Create agent card
   agent_card = AgentCard(
       name='Microsoft Foundry Title Agent',
       description='An intelligent title generator agent powered by Foundry. '
       'I can help you generate catchy titles for your articles.',
       url=f'http://{host}:{port}/',
       version='1.0.0',
       default_input_modes=['text'],
       default_output_modes=['text'],
       capabilities=AgentCapabilities(),
       skills=skills,
   )
    ```

1. **Create agent executor** 주석을 찾고 다음 코드를 추가하여 에이전트 카드를 사용해 에이전트 실행기를 초기화합니다.

    ```python
   # Create agent executor
   agent_executor = create_foundry_agent_executor(agent_card)
    ```

    에이전트 실행기는 만든 제목 에이전트의 래퍼 역할을 합니다.

1. **Create request handler** 주석을 찾고 다음 코드를 추가하여 실행기를 사용해 들어오는 요청을 처리합니다.

    ```python
   # Create request handler
   request_handler = DefaultRequestHandler(
       agent_executor=agent_executor, task_store=InMemoryTaskStore()
   )
    ```

1. **Create A2A application** 주석 아래에 다음 코드를 추가하여 A2A 호환 애플리케이션 인스턴스를 만듭니다.

    ```python
   # Create A2A application
   a2a_app = A2AStarletteApplication(
       agent_card=agent_card, http_handler=request_handler
   )
    ```

    이 코드는 제목 에이전트의 정보를 공유하고 제목 에이전트 실행기를 사용하여 이 에이전트에 대한 수신 요청을 처리하는 A2A 서버를 만듭니다.

1. 완료되면 코드 파일을 저장합니다(*CTRL+S*).

## 에이전트 간 메시지 사용 설정

이 작업에서는 A2A 프로토콜을 사용하여 라우팅 에이전트가 다른 에이전트로 메시지를 보낼 수 있도록 합니다. 또한 에이전트 실행기 클래스를 구현하여 제목 에이전트가 메시지를 받을 수 있도록 합니다.

1. 코드 편집기에서 **routing_agent/agent.py** 파일을 엽니다.

    라우팅 에이전트는 사용자 메시지를 처리하고 어떤 원격 에이전트가 요청을 처리해야 하는지 결정하는 오케스트레이터 역할을 합니다.

    사용자 메시지가 수신되면 라우팅 에이전트는 다음을 수행합니다.
    - 대화 스레드를 시작합니다.
    - `create_and_process` 메서드를 사용하여 사용자의 메시지에 가장 잘 맞는 에이전트를 평가합니다.
    - 메시지는 `send_message` 함수를 사용하여 HTTP를 통해 적절한 에이전트로 라우팅됩니다.
    - 원격 에이전트가 메시지를 처리하고 응답을 반환합니다.

    마지막으로 라우팅 에이전트는 응답을 캡처하고 스레드를 통해 사용자에게 반환합니다.

    `send_message` 메서드는 async이며 에이전트 실행이 성공적으로 완료되도록 await해야 합니다.

1. **Retrieve the remote agent's A2A client using the agent name** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Retrieve the remote agent's A2A client using the agent name 
   client = self.remote_agent_connections[agent_name]
    ```

1. **Construct the payload to send to the remote agent** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Construct the payload to send to the remote agent
   payload: dict[str, Any] = {
       'message': {
           'role': 'user',
           'parts': [{'kind': 'text', 'text': task}],
           'messageId': message_id,
       },
   }
    ```

1. **Wrap the payload in a SendMessageRequest object** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Wrap the payload in a SendMessageRequest object
   message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. **Send the message to the remote agent client and await the response** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Send the message to the remote agent client and await the response
   send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```

1. 완료되면 코드 파일을 저장합니다(*CTRL+S*). 이제 라우팅 에이전트가 제목 에이전트를 검색하고 메시지를 보낼 수 있습니다. 라우팅 에이전트에서 들어오는 메시지를 처리하도록 에이전트 실행기 코드를 만들어 보겠습니다.

1. 코드 편집기에서 **title_agent/agent_executor.py** 파일을 엽니다.

    `AgentExecutor` 클래스 구현에는 `execute` 및 `cancel` 메서드가 포함되어야 합니다. cancel 메서드는 이미 제공되어 있습니다. `execute` 메서드에는 이벤트를 관리하고 작업이 완료되면 호출자에게 신호를 보내는 `TaskUpdater` 개체가 포함됩니다. 작업 실행 논리를 추가해 보겠습니다.

1. `execute` 메서드에서 **Process the request** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Process the request
   await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. `_process_request` 메서드에서 **Get the title agent** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Get the title agent
   agent = await self._get_or_create_agent()
    ```

1. **Update the task status** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Update the task status
   await task_updater.update_status(
       TaskState.working,
       message=new_agent_text_message('Title Agent is processing your request...', context_id=context_id),
   )
    ```

1. **Run the agent conversation** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Run the agent conversation
   responses = await agent.run_conversation(user_message)
    ```

1. **Update the task with the responses** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Update the task with the responses
   for response in responses:
       await task_updater.update_status(
           TaskState.working,
           message=new_agent_text_message(response, context_id=context_id),
       )
    ```

1. **Mark the task as complete** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Mark the task as complete
   final_message = responses[-1] if responses else 'Task completed.'
   await task_updater.complete(
       message=new_agent_text_message(final_message, context_id=context_id)
   )
    ```

    이제 제목 에이전트는 A2A 프로토콜이 메시지를 처리하는 데 사용할 에이전트 실행기로 래핑되었습니다. 잘하셨습니다!

## 애플리케이션 테스트

1. 통합 터미널에서 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
   az login
    ```

    ```
   python run_all.py
    ```

    애플리케이션은 인증된 Azure 세션의 자격 증명을 사용하여 프로젝트에 연결하고 에이전트를 만들고 실행합니다. 각 서버가 시작될 때 일부 출력이 표시됩니다.

1. 입력 프롬프트가 나타날 때까지 기다린 다음 다음과 같은 프롬프트를 입력합니다.

    ```
   Create a title and outline for an article about React programming.
    ```

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```prompt
   React 프로그래밍에 관한 문서의 제목과 개요를 만듭니다.
    ```

    몇 분 후 에이전트의 결과 응답이 표시됩니다.

1. 프로그램을 종료하고 서버를 중지하려면 `quit`을 입력합니다.

    터미널에서 Python 가상 환경을 종료하려면 `deactivate`도 사용할 수 있습니다.

## 정리

Azure AI Agent Service 탐색을 마쳤으면 불필요한 Azure 비용이 발생하지 않도록 이 연습에서 만든 리소스를 삭제해야 합니다.

1. Azure portal이 포함된 브라우저 탭으로 돌아가거나, 새 브라우저 탭에서 `https://portal.azure.com`의 [Azure portal](https://portal.azure.com)을 다시 열고 이 연습에서 사용한 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
1. 도구 모음에서 **Delete resource group**을 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
