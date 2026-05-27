# gpt_action_github

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/examples/chatgpt/gpt_actions_library/gpt_action_github.md
- 미러 경로: `examples/chatgpt/gpt_actions_library/gpt_action_github.md`

## 한글 요약

GPT 작업 라이브러리: GitHub 소개 이 페이지에서는 GPT 작업을 GitHub에 연결하는 개발자를 위한 지침을 제공합니다. 계속하기 전에 다음 리소스를 숙지하세요. GPT 작업 소개 GPT 작업 라이브러리 처음부터 GPT 작업 구축 이 GPT 작업은 개발자가 GitHub 끌어오기 요청 차이의 품질과 보안을 평가하는 데 도움이 됩니다. 이는 각 도메인에 대한 피드백과 제안을 제공하므로 개발자는 피드백을 풀 요청에 대한 코멘트로 자동 제출하기 전에 피드백을 수정하거나 수락할 수 있습니다. 가치 및 예시 비즈니스 사용 사례 가치: 사용자는 ChatGPT의 자연어 기능을 활용하여 GitHub Pull Request 검토를 지원할 수 있습니다. 개발자용: 제안된 수정 사항에 대한 즉각적인 피드백을 통해 코드 변경 사항을 분석하고 고품질 검토를 수행합니다. 조직의 경우: 차이점이 모범 사례 및 코딩 표준을 준수하는지 확인하거나 리팩토링된 대안을 자동으로 제안합니다(최선을 정의하려면 추가 API 요청이 필요할 수 있음).

## 핵심 발췌

관행). 전체: 이 AI 기반 코드 검토 도우미를 사용하여 생산성을 높이고 더 높은 품질과 보안 코드를 보장하세요. 사용 사례 예시: 검토자는 제안된 코드 변경의 품질과 보안에 대한 피드백을 구합니다. 조직에서는 코드 검토 중에 모범 사례 및 표준을 자동으로 준수하도록 권장합니다. 데모 비디오: 애플리케이션 정보 주요 링크 시작하기 전에 다음 리소스를 탐색하십시오. GitHub GitHub API 문서 전제 조건 공개 풀 요청이 있는 저장소가 있는지 확인하십시오. 애플리케이션 설정 풀 요청 선택 1. 저장소(예: PR 예시)로 이동합니다. 소유자(예: "microsoft"), 저장소 이름(예: "vscode") 및 PR 번호(예: "229241")를 기록해 둡니다. 저장소 소유자가 SSO 조직인 경우 토큰에 승인이 필요할 수 있습니다. 2

## 원문 내용

# GPT Action Library: GitHub

## Introduction

