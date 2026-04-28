# Scheduled Task SOP

Directory structure: `../sche_tasks/` contains task definition JSON files, and `../sche_tasks/done/` contains execution reports.

## Task JSON Format (`*.json`)

```json
{"schedule":"08:00", "repeat":"daily", "enabled":true, "prompt":"...", "max_delay_hours":6}
```

`repeat` options: `daily` | `weekday` | `weekly` | `monthly` | `once` | `every_Nh` (every N hours) | `every_Nd` (every N days).

`max_delay_hours` (optional, default 6): after how many hours past `schedule` the task should no longer trigger. This prevents executing stale tasks when the machine is turned on too late.

## Trigger Flow

1. `scheduler.py` (under `reflect/`) polls `sche_tasks/*.json` every 60 seconds.
2. The task triggers only if all conditions are met: `enabled = true` + current time ≥ `schedule` + cooldown elapsed (based on the latest report timestamp under `done/`).
3. When triggered, it builds the prompt, including a report path like `../sche_tasks/done/YYYY-MM-DD_taskname.md`.
4. **First thing after receiving the task**: use `update_working_checkpoint` to record the target report file path to avoid forgetting it during a long-running task.
5. After execution, write the report to that path (the scheduler uses this file to determine that today’s run is complete).

## Logging and Monitoring

- The scheduler automatically writes logs to `sche_tasks/scheduler.log` (triggered/skipped/errors).
- `scheduler.health_check()` returns a list of statuses for all tasks (`HEALTHY` / `OVERDUE` / `DISABLED` / `NEVER_RUN` / `ERROR`).
- JSON parse errors, invalid `schedule` formats, and unknown `repeat` values are all logged.

## Notes

- `once` type: runs once and then cools down for 100 years (effectively a permanent skip afterward).
- Task files only describe "what to do"; the report path is generated and injected into the prompt by the scheduler.
- The `sche_tasks` directory is `../`, i.e., under the code root.
