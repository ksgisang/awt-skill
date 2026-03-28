---
name: awt
version: 1.5.0
description: AI-powered E2E web app testing with self-healing DevQA loop. Generate test scenarios from URLs or natural language, execute with Playwright, auto-fix failures. Supports YAML scenarios, visual matching (OpenCV + OCR), pattern learning. Use for QA, bug detection, regression testing.
---

# AWT — Eyes and Hands for Your AI Coding Tool

AWT (AI Watch Tester) gives your AI coding tool the ability to **see and interact with web applications**. Your AI designs the test strategy; AWT executes it with a real browser — clicking, typing, taking screenshots, and reporting back.

## When to Use This Skill

- User wants to **test a web application** (E2E, QA, regression)
- User says **"test it"**, **"check if it works"**, **"verify the login"**
- User needs to **detect bugs** after code changes
- User wants **automated regression testing**
- E2E test failed and user wants to **find the root cause in source code**

## When NOT to Use

- Unit testing or API-only testing (AWT is for UI/E2E)
- Performance/load testing (use k6, Artillery)
- Mobile native app testing (web and desktop only)

## ⛔ CRITICAL RULES — READ BEFORE DOING ANYTHING

1. **NEVER use `-y` or `--auto-approve`** — the user MUST approve before any test runs.
2. **NEVER use `aat devqa`** — it runs the entire pipeline without user checkpoints. Always use the 4-step workflow below.
3. **NEVER run a test without showing the scenario to the user first** and getting explicit approval (e.g. "진행해", "go ahead", "yes", "run it").
4. **NEVER auto-fix code or scenarios without user permission** — always report failures and ask the user what to do.
5. **NEVER set `headless: true`** — users MUST see the browser.
6. **NEVER guess element names** — always scan first and use real data from `scan_result.json`.

---

## ★ MANDATORY 4-STEP WORKFLOW ★

When the user asks to test anything, you MUST follow these 4 steps **in order**. Do NOT skip any step. Do NOT combine steps.

### STEP 1: SCAN — Analyze the target site

```bash
aat scan --url <URL>
```

After scanning, **read `.aat/scan_result.json`** and present a summary to the user:

```
"I scanned http://localhost:3000 and found 83 interactive elements:
 - 5 input fields (email, password, search, ...)
 - 12 buttons (Login, Sign Up, Submit, ...)
 - 15 links (Home, About, Dashboard, ...)
 
 Should I create a test scenario based on these elements?"
```

**⏸️ WAIT for user response before proceeding to Step 2.**

### STEP 2: GENERATE + PRESENT — Create scenario and show it

Write a YAML scenario using the exact data from `scan_result.json`. Then **show the full scenario to the user**:

```
"Here is the test scenario I created (5 steps):

 1. 🌐 Navigate to http://localhost:3000
 2. 🖱  Type 'test@example.com' into email field
 3. 🖱  Type 'password123' into password field  
 4. 🖱  Click 'Login' button (critical)
 5. ✅ Verify URL contains '/dashboard'

 Should I run this test? Or would you like me to modify anything?"
```

**⏸️ WAIT for user approval before proceeding to Step 3.**
- If user says "modify X" → edit the scenario and show it again
- If user says "go ahead" / "진행해" / "yes" → proceed to Step 3
- If user says "cancel" → stop entirely

### STEP 3: EXECUTE — Run the approved scenario

```bash
aat run --skill-mode --fast scenarios/<scenario>.yaml
```

Monitor the output. **If any step fails:**

1. **STOP immediately** — do not continue to the next scenario
2. **Read the failure details** from the AWT output
3. **Read the SCREENSHOT** if provided in the `=== AWT SKILL DEVQA ===` block
4. **Report to the user:**

```
"Test failed at Step 3 (Click Login button):
 - Error: Element 'Login' not found in DOM
 - Possible cause: The button text might be 'Sign In' instead of 'Login'
 - Screenshot: .aat/screenshots/step_003_fail.png
 
 Should I:
 (a) Fix the scenario (change 'Login' to 'Sign In') and re-test?
 (b) Fix the source code instead?
 (c) Skip this step and continue?"
```

**⏸️ WAIT for user instruction before fixing or re-running.**

### STEP 4: REPORT — Summarize results

When all steps pass, report to the user:

```
"✅ Test complete: 5/5 steps passed (37.8 seconds)
 
 All navigation links and login flow are working correctly."
```

---

## Auto-Setup

```bash
which aat || (pip install aat-devqa && playwright install chromium)
```

---

## Scenario YAML Reference

### Writing Scenarios from scan_result.json

**Read `.aat/scan_result.json`** and write YAML using the exact element data:

```yaml
id: "SC-001"
name: "Login Flow"
steps:
  - step: 1
    action: navigate
    value: "http://localhost:3000/login"
    description: "Go to login page"

  # Use EXACT coordinates/selectors from scan_result.json
  - step: 2
    action: find_and_type
    target:
      selector: "#email"           # from scan_result.json
      text: "Email"
    value: "test@example.com"
    description: "Enter email"

  - step: 3
    action: find_and_click
    target:
      text: "Login"                # from scan_result.json
    region: main                   # exclude nav panel
    critical: true                 # stop if login fails
    description: "Click login button"

  - step: 4
    action: assert_url
    value: "/dashboard"
    on_fail: stop
    message: "Login failed — not redirected to dashboard"
    description: "Verify redirect to dashboard"
```

