# Orkestro — Schema Specification v0.2

> Vendor-agnostic multi-agent workflow orchestration.
> Define what happens. Your tool handles how.

---

## Philosophy

This schema defines **what** a workflow does — phases, agents, context flow, gates, retries. It says nothing about which LLM runs it or which coding tool executes it.

No API keys. No model config. No provider selection.

---

## 1. File Structure

```
your-repo/
├── .orkestro/
│   ├── executor.md              # How to run workflows (from orkestro repo)
│   ├── my-workflow.yaml         # Your workflow definition
│   └── another-workflow.yaml    # Multiple workflows supported
├── agents/                      # Agent definitions (role + skills)
│   ├── planner.md
│   ├── developer.md
│   └── reviewer.md
└── skills/                      # Knowledge files (markdown)
    ├── planning.md
    ├── development.md
    └── review.md
```

The executor is shared across all workflows. Each workflow references agents by name. Agents reference skills by path.

---

## 2. Workflow YAML — Top Level

```yaml
name: string                    # Required. Workflow identifier.
version: string                 # Required. Semver (e.g. "1.0.0").
description: string             # Optional. What this workflow does.

input: []                       # Optional. Parameters the workflow accepts.
phases: {}                      # Required. The workflow steps.
defaults: {}                    # Optional. Default settings for all phases.
safety: {}                      # Optional. Global constraints.
```

---

## 3. Input

Parameters passed to the workflow at invocation. Available in prompts via `{{ input.<name> }}`.

```yaml
input:
  - name: requirement
    type: string                # string | number | boolean
    required: true
    description: string         # Optional. Help text.
```

---

## 4. Phases

A phase is a single step. Phases form a directed acyclic graph via `depends_on`.

### Full field reference

```yaml
phases:
  <phase_name>:                 # Unique identifier (snake_case)

    description: string         # Optional. Human-readable summary.

    agent: string               # Optional. Agent name — resolved from agents/<name>.md
                                # Omit for orchestrator-handled phases (gates, summaries).

    prompt: string              # Optional. Template sent to the agent.
                                # Supports {{ }} interpolation.

    depends_on: [string]        # Optional. Phases that must complete first.

    context:                    # Optional. Data from a previous phase.
      from_phase: string        #   Source phase name.
      compaction: enum          #   none | summary | structured | custom
      compaction_prompt: string #   Required when compaction is 'custom'.

    gate:                       # Optional. Pause until condition is met.
      type: enum                #   human_approval | agent_verdict
      field: string             #   For agent_verdict: output field to check.
      actions:                  #   Outcome → action mapping.
        <outcome>: string       #     proceed | stop | goto:<phase>

    retry:                      # Optional. Retry policy.
      max: number               #   Maximum attempts.
      on: string                #   Trigger (e.g. "failure"). Default: any failure.
      on_exceeded: string       #   stop | goto:<phase>

    condition: string           # Optional. Phase runs only if true.

    parallel: boolean           # Optional. Default: false.
                                # When true, this phase runs concurrently with
                                # other parallel phases in the same wave.
                                # Phases are grouped into waves by depends_on:
                                #   Wave 1: no depends_on
                                #   Wave 2: depends only on Wave 1, etc.
                                # Within a wave:
                                #   - sequential phases (parallel: false) run first, one at a time
                                #   - parallel phases (parallel: true) then launch together
                                #   - all must complete before the next wave starts
                                # If the AI tool doesn't support concurrency,
                                # parallel phases run sequentially — safe fallback.

    on_complete: string         # Optional. Override next phase. goto:<phase>
```

---

## 5. Defaults

Applied to all phases unless overridden.

```yaml
defaults:
  timeout: string               # e.g. "300s", "5m"
  retry:
    max: number
    on_exceeded: string         # stop | goto:<phase>
```

---

## 6. Safety

Global constraints. Enforced before every command and file write.

```yaml
safety:
  blocked_commands:             # Substrings matched against any shell command.
    - "rm -rf"
    - "git push --force"
    - "git reset --hard"
  blocked_files:                # Glob patterns matched against file paths.
    - ".env"
    - "credentials.json"
    - "**/*.secret"
```




---

