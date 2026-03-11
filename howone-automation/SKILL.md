---
name: howone-automation
description: "Browser automation for HowOne AI (howone.ai) using playwright-cli.
  Covers login, creating AI apps from prompts, browsing App Store, using app previews,
  publishing apps, and saving results. Use when: user mentions HowOne, howone.ai,
  or wants to create/publish/browse apps on HowOne.
  Do NOT use for general web scraping or other platforms."
---

# HowOne Automation

Automate HowOne AI platform operations using `playwright-cli` snapshot-based browser control.

**How it works**: snapshot the page → find element by text/role in YAML → act on its ref → snapshot again to verify.

## Prerequisites

```bash
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium
```

## Critical Rules

1. **Snapshot before every action.** Refs are ephemeral — they change on every page load. Never reuse a ref from a previous snapshot.
2. **Find elements by text/role, not by ref number.** Search the snapshot YAML for the element's `name` or `role`, then use its `ref`.
3. **Use `--headed` only for OAuth login.** Everything else runs headless (default).
4. **iframe elements have `f`-prefix refs** (e.g., `f19e17`). App previews live inside an iframe.
5. **Wait after navigation.** After `open` or `goto`, wait 3-5 seconds before snapshot. After `click` that triggers navigation, wait 2-3 seconds.
6. **Generation takes 5-15 minutes.** Poll with snapshot every 60-90 seconds. Do not rapid-fire.
7. **Always verify after acting.** After every click/fill, take a snapshot to confirm it worked before proceeding.

## Session Management

| Action | Command |
|--------|---------|
| First login (headed) | `playwright-cli --headed open https://howone.ai` |
| Save session | `mkdir -p ~/.howone && playwright-cli state-save ~/.howone/session.json` |
| Load session | `playwright-cli --state ~/.howone/session.json open https://howone.ai` |
| Clear session | `rm ~/.howone/session.json` |

## Capabilities

| # | Capability | When to use |
|---|-----------|-------------|
| 1 | **Login** | First time, or session expired (snapshot shows login page) |
| 2 | **Create App** | User provides an app idea/prompt |
| 3 | **Use App** | Interact with a generated app's preview |
| 4 | **Browse Store** | Search or explore apps on howone.ai/apps |
| 5 | **Publish** | Make a project publicly available |
| 6 | **Save Results** | Download screenshots, images, create result files |

For detailed step-by-step commands, see [references/playwright-workflows.md](references/playwright-workflows.md).

## URL Reference

| Page | URL |
|------|-----|
| Dashboard | `https://howone.ai` |
| Project | `https://howone.ai/project/<id>` |
| App Store | `https://howone.ai/apps` |
| Public app | `https://howone.ai/apps/<id>` |
| Live app | `https://<id>-<hash>.howone.app` |

## HowOne AI Agents

During generation, these agents collaborate (visible in left panel):

| Agent | Role |
|-------|------|
| **Ava** | Product Manager — plans app structure |
| **Gabriel** | Workflow Designer — creates agentic workflows |
| **Mia** | Frontend Designer — designs UI |
| **Noah** | Developer — implements code |
| **Olivia** | QA — tests the app |

## Quick Error Recovery

| Symptom | Action |
|---------|--------|
| Snapshot shows login page | Session expired → delete `~/.howone/session.json`, re-login with `--headed` |
| Click/fill fails | Ref is stale → snapshot again, find element by text/role |
| No `f`-prefix refs | iframe not loaded → wait 5 seconds, snapshot again |
| Generation >15 min no progress | Refresh: `playwright-cli goto https://howone.ai/project/<id>` |
| Page blank after open | Wait longer (3-5s), then snapshot; check URL is correct |

## References

- [references/playwright-workflows.md](references/playwright-workflows.md) — Step-by-step command sequences for all 6 capabilities
- [references/prompt-guide.md](references/prompt-guide.md) — Prompt templates for HowOne app generation
- [references/workflow-types.md](references/workflow-types.md) — HowOne workflow patterns and node types
