---
name: stairway
description: Structured plan-then-execute workflow. Use when the user asks to build a feature, implement a task, refactor existing code, says "run stairway", or says "validate/check this stairway" (read-only audit mode). Enforces sequential steps (requirements → features → pseudotests → pseudocode → iterative failing tests → minimal code → coverage → refactor → recursive change layers → runnable script) with per-step documentation in pipeline/. Every follow-up requirement after the initial build (refactor, swap, add, remove) gets its own full mini-stairway under pipeline/change-XXX/.
---

# stairway

A disciplined build workflow. The aim is to stop jumping straight to code, and instead walk through a sequence that forces clarity, isolation, and verifiable outcomes.

XP-flavored flow: human intent → requirements → features → pseudotests (what to validate) → pseudocode (how to service those tests) → real failing tests → simplest passing implementation (stubs OK) → coverage → refactor → layer-by-layer change resolution (recursive mini-stairway per stub or follow-up requirement) → runnable script.

The pipeline is for *both* greenfield builds and follow-up changes. Every new requirement after the initial build — refactor, renderer swap, feature add, feature remove — is treated as a recursive change layer, not as ad-hoc patching.

## Rules

* Do the steps in order. Do not combine or skip.
* Every step that produces non-code output writes a file into `pipeline/`.
* Every step's output file ends with a sentinel line so the next step can verify the previous one ran:
  * Outer steps: `<!-- stairway: step N complete -->`
  * Recursive change layers: `<!-- stairway: change-NNN/<deeper-path> step N complete -->`
* Before starting step N, grep for step N-1's sentinel. If absent, go back and complete that step first. The directory and its sentinels are the overseer.
* After writing each step's file, also state aloud: "step N complete — on to step N+1".
* Top-level features go in top-level `features/` as one YAML file per feature (intent, not implementation).
* Tests and code go in their normal project locations.
* YAML is for expressing intent + validation criteria, not for encoding logic. If you find yourself inventing a DSL in YAML, stop — write code instead.
* **Files are not planned up front.** They emerge as the tests demand them in step 5. There is no "file plan" step. The shape of the code is a consequence of the tests, not a prerequisite.
* **Pseudotests come before pseudocode.** Pseudocode is written to service the intended outcomes described in the pseudotests. The test describes what must be true; the code grows from there.
* **No fenced code blocks in `03-pseudotests.md` or `04-pseudocode.md`.** These files must read as prose, not as runnable code. Pseudo-method names and dataclass-shaped sketches in inline backticks are fine; ` ```python ` blocks containing real syntax are not. If a fenced block makes the file feel implementable directly, it has gone too far.
* **"Is this enough?" test for pseudocode/pseudotests:** Could any coding agent / AI entity understand the intent of the implementation without asking follow-up questions? The description must be free-form, disambiguated, and as simple as possible to express the idea — no human-centric assumptions.
* **From step 6 onward, every step ends with `make test && make coverage`.** Tests must stay green; coverage must stay where step 7 left it (100% at that point) or improve. Paste the tail of the output into that step's file as evidence. If either fails, fix before moving on — never advance with a red bar.

## Directory layout

```
pipeline/
  01-requirements.md     # restated requirements + explicit outcome
  02-features.md         # list of features/*.yaml files created
  03-pseudotests.md      # language-agnostic test descriptions, feature-prefixed IDs
  04-pseudocode.md       # conceptual model + per-pseudotest code shape, with Services links
  05-tests.md            # iterative log: pseudotest → real failing test → files emerged → failure output
  06-implementation.md   # minimal code (stubs OK), with stub checklist at bottom
  07-coverage.md         # make coverage output, 100% expected
  08-refactor.md         # what changed, coverage maintained
  09-changes.md          # high-level summary of every recursive change layer (stubs + follow-ups)
  10-script.md           # path to runnable script + sample output

features/
  <feature-name>.yaml    # one per top-level feature (intent + outcome + validation)

# Every recursive change layer (step 9 stubs OR any follow-up requirement) gets its own
# full subdir mirroring the top layer's first 8 steps:
pipeline/change-001/
  01-requirements.md
  02-features.md          # internal capabilities only; promotes to top-level features/ only if user-visible
  03-pseudotests.md
  04-pseudocode.md
  05-tests.md
  06-implementation.md
  07-coverage.md
  08-refactor.md
