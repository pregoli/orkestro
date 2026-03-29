# Contributing to Orkestro

## Ways to Contribute

### Add an example workflow
If you've built a workflow for your team, add it to `examples/`. Include a comment at the top explaining what it does and which agents it uses.

### Add a starter skill
Generic skills that work across projects are welcome in `skills/`. Domain-specific skills belong in your own repo, not here.

### Improve the executor
The executor (`executor.md`) is the core of orkestro. Improvements to clarity, edge case handling, or new capabilities are valuable. Test your changes by running a workflow in at least two different AI tools.

### Improve the schema spec
The schema spec (`docs/schema-spec.md`) should be complete enough that someone can build their own executor from it. If you find a gap, fill it.

## Guidelines

- **Keep it vendor-agnostic** — no references to specific AI providers, models, or tools in the core files (executor, schema, starter agents/skills). Examples may reference specific tools for illustration.
- **Keep it simple** — orkestro's value is its simplicity. A new feature must justify the complexity it adds.
- **Markdown only** — the executor, agents, and skills are markdown files. No code dependencies.
- **Test in multiple tools** — if you change the executor, verify it works in at least two AI tools (e.g. Claude Code and Copilot).

## Structure

```
orkestro/
├── executor.md                 # Core — how to run workflows
├── workflow-template.yaml      # Starting template
├── agents/                     # Starter agent definitions
├── skills/                     # Starter skill files
├── examples/                   # Example workflows
└── docs/
    └── schema-spec.md          # Full schema reference
```

## Pull Request Process

1. Fork the repo
2. Create a branch (`feature/my-contribution`)
3. Make your changes
4. Ensure all example workflows are still valid YAML
5. Open a PR with a clear description of what and why
