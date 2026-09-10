# TradingAgents — Multi-Agent Financial Research System

> A mentor-guided software engineering project that explores how specialized AI agents can collaborate on market research, challenge one another's conclusions, and produce a risk-aware trading decision.

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![LangGraph](https://img.shields.io/badge/Orchestration-LangGraph-1C3C3C)](https://www.langchain.com/langgraph)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

## Project overview

TradingAgents models the research workflow of an investment team as a stateful multi-agent system. Instead of asking one language model for a prediction, the application assigns focused responsibilities to analysts, bullish and bearish researchers, a trader, risk reviewers, and a portfolio manager.

Each agent contributes a different perspective. Their reports and debates are carried through a LangGraph workflow, producing an explainable final recommendation rather than an isolated model response.

This repository is the implementation I developed with guidance from my mentor. The README and presentation have been adapted to document the system as a portfolio project. The academic concept and original open-source foundation are credited in [Acknowledgments](#acknowledgments).

> This project is for education and research only. It does not provide financial, investment, or trading advice. Outputs can be incomplete or incorrect and should not be used to make real financial decisions.

## What the system demonstrates

- Multi-agent orchestration with conditional routes and shared state
- Tool-using LLM agents grounded in market, fundamental, sentiment, and news data
- Structured bull-versus-bear research and multi-perspective risk debate
- Separation of fast analysis models from deeper decision-making models
- Provider-independent LLM configuration
- Checkpoint recovery and a persistent decision/reflection log
- A terminal interface for configuring and observing an analysis run

## Architecture

![TradingAgents workflow](assets/schema.png)

The workflow has five stages:

1. **Analyst team** — technical, fundamental, sentiment, and news agents gather evidence using dedicated data tools.
2. **Research debate** — bullish and bearish researchers challenge the analysts' findings.
3. **Research manager** — a deeper-reasoning model evaluates the debate and creates a consolidated research plan.
4. **Trader and risk team** — the trader proposes an action while aggressive, neutral, and conservative agents test its risk assumptions.
5. **Portfolio manager** — the final agent evaluates the full state and returns the portfolio decision.

The orchestration layer is implemented in `tradingagents/graph/`, agent roles live in `tradingagents/agents/`, and external market-data adapters are organized in `tradingagents/dataflows/`.

## Agent roles

| Team | Responsibility |
| --- | --- |
| Technical analyst | Interprets price history and indicators such as RSI and MACD |
| Fundamentals analyst | Reviews company financial statements and business metrics |
| Sentiment analyst | Summarizes market discussion and short-term sentiment |
| News analyst | Evaluates company, industry, and macroeconomic news |
| Bull and bear researchers | Build competing cases and challenge weak assumptions |
| Trader | Converts the research conclusion into an actionable proposal |
| Risk analysts | Evaluate the proposal from aggressive, neutral, and conservative perspectives |
| Portfolio manager | Produces the final risk-aware decision |

## Technology stack

- **Python 3.10+** for the application and data pipelines
- **LangGraph and LangChain** for agent orchestration, tool calls, and state management
- **yfinance, Alpha Vantage, FRED, Stocktwits, and other adapters** for financial data
- **Rich, Typer, and Questionary** for the interactive terminal experience
- **SQLite checkpoints** for interrupted-run recovery
- **pytest and Ruff** for testing and code quality
- **Docker** for reproducible execution

The model abstraction supports OpenAI, Anthropic, Google, xAI, DeepSeek, Qwen, GLM, MiniMax, OpenRouter, Ollama, Azure OpenAI, AWS Bedrock, and OpenAI-compatible endpoints.

## Run locally

### 1. Clone this repository

```bash
git clone https://github.com/goldenfishome/TradingAgents.git
cd TradingAgents
```

### 2. Create a Python environment

Using Conda:

```bash
conda create -n tradingagents python=3.12
conda activate tradingagents
```

Or using `venv`:

```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Install the project

```bash
pip install .
```

For development and testing:

```bash
pip install -e ".[dev]"
```

### 4. Configure API credentials

Copy the environment template and add only the keys required by the providers you plan to use:

```bash
cp .env.example .env
```

For example:

```bash
OPENAI_API_KEY=your_key_here
ALPHA_VANTAGE_API_KEY=your_key_here
```

Never commit the completed `.env` file or expose real API keys in screenshots.

### 5. Start an analysis

```bash
tradingagents
```

You can also launch the CLI directly from the source tree:

```bash
python -m cli.main
```

The interface prompts for a ticker, analysis date, analyst team, LLM provider, model, and research depth.

![TradingAgents CLI](assets/cli/cli_init.png)

## Programmatic example

```python
from tradingagents.default_config import DEFAULT_CONFIG
from tradingagents.graph.trading_graph import TradingAgentsGraph

config = DEFAULT_CONFIG.copy()
config["llm_provider"] = "openai"
config["deep_think_llm"] = "gpt-5.5"
config["quick_think_llm"] = "gpt-5.4-mini"
config["max_debate_rounds"] = 2

graph = TradingAgentsGraph(debug=True, config=config)
state, decision = graph.propagate("NVDA", "2026-01-15")

print(decision)
```

The framework accepts exchange-qualified Yahoo Finance symbols, including `AAPL`, `0700.HK`, `7203.T`, `RELIANCE.NS`, and `BTC-USD`.

## Persistence and recovery

The system maintains two forms of state:

- **Decision log:** completed analyses are written to `~/.tradingagents/memory/trading_memory.md`. Later runs can incorporate reflections on earlier decisions.
- **Checkpoint recovery:** when enabled, LangGraph stores completed graph steps in per-ticker SQLite databases so an interrupted run can continue without restarting every agent.

Enable checkpointing from the CLI:

```bash
tradingagents analyze --checkpoint
```

## Testing and quality checks

Run the test suite:

```bash
pytest
```

Run static checks:

```bash
ruff check .
```

Because the application uses live market data and generative models, identical inputs do not guarantee identical outputs. Historical dates constrain price data, but news, social sources, provider behavior, and model sampling can still change between runs.

## Repository structure

```text
TradingAgents/
├── cli/                       # Interactive terminal application
├── tradingagents/
│   ├── agents/                # Analyst, researcher, trader, and manager roles
│   ├── dataflows/             # Market-data providers and normalization
│   ├── graph/                 # LangGraph workflow and routing
│   └── llm_clients/           # Model-provider abstraction
├── tests/                     # Unit, integration, and smoke tests
├── main.py                    # Minimal programmatic example
└── docker-compose.yml         # Containerized execution
```

## Engineering lessons

This project gave me practical experience with problems that appear in production AI systems: controlling multi-step workflows, grounding model output in tools, managing provider differences, preserving state across failures, separating model roles by cost and reasoning needs, and testing software whose external dependencies are not fully deterministic.

## Roadmap

- Add a browser-based dashboard for reports and agent debates
- Add reproducible backtesting summaries and benchmark visualizations
- Add portfolio-level analysis and position-sizing controls
- Expand automated evaluations for factual grounding and decision consistency

## Acknowledgments

I built this project with guidance from my mentor using the open-source [TradingAgents](https://github.com/TauricResearch/TradingAgents) framework and its associated research as a foundation. Credit for the original architecture and paper belongs to Yijia Xiao, Edward Sun, Di Luo, Wei Wang, and the Tauric Research contributors.

If you use the original research, cite:

```bibtex
@misc{xiao2025tradingagentsmultiagentsllmfinancial,
  title={TradingAgents: Multi-Agents LLM Financial Trading Framework},
  author={Yijia Xiao and Edward Sun and Di Luo and Wei Wang},
  year={2025},
  eprint={2412.20138},
  archivePrefix={arXiv},
  primaryClass={q-fin.TR},
  url={https://arxiv.org/abs/2412.20138}
}
```

This repository retains the original Apache License 2.0. See [LICENSE](LICENSE) for details.
