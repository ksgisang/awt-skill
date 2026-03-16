---
name: awt
version: 1.1.0
description: AI-powered E2E web app testing with self-healing DevQA loop. Generate test scenarios from URLs or natural language, execute with Playwright, auto-fix failures. Supports YAML scenarios, visual matching (OpenCV + OCR), pattern learning. Use for QA, bug detection, regression testing.
---

# AWT — Eyes and Hands for Your AI Coding Tool

AWT (AI Watch Tester) gives your AI coding tool the ability to **see and interact with web applications**. Your AI designs the test strategy; AWT executes it with a real browser — clicking, typing, taking screenshots, and reporting back.

## When to Use This Skill

- User wants to **test a web application** (E2E, QA, regression)
- User needs to **generate test scenarios** from a URL, spec document, or natural language description
- User wants to **write or validate YAML test scenarios** for AWT
- User mentions **bug detection**, **test automation**, or **UI testing**
- User asks about **self-healing tests** or **auto-fix on test failure**
- User wants to set up a **DevQA loop** (test → analyze → fix → retest)
- E2E test failed and user wants to **find the root cause in source code** — trace from failed test to route/controller/component/query

## When NOT to Use This Skill

- Unit testing or API-only testing (AWT is for UI/E2E)
- Performance/load testing (use k6, Artillery, etc.)
- Mobile native app testing (AWT supports web and desktop only)
- Static code analysis without UI interaction

## Skill Mode vs CLI Mode

AWT works in two modes:

### Skill Mode (recommended — you are here)
Your AI coding tool is the **brain**. AWT is the **eyes and hands**.

- **Your AI** designs test scenarios, analyzes failures, fixes source code
- **AWT** executes tests in a real browser via `aat run`
- **No extra AI API key needed** — your AI coding tool's intelligence is used directly
- **Superior results** — your AI sees the full project context, screenshots, and source code

```
User: "Test the login flow on my app"
→ Your AI writes 5 scenarios (30 steps) as YAML
→ aat run --learn scenarios/ executes them with Playwright
→ Step 4 fails: "Dashboard text not visible"
→ Your AI reads the screenshot + source code
→ Your AI fixes src/pages/login.js
→ aat run --learn scenarios/ passes all steps ✓
```

### CLI Mode (standalone, without AI coding tool)
AWT handles everything independently using its own AI adapter.

- Requires separate AI API key (Claude, OpenAI, Gemini, DeepSeek, or Ollama)
- `aat generate` creates scenarios via AI, `aat loop` auto-heals
- Good for CI/CD pipelines and automated environments

**In Skill Mode, always prefer writing YAML scenarios directly and running `aat run`, over using `aat generate` or `aat loop`.** Your AI coding tool produces better scenarios because it understands the project context.

## 5-Step Workflow

```
1. Scan    → Crawl site, analyze DOM, detect features (login, cart, search...)
2. Plan    → AI generates YAML test scenarios (no code required)
3. Review  → User reviews/edits scenarios before execution
4. Execute → Playwright runs tests with live screenshots, human-like interaction
5. Heal    → Failed? AI analyzes cause → suggests fix → creates PR → re-tests
```

The loop repeats (up to 10 iterations by default) until all tests pass or the user stops it.

## Auto-Setup (for AI coding tools)

**IMPORTANT:** Before running any `aat` command, check if it exists. If not, install it automatically:

```bash
# Run this FIRST — if aat is not found, install everything needed
which aat || (pip install aat-devqa && playwright install chromium)
```

If `pip install` fails with "externally-managed-environment" (PEP 668), use:
```bash
pipx install aat-devqa && playwright install chromium
```

After install, run `aat doctor` to verify. Do NOT ask the user to install manually — handle it silently.

## Prerequisites (manual install)

If the `aat` command is not found, install it:

**macOS:**
```bash
brew install python@3.12 tesseract
pipx install aat-devqa
playwright install chromium
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt install python3.12 python3.12-venv tesseract-ocr
pipx install aat-devqa
playwright install chromium
```

**Windows:**
```bash
winget install Python.Python.3.12
pip install aat-devqa
playwright install chromium
```

