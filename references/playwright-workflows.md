# Playwright CLI Workflows for HowOne

Step-by-step command sequences for all HowOne operations. This is the primary command reference.

## How Snapshots Work

`playwright-cli snapshot` returns YAML describing every visible element on the page.

Each element has:
- **`ref`**: Short ID (e.g., `e5a3`) — use this in commands like `click`, `fill`
- **`role`**: Element type — `button`, `link`, `textbox`, `heading`, `img`, etc.
- **`name`**: Visible text or aria-label

Ref prefixes:
- **`e`-prefix** (e.g., `e5a3`): Main page elements
- **`f`-prefix** (e.g., `f19e17`): Elements inside iframes (app preview)

Example snapshot fragment:
```yaml
- role: textbox
  name: "Describe the application you want to create"
  ref: e4a2
- role: button
  name: "Generate"
  ref: e5b3
```

To use: `playwright-cli fill e4a2 "my prompt"` then `playwright-cli click e5b3`.

**The pattern for every action:**
1. `playwright-cli snapshot` — read the YAML
2. Find the element by its `name` or `role` — note its `ref`
3. Act: `playwright-cli click <ref>` or `playwright-cli fill <ref> "text"`
4. `playwright-cli snapshot` — verify the action worked

---

## Workflow 1: First-Time Login

OAuth requires a visible browser. Use `--headed`.

### Google/GitHub OAuth
```bash
# 1. Open headed browser
playwright-cli --headed open https://howone.ai

# 2. Snapshot → find "Log in" or "Get Started" button
playwright-cli snapshot

# 3. Click login button
playwright-cli click <ref-of-login-button>

# 4. Snapshot → find "Continue with Google" or "Continue with GitHub"
playwright-cli snapshot

# 5. Click OAuth provider
playwright-cli click <ref-of-oauth-button>

# 6. MANUAL STEP: Complete OAuth in the visible browser window
#    Wait for redirect back to howone.ai dashboard

# 7. Verify login succeeded — snapshot should show dashboard, NOT login page
playwright-cli snapshot

# 8. Save session
mkdir -p ~/.howone
playwright-cli state-save ~/.howone/session.json
```

### Email + Code
```bash
playwright-cli --headed open https://howone.ai
playwright-cli snapshot
playwright-cli click <ref-of-login-button>

# Find and fill email input
playwright-cli snapshot
playwright-cli fill <ref-of-email-textbox> "your@email.com"
playwright-cli click <ref-of-continue-button>

# Wait for email, then enter code
playwright-cli snapshot
playwright-cli fill <ref-of-code-textbox> "123456"
playwright-cli click <ref-of-verify-button>

# Verify + save
playwright-cli snapshot
mkdir -p ~/.howone
playwright-cli state-save ~/.howone/session.json
```

**How to verify login succeeded:** Snapshot should show a prompt textarea ("Describe the application you want to create") or a dashboard with recent projects. If you still see a "Log in" button, login failed.

---

## Workflow 2: Create App from Prompt

```bash
# 1. Load session, open dashboard
playwright-cli --state ~/.howone/session.json open https://howone.ai

# 2. Wait for page load, then snapshot
#    (wait 3-5 seconds after open)
playwright-cli snapshot

# 3. Find prompt textarea — look for role: textbox with name like
#    "Describe the application..." or "What do you want to build..."
playwright-cli fill <ref-of-textarea> "Create a pet hospital management app that records pet care details and generates health reports with one click"

# 4. Snapshot → find submit button (may be "Generate", "Create", or an arrow icon)
playwright-cli snapshot
playwright-cli click <ref-of-generate-button>

# 5. Wait 5 seconds for redirect, then snapshot
#    URL should change to howone.ai/project/<id>
playwright-cli snapshot
```

### Monitoring Generation (5-15 minutes)

Poll every 60-90 seconds:
```bash
playwright-cli snapshot
```

**What to look for in each snapshot:**

| Stage | What you see in snapshot |
|-------|------------------------|
| Starting | Agent names (Ava, Gabriel, etc.) with activity status |
| In progress | Agent messages like "Planning app structure...", "Designing workflow..." |
| Nearly done | Preview iframe starts appearing (`f`-prefix refs) |
| Complete | "Publish" button visible, iframe fully loaded with app UI |

**Generation is complete when:**
- You see a `role: button` with `name` containing "Publish"
- OR you see multiple `f`-prefix refs (iframe content loaded)
- OR agent activity panel shows all agents idle/done

**If generation exceeds 15 minutes with no change:**
```bash
# Refresh the page
playwright-cli goto https://howone.ai/project/<id>
playwright-cli snapshot
```

---

## Workflow 3: Use App (Preview Iframe)

