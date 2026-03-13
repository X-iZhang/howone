---
name: howone-automation
description: "Browser automation for HowOne AI (howone.ai) using browser-use or playwright-cli.
  Covers login, creating AI apps from prompts, browsing App Store, using app previews,
  publishing apps, and saving results. Use when: user mentions HowOne, howone.ai,
  or wants to create/publish/browse apps on HowOne.
  Do NOT use for general web scraping or other platforms."
---

# HowOne Automation

Automate HowOne AI platform operations. Supports two browser tools — **auto-detect which is available first**.

---

## Step 0: Detect Available Tool (Always Do This First)

```bash
# Check browser-use
browser-use --help > /dev/null 2>&1 && echo "USE: browser-use" || echo "browser-use not found"

# Check playwright-cli
playwright-cli --help > /dev/null 2>&1 && echo "USE: playwright-cli" || echo "playwright-cli not found"
```

Use whichever tool is available. If both are installed, **prefer `browser-use`** (numeric indices are more reliable than hex refs).

---

## Prerequisites (if tool not yet installed)

**browser-use** (Python, preferred):
```bash
pipx install 'browser-use[cli]'
# OR
uv tool install 'browser-use[cli]'
# Requires Python 3.11+. First run auto-installs Chromium.
export PATH="$HOME/.local/bin:$PATH"
```

**playwright-cli** (Node.js, fallback):
```bash
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium
```

---

## Command Reference

Every action maps to both tools:

| Action | `browser-use` | `playwright-cli` |
|--------|--------------|-----------------|
| Open page (headless) | `browser-use open <url>` | `playwright-cli --state ~/.howone/session.json open <url>` |
| Open page (headed/OAuth) | `browser-use --headed open <url>` | `playwright-cli --headed open <url>` |
| See page elements | `browser-use state` | `playwright-cli snapshot` |
| Click element | `browser-use click <index>` | `playwright-cli click <ref>` |
| Type into focused field | `browser-use type "text"` | *(use fill instead)* |
| Fill a field | `browser-use input <index> "text"` | `playwright-cli fill <ref> "text"` |
| Press key | `browser-use keys "Enter"` | `playwright-cli press <ref> Enter` |
| Screenshot | `browser-use screenshot path.png` | `playwright-cli screenshot path.png` |
| Save session | `browser-use cookies export ~/.config/browser-use/howone.json` | `playwright-cli state-save ~/.howone/session.json` |
| List tabs | `browser-use sessions` | `playwright-cli tab-list` |
| Switch tab | `browser-use switch <n>` | `playwright-cli tab-switch <n>` |
| Run JS | `browser-use eval "js"` | `playwright-cli evaluate "js"` |

### How Elements Work (Key Difference)

**`browser-use`** — `state` returns a numbered list:
```
[15]<textarea id=app-prompt />
[16]<button aria-label=Generate />
```
→ Use the number: `browser-use click 16`
→ Find elements: `browser-use state | grep textarea`

**`playwright-cli`** — `snapshot` returns YAML with hex refs:
```yaml
- role: textbox
  name: "Describe the application..."
  ref: e4a2
- role: button
  name: "Generate"
  ref: e5b3
```
→ Use the ref: `playwright-cli click e5b3`
→ Find elements by `role` + `name`, never reuse a ref from a previous snapshot

---

## Critical Rules

1. **Always re-read elements before acting.** Indices (browser-use) and refs (playwright-cli) change on every page load.
2. **Use `--headed` only for OAuth login.** Everything else runs headless.
3. **Wait after navigation.** After `open`, wait 3-5 seconds before reading elements.
4. **Generation takes 5-15 minutes.** Poll every 60-90 seconds. Do not rapid-fire.
5. **Verify after every action.** Read page state again to confirm the action worked.

---

## Workflow 1: Login

OAuth requires a visible browser (`--headed`). Three auth methods: Google OAuth, GitHub OAuth, Email+Code.

