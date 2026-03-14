# HowOne Automation

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code) that automates [HowOne AI](https://howone.ai/) — an agentic app builder platform. Give it an idea, and it creates a full web app with AI-powered workflows.

Supports two browser tools: **`browser-use`** (recommended) and `playwright-cli` (fallback). The agent auto-detects which is available.

## Install

**Option A: npx skills CLI**
```bash
npx skills add X-iZhang/howone
```

**Option B: Manual install**
```bash
git clone https://github.com/X-iZhang/howone
# Copy the whole skill directory:
cp -r howone ~/.claude/skills/howone
# Or just the main SKILL.md:
cp howone/SKILL.md ~/.claude/skills/howone.md
```

**Option C: Direct URL (no install needed)**

Claude Code can read the skill directly:
```
https://raw.githubusercontent.com/X-iZhang/howone/main/SKILL.md
```

## What It Does

You tell the agent what app you want. It:

1. **Expands your idea** into a detailed prompt optimized for HowOne
2. **Opens howone.ai** and logs in (session persisted after first login)
3. **Submits the prompt** and monitors generation (5–15 minutes)
4. **Publishes the app** and returns the public URL
5. **Tests the app** by interacting with the preview iframe

## Capabilities

| Capability | Description |
|-----------|-------------|
| **Login** | Google/GitHub/Email OAuth with session persistence |
| **Create App** | Expand user ideas into detailed prompts, submit, monitor generation |
| **Publish** | Deploy to App Store, get public URL |
| **Browse Store** | Search and view apps on howone.ai/apps |
| **Test App** | Interact with app preview iframe |
| **Save Results** | Screenshot, save state, create result markdown |

## Quick Start

### Prerequisites

```bash
# Install browser-use (recommended)
pipx install 'browser-use[cli]'
export PATH="$HOME/.local/bin:$PATH"

# OR install playwright-cli (fallback)
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium
```

### First-Time Login

```bash
# OAuth requires a visible browser
browser-use --headed open https://howone.ai
# Complete login, then save session:
mkdir -p ~/.config/browser-use
browser-use cookies export ~/.config/browser-use/howone.json
```

### Create an App

Just tell the agent:
> "Help me create a pet hospital management app on HowOne"

The agent will expand your idea into a detailed prompt, submit it to HowOne, and monitor the generation process.

## Project Structure

```
├── SKILL.md                     # Main skill — self-contained, prompt guide included
├── references/                  # Deep-dive reference docs
│   ├── browser-use-workflows.md # Full browser-use command sequences + debugging
│   ├── playwright-workflows.md  # Full playwright-cli workflows + UI map
│   ├── prompt-guide.md          # Prompt templates by app type
│   └── workflow-types.md        # HowOne workflow patterns & self-evolution
└── howone-automation/
    └── SKILL.md                 # Detailed version with all 7 workflows + UI map
```

## Key Concepts

- **Prompt quality is everything**: HowOne builds apps from your prompt — detailed prompts produce better apps
- **Preferred tool**: `browser-use` — numeric indices are more stable, better shadow DOM support
- **Session persistence**: Login once, reuse across sessions
- **Headed vs headless**: `--headed` only for OAuth login; headless for everything else
- **iframe interaction**: App previews are in iframes; `playwright-cli` handles these better (`f`-prefix refs)
- **Generation time**: 5–15 minutes — poll every 60–90 seconds

## License

MIT
