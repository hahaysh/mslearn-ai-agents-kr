---
title: 'Agent Framework로 다중 에이전트 솔루션 빌드'
lab:
    title: 'Agent Framework로 다중 에이전트 솔루션 빌드'
    description: 'Microsoft Agent Framework로 Caldova 운영 에이전트를 빌드합니다. 먼저 도구를 사용하는 단일 에이전트로 시작한 다음, 여러 에이전트를 순차적으로 오케스트레이션하고, 마지막으로 A2A 프로토콜을 사용해 프로세스 간 원격 에이전트를 연결합니다. 처음부터 끝까지 또는 작업별로 완료할 수 있는 모듈식 랩입니다.'
    type: 'lab'
    id: 'C'
    order: 3
    difficulty: 3
    duration: 30
    access: 'open'
    level: 300
    concepts: 'Microsoft Agent Framework, 도구, 다중 에이전트 오케스트레이션, A2A 프로토콜'
    islab: true
    status: 'draft'
---

# Agent Framework로 다중 에이전트 솔루션 빌드

**수준** ▰▰▰▱▱ **L300**  (**L100** 초급 → **L500** 전문가)

단일 에이전트도 유용합니다. 각자 특정 영역에 집중하고 서로에게 작업을 넘길 수 있는 에이전트 *팀*이야말로 실제 운영을 구축하는 방식입니다. 이 랩에서는 **Microsoft Agent Framework (MAF)**를 사용해 Caldova 다중 에이전트 시스템을 단계적으로 빌드합니다. 도구를 사용하는 하나의 에이전트에서 시작해, 프로토콜을 통해 서로 호출하는 원격 에이전트 집합으로 확장합니다.

