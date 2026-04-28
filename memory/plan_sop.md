# Plan Mode SOP

**Trigger**: 3+ steps with dependencies / multi-file coordination / conditional branches / need for parallelism.  
**Disabled**: simple tasks with 1–2 steps should be executed directly.

Before starting the task, you must create a working directory `./plan_XXX/` (`XXX` = short English name for the task).

Use a dedicated

```python
code_run({'inline_eval': True, 'script': 'handler.enter_plan_mode("./plan_XXX/plan.md")'})
```

to enter plan mode.

---

## I. Exploration Phase (Pre-planning, mandatory)

⛔ **Hard rules (read before doing):**

- **The main agent is forbidden from performing environment probing directly** (must delegate to a subagent, no exceptions).
- The main agent only: creates directories, reads the SOP index, starts subagents, and reads conclusions.
- The exploration subagent is read-only: it must not modify any files or perform side-effectful operations.
- **If the exploration subagent fails to start**: diagnose the cause → retry, up to 2 times. The main agent must not fall back to probing on its own.

**Goal**: Before writing any plan, clarify three things:
1. Current environment (what exists, what is missing)
2. Available SOPs
3. Key uncertainties

**Why subagents are mandatory**: The main agent’s context is the scarcest resource; long exploration output would crowd out space for planning and execution.

### Step 1: Create directory (mandatory) + SOP matching + set plan flag (main agent)

1. Create working directory `mkdir plan_XXX/`.
2. Read `sop_index.md` and match applicable domain SOPs.
3. Update checkpoint: `[Task] XXX | [Need] One-line description | [Constraints] Key limitations | [Matched SOPs] ... | [Progress] Exploration phase`.

### Step 2: Start exploration subagent (watchdog mode)

Start the exploration subagent as described in `subagent.md`, with `--verbose` enabled for watchdog mode. Input should specify:

- **Task**: Probe environment information and write to `plan_XXX/exploration_findings.md`.
- **Probe items** (choose based on task type, not all of them):
  - Code tasks → key file structure, dependencies, entry points.
  - Browser tasks → current target page state, interactive elements.
  - Automation tasks → environment checks (which/pip/path/permissions).
  - Data tasks → sampled data (first 5 lines + last 5 lines + total size).
- **Output format**: `## Environment Status` / `## Key Findings` / `## Risks & Uncertainties`.
- **Constraints**: read-only probing, no file modifications, ≤ 10 tool calls.
- **Complexity assessment**: while probing, record data scale (number of files/lines/pages) and write it to `findings` to help planning and delegation decisions.

### Step 3: Watchdog waiting + read conclusions

The main agent proactively monitors `output.txt` progress (`--verbose` includes raw tool outputs), instead of blindly sleeping:

1. **Observe**: read `output.txt` and inspect the exploration direction and raw data.
2. **Correct** (if needed):
   - Direction off → write `_intervene` with corrective guidance.
   - Missing key context → write `_keyinfo` to inject information.
   - Sufficient information gathered → write `_stop` to terminate early and save turns.
3. **Collect**: wait for `[ROUND END]`, then read `exploration_findings.md`.

**Output**: `exploration_findings.md` (structured findings). The main agent uses this to enter the planning phase and write the “Exploration Findings” section at the top of `plan.md`. First-hand knowledge the main agent acquired while watching can also be used for planning.

---

## II. Planning Phase (with review gate)

### Step 4: Read domain SOPs → write `plan.md`

First read the domain SOPs matched in the exploration phase, then write the plan skeleton. You can leave items marked as “⚠ pending confirmation”, but cannot postpone due to “insufficient investigation”.

**[D] Delegation annotation rules**: when writing each step, assess workload using exploration findings. Mark the step with `[D]` if any of these holds:

- Requires reading a large amount of code/files (estimated > 3 files or > 100 lines).
- Requires browsing web pages and extracting information.
- Requires 3+ repeated operations.
- Requires running tests/builds and analyzing output.

Cases where you **don’t** mark `[D]`:
- Reading/updating `plan.md`.
- Small-scale edits to a single file.
- `ask_user` interactions.
- Simple one-off commands.

**`plan.md` format:**

