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
Use $your-prompt-engineer 帮我写个 prompt，让 agent 检查这个项目的优化点
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
Use $your-prompt-engineer 帮我写个 prompt，让 agent 检查这个项目的优化点
```

Prepare a worker task:

```text
Use $your-prompt-engineer 帮我把这个需求整理成 agent task：让 worker 修改 README，补充安装和启动步骤
```

Directly dispatch a read-only scout task:

```text
Use $your-prompt-engineer 直接发：让 explorer 只读检查这个项目的测试覆盖风险
```

Prepare a Claude Code style task prompt:

```text
Use $your-prompt-engineer 帮我把这个需求整理成 Claude Code task prompt：先只读检查项目结构和高风险技术债
```

## Confirmation Behavior

By default, the skill prepares a prompt and asks before dispatching.

When the host supports native interactive choices, it should prefer those choices.

When native choices are unavailable, it uses a text fallback:

```text
准备派发给：explorer
派发原因：需求需要先只读侦察项目上下文
风险等级：低
默认操作：发送

────────────────
1. 发送
   立即把上面的 prompt 发给 explorer

2. 修改
   说明你想改哪里，我会更新 prompt 后再次确认

3. 停止
   不发送，保留当前 prompt 供你手动使用

直接回车默认：发送
也可以回复：1 / 发送 / send / 送信
```

For safety-gated tasks, the default action changes to `Stop`.

## Target Resolution

The skill should not treat "this project" as valid unless the current workspace looks like a real project or the user provides a concrete target.

If the workspace is empty or projectless, it should ask for the project path before preparing or dispatching:

```text
你想让 agent 检查哪个项目？请提供项目路径、仓库、文件夹，或在目标项目 workspace 里重新打开会话。
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
