---
title: '시작하기: 환경 설정'
lab:
    title: '시작하기: 환경 설정'
    description: 'AI 에이전트 빌드 및 확장 랩의 공유 설정입니다. Microsoft Foundry 프로젝트를 만들고, 스타터 코드를 가져오며, 환경을 구성합니다. 어떤 작업을 시작하기 전에 한 번 완료합니다.'
    type: 'task'
    parent: 'A'
    order: 0
    section: 'setup'
    access: 'open'
    level: 300
    concepts: '환경 설정, Microsoft Foundry 프로젝트'
    status: 'draft'
---

# 시작하기

**AI 에이전트 빌드 및 확장** 랩을 위한 공유 클라우드 리소스와 로컬 Python 환경을
준비합니다. 작업을 시작하기 전에 이 설정을 한 번 완료합니다.

## 사례

**Caldova**는 조기 제품 출시를 준비하는 제약 제조업체입니다. 세 공장만으로는 출시
요구량을 모두 생산할 수 없으므로, 계획 팀이 선택지를 평가할 수 있도록 공급망 도우미를
빌드합니다.

도우미에 기능을 추가하기 전에 Microsoft Foundry 프로젝트, 배포된 모델, 스타터 코드가
필요합니다. 하나의 작업만 완료하든 전체 순서를 완료하든 랩 전체에서 이 설정을
재사용합니다.

> **Note**: 이 랩에 사용되는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다.
> 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

시작하기 전에 다음이 준비되어 있는지 확인합니다.

