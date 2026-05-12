---
name: agent-browser-opera-gx
description: Control Opera GX browser via agent-browser CLI using CDP (Chrome DevTools Protocol). Enables AI-driven browser automation on the user's real browser with their sessions, cookies, and logins intact.
version: 1.1.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [browser, automation, opera-gx, cdp, agent-browser, playwright-alternative]
    related_skills: [playwright-mcp]
---

# Agent Browser + Opera GX

Control the user's **real Opera GX browser** via the `agent-browser` CLI using Chrome DevTools Protocol (CDP). Unlike headless browsers, this uses the user's actual logged-in sessions.

## Why This Over Playwright MCP?

- Uses the user's **real browser** with all their logins/cookies
- No extension installation needed
- No WebSocket connection drops
- Works with any Chromium-based browser (Opera GX, Chrome, Brave, Edge)

## Prerequisites

```bash
npm install -g agent-browser
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

# 4. Connect agent-browser
agent-browser connect 9222
```

## Usage

### Navigation
```bash
agent-browser open <url>           # Navigate to URL
agent-browser open https://linkedin.com  # Opens logged-in LinkedIn feed
agent-browser snapshot              # Get accessibility tree with refs
agent-browser screenshot page.png   # Take screenshot
agent-browser get url               # Get current URL
agent-browser get title             # Get page title
```

### Interacting with Elements
Refs come from `agent-browser snapshot` output (e.g., `@e2`):
```bash
agent-browser click @e2             # Click element
agent-browser fill @e3 "text"       # Fill input field (clears first)
agent-browser type @e3 "text"       # Type into field
agent-browser select @e4 "value"    # Select dropdown option
agent-browser hover @e5             # Hover element
agent-browser press Enter           # Press keyboard key
```

### File Upload
```bash
# Trigger file upload on a file input element
agent-browser upload "input[type=file]" "/path/to/file.pdf"
```

### JavaScript Evaluation
```bash
agent-browser eval "document.title"
agent-browser eval "document.querySelectorAll('input[type=file]').length"
```

### Get Element Info
```bash
agent-browser get text @e2          # Get text content
agent-browser get html @e2          # Get innerHTML
agent-browser get value @e3         # Get input value
agent-browser get attr @e2 href     # Get attribute
```

### Multi-Tab
```bash
agent-browser open https://example.com  # Opens in current tab
# Use eval to manage tabs if needed
```

## Pitfalls

1. **Opera GX must be restarted with `--remote-debugging-port=9222`** — CDP is not enabled by default. Always warn the user before killing their browser.
2. **File upload dialogs are native** — use `agent-browser upload "input[type=file]" "/path/to/file"` to handle them programmatically. osascript keystroke injection requires Accessibility permissions and often fails.
3. **Port 9222 must be free** — check with `lsof -i :9222` if connection fails.
4. **agent-browser refs are ephemeral** — re-run `snapshot` after page changes or clicks.
5. **Background process** — when launching Opera GX with CDP, use `terminal(background=true)` since it's a long-lived process.
6. **CDP verification** — always run `curl -s http://localhost:9222/json/version` after launch to confirm CDP is listening before connecting.

## Connecting to Other Chromium Browsers

Same CDP approach works for Chrome, Brave, Edge:
```bash
# Chrome
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --remote-debugging-port=9222

# Brave
"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser" --remote-debugging-port=9222
```

## Verified Workflows

- ✅ Google Drive — upload files via `agent-browser upload`
- ✅ LinkedIn — navigate feed, read posts, take screenshots
- ✅ Any site with Google/SSO login — uses real browser sessions
