# Instrumenting your own apps into this stack

This stack accepts standard OTLP over:

- gRPC: `http://YOUR-HOST:4317`
- HTTP: `http://YOUR-HOST:4318`

## Required resource attributes

Set these in your app:

- `service.name=cyrene-labs` or `service.name=infrapeek`
- `deployment.environment=dev|staging|prod`
- `service.version=<git-sha-or-semver>`

## Example OTLP env vars

### Node / Next / Express

```bash
export OTEL_SERVICE_NAME=cyrene-labs
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
```

### Python

```bash
export OTEL_SERVICE_NAME=infrapeek
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
```

### Go

```bash
export OTEL_SERVICE_NAME=cyrene-labs
export OTEL_EXPORTER_OTLP_ENDPOINT=localhost:4317
export OTEL_EXPORTER_OTLP_INSECURE=true
```

## Recommended metrics to expose

For app dashboards to light up cleanly, emit:

- `http.server.request.duration`
- trace spans with `service.name`
- log records with `service.name`

Prometheus-normalized names used by bundled dashboards:

- `http_server_request_duration_bucket`
- `http_server_request_duration_count`
- `target_info`

## Grafana layout

Provisioned folders:

- `GitHub Copilot`
- `Applications`

Provisioned application dashboards:

- `Application OTEL Dashboard` (service variable template)
- `Cyrene Labs Overview`
- `InfraPeek Overview`
