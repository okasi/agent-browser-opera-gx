---
name: agent-browser-opera-gx
description: Control Opera GX browser via agent-browser CLI using CDP (Chrome DevTools Protocol). Enables AI-driven browser automation on the user's real browser with their sessions, cookies, and logins intact.
version: 1.2.1
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [browser, automation, opera-gx, cdp, agent-browser]
    related_skills: []
---

# Agent Browser + Opera GX

Control the user's **real Opera GX browser** via the `agent-browser` CLI using Chrome DevTools Protocol (CDP). Unlike headless browsers, this uses the user's actual browser — but login sessions may not survive CDP restarts.

## ⚠️ CRITICAL RULES

1. **NEVER use MCP Playwright** — it crashes, drops connections, and is unreliable. Use agent-browser or Direct CDP only.
2. **NEVER let agent-browser launch its own browser** — always connect to the user's Opera GX via CDP. If agent-browser launches its own headless Chrome, you lose all login sessions.
3. **Always use `--cdp 9222` flag** — this connects to Opera GX's CDP port instead of launching a new browser.
4. **For screenshots, ALWAYS use Direct CDP Python approach** — `agent-browser --cdp 9222 screenshot` is **BROKEN**: it connects to the wrong context, takes screenshots of its own headless browser, or returns blank/black images. The ONLY reliable method is Python websockets connecting directly to the tab's CDP endpoint.

## Prerequisites

```bash
npm install -g agent-browser
pip3 install websockets  # Required for Direct CDP screenshots
```

## Setup: Connect to Opera GX

Opera GX must be launched with CDP enabled:

```bash
# 1. Close Opera GX (warn user first!)
pkill -f "Opera GX.app"

# 2. Reopen with CDP (use background=true in terminal)
"/Applications/Opera GX.app/Contents/MacOS/Opera" --remote-debugging-port=9222

# 3. Verify CDP is running
curl -s http://localhost:9222/json/version

# 4. Open tabs via Direct CDP (agent-browser open is unreliable — opens in wrong context)
curl -s -X PUT "http://127.0.0.1:9222/json/new?https://www.linkedin.com"
```

**NEVER use `agent-browser connect`** — it often times out. Always use `--cdp 9222` flag with commands.

## Usage

### Opening Tabs (Primary: Direct CDP)

**`agent-browser --cdp 9222 open` is UNRELIABLE** — it reports success but opens URLs in its own headless Chrome context, NOT in Opera GX. Always use curl PUT to open new tabs:

```bash
# ✅ RELIABLE: Open new tab in Opera GX via CDP (MUST use PUT, not GET)
curl -s -X PUT "http://127.0.0.1:9222/json/new?https://www.linkedin.com"
# Returns JSON with tab id and webSocketDebuggerUrl

# ❌ UNRELIABLE: Reports success but opens in wrong context
agent-browser --cdp 9222 open https://linkedin.com
```

### Navigation (agent-browser for interaction within a tab)

```bash
agent-browser --cdp 9222 snapshot              # Get accessibility tree with refs
agent-browser --cdp 9222 get url               # Get current URL
agent-browser --cdp 9222 get title             # Get page title
# For screenshots: use Direct CDP Python approach (see below)
```

### Interacting with Elements (always with --cdp 9222)
Refs come from `agent-browser --cdp 9222 snapshot` output (e.g., `@e2`):
```bash
agent-browser --cdp 9222 click @e2             # Click element
agent-browser --cdp 9222 fill @e3 "text"       # Fill input field (clears first)
agent-browser --cdp 9222 type @e3 "text"       # Type into field
agent-browser --cdp 9222 select @e4 "value"    # Select dropdown option
agent-browser --cdp 9222 hover @e5             # Hover element
agent-browser --cdp 9222 press Enter           # Press keyboard key
```

### File Upload
```bash
agent-browser --cdp 9222 upload "input[type=file]" "/path/to/file.pdf"
```

### JavaScript Evaluation
```bash
agent-browser --cdp 9222 eval "document.title"
agent-browser --cdp 9222 eval "document.querySelectorAll('input[type=file]').length"
```

### Get Element Info
```bash
agent-browser --cdp 9222 get text @e2          # Get text content
agent-browser --cdp 9222 get html @e2          # Get innerHTML
agent-browser --cdp 9222 get value @e3         # Get input value
agent-browser --cdp 9222 get attr @e2 href     # Get attribute
```

