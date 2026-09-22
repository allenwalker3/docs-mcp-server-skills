# CLAUDE.md snippet for projects that use `docs-lookup`

Paste the block below into a project's `CLAUDE.md` and fill in the placeholders. It holds
**facts about this project's docs server and indexed libraries only**; query technique lives
in the `docs-lookup` skill, so do not copy it here.

Get the exact library names with the `list_libraries` tool (or the web UI) and copy them
verbatim. Credentials for a remote server belong in the MCP client config, never in this file.

---

```markdown
## Documentation lookup: docs-mcp-server — REQUIRED

This project's library docs are indexed in a docs-mcp-server instance.

| | |
|---|---|
| MCP server name | `docs-mcp-server` (tools are `mcp__docs-mcp-server__*`) |
| Endpoint | `http://localhost:6280/mcp` <!-- or https://docs.example.com/mcp --> |
| Web UI | `http://localhost:6280` <!-- 6281 if web runs as a separate container --> |
| Health check | `curl -s -o /dev/null -w '%{http_code}\n' http://localhost:6280/` |
| Write tools | read-only <!-- or: allowed; see "Re-indexing" --> |

### Indexed libraries

| `library` | Versioned? | `version` to pass | Docs root | `.md` pages? | Last indexed |
|---|---|---|---|---|---|
| `react` | no | omit | https://react.dev/reference | yes | 2026-09-21 |
| `typescript` | yes | installed major as `N.x` (from `package.json`) | https://www.typescriptlang.org/docs | no | 2026-09-01 |

**Before writing, editing, or debugging code that uses a library in this table, invoke the
`docs-lookup` skill and query this server.** Do not answer from memory about these
libraries. For a library that is not listed, call `list_libraries` first; the table may be
behind the index.

**Source precedence:** for the libraries above, this server comes first. Use Context7,
WebFetch, or web search only as the fallback that `docs-lookup` describes, and say in your
reply which source you used.

<!-- Optional: only when write tools are allowed and a library needs a special recipe. -->
### Re-indexing (only when explicitly asked)

Any scrape of an existing library and version wipes it first. `<library>` must be rebuilt
with the CLI because it needs `--header "Accept: text/html"`, which the MCP `scrape_docs`
tool cannot set:

    <exact CLI command>

Afterwards, update "Last indexed" above.
```

---

## Why each field is there

- **MCP server name.** The tool prefix comes from the name in the user's MCP config, not
  from the server. The skill refers to tools by bare name (`search_docs`), so the agent
  needs this to find them.
- **Health check.** A local Docker setup and a cloud deployment are checked differently.
  A plain HTTP probe works for both.
- **Write tools.** A server started with `DOCS_MCP_READ_ONLY=true` hides `scrape_docs`,
  `refresh_version`, and `remove_docs`. Shared and cloud servers often run this way.
- **Versioned? / `version` to pass.** Omitting `version` on a versioned library searches
  the *latest* indexed version, which may be newer than what the project installs. Passing
  the installed major (`18.x`) or the exact version (resolves to that version or the
  nearest older one) matches the code in the repo. On an unversioned library any `version`
  falls back to the unversioned docs, so passing one is harmless there.
- **`.md` pages?** Tells the agent whether `<url>.md` returns lean Markdown when it
  fetches the full page behind a hit.
- **Last indexed.** Lets the agent judge staleness against the installed version.
- **Source precedence.** Other doc servers such as Context7 tell the agent to use them
  for every library. Without an explicit order, the two compete.
