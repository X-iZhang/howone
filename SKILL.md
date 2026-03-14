---
name: howone
description: "Invoke whenever 'HowOne' or 'howone.ai' appears in the user's message — regardless
  of task type or language. Automates all HowOne AI platform operations: login (Google/GitHub/Email),
  creating AI apps from prompts, monitoring generation, publishing apps, browsing the App Store,
  testing app previews in iframes, and session recovery. Uses browser-use (preferred) or
  playwright-cli for browser control. Not for other platforms or generic browser tasks."
---

# HowOne Automation Skill

Automate [HowOne AI](https://howone.ai/) — an agentic app builder that turns natural language prompts into full web apps with AI-powered workflows.

## Getting Started

### 1. Check which browser tool you have

```bash
browser-use --help 2>/dev/null && echo "✓ browser-use" || echo "✗ browser-use"
playwright-cli --help 2>/dev/null && echo "✓ playwright-cli" || echo "✗ playwright-cli"
```

**Prefer `browser-use`** — it handles HowOne's shadow DOM elements better.

### 2. Install one if needed

```bash
# browser-use (recommended, Python 3.11+)
pipx install 'browser-use[cli]'
export PATH="$HOME/.local/bin:$PATH"

# OR playwright-cli (fallback, Node.js)
npm install -g @anthropic-ai/playwright-cli@latest
playwright-cli install chromium
```

### 3. Login (first time only)

OAuth requires a visible browser window.

```bash
# browser-use
browser-use --headed open https://howone.ai
# Complete login in browser, then:
mkdir -p ~/.config/browser-use
browser-use cookies export ~/.config/browser-use/howone.json

# OR playwright-cli
playwright-cli --headed open https://howone.ai
# Complete login in browser, then:
mkdir -p ~/.howone
playwright-cli state-save ~/.howone/session.json
```

### 4. Write a detailed prompt, then submit it

HowOne generates apps entirely from the prompt you submit. **The prompt quality directly determines app quality.** When a user gives you a brief instruction like "make me a task manager", you must expand it into a detailed, structured prompt before submitting to HowOne.

#### Prompt Writing Rules

1. **Always expand the user's idea** — a one-line request should become a multi-sentence prompt
2. **Specify data flow**: what users input → what AI processes → what output appears
3. **List concrete features**: don't say "with some features", list each one
4. **Describe the UI**: mention layout preferences (dashboard, cards, sidebar, tabs)
5. **Name the target user**: "for small business owners", "for students", "for teams"
6. **Include AI behavior**: what should the AI agent inside the app actually do?

#### Prompt Template

```
Create a [type] application for [target users].

Users can: [list all user actions — upload, enter, select, etc.]
The AI should: [list all AI behaviors — analyze, generate, recommend, etc.]

Features:
- [Feature 1 with specific details]
- [Feature 2 with specific details]
- [Feature 3 with specific details]

UI: [layout preferences — dashboard, sidebar, cards, tabs, etc.]
Output format: [charts, tables, PDF export, reports, etc.]
```

#### Examples

**User says:** "make me a pet hospital app"
**You submit to HowOne:**
```
Create a pet hospital management application for veterinary clinics.

Users can: register pets (name, species, breed, age, owner contact), log visits
with symptoms and diagnoses, schedule appointments, and record medications.
The AI should: generate health reports summarizing a pet's visit history,
flag overdue vaccinations, suggest follow-up appointments based on diagnosis,
and detect abnormal patterns across visits.

Features:
- Pet profiles with photo upload and medical history timeline
- Appointment calendar with drag-and-drop scheduling
- Medication tracker with dosage reminders
- Dashboard showing today's appointments, pending follow-ups, and alerts
- Owner communication panel for sending visit summaries via email
- Search by pet name, owner name, or diagnosis

UI: Dashboard layout with sidebar navigation, cards for pet profiles,
calendar view for appointments, table view for visit history.
Output format: PDF health reports, printable prescription slips.
```

**User says:** "帮我做一个分析销售数据的工具"
**You submit to HowOne:**
```
Create a sales data analysis application for business managers.

Users can: upload CSV/Excel files containing sales records (date, product,
quantity, revenue, region, salesperson), select date ranges, and filter by
product category or region.
The AI should: identify top-performing products, detect sales trends and
seasonality, compare salesperson performance, flag anomalies (sudden drops
or spikes), and generate monthly/quarterly summary reports.

Features:
- File upload supporting CSV and Excel formats
- Interactive charts: line chart for trends, bar chart for comparisons,
  pie chart for category distribution
- Drill-down from summary to individual transactions
- Automatic insight generation (top 3 findings highlighted)
- Export reports to PDF with charts included

UI: Dashboard with filter sidebar, chart area in center, data table below.
Output format: Interactive charts, downloadable PDF reports.
```

#### After writing the prompt, submit it:

```bash
# browser-use
browser-use open https://howone.ai
sleep 3
browser-use state | grep textarea
browser-use click <textarea-index>
browser-use type "<your detailed prompt>"
browser-use keys "Enter"

# OR playwright-cli
playwright-cli --state ~/.howone/session.json open https://howone.ai
playwright-cli snapshot
playwright-cli fill <ref> "<your detailed prompt>"
playwright-cli click <generate-ref>
```

Generation takes **5–15 minutes**. Poll every 60–90 seconds until "Publish Your App Now" appears.

---

## Command Quick Reference

| What you want to do | browser-use | playwright-cli |
|---------------------|-------------|----------------|
| Open a page | `browser-use open <url>` | `playwright-cli --state ~/.howone/session.json open <url>` |
| See page elements | `browser-use state` | `playwright-cli snapshot` |
| Click something | `browser-use click <index>` | `playwright-cli click <ref>` |
| Type text | `browser-use type "text"` | `playwright-cli fill <ref> "text"` |
| Press a key | `browser-use keys "Enter"` | `playwright-cli press <ref> Enter` |
| Take a screenshot | `browser-use screenshot out.png` | `playwright-cli screenshot out.png` |
| Run JavaScript | `browser-use eval "code"` | `playwright-cli evaluate "code"` |
| Save login session | `browser-use cookies export ~/.config/browser-use/howone.json` | `playwright-cli state-save ~/.howone/session.json` |

### How to find elements

**browser-use** — `state` gives numbered elements. Use the number:
```
[15]<textarea id=app-prompt />   →   browser-use click 15
```

**playwright-cli** — `snapshot` gives YAML with hex refs. Use the ref:
```yaml
- role: textbox, name: "Describe...", ref: e4a2   →   playwright-cli click e4a2
```

Elements change on every page load — always re-read before acting.

---

## Core Workflows

### Publish an app
```bash
# browser-use
browser-use state | grep -i publish
browser-use click <publish-index>
browser-use click <confirm-index>
browser-use eval "[...document.querySelectorAll('input')].map(i=>i.value).filter(v=>v.includes('howone'))"

# OR playwright-cli
playwright-cli click <publish-ref>
playwright-cli click <confirm-ref>
playwright-cli snapshot   # look for public URL
```

### Test app preview (iframe)
```bash
# playwright-cli is better for iframes (f-prefix refs)
playwright-cli snapshot
playwright-cli fill <f-ref> "test input"
playwright-cli click <f-ref-submit>

# browser-use alternative
browser-use scroll down
browser-use state | grep -iE "input|button"
```

### Browse App Store
```bash
browser-use open https://howone.ai/apps
browser-use state | grep -iE "search|app"
```

### Save results
```bash
mkdir -p output/<app-name>
browser-use screenshot output/<app-name>/screenshot.png
browser-use state > output/<app-name>/state.txt
```

---

## Important Rules

1. **Re-read elements before every action** — indices and refs change on page load
2. **`--headed` only for login** — everything else runs headless
3. **Wait 3–5 seconds after navigation** before reading the page
4. **Poll every 60–90 seconds** during generation — don't rapid-fire
5. **Use playwright-cli for iframes** — it can directly interact with `f`-prefix refs inside app previews

## Error Recovery

| Problem | Fix |
|---------|-----|
| `command not found` | `export PATH="$HOME/.local/bin:$PATH"` or reinstall the tool |
| Element not found | Re-run `state` / `snapshot` — refs are stale |
| Still on login page | Session expired — re-login with `--headed` |
| No iframe content | Wait 5–10s, re-snapshot — look for `f`-prefix refs |
| Generation stuck >15 min | Refresh: `browser-use open https://howone.ai/project/<id>` |

## HowOne URLs

| Page | URL |
|------|-----|
| Dashboard | `https://howone.ai` |
| Project | `https://howone.ai/project/<id>` |
| App Store | `https://howone.ai/apps` |
| Published app | `https://howone.ai/apps/<id>` |
| Live app | `https://<id>-<hash>.howone.app` |

## Deep-Dive References

For detailed command sequences, prompt templates, and workflow patterns:
- [browser-use-workflows.md](https://raw.githubusercontent.com/X-iZhang/howone/main/references/browser-use-workflows.md) — full browser-use commands with debugging tips
- [playwright-workflows.md](https://raw.githubusercontent.com/X-iZhang/howone/main/references/playwright-workflows.md) — full playwright-cli workflows with UI map
- [prompt-guide.md](https://raw.githubusercontent.com/X-iZhang/howone/main/references/prompt-guide.md) — prompt templates for different app types
- [workflow-types.md](https://raw.githubusercontent.com/X-iZhang/howone/main/references/workflow-types.md) — HowOne workflow patterns and self-evolution
