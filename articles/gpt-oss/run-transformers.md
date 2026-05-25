# 실행 변압기

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/articles/gpt-oss/run-transformers.md
- 미러 경로: `articles/gpt-oss/run-transformers.md`

## 한글 요약

Hugging Face Transformers로 gpt oss를 실행하는 방법 Hugging Face의 Transformers 라이브러리는 대규모 언어 모델을 로컬로 또는 서버에서 로드하고 실행할 수 있는 유연한 방법을 제공합니다. 이 가이드는 Transformers를 사용하여 상위 수준 파이프라인 또는 원시 토큰 ID가 있는 하위 수준 생성 호출을 통해 OpenAI gpt oss 20b 또는 OpenAI gpt oss 120b를 실행하는 과정을 안내합니다. 우리는 OpenAI gpt oss 20b 또는 OpenAI gpt oss 120b의 사용에 대해 높은 수준의 파이프라인 추상화, 낮은 수준의 \generate\ 호출, Responses API와 호환되는 방식으로 \transformers Serve\를 통한 로컬 제공 모델을 다룰 것입니다. 이 가이드에서는 Transformer를 통해 gpt oss 모델을 실행하는 최적화된 다양한 방법을 살펴보겠습니다. 보너스: 변압기를 통해 모델을 미세 조정할 수도 있습니다. 여기에서 미세 조정 가이드를 확인하세요. 모델 선택 Hugging Face에서 두 가지 gpt oss 모델을 모두 사용할 수 있습니다. MXFP4 사용 시 openai/gpt oss 20b \ 16GB VRAM 요구 사항 단일 고급 소비자 GPU에 적합 openai/gpt oss 120b ≥60GB VR 필요

## 핵심 발췌

AM 또는 다중 GPU 설정 H100 클래스 하드웨어에 이상적입니다. 둘 다 기본적으로 MXFP4 양자화됩니다. MXFP4는 Hopper 이상의 아키텍처에서 지원됩니다. 여기에는 H100 또는 GB200과 같은 데이터 센터 GPU와 최신 RTX 50xx 소비자 카드 제품군이 포함됩니다. MXFP4 대신 bfloat16을 사용하면 메모리 소비가 더 커집니다(20b 매개변수 모델의 경우 \48GB). 빠른 설정 1. 종속성 설치 새로운 Python 환경을 만드는 것이 좋습니다. MXFP4 호환성을 위해 변환기, 가속 및 Triton 커널을 설치합니다. 2. (선택 사항) 다중 GPU 활성화 대규모 모델을 실행하는 경우 Accelerate 또는 torchrun을 사용하여 장치 매핑을 자동으로 처리합니다. Open AI 응답/채팅 완료 엔드포인트 생성 서버를 시작하려면 변환기 서비스를 사용하면 됩니다.

## 원문 내용

# How to run gpt-oss with Hugging Face Transformers

