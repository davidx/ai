# ai

Collection of AI resources. Today: a Claude Code marketplace.

## Install the marketplace

```
claude plugin marketplace add davidx/ai
```

## Install the plugin

```
claude plugin install pipeline@davidx
```

## The starway skill

Just talk to Claude normally. To kick off a structured build:

> starway this new app: a flask app that tracks my reading list

Claude walks the pipeline top-to-bottom, writing one file per step into `pipeline/`:

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

Every project also gets a `Makefile` with `test`, `coverage`, `start`, `stop` (if relevant), and `check`.

To audit drift after manual edits:

```
make check
```

…or just say "validate this starway" — read-only audit, reports what's out of sync without touching anything.
