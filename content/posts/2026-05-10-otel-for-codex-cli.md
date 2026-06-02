---
title:  "OpenTelemetry for Codex CLI"
date:   2026-05-10 19:37:00 +0100
---

Codex CLI can export traces, metrics, OTel log records with event names and attributes. With that telemetry, you can track API requests, tool invocations, token usage, MCP calls and run latency.

This article walks through a local OpenTelemetry stack and Codex CLI configuration needed to send telemetry to it. The setup should help you explore the traces, logs and metrics, build dashboards and answer questions such as:
- Which models were used?
- How many tool calls were made?
- Was MCP used?
- How long did the run take?

## Traces

While Codex CLI can export traces, its trace spans are not fully documented, and there is no official list of span names.

In captured traces, I saw these broad span categories:
- session lifecycle related (useful for knowing when turn begins and ends)
- network and streaming related (useful to troubleshoot network vs model latency issues)
- tool use (useful to see how often and which tools are used, as well as how long it takes)
- startup related (useful to troubleshoot first turn overhead that comes from auth, environment setup or model catalog lookups)
- low-level internal spans (useful to Codex CLI developers debugging runtime behavior)

These traces help identify the largest contributors to end-to-end latency.

## Logs

Codex CLI log telemetry is emitted as structured OTel log records. As with traces, the event payloads are not fully documented, and there is no stable per-event schema.

From captured logs, I've noted some useful log events:
- `codex.user_prompt` exposes fields such as `model` and `prompt`
- `codex.tool_result` exposes fields `tool_name`, `success`, `duration_ms` and others
- `codex.sse_event` with kind `response.completed` exposes `input_token_count`, `cached_token_count`, `output_token_count`, `reasoning_token_count` and `tool_token_count`

There are many use cases for collecting this data:
- use `codex.user_prompt` to reproduce a run or check for accidental disclosure of sensitive data
- use `codex.tool_result` to investigate failed or slow tool calls
- use `codex.sse_event` and `response.completed` to inspect token usage
- use `codex.conversation_starts` and `codex.tool_decision` to audit safety and approval behavior

```
# Loki query examples
{service_name=~"codex_cli_rs|codex_exec"} | event_name="codex.user_prompt"
{service_name=~"codex_cli_rs|codex_exec"} | event_name="codex.tool_result"
{service_name=~"codex_cli_rs|codex_exec"} | event_name="codex.sse_event" | event_kind="response.completed"
```
## Metrics

For metrics, Codex exposes counters and histograms for runtime activity such as API requests, streaming events, WebSocket request, tool calls, turn latency, token usage, and MCP activity.

The most useful metrics I've found from my captures are:
- token related metrics, which can be used to track cost
- tool related metrics, which can be used to explain agent behavior
- turn latency and MCP related metrics, which can be used to identify performance issues

```
# PromQL examples
sum(max_over_time(codex_turn_token_usage_sum{token_type="total"}[$__range]))
sum by (tool) (max_over_time(codex_tool_call_total[$__range]))
```

## Configuration

To try this locally, you need an OTel stack. If you do not already have one, the following Compose file starts Grafana LGTM with Prometheus, Loki, Tempo, Pyroscope, and an OTel Collector.

```
name: otel

services:
  lgtm:
    image: docker.io/grafana/otel-lgtm:0.28.0
    restart: unless-stopped
    ports:
      - "127.0.0.1:3000:3000" # Grafana UI
      - "127.0.0.1:3100:3100" # Loki API
      - "127.0.0.1:3200:3200" # Tempo API
      - "127.0.0.1:4040:4040" # Pyroscope API
      - "127.0.0.1:4317:4317" # OTLP gRPC ingest for Codex CLI
      - "127.0.0.1:4318:4318" # OTLP HTTP ingest for Copilot CLI
      - "127.0.0.1:9090:9090" # Prometheus UI and API
    volumes:
      - ./otel/collector-config.yaml:/otel-lgtm/otelcol-config.yaml:ro
      - ./prometheus/prometheus.yml:/otel-lgtm/prometheus.yaml:ro
      - ./grafana/provisioning/dashboards/dashboards.yaml:/otel-lgtm/grafana/conf/provisioning/dashboards/agents.yaml:ro
      - ./grafana/provisioning/dashboards-json:/otel-lgtm/grafana/conf/provisioning/dashboards-json:ro
      - lgtm-data:/data
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: admin
      GF_USERS_ALLOW_SIGN_UP: "false"
      GF_PATHS_DATA: /data/grafana
    healthcheck:
      test:
        - CMD-SHELL
        - test -f /tmp/ready && /otel-lgtm/docker/healthcheck.sh
      interval: 5s
      timeout: 5s
      retries: 24
      start_period: 10s

volumes:
  lgtm-data: {}
```

