# Memory Scanner SOP

## 1. Quick Start

Memory pattern search tool that supports Hex (Cheat Engine-style) and string matching. It also provides an LLM mode for convenient analysis of memory context.

**Python usage:**

```python
import sys
sys.path.append('../memory')  # Mount tools directory directly
from procmem_scanner import scan_memory

# Example: search for a specific hex signature with llm_mode enabled to get context
results = scan_memory(pid, "48 8b ?? ?? 00", mode="hex", llm_mode=True)
```

**CLI:**

```powershell
# Basic search
python ../memory/procmem_scanner.py <PID> "pattern" --mode string

# LLM-enhanced mode (outputs JSON with context, recommended)
python ../memory/procmem_scanner.py <PID> "pattern" --llm
```

## 2. Typical Scenario: Locating Structs or Critical Data

1. Determine leading features or known constants for the target data (e.g. a specific header or magic number).
2. Search for this feature in the target process:
   `scan_memory(pid, "4D 5A 90 00", mode="hex", llm_mode=True)`.
3. Analyze the `context` field in the returned JSON to inspect raw bytes and ASCII preview before and after the target address.

## 3. Notes

- **Permissions**: Administrator privileges are not strictly required, but you must have `PROCESS_QUERY_INFORMATION` and `PROCESS_VM_READ` rights for the target process.
- **Efficiency**: When scanning large memory regions, use more unique signatures to reduce false positives.

## 4. CE-style Differential Scan to Locate Dynamic Fields

Used for locating memory fields that change with actions in custom-drawn UIs like WeChat (e.g. current session title). Core idea: one full scan + multiple `ReadProcessMemory` passes to filter candidates.
