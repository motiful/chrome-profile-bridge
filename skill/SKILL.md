---
name: chrome-profile-bridge
description: Enhanced browser automation with Chrome profile targeting. Use this skill BEFORE playwright-cli when the user needs to work with a specific Chrome profile (e.g., "open my tax site", "check my bank account", "use my work profile"). This skill handles profile selection, then hands off to playwright-cli for actual browser control.
allowed-tools: Bash(pw:*), Bash(playwright-cli:*)
---

# Chrome Profile Bridge

Enhancement layer on top of `playwright-cli` that adds **Chrome profile targeting**.

## When to Use This Skill

Use this when the user mentions:
- A specific Chrome profile ("use my work profile", "open in Jack's browser")
- An authenticated site that requires existing login state (tax, banking, email)
- Any task where the correct Chrome profile matters

For tasks that don't need a specific profile, use `playwright-cli` directly.

## Architecture

```
This skill (profile targeting) → playwright-cli (browser control) → Chrome Extension → Real Chrome
```

## Why This Exists

| Approach | Problem |
|----------|---------|
| MCP | Loads ~25 tool definitions per session (~114K tokens). 4x heavier than Skill mode. |
| Claude in Chrome | Blocks financial/banking sites. Requires Anthropic login. Ecosystem lock-in. |
| Direct CDP | Chrome 136+ blocks remote debugging on default user data dir. |
| **This Skill** | Works inside Chrome (no restrictions), preserves all login state, zero site blocking. |

## Prerequisites

1. **@playwright/cli** — `npm install -g @playwright/cli@latest`
2. **Playwright MCP Bridge** Chrome extension — [Install from Chrome Web Store](https://chromewebstore.google.com/detail/playwright-mcp-bridge/mmlmfjhmonkocbjadbfplnigmagldckm)
3. **Token** — Copy from the extension's status page, set as `PLAYWRIGHT_MCP_EXTENSION_TOKEN` env var
4. **pw wrapper** — Copy `scripts/pw` to somewhere in your `$PATH` (e.g., `~/.local/bin/pw`)

## Usage

### List available Chrome profiles

```bash
pw list-profiles
```

### Open browser with a target profile

```bash
# Default profile
pw open --extension

# Specific profile
pw open --extension --pw-profile="Profile 1"
```

### Control the browser (all playwright-cli commands work)

```bash
playwright-cli goto https://example.com
playwright-cli snapshot
playwright-cli click e15
playwright-cli fill e7 "some text"
playwright-cli screenshot --filename=result.png
playwright-cli close
```

## Troubleshooting

- **ERR_BLOCKED_BY_CLIENT**: Extension not installed in target profile. Install from Chrome Web Store.
- **Extension timeout**: Chrome opened wrong profile. Use `--pw-profile="Default"` explicitly.
- **Token mismatch**: Verify env var matches the token shown in the extension's status page (`chrome-extension://mmlmfjhmonkocbjadbfplnigmagldckm/status.html`).
- **Extension ID confusion**: The Web Store version is `mmlmfjhmonkocbjadbfplnigmagldckm`. The dev/sideloaded version `jakfalbnbhgkpmoaakfflhflbfpkailf` is NOT compatible with `@playwright/cli`.
