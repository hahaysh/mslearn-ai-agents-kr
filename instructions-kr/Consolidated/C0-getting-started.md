---
title: '시작하기: 환경 설정'
lab:
    title: '시작하기: 환경 설정'
    description: 'Agent Framework로 다중 에이전트 솔루션 빌드 랩의 공유 설정입니다. Microsoft Foundry 프로젝트를 만들고, 시작 코드를 가져오고, 환경을 구성합니다. 어떤 작업이든 시작하기 전에 한 번 완료합니다.'
    type: 'task'
    parent: 'C'
    order: 0
    section: 'setup'
    access: 'open'
    level: 300
    concepts: '환경 설정, Microsoft Foundry 프로젝트'
    status: 'draft'
---

# 시작하기

이 페이지에서는 **Agent Framework로 다중 에이전트 솔루션 빌드** 랩에 필요한 모든 항목을 설정합니다. **모든 작업은 여기에서 시작합니다** — 먼저 이 페이지를 완료합니다. 각 작업은 이후 단독으로 수행할 수 있도록 작성되어 있습니다. 전체 랩을 한 번에 진행한다면 이 설정은 한 번만 수행하면 됩니다.

**시나리오:** 사용자는 가속화된 제품 출시를 준비하는 제약 제조업체 **Caldova**에서 일합니다. 이 랩 전체에서 하나의 에이전트로 시작해 조정된 에이전트 팀으로 확장하면서 Caldova 운영의 자동화를 빌드합니다.

> **Note**: 이 랩에서 사용하는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다.
> 예상치 못한 동작, 경고 또는 오류가 발생할 수 있습니다.

## 필수 구성 요소

시작하기 전에 다음 항목을 갖추었는지 확인합니다.

