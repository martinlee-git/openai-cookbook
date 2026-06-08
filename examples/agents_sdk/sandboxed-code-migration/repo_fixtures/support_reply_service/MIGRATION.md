# 이주

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/examples/agents_sdk/sandboxed-code-migration/repo_fixtures/support_reply_service/MIGRATION.md
- 미러 경로: `examples/agents_sdk/sandboxed-code-migration/repo_fixtures/support_reply_service/MIGRATION.md`

## 한글 요약

마이그레이션 요청: 채팅 완료를 응답으로 이 패키지를 레거시 채팅 완료 호출 형태에서 응답 API 호출 형태로 마이그레이션합니다. 현재 구조 고객 지원 bot/client.py에는 OpenAI 클라이언트 래퍼가 포함되어 있습니다. 고객 지원 bot/replies.py는 지원 응답 프롬프트를 작성하고 래퍼를 호출합니다. 테스트/에는 레거시 채팅 완료 형태에 대한 오프라인 가짜가 포함되어 있습니다. 대상 형태 고객 지원 bot/client.py에서 client.chat.completions.create(...) 대신 client.responses.create(...)를 호출하세요. 동일한 모델 인수를 유지하십시오. 래퍼의 메시지 인수를 입력 항목 인수로 바꿉니다. 고객 지원 bot/replies.py에서 두 개의 메시지 시스템/사용자 대화를 입력 항목으로 전달합니다. 입력 항목을 Responses API 입력 인수로 전달합니다. 온도를 0으로 유지하세요. Completion.choices[0].message.content 대신 response.output 텍스트를 반환합니다. 초안 답변(클라이언트, 모델, 케이스 ID, 고객 메시지) 기능 서명을 유지합니다. 클라이언트 래퍼를 업데이트하고 테스트를 응답합니다.

## 핵심 발췌

채팅 완료 대신 응답 API를 가짜로 만듭니다. 테스트는 오프라인 상태로 유지되어야 합니다. 실제 OpenAI 클라이언트를 가져오거나 인스턴스화하지 마십시오. 필수 검증 파이프라인 편집하기 전에 기본 테스트를 실행하십시오: python m unittest discover s 테스트 t .. 편집 후 컴파일/확인 명령을 실행하십시오: python m compileall q 고객 지원 봇 테스트. 컴파일/검사 명령이 통과된 후 최종 테스트를 실행합니다: python m unittest discover s 테스트 t .. python m unittest discover s 테스트 t ..로 유효성 검증

## 원문 내용

# Migration request: Chat Completions to Responses

Migrate this package from the legacy Chat Completions call shape to the
Responses API call shape.

## Current structure

- `customer_support_bot/client.py` contains the OpenAI client wrapper.
- `customer_support_bot/replies.py` builds the support reply prompt and calls the wrapper.
- `tests/` contains offline fakes for the legacy Chat Completions shape.

## Target shape

- In `customer_support_bot/client.py`, call `client.responses.create(...)`
  instead of `client.chat.completions.create(...)`.
- Keep the same `model` argument.
- Replace the wrapper's `messages` argument with an `input_items` argument.
- In `customer_support_bot/replies.py`, pass the two-message system/user
  conversation as `input_items`.
- Forward `input_items` as the Responses API `input` argument.
- Keep `temperature=0`.
- Return `response.output_text` instead of `completion.choices[0].message.content`.
- Preserve the `draft_reply(client, *, model, case_id, customer_message)` function signature.
- Update client-wrapper and reply tests to fake the Responses API instead of
  Chat Completions.
- Tests must remain offline; do not import or instantiate the real OpenAI client.

## Required validation pipeline

- Before editing, run baseline tests: `python -m unittest discover -s tests -t .`.
- After editing, run the compile/check command: `python -m compileall -q customer_support_bot tests`.
- After the compile/check command passes, run final tests: `python -m unittest discover -s tests -t .`.
- Validate with `python -m unittest discover -s tests -t .`.