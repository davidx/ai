---
name: starway
description: Structured plan-then-execute workflow. Use when the user asks to build a feature, implement a task, or says "run starway". Enforces sequential steps (requirements → features → pseudocode → failing tests → minimal code → coverage → refactor → real providers → runnable script) with per-step documentation in pipeline/.
---

# starway

A disciplined build workflow. The aim is to stop jumping straight to code, and instead walk through a sequence that forces clarity, isolation, and verifiable outcomes.

## Rules

- Do the steps **in order**. Do not combine or skip.
- Every step that produces non-code output writes a file into `pipeline/`.
- Features go in top-level `features/` as one YAML file per feature (intent, not implementation).
- Tests and code go in their normal project locations.
- Before starting step N, confirm step N-1's file exists in `pipeline/`. If not, go back.
- At the end of each step, state: "step N complete — on to step N+1" so progress is explicit.
- YAML is for expressing intent + validation criteria, not for encoding logic. If you find yourself inventing a DSL in YAML, stop — write code instead.

## Directory layout

```
pipeline/
  01-requirements.md     # restated requirements + explicit outcome
  02-features.md         # list of features/*.yaml files created
  03-pseudocode.md       # language-agnostic design
  04-file-plan.md        # files to be created/modified
  05-tests.md            # summary of failing tests written
  06-implementation.md   # summary of minimal code to pass tests
  07-coverage.md         # make coverage output, 100% expected
  08-refactor.md         # what changed, coverage maintained
  09-providers.md        # real data providers replacing stubs
  10-script.md           # path to runnable script + sample output
features/
  <feature-name>.yaml    # one per feature (intent + validation)
```

## The steps

### 1. Requirements
Read the user's request back to them in structured form. State the **precise outcome** sought, not just the task. Write `pipeline/01-requirements.md`. Stop and confirm before continuing if anything is ambiguous.

### 2. Features
Create top-level `features/` directory. For each distinct feature, write `features/<name>.yaml` capturing:
- `intent:` what the feature is for (1-2 sentences)
- `outcome:` the observable result when it works
- `validation:` how to independently prove the outcome (programmatic check, not vibes)

Do **not** encode logic, branches, or configuration trees. If a feature needs a conditional, that belongs in code. Write `pipeline/02-features.md` listing the yaml files.

### 3. Pseudocode
Language-agnostic description of the functionality. Methods, inputs, outputs, flow. Write `pipeline/03-pseudocode.md`.

### 4. File plan
List the files to be created or modified with a one-line purpose each. Write `pipeline/04-file-plan.md`.

A `Makefile` is **always** in this list. It must define:
- `make test` — run the full test suite
- `make coverage` — run tests with coverage (100% expected at step 7)
- `make start` — run the app (long-running services) or the primary entrypoint script (CLIs / one-shots)
- `make stop` — stop the app, only if `make start` launches a long-running process. Omit for pure CLIs.

Targets should call into the project's normal tooling (e.g. `.venv/bin/pytest`, `python -m <pkg>`), not duplicate logic.

### 5. Failing tests
Write unit tests that exercise the expected outcome per feature, in the most isolated way. Prefer real objects over mocks (user preference); mock only when necessary. Run the tests — they must fail (code does not exist). Write `pipeline/05-tests.md` summarizing test files and what each asserts.

### 6. Minimal implementation
Write the least code needed to make the tests pass. Stubs returning constants (`return 1`) are acceptable here — we are validating interfaces and architecture, not real data. Write `pipeline/06-implementation.md`.

### 7. Coverage
Run `make coverage`. Expect 100% at this stage (code is minimal). If not 100%, fix before continuing. Save output to `pipeline/07-coverage.md`.

### 8. Refactor
Clean up code and tests while keeping tests green and coverage high. Coverage priority lowers practically as complexity grows, but aim to hold it. Write `pipeline/08-refactor.md` describing what changed.

### 9. Real providers
Replace stubbed returns with real data providers. Tests should still pass. Write `pipeline/09-providers.md`.

### 10. Runnable script
Write the simplest possible script that uses the code and prints real output — proof of life. Optionally add an integration test that exercises the full flow with zero mocking. Write `pipeline/10-script.md` with the script path and a sample of its output.

## Enforcement

Before writing any code in a given step, check `pipeline/` for the previous step's file. If missing, go back and complete that step first. This is non-negotiable — the directory itself is the overseer.

When the user says "continue pipeline" or interrupts and resumes, read `pipeline/` to determine the next step.
