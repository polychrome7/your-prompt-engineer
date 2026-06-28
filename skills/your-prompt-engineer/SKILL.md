---
name: your-prompt-engineer
description: Use when the user invokes $your-prompt-engineer or asks to prepare, refine, structure, or send an agent prompt/task prompt for Codex, Claude Code, or another compatible agent host. Use for prompt handoff workflows before delegating to explorers, workers, tasks, or subagents. Do not trigger implicitly when the user asks only to run/delegate/inspect/fix with an agent and does not mention prompt preparation.
---

# Your Prompt Engineer

## Overview

Turn a user's rough request into a precise delegation prompt, select the right agent mode, show the prepared prompt to the user, and dispatch it after confirmation when the current host exposes a delegation mechanism.

Codex and Claude Code are first-class targets. Other compatible agent hosts should use the same prompt preparation, target validation, confirmation, and safety-gate behavior, then dispatch through native host tools when available. If no dispatch tool is available, provide the prepared prompt and clear handoff instructions instead of claiming the task was sent.

Support Chinese, English, and Japanese in user-facing communication. Choose the prompt language that best fits execution: usually English for technical/code tasks, and the project or audience language for content, localization, or domain-specific work. Replies intended for the user's customer should follow the project language; when unclear, follow the user's current language.

## Invocation

Do not require the user to type `$your-prompt-engineer`. Treat explicit skill invocation and natural-language intent as equivalent.

For reliable behavior across hosts, prefer explicit invocation:

```text
Use $your-prompt-engineer <your rough request>
```

Natural-language invocation may work when the host supports implicit personal skill discovery, but explicit invocation is recommended for predictable prompt preparation, confirmation, and dispatch behavior.

If the user combines prompt/requirement preparation language with agent delegation language, use this skill before any delegation tool or parallel-agent skill. Do not dispatch agents first and summarize later unless the user explicitly requested direct sending.

Trigger on requests such as:

- "Write an agent prompt for this request."
- "Turn this rough request into a worker task prompt."
- "Prepare a Claude Code task prompt for this bug report."
- "Refine this prompt and ask before sending it to an agent."
- "Create a read-only scout prompt for a subagent to inspect this project."
- "Send this prepared prompt to an explorer after confirmation."

Equivalent Chinese or Japanese requests should also trigger this skill when they clearly ask for prompt preparation or agent handoff. Keep public examples English-first; localize user-facing replies at runtime.

Do not trigger implicitly when the user asks to run, spawn, dispatch, delegate, inspect, fix, implement, or check something with an agent/subagent but does not mention prompt writing or prompt preparation. In that case, prefer the host's normal delegation behavior unless the user explicitly invokes `$your-prompt-engineer`.

## Workflow

1. Identify whether the user wants prompt preparation, agent delegation, or both.
2. Decide whether the request is clear enough to execute or needs scouting.
3. Extract the task intent, target object, output, constraints, acceptance signal, risk level, language, and best host adapter.
4. Resolve the task target before writing the prompt.
5. Choose confirm-first, direct-send, or scout-first mode.
6. Produce a prepared prompt using `references/prompt-templates.md`.
7. Show the prompt to the user and ask whether to send it unless direct-send mode applies. Prefer the host's native interactive choice UI when available.
8. After confirmation or explicit direct-send instruction, dispatch with the best available host adapter.
9. Report the agent type/tool used and the agent id or task handle when available.

Skip the confirmation step only when the user explicitly says to send directly, auto-dispatch, or not ask for confirmation.

## Dispatch Modes

- Use confirm-first mode by default: prepare the prompt, show it to the user, and ask whether to send.
- Use direct-send mode only when the user explicitly says "send directly", "no confirmation", "auto-dispatch", or equivalent localized phrasing.
- Use scout-first mode when the request is vague but can be investigated from current context. Prepare a scout prompt for an explorer/read-only task before preparing an execution prompt.

