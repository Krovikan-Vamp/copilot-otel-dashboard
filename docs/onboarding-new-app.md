# Onboarding a New App to the OTEL Hub

This stack is a shared OpenTelemetry hub. Any app can send metrics, traces, and logs here and get a pre-built Grafana dashboard automatically.

---

## 1. Set Required Resource Attributes

Every app **must** set these two OTEL resource attributes:

| Attribute | Example | Notes |
|---|---|---|
| `service.name` | `cyrene-labs-api` | Used as the primary filter in all dashboards |
| `deployment.environment` | `dev`, `staging`, `prod` | Defaults to `local` if omitted |

Optional but recommended:

| Attribute | Example |
|---|---|
| `service.namespace` | `cyrene-labs` |
| `service.version` | `1.4.2` |
| `host.name` | `api-01` |

---

## 2. Point Your App at the Collector

Set these environment variables in your app or container:

```bash
# gRPC (preferred for server-side apps)
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
OTEL_EXPORTER_OTLP_PROTOCOL=grpc

# OR HTTP (good for browser/edge apps)
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf

# Resource attributes
OTEL_SERVICE_NAME=your-app-name
OTEL_RESOURCE_ATTRIBUTES=deployment.environment=dev,service.namespace=your-namespace
```

For apps running in the same Docker network (`otel`):
```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

---

## 3. Add Your App to the Collector Config (optional)

The collector already accepts all OTLP traffic without any app-specific config. However, if you want per-app attribute enrichment or redaction, add a named processor in `config/otel-collector.yaml`:

```yaml
processors:
  # ... existing processors ...

  resource/your-app:
    attributes:
      - key: service.namespace
        value: your-namespace
        action: upsert
```

Then wire it into the pipelines under `service.pipelines`.

---

## 4. Create a Dashboard

Copy one of the existing app dashboard templates:

```bash
cp config/grafana/dashboards/apps/cyrene-labs.json \
   config/grafana/dashboards/apps/your-app.json
```

Then edit the copy:
1. Change `"uid"` to a unique value (e.g. `your-app`)
2. Change `"title"` to your app name
3. Replace all `cyrene-labs.*` regex patterns with `your-app-name.*`
4. Commit and push — Grafana auto-reloads dashboards every 30s

---

## 5. Verify Data is Flowing

```bash
# Check metrics landed in Prometheus
curl -s http://localhost:8889/metrics | grep your_app_name

# Check logs landed in Loki
curl -s "http://localhost:3100/loki/api/v1/query?query={service_name%3D%22your-app-name%22}&limit=5"

# Check traces landed in Tempo
curl -s "http://localhost:3200/api/search?tags=service.name%3Dyour-app-name&limit=5" | jq .traces
```

---

## Instrumentation Libraries

| Language | Library | Docs |
|---|---|---|
| **Go** | `go.opentelemetry.io/otel` | [opentelemetry.io/docs/languages/go](https://opentelemetry.io/docs/languages/go/) |
| **TypeScript/Node** | `@opentelemetry/sdk-node` | [opentelemetry.io/docs/languages/js](https://opentelemetry.io/docs/languages/js/) |
| **Python** | `opentelemetry-sdk` | [opentelemetry.io/docs/languages/python](https://opentelemetry.io/docs/languages/python/) |
| **Cloudflare Workers** | `@microlabs/otel-cf-workers` | Wraps the OTel SDK for the Workers runtime |
