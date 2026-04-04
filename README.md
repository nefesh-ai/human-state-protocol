# Human State Protocol (HSP)

An open specification for exchanging human physiological state between AI systems.

## What is HSP?

The Human State Protocol defines a standard JSON format for representing human physiological and behavioral state in a way that any AI system can consume. It answers one question: **How is the human doing right now?**

HSP is to human state what JSON is to structured data: a shared format that many systems can implement.

## Why a protocol?

AI agents today have no standard way to receive, interpret, or act on human physiological signals. Every integration is custom. HSP provides:

- A canonical JSON schema for human state output
- Defined state levels with machine-readable actions
- Signal category taxonomy (cardiovascular, vocal, visual, textual, and more)
- Confidence and device trust metadata
- Cross-session context via trigger memory

## Schema

The HSP schema is defined in [`schema/hsp-1.0.schema.json`](schema/hsp-1.0.schema.json).

### Core fields

| Field | Type | Description |
|---|---|---|
| `state` | string | `calm`, `relaxed`, `focused`, `stressed`, `acute_stress` |
| `stress_score` | integer | 0-100 (lower = calmer) |
| `confidence` | float | 0.0-1.0 (signal quality and device trust) |
| `suggested_action` | string | `maintain_engagement`, `simplify_and_focus`, `de-escalate_and_shorten`, `pause_and_ground` |
| `action_reason` | string | Human-readable explanation |
| `signals_received` | array | Signal categories present in this reading |
| `adaptation_effectiveness` | object | Closed-loop feedback (previous action, stress delta, effective boolean) |

### Signal categories

| Category | Example signals |
|---|---|
| `cardiovascular` | heart_rate, rmssd, sdnn, spo2 |
| `vocal` | tone, speech_rate, pitch_variability |
| `visual` | expression, gaze, posture, engagement |
| `textual` | sentiment, urgency |
| `metabolic` | glucose_mg_dl, glucose_trend |
| `neural` | eeg_alpha_power, cognitive_load |
| `electrodermal` | eda, skin_temperature |
| `respiratory` | respiratory_rate |
| `movement` | steps_last_minute, activity_level |
| `sleep` | sleep_stage |

### State mapping

| Score | State | Suggested Action |
|---|---|---|
| 0-19 | `calm` | `maintain_engagement` |
| 20-39 | `relaxed` | `maintain_engagement` |
| 40-59 | `focused` | `simplify_and_focus` |
| 60-79 | `stressed` | `de-escalate_and_shorten` |
| 80-100 | `acute_stress` | `pause_and_ground` |

## Example

```json
{
  "hsp_version": "1.0",
  "session_id": "session-001",
  "timestamp": "2026-04-01T12:00:00Z",
  "state": "stressed",
  "stress_score": 68,
  "confidence": 0.87,
  "signals_received": ["cardiovascular", "vocal", "textual"],
  "suggested_action": "de-escalate_and_shorten",
  "action_reason": "elevated heart rate + tense vocal tone + negative sentiment",
  "recommendation": "Max 2 sentences. Direct, factual, no ambiguity.",
  "adaptation_effectiveness": {
    "previous_action": "simplify_and_focus",
    "previous_score": 52,
    "current_score": 68,
    "stress_delta": 16,
    "effective": false
  }
}
```

See [`examples/`](examples/) for more.

## Implementations

| Implementation | Protocol | Status |
|---|---|---|
| [Nefesh API](https://nefesh.ai) | REST (`/v1/ingest`, `/v1/state`) | Production |
| [Nefesh MCP Server](https://github.com/nefesh-ai/nefesh-mcp-server) | MCP (Streamable HTTP) | Production |
| [Nefesh A2A Agent](https://github.com/nefesh-ai/nefesh-a2a) | A2A v1.0 (JSON-RPC) | Production |
| [Nefesh Gateway](https://github.com/nefesh-ai/nefesh-gateway) | OpenAI-compatible LLM proxy | Production |

## Design principles

1. **Format, not algorithm.** HSP defines the output shape, not how to compute it. Any fusion algorithm that produces HSP-compliant output is valid.
2. **Edge-first.** Raw signals (video, audio) should be processed on-device. Only derived metrics cross the network.
3. **Privacy by default.** No PII in the schema. Session IDs are client-generated. Subject IDs must be pre-hashed.
4. **Not a medical device.** HSP is for contextual AI adaptation only. No clinical interpretation.

## License

Apache 2.0