This page provides instructions for developers connecting a GPT Action to GitHub. Before proceeding, familiarize yourself with the following resources:
- [Introduction to GPT Actions](https://platform.openai.com/docs/actions)
- [GPT Actions Library](https://platform.openai.com/docs/actions/actions-library)
- [Building a GPT Action from Scratch](https://platform.openai.com/docs/actions/getting-started)

This GPT Action helps developers evaluate the quality and security of a GitHub Pull Request diff. It provides feedback and suggestions for each domain, allowing developers to modify or accept the feedback before automatically submitting it as a comment on the Pull Request.

## Value & Example Business Use Cases

### **Value**:
Users can leverage ChatGPT's natural language capabilities to assist with GitHub Pull Request reviews.

- **For developers**: Analyze code changes and perform high-quality reviews with instant feedback on proposed modifications.
- **For organizations**: Ensure diffs adhere to best practices and coding standards, or automatically propose refactored alternatives (additional API requests may be required to define best practices).
- **Overall**: Boost productivity and ensure higher-quality, more secure code with this AI-powered Code Review assistant.

### **Example Use Cases**:
- A reviewer seeks feedback on the quality and security of a proposed code change.
- An organization encourages adherence to best practices and standards automatically during code review.

## Demonstration Video:
[![Watch the video](https://img.youtube.com/vi/bcjybCh-x-Q/0.jpg)](https://www.youtube.com/watch?v=bcjybCh-x-Q)

## Application Information

### **Key Links**
Before starting, explore these resources:
- [GitHub](https://github.com)
- [GitHub API Documentation](https://docs.github.com/en/rest/pulls?apiVersion=2022-11-28)

### **Prerequisites**
Ensure you have a repository with an open pull request.

## Application Setup

### **Select a Pull Request**
1. Navigate to a repository, e.g., [example PR](https://github.com/microsoft/vscode/pull/229241).
   - Note the owner (e.g., "microsoft"), repository name (e.g., "vscode"), and PR number (e.g., "229241").
   - If the repository owner is an SSO organization, your token may need [approval](https://docs.github.com/en/organizations/managing-programmatic-access-to-your-organization/managing-requests-for-personal-access-tokens-in-your-organization#managing-fine-grained-personal-access-token-requests).
2. Review [how to perform a high-quality code review](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/getting-started/best-practices-for-pull-requests).

### **Generate a "Fine Grained" GitHub Personal Access Token**
1. Log in to GitHub and go to **Settings**.
2. Navigate to **Developer settings** > **Fine Grained Personal access tokens**.
3. Click **Generate new token**, name it, set an expiration date, and select the necessary scopes (e.g., `read:content`, `read&write:pull_requests`).
4. Copy and securely store the token.

## ChatGPT Steps

### **Custom GPT Instructions**

Once you've created a Custom GPT, copy the following into the Instructions panel:

```
# **Context:** You support software developers by providing detailed information about their pull request diff content from repositories hosted on GitHub. You help them understand the quality, security and completeness implications of the pull request by providing concise feedback about the code changes based on known best practices. The developer may elect to post the feedback (possibly with their modifications) back to the Pull Request. Assume the developer is familiar with software development.

# **Instructions:**

## Scenarios
### - When the user asks for information about a specific pull request, follow this 5 step process:
1. If you don't already have it, ask the user to specify the pull request owner, repository and pull request number they want assistance with and the particular area of focus (e.g., code performance, security vulnerabilities, and best practices).
2. Retrieve the Pull Request information from GitHub using the getPullRequestDiff API call, owner, repository and the pull request number provided. 
3. Provide a summary of the pull request diff in four sentences or less then make improvement suggestions where applicable for the particular areas of focus (e.g., code performance, security vulnerabilities, and best practices).
4. Ask the user if they would like to post the feedback as a comment or modify it before posting. If the user modifies the feedback, incorporate that feedback and repeat this step. 
5. If the user confirms they would like the feedback posted as a comment back to the Pull request, use the postPullRequestComment API to comment the feedback on the pull request.
```

### OpenAPI Schema

Once you've created a Custom GPT, copy the text below in the Actions panel. Have questions? Check out [Getting Started Example](https://platform.openai.com/docs/actions/getting-started) to see how this step works in more detail.

Below is an example of what connecting to GitHub to GET the Pull Request Diff and POST the Feedback to the Pull Request might look like.

```javascript
openapi: 3.1.0
info:
  title: GitHub Pull Request API
  description: Retrieve the diff of a pull request and post comments back to it.
  version: 1.0.0
servers:
  - url: https://api.github.com
    description: GitHub API
paths:
  /repos/{owner}/{repo}/pulls/{pull_number}:
    get:
      operationId: getPullRequestDiff
      summary: Get the diff of a pull request.
      parameters:
        - name: owner
          in: path
          required: true
          schema:
            type: string
          description: Owner of the repository.
        - name: repo
          in: path
          required: true
          schema:
            type: string
          description: Name of the repository.
        - name: pull_number
          in: path
          required: true
          schema:
            type: integer
          description: The number of the pull request.
        - name: Accept
          in: header
          required: true
          schema:
            type: string
            enum:
              - application/vnd.github.v3.diff
          description: Media type for the diff format.
      responses:
        "200":
          description: Successfully retrieved the pull request diff.
          content:
            text/plain:
              schema:
                type: string
        "404":
          description: Pull request not found.
  /repos/{owner}/{repo}/issues/{issue_number}/comments:
    post:
      operationId: postPullRequestComment
      summary: Post a comment to the pull request.
      parameters:
        - name: owner
          in: path
          required: true
          schema:
            type: string
          description: Owner of the repository.
        - name: repo
          in: path
          required: true
          schema:
            type: string
          description: Name of the repository.
        - name: issue_number
          in: path
          required: true
          schema:
            type: integer
          description: The issue or pull request number.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                body:
                  type: string
                  description: The content of the comment.
      responses:
        "201":
          description: Successfully created a comment.
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: integer
                  body:
                    type: string
                  user:
                    type: object
                    properties:
                      login:
                        type: string
                      id:
                        type: integer
        "404":
          description: Pull request not found.
```

## Authentication Instructions

Below are instructions on setting up authentication with this 3rd party application. Have questions? Check out [Getting Started Example](https://platform.openai.com/docs/actions/getting-started) to see how this step works in more detail.

### In ChatGPT (refer to Step 2 in the Getting Started Example)

In ChatGPT, click on "Authentication" and choose **"Bearer"**. Enter in the information below. Ensure your token has the permissions described in Application setup, above.

- Authentication Type: API Key
- Auth Type: Bearer
- API Key 
  <personal_access_token>

### Test the GPT

You are now ready to test out the GPT. You can enter a simple prompt like "Can you review my pull request? owner: <org_name>, repo: <repo_name>, pull request number: <PR_Number>" and expect to see the following:

![landing_page.png](../../../../images/landing_page.png)

1. A summary of changes in the referenced pull request(PR).

![First Interaction](../../../images/first_interaction.png)

2. Quality and Security feedback and suggestions to incorporate in the next iteration of the PR.

![First Feedback](../../../images/first_feedback.png)

3. An option to iterate on the feedback or accept it and have the GPT post it directly to the PR as a comment from you. 

![First Interaction](../../../images/final_result.png)

*Are there integrations that you’d like us to prioritize? Are there errors in our integrations? File a PR or issue in our github, and we’ll take a look.*