## 7. Agents

An agent is a markdown file in `agents/` that defines a role. The workflow references it by name.

```markdown
# Agent: Planner

You are a software implementation planner. Given a requirement,
you produce a sequenced work breakdown.

## Skills

- [Planning Skill](../skills/planning.md)

## Output

Your final message MUST contain a `<plan-output>` block...

## Rules

- One task = one file
- Do not write code, only plan
```

The executor loads the agent file and prepends it to the phase prompt. Skills referenced in the agent are also loaded.

**Agents are optional.** A phase without an `agent` field is executed by the orchestrator (the AI tool itself).

---

## 8. Skills

A skill is a markdown knowledge file. It contains domain expertise, conventions, checklists, or patterns. Skills are loaded by agents, not by the workflow directly.

Skills are vendor-agnostic by nature — they're text, not code. The same skill works in any AI tool.

---

## 9. Variable Interpolation

```
{{ input.<name> }}              → Workflow input parameter
{{ phases.<name>.output }}      → Captured output from a completed phase
```

Variables are resolved at runtime before the prompt is sent to the agent.

Unresolved variables (referencing missing inputs or incomplete phases) are left as-is in the prompt — the agent sees `{{ phases.plan.output }}` literally, which signals a wiring error.

---

## 10. Context Compaction

| Strategy | Behaviour |
|---|---|
| `none` | Pass raw output. Simple but token-heavy. |
| `summary` | Summarise into 5-10 bullet points. Uses the active session. |
| `structured` | Extract: decisions made, files changed, open questions. JSON format. |
| `custom` | Apply `compaction_prompt` to transform the output. |

The session does the compaction. The executor just asks.

---

## 11. Gate Types

### Human Approval

```yaml
gate:
  type: human_approval
  actions:
    approve: proceed
    reject: stop
    request_changes: goto:plan
```

The executor presents the phase output to the user and asks them to choose.

### Agent Verdict

```yaml
gate:
  type: agent_verdict
  field: verdict
  actions:
    APPROVE: proceed
    REQUEST_CHANGES: goto:fix
```

The executor extracts the `field` value from the agent's output (looks for XML tags like `<verdict>APPROVE</verdict>`) and follows the mapped action.

---

## 12. Execution Model

1. Load workflow YAML, validate required inputs
2. Group phases into **waves** based on `depends_on`:
   - **Wave 1**: phases with no `depends_on`
   - **Wave 2**: phases whose dependencies are all in Wave 1
   - **Wave N**: phases whose dependencies are all in waves < N
3. Execute wave by wave. Within each wave:
   - Run **sequential phases** (`parallel: false` or omitted) one at a time
   - Launch **parallel phases** (`parallel: true`) concurrently
   - Wait for ALL phases in the wave to complete before the next wave
4. For each phase: check condition → resolve context → build prompt → load agent → dispatch → capture output → evaluate gate → handle failure
5. Workflow completes when all waves are done or `stop` is triggered

```
Wave 1:  [plan]                         ← sequential
              |
Wave 2:  [setup_db]                     ← sequential (parallel: false)
         [implement_api, implement_ui]   ← concurrent (parallel: true)
              |
Wave 3:  [review]                       ← sequential
```

Gates on parallel phases are evaluated after all parallel phases in the wave complete. If the AI tool doesn't support concurrency, parallel phases run sequentially — safe fallback.

The executor does not call LLMs directly. It feeds prompts into the active session and captures responses.

---

## 13. What the Schema Does NOT Define

| Not defined | Why |
|---|---|
| Agent internals | Agents are markdown files — the schema just references them by name |
| Skill content | Skills are knowledge. Portable by nature. |
| Model/provider | Vendor-agnostic — the runtime decides |
| Session mechanics | How prompts are sent/captured depends on the tool |
| Domain logic | Build commands, test commands, file structures belong in skills |

---

## Changelog

| Version | Date | Changes |
|---|---|---|
| 0.2 | 2026-03-29 | Add agent and skill sections. Add context compaction reference. Add gate types detail. Add file structure convention. Tighten field reference. |
| 0.1 | 2026-03-29 | Initial draft. Core schema: phases, gates, retries, context, safety. |
