# Live Web Data CLI

How fresh is the web your AI reads? Live data, explained plainly.

`livewebdata` fetches live web pages as clean Markdown, text or HTML, straight from your terminal or your agent's shell.
It is the command line client for the [Live Web Data API](https://livewebdata.com).

## Quick start

```bash
npx @livewebdata-com/cli login                                   # paste your API key (not echoed)
npx @livewebdata-com/cli fetch https://example.com -o text       # the page as Markdown
```

Get a key at [livewebdata.com](https://livewebdata.com). Node.js 18 or later.

## Install

```bash
npm install -g @livewebdata-com/cli
livewebdata --help
```

## Use

```bash
livewebdata fetch https://example.com/a https://example.com/b      # up to 10 URLs; JSON with results[] and statuses[]
livewebdata fetch https://example.com -o text                      # just the content (Markdown, else text, else HTML)
livewebdata fetch https://example.com -o html --html               # the page HTML
livewebdata fetch https://example.com --country-code de --render   # from Germany, rendered in a real browser
livewebdata fetch --data @request.json                             # the whole request body from a file (- for stdin)
livewebdata account                                                # plan, credits and rate limit
livewebdata fetch --help                                           # every option, with the API's own descriptions
```

A fetch that is still running after about 100 seconds is collected automatically with the same
`Idempotency-Key`, so it is charged once.

## MCP server

The Live Web Data API is also a remote MCP server (Streamable HTTP) at `https://livewebdata.com/mcp`, with the tools
`get_page_contents` and `get_account_status`. It takes the same API key as a bearer token.

```bash
claude mcp add --transport http live-web-data https://livewebdata.com/mcp \
  --header "Authorization: Bearer $LIVE_WEB_DATA_API_KEY"
```

Any other client: URL `https://livewebdata.com/mcp`, header `Authorization: Bearer <key>`. Config for Cursor, VS Code and
Claude Desktop: [MCP docs](https://livewebdata.com/docs/mcp). The server's entry for the MCP Registry is [server.json](server.json)
(`com.livewebdata/live-web-data`).

## Agent plugin and skill

This repo is also a plugin for Claude Code, Grok Build and Gemini CLI, and holds an agent skill. Each one connects the
MCP server above and teaches the agent when to fetch a page with it or with `livewebdata`.

```bash
npx skills add livewebdata-com/cli                                    # the skill, for any agent that reads SKILL.md
claude plugin marketplace add livewebdata-com/cli && claude plugin install livewebdata@livewebdata
gemini extensions install https://github.com/livewebdata-com/cli
```

Network and credentials: the plugin calls `https://livewebdata.com/mcp` only. The CLI calls `https://livewebdata.com/api` and
`https://livewebdata.com/openapi.json`, and `https://registry.npmjs.org` (`npx` and a once-a-day update check, off with
`LIVEWEBDATA_NO_UPDATE_NOTIFIER=1`). Both need one credential, your Live Web Data API key,
read from `LIVE_WEB_DATA_API_KEY` or saved by `livewebdata login`. No hooks, no scripts run on install.

## The commands follow the live API

`livewebdata` builds its commands and options from the API's OpenAPI document at
[https://livewebdata.com/openapi.json](https://livewebdata.com/openapi.json), cached for an hour. When the API gains a field or an endpoint, the
option or command is there on your next run, without updating the CLI. Offline, it uses the last cached
copy, then the copy bundled with the installed version (`livewebdata spec` shows which one; `--offline` skips the
download). An hourly job here publishes a new version whenever the API spec changed.

## Configuration

| Setting | Option | Environment | Default |
|---|---|---|---|
| API key | `--api-key` | `LIVE_WEB_DATA_API_KEY` | the key saved by `livewebdata login` (`~/.config/livewebdata/config.json`, mode 0600) |
| API base URL | `--base-url` | `LIVE_WEB_DATA_API_BASE_URL` | `https://livewebdata.com/api` |
| API spec URL | `--spec-url` | `LIVE_WEB_DATA_SPEC_URL` | `https://livewebdata.com/openapi.json` |
| Output | `-o json\|text\|<field>` | | `json` |
| Update notice | | `LIVEWEBDATA_NO_UPDATE_NOTIFIER=1` turns it off | once a day, terminals only |

Requests and your key only ever go to the base URL, never to a server named in the spec.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | OK |
| 1 | The API answered with an error (JSON on stderr, with its `requestId`), every URL failed with `-o text`, or the fetch was still running at `--max-wait` |
| 2 | Usage: unknown command or option, a value the API does not accept |
| 3 | Network: no answer. The error carries the `Idempotency-Key` to collect the result without a new charge |

## Links

- [livewebdata.com](https://livewebdata.com)
- [CLI docs](https://livewebdata.com/docs/cli)
- [API docs](https://livewebdata.com/docs)
- [API spec](https://livewebdata.com/openapi.json)

## License

MIT, © livewebdata.com
