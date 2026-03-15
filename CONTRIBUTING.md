# Contributing to AWT Skill

Thank you for your interest in contributing!

## How to Contribute

### Reporting Issues

- Use [GitHub Issues](https://github.com/ksgisang/awt-skill/issues)
- Include: what you expected, what happened, steps to reproduce
- Mention which AI coding tool you're using (Claude Code, Cursor, etc.)

### Suggesting Improvements

- Open an issue with the `enhancement` label
- Describe the use case and why it would help

### Pull Requests

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/my-improvement`
3. Make your changes
4. Test with at least one AI coding tool (e.g., Claude Code)
5. Submit a PR with a clear description

### What to Contribute

- **SKILL.md improvements** — Better instructions, more examples, clearer triggers
- **Reference docs** — Additional schema details, edge cases, tips
- **Templates** — New scenario templates for common patterns (auth, CRUD, e-commerce)
- **Translations** — SKILL.md in other languages

### New AI Provider Adapters — PRs Welcome!

We actively welcome community contributions for new AI provider adapters. Currently supported: Claude, OpenAI, Gemini, DeepSeek, Ollama.

**Wanted adapters:**
- **Grok (xAI)** — OpenAI-compatible API at `api.x.ai`
- **Mistral** — OpenAI-compatible API at `api.mistral.ai`
- **Cohere** — Command R+
- **Any OpenAI-compatible provider** — Usually 30 lines of code (see `deepseek.py` or `gemini.py` as examples)

**How to add a new adapter:**
1. Create `src/aat/adapters/your_provider.py` — inherit from `OpenAIAdapter`, override `__init__` and `_call_api`
2. Register in `src/aat/adapters/__init__.py` — add to `ADAPTER_REGISTRY`
3. Add pricing in `src/aat/core/cost.py` — add to `PRICING` dict
4. Add connection test in `src/aat/core/connection.py`
5. Add to `src/aat/cli/commands/setup_cmd.py` — `PROVIDERS` list
6. Submit PR to [AI-Watch-Tester](https://github.com/ksgisang/AI-Watch-Tester)

Most OpenAI-compatible providers need only ~30 lines (see `src/aat/adapters/gemini.py` as a minimal example).

### What NOT to Change

- Don't add AWT core code here (that belongs in [AI-Watch-Tester](https://github.com/ksgisang/AI-Watch-Tester))
- Don't add runtime scripts that execute code (skills should be documentation-first)

## Code Style

- Markdown files: ATX headings (`#`), clear structure
- YAML templates: 2-space indentation
- Keep SKILL.md under 5K tokens

## License

By contributing, you agree that your contributions will be licensed under AGPL-3.0.
