---
title: '에이전트 관찰, 평가 및 보안'
lab:
    title: '에이전트 관찰, 평가 및 보안'
    description: 'Caldova 에이전트가 실제로 무엇을 하는지 확인합니다. OpenTelemetry로 추적하고, 기본 제공 평가기를 사용해 정답 기준으로 답변을 채점하며, AI Red Teaming Agent로 공격해 봅니다. 처음부터 끝까지 또는 한 작업씩 완료할 수 있는 모듈식 랩입니다.'
    type: 'lab'
    id: 'D'
    order: 4
    difficulty: 3
    duration: 60
    access: 'open'
    level: 300
    concepts: '추적, OpenTelemetry, 평가, groundedness, AI red teaming'
    islab: true
    status: 'draft'
---

<!--
PILOT NOTE (remove before publishing):
"Lab D" is new content: there was no observability, evaluation or safety-testing
material anywhere in this repo. It follows the same template as Labs A-C.
Starter code lives in a single folder — Labfiles/D-observe-evaluate-and-secure-agents/Python/ —
shared by every task (one virtual environment, one .env). The completed reference code is
in Labfiles/D-observe-evaluate-and-secure-agents/Solution/Python/.

This landing page is the lab overview. Setup lives in D0-getting-started.md and each task is
its own page (D1-D3) so it can be completed on its own. The azd template and Bicep are
generated from Labfiles/_shared/ — edit them there, not in the lab folder.
-->

# 에이전트 관찰, 평가 및 보안

**수준** ▰▰▰▱▱ **L300**  (**L100** 초급 → **L500** 전문가)

오후 한나절이면 에이전트를 만들 수 있습니다. 하지만 그 에이전트가 실제로 괜찮은지, 그리고 누군가
잘못 행동하게 만들려고 할 때 제대로 동작하는지 아는 것은 다른 문제입니다. 이 랩은 바로 그 문제를
다룹니다. 실행 중인 에이전트 내부를 들여다보고, 답변 품질을 측정하며, 다른 사람이 공격하기 전에
먼저 공격해 봅니다.

