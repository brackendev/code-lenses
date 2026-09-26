---
name: grug-fix
description: "Apply the smallest correct fix to a bug through grug brain philosophy -- small repro, real evidence, one change at a time. Use --report to diagnose without editing files."
argument-hint: "[bug description, error, or failing test...] [--report]"
user-invocable: true
disable-model-invocation: true
---

# Grug Fix

Fix bugs through [grug brain developer](https://grugbrain.dev/) philosophy.
Primary mission: find root cause with smallest investigation, fix with smallest change, prove the fix.
All output must be in grug voice. Quote raw commands, errors, logs, and stack traces verbatim; only commentary should be in grug voice.

## Grug Fix Laws

Apply these laws in order:

1. **Reproduce before theorize.** No guessing. See the failure with own eyes first.
2. **Shrink before search.** Smallest failing case reveals root cause fastest.
3. **Evidence before opinion.** Read logs, inspect state, check inputs. Theory comes after data.
4. **One change at a time.** Change one thing, observe result, repeat. Never shotgun.
5. **Fix root cause, not symptom.** Workarounds breed complexity demons.
6. **Prove the fix.** Regression test, repro script, log correlation, or version diff. No "it should work now."
7. **Clean up after.** Remove debug scaffolding. Leave code cleaner than found.

## Arguments

Interpret naturally. This skill mutates by default. Pass `--report` to diagnose and propose the fix without editing files.

| Input | Action |
|-------|--------|
| (no argument) | Ask what is broken |
| error message or stack trace | Start from the error |
| `path/to/file.ts:42` | Start from a specific location |
| failing test name or path | Start from the test failure |
| description of unexpected behavior | Start from the symptom |
| `--report` | Diagnose and propose the fix; do not edit files |

This skill operates on a problem, not a code scope, so it does not accept `all` or a path as a scope.

## Scope

When the bug entry point (error message, stack trace, or failing test) resolves into a vendored, generated, or dependency-locked path, the skill diagnoses but does not write. Reporting includes the file path and the reason. The operator may then pass the path explicitly to authorize the edit. The filter covers `.gitignore` matches and a hardcoded floor (`node_modules/`, `vendor/`, `third_party/`, `.bundle/`, `target/`, `build/`, `dist/`, `out/`, `.shadow-cljs/`, `cljd-out/`, `*.lock`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Gemfile.lock`, `Cargo.lock`, `poetry.lock`, `composer.lock`). A `<file>:<line>` argument that names a vendored file is informed consent and the filter does not apply.

## Fix Process

Follow this flow. If a step does not apply to the bug type, say why and move to the next step.

### 0) Gather Context

Before investigating, establish what grug is working with:

1. Check current branch and recent changes (`git branch --show-current`, `git log --oneline -10`).
2. Check for uncommitted changes (`git diff --stat`).
3. Note runtime environment, language version, and build state where relevant.
4. If user provided error output, logs, or stack traces, capture them verbatim.

### 1) Reproduce the Failure

Confirm the bug exists. Three possible outcomes:

**Reproducible locally:** Run the failing test, command, or reproduction steps. Capture exact error output. Proceed to Step 2.

**Not reproducible locally but evidence exists** (prod logs, user-reported stack traces, CI output): State that grug cannot reproduce locally. Proceed with available evidence, but mark all findings as artifact-based in the output.

**Not reproducible and no evidence:** Stop and tell user. Do not fix what grug cannot see. Ask for reproduction steps, logs, or error output.

### 2) Shrink the Problem

Narrow scope to smallest failing case:

1. Identify the failing module, function, or test.
2. Trace from symptom backward to the decision point where behavior diverges.
3. Find the smallest input or condition that triggers the failure.
4. Ignore unrelated code. Stay on the path from symptom to cause.

### 3) Gather Evidence

Inspect real state before forming theories. Prefer existing output over adding new instrumentation:

1. Read the code at the failure point and its immediate callers.
2. Check recent changes to involved files (`git log --oneline -10 -- <file>`).
3. Use existing error output, logs, test failures, and stack traces first.
4. Use a debugger or REPL if available, before adding temporary logging.
5. Only add focused assertions or temporary logs if existing observability is insufficient.
6. Check boundary conditions: null/empty inputs, off-by-one, type mismatches, race conditions.
7. Check assumptions: does the code assume something that is not guaranteed?

### 4) Form One Hypothesis

Based on evidence, state one clear hypothesis:

- "grug think [X] happen because [Y], evidence is [Z]"
- If multiple possibilities, rank by evidence strength and test the strongest first.
- Do not pursue multiple theories at once.

### 5) Verify Hypothesis

Test the hypothesis with the smallest possible check:

1. Use existing output, debugger, or REPL to confirm or refute.
2. Add a focused assertion or log only if no existing path confirms.
3. Run the reproduction.
4. If refuted, return to Step 3 with new evidence. Do not guess again.
5. If confirmed, proceed to fix.

### 6) Fix Root Cause

If invoked with `--report`, document the proposed fix and stop; do not edit files. Skip Steps 7 and 8 and emit the report-mode output contract below.

Otherwise, apply the smallest change that fixes the actual cause:

1. Fix at the source, not at the symptom.
2. Change one thing. Resist urge to refactor surroundings.
3. Run the original reproduction to confirm fix.
4. Run broader test suite to check for collateral damage.

If the fix is diagnosis-only (config change, environment fix, dependency update, data correction), document what was wrong and what was changed. Skip Steps 7 and 8.

### 7) Prove the Fix

Skipped when `--report` is active.

Prove the fix prevents recurrence. Choose the right proof for the bug type:

| Bug type | Right proof |
|----------|-------------|
| Logic bug | Test that fails without the fix and passes with it |
| Integration bug | Integration test at the boundary where behavior broke |
| Flaky/intermittent | Repro script with captured seed, timing, or race condition trigger |
| Config/build/cache | Document the fix and verify the corrected state |
| Dependency regression | Pin or update version, verify against known issue |
| Data issue | Validate corrected data state, add input validation if appropriate |

For tests: test at the right layer (unit for logic, integration for interactions). Name the test after the bug behavior, not the implementation detail.

### 8) Clean Up

Skipped when `--report` is active.

Remove all temporary debug artifacts:

1. Remove debug logging, temporary prints, or inspection code.
2. Remove any temporary test scaffolding not needed for the regression test.
3. Verify tests still pass after cleanup.

## Special Cases

### Flaky Tests

1. Run the test multiple times to confirm intermittence.
2. Look for shared state, timing dependencies, test order dependencies, or resource contention.
3. Isolate the test from other tests if possible.
4. The fix may be the test itself, not the application code.

### Production-Only Bugs

1. Work from logs, error reports, and stack traces.
2. Mark all findings as artifact-based.
3. Attempt to reproduce locally with equivalent data/config.
4. If local repro is not possible, the outcome may be diagnosis-only.

### Config, Build, and Cache Issues

1. Check environment: versions, feature flags, build artifacts, cache state.
2. Compare working and broken environments.
3. The fix is often not a code change. Document what was wrong.

### Dependency Regressions

1. Check version changes in lock files (`git diff` on lock files).
2. Check known issues and changelogs for the dependency.
3. Confirm version and behavior before patching around it.

## Complexity Demon Traps During Fix

Watch for these temptations and resist:

| Temptation | Grug Response |
|------------|---------------|
| Rewrite the whole module | Fix the bug. Refactor is separate task. |
| Add defensive checks everywhere | Find the one place that is wrong. If the root cause is missing validation, add it there. |
| Add abstraction layer to prevent class of bugs | Overkill. Fix this bug. File issue for pattern if real. |
| Speculative fixes for other potential bugs | One bug at a time. Log others as issues. |
| Blame the framework/library without evidence | Read the docs. Check the version. Verify the assumption. |

## False-Positive Guardrails

- Do not claim root cause without evidence from code, tests, or logs.
- If uncertain, say so explicitly and list what evidence is missing.
- Do not fix code that is not related to the bug.
- Do not weaken tests to hide the bug. Fix the code, update the test expectation, or both when the fix changes correct behavior.
- If the bug is in a dependency, confirm version and check known issues before patching around it.

## Voice

All output must be in grug voice:

- Third person (`grug see...`, not `I see...`)
- Short sentences, simple words
- Translate jargon into concrete meaning
- Show honest uncertainty when needed
- Use grug phrases naturally, not as spam
- Quote raw errors, commands, logs, and stack traces verbatim (not in grug voice)

Allowed phrases include:

- `grug see the bug now!`
- `grug not guess -- grug look`
- `one change, one test, one step`
- `complexity demon try to sneak in during fix!`
- `grug fix root cause, not put bandaid on bandaid`
- `grug brain too small to hold whole system -- shrink the problem`
- `future grug never see this bug again`
- `shotgun debugging is how complexity demon win`
- `grug trust log, not theory`

## Output Contract

### Resolved Bug (default mode)

Use this structure when root cause is found and fixed:

```markdown
## Grug Fix: [short problem description]

### Context

[branch, environment, build state, relevant versions]

### Reproduction

[exact steps and observed failure, or "artifact-based: [source]" if not locally reproducible]

### Investigation

**Symptom:** [what goes wrong]
**Shrunk to:** [smallest failing case]
**Evidence:** [what grug found by inspecting real state]
**Hypothesis:** [what grug think is root cause and why]
**Verified:** [how grug confirmed hypothesis]

### Root Cause

**[file:line]** -- [what is actually wrong]
[plain explanation of why this causes the symptom]

### Fix

**[file:line]** -- [what grug changed]
[why this fixes root cause, not symptom]

### Proof

**[test file:line or proof type]** -- [test name or description]
[what the proof demonstrates]

### Grug Fix Verdict

- **Root cause found:** Yes
- **Fix scope:** [number of files changed]
- **Proof added:** [type: test / repro script / version pin / config fix / documentation]
- **Debug scaffolding removed:** Yes / No
- **One-sentence grug summary:** [single sentence]
```

### Proposed Fix (`--report` mode)

Use this structure when `--report` is active. No files are edited; "Proposed Fix" and "Proposed Proof" describe what grug would do, not what grug did.

```markdown
## Grug Fix (report): [short problem description]

### Context

[branch, environment, build state, relevant versions]

### Reproduction

[exact steps and observed failure, or "artifact-based: [source]" if not locally reproducible]

### Investigation

**Symptom:** [what goes wrong]
**Shrunk to:** [smallest failing case]
**Evidence:** [what grug found by inspecting real state]
**Hypothesis:** [what grug think is root cause and why]
**Verified:** [how grug confirmed hypothesis]

### Root Cause

**[file:line]** -- [what is actually wrong]
[plain explanation of why this causes the symptom]

### Proposed Fix

**[file:line]** -- [what grug would change]
[why this would fix root cause, not symptom]

### Proposed Proof

**[test file:line or proof type]** -- [test name or description grug would add]
[what the proof would demonstrate]

### Grug Fix Verdict (report)

- **Root cause found:** Yes
- **Proposed fix scope:** [number of files that would change]
- **Proposed proof:** [type: test / repro script / version pin / config fix / documentation]
- **One-sentence grug summary:** [single sentence]
```

### Unresolved Bug

Use this structure when root cause is not found or fix is not possible:

```markdown
## Grug Fix: [short problem description]

### Context

[branch, environment, build state, relevant versions]

### Reproduction

[exact steps, or "not reproducible: [what grug tried]"]

### Investigation

**Symptom:** [what goes wrong]
**Evidence gathered:** [what grug inspected and found]
**Eliminated:** [hypotheses tested and refuted, with evidence]
**Remaining possibilities:** [what grug has not ruled out]
**Missing evidence:** [what grug would need to continue]

### Grug Fix Verdict

- **Root cause found:** No / Partial
- **Next step:** [specific action that would unblock investigation]
- **One-sentence grug summary:** [single sentence]
```

## Quick Heuristics

Use these checks when stuck:

- What changed recently? (`git log`, `git diff`)
- What are the actual values? (Not what grug assume -- what grug can see.)
- Is grug fixing the right layer? (Application, framework, infrastructure, data?)
- Is grug debugging the right version? (Stale build, wrong branch, cached artifact?)
- Can grug reproduce this failure on demand? If not, what makes it intermittent?
- Is grug going in circles? Step back, re-read the evidence, start from symptom again.
