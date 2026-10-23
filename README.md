# InsightFlow: LangGraph Data-Analyst Agent

An AI agent built with **LangGraph + LangChain + Python** that takes a messy CSV and a business
question, then plans, cleans the data, analyzes it, **self-critiques its own answer**, and writes a
Markdown report.

## Why it's built this way
- **No hallucinated numbers:** all statistics are computed deterministically in pandas; the LLM only
  narrates the verified figures.
- **Self-review loop:** a critique node checks the analysis against the stats and routes back for
  revision (max 2 passes) via a LangGraph conditional edge.
- **Provider-agnostic:** works with Anthropic or OpenAI via env vars, and falls back to an offline
  mock LLM so anyone can run and test it with no API key.

## Architecture
```
plan -> preprocess -> analyze -> critique --PASS--> report -> END
                         ^           |
                         +--REVISE---+ (max 2)
```

| Module | Role |
|---|---|
| `graph.py` | LangGraph state, nodes, conditional routing |
| `tools.py` | Cleaning (dedupe, imputation, date parsing) and statistics |
| `llm.py` | LLM provider abstraction + mock |
| `main.py` | CLI entry point |

## Quickstart
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env            # add a key (optional)
python scripts/make_sample_data.py
PYTHONPATH=src python -m insightflow.main --csv data/sample_sales.csv \
  --question "Which regions drive revenue and what is the trend?"
PYTHONPATH=src pytest -q
```
Windows PowerShell: use `$env:PYTHONPATH="src"` instead of the inline variable.

Docker: `docker build -t insightflow . && docker run --env-file .env -v $(pwd)/data:/app/data insightflow --csv data/sample_sales.csv`

## Sample output
Data quality log (615 raw rows, 15 duplicates removed, 75 missing values imputed), revenue by
region/category, monthly trend, and an LLM-written analysis with a self-review verdict.

## Roadmap
- Add a RAG tool over company docs (FAISS/Chroma)
- Deploy as a FastAPI service on AWS/Azure
- Evaluation harness (LLM-as-judge) for report quality
