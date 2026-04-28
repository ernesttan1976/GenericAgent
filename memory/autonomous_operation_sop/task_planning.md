# Task Planning Mode

- **If TODO exists**: If `TODO.txt` under the current working directory has items to execute → go directly to the “Execution Flow”.

Value formula: **“Not coverable by AI training data” × “Has lasting benefit for future collaboration”**. The core deliverable is memory: valuable findings should be consolidated into memory update proposals and included in the report.

Entry conditions:
- **No TODO → Enter Task Planning Mode** (this run focuses only on planning, not executing tasks):
  0. `update_working_checkpoint`: `Planning mode: End this run immediately after producing TODOs, strictly forbidden to execute any TODO; wait until the next autonomous run to enter execution mode`.
  1. ⚠️ **Critically read `history.txt`**: 90% of historical tasks are low value. The goal is to **identify failure patterns and avoid them**, not to search for examples to imitate.
     - Identify low-value patterns: shallow validation, hypothesis-free inspections, repeated exploration, broad data collection without focus, basic usage of well-known tools.
     - Extract high-value leads: findings that were not followed up, tools that still need testing, outputs that could be improved.
  2. Reflect: Why were these tasks low value? How can we design tasks that are high value?
  3. Critically review existing reports and memories (`ls autonomous_reports/` + `../memory`) and consider how to increase their value or optimize them.
  4. Based on the above, produce 5–7 TODO items and write them into `TODO.txt`. Completed content can be compressed and moved to the bottom.
  5. Each TODO line format: `[ ] Type(Output/Surfing/Environment) | One-sentence goal | Acceptance criteria`.
  6. Summon a subagent to review the TODO list: provide only the TODO list as input + "Read the memory store yourself, then rate each item from 1–10 and briefly explain the reason" (do not give extra prior information).
  7. Read the subagent’s scores, and delete or replace low-scoring items.
  8. Immediately **end** this run. Execution happens in the next autonomous operation.

Priority of goals (in descending order of value):
1. **Practical outputs and capability expansion**: write tools to solve pain points and unlock new capabilities on top of existing ones (each new capability node expands the space of possibilities).
2. **Environment discovery**: scan for existing but unused tools/libraries/data sources/configuration.
3. **Niche tool mining**: search for lesser-known but practical tools on GitHub/V2EX/吾爱破解/果核剥壳, etc., and test solutions that AI often recommends but that may have pitfalls.
4. **Understanding the user and recommendations**: analyze old code/PC files/bookmarks to infer preferences and give personalized recommendations (games/videos/tools, with reasons) (low frequency).
5. **Self-evolution**: think about framework limitations and propose improvement plans.
6. **Memory review**: fix incorrect or outdated records.

**Large tasks**: You may design **high-value** large tasks and break them down into multiple modules or steps, writing each into the TODO list. Each autonomous run handles one module.

Selection principles: personalization first (knowledge that can only be obtained by probing this specific PC) → blind spots first (things the model cannot reconstruct from its parameters, with some difficulty) → hypothesis-driven (clear what to validate, explore and experiment while probing) → prohibit low-value validations (no checking static configs, no hypothesis-free inspections, no work that the user can easily do).

Exploration strategy (focus principles, not a menu):
- **Clue-driven**: follow-up tasks derived from recent reports have higher priority than topics picked from thin air.
- **Capability tree expansion**: prioritize tools/skills that can unlock new capability nodes (one node enables many possibilities).
- **Personalization first**: knowledge specific to this PC/user > generic knowledge.
- Surfing rules: at most 2 topics per session; must read content and extract insights, no headline harvesting. If you find a good tool → add a testing task for it to the next TODO.

Forbidden zones: ❌ Hacker News · scrolling news headlines · broad title collection without goals · exploring basic usage of well-known tools · researching agents weaker than the current framework · researching other web-automation/computer-use frameworks · reading this agent’s own codebase.
