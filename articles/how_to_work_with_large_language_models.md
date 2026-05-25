# How_to_work_with_large_언어_모델

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/articles/how_to_work_with_large_language_models.md
- 미러 경로: `articles/how_to_work_with_large_language_models.md`

## 한글 요약

대규모 언어 모델을 사용하는 방법 대규모 언어 모델이 작동하는 방식 [대형 언어 모델][대형 언어 모델 블로그 게시물]은 텍스트를 텍스트로 매핑하는 기능입니다. 입력된 텍스트 문자열이 주어지면 대규모 언어 모델이 다음에 올 텍스트를 예측합니다. 대규모 언어 모델의 마법은 방대한 양의 텍스트에 대한 예측 오류를 최소화하도록 학습함으로써 모델이 결국 이러한 예측에 유용한 개념을 학습하게 된다는 것입니다. 예를 들어, 문법의 철자법, 질문에 답하는 방법, 대화를 진행하는 방법, 여러 언어로 작성하는 방법, 코딩하는 방법 등을 배웁니다. 많은 양의 기존 텍스트를 "읽고" 단어가 다른 단어와의 맥락에서 어떻게 나타나는 경향이 있는지 학습하고, 학습한 내용을 사용하여 사용자 요청에 응답하여 나타날 가능성이 가장 높은 다음 단어와 그 이후의 각 단어를 예측합니다. GPT 3 및 GPT 4는 생산성 앱을 포함한 [많은 소프트웨어 제품][OpenAI 고객 사례]을 지원합니다.

## 핵심 발췌

교육 앱, 게임 등 대규모 언어 모델을 제어하는 ​​방법 대규모 언어 모델에 대한 모든 입력 중에서 단연 가장 영향력 있는 것은 텍스트 프롬프트입니다. 대형 언어 모델은 다음과 같은 몇 가지 방법으로 출력을 생성하도록 프롬프트될 수 있습니다. 지침: 원하는 것을 모델에 알리기 완료: 모델이 원하는 것의 시작을 완료하도록 유도 시나리오: 모델에 실행할 상황 제공 데모: 원하는 것을 모델에 표시합니다. 다음 중 하나를 사용합니다. 프롬프트의 몇 가지 예 미세 조정 훈련 데이터 세트에 있는 수백 또는 수천 개의 예 각 예가 아래에 나와 있습니다. 지침 프롬프트 프롬프트 상단(또는 하단 또는 둘 다)에 지침을 작성하면 모델이 최선을 다해 지침을 따른 후 중지됩니다. 자세한 안내가 가능하므로

## 원문 내용

# How to work with large language models

## How large language models work

[Large language models][Large language models Blog Post] are functions that map text to text. Given an input string of text, a large language model predicts the text that should come next.

The magic of large language models is that by being trained to minimize this prediction error over vast quantities of text, the models end up learning concepts useful for these predictions. For example, they learn:

- how to spell
- how grammar works
- how to paraphrase
- how to answer questions
- how to hold a conversation
- how to write in many languages
- how to code
- etc.

They do this by “reading” a large amount of existing text and learning how words tend to appear in context with other words, and uses what it has learned to predict the next most likely word that might appear in response to a user request, and each subsequent word after that.

GPT-3 and GPT-4 power [many software products][OpenAI Customer Stories], including productivity apps, education apps, games, and more.

## How to control a large language model

Of all the inputs to a large language model, by far the most influential is the text prompt.

Large language models can be prompted to produce output in a few ways:

- **Instruction**: Tell the model what you want
- **Completion**: Induce the model to complete the beginning of what you want
- **Scenario**: Give the model a situation to play out
- **Demonstration**: Show the model what you want, with either:
  - A few examples in the prompt
  - Many hundreds or thousands of examples in a fine-tuning training dataset

An example of each is shown below.

### Instruction prompts

Write your instruction at the top of the prompt (or at the bottom, or both), and the model will do its best to follow the instruction and then stop. Instructions can be detailed, so don't be afraid to write a paragraph explicitly detailing the output you want, just stay aware of how many [tokens](https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them) the model can process.

Example instruction prompt:

```text
Extract the name of the author from the quotation below.

“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation
```

Output:

```text
Ted Chiang
```

### Completion prompt example

