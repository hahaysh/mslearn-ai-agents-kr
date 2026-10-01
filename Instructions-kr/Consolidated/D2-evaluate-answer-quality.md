---
title: '작업 2 – 답변 품질 평가'
lab:
    title: '작업 2 – 답변 품질 평가'
    description: '기본 제공 groundedness, relevance, similarity 평가기와 행 수준 결과를 사용해 정답 데이터 세트 기준으로 grounded agent를 채점합니다.'
    type: 'task'
    parent: 'D'
    order: 2
    section: 'core'
    difficulty: 3
    duration: 35
    access: 'open'
    level: 300
    concepts: '평가, groundedness, relevance, similarity, 정답'
    islab: true
    status: 'draft'
---

# 작업 2 — 답변 품질 평가

***에이전트 관찰, 평가 및 보안** 랩의 일부입니다. 처음이라면 [시작하기](D0-getting-started.md)부터 시작하세요.*

> **Set up (start here):** 이 작업에는 Foundry 프로젝트, 시작 코드, **측정할 grounded agent**가
> 필요합니다. 아직 완료하지 않았다면 [시작하기](D0-getting-started.md)를 완료해 프로젝트를 만들고,
> 코드를 복제하고, `PROJECT_ENDPOINT`와 `MODEL_DEPLOYMENT_NAME`을 `Python/.env`에 설정합니다.
> 또한 `AGENT_NAME`을 [Lab B](B-integrate-agents-with-enterprise-knowledge-and-m365.md) 에이전트로
> 지정하거나 `python ../setup/bootstrap_agent.py`로 새로 만듭니다. 그런 다음 VS Code에서 연
> `Python` 폴더에서 준비되었는지 확인합니다.

```
python ../setup/check_env.py --task 2
```

> **Continuing from a previous task?** 같은 `Python` 폴더에서 방금 다른 작업을 마쳤다면 프로젝트,
> 가상 환경, `.env`가 이미 설정되어 있습니다. 아래 **먼저 데이터 세트 살펴보기**로 바로 이동합니다.

---

Caldova 지식 에이전트는 용량, 계약 제조업체, 공급업체에 대한 질문에 답합니다. 매번 확신 있게 들립니다.
바로 그 점이 문제입니다. 답변 열 개를 읽고 괜찮다고 느껴도, 열한 번째 답변에서 존재하지 않는 반품 기간을
지어내는지 전혀 알 수 없습니다.

**Evaluation**은 그 느낌을 숫자로 바꿉니다. 이미 정답을 알고 있는 질문 세트를 가져와 에이전트에 실행하고,
두 번째 모델이 돌아온 답변을 채점하게 합니다.

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
<summary>groundedness, relevance, similarity란 무엇인가요?</summary>
<div class="concept-body" markdown="1">

답변이 틀릴 수 있는 방식은 세 가지로 나뉘므로 각각 따로 측정합니다.

- **Groundedness** — 답변이 제공된 **context**로 뒷받침되나요? 낮은 점수는 에이전트가 무언가를
  지어냈다는 뜻입니다. 우연히 그 내용이 사실이어도 마찬가지입니다.
- **Relevance** — 답변이 실제로 **question**에 답하나요? 답변은 완전히 grounded되어 있어도 물어본 것과
  다를 수 있습니다.
- **Similarity** — 답변이 작성한 **ground truth**와 얼마나 가까운가요? 사람이 작성한 정답이 필요한
  항목입니다.

각 항목은 judge 역할을 하는 모델이 1~5점으로 채점합니다. Groundedness와 relevance는 `reason`도
반환하며, 이는 보통 점수보다 더 유용합니다.

</div>
</details>

[시작하기](D0-getting-started.md)의 `Python` 폴더를 열고 가상 환경(`.\labenv\Scripts\Activate.ps1`)을 활성화한 다음 아래를 계속 진행합니다.

### 먼저 데이터 세트 살펴보기

**data/caldova_eval.jsonl**을 엽니다. 각 줄은 하나의 테스트 사례이며, 평가는 항상 이 파일의 품질에
좌우됩니다.

```json
{"query": "A planner wants to raise a capacity request with a complete program brief. How long does review take?", "context": "Standard Request Window: Requests with a complete program brief: reviewed within 5 business days ...", "ground_truth": "Five business days for a request with a complete program brief ..."}
```

- `query` — 에이전트에 묻는 내용입니다.
- `context` — 답변이 기반으로 삼아야 하는 원본 자료입니다. Groundedness는 이를 기준으로 채점합니다.
- `ground_truth` — 지식이 있는 사람이 제공할 답변입니다. Similarity는 이를 기준으로 채점합니다.

에이전트 자체의 `response`는 파일에 없습니다. evaluation 시점에 **target**을 지정해 생성합니다.

**agent_target.py**를 열고 읽습니다. 편집하지는 않습니다. 이 파일은 하나의 `query`를 받아 에이전트에서
`{"response": ...}`를 반환하는 호출 가능한 클래스입니다. target이 충족해야 하는 계약은 이것이 전부입니다.

### evaluation 작성

**evaluate_agent.py**를 열고 주석 처리된 각 자리 표시자에 코드를 추가합니다.

1. **참조 추가**:

    ```python
    # Add references
    from azure.ai.evaluation import (
        AzureOpenAIModelConfiguration,
        GroundednessEvaluator,
        RelevanceEvaluator,
        SimilarityEvaluator,
        evaluate,
    )
    from agent_target import CaldovaAgentTarget
    ```

