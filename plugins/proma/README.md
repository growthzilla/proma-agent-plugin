# Proma

Work in [Proma](https://proma.ai) from your AI assistant. This plugin connects
your client to Proma's hosted MCP server and adds the `proma-tools` skill, which
teaches the model how to use it well.

## What you get

| Component | Detail |
| --- | --- |
| MCP server | `proma` - Streamable HTTP at `https://server.proma.ai/mcp/connectors` |
| Skill | `proma-tools` - how to work with Spaces, Systems, Datasets, columns, interfaces, forms, rows and automations through the server's tools |

With them you can:

- look around your workspace and find any space, system, dataset or interface;
- read rows, including exactly what a given interface's audience sees, and ask
  questions across datasets with SQL;
- write rows, with a dry run first for imports;
- build complete systems with datasets, interfaces and roles, and add to them;
- design forms;
- set up automations, check them, and debug failed runs.

## Sign-in

Sign-in is OAuth in your browser. When your client asks you to sign in to
Proma, it opens a Proma page where you approve access. There is nothing to
paste: no token, no API key. Every action runs with your own Proma permissions,
and a connection granted only read access cannot write.

## Install in Claude Code

```
/plugin marketplace add growthzilla/proma-agent-plugin
/plugin install proma@proma
```

Restart Claude Code (or start a new session), then run `/mcp` and authorize
`plugin:proma:proma`.

## Install in Codex

```
codex plugin marketplace add growthzilla/proma-agent-plugin
```

Then install `proma` from `/plugins`.

## Manual alternatives

To add only the MCP server, without the skill:

```
claude mcp add --transport http proma https://server.proma.ai/mcp/connectors --scope user
```

```
codex mcp add proma --url https://server.proma.ai/mcp/connectors
codex mcp login proma
```

## Links

- [Documentation](https://proma.ai/docs/)
- [Privacy policy](https://proma.ai/privacy/)
- [Terms of service](https://proma.ai/tos/)
- [Support](https://proma.ai/contact-us/)