pipeline/change-002/
  ... (same complete set)
```

## The outer steps

### 1. Requirements
Read the user's request back to them in structured form. State the precise outcome sought, not just the task. Break into clear bullets. If anything is ambiguous, stop and confirm before continuing.

Write `pipeline/01-requirements.md`. End with: `<!-- stairway: step 1 complete -->`

### 2. Features
Create the top-level `features/` directory if it doesn't exist.

A feature is **any major top-level or grouped capability of the system** — not just UI or user-visible behavior. "Parses PDFs", "stores results in a queryable form", "exposes an HTTP endpoint for ingestion" all count. Anything that meaningfully services the intent counts.

For each feature, write `features/<n>.yaml`:
- `intent`: what the feature is for (1–2 sentences)
- `outcome`: the observable result when it works
- `validation`: how to independently prove the outcome (programmatic check, not vibes)

Don't encode logic, branches, or configuration trees. If a feature needs a conditional, that belongs in code.

Write `pipeline/02-features.md` listing every YAML file with a one-line summary. End with: `<!-- stairway: step 2 complete -->`

### 3. Pseudotests
For each feature, describe what its tests will assert: setup state, action taken, observable result that proves it worked. Free form — given/when/then, prose, fragments, whatever fits. Audience is any coding agent. Test of "is this enough?": could another agent translate this into runnable test code without follow-up questions?

**Use feature-prefixed IDs** so coverage can be checked by eye and by `make check`. E.g., for a `list-notes` feature: `PT-LIST-1`, `PT-LIST-2`. For a `persistence-dir` feature: `PT-DIR-1`, `PT-DIR-2`.

Pseudotests come *before* pseudocode because they describe what must be true; the code in step 4 will be shaped to service them.

No fenced code blocks. Read as prose.

Write `pipeline/03-pseudotests.md`. Cover every feature. End with: `<!-- stairway: step 3 complete -->`

### 4. Pseudocode
Open with a brief **conceptual model** paragraph: what are the layers / major pieces, how do they relate, where do side effects live? This 100–300 word preamble is what makes the rest of the file (and the eventual code) coherent. Examples of useful layer splits:
- storage (pure I/O) / domain state (pure data + transitions) / renderer (driver, side effects)
- ingestion / normalization / query
- parsing / validation / persistence

Then for each pseudotest in step 3, describe the shape of code that services it — flow, data, decisions, enough to implement from. Free form, not constrained to functions/classes/methods or any language. The audience is any coding agent.

**Tag each pseudocode block with the pseudotests it services.** E.g., end the block describing `resolve_notes_dir` and `NoteStore.__init__` with `Services: PT-DIR-1, PT-DIR-2`. This makes the bidirectional link visible and lets `make check` verify coverage of pseudo-stuff trivially.

No fenced code blocks. The file should read as a layered explanation, not as Python wearing a costume.

Write `pipeline/04-pseudocode.md`. End with: `<!-- stairway: step 4 complete -->`

### 5. Iteratively write failing tests
Loop over the pseudotests from step 3, ordered from simplest to most complex. For each:

1. Translate it into a real failing test (express the pseudotest idea in actual test code — loose, individually focused, no rigid structure or interdependencies yet).
2. Create **only** the files the test forces into existence — typically the test file itself, plus the smallest possible production stub the test imports (an empty class, a function returning `None` or a constant). No more.
3. Run the test. It must fail — production code is empty or stubbed.
4. Append a short entry to `pipeline/05-tests.md`: which pseudotest, the test name, which files appeared this iteration, the tail of the failure output.

Do this as one complete pass for all pseudotests before moving on. The shape of the code emerges from the tests, not from a prescribed file list. Resist the urge to scaffold modules ahead of need.

Prefer real objects over mocks (user preference); mock only when necessary.

#### Makefile (emerges on the first iteration)
The Makefile is one of the files that emerges — the moment you go to run your first failing test. It's infrastructure with fixed required targets:

* `make test` — run the full test suite
* `make coverage` — run tests with coverage (100% expected at step 7)
* `make start` — run the app (long-running services) or the primary entrypoint script (CLIs / one-shots)
* `make stop` — stop the app, only if `make start` launches a long-running process. Omit for pure CLIs.
* `make check` — validate the pipeline itself (see below).

Targets call into the project's normal tooling (e.g. `.venv/bin/pytest`, `python -m <pkg>`), not duplicate logic.

`make check` shells out to claude — do not write a Python validator:

```makefile
check:
	@claude -p "validate this stairway"
