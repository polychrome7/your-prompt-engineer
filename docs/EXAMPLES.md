# Examples

These examples show the intended behavior of Your Prompt Engineer. The exact wording may vary by host, but the shape should stay consistent: target check, agent routing, prepared prompt, confirmation, and safety handling.

## 1. Vague Request -> Scout Prompt

User:

```text
Use $your-prompt-engineer Write a prompt for an agent to inspect this project for optimization opportunities.
```

Expected behavior:

- Detect that the request needs context scouting before implementation.
- Choose `explorer`.
- Keep the task read-only.
- Ask for confirmation before dispatch.

Example response:

````text
I will send this to `explorer` because the request needs a read-only project scout before any implementation work.

**Scout Prompt**

```text
# Task

Inspect the current project and identify practical optimization opportunities. Do not modify files.

# Context To Inspect

Use the current workspace as the target project. Inspect repository structure, configuration, dependencies, tests, build scripts, source organization, performance-sensitive areas, developer experience, maintainability, and obvious runtime risks.

# Work Mode

This is a read-only scouting task. Identify concrete optimization opportunities and prioritize them by impact and confidence.

# Output

Return:
- A concise summary of the project type and main technologies.
- The highest-value optimization opportunities, grouped by category when helpful.
- For each opportunity: affected files/areas, why it matters, estimated effort, and suggested next step.
- Any risks, missing context, or assumptions.
- A recommended executable worker prompt for the top one or two improvements.

# Constraints

Do not modify files. Do not run destructive commands. Do not invent requirements that are not supported by the repository; label assumptions clearly.
```

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
````

## 2. Clear Implementation Request -> Worker Prompt

User:

```text
Use $your-prompt-engineer Turn this request into an agent task: ask a worker to update the README with installation and startup instructions.
```

Expected behavior:

- Detect that the task is clear and requires file edits.
- Choose `worker`.
- Include file ownership and verification expectations.
- Ask for confirmation before dispatch.

Example response:

````text
I will send this to `worker` because the request is clear and requires documentation edits.

**Prepared Prompt**

```text
# Task

Update the project README with clear installation and startup instructions.

# Context

Use the current workspace as the target project. Inspect package/config files and existing README content before editing.

# Work Mode

This is an execution task. Edit files directly within the assigned scope.

You are not alone in the codebase; do not revert edits made by others, and adapt to nearby changes.

# Ownership

You are responsible for:
- README.md

Avoid changing:
- source files
- dependency lockfiles
- unrelated documentation

# Requirements

- Add installation steps.
- Add local startup steps.
- Mention required environment variables only if they are discoverable in the repo.
- Keep instructions concise and accurate.

# Acceptance Criteria

- README includes installation and startup sections.
- Commands match the project tooling.
- No unrelated files are changed.

# Output

In your final response, include:
- Files changed.
- What changed.
- Verification performed and results.
- Any remaining risks or follow-up needed.
```

Preparing to dispatch to: worker
Reason: the request is clear and requires README edits
Risk: low
Default action: Send

────────────────
1. Send
2. Modify
3. Stop
````

## 3. Missing Target -> Ask For Path

User in an empty or projectless workspace:

```text
Use $your-prompt-engineer Write a prompt for an agent to inspect the project for optimization opportunities.
```

Expected behavior:

- Do not assume the generated workspace is a real project.
- Do not prepare a scout or worker prompt yet.
- Ask for the missing target.

Example response:

```text
Which project should the agent inspect? Please provide a project path, repository, folder, or reopen the session in the target project workspace.
```

## 4. Safety-Gated Task -> Default To Stop

User:

```text
Use $your-prompt-engineer Write a prompt for a worker to update the production deployment workflow and rotate API keys.
```

Expected behavior:

- Detect production/deployment/credential risk.
- Prepare a careful prompt only if the target is clear.
- Require explicit confirmation.
- Do not make Send the default action.

Example confirmation:

````text
This task may involve high-risk actions and requires explicit confirmation.

Preparing to dispatch to: worker
Reason: the request touches production deployment and credentials
Risk: High, production workflow and API key handling
Default action: Stop

────────────────
1. Send
   Dispatch only after explicit confirmation

2. Modify
   Tell me what to change; I will revise the prompt and confirm again

3. Stop
   Do not send; keep the prompt for manual use

Press Enter for default: Stop
````

## 5. Direct Send -> Dispatch Without Confirmation

User:

```text
Use $your-prompt-engineer Send directly: ask an explorer to inspect this project for test coverage risks without modifying files.
```

Expected behavior:

- Detect explicit direct-send intent.
- Choose `explorer`.
- Dispatch without the confirmation panel when no safety gate applies.
- Report the agent/tool used and handle when available.

Example post-dispatch response:

```text
Sent to `explorer` using Codex multi-agent tools. Handle: <agent id>
```
