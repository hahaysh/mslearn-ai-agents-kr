---
lab:
    title: 'AI 에이전트에서 사용자 지정 함수 사용'
    description: '함수를 사용하여 에이전트에 사용자 지정 기능을 추가하는 방법을 알아봅니다.'
    level: 300
    duration: 50
    islab: true
    status: 'released'
---

# AI 에이전트에서 사용자 지정 함수 사용

이 연습에서는 사용자 지정 함수를 도구로 사용하여 작업을 완료할 수 있는 에이전트를 만드는 방법을 살펴봅니다. 에이전트는 천문학 도우미 역할을 하며, 천문 이벤트에 대한 정보를 제공하고 사용자 입력을 기반으로 망원경 대여 비용을 계산할 수 있습니다. 함수 도구를 정의하고 에이전트가 수행하는 함수 호출을 처리하는 논리를 구현합니다.

> **Tip**: 이 연습에서 사용하는 코드는 Python용 Microsoft Foundry SDK를 기반으로 합니다. Microsoft .NET, JavaScript 및 Java용 SDK를 사용하여 유사한 솔루션을 개발할 수 있습니다. 자세한 내용은 [Microsoft Foundry SDK 클라이언트 라이브러리](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)를 참조하세요.

이 연습을 완료하는 데 약 **50**분이 걸립니다.

> **Note**: 이 연습에서 사용하는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

이 연습을 시작하기 전에 다음이 준비되어 있는지 확인합니다.

