# AWT — AI-Powered E2E Testing Skill

**Test smarter, fix faster.** AWT scans your website, generates test scenarios with AI, executes them with Playwright, and auto-fixes failures in a self-healing DevQA loop.

[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/Agent_Skills-Compatible-brightgreen.svg)](https://agentskills.io)

---

## What is AWT?

AWT (AI Watch Tester) is an Agent Skill that brings AI-powered E2E testing to your coding workflow. Instead of writing Playwright scripts manually, you describe tests in plain YAML or natural language — AWT handles the rest.

**Key capabilities:**
- **Scan** any website and auto-detect features (login forms, carts, search bars)
- **Generate** test scenarios from URLs, spec documents, or natural language
- **Execute** with Playwright + human-like mouse/keyboard interaction
- **Self-heal** — when tests fail, AI analyzes the cause, fixes the code, and re-tests
- **Learn** — successful matches are stored in SQLite, making tests faster over time

## Installation

### Claude Code (recommended)

```bash
npx skills add ksgisang/awt-skill -g
```

### Manual Installation

```bash
# Clone to your global skills directory
git clone https://github.com/ksgisang/awt-skill.git ~/.claude/skills/awt
```

### Per-Project Installation

```bash
# Clone to project-level skills directory
git clone https://github.com/ksgisang/awt-skill.git .claude/skills/awt
```

After installation, the skill is available as `/awt` in Claude Code and auto-triggers on testing-related prompts.

## Comparison with Other Testing Skills

| Feature | webapp-testing | playwright-skill | **AWT** |
|---------|---------------|-----------------|---------|
| Scenario format | Python scripts | Python scripts | **Declarative YAML** |
| No-code test creation | No | No | **Yes** |
| Natural language → test | No | No | **Yes** |
| Self-healing loop | No | No | **Yes (DevQA Loop)** |
| Auto-fix on failure | No | No | **Yes (AI → PR)** |
| Vision AI matching | No | No | **Yes (OpenCV + OCR)** |
| Pattern learning | No | No | **Yes (SQLite)** |
| Server lifecycle mgmt | Yes | Yes | **Yes (aat dashboard)** |
| Multiple AI providers | No | No | **Yes (4 providers)** |
| Human-like interaction | No | No | **Yes (Bezier mouse)** |
| Approval modes | No | No | **Yes (manual/branch/auto)** |
| Cost optimization | N/A | N/A | **$0.02–0.05/test** |
| Live screenshots | No | Screenshot only | **Yes (WebSocket stream)** |
| Multi-language OCR | No | No | **Yes (10+ languages)** |
| Offline support | No | No | **Yes (Ollama)** |

### When to use each:
- **webapp-testing** — Quick one-off Playwright scripts for simple page checks
- **playwright-skill** — When you need fine-grained Playwright API control
- **AWT** — When you want AI-driven test generation, self-healing, and a complete DevQA pipeline

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
| DeepSeek | No | Low | No |
| Ollama | No | Free | Yes |

## Project Structure

```
awt-skill/
├── SKILL.md              # Main skill definition (loaded by AI agents)
├── references/
│   ├── scenario-schema.md  # Full YAML schema reference
│   ├── cli-reference.md    # CLI command reference
│   └── config-reference.md # Configuration options
├── templates/
│   ├── scenario-template.yaml  # Blank scenario template
│   └── config-template.yaml    # Default config template
├── README.md
├── LICENSE
├── CONTRIBUTING.md
└── package.json
```

## Requirements

AWT skill provides guidance and scenario generation. To execute tests, you need:

- **Python 3.11+**
- **AWT CLI** — `pip install aat-devqa` (or clone [AI-Watch-Tester](https://github.com/ksgisang/AI-Watch-Tester))
- **Playwright** — `playwright install chromium`
- **Tesseract OCR** — `brew install tesseract` (macOS) / `apt install tesseract-ocr` (Linux)

## Links

- **Main Repository:** [github.com/ksgisang/AI-Watch-Tester](https://github.com/ksgisang/AI-Watch-Tester)
- **Cloud Demo:** [ai-watch-tester.vercel.app](https://ai-watch-tester.vercel.app)
- **Agent Skills Standard:** [agentskills.io](https://agentskills.io)

## License

[AGPL-3.0](LICENSE) — see LICENSE file for full text.

Built by [AILoopLab](https://github.com/ksgisang).
