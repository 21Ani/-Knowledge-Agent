# Architecture

## High-level flow

```
User (REPL)
    │
    ▼
main.py ── displays tool steps, SQL, sources, answer
    │
    ▼
KnowledgeAgent.run()  ◄── loop until final text or max steps
    │
    ├──► OpenAI-compatible Chat API (Groq, etc.)
    │         decides: answer OR tool_calls[]
    │
    └──► Tool executor (Python)
              ├── retrieve_memory  → ChromaDB vector search
              ├── query_database   → LLM text-to-SQL → SQLite (read-only)
              ├── web_search       → Tavily or DuckDuckGo
              └── read_url         → HTTP + BeautifulSoup
    │
    ▼
After final answer: MemoryStore.save_turn(question, answer)
```

## Who controls the next step?

| Layer | Responsibility |
|-------|----------------|
| **Model** | Chooses which tools to call, in what order, and when to stop with a natural-language answer |
| **Python (`agent.py`)** | Runs the loop, executes tools, enforces max steps, persists Q&A to memory |
| **Python (`sql_tool.py`)** | Generates SQL via LLM, validates read-only, executes query, optional retry on error |
| **Python (`sql_tool.is_read_only_sql`)** | Hard reject of non-SELECT / DDL / multi-statement before DB touch |

There is no fixed `if sql_question then ...` pipeline in the agent — routing is LLM-driven via function calling.

## Component map

| Component | File | Role |
|-----------|------|------|
| CLI | `main.py` | REPL, pretty-print SQL and sources |
| Agent loop | `agent.py` | Messages + tool_calls protocol |
| Memory | `tools/memory.py` | Chroma persistent collection, cosine similarity |
| SQL | `tools/sql_tool.py` | Schema in prompt, guard, execute |
| Web | `tools/web.py` | Search API + page extraction |
| Config | `config.py` | Paths, API keys, schema text for prompts |
| Data | `db/schema.sql`, `db/seed.py` | SQLite DDL and sample data |

## Tool result flow

1. User message appended to `messages`.
2. LLM response may include `tool_calls`.
3. Each tool runs synchronously; JSON/string result appended as `role: tool`.
4. Loop repeats with full history until the model returns content without tools.
5. Final answer saved to Chroma for future `retrieve_memory` calls.

## Persistence

- **SQLite**: `data/store.db` (seeded once)
- **Chroma**: `data/chroma/` (grows each successful turn)
