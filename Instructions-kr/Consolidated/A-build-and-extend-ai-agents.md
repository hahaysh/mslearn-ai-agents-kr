---
title: 'AI 에이전트 빌드 및 확장'
lab:
    title: 'AI 에이전트 빌드 및 확장'
    description: 'Caldova 공급망 도우미를 빌드합니다. 회사 정책으로 그라운딩한 다음 원격 MCP 서버, 사용자 지정 함수, 클라이언트 앱을 사용하여 도구로 확장합니다. 처음부터 끝까지 또는 작업별로 완료할 수 있는 모듈형 랩입니다.'
    type: 'lab'
    id: 'A'
    order: 1
    difficulty: 3
    duration: 35
    access: 'open'
    level: 300
    concepts: '에이전트 생성 및 그라운딩, 도구, Model Context Protocol (MCP)'
    islab: true
    status: 'draft'
---

# AI 에이전트 빌드 및 확장

**수준** ▰▰▰▱▱ **L300**

(**L100** 초급 → **L500** 전문가)

Microsoft Foundry와 Python으로 실용적인 AI 에이전트를 빌드한 다음, 회사 지식과
도구로 확장합니다.

## 사례

여러분은 제약 제조업체인 **Caldova**에서 근무합니다. Caldova는 예상보다 이른 제품
출시를 계획하고 있지만, 세 공장의 생산량은 출시 요구량보다 약 7% 부족합니다.

계획 팀은 Caldova 공장 간에 작업을 이동할지, 또는 계약 제조업체라고도 하는 승인된
제조 파트너를 고용할지 결정해야 합니다. 여러분은 팀이 의사 결정을 내릴 수 있도록
공급망 도우미를 빌드합니다. 이 도우미는 회사 정책을 사용하고, 공장 산출량을 분석하며,
사용 가능한 생산 시간을 찾고, 파트너 비용을 추정하고, 충분한 자재 재고가 있는지
확인합니다.

## 수행할 작업

작동하는 에이전트를 위한 두 가지 핵심 작업을 완료한 다음, 연습하고 싶은 내용에 따라
선택 작업을 진행합니다.

<!-- BEGIN GENERATED: task-table - do not edit by hand; run: python tools/generate_lab_blocks.py -->

| 섹션 | 작업 | 수준 | 시간 |
| --- | --- | --- | --- |
| **핵심** | [작업 1 – 에이전트 만들기 및 그라운딩](A1-create-and-ground-an-agent.md) | ▰▰▱▱▱ L200 | ~15분 |
| **핵심** | [작업 2 – 원격 MCP 서버 연결](A2-connect-a-remote-mcp-server.md) | ▰▰▰▱▱ L300 | ~20분 |
| *선택 사항* | [작업 3 – 클라이언트 앱에서 에이전트 호출](A3-call-your-agent-from-a-client-app.md) | ▰▰▰▱▱ L300 | ~20분 |
| *선택 사항* | [작업 4 – 사용자 지정 함수 도구 추가](A4-add-custom-function-tools.md) | ▰▰▰▱▱ L300 | ~25분 |
| *선택 사항* | [작업 5 – 종합 과제: 나만의 MCP 서버 빌드](A5-capstone-build-your-own-mcp-server.md) | ▰▰▰▰▱ L400 | ~35분 |
| *선택 사항* | [작업 6 – 도우미를 호스티드 에이전트로 승격](A6-promote-your-assistant-to-a-hosted-agent.md) | ▰▰▰▱▱ L300 | ~30분 |

**핵심 작업:** 약 **35분**. 모든 선택 작업을 포함한 **전체 랩**: 약 **2시간 25분**.

<!-- END GENERATED: task-table -->

> **Note**: 이 연습에 사용되는 일부 기술은 미리 보기 상태이거나 활발히 개발 중입니다.
> 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

[시작하기](A0-getting-started.md)에서 Microsoft Foundry 프로젝트를 만들고 공유
스타터 코드를 준비합니다. 각 작업에는 독립적으로 시작하는 데 필요한 설정이 포함되어
있습니다. 랩을 순서대로 완료하면 같은 환경을 재사용하고 반복 설정을 건너뜁니다.

![Anton](../Media/anton-avatar.png)

**AI 가이드 Anton을 소개합니다.**

이 랩 전체에서 **Ask Anton** 팁을 볼 수 있습니다. 더 대화형 도움말을 보려면
*[Ask Anton](https://aka.ms/choose-anton)* 앱을 사용합니다.

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
<summary>에이전트란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

AI 에이전트는 생성형 AI를 사용하여 요청을 이해하고, 무엇을 할지 결정하며, 사용자를
대신해 작업을 수행하는 소프트웨어 서비스입니다. 에이전트를 진정으로 유용하게 만드는
것은 모델만이 아닙니다. 관련 **지식**과 **도구**도 필요합니다.

[자세히 알아보기 →](https://review.learn.microsoft.com/en-us//training/modules/build-extend-ai-agents/1-introduction?branch=pr-en-us-55509)

</div>
</details>

## 이 랩이 적합한가요?

Microsoft Foundry와 Python으로 에이전트를 빌드하는 실습을 원한다면 이 랩을 선택합니다.
에이전트 코드를 작성하고, 원격 및 사용자 지정 도구를 연결하며, 도구 호출 루프를 직접
다룹니다. 웹 채팅 인터페이스는 제공됩니다.

작업 4와 5에는 비교를 위해 바로 실행할 수 있는 Microsoft Agent Framework 버전도
포함되어 있습니다. 작업 6에서는 코드를 호스티드 에이전트로 배포할 수 있습니다.

## 정리

작업을 마쳤으면 불필요한 Azure 비용을 방지하기 위해 만든 리소스를 삭제합니다.

1. [Azure portal](https://portal.azure.com)에서 Foundry 리소스가 포함된 리소스 그룹으로 이동합니다.
1. 도구 모음에서 **리소스 그룹 삭제(Delete resource group)**를 선택하고, 리소스 그룹 이름을 입력한 다음 확인합니다.

> 작업 2에서 실행한 코드는 생성한 에이전트 버전을 이미 삭제합니다. 포털 에이전트는
> 리소스 그룹을 삭제할 때 제거됩니다. `azd`로 프로비전한 경우 대신 `azd down`을 실행하여
> 생성된 모든 항목을 제거합니다.
