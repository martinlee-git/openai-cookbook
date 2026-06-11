# gpt_action_retool_workflow

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/examples/chatgpt/gpt_actions_library/gpt_action_retool_workflow.md
- 미러 경로: `examples/chatgpt/gpt_actions_library/gpt_action_retool_workflow.md`

## 한글 요약

GPT 작업 라이브러리: 재도구 작업 흐름 소개 이 페이지에서는 특정 애플리케이션에 대한 GPT 작업을 구축하는 개발자를 위한 지침 및 가이드를 제공합니다. 계속하기 전에 먼저 다음 정보를 숙지하십시오. GPT 작업 소개 GPT 작업 라이브러리 소개 처음부터 GPT 작업 구축의 예 이 특정 GPT 작업은 재구성 작업 흐름에 연결하는 방법에 대한 개요를 제공합니다. 이 작업은 사용자 입력을 받아 웹훅 트리거를 사용하여 Retool의 워크플로로 보냅니다. 그런 다음 Retool은 구성된 워크플로를 수행하고 응답을 JSON 개체로 ChatGPT에 다시 보냅니다. 가치 + 비즈니스 사용 사례의 예 가치: 이제 사용자는 ChatGPT의 자연어 기능을 활용하여 Retool의 모든 워크플로에 직접 연결할 수 있습니다. 사용 사례 예시: GPT에 통합하려는 Retool 워크플로에서 실행되는 맞춤 코드가 있습니다. 데이터 과학자는 Retool Vector 또는 다른 벡터를 사용하여 외부 VectorDB를 유지합니다.

## 핵심 발췌

DB) 벡터 검색 결과를 다시 ChatGPT로 보내고 싶습니다. Retool은 내부 서비스에 연결하기 위한 미들웨어로 사용되며 Retool의 웹후크를 사용하여 ChatGPT에 이러한 서비스에 대한 액세스를 제공하려고 합니다. 애플리케이션 정보 애플리케이션 키 링크 시작하기 전에 애플리케이션에서 다음 링크를 확인하십시오. 애플리케이션 웹 사이트: https://retool.com/products/workflows 애플리케이션 API 설명서: https://docs.retool.com/workflows 애플리케이션 전제 조건 시작하기 전에 Retool 환경에서 다음 단계를 수행해야 합니다. Retool 계정 설정 간단한 워크플로 생성 애플리케이션 워크플로 단계 다음은 기본 Retool 워크플로의 예입니다. 이 워크플로는 2개의 값을 가져와 추가하고 th로 웹훅 트리거에 응답합니다.

## 원문 내용

# GPT Action Library: Retool Workflow

## Introduction

This page provides an instruction & guide for developers building a GPT Action for a specific application. Before you proceed, make sure to first familiarize yourself with the following information: 
- [Introduction to GPT Actions](https://platform.openai.com/docs/actions)
- [Introduction to GPT Actions Library](https://platform.openai.com/docs/actions/actions-library)
- [Example of Building a GPT Action from Scratch](https://platform.openai.com/docs/actions/getting-started)

This particular GPT Action provides an overview of how to connect to a **Retool Workflow**. This Action takes a users input and sends it to the workflow in Retool using a webhook trigger. Retool then performns the configured workflow and sends a response back to ChatGPT as a JSON object.

### Value + Example Business Use Cases

**Value**: Users can now leverage ChatGPT's natural language capability to connect directly to any workflow in Retool.

**Example Use Cases**: 
- You have custom code running in a Retool workflow that you'd like to incorporate into a GPT. 
- Data Scientists maintain an external VectorDB (either using Retool Vector or another vector DB) and would like to send the results of the vector search back to ChatGPT. 
- Retool is used as middleware to connect to internal services, and you'd like to use Retool's webhooks to provide access to these services to ChatGPT. 

## Application Information

### Application Key Links

Check out these links from the application before you get started:
- Application Website: https://retool.com/products/workflows
- Application API Documentation: https://docs.retool.com/workflows

### Application Prerequisites

Before you get started, make sure you go through the following steps in your Retool environment:
- Set up a Retool account
- Create a simple workflow

### Application Workflow Steps

Below is an example of a basic Retool Workflow. This workflow takes in 2 values and adds them and responds to the webhook trigger with the result.

***Note:*** Your workflow must be deployed before it will be accessible from your GPT.

<!--ARCADE EMBED START--><div style="position: relative; padding-bottom: calc(57.26681127982647% + 41px); height: 0; width: 100%;"><iframe src="https://demo.arcade.software/MG7PcF8fh3RH722eonUb?embed&embed_mobile=tab&embed_desktop=inline&show_copy_link=true" title="Retool Workflow Cookbook" frameborder="0" loading="lazy" webkitallowfullscreen mozallowfullscreen allowfullscreen allow="clipboard-write" style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; color-scheme: light;" ></iframe></div><!--ARCADE EMBED END-->

## ChatGPT Steps

### Custom GPT Instructions 

Once you've created a Custom GPT, you should add Instructions to the GPT providing context about the GPTs role, and the actions it is able to perform. Have questions? Check out [Getting Started Example](https://platform.openai.com/docs/actions/getting-started) to see how this step works in more detail.

### OpenAPI Schema 

Once you've created a Custom GPT, copy the text below in the Actions panel. Have questions? Check out [Getting Started Example](https://platform.openai.com/docs/actions/getting-started) to see how this step works in more detail.

***Note:*** You need to replace the __<WORKFLOW_ID>__ value in the OpenAPI spec below with the ID for your workflow.


```yaml
openapi: 3.1.0
info:
  title: Retool Workflow API
  description: API for interacting with Retool workflows.
  version: 1.0.0
servers:
  - url: https://api.retool.com/v1
    description: Main (production) server
paths:
  /workflows/<WORKFLOW_ID>/startTrigger:
    post:
      operationId: add_numbers
      summary: Takes 2 numbers and adds them.
      description: Initiates a workflow in Retool by triggering a specific workflow ID.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                first:
                  type: integer
                  description: First parameter for the workflow.
                second:
                  type: integer
                  description: Second parameter for the workflow.
      responses:
        "200":
          description: Workflow triggered successfully.
        "400":
          description: Bad Request - Invalid parameters or missing data.
        "401":
          description: Unauthorized - Invalid or missing API key.
      security:
        - apiKeyAuth: []
```

## Authentication Instructions

Below are instructions on setting up authentication with this 3rd party application. Have questions? Check out [Getting Started Example](https://platform.openai.com/docs/actions/getting-started) to see how this step works in more detail.

### Pre-Action Steps

Before you set up authentication in ChatGPT, please take the following steps in the application.
- Get your API Key from the Webhook config panel

![retool_api_key.png](../../../images/retool_api_key.png)

### In ChatGPT

In ChatGPT, click on "Authentication" and choose **"API Key"**. Enter in the information below. 

- **API Key**: (Paste your API Key provided by the Retool Workflow Webhook Trigger)
- **Auth Type**: Custom
- **Custom Header Name**: X-Workflow-Api-Key

### FAQ & Troubleshooting

- *Auth Error:* Ensure you have set the custom header name correctly.
- *Invalid Workflow Error:* Ensure you have deployed your workflow within Retool.

*Are there integrations that you’d like us to prioritize? Are there errors in our integrations? File a PR or issue in our github, and we’ll take a look.*