- 로컬 컴퓨터에 설치된 [Visual Studio Code](https://code.visualstudio.com/)
- 활성 [Azure 구독](https://azure.microsoft.com/free/)
- 설치된 [Python 3.13](https://www.python.org/downloads/)
- 로컬 컴퓨터에 설치된 [Git](https://git-scm.com/downloads)

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에는 3.14 빌드가 없습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

## Foundry Toolkit for VS Code 확장으로 Foundry 프로젝트 만들기

개발자는 Foundry 포털에서 작업하는 시간도 있지만 Visual Studio Code에서 많은 시간을 보낼 가능성이 큽니다. Foundry Toolkit for VS Code 확장은 개발 환경을 벗어나지 않고 Foundry 프로젝트 리소스로 작업할 수 있는 편리한 방법을 제공합니다.

1. Visual Studio Code를 엽니다.

2. 왼쪽 창에서 **확장(Extensions)** 을 선택하거나 **Ctrl+Shift+X**를 누릅니다.

3. 확장 Marketplace에서 Microsoft의 `Foundry Toolkit for VS Code` 확장을 검색하고 **설치(Install)** 를 선택합니다.

    Foundry Toolkit Extension을 설치하면 VS Code에 Foundry Toolkit 확장이 추가됩니다.

    > **Note**: 현재 확장은 **Foundry Toolkit**으로 표시되지만, 일부 VS Code 레이블, 명령 또는 이전 스크린샷에서는 여전히 **AI Toolkit**으로 표시될 수 있습니다. 이 랩에서는 이러한 이름이 동일한 확장 환경을 가리키는 것으로 간주합니다.

4. 확장을 설치한 후 사이드바에서 Foundry Toolkit 아이콘을 선택합니다.

    아직 Azure 계정에 로그인하지 않았다면 로그인하라는 메시지가 표시됩니다.

5. **Microsoft Foundry Resources** 아래에서 **Create Project**를 선택합니다.

    기본 프로젝트가 이미 활성 상태이면 프로젝트 이름이 **My Resources** 아래에 표시됩니다. 활성 프로젝트를 마우스 오른쪽 단추로 클릭하고 **Switch Default Project in Azure Extension**을 선택하여 새 프로젝트를 만들 수 있습니다.

6. Azure 구독과 리소스 그룹을 선택한 다음, 이 연습을 위한 새 프로젝트를 만들 Foundry 프로젝트 이름을 입력합니다.

    배포가 완료되면 Foundry Toolkit 창에 프로젝트가 기본 프로젝트로 표시됩니다.

## 모델 배포

모든 생성형 AI 프로젝트의 핵심에는 하나 이상의 생성형 AI 모델이 있습니다. 이 작업에서는 에이전트와 함께 사용할 모델을 Model Catalog에서 배포합니다.

1. "Project deployed successfully" 팝업이 나타나면 **Deploy a new model** 단추를 선택합니다. 그러면 Model Catalog가 열립니다.

   > **Tip**: Resources 섹션의 **Models** 옆에 있는 **+** 아이콘을 선택하거나 **F1**을 누른 뒤 **Foundry Toolkit: Show model catalog** 명령을 실행하여 Model Catalog에 액세스할 수도 있습니다.

1. Model Catalog에서 **gpt-5** 모델을 찾습니다. 검색 창을 사용하면 빠르게 찾을 수 있습니다.

1. gpt-5 모델 옆의 **Deploy**를 선택합니다.

1. 배포 설정을 구성합니다.
   - **Deployment name**: "gpt-5"와 같은 이름을 입력합니다.
   - **Deployment type**: **Global Standard**를 선택합니다. Global Standard를 사용할 수 없는 경우 **Standard**를 선택합니다.
   - **Model version**: 기본값으로 둡니다.
   - **Tokens per minute**: 기본값으로 둡니다.

1. 왼쪽 아래 모서리에서 **Deploy to Microsoft Foundry**를 선택합니다.

1. 배포가 완료될 때까지 기다립니다. 배포된 모델은 Resources 보기의 **Models** 섹션 아래에 표시됩니다.

1. 프로젝트 배포 이름을 마우스 오른쪽 단추로 클릭하고 **Copy Project Endpoint**를 선택합니다. 다음 단계에서 에이전트를 Foundry 프로젝트에 연결하려면 이 URL이 필요합니다.

    ![Foundry Toolkit VS Code 확장에서 프로젝트 엔드포인트를 복사하는 스크린샷.](../Media/vs-code-endpoint.png)

## 스타터 코드 리포지토리 복제

이 연습에서는 Foundry 프로젝트에 연결하고 사용자 지정 함수 도구를 사용하는 에이전트를 만드는 데 도움이 되는 스타터 코드를 사용합니다.

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P** 또는 **View > Command Palette**)를 엽니다.

1. **Git: Clone**을 입력하고 목록에서 선택합니다.

1. 리포지토리 URL을 입력합니다.

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. 로컬 컴퓨터에서 리포지토리를 복제할 위치를 선택합니다.

1. 메시지가 표시되면 **Open**을 선택하여 복제한 리포지토리를 VS Code에서 엽니다.

1. 리포지토리가 열리면 **File > Open Folder**를 선택하고 `mslearn-ai-agents/Labfiles/02-agent-custom-tools`로 이동한 다음 **Select Folder**를 선택합니다.

1. 탐색기 창에서 **Python** 폴더를 확장하여 이 연습의 코드 파일을 확인합니다.

1. **requirements.txt** 파일을 마우스 오른쪽 단추로 클릭하고 **Open in Integrated Terminal**을 선택합니다.

1. 터미널에서 다음 명령을 입력하여 필요한 Python 패키지를 가상 환경에 설치합니다.

    ```
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

1. **.env** 파일을 열고 **your_project_endpoint** 자리 표시자를 프로젝트 엔드포인트(Foundry Toolkit VS Code 확장의 프로젝트 배포 리소스에서 복사)로 바꾸고 MODEL_DEPLOYMENT_NAME 변수가 모델 배포 이름으로 설정되어 있는지 확인합니다. 이러한 변경을 완료한 후 **Ctrl+S**를 사용하여 파일을 저장합니다.

이제 외부 데이터 원본 및 API에 액세스하기 위해 MCP 서버 도구를 사용하는 AI 에이전트를 만들 준비가 되었습니다.

## 에이전트가 사용할 함수 만들기

1. **functions.py** 파일을 열고 기존 코드를 검토합니다.

    이 파일에는 에이전트의 도구로 사용할 수 있는 여러 함수가 포함되어 있습니다. 함수는 **data** 폴더에 있는 샘플 파일을 사용하여 천문 이벤트 및 위치에 대한 정보를 검색합니다.

1. **Determine the next visible astronomical event for a given location** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Determine the next visible astronomical event for a given location
   def next_visible_event(location: str) -> str:
       """Returns the next visible astronomical event for a location."""
       today = int(datetime.now().strftime("%m%d"))
       loc = location.lower().replace(" ", "_")

       # Retrieve the next event visible from the location, starting with events later this year
       for name, event_type, date, date_str, locs in EVENTS:
           if loc in locs and date >= today:
               return json.dumps({"event": name, "type": event_type, "date": date_str, "visible_from": sorted(locs)})

       return json.dumps({"message": f"No upcoming events found for {location}."})
    ```

    이 함수는 샘플 이벤트 데이터를 확인하여 지정한 위치에서 볼 수 있는 다음 천문 이벤트를 찾고, 이벤트 세부 정보를 JSON 문자열로 반환합니다. 다음으로 이 함수를 사용할 수 있는 에이전트를 만들어 보겠습니다.

## Foundry 프로젝트에 연결

1. **agent.py** 파일을 엽니다.

   > **Tip**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다. 주석의 들여쓰기 수준을 기준으로 사용합니다.

1. **Add references** 주석을 찾고 함수 도구를 사용하는 Azure AI 에이전트를 빌드하는 데 필요한 클래스를 가져오도록 다음 코드를 추가합니다.

    ```python
   # Add references
   from azure.ai.projects import AIProjectClient
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects.models import PromptAgentDefinition, FunctionTool
   from openai.types.responses.response_input_param import FunctionCallOutput, ResponseInputParam
   from functions import next_visible_event, calculate_observation_cost, generate_observation_report
    ```

    **functions.py** 파일에서 정의한 함수가 에이전트의 도구로 사용될 수 있도록 가져와진다는 점에 주목합니다.

1. **Connect to the project client** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Connect to the project client
   with (
       DefaultAzureCredential() as credential,
       AIProjectClient(endpoint=project_endpoint, credential=credential) as project_client,
       project_client.get_openai_client() as openai_client,
   ):
    ```

## 함수 도구 정의

이 작업에서는 에이전트가 사용할 수 있는 각 함수 도구를 정의합니다. 각 함수 도구의 매개 변수는 JSON 스키마를 사용하여 정의되며, 이 스키마는 함수의 각 매개 변수에 대한 이름, 형식, 설명 및 기타 특성을 지정합니다.

1. **Define the event function tool** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Define the event function tool
   event_tool = FunctionTool(
       name="next_visible_event",
       description="Get the next visible event in a given location.",
       parameters={
           "type": "object",
           "properties": {
               "location": {
                   "type": "string",
                   "description": "continent to find the next visible event in (e.g. 'north_america', 'south_america', 'australia')",
               },
           },
           "required": ["location"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

1. **Define the observation cost function tool** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Define the observation cost function tool
   cost_tool = FunctionTool(
       name="calculate_observation_cost",
       description="Calculate the cost of an observation based on the telescope tier, number of hours, and priority level.",
       parameters={
           "type": "object",
           "properties": {
               "telescope_tier": {
                   "type": "string",
                   "description": "the tier of the telescope (e.g. 'standard', 'advanced', 'premium')",
               },
               "hours": {
                   "type": "number",
                   "description": "the number of hours for the observation",
               },
               "priority": {
                   "type": "string",
                   "description": "the priority level of the observation (e.g. 'low', 'normal', 'high')",
               },
           },
           "required": ["telescope_tier", "hours", "priority"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

1. **Define the observation report generation function tool** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Define the observation report generation function tool
   report_tool = FunctionTool(
       name="generate_observation_report",
       description="Generate a report summarizing an astronomical observation",
       parameters={
           "type": "object",
           "properties": {
               "event_name": {
                   "type": "string",
                   "description": "the name of the astronomical event being observed",
               },
               "location": {
                   "type": "string",
                   "description": "the location of the observer",
               },
               "telescope_tier": {
                   "type": "string",
                   "description": "the tier of the telescope used for the observation (e.g. 'standard', 'advanced', 'premium')",
               },
               "hours": {
                   "type": "number",
                   "description": "the number of hours the telescope was used for the observation",
               },
               "priority": {
                   "type": "string",
                   "description": "the priority level of the observation (e.g. 'low', 'normal', 'high')",
               },
               "observer_name": {
                   "type": "string",
                   "description": "the name of the person who conducted the observation",
               },                   
           },
           "required": ["event_name", "location", "telescope_tier", "hours", "priority", "observer_name"],
           "additionalProperties": False,
       },
       strict=True,
   )
    ```

## 함수 도구를 사용하는 에이전트 만들기

이제 함수 도구를 정의했으므로 해당 도구를 사용하여 작업을 완료할 수 있는 에이전트를 만들 수 있습니다.

1. **Create a new agent with the function tools** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Create a new agent with the function tools
   agent = project_client.agents.create_version(
       agent_name="astronomy-agent",
       definition=PromptAgentDefinition(
           model=model_deployment,
           instructions=
               """You are an astronomy observations assistant that helps users find 
               information about astronomical events and calculate telescope rental costs. 
               Use the available tools to assist users with their inquiries.""",
           tools=[event_tool, cost_tool, report_tool],
       ),
   )
    ```

## 에이전트에 메시지를 보내고 응답 처리

이제 함수 도구가 있는 에이전트를 만들었으므로 에이전트에 메시지를 보내고 응답을 처리할 수 있습니다.

1. **Create a thread for the chat session** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Create a thread for the chat session
   conversation = openai_client.conversations.create()
    ```

    이 코드는 에이전트와의 채팅 세션을 만듭니다.

1. **Create a list to hold function call outputs that will be sent back as input to the agent** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Create a list to hold function call outputs that will be sent back as input to the agent
   input_list: ResponseInputParam = []
    ```

    이 목록은 채팅 루프 안에서 만들어지므로 각 턴은 새로운 함수 호출 출력 집합으로 시작됩니다.

1. **Send a prompt to the agent** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Send a prompt to the agent
   openai_client.conversations.items.create(
       conversation_id=conversation.id,
       items=[{"type": "message", "role": "user", "content": user_input}],
   )
    ```

1. **Retrieve the agent's response, which may include function calls** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Retrieve the agent's response, which may include function calls
   response = openai_client.responses.create(
       conversation=conversation.id,
       extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       input=input_list,
   )

   # Check the run status for failures
   if response.status == "failed":
       print(f"Response failed: {response.error}")
    ```

    이 코드에서는 사용자 프롬프트를 에이전트에 보내고 응답을 검색합니다. 또한 응답이 실패를 나타내는지 확인하고 실패한 경우 오류를 출력합니다.

## 함수 호출 처리 및 에이전트 응답 표시

1. **Process function calls** 주석을 찾고, 에이전트가 수행한 함수 호출을 처리하도록 다음 코드를 추가합니다.

    ```python
   # Process function calls
   for item in response.output:
       if item.type == "function_call":
           # Retrieve the matching function tool
           function_name = item.name
           result = None
           if item.name == "next_visible_event":
               result = next_visible_event(**json.loads(item.arguments))
           elif item.name == "calculate_observation_cost":
               result = calculate_observation_cost(**json.loads(item.arguments))
           elif item.name == "generate_observation_report":
               result = generate_observation_report(**json.loads(item.arguments))

           # Append the output text
           input_list.append(
               FunctionCallOutput(
                   type="function_call_output",
                   call_id=item.call_id,
                   output=result,
               )
           )
    ```

    이 코드는 에이전트 응답의 항목을 반복하여 함수 호출이 있는지 확인합니다. 함수 호출이 있으면 해당 함수 도구를 검색하고 제공된 인수로 함수를 실행한 다음, 에이전트로 다시 전송할 입력 목록에 결과를 추가합니다.

1. **Send function call outputs back to the model and retrieve a response** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Send function call outputs back to the model and retrieve a response
   if input_list:
       response = openai_client.responses.create(
           conversation=conversation.id,
           input=input_list,
           extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
       )
   # Display the agent's response
   print(f"AGENT: {response.output_text}")
    ```

    이 코드는 입력 목록에 함수 호출 출력이 있는지 확인하고, 있으면 업데이트된 응답을 검색하기 위해 에이전트에 입력으로 다시 보냅니다. 마지막으로 에이전트의 응답을 출력합니다.

    출력은 같은 **conversation**에 연결되므로 함수 호출은 대화 상태에서 해결되고 에이전트의 답변은 채팅 기록에 저장됩니다. 대신 `previous_response_id`로 다시 보내면 *다음* 메시지가 *"No tool output found for function call"* 오류로 실패합니다.

1. **Delete the agent when done** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Delete the agent when done
   project_client.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
   print("Deleted agent.")
    ```

1. 파일에 추가한 전체 코드를 검토합니다. 이제 다음 섹션이 포함되어야 합니다.
   - 필요한 라이브러리 가져오기
    - Foundry 프로젝트 및 OpenAI 클라이언트에 연결
    - 에이전트가 사용할 함수 도구 정의
    - 해당 함수 도구가 있는 에이전트 만들기
    - 에이전트에 메시지를 보내고 응답 검색
    - 에이전트가 수행한 함수 호출을 처리하고 출력을 에이전트에 다시 보내기
    - 에이전트의 응답 표시
    - 완료되면 에이전트 삭제

1. 완료되면 코드 파일을 저장합니다(*CTRL+S*).

## 에이전트 애플리케이션 실행

1. 통합 터미널에서 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
   az login
    ```

    ```
   python agent.py
    ```

1. 메시지가 표시되면 다음과 같은 프롬프트를 입력합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Find me the next event I can see from South America and give me the cost for 5 hours of premium telescope time at normal priority.
    ```

    ```prompt
   South America에서 볼 수 있는 다음 이벤트를 찾아 주고, normal priority로 premium telescope time 5시간의 비용을 알려 주세요.
    ```

    이 프롬프트는 에이전트가 정의한 두 함수 도구인 `next_visible_event` 및 `calculate_observation_cost`를 모두 사용하도록 요청합니다. 에이전트는 동일한 대화 턴에서 두 함수를 모두 호출하고, 해당 함수 호출의 출력을 사용하여 사용자에게 유용한 응답을 제공할 수 있습니다.

    > **Tip**: 속도 제한 초과로 앱이 실패하면 몇 초간 기다렸다가 다시 시도합니다. 구독에서 사용할 수 있는 할당량이 부족하면 모델이 응답하지 못할 수 있습니다.

    다음과 유사한 출력이 표시됩니다.

    ```output
   AGENT: The next astronomical event you can observe from South America is the Jupiter-Venus Conjunction, taking place on May 1st.
   The cost for 5 hours of premium telescope time at normal priority for this observation will be $1,875. 
    ```

1. 관측 보고서를 생성하기 위해 다음과 같은 후속 프롬프트를 입력합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Generate that information in a report for Bellows College.
    ```

    ```prompt
   해당 정보를 Bellows College용 보고서로 생성해 주세요.
    ```

    다음과 유사한 응답이 표시됩니다.

    ```output
   AGENT: Here is your report for Bellows College:

   - Next visible astronomical event: Jupiter-Venus Conjunction
   - Date: May 1st
   - Visible from: South America
   - Observation details:
       - Telescope tier: Premium
       - Duration: 5 hours
       - Priority: Normal
   - Observation cost: $1,875

   A formal report has been generated for Bellows College.
    ```

    파일 탐색기에서 생성된 보고서가 포함된 `report-<event-type>.txt`라는 새 파일이 만들어진 것을 볼 수 있습니다. 이 파일을 열어 보고서 내용을 확인할 수 있습니다.

1. 애플리케이션을 종료하려면 `quit`을 입력합니다.

    터미널에서 Python 가상 환경을 종료하려면 `deactivate`도 사용할 수 있습니다.

## 정리

Foundry Toolkit for VS Code 확장을 살펴본 후에는 불필요한 Azure 비용이 발생하지 않도록 리소스를 정리해야 합니다.

### 모델 삭제

1. VS Code에서 **Azure Resources** 보기를 새로 고칩니다.

1. **Models** 하위 섹션을 확장합니다.

1. 배포한 모델을 마우스 오른쪽 단추로 클릭하고 **Delete**를 선택합니다.

### 리소스 그룹 삭제

1. [Azure 포털](https://portal.azure.com)을 엽니다.

1. Microsoft Foundry 리소스가 포함된 리소스 그룹으로 이동합니다.

1. **Delete resource group**을 선택하고 삭제를 확인합니다.