**Rules:**
- Use `selector` from scan data when available (most reliable)
- Use `text` as fallback (OCR-based)
- Use `region: main` for all clicks (avoid nav panel False Positives)
- Mark login/auth steps as `critical: true`
- Add `assert_url` after navigation-triggering clicks

### Actions (21 types)

| Category | Actions |
|----------|---------|
| Navigation | `navigate`, `go_back`, `refresh` |
| Find + Mouse | `find_and_click`, `find_and_double_click`, `find_and_right_click` |
| Find + Keyboard | `find_and_type`, `find_and_clear` |
| Direct | `click_at`, `type_text` (supports `verify: true`), `press_key`, `key_combo` |
| Assert | `assert`, `assert_text`, `assert_screen_changed`, `assert_url` |
| Session | `save_session`, `load_session` |
| Utility | `wait`, `screenshot`, `scroll` |

### Key Step Options

```yaml
- action: find_and_click
  target:
    text: "Submit"               # OCR text
    selector: "#submit-btn"      # CSS selector (highest priority)
  region: main                   # top/bottom/left/right/center/main/full
  critical: true                 # stop test on failure
  on_fail: stop                  # same as critical
  method: auto                   # auto/semantics/template/ocr/vision
  match_index: 0                 # 0=first match, -1=last
  change_threshold: 0.05         # for critical auto-verification
  description: "Click submit"
```

### assert_url (login/navigation verification)

```yaml
- action: assert_url
  value: "/dashboard"
  on_fail: stop
  message: "Login failed"
  description: "Verify redirect"
```

### Session Reuse

```yaml
# Save after login
- action: save_session
  name: "my_app_login"

# Load in next run (24h expiry)
- action: load_session
  name: "my_app_login"
```

### Region Parameter

| Region | Area | Use When |
|--------|------|----------|
| `full` | Entire screen (default) | General use |
| `main` | Right 80% | **Always use for clicks** (avoids nav panel) |
| `top` | Top 30% | Header elements |
| `bottom` | Bottom 30% | Footer, floating buttons |
| `center` | Central 60%x60% | Modal dialogs |

---

## Flutter CanvasKit Support

AWT automatically detects Flutter CanvasKit and activates Semantics:

1. After `navigate`, clicks `flt-semantics-placeholder` (3 retries, 3s each)
2. Reads `flt-semantics[aria-label]` for element coordinates
3. Falls back to OCR if Semantics unavailable

**Matching priority on Flutter:**
CSS selector → Flutter Semantics → Playwright text → OCR → Vision AI

**Flutter-specific rules:**
- Always use `region: main` (Canvas OCR picks up nav text)
- Use `verify: true` on `type_text` (Canvas input may not render)
- Add `assert_screen_changed` after clicks
- Use `method: semantics` to force Semantics lookup

---

## Source Code Root Cause Analysis

When a test fails, trace to the source code:

1. **Read test output** — which step, what error, what URL
2. **Search codebase** — grep for the expected text, URL route, component
3. **Read the component** — find why it doesn't render/redirect/respond
4. **Propose a fix to the user** — show a concrete diff, ask for approval
5. **After user approves** — apply fix, re-build, re-scan, re-test

**Key principle:** NEVER auto-apply fixes. Always show the diff and ask.

---

## CLI Commands

| Command | Description |
|---------|-------------|
| `aat scan --url URL` | Scan page, collect elements to scan_result.json |
| `aat run --skill-mode PATH` | Execute with structured output for AI |
| `aat run --skill-mode --fast PATH` | Execute in fast DOM-only mode (Next.js, React, etc.) |
| `aat run --debug PATH` | Execute with OCR candidate debug logs |
| `aat doctor` | Check environment |
| `aat setup` | Configure AI + Vision providers |
| `aat validate PATH` | Validate YAML scenarios |
| `aat cost` | View AI API costs |

### Key Flags

```
--skill-mode    Structured output for AI assistants
--fast          DOM-only matching (skip Vision/OCR — fastest for standard web apps)
--debug         Show OCR candidates and matcher details
--strict        Treat skipped steps as failures
--learn         Record healed steps for pattern learning
--slow-mo N     Slow down actions by N ms
```

### ⛔ Banned Flags (NEVER use these)

```
-y / --auto-approve    Bypasses user approval — NEVER USE
```

---

## Best Practices

1. **Always scan before writing scenarios** — never guess element names
2. **Use `region: main`** on all `find_and_click` — prevents nav panel False Positives
3. **Mark auth steps `critical: true`** — no point testing after login fails
4. **Add `assert_url` after form submits** — verify navigation happened
5. **Use `save_session`/`load_session`** — skip login on repeated runs
6. **Read screenshots on failure** — the SCREENSHOT path is a real PNG file
7. **One fix per retry** — change only the failing step
8. **Always ask user before fixing** — never auto-modify code or scenarios

## AI Providers (for Vision AI Tier 3)

| Provider | Vision | Cost | Setup |
|----------|--------|------|-------|
| Gemini Flash | Yes | Free | `aat setup` → Gemini |
| Claude | Yes | Medium | `aat setup` → Claude |
| GPT-4o | Yes | Higher | `aat setup` → OpenAI |

Vision AI is optional — Tier 1 (template) + Tier 2 (OCR) are free and work without API keys.

## Reference Files

- `references/scenario-schema.md` — Full YAML schema
- `references/cli-reference.md` — Complete CLI reference
- `references/config-reference.md` — All configuration options
- `templates/scenario-template.yaml` — Blank scenario template
- `templates/config-template.yaml` — Default config template