**browser-use:**
```bash
export PATH="$HOME/.local/bin:$PATH"
browser-use --headed open https://howone.ai
sleep 3
browser-use state | grep -iE "login|sign|google|github|email"
# Click the login button
browser-use click <login-button-index>
# Fill email if using Email+Code
browser-use input <email-index> "your@email.com"
browser-use click <continue-index>
# Enter verification code
browser-use input <code-index> "123456"
browser-use click <verify-index>
# Verify success (should see your name)
browser-use state | grep -i "hello"
# Save session
mkdir -p ~/.config/browser-use
browser-use cookies export ~/.config/browser-use/howone.json
```

**playwright-cli:**
```bash
playwright-cli --headed open https://howone.ai
playwright-cli snapshot
playwright-cli click <ref-of-login-button>
playwright-cli snapshot
playwright-cli fill <ref-of-email-textbox> "your@email.com"
playwright-cli click <ref-of-continue-button>
playwright-cli snapshot
playwright-cli fill <ref-of-code-textbox> "123456"
playwright-cli click <ref-of-verify-button>
playwright-cli snapshot   # verify: should show dashboard, not login page
mkdir -p ~/.howone
playwright-cli state-save ~/.howone/session.json
```

**Login success:** Page shows your name (e.g. "Hello, Xi") or a project prompt textarea — not a login button.

---

## Workflow 2: Create App from Prompt

**browser-use:**
```bash
export PATH="$HOME/.local/bin:$PATH"
browser-use open https://howone.ai
sleep 3
browser-use state | grep textarea
browser-use click <textarea-index>
browser-use type "Your app prompt here"
browser-use keys "Enter"
sleep 5
browser-use state | grep -iE "generating|building|project"
```

**playwright-cli:**
```bash
playwright-cli --state ~/.howone/session.json open https://howone.ai
playwright-cli snapshot
playwright-cli fill <ref-of-textarea> "Your app prompt here"
playwright-cli snapshot
playwright-cli click <ref-of-generate-button>
playwright-cli snapshot   # URL should change to howone.ai/project/<id>
```

**Generation stages** (check with `state` / `snapshot`):

| Stage | browser-use sees | playwright-cli sees |
|-------|-----------------|-------------------|
| Starting | "Generating info..." | agent names visible |
| In progress | "Our AI agents are building..." | agent messages |
| Complete | "Completed" × many, "Publish Your App Now" | Publish button + f-prefix refs |

---

## Workflow 3: Monitor Generation (5-15 minutes)

**browser-use:**
```bash
# Poll every 60 seconds
browser-use state | grep -iE "ready|Publish Your App Now|Completed"
browser-use screenshot /tmp/progress.png
```

**playwright-cli:**
```bash
playwright-cli snapshot
# Look for: role: button, name: "Publish" — or multiple f-prefix refs
```

**Complete when:** `browser-use` shows "Publish Your App Now" / `playwright-cli` snapshot shows Publish button + iframe `f`-prefix refs.

---

## Workflow 4: Publish App

**browser-use:**
```bash
browser-use state | grep -i "publish"
browser-use click <publish-button-index>
sleep 2
browser-use state | grep -iE "confirm|publish your app|url"
browser-use click <confirm-publish-index>
sleep 15
browser-use state | grep -E "howone\.app"
# Or extract URL with:
browser-use eval "[...document.querySelectorAll('input')].map(i=>i.value).filter(v=>v.includes('howone'))"
```

**playwright-cli:**
```bash
playwright-cli snapshot
playwright-cli click <ref-of-publish-button>
playwright-cli snapshot
playwright-cli click <ref-of-confirm-button>
playwright-cli snapshot   # look for public URL in the snapshot
```

---

## Workflow 5: Use App Preview (iframe)

The app preview lives inside an iframe.

**browser-use** — interact via `eval` (cross-origin may block some actions):
```bash
browser-use state | grep -i iframe
browser-use eval "
const iframe = document.querySelector('iframe[title=\"Preview\"]');
iframe ? 'iframe found' : 'not found';
"
# For visible inputs, scroll into view and try clicking by index
browser-use scroll down
browser-use state | grep -iE "input|textarea|button"
```

**playwright-cli** — iframe elements get `f`-prefix refs automatically:
```bash
playwright-cli snapshot
# Elements inside iframe have refs like f3a2, f4b1
playwright-cli fill <f-ref-of-input> "Test input"
playwright-cli click <f-ref-of-submit>
playwright-cli snapshot   # verify response appeared
```

