---
name: howone-automation
description: "Invoke whenever 'HowOne' or 'howone.ai' appears in the user's message — regardless
  of task type or language. Automates all HowOne AI platform operations: login (Google/GitHub/Email),
  creating AI apps from prompts, monitoring generation, publishing apps, browsing the App Store,
  testing app previews in iframes, and session recovery. Uses browser-use (preferred) or
  playwright-cli for browser control. Not for other platforms or generic browser tasks."
---

# HowOne Automation — Detailed Reference

This is the full reference for automating [HowOne AI](https://howone.ai/). For a quick-start version, see the root [`SKILL.md`](../SKILL.md).

---

## Tool Detection & Install

```bash
# Detect
browser-use --help 2>/dev/null && echo "✓ browser-use" || echo "✗ browser-use"
playwright-cli --help 2>/dev/null && echo "✓ playwright-cli" || echo "✗ playwright-cli"
```

Prefer **browser-use** (more stable indices, better shadow DOM support). Install if missing:

```bash
# browser-use (Python 3.11+)
pipx install 'browser-use[cli]'
export PATH="$HOME/.local/bin:$PATH"

# playwright-cli (Node.js)
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium
```

---

## Complete Command Reference

| Action | `browser-use` | `playwright-cli` |
|--------|--------------|-----------------|
| Open page (headless) | `browser-use open <url>` | `playwright-cli --state ~/.howone/session.json open <url>` |
| Open page (headed) | `browser-use --headed open <url>` | `playwright-cli --headed open <url>` |
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

### How Elements Work

**`browser-use`** — `state` returns numbered elements:
```
[15]<textarea id=app-prompt />
[16]<button aria-label=Generate />
```
→ `browser-use click 16`, `browser-use state | grep textarea`

**`playwright-cli`** — `snapshot` returns YAML with hex refs:
```yaml
- role: textbox
  name: "Describe the application..."
  ref: e4a2
```
→ `playwright-cli click e4a2` — find elements by `role` + `name`, never reuse old refs

---

## Critical Rules

1. **Always re-read elements before acting** — indices/refs change on every page load
2. **`--headed` only for OAuth login** — everything else runs headless
3. **Wait 3–5 seconds after navigation** before reading elements
4. **Generation takes 5–15 minutes** — poll every 60–90 seconds, do not rapid-fire
5. **Verify after every action** — re-read page state to confirm it worked

---

## Workflow 1: Login

OAuth requires `--headed`. Three methods: Google OAuth, GitHub OAuth, Email+Code.

**browser-use:**
```bash
export PATH="$HOME/.local/bin:$PATH"
browser-use --headed open https://howone.ai
sleep 3
browser-use state | grep -iE "login|sign|google|github|email"
browser-use click <login-button-index>
# Email+Code flow:
browser-use input <email-index> "your@email.com"
browser-use click <continue-index>
browser-use input <code-index> "123456"
browser-use click <verify-index>
browser-use state | grep -i "hello"   # verify success
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
playwright-cli snapshot   # verify: dashboard, not login page
mkdir -p ~/.howone
playwright-cli state-save ~/.howone/session.json
```

**Success:** Page shows your name (e.g. "Hello, Xi") or a prompt textarea — not a login button.

---

## Workflow 2: Create App from Prompt

**browser-use:**
```bash
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

**Generation stages:**

| Stage | browser-use sees | playwright-cli sees |
|-------|-----------------|-------------------|
| Starting | "Generating info..." | agent names visible |
| In progress | "Our AI agents are building..." | agent messages |
| Complete | "Publish Your App Now" | Publish button + f-prefix refs |

---

## Workflow 3: Monitor Generation (5–15 minutes)

```bash
# browser-use
browser-use state | grep -iE "ready|Publish Your App Now|Completed"
browser-use screenshot /tmp/progress.png

# playwright-cli
playwright-cli snapshot
# Look for: role: button, name: "Publish" — or f-prefix refs
```

**Done when:** "Publish Your App Now" appears or Publish button + iframe `f`-prefix refs are visible.

---

## Workflow 4: Publish App

**browser-use:**
```bash
browser-use state | grep -i "publish"
browser-use click <publish-button-index>
sleep 2
browser-use click <confirm-publish-index>
sleep 15
browser-use eval "[...document.querySelectorAll('input')].map(i=>i.value).filter(v=>v.includes('howone'))"
```

**playwright-cli:**
```bash
playwright-cli click <ref-of-publish-button>
playwright-cli snapshot
playwright-cli click <ref-of-confirm-button>
playwright-cli snapshot   # look for public URL
```

---

## Workflow 5: Use App Preview (iframe)

The app preview is inside an iframe. **playwright-cli is better here** — iframe elements get `f`-prefix refs.

**playwright-cli (recommended for iframes):**
```bash
playwright-cli snapshot
# Elements inside iframe: refs like f3a2, f4b1
playwright-cli fill <f-ref-of-input> "Test input"
playwright-cli click <f-ref-of-submit>
playwright-cli snapshot
```

**browser-use (alternative):**
```bash
browser-use state | grep -i iframe
browser-use eval "
const iframe = document.querySelector('iframe[title=\"Preview\"]');
iframe ? 'iframe found' : 'not found';
"
browser-use scroll down
browser-use state | grep -iE "input|textarea|button"
```

---

## Workflow 6: Browse App Store

```bash
# browser-use
browser-use open https://howone.ai/apps
sleep 3
browser-use state | grep -iE "app|search|button" | head -20
browser-use input <search-index> "your search term"
browser-use keys "Enter"

# playwright-cli
playwright-cli --state ~/.howone/session.json open https://howone.ai/apps
playwright-cli snapshot
playwright-cli fill <ref-of-search> "your search term"
playwright-cli click <ref-of-search-button>
```

---

## Workflow 7: Save Results

```bash
mkdir -p output/<app-name>
browser-use screenshot output/<app-name>/screenshot.png
browser-use state > output/<app-name>/state.txt

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
| Save | `browser-use cookies export ~/.config/browser-use/howone.json` | `playwright-cli state-save ~/.howone/session.json` |
| Load | *(auto-loaded)* | `playwright-cli --state ~/.howone/session.json open <url>` |
| Clear | `rm ~/.config/browser-use/howone.json` | `rm ~/.howone/session.json` |

---

## Error Recovery

| Symptom | browser-use fix | playwright-cli fix |
|---------|----------------|-------------------|
| Command not found | `export PATH="$HOME/.local/bin:$PATH"` | `npm install -g @anthropic-ai/playwright-cli@latest` |
| Element not found | Run `browser-use state` again | Run `playwright-cli snapshot` again |
| Still on login page | Re-login `--headed`, re-export cookies | Re-login `--headed`, re-run `state-save` |
| iframe inaccessible | `browser-use scroll` + retry, or use playwright-cli | Wait 5–10s, re-snapshot for `f`-prefix refs |
| Generation stuck >15 min | `browser-use open https://howone.ai/project/<id>` | `playwright-cli goto https://howone.ai/project/<id>` |
| Publish URL missing | Wait 15–30s, `eval` to query inputs | Snapshot again after 15s |

---

## HowOne UI Map

### Dashboard (howone.ai)
- **Top**: Nav bar — logo, search, notifications, avatar
- **Center**: Large prompt textarea
- **Below**: Generate button, recent projects

### Project Page (howone.ai/project/\<id\>)
- **Left panel**: Agent chat (Ava, Gabriel, Mia, Noah, Olivia)
- **Center**: App preview iframe (`f`-prefix refs)
- **Right panel**: Workflow editor (nodes + connections)
- **Top bar**: Project name, Publish button

### App Store (howone.ai/apps)
- **Top**: Search bar, category filters
- **Main**: Grid of app cards with View/Remix buttons

---

## URL Reference

| Page | URL |
|------|-----|
| Dashboard | `https://howone.ai` |
| Project editor | `https://howone.ai/project/<id>` |
| App Store | `https://howone.ai/apps` |
| Public app | `https://howone.ai/apps/<id>` |
| Live app | `https://<id>-<hash>.howone.app` |
