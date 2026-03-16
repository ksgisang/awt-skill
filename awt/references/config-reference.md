# AWT Configuration Reference

Config file: `aat.config.yaml`
Search order: `./aat.config.yaml` → `./.aat/aat.config.yaml` → parent directories

All values can be overridden with env vars: `AAT_<SECTION>__<KEY>` (e.g., `AAT_AI__PROVIDER=openai`)

## Full Config

```yaml
# Project
project_name: "aat-project"
source_path: "."
url: ""

# AI Provider
ai:
  provider: "claude"                    # claude | openai | deepseek | ollama
  api_key: ""                           # Or use AAT_AI__API_KEY env var
  model: "claude-sonnet-4-20250514"
  max_tokens: 4000                      # 100–32000
  temperature: 0.3                      # 0.0–1.0

# Browser Engine
engine:
  type: "web"                           # web | desktop
  browser: "chromium"                   # chromium | firefox | webkit
  headless: false
  viewport_width: 1280                  # 320–3840
  viewport_height: 720                  # 240–2160
  timeout_ms: 30000                     # 1000–120000

# Image/Text Matching
matching:
  confidence_threshold: 0.85            # 0.0–1.0
  multi_scale: true
  scale_range_min: 0.5                  # 0.1–1.0
  scale_range_max: 2.0                  # 1.0–4.0
  grayscale: true
  ocr_languages: ["eng", "kor"]         # Tesseract language codes
  chain_order:                          # Matcher priority
    - "learned"
    - "template"
    - "ocr"
    - "feature"

# Human-like Interaction
humanizer:
  enabled: true
  mouse_speed_min: 0.1                  # seconds
  mouse_speed_max: 0.25
  typing_delay_min: 0.05                # seconds per char
  typing_delay_max: 0.15
  bezier_control_points: 3              # 2–5

# Directories
scenarios_dir: "scenarios"
reports_dir: "reports"
assets_dir: "assets"
data_dir: ".aat"

# DevQA Loop
max_loops: 10                           # 1–100
approval_mode: "manual"                 # manual | branch | auto
```

## AI Provider Setup

### Claude (default)
```bash
export AAT_AI__API_KEY=sk-ant-...
# Or in config:
# ai:
#   provider: claude
#   api_key: sk-ant-...
```

### OpenAI
```yaml
ai:
  provider: openai
  model: gpt-4o
```

### DeepSeek (cost-optimized)
```yaml
ai:
  provider: deepseek
  model: deepseek-chat
```

### Ollama (offline)
```yaml
ai:
  provider: ollama
  model: codellama:7b
```
No API key needed. Requires Ollama running locally.
