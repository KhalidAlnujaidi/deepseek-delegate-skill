# Delegate — Strategic Task Delegation Skill for DeepSeek TUI

Plan the work, then ship the labor to `deepseek-v4-flash` sub-agents.

## What it does

The coordinating agent (you, typically running `deepseek-v4-pro`) handles reasoning, architecture, and design decisions. Sub-agents running `deepseek-v4-flash` (~10x cheaper per token) handle execution: scaffold generation, boilerplate, code writing, search/grep, test creation, and file edits.

## How it works

```
USER PROMPT
    │
    ▼
┌──────────────────────────────────────┐
│  PARENT (deepseek-v4-pro)            │
│  • Understand the request            │
│  • Reason about architecture         │
│  • Plan with checklist_write         │
│  • Identify delegation units         │
└──────────┬───────────────────────────┘
           │
           │  spawn flash sub-agents
           │  (all in one turn → parallel)
           ▼
    ┌──────┴──────┬──────────────┐
    ▼             ▼              ▼
┌────────┐   ┌────────┐   ┌────────┐
│ FLASH  │   │ FLASH  │   │ FLASH  │
│ code   │   │ boiler │   │ search │
│ gen    │   │ plate  │   │ / test │
└───┬────┘   └───┬────┘   └───┬────┘
    │            │             │
    └──────┬─────┴─────────────┘
           │  agent_wait → collect
           ▼
┌──────────────────────────────────────┐
│  PARENT (deepseek-v4-pro)            │
│  • Verify sub-agent output           │
│  • Cross-check findings              │
│  • Run gates (cargo check, test)     │
│  • Synthesize for user               │
└──────────────────────────────────────┘
```

## What you'll see in the TUI

**Planning phase** — parent lays out the work:

```
checklist_write
  [1/4] Create project scaffold ......... delegate → flash
  [2/4] Write CSV parser module ......... delegate → flash
  [3/4] Write stats module .............. delegate → flash
  [4/4] Integration + cargo check ....... direct
```

**Execution phase** — flash sub-agents fire in parallel:

```
agent_spawn  { model: "deepseek-v4-flash", type: "implementer" }
agent_spawn  { model: "deepseek-v4-flash", type: "implementer" }
agent_spawn  { model: "deepseek-v4-flash", type: "implementer" }
       ↑ all three run simultaneously
```

**Synthesis phase** — parent collects and verifies:

```
agent_wait   → all three complete
cargo check  → PASS
cargo test   → PASS
```

## Quick test

Open a DeepSeek TUI session and give it a multi-step task:

> Build a small Rust CLI in /tmp/csvstats that reads a CSV and prints row count,
> column names, and min/max/mean for numeric columns. Include Cargo.toml,
> main.rs, and a lib.rs with parser + stats modules.

Then watch the TUI — you'll see `agent_spawn` calls with `model: "deepseek-v4-flash"` fire in parallel for file generation. After the session, inspect the log:

```bash
# Count flash sub-agents used
grep -c '"model":"deepseek-v4-flash"' ~/.deepseek/audit.log

# Show all sub-agent spawns with timestamps
grep "agent_spawn" ~/.deepseek/audit.log | jq '{time: .timestamp, model: .model}'

# Show full delegation timeline
grep -E "agent_spawn|agent_wait|agent_result" ~/.deepseek/audit.log | jq -r '[.timestamp, .tool, .model // ""] | @tsv'
```

## Install

```
/skill install github:KhalidAlnujaidi/deepseek-delegate-skill
```

## Trigger

The skill auto-activates when your task involves multi-step implementation, code generation, or refactoring. You can also trigger it explicitly:

```
/skill delegate
```

## Structure

```
delegate/
└── SKILL.md    — Full skill instructions (165 lines)
```

## License

MIT