![Anton](../Media/anton-avatar.png)<br /><strong>AI 가이드 Anton을 만나 보세요.</strong><br />이 랩 전체에서 **Ask Anton** 팁을 볼 수 있습니다. 더 대화형의 실습 도움말이 필요하신가요? *[Ask Anton](https://aka.ms/choose-anton)* 앱에서 Anton과 채팅하세요.

<details>
<summary><strong><i>Ask Anton 앱 정보</i></strong></summary>

<strong><i><a href="https://aka.ms/choose-anton" target="_blank">Ask Anton</a></i></strong>은 AI 개념과 Microsoft Foundry 기술에 관한 질문에 답할 수 있는 생성형 AI 에이전트입니다. <code>https://aka.ms/choose-anton</code>에서 두 가지 버전으로 사용할 수 있습니다.
<ul>
<li><strong>Azure 기반</strong>: 최상의 환경 <i>(Azure 구독과 Foundry 프로젝트의 모델 배포가 필요함)</i>.</li>
<li><strong>브라우저 기반</strong>: 브라우저에서 작은 언어 모델 사용 <i>(기능이 제한됨 - 이전 또는 사양이 낮은 디바이스에서는 느리거나 "basic" 모드에서만 작동할 수 있음)</i>.</li>
</ul>
<blockquote><i>Ask Anton은 Microsoft Learn 또는 AI Skills Navigator의 지원되는 Microsoft 제품이나 구성 요소가 <u>아닙니다</u>.</i></blockquote>
</details>

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
<summary>Microsoft Agent Framework란?</summary>
<div class="concept-body" markdown="1">

**Microsoft Agent Framework (MAF)**는 Microsoft Foundry에서 에이전트를 빌드하기 위한 상위 수준 SDK입니다. 일반 Python 함수에 `@tool`을 데코레이트하면 스키마가 자동으로 생성되고, `await agent.run(...)`을 호출하면 전체 도구 호출 루프가 자동으로 실행됩니다. 또한 여러 에이전트를 함께 실행하는 오케스트레이션 같은 **다중 에이전트** 솔루션용 빌딩 블록을 제공하므로 배관 작업을 직접 연결할 필요가 없습니다.

[자세히 알아보기 →](https://learn.microsoft.com/azure/ai-foundry/agents/overview)

</div>
</details>

**시나리오:** 사용자는 가속화된 제품 출시를 준비하는 제약 제조업체 **Caldova**에서 일합니다. 이 랩 전체에서 Caldova 운영의 자동화를 빌드합니다. 사이트 방문 경비 청구를 제출하는 단일 에이전트로 시작해, 사이트 피드백을 분류하는 에이전트 파이프라인으로 확장하고, 마지막으로 별도 프로세스에서 실행되며 프로토콜을 통해 협업하는 전문 이전 계획 에이전트 집합을 만듭니다.

가능한 한 빠르게 작동하는 도구 사용 에이전트에 도달하는 **Core** 작업부터 시작합니다. 그다음 **Optional** 작업 집합을 통해 다중 에이전트 패턴을 더 깊이 살펴볼 수 있습니다.

> **Note**: 이 연습에서 사용하는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다.
> 예상치 못한 동작, 경고 또는 오류가 발생할 수 있습니다.

## 학습할 내용

이 연습의 **Core** 작업을 완료하면 다음을 수행할 수 있습니다.

- Microsoft Agent Framework를 사용해 **사용자 지정 도구가 있는 에이전트 빌드** — Python 함수에 `@tool`을 데코레이트하고, 이를 `Agent`에 전달한 뒤, `agent.run()`이 도구 호출 루프를 진행하도록 합니다.

**Optional** 작업을 통해 추가로 다음을 수행할 수 있습니다.

- 여러 에이전트를 순서대로 실행하여 한 전문 에이전트에서 다음 에이전트로 작업을 전달하고 모든 에이전트의 출력을 수집하는 방식으로 **여러 에이전트 오케스트레이션**.
- 별도 프로세스에서 실행되고 라우팅 에이전트가 조정하는 **Agent-to-Agent (A2A)** 프로토콜을 사용해 서로 호출하는 **원격 에이전트 연결**.
- 한 에이전트의 **구조화된 출력**을 자체 코드의 조건부 라우팅으로 전환하여 지원 티켓 **분류 및 라우팅**.

## 이 랩의 구성

이 랩은 **모듈식**입니다. 각 작업은 **처음부터 새로 시작해 단독으로 완료**할 수 있도록 작성되어 있으므로, 원하는 작업 하나만 선택해 진행할 수 있습니다. 또한 모든 작업은 하나의 시작 폴더, 하나의 가상 환경, 하나의 `.env`를 공유하므로 처음부터 끝까지 이어서 작업할 수도 있습니다.

1. **[시작하기](C0-getting-started.md)부터 시작합니다** — Microsoft Foundry 프로젝트를 만들고(포털 또는 하나의 `azd up` 명령 사용), 시작 코드를 가져오고, `.env`를 설정합니다. 모든 작업은 여기에서 시작합니다. 전체 랩을 한 번에 진행한다면 이 작업은 한 번만 수행하면 됩니다.
2. **원하는 작업을 수행합니다.** 각 작업에는 독립적으로 시작하는 데 필요한 설정이 나열되어 있습니다. 이전 작업에서 바로 이어서 진행하는 경우, 위쪽의 짧은 *"이전 작업에서 계속하나요?"* 참고를 통해 반복 설정을 건너뛰고 계속 진행할 수 있습니다.

## 랩 한눈에 보기

먼저 **Core** 작업을 완료합니다. 이 작업이 끝나면 작동하는 도구 사용 에이전트가 만들어집니다. 그런 다음 관심 있는 **Optional** 작업을 확장합니다.

<!-- BEGIN GENERATED: task-table - do not edit by hand; run: python tools/generate_lab_blocks.py -->

| 섹션 | 작업 | 수준 | 시간 |
| --- | --- | --- | --- |
| **Core** | [작업 1 – 도구가 있는 에이전트 빌드](C1-create-an-agent-with-a-tool.md) | ▰▰▰▱▱ L300 | 약 30분 |
| *Optional* | [작업 2 – 여러 에이전트를 순서대로 오케스트레이션](C2-orchestrate-multiple-agents.md) | ▰▰▰▱▱ L300 | 약 30분 |
| *Optional* | [작업 3 – A2A로 원격 에이전트 연결](C3-connect-remote-agents-with-a2a.md) | ▰▰▰▰▱ L400 | 약 30분 |
| *Optional* | [작업 4 – 지원 티켓 분류 및 라우팅](C4-classify-and-route-a-ticket.md) | ▰▰▰▱▱ L300 | 약 30분 |

**Core 작업:** 약 **30분**. 모든 선택 작업을 포함한 **전체 랩**: 약 **2시간**.

<!-- END GENERATED: task-table -->

**경로 선택** — 보유한 시간에 맞는 작업을 선택합니다.

- **Core만(~99분):** 작업 1을 수행합니다.
- **Core + 패턴 하나(~1시간):** **작업 2**(순차 오케스트레이션) 또는 **작업 4**(분류 + 라우팅)를 추가합니다.
- **전체(~2시간):** **작업 2**, **작업 3**(A2A를 사용하는 원격 에이전트), **작업 4**를 추가합니다.

## 하나의 프레임워크, 하나의 에이전트에서 여러 에이전트로 확장

이 랩의 모든 작업은 **Microsoft Agent Framework**를 기반으로 하므로 솔루션이 더 야심 차게 발전해도 코드의 형태는 익숙하게 유지됩니다.

- **작업 1**에서는 **단일** 에이전트를 빌드합니다. `@tool`로 도구를 설명하고, `Agent`에 연결한 다음, 이 에이전트는 `FoundryChatClient`가 지원하며, `agent.run(...)`을 호출합니다. 프레임워크가 도구 호출 루프를 대신 실행합니다.
- **작업 2**에서는 동일한 클라이언트를 유지하면서 **여러** 에이전트를 만들고, 이를 `SequentialBuilder` 오케스트레이션에 전달합니다. 오케스트레이션은 에이전트를 순서대로 실행하고 각 에이전트의 출력을 수집합니다.
- **작업 3**에서는 에이전트를 **별도 프로세스**로 분리하고 라우팅 에이전트가 **A2A 프로토콜**을 사용해 이를 검색하고 호출하게 합니다. 같은 협업 아이디어를 네트워크를 통해 확장합니다.
- **작업 4**에서는 다시 **단일** 에이전트로 돌아옵니다. 하지만 해당 에이전트의 **구조화된 출력**(JSON 분류)이 코드의 **조건부 라우팅**을 구동하여 각 지원 티켓을 에스컬레이션하거나 자동 처리합니다.

먼저 단일 에이전트의 동작 방식을 확인하면 이후 다중 에이전트 패턴의 의미를 더 잘 이해할 수 있습니다.

## 요약

이 랩 전체에서 다음을 수행했습니다.

- Microsoft Agent Framework를 사용해 **사용자 지정 도구가 있는 에이전트**를 빌드했습니다.
- (선택 사항) 여러 에이전트를 순서대로 **오케스트레이션**하여 작업을 단계별로 분류했습니다.
- (선택 사항) 조정 에이전트가 라우팅하는 **A2A 프로토콜**을 사용해 프로세스 간 **원격 에이전트**를 연결했습니다.
- (선택 사항) 에이전트의 **구조화된 분류**를 코드의 **조건부 라우팅**으로 전환했습니다.

이러한 내용은 Agent Framework가 집중된 단일 에이전트에서 조정된 에이전트 팀으로 확장되는 방식을 보여 줍니다.

## 정리

완료했다면 불필요한 Azure 비용을 방지하기 위해 만든 리소스를 삭제합니다.

1. [Azure 포털](https://portal.azure.com)에서 Foundry 리소스가 포함된 리소스 그룹으로 이동합니다.
1. 도구 모음에서 **리소스 그룹 삭제(Delete resource group)**를 선택하고, 리소스 그룹 이름을 입력한 다음 확인합니다.

> `azd`로 프로비저닝한 경우 대신 `azd down`을 실행하여 생성된 모든 항목을 제거합니다.