The stack uses OTel collector which receives OTLP traffic, Prometheus for metrics, Loki for logs, Tempo for traces and Grafana for exploring and dashboards.

OTel collector configuration:

```
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
        cors:
          allowed_origins:
            - http://*
  prometheus/collector:
    config:
      scrape_configs:
        - job_name: opentelemetry-collector
          scrape_interval: 1s
          static_configs:
            - targets:
                - 127.0.0.1:8888

extensions:
  health_check:
    endpoint: 0.0.0.0:13133
    path: /ready

processors:
  batch:

connectors:
  spanmetrics:

exporters:
  prometheus:
    endpoint: 0.0.0.0:9464
    resource_to_telemetry_conversion:
      enabled: true
  otlphttp/tempo:
    endpoint: http://127.0.0.1:4418
  otlphttp/loki:
    endpoint: http://127.0.0.1:3100/otlp
  otlp/profiles:
    endpoint: 127.0.0.1:4040
    tls:
      insecure: true

service:
  extensions:
    - health_check
  pipelines:
    traces:
      receivers:
        - otlp
      processors:
        - batch
      exporters:
        - otlphttp/tempo
        - spanmetrics
    metrics:
      receivers:
        - otlp
        - prometheus/collector
        - spanmetrics
      processors:
        - batch
      exporters:
        - prometheus
    logs:
      receivers:
        - otlp
      processors:
        - batch
      exporters:
        - otlphttp/loki
    profiles:
      receivers:
        - otlp
      exporters:
        - otlp/profiles
```

Prometheus configuration:

```
global:
  scrape_interval: 5s
  evaluation_interval: 5s
  scrape_native_histograms: true

otlp:
  keep_identifying_resource_attributes: true
  promote_resource_attributes:
    - service.instance.id
    - service.name
    - service.namespace
    - service.version
    - deployment.environment
    - deployment.environment.name
    - host.name

storage:
  tsdb:
    out_of_order_time_window: 10m

scrape_configs:
  - job_name: otel-collector
    static_configs:
      - targets:
          - 127.0.0.1:9464
```

Now configure Codex CLI by editing configuration file `~/.codex/config.toml`:

```
[otel]
environment = "local-podman"
log_user_prompt = true
exporter = { otlp-grpc = { endpoint = "http://127.0.0.1:4317" } }
metrics_exporter = { otlp-grpc = { endpoint = "http://127.0.0.1:4317" } }
trace_exporter = { otlp-grpc = { endpoint = "http://127.0.0.1:4317" } }
```

Alternatively, you can use CLI flags to pass OTel configuration:


```
#!/usr/bin/env bash
set -euo pipefail

exec codex \
  --config 'otel.environment="local-podman"' \
  --config 'otel.log_user_prompt=true' \
  --config 'otel.exporter={ "otlp-grpc" = { endpoint = "http://127.0.0.1:4317" } }' \
  --config 'otel.metrics_exporter={ "otlp-grpc" = { endpoint = "http://127.0.0.1:4317" } }' \
  --config 'otel.trace_exporter={ "otlp-grpc" = { endpoint = "http://127.0.0.1:4317" } }' \
  "$@"
```

## Conclusion

Codex CLI's OTel support is already useful for local debugging. Expect the details to evolve, especially while OTel semantic conventions for generative AI continue to mature. For production use, remember that prompts may contain sensitive information, so keep `log_user_prompt = false` unless you explicitly need raw prompt text.

## References

- [Advanced configuration](https://developers.openai.com/codex/config-advanced#observability-and-telemetry)
- [Semantic conventions for generative AI systems](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
