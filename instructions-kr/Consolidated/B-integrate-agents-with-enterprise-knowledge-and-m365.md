---
title: '엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합'
lab:
    title: '엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합'
    description: 'Caldova 직원 지식 도우미를 빌드합니다. Foundry IQ로 엔터프라이즈 문서를 기반으로 응답하게 한 다음 Microsoft Teams, Microsoft 365 Copilot, Work IQ를 통해 제공합니다. 처음부터 끝까지 완료하거나 한 번에 하나의 작업만 진행할 수 있는 모듈식 랩입니다.'
    type: 'lab'
    id: 'B'
    order: 2
    difficulty: 3
    duration: 35
    access: 'open'
    level: 300
    concepts: '엔터프라이즈 지식 기반화, Foundry IQ, Microsoft 365, Model Context Protocol (MCP)'
    islab: true
    status: 'draft'
---

# 엔터프라이즈 지식 및 Microsoft 365와 에이전트 통합

**수준** ▰▰▰▱▱ **L300**  (**L100** 초급 → **L500** 전문가)

에이전트가 회사의 *자체* 지식을 기반으로 답변하고 직원들이 이미 일하는 위치에 나타날 때, 비즈니스에 진정으로 유용해집니다. 이 랩에서는 **기반화된 엔터프라이즈 지식 에이전트**를 빌드한 다음 **Microsoft 365를 통해 제공**합니다.