- Azure AI 리소스를 프로비저닝할 수 있는 충분한 권한과 할당량이 있는 [Azure 구독](https://azure.microsoft.com/free/)
- 로컬 컴퓨터에 [Visual Studio Code](https://code.visualstudio.com/) 설치
- [Python 3.13](https://www.python.org/downloads/) 설치
- 로컬 컴퓨터에 [Git](https://git-scm.com/downloads) 설치
- Python에 대한 기본 지식

> \* Python 3.14는 아직 지원되지 않습니다. 일부 종속성에는 3.14 빌드가 없습니다. 이 랩은 Python 3.13.12로 테스트되었습니다.

## Microsoft Foundry 프로젝트 만들기

모든 작업에는 Foundry 프로젝트와 배포된 모델이 필요합니다. 포털(기본값)에서 만들거나 Azure Developer CLI(`azd`)를 사용해 한 명령으로 프로비저닝할 수 있습니다.

### 옵션 A — 포털에서 프로젝트 만들기(기본값)

Microsoft Foundry는 프로젝트를 사용해 모델, 리소스, 데이터 및 기타 자산을 구성합니다.

1. 웹 브라우저에서 [Foundry 포털](https://ai.azure.com)을 `https://ai.azure.com`에서 열고 Azure 자격 증명으로 로그인합니다. 팁 또는 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위의 **Foundry** 로고를 사용해 홈 페이지로 이동합니다.

    > **Important**: 이 랩에서는 **New** Foundry 환경을 사용합니다.

1. 위쪽 배너에서 **빌드 시작(Start building)**을 선택합니다.

1. 메시지가 표시되면 **새** 프로젝트를 만들고 유효한 이름(예: `agents-lab-project`)을 입력합니다.

1. **고급 옵션(Advanced options)**을 확장하고 다음을 지정합니다.
    - **Microsoft Foundry 리소스(Microsoft Foundry resource)**: *Foundry 리소스에 사용할 유효한 이름*
    - **지역(Region)**: *가까운 사용 가능한 지역 선택*\*
    - **구독(Subscription)**: *Azure 구독*
    - **리소스 그룹(Resource group)**: *리소스 그룹 선택 또는 만들기*

    > \* 일부 Azure AI 리소스는 지역별 모델 할당량의 제약을 받습니다. 나중에 할당량 제한에 도달하면 다른 지역에 또 다른 리소스를 만들어야 할 수 있습니다.

1. **만들기(Create)**를 선택하고 프로젝트가 만들어질 때까지 기다립니다. 메시지가 표시되면 시작 대화 상자를 계속 진행합니다.

1. 모델 배포 메시지가 표시되면 **gpt-4o** 모델(또는 사용 가능한 다른 채팅 모델)을 배포합니다. **배포 이름(deployment name)**을 기록해 둡니다. 이 값을 `MODEL_DEPLOYMENT_NAME`으로 `.env`에서 설정합니다.

1. 프로젝트 개요에서 **프로젝트 엔드포인트(Project endpoint)**를 복사합니다. 이 값을 `PROJECT_ENDPOINT`로 설정합니다.

### 옵션 B — azd로 프로비저닝(선택 사항, 한 명령)

포털을 클릭해 진행하지 않으려는 경우, 이 랩에는 Foundry 리소스, 프로젝트 및 모델 배포를 만들어 주는 선택적 `azd` 템플릿이 포함되어 있습니다.

1. [Azure Developer CLI](https://learn.microsoft.com/azure/developer/azure-developer-cli/install-azd)를 설치합니다.

1. `Labfiles/C-build-multi-agent-solutions-with-agent-framework` 폴더에서 다음을 실행합니다.

    ```
    azd auth login
    azd up
    ```

1. 프롬프트(환경 이름, 지역)에 답합니다. 완료되면 `azd`가 `PROJECT_ENDPOINT` 및 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에 작성합니다.

    > **Note**: 랩을 마치면 `azd down`을 실행하여 생성된 모든 항목을 삭제합니다.

## 시작 코드 가져오기

1. VS Code에서 명령 팔레트(**Ctrl+Shift+P**)를 열고 **Git: Clone**을 실행한 다음 다음을 입력합니다.

    ```
    https://github.com/MicrosoftLearning/mslearn-ai-agents.git
    ```

1. 복제된 리포지토리를 연 다음 **파일 > 폴더 열기(File > Open Folder)**를 선택하고 `mslearn-ai-agents/Labfiles/C-build-multi-agent-solutions-with-agent-framework/Python`을 선택합니다. 이 단일 폴더에는 이 랩의 **모든** 작업에 사용할 시작 코드가 들어 있습니다. 전체에서 하나의 가상 환경과 하나의 `.env`를 사용합니다.

1. **requirements.txt**를 마우스 오른쪽 단추로 클릭하고 **통합 터미널에서 열기(Open in Integrated Terminal)**를 선택합니다. 그런 다음 가상 환경을 만들고 패키지를 설치합니다.

    ```
    python -m venv labenv
    .\labenv\Scripts\Activate.ps1
    pip install -r requirements.txt
    ```

1. **.env** 파일을 열고 `PROJECT_ENDPOINT`를 프로젝트 엔드포인트로, `MODEL_DEPLOYMENT_NAME`을 모델 배포 이름으로 설정합니다. 파일을 저장합니다. (`azd up`을 사용했다면 이미 채워져 있습니다.)

    > **Tip**: Foundry Toolkit VS Code 확장에서 프로젝트 배포를 마우스 오른쪽 단추로 클릭하고 **프로젝트 엔드포인트 복사(Copy Project Endpoint)**를 선택하여 엔드포인트 URL을 가져옵니다.

    > `.env`에는 `SERVER_URL` 및 세 개의 `*_PORT` 값도 미리 채워져 있습니다. 이 값들은 **작업 3**(원격 에이전트)에서만 사용되며, 일반적으로 변경할 필요가 없습니다.

## 작업을 시작할 준비가 되었는지 확인

각 작업에는 `.env`의 특정 값이 필요합니다. 작업을 시작하기 전에 VS Code에서 연 `Python` 폴더에서 사전 검사(preflight check)를 실행합니다. 이 검사는 `.env`를 읽고 누락된 항목이 있는지 알려 줍니다.

```
python ../setup/check_env.py --task 1
```

시작하려는 작업 번호에 맞게 `1`을 바꿉니다.

> **Tip**: 사전 검사는 Python 표준 라이브러리만 사용하므로 `pip install` 전이나 가상 환경을 활성화하지 않은 상태에서도 안전하게 실행할 수 있습니다.

이제 원하는 작업으로 이동하면 됩니다.

| 작업 | 페이지 |
| --- | --- |
| 작업 1 – 도구가 있는 에이전트 빌드 | [C1](C1-create-an-agent-with-a-tool.md) |
| 작업 2 – 여러 에이전트를 순서대로 오케스트레이션 | [C2](C2-orchestrate-multiple-agents.md) |
| 작업 3 – A2A로 원격 에이전트 연결 | [C3](C3-connect-remote-agents-with-a2a.md) |
| 작업 4 – 지원 티켓 분류 및 라우팅 | [C4](C4-classify-and-route-a-ticket.md) |