> **playwright-cli has better iframe support** — use it if available when interacting inside app previews.

---

## Workflow 6: Browse App Store

**browser-use:**
```bash
browser-use open https://howone.ai/apps
sleep 3
browser-use state | grep -iE "app|search|button" | head -20
browser-use input <search-index> "your search term"
browser-use keys "Enter"
browser-use state | grep -iE "view app|remix"
```

**playwright-cli:**
```bash
playwright-cli --state ~/.howone/session.json open https://howone.ai/apps
playwright-cli snapshot
playwright-cli fill <ref-of-search> "your search term"
playwright-cli click <ref-of-search-button>
playwright-cli snapshot
```

---

## Workflow 7: Save Results

```bash
mkdir -p output/<app-name>

# Screenshot (same for both tools)
browser-use screenshot output/<app-name>/screenshot.png
# OR
playwright-cli screenshot output/<app-name>/screenshot.png

# Save page state for reference
browser-use state > output/<app-name>/state.txt
# OR
playwright-cli snapshot > output/<app-name>/snapshot.yaml

# Results summary
cat > output/<app-name>/results.md << 'EOF'
# App Results
- **App Name**: <name>
- **Project URL**: https://howone.ai/project/<id>
- **Public URL**: https://howone.ai/apps/<id>
- **Prompt**: <prompt used>
- **Tool used**: browser-use / playwright-cli
- **Created**: <date>
EOF
```

---

## Session Management

| Action | browser-use | playwright-cli |
|--------|------------|----------------|
| Save session | `browser-use cookies export ~/.config/browser-use/howone.json` | `playwright-cli state-save ~/.howone/session.json` |
| Load session | *(auto-loaded from cookies)* | `playwright-cli --state ~/.howone/session.json open <url>` |
| Clear session | `rm ~/.config/browser-use/howone.json` | `rm ~/.howone/session.json` |

---

## Error Recovery

| Symptom | browser-use fix | playwright-cli fix |
|---------|----------------|-------------------|
| Command not found | `export PATH="$HOME/.local/bin:$PATH"` | Reinstall: `npm install -g @anthropic-ai/playwright-cli@latest` |
| Element not found / index stale | Run `browser-use state` again | Run `playwright-cli snapshot` again |
| Still on login page | Re-login with `--headed`, re-export cookies | Re-login with `--headed`, re-run `state-save` |
| iframe content not accessible | Use `browser-use scroll` + retry, or switch to playwright-cli | Wait 5-10s, snapshot again — look for `f`-prefix refs |
| Generation >15 min, no progress | `browser-use open https://howone.ai/project/<id>` | `playwright-cli goto https://howone.ai/project/<id>` |
| Publish URL not appearing | Wait 15-30s longer, use `eval` to query inputs | Snapshot again after 15s |

---

## HowOne AI Agents

During generation, these agents collaborate (visible in left panel):

| Agent | Role |
|-------|------|
| **Ava** | Product Manager — plans app structure |
| **Gabriel** | Workflow Designer — creates agentic workflows |
| **Mia** | Frontend Designer — designs UI |
| **Noah** | Developer — implements code |
| **Olivia** | QA — tests the app |

---

## URL Reference

| Page | URL |
|------|-----|
| Dashboard | `https://howone.ai` |
| My Apps | `https://howone.ai` → click Dashboard |
| Project editor | `https://howone.ai/project/<id>` |
| App Store | `https://howone.ai/apps` |
| Public app | `https://howone.ai/apps/<id>` |
| Live app | `https://<id>-<hash>.howone.app` |

---

## References

- [references/browser-use-workflows.md](references/browser-use-workflows.md) — Detailed browser-use commands and debugging
- [references/playwright-workflows.md](references/playwright-workflows.md) — Detailed playwright-cli command sequences
- [references/prompt-guide.md](references/prompt-guide.md) — Prompt templates for HowOne app generation
- [references/workflow-types.md](references/workflow-types.md) — HowOne workflow patterns and node types
