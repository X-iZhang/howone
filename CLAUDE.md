# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **howone-automation**, an EvoScientist skill that automates operations on the [HowOne AI](https://howone.ai/) agentic app builder platform using `playwright-cli` snapshot-based browser automation. It creates, lists, publishes, and interacts with AI-generated apps programmatically.

## Commands

```bash
# Prerequisites
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium

# First-time login (requires visible browser for OAuth)
playwright-cli --headed open https://howone.ai
# Complete login, then save session:
mkdir -p ~/.howone
playwright-cli state-save ~/.howone/session.json

# All subsequent operations load the saved session:
playwright-cli --state ~/.howone/session.json open https://howone.ai

# Take a snapshot to see available elements
playwright-cli snapshot

# Interact with elements by ref (from snapshot)
playwright-cli fill <ref> "prompt text"
playwright-cli click <ref>

# Save session after any changes
playwright-cli state-save ~/.howone/session.json
```

## Architecture

The skill uses `playwright-cli` directly — no Python scripts. The agent reads SKILL.md and reference docs to learn the command sequences.

### Session Management

- First login stores browser state (cookies + localStorage) to `~/.howone/session.json`
- Subsequent runs load this session to skip login via `--state` flag
- Three auth methods: Email+Code, Google OAuth, GitHub OAuth
- Clear session with `rm ~/.howone/session.json` if login breaks

### Data Flow

```
Agent reads SKILL.md → issues playwright-cli commands → howone.ai browser session
                                                              ↓
                                                     Login (if needed, --headed)
                                                              ↓
                                                     Snapshot → find elements by text/role
                                                              ↓
                                                     Interact (fill, click, etc.)
                                                              ↓
                                                     Results → output/<app-name>/
```

### Key Design Decisions

- **No hardcoded refs**: Refs change every page load. Always snapshot first, find elements by text/role in the YAML output, then use the ref.
- **Headed vs headless**: Login = `--headed` (OAuth needs visible browser). Everything else = headless (default).
- **iframe as first-class concept**: App previews live inside `iframe[title="Preview"]`. Iframe elements get `f`-prefix refs (e.g., `f19e17`) in snapshots.
- **Session persistence**: `state-save` / `--state` manages session across browser launches.
- **Generation polling**: App generation takes 5-15 minutes. Poll with `playwright-cli snapshot` every 60-90 seconds.

### File Roles

| File | Purpose |
|------|---------|
| `SKILL.md` | EvoScientist skill manifest — capabilities, commands, rules |
| `references/playwright-workflows.md` | Detailed step-by-step command sequences for each operation |
| `references/prompt-guide.md` | Prompt templates and best practices for HowOne app generation |
| `references/workflow-types.md` | Documents HowOne's workflow node types and self-evolution modes |
| `output/` | Saved results from app generation (screenshots, markdown, images) |

### URL Reference

| Page | URL Pattern |
|------|-------------|
| Dashboard | `https://howone.ai` (logged in) |
| Project editor | `https://howone.ai/project/<id>` |
| App Store | `https://howone.ai/apps` |
| Public app page | `https://howone.ai/apps/<id>` |
| Live app | `https://<id>-<hash>.howone.app` |

## EvoScientist Skill Context

This directory is designed as an installable EvoScientist skill. The `SKILL.md` frontmatter defines the trigger: mentions of HowOne, howone.ai, agentic app builder, or requests to create/publish/browse apps. When loaded as a skill, the agent uses `playwright-cli` commands and the reference docs to fulfill user requests.
