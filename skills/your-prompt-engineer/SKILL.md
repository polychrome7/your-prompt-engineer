---
name: your-prompt-engineer
description: Use when the user invokes $your-prompt-engineer or asks to prepare, refine, structure, or send an agent prompt/task prompt for Codex, Claude Code, or another compatible agent host. Use for prompt handoff workflows before delegating to explorers, workers, tasks, or subagents. Do not trigger implicitly when the user asks only to run/delegate/inspect/fix with an agent and does not mention prompt preparation.
---

# Your Prompt Engineer

## Overview

Your Prompt Engineer turns rough requests into clear, dispatch-ready prompts for agent work. It is designed for Codex, Claude Code, and compatible agent hosts that support task, worker, explorer, or subagent delegation.

Use it to prepare the prompt first, choose the right agent mode, validate the task target, show the prepared prompt, and dispatch only when the user confirms or explicitly asks for direct sending.

The skill supports Chinese, English, and Japanese conversations. Use the user's language for replies, but choose the best execution language for the agent prompt: usually English for technical work and the project or audience language for content work.

## When To Use

Use this skill when the user explicitly invokes `$your-prompt-engineer` or asks for an agent-ready prompt, for example:

- "Write an agent prompt for this request."
- "Turn this rough request into a worker task prompt."
- "Prepare a Claude Code task prompt for this bug report."
- "Refine this prompt and ask before sending it to an agent."
- "Create a read-only scout prompt for a subagent to inspect this project."
- "Send this prepared prompt to an explorer after confirmation."

Equivalent Chinese or Japanese requests should also trigger this skill when they clearly ask for prompt preparation or agent handoff.

Do not trigger implicitly when the user asks only to run, spawn, delegate, inspect, fix, implement, or check something with an agent/subagent and does not mention prompt writing or prompt preparation. In that case, use the host's normal delegation behavior unless the user explicitly invokes this skill.

## Core Workflow

1. Decide whether the user wants prompt preparation, agent delegation, or both.
2. Extract the goal, target, expected output, constraints, acceptance signal, risk level, language, host, and agent type.
3. Resolve the target before writing the prompt.
4. Choose one mode: confirm-first, direct-send, or scout-first.
5. Draft the prompt with `references/prompt-templates.md`.
6. Show the prepared prompt and ask whether to send it unless direct-send mode applies.
7. Dispatch through the current host's native agent/task mechanism when available.
8. Report the agent type/tool used and the agent id or task handle when available.

## Target Resolution

Do not treat "project", "this project", "current project", or equivalent localized phrases as a valid target unless the current workspace appears to contain a real project or the user supplied a path, repository, file, issue, document, or other concrete object.

A workspace is probably valid when it contains project indicators such as `.git/`, `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `src/`, `app/`, `lib/`, or `tests/`.

A workspace is not enough when it is empty, generated, projectless, or only contains scratch folders such as `work/` or `outputs/`.

When the target is missing, ask one concise question in the user's language before producing a prompt:

```text
Which project should the agent inspect? Please provide a project path, repository, folder, or reopen the session in the target project workspace.
```

## Dispatch Modes

- Confirm-first: default. Prepare the prompt, show it, and ask whether to send.
- Direct-send: use only when the user explicitly says "send directly", "no confirmation", "auto-dispatch", or equivalent localized phrasing.
- Scout-first: use when the request is vague but current context can be investigated safely. Prepare a read-only explorer prompt before preparing an execution prompt.

Always require explicit confirmation before dispatching tasks that may affect production systems, deployments, accounts, billing, credentials, external APIs, sensitive data, legal/compliance content, or broad multi-file changes.

## Agent Selection

- Use `explorer` for vague, read-only, context-discovery, or risk-scoping tasks.
- Use `worker` for clear implementation, file changes, repair, or execution work.
- Use `default` for clear analysis, planning, editorial, or non-code tasks.
- Split into multiple agents only when the tasks are independent and have distinct responsibilities.
- Do not delegate tiny tasks that the current agent can complete more directly unless the user explicitly wants delegation.

For worker prompts, include ownership/scope and this instruction: "You are not alone in the codebase; do not revert edits made by others, and adapt to nearby changes."

## Confirmation

When native interactive choices are available, use them. Offer exactly:

- Send (default for normal-risk tasks)
- Modify
- Do not send

When native choices are unavailable, use a text fallback with the same three actions. For safety-gated tasks, make Stop the default and require an explicit send choice.

Accept numeric replies and localized equivalents for:

- `1` / Send
- `2` / Modify
- `3` / Stop

## Host Behavior

Use the current host's native delegation mechanism. Do not claim a dispatch happened unless a tool call or host action actually succeeded.

- Codex: use available multi-agent tools after confirmation.
- Claude Code: use available Task/subagent mechanisms after confirmation.
- Compatible hosts: inspect available tools and use native task, worker, explorer, or subagent dispatch when available.
- No dispatch tool: provide the prepared prompt and clear handoff instructions instead of claiming it was sent.

Do not override model, reasoning effort, or service tier unless the user asks or the task clearly requires it.

## Reference

Read `references/prompt-templates.md` before drafting scout, worker, default, multi-agent, Codex-specific, Claude Code-specific, or compatible-host fallback prompts.
