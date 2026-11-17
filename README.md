# benchwire

A wire into a Laravel index, over MCP. benchwire is a small forwarding client for the hosted LaraBench service: the tool list is served from your own machine using the captured definitions, and each call is sent upstream, with the service's answer returned exactly as it arrived.

Both of the service's captured tools are exposed, and both are read-only catalogue searches. One looks through recent Laravel jobs; the other looks through recent Laravel ecosystem news and package releases. No application is submitted, nothing is posted, and no person's details are collected by either search.

## The tools

- **`search_laravel_jobs`** — Search recent Laravel jobs indexed by LaraBench, limited to the last 30 days. Results include full description, salary, seniority and employment type, a LaraBench detail page and, when the source published one, the original apply URL. Every argument is optional: free-text `query`, `technologies`, `workArrangement` (`remote`, `hybrid`, `onsite` or `any`), `locations`, `countries`, `salaryMinimum` with a three-letter `salaryCurrency`, `seniority`, `employmentTypes`, and a `limit` of 1 to 20.
- **`search_laravel_news`** — Search recent Laravel ecosystem news and package releases indexed by LaraBench, limited to the last 7 days. Each item carries a LaraBench detail page and the original article or release notes URL. Every argument is optional: free-text `query`, `types` (`article` or `release`), `topics`, `projects`, `packages`, `sources`, a release `channel` (`stable`, `preview` or `all`), and a `limit` of 1 to 20.

Neither tool has a required field — call either with an empty object to see the newest entries in its window.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/benchwire mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "benchwire": {
      "command": "npx",
      "args": ["-y", "@reneturcios/benchwire", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/benchwire
docker run --rm ghcr.io/cloudcorprecords/benchwire search_laravel_jobs '{}'
```

## CLI

A single binary covers both the command line and the MCP server. Every tool is a subcommand whose one argument is a JSON string of that tool's inputs:

```bash
benchwire search_laravel_jobs '{"query":"inertia","workArrangement":"remote","limit":5}'
benchwire search_laravel_news '{"topics":["laravel"],"channel":"stable"}'
benchwire --version
```

Exit codes: `0` for a successful forwarded call, `1` when the result that comes back is an error, and `2` for usage mistakes like an unknown subcommand or a JSON argument that does not parse.

## Configuration

- `LARABENCH_MCP_URL` — Replaces the upstream endpoint. Defaults to the pinned hosted endpoint, `https://larabench.com/mcp`. (This override variable is a reconstruction introduced by this rebuild.)

The LaraBench registry record lists no headers for its remote, so benchwire never attaches an authentication header — every call goes upstream keyless.

When the upstream cannot answer — it returns a JSON-RPC error, answers with a non-2xx status, sends a body that cannot be read, or the request itself fails — the caller receives an error result whose text begins with `benchwire error:`.

## How forwarding works

The first call of a process opens a session against the upstream: an `initialize` request (protocol version `2025-03-26`), a `notifications/initialized` notification, and then the `tools/call`. If the service issues an `Mcp-Session-Id` header during initialize, benchwire repeats it on subsequent requests and keeps reusing that one session until the process ends. Replies are understood in both plain JSON and server-sent-events framing. Successful results are passed through untouched, `isError` flag and any extra fields included.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The suite compares the served tool list against the captured surface tool for tool, walks the session handshake and its reuse, proves no `Authorization` header is sent under any environment, and confirms that arguments travel upstream unchanged and results return unchanged.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