Always require confirmation before dispatching if the task may affect production systems, deployments, accounts, billing, credentials, external APIs, sensitive data, legal/compliance content, or broad multi-file changes, even if the user generally prefers automation.

## Intent Extraction

Before writing the final prompt, extract:

- Goal: the concrete outcome.
- Target: files, repo, product, document, text, page, issue, dataset, or system.
- Output: implementation, report, plan, document, prompt, test result, or other deliverable.
- Constraints: user-stated requirements, exclusions, style, platform, tools, time, budget, or scope.
- Acceptance: how the agent can know it is done.
- Risk: production, account, money, privacy, security, compliance, or large-scope impact.
- Language: user language, project language, customer language, and best execution language.
- Host: Codex, Claude Code, compatible agent host, or unavailable/unknown.
- Agent type: explorer, worker, default, or multiple independent agents.

If a field is missing but safely inferable, add it as an explicit assumption in the prompt. If the goal or target is missing and not inferable, ask one concise follow-up question.

## Target Resolution

Do not treat "project", "this project", "current project", or equivalent localized phrases as a valid target unless the current workspace appears to contain a real project or the user supplied a path, repository, file, issue, document, or other concrete object.

A workspace is probably a valid project when it contains project indicators such as:

- `.git/`
- `package.json`, `pnpm-lock.yaml`, `pyproject.toml`, `requirements.txt`, `go.mod`, `Cargo.toml`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`
- application source directories such as `src/`, `app/`, `pages/`, `lib/`, `server/`, or `tests/`
- clear product/document assets named by the user

A workspace is not enough by itself when it is a generated, empty, or projectless directory, especially when it only contains folders such as `work/`, `outputs/`, scratch files, or no recognizable project indicators.

When the target is missing, ask one concise question before producing a prompt:

```text
Which project should the agent inspect? Please provide a project path, repository, folder, or reopen the session in the target project workspace.
```

Ask the same question in the user's language. Do not produce a scout or worker prompt until the target is resolved.

## Information Sufficiency

A request is ready for an execution agent when it has:

- A clear goal: what should be done.
- A task object: the codebase, files, text, product, page, dataset, issue, or problem to work on.
- An expected output: code changes, analysis, prompt, plan, document, test result, list, or other deliverable.
- Constraints or preferences that the user stated.
- An acceptance signal: enough detail to judge whether the task is complete. If absent, define a short acceptance standard in the prompt.

Ask a concise follow-up only when the goal or task object is missing and cannot be inferred from the current context.

If the request is vague but has enough context to investigate, prepare a scout prompt for an explorer/research agent instead of asking many questions. The scout should inspect context, identify the concrete task, list missing information and risks, and recommend an executable worker prompt.

## Agent Selection

- Use `explorer` or a read-only task when the request is vague, needs repository/context discovery, or should not modify files yet.
- Use `worker` when the request is clear and requires code edits, file changes, implementation, or repair.
- Use `default` when the task is clear but mostly analytical, editorial, planning-oriented, or outside code ownership.
- Split into multiple agents only when tasks are independent and have distinct responsibilities.
- Do not delegate tiny tasks that the current agent can complete more directly unless the user explicitly wants delegation.

For worker prompts, always include ownership/scope and this instruction: "You are not alone in the codebase; do not revert edits made by others, and adapt to nearby changes."

## Dispatch Adapters

Use the current host's native delegation mechanism. Do not claim a dispatch happened unless a tool call or host action actually succeeded.

### Codex

When Codex exposes multi-agent tools, use them after confirmation:

- `multi_agent_v1.spawn_agent` for new explorer, worker, or default agents.
- `multi_agent_v1.send_input` for continuing a relevant existing agent.
- `multi_agent_v1.wait_agent` only when the next step is blocked on the result.
- `multi_agent_v1.close_agent` when an agent is no longer needed.

Do not override model, reasoning effort, or service tier unless the user asks or the task clearly requires it.

### Claude Code

When Claude Code exposes a Task/subagent mechanism, use it after confirmation:

- Use read-only/scout tasks for ambiguous requests.
- Use implementation tasks for clear execution work.
- Include file/module ownership for code-editing tasks.
- Ask the task to report changed files, verification, and unresolved risks.

### Compatible Agent Hosts

For other agent hosts, inspect the available tools and host conventions before dispatching:

- If a native agent/task/subagent/worker delegation tool is exposed, use it after confirmation.
- If the host supports interactive choices, use the same Send / Modify / Stop confirmation flow.
- If the host supports skill files but not automatic dispatch, present the prepared prompt and explain how to hand it off manually.
- If the host has a known prompt style, adapt formatting while preserving task, context, requirements, constraints, acceptance criteria, and output sections.

Do not claim first-class support for a host unless its delegation behavior is documented in this skill or verified in the current environment.

If the host does not expose an automatic dispatch tool, present the prepared prompt and state that this environment cannot auto-dispatch from the skill.

## Interactive Confirmation

When the current host supports native interactive choices, use them instead of plain text confirmation.

Offer exactly these choices for normal-risk tasks:

- Send (default): dispatch the prepared prompt.
- Modify: ask what to change, revise the prompt, then show confirmation again.
- Do not send: stop after providing the prompt.

For safety-gated tasks, do not make "Send" the default. Require an explicit send choice after explaining the safety reason.

When native interactive choices are unavailable, use text fallback:

```text
Preparing to dispatch to: <agent type>
Reason: <brief reason>
Risk: <low|medium|high and reason>
Default action: Send

