# PolicyLayer Registry MCP Server

**The MCP server that vets MCP servers.**

Connect your agent to the PolicyLayer registry and it can ask one question before installing or allowing any MCP server: *is this safe?* The answer is the published registry record: verified identity with the evidence, an A to F risk grade, probed auth posture and every tool's risk classification.

- **Endpoint:** `https://api.policylayer.com/mcp` (Streamable HTTP)
- **Free single-server lookups**, no key needed
- 32,500+ published servers and 515,000+ classified tools, kept current by continuous scanning
- **Docs:** [policylayer.com/registry/mcp](https://policylayer.com/registry/mcp)

## Connect

**Claude Code**

```sh
claude mcp add --transport http policylayer https://api.policylayer.com/mcp
```

**Claude (desktop / web)**

Settings → Connectors → Add custom connector → URL `https://api.policylayer.com/mcp`

**Cursor / Windsurf / VS Code**

```json
{ "mcpServers": {
  "policylayer": {
    "url": "https://api.policylayer.com/mcp"
} } }
```

Then tell your agent: *"check any MCP server against PolicyLayer before installing it."* Add it to your team's agent instructions and every install gets vetted.

## Tools

| Tool | What it returns | Tier |
|------|-----------------|------|
| `check_mcp_server` | Full record for one server: identity + evidence, risk grade, posture, tools riskiest-first. Unknown servers are queued for scanning by the lookup itself. | Free |
| `search_registry` | Published servers matching a name, slug or package substring, with grade and verification on every match. | Free |
| `check_tool` | One tool's full classification: risk analysis and evidence, OWASP classes, parameters, recommended policy default. | Free |
| `get_change_events` | The ordered change feed: drift, posture flips, impostor flags. Cursor-based, with severity filters. | Licence |

## Example

Ask your agent *"is the github MCP server safe to install?"* and it calls:

```json
{ "name": "check_mcp_server", "arguments": { "server": "github" } }
```

and gets back the published record (trimmed):

```json
{
  "slug": "github",
  "riskGrade": "D",
  "identity": { "verification": "official", "confidence": "verified" },
  "posture": { "auth": "gated", "rateLimited": false },
  "toolCount": 86,
  "tools": [
    { "name": "delete_file", "category": "Destructive", "severity": "High", "recommendedPolicy": { "action": "deny" } }
  ]
}
```

## Things to ask your agent

- *"Is the github MCP server safe to install?"*
- *"We're adding the notion and linear MCP servers this week. Vet both and summarise the risks."*
- *"Which of the MCP servers in this project can delete data or move money?"*
- *"Should I allow the execute_sql tool on the postgres server? What policy does PolicyLayer recommend?"*
- *"Find a verified MCP server for Slack and show me its risk grade."*
- *"Check every server in my mcpServers config against PolicyLayer and flag anything below grade C."*
- With a licence key: *"What changed in the servers we depend on since yesterday? Any posture flips or impostor flags?"*

## Adding a licence key

`get_change_events` needs a Registry Licence key (`plr_...`), self-serve at [policylayer.com/registry/pricing](https://policylayer.com/registry/pricing).

- **Claude Code:** append `--header "Authorization: Bearer plr_..."` to the command above
- **Cursor / Windsurf / VS Code:** add `"headers": { "Authorization": "Bearer plr_..." }` inside the server entry
- Claude desktop/web connectors can't send headers; free lookups only

## Free and licensed

Everything about a single server is free and complete: the full record, every tool, the whole classification. A Registry Licence adds breadth across the catalogue: the change feed above, plus bulk snapshots, keyset paging and webhooks on the [/v1 API](https://policylayer.com/registry/api).

## What this repository is

The connect kit for a hosted service. The server runs at `api.policylayer.com`; there is no code to run from here. `server.json` is the listing published to the official MCP Registry as `com.policylayer/registry`. The MIT licence covers the contents of this repository only; the hosted service and the registry data are licensed separately.

---

[PolicyLayer](https://policylayer.com) · [Registry lookup](https://policylayer.com/registry) · [API reference](https://policylayer.com/registry/api) · [Licensing](https://policylayer.com/registry/pricing)