Completion-style prompts take advantage of how large language models try to write text they think is most likely to come next. To steer the model, try beginning a pattern or sentence that will be completed by the output you want to see. Relative to direct instructions, this mode of steering large language models can take more care and experimentation. In addition, the models won't necessarily know where to stop, so you will often need stop sequences or post-processing to cut off text generated beyond the desired output.

Example completion prompt:

```text
“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation

The author of this quote is
```

Output:

```text
 Ted Chiang
```

### Scenario prompt example

Giving the model a scenario to follow or role to play out can be helpful for complex queries or when seeking imaginative responses. When using a hypothetical prompt, you set up a situation, problem, or story, and then ask the model to respond as if it were a character in that scenario or an expert on the topic.

Example scenario prompt:

```text
Your role is to extract the name of the author from any given text

“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation
```

Output:

```text
 Ted Chiang
```

### Demonstration prompt example (few-shot learning)

Similar to completion-style prompts, demonstrations can show the model what you want it to do. This approach is sometimes called few-shot learning, as the model learns from a few examples provided in the prompt.

Example demonstration prompt:

```text
Quote:
“When the reasoning mind is forced to confront the impossible again and again, it has no choice but to adapt.”
― N.K. Jemisin, The Fifth Season
Author: N.K. Jemisin

Quote:
“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation
Author:
```

Output:

```text
 Ted Chiang
```

### Fine-tuned prompt example

With enough training examples, you can [fine-tune][Fine Tuning Docs] a custom model. In this case, instructions become unnecessary, as the model can learn the task from the training data provided. However, it can be helpful to include separator sequences (e.g., `->` or `###` or any string that doesn't commonly appear in your inputs) to tell the model when the prompt has ended and the output should begin. Without separator sequences, there is a risk that the model continues elaborating on the input text rather than starting on the answer you want to see.

Example fine-tuned prompt (for a model that has been custom trained on similar prompt-completion pairs):

```text
“Some humans theorize that intelligent species go extinct before they can expand into outer space. If they're correct, then the hush of the night sky is the silence of the graveyard.”
― Ted Chiang, Exhalation

###


```

Output:

```text
 Ted Chiang
```

## Code Capabilities

Large language models aren't only great at text - they can be great at code too. OpenAI's [GPT-4][GPT-4 and GPT-4 Turbo] model is a prime example.

GPT-4 powers [numerous innovative products][OpenAI Customer Stories], including:

- [GitHub Copilot] (autocompletes code in Visual Studio and other IDEs)
- [Replit](https://replit.com/) (can complete, explain, edit and generate code)
- [Cursor](https://cursor.sh/) (build software faster in an editor designed for pair-programming with AI)

GPT-4 is more advanced than previous models like `gpt-3.5-turbo-instruct`. But, to get the best out of GPT-4 for coding tasks, it's still important to give clear and specific instructions. As a result, designing good prompts can take more care.

### More prompt advice

For more prompt examples, visit [OpenAI Examples][OpenAI Examples].

In general, the input prompt is the best lever for improving model outputs. You can try tricks like:

- **Be more specific** E.g., if you want the output to be a comma separated list, ask it to return a comma separated list. If you want it to say "I don't know" when it doesn't know the answer, tell it 'Say "I don't know" if you do not know the answer.' The more specific your instructions, the better the model can respond.
- **Provide Context**: Help the model understand the bigger picture of your request. This could be background information, examples/demonstrations of what you want or explaining the purpose of your task.
- **Ask the model to answer as if it was an expert.** Explicitly asking the model to produce high quality output or output as if it was written by an expert can induce the model to give higher quality answers that it thinks an expert would write. Phrases like "Explain in detail" or "Describe step-by-step" can be effective.
- **Prompt the model to write down the series of steps explaining its reasoning.** If understanding the 'why' behind an answer is important, prompt the model to include its reasoning. This can be done by simply adding a line like "[Let's think step by step](https://arxiv.org/abs/2205.11916)" before each answer.

[Fine Tuning Docs]: https://platform.openai.com/docs/guides/fine-tuning
[OpenAI Customer Stories]: https://openai.com/customer-stories
[Large language models Blog Post]: https://openai.com/research/better-language-models
[GitHub Copilot]: https://github.com/features/copilot/
[GPT-4 and GPT-4 Turbo]: https://platform.openai.com/docs/models/gpt-4-and-gpt-4-turbo
[GPT3 Apps Blog Post]: https://openai.com/blog/gpt-3-apps/
[OpenAI Examples]: https://platform.openai.com/examples