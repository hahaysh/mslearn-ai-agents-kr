---
title: '작업 3 – 에이전트를 Microsoft 365 Copilot에 게시'
lab:
    title: '작업 3 – 에이전트를 Microsoft 365 Copilot에 게시'
    description: '직원이 Copilot 안에서 접근할 수 있도록 Caldova 지식 에이전트를 Microsoft 365 Copilot에 게시합니다.'
    type: 'task'
    parent: 'B'
    order: 3
    section: 'optional'
    difficulty: 2
    duration: 15
    access: 'gated'
    requires: 'A Microsoft 365 Copilot licence, and permission to publish agents to Copilot in your tenant'
    verify: 'In the Foundry portal, open your agent and select **Publish**. If Microsoft 365 Copilot is greyed out or returns a consent error, you don''t have the rights this task needs.'
    level: 200
    concepts: '에이전트 게시, Microsoft 365 Copilot, Copilot 에이전트'
    status: 'draft'
---

# 작업 3 — 에이전트를 Microsoft 365 Copilot에 게시

*이 작업은 **엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합** 랩의 일부입니다. 처음 오셨나요? [시작하기](B0-getting-started.md)부터 시작하세요.*

<!-- BEGIN GENERATED: gated-notice - do not edit by hand; run: python tools/generate_lab_blocks.py -->

> ### 시작하기 전에 액세스 확인
>
> **This task needs:** A Microsoft 365 Copilot licence, and permission to publish agents to Copilot in your tenant.
>
> Foundry 포털에서 에이전트를 열고 **Publish**를 선택합니다. Microsoft 365 Copilot이 회색으로 표시되거나 동의 오류가 반환되면 이 작업에 필요한 권한이 없는 것입니다.
>
> **Don't have it?** 이 작업을 건너뜁니다. 이 랩의 다른 어떤 내용도 이 작업에 의존하지 않으며, 작동 방식을 알아보기 위해 단계를 읽어볼 수는 있습니다.

<!-- END GENERATED: gated-notice -->

> **Set up (start here):** 이 작업은 [작업 1](B1-create-a-foundry-iq-knowledge-agent.md)의 기반화된 `caldova-knowledge-agent`를 게시합니다. 아직 해당 에이전트가 없다면 먼저 작업 1을 완료합니다. 또는 `python ../setup/bootstrap_agent.py`를 VS Code에서 연 `Python` 폴더에서 실행하여 코드로 만들고 기반화합니다. 이 작업은 포털과 Copilot에서만 완료되므로 로컬 코드나 `.env` 파일은 필요하지 않습니다.

> **Continuing from a previous task?** [작업 2](B2-publish-to-microsoft-teams.md)에서 이미 Teams에 게시했다면 동일한 게시 흐름으로 Copilot에서도 에이전트를 사용할 수 있습니다. 아래의 **Microsoft 365 Copilot에 게시**로 바로 이동하세요.

---

**Microsoft 365 Copilot**에 게시하면 에이전트가 직원이 Copilot 안에서 직접 사용할 수 있는 **Copilot 에이전트**가 됩니다. 이 작업은 **게시 워크플로**에 중점을 둡니다. 코드는 작성하지 않습니다.

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
<summary>Microsoft 365 Copilot 에이전트란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

Copilot에 게시하면 에이전트는 **Copilot 에이전트**(확장 또는 선언적 에이전트라고도 함)가 됩니다. 직원은 Copilot에서 **@멘션**으로 에이전트를 호출하고, Copilot 자체 기능과 함께 에이전트의 지식에 액세스하며, Copilot과 에이전트 사이를 원활하게 전환할 수 있습니다.

</div>
</details>

## Microsoft 365 Copilot에 게시

Copilot에 게시하면 사용자는 다음을 수행할 수 있습니다.

- Copilot에서 @멘션을 사용하여 에이전트 호출
- Copilot 기능과 함께 에이전트의 지식에 액세스
- Copilot과 에이전트 간 원활하게 전환

### 포털에서 게시

