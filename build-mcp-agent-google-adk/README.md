# Build an AI Agent with Google ADK - Video 2: MCP Integration

This repository contains the code for the second video in the series "Build an AI Agent with Google ADK". In this video, we extend the basic blogger agent by connecting it to a live Google Trends MCP server.

## Project Structure

- `blogger/agent.py`: The main agent application with MCP integration.
- `blogger/server.py`: The minimal MCP server exposing the `trends` tool.
- `requirements.txt`: Python dependencies.
- `.env`: Environment variables (API key).
- `index.lab.md`: The step-by-step codelab.

## Setup Instructions

### 1. Navigate to the Project Directory
```bash
cd build-mcp-agent-google-adk
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Setup Environment Variables
Update your `.env` file with your Gemini API key:
```text
MODEL=gemini-flash-latest
GOOGLE_API_KEY=your_api_key
```

## Running the Agent

Run the following command to start the ADK Web UI:
```bash
adk web .
```

Open your browser and navigate to `http://127.0.0.1:8000` to interact with the agent!

*Note: The MCP server starts automatically when the agent needs it. You do not need to run it separately.*

## Key Learnings & Troubleshooting

- **Handling Rate Limits**: Google Trends (via `pytrends`) often returns `429 Too Many Requests`. The `trends_server.py` is designed with a multi-layer fallback strategy (Related Queries -> Daily Trending -> Realtime Trending -> Keyword Echo) to ensure the agent remains robust even when the external API fails.
- **Timeout Bug**: A previous version encountered `requests.api.get() got multiple values for keyword argument 'timeout'`. This was fixed by removing the explicit `requests_args` in `TrendReq` initialization.