The Transformers library by Hugging Face provides a flexible way to load and run large language models locally or on a server. This guide will walk you through running [OpenAI gpt-oss-20b](https://huggingface.co/openai/gpt-oss-20b) or [OpenAI gpt-oss-120b](https://huggingface.co/openai/gpt-oss-120b) using Transformers, either with a high-level pipeline or via low-level `generate` calls with raw token IDs.

We'll cover the use of [OpenAI gpt-oss-20b](https://huggingface.co/openai/gpt-oss-20b) or [OpenAI gpt-oss-120b](https://huggingface.co/openai/gpt-oss-120b) with the high-level pipeline abstraction, low-level \`generate\` calls, and serving models locally with \`transformers serve\`, with in a way compatible with the Responses API.

In this guide we’ll run through various optimised ways to run the **gpt-oss models via Transformers.**

Bonus: You can also fine-tune models via transformers, [check out our fine-tuning guide here](https://cookbook.openai.com/articles/gpt-oss/fine-tune-transformers).

## Pick your model

Both **gpt-oss** models are available on Hugging Face:

- **`openai/gpt-oss-20b`**
  - \~16GB VRAM requirement when using MXFP4
  - Great for single high-end consumer GPUs
- **`openai/gpt-oss-120b`**
  - Requires ≥60GB VRAM or multi-GPU setup
  - Ideal for H100-class hardware

Both are **MXFP4 quantized** by default. Please, note that MXFP4 is supported in Hopper or later architectures. This includes data center GPUs such as H100 or GB200, as well as the latest RTX 50xx family of consumer cards.

If you use `bfloat16` instead of MXFP4, memory consumption will be larger (\~48 GB for the 20b parameter model).

## Quick setup

1. **Install dependencies**  
   It’s recommended to create a fresh Python environment. Install transformers, accelerate, as well as the Triton kernels for MXFP4 compatibility:

```bash
pip install -U transformers accelerate torch triton==3.4 kernels
```

2. **(Optional) Enable multi-GPU**  
   If you’re running large models, use Accelerate or torchrun to handle device mapping automatically.

## Create an Open AI Responses / Chat Completions endpoint

To launch a server, simply use the `transformers serve` CLI command:

```bash
transformers serve
```

The simplest way to interact with the server is through the transformers chat CLI

```bash
transformers chat localhost:8000 --model-name-or-path openai/gpt-oss-20b
```

or by sending an HTTP request with cURL, e.g.

```bash
curl -X POST http://localhost:8000/v1/responses -H "Content-Type: application/json" -d '{"messages": [{"role": "system", "content": "hello"}], "temperature": 0.9, "max_tokens": 1000, "stream": true, "model": "openai/gpt-oss-20b"}'
```

Additional use cases, like integrating `transformers serve` with Cursor and other tools, are detailed in [the documentation](https://huggingface.co/docs/transformers/main/serving).

## Quick inference with pipeline

The easiest way to run the gpt-oss models is with the Transformers high-level `pipeline` API:

```py
from transformers import pipeline

generator = pipeline(
    "text-generation",
    model="openai/gpt-oss-20b",
    torch_dtype="auto",
    device_map="auto"  # Automatically place on available GPUs
)

messages = [
    {"role": "user", "content": "Explain what MXFP4 quantization is."},
]

result = generator(
    messages,
    max_new_tokens=200,
    temperature=1.0,
)

print(result[0]["generated_text"])
```

## Advanced inference with `.generate()`

If you want more control, you can load the model and tokenizer manually and invoke the `.generate()` method:

```py
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "openai/gpt-oss-20b"

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype="auto",
    device_map="auto"
)

messages = [
    {"role": "user", "content": "Explain what MXFP4 quantization is."},
]

inputs = tokenizer.apply_chat_template(
    messages,
    add_generation_prompt=True,
    return_tensors="pt",
    return_dict=True,
).to(model.device)

outputs = model.generate(
    **inputs,
    max_new_tokens=200,
    temperature=0.7
)

print(tokenizer.decode(outputs[0]))
```

## Chat template and tool calling

OpenAI gpt-oss models use the [harmony response format](https://cookbook.openai.com/article/harmony) for structuring messages, including reasoning and tool calls.

To construct prompts you can use the built-in chat template of Transformers. Alternatively, you can install and use the [openai-harmony library](https://github.com/openai/harmony) for more control.

To use the chat template:

```py
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "openai/gpt-oss-20b"

tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    device_map="auto",
    torch_dtype="auto",
)

messages = [
    {"role": "system", "content": "Always respond in riddles"},
    {"role": "user", "content": "What is the weather like in Madrid?"},
]

inputs = tokenizer.apply_chat_template(
    messages,
    add_generation_prompt=True,
    return_tensors="pt",
    return_dict=True,
).to(model.device)

generated = model.generate(**inputs, max_new_tokens=100)
print(tokenizer.decode(generated[0][inputs["input_ids"].shape[-1] :]))
```

To integrate the [`openai-harmony`](https://github.com/openai/harmony) library to prepare prompts and parse responses, first install it like this:

```bash
pip install openai-harmony
```

Here’s an example of how to use the library to build your prompts and encode them to tokens:

```py
import json
from openai_harmony import (
    HarmonyEncodingName,
    load_harmony_encoding,
    Conversation,
    Message,
    Role,
    SystemContent,
    DeveloperContent
)
from transformers import AutoModelForCausalLM, AutoTokenizer

encoding = load_harmony_encoding(HarmonyEncodingName.HARMONY_GPT_OSS)

# Build conversation
convo = Conversation.from_messages([
    Message.from_role_and_content(Role.SYSTEM, SystemContent.new()),
    Message.from_role_and_content(
        Role.DEVELOPER,
        DeveloperContent.new().with_instructions("Always respond in riddles")
    ),
    Message.from_role_and_content(Role.USER, "What is the weather like in SF?")
])

# Render prompt
prefill_ids = encoding.render_conversation_for_completion(convo, Role.ASSISTANT)
stop_token_ids = encoding.stop_tokens_for_assistant_actions()

# Load model
model_name = "openai/gpt-oss-20b"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, torch_dtype="auto", device_map="auto")

# Generate
outputs = model.generate(
    input_ids=[prefill_ids],
    max_new_tokens=128,
    eos_token_id=stop_token_ids
)

# Parse completion tokens
completion_ids = outputs[0][len(prefill_ids):]
entries = encoding.parse_messages_from_completion_tokens(completion_ids, Role.ASSISTANT)

for message in entries:
    print(json.dumps(message.to_dict(), indent=2))
```

Note that the `Developer` role in Harmony maps to the `system` prompt in the chat template.

## Multi-GPU & distributed inference

The large gpt-oss-120b fits on a single H100 GPU when using MXFP4. If you want to run it on multiple GPUs, you can:

- Use `tp_plan="auto"` for automatic placement and tensor parallelism
- Launch with `accelerate launch or torchrun` for distributed setups
- Leverage Expert Parallelism
- Use specialised Flash attention kernels for faster inference

```py
from transformers import AutoModelForCausalLM, AutoTokenizer
from transformers.distributed import DistributedConfig
import torch

model_path = "openai/gpt-oss-120b"
tokenizer = AutoTokenizer.from_pretrained(model_path, padding_side="left")

device_map = {
    # Enable Expert Parallelism
    "distributed_config": DistributedConfig(enable_expert_parallel=1),
    # Enable Tensor Parallelism
    "tp_plan": "auto",
}

model = AutoModelForCausalLM.from_pretrained(
    model_path,
    torch_dtype="auto",
    attn_implementation="kernels-community/vllm-flash-attn3",
    **device_map,
)

messages = [
     {"role": "user", "content": "Explain how expert parallelism works in large language models."}
]

inputs = tokenizer.apply_chat_template(
    messages,
    add_generation_prompt=True,
    return_tensors="pt",
    return_dict=True,
).to(model.device)

outputs = model.generate(**inputs, max_new_tokens=1000)

# Decode and print
response = tokenizer.decode(outputs[0])
print("Model response:", response.split("<|channel|>final<|message|>")[-1].strip())
```

You can then run this on a node with four GPUs via

```bash
torchrun --nproc_per_node=4 generate.py
```