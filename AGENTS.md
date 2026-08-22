# AGENTS.md

## Project

Gradio WebUI wrapping the `browser-use` library for AI-driven browser agents. Python 3.11, no package manager config — plain `requirements.txt`.

## Commands

```bash
# Setup
uv venv --python 3.11 && source .venv/bin/activate
uv pip install -r requirements.txt
playwright install chromium --with-deps
cp .env.example .env   # API keys required for anything agent-related

# Run the UI (default port 7788)
python webui.py --ip 127.0.0.1 --port 7788

# Docker (ARM64 / Apple Silicon needs TARGETPLATFORM)
docker compose up --build
TARGETPLATFORM=linux/arm64 docker compose up --build
```

## Gotchas

- `browser-use` is pinned to `0.1.48`. `src/browser/custom_browser.py`, `custom_context.py`, and `custom_controller.py` subclass that version's internal APIs (e.g. `browser_use.browser.browser.Browser`). Upgrading browser-use will break these imports.
- `webui.py` calls `load_dotenv()` as the first statement, before other imports, because several modules read env vars at import time. Keep this order when adding imports there.
- Tests in `tests/` are manual scripts hitting real LLM APIs and launching a real browser window — they need `.env` keys and are not a CI suite. They do `sys.path.append(".")`, so run them from the repo root (`python tests/test_llm_api.py`).
- No pytest/ruff/typecheck config exists in the repo. Ruff formatting is only declared in `.vscode/settings.json`; CI (`.github/workflows/build.yml`) only builds/pushes Docker images on `main` pushes and releases.

## Architecture

- `webui.py` → `src/webui/interface.py` (`create_ui`, theme registry) builds the Gradio app.
- Each Gradio tab lives in `src/webui/components/*_tab.py`. `src/webui/webui_manager.py` is the central registry of component IDs and persists user settings to `.config.pkl` under `./tmp/webui_settings`.
- `src/utils/llm_provider.py` is the single factory for all LLM providers; it maps provider names to env keys from `.env` (see `.env.example`).
- Agents: `src/agent/browser_use/browser_use_agent.py` and `src/agent/deep_research/deep_research_agent.py`.
