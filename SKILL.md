---
name: awt
description: AI-powered E2E web app testing with self-healing DevQA loop. Generate test scenarios from URLs or natural language, execute with Playwright, auto-fix failures. Supports YAML scenarios, vision AI matching, pattern learning. Use for QA, bug detection, regression testing.
---

# AWT — AI-Powered E2E Testing

AWT (AI Watch Tester) is an AI-powered DevQA loop orchestrator. It scans websites, generates test scenarios, executes them with Playwright, and auto-fixes failures — repeating until all tests pass.

## When to Use This Skill

- User wants to **test a web application** (E2E, QA, regression)
- User needs to **generate test scenarios** from a URL, spec document, or natural language description
- User wants to **write or validate YAML test scenarios** for AWT
- User mentions **bug detection**, **test automation**, or **UI testing**
- User asks about **self-healing tests** or **auto-fix on test failure**
- User wants to set up a **DevQA loop** (test → analyze → fix → retest)

## When NOT to Use This Skill

- Unit testing or API-only testing (AWT is for UI/E2E)
- Performance/load testing (use k6, Artillery, etc.)
- Mobile native app testing (AWT supports web and desktop only)
- Static code analysis without UI interaction

## 5-Step Workflow

```
1. Scan    → Crawl site, analyze DOM, detect features (login, cart, search...)
2. Plan    → AI generates YAML test scenarios (no code required)
3. Review  → User reviews/edits scenarios before execution
4. Execute → Playwright runs tests with live screenshots, human-like interaction
5. Heal    → Failed? AI analyzes cause → suggests fix → creates PR → re-tests
```

The loop repeats (up to 10 iterations by default) until all tests pass or the user stops it.

## Quick Start

```bash
# 1. Initialize project
aat init --name my-project --url https://mysite.com

# 2. Set AI provider API key (choose one)
aat config set ai.provider claude
export AAT_AI__API_KEY=sk-ant-...          # Claude
# export AAT_AI__API_KEY=sk-...            # OpenAI
# aat config set ai.provider ollama        # Ollama — no API key needed, free & offline

# 3. Generate scenarios from a spec document
aat generate --from docs/spec.md

# 4. Validate scenarios
aat validate scenarios/

# 5. Run tests (single execution)
aat run scenarios/

# 6. Run DevQA loop (auto-heal on failure)
aat loop scenarios/ --approval-mode manual

# 7. Interactive guided mode (all steps in one command)
aat start
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `aat init` | Initialize project (`.aat/`, `scenarios/`, config) |
| `aat config show` | Display current configuration |
| `aat config set <key> <value>` | Set config value (e.g., `ai.provider openai`) |
| `aat validate <path>` | Validate YAML scenario files |
| `aat run <path>` | Execute test scenarios (single run) |
| `aat loop <path>` | Run DevQA loop with auto-healing |
| `aat analyze <file>` | AI-analyze a spec document |
| `aat generate --from <file>` | Generate scenarios from spec |
| `aat start` | Interactive guided mode |
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

## YAML Scenario Format

```yaml
id: "SC-001"
name: "User Login"
description: "Test login with valid credentials"
tags: ["auth", "login"]
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

### Actions (19 types)

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
  provider: "claude"          # claude | openai | deepseek | ollama
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

## AI Providers

| Provider | Vision | Structured Output | Cost | Offline |
|----------|--------|-------------------|------|---------|
| Claude | Yes | Yes | Medium | No |
| OpenAI | Yes (GPT-4o) | Yes | Higher | No |
| DeepSeek | No | Partial | Low | No |
| Ollama | No | No | Free | Yes |

## Reference Files

- `references/scenario-schema.md` — Full YAML schema with all fields and validators
- `references/cli-reference.md` — Complete CLI command reference
- `references/config-reference.md` — All configuration options
- `templates/scenario-template.yaml` — Blank scenario template
- `templates/config-template.yaml` — Default config template
