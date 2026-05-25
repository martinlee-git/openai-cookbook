# 읽어보기

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/examples/agents_sdk/sandboxed-code-migration/repo_fixtures/case_summary_service/README.md
- 미러 경로: `examples/agents_sdk/sandboxed-code-migration/repo_fixtures/case_summary_service/README.md`

## 한글 요약

사례 요약 서비스 샌드박스 마이그레이션 요리책을 위한 소규모 오프라인 장치입니다. 사전 마이그레이션 서비스는 채팅 완료 통화를 래핑하고 이를 사용하여 내부 사례 메모를 요약합니다. 테스트에서는 가짜를 사용합니다. 절대로 네트워크에 전화하면 안 됩니다.

## 핵심 발췌

사례 요약 서비스 샌드박스 마이그레이션 요리책을 위한 소규모 오프라인 장치입니다. 사전 마이그레이션 서비스는 채팅 완료 통화를 래핑하고 이를 사용하여 내부 사례 메모를 요약합니다. 테스트에서는 가짜를 사용합니다. 그들은 절대 안 돼

## 원문 내용

# Case summary service

Small offline fixture for the sandboxed migration cookbook.

The pre-migration service wraps a Chat Completions call and uses it to summarize
internal case notes. Tests use fakes; they should never call the network.