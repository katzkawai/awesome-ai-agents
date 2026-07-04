# Build an OpenAI-Compatible Agent with Google ADK

This tutorial project shows the same blog-writing agent pattern as the Gemini examples, but routes model calls through ADK's `LiteLlm` connector. It can use OpenAI-hosted models with `OPENAI_API_KEY`, and can also point at OpenAI-compatible endpoints supported by LiteLLM.

## Project Structure

- `blogger/agent.py`: ADK Web UI entry point and multi-agent workflow.
- `blogger/__init__.py`: Python package marker for ADK discovery.
- `requirements.txt`: Legacy dependency list for readers not using PEP 723.

## Setup

Navigate to this project directory:

```bash
cd build-openai-agent-google-adk
```

Create a local `.env` file:

```text
MODEL=openai/gpt-5.5
OPENAI_API_KEY=your_actual_api_key
```

The `MODEL` value uses LiteLLM's provider/model format. For another OpenAI-compatible endpoint, keep the `openai/` provider prefix and configure the endpoint through the environment variables expected by LiteLLM, such as `OPENAI_API_BASE`.

## Running the Agent

Start the ADK Web UI:

```bash
uv run --script blogger/agent.py
```

Open the local URL shown in the terminal, usually `http://127.0.0.1:8000`, and try:

```text
Write a technical blog post about evaluating AI coding agents.
```

## How It Works

- `LiteLlm(model=MODEL)` adapts the configured OpenAI-compatible model for ADK.
- `BlogPlanner` creates an outline and a validation loop retries weak outlines.
- `BlogWriter` writes the final article and a validation loop checks the draft.
- The root agent exposes planner and writer agents as tools so the workflow stays explicit.

## Troubleshooting

- If authentication fails, confirm `.env` is in this directory and contains `OPENAI_API_KEY`.
- If the model is unavailable, set `MODEL` to another LiteLLM OpenAI model string your account supports.
- If dependency installation fails, rerun with a working network connection because `uv run --script` resolves PEP 723 dependencies automatically.