## Pitfalls

1. **Opera GX must be restarted with `--remote-debugging-port=9222`** — CDP is not enabled by default. Always warn the user before killing their browser.
2. **`agent-browser --cdp 9222 open` is BROKEN for opening new tabs** — it reports success but opens URLs in agent-browser's own headless Chrome, NOT in Opera GX. Always use `curl -s -X PUT "http://127.0.0.1:9222/json/new?URL"` to open tabs.
3. **File upload dialogs are native** — use `agent-browser --cdp 9222 upload "input[type=file]"/path/to/file"` to handle them programmatically. osascript keystroke injection requires Accessibility permissions and often fails.
4. **Port 9222 must be free** — check with `lsof -i :9222` if connection fails.
5. **agent-browser refs are ephemeral** — re-run `snapshot` after page changes or clicks.
6. **Background process** — when launching Opera GX with CDP, use `terminal(background=true)` since it's a long-lived process.
7. **CDP verification** — always run `curl -s http://localhost:9222/json/version` after launch to confirm CDP is listening before connecting.
8. **Sessions lost on CDP restart** — launching Opera GX with `--remote-debugging-port` can drop login cookies for sites like LinkedIn and Google. User must re-login after CDP restart, then agent-browser can use the refreshed session.
9. **`agent-browser --cdp 9222 screenshot` is BROKEN** — it connects to agent-browser's own headless Chrome context, NOT the Opera GX tab. It will take screenshots of the wrong page, return blank/black images, or show a different site entirely. **ALWAYS use the Direct CDP Python approach** for screenshots (see below).
10. **Clipboard paste doesn't work** — `Meta+v` via agent-browser does NOT paste from system clipboard into web apps. Use `agent-browser --cdp 9222 keyboard inserttext "text"` instead for inserting text (works in Google Docs, textareas, etc.).
11. **osascript keystroke injection** — requires macOS Accessibility permissions (System Settings → Privacy → Accessibility). Without permission, it fails with error 1002. Prefer agent-browser's built-in commands.
12. **NEVER use `agent-browser connect`** — it times out. Always use `--cdp 9222` flag.
13. **NEVER use `agent-browser navigate`** — it launches its own headless Chrome. Use `agent-browser --cdp 9222 open` instead (but see pitfall #2 — even `open` is broken for new tabs; use curl PUT).
14. **`websockets` Python package required** — `pip3 install websockets` if not already installed. Needed for Direct CDP screenshots.

## Direct CDP Commands (Fallback)

When `agent-browser --cdp 9222` commands hang or produce wrong results, use raw CDP via curl:

```bash
# List all open tabs (returns JSON array)
curl -s http://127.0.0.1:9222/json/list

# Get browser WebSocket URL (changes on each restart)
curl -s http://127.0.0.1:9222/json/version

# Open a new tab (MUST use PUT, not GET)
curl -s -X PUT "http://127.0.0.1:9222/json/new?https://www.linkedin.com"
```

### Screenshot of a Specific Tab (Direct CDP WebSocket — PRIMARY METHOD)

**This is the ONLY reliable method for taking screenshots.** `agent-browser --cdp 9222 screenshot` connects to the wrong context and takes screenshots of agent-browser's own headless Chrome, not the Opera GX tab.

```python
import json, base64, asyncio, websockets

async def screenshot_tab(ws_url, output_path):
    # max_size=10MB needed — screenshots can be 1-2MB as base64
    async with websockets.connect(ws_url, max_size=10*1024*1024) as ws:
        await ws.send(json.dumps({
            "id": 1,
            "method": "Page.captureScreenshot",
            "params": {"format": "png"}
        }))
        response = json.loads(await ws.recv())
        img_data = base64.b64decode(response["result"]["data"])
        with open(output_path, "wb") as f:
            f.write(img_data)

# Get the target tab's WebSocket URL
# curl -s http://127.0.0.1:9222/json/list → find tab → use webSocketDebuggerUrl

asyncio.run(screenshot_tab(
    "ws://127.0.0.1:9222/devtools/page/TAB_ID_HERE",
    "/tmp/screenshot.png"
))
```

**Steps:**
1. `curl -s http://127.0.0.1:9222/json/list` — list tabs, find target by URL/title
2. Copy the `webSocketDebuggerUrl` from the matching tab
3. Run the Python script above with that WebSocket URL
4. Share the screenshot with `MEDIA:/tmp/screenshot.png`

**Why this works:** Connects directly to the exact tab's CDP endpoint, bypasses agent-browser's session management which can route to the wrong tab or a headless Chrome instance.

## Connecting to Other Chromium Browsers

Same CDP approach works for Chrome, Brave, Edge:
```bash
# Chrome
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --remote-debugging-port=9222

# Brave
"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser" --remote-debugging-port=9222
```

## Verified Workflows

- ✅ Google Drive — upload files via `agent-browser --cdp 9222 upload "input[type=file]"`
- ✅ Google Docs — create doc, insert text via `agent-browser --cdp 9222 keyboard inserttext`
- ✅ LinkedIn — navigate feed, read posts, take screenshots (must be logged in before CDP)
- ✅ Any site with Google/SSO login — uses real browser sessions (but Google blocks sign-in on CDP browsers)

## Important: Login Before Enabling CDP

Google, LinkedIn, and other services may block sign-in when they detect CDP (`--remote-debugging-port`). **Best practice:**
1. Have user log into all needed services in Opera GX normally (without CDP)
2. Then restart Opera GX with CDP enabled
3. Sessions/cookies *may* persist (varies by site) — LinkedIn sessions were lost on CDP restart in testing. Always verify login state after connecting; re-login manually if needed.

### Google Docs Creation Workflow

```bash
# 1. Create a new doc
agent-browser --cdp 9222 open https://docs.google.com/document/create

# 2. Find the document content area
agent-browser --cdp 9222 snapshot | grep "Document content"

# 3. Click the editor
agent-browser --cdp 9222 click @e86

# 4. Insert text — MUST use keyboard inserttext (NOT clipboard)
agent-browser --cdp 9222 keyboard inserttext "Your text here"

# For large content, split into chunks:
TEXT=$(head -50 /tmp/content.txt)
agent-browser --cdp 9222 keyboard inserttext "$TEXT"

# 5. Rename the doc
agent-browser --cdp 9222 snapshot | grep "Rename"
agent-browser --cdp 9222 fill @e76 "My Document Title"
agent-browser --cdp 9222 press Enter
```

**CRITICAL: Clipboard does NOT work with agent-browser.**
- `pbcopy` + `Cmd+V` — ❌ browser can't access system clipboard
- `navigator.clipboard.readText()` — ❌ times out (no user gesture)
- `agent-browser --cdp 9222 keyboard inserttext` — ✅ works perfectly

## Full Workflow: Navigate + Screenshot + Describe

When the user says "go to X and show me", use this exact sequence:

```bash
# 1. Check CDP is running
curl -s http://localhost:9222/json/version

# 2. Open new tab via Direct CDP (NOT agent-browser open — that's broken)
TAB_JSON=$(curl -s -X PUT "http://127.0.0.1:9222/json/new?https://www.example.com")
WS_URL=$(echo "$TAB_JSON" | python3 -c "import json,sys; print(json.load(sys.stdin)['webSocketDebuggerUrl'])")

# 3. Wait for page to load
sleep 4

# 4. Take screenshot via Direct CDP Python
python3 << 'PYEOF'
import json, base64, asyncio, websockets

async def screenshot(ws_url, output):
    async with websockets.connect(ws_url, max_size=10*1024*1024) as ws:
        await ws.send(json.dumps({"id": 1, "method": "Page.captureScreenshot", "params": {"format": "png"}}))
        resp = json.loads(await ws.recv())
        with open(output, "wb") as f:
            f.write(base64.b64decode(resp["result"]["data"]))
        print(f"Saved to {output}")

asyncio.run(screenshot("WS_URL_HERE", "/tmp/screenshot.png"))
PYEOF

# 5. Share with user via MEDIA: tag in response
# MEDIA:/tmp/screenshot.png
```

**Why this workflow:** `agent-browser --cdp 9222 open` opens URLs in its own headless Chrome (wrong context). `curl -X PUT /json/new` opens a real tab in Opera GX. The Python websockets screenshot captures exactly what's in that Opera GX tab.