**Troubleshooting:**
- `python3 --version` shows < 3.11? Install Python 3.12 first (see above)
- `pip install` blocked by PEP 668? Use `pipx install aat-devqa` instead
- `pipx` not found? `brew install pipx` (macOS) or `pip install --user pipx` (Linux)
- After install, run `aat doctor` to verify everything works

## Failure Diagnosis (AI-independent)

When `aat run` fails, it automatically outputs a **structured diagnosis** that does NOT depend on any AI:

```
📊 AWT Diagnosis (deterministic — no AI)
  Step:     4 — Verify dashboard loaded
  Error:    Text 'Welcome' not visible on page
  URL:      https://mysite.com/login    ← still on login, not dashboard
  Title:    Login Page
  Screenshot: .aat/screenshots/fail_step4.png
  Category: assertion_failed
  Investigation:
    □ Check if the content is loaded dynamically (add wait)
    □ Check the screenshot to see what's actually displayed
  Retest: aat run --learn scenarios/SC-001.yaml
```

**This ensures consistent analysis quality regardless of which AI coding tool you use.** Feed this diagnosis output to your AI — even a basic AI can fix the issue with this level of detail.

Before running scenarios, validate quality with:
```bash
aat validate scenarios/ --strict
# Warns: missing assertions, hardcoded URLs, empty descriptions
```

## Quick Start

```bash
# 0. Check environment (auto-runs during init, or run manually)
aat doctor

# 1. Initialize project (includes AI setup + environment check)
aat init --name my-project --url https://mysite.com
#    → Prompts: choose provider (Claude/OpenAI/Gemini/DeepSeek/Ollama) + enter API key
#    → Gemini and Ollama have generous free tiers
#    → API key stored locally only, auto-added to .gitignore

# 2. Re-configure AI provider anytime
aat setup

# 3. Generate scenarios (shows cost estimate, asks confirmation, caches result)
aat generate --from docs/spec.md

# 4. Validate scenarios
aat validate scenarios/

# 5. Run tests (single execution)
aat run --learn scenarios/

# 6. Run DevQA loop (auto-heal on failure)
aat loop scenarios/ --approval-mode manual

# 7. View AI usage costs
aat cost

# 8. Interactive guided mode (all steps in one command)
aat start
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `aat doctor` | Check environment: Python, Playwright, Tesseract, AI connection |
| `aat init` | Initialize project + AI setup + environment check |
| `aat setup` | Re-configure AI provider and API key anytime |
| `aat config show` | Display current configuration |
| `aat config set <key> <value>` | Set config value (e.g., `ai.provider openai`) |
| `aat validate <path>` | Validate YAML scenario files |
| `aat run --learn <path>` | Execute tests + learn from fixes (always use --learn) |
| `aat loop <path>` | Run DevQA loop with auto-healing |
| `aat analyze <file>` | AI-analyze a spec document |
| `aat generate --from <file>` | Generate scenarios from spec |
| `aat start` | Interactive guided mode |
| `aat learn platform -p <key> -t <tip>` | Add custom platform-specific tip |
| `aat cost` | View AI API usage costs (daily/monthly) |
| `aat dashboard` | Web UI with live screenshots |

### Key Options

```
aat loop scenarios/ \
  --config aat.config.yaml \    # Config file path
  --max-loops 10 \              # Max heal iterations (1-100)
  --approval-mode manual        # manual | branch | auto
```

**Approval Modes:**
- `manual` — Terminal prompt, no file changes (safest, default)
- `branch` — Git branch isolation (`aat/fix-001`), auto-commit, auto-retest
- `auto` — Direct file modification + retest (fastest, riskiest)

### Headed Mode Features

When running tests with `headless: false` (default), AWT provides:
- **Browser overlay** — real-time step progress bar at top of browser window (step name, pass/fail status, counter)
- **slowMo** — each Playwright action paused by 100ms (default) so you can see what's happening
  ```
  aat run --learn scenarios/ --slow-mo 200    # Slower for demos
  aat run --learn scenarios/ --slow-mo 0      # Disable slowMo
  ```

## YAML Scenario Format

```yaml
id: "SC-001"
name: "User Login"
description: "Test login with valid credentials"
tags: ["auth", "login"]
depends_on: ["SC-000"]          # Run after SC-000 passes (optional)
variables:
  url: "https://mysite.com"

