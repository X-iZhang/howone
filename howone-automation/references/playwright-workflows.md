# Playwright CLI Workflows for HowOne

Detailed step-by-step command sequences for each HowOne operation using `playwright-cli`.

## Reading Snapshots

`playwright-cli snapshot` returns YAML describing all visible elements. Key concepts:

- **`ref`**: A short identifier (e.g., `e5a3`, `f12b`) used to target elements in commands
- **`e`-prefix refs**: Elements in the main page
- **`f`-prefix refs**: Elements inside iframes (e.g., app preview)
- **Roles**: `button`, `link`, `textbox`, `heading`, `img`, etc.
- **Names**: Text content or aria-label of the element

Example snapshot fragment:
```yaml
- role: textbox
  name: "Describe the application you want to create"
  ref: e4a2
- role: button
  name: "Generate"
  ref: e5b3
```

To interact: `playwright-cli fill e4a2 "my prompt"` then `playwright-cli click e5b3`.

**Rule**: Never reuse refs from a previous snapshot. Always take a fresh snapshot before each action.

---

## Workflow 1: First-Time Setup & Login

### Prerequisites
```bash
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium
mkdir -p ~/.howone
```

### Login with Google/GitHub OAuth
```bash
# Step 1: Open headed browser (OAuth needs visible window)
playwright-cli --headed open https://howone.ai

# Step 2: Find login button
playwright-cli snapshot
# Look for: role: button, name containing "Log in" or "Sign in" or "Get Started"

# Step 3: Click login
playwright-cli click <login-ref>

# Step 4: Snapshot to find OAuth buttons
playwright-cli snapshot
# Look for: "Continue with Google" or "Continue with GitHub"

# Step 5: Click OAuth provider
playwright-cli click <google-or-github-ref>
# Complete OAuth in the visible browser window (manual step)

# Step 6: Wait for redirect back to howone.ai, then save session
playwright-cli wait https://howone.ai
playwright-cli state-save ~/.howone/session.json
```

### Login with Email+Code
```bash
playwright-cli --headed open https://howone.ai
playwright-cli snapshot
playwright-cli click <login-ref>
playwright-cli snapshot
# Find email input
playwright-cli fill <email-input-ref> "your@email.com"
playwright-cli click <continue-ref>
# Wait for code email, then:
playwright-cli snapshot
playwright-cli fill <code-input-ref> "123456"
playwright-cli click <verify-ref>
playwright-cli state-save ~/.howone/session.json
```

---

## Workflow 2: Create App from Prompt

```bash
# Step 1: Load session and navigate to dashboard
playwright-cli --state ~/.howone/session.json open https://howone.ai

# Step 2: Find the prompt input
playwright-cli snapshot
# Look for: role: textbox, name containing "describe" or "create" or "application"

# Step 3: Enter the prompt
playwright-cli fill <textarea-ref> "Create an application where users can upload CSV files with sales data, and the AI can analyze trends, identify top products, and generate monthly reports with charts."

# Step 4: Find and click generate button
playwright-cli snapshot
# Look for: role: button, name containing "Generate" or "Create" or arrow/submit icon

# Step 5: Click generate
playwright-cli click <generate-ref>

# Step 6: Monitor generation (5-15 minutes)
# The URL will change to howone.ai/project/<id>
# Poll every 60-90 seconds:
playwright-cli snapshot
# Look for:
#   - Agent activity: "Ava is planning...", "Gabriel is designing..."
#   - Progress bar or step indicators
#   - Preview iframe appearing (signals completion)
#   - "Publish" button appearing (signals completion)

# Step 7: Repeat snapshot until generation completes
# When you see the preview iframe or all agents show "completed":
playwright-cli snapshot
# Extract project ID from the URL bar or page content
```

### Generation Completion Indicators
- URL contains `/project/<uuid>`
- Preview iframe is visible in snapshot (elements with `f`-prefix refs)
- "Publish" button appears
- All agent avatars show idle/complete status
- Chat panel shows "Your app is ready" or similar

---

## Workflow 3: Use App (Interact with Preview)

The app preview runs inside an iframe. Its elements have `f`-prefix refs.

