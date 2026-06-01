# copilot-otel-dashboard

A self-hosted OpenTelemetry monitoring stack for **GitHub Copilot Chat in VS Code**.

Receives traces, metrics, and events from the Copilot extension and surfaces them in Grafana with pre-built dashboards for:

- **Activity overview** — request count, error rate, tool calls, p95 latency
- **Request trends** — LLM request rate by model, latency percentiles over time
- **Agent & tool execution** — tool invocations by name, agent run durations
- **Trace explorer** — full distributed trace view via Tempo + TraceQL

---

## Stack

| Service | Purpose | Port |
|---|---|---|
| `otel-collector` | Receives OTLP from VS Code, fans out to Prometheus + Tempo | 4317 (gRPC), 4318 (HTTP) |
| `prometheus` | Stores metrics, scraped by Grafana | 9090 |
| `tempo` | Stores traces, queried by Grafana via TraceQL | 3200 |
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
```

Verify services are healthy:

```bash
docker compose ps
```

### 2. Configure VS Code

Open your user settings JSON (`Ctrl+Shift+P` → *Open User Settings (JSON)*) and add the contents of [`vscode/settings.json`](./vscode/settings.json):

```json
{
  "github.copilot.chat.otel.enabled": true,
  "github.copilot.chat.otel.exporterType": "otlp-http",
  "github.copilot.chat.otel.otlpEndpoint": "http://localhost:4318",
  "github.copilot.chat.otel.captureContent": false
}
```

> Restart VS Code after adding these settings so the OTEL exporter initialises.

### 3. Open Grafana

Navigate to [http://localhost:3000](http://localhost:3000) and log in with:

- **Username:** `admin`
- **Password:** `copilot` (or whatever you set in `.env`)

The **GitHub Copilot — OTEL Dashboard** is auto-provisioned under *Dashboards → GitHub Copilot*.

---

## Sending Test Data

Fire a quick Copilot Chat prompt in VS Code, then check the collector logs to confirm data is flowing:

```bash
docker compose logs -f otel-collector
```

You should see `ScopeSpans` and `ScopeMetrics` lines within seconds of a chat interaction.

---

## Security Notes

- `captureContent: false` by default — prompts and completions are **not** stored in traces.
- Change `GF_SECURITY_ADMIN_PASSWORD` before exposing Grafana on a non-loopback interface.
- The OTEL collector's CORS policy allows `*` by default — lock this down if you deploy remotely.

---

## Metrics Reference

The dashboard queries the following metric names as exported by the Copilot OTEL collector:

| Metric | Type | Description |
|---|---|---|
| `copilot_llm_requests_total` | Counter | Total LLM calls |
| `copilot_llm_duration_milliseconds` | Histogram | LLM response time |
| `copilot_llm_errors_total` | Counter | LLM failures |
| `copilot_tool_invocations_total` | Counter | MCP / tool calls |
| `copilot_agent_duration_milliseconds` | Histogram | End-to-end agent run time |

> Metric names may vary with Copilot Chat extension versions — adjust PromQL queries in Grafana as needed.

---

## Stopping the Stack

```bash
docker compose down          # stop, keep volumes
docker compose down -v       # stop + delete all data
```