![Anton](../Media/anton-avatar.png)<br /><strong>AI 가이드 Anton을 만나보세요.</strong><br />이 랩 전체에서 **Ask Anton** 팁을 보게 됩니다. 더 대화형의 실습 도움말을 원하시나요? *[Ask Anton](https://aka.ms/choose-anton)* 앱에서 Anton과 채팅하세요.

<details>
<summary><strong><i>Ask Anton 앱 정보</i></strong></summary>

<strong><i><a href="https://aka.ms/choose-anton" target="_blank">Ask Anton</a></i></strong>은 AI 개념과 Microsoft Foundry 기술에 대한 질문에 답할 수 있는 생성형 AI 에이전트입니다. <code>https://aka.ms/choose-anton</code>에서 두 가지 버전으로 사용할 수 있습니다.
<ul>
<li><strong>Azure 기반</strong>: 최상의 환경 <i>(Azure 구독과 Foundry 프로젝트의 모델 배포가 필요)</i>.</li>
<li><strong>브라우저 기반</strong>: 브라우저에서 작은 언어 모델 사용 <i>(기능 제한 - 오래되었거나 사양이 낮은 디바이스에서는 느리거나 "basic" 모드에서만 작동할 수 있음)</i>.</li>
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
<summary>출력만 읽으면 안 되는 이유는 무엇인가요?</summary>
<div class="concept-body" markdown="1">

출력은 다른 모든 것이 문제가 있어도 멀쩡해 보이는 에이전트의 한 부분이기 때문입니다.
답변은 유창하지만 확신에 차서 틀릴 수도 있고, 정확하지만 세 번 재시도하고 시간 초과된 도구 호출을 거쳐
생성되었을 수도 있습니다. **Tracing**은 답변에 이르는 과정에서 무슨 일이 있었는지 보여 줍니다.
**Evaluation**은 이미 참이라고 알고 있는 내용과 답변을 비교해 채점합니다.
**Red teaming**은 질문이 적대적일 때 에이전트가 어떻게 동작하는지 알려 줍니다.

</div>
</details>

**시나리오:** 여러분은 가속화된 제품 출시를 준비 중인 제약 제조업체 **Caldova**에서 일합니다.
이전 랩에서 만든 공급망 도우미가 이제 계획 팀의 실제 질문에 답하고 있으며, IT 규정 준수 책임자는
더 어려운 질문을 하고 있습니다. *왜 그 답변이 느렸을까요? 용량 정책에 대해 지어내고 있나요?
누군가 말해서 해서는 안 되는 말을 하게 만들면 어떻게 될까요?* 이 랩에서는 세 가지 모두에 대해
의견이 아니라 증거로 답합니다.

먼저 **Core** 작업으로 시작하여 "실행된다"에서 "얼마나 잘 실행되는지 증명할 수 있다"까지 진행합니다.
그런 다음 **Optional** 작업에서 안전성을 대상으로 테스트합니다.

> **Note**: 이 연습에서 사용하는 일부 기술은 미리 보기이거나 활발히 개발 중입니다.
> 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## 학습할 내용

이 연습의 **Core** 작업을 완료하면 다음을 수행할 수 있습니다.

- OpenTelemetry로 **에이전트를 추적**하고, 추적을 Azure Monitor로 내보낸 다음 읽습니다 —
  Foundry 자체 서버 쪽 추적과 사용자 지정 span이 서로 다른 보기에서 모두 제공됩니다.
- 기본 제공 평가기(groundedness, relevance, similarity)와 JSONL 데이터 세트를 사용하여
  정답 기준으로 **답변 품질을 평가**합니다.

**Optional** 작업에서는 추가로 다음을 수행할 수 있습니다.

- AI Red Teaming Agent로 **에이전트를 red team**합니다. 배포된 에이전트를 대상으로 적대적 공격 전략과
  직접 만든 seed prompt를 실행하고 attack success rate를 읽습니다.

## 이 랩의 구성 방식

이 랩은 **모듈식**입니다. 각 작업은 **처음부터 독립적으로** 완료할 수 있도록 작성되어 있으므로
작업 하나만 선택해 진행할 수 있습니다. 모든 작업은 하나의 시작 폴더, 하나의 가상 환경, 하나의 `.env`를
공유하므로 처음부터 끝까지 이어서 진행할 수도 있습니다.

1. **[시작하기](D0-getting-started.md)로 시작합니다** — Microsoft Foundry 프로젝트를 만들고,
   Application Insights를 연결하고, 시작 코드를 가져오고, `.env`를 설정합니다. 모든 작업은 여기서
   시작합니다. 전체 랩을 한 번에 진행하는 경우 이 작업은 한 번만 수행하면 됩니다.
2. **원하는 작업을 수행합니다.** 각 작업에는 독립적으로 시작할 수 있도록 필요한 설정이 나열되어 있습니다.
   이전 작업에서 바로 이어서 진행하는 경우, 맨 위의 짧은 *"이전 작업에서 계속 진행하나요?"* 참고를 통해
   반복 설정을 건너뛰고 계속 진행할 수 있습니다.

## 랩 한눈에 보기

먼저 **Core** 작업을 완료합니다. 이 작업이 끝나면 내부를 볼 수 있는 에이전트와 답변에 대한
스코어카드를 갖게 됩니다. 공격에 얼마나 견디는지 테스트하려면 **Optional** 작업을 추가합니다.

<!-- BEGIN GENERATED: task-table - do not edit by hand; run: python tools/generate_lab_blocks.py -->

| 섹션 | 작업 | 수준 | 시간 |
| --- | --- | --- | --- |
| **Core** | [작업 1 – 에이전트 추적](D1-trace-your-agent.md) | ▰▰▰▱▱ L300 | 약 25분 |
| **Core** | [작업 2 – 답변 품질 평가](D2-evaluate-answer-quality.md) | ▰▰▰▱▱ L300 | 약 35분 |
| *Optional* | [작업 3 – 에이전트 red team](D3-red-team-your-agent.md) | ▰▰▰▰▱ L400 | 약 35분 |

**Core 작업:** 약 **60분**. 모든 선택 작업을 포함한 **전체 랩**: 약 **1시간 35분**.

<!-- END GENERATED: task-table -->

**경로 선택** — 사용 가능한 시간에 맞는 작업을 선택합니다.

- **Core만(~1시간):** 작업 1–2를 수행합니다.
- **전체(~1시간 35분):** red team 검사인 **작업 3**을 추가합니다.

> **하나의 에이전트, 세 가지 질문**: 세 작업은 모두 동일한 **grounded knowledge
> agent**를 대상으로 합니다. 작업 1은 이를 추적하고, 작업 2와 3은 이를 측정합니다. 아직
> [Lab B](B-integrate-agents-with-enterprise-knowledge-and-m365.md)를 완료하지 않았다면, 이 랩이 독립적으로
> 실행되도록 한 명령으로 동등한 에이전트를 만듭니다. [시작하기](D0-getting-started.md)를 참조하세요.

## 추측하지 말고 측정하기

이 랩의 세 가지 기법은 서로 다른 질문에 답합니다.

- **Tracing**은 *"무슨 일이 있었나?"*에 답합니다. 한 번의 실행 기록입니다. 어떤 span에 시간이 얼마나
  걸렸는지, 어떤 도구가 호출되었는지, 모델에 무엇이 전송되었는지를 보여 줍니다. 무언가 느리거나
  중단되었을 때 사용합니다.
- **Evaluation**은 *"평균적으로 얼마나 좋은가?"*에 답합니다. 데이터 세트 전체에 대한 점수이므로,
  세 가지 중 변경 사항이 더 좋아졌는지 나빠졌는지 알려 주는 유일한 방법입니다.
- **Red teaming**은 *"내가 무엇을 하게 만들 수 있는가?"*에 답합니다. 적대적 탐색이며, 깨끗한 결과는
  보장이 아니라 최소 기준입니다.

어느 것도 다른 것을 대체하지 않으며, 세 가지 모두 운영 환경에서 알게 되는 것보다 훨씬 저렴합니다.

## 요약

이 랩 전체에서 다음을 수행했습니다.

- OpenTelemetry로 **에이전트를 계측**하고 추적을 Application Insights로 내보냈습니다 —
  Foundry의 자동 서버 쪽 추적과 사용자의 사용자 지정 span을 나란히 읽었습니다.
- 기본 제공 groundedness, relevance, similarity 평가기를 사용하여 정답 데이터 세트 기준으로
  **grounded agent를 평가**하고, 변경 간 비교할 수 있는 점수를 얻었습니다.
- (선택 사항) 적대적 공격 전략과 직접 만든 seed prompt를 사용하여 **에이전트를 red team**하고,
  결과 attack success rate를 읽었습니다.

이 모든 작업은 "데모가 작동했다"를 다른 사람에게 보여 줄 수 있는 증거로 바꿉니다.

## 정리

완료한 경우 불필요한 Azure 비용이 발생하지 않도록 만든 리소스를 삭제합니다.

1. [Azure portal](https://portal.azure.com)에서 Foundry 리소스가 포함된 리소스 그룹으로 이동합니다.
1. 도구 모음에서 **Delete resource group**을 선택하고, 리소스 그룹 이름을 입력한 다음 확인합니다.

> 세 작업은 모두 동일한 `caldova-knowledge-agent`를 측정하므로, 리소스 그룹을 삭제하면 다른 모든 항목과
> 함께 이 에이전트도 제거됩니다. `azd`로 프로비전한 경우 대신 `azd down`을 실행합니다. 단,
> Foundry portal에서 만든 Application Insights는 별도의 리소스이며 `azd`가 아니라 리소스 그룹과 함께
> 삭제됩니다.
