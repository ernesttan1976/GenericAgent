# TMWebDriver SOP

- Use the `web_scan` / `web_execute_js` tools directly. This document only records features and pitfalls.
- Under the hood: `../TMWebDriver.py` takes over the user's Chrome via an extension (preserves login state / cookies).
- Not Selenium/Playwright — it reuses the user's browser session.

## General Features
- ⚠ When using `await` inside `web_execute_js`, you must **explicitly `return`** the value to receive it (the underlying code wraps it in `async`; without `return` it returns `null`).
- ✅ `web_scan` automatically penetrates same-origin iframes; for cross-origin iframes you need CDP or `postMessage` (see section below).

## Limitations (isTrusted)
- JS events have `isTrusted=false`; sensitive operations (e.g. file uploads / some buttons) may be intercepted. For these scenarios prefer the **CDP bridge**.
- ⚠ Clicking a button via JS that doesn't open a new tab may be due to browser popup blocking — try clicking via CDP.
- File uploads: JS cannot populate `<input type=file>`; prefer CDP batch: getDocument → querySelector → DOM.setFileInputFiles; alternative is ljqCtrl physical click.
- When converting to physical coordinates: `physX = (screenX + rect.center_x) * dpr`, `physY = (screenY + chromeH + rect.center_y) * dpr`, where `chromeH = outerHeight - innerHeight`.

## Navigation
- `web_scan` only scans the current page and does not navigate. To switch sites use `web_execute_js` + `location.href='url'`.

## Google Image Search
- Class names are obfuscated; avoid hardcoding. Use `[role=button]` divs for clicks.
- `web_scan` filters sidebars. For popups use JS: use `document.body.innerText` for text; traverse large images by picking the `img` with the largest `naturalWidth` and take its `src`.
- "Visit" links: iterate anchors and find `a` whose `textContent.includes('访问')` to get the href.
- Thumbnails: `img[src^="data:image"]` can be extracted directly; large image src may be truncated — use `return img.src`.

## Chrome Downloading PDF
Scenario: PDF links open in the browser instead of downloading.
```js
fetch('PDF_URL').then(r=>r.blob()).then(b=>{
  const a=document.createElement('a');
  a.href=URL.createObjectURL(b);
  a.download='filename.pdf';
  a.click();
});
```
Note: must be same-origin or CORS allowed. For cross-origin, navigate to the target origin first.

## Chrome Background Tab Throttling
- In background tabs `setTimeout` may be throttled to ≥1 minute by Chrome's intensive throttling. Avoid relying on `setTimeout` polling in extension scripts.
- Some SPAs require CDP `Page.bringToFront` to switch the tab to foreground before data loads.

## CDP Bridge (tmwd_cdp_bridge extension) — preferred
Extension path: `assets/tmwd_cdp_bridge/` (must be installed; includes debugger permission).
⚠ TID convention: generated on first run at `assets/tmwd_cdp_bridge/config.js` (this file is gitignored). The extension references it via the manifest.

Call pattern: pass a JSON string directly to `web_execute_js` (the tool layer auto-detects object format and routes it via WS → background.js cmd routing).
```js
// Pass a JSON string as the script argument; no DOM ops needed
web_execute_js script='{"cmd": "cookies"}'
web_execute_js script='{"cmd": "tabs"}'
web_execute_js script='{"cmd": "cdp", "tabId": N, "method": "...", "params": {...}}'
web_execute_js script='{"cmd": "batch", "commands": [...]}'
// Returns JSON results directly
```
Communication modes: ⭐ JSON string direct (preferred) | TID DOM method (TID element + MutationObserver; `web_scan`/`web_execute_js` support both).
Single commands examples: `{cmd:'tabs'}` | `{cmd:'cookies'}` | `{cmd:'cdp', tabId:N, method:'...', params:{...}}` | `{cmd:'management', method:'list|reload|disable|enable', extId:'...'}`
- `management`: `list` returns extension info; `reload`/`disable`/`enable` require `extId`.
- ⭐ `batch` mixing: `{cmd:'batch', commands:[{cmd:'cookies'},{cmd:'tabs'},{cmd:'cdp',...},...]}`
  - Returns `{ok:true, results:[...]}` — multiple commands in one request; CDP lazy-attach reuses sessions.
  - Subcommands inherit outer batch `tabId` (e.g. `cookies` can correctly get the current page URL).
  - `$N.path` references the Nth result field (0-indexed), e.g. `"nodeId":"$2.root.nodeId"`.
  - ⚠ If a prior command in batch fails, later `$N` references silently become `undefined`; you must check each item’s `ok` flag in the `results` array.
  - Typical file upload: `getDocument({depth:1})` → `querySelector('input[type=file]')` → `setFileInputFiles`.
  - Design notes:
    - Keep `nodeId` provenance consistent within a chain; do not mix querySelector paths with performSearch paths.
    - After setting files, front-end frameworks may not notice; dispatch `input`/`change` events in JS if necessary.
    - Check `input.accept` before uploading; if multiple inputs exist use `accept` or parent container semantics to disambiguate.
    - Prefer `DOM.performSearch('input[type=file]')` for lightweight polling when waiting for elements.
    - For transient inputs the key is to **minimize the time between discovery and setFileInputFiles**: prefer doing it in the same batch; if not possible use DOM event listeners; monkey patches are last-resort fallbacks.
  - ⚠ `tabId`: CDP defaults to the sender.tab.id (the current injected page); cross-tab operations require an explicit `tabId` or use a batch-internal `tabs` query first.
