# Your Prompt Engineer

Your Prompt Engineer turns rough requests into dispatch-ready prompts for Codex and Claude Code agents.

It is for people who think in natural language first, then want a safer, clearer handoff to an explorer, worker, or task agent.

## What It Does

- Rewrites rough requests into structured agent prompts.
- Chooses an agent mode: `explorer`, `worker`, `default`, or multiple independent agents.
- Checks whether the task target is clear before preparing a prompt.
- Uses scout-first behavior for vague tasks.
- Shows the prepared prompt before dispatch by default.
- Supports direct-send mode when explicitly requested.
- Adds safety gates for production, billing, secrets, external APIs, sensitive data, and broad changes.
- Supports Chinese, English, and Japanese user-facing flows.

## Recommended Invocation

Explicit invocation is recommended for reliable behavior:

```text
Use $your-prompt-engineer Write a prompt for an agent to inspect this project for optimization opportunities.
```

Natural-language invocation may work in hosts that support implicit personal skill discovery, but it is not guaranteed. If the host does not load the skill implicitly, it may simply answer as a normal agent.

## Installation

Copy the skill folder into your agent skills directory:

```bash
mkdir -p ~/.agents/skills
cp -R skills/your-prompt-engineer ~/.agents/skills/
```

For Codex-only usage, you may also install it into:

```bash
mkdir -p ~/.codex/skills
cp -R skills/your-prompt-engineer ~/.codex/skills/
```

Restart or reload your agent host if it does not discover new skills automatically.

## Usage Examples

Prepare a prompt and confirm before sending:

```text
Use $your-prompt-engineer Write a prompt for an agent to inspect this project for optimization opportunities.
```

Prepare a worker task:

```text
Use $your-prompt-engineer Turn this request into an agent task: ask a worker to update the README with installation and startup instructions.
```

Directly dispatch a read-only scout task:

```text
Use $your-prompt-engineer Send directly: ask an explorer to inspect this project for test coverage risks without modifying files.
```

Prepare a Claude Code style task prompt:

```text
Use $your-prompt-engineer Turn this request into a Claude Code task prompt: inspect the project structure and identify high-risk technical debt without changing files.
```

Multilingual examples:

```text
Use $your-prompt-engineer 帮我写个 prompt，让 agent 检查这个项目的优化点
```

```text
Use $your-prompt-engineer agent に渡すプロンプトを書いて：このプロジェクトの改善点を読み取り専用で調査する
```

## Confirmation Behavior

By default, the skill prepares a prompt and asks before dispatching.

When the host supports native interactive choices, it should prefer those choices.

When native choices are unavailable, it uses a text fallback:

```text
Preparing to dispatch to: explorer
Reason: the request needs a read-only context scout before implementation
Risk: low
Default action: Send

────────────────
1. Send
   Send the prepared prompt to explorer

2. Modify
   Tell me what to change; I will revise the prompt and confirm again

3. Stop
   Do not send; keep the prompt for manual use

Press Enter for default: Send
You can also reply: 1 / Send
```

For safety-gated tasks, the default action changes to `Stop`.

## Target Resolution

The skill should not treat "this project" as valid unless the current workspace looks like a real project or the user provides a concrete target.

If the workspace is empty or projectless, it should ask for the project path before preparing or dispatching:

```text
Which project should the agent inspect? Please provide a project path, repository, folder, or reopen the session in the target project workspace.
```

## Limitations

- The skill cannot create UI controls by itself. It can only ask the host to use native choices when the host supports them.
- Automatic dispatch depends on the host exposing tools such as Codex multi-agent tools or Claude Code task/subagent tools.
- Natural-language implicit triggering is host-dependent. Explicit `$your-prompt-engineer` invocation is the reliable path.
- If no dispatch tool is available, the skill should output the prepared prompt and explain that automatic dispatch is unavailable in the current host.

## Repository Layout

```text
skills/
└── your-prompt-engineer/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    └── references/
        └── prompt-templates.md
```

## Validation

If you have the Codex `skill-creator` system skill available, validate with:

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/your-prompt-engineer
```

## License

MIT
