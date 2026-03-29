# Skills

A skill is a markdown knowledge file. It contains domain expertise, conventions, checklists, or patterns that an agent reads before doing its work.

## Starter Skills

| Skill | Purpose |
|---|---|
| [planning.md](planning.md) | Task sequencing, risk assessment, dependency tracking |
| [development.md](development.md) | Code implementation, build verification, quality standards |
| [review.md](review.md) | Code review checklist, severity levels, verdict rules |

## Creating Your Own Skill

Create a markdown file in this directory. A skill is just text — guidelines, examples, checklists, patterns. Example:

```markdown
# My Domain Skill

## Conventions
- Use camelCase for variables
- Every function must have error handling
- Database queries must use parameterised queries

## Patterns
(describe the patterns your codebase follows)

## Anti-Patterns
(describe what to avoid)
```

Then reference it from an agent:

```markdown
## Skills
- [My Domain Skill](../skills/my-domain-skill.md)
```

## How Skills Work

Skills are loaded by agents, not by workflows. The chain is:

```
workflow.yaml → references agent by name
    agent.md → lists skills to load
        skill.md → knowledge injected into the agent's context
```

## Why Markdown?

Skills are markdown because:
- They work in any AI tool — just text injected into the prompt
- They're version-controlled and diffable
- They're human-readable — a developer can read a skill and learn from it
- They're composable — an agent can reference multiple skills
- They're shareable — copy a skill from one project to another

## Domain-Specific Skills

The starter skills are generic. For real projects, you'll create domain-specific skills that capture your team's conventions. Examples:

- `dotnet-cqrs.md` — CQRS patterns with Paramore Brighter
- `react-components.md` — React component conventions
- `terraform-modules.md` — Infrastructure-as-code standards
- `api-design.md` — REST API design guidelines

The more specific your skills, the better your agents perform.
