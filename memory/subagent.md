# Subagent Invocation SOP

This document describes conventions and best practices for launching and managing subagents (child agents) from a main agent.

## File and working directory protocol

- Each task runs in a temporary task directory: `temp/{task_name}/`.
  - When the main `cwd` is `temp/`, the subagent should use `./{task_name}/` as its working directory.
- Startup command (run from the repo root):

```bash
python agentmain.py --task {name} [--input "short input text"] [--bg] [--llm_no N]
```

- `--input` flag will instruct the launcher to create the task directory, clear previous outputs, and write `input.txt`.
  - For large inputs, the main agent should write `input.txt` manually and start the subagent without `--input`.
- Prefer `--bg` (background) to start subagents so the launcher prints the PID and exit status; the main agent can then poll outputs. When not using `--bg`, avoid combining start + poll in the same command.
- The subagent’s `cwd` must remain within `temp/`, never the main repo root.

## I/O files and communication

- Input files:
  - `input.txt`: primary short text instruction (created by launcher when `--input` used)
  - For longer structured inputs, the launcher should place files in the task dir and pass their absolute paths in `context.json`.
- Output files:
  - `output.txt`: the subagent appends progress outputs; batches are separated by `[ROUND END]` markers
  - `reply.txt`: main agent can write to this to continue the conversation; subagent responds in `output1.txt`, `output2.txt`, ...
- Intervention files:
  - `_stop`: if present, the subagent should stop after the current round.
  - `_keyinfo`: the main agent can inject working memory updates.
  - `_intervene`: append ad-hoc instructions.
- Timeout behavior: if `reply.txt` is not written within 10 minutes, the subagent should exit.

## Monitoring and responsible behavior

- The main agent should read the subagent’s `output.txt` periodically to observe real progress and avoid blind trust in summaries.
- Use `_intervene` and `_keyinfo` to correct drift or inject constraints; do not rely on long polling without actionable interventions.
- When running in verbose/monitoring mode, subagents may include raw tool execution logs in `output.txt`; the main agent should prefer raw logs over summaries for debugging.

## Use cases

1. Testing mode (behavior verification)

Purpose: Observe the subagent’s raw behavior and validate that it follows rules.
Flow:
- Prepare `test_path/` and write `input.txt`.
- Start the subagent.
- Poll `output.txt` at short intervals (e.g., every 2s) and validate outputs.
- Cleanup and iterate on failing scenarios.

Testing constraints:
- Only provide the goal and constraints, avoid suggesting steps or giving the subagent the exact SOP to use.
- Tests should reveal whether the subagent can independently find and use the correct SOP.

2. Map mode (parallel processing)

Purpose: Distribute many independent, similar tasks across multiple subagents concurrently.
Advantages: Isolated contexts reduce cross-task contamination.
Constraints:
- Use absolute file paths supplied in `context.json` for all file operations.
- Avoid sharing interactive resources like keyboard/mouse or browser tabs.
Process:
1. Main agent prepares multiple input files.
2. Launch a subagent per input.
3. Wait for completion and collect outputs.

3. Plan-mode inside a subagent

Principle: Subagents are full agents. For tasks with multiple dependent steps, a subagent should internally create a `subagent_plan.md` and execute using plan-mode.
When to use plan-mode:
- Task contains 3+ steps, or requires checkpoints and recovery.
How it’s done:
1. The main agent indicates in `input.txt` that the task has multiple steps and recommends plan-mode.
2. The subagent creates `./subagent_plan.md` and runs in plan-mode, updating the plan as it progresses.
3. The main agent only needs to read the final `output*.txt` files; it does not inspect the internal plan unless debugging is needed.

Context.json format example (main agent should create this file in the task directory):

```json
{
  "task": "task description",
  "work_dir": "/absolute/path/to/plan_dir/",
  "input_files": {
    "paper_info": "/absolute/path/to/paper_info.txt"
  },
  "output_files": {
    "pdf": "/absolute/path/to/paper.pdf",
    "report": "/absolute/path/to/paper_report.md"
  },
  "dependencies": ["paper_info.txt must exist"]
}
```

Important: On startup a subagent must read `context.json` first and then use the absolute paths listed for any file operations.