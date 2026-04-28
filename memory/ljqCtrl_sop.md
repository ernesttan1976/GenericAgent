# ljqCtrl Usage & Coordinate Conversion SOP

> **Must call update working checkpoint**: `Always use physical coordinates with ljqCtrl | Do not use pyautogui | Activate window with gw before operating`.

## 0. API Quick Reference (Signatures)

- `ljqCtrl.dpi_scale`: `float` (scale factor = logical width / physical width).
- `ljqCtrl.SetCursorPos(z)`: move the mouse to logical coordinate `z = (x, y)`.
- `ljqCtrl.Click(x, y=None)`: simulate a click. Supports `Click((x, y))` or `Click(x, y)`.
- `ljqCtrl.Press(cmd, staytime=0)`: simulate key presses, e.g. `Press('ctrl+c')`.
- `ljqCtrl.FindBlock(fn, wrect=None, threshold=0.8)`: image matching. Returns `((center_x, center_y), is_found)`.
- `ljqCtrl.MouseDClick(staytime=0.05)`: mouse double-click.

## 1. Environment Setup

You must add `../memory` to `sys.path` before importing tool modules:

```python
import sys, os, pygetwindow as gw
sys.path.append("../memory")
import ljqCtrl
```

## 2. Core: High-DPI Physical Coordinate Conversion

`ljqCtrl`’s `Click/MoveTo` interfaces accept **physical pixel coordinates**.

When using `pygetwindow` or similar tools to get window positions (logical coordinates), you must divide by the scale factor:

- **Conversion formula**: `physical_coord = logical_coord / ljqCtrl.dpi_scale`.
- **Note**: 3840 (4K) is just an example for the development machine; actual physical bounds depend on the system. Always compute using `dpi_scale` dynamically.

## 3. Window Operations & Click Flow

1. **Activate window**: use `gw.getWindowsWithTitle('title')` to get the window, then call `restore()` and `activate()`.
2. **Coordinate calculation**:

```python
win = gw.getWindowsWithTitle('WeChat')[0]
# Compute logical coordinates (lx, ly) for a point inside the window
# Convert to physical coordinates and click
px, py = lx / ljqCtrl.dpi_scale, ly / ljqCtrl.dpi_scale
ljqCtrl.Click(px, py)
```

## 4. Pitfall Guide

- **⚠️ Always use physical coordinates**: coordinates passed to `ljqCtrl.Click/SetCursorPos` must be physical coordinates (equal to screenshot pixel coordinates). Logical coordinates from `pygetwindow` must be converted with `/ dpi_scale`. Do not pass logical coordinates directly.
- **Physical verification**: before simulating, ensure the window has been brought to the foreground via `activate()`.
- **Offsets**: all relative pixel offsets (e.g. “move 10 pixels to the right”) must also be divided by `dpi_scale`.
- **Coordinate alignment**: physical coordinates must match screenshot coordinates; `ljqCtrl` handles DPI conversion internally—do not manually recalculate twice.
- **⚠️ Window coordinate conversion trap**: `win32gui.GetWindowRect(hwnd)` includes title bar and borders, while screenshots cover the client area. When clicking screenshot elements, use `win32gui.ClientToScreen(hwnd, (0, 0))` to get the client area origin in screen coordinates, then add screenshot coordinates. Do not simply use `GetWindowRect` top-left + screenshot coordinates.
- **⚠️ Win32 DPI coordinate trap**: without calling `SetProcessDPIAware()`, APIs like `GetWindowRect/ClientToScreen/GetClientRect` often return **logical coordinates**. If screenshots or `ljqCtrl` use physical pixels, you must normalize with `/ ljqCtrl.dpi_scale`. Alternative: call `SetProcessDPIAware()` first and then use raw physical coordinates everywhere, never mixing logical and physical coordinates.
- **Text input**: `ljqCtrl` has no `TypeText/SendKeys`. To enter text into an input field: first click or triple-click to select it, then `pyperclip.copy('text'); ljqCtrl.Press('ctrl+v')`.