----------------
1. Send
   Send the prepared prompt to <agent type>

2. Modify
   Tell me what to change; I will revise the prompt and confirm again

3. Stop
   Do not send; keep the prompt for manual use

Press Enter for default: Send
You can also reply: 1 / Send
```

Use the same language as the user for the confirmation choices at runtime while preserving the three actions: Send, Modify, and Stop.

Accept numeric and text replies:

- `1`, `send`, or localized equivalents: dispatch the prepared prompt.
- `2`, `modify`, or localized equivalents: ask what to change, revise the prompt, then show the confirmation choices again.
- `3`, `stop`, `do not send`, or localized equivalents: stop after providing the prompt.

For safety-gated tasks, use this fallback instead:

```text
This task may involve high-risk actions and requires explicit confirmation.

Preparing to dispatch to: <agent type>
Reason: <brief reason>
Risk: High, <safety reason>
Default action: Stop

----------------
1. Send
   Dispatch only after explicit confirmation

2. Modify
   Tell me what to change; I will revise the prompt and confirm again

3. Stop
   Do not send; keep the prompt for manual use

Press Enter for default: Stop
```

Do not treat empty input as Send when a safety gate applies.

## Safety Gates

Always pause for explicit confirmation before dispatch when the prepared task may:

- Modify production, deployment, CI/CD, infrastructure, billing, accounts, secrets, or access control.
- Call external APIs, send messages, publish content, or trigger irreversible actions.
- Process sensitive personal, financial, legal, medical, customer, or credential data.
- Make broad or ambiguous changes across many files.
- Require assumptions that could materially change the outcome.

When a safety gate applies, state the reason briefly and ask for confirmation after showing the prompt.

## User-Facing Format

For a clear execution task:

````markdown
I will send this to `worker` because the request is clear and requires implementation.

**Prepared Prompt**

```text
...
```

Send it?
````

For a vague task:

````markdown
I will first send this to `explorer` because the request needs context scouting.

**Scout Prompt**

```text
...
```

Send it?
````

If the user asks for changes, revise the prompt and show it again. If the user says not to send, stop after providing the prepared prompt.

## Reference

Read `references/prompt-templates.md` when preparing scout, worker, default, multi-agent, Codex-specific, Claude Code-specific, or compatible-host fallback prompts.
