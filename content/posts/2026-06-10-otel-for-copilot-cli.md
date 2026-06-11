---
title:  "OpenTelemetry for GitHub Copilot CLI"
date:   2026-06-10 18:51:00 +0100
---

GitHub Copilot CLI can export OpenTelemetry data that helps inspect model calls, tool invocations, MCP activity, token usage and latency.

This article relates to [OpenTelemetry for Codex CLI](/2026/05/10/opentelemetry-for-codex-cli/) and focuses on the Copilot-specific setup and telemetry behavior. Use the Compose example from the Codex article to bring up local OTel stack.

The important Copilot-specific differences are:

- Copilot CLI exports OTLP over HTTP to `http://127.0.0.1:4318`.
- Codex CLI exports OTLP over gRPC to `http://127.0.0.1:4317`.

## Traces

Copilot CLI telemetry is most useful when inspected as traces. From local captures, the most useful span categories are:

- chat spans, named like `chat <model>`, useful for model usage and model-call latency
- tool spans, named like `execute_tool <tool>`, useful for understanding tool usage and slow tool calls
- MCP tool spans, named like `execute_tool github-mcp-server-*`, useful for tracking MCP-backed tool activity
- permission spans, useful for understanding approval and permission checks
- internal spans, useful for troubleshooting Copilot CLI runtime behavior

In Grafana Explore, start with broad Tempo queries and then narrow them after inspecting one trace, because exact attribute names can vary by Copilot CLI version and instrumentation.

```traceql
{ resource.service.name =~ "github-copilot" }
{ resource.service.name =~ "github-copilot" && status = error }
{ resource.service.name =~ "github-copilot" && duration > 30s }
{ resource.service.name =~ "github-copilot" && name =~ "chat .+" }
{ resource.service.name =~ "github-copilot" && name =~ "execute_tool .+" }
{ resource.service.name =~ "github-copilot" && name =~ "execute_tool github-mcp-server-.+" }
```

## Metrics

The local stack described in the Codex article can derive Copilot operation metrics from traces through the collector's `spanmetrics` connector. That produces Prometheus metrics such as:

- `traces_span_metrics_calls_total`
- `traces_span_metrics_duration_milliseconds_bucket`

These metrics can be used to count operations, find errors and calculate latency percentiles.

```promql
{__name__=~"gen_ai_client_token_usage(_tokens)?_sum",service_name=~"github-copilot"}
```

Other useful PromQL examples:

```promql
sum by (span_name) (
  max_over_time(traces_span_metrics_calls_total{service_name=~"github-copilot"}[$__range])
)

sum by (model) (
  label_replace(
    max_over_time(traces_span_metrics_calls_total{service_name=~"github-copilot",span_name=~"chat .+"}[$__range]),
    "model",
    "$1",
    "span_name",
    "chat (.+)"
  )
)

sum by (tool) (
  label_replace(
    max_over_time(traces_span_metrics_calls_total{service_name=~"github-copilot",span_name=~"execute_tool .+"}[$__range]),
    "tool",
    "$1",
    "span_name",
    "execute_tool (.+)"
  )
)

histogram_quantile(
  0.95,
  sum by (le, span_name) (
    rate(traces_span_metrics_duration_milliseconds_bucket{service_name=~"github-copilot"}[$__rate_interval])
  )
)

sum by (gen_ai_token_type) (
  max_over_time({__name__=~"gen_ai_client_token_usage(_tokens)?_sum",service_name=~"github-copilot"}[$__range])
)
```

Queries using `max_over_time(...)` are useful for dashboard snapshots because latest reported totals remain visible after a run finishes. Queries using `rate(...[$__rate_interval])` are more useful while Copilot traffic is active.

## Configuration

Run Copilot CLI with telemetry enabled and point it at the OTLP HTTP endpoint exposed by the local stack:

```bash
COPILOT_OTEL_ENABLED=true \
COPILOT_OTEL_EXPORTER_TYPE=otlp-http \
OTEL_EXPORTER_OTLP_ENDPOINT=http://127.0.0.1:4318 \
OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE=cumulative \
OTEL_SERVICE_NAME=github-copilot \
COPILOT_OTEL_SOURCE_NAME=github.copilot \
OTEL_LOG_LEVEL=INFO \
OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT=false \
OTEL_RESOURCE_ATTRIBUTES=service.namespace=copilot-cli,deployment.environment=local \
copilot
```

If you use `gh copilot` instead of the standalone `copilot` binary, keep the same environment variables and replace the final command.

Keeping `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT=false` avoids exporting raw message content. If you enable message capture, treat the telemetry backend as sensitive because prompts and responses can contain private data.

## Conclusion

GitHub Copilot CLI telemetry is useful for local debugging and for understanding agent behavior over time. The most important pieces are traces in Tempo, span-derived metrics in Prometheus and GenAI token usage metrics. For production use, be deliberate about message capture and resource attributes, and expect some metric or span details to evolve as Copilot CLI and OpenTelemetry GenAI conventions mature.

## References

- [OpenTelemetry for Codex CLI](https://ivanfranjic.net/2026/05/10/opentelemetry-for-codex-cli/)
- https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference
- https://opentelemetry.io/docs/specs/semconv/gen-ai/