![Anton](../Media/anton-avatar.png)<br /><strong>AI 가이드 Anton을 만나보세요.</strong><br />이 랩 곳곳에서 **Ask Anton** 팁을 볼 수 있습니다. 더 대화형의 실습 도움말이 필요하신가요? *[Ask Anton](https://aka.ms/choose-anton)* 앱에서 Anton과 채팅하세요.

<details>
<summary><strong><i>Ask Anton 앱 정보</i></strong></summary>

<strong><i><a href="https://aka.ms/choose-anton" target="_blank">Ask Anton</a></i></strong>은 AI 개념과 Microsoft Foundry 기술에 대한 질문에 답할 수 있는 생성형 AI 에이전트입니다. <code>https://aka.ms/choose-anton</code>에서 두 가지 버전으로 사용할 수 있습니다.
<ul>
<li><strong>Azure 기반</strong>: 최상의 환경 <i>(Azure 구독 및 Foundry 프로젝트의 모델 배포 필요)</i>.</li>
<li><strong>브라우저 기반</strong>: 브라우저에서 작은 언어 모델 사용 <i>(기능 제한 - 구형/저사양 디바이스에서는 느리거나 "basic" 모드에서만 작동할 수 있음)</i>.</li>
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
<summary>엔터프라이즈 지식 기반화란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

**엔터프라이즈 지식**을 기반으로 에이전트를 구성한다는 것은 에이전트를 조직의 자체 문서(정책, 카탈로그, 절차)에 연결하여 추측이 아니라 신뢰할 수 있는 자료를 바탕으로 답변하게 한다는 의미입니다. **Foundry IQ**는 이를 대규모로 수행합니다. 지식 베이스를 인덱싱하고 *에이전트형 검색*을 수행하며, 각 조회 전에 **승인** 단계를 요구하도록 설정하여 앱이 제어권을 유지할 수 있습니다.

[자세히 알아보기 →](https://learn.microsoft.com/azure/ai-foundry/)

</div>
</details>

**시나리오:** 사용자는 신속한 제품 출시를 준비 중인 제약 제조업체 **Caldova**에서 근무합니다. 계획 및 자재 팀은 사이트 용량, 계약 제조업체, 기술 이전, 공급업체에 대한 질문을 끊임없이 처리하며, 그 답은 모두 내부 문서에 있습니다. 이 랩에서는 **Caldova 직원 지식 도우미**를 빌드합니다. 먼저 Foundry IQ로 이러한 엔터프라이즈 문서를 기반으로 응답하게 하고, 그런 다음 직원들이 이미 일하는 위치에서 사용할 수 있도록 Microsoft Teams와 Microsoft 365 Copilot에 게시하며, 마지막으로 **Work IQ**를 살펴보면서 라이브 Microsoft 365 신호를 에이전트로 가져옵니다.

먼저 작동하는 기반화된 엔터프라이즈 지식 에이전트를 만드는 **Core** 작업부터 시작합니다. 그다음 일련의 **Optional** 작업을 통해 이를 제공하고 확장할 수 있습니다.

> **Note**: 이 연습에서 사용하는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 학습 내용

이 연습의 **Core** 작업을 완료하면 다음을 수행할 수 있습니다.

- Microsoft Foundry 포털에서 **Foundry IQ**를 사용하여 **엔터프라이즈 지식 에이전트를 만들고 기반화**하며, 에이전트가 지식 베이스를 검색하기 전에 **승인**을 요구하도록 설정합니다.
- **코드에서 에이전트에 연결**하고 지식 도구 승인 흐름을 직접 처리합니다.

**Optional** 작업을 통해 추가로 다음을 수행할 수 있습니다.

- 직원들이 Teams에서 채팅할 수 있도록 **에이전트를 Microsoft Teams에 게시**합니다.
- Copilot 에이전트로 **에이전트를 Microsoft 365 Copilot에 게시**합니다.
- MCP를 통해 **Work IQ로 Microsoft 365 업무 신호를 에이전트에 가져옵니다**.

## 이 랩의 구성 방식

이 랩은 **모듈식**입니다. 각 작업은 **처음부터 시작해 독립적으로 완료**할 수 있도록 작성되어 있으므로, 원하는 작업 하나만 선택해 진행할 수 있습니다. 모든 코드 작업은 하나의 시작 폴더, 하나의 가상 환경, 하나의 `.env`도 공유하므로, 처음부터 끝까지 이어서 진행하고 싶다면 그렇게 할 수 있습니다.

1. **[시작하기](B0-getting-started.md)부터 시작합니다** — Microsoft Foundry 프로젝트를 만들고(포털에서 또는 `azd up` 명령 하나로), 시작 코드를 가져오고, `.env`를 설정합니다. 모든 작업은 여기서 시작합니다. 전체 랩을 한 번에 진행한다면 이 작업은 한 번만 수행하면 됩니다.
2. **원하는 작업을 수행합니다.** 각 작업에는 필요한 설정이 나열되어 있어 독립적으로 시작할 수 있습니다. 이전 작업에서 바로 이어서 진행하는 경우, 맨 위의 짧은 *"이전 작업에서 계속 진행하나요?"* 참고를 통해 반복 설정을 건너뛰고 계속 진행할 수 있습니다.

## 랩 한눈에 보기

먼저 **Core** 작업을 완료합니다. 이 작업이 끝나면 코드에서 호출할 수 있는 작동하는 기반화된 엔터프라이즈 지식 에이전트가 완성됩니다. 그런 다음 관심 있는 **Optional** 작업을 확장합니다.

<!-- BEGIN GENERATED: task-table - do not edit by hand; run: python tools/generate_lab_blocks.py -->

| 섹션 | 작업 | 수준 | 시간 |
| --- | --- | --- | --- |
| **Core** | [작업 1 – Foundry IQ 지식 에이전트를 만들고 코드에서 연결](B1-create-a-foundry-iq-knowledge-agent.md) | ▰▰▰▱▱ L300 | 약 35분 |
| *Optional* | [작업 2 – 에이전트를 Microsoft Teams에 게시](B2-publish-to-microsoft-teams.md) 🔒 | ▰▰▱▱▱ L200 | 약 20분 |
| *Optional* | [작업 3 – 에이전트를 Microsoft 365 Copilot에 게시](B3-publish-to-microsoft-365-copilot.md) 🔒 | ▰▰▱▱▱ L200 | 약 15분 |
| *Optional* | [작업 4 – Work IQ: Microsoft 365 신호를 에이전트에 가져오기](B4-work-iq-workplace-intelligence.md) 🔒 | ▰▰▰▰▱ L400 | 약 40분 |

**Core 작업:** 약 **35분**. 모든 선택 작업을 포함한 **전체 랩**: 약 **1시간 50분**.

> 🔒 잠금 표시가 있는 작업은 계정에 없는 액세스 권한이 필요할 수 있습니다. 각 작업은 빠른 확인으로 시작하며 해당 권한이 없을 때 수행할 작업을 알려줍니다. 이 랩의 다른 어떤 내용도 해당 작업에 의존하지 않습니다.

<!-- END GENERATED: task-table -->

**경로 선택** — 사용 가능한 시간에 맞는 작업을 선택합니다.

- **Core만 진행(약 35분):** 작업 1을 수행합니다.
- **Core + 제공(약 1시간 10분):** **작업 2**와 **작업 3**도 수행하여 에이전트를 M365에 게시합니다.
- **전체 진행(약 1시간 50분):** 라이브 업무 인텔리전스를 위해 **작업 4**(Work IQ)를 추가합니다.

> **하나의 도우미를 어디서나 제공**: 작업 1~3은 모두 **동일한** 기반화된 에이전트(`caldova-knowledge-agent`)를 중심으로 진행됩니다. 작업 1에서 한 번 만들고 기반화한 다음, 작업 2와 3에서는 동일한 에이전트를 Teams와 Copilot에 간단히 *게시*합니다. 새 코드는 필요하지 않습니다. 작업 4에서는 별도의 에이전트로 다른 Microsoft 365 기능(Work IQ)을 살펴봅니다.

## 요약

이 랩 전체에서 다음을 수행했습니다.

- Foundry 포털에서 **Foundry IQ**로 엔터프라이즈 지식 에이전트를 만들고 **기반화**했으며, 각 지식 조회 전에 승인을 요구하도록 설정했습니다.
- **코드에서 에이전트에 연결**하고 승인 흐름을 직접 처리했습니다.
- (선택 사항) 에이전트를 **Microsoft Teams** 및 **Microsoft 365 Copilot**에 **게시**하고, **Work IQ**를 살펴보며 라이브 Microsoft 365 신호를 에이전트로 가져왔습니다.

이 과정을 통해 기반화된 지식 베이스에서 시작해 조직이 매일 사용하는 Microsoft 365 화면까지 에이전트를 가져가는 방법을 보여줍니다.

## 정리

완료한 후에는 불필요한 Azure 비용을 방지하기 위해 만든 리소스를 삭제합니다.

1. [Azure 포털](https://portal.azure.com)에서 Foundry 및 Azure AI Search 리소스가 포함된 리소스 그룹으로 이동합니다.
1. 도구 모음에서 **Delete resource group**을 선택하고 리소스 그룹 이름을 입력한 다음 확인합니다.

> 작업 4에서 실행하는 코드는 생성한 에이전트 버전을 이미 삭제합니다. 포털 에이전트는 리소스 그룹을 삭제할 때 제거됩니다. `azd`로 프로비전한 경우에는 대신 `azd down`을 실행하여 생성된 모든 항목을 제거합니다.
