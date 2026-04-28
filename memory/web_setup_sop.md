# Web Toolchain Initialization SOP

If `web_scan` and `web_execute_js` have already been tested and work, you do not need this SOP.

This is only for initial installation when `code_run` is available but web tools are not yet configured.

## Goal

With only system-level permissions (`code_run`), establish web interaction capabilities (`web_scan` / `web_execute_js`).

## Prerequisite: Detect Browser

(Detect which browser is available and suitable for extension installation.)

## Install `tmwd_cdp_bridge` Extension

Extension path: `../assets/tmwd_cdp_bridge/` (MV3 Chrome extension with CDP debugger + scripting + cookie capabilities).

### Automatically Open the Extensions Page

`chrome://extensions` cannot be opened directly via command line or JavaScript; you must use the clipboard + address bar approach.

### Installation Steps (Chrome extension page is hard to automate)

1. Open the extensions management page and turn on "Developer mode".
2. Click "Load unpacked" and select the `assets/tmwd_cdp_bridge/` directory, or ask the user to drag it in manually.
3. Ignore the initial “Error” banner; it usually appears simply because GA has not connected yet.

## Verification

⚠ If `web_scan` shows “no available tabs”, that does not necessarily mean the extension is not installed. It may be that the browser is closed or only a blank page is open.

In this case, do not randomly try things. First use

```bash
start "" "https://www.baidu.com"
```

(or platform equivalent) to open a normal page, then call `web_scan` to verify.

If it still doesn’t work, you cannot automatically detect which is the default browser, where the extension is installed, or whether it is installed at all — at that point, request user assistance.
