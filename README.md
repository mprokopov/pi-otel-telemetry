# Pi OTEL Telemetry Extension

OpenTelemetry tracing and metrics for [pi coding agent](https://github.com/badlogic/pi-mono). Exports spans and counters for sessions, agent prompts, LLM turns, and tool executions.

## Trace Structure

```
session                          (root span, entire session lifecycle)
├── agent.prompt                 (one per user message)
│   └── agent.turn               (one per LLM call + tool execution cycle)
│       ├── llm.request          (span event: before provider API call)
│       ├── tool.bash            (tool execution span)
│       ├── tool.read            (tool execution span)
│       └── tool.edit            (tool execution span)
├── model.changed                (span event on model switch)
└── session.compacted            (span event on compaction)
```

## Metrics

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `pi.tokens.input` | Counter | `llm.model` | Total input tokens consumed |
| `pi.tokens.output` | Counter | `llm.model` | Total output tokens produced |
| `pi.tool.calls` | Counter | `tool.name` | Total tool invocations |
| `pi.tool.errors` | Counter | `tool.name` | Total failed tool invocations |
| `pi.tool.duration` | Histogram (ms) | `tool.name` | Tool execution time |
| `pi.turns` | Counter | `llm.model` | Total LLM turns |
| `pi.prompts` | Counter | `llm.model` | Total user prompts |
| `pi.session.duration` | Histogram (s) | `llm.model` | Session duration |

## Trace Attributes

### Session span
| Attribute | Description |
|-----------|-------------|
| `session.id` | Session file path or "ephemeral" |
| `session.cwd` | Working directory |
| `session.turns` | Total turn count |
| `session.tool_calls` | Total tool calls |
| `session.tokens.input` | Total input tokens |
| `session.tokens.output` | Total output tokens |
| `llm.model` | Current model (provider/id) |

### Turn span
| Attribute | Description |
|-----------|-------------|
| `turn.index` | Turn index within agent prompt |
| `turn.number` | Global turn number in session |
| `turn.tool_results` | Number of tool results |
| `llm.usage.input_tokens` | Tokens consumed (input) |
| `llm.usage.output_tokens` | Tokens produced (output) |

### Tool span
| Attribute | Description |
|-----------|-------------|
| `tool.name` | Tool name (bash, read, edit, write, etc.) |
| `tool.call_id` | Unique tool call ID |
| `tool.args_summary` | Brief summary of arguments (truncated) |
| `tool.is_error` | Whether execution failed |
| `tool.duration_ms` | Execution time in milliseconds |

## Configuration

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `OTEL_EXPORTER_OTLP_ENDPOINT` | `http://localhost:4318` | OTLP HTTP endpoint |
| `OTEL_SERVICE_NAME` | `pi-coding-agent` | Service name in traces |
| `PI_OTEL_ENABLED` | `true` | Set to `false` to disable |
| `PI_OTEL_DEBUG` | `false` | Set to `true` to also log spans/metrics to console |
| `OTEL_METRIC_EXPORT_INTERVAL` | `10000` | Metric export interval in ms |

## Quick Start

### With Jaeger (local)

```bash
# Start Jaeger with OTLP support
docker run -d --name jaeger \
  -p 16686:16686 \
  -p 4318:4318 \
  jaegertracing/jaeger:2 \
  --set receivers.otlp.protocols.http.endpoint=0.0.0.0:4318

# Start pi (extension auto-discovered from ~/.pi/agent/extensions/)
pi

# Open Jaeger UI
open http://localhost:16686
```

### With Grafana Tempo

```bash
OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4318 pi
```

### Debug mode (console output)

```bash
PI_OTEL_DEBUG=true pi
```

### Disable

```bash
PI_OTEL_ENABLED=false pi
```

## Installation

Already installed at `~/.pi/agent/extensions/otel-telemetry/`. The extension is auto-discovered by pi.

To reinstall dependencies:

```bash
cd ~/.pi/agent/extensions/otel-telemetry
npm install
```
