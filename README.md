# Proma agent plugins

A plugin marketplace for [Claude Code](https://claude.com/claude-code) and
[Codex](https://github.com/openai/codex), hosted as a plain git repository. It
contains one plugin, `proma`, which registers Proma's hosted MCP server and
ships a skill for working with Proma through it.

## Install in Claude Code

Two commands inside Claude Code:

```
/plugin marketplace add growthzilla/proma-agent-plugin
/plugin install proma@proma
```

Restart Claude Code (or start a new session) so the MCP server connects and the
skill loads. Run `/mcp` to check the `plugin:proma:proma` server's status and to
authorize it the first time.

The install id is `proma@proma` - `<plugin name>@<marketplace name>`. It does
not come from the repo name.

## Install in Codex

```
codex plugin marketplace add growthzilla/proma-agent-plugin
```

Then install `proma` from `/plugins`. To add only the MCP server without the
skill, see [plugins/proma/README.md](plugins/proma/README.md).

## Zero-command alternative

To have a repo set this up for everyone who opens it, commit a
`.claude/settings.json` in that repo:

```json
{
  "extraKnownMarketplaces": {
    "proma": {
      "source": {
        "source": "github",
        "repo": "growthzilla/proma-agent-plugin"
      }
    }
  },
  "enabledPlugins": {
    "proma@proma": true
  }
}
```

Claude Code registers the marketplace and enables the plugin on first launch in
that directory, so nobody has to run the two commands above. Anyone who has not
used this marketplace before is asked once to trust it.

## What the plugin provides

| Component | Detail |
| --- | --- |
| MCP server | `proma` - streamable HTTP at `https://server.proma.ai/mcp/connectors` |
| Skill | `proma-tools` - how to work with Spaces, Systems, Datasets, columns, interfaces, forms, rows and automations through the MCP tools |

## Authentication

There are no credentials, tokens, or API keys in this repository, and none are
needed here. The client (Claude Code or Codex) negotiates auth with
`server.proma.ai` over OAuth and stores the resulting credentials locally on the
user's machine. If the server shows as unauthorized in Claude Code, run `/mcp`
and authorize `plugin:proma:proma` (or `proma`, if you added it with
`claude mcp add`). If you added it to Codex with `codex mcp add`, run
`codex mcp login proma`.

## Layout

```
.claude-plugin/marketplace.json      the marketplace index (must be at the repo root)
plugins/proma/
  .claude-plugin/plugin.json         the plugin manifest (must be at the plugin root)
  .codex-plugin/plugin.json          OpenAI / Codex listing metadata
  .mcp.json                          auto-scanned MCP server definition
  assets/                            icon and logo
  skills/proma-tools/SKILL.md        the skill
  README.md                          the plugin's own README
LICENSE                              MIT
```

The `version` must stay identical in three places, or installed copies never
see updates:

- `plugins/proma/.claude-plugin/plugin.json`
- `plugins/proma/.codex-plugin/plugin.json`
- the `proma` entry in `.claude-plugin/marketplace.json`

`claude plugin tag --dry-run` checks the two Claude files. Check the Codex
manifest with `jq -r .version plugins/proma/.codex-plugin/plugin.json`.

## License

MIT. See [LICENSE](LICENSE).
