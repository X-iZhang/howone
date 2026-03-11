# HowOne Automation

Browser automation skill for [HowOne AI](https://howone.ai/) — an agentic app builder platform. Uses `playwright-cli` snapshot-based browser control to create, publish, browse, and interact with AI-generated apps programmatically.

Designed as an [EvoScientist](https://github.com/X-iZhang/EvoScientist) installable skill, consumed by Claude Code / AI agents directly — no Python scripts needed.

## How It Works

Instead of fragile CSS selectors, this skill teaches the agent to:

1. **Snapshot** the page → get a YAML description of all visible elements with refs
2. **Find** the target element by text/role (not hardcoded ref numbers)
3. **Act** on it (`click`, `fill`, `screenshot`, etc.)

This approach is resilient to UI changes since it doesn't depend on specific selectors.

## Capabilities

| Capability | Description |
|-----------|-------------|
| **Login** | Google/GitHub/Email OAuth with session persistence |
| **Create App** | Enter a prompt, monitor 5-15 min generation, get project URL |
| **Use App** | Interact with app preview iframe (input, click, read results) |
| **Browse Store** | Search and view apps on howone.ai/apps |
| **Publish** | Deploy app to App Store, get public URL |
| **Save Results** | Screenshot, download images, create result markdown |

## Quick Start

### Prerequisites

```bash
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium
```

### First-Time Login

```bash
# OAuth requires a visible browser
playwright-cli --headed open https://howone.ai

# Complete login, then save session
mkdir -p ~/.howone
playwright-cli state-save ~/.howone/session.json
```

### Create an App

```bash
# Load session (headless by default)
playwright-cli --state ~/.howone/session.json open https://howone.ai

# Snapshot → find prompt input → fill → generate
playwright-cli snapshot
playwright-cli fill <textarea-ref> "Create a pet hospital management app"
playwright-cli snapshot
playwright-cli click <generate-button-ref>

# Poll every 60-90 seconds until complete
playwright-cli snapshot
```

> **Note:** `<ref>` values change every page load. Always snapshot first, find elements by their text/role in the YAML output.

## Project Structure

```
├── howone-automation/
│   ├── SKILL.md                             # Skill manifest (triggers, rules, capabilities)
│   └── references/
│       ├── playwright-workflows.md          # Detailed step-by-step command sequences
│       ├── prompt-guide.md                  # HowOne prompt templates & best practices
│       └── workflow-types.md                # HowOne workflow patterns & node types
```

## Key Concepts

- **Session persistence**: `state-save` / `--state` flag manages login across browser launches (`~/.howone/session.json`)
- **Headed vs headless**: Use `--headed` for OAuth login; headless (default) for everything else
- **iframe refs**: App previews live in an iframe; their elements get `f`-prefix refs (e.g., `f19e17`)
- **No hardcoded refs**: Refs are ephemeral — always snapshot before acting

## EvoScientist Skill

Install as an EvoScientist skill:

```bash
EvoSci /install-skill path/to/howone/howone-automation
```

Triggers on mentions of: HowOne, howone.ai, agentic app builder, create/publish/browse apps.

## License

MIT
