# Claude SDLC Pipeline

A 4-stage software development pipeline powered by [Claude Code](https://claude.ai/code). Each stage runs as a Claude Code slash command — no API key needed, just a Claude Code subscription.

## Stages

| Stage | Command | What it does |
|-------|---------|--------------|
| 1 | `pipeline-req` | Interviews you (or reads mock answers) and writes `spec.md` |
| 2 | `pipeline-build` | Reads `spec.md` and writes working `code.py` |
| 3 | `pipeline-test` | Syntax-checks and reviews the code, writes `report.md` |
| 4 | `pipeline-deploy` | Commits artifacts and tags `v0.1.0` if tests passed |

Each run gets its own timestamped workspace under `pipeline/runs/`.

## Requirements

- [Claude Code](https://claude.ai/code) installed and authenticated
- Python 3.9+

## Usage

### Interactive (asks you questions)

```bash
python pipeline/orchestrator.py --project my-tool
```

### Automated / CI (mock answers, no human input)

```bash
# CSV summarizer demo
python pipeline/harness.py

# HTTP health checker demo
python pipeline/harness.py health
```

Add `--skip-permissions` (or let the harness pass it automatically) to run without approval prompts.

## Project structure

```
.claude/commands/        # Slash command definitions (one per stage)
pipeline/
  orchestrator.py        # Drives the 4 stages in sequence
  harness.py             # Mock presets for CI / demo use
  runs/                  # Timestamped workspaces (gitignored)
```

## Adding your own mock

Edit `harness.py` and add an entry to `MOCKS`:

```python
"my-tool": {
    "project": "my-tool-name",
    "mock_answers": [
        "What it does",
        "Language/library constraints",
        "Exact inputs and outputs",
        "Edge cases to handle",
    ],
},
```

Then run `python pipeline/harness.py my-tool`.

## How it works

`orchestrator.py` calls `claude -p /<stage> <workspace>` for each stage. Claude Code executes the slash command defined in `.claude/commands/`, reads/writes files in the workspace, and exits. The orchestrator checks the exit code and halts if any stage fails.
