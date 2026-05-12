# agent-browser-opera-gx 🎮🌐

Control your **real Opera GX browser** from the terminal using [agent-browser](https://github.com/vercel-labs/agent-browser) and CDP (Chrome DevTools Protocol).

## Why?

- Uses your **real browser** with all your logins, cookies, and sessions
- No extension installation needed
- No WebSocket connection drops
- Works with any Chromium-based browser (Opera GX, Chrome, Brave, Edge)

## Prerequisites

```bash
npm install -g agent-browser
```

## Quick Start

```bash
# 1. Close Opera GX
pkill -f "Opera GX.app"

# 2. Reopen with CDP enabled
"/Applications/Opera GX.app/Contents/MacOS/Opera" --remote-debugging-port=9222

# 3. Connect agent-browser
agent-browser connect 9222

# 4. Start automating!
agent-browser open https://google.com
agent-browser snapshot
agent-browser click @e2
```

## Usage

### Navigation
```bash
agent-browser open <url>           # Navigate to URL
agent-browser snapshot              # Get accessibility tree with refs
agent-browser screenshot page.png   # Take screenshot
agent-browser get url               # Get current URL
agent-browser get title             # Get page title
```

### Interacting with Elements
Refs come from `agent-browser snapshot` output (e.g., `@e2`):
```bash
agent-browser click @e2             # Click element
agent-browser fill @e3 "text"       # Fill input field
agent-browser type @e3 "text"       # Type into field
agent-browser select @e4 "value"    # Select dropdown option
agent-browser hover @e5             # Hover element
agent-browser press Enter           # Press keyboard key
```

### File Upload
```bash
agent-browser upload "input[type=file]" "/path/to/file.pdf"
```

### JavaScript Evaluation
```bash
agent-browser eval "document.title"
```

### Get Element Info
```bash
agent-browser get text @e2          # Get text content
agent-browser get html @e2          # Get innerHTML
agent-browser get value @e3         # Get input value
agent-browser get attr @e2 href     # Get attribute
```

## Other Chromium Browsers

Same CDP approach works for Chrome, Brave, Edge:
```bash
# Chrome
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --remote-debugging-port=9222

# Brave
"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser" --remote-debugging-port=9222
```

## Pitfalls

1. **Opera GX must be restarted with `--remote-debugging-port=9222`** — CDP is not enabled by default
2. **File upload dialogs are native** — use `agent-browser upload "input[type=file]" "/path/to/file"` to handle them programmatically
3. **User's Opera GX will close** — warn them before running `pkill -f "Opera GX.app"`
4. **Port 9222 must be free** — check with `lsof -i :9222` if connection fails
5. **agent-browser refs are ephemeral** — re-run `snapshot` after page changes

## License

MIT
