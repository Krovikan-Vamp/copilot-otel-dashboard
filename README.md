# copilot-otel-dashboard

A local OpenTelemetry observability hub. Ships with:

- **GitHub Copilot Chat** metrics, traces, and logs out of the box
- **Multi-app support** — wire in any service with a `service.name` and two env vars
- **Grafana** dashboards for Copilot, Cyrene Labs, InfraPeek, and a cross-service overview
- **Full OTEL stack** — Prometheus (metrics) · Tempo (traces) · Loki (logs)

## Quick Start

```bash
git clone https://github.com/Krovikan-Vamp/copilot-otel-dashboard
cd copilot-otel-dashboard
cp .env.example .env
docker compose up -d
```

Grafana → http://localhost:3000 (admin / copilot)

## VS Code Setup

Add to your `settings.json`:

```json
{
  "github.copilot.chat.otel.enabled": true,
  "github.copilot.chat.otel.exporterType": "otlp-http",
  "github.copilot.chat.otel.otlpEndpoint": "http://localhost:4318",
  "github.copilot.chat.otel.captureContent": true
}
```

## Adding a New App

See **[docs/onboarding-new-app.md](docs/onboarding-new-app.md)** for the full guide.

TL;DR — set two env vars in your app and get instant metrics/traces/logs:

```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
OTEL_SERVICE_NAME=your-app-name
```

## Stack

| Service | Port | Purpose |
|---|---|---|
| otel-collector | 4317 (gRPC), 4318 (HTTP) | OTLP ingress for all apps |
| Prometheus | 9090 | Metrics storage |
| Loki | 3100 | Log storage |
| Tempo | 3200 | Trace storage |
| Grafana | 3000 | Dashboards |

## Dashboards

| Dashboard | Description |
|---|---|
| Service Overview | Cross-app throughput, LLM ops, and logs |
| GitHub Copilot Chat | Chat sessions, tool calls, token usage, latency |
| Cyrene Labs | HTTP performance, traces, logs |
| InfraPeek | HTTP performance, infra checks, traces, logs |