- Azure AI 리소스를 프로비전할 수 있는 충분한 권한과 할당량이 있는 [Azure 구독](https://azure.microsoft.com/free/)
- 로컬 컴퓨터에 설치된 [Visual Studio Code](https://code.visualstudio.com/)
- 설치된 [Python 3.13](https://www.python.org/downloads/)
- 로컬 컴퓨터에 설치된 [Git](https://git-scm.com/downloads)
- Python에 대한 기본 지식

> **Note**: 일부 종속성에 3.14 빌드가 없으므로 Python 3.14는 아직 지원되지 않습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

## Microsoft Foundry 프로젝트 만들기

모든 코드 작업에는 Foundry 프로젝트와 배포된 모델이 필요합니다. 포털(기본값)에서 만들
수도 있고, Azure Developer CLI(`azd`)를 사용해 한 명령으로 프로비전할 수도 있습니다.

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
.setup-tabs { display:grid; grid-template-columns:auto auto 1fr; margin:1rem 0; }
.setup-tabs > input { position:absolute; width:1px; height:1px; overflow:hidden;
    clip:rect(0 0 0 0); white-space:nowrap; }
.setup-tabs > label { padding:.55rem .9rem; border-bottom:2px solid #d4d4d8;
    cursor:pointer; font-weight:600; color:#52525b; }
.setup-tabs > input:focus-visible + label { outline:2px solid #1a45a5; outline-offset:2px; }
.setup-tabs > input:checked + label { color:#1a45a5; border-bottom-color:#1a45a5; }
.setup-tabs .setup-panel { display:none; grid-column:1 / -1; padding-top:.75rem; }
#setup-portal:checked ~ .setup-portal-panel,
#setup-azd:checked ~ .setup-azd-panel { display:block; }
</style>

<details markdown="1" class="concept">
<summary>Microsoft Foundry 프로젝트란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

Microsoft Foundry 프로젝트는 AI 애플리케이션을 빌드하고 관리하기 위한 작업 영역입니다.
애플리케이션의 에이전트, 모델, 도구, 평가를 한곳에서 다룰 수 있습니다. 이 랩에서
프로젝트에는 Caldova 도우미에서 사용하는 모델 배포와 에이전트가 포함됩니다.

[자세히 알아보기 →](https://learn.microsoft.com/azure/ai-foundry/what-is-foundry)

</div>
</details>

<div class="setup-tabs">
<input type="radio" name="setup-method" id="setup-portal" checked="checked" />
<label for="setup-portal">옵션 A: Azure portal</label>
<input type="radio" name="setup-method" id="setup-azd" />
<label for="setup-azd">옵션 B: Azure Developer CLI</label>
<div class="setup-panel setup-portal-panel" markdown="1">

### 포털에서 프로젝트 만들기

1. 웹 브라우저에서 `https://ai.azure.com`의 [Foundry portal](https://ai.azure.com)을 열고 Azure 자격 증명으로 로그인합니다. 팁 또는 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위의 **Foundry** 로고를 사용해 홈페이지로 이동합니다.

    > **Important**: 이 랩에서는 **새(New)** Foundry 환경을 사용합니다.

1. 위쪽 배너에서 **빌드 시작(Start building)**을 선택합니다.

1. 메시지가 표시되면 **새(new)** 프로젝트를 만들고 유효한 이름(예: `agents-lab-project`)을 입력합니다.

1. **고급 옵션(Advanced options)**을 확장하고 다음을 지정합니다.
    - **Microsoft Foundry resource**: *Foundry 리소스에 사용할 유효한 이름*
    - **Region**: *가까운 사용 가능 지역 선택*\*
    - **Subscription**: *Azure 구독*
    - **Resource group**: *리소스 그룹 선택 또는 만들기*

    > \* 일부 Azure AI 리소스는 지역별 모델 할당량의 제약을 받습니다. 나중에 할당량 한도에 도달하면 다른 지역에 다른 리소스를 만들어야 할 수 있습니다.

1. **만들기(Create)**를 선택하고 프로젝트가 만들어질 때까지 기다립니다. 메시지가 표시되면 시작 대화 상자를 계속 진행하고 **에이전트 만들기(Create agent)**를 선택합니다.

1. **Agent name**을 `caldova-agent`로 설정하고 에이전트를 만듭니다. 플레이그라운드는 이미 배포된 모델이 선택된 상태로 열립니다.

이 브라우저 탭을 열어 둡니다. 작업 1에서 사용합니다.

</div>
<div class="setup-panel setup-azd-panel" markdown="1">

### azd로 프로비전

터미널에서 Azure 리소스를 설정하려는 경우 이 옵션을 선택합니다. 포함된 `azd` 템플릿은
Foundry 리소스, 프로젝트, 모델 배포를 자동으로 만듭니다.

1. [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)를 설치합니다.

1. `Labfiles/A-build-and-extend-ai-agents` 폴더에서 다음을 실행합니다.

    ```
    azd auth login
    azd up
    ```

1. 프롬프트(환경 이름, 지역)에 응답합니다. 완료되면 `azd`가
    `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에 작성합니다.

    > **Note**: 이 명령은 Azure 리소스를 만들지만 작업 1에서 사용하는 그라운딩된
    > 에이전트는 만들지 않습니다. 작업 3부터 시작하는 경우 `python ../setup/bootstrap_agent.py`를
    > `Python` 폴더에서 `azd up` 실행 후에 실행하여 에이전트를 만듭니다. 랩을 마치면
    > `azd down`을 실행하여 생성된 모든 항목을 삭제합니다.

</div>
</div>

## 스타터 코드 가져오기

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P**)를 열고 **Git: Clone**을 실행한 후 다음을 입력합니다.

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. 복제한 리포지토리를 연 다음 **File > Open Folder**를 선택하고 `mslearn-ai-agents/Labfiles/A-build-and-extend-ai-agents/Python`을 선택합니다. 이 단일 폴더에는 이 랩의 **모든** 작업에 대한 스타터 코드가 들어 있습니다. 랩 전체에서 하나의 가상 환경과 하나의 `.env`를 사용합니다.

1. **requirements.txt**를 마우스 오른쪽 단추로 클릭하고 **Open in Integrated Terminal**을 선택합니다. 그런 다음 가상 환경을 만들고 패키지를 설치합니다.

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

1. **.env** 파일을 열고 `PROJECT_ENDPOINT`를 프로젝트 엔드포인트로, `MODEL_DEPLOYMENT_NAME`을 모델 배포 이름으로 설정합니다. 파일을 저장합니다. (`azd up`을 사용한 경우 이미 채워져 있습니다.)

    > **Tip**: Foundry Toolkit VS Code 확장에서 프로젝트 배포를 마우스 오른쪽 단추로 클릭하고 **Copy Project Endpoint**를 선택하여 엔드포인트 URL을 가져옵니다.

## 작업을 시작할 준비가 되었는지 확인

각 작업에는 `.env`에 특정 값이 필요합니다. 작업을 시작하기 전에 VS Code에서 연
`Python` 폴더에서 사전 검사(preflight)를 실행합니다. 이 검사는 `.env`를 읽고
누락된 항목이 있는지 알려줍니다.

```
python ../setup/check_env.py --task 2
```

시작하려는 작업 번호에 맞게 `2`를 바꿉니다.

> **Tip**: 사전 검사는 Python 표준 라이브러리만 사용하므로 `pip install` 전이나
> 가상 환경이 활성화되지 않은 상태에서도 안전하게 실행할 수 있습니다.

이제 준비가 끝났습니다. 원하는 작업으로 이동합니다.

| 작업 | 페이지 |
| --- | --- |
| 작업 1 – 에이전트 만들기 및 그라운딩(포털) | [A1](A1-create-and-ground-an-agent.md) |
| 작업 2 – 원격 MCP 서버 연결 | [A2](A2-connect-a-remote-mcp-server.md) |
| 작업 3 – 클라이언트 앱에서 에이전트 호출 | [A3](A3-call-your-agent-from-a-client-app.md) |
| 작업 4 – 사용자 지정 함수 도구 추가 | [A4](A4-add-custom-function-tools.md) |
| 작업 5 – 종합 과제: 나만의 MCP 서버 빌드 | [A5](A5-capstone-build-your-own-mcp-server.md) |
| 작업 6 – 도우미를 호스티드 에이전트로 승격 | [A6](A6-promote-your-assistant-to-a-hosted-agent.md) |
