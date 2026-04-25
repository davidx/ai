---
name: starway
description: Structured plan-then-execute workflow. Use when the user asks to build a feature, implement a task, says "run starway", or says "validate/check this starway" (read-only audit mode). Enforces sequential steps (requirements → features → pseudocode → failing tests → minimal code → coverage → refactor → real providers → runnable script) with per-step documentation in pipeline/.
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
- **From step 7 onward, every step ends with `make test && make coverage`.** Tests must stay green and coverage must stay where step 8 left it. Paste the tail of the output into that step's `pipeline/` file as evidence. If either fails, fix before moving on — do not advance with a red bar.

## Directory layout

```
pipeline/
  01-requirements.md     # restated requirements + explicit outcome
  02-features.md         # list of features/*.yaml files created
  03-pseudocode.md       # language-agnostic design of the implementation
  04-pseudotests.md      # what each test will assert, before code is written
  05-file-plan.md        # files to be created/modified
  06-tests.md            # summary of failing tests written
  07-implementation.md   # summary of minimal code to pass tests
  08-coverage.md         # make coverage output, 100% expected
  09-refactor.md         # what changed, coverage maintained
  10-providers.md        # real data providers replacing stubs
  11-script.md           # path to runnable script + sample output
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
A freeform logical description of how the thing works — the flow, the data, the decisions, enough to implement from. Not constrained to functions/classes/methods or any particular programming language or paradigm. The agent picks the form (prose, indented flow, branching outline, whatever conveys the logic clearly). The test of "is this enough?" is: could a competent engineer turn this into code without asking follow-up questions? Write `pipeline/03-pseudocode.md`.

### 4. Pseudotests
Same spirit as step 3, applied to the tests. A freeform description of what's being tested: the setup state, the action taken, and the observable result that proves it worked. No prescribed shape — `given/when/then`, narrative paragraphs, or whatever fits. The point is that step 6 becomes a translation, not a design exercise. Write `pipeline/04-pseudotests.md`.

### 5. File plan
List the files to be created or modified with a one-line purpose each. Write `pipeline/05-file-plan.md`.

A `Makefile` is **always** in this list. It must define:
- `make test` — run the full test suite
- `make coverage` — run tests with coverage (100% expected at step 8)
- `make start` — run the app (long-running services) or the primary entrypoint script (CLIs / one-shots)
- `make stop` — stop the app, only if `make start` launches a long-running process. Omit for pure CLIs.
- `make check` — validate the pipeline itself (see below).

Targets should call into the project's normal tooling (e.g. `.venv/bin/pytest`, `python -m <pkg>`), not duplicate logic.

#### `make check` — pipeline self-audit (claude does it)

Do **not** write a Python validator. `make check` just shells out to claude and asks it to audit the pipeline:

```makefile
check:
	@claude -p "validate this starway"
```

The starway skill (this file) handles the rest — see "Validation mode" below.

### 6. Failing tests
Translate the pseudotests into real test code, one-for-one. Prefer real objects over mocks (user preference); mock only when necessary. Run the tests — they must fail (code does not exist). Write `pipeline/06-tests.md` summarizing test files and what each asserts.

### 7. Minimal implementation
Write the least code needed to make the tests pass. Stubs returning constants (`return 1`) are acceptable here — we are validating interfaces and architecture, not real data. Write `pipeline/07-implementation.md`.

### 8. Coverage
Run `make coverage`. Expect 100% at this stage (code is minimal). If not 100%, fix before continuing. Save output to `pipeline/08-coverage.md`.

### 9. Refactor
Clean up code and tests while keeping tests green and coverage high. Coverage priority lowers practically as complexity grows, but aim to hold it. Write `pipeline/09-refactor.md` describing what changed.

### 10. Real providers
Replace stubbed returns with real data providers. Tests should still pass. Write `pipeline/10-providers.md`.

### 11. Runnable script
Write the simplest possible script that uses the code and prints real output — proof of life. Optionally add an integration test that exercises the full flow with zero mocking. Write `pipeline/11-script.md` with the script path and a sample of its output.

## Enforcement

Before writing any code in a given step, check `pipeline/` for the previous step's file. If missing, go back and complete that step first. This is non-negotiable — the directory itself is the overseer.

When the user says "continue pipeline" or interrupts and resumes, read `pipeline/` to determine the next step.

## Validation mode

Trigger phrases: "validate this starway", "check this starway", "audit the pipeline", or `make check` (which runs `claude -p "validate this starway"`).

In this mode you **do not modify anything**. You read, you report. The user decides whether to fix.

Walk the repo and check:

1. All eleven `pipeline/NN-*.md` files exist (`01-requirements.md` … `11-script.md`).
2. Every requirement bullet in `01-requirements.md` maps to at least one `features/*.yaml`.
3. Every `features/*.yaml` has `intent`, `outcome`, and `validation` fields.
4. Every feature yaml is referenced in `02-features.md`.
5. `03-pseudocode.md` covers every feature by name.
6. `04-pseudotests.md` lists at least one test bullet for every feature, and each real test in `06-tests.md` traces back to a pseudotest bullet.
7. Every path in `05-file-plan.md` exists on disk (or is annotated as deleted/skipped).
8. Test files referenced in `06-tests.md` exist and cover each feature.
9. `Makefile` defines `test`, `coverage`, `start`, `check`, and `stop` if `start` is long-running.
10. No stub markers left in source (grep for `# stub`, `TODO: real impl`, `return 1  # placeholder`, etc.) given that step 10 claims providers are real.
11. The runnable script in `11-script.md` exists and is executable.

Output: a per-step checklist (✓ / ✗ with one line of evidence each) and at the end a punch list of what's missing or out of sync. Do not edit files. End with: "run starway from step N to fix" if anything failed, or "pipeline is in sync" if all green.
