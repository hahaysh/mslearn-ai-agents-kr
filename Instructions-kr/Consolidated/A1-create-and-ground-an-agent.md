---
title: '작업 1 – 에이전트 만들기 및 그라운딩'
lab:
    title: '작업 1 – 에이전트 만들기 및 그라운딩'
    description: 'Microsoft Foundry 포털에서 에이전트를 만들고 Caldova 공급망 정책으로 그라운딩하여 데이터에 기반해 답변하도록 합니다.'
    type: 'task'
    parent: 'A'
    order: 1
    section: 'core'
    difficulty: 2
    duration: 15
    access: 'open'
    level: 200
    concepts: '에이전트 생성, 그라운딩, 파일 검색'
    status: 'draft'
---

# 작업 1 — 에이전트 만들기 및 그라운딩

***AI 에이전트 빌드 및 확장** 랩의 일부입니다. 처음 오셨나요? [시작하기](A0-getting-started.md)부터 진행합니다.*

> **What you need:** **배포된 모델이 있는 Microsoft Foundry 프로젝트**가 필요합니다.
> 아직 없나요? 먼저 [시작하기](A0-getting-started.md)를 완료합니다(옵션 A는 포털에서
> 프로젝트를 만듭니다). 이 작업은 전적으로 포털에서 완료하므로 로컬 코드나 `.env`
> 파일이 필요하지 않으며, 다른 작업에서 이어받을 항목도 없습니다.

---

그라운딩은 에이전트가 추측하지 않고 정확하게 답변하도록 신뢰할 수 있는 원본 자료를
제공합니다.

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
<summary>그라운딩이란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

Caldova 에이전트에서 가장 중요한 단일 기능은 **그라운딩**입니다.
그라운딩은 공급망 정책 문서와 같은 신뢰할 수 있는 원본 자료를 연결하여, 에이전트가
응답을 지어내는 대신 *해당 데이터에 기반해* 답변하도록 합니다.

정책 세부 정보를 에이전트 지침에 복사할 수도 있지만, 정책이 바뀌면 유지 관리가
어려워집니다. 파일 검색을 사용하면 지침은 에이전트의 역할에 집중하고, 정책은 교체하거나
업데이트할 수 있는 별도의 원본으로 유지할 수 있습니다.

[자세히 알아보기 →](https://review.learn.microsoft.com/en-us/training/modules/build-extend-ai-agents/2-understand-agents-foundry?branch=pr-en-us-55509)

</div>
</details>

1. 에이전트 플레이그라운드에서 **Instructions**를 다음으로 설정합니다.

    ```prompt
    You are the Caldova supply chain assistant.
    You help planning and materials teams with questions about capacity, contract manufacturers, and materials.

    Guidelines:
    - Always be friendly and helpful
    - Use the supply chain policy documentation to answer questions accurately
    - If you don't know the answer, admit it and suggest contacting the support team directly
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    당신은 Caldova 공급망 도우미입니다.
    계획 팀과 자재 팀이 생산 능력, 계약 제조업체, 자재에 관한 질문을 해결하도록 돕습니다.

    지침:
    - 항상 친절하고 도움이 되도록 응답합니다
    - 공급망 정책 문서를 사용하여 질문에 정확하게 답변합니다
    - 답을 모르면 모른다고 인정하고 지원 팀에 직접 문의하라고 제안합니다
    ```

1. 샘플 공급망 정책 문서를 다운로드합니다. 새 브라우저 탭을 열고 다음으로 이동합니다.

    ```
    https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-agents/main/Labfiles/A-build-and-extend-ai-agents/Python/Supply_Chain_Policy.txt
    ```

    파일을 로컬 컴퓨터에 저장합니다.

1. 플레이그라운드로 돌아가 **도구(Tools)** 섹션에서 **추가(Add)**를 선택하고 **파일 검색(File search)**을 추가합니다.

1. **Add** 오른쪽에서 **Upload files**를 선택하고, 다운로드한 `Supply_Chain_Policy.txt` 파일을 찾아 **Attach**를 선택합니다. 파일의 인덱싱이 끝날 때까지 기다립니다.

1. 에이전트를 **Save**합니다.

### 그라운딩된 에이전트 테스트

1. 채팅 창에 다음을 입력합니다.

    ```
    How long does review take for a standard capacity request?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    표준 생산 능력 요청의 검토에는 얼마나 걸리나요?
    ```

    에이전트는 답변에서 공급망 정책 문서를 참조해야 합니다.

1. 그라운딩 데이터를 사용하는지 확인하기 위해 두 번째 질문을 시도합니다.

    ```
    How much is five weeks of premium contract capacity at expedited priority?
    ```
    다음 영문 또는 한국어 프롬프트 중 하나를 입력합니다.
    ```prompt
    긴급 우선순위로 프리미엄 계약 생산 능력 5주분은 얼마인가요?
    ```

> ✅ **Checkpoint**: 에이전트가 업로드한 정책 문서를 사용하여 공급망 질문에 답변합니다.
> 포털에서만 에이전트를 만들고 그라운딩했습니다.

---

**다음:** [작업 2 — 원격 MCP 서버 연결](A2-connect-a-remote-mcp-server.md)