App previews run inside an iframe. All interactive elements inside it have **`f`-prefix refs**.

```bash
# 1. Navigate to the project
playwright-cli --state ~/.howone/session.json open https://howone.ai/project/<id>

# 2. Wait 5 seconds for iframe to load, then snapshot
playwright-cli snapshot

# 3. Find app elements — they all have f-prefix refs
#    Example: role: textbox, ref: f3a2 / role: button, name: "Submit", ref: f4b1

# 4. Interact with the app
playwright-cli fill <f-ref-of-input> "Test input: analyze this sales data"
playwright-cli click <f-ref-of-submit-button>

# 5. Wait 3-5 seconds for app to process, then verify
playwright-cli snapshot
# Look for: new text content, images, charts, tables in the f-prefix elements
```

**If no `f`-prefix refs appear:**
- The iframe hasn't loaded yet — wait 5-10 seconds and snapshot again
- If the app is still generating, wait for generation to complete first
- Try refreshing: `playwright-cli goto https://howone.ai/project/<id>`

---

## Workflow 4: Browse App Store

```bash
# 1. Open App Store
playwright-cli --state ~/.howone/session.json open https://howone.ai/apps

# 2. Snapshot to see app listings
playwright-cli snapshot
# Apps appear as cards — look for role: link or role: heading with app names

# 3. Search for a specific app (if search box exists)
playwright-cli fill <ref-of-search-box> "data analysis"
playwright-cli snapshot
playwright-cli click <ref-of-search-button>

# 4. Click an app card to view details
playwright-cli snapshot
playwright-cli click <ref-of-app-card>

# 5. View app detail page
playwright-cli snapshot
# Shows: description, screenshots, "Use" or "Remix" buttons
```

---

## Workflow 5: Publish App

```bash
# 1. Navigate to your project
playwright-cli --state ~/.howone/session.json open https://howone.ai/project/<id>

# 2. Find Publish button
playwright-cli snapshot
# Look for: role: button, name containing "Publish"

# 3. Click Publish
playwright-cli click <ref-of-publish-button>

# 4. Handle confirmation dialog (if one appears)
playwright-cli snapshot
# May show: visibility options, description field, confirm button
playwright-cli click <ref-of-confirm-button>

# 5. Verify publication — snapshot and look for:
#    - Public URL (howone.ai/apps/<id>)
#    - Live URL (<id>-<hash>.howone.app)
#    - Success message
playwright-cli snapshot
```

---

## Workflow 6: Save Results Locally

```bash
# Create output directory
mkdir -p output/<app-name>

# Screenshot the current page
playwright-cli screenshot output/<app-name>/screenshot.png

# Save snapshot as reference
playwright-cli snapshot > output/<app-name>/snapshot.yaml

# If the app generated images inside the iframe:
# 1. Look for role: img elements in the snapshot (f-prefix refs)
# 2. The snapshot may show src URLs — extract them
# 3. Download with curl:
curl -o output/<app-name>/image1.png "<image-src-url>"

# Create a results summary
cat > output/<app-name>/results.md << 'EOF'
# App Results
- **Project URL**: https://howone.ai/project/<id>
- **Public URL**: https://howone.ai/apps/<id>
- **Prompt**: <the prompt used>
- **Created**: <date>
EOF
```

---

## HowOne UI Map

### Dashboard (howone.ai)
- **Top**: Navigation bar — logo, search, notifications, user avatar
- **Center**: Large prompt textarea ("Describe the application you want to create")
- **Below prompt**: Generate / submit button (may be arrow icon)
- **Below**: Recent projects list

### Project Page (howone.ai/project/<id>)
- **Left panel**: Agent activity / chat messages (Ava, Gabriel, Mia, Noah, Olivia)
- **Center**: App preview iframe — this is where `f`-prefix refs live
- **Right panel**: Workflow editor canvas (nodes and connections)
- **Top bar**: Project name, Publish button, settings gear

### App Store (howone.ai/apps)
- **Top**: Search bar, category filters
- **Main**: Grid of app cards — thumbnail, name, description, author

---

## Common Pitfalls

| Pitfall | Prevention |
|---------|-----------|
| Using a ref from a previous snapshot | Always snapshot immediately before acting |
| Clicking too fast after page load | Wait 3-5 seconds after `open`/`goto` before snapshot |
| Polling generation too frequently | Wait 60-90 seconds between snapshots during generation |
| Trying to interact with iframe before it loads | Look for `f`-prefix refs; if none, wait and re-snapshot |
| Forgetting `--headed` for OAuth | Headless OAuth will hang — always use `--headed` for login |
| Forgetting `--state` flag | Without it, session won't load and you'll see the login page |
| Not saving session after login | Always `state-save` immediately after successful login |
