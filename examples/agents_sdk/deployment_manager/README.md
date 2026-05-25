# 읽어보기

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/examples/agents_sdk/deployment_manager/README.md
- 미러 경로: `examples/agents_sdk/deployment_manager/README.md`

## 한글 요약

Agents SDK 배포 관리자 Agents SDK 데모 프로젝트를 실행하고 관찰하기 위한 로컬 제어 평면 앱입니다. 전제 조건 uv npm Docker, 기본 로컬 docker 대상 Run Open 사용 시: Vite는 프론트엔드/에서 dist/로 React UI를 빌드하고 Flask 백엔드는 dist/와 /api/를 제공합니다. 스크린샷 배포 앱 세부정보 추적 범위 로컬 에이전트 SDK 프로젝트를 가져옵니다. 진입점, 종속성, 환경 변수 및 샌드박스 사용량을 검사합니다. 로컬 배포 기록을 만듭니다. 기본적으로 앱/오케스트레이터를 레이블이 지정된 Docker 컨테이너로 시작하고 중지합니다. 컨테이너화된 배포를 위해 앱 수준 Dockerfile을 생성하거나 재사용합니다. 배포 로그, 추적, 앱 이벤트 및 Docker 컨테이너 활동을 표시합니다. 에이전트 SDK를 사용하여 샌드박스 컨테이너에 라벨을 지정합니다. 관리자가 Docker 컨테이너를 배포에 다시 매핑하고 실행할 수 있도록 메타데이터를 제공합니다. Docker 배포의 경우 기본 대상은 앱 Dockerfile을 빌드하고 다음을 사용하여 오케스트레이터를 실행합니다. Docker 소켓 마운트를 사용하면 SandboxAgent가 Docker를 통해 샌드박스 컨테이너를 생성할 수 있습니다.

## 핵심 발췌

. 관리자는 앱 프로세스에 AGENTS SDK MANAGER, AGENTS SDK PROJECT, AGENTS SDK DEPLOYMENT DATA DIR, AGENTS SDK MANAGER TRACE ENDPOINT를 설정합니다. 추적 캡처는 관리자에게 다시 게시되고 관리자가 소유한 SQLite 데이터베이스에 저장됩니다. 이 관리자를 통해 배포된 배포 가능한 앱 계약 앱에는 종속성에 openai 에이전트가 있는 pyproject.toml이 있어야 합니다. main.py를 앱 진입점으로 사용합니다. 로컬 시작을 위한 PORT 지원. uv는 PORT가 설정되면 HTTP 서비스를 시작하는 python main.py를 실행합니다. /health 준비 엔드포인트. 생성된 Dockerfile이 서비스를 안정적으로 시작할 수 있도록 명시적 인수 뒤에 CLI 전용 연기 동작을 유지하거나 PORT 없음 경로를 유지하세요. 추적은 에이전트 SDK가 여전히 OpenAI로 추적을 스트리밍하는 동안 관리자의 HTTP 수집 엔드포인트를 통해 로컬로 캡처됩니다.

## 원문 내용

# Agents SDK Deployment Manager

Local control-plane app for running and observing Agents SDK demo projects.

## Prerequisites

- `uv`
- `npm`
- Docker, when using the default `local-docker` target

## Run

```bash
make run
```

Open:

```text
http://127.0.0.1:8732
```

Vite builds the React UI from `frontend/` into `dist/`, and the Flask backend
serves `dist/` plus `/api/*`.

## Screenshots

### Deployments

![Deployments](docs/screenshots/deployments.png)

### App details

![App details](docs/screenshots/app-details.png)

### Traces

![Traces](docs/screenshots/traces.png)

## Scope

- Import a local Agents SDK project.
- Inspect entrypoints, dependencies, env vars, and sandbox usage.
- Create a local deployment record.
- Start and stop the app/orchestrator as a labeled Docker container by default.
- Generate or reuse an app-level Dockerfile for containerized deployment.
- Show deployment logs, traces, app events, and Docker container activity.
- Label sandbox containers with `agents-sdk.*` metadata so the manager can map a
  Docker container back to its deployment and run.

For Docker deployments, the default target builds an app Dockerfile and runs the
orchestrator with:

```bash
docker run -p 127.0.0.1:<app-port>:<app-port> -v /var/run/docker.sock:/var/run/docker.sock ...
```

The Docker socket mount lets `SandboxAgent` create sandbox containers through Docker.

The manager sets `AGENTS_SDK_MANAGER`, `AGENTS_SDK_PROJECT`,
`AGENTS_SDK_DEPLOYMENT_DATA_DIR`, and `AGENTS_SDK_MANAGER_TRACE_ENDPOINT` on the
app process. Trace capture is posted back to the manager and stored in the
manager-owned SQLite database.

## Deployable App Contract

Apps deployed through this manager should have:

- `pyproject.toml` with `openai-agents` in the dependencies.
- `main.py` as the app entrypoint.
- `PORT` support for local startup.
- `uv run python main.py` starting the HTTP service when `PORT` is set.
- A `/health` readiness endpoint.

Keep CLI-only smoke behavior behind explicit arguments or the no-`PORT` path so
the generated Dockerfile can start the service reliably.

## Tracing

Tracing is captured locally through the manager's HTTP ingestion endpoint while
the Agents SDK still streams traces to the OpenAI Platform through its default
processor.

When the manager starts an app it adds `runtime/trace_capture` to `PYTHONPATH`
and sets:

```text
AGENTS_SDK_MANAGER_TRACE_ENDPOINT=http://<manager-host>:8732/api/traces/ingest
AGENTS_SDK_DEPLOYMENT_ID=<deployment-id>
AGENTS_SDK_PROJECT_ID=<project-id>
```

For local-process deployments `<manager-host>` is `127.0.0.1`. For Docker
deployments it is `host.docker.internal` so the container can reach the manager
running on the host.

The runtime package uses `sitecustomize.py` to install a lightweight Agents SDK
trace processor inside the app process with `add_trace_processor()`. That adds a
second local destination without replacing the SDK's default OpenAI Platform
exporter. The local processor posts exported trace records to the manager:

- `trace_start`
- `trace_end`
- `span_start`
- `span_end`

The manager stores traces in `state/traces.sqlite3`. The backend reads that
SQLite store in `app/timeline.py`, groups records by `trace_id`, derives the run
key from trace metadata (`expense_id`, `run_id`, or `session_id`), then falls
back to `group_id` and finally the `trace_id`. The UI uses those reconstructed
records for the Traces view, nested span list, event detail pane, trace deep
links via `?trace_id=...`, and JSON export.

This means local tracing keeps working without bind-mounting trace files into app
containers and without fetching spans from the OpenAI Platform dashboard API.

## Make Targets

```bash
make start
make health
make deploy PROJECT_PATH=/path/to/agents-sdk-app APP_PORT=8421 SANDBOX_BACKEND=docker
make deploy PROJECT_PATH=/path/to/agents-sdk-app APP_PORT=8421 TARGET=local-process
make stop
```