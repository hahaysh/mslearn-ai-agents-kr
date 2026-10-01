---
title: '시작하기: 환경 설정'
lab:
    title: '시작하기: 환경 설정'
    description: '에이전트 관찰, 평가 및 보안 랩의 공통 설정입니다. Microsoft Foundry 프로젝트를 만들고, Application Insights를 연결하고, 시작 코드를 가져오고, 환경을 구성합니다. 작업을 시작하기 전에 한 번 완료합니다.'
    type: 'task'
    parent: 'D'
    order: 0
    section: 'setup'
    access: 'open'
    level: 300
    concepts: '환경 설정, Microsoft Foundry 프로젝트, Application Insights'
    status: 'draft'
---

# 시작하기

이 페이지에서는 **에이전트 관찰, 평가 및 보안** 랩에 필요한 모든 항목을 설정합니다.
**모든 작업은 여기서 시작합니다**. 먼저 이 페이지를 완료합니다. 이후 각 작업은 독립적으로 수행할 수
있도록 작성되어 있습니다. 전체 랩을 한 번에 진행하는 경우 이 설정은 한 번만 수행하면 됩니다.

**시나리오:** 여러분은 가속화된 제품 출시를 준비 중인 제약 제조업체 **Caldova**에서 일합니다.
공급망 도우미가 운영 중이며, 이 랩에서는 이를 추적하고, 답변을 채점하고, 공격해 보면서 실제로
무엇을 하는지 확인합니다.

> **Note**: 이 랩에서 사용하는 일부 기술은 미리 보기이거나 활발히 개발 중입니다.
> 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

시작하기 전에 다음을 갖추었는지 확인합니다.

