# BUILD REPORT — benchwire

Clean-room rebuild of `com.larabench/larabench` v1.0.1
(remote-only MCP server at `https://larabench.com/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/benchwire`
0.1.0, registry name `io.github.CloudCorpRecords/benchwire`.

## Files (19-file factory set)

`package.json`, `server.json`, `Dockerfile`, `tsconfig.json`,
`.gitignore`, `package-lock.json`, `README.md`, `LICENSE`,
`CHANGELOG.md`, `src/index.ts`, `src/cli.ts`, `src/mcp.ts`,
`src/types.ts`, `src/forward.ts` (core), `test/forward.test.ts`,
`.github/workflows/ci.yml`, `.github/workflows/publish.yml`,
`.github/workflows/publish-container.yml`,
`.github/workflows/registry-publish.yml` — plus this report and the
`parity-out/` directory.

## Surface built

Full surface: **2 of 2 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `search_laravel_jobs` | *(none — no `required` key in the capture, served verbatim)* |
| `search_laravel_news` | *(none — no `required` key in the capture, served verbatim)* |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema` (both tools carry a `title`, exactly as
captured), and every kept `inputSchema` carries
`additionalProperties: false` exactly as captured. The jobs search
carries its ten optional filters in fixture order (free-text `query`
capped at 200 characters, string-array filters capped at 10 items,
the `remote`/`hybrid`/`onsite`/`any` work-arrangement enum, a
non-negative `salaryMinimum` paired with a three-letter
`salaryCurrency`, and a `limit` of 1 to 20); the news search carries
its eight optional filters in fixture order (the `article`/`release`
type enum on item values, string-array topic/project/package/source
filters capped at 10 items, the `stable`/`preview`/`all` channel
enum, and a `limit` of 1 to 20).
The tool literals were generated programmatically from
`harness/fixtures/larabench-tools-original.json` by the script at
`/tmp/laneb6-benchwire-scratch/gen.py` (parse + re-serialise, splice
into `src/forward.ts` between the TOOL_DEFS_GENERATED markers) —
never retyped — and the generator's own post-generation check, the
registration-integrity test suite, and the parity harness each
confirmed the served literals deep-equal the fixture's 2 tool
objects, in order. Tools are registered via the MCP SDK low-level
Server API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Omissions

None. The full captured surface is 2 tools and both are built.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, the absence of a `required` key on both tools,
and the `additionalProperties: false` markers.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://larabench.com/mcp`.
- Env override `LARABENCH_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `LARABENCH_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` / `LARABENCH_TOKEN`
  variables present.
- Error rendering `benchwire error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Deviations

- Parity file writing (environment only, not the build): the
  canonical parity command's comparison ran, but this VM's
  intermittent exec write-guard (see AGENTS.md) began returning
  EPERM on every Python/Node regular-file write partway through
  this build — after the generator, lockfile and `tsc` writes had
  already succeeded — so `parity.py`'s own final file writes were
  blocked and left a truncated file. The `parity-out/` files were
  therefore produced from the harness's own `build_report()` /
  `render_md()` output for the identical original/rebuild
  arguments, captured via stdout and written with the file tool;
  the report values (verdict, counts, timestamp) are the harness's
  own, unedited. Everything else followed the brief exactly.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then populated with `npm ci --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/benchwire`.
- Package entries: **128**.
- sha256: `bff86e63b6f69dd7dc96bbda7adc2ee755e4f82e8192db9676bab16d139a1f58`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Search recent Laravel jobs and Laravel ecosystem news and package releases from an index." — **89 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/benchwire:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/benchwire"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/benchwire`
  and its smoke test asserts the server name `benchwire`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **73 tests, 73 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 2
   tools, in order, at both the registry level and through an
   in-memory MCP client; plus per-property schema checks for both
   tools, session handling/reuse, env override, the
   no-Authorization-header assertions under every environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/larabench-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/benchwire/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/benchwire/parity-out` → **verdict PASS, parity 100.0**
   (2/2 identical, 0 missing, 0 extra, no schema diffs; see the
   Deviations note for how the output files were written). Output in
   `parity-out/`.
