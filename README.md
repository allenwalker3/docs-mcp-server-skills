# docs-lookup: an agent skill for docs-mcp-server

`docs-lookup` teaches Claude Code and other coding agents to query a
[docs-mcp-server](https://github.com/arabold/docs-mcp-server) (Grounded Docs MCP Server)
documentation index well. The server is hybrid keyword and vector search over the docs
you index, running locally or in the cloud. The skill covers how to use it: the query
phrasing that finds the right page first, which `version` to pass, when to fetch the full
page, what to do when the server is down, and how to catch an index that scraped badly.

It follows the [Agent Skills](https://agentskills.io) format, so it works with any agent
that loads `SKILL.md` skills.

## What it fixes

- **Question-shaped queries.** The keyword half of the search ORs every word, so "how do I
  make my effect run once" drags in noise. The skill has the agent write documentation
  language instead: `useEffect cleanup dependencies`.
- **Wrong-version answers.** Leaving out `version` searches the *latest* indexed version,
  not the one your project installs. The skill has the agent read the installed version
  and pass `N.x`.
- **Answering from excerpts.** Search results are chunks. The skill has the agent fetch
  the winning page in full, with the `#fragment` stripped and `.md` appended where the
  site supports it.
- **Silent guessing.** When the server is unreachable, the agent falls back in a fixed
  order and tells you which source it used.
- **Bad indexes.** A three-probe smoke test catches JavaScript-only shells, truncated
  crawls, and Markdown-negotiated sites that lose their navigation.

## Requirements

- A running [docs-mcp-server](https://github.com/arabold/docs-mcp-server) with at least
  one library indexed.
- The server connected to your agent as an MCP server, for example:

  ```bash
  claude mcp add --transport http docs-mcp-server http://localhost:6280/mcp
  ```

## Install

**Skills CLI** (Claude Code, Cursor, Codex, Gemini CLI, Copilot, and others):

```bash
npx skills add allenwalker3/docs-mcp-server-skills
```

**Claude Code plugin:**

```bash
claude plugin marketplace add allenwalker3/docs-mcp-server-skills
claude plugin install docs-lookup@docs-mcp-server-skills
```

**Manual:** copy `skills/docs-lookup/` into `~/.claude/skills/` (all projects) or
`.claude/skills/` (one project).

## Set up a project

The skill holds query technique. Each project's `CLAUDE.md` (or `AGENTS.md`) holds the
facts: the server's name and address, a health check, which libraries are indexed, and
whether each is versioned. Copy the block in
[`skills/docs-lookup/claude-md-template.md`](skills/docs-lookup/claude-md-template.md) and
fill it in, or ask your agent to do it with the skill loaded.

The template also sets source order: the local index first for the listed libraries, and
Context7 or the web only as a fallback. Without that line, other documentation servers
that ask to be used for every library compete with this one.

## Relation to the upstream skills

The docs-mcp-server repo ships [its own skills](https://github.com/arabold/docs-mcp-server/tree/main/skills)
(`docs-search`, `docs-manage`, `fetch-url`). Those drive the server through its `npx` CLI.
`docs-lookup` works through the MCP tools an agent already has connected and focuses on
getting good results from them. The two sets can be installed together.

## License

[MIT](LICENSE)
