# claude — Claude Code marketplace

Plugins, skills, and workflows for [Claude Code](https://www.anthropic.com/claude-code).

> The marketplace manifest itself lives at the repo root in `../.claude-plugin/marketplace.json` (a Claude Code requirement). This directory holds the plugin source.

## Install the marketplace

```
claude plugin marketplace add davidx/ai
```

## Install a plugin

```
claude plugin install <plugin>@davidx
```

To pick up upstream changes:

```
claude plugin marketplace update davidx
```

To remove (also clears the local clone):

```
claude plugin marketplace remove davidx
```

## Plugins

### `pipeline`

A disciplined build workflow exposed as the **`starway`** skill. Start an interactive Claude Code session and say:

> starway this new app: a flask app that tracks my reading list

Claude walks an 11-step pipeline, writing one markdown artifact per step into `pipeline/`:

```
  1. requirements      ── pipeline/01-requirements.md
  2. features          ── pipeline/02-features.md  +  features/*.yaml
  3. pseudocode        ── pipeline/03-pseudocode.md
  4. pseudotests       ── pipeline/04-pseudotests.md
  5. file plan         ── pipeline/05-file-plan.md
  6. failing tests     ── pipeline/06-tests.md
  7. minimal code      ── pipeline/07-implementation.md
  8. coverage (100%)   ── pipeline/08-coverage.md
  9. refactor          ── pipeline/09-refactor.md
 10. real providers    ── pipeline/10-providers.md
 11. runnable script   ── pipeline/11-script.md
```

From step 7 onward, every step ends with `make test && make coverage`. Tests stay green, coverage stays where step 8 left it.

Every project gets a `Makefile` with:

| target          | purpose                                            |
|-----------------|----------------------------------------------------|
| `make test`     | run the test suite                                 |
| `make coverage` | run tests with coverage (100% expected at step 8)  |
| `make start`    | run the app or primary entrypoint                  |
| `make stop`     | stop the app (only if `start` is long-running)     |
| `make check`    | read-only pipeline audit (delegates to claude)     |

#### Audit a project

If work was done outside the pipeline (manual edits, partial runs, copy-paste), audit drift with:

```
make check
```

…or just say "validate this starway" inside an interactive Claude session. It's read-only — reports what's out of sync, never edits.

## Layout

```
claude/
├── README.md
└── pipeline/                       ← plugin
    ├── .claude-plugin/plugin.json
    └── skills/
        └── starway/
            └── SKILL.md            ← the skill content claude loads
```