1. Foundry 포털(**<https://ai.azure.com>**)로 돌아갑니다.

2. 에이전트로 이동합니다(**Build** → **Agents** → **caldova-knowledge-agent**).

3. **Publish** 단추를 선택합니다.

4. **Publish to Teams and Microsoft 365 Copilot**을 선택합니다.

5. **Continue**를 선택합니다.

> **Note**: 이는 Teams에 사용되는 것과 동일한 게시 흐름입니다. 단일 게시 프로세스를 통해 에이전트를 Teams와 Copilot 모두에서 사용할 수 있게 됩니다.

### 게시 세부 정보 구성

아직 이 에이전트를 게시하지 않았다면 구성 정보를 입력합니다(Teams 섹션과 동일).

- **Name**: Caldova Knowledge Assistant
- **Description**: AI assistant for Caldova staff
- **Icons**: 192x192 및 32x32 아이콘 업로드
- **Publisher information**: 사용자 이름과 자리 표시자 URL

### 게시 범위 선택

배포 범위를 선택합니다.

| 범위 | 표시 위치 | 관리자 승인 | 적합한 용도 |
|-------|-----------|----------------|----------|
| **Shared** | 에이전트 스토어의 "Your agents" 아래 | 필요 없음 | 개인 테스트, 소규모 팀 |
| **Organization** | 모든 사용자의 "Built by your org" 아래 | 필요 | 조직 전체 배포 |

이 랩에서는 관리자 승인 없이 즉시 액세스할 수 있도록 **Shared scope**를 선택합니다.

### 게시 완료

1. **Prepare Agent**를 선택하고 패키징이 완료될 때까지 기다립니다(1~2분).

2. **Continue the in-product publishing flow**를 선택합니다.

3. 범위 선택을 확인하고 **Publish**를 선택합니다.

4. 게시가 완료될 때까지 기다립니다.

### Microsoft 365 Copilot에서 액세스

공유 범위로 게시되면 에이전트를 즉시 사용할 수 있습니다.

1. **Microsoft 365 Copilot**을 엽니다(copilot.microsoft.com 또는 Microsoft 365 앱에서).

2. 에이전트 스토어 또는 **Extensions** 패널을 찾습니다.

3. **Your agents** 아래에서 에이전트를 찾습니다(공유 범위의 경우).

4. 대화를 시작합니다.

    ```
    @Caldova Knowledge Assistant How much headroom does Calderwood have?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    @Caldova Knowledge Assistant Calderwood에는 여유 용량이 얼마나 있나요?
    ```

5. 또는 에이전트를 선택하고 직접 질문합니다.

    ```
    When should we reorder sterile vials, and who is our component supplier?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    무균 바이알은 언제 재주문해야 하며 당사의 구성품 공급업체는 누구인가요?
    ```

6. Copilot이 쿼리를 에이전트로 라우팅하고 Caldova 지식 베이스의 정보를 반환합니다.

> **Note**: **organization scope**의 경우 관리자가 먼저 [Microsoft 365 admin center](https://admin.cloud.microsoft/?#/agents/all/requested)의 **Requests** 아래에서 앱을 승인해야 합니다. 승인되면 에이전트가 모든 사용자의 **Built by your org** 아래에 표시됩니다.

> ✅ **Checkpoint**: 이제 기반화된 지식 에이전트를 Microsoft 365 Copilot 안에서 사용할 수 있으며, 엔터프라이즈 지식 베이스를 기반으로 직원 질문에 답합니다.

## 정리

불필요한 요금을 방지하려면 완료 후 리소스를 정리합니다.

### 에이전트 삭제

1. Foundry 포털에서 **Build** → **Agents**로 이동합니다.

2. **caldova-knowledge-agent**를 찾습니다.

3. **...** 메뉴 → **Delete**를 선택합니다.

4. 삭제를 확인합니다.

그러면 다음도 제거됩니다.

- Azure Bot Service
- 연결된 구성
- 게시된 배포

### Teams에서 제거

1. Microsoft Teams를 엽니다.

2. **Apps** → **Manage your apps**로 이동합니다.

3. **Caldova Knowledge Assistant**를 찾습니다.

4. **...** → **Uninstall**을 선택합니다.

5. 제거를 확인합니다.

### Copilot 에이전트 제거

Copilot에 게시한 경우:

1. 기본 에이전트가 삭제되면 에이전트가 비활성 상태가 됩니다.
2. 사용자가 이를 사용하려고 하면 오류가 표시됩니다.
3. 관리자가 조직 카탈로그에서 제거해야 할 수 있습니다.

---

**다음(선택 사항):** [작업 4 — Work IQ: Microsoft 365 신호를 에이전트에 가져오기](B4-work-iq-workplace-intelligence.md)

