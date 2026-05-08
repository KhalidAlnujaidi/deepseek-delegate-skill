# Delegate — Strategic Task Delegation Skill for DeepSeek TUI

Plan the work, then ship the labor to `deepseek-v4-flash` sub-agents.

## What it does

The coordinating agent (you, typically running `deepseek-v4-pro`) handles reasoning, architecture, and design decisions. Sub-agents running `deepseek-v4-flash` (~10x cheaper per token) handle execution: scaffold generation, boilerplate, code writing, search/grep, test creation, and file edits.

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
