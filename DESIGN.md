# Design Notes

## (a) LLM-driven tool use vs hard-coded pipeline

**LLM-driven (this project):** The model picks tools and order per question. Flexible for mixed queries (e.g. SQL for top product, then web for reviews) without new code paths.

**Hard-coded pipeline:** Faster and cheaper for known patterns (e.g. always SQL first if keywords match “customer” or “order”). Predictable but brittle — new phrasing or multi-intent questions need new rules.

**Tradeoff:** We accept variable latency and occasional wrong tool choice in exchange for one loop that handles SQL-only, web-only, memory-only, and combined cases. Safety-critical parts (SQL read-only guard, max steps) stay in Python, not the model.

## (b) Failure modes and mitigations

| Failure | What happens | Mitigation |
|---------|----------------|------------|
| **Bad SQL** (wrong column, bad join) | Query errors or empty/wrong results | Schema in system + SQL tool prompts; **query repair** feeds SQLite error back to LLM (max 2 retries); CLI shows generated SQL so users can spot issues |
| **Hallucinated columns / tables** | Rejected at execution or nonsense answer | `is_read_only_sql()` blocks writes; schema text lists real tables; show SQL in CLI |
| **Stale or irrelevant memory** | Old Q&A skews answer | Top-k retrieval only; memory tool is optional — model can ignore weak hits; distance available for future thresholding |
| **Web noise / wrong page** | Weak synthesis | `read_url` strips nav/scripts; sources listed for verification; model instructed to ground in tool output |

## (c) Context too large (memory + tool results)

If prompts grow too large:

1. **Truncate tool payloads** — already cap SQL rows (20) and page text (6000 chars).
2. **Summarize older tool results** — replace raw JSON in history with a short assistant summary before the next LLM call.
3. **Limit memory injection** — reduce `MEMORY_TOP_K` or add a similarity threshold to drop low-relevance hits.
4. **Rolling window** — keep system prompt + latest user message + last N messages only for the chat API.
5. **Separate retrieval** — don’t dump full memory collection; only pass top-k snippets (current behavior).

Priority for a production version: summarization of tool outputs + message window, since they grow fastest during multi-step runs.

## Optional: query repair (implemented)

On SQL execution failure, the error message is passed back to the LLM for a corrected query, up to 2 retries (`SQL_MAX_RETRIES` in `config.py`).
