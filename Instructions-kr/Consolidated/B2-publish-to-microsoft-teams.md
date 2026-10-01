---
title: '작업 2 – 에이전트를 Microsoft Teams에 게시'
lab:
    title: '작업 2 – 에이전트를 Microsoft Teams에 게시'
    description: '직원들이 이미 일하는 위치에서 채팅할 수 있도록 Caldova 지식 에이전트를 Microsoft Teams에 게시합니다.'
    type: 'task'
    parent: 'B'
    order: 2
    section: 'optional'
    difficulty: 2
    duration: 20
    access: 'gated'
    requires: 'A Microsoft 365 account with Teams access, and permission to publish agents to Teams in your tenant'
    verify: 'In the Foundry portal, open your agent and select **Publish**. If Teams is greyed out or returns a consent error, you don''t have the rights this task needs.'
    level: 200
    concepts: '에이전트 게시, Microsoft Teams, Azure Bot Service'
    status: 'draft'
---

# 작업 2 — 에이전트를 Microsoft Teams에 게시

*이 작업은 **엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합** 랩의 일부입니다. 처음 오셨나요? [시작하기](B0-getting-started.md)부터 시작하세요.*

<!-- BEGIN GENERATED: gated-notice - do not edit by hand; run: python tools/generate_lab_blocks.py -->

> ### 시작하기 전에 액세스 확인
>
> **This task needs:** A Microsoft 365 account with Teams access, and permission to publish agents to Teams in your tenant.
>
> Foundry 포털에서 에이전트를 열고 **Publish**를 선택합니다. Teams가 회색으로 표시되거나 동의 오류가 반환되면 이 작업에 필요한 권한이 없는 것입니다.
>
> **Don't have it?** 이 작업을 건너뜁니다. 이 랩의 다른 어떤 내용도 이 작업에 의존하지 않으며, 작동 방식을 알아보기 위해 단계를 읽어볼 수는 있습니다.

<!-- END GENERATED: gated-notice -->

> **Set up (start here):** 이 작업은 [작업 1](B1-create-a-foundry-iq-knowledge-agent.md)의 기반화된 `caldova-knowledge-agent`를 게시합니다. 아직 해당 에이전트가 없다면 먼저 작업 1을 완료합니다. 또는 가장 빠른 경로로, `python ../setup/bootstrap_agent.py`를 VS Code에서 연 `Python` 폴더에서 실행하여 코드로 만들고 기반화합니다. 이 작업은 포털과 Teams에서만 완료되므로 로컬 코드나 `.env` 파일은 필요하지 않습니다.

> **Continuing from a previous task?** 작업 1을 막 완료했고 `caldova-knowledge-agent`가 Foundry 포털에서 기반화되어 저장되어 있다면 준비가 된 것입니다. 아래의 **Microsoft Teams에 게시**로 바로 이동하세요.

---

**Microsoft Teams**에 게시하면 Caldova 직원이 이미 사용하는 도구를 떠나지 않고 Teams에서 직접 지식 도우미와 채팅할 수 있습니다. 이 작업은 **배포 및 게시 워크플로**에 중점을 둡니다. 코드는 작성하지 않습니다.

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
</style>

<details markdown="1" class="concept">
<summary>Teams에 게시하면 어떤 일이 발생하나요?</summary>
<div class="concept-body" markdown="1">

에이전트를 Teams에 게시하면 Foundry 포털이 자동으로 **Azure Bot Service를 만들고**, **Teams 앱 매니페스트를 생성**하고, **앱 아이콘 및 구성을 패키지**하고, **다운로드 가능한 앱 패키지**를 제공합니다. 이 모든 것을 직접 만들 필요는 없습니다. 짧은 양식을 작성하면 포털이 연결을 구성합니다.

</div>
</details>

## Microsoft Teams에 게시

Teams에 게시하면 Foundry 포털이 자동으로 다음을 수행합니다.

- Azure Bot Service 만들기
- Teams 앱 매니페스트 생성
- 앱 아이콘 및 구성 패키지
- 다운로드 가능한 앱 패키지 제공

### 앱 정보 준비

게시하기 전에 다음 정보를 준비합니다.

| 필드 | 값 |
|-------|-------|
| **App Name** | Caldova Knowledge Assistant |
| **Short Description** | AI assistant for Caldova staff |
| **Full Description** | Enterprise AI assistant that answers staff questions about plant capacity, site operations, contract manufacturers, and suppliers |
| **Developer Name** | Your name or company name |
| **Website URL** | <https://caldova.example> (랩에서는 자리 표시자로 충분함) |
| **Privacy Policy URL** | <https://caldova.example/privacy> |
| **Terms of Use URL** | <https://caldova.example/terms> |

### 앱 아이콘 만들기

Teams 앱에는 두 개의 아이콘이 필요합니다.

1. **Color icon**(192x192픽셀)
   - 앱 로고의 전체 색상 버전
   - PNG 형식

2. **Outline icon**(32x32픽셀)
   - 투명 배경의 흰색 윤곽선
   - PNG 형식
   - Teams 사이드바에서 사용됨

