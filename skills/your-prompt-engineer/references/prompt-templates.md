# Prompt Templates

Use these templates as starting points. Keep the final prompt specific to the user's actual request and current host.

## Prompt Preparation Checklist

Before drafting, fill these fields mentally or in the prompt when useful:

- Raw payload:
- Goal:
- Target:
- Output:
- Constraints:
- Acceptance:
- Risk level:
- Language:
- Host adapter:
- Agent type:
- Assumptions:

Include assumptions only when they help the delegated agent avoid ambiguity.

When the raw payload is fragmentary, preserve the user's intent and make the missing structure explicit. For example, "make this image softer" can become a prompt that identifies the image target, defines "softer" as visual direction, asks the agent to inspect the available image/context, and requests concrete edits or recommendations.

## Scout / Explorer Prompt

```text
# Task

Investigate the user's rough request and the current context. Do not modify files.

# User Request

<quote or summarize the user's original request>

# Context To Inspect

<workspace, files, repo, product, text, issue, or other relevant objects>

# Work Mode

This is a scouting task. Identify the concrete task, relevant files or artifacts, missing information, risks, and the best next action.

# Output

Return:
- A concise interpretation of what the user likely wants.
- Relevant files, modules, systems, or artifacts.
- Missing information that blocks execution, if any.
- Risks or constraints.
- A recommended executable prompt for a worker/default agent.

# Constraints

Do not make code or file changes. Do not invent requirements that are not supported by the context; label assumptions clearly.
```

## Worker Prompt

```text
# Task

<clear implementation or file-change task>

# User Request

<quote or summarize the user's original request>

# Context

<repo/workspace details, relevant files, prior findings, constraints, target behavior>

# Work Mode

This is an execution task. Edit files directly within the assigned scope.

You are not alone in the codebase; do not revert edits made by others, and adapt to nearby changes.

# Ownership

You are responsible for:
- <files, directories, modules, or behavior area>

Avoid changing:
- <unrelated files, generated files, live config, or sensitive areas>

# Requirements

- <requirement 1>
- <requirement 2>
- <requirement 3>

# Acceptance Criteria

- <how to know the task is done>
- <tests or checks expected>

# Output

In your final response, include:
- Files changed.
- What changed.
- Verification performed and results.
- Any remaining risks or follow-up needed.
```

## Default / Analysis Prompt

```text
# Task

<clear non-editing, writing, planning, analysis, or prompt-generation task>

# User Request

<quote or summarize the user's original request>

# Context

<relevant background, examples, files, project language, audience>

# Requirements

- <requirement 1>
- <requirement 2>
- <requirement 3>

# Output

Return <specific format>. Keep assumptions explicit and separate from facts.
```

## Multi-Agent Split Prompt

Use only when tasks are independent.

```text
Split this request into independent agent tasks. For each task, provide:
- Agent type: explorer, worker, or default.
- Responsibility and write scope, if any.
- Prompt.
- Expected output.
- Dependencies, if any.

Avoid overlapping file ownership between worker agents.
```

## Codex Dispatch Notes

Prefer Markdown task blocks for Codex agents:

```text
# Task

<one concrete task>

# Context

<current workspace, relevant files, original request, known constraints>

# Agent Role

Use <explorer|worker|default>.

# Instructions

<host-specific execution instructions>

# Output

<exact final response requirements>
```

For Codex workers, include file ownership and the instruction not to revert others' edits.

## Claude Code Dispatch Notes

Prefer compact, structured prompts for Claude Code tasks. XML-like tags are acceptable when they improve clarity:

```text
<task>
<one concrete task>
</task>

<context>
<original request, repo/files, constraints>
</context>

<mode>
<scout|implementation|analysis>
</mode>

<requirements>
- <requirement>
- <requirement>
</requirements>

<output>
- Changed files, if any
- Verification performed
- Risks or blockers
</output>
```

For Claude Code implementation tasks, specify ownership and remind the task not to overwrite unrelated edits.

## Compatible Host Fallback Notes

Use these notes for hosts without a first-class adapter in this skill:

```text
# Task

<one concrete delegated task>

# Context

<target, user request, relevant files or artifacts, assumptions>

# Agent Mode

Use <explorer|worker|default>.

# Requirements

- <requirement>
- <requirement>

# Constraints

- <read-only, ownership, safety, language, or scope constraints>

# Acceptance Criteria

- <how the host agent should know it is done>

# Output

Return <specific final format>.
```

If the host has no native dispatch tool, show this prompt and say:

```text
This host did not expose an automatic delegation tool. Use the prepared prompt above with your agent/task workflow.
```

## Confirmation Copy

Use native host choices when available. For normal-risk tasks, make "Send" the default choice; for safety-gated tasks, require explicit send confirmation and do not default to sending.

Use short confirmation copy when native choices are unavailable:

````text
I will send this to `<agent type>` because <reason>.

**Prepared Prompt**

```text
<prompt>
```

Send it?
````

Text fallback options:

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

Localize these labels to the user's language when useful, but preserve the same three actions and defaults: Send, Modify, Stop.

Safety-gated fallback:

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

For direct-send mode, report after dispatch instead:

```text
Sent to `<agent type>` using <host adapter>. Handle: <id if available>.
```