steps:
  - step: 1
    action: navigate
    value: "{{url}}/login"
    description: "Go to login page"

  - step: 2
    action: find_and_type
    target:
      text: "Email"
      match_method: ocr
    value: "test@example.com"
    humanize: true
    description: "Enter email"

  - step: 3
    action: find_and_click
    target:
      text: "Login"
    description: "Click login button"

  - step: 4
    action: assert
    assert_type: text_visible
    expected:
      - type: text_visible
        value: "Welcome back"
    description: "Verify login success"

expected_result:
  - type: url_contains
    value: "/dashboard"
```

### Actions (16 types)

| Category | Actions |
|----------|---------|
| Navigation | `navigate`, `go_back`, `refresh` |
| Find + Mouse | `find_and_click`, `find_and_double_click`, `find_and_right_click` |
| Find + Keyboard | `find_and_type`, `find_and_clear` |
| Direct | `click_at`, `type_text`, `press_key`, `key_combo` |
| Assert | `assert` (types: `text_visible`, `text_equals`, `image_visible`, `url_contains`, `url_not_contains`, `screenshot_match`) |
| Utility | `wait`, `screenshot`, `scroll` |

### Target Matching

```yaml
target:
  text: "Login"              # OCR text matching
  image: "btn-login.png"     # Template image matching
  selector: "#login-btn"     # CSS selector (highest priority)
  match_method: ocr           # Force specific matcher
  confidence: 0.9             # Override threshold (default: 0.85)
```

**Matcher chain** (tried in order): `learned → template → ocr → feature`

## Configuration

Config file: `aat.config.yaml` (also supports env vars with `AAT_` prefix)

```yaml
project_name: "my-project"
url: "https://mysite.com"

ai:
  provider: "claude"          # claude | openai | gemini | deepseek | ollama
  model: "claude-sonnet-4-20250514"
  temperature: 0.3

engine:
  type: "web"                 # web | desktop
  browser: "chromium"         # chromium | firefox | webkit
  headless: false
  viewport_width: 1280
  viewport_height: 720

matching:
  confidence_threshold: 0.85
  ocr_languages: ["eng", "kor"]
  chain_order: ["learned", "template", "ocr", "feature"]

humanizer:
  enabled: true

max_loops: 10
approval_mode: "manual"
```

## Source Code Root Cause Analysis (Skill-Exclusive)

**This is the decisive advantage of using AWT as an Agent Skill** — unlike AWT Cloud, which can only see the browser, the skill runs inside your IDE where it has full access to your project source code.

When an E2E test fails, DO NOT just report the test failure. Follow this procedure:

### Step 1: Identify the failing behavior
From the `aat run` output, extract: which step failed, the action (navigate, click, assert), the error message, and the expected vs actual result.

### Step 2: Trace to source code
Based on the failing URL/action, search the project codebase for the responsible code:

```
Failing URL: /api/login → search for route handlers
  - Look for: routes/login, api/auth, controllers/auth, pages/login
  - Frameworks: Next.js (app/api/), Express (router.post), FastAPI (@app.post), Django (urls.py)

Failing assertion: "Welcome back" not visible → search for where that text is rendered
  - Grep for the expected text in templates, components, or API responses

Failing click: "Submit" button not found → search for the form component
  - Look for: <button>, <input type="submit">, onClick handlers
```

### Step 3: Analyze root cause
Read the identified source files and determine WHY the test failed:
- **Route not defined** → missing endpoint
- **Conditional rendering** → auth check failing, feature flag off
- **API returns error** → DB query failing, validation error
- **Wrong redirect** → incorrect router.push/redirect path
- **Text mismatch** → i18n key wrong, hardcoded vs dynamic text

### Step 4: Propose fix with code diff
Show a concrete before/after code change:

```
File: src/app/api/login/route.ts
Issue: Returns 401 instead of redirecting to /dashboard on success

// BEFORE
return NextResponse.json({ error: "Invalid" }, { status: 401 });

