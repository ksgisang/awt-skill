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
| `--report-screenshots` | — | `failures` | Which steps the PDF illustrates: `failures` / `all` / `none` |

- `--report pdf` writes `reports/<scenario id>/report.pdf` (and the `report.html`
  it was printed from). Screenshots of failed and warned steps are embedded in
  the file, so the report can be sent on its own. A run whose steps all passed
  but carries a warning is titled `PASS WITH WARNINGS`, never `PASS`.
- `--report-screenshots all` embeds every step's screenshot, which is the way to
  show what worked: with the default a clean run's report has no images at all.
  `none` keeps the file small. Markdown links its screenshots rather than
  embedding them, so it ignores this option.
- A report that cannot be written is reported on stderr and never changes the
  exit code: the code reflects the test, not the paperwork.
- Exit code: 0 = all pass, 1 = failed, 2 = critical failure, 3 = warnings only,
  4 = **did not run** (approval needed a terminal and there was none). 0-3 are
  verdicts about a run that happened; 4 sits outside that range so a pipeline
  can tell "nothing was tested" from "everything passed"
- A click that changed nothing on screen is reported as `WARNING`, not `PASSED`:
  it ran, but it almost certainly missed its target. Read the screenshot for
  that step before calling the run good — exit code 3 exists so this cannot
  pass silently in a pipeline.
- `--skill-mode` outputs `=== AWT SKILL DEVQA ===` block on failure for AI parsing
- Tracks attempt count across runs (resets on success or different scenario)

**The approval gate.** Without `--skill-mode`, `aat run` shows the scenario and
waits for a keypress before opening the browser — Enter to run, `e` to edit the
YAML, `n` to cancel (exit 0). A process with no terminal cannot be asked at all,
and that exits **4**, not 0: no step ran, so reporting success would be a lie to
whatever is reading the code. The prompt reads `/dev/tty`, not stdin, so piping
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
| `--report-screenshots` | — | `failures` | Which steps the PDF illustrates: `failures` / `all` / `none` |

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

A remembered position is the last thing a step consults — after the selector,
the input finder, the text search and any banked picture — so it only acts when
nothing on the page could be located at all. A stale one then keeps the step
clicking an empty spot. Reset it after the UI moves. Targets whose position follows the content — modal buttons,
choice overlays drawn on an image, list rows — are better marked
`learn: false` on the step so nothing is remembered in the first place.

### `aat learned list`
List everything AWT remembers: learned element mappings, remembered
coordinates (target, page state, position, confidence, use count), failure
patterns, platform tips, and banked element pictures (host, target, crop size,
age).

Banked pictures are what self-healing matches against. Every step that finds
its element through the DOM crops that element out of the screenshot it already
took and keeps it, so a later run can still find the element after the selector
breaks. They are stored per host under `~/.awt/templates/<host>/`, expire after
30 days, and are capped at 300 per host.

A banked picture is used only after a DOM lookup fails, and only if the store
already holds a picture for that target on that host — the lookup happens before
any screenshot is taken, so a run that has banked nothing pays nothing for the
attempt. `--fast` makes the same attempt: it skips OCR and Vision AI, not
healing. A step healed this way reports `saved_template` as its match method,
which is how you tell a heal apart from a scenario that supplied its own
`target.image`, and `aat cost` and the learned strategies count it separately.
A heal never re-banks the picture it matched against, so the crop cannot drift
across runs.

Healing is scoped to the host the picture came from. A picture banked on
`localhost:3000` will not answer for `staging.example.com`, because a lost heal
costs one failed step while a wrong one reports a passing test that never ran.

A banked picture is tried before a remembered coordinate. Both are left behind
by a successful run, but a picture that matches is evidence the element is on
screen now, while a coordinate is a guess that nothing has moved — so the
evidence is asked first. This also means a step that heals reports
`saved_template` rather than `learned`, which is what makes heals countable.
The same reasoning puts the coordinate behind every DOM route as well, not just
behind the selector: a text search that finds the element is an observation too.
A step that used to pass by clicking a remembered spot may now report
`playwright` or `saved_template` instead — same click, better reason.

### `aat learned clear`
Clear learned data. The database and the pictures are separate stores, so
clearing one leaves the other alone.

```bash
aat learned clear                               # elements, coordinates, failures, tips
aat learned clear --templates                   # banked element pictures only
aat learned clear --templates --host localhost:3000   # one host's pictures
aat learned clear --templates --yes             # skip the confirmation
```

Clear the pictures for a host after a redesign: a picture of the old UI is a
guess dressed as evidence. `AWT_TEMPLATES_DIR` relocates the whole store.

## Environment Variables

All config values can be set via environment variables with `AAT_` prefix and `__` delimiter:

```bash
export AAT_AI__PROVIDER=openai
export AAT_AI__API_KEY=sk-...
export AAT_AI__MODEL=gpt-4o
export AAT_ENGINE__HEADLESS=true
```
