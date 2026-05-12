# agent-browser-opera-gx

Control your real Opera GX browser from the terminal using [agent-browser](https://github.com/vercel-labs/agent-browser) and CDP.

Uses your **actual browser** with all logins, cookies, and sessions — no extensions needed.

## Setup

```bash
npm install -g agent-browser

# Close and reopen Opera GX with CDP
pkill -f "Opera GX.app"
"/Applications/Opera GX.app/Contents/MacOS/Opera" --remote-debugging-port=9222

# Connect
agent-browser connect 9222
```

## Usage

```bash
agent-browser open https://google.com   # Navigate
agent-browser snapshot                    # Get page elements with refs
agent-browser click @e2                   # Click element by ref
agent-browser fill @e3 "hello"            # Fill input
agent-browser screenshot out.png          # Screenshot
agent-browser upload "input[type=file]" "/path/to/file"  # Upload file
agent-browser eval "document.title"       # Run JavaScript
```

## Pitfalls

- Opera GX must be restarted with `--remote-debugging-port=9222` (not enabled by default)
- Refs are ephemeral — re-run `snapshot` after page changes
- Port 9222 must be free — check with `lsof -i :9222`

Works with any Chromium browser (Chrome, Brave, Edge) — same CDP flag.
