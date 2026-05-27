# 구현 확인

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/articles/gpt-oss/verifying-implementations.md
- 미러 경로: `articles/gpt-oss/verifying-implementations.md`

## 한글 요약

gpt oss 구현 확인 OpenAI gpt oss 모델은 개방형 모델 생태계에 많은 새로운 개념을 도입하고 있으며 예상대로 작동하려면 시간이 걸릴 수 있습니다. 이 가이드는 개발자가 구현을 확인하기 위해 추론 솔루션을 구축하는 데 도움을 주거나 공급자의 구현을 스스로 테스트하여 자신감을 얻으려는 개발자를 돕기 위해 작성되었습니다. gpt oss 모델 구현이 다른 이유는 무엇입니까? 새로운 모델은 기존 개방형 모델보다 다른 일부 OpenAI 모델과 더 유사하게 작동합니다. 몇 가지 예는 다음과 같습니다. 1. 조화 응답 형식. 이러한 모델은 대화를 구성하기 위해 OpenAI 조화 형식으로 훈련되었습니다. 일반 API 개발자는 대부분의 경우 조화를 처리할 필요가 없지만 채팅 완료 호환, 응답 호환 또는 기타 추론 API를 제공하는 추론 공급자는 입력을 OpenAI 조화 형식에 올바르게 매핑해야 합니다. 모델이 올바른 형식의 프롬프트를 받지 못하는 경우 이 CA

## 핵심 발췌

n 계단식 생성 문제가 있고 최소한 성능 호출 기능이 더 나쁩니다. 2. 도구 호출 간 CoT(사고 사슬) 처리. 이러한 모델은 CoT의 일부로 도구 호출을 수행할 수 있습니다. 결과적으로 모델은 최종 응답에 도달할 때까지 후속 샘플링에서 CoT를 수신해야 합니다. 이는 원시 CoT가 최종 사용자에게 표시되어서는 안 되지만 개발자가 도구 호출 및 도구 출력과 함께 CoT를 다시 전달할 수 있도록 API에 의해 반환되어야 함을 의미합니다. 이에 대한 자세한 내용은 별도의 가이드에서 확인할 수 있습니다. 3. 실제 추론 코드의 차이점. 우리는 MXFP4 형식으로만 MoE(전문가 혼합) 가중치를 게시했습니다. 이는 여전히 상대적으로 새로운 형식이며 다른 아키텍처 결정과 함께 작성된 기존 추론 코드입니다.

## 원문 내용

# Verifying gpt-oss implementations

