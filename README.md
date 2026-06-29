# Your Prompt Engineer

Turn rough asks into agent-ready work.

Your Prompt Engineer is a prompt execution layer for Codex, Claude Code, and compatible agent hosts. It turns rough requests into structured prompts with target checks, execution routing, confirmation, and safety gates.

It is for people who think in natural language first, then want a safer, clearer way to execute the refined prompt in the current conversation or explicitly hand it off to an explorer, worker, or task agent.

## Before / After

Before:

```text
Check this project for optimization opportunities.
```

After:

```text
# Task

Inspect the current project and identify practical optimization opportunities. Do not modify files.

# Work Mode

This is a read-only scouting task. Identify concrete opportunities and prioritize them by impact and confidence.

# Output

Return:
- A concise summary of the project type and main technologies.
- The highest-value optimization opportunities.
- Affected files or areas for each opportunity.
- Estimated effort, risks, and suggested next steps.
- A recommended worker prompt for the top improvements.
```

The point is not just "write a prompt." The point is a repeatable execution flow: clear target, right mode, safe constraints, expected output, and confirmation before running it here or sending it to a subagent.

## What It Does

- Rewrites rough requests into structured execution prompts.
- Chooses an agent mode: `explorer`, `worker`, `default`, or multiple independent agents.
- Checks whether the task target is clear before preparing a prompt.
- Uses scout-first behavior for vague tasks.
- Shows the prepared prompt before execution by default.
- Executes in the current conversation by default when the user confirms with `1`.
- Sends to a subagent only when explicitly selected or requested.
- Treats text after explicit invocation as the raw request, even when it is fragmentary.
- Adds safety gates for production, billing, secrets, external APIs, sensitive data, and broad changes.
- Supports Chinese, English, and Japanese user-facing flows.

## Use Cases

- Turn vague feature ideas into implementation-ready worker tasks.
- Ask an explorer to inspect a codebase safely before any edits.
- Convert bug reports into structured investigation or repair prompts.
- Prepare Claude Code task prompts from rough notes.
- Split independent work into multiple agent tasks with non-overlapping ownership.
- Add confirmation and safety gates before executing or dispatching risky agent work.

## Why Not Just Ask the Model?

You can ask a model to write prompts. This skill is for repeatable prompt execution behavior.

It adds a consistent process around prompt handoff:

- target validation before writing a prompt
- scout-first routing for vague tasks
- explorer/worker/default agent selection
- first-class Codex and Claude Code prompt patterns
- graceful fallback for compatible agent hosts
- confirmation before current-session execution or dispatch
- safety gates for risky work

## Recommended Invocation

Explicit invocation is recommended for reliable behavior:

```text
Use $your-prompt-engineer Write a prompt for an agent to inspect this project for optimization opportunities.
```

You can also put the raw request directly after the skill name. It does not need to be organized or phrased as "write a prompt":

```text
$your-prompt-engineer make this image softer
```

If the host maps slash commands to skills, this style may also work:

```text
/your-prompt-engineer make the current image feel softer
```

Natural-language invocation may work in hosts that support implicit personal skill discovery, but it is not guaranteed. If the host does not load the skill implicitly, it may simply answer as a normal agent.

Codex and Claude Code are first-class targets. Other compatible hosts can still use the prepared prompt and confirmation workflow. The default confirmed action is current-session execution; automatic dispatch to a subagent depends on whether the host exposes a native delegation tool and the user explicitly chooses it.

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

Prepare a prompt and confirm before executing:

```text
Use $your-prompt-engineer Write a prompt for an agent to inspect this project for optimization opportunities.
```

Prepare a prompt from a fragmentary request:

```text
$your-prompt-engineer make this image softer
```

Prepare a worker task:

```text
Use $your-prompt-engineer Turn this request into an agent task: ask a worker to update the README with installation and startup instructions.
```

Directly dispatch a read-only scout task to a subagent:

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

See [docs/EXAMPLES.md](docs/EXAMPLES.md) for full example flows, including scout-first, worker, missing-target, safety-gated, fragmentary-payload, and explicit direct-send-to-subagent scenarios.

## Confirmation Behavior

By default, the skill prepares a prompt and asks what to do next. Replying `1` means "execute this prompt in the current conversation", not "send it to a subagent."

When the host supports native interactive choices, it should prefer those choices.

When native choices are unavailable, it uses a text fallback:

```text
Prepared mode: explorer
Reason: the request needs a read-only context scout before implementation
Risk: low
Default action: Execute here

----------------
1. Execute here
   Use the prepared prompt in this current conversation

2. Modify
   Tell me what to change; I will revise the prompt and confirm again

3. Send to subagent
   Dispatch the prepared prompt to explorer

4. Stop
   Do not execute or send; keep the prompt for manual use

Press Enter for default: Execute here
You can also reply: 1 / Execute here
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
- Automatic dispatch to a subagent depends on the host exposing tools such as Codex multi-agent tools, Claude Code task/subagent tools, or another native delegation mechanism.
- Natural-language implicit triggering is host-dependent. Explicit `$your-prompt-engineer` invocation is the reliable path.
- Slash-command invocation such as `/your-prompt-engineer` only works when the host maps slash commands to skills.
- Explicit invocation with no payload starts no guaranteed persistent mode. The skill should ask for the raw request or object to transform.
- If no dispatch tool is available, the skill can still execute the prepared prompt in the current conversation or output it for manual use.

## Roadmap

- Add more first-class host adapter guidance.
- Expand Claude Code task prompt examples.
- Add compact and verbose confirmation modes.
- Collect community prompt recipes for common workflows.
- Add more forward-test examples for empty workspace, scout-first, worker, and safety-gated scenarios.

## Contributing

Good contribution areas:

- `good first issue`: small copy, examples, docs, or test prompt improvements
- `host-adapter`: support or document another compatible host
- `prompt-template`: improve prompt templates for a repeatable workflow

Please keep the skill itself concise. Put detailed examples and host-specific notes in `references/` when they would make `SKILL.md` too large.

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
