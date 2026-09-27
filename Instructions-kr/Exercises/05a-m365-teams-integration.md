---
lab:
    title: '에이전트를 Microsoft Teams 및 Copilot에 배포'
    description: '엔터프라이즈 액세스를 위해 AI 에이전트를 Microsoft Teams 및 Microsoft 365 Copilot에 게시합니다.'
    level: 300
    duration: 40
    islab: true
    status: 'released'
---

# 에이전트를 Microsoft Teams 및 Copilot에 배포

이 랩에서는 직원들이 이미 작업하는 위치에서 AI 에이전트에 액세스할 수 있도록 AI 에이전트를 **Microsoft Teams** 및 **Microsoft 365 Copilot**에 게시하는 방법을 알아봅니다. Foundry 포털에서 간단한 에이전트를 만들고, 지식 기반 그라운딩을 추가한 다음, 두 플랫폼에 배포합니다.

이 랩은 에이전트 개발이 아니라 **배포 및 게시 워크플로**에 중점을 둡니다.

이 랩은 약 **40**분이 걸립니다.

> **Note**: Microsoft 365 Copilot에 게시하려면 Copilot 라이선스가 필요합니다. Teams 배포는 표준 Microsoft 365 계정에서 작동합니다.

## 필수 구성 요소

이 랩을 시작하기 전에 다음이 준비되어 있는지 확인합니다.

