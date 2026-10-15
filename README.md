# AWT — Eyes and Hands for Your AI Coding Tool

**Your AI coding tool is smart. But it can't see or click a web page.** AWT gives it a real browser — so it can test, find bugs, and fix them without you lifting a finger.

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/Agent_Skills-Compatible-brightgreen.svg)](https://agentskills.io)

> **Works without AI API** — write YAML scenarios manually and run `aat run`. No API key needed. AI coding tools make it easier, but are not required.

---

## What is AWT?

AWT is the **execution engine** for AI-powered E2E testing. Your AI coding tool designs the tests; AWT runs them in a real browser with Playwright.

**How it works in Skill Mode (no extra AI API key needed):**

```
You: "Test the login flow on https://mysite.com"

Your AI coding tool:
  → Writes 5 YAML scenarios (30 steps)
  → Runs: aat run scenarios/
  → Reads failure: "Step 4: Dashboard text not visible"
  → Reads screenshot + source code
  → Fixes src/pages/login.js:23
  → Re-runs: aat run scenarios/
  → All 5 scenarios pass ✓
```

**Key capabilities:**
- **Execute** tests in a real browser with human-like mouse/keyboard interaction
- **See** — take screenshots, detect elements via OCR and image matching
- **Report** — step-by-step pass/fail with error details and screenshots
- **Self-heal** (CLI mode) — `aat loop` auto-fixes failures with its own AI
- **Learn** — successful matches stored in SQLite, getting faster over time

## Installation

Install the `aat` command first — every route below drives it:

```bash
pip install aat-devqa
python -m playwright install chromium
```

### Claude Code plugin (recommended)

This repository is a plugin bundle. It ships the `awt` skill **and** the MCP
server that exposes the `aat_*` tools, so one install gives you both.

```bash
claude plugin marketplace add ksgisang/awt-skill
claude plugin install ai-watch-tester@awt
```

The MCP server needs the Python MCP SDK:

```bash
pip install "mcp[cli]>=1.0.0"
```

Verify what got installed:

```bash
claude plugin details ai-watch-tester@awt
```

You should see `Skills (1) awt` and `MCP servers (1) awt`.

### Skill only

If you want the skill without the MCP server:

```bash
npx skills add ksgisang/awt-skill --skill awt -g
```

Or copy it by hand, globally or per project:

```bash
git clone https://github.com/ksgisang/awt-skill.git /tmp/awt-skill
cp -r /tmp/awt-skill/skills/awt ~/.claude/skills/awt   # or .claude/skills/awt
rm -rf /tmp/awt-skill
```

After installation, the skill auto-triggers on testing-related prompts.

### Updating

```bash
claude plugin update ai-watch-tester@awt          # plugin
npx skills add ksgisang/awt-skill --skill awt -g -y  # skill only
```

For hand-copied installations: `cd ~/.claude/skills/awt && git pull`

## Comparison with Other Testing Skills

| Feature | webapp-testing | playwright-skill | **AWT** |
|---------|---------------|-----------------|---------|
| Scenario format | Python scripts | Python scripts | **Declarative YAML** |
| No-code test creation | No | No | **Yes** |
| Natural language → test | No | No | **Yes** |
| Self-healing loop | No | No | **Yes (DevQA Loop)** |
| Auto-fix on failure | No | No | **Yes (AI → PR)** |
| Visual matching (OpenCV + OCR) | No | No | **Yes (OpenCV + OCR)** |
| Pattern learning | No | No | **Yes (SQLite)** |
| Server lifecycle mgmt | Yes | Yes | **Yes (aat dashboard)** |
| Multiple AI providers | No | No | **Yes (5 providers)** |
| Human-like interaction | No | No | **Yes (Bezier mouse)** |
| Approval modes | No | No | **Yes (manual/branch/auto)** |
| Cost optimization | N/A | N/A | **$0.02–0.05/test** |
| Live screenshots | No | Screenshot only | **Yes (WebSocket stream)** |
| Multi-language OCR | No | No | **Yes (10+ languages)** |
| Offline support | No | No | **Yes (Ollama)** |
| Cost tracking | No | No | **Yes (per-call logging + aat cost)** |
| Source code analysis | No | No | **Yes (Skill-exclusive)** |
| Scenario caching | No | No | **Yes (same spec = no re-call)** |
| Browser test overlay | No | No | **Yes (live step progress)** |
| Dependency ordering | No | No | **Yes (depends_on field)** |
| **Skill Mode (no extra AI cost)** | No | No | **Yes** |
| Canvas/Flutter OCR fallback | No | No | **Yes (auto)** |
| Platform auto-detection | No | No | **Yes (7 frameworks)** |
| Structured failure diagnosis | No | No | **Yes (AI-independent)** |
| Multi-document YAML | No | No | **Yes (--- separator)** |
| Strict validation | No | No | **Yes (aat validate --strict)** |

### When to use each:
- **webapp-testing** — Quick one-off Playwright scripts for simple page checks
- **playwright-skill** — When you need fine-grained Playwright API control
- **AWT Skill Mode** — Your AI coding tool designs tests, AWT executes them. **No extra AI API key needed.**
- **AWT CLI Mode** — Standalone automated testing with `aat generate` + `aat loop` for CI/CD

## Supported AI Coding Tools

AWT follows the [Agent Skills open standard](https://agentskills.io) and works with:

| Tool | Status |
|------|--------|
| Claude Code | Supported |
| Cursor | Compatible |
| Codex | Compatible |
| Gemini CLI | Compatible |
| Amp | Compatible |
| Cline | Compatible |
| Aider | Compatible |
| Windsurf | Compatible |
| Roo Code | Compatible |
| PearAI | Compatible |
| Antigravity | Supported |

## Quick Example

### YAML Scenario

```yaml
id: "SC-001"
name: "User Login"
tags: ["auth", "login"]
depends_on: ["SC-000"]
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
```

### Natural Language (in Claude Code)

> "Login to mysite.com with test@example.com, then check if the dashboard loads"

AWT converts this to a YAML scenario and executes it automatically.

## 5-Step DevQA Workflow

```
Scan → Plan → Review → Execute → Heal
  ↑                                 |
  └─── Loop back on failure ────────┘
```

1. **Scan** — Crawl site, analyze DOM, detect features
2. **Plan** — AI generates YAML test scenarios
3. **Review** — User reviews/edits before execution
4. **Execute** — Playwright runs with live screenshots
5. **Heal** — AI analyzes failure → suggests fix → re-tests

## AI Providers

| Provider | Vision | Cost | Offline |
|----------|--------|------|---------|
| Claude (default) | Yes | Medium | No |
| OpenAI (GPT-4o) | Yes | Higher | No |
| Gemini (default: free) | Yes | Free tier | Yes |
| DeepSeek | No | Low | No |
| Ollama | No | Free | Yes |

## Project Structure

```
awt-skill/                      # ← this repository is the plugin folder
├── .claude-plugin/
│   ├── plugin.json             # Plugin manifest
│   └── marketplace.json        # Marketplace entry for this one plugin
├── .mcp.json                   # Declares the bundled MCP server
├── skills/
│   └── awt/                    # ← Skill content (installed by npx skills)
│       ├── SKILL.md            # Main skill definition
│       ├── references/
│       │   ├── scenario-schema.md  # Full YAML schema reference
│       │   ├── cli-reference.md    # CLI command reference
│       │   └── config-reference.md # Configuration options
│       └── templates/
│           ├── scenario-template.yaml
│           └── config-template.yaml
├── mcp/
│   ├── server.py               # MCP server — the 12 aat_* tools
│   └── requirements.txt
├── server.json                 # MCP Registry entry
├── README.md
├── LICENSE
├── CONTRIBUTING.md
└── package.json
```

## Key CLI Commands

| Command | Description |
|---------|-------------|
| `aat doctor` | Check environment (Python, Playwright, Tesseract, AI) |
| `aat init` | Initialize project + AI setup + environment check |
| `aat setup` | Configure AI provider and API key |
| `aat generate` | AI-generate scenarios (with cost estimate + caching) |
| `aat run --learn` | Execute tests + learn from fixes (always use --learn) |
| `aat loop` | Self-healing DevQA loop |
| `aat cost` | View AI API usage costs |
| `aat validate --strict` | Validate YAML + quality checks |
| `aat learn platform -p <key> -t <tip>` | Add platform-specific tip |

## System Dependencies

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
choco install tesseract
pip install aat-devqa
playwright install chromium
```

After install, run `aat doctor` to verify everything works.

**OCR language packs.** Tesseract installs English only. If you test an app in
another language, install that language's pack too — otherwise text assertions
that fall back to OCR report `not visible on page` for text that is plainly
there. `aat doctor` names any missing pack.

```bash
brew install tesseract-lang                 # macOS, all languages
sudo apt install tesseract-ocr-kor          # Linux, one language
```

## What this plugin runs, sends, and fetches

Everything below is in this repository in readable source. Nothing is minified,
compiled, or downloaded at install time.

**Runs on your machine.** `mcp/server.py` starts under `python3` and does one
thing: it builds an argument list and runs the `aat` command you installed with
`pip install aat-devqa`, as a subprocess. It never runs a shell, so nothing it
receives is re-parsed as a command. `aat` in turn drives a local Chromium
through Playwright and, for OCR, the local `tesseract` binary.

**Writes to disk.** Under your project: `.aat/` (config, run history,
screenshots, the SQLite learning database, the append-only audit log) and
`reports/`. Under your home directory: `~/.awt/templates/<host>/` for the
element images AWT saves so it can find a target again after a selector breaks.

**Sends over the network.** Two destinations, both only when you ask for them:

- **The site you are testing.** Chromium loads the URL in your scenario. That
  is the whole point of the tool, and no other site is contacted by a test run.
- **An AI provider, only for the commands that need one** — `aat generate`,
  `aat loop`, `aat devqa`, and Vision matching. The request carries the page
  text or screenshot and your scenario. You choose the provider and supply the
  key; with Ollama the request never leaves your machine. Plain `aat run`
  contacts no AI provider at all.

**Fetches at runtime.** Nothing. The plugin downloads no code after install.

**Approval.** `aat run` asks for confirmation by reading `/dev/tty` directly,
so a piped stdin cannot answer for you, and every attempt is recorded in
`.aat/audit.log`. There is no `--auto-approve` flag and no way for an AI agent
to supply the approval on your behalf. Scenario edits that an AI proposes are
not written to your files in the default `manual` mode.

## Links

- **Main Repository:** [github.com/ksgisang/AI-Watch-Tester](https://github.com/ksgisang/AI-Watch-Tester)
- **Cloud Demo:** [ai-watch-tester.vercel.app](https://ai-watch-tester.vercel.app)
- **Agent Skills Standard:** [agentskills.io](https://agentskills.io)

## License

[AGPL-3.0](LICENSE) — see LICENSE file for full text.

Built by [AILoopLab](https://github.com/ksgisang).