```markdown
<!-- EXECUTION PROTOCOL (read every run; this is your execution guide)
1. file_read(plan.md), find the first [ ] item.
2. If that step references an SOP → file_read that SOP’s quick reference section.
3. Execute the step + mini-verify the output.
4. file_patch to change [ ] → [✓] with a short result note, then go back to step 1 for the next [ ].
5. After all steps (including verification steps) are marked done → termination check: file_read(plan.md) and confirm there are 0 remaining [ ].
⚠ Forbidden: executing from memory | skipping verification steps | skipping termination checks | stopping to output pure text reports.
💡 For “grunt work” (reading large amounts of code/files/web pages/repeated actions), delegate to a subagent to keep the main agent’s context clean.
-->
# Task Title
Need: One-line need | Constraints: key limitations

## Exploration Findings
- Finding 1: XXX (source: file_read/web_scan/code_run)
- Finding 2: YYY
- Uncertainties: ZZZ

## Execution Plan
1. [ ] Step 1 summary
   SOP: xxx_sop.md
2. [D] Step 2 summary (delegate to subagent)
   SOP: yyy_sop.md
   Dependencies: 1
3. [P] Step 3 summary (parallel; see `subagent.md` for Map mode)
   SOP: yyy_sop.md
4. [?] Step 4 (conditional branch)
   SOP: (none) ← high risk
   Condition: if X succeeds → 4.1, else → 4.2

---

## Verification Checkpoints
N+1. [ ] **[VERIFY] Launch independent verification subagent**
     SOP: verify_sop.md plan_sop.md
     Action: read `plan_sop.md` Section 4 → prepare `verify_context.json` → start verification subagent → read VERDICT → act on the result.
     ⚠ Do not skip this step or mark `[✓]` without actually starting the subagent.

---
```

### Step 5: Self-checklist (main agent, item by item)

- □ Are all exploration findings reflected in the plan? (no missing key constraints)
- □ Are SOP references for each step appropriate? (does the SOP actually address that step?)
- □ Are dependencies between steps correct? (no hidden dependencies left implicit)
- □ Do high-risk steps (SOP: none / irreversible operations) have a clear execution strategy?
- □ Is step granularity appropriate? (no vague “process all files”—must be broken into concrete items)
- □ **Are complex/tedious steps marked with `[D]`?** (reading lots of code/pages/repetitive actions must be delegated.)
- □ **Does the plan include a "Verification Checkpoints" section with a `[VERIFY]` step? (mandatory)**

### Step 6: User confirmation

Use `ask_user` to get user confirmation before switching to execution.  
**⛔ Without user confirmation, you must not execute the plan.**

### Step 7: Switch to execution phase

Update checkpoint: `[Execution] plan.md | Current: Step 1 | ⚡ Steps marked [P] require reading subagent.md and using Map mode`.

---

## III. Execution Phase Loop

> **Core principle: execute continuously, no stopping for narrative reports.** After finishing one step, immediately `file_read(plan.md)` and move on to the next `[ ]` until all are done.

### Per-run process

1. **Read plan** — `file_read(plan.md)` and locate the first `[ ]` item.
2. **Read SOP** — if that step references an SOP, read that SOP first.
3. **Check markers** — if `[D]` → must delegate to a subagent (main agent only consumes the summary); `[P]` → read `subagent_sop.md` and use Map mode; `[?]` → evaluate the condition and select a branch, marking the unchosen branch as `[SKIP]`.
4. **Execute** — steps without special markers are executed by the main agent.
5. **Mini-verification** — quickly ensure the output exists and is reasonable (e.g. file_read confirms non-empty, check exit code, etc.).
6. **Mark complete** — `file_patch` to change `[ ]` → `[✓ short result]` (progress is recorded in `plan.md`).
7. **Continue** — immediately return to Step 1 and execute the next `[ ]`.

### Termination check (mandatory after the last step)

After marking the last step, `file_read(plan.md)` and scan the whole file to confirm all steps (including `[VERIFY]`) are `[✓]`/`[✗]`, with **zero** remaining `[ ]`.

Output: `🏁 Termination check: [total steps] steps completed, 0 remaining [ ] → Task finished`.

If you find any leftover `[ ]`, continue executing; you may not claim the task is complete.

### ⚠ Execution-phase prohibitions

- **No executing from memory**: before each new step, you must `file_read(plan.md)`; “I remember the next step is…” is forbidden.
- **No skipping verification steps**: `[VERIFY]` is mandatory and cannot be skipped because “everything else is done”.
- **No ending without termination check**: after marking the last step, you must `file_read` and confirm 0 remaining `[ ]` and output the termination line.
- **No stopping to output pure narrative reports**: after finishing a step, you must immediately `file_read(plan.md)` and proceed, not output progress summaries instead.

### 💡 Dynamic delegation principle

Even if a step was not originally marked `[D]`, if you discover any of the following during execution, proactively delegate it to a subagent:

- Requires reading many files/large volumes of code to understand context (> 3 files or estimated > 100 lines).
- Requires repeated trial-and-error debugging.
- Requires browsing the web and extracting information.

