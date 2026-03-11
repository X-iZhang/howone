---
name: howone-automation
description: "Browser automation for HowOne AI (howone.ai) using playwright-cli.
  Covers login, creating AI apps from prompts, browsing App Store, using app previews,
  publishing apps, and saving results. Use when: user mentions HowOne, howone.ai,
  or wants to create/publish/browse apps on HowOne.
  Do NOT use for general web scraping or other platforms."
---

# HowOne Automation

Automate HowOne AI platform operations using `playwright-cli` snapshot-based browser control. No Python scripts — the agent issues `playwright-cli` commands directly.

## Prerequisites

```bash
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium
```

## Session Management

Session file: `~/.howone/session.json`

| Action | Command |
|--------|---------|
| Load session | `playwright-cli --state ~/.howone/session.json open https://howone.ai` |
| Save session | `playwright-cli state-save ~/.howone/session.json` |
| Clear session | `rm ~/.howone/session.json` |

After first login, **always save the session**. All subsequent operations load the session to skip login.

## Critical Rules

1. **NEVER hardcode ref numbers** — refs change every page load. Always `playwright-cli snapshot` first, then find elements by their text/role/label in the YAML output.
2. **Use `--headed` for OAuth login** — Google/GitHub OAuth require a visible browser window.
3. **Headless (default) for everything else** — after session is saved, all operations run headless.
4. **iframe elements use `f`-prefix refs** (e.g., `f19e17`) — app preview content lives inside `iframe[title="Preview"]`.
5. **Generation takes 5-15 minutes** — poll with `playwright-cli snapshot` every 60-90 seconds, don't rapid-fire.
6. **Always snapshot before acting** — read the YAML output to find the correct `<ref>` for the element you need.

## Capabilities

### 1. Login (First-Time Setup)

```bash
# Open headed browser (required for OAuth)
playwright-cli --headed open https://howone.ai

# Snapshot to find login button
playwright-cli snapshot

# Click login, complete OAuth in visible browser
playwright-cli click <login-ref>

# After login completes, save session
mkdir -p ~/.howone
playwright-cli state-save ~/.howone/session.json
```

Supports: Google OAuth, GitHub OAuth, Email+Code.

### 2. Create App from Prompt

```bash
# Load session and navigate
playwright-cli --state ~/.howone/session.json open https://howone.ai

# Snapshot → find prompt textarea
playwright-cli snapshot
playwright-cli fill <textarea-ref> "Your app description here"

# Snapshot → find generate/create button
playwright-cli snapshot
playwright-cli click <generate-button-ref>

# Monitor generation (poll every 60-90 seconds)
playwright-cli snapshot  # Look for agent activity, progress indicators

# When complete: URL changes to howone.ai/project/<id>
# Snapshot shows preview iframe and "Publish" button
```

**Generation indicators**: Look for agent names (Ava, Gabriel, Mia, Noah, Olivia) showing activity status. When all agents finish, the app preview appears.

### 3. Use App (Interact with Preview)

```bash
# Navigate to project
playwright-cli --state ~/.howone/session.json open https://howone.ai/project/<id>

# Snapshot — preview is inside iframe, elements have f-prefix refs
playwright-cli snapshot

# Interact with app elements inside the iframe
playwright-cli fill <f-input-ref> "test input"
playwright-cli click <f-button-ref>
```

### 4. Browse App Store

```bash
playwright-cli --state ~/.howone/session.json open https://howone.ai/apps

# Snapshot to see app cards
playwright-cli snapshot

# Click an app to view details
playwright-cli click <app-card-ref>
```

### 5. Publish App

```bash
# Navigate to project page
playwright-cli --state ~/.howone/session.json open https://howone.ai/project/<id>

# Snapshot → find Publish button
playwright-cli snapshot
playwright-cli click <publish-button-ref>

# Snapshot → confirm in dialog
playwright-cli snapshot
playwright-cli click <confirm-ref>

# Public URL: howone.ai/apps/<id>
# Live URL: <id>-<hash>.howone.app
```

### 6. Save Results Locally

```bash
# After app is created/used, extract results:

# Take screenshot
playwright-cli screenshot output/screenshot.png

# Get page content
playwright-cli snapshot > output/app-snapshot.yaml

# Download images from iframe (if app generates images)
# 1. Snapshot to find img elements (f-prefix refs in iframe)
# 2. Extract src URLs from snapshot YAML
# 3. curl to download
curl -o output/image.png "<image-url-from-snapshot>"

# Create results markdown
cat > output/results.md << 'EOF'
# App Results
- Project URL: https://howone.ai/project/<id>
- Public URL: https://howone.ai/apps/<id>
- Created: <date>
EOF
```

## URL Reference

| Page | URL Pattern |
|------|-------------|
| Dashboard | `https://howone.ai` (logged in) |
| Project editor | `https://howone.ai/project/<id>` |
| App Store | `https://howone.ai/apps` |
| Public app page | `https://howone.ai/apps/<id>` |
| Live app | `https://<id>-<hash>.howone.app` |

## HowOne AI Agents

During app generation, these AI agents collaborate:

| Agent | Role |
|-------|------|
| **Ava** | Product Manager — plans app structure |
| **Gabriel** | Workflow Designer — creates agentic workflows |
| **Mia** | Frontend Designer — designs UI |
| **Noah** | Developer — implements code |
| **Olivia** | QA — tests the app |

## Error Handling

| Issue | Solution |
|-------|----------|
| Session expired | Delete `~/.howone/session.json`, re-login with `--headed` |
| Element not found | `playwright-cli snapshot` and search YAML for the element by text/role |
| Generation stuck (>15 min) | Snapshot to check agent status; refresh page if no activity |
| iframe not loading | Wait 5-10 seconds, then snapshot again; iframe loads after main page |
| OAuth popup blocked | Must use `--headed` mode for OAuth flows |

## References

- [references/prompt-guide.md](references/prompt-guide.md) — Prompt templates and best practices
- [references/workflow-types.md](references/workflow-types.md) — HowOne workflow patterns and node types
- [references/playwright-workflows.md](references/playwright-workflows.md) — Detailed step-by-step command sequences