- Azure AI 리소스를 프로비전할 충분한 권한과 할당량이 있는 [Azure 구독](https://azure.microsoft.com/free/)
- 로컬 컴퓨터에 설치된 [Visual Studio Code](https://code.visualstudio.com/)
- 설치된 [Python 3.13](https://www.python.org/downloads/)
- 로컬 컴퓨터에 설치된 [Git](https://git-scm.com/downloads)
- Python에 대한 기본적인 이해

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에는 3.14 빌드가 없습니다.

## Microsoft Foundry 프로젝트 만들기

모든 작업에는 Foundry 프로젝트와 배포된 모델이 필요합니다. 포털(기본값)에서 만들거나
Azure Developer CLI(`azd`)를 사용해 한 명령으로 프로비전할 수 있습니다.

### 옵션 A — 포털에서 프로젝트 만들기(기본값)

1. 웹 브라우저에서 `https://ai.azure.com`의 [Foundry portal](https://ai.azure.com)을 열고 Azure 자격 증명으로 로그인합니다. 팁 또는 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위의 **Foundry** 로고를 사용해 홈 페이지로 이동합니다.

    > **Important**: 이 랩에서는 **New** Foundry 환경을 사용합니다.

1. 위쪽 배너에서 **Start building**을 선택합니다.

1. 메시지가 표시되면 **new** 프로젝트를 만들고 유효한 이름(예: `observability-lab-project`)을 입력합니다.

1. **Advanced options**를 확장하고 다음을 지정합니다.
    - **Microsoft Foundry resource**: *Foundry 리소스의 유효한 이름*
    - **Region**: *가까운 사용 가능한 지역 선택*\*
    - **Subscription**: *Azure 구독*
    - **Resource group**: *리소스 그룹 선택 또는 만들기*

    > \* **작업 3**(red teaming)을 수행할 계획이라면 AI Red Teaming Agent는
    > **East US 2**, **France Central**, **Sweden Central**, **Switzerland West** 및
    > **North Central US**에서만 사용할 수 있습니다. 지금 이 중 하나를 선택하면 나중에 두 번째 프로젝트를
    > 만들 필요가 없습니다.

1. **Create**를 선택하고 프로젝트가 생성될 때까지 기다립니다.

1. 프로젝트 **Overview** 페이지에서 **project endpoint**와 자동으로 만들어진 모델 배포 이름을 기록합니다.
    둘 다 `.env`에 입력합니다.

### 옵션 B — azd로 프로비전(선택 사항, 한 명령)

포털을 클릭해 진행하지 않으려면, 이 랩에는 Foundry 리소스, 프로젝트, 모델 배포를 만들어 주는
선택적 `azd` 템플릿이 포함되어 있습니다. 이 작업은 리포지토리 내부에서 실행되므로, 아직 복제하지 않았다면
먼저 복제합니다.

```
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
```

1. [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)를 설치합니다.

1. 방금 복제한 리포지토리의 `Labfiles/D-observe-evaluate-and-secure-agents` 폴더에서 다음을 실행합니다.

    ```
    azd auth login
    azd up
    ```

1. 프롬프트(환경 이름, 지역)에 답합니다. 완료되면 `azd`가 `PROJECT_ENDPOINT`와
    `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에 대신 기록합니다.

    > **Note**: `azd up`은 작업 1에 필요한 Application Insights 리소스를 만들지 **않습니다**.
    > 아래 단계에 따라 포털에서 연결합니다. 랩이 끝나면 `azd down`을 실행해 생성된 모든 항목을
    > 삭제합니다.

## 시작 코드 가져오기

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P**)를 열고 **Git: Clone**을 실행한 다음 다음을 입력합니다.

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

    > 위의 `azd` 옵션을 위해 이미 리포지토리를 복제했다면 이 단계를 건너뛰고 열기만 하면 됩니다.

1. 복제한 리포지토리를 연 다음 **File > Open Folder**를 선택하고 `mslearn-ai-agents/Labfiles/D-observe-evaluate-and-secure-agents/Python`을 선택합니다. 이 단일 폴더에는 이 랩의 **모든** 작업에 대한 시작 코드가 들어 있습니다. 전체에서 하나의 가상 환경과 하나의 `.env`를 사용합니다.

1. **requirements.txt**를 마우스 오른쪽 단추로 클릭하고 **Open in Integrated Terminal**을 선택합니다. 그런 다음 가상 환경을 만들고 패키지를 설치합니다.

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

    > 이 설치는 다른 랩보다 큽니다. 평가 SDK와 작업 3용 PyRIT가 포함되어 있습니다.
    > 몇 분 정도 걸릴 수 있습니다.

1. **.env** 파일을 열고 `PROJECT_ENDPOINT`를 프로젝트 엔드포인트로, `MODEL_DEPLOYMENT_NAME`을 모델 배포 이름으로 설정합니다. 파일을 저장합니다. (`azd up`을 사용했다면 이미 채워져 있습니다.)

    > **Tip**: Foundry Toolkit VS Code 확장에서 프로젝트 배포를 마우스 오른쪽 단추로 클릭하고 **Copy Project Endpoint**를 선택하면 엔드포인트 URL을 가져올 수 있습니다.

## 측정할 에이전트 가져오기(모든 작업에 필요)

모든 작업은 **grounded** 에이전트를 사용합니다. 이 에이전트는 모델 자체의 메모리가 아니라 Caldova
기술 자료를 기반으로 답합니다. 에이전트를 가져오는 방법은 두 가지입니다.

- **[Lab B](B-integrate-agents-with-enterprise-knowledge-and-m365.md)를 완료한 경우**: `AGENT_NAME`을 `.env`에서 해당 에이전트 이름(기본값을 유지했다면 `caldova-knowledge-agent`)으로 설정하면 완료됩니다.
- **완료하지 않은 경우**: 여기서 동등한 에이전트를 만듭니다. 로그인한 다음 가상 환경이 활성화된 `Python` 폴더에서 다음을 실행합니다.

    ```
    az login
    ```

    ```
    python ../setup/bootstrap_agent.py
    ```

    이 작업은 `Python/knowledge/`의 문서를 업로드하고, File Search를 사용해 해당 문서에 기반한
    `caldova-knowledge-agent`라는 에이전트를 만든 다음 `AGENT_NAME`을 `.env`에 기록합니다.

## Application Insights 연결(작업 1에 필요)

Foundry는 프로젝트에 연결된 **Application Insights** 리소스에 추적을 저장합니다. 지금 연결합니다.
1분 정도 걸리며, 연결되면 Foundry가 코드 없이도 에이전트에 대한 서버 쪽 추적 기록을 시작합니다.

1. [Foundry portal](https://ai.azure.com)에서 프로젝트를 엽니다.

1. 왼쪽 탐색에서 **Agents**를 선택한 다음 위쪽의 **Traces**를 선택합니다.

1. **Connect**를 선택한 다음 기존 Application Insights 리소스를 선택하거나 **Create new**를 선택하고 마법사를 완료합니다.

    > **Connect** 단추가 보이지 않으면 오른쪽 위의 **Manage**를 선택한 다음
    > **Project details** > **Connected resources** > **Add connection** > **Application Insights**를 선택합니다.

1. 추적을 *읽으려면* 해당 Application Insights 리소스에 대한 **Log Analytics Reader** 역할이 필요합니다.
    직접 만든 경우 이미 보유하고 있습니다.

> **Why this matters**: Foundry 프로젝트는 연결된 항목이 있을 때만 코드에 연결 문자열을 전달할 수 있습니다.
> 프로젝트에 해당 문자열을 요청하는 작업 1을 시작하기 전에 이 작업을 수행합니다.

## 작업 준비 확인

각 작업에는 `.env`의 특정 값이 필요합니다. 작업을 시작하기 전에 VS Code에서 연 `Python` 폴더에서
preflight 검사를 실행합니다. 이 검사는 `.env`를 읽고 누락된 항목이 있는지 알려 줍니다.

```
python ../setup/check_env.py --task 1
```

`1`을 시작하려는 작업 번호로 바꿉니다.

> **Tip**: preflight 검사는 Python 표준 라이브러리만 사용하므로 `pip install` 전이나 가상 환경이
> 활성화되지 않은 상태에서도 안전하게 실행할 수 있습니다. Application Insights가 연결되었는지는
> 확인할 수 없습니다. 이는 `.env` 값이 아니라 프로젝트 설정이므로, 작업 1에서 시작하는 경우 위의
> 연결 단계를 수행합니다.

이제 끝났습니다. 원하는 작업으로 이동합니다.

| 작업 | 페이지 |
| --- | --- |
| 작업 1 – 에이전트 추적 | [D1](D1-trace-your-agent.md) |
| 작업 2 – 답변 품질 평가 | [D2](D2-evaluate-answer-quality.md) |
| 작업 3 – 에이전트 red team | [D3](D3-red-team-your-agent.md) |
