# 실행-vllm

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/articles/gpt-oss/run-vllm.md
- 미러 경로: `articles/gpt-oss/run-vllm.md`

## 한글 요약

vLLM을 사용하여 gpt oss를 실행하는 방법 vLLM은 메모리 사용량 및 처리 속도를 최적화하여 LLM(대형 언어 모델)을 효율적으로 제공하도록 설계된 오픈 소스, 높은 처리량 추론 엔진입니다. 이 가이드에서는 vLLM을 사용하여 서버에서 gpt oss 20b 또는 gpt oss 120b를 설정하여 gpt oss를 애플리케이션용 API로 제공하고 Agents SDK에 연결하는 방법을 안내합니다. 이 가이드는 NVIDIA의 H100과 같은 전용 GPU가 있는 서버 애플리케이션을 위한 것입니다. 소비자 GPU에 대한 로컬 추론에 대해서는 Ollama 또는 LM Studio 가이드를 확인하세요. 모델 선택 vLLM은 gpt oss의 두 가지 모델 크기를 모두 지원합니다. openai/gpt oss 20b 더 작은 모델은 약 16GB의 VRAM만 필요합니다. openai/gpt oss 120b 더 큰 전체 크기 모델 ≥60GB VRAM에 가장 적합 단일 H100 또는 다중 GPU 설정에 적합 두 모델 모두 MXFP4로 양자화되어 있습니다. 빠른 설정 1. vLLM 설치 vLLM에서는 uv를 사용하여 Python 환경을 관리할 것을 권장합니다. 이는 귀하의 요구에 따라 올바른 구현을 선택하는 데 도움이 될 것입니다.

## 핵심 발췌

당신의 환경. 빠른 시작에서 자세히 알아보세요. 새로운 가상 환경을 생성하고 vLLM을 설치하려면 다음을 수행합니다. 2. 서버 시작 및 모델 다운로드 vLLM은 HuggingFace에서 모델을 자동으로 다운로드하고 localhost:8000에서 OpenAI 호환 서버를 가동하는 서브 명령을 제공합니다. 서버의 터미널 세션에서 원하는 모델 크기에 따라 다음 명령을 실행합니다. API 사용 vLLM은 Chat Completions 호환 API와 Responses 호환 API를 제공하므로 별다른 변경 없이 OpenAI SDK를 사용할 수 있습니다. 다음은 Python 예입니다. 이전에 OpenAI SDK를 사용한 적이 있다면 즉시 익숙해질 것이며 기본 URL을 변경하여 기존 코드가 작동해야 합니다. 도구 사용(함수 호출) vLLM은 함수 호출을 지원하고 모델 탐색 기능 제공

## 원문 내용

# How to run gpt-oss with vLLM