In such cases, start a subagent to complete the concrete operation, have it return a concise summary, and let the main agent continue making decisions based on that summary. Keeping the main agent’s context clean is top priority.

---

## IV. Verification Phase (independent subagent)

> Entered after all steps are marked `[✓]`. You **must** start an independent subagent for adversarial verification to avoid context contamination.

### Trigger conditions

- All execution steps are marked `[✓]`.
- **All plan-mode tasks must go through subagent-based verification** (the main agent is prone to confirmation bias and may be misled by apparent success).

### Step 8: Prepare verification context

Under `./plan_XXX/`, create `verify_context.json` containing:

- `task_description`: original task description (user’s words).
- `plan_file`: absolute path to `plan.md`.
- `task_type`: one of `code` | `data` | `browser` | `file` | `system`.
- `deliverables`: list of deliverables (type/path/expected).
- `required_checks`: list of mandatory checks (check/tool).

**What to pass**: task description, plan path, deliverable list, and required checks.  
**What not to pass**: execution process, debugging logs.

### Step 9: Start verification subagent

Start the verification subagent following `subagent.md`. Key input points:

- **Role**: you are an independent verifier whose job is adversarial validation (find evidence that the deliverables don’t work).
- **First mandatory step**: `file_read verify_sop.md` and fully read the verification SOP.
- **According to Section 3 of `verify_sop.md`**, choose the appropriate verification strategy based on `task_type`.
- **Each check must have tool-call evidence** (actual execution, not narration).
- **Task description**: fill in original task description.
- **Deliverables list**: fill in deliverables from `deliverables`.
- **Output**: write to `result.md` following Section 6 of `verify_sop.md`, with the last line `VERDICT: PASS / FAIL / PARTIAL`.
- **Constraints**: finish in 3 rounds, with at least 1 tool call per round.

Also pass the path to `verify_context.json` so the subagent can read detailed context itself.

### Step 10: Collect verification result

Poll `output.txt` until `[ROUND END]`, then read `result.md`:

1. **Locate VERDICT line**: read the last few lines of `result.md` and extract `VERDICT: PASS / FAIL / PARTIAL`.
2. **Check validity**: if all PASS items lack tool outputs (only narrative), treat verification as invalid and handle as FAIL.
3. **Act on the result**:
   - **PASS** → proceed to task completion wrap-up.
   - **FAIL** → enter the fix loop.
   - **PARTIAL** → main agent decides whether it is acceptable; otherwise fix.
   - **No VERDICT line** → extract key info from `output.txt` and have the main agent judge PASS/FAIL.

**Task completion wrap-up** (after PASS):

1. Mark the `[VERIFY]` step in `plan.md` as `[✓]`.
2. Update checkpoint: `[Done] Task XXX | [Output] ... | [Lessons] ...`.
3. Confirm task completion to the user.

**Important**: Only after verification PASS may you mark `[VERIFY]` as `[✓]` and claim task completion. If verification FAILs, you must enter the fix loop.

**Fallback**: if the subagent fails to produce `result.md` (turns exhausted), extract VERDICT-related key information from `output.txt`.

### Fix loop (after FAIL)

On FAIL: extract specific failure points → return to execution phase to fix (without re-planning) → after fixing → restart verification subagent → at most 2 FAIL–retry cycles, then `ask_user` for intervention.

During fixes:

1. Append new steps for failed items to `plan.md` (mark them `[FIX]`).
2. Fix only the failed items, don’t redo passed parts.
3. After fixing, recreate `verify_context.json` (containing only the failure items).

### Special scenarios

For browser/mouse-keyboard/scheduled-task scenarios: main agent performs operations and exports evidence (screenshots/recordings/logs) → subagent validates evidence files. **The main agent must not unilaterally decide PASS/FAIL.**

---

## V. Failure Handling

1. **Record**: in checkpoint, write `step_X: [FAILED] reason (retry: N/3)`.
2. **Retry**: network timeouts → auto-retry 3 times (2s/4s/8s); config errors → ask user; others → mark `[✗]` and skip.
3. **Subagent failure**: inspect `stderr.log` → have main agent fix clear issues and restart; unknown error → retry once; max 2 restarts.
4. **Dependency propagation**: when a step fails, mark dependent steps as `[SKIP]`.
5. **Plan error**: revert to planning phase and fix `plan.md`, go through the review gate again.

## Hard Constraints

- Every step must have an independent completion criterion.
- "Process all files" is forbidden; break into specific items.
- Only one step at a time; if the plan is wrong, return to planning phase and fix it.
- Add an extra verification step before irreversible operations.
