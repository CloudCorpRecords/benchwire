# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted LaraBench MCP server. Both captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `search_laravel_jobs` (recent Laravel jobs from the last 30 days, with description, salary, seniority and employment type) and `search_laravel_news` (recent Laravel ecosystem news and package releases from the last 7 days). Neither tool carries a required field; every argument is an optional filter.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `benchwire error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LARABENCH_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
