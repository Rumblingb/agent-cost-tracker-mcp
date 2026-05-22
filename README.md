# 💰 Agent Cost Tracker MCP

[![MCP Server](https://img.shields.io/badge/MCP-Server-blue)](https://modelcontextprotocol.io)
[![Python](https://img.shields.io/badge/Python-3.10%2B-green)](https://python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Smithery](https://img.shields.io/badge/Smithery-Listed-purple)](https://smithery.ai)
[![Pro $19/mo](https://img.shields.io/badge/Pro-%2419%2Fmo-635bff)](https://buy.stripe.com/9B6cN7gpVescd289351oI0u)

**Give your AI agents self-awareness about their spending.** Track token usage, API costs, and set budget alerts — all from within your agent's tool calls.

## Why Agent Cost Tracker?

AI agents burn tokens. They call APIs. They cost money. And **nobody tracks it**. 

Every agent builder has the same pain point: *"I don't know what my agents are costing me."* 

Agent Cost Tracker MCP solves this. Agents self-report their API calls, and you get real-time cost visibility — per agent, per model, per day. No external dashboard. No complex setup. Just install the MCP server and your agents gain cost awareness.

```python
# Your agent tracks its own costs with a single tool call
await cost_track_call(
    model="claude-sonnet-4",
    input_tokens=1500,
    output_tokens=800,
    agent="code-reviewer-01"
)
# → "$0.0165 spent on claude-sonnet-4. Monthly total: $3.42 of $50 budget (6.8%)"
```

## Features

- **6 structured tools** for complete cost visibility
- **30+ pre-seeded model prices** (OpenAI, Anthropic, DeepSeek, Google, Meta, Cohere)
- **Budget management** with configurable alerts
- **Cost estimation** before making API calls
- **Per-agent tracking** via optional agent identifier
- **Multi-period views** — today, this week, this month, all time
- **Zero dependencies** beyond `mcp` and `tiktoken`
- **JSON-file storage** — data stays on your machine in `~/.agent-cost-tracker/`
- **Response formatting** — markdown (human) or JSON (programmatic)

## Installation

```bash
# Clone the repository
git clone https://github.com/Rumblingb/agent-cost-tracker-mcp.git
cd agent-cost-tracker-mcp

# Install dependencies (system Python — no venv needed)
pip install mcp tiktoken

# Or with requirements.txt
pip install -r requirements.txt
```

### MCP Client Configuration

Add to your MCP client's `config.yaml`:

```yaml
mcpServers:
  agent-cost-tracker:
    command: python3
    args:
      - /path/to/agent-cost-tracker-mcp/server.py
    description: Track AI agent token usage and API costs
```

**Claude Desktop:**
```json
{
  "mcpServers": {
    "agent-cost-tracker": {
      "command": "python3",
      "args": ["/path/to/agent-cost-tracker-mcp/server.py"]
    }
  }
}
```

**VS Code / Cursor:**
```json
{
  "mcpServers": {
    "agent-cost-tracker": {
      "command": "python3",
      "args": ["server.py"],
      "cwd": "/path/to/agent-cost-tracker-mcp"
    }
  }
}
```

## Tools Reference

### 1. `cost_track_call`

Track a single API call. Auto-calculates cost if omitted.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | ✅ | Model identifier (e.g., `"claude-sonnet-4"`) |
| `input_tokens` | integer | ✅ | Number of input/prompt tokens |
| `output_tokens` | integer | ✅ | Number of output/completion tokens |
| `cost` | float | ❌ | Manual cost override (auto-calculated if omitted) |
| `agent` | string | ❌ | Agent identifier for per-agent tracking |
| `purpose` | string | ❌ | Description of the call (e.g., "code review") |

**Example:**
```python
await cost_track_call(
    model="gpt-4o",
    input_tokens=2500,
    output_tokens=1200,
    agent="refactoring-bot",
    purpose="Refactoring auth module"
)
```

### 2. `cost_get_usage`

Get usage summary with breakdown by model.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `period` | string | ❌ | `"today"`, `"week"`, `"month"`, `"all"` (default: `"month"`) |
| `agent` | string | ❌ | Filter by agent identifier |
| `format` | string | ❌ | `"markdown"` (default) or `"json"` |

**Example:**
```python
# Get monthly spending broken down by model
await cost_get_usage(period="month", format="markdown")
```

**Response:**
```
## 💰 Monthly Usage (May 2026)

| Metric | Value |
|--------|-------|
| Total Calls | 847 |
| Total Tokens | 1,245,000 |
| Total Cost | $4.23 |
| Budget | $50.00 |
| Remaining | $45.77 (91.5%) |

### By Model
| Model | Calls | Tokens | Cost |
|-------|-------|--------|------|
| claude-sonnet-4 | 312 | 520,000 | $2.34 |
| gpt-4o-mini | 425 | 680,000 | $0.57 |
| deepseek-chat | 110 | 45,000 | $1.32 |
```

### 3. `cost_set_budget`

Set monthly spending limit and alert threshold.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `monthly_limit` | float | ✅ | Maximum monthly spend in USD |
| `alert_threshold` | float | ❌ | Alert when % of budget reached (default: 80) |
| `currency` | string | ❌ | Currency code (default: `"USD"`) |

### 4. `cost_list_models`

List all supported models with their per-token pricing.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `provider` | string | ❌ | Filter by provider (`"openai"`, `"anthropic"`, etc.) |
| `format` | string | ❌ | `"markdown"` or `"json"` |

**Pre-seeded model pricing includes:**
- **OpenAI:** gpt-4o, gpt-4o-mini, gpt-4-turbo, o3-mini, o1
- **Anthropic:** claude-sonnet-4, claude-haiku-3.5, claude-opus-4
- **DeepSeek:** deepseek-chat, deepseek-reasoner
- **Google:** gemini-2.5-pro, gemini-2.5-flash
- **Meta:** llama-4-maverick
- **Cohere:** command-r-plus, command-r

### 5. `cost_check_budget`

Check current budget status with projected monthly spend.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `format` | string | ❌ | `"markdown"` or `"json"` |

**Response includes:**
- Current spend vs. budget
- Remaining balance
- Projected monthly spend (based on daily run rate)
- Days remaining in billing period
- Alert status (green/yellow/red)

### 6. `cost_estimate`

Estimate cost BEFORE making a call — avoid surprises.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | ✅ | Model identifier |
| `input_tokens` | integer | ✅ | Estimated input tokens |
| `output_tokens` | integer | ✅ | Estimated output tokens |

**Example:**
```python
await cost_estimate(
    model="claude-sonnet-4",
    input_tokens=5000,
    output_tokens=2000
)
# → "Estimated: $0.045 (5000 in × $3.00/M + 2000 out × $15.00/M)"
```

## Pricing

| Tier | Price | Limits |
|------|-------|--------|
| **Free** | $0 | 1,000 tracked calls/month, 1 budget, basic model list |
| **Pro** | [$19/month](https://buy.stripe.com/9B6cN7gpVescd289351oI0u) | Unlimited calls, unlimited budgets, 30+ models, per-agent tracking, priority support |

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                   AI Agent (Claude/GPT)               │
│  Uses cost_track_call() after each API invocation     │
└──────────────────────┬───────────────────────────────┘
                       │ MCP Protocol (stdio JSON-RPC)
┌──────────────────────▼───────────────────────────────┐
│              Agent Cost Tracker MCP Server            │
│                                                       │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐   │
│  │ Track    │  │ Budget   │  │ Model Pricing     │   │
│  │ Calls    │  │ Manager  │  │ Registry          │   │
│  └────┬─────┘  └────┬─────┘  └────────┬──────────┘   │
│       │              │                │               │
│  ┌────▼──────────────▼────────────────▼──────────┐   │
│  │           ~/.agent-cost-tracker/              │   │
│  │  calls.json  │  budgets.json  │  models.json │   │
│  └───────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────┘
```

### Data Storage

All data persists as JSON files in `~/.agent-cost-tracker/`:

| File | Contents |
|------|----------|
| `calls.json` | Array of call records with timestamps, model, tokens, cost |
| `budgets.json` | Budget configurations with monthly limits and alert thresholds |
| `models.json` | Pre-seeded model pricing data (30+ models) |

### Tool Annotations

| Tool | readOnlyHint | destructiveHint | idempotentHint | openWorldHint |
|------|-------------|-----------------|----------------|---------------|
| `cost_track_call` | false | false | false | false |
| `cost_get_usage` | true | false | true | false |
| `cost_set_budget` | false | false | false | false |
| `cost_list_models` | true | false | true | false |
| `cost_check_budget` | true | false | true | false |
| `cost_estimate` | true | false | true | true |

## Usage Scenarios

### Self-Aware Coding Agent

```python
# Agent tracks every LLM call automatically
async def call_llm(model, prompt):
    input_tokens = count_tokens(prompt)
    response = await llm_api(model, prompt)
    output_tokens = count_tokens(response)
    
    await cost_track_call(
        model=model,
        input_tokens=input_tokens,
        output_tokens=output_tokens,
        agent="coding-assistant",
        purpose="Code generation"
    )
    return response
```

### Budget-Controlled Autonomous Agent

```python
# Agent checks budget before expensive operations
budget = await cost_check_budget()
if budget["remaining"] < 1.0:
    print("⚠️ Budget running low — switching to cheaper model")
    model = "gpt-4o-mini"  # fallback to cheaper model
else:
    model = "claude-sonnet-4"
```

### Multi-Agent Cost Dashboard

```python
# Get per-agent breakdown for billing clients
usage = await cost_get_usage(period="month", format="json")
for model_data in usage["by_model"]:
    print(f"{model_data['model']}: ${model_data['cost']:.2f}")
```

## Development

```bash
# Clone and install
git clone https://github.com/Rumblingb/agent-cost-tracker-mcp.git
cd agent-cost-tracker-mcp
pip install -r requirements.txt

# Test with MCP Inspector
npx @modelcontextprotocol/inspector python3 server.py

# Run tests
python3 -m pytest tests/
```

### Requirements

- Python 3.10+
- `mcp>=1.0.0`
- `tiktoken`

## License

MIT — see [LICENSE](LICENSE) for details.

## Related MCP Servers

- [Agent Memory MCP](https://github.com/Rumblingb/agent-memory-mcp) — Persistent memory for AI agents
- [Search Proxy MCP](https://github.com/Rumblingb/search-proxy-mcp) — Web search for AI agents
- [Hallucination Guard API](https://github.com/Rumblingb/hallucination-guard) — Fact verification for agents
- [MCP Server Directory](https://rumblingb.github.io/mcp-directory/) — Curated list of all MCP servers

---

Built by [AgentPay Labs](https://agentpay.so) — Governed payment middleware for AI agents.
