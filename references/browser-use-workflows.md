# browser-use Workflows for HowOne

Complete step-by-step command sequences for HowOne operations using browser-use CLI.

## Key Concepts

### State Output Format

`browser-use state` returns indexed elements:

```
viewport: 894x804
page: 894x804
scroll: (0, 0)
[15]<textarea id=app-prompt />
[16]<button aria-label=Generate />
[17]<div>Welcome to HowOne</div>
```

- Numbers in brackets `[15]` are element indices
- Use these indices in `click`, `input` commands
- Indices change when page updates — always run `state` before acting

### Finding Elements

```bash
# Find textarea
browser-use state | grep textarea

# Find buttons
browser-use state | grep button

# Find by aria-label
browser-use state | grep "aria-label"

# Find by text content
browser-use state | grep -i "generate\|publish\|login"
```

### Text Input Strategy

1. **Method 1**: Click + Type (recommended for chat inputs)
   ```bash
   browser-use click <textarea-index>
   browser-use type "your text"
   browser-use keys "Enter"
   ```

2. **Method 2**: Input command
   ```bash
   browser-use input <textarea-index> "your text"
   browser-use keys "Enter"
   ```

---

## Complete Workflows

### Workflow 1: Full Login Flow

```bash
# Setup
export PATH="$HOME/.local/bin:$PATH"

# Open browser
browser-use --headed open https://howone.ai
sleep 3

# Check if already logged in
browser-use state | grep -i "hello\|dashboard"

# If not logged in, trigger login dialog
browser-use state | grep -i "login\|sign"
# If login button visible, click it
browser-use click <login-button-index>

# Fill email
browser-use state | grep -i "email\|input"
browser-use input <email-input-index> "your@email.com"
browser-use click <continue-button-index>

# Wait for verification email
sleep 10

# Enter code (replace with actual code)
browser-use state | grep -i "code\|verify"
browser-use input <code-input-index> "123456"
browser-use click <verify-button-index>

# Verify success
sleep 3
browser-use state | grep -i "hello"

# Save session
mkdir -p ~/.config/browser-use
browser-use cookies export ~/.config/browser-use/howone_cookies.json
```

### Workflow 2: Create App from Scratch

```bash
# Setup
export PATH="$HOME/.local/bin:$PATH"

# Open HowOne
browser-use --headed open https://howone.ai
sleep 3

# Find and click prompt textarea
browser-use state | grep textarea
browser-use click <first-textarea-index>

# Type app prompt
browser-use type "Create a student classroom pet management app. Features: pet selection (cat/dog/rabbit/hamster/bird), pet naming, feeding system, interaction (pet/play/train), cleaning, status display (mood/health/hunger/intimacy), growth system, daily check-in rewards, homework task rewards, leaderboard, teacher dashboard for class overview and task management."

# Submit
browser-use keys "Enter"

# Wait for project creation
sleep 5

# Get project URL
browser-use state | grep -iE "project|howone.ai" | head -5
```

### Workflow 3: Supplement Requirements During Generation

```bash
# Check current state
browser-use state | head -60

# Look for AI questions (e.g., "补充这个应用具体功能")
# Find chat textarea
browser-use state | grep textarea

# Click to focus
browser-use click <chat-textarea-index>

# Type additional requirements
browser-use type "Additional requirements: Student features include pet selection (cat/dog/rabbit/hamster/bird), naming, feeding, interaction, cleaning, status display. Classroom tasks include daily check-in, homework rewards. Leaderboard for intimacy/health/completion. Teacher features include class overview, task publishing, rewards, statistics."

# Submit
browser-use keys "Enter"

# Verify sent
sleep 3
browser-use state | grep -A5 "Joe"  # or your username
```

### Workflow 4: Monitor and Wait for Completion

```bash
# Poll function
check_ready() {
  export PATH="$HOME/.local/bin:$PATH"
  browser-use state 2>&1 | grep -qE "ready|Publish Your App Now"
}

# Poll every 60 seconds
while ! check_ready; do
  echo "Still building... $(date)"
  sleep 60
done

echo "Build complete!"
browser-use screenshot /tmp/app-complete.png
```

### Workflow 5: Publish and Get URL

```bash
# Verify ready
browser-use state | grep -iE "ready|publish"

# Click Publish button (usually in top right area)
browser-use state | grep -i "Publish"
browser-use click <publish-button-index>

# Wait for dialog
sleep 2

# Find and click confirm publish button
browser-use state | grep -iE "Publish Your App|button.*Publish"
browser-use click <confirm-publish-index>

# Wait for publication
sleep 10

# Extract URL
browser-use state | grep -E "https://.*howone\.app"

# Alternative: use eval to extract URL
browser-use eval "
const urlElement = [...document.querySelectorAll('a, span, div')].find(e => e.textContent?.match(/howone\.app/));
urlElement?.textContent || 'URL not found';
"
```

### Workflow 6: Handle Multiple Projects/Tabs

```bash
# List sessions
browser-use sessions

# If multiple tabs open
browser-use switch 0  # Go to first tab
browser-use state | grep -iE "StudyPets|App Name"  # Check which app

browser-use switch 1  # Go to second tab
browser-use state | grep -iE "StudyPets|App Name"  # Check which app

# Close current tab (if needed)
browser-use close

# Or close specific tab by switching first
```

### Workflow 7: Resume Session After Restart

```bash
# If you saved cookies previously
export PATH="$HOME/.local/bin:$PATH"

# Open with session (browser-use auto-loads cookies)
browser-use --headed open https://howone.ai

# Verify still logged in
browser-use state | grep -i "hello\|dashboard"

# If session expired, re-login (see Workflow 1)
```

---

## Debugging Tips

### Screenshot Current State

```bash
browser-use screenshot /tmp/debug.png
```

### View Full Page HTML

```bash
browser-use eval "document.body.innerHTML.substring(0, 5000)"
```

### Find All Interactive Elements

```bash
browser-use eval "
[...document.querySelectorAll('button, a, input, textarea, [role=button]')]
  .filter(e => e.offsetParent !== null)
  .map((e, i) => ({tag: e.tagName, type: e.type, text: e.textContent?.substring(0,50), aria: e.getAttribute('aria-label')}))
  .slice(0, 20)
"
```

### Check if Element is Visible

```bash
browser-use eval "
const el = document.querySelectorAll('textarea')[0];
el && el.offsetParent !== null ? 'visible' : 'hidden or not found';
"
```

### Force Click with JavaScript

```bash
browser-use eval "
document.querySelector('button[aria-label=\"Publish\"]')?.click();
'clicked';
"
```

---

## Quick Reference Card

```bash
# Always start with this
export PATH="$HOME/.local/bin:$PATH"

# Open
browser-use --headed open https://howone.ai

# See page
browser-use state | head -40

# Find input
browser-use state | grep textarea

# Type
browser-use click <index> && browser-use type "text" && browser-use keys "Enter"

# Check progress
browser-use state | grep -iE "ready|publish|building"

# Publish
browser-use click <publish-index> && sleep 2 && browser-use click <confirm-index>

# Get URL
browser-use state | grep "howone.app"

# Save session
browser-use cookies export ~/.config/browser-use/howone_cookies.json
```
