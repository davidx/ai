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

Once installed, just talk to Claude normally. To kick off a structured build, say something like:

> starway this new app: a flask app that tracks my reading list

Claude will walk through requirements → features → pseudocode → failing tests → minimal code → coverage → refactor → real providers → runnable script, dropping a file per step into `pipeline/`.
