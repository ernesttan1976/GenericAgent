# Memory Architecture & Management SOP

## 0. Core Axioms (Highest Priority)

1. **Action-Verified Only**
   - **Definition**: Any information stored in L1/L2/L3 must come from **successful tool calls** (e.g. `shell` executed successfully, `file_read` confirmed content, code executed correctly).
   - **Forbidden**: it is strictly forbidden to store the model’s “built-in knowledge”, speculative reasoning, unexecuted plans, or unverified assumptions as facts.
   - **Motto**: **No Execution, No Memory.**

2. **Sanctity of Verified Data**
   - **Definition**: Any verified configuration, pitfall guide, or critical path must **never be dropped** during refactoring or garbage collection.
   - **Operations**: you may compress wording and move items between layers (e.g. from L2 to L3), but you must not lose accuracy or traceability.
   - Memory edits are dangerous; avoid overwrites or large rewrites. Prefer small patches; if a safe patch is unclear, do nothing.

3. **No Volatile State**
   - **Definition**: highly volatile information must never be stored.
   - **Examples**: current timestamps, temporary session IDs, running PIDs, specific absolute paths, connected device lists.

4. **Minimum Sufficient Pointer**
   - Upper layers only keep the shortest identifiers needed to locate lower layers. Any extra token is redundancy.

---

## Memory Layer Architecture

```text
L1: global_mem_insight.txt (minimal index layer – hard limit ≤ 30 lines)
    ↓ navigation pointers
L2: global_mem.txt (global fact layer – short now, will grow)
    ↓ detailed references
L3: ../memory/ (record layer – contains .md/.py and other files)
L4: ../memory/L4_raw_sessions/ (historical session layer – scheduler collects references automatically)
```

---

## Layer Responsibilities & Principles

### L1: Global Memory Index (`global_mem_insight.txt`)

**Role**: provide a minimal navigation index into L2 and L3 so that key capabilities can be discovered.

**Characteristics**:
- Size: ≤ 30 lines (hard limit), < 1k tokens (target). No detailed content (unless for extremely high-frequency tasks).
- Content: two-layer mapping from scenario keywords → memory locations, plus RULES (red-line rules + frequent mistakes).
  - First layer: high-frequency scenario `key → value` (direct SOP/py/L2 section names). Self-explanatory names should be single tokens without duplicate translations.
  - Second layer: low-frequency scenarios list only keywords; when needed, read L2 or `ls` L3 to locate details.
  - Core: scenario trigger phrases are crucial (without them, you don’t know a capability exists), but detailed "how-to" instructions are banned.
  - RULES: compressed pitfall guidelines, including:
    - Red-line rules (catastrophic): violations cause process termination or system crashes (e.g. `Do not kill python without conditions (may kill this process)`).
    - Red-line rules (subtle): violations don’t crash but silently produce wrong results (e.g. `Use google instead of baidu for search`).
    - Frequent mistakes: easy-to-forget constraints (e.g. `es(PATH exists)` to prevent missing paths).
- Updates: when L2/L3 changes, adjust L1 topic navigation. Change as little as possible; only micro-patch, avoid overwrites.

**Forbidden**:
- Storing passwords or API keys.
- Writing "How to" or detailed explanations.
- Storing task-specific technical details (these belong in L3).  
- Storing logs.

---

### L2: Global Fact Store (`global_mem.txt`)

**Role**: store global environment facts (paths, credentials, configuration, constants, etc.).

**Characteristics**:
- Trend: grows with the environment (acceptable).
- Content: organized into sections under `## [SECTION]`.
- Sync: when values change, update corresponding L1 topics only if scenario discovery is affected.

**Forbidden**: volatile state, guesses, and common sense that the model can already infer.

---

### L3: Task-level Record Store (`../memory/`)

**Role**: store a small amount of detail that L1/L2 cannot hold but is crucial for future reuse in **specific tasks**. The content must be as short as possible while still supporting reuse.

Principles:
- Only record information that remains important across sessions and cannot be quickly reconstructed via a few `file_read`/`web_scan`/simple scripts.
- Prioritize hidden prerequisites and typical pitfalls for a task that would be expensive to rediscover.
- Do not record ordinary steps or state that can be rediscovered cheaply.

Forms:
- SOPs (`*_sop.md`): minimal lists of "critical prerequisites + typical pitfalls" for a specific task or small class of tasks. No long tutorials.
- Tool scripts (`*.py`): encapsulate highly reusable, relatively complex logic that we don’t want to re-derive every time.

---

## L1 ↔ L2/L3 Sync Rules

| Operation          | L1 sync behavior |
|--------------------|------------------|
| New L2/L3 scenario | Add to L3 list as low-frequency by default (self-explanatory names only; add parentheses trigger phrase only when counterintuitive). |
| Delete L2/L3 item  | Remove corresponding keyword/mapping line. |
| Modify L2/L3 value | Do not change L1 unless scenario discoverability changes. |
| Discover new global rule | Compress into a single sentence and add to RULES. |

> **Sync red line**: L1 stores only keywords/names, never detailed content. Always check L1 token count and index usefulness.

---

## Information Classification Decision Tree

```text
"Where should this piece of information go?"

Is it an "environment-specific fact"? (IP, non-standard path, credential, ID, API key, etc., that the model cannot zero-shot accurately)
  ├─ YES → L2 (global_mem.txt)
  │        then → according to frequency, add to L1 first-layer key→value or second-layer keyword only.
  │
  └─ NO
       ↓
       Is it a "general operating rule"? (global pitfall guide, troubleshooting method, non-task-specific rules)
       ├─ YES → L1 [RULES] (single compressed sentence per rule)
       │
       └─ NO
            ↓
            Is it "task-specific technique"? (hard-won tricks and setups that are likely to be reused later, e.g. WeChat parameter parsing, specific game coordinates, temporary tool configuration)
            ├─ YES → L3 (`../memory/` SOP or script)
            │
            └─ NO → classify as "general knowledge" or "redundant" → must not be stored, discard.
```
