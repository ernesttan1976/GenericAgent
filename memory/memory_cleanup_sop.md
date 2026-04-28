# Memory Cleanup SOP

## Core Principle: Existence Encoding

LLMs act as compressors and decompressors. L1 only needs to make the model **aware that a class of knowledge exists**; then it can use tool calls to retrieve deep content when needed.

**Essence of L1:** express, in the fewest tokens, **what kinds of memories exist for which scenarios** (existence).

Two types of L1 content, both evaluated by ROI:
- **Existence pointers**: shortest trigger phrases pointing to L2/L3 knowledge.
- **Behavior rules**: mistakes that would be made without reminders (either critical or frequent, as long as ROI passes the threshold).

ROI = (probability of error without these tokens × cost of that error) / token cost per turn.

## Quick Judgement

**Keep**: counterintuitive trigger phrases — scenario keywords where you wouldn’t think to check an SOP without the prompt. Example: `tmwebdriver_sop(httponly cookie)`: without the phrase `httponly cookie`, you would not think to check `tmwebdriver_sop` when you need to fetch cookies.

**Delete**:
- Name translations: `proxy-pool/(proxy pool)` → the name is self-explanatory; the parenthesized translation is waste; use just `proxy-pool`.
- Content descriptions: `opencli_sop(66-site CLI, reuses Chrome session)` → implementation details belong inside the SOP, not L1.
- Intuitive capabilities: things you would obviously think of without a reminder → 0 benefit, wasted cost every turn.
- Redundancy: rules already covered in L3, or fragments already present in other L1 lines.

## Four Compression Principles

1. **Self-explanatory names > descriptions**: When the SOP name can describe itself, don’t add comments in L1. Renaming often has higher ROI than changing L1.
2. **Minimum description of existence sets**: For several similar entries that can be covered by one higher-level scenario, use the set name to encode existence of that ability rather than listing all subitems. For example, `QQ ops/Feishu ops/WeCom ops` → `IM ops: *_im_sop`; if child names are self-explanatory, list only names, don’t translate.
3. **Each entry = scenario ↔ solution existence**: `Video understanding: yt-dlp subtitles`, `fofa(asset mapping)` — scenario name is the trigger, scheme name encodes existence. Parentheses should contain **only counterintuitive trigger terms**; non-counterintuitive text (translations/content/implementation details) is waste.
4. **Layered placement**: entries with behavior rules or high-frequency, high-ROI rules go to top-level scenario lines, pure existence pointers go to L2/L3 flat lists.

## Cleanup Process

1. Read L1 line by line. Split by `|` into fragments and classify: existence pointers / RULES / translations / content description / implementation details / redundancy.
2. Triage RULES first: for each, ask “Is this a high-ROI global rule, or a low-risk rule for a specific scenario?”
   - High-ROI global → keep.
   - Scenario-specific / low-risk → demote to L3 or delete.
3. Then handle existence pointers: check whether each expresses **scenario ↔ solution existence**; scenario trigger words should be included **only** when counterintuitive; delete translations/content descriptions/implementation details.
4. Check whether L3 filenames are self-explanatory. If renaming can solve discoverability, don’t add description to L1. Finally, ensure total L1 lines ≤ 30.

## Red Line

Memory edits are persistent injuries; errors compound every turn. L1 can only be patched at the word level; overwrites are forbidden.

If something is misleading, promptly fix L1 or rename the memory entry.
