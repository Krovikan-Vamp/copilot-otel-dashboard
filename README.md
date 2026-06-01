# copilot-otel-dashboard

A self-hosted OpenTelemetry monitoring stack for **GitHub Copilot Chat in VS Code**.

Receives traces, metrics, and events from the Copilot extension and surfaces them in Grafana with pre-built dashboards. Metric names follow the [official VS Code OTEL docs](https://code.visualstudio.com/docs/copilot/guides/monitoring-agents) (updated 2026-05-28).

**Dashboard rows:**

- **Activity Overview** — LLM operations, p95 latency, tool calls, sessions started (24h stat cards)
- **LLM Request Trends** — operation rate by model, response duration p50/p95/p99
- **Token Usage** — input/output token rate, time to first token p50/p95
- **Agent & Tool Execution** — tool invocations by `gen_ai.tool.name`, agent invocation duration
- **Trace Explorer** — TraceQL trace list filtered to `service.name="copilot-chat"` with agent name, model, and token counts

---

## Stack

| Service | Purpose | Port |
|---|---|---|
| `otel-collector` | Receives OTLP from VS Code, fans out to Prometheus + Tempo | 4317 (gRPC), 4318 (HTTP) |
| `prometheus` | Stores metrics scraped from the collector | 9090 |
| `tempo` | Stores distributed traces | 3200 |
| `grafana` | Dashboards + trace explorer | 3000 |

---

## Prerequisites

- Docker Desktop (Windows) or Docker Engine + Compose plugin (Linux)
- VS Code 1.119+
- GitHub Copilot Chat extension

---

## Quick Start

### 1. Clone and start the stack

```bash
git clone https://github.com/Krovikan-Vamp/copilot-otel-dashboard.git
cd copilot-otel-dashboard

cp .env.example .env          # edit passwords if desired
docker compose up -d
docker compose ps             # verify all 4 services are healthy
```

### 2. Configure VS Code

Open user settings JSON (`Ctrl+Shift+P` → *Open User Settings (JSON)*) and add the contents of [`vscode/settings.json`](./vscode/settings.json):

```json
{
  "github.copilot.chat.otel.enabled": true,
  "github.copilot.chat.otel.exporterType": "otlp-http",
  "github.copilot.chat.otel.otlpEndpoint": "http://localhost:4318",
  "github.copilot.chat.otel.captureContent": false
}
```

Restart VS Code after adding these settings so the OTEL exporter initialises.

> **Tip:** You can also enable OTel without touching settings by setting the environment variable `COPILOT_OTEL_ENABLED=true` or `OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318` before launching VS Code.

### 3. Open Grafana

Navigate to [http://localhost:3000](http://localhost:3000) and log in:

- **Username:** `admin`
- **Password:** `copilot` (or whatever you set in `.env`)

The **GitHub Copilot — OTEL Dashboard** auto-provisions under *Dashboards → GitHub Copilot*.

### 4. Verify data flow

Fire a Copilot Chat prompt in VS Code, then tail the collector logs:

```bash
docker compose logs -f otel-collector
```

You should see `ScopeSpans` and `ScopeMetrics` entries within seconds.

---

## Metric Reference

All metric names are taken directly from the [VS Code OTEL monitoring docs](https://code.visualstudio.com/docs/copilot/guides/monitoring-agents#_metrics). Prometheus normalises dots to underscores when scraping.

### GenAI Semantic Convention metrics

| OTel metric | Prometheus name | Type | Description |
|---|---|---|---|
| `gen_ai.client.operation.duration` | `gen_ai_client_operation_duration_seconds` | Histogram | LLM API call duration |
| `gen_ai.client.token.usage` | `gen_ai_client_token_usage` | Histogram | Token counts (input/output) |

### Extension-specific metrics

| OTel metric | Prometheus name | Type | Description |
|---|---|---|---|
| `copilot_chat.tool.call.count` | `copilot_chat_tool_call_count_total` | Counter | Tool invocations |
| `copilot_chat.tool.call.duration` | `copilot_chat_tool_call_duration` | Histogram | Tool execution latency (ms) |
| `copilot_chat.agent.invocation.duration` | `copilot_chat_agent_invocation_duration_seconds` | Histogram | Agent end-to-end duration |
| `copilot_chat.agent.turn.count` | `copilot_chat_agent_turn_count` | Histogram | LLM round-trips per invocation |
| `copilot_chat.session.count` | `copilot_chat_session_count_total` | Counter | Chat sessions started |
| `copilot_chat.time_to_first_token` | `copilot_chat_time_to_first_token_seconds` | Histogram | Time to first SSE token |

---

## Attribute / Filter Reference

Useful label names for PromQL `by()` clauses and Tempo TraceQL filters:

| Attribute | Where | Example values |
|---|---|---|
| `gen_ai.request.model` | traces + metrics | `gpt-4o`, `claude-opus-4-5` |
| `gen_ai.agent.name` | `invoke_agent` span | `GitHub Copilot Chat`, `copilotcli`, `claude` |
| `gen_ai.tool.name` | `execute_tool` span + `copilot_chat.tool.call.count` | `readFile`, `runCommand` |
| `gen_ai.token_type` | `gen_ai.client.token.usage` | `input`, `output` |
| `service.name` | all signals | `copilot-chat`, `github-copilot` |
| `error.type` | failure spans | error class string |

---

## Security Notes

- `captureContent: false` by default — prompts and completions are **not** stored.
- Change `GF_SECURITY_ADMIN_PASSWORD` before exposing Grafana beyond localhost.
- OTEL collector CORS allows `*` by default — restrict if deploying remotely.
- OTel is off by default; the SDK is not loaded until you explicitly enable it.

---

## Stopping the Stack

```bash
docker compose down        # stop, keep volumes
docker compose down -v     # stop + delete all stored data
```
