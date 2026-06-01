# copilot-otel-dashboard

A self-hosted OpenTelemetry monitoring stack for GitHub Copilot Chat in VS Code **and your own applications**.

This repo is now structured as a **one-stop shop** for dashboards across:

- GitHub Copilot / VS Code agent telemetry
- Cyrene Labs
- InfraPeek
- any future OTEL-enabled app that exports metrics, logs, and traces

## Core idea

Everything ships OTLP into the same collector.
Use `service.name` to separate sources.
Grafana provides:

- a dedicated `GitHub Copilot` folder
- an `Applications` folder
- a reusable application dashboard template driven by a service selector
- starter dashboards for `cyrene-labs` and `infrapeek`

## Services

| Service | Purpose | Port |
|---|---|---|
| `otel-collector` | Receives OTLP from Copilot and your apps | 4317, 4318 |
| `prometheus` | Metrics store | 9090 |
| `tempo` | Trace store | 3200 |
| `grafana` | Dashboards and traces | 3000 |

## Quick start

```bash
git clone https://github.com/Krovikan-Vamp/copilot-otel-dashboard.git
cd copilot-otel-dashboard
cp .env.example .env
docker compose up -d
```

Then instrument apps to point at:

- OTLP gRPC: `localhost:4317`
- OTLP HTTP: `http://localhost:4318`

with `service.name` values such as:

- `copilot-chat`
- `cyrene-labs`
- `infrapeek`

## Included dashboards

### GitHub Copilot
- existing Copilot overview dashboard

### Applications
- `Application OTEL Dashboard` — choose a service from a variable dropdown
- `Cyrene Labs Overview`
- `InfraPeek Overview`

## Adding a new app

1. Set `service.name` in the app
2. Export OTLP to this collector
3. Open Grafana → Applications → Application OTEL Dashboard
4. Pick the service from the dropdown
5. Clone that dashboard if you want an app-specific version

See [`docs/ADDING-APPS.md`](./docs/ADDING-APPS.md).
