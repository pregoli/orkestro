# Agents

An agent is a role definition — a markdown file that describes who the agent is, what skills it uses, what output format it produces, and what rules it follows.

## Starter Agents

| Agent | Role | Output |
|---|---|---|
| [planner.md](planner.md) | Creates sequenced implementation plans | `<plan-output>` XML |
| [developer.md](developer.md) | Implements code, runs builds and tests | `<dev-output>` XML |
| [reviewer.md](reviewer.md) | Reviews diffs for correctness and coverage | `<review-output>` XML |

## Creating Your Own Agent

Create a markdown file in this directory:

```markdown
# Agent: My Agent

You are a [role description].

## Skills

- [Skill Name](../skills/my-skill.md)

## Output

Your final message MUST contain a `<my-output>` block:
(define your output contract)

## Rules

- (constraints and behavioural rules)
```

Then reference it in your workflow:

```yaml
phases:
  my_phase:
    agent: my-agent    # matches agents/my-agent.md
```

## How Agents Work

1. The executor reads `agents/<name>.md`
2. It loads any skills listed in the agent file
3. It prepends the agent definition + skills to the phase prompt
4. The AI tool receives the full context and acts as that agent

Agents don't contain domain knowledge — that lives in skills. Agents define the role, the output contract, and the rules. This separation means you can swap skills without changing the agent, or reuse an agent across different projects with different skills.
