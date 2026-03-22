# AWT CLI Reference

Entry point: `aat` (installed via `pip install -e .`)

## Project Setup

### `aat init`
Initialize AWT project structure.

```bash
aat init [OPTIONS]
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--name` | `-n` | `aat-project` | Project name |
| `--source` | `-s` | `.` | Source code path |
| `--url` | `-u` | `""` | Application URL |

Creates: `.aat/` directory, `scenarios/` directory, `aat.config.yaml`

### `aat config show`
Display current merged configuration.

```bash
aat config show [--config PATH]
```

### `aat config set`
Set a configuration value.

```bash
aat config set <key> <value> [--config PATH]
```

Example: `aat config set ai.provider openai`

## Scenario Management

### `aat validate`
Validate YAML scenario files against schema.

```bash
aat validate <path>
```

- `path`: Single `.yaml` file or directory (scans recursively)
- Reports validation errors per file

### `aat analyze`
AI-analyze a specification document.

```bash
aat analyze <file_path> [--config PATH]
```

- Supported formats: `.md`, `.txt`
- Outputs: screens, elements, flows to `.aat/analysis/`

### `aat generate`
Generate YAML test scenarios from a document.

```bash
aat generate --from <file> [--config PATH] [--output DIR]
```

- AI reads document and produces scenario YAML files
- Default output: `scenarios/` directory

## Scanning

### `aat scan`
Scan a URL and collect UI elements for scenario authoring.

```bash
aat scan --url <URL> [OPTIONS]
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--url` | `-u` | required | URL to scan |
| `--compare` | — | — | Previous scan_result.json to diff against |
| `--config` | `-c` | auto-detect | Config file path |

Output: `.aat/scan_result.json` with elements (label, type, selector, x, y, source)

### `aat hook install`
Install post-commit git hook for auto-scan on UI file changes.

### `aat hook uninstall`
Remove AWT post-commit hook.

## Test Execution

### `aat run`
Execute scenarios once (no healing loop).

```bash
aat run <scenarios_path> [OPTIONS]
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--config` | `-c` | auto-detect | Config file path |
| `--slow-mo` | — | `100` (headed) | Slow down actions by N ms |
| `--learn` | — | `false` | Learn from fixes (record healed steps) |
| `--skill-mode` | — | `false` | Output structured diagnosis for AI coding assistants |
| `--debug` | — | `false` | Enable debug logging (OCR candidates, matcher details) |
| `--strict` | — | `false` | Treat skipped steps as failures (exit code 1) |

- Exit code: 0 = all pass, 1 = failed, 2 = critical failure
- `--skill-mode` outputs `=== AWT SKILL DEVQA ===` block on failure for AI parsing
- Tracks attempt count across runs (resets on success or different scenario)

### `aat loop`
Execute DevQA healing loop.

```bash
aat loop <scenarios_path> [OPTIONS]
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--config` | `-c` | auto-detect | Config file path |
| `--max-loops` | `-m` | `10` | Maximum iterations (1–100) |
| `--approval-mode` | `-a` | `manual` | `manual` / `branch` / `auto` |

**Approval Modes:**
- `manual` — Prompts in terminal, shows fix suggestion, no file changes
- `branch` — Creates `aat/fix-NNN` git branch, applies fix, commits, retests
- `auto` — Modifies source files directly, retests immediately

### `aat start`
Interactive guided mode — walks through entire workflow.

```bash
aat start [--config PATH]
```

Flow: Setup → Analyze → Generate → Test → Loop → Report

## Dashboard

### `aat dashboard` / `aat serve`
Launch web UI with live screenshots.

```bash
aat dashboard [--host HOST] [--port PORT] [--no-open]
```

Default: `http://127.0.0.1:8420`

## Learning

### `aat learn add`
Add a learned element mapping.

### `aat learned list`
List all learned element mappings.

### `aat learned clear`
Clear learned element database.

## Environment Variables

All config values can be set via environment variables with `AAT_` prefix and `__` delimiter:

```bash
export AAT_AI__PROVIDER=openai
export AAT_AI__API_KEY=sk-...
export AAT_AI__MODEL=gpt-4o
export AAT_ENGINE__HEADLESS=true
```
