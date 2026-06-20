# 로컬로 실행-올라마

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/articles/gpt-oss/run-locally-ollama.md
- 미러 경로: `articles/gpt-oss/run-locally-ollama.md`

## 한글 요약

Ollama를 사용하여 로컬에서 gpt oss를 실행하는 방법 OpenAI gpt oss를 자신의 하드웨어에서 실행하고 싶으십니까? 이 가이드는 Ollama를 사용하여 gpt oss 20b 또는 gpt oss 120b를 로컬로 설정하고, 오프라인으로 채팅하고, API를 통해 사용하고, Agents SDK에 연결하는 방법을 안내합니다. 이 가이드는 PC 또는 Mac에서 모델을 실행하는 것과 같은 소비자 하드웨어를 위한 것입니다. NVIDIA의 H100과 같은 전용 GPU가 있는 서버 애플리케이션의 경우 vLLM 가이드를 확인하세요. 모델 선택 Ollama는 gpt oss의 두 가지 모델 크기를 모두 지원합니다: gpt oss 20b 더 작은 모델 ≥16GB VRAM 또는 통합 메모리에 가장 적합 고급 소비자 GPU 또는 Apple Silicon Mac에 적합 gpt oss 120b 더 큰 풀 사이즈 모델 ≥60GB VRAM 또는 통합 메모리에 가장 적합 멀티 GPU 또는 강력한 워크스테이션 설정에 이상적 몇 가지 참고 사항: 이 모델은 즉시 양자화된 MXFP4를 제공하며 현재 다른 양자화는 없습니다. VRAM이 부족한 경우 CPU로 오프로드할 수 있지만 실행 속도가 느려질 것으로 예상됩니다. 빠른 설정 1. Ollama 설치 → 다운로드

## 핵심 발췌

2. 원하는 모델을 선택하세요. gpt oss와 채팅하세요. 모델과 대화할 준비가 되셨나요? 앱이나 터미널에서 채팅을 시작할 수 있습니다. Ollama는 OpenAI 조화 형식을 모방한 기본 채팅 템플릿을 적용합니다. 메시지를 입력하고 대화를 시작하세요. API 사용 Ollama는 Chat Completions 호환 API를 공개하므로 별다른 변경 없이 OpenAI SDK를 사용할 수 있습니다. 다음은 Python 예입니다. 이전에 OpenAI SDK를 사용해 본 적이 있다면 즉시 익숙해질 것입니다. 또는 Python 또는 JavaScript에서 Ollama SDK를 직접 사용할 수 있습니다. 도구 사용(함수 호출) Ollama는 다음을 수행할 수 있습니다. 기능 호출 내장된 브라우저 도구 사용(앱에서) 채팅 완료를 통한 함수 호출의 예: 모델이 CoT(사고 사슬)의 일부로 도구 호출을 수행할 수 있으므로 중요합니다.

## 원문 내용

# How to run gpt-oss locally with Ollama

