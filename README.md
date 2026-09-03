# Proma agent plugins

A [Claude Code](https://claude.com/claude-code) plugin marketplace, hosted as a
plain git repository. It contains one plugin, `proma`, which registers Proma's
hosted MCP server and ships a skill for working with Proma through it.

Nothing here is published to Anthropic. This repo *is* the hosting: installing
is a `git clone` performed by your own git credentials.

## Install

Two commands inside Claude Code:

```
/plugin marketplace add growthzilla/proma-agent-plugin
/plugin install proma@proma
```

Restart Claude Code (or start a new session) so the MCP server connects and the
skill loads. Run `/mcp` to check the `proma` server's status and to authorize it
the first time.

The install id is `proma@proma` - `<plugin name>@<marketplace name>`. It does
not come from the repo name.

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
| Skill | `proma-tools` - how to work with Spaces, Systems, Sheets, columns, views, rows, automations and forms through the MCP tools |

## Authentication

There are no credentials, tokens, or API keys in this repository, and none are
needed here. Claude Code negotiates auth with `server.proma.ai` at connect time
over OAuth and stores the resulting credentials locally on the user's machine.
If the server shows as unauthorized, run `/mcp` and authorize `proma`.

## Layout

```
.claude-plugin/marketplace.json      the marketplace index (must be at the repo root)
plugins/proma/
  .claude-plugin/plugin.json         the plugin manifest (must be at the plugin root)
  .mcp.json                          auto-scanned MCP server definition
  skills/proma-tools/SKILL.md        the skill
```

The `version` in `plugins/proma/.claude-plugin/plugin.json` and in the
marketplace entry must stay identical, or installed copies never see updates.
`claude plugin tag --dry-run` checks that they agree.
