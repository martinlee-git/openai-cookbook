# 손잡이가 달린 침대

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/articles/gpt-oss/handle-raw-cot.md
- 미러 경로: `articles/gpt-oss/handle-raw-cot.md`

## 한글 요약

gpt oss에서 원시 사고 사슬을 처리하는 방법 gpt oss 모델은 모델 구현자의 분석 및 안전 연구를 위한 원시 CoT(사고 사슬)에 대한 액세스를 제공하지만 도구 호출이 CoT의 일부로 수행될 수 있으므로 도구 호출 성능에도 중요합니다. 동시에 원시 CoT에는 잠재적으로 유해한 콘텐츠가 포함되어 있거나 모델을 구현하는 사람이 의도하지 않은 정보(예: 모델에 제공된 지침에 지정된 규칙)가 사용자에게 공개될 수 있습니다. 따라서 최종 사용자에게 원시 CoT를 표시해서는 안 됩니다. 조화/채팅 템플릿 처리 모델은 조화 응답 형식의 일부로 원시 CoT를 인코딩합니다. 자체 채팅 템플릿을 작성하거나 토큰을 직접 처리하는 경우 먼저 조화 가이드를 확인하세요. 몇 가지 사항을 요약하면 다음과 같습니다. 1. CoT가 분석 채널에 발행됩니다. 2. 후속 샘플링 차례에서 최종 채널에 대한 메시지가 전송된 후 모든 분석 메시지가 삭제되어야 합니다. 함수 호출

## 핵심 발췌

해설 채널은 그대로 유지될 수 있습니다. 3. 어시스턴트의 마지막 메시지가 모든 유형의 도구 호출인 경우 이전 최종 메시지까지의 분석 메시지는 최종 메시지가 발행될 때까지 후속 샘플링에서 보존되어야 합니다. Chat Completions API Chat Completions API를 구현하는 경우 게시된 OpenAI 사양에는 생각의 사슬을 처리하기 위한 공식 사양이 없습니다. 호스팅 모델이 당분간 이 기능을 제공하지 않기 때문입니다. 대신 OpenRouter의 다음 규칙을 따르시기 바랍니다. 포함: 1. 원시 CoT는 추론: { 제외: true }가 요청의 일부로 지정되지 않는 한 응답의 일부로 반환됩니다. 자세한 내용은 여기를 참조하세요. 2. 원시 CoT는 출력 메시지의 추론 속성으로 노출됩니다. 3. 델타 이벤트의 경우 델타에는

## 원문 내용

# How to handle the raw chain of thought in gpt-oss

The [gpt-oss models](https://openai.com/open-models) provide access to a raw chain of thought (CoT) meant for analysis and safety research by model implementors, but it’s also crucial for the performance of tool calling, as tool calls can be performed as part of the CoT. At the same time, the raw CoT might contain potentially harmful content or could reveal information to users that the person implementing the model might not intend (like rules specified in the instructions given to the model). You therefore should not show raw CoT to end users.

## Harmony / chat template handling

The model encodes its raw CoT as part of our [harmony response format](https://cookbook.openai.com/articles/openai-harmony). If you are authoring your own chat templates or are handling tokens directly, make sure to [check out harmony guide first](https://cookbook.openai.com/articles/openai-harmony).

To summarize a couple of things:

1. CoT will be issued to the `analysis` channel
2. After a message to the `final` channel in a subsequent sampling turn all `analysis` messages should be dropped. Function calls to the `commentary` channel can remain
3. If the last message by the assistant was a tool call of any type, the analysis messages until the previous `final` message should be preserved on subsequent sampling until a `final` message gets issued

## Chat Completions API

If you are implementing a Chat Completions API, there is no official spec for handling chain of thought in the published OpenAI specs, as our hosted models will not offer this feature for the time being. We ask you to follow [the following convention from OpenRouter instead](https://openrouter.ai/docs/use-cases/reasoning-tokens). Including:

1. Raw CoT will be returned as part of the response unless `reasoning: { exclude: true }` is specified as part of the request. [See details here](https://openrouter.ai/docs/use-cases/reasoning-tokens#legacy-parameters)
2. The raw CoT is exposed as a `reasoning` property on the message in the output
3. For delta events the delta has a `reasoning` property
4. On subsequent turns you should be able to receive the previous reasoning (as `reasoning`) and handle it in accordance with the behavior specified in the chat template section above.

When in doubt, please follow the convention / behavior of the OpenRouter implementation.

## Responses API

For the Responses API we augmented our Responses API spec to cover this case. Below are the changes to the spec as type definitions. At a high level we are:

1. Introducing a new `content` property on `reasoning`. This allows a reasoning `summary` that could be displayed to the end user to be returned at the same time as the raw CoT (which should not be shown to the end user, but which might be helpful for interpretability research).
2. Introducing a new content type called `reasoning_text`
3. Introducing two new events `response.reasoning_text.delta` to stream the deltas of the raw CoT and `response.reasoning_text.done` to indicate a turn of CoT to be completed
4. On subsequent turns you should be able to receive the previous reasoning and handle it in accordance with the behavior specified in the chat template section above.

**Item type changes**

```typescript
type ReasoningItem = {
  id: string;
  type: "reasoning";
  summary: SummaryContent[];
  // new
  content: ReasoningTextContent[];
};

type ReasoningTextContent = {
  type: "reasoning_text";
  text: string;
};

type ReasoningTextDeltaEvent = {
  type: "response.reasoning_text.delta";
  sequence_number: number;
  item_id: string;
  output_index: number;
  content_index: number;
  delta: string;
};

type ReasoningTextDoneEvent = {
  type: "response.reasoning_text.done";
  sequence_number: number;
  item_id: string;
  output_index: number;
  content_index: number;
  text: string;
};
```

**Event changes**

```typescript
...
{
	type: "response.content_part.added"
	...
}
{
	type: "response.reasoning_text.delta",
	sequence_number: 14,
	item_id: "rs_67f47a642e788191aec9b5c1a35ab3c3016f2c95937d6e91",
	output_index: 0,
	content_index: 0,
	delta: "The "
}
...
{
	type: "response.reasoning_text.done",
	sequence_number: 18,
	item_id: "rs_67f47a642e788191aec9b5c1a35ab3c3016f2c95937d6e91",
	output_index: 0,
	content_index: 0,
	text: "The user asked me to think"
}
```

**Example responses output**

```typescript
"output": [
  {
    "type": "reasoning",
    "id": "rs_67f47a642e788191aec9b5c1a35ab3c3016f2c95937d6e91",
    "summary": [
      {
        "type": "summary_text",
        "text": "**Calculating volume of gold for Pluto layer**\n\nStarting with the approximation..."
      }
    ],
    "content": [
      {
        "type": "reasoning_text",
        "text": "The user asked me to think..."
      }
    ]
  }
]

```

## Displaying raw CoT to end-users

If you are providing a chat interface to users, you should not show the raw CoT because it might contain potentially harmful content or other information that you might not intend to show to users (like, for example, instructions in the developer message). Instead, we recommend showing a summarized CoT, similar to our production implementations in the API or ChatGPT, where a summarizer model reviews and blocks harmful content from being shown.