- AI 리소스를 만들 수 있는 권한이 있는 [Azure 구독](https://azure.microsoft.com/free/)
- Teams 액세스 권한이 있는 **Microsoft 365 account**
- **Microsoft 365 Copilot license**(선택 사항, Copilot 배포용)
- Microsoft Foundry 포털에 대한 기본 지식

## Foundry 프로젝트 만들기

Microsoft Foundry는 프로젝트를 사용하여 AI 솔루션 개발에 사용되는 모델, 리소스, 데이터 및 기타 자산을 구성합니다.

1. 웹 브라우저에서 [Foundry 포털](https://ai.azure.com)을 `https://ai.azure.com`에서 열고 Azure 자격 증명으로 로그인합니다. 처음 로그인할 때 열리는 팁 또는 빠른 시작 창을 모두 닫고, 필요한 경우 왼쪽 위의 **Foundry** 로고를 사용하여 홈 페이지로 이동합니다.

    > **Important**: 이 랩에서는 **New** Foundry 환경을 사용합니다.

1. 위쪽 배너에서 **Start building**을 선택하여 새 Microsoft Foundry Experience를 사용해 봅니다.

1. 메시지가 표시되면 **new** 프로젝트를 만들고 프로젝트에 사용할 유효한 이름을 입력합니다(예: *m365-lab*).

1. **Advanced options**를 확장하고 다음 설정을 지정합니다.
    - **Foundry resource**: *Create a new Foundry resource or select an existing one*
    - **Subscription**: *Your Azure subscription*
    - **Resource group**: *Create or select a resource group*
    - **Location**: *Select any available region*\

    > \* 일부 Azure AI 리소스는 지역별 모델 할당량의 제약을 받습니다. 연습 후반에 할당량 제한을 초과하는 경우 다른 지역에 다른 리소스를 만들어야 할 수도 있습니다.

1. **Create**를 선택하고 프로젝트가 만들어질 때까지 기다립니다.

2. 프로젝트가 만들어지면 환영 대화 상자가 나타날 수 있습니다. **Next**를 선택하여 환영 메시지를 읽은 다음 **Create agent**를 선택합니다.

    홈 페이지에서 **Start building**을 선택한 다음 드롭다운 메뉴에서 **Create agents**를 선택할 수도 있습니다.

3. **Agent name**을 `enterprise-knowledge-agent`로 설정하고 에이전트를 만듭니다.

새로 만든 에이전트의 플레이그라운드가 열립니다. 사용 가능한 배포된 모델이 이미 선택되어 있는 것을 볼 수 있습니다.

## 지침 및 그라운딩 데이터로 에이전트 구성

이제 에이전트를 만들었으므로 게시를 준비하기 위해 지침과 지식을 구성하겠습니다.

1. **Instructions**를 다음으로 설정합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   You are an Enterprise Knowledge Assistant for Contoso Corporation.

   Your role:
   - Answer questions about company policies and procedures
   - Provide accurate information from uploaded documents
   - Be professional, helpful, and concise
   - If you don't know the answer, say so and suggest who to contact

   Always cite your sources when referencing specific policies.
    ```

    ```prompt
   당신은 Contoso Corporation의 엔터프라이즈 지식 도우미입니다.

   역할:
   - 회사 정책과 절차에 대한 질문에 답합니다.
   - 업로드된 문서에서 정확한 정보를 제공합니다.
   - 전문적이고 유용하며 간결하게 응답합니다.
   - 답을 모르는 경우 그렇게 말하고 누구에게 문의해야 하는지 제안합니다.

   특정 정책을 참조할 때는 항상 출처를 인용하세요.
    ```

2. **Save**를 선택하여 현재 에이전트 구성을 저장합니다.

3. 샘플 정책 문서를 다운로드합니다. 새 브라우저 탭을 열고 각 파일을 저장합니다.

    **IT Security Policy:**

    ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/05a-m365-teams-integration/Python/sample_documents/it_security_policy.txt
    ```

    **Remote Work Policy:**

    ```
   https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/05a-m365-teams-integration/Python/sample_documents/remote_work_policy.txt
    ```

4. 에이전트 구성으로 돌아가 **Tools** 섹션까지 스크롤합니다.

5. **Upload files**를 선택합니다.

6. 파일을 첨부하는 팝업이 나타납니다. 이전에 다운로드한 파일을 첨부합니다.

7. 완료되면 **Attach**를 선택합니다.

## 플레이그라운드에서 에이전트 테스트

1. 플레이그라운드에서 IT 보안에 대해 질문합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What are the password requirements for my laptop?
    ```

    ```prompt
   내 노트북의 암호 요구 사항은 무엇인가요?
    ```

2. 에이전트는 IT 보안 정책의 구체적인 정보(최소 12자, 대문자, 소문자, 숫자, 특수 문자 등)를 제공해야 합니다.

3. 원격 근무에 대해 질문해 봅니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What are the core hours for remote employees?
    ```

    ```prompt
   원격 직원의 핵심 근무 시간은 언제인가요?
    ```

4. 에이전트는 원격 근무 정책의 정보(오전 9시~오후 3시)로 응답해야 합니다.

5. 다른 쿼리를 시도합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What encryption is required on company laptops?
    ```

    ```prompt
   회사 노트북에는 어떤 암호화가 필요하나요?
    ```

6. 에이전트가 올바른 문서를 찾아 BitLocker 요구 사항에 대한 정확한 답변을 제공하는 방식을 확인합니다.

    이제 에이전트는 지식 기반 그라운딩을 갖추었으며 회사 문서를 기반으로 질문에 답할 수 있습니다.

7. **Save**를 선택합니다.

## Microsoft Teams에 게시

이제 직원들이 Teams에서 직접 에이전트와 채팅할 수 있도록 에이전트를 Microsoft Teams에 게시합니다. Teams에 게시하면 Foundry 포털이 자동으로 다음을 수행합니다.

- Azure Bot Service를 만듭니다.
- Teams 앱 매니페스트를 생성합니다.
- 앱 아이콘과 구성을 패키징합니다.
- 다운로드 가능한 앱 패키지를 제공합니다.

### 앱 정보 준비

게시하기 전에 다음 정보를 수집합니다.

| Field | Value |
|-------|-------|
| **App Name** | Enterprise Knowledge Agent |
| **Short Description** | AI assistant for company policies |
| **Full Description** | Enterprise AI assistant that answers questions about company policies, IT procedures, and employee resources |
| **Developer Name** | Your name or company name |
| **Website URL** | <https://contoso.com> (placeholder is fine for lab) |
| **Privacy Policy URL** | <https://contoso.com/privacy> |
| **Terms of Use URL** | <https://contoso.com/terms> |

### 앱 아이콘 만들기

Teams 앱에는 아이콘 두 개가 필요합니다.

1. **Color icon**(192x192픽셀)
   - 앱 로고의 전체 색상 버전
   - PNG 형식

2. **Outline icon**(32x32픽셀)
   - 투명 배경의 흰색 윤곽선
   - PNG 형식
   - Teams 사이드바에서 사용됨

> **Quick option for this lab**: PowerPoint, Paint 또는 Canva 같은 온라인 도구를 사용하여 텍스트나 이니셜이 포함된 단순한 색상 사각형을 만듭니다.

### 포털에서 게시

1. Foundry 포털에서 에이전트를 엽니다(**Build** → **Agents** → **enterprise-knowledge-agent**).

2. 페이지 위쪽에서 **Publish** 단추를 선택합니다.

3. **Publish to Teams and Microsoft 365 Copilot**을 선택합니다.

4. **Continue**를 선택합니다.

### Teams 앱 세부 정보 구성

구성 양식을 작성합니다.

**Basic Information:**

- **App Name**: Enterprise Knowledge Agent
- **Short Description**: AI assistant for company policies
- **Full Description**: Enterprise AI assistant that answers questions about company policies, IT procedures, and employee resources

**Developer Information:**

- **Developer Name**: Your name
- **Website**: <https://contoso.com>
- **Privacy Policy**: <https://contoso.com/privacy>
- **Terms of Use**: <https://contoso.com/terms>

**App Icons:**

- **color icon**(192x192 px)을 업로드합니다.
- **outline icon**(32x32 px)을 업로드합니다.

**App Scope:**

- 개별 채팅 액세스에는 **Personal**을 선택합니다.
- 필요에 따라 채널 액세스에는 **Team**을 선택합니다.

**Prepare Agent**를 선택합니다.

### Teams에 배포

에이전트 패키지가 준비되면(1~2분 소요) Teams에 배포할 수 있습니다.

1. 패키지가 준비되면 **Continue the in-product publishing flow**를 선택합니다.

2. 게시 범위를 선택합니다.
   - **Individual scope**: 에이전트가 Teams 에이전트 스토어의 "Your agents" 아래에 표시됩니다. 관리자 승인이 필요하지 않습니다. 개인 테스트에 가장 적합합니다.
   - **Organization (tenant) scope**: 에이전트가 모든 사용자에게 "Built by your org" 아래에 표시됩니다. 관리자 승인이 필요합니다.

3. 이 랩에서는 **Individual scope**를 선택합니다.

4. **Submit**을 선택합니다.

5. 게시가 완료될 때까지 기다립니다(성공 메시지가 표시됨).

> **직접 게시가 실패하는 경우의 대안**: 게시 대화 상자에서 **400** 오류가 반환되고 Microsoft 365 계정에 사용자 지정 앱을 게시할 권한이 있는 경우, 대신 **Download & customize** 탭을 열고 지침을 따릅니다.

6. 이제 에이전트를 Teams에서 사용할 수 있습니다. **Apps** → **Your agents** 아래에서 찾습니다.

### Teams에서 에이전트 테스트

1. 설치 후 에이전트 채팅이 열려야 합니다(또는 **Apps** → **Your agents** 아래에서 찾습니다).

2. 인사말을 보냅니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   Hello! What can you help me with?
    ```

    ```prompt
   안녕하세요! 무엇을 도와줄 수 있나요?
    ```

3. 지식 쿼리를 테스트합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What are the laptop password requirements?
    ```

    ```prompt
   노트북 암호 요구 사항은 무엇인가요?
    ```

4. 다른 질문을 시도합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What MFA methods are supported?
    ```

    ```prompt
   어떤 MFA 방법이 지원되나요?
    ```

5. 에이전트는 IT 보안 정책 문서의 정보로 응답해야 합니다.

**🎉 축하합니다!** 이제 에이전트를 Microsoft Teams에서 사용할 수 있습니다.

### Teams 배포 문제 해결

**Teams에서 에이전트를 찾을 수 없음(직접 게시 후):**

- Teams에서 **Apps** → **Your agents** 섹션을 확인합니다.
- 게시 후 에이전트가 표시될 때까지 1~2분 기다립니다.
- Foundry 포털에서 게시가 성공적으로 완료되었는지 확인합니다.

**앱을 업로드할 수 없음(수동 업로드):**

- manifest.zip 파일이 손상되지 않았는지 확인합니다(필요하면 다시 다운로드).
- Teams 관리자가 사용자 지정 앱 업로드를 사용하지 않도록 설정하지 않았는지 확인합니다.
- 아이콘 크기가 올바른지 확인합니다(192x192 및 32x32).

**에이전트가 응답하지 않음:**

- 설치 후 봇이 초기화될 때까지 30초 기다립니다.
- Azure Bot Service가 만들어졌는지 확인합니다(게시 중 표시됨).
- 먼저 Foundry 플레이그라운드에서 에이전트를 테스트합니다.

**응답이 일반적임(지식 없음):**

- 에이전트에서 파일 검색이 사용하도록 설정되어 있는지 확인합니다.
- 문서가 업로드되고 인덱싱되었는지 확인합니다.
- Foundry 플레이그라운드에서 지식 쿼리를 테스트합니다.

## Microsoft 365 Copilot에 게시

이제 에이전트를 Microsoft 365 Copilot 확장으로 게시하여 사용자가 Copilot 내에서 직접 액세스할 수 있도록 합니다. Copilot에 게시하면 에이전트가 **Copilot extension**(플러그 인 또는 선언적 에이전트라고도 함)이 됩니다. 사용자는 다음을 수행할 수 있습니다.

- Copilot에서 @멘션을 사용하여 에이전트를 호출합니다.
- Copilot의 기능과 함께 에이전트의 지식에 액세스합니다.
- Copilot과 에이전트 간에 원활하게 전환합니다.

> **Note**: 이 섹션에는 Microsoft 365 Copilot 라이선스가 필요합니다. 라이선스가 없으면 단계를 읽어 프로세스를 이해할 수 있습니다.

### 포털에서 게시

1. Foundry 포털(**<https://ai.azure.com>**)로 돌아갑니다.

2. 에이전트로 이동합니다(**Build** → **Agents** → **enterprise-knowledge-agent**).

3. **Publish** 단추를 선택합니다.

4. **Publish to Teams and Microsoft 365 Copilot**을 선택합니다.

5. **Continue**를 선택합니다.

> **Note**: 이는 Teams에 사용한 것과 동일한 게시 흐름입니다. 단일 게시 프로세스를 통해 에이전트를 Teams와 Copilot 모두에서 사용할 수 있게 됩니다.

### 게시 세부 정보 구성

아직 이 에이전트를 게시하지 않았다면 Teams 섹션과 동일하게 구성을 작성합니다.

- **Name**: Enterprise Knowledge Agent
- **Description**: AI assistant for company IT policies
- **Icons**: 192x192 및 32x32 아이콘을 업로드합니다.
- **Publisher information**: 사용자 이름 및 자리 표시자 URL

### 게시 범위 선택

배포 범위를 선택합니다.

| Scope | Visibility | Admin Approval | Best For |
|-------|-----------|----------------|----------|
| **Shared** | 에이전트 스토어의 "Your agents" 아래 | 필요하지 않음 | 개인 테스트, 소규모 팀 |
| **Organization** | 모든 사용자에게 "Built by your org" 아래 | 필요 | 조직 전체 배포 |

이 랩에서는 관리자 승인 없이 즉시 액세스할 수 있도록 **Shared scope**를 선택합니다.

### 게시 완료

1. **Prepare Agent**를 선택하고 패키징이 완료될 때까지 기다립니다(1~2분).

2. **Continue the in-product publishing flow**를 선택합니다.

3. 범위 선택을 확인하고 **Publish**를 선택합니다.

4. 게시가 완료될 때까지 기다립니다.

### Microsoft 365 Copilot에서 액세스

공유 범위로 게시되면 에이전트를 즉시 사용할 수 있습니다.

1. **Microsoft 365 Copilot**(copilot.microsoft.com 또는 Microsoft 365 앱 내)을 엽니다.

2. 에이전트 스토어 또는 **Extensions** 패널을 찾습니다.

3. **Your agents** 아래에서 에이전트를 찾습니다(공유 범위의 경우).

4. 대화를 시작합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   @Enterprise Knowledge Agent What are the laptop security requirements?
    ```

    ```prompt
   @Enterprise Knowledge Agent 노트북 보안 요구 사항은 무엇인가요?
    ```

5. 또는 에이전트를 선택하고 직접 질문합니다.

    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.

    ```
   What MFA methods are supported for company systems?
    ```

    ```prompt
   회사 시스템에서는 어떤 MFA 방법이 지원되나요?
    ```

6. Copilot은 쿼리를 에이전트로 라우팅하고 IT 보안 정책의 정보를 반환합니다.

> **Note**: **organization scope**의 경우 관리자가 먼저 [Microsoft 365 관리 센터](https://admin.cloud.microsoft/?#/agents/all/requested)의 **Requests** 아래에서 앱을 승인해야 합니다. 승인되면 에이전트가 모든 사용자에게 **Built by your org** 아래에 표시됩니다.

## 정리

불필요한 요금이 발생하지 않도록 완료 후 리소스를 정리합니다.

### 에이전트 삭제

1. Foundry 포털에서 **Build** → **Agents**로 이동합니다.

2. **enterprise-knowledge-agent**를 찾습니다.

3. **...** 메뉴 → **Delete**를 선택합니다.

4. 삭제를 확인합니다.

이 작업은 다음도 제거합니다.

- Azure Bot Service
- 연결된 구성
- 게시된 배포

### Teams에서 제거

1. Microsoft Teams를 엽니다.

2. **Apps** → **Manage your apps**로 이동합니다.

3. **Enterprise Knowledge Agent**를 찾습니다.

4. **...** → **Uninstall**을 선택합니다.

5. 제거를 확인합니다.

### Copilot 확장 제거

Copilot에 게시한 경우:

1. 에이전트가 삭제되면 확장이 비활성 상태가 됩니다.
2. 사용자가 이를 사용하려고 하면 오류가 표시됩니다.
3. 관리자가 조직 카탈로그에서 이를 제거해야 할 수 있습니다.
