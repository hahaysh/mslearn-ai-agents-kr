---
lab:
    title: '포털과 VS Code를 사용하여 AI 에이전트 빌드'
    description: 'Microsoft Foundry 포털과 Foundry Toolkit VS Code 확장을 모두 사용하여 파일 검색 및 코드 인터프리터 같은 기본 제공 도구가 있는 AI 에이전트를 만듭니다.'
    level: 300
    duration: 45
    islab: true
    status: 'released'
---

# 포털과 VS Code를 사용하여 AI 에이전트 빌드

이 연습에서는 Microsoft Foundry 포털과 Foundry Toolkit VS Code 확장을 모두 사용하여 완전한 AI 에이전트 솔루션을 빌드합니다. 먼저 포털에서 grounding data와 기본 제공 도구가 있는 기본 에이전트를 만든 다음, VS Code를 사용하여 프로그래밍 방식으로 상호 작용하면서 데이터 분석을 위한 코드 인터프리터 같은 고급 기능을 사용합니다.

이 연습을 완료하는 데 약 **45**분이 걸립니다.

> **Note**: 이 연습에서 사용하는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

이 연습을 시작하기 전에 다음이 준비되어 있는지 확인합니다.

- Azure AI 리소스를 프로비저닝할 수 있는 충분한 권한과 할당량이 있는 [Azure 구독](https://azure.microsoft.com/free/)
- 로컬 컴퓨터에 설치된 [Visual Studio Code](https://code.visualstudio.com/)
- 설치된 [Python 3.13](https://www.python.org/downloads/)
- 로컬 컴퓨터에 설치된 [Git](https://git-scm.com/downloads)
- Azure AI 서비스 및 Python 프로그래밍에 대한 기본 지식

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에는 3.14 빌드가 없습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

## Microsoft Foundry 프로젝트 만들기

Microsoft Foundry는 프로젝트를 사용하여 AI 솔루션 개발에 사용되는 모델, 리소스, 데이터 및 기타 자산을 구성합니다.

1. 웹 브라우저에서 [Foundry 포털](https://ai.azure.com)을 `https://ai.azure.com`에서 열고 Azure 자격 증명으로 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위의 **Foundry** 로고를 사용하여 홈페이지로 이동합니다.

    > **Important**: 이 랩에서는 **새(New)** Foundry 환경을 사용합니다.

1. 위쪽 배너에서 새 Microsoft Foundry 환경을 사용해 보려면 **빌드 시작(Start building)** 을 선택합니다.

1. 메시지가 표시되면 **새(new)** 프로젝트를 만들고 프로젝트의 유효한 이름을 입력합니다(예: `it-support-agent-project`).

1. **고급 옵션(Advanced options)** 을 확장하고 다음 설정을 지정합니다.
    - **Microsoft Foundry resource**: *Foundry 리소스의 유효한 이름*
    - **Region**: *가까운 사용 가능한 지역 선택*\**
    - **Subscription**: *Azure 구독*
    - **Resource group**: *리소스 그룹을 선택하거나 새로 만듭니다.*

    > \* 일부 Azure AI 리소스는 지역별 모델 할당량의 제약을 받습니다. 연습 후반에 할당량 제한을 초과하는 경우 다른 지역에 또 다른 리소스를 만들어야 할 수 있습니다.

1. **만들기(Create)** 를 선택하고 프로젝트가 만들어질 때까지 기다립니다.

1. 프로젝트가 만들어지면 시작 대화 상자가 나타날 수 있습니다. **다음(Next)** 을 선택하여 시작 메시지를 읽은 다음 **에이전트 만들기(Create agent)** 를 선택합니다.

    홈페이지에서 **Start building**을 선택한 다음 드롭다운 메뉴에서 **Create agents**를 선택할 수도 있습니다.

1. **Agent name**을 `it-support-agent`로 설정하고 에이전트를 만듭니다.

새로 만든 에이전트의 playground가 열립니다. 사용 가능한 배포된 모델이 이미 선택되어 있는 것을 볼 수 있습니다.

## 지침 및 grounding data로 에이전트 구성

이제 에이전트를 만들었으므로 지침을 구성하고 grounding data를 추가해 보겠습니다.

1. 에이전트 playground에서 **Instructions**를 다음으로 설정합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```prompt
   You are an IT Support Agent for Contoso Corporation.
   You help employees with technical issues and IT policy questions.

   Guidelines:
   - Always be professional and helpful
   - Use the IT policy documentation to answer questions accurately
   - If you don't know the answer, admit it and suggest contacting IT support directly
   - When creating tickets, collect all necessary information before proceeding
    ```

    ```prompt
   Contoso Corporation의 IT 지원 에이전트입니다.
   직원의 기술 문제와 IT 정책 질문을 지원합니다.

   지침:
   - 항상 전문적이고 친절하게 응답합니다.
   - IT 정책 문서를 사용하여 질문에 정확하게 답변합니다.
   - 답을 모르면 모른다고 인정하고 IT 지원팀에 직접 문의하도록 제안합니다.
   - 티켓을 만들 때는 진행하기 전에 필요한 모든 정보를 수집합니다.
    ```

1. 랩 리포지토리에서 IT 정책 문서를 다운로드합니다. 새 브라우저 탭을 열고 다음 위치로 이동합니다.

    ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/IT_Policy.txt
    ```

    파일을 로컬 컴퓨터에 저장합니다.

    > **Note**: 이 문서에는 암호 재설정, 소프트웨어 설치 요청 및 하드웨어 문제 해결에 대한 샘플 IT 정책이 포함되어 있습니다.

1. 에이전트 playground로 돌아갑니다. **도구(Tools)** 섹션에서 **추가(Add)** 를 선택한 다음 **파일 검색(File search)** 과 **</> Code interpreter**를 모두 추가합니다.

1. **Add** 오른쪽에서 **Upload files**를 선택합니다. **Attach files**에서 찾아보기를 사용하여 방금 다운로드한 `IT_Policy.txt` 파일을 업로드한 다음 **Attach**를 선택합니다.

1. 파일이 인덱싱될 때까지 기다립니다. 준비되면 확인 메시지가 표시됩니다.

1. 이제 코드 인터프리터가 분석할 성능 데이터를 추가해 보겠습니다. 다음에서 시스템 성능 데이터 파일을 다운로드합니다.

    ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/01-build-agent-portal-and-vscode/system_performance.csv
    ```

    이 파일을 로컬 컴퓨터에 저장합니다.

1. **</> Code interpreter** 오른쪽에서 **+ Files**를 선택한 다음 방금 다운로드한 `system_performance.csv` 파일을 업로드합니다.

    > **Note**: 이 CSV 파일에는 에이전트가 분석할 수 있는 시간 경과에 따른 시뮬레이션된 시스템 메트릭(CPU, 메모리, 디스크 사용량)이 포함되어 있습니다.

1. 에이전트를 저장합니다.

## 에이전트 테스트

grounding data를 사용하여 에이전트가 어떻게 응답하는지 테스트해 보겠습니다.

1. playground 오른쪽의 채팅 인터페이스에 다음 프롬프트를 입력합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What's the policy for password resets?
    ```

    ```prompt
   암호 재설정 정책은 무엇인가요?
    ```

1. 응답을 검토합니다. 에이전트는 IT 정책 문서를 참조하고 암호 재설정 절차에 대한 정확한 정보를 제공해야 합니다.

1. 다른 프롬프트도 시도합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   How do I request new software?
    ```

    ```prompt
   새 소프트웨어는 어떻게 요청하나요?
    ```

1. 다시 응답을 검토하고 에이전트가 grounding data를 어떻게 사용하는지 관찰합니다.

1. 이제 데이터 분석 요청으로 코드 인터프리터를 테스트합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Can you analyze the system performance data and tell me if there are any concerning trends?
    ```

    ```prompt
   시스템 성능 데이터를 분석하고 우려되는 추세가 있는지 알려줄 수 있나요?
    ```

1. 에이전트는 코드 인터프리터를 사용하여 CSV 파일을 분석하고 시스템 성능에 대한 인사이트를 제공해야 합니다.

1. 시각화를 요청해 봅니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Create a chart showing CPU usage over time from the performance data
    ```

    ```prompt
   성능 데이터에서 시간 경과에 따른 CPU 사용량을 보여 주는 차트를 만들어 주세요.
    ```

1. 에이전트는 코드 인터프리터를 사용하여 시각화와 분석을 생성합니다.

좋습니다. grounding data, 파일 검색 및 코드 인터프리터 기능이 있는 에이전트를 만들었습니다. 다음 섹션에서는 VS Code를 사용하여 이 에이전트와 프로그래밍 방식으로 상호 작용합니다.

## VS Code를 사용하여 에이전트와 상호 작용

개발자는 Foundry 포털에서 작업하는 시간도 있지만 Visual Studio Code에서 많은 시간을 보낼 가능성이 큽니다. Foundry Toolkit for VS Code 확장은 개발 환경을 벗어나지 않고 Foundry 프로젝트 리소스로 작업할 수 있는 편리한 방법을 제공합니다.

### VS Code 확장 설치 및 구성

Foundry Toolkit 확장을 이미 설치했다면 이 섹션을 건너뛸 수 있습니다.

1. Visual Studio Code를 엽니다.

2. 왼쪽 창에서 **확장(Extensions)** 을 선택하거나 **Ctrl+Shift+X**를 누릅니다.

3. 확장 Marketplace에서 Microsoft의 `Foundry Toolkit for VS Code` 확장을 검색하고 **설치(Install)** 를 선택합니다.

    Foundry Toolkit Extension을 설치하면 VS Code에 Foundry Toolkit 확장이 추가됩니다.

    > **Note**: 현재 확장은 **Foundry Toolkit**으로 표시되지만, 일부 VS Code 레이블, 명령 또는 이전 스크린샷에서는 여전히 **AI Toolkit**으로 표시될 수 있습니다. 이 랩에서는 이러한 이름이 동일한 확장 환경을 가리키는 것으로 간주합니다.

4. 확장을 설치한 후 사이드바에서 Foundry Toolkit 아이콘을 선택합니다.

    아직 Azure 계정에 로그인하지 않았다면 로그인하라는 메시지가 표시됩니다.

### VS Code에서 에이전트 테스트

코드를 작성하기 전에 확장 인터페이스에서 직접 에이전트와 상호 작용할 수 있습니다.

1. **Microsoft Foundry Resources** 아래에서 **Set Default Project**를 선택합니다.

    기본 프로젝트가 이미 활성 상태이면 프로젝트 이름이 **My Resources** 아래에 표시됩니다. 활성 프로젝트를 마우스 오른쪽 단추로 클릭하고 **Switch Default Project in Azure Extension**을 선택하여 다른 프로젝트로 전환할 수 있습니다.

2. 프로젝트 섹션을 확장합니다. **Prompt Agents** 아래에 포털에서 만든 `it-support-agent`가 표시됩니다. 에이전트 이름을 선택하여 Agent Builder 인터페이스를 엽니다.

    Agent Builder 인터페이스에 에이전트 playground가 표시되어 VS Code를 벗어나지 않고 에이전트와 상호 작용하고 설정을 구성할 수 있습니다.

3. playground 채팅 창에서 다음과 같은 질문을 입력합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What is the policy for reporting a lost or stolen device?
    ```

    ```prompt
   분실되었거나 도난당한 장치를 신고하는 정책은 무엇인가요?
    ```

4. 에이전트의 응답을 검토합니다. 앞서 업로드한 grounding data를 사용하여 관련 IT 정책 정보를 제공해야 합니다.

    > **Tip**: 이 기본 제공 playground를 사용하면 코드를 작성하지 않고도 에이전트의 지침과 지식을 빠르게 테스트할 수 있습니다.

## 에이전트와 상호 작용하는 클라이언트 애플리케이션 만들기

이제 에이전트와 프로그래밍 방식으로 상호 작용하는 클라이언트 애플리케이션을 만들어 보겠습니다.

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P** 또는 **View > Command Palette**)를 엽니다.

1. **Git: Clone**을 입력하고 목록에서 선택합니다.

1. 리포지토리 URL을 입력합니다.

    ```
   https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. 로컬 컴퓨터에서 리포지토리를 복제할 위치를 선택합니다.

1. 메시지가 표시되면 **Open**을 선택하여 복제한 리포지토리를 VS Code에서 엽니다.

1. 리포지토리가 열리면 **File > Open Folder**를 선택하고 `mslearn-ai-agents/Labfiles/01-build-agent-portal-and-vscode/Python`으로 이동한 다음 **Select Folder**를 선택합니다.

1. 탐색기 창에서 `agent_with_functions.py` 파일을 엽니다. 파일이 비어 있으면 해당 내용을 다음 코드로 바꿉니다.

1. 다음 코드를 사용합니다.

    ```python
   import base64
   import os
   from pathlib import Path

   from azure.ai.projects import AIProjectClient
   from azure.identity import DefaultAzureCredential
   from dotenv import load_dotenv


   OUTPUT_DIR = Path("agent_outputs")


   def get_output_path(filename):
       """Create a unique path for generated files."""
       OUTPUT_DIR.mkdir(exist_ok=True)
       file_name = Path(filename).name
       stem = Path(file_name).stem or "output"
       suffix = Path(file_name).suffix
       output_path = OUTPUT_DIR / file_name

       counter = 1
       while output_path.exists():
           output_path = OUTPUT_DIR / f"{stem}_{counter}{suffix}"
           counter += 1

       return output_path


   def save_bytes(file_bytes, filename):
       """Save binary content to a local file."""
       output_path = get_output_path(filename)
       with open(output_path, "wb") as file_handle:
           file_handle.write(file_bytes)
       return output_path


   def save_image(image_data, filename):
       """Save base64 image data to a file."""
       return save_bytes(base64.b64decode(image_data), filename)


   def download_container_file(openai_client, annotation, downloaded_files):
       """Download a cited container file once and return its local path."""
       cache_key = (annotation.container_id, annotation.file_id)
       if cache_key in downloaded_files:
           return downloaded_files[cache_key]

       file_content = openai_client.containers.files.content.retrieve(
           file_id=annotation.file_id,
           container_id=annotation.container_id,
       )
       output_path = save_bytes(
           file_content.read(),
           annotation.filename or f"{annotation.file_id}.bin",
       )
       downloaded_files[cache_key] = output_path
       return output_path


   def format_output_text(content_item, openai_client, downloaded_files):
       """Replace sandbox file citations with local file paths."""
       text = content_item.text or ""
       replacements = []
       referenced_files = set()

       for annotation in content_item.annotations or []:
           if getattr(annotation, "type", "") != "container_file_citation":
               continue

           output_path = download_container_file(openai_client, annotation, downloaded_files)
           replacement_text = f"{annotation.filename} (saved to {output_path})"
           referenced_files.add(output_path)

           start_index = getattr(annotation, "start_index", None)
           end_index = getattr(annotation, "end_index", None)
           if start_index is not None and end_index is not None:
               replacements.append((start_index, end_index, replacement_text))
               continue

           annotated_text = getattr(annotation, "text", "")
           if annotated_text:
               text = text.replace(annotated_text, replacement_text)

       for start_index, end_index, replacement_text in sorted(replacements, reverse=True):
           text = f"{text[:start_index]}{replacement_text}{text[end_index:]}"

       return text, referenced_files


   def main():
       # Initialize the project client
       load_dotenv()
       project_endpoint = os.environ.get("PROJECT_ENDPOINT")
       agent_name = os.environ.get("AGENT_NAME", "it-support-agent")

       if not project_endpoint:
           print("Error: PROJECT_ENDPOINT environment variable not set")
           print("Please set it in your .env file or environment")
           return

       print("Connecting to Microsoft Foundry project...")
       credential = DefaultAzureCredential()
       project_client = AIProjectClient(
           credential=credential,
           endpoint=project_endpoint
       )

       # Get the OpenAI client for Responses API
       openai_client = project_client.get_openai_client()

       # Get the agent created in the portal
       print(f"Loading agent: {agent_name}")
       agent = project_client.agents.get(agent_name=agent_name)
       print(f"Connected to agent: {agent.name} (id: {agent.id})")

       # Create a conversation
       conversation = openai_client.conversations.create(items=[])
       print(f"Conversation created (id: {conversation.id})")

       # Chat loop
       print("\n" + "="*60)
       print("IT Support Agent Ready!")
       print("Ask questions, request data analysis, or get help.")
       print("Type 'exit' to quit.")
       print("="*60 + "\n")

       while True:
           user_input = input("You: ").strip()

           if user_input.lower() in ['exit', 'quit', 'bye']:
               print("Goodbye!")
               break

           if not user_input:
               continue

           # Add user message to conversation
           openai_client.conversations.items.create(
               conversation_id=conversation.id,
               items=[{"type": "message", "role": "user", "content": user_input}]
           )

           # Get response from agent
           print("\n[Agent is thinking...]")
           response = openai_client.responses.create(
               conversation=conversation.id,
               extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
               input=""
           )

           # Display response and save any generated files locally
           handled_output = False
           downloaded_files = {}
           referenced_files = set()
           image_count = 0

           if hasattr(response, "output") and response.output:
               for item in response.output:
                   item_type = getattr(item, "type", "")

                   if item_type == "message" and getattr(item, "content", None):
                       for content_item in item.content:
                           if getattr(content_item, "type", "") != "output_text":
                               continue

                           formatted_text, message_files = format_output_text(
                               content_item,
                               openai_client,
                               downloaded_files,
                           )
                           referenced_files.update(message_files)

                           if formatted_text:
                               print(f"\nAgent: {formatted_text}\n")
                               handled_output = True

                   elif hasattr(item, "text") and item.text:
                       print(f"\nAgent: {item.text}\n")
                       handled_output = True

                   elif item_type == "image":
                       image_count += 1
                       filename = f"chart_{image_count}.png"

                       if hasattr(item, "image") and hasattr(item.image, "data"):
                           file_path = save_image(item.image.data, filename)
                           print(f"\n[Agent generated a chart - saved to: {file_path}]")
                       else:
                           print("\n[Agent generated an image]")
                       handled_output = True

               for file_path in downloaded_files.values():
                   if file_path not in referenced_files:
                       print(f"\n[Agent generated a file - saved to: {file_path}]")
                       handled_output = True

           if not handled_output and hasattr(response, "output_text") and response.output_text:
               print(f"\nAgent: {response.output_text}\n")

   if __name__ == "__main__":
       main()
    ```

1. `agent_with_functions.py` 파일을 저장합니다(**Ctrl+S** 또는 **File > Save**).

### 환경 구성 및 애플리케이션 실행

1. 탐색기 창에 `.env.example` 및 `requirements.txt` 파일이 이미 폴더에 있는 것을 볼 수 있습니다.

1. `.env.example` 파일을 복제하고 이름을 `.env`로 바꿉니다.

1. `.env` 파일에서 `your_project_endpoint_here`를 실제 프로젝트 엔드포인트로 바꿉니다.

    ```
   PROJECT_ENDPOINT=<your_project_endpoint>
   AGENT_NAME=it-support-agent
    ```

    **프로젝트 엔드포인트를 가져오려면:** VS Code에서 **Foundry Toolkit** 확장을 열고 활성 프로젝트를 마우스 오른쪽 단추로 클릭한 다음 **Copy Endpoint**를 선택합니다. 설치된 Foundry Toolkit 버전에서 **Copy Endpoint**를 사용할 수 없는 경우 Microsoft Foundry 포털을 열고 프로젝트로 이동한 다음 프로젝트 개요 페이지에서 프로젝트 엔드포인트를 복사합니다.

1. `.env` 파일을 저장합니다(**Ctrl+S** 또는 **File > Save**).

1. VS Code에서 터미널(**Terminal > New Terminal**)을 열고 작업 디렉터리로 이동합니다.

1. 필요한 패키지를 설치하고 로그인합니다.

    ```bash
   python -m venv labenv
   .\labenv\Scripts\Activate.ps1
   pip install -r requirements.txt
    ```

    ```bash
   az login
    ```

1. 애플리케이션을 실행합니다.

    ```bash
   python agent_with_functions.py
    ```

## 클라이언트 애플리케이션 테스트

에이전트가 시작되면 다음 프롬프트를 사용해 여러 기능을 테스트합니다.

1. 파일 검색으로 정책 검색을 테스트합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What's the policy for password resets?
    ```

    ```prompt
   암호 재설정 정책은 무엇인가요?
    ```

2. 코드 인터프리터로 데이터 분석을 요청합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Analyze the system performance data and identify any periods where CPU usage exceeded 80%
    ```

    ```prompt
   시스템 성능 데이터를 분석하고 CPU 사용량이 80%를 초과한 기간을 찾아 주세요.
    ```

3. 시각화를 요청합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Create a line chart showing memory usage trends over time
    ```

    ```prompt
   시간 경과에 따른 메모리 사용량 추세를 보여 주는 선형 차트를 만들어 주세요.
    ```

    애플리케이션은 생성된 차트와 인용된 파일을 `agent_outputs` 폴더에 저장하고 터미널에 로컬 파일 경로를 출력합니다.

4. 통계 분석을 요청합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What are the average, minimum, and maximum values for disk usage in the performance data?
    ```

    ```prompt
   성능 데이터에서 디스크 사용량의 평균, 최솟값, 최댓값은 무엇인가요?
    ```

5. 결합 분석을 요청합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Find any correlation between high CPU usage and memory usage in the performance data
    ```

    ```prompt
   성능 데이터에서 높은 CPU 사용량과 메모리 사용량 사이의 상관관계를 찾아 주세요.
    ```

에이전트가 파일 검색(정책 질문용)과 코드 인터프리터(데이터 분석용)를 모두 사용하여 요청을 수행하는 방식을 관찰합니다. 코드 인터프리터는 CSV 데이터를 분석하고 계산을 수행하며 시각화도 생성할 수 있습니다. 테스트가 끝나면 `exit`를 입력합니다.

## 정리

불필요한 Azure 요금을 방지하려면 만든 리소스를 삭제합니다.

1. Foundry 포털에서 프로젝트로 이동합니다.
1. **Settings** > **Delete project**를 선택합니다.
1. 또는 Azure 포털에서 전체 리소스 그룹을 삭제합니다.