1. **답변을 채점할 모델 구성** — 평가기 자체도 모델 호출이므로 실행할 배포가 필요합니다. 에이전트가
    사용하는 것과 동일한 배포를 다시 사용합니다.

    ```python
    # Configure the model that grades the answers
    model_config = AzureOpenAIModelConfiguration(
        azure_endpoint=evaluator_endpoint(),
        azure_deployment=model_deployment,
        api_version=os.getenv("AZURE_OPENAI_API_VERSION", "2024-10-21"),
    )
    ```

    > API 키는 없습니다. `az login`을 완료하면 평가기가 사용자 자격 증명으로 인증합니다.
    > `evaluator_endpoint()`는 파일 맨 위에 제공되어 있습니다. `PROJECT_ENDPOINT`에서 리소스 엔드포인트를
    > 파생하거나, 설정한 경우 `AZURE_OPENAI_ENDPOINT`를 사용합니다.

1. **평가기 만들기**:

    ```python
    # Create the evaluators
    groundedness = GroundednessEvaluator(model_config)
    relevance = RelevanceEvaluator(model_config)
    similarity = SimilarityEvaluator(model_config)
    ```

1. **evaluation 실행** — `evaluate()`는 데이터 세트를 읽고, 각 행마다 target을 한 번 호출한 다음,
    각 평가기에 필요한 열만 정확히 전달합니다. `column_mapping`은 어떤 열이 무엇인지 지정하는 방법입니다.
    `${data.x}`는 파일에서 오고, `${target.x}`는 target에서 돌아옵니다.

    ```python
    # Run the evaluation
    result = evaluate(
        data=str(DATASET),
        target=CaldovaAgentTarget(),
        evaluators={
            "groundedness": groundedness,
            "relevance": relevance,
            "similarity": similarity,
        },
        evaluator_config={
            "groundedness": {
                "column_mapping": {
                    "query": "${data.query}",
                    "context": "${data.context}",
                    "response": "${target.response}",
                }
            },
            "relevance": {
                "column_mapping": {
                    "query": "${data.query}",
                    "response": "${target.response}",
                }
            },
            "similarity": {
                "column_mapping": {
                    "query": "${data.query}",
                    "ground_truth": "${data.ground_truth}",
                    "response": "${target.response}",
                }
            },
        },
        azure_ai_project=project_endpoint,
        output_path=str(OUTPUT),
    )
    ```

    > `azure_ai_project`는 선택 사항입니다. 전달하면 실행이 프로젝트에 업로드되어 portal에서 다른 항목과
    > 함께 결과를 볼 수 있습니다.

1. **집계 점수 출력**:

    ```python
    # Print the aggregate scores
    print("\nAggregate scores (1-5, higher is better):")
    print(json.dumps(result["metrics"], indent=2))
    print(f"\nRow-level detail: {OUTPUT.resolve()}")
    if result.get("studio_url"):
        print(f"View in the Foundry portal: {result['studio_url']}")
    ```

1. 파일을 저장합니다(**Ctrl+S**).

### 실행 및 테스트

1. 터미널에서 로그인하고 evaluation을 실행합니다.

    ```
    az login
    ```

    ```
    python evaluate_agent.py
    ```

1. 에이전트에 열 개의 질문을 모두 묻고 각 답변을 세 가지 방식으로 채점하므로 몇 분 정도 기다립니다.
    다음과 비슷한 출력이 표시됩니다.

    ```
    Aggregate scores (1-5, higher is better):
    {
      "groundedness.groundedness": 4.6,
      "relevance.relevance": 4.4,
      "similarity.similarity": 4.1
    }

    Row-level detail: ...\eval_results.json
    ```

    > 실제 숫자는 달라집니다. 정확한 값보다 변경 후 이를 *재현*할 수 있는지가 훨씬 더 중요합니다.

1. **eval_results.json**을 열고 가장 낮은 점수의 행을 찾습니다. 해당 `groundedness_reason` 또는
    `relevance_reason`을 읽습니다. judge가 스스로 설명하며, 바로 그 설명을 토대로 조치하게 됩니다.

1. 점수가 에이전트의 잘못인지 판단합니다. 때때로 낮은 similarity 점수는 에이전트가 ground truth보다
    *더 나은* 답변을 제공했거나, `context`가 질문에 비해 너무 얇다는 뜻일 수 있습니다. Evaluation은
    에이전트만큼이나 데이터 세트도 채점합니다.

### 변경하고 증명하기

이것이 evaluation의 실제 용도입니다.

1. **data/caldova_eval.jsonl** 끝에 잘못된 테스트 사례를 추가합니다. `context`가 `ground_truth`를
    뒷받침하지 않는 질문이므로 에이전트가 답할 근거가 없습니다.

    ```json
    {"query": "What is the fast-track window for Halden Biologics?", "context": "Planning Desk Hours: Monday-Friday 8:00 AM - 6:00 PM.", "ground_truth": "Four months to first commercial batch."}
    ```

1. `python evaluate_agent.py`를 다시 실행하고 해당 행을 확인합니다. 에이전트의 답이 정확하더라도
    groundedness는 급격히 떨어져야 합니다. 주어진 context로 답변이 뒷받침되지 않기 때문입니다.
    이 구분이 바로 metric에서 hallucination이 보이는 방식입니다.

1. 완료되면 해당 행을 다시 제거합니다.

> ✅ **Checkpoint**: 에이전트 답변에 대한 반복 가능한 점수, 모든 점수의 행 수준 이유, 그리고 내일의
> 프롬프트 변경이 좋아졌는지 나빠졌는지 판단할 방법을 갖게 되었습니다. 이것이 이 랩의 Core입니다.
> 남은 작업은 선택 사항입니다.

완료되면 `deactivate`를 입력해 가상 환경을 종료합니다.

---

**다음(선택 사항):** [작업 3 — 에이전트 red team](D3-red-team-your-agent.md)
