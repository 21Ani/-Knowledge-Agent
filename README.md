# Knowledge Agent — SQL, Web & Memory

Terminal-based research assistant that answers questions using **conversation memory (RAG)**, a **local SQLite database (text-to-SQL)**, **web search**, and an **LLM** with a tool-calling loop.

## Requirements

- Python 3.11+
- API key for an OpenAI-compatible chat API (e.g. [Groq](https://console.groq.com/) free tier)

## Setup

```bash
cd "d:\Projects\New folder"
python -m venv .venv

# Windows
.venv\Scripts\activate

pip install -r requirements.txt
copy .env.example .env
```

Edit `.env` and set `LLM_API_KEY`. Defaults target Groq:

```
LLM_API_KEY=gsk_...
LLM_BASE_URL=https://api.groq.com/openai/v1
LLM_MODEL=llama-3.3-70b-versatile
```

Optional: set `TAVILY_API_KEY` for Tavily web search (otherwise DuckDuckGo is used, no key required).

## Seed the database

```bash
python -m knowledge_agent.db.seed
```

Creates `data/store.db` with customers, products, orders, and order_items (~150+ rows).

## Run

```bash
python -m knowledge_agent
```

Type questions at the `You>` prompt. Use `exit` or `quit` to leave.

## Example questions

**SQL only**

- Which products were most ordered last month?
- Show me the top 5 customers by total spend, with their cities.
- Which customers from Mumbai have unpaid orders?

**SQL + Web**

- Pick our best-selling product — what are recent reviews or news about similar products online?

**Web only**

- What are the latest changes to GSTR-3B filing rules?

**Memory only** (ask after a prior turn)

- What did I ask you earlier about top customers?
- Can you remind me what we discussed about products?

## Project layout

```
knowledge_agent/
  main.py          # CLI REPL
  agent.py         # Tool-calling loop
  config.py        # Env and paths
  tools/
    memory.py      # ChromaDB RAG
    sql_tool.py    # Text-to-SQL + guard
    web.py         # web_search, read_url
  db/
    schema.sql
    seed.py
data/              # Created at runtime (gitignored)
```

## Environment variables

| Variable | Description |
|----------|-------------|
| `LLM_API_KEY` | Required. Chat API key |
| `LLM_BASE_URL` | API base URL (default: Groq) |
| `LLM_MODEL` | Model name |
| `TAVILY_API_KEY` | Optional. Tavily search |
| `DB_PATH` | SQLite path (default: `data/store.db`) |
| `CHROMA_PATH` | Vector store path (default: `data/chroma`) |
| `MAX_AGENT_STEPS` | Max tool loop iterations (default: 8) |

See also: [SCHEMA.md](SCHEMA.md), [ARCHITECTURE.md](ARCHITECTURE.md), [DESIGN.md](DESIGN.md).
