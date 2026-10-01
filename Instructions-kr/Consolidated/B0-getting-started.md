---
title: '시작하기: 환경 설정'
lab:
    title: '시작하기: 환경 설정'
    description: '엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합 랩을 위한 공통 설정입니다. Microsoft Foundry 프로젝트를 만들고, 시작 코드를 가져오고, 환경을 구성합니다. 모든 작업 전에 한 번 완료하세요.'
    type: 'task'
    parent: 'B'
    order: 0
    section: 'setup'
    access: 'open'
    level: 300
    concepts: '환경 설정, Microsoft Foundry 프로젝트'
    status: 'draft'
---

# 시작하기

이 페이지에서는 **엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합** 랩에 필요한 모든 항목을 설정합니다. **모든 작업은 여기서 시작합니다**. 먼저 이 페이지를 완료하세요. 각 작업은 이후 독립적으로 수행할 수 있도록 작성되어 있습니다. 전체 랩을 한 번에 진행하는 경우 이 설정은 한 번만 수행하면 됩니다.

**시나리오:** 사용자는 신속한 제품 출시를 준비 중인 제약 제조업체 **Caldova**에서 근무합니다. 랩 전체에서 직원 지식 도우미를 빌드하고, 엔터프라이즈 문서를 기반으로 응답하게 하며, Microsoft 365를 통해 제공합니다.

> **Note**: 이 랩에서 사용하는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

시작하기 전에 다음 항목이 있는지 확인합니다.

