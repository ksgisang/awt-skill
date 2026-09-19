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
| `--no-learn` | — | `false` | Neither use nor update remembered coordinates in this run |
| `--report` | — | none | Write a report per scenario: `pdf` or `markdown` |

- `--report pdf` writes `reports/<scenario id>/report.pdf` (and the `report.html`
  it was printed from). Screenshots of failed and warned steps are embedded in
  the file, so the report can be sent on its own. A run whose steps all passed
  but carries a warning is titled `PASS WITH WARNINGS`, never `PASS`.
- A report that cannot be written is reported on stderr and never changes the
  exit code: the code reflects the test, not the paperwork.
- Exit code: 0 = all pass, 1 = failed, 2 = critical failure, 3 = warnings only
- A click that changed nothing on screen is reported as `WARNING`, not `PASSED`:
  it ran, but it almost certainly missed its target. Read the screenshot for
  that step before calling the run good — exit code 3 exists so this cannot
  pass silently in a pipeline.
- `--skill-mode` outputs `=== AWT SKILL DEVQA ===` block on failure for AI parsing
- Tracks attempt count across runs (resets on success or different scenario)

**The approval gate.** Without `--skill-mode`, `aat run` shows the scenario and
waits for a keypress before opening the browser — Enter to run, `e` to edit the
YAML, `n` to cancel (exit 0). The prompt reads `/dev/tty`, not stdin, so piping
input at it does nothing; only the person at the terminal can answer. There is no
flag that turns the gate off. `--skill-mode` moves it rather than removes it: the
terminal prompt does not appear, because approval is taken to have happened when
the user approved your tool call. That holds only if you actually showed the
scenario and got a real "yes" first — the audit line records
`approval_method: skill` either way, so a run you never asked about is
indistinguishable from one you did. Every attempt, approved or cancelled, is
appended to `.aat/audit.log`.

The DEVQA block carries these fields:

| Field | Meaning |
|---|---|
| `SCENARIO` | Scenario file that failed |
| `FAILED_STEP` | Step number and action |
| `ERROR` | The step's own `message`, i.e. what the author expected |
| `ACTUAL_CAUSE` | What actually went wrong — present only when it differs from `ERROR` |
| `SCREENSHOT` | Path to the failure screenshot |
| `URL`, `PAGE_TITLE` | Where the browser was when it failed |
| `CATEGORY` | Failure class (`element_not_found`, `timeout`, `auth_error`, ...) |
| `POSSIBLE_CAUSE` | Generic hint derived from `CATEGORY` |
| `CRITICAL_FAILURE`, `EFFECT` | Present when a critical step stopped the run |
| `FIX_TARGET`, `RETRY_CMD`, `ATTEMPTS` | What to edit, how to re-run, how many tries so far |

**Diagnose from `ACTUAL_CAUSE` when it is present.** `ERROR` is a label the
scenario author wrote before the run, so taking it as the cause can invert the
diagnosis entirely.

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
| `--report-format` | — | `markdown` | Loop report format: `markdown` or `pdf` |

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

### `aat learn reset`
Forget the coordinates AWT remembered for a target.

```bash
aat learn reset "채점"      # one target, by name or selector
aat learn reset --all      # every remembered coordinate
aat learn --reset "채점"    # same thing, as an option on the group
```

A remembered position is only a fallback for targets the scenario did not name
with a selector, but a stale one keeps a step clicking an empty spot. Reset it
after the UI moves. Targets whose position follows the content — modal buttons,
choice overlays drawn on an image, list rows — are better marked
`learn: false` on the step so nothing is remembered in the first place.

### `aat learned list`
List all learned element mappings, including remembered coordinates
(target, page state, position, confidence, use count).

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