- ⭐ Cross-tab operations do not require the tab to be foreground: just specify `tabId` to operate on a background tab.

## CDP Click Lifecycle (unverified, BBS#23)
- A generic click usually needs a **three-event sequence**: `mouseMoved` → `mousePressed` → `mouseReleased` (50–100ms interval).
  - Omitting `mouseMoved` can break hover-dependent components (MUI Tooltip / Ant Design Dropdown).
  - Autofill release is an exception: `mousePressed` alone may be sufficient (see autofill section).
- Coordinate correction when page has `transform:scale` / `zoom`:
```js
var scale = window.visualViewport ? window.visualViewport.scale : 1;
var zoom = parseFloat(getComputedStyle(document.documentElement).zoom) || 1;
var realX = x * zoom; var realY = y * zoom;
```
- For elements inside iframes, composed coordinates are `finalX = iframeRect.x + elRect.x`.
  - Cross-origin iframes cannot expose `contentDocument`.
  - ⚠ `Target.getTargets` / `Target.attachToTarget` may return "Not allowed" in the CDP bridge due to chrome.debugger permission restrictions.
  - ⭐ Verified approach: use `Page.getFrameTree` to find `frameId` → `Page.createIsolatedWorld({frameId})` to obtain `contextId` → `Runtime.evaluate({expression, contextId})` to run JS in the iframe.
  - In batch chains: `$0.frameTree.childFrames` iterate to find the frame whose URL matches, then pass `$1.executionContextId` to `evaluate`.
  - A `postMessage` relay works only if a content script has been injected into the iframe; third-party payment iframes usually prevent injection.

## CDP Text Input (unverified, BBS#23)
- `insertText` is fast but does not emit key events; controlled components may require dispatching `input` events.
- For full keyboard simulation use `dispatchKeyEvent` to send keystrokes one-by-one.

## CDP DOM Penetration into closed Shadow DOM (unverified, BBS#24/#25)
- `DOM.getDocument({depth:-1, pierce:true})` can pierce Shadow boundaries (including closed).
- `DOM.querySelector({nodeId, selector})` → `DOM.getBoxModel({nodeId})` to retrieve coordinates.
- `getBoxModel` returns content eight values `[x1,y1,...x4,y4]`; compute the center as the average of the four points: `centerX = sum(x)/4, centerY = sum(y)/4`.
  - ⚠ Do not simplify to diagonal midpoint — for rotated/skewed elements the four points do not form an axis-aligned rectangle.
- `querySelector` cannot cross Shadow boundaries with a combined selector — perform it in steps: find the host first, then query inside its shadow.
- ⚠ `nodeId` becomes invalid after DOM mutations → use `backendNodeId` or re-run `getDocument` to refresh.

## Autofill Release & Login
Detection: `web_scan` outputs inputs marked with `data-autofilled="true"` and values shown as protected hints (not the real value; Chrome protects autofill and requires a click to release).
- ⚠️ **Prerequisite**: you must `Page.bringToFront` the tab via CDP first — Chrome only releases autofill protections in a foreground tab; physical clicks in a background tab won't work.
- ⭐ One-click release & login: `bringToFront` → `mousePressed` on any field (no `mouseReleased` required; one press releases autofill for the whole page) → wait 500ms → dispatch `input/change` events if needed → click login.

## Captchas / Visual Page Screenshots
- ⭐ Preferred: CDP screenshot via `Page.captureScreenshot` (format:'png') → returns base64. Works in foreground or background tabs and yields full-page high-quality images.
- Captcha canvases/images: use `canvas.toDataURL()` in JS to get base64 directly when possible.

## simphtml & TMWebDriver Debugging
- `simphtml` debugging must inject JS into the real browser via `code_run` (Python side cannot simulate the DOM).
- `d = TMWebDriver()`, `d.set_session('url_pattern')`, `d.execute_js(code)` → returns `{'data': value}`.
- `simphtml`: `str(simphtml.optimize_html_for_tokens(html))` — returns a BS4 Tag; call `str()` to get the string.

## Connectivity Troubleshooting
When `web_scan` fails, troubleshoot in order (automatic checks first; ask user only as a last resort):
1. Is the browser running? → check processes (`tasklist`/`ps`). If not running, start it and open a normal URL (⚠ `about:blank` and other internal pages may not load extensions).
2. Is the WebSocket backend alive? → if port 18766 on local machine isn't listening it's dead → manually start the master with `from TMWebDriver import TMWebDriver; TMWebDriver()`.
3. Is the extension installed? → inspect Chrome user directory `Secure Preferences` → `extensions.settings` for a `path` entry containing `tmwd_cdp_bridge`.
   - If found → extension installed; investigate other causes.
   - If not found → follow `web_setup_sop`.
4. If all above are normal and still unable to connect → request user assistance.
