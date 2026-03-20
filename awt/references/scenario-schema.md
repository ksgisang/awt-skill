# AWT Scenario Schema Reference

## Scenario (Root)

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `id` | string | Yes | — | Pattern: `SC-\d{3,}` (e.g., SC-001) |
| `name` | string | Yes | — | Scenario name (min 1 char) |
| `description` | string | No | `""` | Description |
| `tags` | list[string] | No | `[]` | Tags for filtering |
| `steps` | list[StepConfig] | Yes | — | At least 1 step required |
| `expected_result` | list[ExpectedResult] | No | `[]` | Scenario-level assertions |
| `variables` | dict[str, str] | No | `{}` | Template variables (`{{url}}`) |

## StepConfig

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `step` | int | Yes | — | Step number (≥ 1) |
| `action` | ActionType | Yes | — | Action to perform |
| `target` | TargetSpec | Conditional | — | Required for find_and_* actions |
| `value` | string | Conditional | — | Required for navigate, type, etc. |
| `description` | string | Yes | — | Step description (min 1 char) |
| `humanize` | bool | No | `true` | Apply human-like interaction |
| `screenshot_before` | bool | No | `false` | Capture before action |
| `screenshot_after` | bool | No | `false` | Capture after action |
| `timeout_ms` | int | No | `10000` | Step timeout (0–120000ms) |
| `optional` | bool | No | `false` | Don't fail scenario if step fails |
| `assert_type` | AssertType | Conditional | — | Required if action is `assert` |
| `expected` | list[ExpectedResult] | Conditional | `[]` | Required if action is `assert` |
| `method` | FindMethod | No | `auto` | Matching method: `auto`, `template`, `ocr`, `vision` |
| `learn` | bool | No | `true` | Save successful match to pattern DB |
| `fallback` | bool | No | `true` | Allow tier fallback when specific method fails |

## TargetSpec

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `image` | string | * | — | Target image path |
| `text` | string | * | — | OCR text to find |
| `selector` | string | * | — | CSS selector (highest priority) |
| `match_method` | MatchMethod | No | — | Force specific matcher |
| `confidence` | float | No | — | Override threshold (0.0–1.0) |

*At least one of `image`, `text`, or `selector` is required.

## ExpectedResult

| Field | Type | Required | Default |
|-------|------|----------|---------|
| `type` | AssertType | Yes | — |
| `value` | string | Yes | — |
| `tolerance` | float | No | `0.0` |
| `case_insensitive` | bool | No | `false` |

## ActionType Enum (19 values)

**Navigation:** `navigate`, `go_back`, `refresh`

**Find + Mouse (require target):**
- `find_and_click` — Click element found by image/text/selector
- `find_and_double_click` — Double-click
- `find_and_right_click` — Right-click

**Find + Keyboard (require target):**
- `find_and_type` — Find element, then type `value` into it
- `find_and_clear` — Find element, then clear its content

**Direct:**
- `click_at` — Click at coordinates (value: `"x,y"`)
- `type_text` — Type into currently focused element
- `press_key` — Press key (value: `Enter`, `Tab`, `Escape`, etc.)
- `key_combo` — Key combination (value: `"Ctrl+A"`, `"Cmd+C"`)

**Assert:**
- `assert` — Validate with `assert_type` and `expected`

**Utility:**
- `wait` — Wait milliseconds (value: `"2000"`)
- `screenshot` — Capture current screen
- `scroll` — Scroll (value: `"x,y,delta"`, delta > 0 = down)

## AssertType Enum

| Value | Description |
|-------|-------------|
| `text_visible` | Text appears anywhere on page |
| `text_equals` | Exact text match |
| `image_visible` | Image template found on screen |
| `url_contains` | Current URL contains substring |
| `url_not_contains` | Current URL does NOT contain substring |
| `screenshot_match` | Current screen matches reference image |

## FindMethod Enum (step-level)

| Value | Description |
|-------|-------------|
| `auto` | 3-tier fallback chain (default): template → OCR → Vision AI |
| `template` | Force template matching only (OpenCV) |
| `ocr` | Force OCR only (Tesseract with CLAHE + sharpening) |
| `vision` | Force Vision AI only (Claude API, cost incurred) |

When `fallback: true` (default), a failed specific method falls back to the full chain.

## MatchMethod Enum (target-level)

| Value | Algorithm | Best For |
|-------|-----------|----------|
| `learned` | SQLite lookup | Previously successful matches |
| `template` | cv2.matchTemplate | Exact visual matching |
| `ocr` | pytesseract + CLAHE + sharpening | Text-based element finding |
| `feature` | ORB keypoints | Rotation/scale invariant |
| `vision_ai` | Claude Vision API | Canvas/CanvasKit text, complex UIs |

## Action-Value Requirements

| Action | Requires target | Requires value | Value format |
|--------|----------------|----------------|-------------|
| `navigate` | No | Yes | URL (supports `{{url}}`) |
| `find_and_click` | Yes | No | — (supports `method`, `learn`, `fallback`) |
| `find_and_type` | Yes | Yes | Text to type |
| `find_and_clear` | Yes | No | — |
| `click_at` | No | Yes | `"x,y"` |
| `type_text` | No | Yes | Text to type |
| `press_key` | No | Yes | Key name |
| `key_combo` | No | Yes | `"Ctrl+A"` |
| `assert` | No | No | Uses `assert_type` + `expected` |
| `wait` | No | Yes | Milliseconds |
| `scroll` | No | Yes | `"x,y,delta"` |
| `screenshot` | No | No | — |
