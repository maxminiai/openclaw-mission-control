# Model routing and savings (local-first)

This runbook documents how Mission Control operations are currently using a local-first model dispatcher in the OpenClaw workspace.

## Why

- Keep most inference on local Ollama models
- Use OpenRouter only as controlled fallback
- Preserve privacy (sensitive tasks remain local)
- Track provider/model usage and estimate cloud spend avoided

## Current local routing setup

Dispatcher files live in:

- `/Users/max/.openclaw/workspace/scripts/model_dispatcher.py`
- `/Users/max/.openclaw/workspace/scripts/dispatch.sh`

Routing policy (summary):

- `quick` -> `llama3.2:3b`
- `normal` -> `qwen2.5:7b-instruct`
- `coding` -> `qwen2.5-coder:7b`
- OpenRouter fallback allowed only when:
  - `--allow-openrouter` is set
  - sensitivity is `non_sensitive`
  - local execution fails

OpenRouter fallback model:

- `google/gemma-3-4b-it:free`

## Logging and audit trail

Per-dispatch JSONL log:

- `/Users/max/.openclaw/workspace/logs/model-routing.log`

Each line includes:

- timestamp (`ts`)
- task class
- sensitivity
- provider (local/openrouter)
- model
- fallback flag
- elapsed time
- success flag

Prompt text is intentionally **not logged**.

## Health checks

Run dispatcher validation:

```bash
/Users/max/.openclaw/workspace/scripts/healthcheck_dispatch.sh
```

Checks include:

- required local model presence
- route correctness by task class
- OpenRouter fallback path

## Reporting + savings estimate

Generate rolling report:

```bash
/Users/max/.openclaw/workspace/scripts/routing_report.sh
```

Outputs windows:

- today
- 7d
- 30d
- all time

For each window:

- provider/model/task-class usage
- average latency
- estimated tokens executed locally
- estimated cloud spend avoided (heuristic)

### Estimation caveat

Savings are approximate and based on task-class token assumptions, not billing exports.

## Security notes

- Keep `OPENROUTER_API_KEY` in a local env file only.
- Rotate keys if ever shared in plaintext chat.
- Treat unknown-sensitivity tasks as local-only.
