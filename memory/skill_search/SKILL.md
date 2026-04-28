# Skill Search — 105K Skill Card Retrieval

> Perform semantic search over 105K+ skill cards to find the most relevant skill. Zero dependencies, built-in default API endpoint, ready to use out of the box.

## Minimal Usage

```python
import sys
sys.path.append('../memory/skill_search')
from skill_search import search

results = search("python send email")  # ⚠️ Queries MUST be in English; Chinese works very poorly.
for r in results:
    s = r.skill
    print(f"[{r.final_score:.2f}] {s.name} — {s.one_line_summary}")
    print(f"  key: {s.key}  category: {s.category}  tags: {s.tags[:3]}")
```

## API Signature

```python
search(query, env=None, category=None, top_k=10) -> list[SearchResult]
#  env: auto-detected in most cases, usually omit
#  category: optional filter, e.g. "devops"
#  top_k: number of results to return, default 10
```

## Return Structure

```
SearchResult
  .final_score    float     Overall score (0–1)
  .relevance      float     Semantic relevance
  .quality        float     Quality score
  .match_reasons  list[str] Reasons for the match
  .warnings       list[str] Warnings
  .skill          SkillIndex ↓

SkillIndex (common fields)
  .key              str       Unique identifier/path
  .name             str       Name
  .one_line_summary str       One-line summary
  .description      str       Detailed description
  .category         str       Category
  .tags             list[str] Tags
  .form             str       Form (sop/script/...)
  .autonomous_safe  bool      Safe for autonomous use
```

## CLI

```bash
python -m skill_search "python testing"
python -m skill_search "docker deployment" --category devops --top 5
python -m skill_search "git" --json
python -m skill_search --stats
python -m skill_search --env
```

## Configuration

| Item      | Default value                   | Description                                      |
|----------|----------------------------------|--------------------------------------------------|
| API URL  | `http://www.fudankw.cn:58787`    | Overridable via environment variable `SKILL_SEARCH_API` |
| API Key  | none (optional)                  | Environment variable `SKILL_SEARCH_KEY`          |
