# 읽어보기

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/examples/agents_sdk/sandboxed-code-migration/repo_fixtures/support_reply_service/README.md
- 미러 경로: `examples/agents_sdk/sandboxed-code-migration/repo_fixtures/support_reply_service/README.md`

## 한글 요약

고객 지원 응답 봇 이 작은 패키지는 OpenAI Python 클라이언트를 사용하여 지원 에이전트 응답 초안을 작성합니다. 현재 구현에서는 여전히 고객 지원 bot/client.py의 작은 래퍼를 통해 채팅 완료를 사용합니다. 마이그레이션 대상은 MIGRATION.md에 있습니다.

## 핵심 발췌

고객 지원 응답 봇 이 작은 패키지는 OpenAI Python 클라이언트를 사용하여 지원 에이전트 응답 초안을 작성합니다. 현재 구현에서는 여전히 `고객 지원 봇/클라이언트'의 작은 래퍼를 통해 채팅 완료를 사용합니다.

## 원문 내용

# Customer Support Reply Bot

This tiny package drafts a support-agent reply with the OpenAI Python client.

The current implementation still uses Chat Completions through a small wrapper
in `customer_support_bot/client.py`. The migration target is in `MIGRATION.md`.