# ai

A collection of AI tooling resources, organized by AI assistant.

## Layout

```
.
├── claude/    ← Claude Code marketplace (plugins, skills, workflows)
└── …          ← future: cursor/, windsurf/, gemini/, codex/, etc.
```

Each top-level directory targets one AI assistant and is self-contained — its own README, install steps, and conventions appropriate to that tool.

## What's here today

### [`claude/`](./claude) — Claude Code marketplace

A registered Claude Code marketplace exposing plugins for Anthropic's Claude Code CLI / IDE.

Quick install:

```
claude plugin marketplace add davidx/ai
claude plugin install pipeline@davidx
```

See [`claude/README.md`](./claude/README.md) for the full marketplace contents, plugin descriptions, and skill workflows.

## Repo conventions

- This repo is intentionally **public** so `claude plugin marketplace add davidx/ai` works without auth. All other repos default private.
- The Claude marketplace manifest lives at the repo root (`./.claude-plugin/marketplace.json`) because the `claude` CLI looks for it there. Plugin source paths point into `./claude/...`.
- New AI tool resources should sit alongside `claude/` at the top level (e.g. `cursor/`, `windsurf/`) — keep each tool's stuff isolated.
