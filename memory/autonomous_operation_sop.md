# Autonomous Operation SOP

⚠️ **Path Warning**: `autonomous_reports` lives under `temp/` and should be accessed as `./autonomous_reports/`. It is **not** `../memory/autonomous_reports/` or `../autonomous_reports/`! TODO lives under the current working directory.

Reports are stored in `./autonomous_reports/` with filenames `RXX_short_description.md` (XX is inferred and auto-incremented from `history.txt`).

You are authorized to perform autonomous actions, as long as they do not cause side effects to the environment.

## Startup (First Step)
- `update_working_checkpoint`: `Autonomous operation | Re-read this SOP when wrapping up | from autonomous_operation_sop.helper import *; set_todo()/complete_task(tasktitle, historyline, report_path)`

Second step:
```python
from autonomous_operation_sop.helper import *
print(get_history(40))  # Understand history to avoid repetition
print(get_todo())       # View pending tasks
```

## Task Selection
- If there are unfinished TODO items → pick **one** and go straight into execution. Other items will be handled in future runs.
- If there is no TODO → read `autonomous_operation_sop/task_planning.md` to plan, and execute in the next run.
- Do not pick the same subtask two runs in a row.
- Value formula: **“Not coverable by AI training data” × “Has lasting benefit for future collaboration”**.

## Execution
- After selecting a task, call `update_working_checkpoint` and append the chosen TODO item and execution notes to the checkpoint.
- Call `code_run` to prepare the finalization callback, with `script` set to `handler._done_hooks.append("Re-read the autonomous operation SOP and check whether your wrap-up is correct; if not, fix it")`, and `inline_eval=True` (secret parameter).
- ≤ 30 turns per run. Move in small, fast steps; probe and experiment as you go.
- Use temporary scripts to validate hypotheses; do not draw conclusions based only on read-only inspection. Fully validate before writing the report.
- Even if the attempt fails, record the experimental process and results. Failure reports are also valuable.
- When the user is offline and a decision is needed, write it into the report for later review; do not get stuck.

**Wrap-up (All three are mandatory)**:
0. Re-read this SOP.
1. Write a report under the current working directory (filename is arbitrary). If you have suggestions for memory updates, append them at the end of the report.
2. `from/import helper; complete_task(tasktitle, historyline, report_path)` → this will auto-assign a number, move the report into `autonomous_reports/`, and prepend history (the `historyline` format is `Type | Topic | Conclusion`, strictly one line).
3. Call `set_todo()` to get the TODO path → mark the completed item as `[x]`.

## Permission Boundaries
- No approval needed: read-only probing; writes and script experiments under the current working directory.
- Must be written into the report for approval: modifying `global_mem` / SOPs under `memory`, installing software, calling external APIs, deleting non-temporary files.
- Absolutely forbidden: reading secrets/keys, modifying core codebase, and any irreversible dangerous operations.

## Waiting for User Review
- When the user returns, they will review the report and decide whether to approve, modify, or reject the proposal.
