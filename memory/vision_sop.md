# Vision API SOP

## ⚠️ Pre-rules (Must Follow)

1. **Enumerate windows first**: before calling vision, use `pygetwindow` to list window titles and confirm that the target window exists and is active in the foreground. If the window does not exist, do not capture screenshots.
2. **🚫 No full-screen captures**: always use `ljqCtrl` to capture only the window region. If you can capture a sub-region (like the title bar), do that instead of the whole window; if you can capture one window, never capture the full screen. Full-screen screenshots are forbidden in all scenarios.
3. **Avoid vision when possible**: if the window title or local OCR (`ocr_utils.py`) can provide the required information, do not call the Vision API—this saves tokens and is usually more reliable. Vision is the last resort.

## Quick Usage

```python
from vision_api import ask_vision
result = ask_vision(image, prompt="Describe the image content", backend="claude", timeout=60, max_pixels=1_440_000)
# image: file path (str/Path) or PIL Image
# backend: 'claude' (default) | 'openai' | 'modelscope'
# returns str: model reply on success, or 'Error: ...' on failure
```

## If `vision_api.py` Does Not Exist: Initial Setup

1. Copy `memory/vision_api.template.py` → `memory/vision_api.py`.
2. Edit only the "User configuration" section at the top: inspect `mykey.py` for variable names (⚠️ look at names only; never output API key values) and try to find suitable config names to fill `CLAUDE_CONFIG_KEY` / `OPENAI_CONFIG_KEY`, choose `DEFAULT_BACKEND`, and test.
3. Fallback: if no usable config exists, go to `https://modelscope.cn/my/myaccesstoken` to apply for a token and fill `MODELSCOPE_API_KEY`.