- Azure AI 리소스를 프로비전할 수 있는 충분한 권한과 할당량이 있는 [Azure 구독](https://azure.microsoft.com/free/)
- 로컬 컴퓨터에 설치된 [Visual Studio Code](https://code.visualstudio.com/)
- 설치된 [Python 3.13](https://www.python.org/downloads/)
- 로컬 컴퓨터에 설치된 [Git](https://git-scm.com/downloads)
- Python에 대한 기본 지식

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에 3.14 빌드가 없습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

일부 선택 작업에는 추가 필수 구성 요소(Teams용 Microsoft 365 계정, Work IQ용 Microsoft 365 Copilot 라이선스 및 Node.js)가 있습니다. 각 선택 작업 페이지에는 필요한 항목이 나열되어 있습니다.

## Microsoft Foundry 프로젝트 만들기

모든 코드 작업에는 Foundry 프로젝트와 배포된 모델이 필요합니다. 포털(기본값)에서 만들거나 Azure Developer CLI(`azd`)를 사용하여 한 명령으로 프로비전할 수 있습니다.

### 옵션 A — 포털에서 프로젝트 만들기(기본값)

Microsoft Foundry는 프로젝트를 사용하여 모델, 리소스, 데이터 및 기타 자산을 구성합니다.

1. 웹 브라우저에서 [Foundry 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명으로 로그인합니다. 팁 또는 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위의 **Foundry** 로고를 사용하여 홈 페이지로 이동합니다.

    > **Important**: 이 랩에서는 **New** Foundry 환경을 사용합니다.

1. 위쪽 배너에서 **Start building**을 선택합니다.

1. 메시지가 표시되면 **new** 프로젝트를 만들고 유효한 이름(예: `caldova-knowledge-project`)을 입력합니다.

1. **Advanced options**를 확장하고 다음을 지정합니다.
    - **Microsoft Foundry resource**: *Foundry 리소스의 유효한 이름*
    - **Region**: *가까운 사용 가능한 지역 선택*\*
    - **Subscription**: *사용자의 Azure 구독*
    - **Resource group**: *리소스 그룹 선택 또는 만들기*

    > \* 일부 Azure AI 리소스는 지역별 모델 할당량의 제약을 받습니다. 나중에 할당량 제한에 도달하면 다른 지역에 다른 리소스를 만들어야 할 수 있습니다.

1. **Create**를 선택하고 프로젝트가 만들어질 때까지 기다립니다. 메시지가 표시되면 시작 대화 상자를 계속 진행하고 **Create agent**를 선택합니다.

1. **Agent name**을 `caldova-knowledge-agent`로 설정하고 에이전트를 만듭니다. 이미 배포된 모델이 선택된 상태로 플레이그라운드가 열립니다.

이 브라우저 탭을 열어 둡니다. 작업 1에서 사용합니다.

### 옵션 B — azd로 프로비전(선택 사항, 한 명령)

포털에서 클릭해 진행하고 싶지 않다면, 랩에는 Foundry 리소스, 프로젝트, 모델 배포를 만들어 주는 선택적 `azd` 템플릿이 포함되어 있습니다.

1. [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)를 설치합니다.

1. `Labfiles/B-integrate-agents-with-enterprise-knowledge-and-m365` 폴더에서 다음을 실행합니다.

    ```
    azd auth login
    azd up
    ```

1. 프롬프트(환경 이름, 지역)에 응답합니다. 완료되면 `azd`가 `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에 기록합니다.

    > **Note**: 이 작업은 리소스를 프로비전하지만 기반화된 지식 에이전트를 만들지는 **않습니다**. 작업 1에서 포털에서 이를 수행합니다. 대신 코드에서 기반화된 에이전트를 만들려면 `python ../setup/bootstrap_agent.py`를 `Python` 폴더에서 `azd up` 후 실행합니다. 랩을 완료한 후에는 `azd down`을 실행하여 생성된 모든 항목을 삭제합니다.

## 시작 코드 가져오기

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P**)를 열고 **Git: Clone**을 실행한 다음 다음을 입력합니다.

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. 복제한 리포지토리를 연 다음 **File > Open Folder**를 선택하고 `mslearn-ai-agents/Labfiles/B-integrate-agents-with-enterprise-knowledge-and-m365/Python`을 선택합니다. 이 단일 폴더에는 이 랩의 코드 작업에 사용할 시작 코드가 들어 있습니다. 하나의 가상 환경과 하나의 `.env`를 전체 기간 동안 사용합니다.

1. **requirements.txt**를 마우스 오른쪽 단추로 클릭하고 **Open in Integrated Terminal**을 선택합니다. 그런 다음 가상 환경을 만들고 패키지를 설치합니다.

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

1. **.env.example**을 **.env**로 복사한 다음 `PROJECT_ENDPOINT`를 프로젝트 엔드포인트로, `MODEL_DEPLOYMENT_NAME`을 모델 배포 이름으로 설정합니다. 파일을 저장합니다. (`azd up`을 사용한 경우 이미 채워져 있습니다.)

    > **Tip**: Foundry Toolkit VS Code 확장에서 프로젝트 배포를 마우스 오른쪽 단추로 클릭하고 **Copy Project Endpoint**를 선택하여 엔드포인트 URL을 가져옵니다.

## 작업을 시작할 준비가 되었는지 확인

각 작업에는 `.env`의 특정 값이 필요합니다. 작업을 시작하기 전에 VS Code에서 연 `Python` 폴더에서 사전 검사(preflight)를 실행합니다. 이 검사는 `.env`를 읽고 누락된 항목이 있으면 알려줍니다.

```
python ../setup/check_env.py --task 1
```

시작하려는 작업 번호에 맞게 `1`을 바꿉니다.

> **Tip**: 사전 검사는 Python 표준 라이브러리만 사용하므로 `pip install` 전이나 가상 환경이 활성화되지 않은 상태에서도 안전하게 실행할 수 있습니다.

이제 다음 작업 중 원하는 작업으로 이동합니다.

| 작업 | 페이지 |
| --- | --- |
| 작업 1 – Foundry IQ 지식 에이전트를 만들고 코드에서 연결 | [B1](B1-create-a-foundry-iq-knowledge-agent.md) |
| 작업 2 – 에이전트를 Microsoft Teams에 게시 | [B2](B2-publish-to-microsoft-teams.md) |
| 작업 3 – 에이전트를 Microsoft 365 Copilot에 게시 | [B3](B3-publish-to-microsoft-365-copilot.md) |
| 작업 4 – Work IQ: Microsoft 365 신호를 에이전트에 가져오기 | [B4](B4-work-iq-workplace-intelligence.md) |

