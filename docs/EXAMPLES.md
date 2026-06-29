# Examples

These examples show the intended behavior of Your Prompt Engineer. The exact wording may vary by host, but the shape should stay consistent: target check, mode routing, prepared prompt, confirmation, and safety handling. Replying `1` executes the prepared prompt in the current conversation. Sending to another agent is a separate explicit choice and depends on whether the host exposes a native delegation tool.

## 1. Vague Request -> Scout Prompt

User:

```text
Use $your-prompt-engineer Write a prompt for an agent to inspect this project for optimization opportunities.
```

Expected behavior:

- Detect that the request needs context scouting before implementation.
- Choose `explorer`.
- Keep the task read-only.
- Ask for confirmation before execution or dispatch.

Example response:

````text
I prepared this as `explorer` because the request needs a read-only project scout before any implementation work.

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

Prepared mode: explorer
Reason: the request needs a read-only context scout before implementation
Risk: low
Default action: Execute here

----------------
1. Execute here
   Use the prepared prompt in this current conversation

2. Modify
   Tell me what to change; I will revise the prompt and confirm again

3. Send to agent
   Dispatch the prepared prompt to explorer

4. Stop
   Do not execute or send; keep the prompt for manual use

Press Enter for default: Execute here
You can also reply: 1 / Execute here
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
- Ask for confirmation before execution or dispatch.

Example response:

````text
I prepared this as `worker` because the request is clear and requires documentation edits.

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

Prepared mode: worker
Reason: the request is clear and requires README edits
Risk: low
Default action: Execute here

----------------
1. Execute here
2. Modify
3. Send to agent
4. Stop
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
- Do not make Execute here or Send to agent the default action.

Example confirmation:

````text
This task may involve high-risk actions and requires explicit confirmation.

Prepared mode: worker
Reason: the request touches production deployment and credentials
Risk: High, production workflow and API key handling
Default action: Stop

----------------
1. Execute here
   Execute in this current conversation only after explicit confirmation

2. Modify
   Tell me what to change; I will revise the prompt and confirm again

3. Send to agent
   Dispatch to worker only after explicit confirmation

4. Stop
   Do not execute or send; keep the prompt for manual use

Press Enter for default: Stop
````

## 5. Direct Send To Agent -> Dispatch Without Confirmation

User:

```text
Use $your-prompt-engineer Send directly: ask an explorer to inspect this project for test coverage risks without modifying files.
```

Expected behavior:

- Detect explicit direct-send-to-agent intent.
- Choose `explorer`.
- Dispatch without the confirmation panel when no safety gate applies.
- Report the agent/tool used and handle when available.

Example post-dispatch response:

```text
Sent to `explorer` using Codex multi-agent tools. Handle: <agent id>
```

## 6. Fragmentary Invocation Payload -> Structured Prompt

User:

```text
$your-prompt-engineer make this image softer
```

Expected behavior:

- Treat the text after `$your-prompt-engineer` as the raw request.
- Do not require the user to say "write a prompt."
- Use the visible or attached image as the target when available.
- Ask one concise question if the image or target is not available.
- Prepare a prompt that turns the fragmentary request into concrete visual direction.

Example response:

````text
I will prepare this for `default` because the request is a visual refinement task.

**Prepared Prompt**

```text
# Task

Refine the provided image so it feels softer.

# User Request

"make this image softer"

# Context

Use the currently attached or visible image as the target. Interpret "softer" as a calmer, gentler visual direction unless the user provided more specific brand or style constraints.

# Requirements

- Reduce harsh contrast and overly sharp visual edges where appropriate.
- Prefer warmer or more balanced tones if the current image feels cold or severe.
- Preserve the main subject, layout, and intent of the image.
- Avoid making the result blurry, washed out, or less legible.
- If editing is not available in this host, return concrete edit instructions instead.

# Output

Return the edited image if the host supports image editing. Otherwise, return a concise set of image-editing instructions that can be used in the available image tool.
```

Prepared mode: default
Reason: the task is clear but depends on the attached or visible image context
Risk: low
Default action: Execute here

----------------
1. Execute here
2. Modify
3. Send to agent
4. Stop
````
