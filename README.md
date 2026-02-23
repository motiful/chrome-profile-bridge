# Chrome Profile Bridge

> A Claude Code skill that adds Chrome profile targeting to `playwright-cli`. Control your real Chrome browser with existing login state — no MCP overhead, no site restrictions, no ecosystem lock-in.

## The Problem

You want an AI coding agent to help with tasks in your browser (tax forms, admin dashboards, authenticated apps). But:

| Existing Solution | Why It Falls Short |
|---|---|
| **Playwright MCP** | Loads ~25 tool definitions per session (~114K tokens). Heavy. |
| **Claude in Chrome** | Blocks financial/banking sites. Requires Anthropic login. |
| **Direct CDP** | Chrome 136+ blocks remote debugging on default user data dir. |
| **playwright-cli `--extension`** | Works great, but no profile targeting — opens in last-active window. |

## The Solution

A thin wrapper (`pw`) that adds `--pw-profile` to `playwright-cli`, routing the browser extension connection to the correct Chrome profile.

```
pw open --extension --pw-profile="Work"     # Opens in your Work profile
playwright-cli goto https://tax-site.com    # Already logged in
playwright-cli snapshot                     # See the page
playwright-cli click e12                    # Interact
playwright-cli close                        # Done
```

## How It Works

```
Claude Code
  → pw (profile targeting)
    → playwright-cli (browser control, Skill mode, ~27K tokens)
      → Playwright MCP Bridge extension (inside Chrome)
        → Your real Chrome tab (all cookies, login state intact)
```

## Installation

### Prerequisites

1. Install `@playwright/cli`:
   ```bash
   npm install -g @playwright/cli@latest
   ```

2. Install the [Playwright MCP Bridge](https://chromewebstore.google.com/detail/playwright-mcp-bridge/mmlmfjhmonkocbjadbfplnigmagldckm) Chrome extension in each profile you want to control.

3. Copy the token from the extension's status page and add to your shell:
   ```bash
   echo 'export PLAYWRIGHT_MCP_EXTENSION_TOKEN="your-token-here"' >> ~/.zshrc
   source ~/.zshrc
   ```

### Install the Skill

**Option A: Plugin marketplace (recommended)**
```
/plugin marketplace add yourusername/chrome-profile-bridge
/plugin install chrome-profile-bridge
```

**Option B: Manual**
```bash
# Install the skill
cp -r skill ~/.claude/skills/chrome-profile-bridge

# Install the pw wrapper
cp scripts/pw ~/.local/bin/pw
chmod +x ~/.local/bin/pw
```

**Option C: Install playwright-cli skill alongside**
```bash
cd ~/.claude/skills && playwright-cli install --skills
```

## Usage

```bash
# List your Chrome profiles
pw list-profiles

# Open default profile
pw open --extension

# Open specific profile
pw open --extension --pw-profile="Profile 1"

# All playwright-cli commands work after opening
playwright-cli goto https://example.com
playwright-cli snapshot
playwright-cli click e15
playwright-cli screenshot
playwright-cli close
```

## Gotchas

- **Extension ID**: Must use the Chrome Web Store version (`mmlmfjhmonkocbjadbfplnigmagldckm`). Dev/sideloaded versions have a different ID and won't work with `@playwright/cli`.
- **Token per profile**: Each Chrome profile generates its own token. If you control multiple profiles, you'll need the token from the profile you're targeting (or use the same extension across profiles).
- **macOS only** (for now): The `pw` script uses macOS Chrome paths. PRs welcome for Linux/Windows support.

## License

MIT