// AFTER
if (valid) {
  return NextResponse.redirect(new URL("/dashboard", request.url));
}
return NextResponse.json({ error: "Invalid" }, { status: 401 });
```

### Step 5: Verify fix scope
Check for side effects — does the fix break other tests? Search for other code that depends on the changed function/route.

### Example: Complete failure analysis

When user runs `aat run --learn scenarios/` and Step 4 fails with "text 'Welcome back' not visible":

1. **Read test output** — Step 4 assert failed on `/dashboard` page
2. **Search codebase** — `Grep for "Welcome back"` → found in `src/components/Dashboard.tsx:42`
3. **Read component** — Shows `{user?.name ? "Welcome back" : "Loading..."}`
4. **Trace auth flow** — `useAuth()` hook → `src/lib/auth.ts` → session cookie not set after login
5. **Find bug** — Login API at `src/app/api/login/route.ts` doesn't set `Set-Cookie` header
6. **Propose fix** — Add `cookies().set("session", token)` to login route
7. **Check impact** — Search for other routes using `cookies().get("session")` to verify compatibility

**Key principle:** The AI agent (you) can read ANY file in the project. Use Grep, Glob, and Read tools aggressively to trace from failed test → UI component → API route → database query until you find the root cause.

## Best Practices & Tips

**1. Real browser ≠ HTML source** — On i18n (multilingual) sites, the HTML source may show default-language text while the browser renders translated text. AWT tests in a real browser, so always write assertions based on **what the user actually sees on screen**, not what's in the HTML source.

**2. Multi-document YAML supported** — Put multiple scenarios in one file using `---` separator. AWT loads all documents from a single file. You can also use separate files if preferred.

**3. Use selector + text together** — When both a CSS selector and OCR text are available, specify both in the target. If OCR fails (font rendering, contrast), the selector provides a fallback:
```yaml
target:
  selector: "#login-btn"    # reliable fallback
  text: "Login"             # human-readable, OCR-matched
```

**4. Skill Mode needs no AI API** — In Skill Mode, your AI coding tool creates scenarios directly. You don't need `aat generate` or an AI API key. Just write YAML and run `aat run --learn scenarios/`.

**5. Dismiss popups first** — Popups, cookie banners, and modals can block tests. Add `press_key Escape` at the start of scenarios:
```yaml
- step: 1
  action: press_key
  value: "Escape"
  description: "Dismiss any popup/modal"
```

**6. Flutter/Canvas SPA** — Flutter Web renders text on Canvas, not DOM. AWT auto-falls back to OCR for `text_visible` assertions. For input fields, use `click_at` + `type_text` instead of `find_and_type` (Flutter creates hidden invisible inputs):
```yaml
# Instead of find_and_type (fails on Flutter):
- step: 2
  action: click_at
  value: "400,300"
  description: "Click the email field"
- step: 3
  action: type_text
  value: "user@example.com"
  description: "Type email"
```

**7. Scroll shortcuts** — Use `"down"`, `"up"`, `"down-far"`, `"up-far"` instead of coordinates:
```yaml
- step: 2
  action: scroll
  value: "down"
  description: "Scroll down 500px from center"
```

**8. click_at verification** — When using `click_at` with `screenshot_before: true` or `screenshot_after: true`, AWT compares before/after screenshots and warns if the click had no visible effect (possible miss).

**9. Platform auto-detection** — AWT automatically detects the frontend framework after the first `navigate` step and shows tailored tips:
- Flutter Web (CanvasKit/HTML), React, Next.js, Vue, Angular, Svelte
- Custom tips can be added: `aat learn platform -p react_spa -t "your tip"`

## AI Providers

| Provider | Vision | Structured Output | Cost | Offline |
|----------|--------|-------------------|------|---------|
| Claude | Yes | Yes | Medium | No |
| OpenAI | Yes (GPT-4o) | Yes | Higher | No |
| Gemini | Yes | Partial | Free tier | No |
| DeepSeek | No | Partial | Low | No |
| Ollama | No | No | Free | Yes |

## Reference Files

- `references/scenario-schema.md` — Full YAML schema with all fields and validators
- `references/cli-reference.md` — Complete CLI command reference
- `references/config-reference.md` — All configuration options
- `templates/scenario-template.yaml` — Blank scenario template
- `templates/config-template.yaml` — Default config template