```

When this step ends, every pseudotest has a failing test, and the project's file structure exists exactly to the extent the tests required it — nothing more.

End with: `<!-- stairway: step 5 complete -->`

### 6. Minimal implementation (stubs OK)
Stay at the **outer layer**. Write the simplest code that makes every test pass, leaning on the pseudocode for shape. **Stubs returning constants are correct here** — `return 1` instead of a real database lookup, `return []` instead of a real fetch, `return "ok"` instead of real validation logic.

Mark every stub in code with a `# STUB` comment on the line of the placeholder return. These markers are what step 9 will iterate over.

At the bottom of `pipeline/06-implementation.md`, include a checklist of every stub introduced — function/method, file, what it currently returns, what it should eventually do. This list is the input to step 9.

Run `make test && make coverage`. All tests must pass. Paste the tail into the file. End with: `<!-- stairway: step 6 complete -->`

### 7. Coverage
Run `make coverage`. Expect 100% at this stage (code is minimal and fully exercised by the tests). If not 100%, fix before continuing. Paste output into `pipeline/07-coverage.md`. End with: `<!-- stairway: step 7 complete -->`

### 8. Refactor
Structural cleanup only — **no functional change**. Look back at the pseudotests, pseudocode, and features for guidance on what shape the code wants to take. Tests stay green; coverage stays at 100%. Do not introduce new complexity, do not add features, do not touch stubs.

Run `make test && make coverage` at the end. Paste the tail into `pipeline/08-refactor.md` along with a description of what changed. End with: `<!-- stairway: step 8 complete -->`

### 9. Recursive change layers (stubs + follow-up requirements)
This step has two triggers:
- **Stubs introduced in step 6.** For each stub on the checklist, in order, peel it.
- **Any new requirement that arrives after step 8** — refactor request, renderer swap, new feature, removal of an existing feature. Any non-trivial follow-up is a change layer.

Each layer gets its own subdir `pipeline/change-NNN/` (NNN = zero-padded sequential: 001, 002, 003…), executing the **full mini-stairway** for that scope. Use the same artifact set and order as the top layer.

```
pipeline/change-NNN/
  01-requirements.md           # restate the change's outcome
  02-features.md               # internal capabilities driving this change
  03-pseudotests.md            # focused, language-agnostic
  04-pseudocode.md             # conceptual model + Services links, same rules
  05-tests.md                  # iterative log
  06-implementation.md         # real code for this scope (deeper stubs OK; they recurse)
  07-coverage.md               # global make coverage; 100% on touched code
  08-refactor.md               # cleanup; tests stay green
  <child-change-NNN>/          # recursion if step 06 introduces deeper stubs
    01-pseudotests.md ...
```

#### Internal capabilities vs top-level features
A change layer's `02-features.md` lists **internal capabilities** for that change. These do **not** automatically promote to top-level `features/*.yaml`. Only promote when the change introduces a genuinely new user-visible capability that didn't exist before. Otherwise the layer's features stay scoped to the layer subdir.

Open the layer's `02-features.md` with: *"Internal capabilities for change-NNN. Promotes to top-level features/ only if user-visible."*

#### Backward compatibility / migration belongs in the layer
If a change requires migrating existing data, files, or schemas, treat the migration as its own pseudotested feature inside the layer. At minimum, two pseudotests:
- `PT-MIG-1` — the migration happens (legacy artifact gets transformed to new shape).
- `PT-MIG-2` — idempotency (re-running the migration on already-migrated state is a no-op).

#### Stub recursion specifics
For stub layers (the "discharge a `# STUB`" case), step 6 of the layer replaces the parent stub with real code. Remove the `# STUB` marker. If the real implementation needs deeper stubs, mark them and they become the next recursion level.