```bash
# Step 1: Navigate to the project
playwright-cli --state ~/.howone/session.json open https://howone.ai/project/<id>

# Step 2: Wait for iframe to load, then snapshot
playwright-cli snapshot
# iframe elements appear with f-prefix refs, e.g.:
#   - role: textbox, ref: f3a2
#   - role: button, name: "Submit", ref: f4b1
#   - role: img, ref: f5c3

# Step 3: Interact with app inside iframe
playwright-cli fill <f-textbox-ref> "Test input data"
playwright-cli click <f-submit-ref>

# Step 4: Check results
playwright-cli snapshot
# Look for new elements: generated text, images, charts, etc.
```

---

## Workflow 4: Browse App Store

```bash
# Step 1: Navigate to App Store
playwright-cli --state ~/.howone/session.json open https://howone.ai/apps

# Step 2: View available apps
playwright-cli snapshot
# Apps appear as cards with name, description, author
# Look for: role: link or role: article with app names

# Step 3: Search (if search box exists)
playwright-cli snapshot
playwright-cli fill <search-ref> "data analysis"
playwright-cli click <search-button-ref>

# Step 4: Click an app to view details
playwright-cli snapshot
playwright-cli click <app-card-ref>

# Step 5: View app details
playwright-cli snapshot
# Shows: description, screenshots, "Use" or "Remix" buttons
```

---

## Workflow 5: Publish App

```bash
# Step 1: Navigate to your project
playwright-cli --state ~/.howone/session.json open https://howone.ai/project/<id>

# Step 2: Find Publish button
playwright-cli snapshot
# Look for: role: button, name containing "Publish" or "Deploy"

# Step 3: Click Publish
playwright-cli click <publish-ref>

# Step 4: Handle publish dialog (if any)
playwright-cli snapshot
# May show: visibility options, description fields, confirm button
# Fill any required fields, then confirm

playwright-cli click <confirm-ref>

# Step 5: Get public URL
playwright-cli snapshot
# Look for the public URL in the page content:
#   - howone.ai/apps/<id>
#   - <id>-<hash>.howone.app
```

---

## Workflow 6: Save Results Locally

```bash
# Create output directory
mkdir -p output/<app-name>

# Take full-page screenshot
playwright-cli screenshot output/<app-name>/screenshot.png

# Save snapshot for reference
playwright-cli snapshot > output/<app-name>/snapshot.yaml

# If app generates images (e.g., inside iframe):
# 1. Snapshot to find img elements
playwright-cli snapshot
# 2. Look for: role: img, with src attributes in the YAML
# 3. Download each image:
curl -o output/<app-name>/image1.png "<src-url>"

# Create results markdown
cat > output/<app-name>/results.md << EOF
# <App Name>

- **Project URL**: https://howone.ai/project/<id>
- **Public URL**: https://howone.ai/apps/<id>
- **Live URL**: https://<id>-<hash>.howone.app
- **Created**: $(date -Iseconds)
- **Prompt**: <the prompt used>

## Screenshots
![Screenshot](screenshot.png)

## Generated Images
![Image 1](image1.png)
EOF
```

---

## HowOne UI Map

### Dashboard (howone.ai, logged in)
- Top navigation: logo, search, notifications, user avatar
- Main area: prompt textarea ("Describe the application you want to create")
- Below prompt: "Generate" or arrow button
- Sidebar or below: recent projects list

### Project Page (howone.ai/project/<id>)
- Left panel: agent activity / chat
- Center: app preview iframe (`iframe[title="Preview"]`)
- Right panel: workflow editor (nodes and connections)
- Top bar: project name, Publish button, settings

### App Store (howone.ai/apps)
- Grid of app cards: thumbnail, name, description, author
- Search bar at top
- Category filters

---

## Error Recovery

### Session Expired
```bash
# Symptom: snapshot shows login page instead of dashboard
rm ~/.howone/session.json
playwright-cli --headed open https://howone.ai
# Complete login flow, then:
playwright-cli state-save ~/.howone/session.json
```

### Element Not Found
```bash
# Symptom: click/fill fails with "element not found"
# Solution: take fresh snapshot and search for the element
playwright-cli snapshot
# The ref numbers have changed — find the element by its text/role
```

### Generation Stuck
```bash
# Symptom: no progress after 15+ minutes
# Check agent status:
playwright-cli snapshot
# If agents show errors, try:
playwright-cli click <retry-ref>  # if retry button exists
# Or refresh:
playwright-cli goto https://howone.ai/project/<id>
```

### iframe Not Loading
```bash
# Symptom: no f-prefix refs in snapshot
# Wait 5-10 seconds for iframe to initialize:
playwright-cli snapshot  # try again after a brief wait
# If still missing, the app may still be generating
```