Want to get [**OpenAI gpt-oss**](https://openai.com/open-models) running on your own hardware? This guide will walk you through how to use [Ollama](https://ollama.ai) to set up **gpt-oss-20b** or **gpt-oss-120b** locally, to chat with it offline, use it through an API, and even connect it to the Agents SDK.

Note that this guide is meant for consumer hardware, like running a model on a PC or Mac. For server applications with dedicated GPUs like NVIDIA’s H100s, [check out our vLLM guide](https://cookbook.openai.com/articles/gpt-oss/run-vllm).

## Pick your model

Ollama supports both model sizes of gpt-oss:

- **`gpt-oss-20b`**
  - The smaller model
  - Best with **≥16GB VRAM** or **unified memory**
  - Perfect for higher-end consumer GPUs or Apple Silicon Macs
- **`gpt-oss-120b`**
  - Our larger full-sized model
  - Best with **≥60GB VRAM** or **unified memory**
  - Ideal for multi-GPU or beefy workstation setup

**A couple of notes:**

- These models ship **MXFP4 quantized** out the box and there is currently no other quantization
- You _can_ offload to CPU if you’re short on VRAM, but expect it to run slower.

## Quick setup

1. **Install Ollama** → [Get it here](https://ollama.com/download)
2. **Pull the model you want:**

```shell
# For 20B
ollama pull gpt-oss:20b

# For 120B
ollama pull gpt-oss:120b
```

## Chat with gpt-oss

Ready to talk to the model? You can fire up a chat in the app or the terminal:

```shell
ollama run gpt-oss:20b
```

Ollama applies a **chat template** out of the box that mimics the [OpenAI harmony format](https://cookbook.openai.com/articles/openai-harmony). Type your message and start the conversation.

## Use the API

Ollama exposes a **Chat Completions-compatible API**, so you can use the OpenAI SDK without changing much. Here’s a Python example:

```py
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",  # Local Ollama API
    api_key="ollama"                       # Dummy key
)

response = client.chat.completions.create(
    model="gpt-oss:20b",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain what MXFP4 quantization is."}
    ]
)

print(response.choices[0].message.content)
```

If you’ve used the OpenAI SDK before, this will feel instantly familiar.

Alternatively, you can use the Ollama SDKs in [Python](https://github.com/ollama/ollama-python) or [JavaScript](https://github.com/ollama/ollama-js) directly.

## Using tools (function calling)

Ollama can:

- Call functions
- Use a **built-in browser tool** (in the app)

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
    model="gpt-oss:20b",
    messages=[{"role": "user", "content": "What's the weather in Berlin right now?"}],
    tools=tools
)

print(response.choices[0].message)
```

Since the models can perform tool calling as part of the chain-of-thought (CoT) it’s important for you to return the reasoning returned by the API back into a subsequent call to a tool call where you provide the answer until the model reaches a final answer.

## Responses API workarounds

Ollama doesn’t (yet) support the **Responses API** natively.

If you do want to use the Responses API you can use [**Hugging Face’s `Responses.js` proxy**](https://github.com/huggingface/responses.js) to convert Chat Completions to Responses API.

For basic use cases you can also [**run our example Python server with Ollama as the backend.**](https://github.com/openai/gpt-oss?tab=readme-ov-file#responses-api) This server is a basic example server and does not have the

```shell
pip install gpt-oss
python -m gpt_oss.responses_api.serve \
    --inference_backend=ollama \
    --checkpoint gpt-oss:20b
```

## Agents SDK integration

Want to use gpt-oss with OpenAI’s **Agents SDK**?

Both Agents SDK enable you to override the OpenAI base client to point to Ollama using Chat Completions or your Responses.js proxy for your local models. Alternatively, you can use the built-in functionality to point the Agents SDK against third party models.

- **Python:** Use [LiteLLM](https://openai.github.io/openai-agents-python/models/litellm/) to proxy to Ollama through LiteLLM
- **TypeScript:** Use [AI SDK](https://openai.github.io/openai-agents-js/extensions/ai-sdk/) with the [ollama adapter](https://ai-sdk.dev/providers/community-providers/ollama)

Here’s a Python Agents SDK example using LiteLLM:

```py
import asyncio
from agents import Agent, Runner, function_tool, set_tracing_disabled
from agents.extensions.models.litellm_model import LitellmModel

set_tracing_disabled(True)

@function_tool
def get_weather(city: str):
    print(f"[debug] getting weather for {city}")
    return f"The weather in {city} is sunny."


async def main(model: str, api_key: str):
    agent = Agent(
        name="Assistant",
        instructions="You only respond in haikus.",
        model=LitellmModel(model="ollama/gpt-oss:120b", api_key=api_key),
        tools=[get_weather],
    )

    result = await Runner.run(agent, "What's the weather in Tokyo?")
    print(result.final_output)

if __name__ == "__main__":
    asyncio.run(main())
```