#### Layer completion gate
Before declaring step 9 complete, run:

```sh
grep -rn '# STUB' .
```

If anything matches, the un-discharged stubs must either be peeled (new layer) or explicitly justified at the bottom of `pipeline/09-changes.md` with a one-line reason each. No silent leftovers.

Write `pipeline/09-changes.md` listing every layer (`change-NNN`), the requirement that triggered it, and the resolution. End with: `<!-- stairway: step 9 complete -->`

### 10. Runnable script
Write the simplest possible script that uses the code end-to-end and prints real output — proof of life. Optionally add an integration test that exercises the full flow with zero mocking.

Run `make test && make coverage` one last time. Write `pipeline/10-script.md` with the script path and a sample of its output. End with: `<!-- stairway: step 10 complete -->`

## The recursive inner pipeline

Every change layer at `pipeline/change-NNN/` (or, for nested layers, at `pipeline/change-NNN/<child>/` and so on) follows steps 1–8 of the outer pipeline with the same rules — including the no-fenced-code rule, the conceptual model preamble in pseudocode, the feature-prefixed PT-IDs, the bidirectional Services links, and the sentinel lines.

A layer is complete when:
- All 8 step files exist with sentinels.
- Any deeper stubs introduced in step 6 have their own nested layers, recursively.
- `make test && make coverage` is green at the end of the layer.

## Enforcement

Before starting any step, grep for the previous step's sentinel:

- Outer step N: `grep -l "stairway: step $((N-1)) complete" pipeline/`
- Recursive step N at layer L: `grep -l "stairway: change-$L step $((N-1)) complete" pipeline/change-$L/`

If the sentinel is missing, refuse to proceed and go back. The sentinels and the directory tree together are the overseer.

When the user says "continue stairway" or interrupts and resumes, walk `pipeline/` to find the deepest layer and highest step number with a sentinel — that's where to resume from.

## Validation mode

Trigger phrases: "validate this stairway", "check this stairway", "audit the pipeline", or `make check` (which runs `claude -p "validate this stairway"`).

In this mode you **do not modify anything**. You read, you report. The user decides whether to fix.

Walk the repo and check:

1. All ten outer `pipeline/NN-*.md` files exist (`01-requirements.md` … `10-script.md`), each ending in its sentinel.
2. Every requirement bullet in `01-requirements.md` maps to at least one `features/*.yaml`.
3. Every `features/*.yaml` has `intent`, `outcome`, and `validation` fields.
4. Every feature yaml is referenced in `02-features.md`.
5. `03-pseudotests.md` lists at least one pseudotest for every feature, with feature-prefixed IDs.
6. `04-pseudocode.md` opens with a conceptual model paragraph, covers every pseudotest from step 3, and tags each block with `Services: PT-...` links.
7. Neither `03-pseudotests.md` nor `04-pseudocode.md` contains fenced code blocks.
8. Every real test logged in `05-tests.md` traces back to a pseudotest in `03-pseudotests.md`, and exists on disk.
9. `Makefile` defines `test`, `coverage`, `start`, `check`, and `stop` (if `start` is long-running).
10. The stub checklist at the bottom of `06-implementation.md` accounts for every `# STUB` marker that existed at that step.
11. `pipeline/change-NNN/` (if any) contains a subdir for every discharged stub from step 6's checklist AND every follow-up requirement processed after step 8. Each subdir has files 01-08 with sentinels; nested layers for any deeper stubs introduced.
12. `grep -rn '# STUB' .` over source returns nothing **unless** `09-changes.md` lists each remaining stub with an explicit one-line justification.
13. The runnable script in `10-script.md` exists and is executable.
14. Every sentinel listed in this skill is present where expected. Missing sentinels are the most direct signal of a half-finished step.

Output: a per-step checklist (✓ / ✗ with one line of evidence each), a tree of `change-NNN/` showing per-layer status, and at the end a punch list of what's missing or out of sync. Do not edit files.

End with: "run stairway from step N to fix" (or "from layer X step N to fix" for recursive holes) if anything failed, or "pipeline is in sync" if all green.
