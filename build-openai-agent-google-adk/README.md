# Build an OpenAI-Compatible Agent with Google ADK

This tutorial project shows the same blog-writing agent pattern as the Gemini examples, but routes model calls through ADK's `LiteLlm` connector. It can use OpenAI-hosted models with `OPENAI_API_KEY`, and can also point at OpenAI-compatible endpoints supported by LiteLLM.

ADK Python has supported non-Gemini models through LiteLLM since the initial `v0.1.0` release. Gemini is still the most direct path in ADK, while LiteLLM is the portability layer for OpenAI, Anthropic, Cohere, local OpenAI-compatible servers, and other providers.

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

## Choosing Gemini or Another LLM

Use Gemini directly in ADK when you want the most integrated Google path. This is usually the right default for tutorials that rely on Google AI Studio or Vertex AI credentials, Gemini-specific capabilities, Google-hosted tooling, or the simplest ADK setup. A Gemini model can normally be passed as a plain model string, such as `gemini-2.5-flash`, without constructing a connector object.

Use `LiteLlm` when model portability matters more than native Google integration. This project does that by passing `LiteLlm(model=MODEL)` into each `LlmAgent`. The `MODEL` setting must use LiteLLM's provider prefix format, for example:

```text
MODEL=openai/<model-name>
```

For OpenAI-compatible gateways or self-hosted servers, keep the provider prefix expected by LiteLLM and configure the endpoint with provider-specific environment variables, such as `OPENAI_API_BASE`. For non-OpenAI providers, set the matching key, such as `ANTHROPIC_API_KEY`, and use that provider's LiteLLM model string.

## Model Compatibility Notes

ADK gives each `LlmAgent` the same orchestration surface, but the model backend can behave differently. Test the full workflow whenever you change providers.

- **Tool calling:** The planner and writer are exposed as agent tools. Some models follow function/tool calls reliably, while others may need simpler instructions or fewer tool choices.
- **Structured output:** Validation loops expect exact strings such as `ok` or `retry`. Models with loose instruction following can cause unnecessary retries, so keep validator prompts strict and outputs short.
- **Streaming and multimodal features:** Gemini, OpenAI-compatible APIs, and local model servers do not expose identical streaming, audio, image, or file behavior through LiteLLM. Avoid assuming a feature works until it is tested with the selected model.
- **Context windows:** Larger context models can keep more outline and draft state, but higher limits often cost more and may increase latency. Smaller models may need shorter prompts and lower retry counts.
- **Latency and rate limits:** LiteLLM adds a provider abstraction, but the actual limit comes from the target API or gateway. If the ADK Web UI stalls or retries fail, check provider rate-limit errors first.
- **Cost tracking:** Provider billing differs. A multi-agent workflow can call the model several times for one user prompt because planning, validation, writing, and final response generation are separate steps.
- **Model names:** LiteLLM model identifiers change as providers add, rename, or retire models. If a model stops resolving, update `MODEL` rather than changing ADK agent code.
- **Secrets:** Keep API keys in the local `.env` file and never commit it. Use only the credentials needed for the selected provider.

## Practical Selection Guide

Start with Gemini when the tutorial goal is learning ADK itself, using Google Cloud services, or matching Google's documentation examples. Start with LiteLLM when the goal is comparing model quality, using an existing OpenAI-compatible gateway, running through a corporate proxy, or swapping providers without changing the agent graph.

For production-style examples, pin dependency versions, document the tested model string, and run a smoke test after every provider change:

```text
Write a technical blog post about evaluating AI coding agents.
```

Confirm that the planner runs, the writer runs, validation completes, and the final response includes the requested alternate titles and hooks.

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