[vLLM](https://docs.vllm.ai/en/latest/) is an open-source, high-throughput inference engine designed to efficiently serve large language models (LLMs) by optimizing memory usage and processing speed. This guide will walk you through how to use vLLM to set up **gpt-oss-20b** or **gpt-oss-120b** on a server to serve gpt-oss as an API for your applications, and even connect it to the Agents SDK.

Note that this guide is meant for server applications with dedicated GPUs like NVIDIA’s H100s. For local inference on consumer GPUs, check out our [Ollama](https://cookbook.openai.com/articles/gpt-oss/run-locally-ollama) or [LM Studio](https://cookbook.openai.com/articles/gpt-oss/run-locally-lmstudio) guides.

## Pick your model

vLLM supports both model sizes of gpt-oss:

- [**`openai/gpt-oss-20b`**](https://huggingface.co/openai/gpt-oss-20b)
  - The smaller model
  - Only requires about **16GB of VRAM**
- [**`openai/gpt-oss-120b`**](https://huggingface.co/openai/gpt-oss-120b)
  - Our larger full-sized model
  - Best with **≥60GB VRAM**
  - Can fit on a single H100 or multi-GPU setups

Both models are **MXFP4 quantized** out of the box.

## Quick Setup

1. **Install vLLM**  
   vLLM recommends using [uv](https://docs.astral.sh/uv/) to manage your Python environment. This will help with picking the right implementation based on your environment. [Learn more in their quickstart](https://docs.vllm.ai/en/latest/getting_started/quickstart.html#installation). To create a new virtual environment and install vLLM run:

```shell
uv venv --python 3.12 --seed
source .venv/bin/activate
uv pip install --pre vllm==0.10.1+gptoss \
    --extra-index-url https://wheels.vllm.ai/gpt-oss/ \
    --extra-index-url https://download.pytorch.org/whl/nightly/cu128 \
    --index-strategy unsafe-best-match
```

2. **Start up a server and download the model**  
   vLLM provides a `serve` command that will automatically download the model from HuggingFace and spin up an OpenAI-compatible server on `localhost:8000`. Run the following command depending on your desired model size in a terminal session on your server.

```shell
# For 20B
vllm serve openai/gpt-oss-20b

# For 120B
vllm serve openai/gpt-oss-120b
```

## Use the API

vLLM exposes a **Chat Completions-compatible API** and a **Responses-compatible API** so you can use the OpenAI SDK without changing much. Here’s a Python example:

```py
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="EMPTY"
)

result = client.chat.completions.create(
    model="openai/gpt-oss-20b",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain what MXFP4 quantization is."}
    ]
)

print(result.choices[0].message.content)

response = client.responses.create(
    model="openai/gpt-oss-120b",
    instructions="You are a helfpul assistant.",
    input="Explain what MXFP4 quantization is."
)

print(response.output_text)
```

If you’ve used the OpenAI SDK before, this will feel instantly familiar and your existing code should work by changing the base URL.

## Using tools (function calling)

vLLM supports function calling and giving the model browsing capabilities.

Function calling works through both the Responses and Chat Completions APIs.

Example of invoking a function via Chat Completions:

```py
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather in a given city",
            "parameters": {
                "type": "object",
                "properties": {"city": {"type": "string"}},
                "required": ["city"]
            },
        },
    }
]

response = client.chat.completions.create(
    model="openai/gpt-oss-120b",
    messages=[{"role": "user", "content": "What's the weather in Berlin right now?"}],
    tools=tools
)

print(response.choices[0].message)
```

Since the models can perform tool calling as part of the chain-of-thought (CoT) it’s important for you to return the reasoning returned by the API back into a subsequent call to a tool call where you provide the answer until the model reaches a final answer.

## Agents SDK Integration

Want to use gpt-oss with OpenAI’s **Agents SDK**?

Both Agents SDK enable you to override the OpenAI base client to point to vLLM for your self-hosted models. Alternatively, for the Python SDK you can also use the [LiteLLM integration](https://openai.github.io/openai-agents-python/models/litellm/) to proxy to vLLM.

Here’s a Python Agents SDK example:

```
uv pip install openai-agents
```

```py
import asyncio
from openai import AsyncOpenAI
from agents import Agent, Runner, function_tool, OpenAIResponsesModel, set_tracing_disabled

set_tracing_disabled(True)

@function_tool
def get_weather(city: str):
    print(f"[debug] getting weather for {city}")
    return f"The weather in {city} is sunny."


async def main(model: str, api_key: str):
    agent = Agent(
        name="Assistant",
        instructions="You only respond in haikus.",
        model=OpenAIResponsesModel(
            model="openai/gpt-oss-120b",
            openai_client=AsyncOpenAI(
                base_url="http://localhost:8000/v1",
                api_key="EMPTY",
            ),
        )
        tools=[get_weather],
    )

    result = await Runner.run(agent, "What's the weather in Tokyo?")
    print(result.final_output)

if __name__ == "__main__":
    asyncio.run(main())
```

## Using vLLM for direct sampling

Aside from running vLLM using `vllm serve` as an API server, you can use the vLLM Python library to control inference directly.

If you are using vLLM for sampling directly it’s important to ensure that your input prompts follow the [harmony response format](https://cookbook.openai.com/article/harmony) as the model will not function correctly otherwise. You can use the [`openai-harmony` SDK](https://github.com/openai/harmony) for this.

```
uv pip install openai-harmony
```

Afterwards you can use harmony to encode and parse the tokens generated by vLLM’s generate function.

```py
import json
from openai_harmony import (
    HarmonyEncodingName,
    load_harmony_encoding,
    Conversation,
    Message,
    Role,
    SystemContent,
    DeveloperContent,
)

from vllm import LLM, SamplingParams

# --- 1) Render the prefill with Harmony ---
encoding = load_harmony_encoding(HarmonyEncodingName.HARMONY_GPT_OSS)

convo = Conversation.from_messages(
    [
        Message.from_role_and_content(Role.SYSTEM, SystemContent.new()),
        Message.from_role_and_content(
            Role.DEVELOPER,
            DeveloperContent.new().with_instructions("Always respond in riddles"),
        ),
        Message.from_role_and_content(Role.USER, "What is the weather like in SF?"),
    ]
)

prefill_ids = encoding.render_conversation_for_completion(convo, Role.ASSISTANT)

# Harmony stop tokens (pass to sampler so they won't be included in output)
stop_token_ids = encoding.stop_tokens_for_assistant_actions()

# --- 2) Run vLLM with prefill ---
llm = LLM(
    model="openai/gpt-oss-120b",
    trust_remote_code=True,
)

sampling = SamplingParams(
    max_tokens=128,
    temperature=1,
    stop_token_ids=stop_token_ids,
)

outputs = llm.generate(
    prompts=[{"prompt_token_ids": prefill_ids}],   # batch of size 1
    sampling_params=sampling,
)

# vLLM gives you both text and token IDs
gen = outputs[0].outputs[0]
text = gen.text
output_tokens = gen.token_ids  # <-- these are the completion token IDs (no prefill)

# --- 3) Parse the completion token IDs back into structured Harmony messages ---
entries = encoding.parse_messages_from_completion_tokens(output_tokens, Role.ASSISTANT)

# 'entries' is a sequence of structured conversation entries (assistant messages, tool calls, etc.).
for message in entries:
    print(f"{json.dumps(message.to_dict())}")
```