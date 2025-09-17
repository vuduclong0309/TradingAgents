# TradingAgents Installation & Usage Guide

This guide condenses the key instructions from `README.md` into a streamlined checklist so you can install TradingAgents, configure required services, and run both the CLI experience and the Python API.

## 1. Prerequisites
- **OS**: macOS, Linux, or Windows with PowerShell.
- **Python**: 3.10 or newer (3.13 recommended).
- **Hardware**: Internet access for API calls; optional GPU is not required.
- **API Accounts**:
  - [Finnhub](https://finnhub.io/) account with an API key (free tier works).
  - [OpenAI](https://platform.openai.com/) API key for the LLM agents.
- (Optional) Docker, if you later want to containerize the service.

## 2. Clone the Repository
```bash
git clone https://github.com/TauricResearch/TradingAgents.git
cd TradingAgents
```

## 3. Create and Activate a Virtual Environment
Use any environment manager you prefer. Example with Conda:
```bash
conda create -n tradingagents python=3.13
conda activate tradingagents
```

Or with `venv`:
```bash
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows
.\.venv\Scripts\activate
```

## 4. Install Python Dependencies
```bash
pip install -r requirements.txt
```

The requirements file mirrors the dependencies declared in `pyproject.toml` (LangGraph, LangChain providers, yfinance, pandas, Chainlit UI, etc.).

## 5. Configure Environment Variables
Set the following before running the CLI or the Python API:
```bash
export FINNHUB_API_KEY="your_finnhub_key"
export OPENAI_API_KEY="your_openai_key"
```
On Windows PowerShell, use:
```powershell
$env:FINNHUB_API_KEY = "your_finnhub_key"
$env:OPENAI_API_KEY = "your_openai_key"
```

### Optional Variables
- `TRADINGAGENTS_RESULTS_DIR`: override the folder used for run artifacts (`./results` by default).
- `GITHUB_TOKEN`, `REDIS_URL`, etc. are not required unless you extend the project.

## 6. Prepare Data Paths (Offline Tools)
`tradingagents/default_config.py` points `data_dir` to `/Users/yluo/Documents/Code/ScAI/FR1-data`, which is the authors' local cache. If you plan to use the offline dataflows (finnhub cached JSON, Reddit corpora, etc.), update this path or download the Tauric dataset when it becomes available. Without adjusting the path, only tools marked as "online" will function.

Example override in a custom config:
```python
config = DEFAULT_CONFIG.copy()
config["data_dir"] = "/path/to/your/datasets"
config["online_tools"] = True  # rely on live APIs instead of cached files
```

## 7. Run the Interactive CLI
```bash
python -m cli.main
```
Steps inside the CLI:
1. Enter a ticker symbol (e.g. `NVDA`).
2. Choose the analysis date (format `YYYY-MM-DD`).
3. Select analyst teams and research depth.
4. Pick the quick-thinking and deep-thinking LLM models.
5. Watch the live dashboard as agents gather data, debate, and produce the final trade decision.

Artifacts (reports, logs) are written to `results/<ticker>/<date>/` by default.

## 8. Use the Python API Directly
Basic example mirroring `main.py`:
```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.default_config import DEFAULT_CONFIG

config = DEFAULT_CONFIG.copy()
config["online_tools"] = True
config["quick_think_llm"] = "gpt-4.1-mini"
config["deep_think_llm"] = "o4-mini"

agents = TradingAgentsGraph(debug=True, config=config)
_, decision = agents.propagate("NVDA", "2024-05-10")
print(decision)
```
- `debug=True` streams intermediate states to stdout.
- After `propagate`, call `reflect_and_remember(returns)` to update memory embeddings when you know the trade outcome.

## 9. Outputs and Logging
- Real-time CLI output is rendered via Rich components (status table, debate feed, tool invocations).
- Final state snapshots are stored under `eval_results/<ticker>/TradingAgentsStrategy_logs/` when reflection runs.
- Chainlit support is bundled; see `chainlit.toml` and README for future UI integrations.

## 10. Troubleshooting Tips
- **Missing data files**: enable `online_tools` or point `data_dir` to valid caches.
- **API errors**: confirm keys are loaded in the environment and the chosen models exist for your provider tier.
- **High token usage**: choose lightweight models (e.g. `gpt-4.1-mini`, `o4-mini`, `gemini-2.0-flash`) and reduce `max_debate_rounds` in the config.
- **Rate limits**: throttle runs or cache upstream responses; LangChain retry handlers are already in place for transient failures.

## 11. Next Steps
- Explore `tradingagents/default_config.py` for every tunable parameter, including analyst selection, recursion limits, and storage directories.
- Review `cli/main.py` for the end-to-end Orchestration UI.
- Consult the upstream README for research background, architecture diagrams, and contribution guidelines.

Happy experimenting with TradingAgents!
