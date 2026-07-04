# Repository Guidelines

## Project Structure & Module Organization
This repository is a collection of small AI agent tutorial projects. Each project is self-contained and should be changed from its own directory.

- `README.md`: top-level collection overview.
- `build-ai-agent-google-adk/`: basic Google ADK blogger agent.
- `build-mcp-agent-google-adk/`: Google ADK blogger agent with a local MCP Google Trends tool.
- `*/blogger/agent.py`: ADK agent definitions and orchestration.
- `build-mcp-agent-google-adk/blogger/server.py`: stdio MCP server exposing the `trends` tool.
- `*/requirements.txt`: per-project Python dependencies.

There is no shared package layer today. Keep new tutorial examples isolated in their own top-level directory with a README, `requirements.txt`, and source package.

## Build, Test, and Development Commands
Run commands from the project directory you are working in.

```bash
cd build-ai-agent-google-adk
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
adk web
```

Use `adk web .` in `build-mcp-agent-google-adk` to start the ADK Web UI with the MCP-enabled agent. The MCP server is launched by the agent when needed.

Create a local `.env` in the relevant project:

```text
MODEL=gemini-flash-latest
GOOGLE_API_KEY=your_api_key
```

## Coding Style & Naming Conventions
This is Python code. Use clear module-level constants such as `MODEL`, `PascalCase` for agent classes, and `snake_case` for functions and variables. Keep imports grouped as standard library, third-party, then local code. Prefer concise comments that explain behavior or integration constraints, especially around ADK and MCP wiring. Use 4-space indentation for new code unless preserving alignment in an existing block.

## Testing Guidelines
No automated test suite is currently present. For changes, perform a manual smoke test with `adk web` or `adk web .`, then verify the agent loads and can answer a simple blog-topic prompt. For MCP changes, also exercise a prompt that triggers the `trends` tool and confirm fallback behavior remains usable when Google Trends rate-limits.

## Commit & Pull Request Guidelines
Recent commits use short, imperative messages such as `adding mcp adk project` and `delete old project`. Keep commits focused and describe the user-visible change. Pull requests should include a brief summary, setup or testing steps run, any required environment variables, and screenshots only when UI behavior changes.

## Security & Configuration Tips
Do not commit `.env`, API keys, local virtual environments, or generated caches. Keep dependency changes scoped to the relevant tutorial directory.