> **Quick option for this lab**: PowerPoint, Paint 또는 Canva와 같은 온라인 도구를 사용하여 텍스트나 이니셜이 있는 간단한 색상 사각형을 만듭니다.

### 포털에서 게시

1. Foundry 포털에서 에이전트를 엽니다(**Build** → **Agents** → **caldova-knowledge-agent**).

2. 페이지 위쪽에서 **Publish** 단추를 선택합니다.

3. **Publish to Teams and Microsoft 365 Copilot**을 선택합니다.

4. **Continue**를 선택합니다.

### Teams 앱 세부 정보 구성

구성 양식을 작성합니다.

**Basic Information:**

- **App Name**: Caldova Knowledge Assistant
- **Short Description**: AI assistant for Caldova staff
- **Full Description**: Enterprise AI assistant that answers staff questions about plant capacity, site operations, contract manufacturers, and suppliers

**Developer Information:**

- **Developer Name**: Your name
- **Website**: <https://caldova.example>
- **Privacy Policy**: <https://caldova.example/privacy>
- **Terms of Use**: <https://caldova.example/terms>

**App Icons:**

- **color icon**(192x192 px)을 업로드합니다.
- **outline icon**(32x32 px)을 업로드합니다.

**App Scope:**

- 개별 채팅 액세스에는 **Personal**을 선택합니다.
- 선택 사항으로 채널 액세스에는 **Team**을 선택합니다.

**Prepare Agent**를 선택합니다.

### Teams에 배포

에이전트 패키지가 준비된 후(1~2분 소요) Teams에 배포할 수 있습니다.

1. 패키지가 준비되면 **Continue the in-product publishing flow**를 선택합니다.

2. 게시 범위를 선택합니다.
   - **Individual scope**: 에이전트가 Teams 에이전트 스토어의 "Your agents" 아래에 표시됩니다. 관리자 승인이 필요 없습니다. 개인 테스트에 가장 적합합니다.
   - **Organization (tenant) scope**: 에이전트가 모든 사용자의 "Built by your org" 아래에 표시됩니다. 관리자 승인이 필요합니다.

3. 이 랩에서는 **Individual scope**를 선택합니다.

4. **Submit**을 선택합니다.

5. 게시가 완료될 때까지 기다립니다(성공 메시지가 표시됩니다).

> **Alternative if direct publishing fails**: 게시 대화 상자가 **400** 오류를 반환하고 Microsoft 365 계정에 사용자 지정 앱 게시 권한이 있는 경우, 대신 **Download & customize** 탭을 열고 지침을 따릅니다.

6. 이제 에이전트를 Teams에서 사용할 수 있습니다! **Apps** → **Your agents** 아래에서 찾습니다.

### Teams에서 에이전트 테스트

1. 설치 후 에이전트 채팅이 열려야 합니다. 또는 **Apps** → **Your agents** 아래에서 찾습니다.

2. 인사말을 보냅니다.

    ```
    Hello! What can you help me with?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    안녕하세요! 무엇을 도와줄 수 있나요?
    ```

3. 지식 쿼리를 테스트합니다.

    ```
    How much headroom does Calderwood have?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    Calderwood에는 여유 용량이 얼마나 있나요?
    ```

4. 다른 질문을 시도합니다.

    ```
    What are our site core hours?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    당사 사이트의 핵심 운영 시간은 어떻게 되나요?
    ```

5. 에이전트가 Caldova 지식 베이스의 정보로 응답해야 합니다!

> ✅ **Checkpoint**: 이제 기반화된 지식 에이전트를 Microsoft Teams에서 사용할 수 있으며, 엔터프라이즈 지식 베이스를 기반으로 직원 질문에 답합니다.

### Teams 배포 문제 해결

**Teams에서 에이전트를 찾을 수 없음(직접 게시 후):**

- Teams의 **Apps** → **Your agents** 섹션을 확인합니다.
- 게시 후 에이전트가 나타날 때까지 1~2분 기다립니다.
- Foundry 포털에서 게시가 성공적으로 완료되었는지 확인합니다.

**앱을 업로드할 수 없음(수동 업로드):**

- Teams 관리자가 사용자 지정 앱 업로드를 사용하지 않도록 설정하지 않았는지 확인합니다.
- 아이콘 크기가 올바른지 확인합니다(192x192 및 32x32).

**에이전트가 응답하지 않음:**

- 설치 후 봇이 초기화될 때까지 30초 기다립니다.
- Azure Bot Service가 만들어졌는지 확인합니다(게시 중에 표시됨).
- 먼저 Foundry 플레이그라운드에서 에이전트를 테스트합니다.

**응답이 일반적임(지식 없음):**

- Foundry IQ(또는 File Search)가 에이전트에서 사용하도록 설정되어 있는지 확인합니다.
- 문서가 업로드되고 인덱싱되었는지 확인합니다.
- Foundry 플레이그라운드에서 지식 쿼리를 테스트합니다.

---

**다음(선택 사항):** [작업 3 — Microsoft 365 Copilot에 게시](B3-publish-to-microsoft-365-copilot.md) · [작업 4 — Work IQ](B4-work-iq-workplace-intelligence.md)

