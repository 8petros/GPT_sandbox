# Periodic multi-engine web-search workflow for n8n

Import [`periodic-web-search.json`](periodic-web-search.json) into a **self-hosted** n8n instance to search several logical expressions on a schedule and append new results to a Markdown log.

## What it does

- Runs every six hours (change the **Every 6 hours** trigger as needed).
- Searches every expression configured in **Build logical searches**. Expressions support engine syntax such as `AND`, `OR`, parentheses, quotes, and exclusions.
- Sends each expression to one SearXNG endpoint with the configured `google`, `bing`, and `brave` backends. SearXNG's response identifies the result's source engine.
- Timestamps every newly discovered result in UTC, records the configured matching keywords, title, clickable Markdown link, and result lead/snippet.
- Uses n8n workflow static data keyed by normalized result URL, so results already written by production executions are not appended again.
- Appends the formatted records to one Markdown file.

## Setup

1. In n8n, choose **Workflows → Import from File** and select `periodic-web-search.json`.
2. Edit the `config` object in **Build logical searches**:
   - Add or remove objects in `searches` for the queries and keyword labels you want logged.
   - Update `engines` to search the engines enabled by your SearXNG deployment.
3. Set environment variables on the n8n worker/container:
   - `SEARXNG_URL` — full SearXNG search endpoint, for example `https://search.example.com/search`. It must allow JSON output and the selected engines. The workflow has `https://searx.be/search` only as a convenience fallback; use your own instance for reliable scheduled operation.
   - `SEARCH_LOG_PATH` — optional output file path. Defaults to `/files/periodic-web-search.md`.
4. Ensure the n8n process can write the chosen directory and has `base64` available. The final **Append Markdown file** Execute Command node creates the parent directory and appends a UTF-8 decoded block safely using base64.
5. Execute once manually to verify access, then activate the workflow. Static data is persisted only for production/active executions, which is when URL de-duplication takes effect.

## Markdown record format

```markdown
## 2026-09-16 12:34:56 UTC — Result title

- **Matching keywords:** AI, LLM, agent, workflow
- **Query:** (AI OR LLM) AND (agent OR workflow) -job
- **Search engine:** google
- **Link:** [Open result](https://example.com/article)
- **Lead:** Search-result excerpt.
```

## Operational notes

- This workflow uses **Execute Command**, so it is intended for self-hosted n8n; n8n Cloud does not support writing a local file this way. Replace that node with S3, Google Drive, or another storage node for Cloud.
- The static URL index grows with every unique result. For large, long-lived workflows, replace it with an n8n Data Table/database and enforce a unique normalized URL there.
- Search syntax and available backend names depend on the SearXNG instance. If a backend does not support a particular operator, refine the matching query in the configuration node.
