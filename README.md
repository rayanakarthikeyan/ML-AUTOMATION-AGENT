<p align="center">
  <a href="https://github.com/huggingface/ml-automation-agent/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/License-Apache_2.0-blue.svg"></a>
  <a href="https://smolagents-ml-automation-agent.hf.space/"><img alt="Website" src="https://img.shields.io/website/https/smolagents-ml-automation-agent.hf.space.svg?down_color=red&down_message=offline&up_message=online"></a>
</p>

<p align="center">
  <img src="frontend/public/smolagents.webp" alt="smolagents logo" width="160" />
</p>

# Agent

> Personalized fork by Rayana Karthikeyan.

Agent is a Hugging Face-powered autonomous assistant for machine learning engineering. It researches, writes, and ships ML-related code with integrated access to datasets, models, papers, and compute.

## Features

- Interactive chat-driven agent interface
- Headless prompt mode for one-shot commands
- Local and remote model support via LiteLLM and HF Inference Providers
- Sandbox execution for HF Spaces and GPU testing
- Trace upload to Hugging Face for session history and auditing
- Slack notification gateway for approval and status updates

## Quick Start

### Install

```powershell
cd "c:\Users\Shiva\Downloads\ml-automation-agent\ML AUTOMATION"
python -m pip install -e .
```

### Environment variables
Create a `.env` file in the project root or export these values in your shell:

```powershell
HF_TOKEN=<your-hugging-face-token>
GITHUB_TOKEN=<your-github-token>
```

### Run the interactive agent

```powershell
cd "c:\Users\Shiva\Downloads\ml-automation-agent\ML AUTOMATION"
python -m agent.main
```

### Run a single prompt headlessly

```powershell
python -m agent.main "Build a data pipeline for CSV to Hugging Face."
```

## CLI Usage

### Available CLI flags

```powershell
python -m agent.main --help
python -m agent.main --model <model-id>
python -m agent.main --max-iterations <n>
python -m agent.main --no-stream
python -m agent.main --sandbox-tools
```

### Running with a local model

```powershell
python -m agent.main --model ollama/llama3.1:8b
```

### Running with sandbox tools

```powershell
python -m agent.main --sandbox-tools
```

### Recommended alias if the CLI is not on PATH

```powershell
cd "c:\Users\Shiva\Downloads\ml-automation-agent\ML AUTOMATION"
python -m agent.main
```

## Local Models

Local models use OpenAI-compatible inference endpoints. The agent does not load weights directly from disk; start your local inference server and then select it with a provider prefix.

```powershell
python -m agent.main --model ollama/llama3.1:8b "your prompt"
python -m agent.main --model vllm/meta-llama/Llama-3.1-8B-Instruct "your prompt"
```

Inside the interactive session, switch models with:

```text
/model ollama/llama3.1:8b
/model lm_studio/google/gemma-3-4b
/model llamacpp/llama-3.1-8b-instruct
```

Supported local prefixes:
- `ollama/`
- `vllm/`
- `lm_studio/`
- `llamacpp/`

### Local server environment variables

```powershell
LOCAL_LLM_BASE_URL=http://localhost:8000
LOCAL_LLM_API_KEY=<optional-local-api-key>
```

Use a shared local endpoint or provider-specific overrides such as `OLLAMA_BASE_URL` and `VLLM_API_KEY`.

## Sandbox Runtime

By default, the CLI uses local filesystem tools (`bash`, `read`, `write`, `edit`). Use `--sandbox-tools` to run with HF Space sandbox tooling and remote GPU test environments.

```powershell
python -m agent.main --sandbox-tools
```

Sandbox mode requires `HF_TOKEN` even for local model workflows because it provisions private HF Spaces.

Set sandbox mode as the default in `~/.config/ml-automation-agent/cli_agent_config.json`:

```json
{ "tool_runtime": "sandbox" }
```

## Trace Sharing

Each session can be uploaded to your own private Hugging Face dataset in Claude Code JSONL format. By default, the dataset is named `{your-hf-username}/ml-automation-agent-sessions` and is created private.

Use these commands inside the agent:

```text
/share-traces
/share-traces public
/share-traces private
```

To disable trace sharing globally, set:

```json
{ "share_traces": false }
```

To override the destination repository:

```json
{ "personal_trace_repo_template": "{hf_user}/my-custom-traces" }
```

## Interactive Commands

Use these slash commands inside the running agent:

- `/help` — show available commands
- `/new` — start a fresh chat session
- `/clear` — clear the terminal and start fresh
- `/undo` — undo the last turn
- `/compact` — compact the session context
- `/resume [index|id|path]` — resume a saved session from `./session_logs`
- `/model [id]` — list or switch models
- `/effort [level]` — set reasoning effort preference
- `/yolo` — toggle auto-approve mode
- `/status` — show current model and turn count
- `/share-traces [public|private]` — show or change trace visibility
- `/quit` — exit the agent
- `exit` or `/exit` — also exit the agent

## Slack Notifications

The agent supports one-way Slack notification gateways for approval and status updates.

Set these environment variables:

```powershell
SLACK_BOT_TOKEN=xoxb-...
SLACK_CHANNEL_ID=C...
```

Optional environment configuration:

```powershell
ML_AUTOMATION_AGENT_SLACK_NOTIFICATIONS=false
ML_AUTOMATION_AGENT_SLACK_DESTINATION=slack.ops
ML_AUTOMATION_AGENT_SLACK_AUTO_EVENTS=approval_required,error,turn_complete
ML_AUTOMATION_AGENT_SLACK_ALLOW_AGENT_TOOL=true
ML_AUTOMATION_AGENT_SLACK_ALLOW_AUTO_EVENTS=true
```

Or configure at `~/.config/ml-automation-agent/cli_agent_config.json`:

```json
{
  "messaging": {
    "enabled": true,
    "auto_event_types": ["approval_required", "error", "turn_complete"],
    "destinations": {
      "slack.ops": {
        "provider": "slack",
        "token": "${SLACK_BOT_TOKEN}",
        "channel": "${SLACK_CHANNEL_ID}",
        "allow_agent_tool": true,
        "allow_auto_events": true
      }
    }
  }
}
```

## Development

### Run pre-commit checks

```powershell
uv run ruff check .
uv run ruff format --check .
```

If formatting fails, run:

```powershell
uv run ruff format .
```

### Add a built-in tool

Edit `agent/core/tools.py` and add a new `ToolSpec` entry:

```python
def create_builtin_tools() -> list[ToolSpec]:
    return [
        ToolSpec(
            name="your_tool",
            description="What your tool does",
            parameters={
                "type": "object",
                "properties": {
                    "param": {"type": "string", "description": "Parameter description"}
                },
                "required": ["param"]
            },
            handler=your_async_handler
        ),
        # existing tools...
    ]
```

### Configure MCP servers

Edit `configs/cli_agent_config.json` or `configs/frontend_agent_config.json`:

```json
{
  "model_name": "anthropic/claude-opus-4.8:fal-ai",
  "mcpServers": {
    "your-server-name": {
      "transport": "http",
      "url": "https://example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${YOUR_TOKEN}"
      }
    }
  }
}
```

## Citation
If you use ML Automation Agent in your work, please cite it using:

```bibtex
@Misc{ml-automation-agent,
  title =        {ML Automation Agent: an agent that autonomously researches, writes, and ships ML related code using the Hugging Face ecosystem},
  author =       {Aksel Joonas Reedi, Henri Bonamy, Yoan Di Cosmo, Leandro von Werra, Lewis Tunstall},
  howpublished = {\url{https://github.com/huggingface/ml-automation-agent}},
  year =         {2026}
}
```
