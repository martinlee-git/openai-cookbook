# 도구_재시도_프롬프트

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/examples/agents_sdk/multi-agent-portfolio-collaboration/prompts/tool_retry_prompt.md
- 미러 경로: `examples/agents_sdk/multi-agent-portfolio-collaboration/prompts/tool_retry_prompt.md`

## 한글 요약

도구 호출 재시도 지침 인증이나 서버 오류(예: 500 내부 서버 오류 또는 4XX 오류), 시간 초과 또는 네트워크 문제로 인해 도구 호출이 실패한 경우 포기하기 전에 동일한 도구 호출을 최대 2회 더 다시 시도해야 합니다. 총 3번의 시도 후에도 도구 호출이 계속 실패하면 출력에 오류를 보고하고 나머지 분석을 최대한 진행하십시오. 기존 리소스가 없는 상황(FRED 시리즈 없음, 유효하지 않은 티커)에서는 동일한 입력을 사용하지 마십시오. 예: 코드 해석기 도구가 "오류: 500 서버 오류: 내부 서버 오류 ..."를 반환하는 경우 동일한 도구 호출을 최대 2번 더 다시 시도하십시오. 도구 호출이 3번 모두 실패하는 경우 출력에 "3번 시도 후 도구 호출 실패: [오류 메시지]"라는 메모를 포함합니다. 워크플로의 모든 도구 호출에 이 재시도 논리를 적용합니다.

## 핵심 발췌

도구 호출 재시도 지침 인증 또는 서버 오류(예: 500 내부 서버 오류 또는 4XX 오류), 시간 초과 또는 네트워크 문제로 인해 도구 호출이 실패한 경우 최대 2개월 동안 동일한 도구 호출을 다시 시도해야 합니다.

## 원문 내용

# Tool Call Retry Instructions

If a tool call fails due to an authentication or server error (such as a 500 Internal Server Error, or 4XX errors), timeout, or network issue, you MUST retry the same tool call up to 2 more times before giving up. If the tool call still fails after 3 total attempts, report the error in your output and proceed with the rest of your analysis as best as possible. In situations where there isn't an existing resource (No FRED Series, Invalid Ticker) don't use the same inputs.

---

**Example:**
- If the code interpreter tool returns: "Error: 500 Server Error: Internal Server Error ...", retry the same tool call up to 2 more times.
- If the tool call fails all 3 times, include a note in your output: "Tool call failed after 3 attempts: [error message]".

---

Apply this retry logic to all tool calls in your workflow.