The [OpenAI gpt-oss models](https://openai.com/open-models) are introducing a lot of new concepts to the open-model ecosystem and [getting them to perform as expected might take some time](https://x.com/ClementDelangue/status/1953119901649891367). This guide is meant to help developers building inference solutions to verify their implementations or for developers who want to test any provider’s implementation on their own to gain confidence.

## Why is implementing gpt-oss models different?

The new models behave more similarly to some of our other OpenAI models than to existing open models. A couple of examples include:

1. **The harmony response format.** These models were trained on our [OpenAI harmony format](https://cookbook.openai.com/articles/openai-harmony) to structure a conversation. While regular API developers won’t need to deal with harmony in most cases, the inference providers that provide a Chat Completions-compatible, Responses-compatible or other inference API need to map the inputs correctly to the OpenAI harmony format. If the model does not receive the prompts in the right format this can have cascading generation issues and at minimum a worse function calling performance.
2. **Handling chain of thought (CoT) between tool calls**. These models can perform tool calls as part of the CoT. A consequence of this is that the model needs to receive the CoT in subsequent sampling until it reaches a final response. This means that while the raw CoT should not be displayed to end-users, it should be returned by APIs so that developers can pass it back in along with the tool call and tool output. [You can learn more about it in this separate guide](https://cookbook.openai.com/articles/gpt-oss/handle-raw-cot).
3. **Differences in actual inference code**. We published our mixture-of-experts (MoE) weights exclusively in MXFP4 format. This is still a relatively new format and along with other architecture decisions, existing inference code that was written for other open-models will have to be adapted for gpt-oss models. For that reason we published both a basic (unoptimized) [PyTorch implementation](https://github.com/openai/gpt-oss/tree/main/gpt_oss/torch), and a [more optimized Triton implementation](https://github.com/openai/gpt-oss/tree/main/gpt_oss/triton). Additionally, we verified the [vLLM implementation](https://github.com/vllm-project/vllm/blob/7e3a8dc90670fd312ce1e0d4eba9bf11c571e3ad/vllm/model_executor/models/gpt_oss.py) for correctness. We hope these can serve as educational material for other implementations.

## API Design

### Responses API

For best performance we recommend inference providers to implement our Responses API format as the API shape was specifically designed for behaviors like outputting raw CoT along with summarized CoTs (to display to users) and tool calls without bolting additional properties onto a format. The most important  
part for accurate performance is to return the raw CoT as part of the `output`.

For this we added a new `content` array to the Responses API’s `reasoning` items. The raw CoT should be wrapped into `reasoning_text` type element, making the overall output item look the following:

```
{
  "type": "reasoning",
  "id": "item_67ccd2bf17f0819081ff3bb2cf6508e60bb6a6b452d3795b",
  "status": "completed",
  "summary": [
    /* optional summary elements */
  ],
  "content": [
    {
      "type": "reasoning_text",
      "text": "The user needs to know the weather, I will call the get_weather tool."
    }
  ]
}
```

These items should be received in subsequent turns and then inserted back into the harmony formatted prompt as outlined in the [raw CoT handling guide](https://cookbook.openai.com/articles/gpt-oss/handle-raw-cot).

[Check out the Responses API docs for the whole specification](https://platform.openai.com/docs/api-reference/responses/create).

### Chat Completions

A lot of providers are offering a Chat Completions-compatible API. While we have not augmented our published API reference on the docs to provide a way to receive raw CoT, it’s still important that providers that offer the gpt-oss models via a Chat Completions-compatible API return the CoT as part of their messages and for developers to have a way to pass them back.

There is currently no generally agreed upon specification in the community with the general properties on a message being either `reasoning` or `reasoning_content`. **To be compatible with clients like the OpenAI Agents SDK we recommend using a `reasoning` field as the primary property for the raw CoT in Chat Completions**.

## Quick verification of tool calling and API shapes

To verify if a provider is working you can use the Node.js script published in our [gpt-oss GitHub repository](https://github.com/openai/gpt-oss) that you can also use to run other evals. You’ll need [Node.js](http://nodejs.org/) or a similar runtime installed to run the tests.

These tests will run a series of tool/function calling based requests to the Responses API or Chat Completions API you are trying to test. Afterwards they will evaluate both whether the right tool was called and whether the API shapes are correct.

This largely acts as a smoke test but should be a good indicator on whether the APIs are compatible with our SDKs and can handle basic function calling. It does not guarantee full accuracy of the inference implementation (see the evals section below for details on that) nor does it guarantee full compatibility with the OpenAI APIs. They should still be a helpful indicator of major implementation issues.

To run the test suite run the following commands:

```shell
# clone the repository
git clone https://github.com/openai/gpt-oss.git

# go into the compatibility test directory
cd gpt-oss/compatibility-test/

# install the dependencies
npm install

# change the provider config in providers.ts to add your provider

# run the tests
npm start -- --provider <your-provider-name>
```

Afterwards you should receive a result of both the API implementation and any details on the function call performance.

If your tests are successful, the output should show 0 invalid requests and over 90% on both pass@k and pass^k. This means the implementation should likely be correct. To be fully sure, you should also inspect the evals as described below.

If you want a detailed view of the individual responses, you can the `jsonl` file that was created in your directory.

You can also enable debug mode to view any of the actual request payloads using `DEBUG=openai-agents:openai npm start -- --provider <provider-name>` but it might get noisy. To run only one test use the `-n 1` flag for easier debugging. For testing streaming events you can use `--streaming`.

## Verifying correctness through evals

The team at Artificial Analysis is running AIME and GPQA evals for a variety of providers. If you are unsure about your provider, [check out Artificial Analysis for the most recent metrics](https://artificialanalysis.ai/models/gpt-oss-120b/providers#evaluations).

To be on the safe side you should consider running evals yourself. To run your own evals, you can find in the same repository as the test above a `gpt_oss/evals` folder that contains the test harnesses that we used to verify the AIME (16 attempts per problem), GPQA (8 attempts per problem) and Healthbench (1 attempt per problem) evals for the vLLM implementation and some of our own reference implementations. You can use the same script to test your implementations.

To test a Responses API compatible API run:

```bash
python -m gpt_oss.evals --base-url http://localhost:8000/v1 --eval aime25 --sampler responses --model openai/gpt-oss-120b --reasoning-effort high
```

To test a Chat Completions API compatible API run:

```bash
python -m gpt_oss.evals --base-url http://localhost:8000/v1 --eval aime25 --sampler chat_completions --model openai/gpt-oss-120b --reasoning-effort high
```

If you are getting similar benchmark results as those published by us and your function calling tests above succeeded you likely have a correct implementation of gpt-oss.