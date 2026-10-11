# DevPulse — Capstone Reference Implementation

DevPulse ties together **MCP**, a **ReAct agent**, **LangGraph multi-agent orchestration**, and an **A2A Agent Card** into one working system backed by the real, live GitHub REST API.

## Quick Start

```bash
cd /home/ssp/projects/Python/devpulse

# 1. Activate virtualenv
source ~/projects/Python/myenv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment
cp .env.example .env
# Edit .env to add GITHUB_TOKEN (optional but recommended)

# 4. For Ollama (default local backend):
#    ollama serve
#    ollama pull qwen2.5:3b

# 5. Run any phase:
#    Phase 1: python mcp_client_test.py
#    Phase 3: python agent_core.py
#    Phase 4: python langgraph_agent.py
#    Phase 5: python a2a_card.py
```

## Project Structure

| File | Phase | Description |
|------|-------|-------------|
| `mcp_server.py` | 1 | Real FastMCP server with 5 GitHub tools, 1 resource, 1 prompt |
| `mcp_client_test.py` | 1 | Bare-bones client proving `initialize → list_tools → call_tool` |
| `model_client.py` | 2 | Backend-agnostic LLM connector (Ollama or Groq) |
| `mcp_bridge.py` | 2 | Converts MCP tool schemas to OpenAI function-calling format |
| `agent_core.py` | 3 | Single ReAct agent with human-in-the-loop approval |
| `langgraph_agent.py` | 4 | Router + 3 specialist agents sharing one MCP server |
| `a2a_card.py` | 5 | Publishes DevPulse as a discoverable A2A Agent Card |

## Phase Overview

### Phase 1: Real MCP Server & Protocol Test
```bash
python mcp_client_test.py
# Or call tools directly:
python -c "import mcp_server as s; print(s.get_repo_details('facebook','react'))"
```

### Phase 2: Shared Infrastructure
```bash
python -c "from model_client import get_client_and_model, check_connection; c,m,b=get_client_and_model(); check_connection(c,m,b)"
```

### Phase 3: Single ReAct Agent with Human-in-the-Loop
```bash
python agent_core.py
# Type queries like "Tell me about facebook/react" then "now show its contributors"
# Each tool call requires explicit (y/n) approval
```

### Phase 4: LangGraph Multi-Agent
```bash
python langgraph_agent.py
# Router classifies intent → dispatches to Repo Analyst / Issue Triage / Release Notes
```

### Phase 5: A2A Agent Card
```bash
python a2a_card.py
# Shows how external agents discover DevPulse via skill tags
```

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GITHUB_TOKEN` | No | — | GitHub PAT (raises rate limit 60→5000/hr) |
| `LLM_BACKEND` | No | `ollama` | `ollama` or `groq` |
| `OLLAMA_BASE_URL` | No | `http://localhost:11434/v1` | Ollama OpenAI-compatible endpoint |
| `OLLAMA_MODEL` | No | `qwen2.5:3b` | Local model to use |
| `GROQ_API_KEY` | If using Groq | — | Groq cloud API key |
| `GROQ_MODEL` | No | `llama-3.1-8b-instant` | Groq model to use |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `GITHUB_TOKEN` silently missing in MCP server | Pass `env=os.environ.copy()` to `StdioServerParameters` |
| `Failed to parse JSONRPC message` | Server `print()` writing to stdout — use `file=sys.stderr` |
| GitHub rate limit exceeded (60/hr) | Set `GITHUB_TOKEN` in `.env` |
| Could not reach Ollama | Run `ollama serve` and `ollama pull qwen2.5:3b` |
| Could not reach Groq | Verify `GROQ_API_KEY` is set correctly |
| LangGraph specialist ignores history | Confirm `history` passes through `DevPulseState` across `graph.invoke()` |

## Key Architectural Lessons

1. **MCP** solves *agent-to-tool* access (GitHub data via real HTTP)
2. **A2A** solves *agent-to-agent* discovery (external agents finding DevPulse)
3. **Each protocol solves a distinct problem** — only reach for the one you need
4. **Real APIs surface real engineering concerns**: env inheritance, stdout discipline, rate limits