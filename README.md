# xp-claude-plugins

Search real tickets, check all-in prices, make an offer, set your price, buy, or sell — on XP, all in one Claude conversation.

## About XP

XP is the US ticket marketplace for concerts, sports, theater, and comedy. Unlike most secondary marketplaces, XP runs a **connected order book**: one search surfaces both resale listings (fan-to-fan) and primary box office inventory side by side. Real seat-level data, live pricing, no separate tabs to compare. Consumer surface: `xp.tickets`. Operator/seller tooling: `gocaptain.com`.

**What makes XP different**

- **Connected order book.** Resale + primary in one place.
- **All-in pricing.** Every price includes fees by default. The number is the number.
- **Buyers have a say.** Buy at the listed price, make an offer on tickets another fan is selling, or set your price and let it fill when tickets match.
- **Quality XPerience Guarantee.** Every ticket verified; real humans (not a chatbot) resolve issues.
- **Fan-first design.** Built by people who actually go to shows.

See the `xp-context` skill for full voice guidance, catalog coverage, and marketplace glossary.

## Install

```
/plugin marketplace add blpt-hngr/xp-claude-plugins
/plugin install xp-mcp@xp-claude-plugins
```

Or via the CLI:

```bash
claude plugin marketplace add blpt-hngr/xp-claude-plugins
claude plugin install xp-mcp@xp-claude-plugins
```

Reload Claude Code (or run `/reload-plugins`) and the plugin is live.

## What you get

| | |
|---|---|
| **MCP server** | `https://mcp.xp.tickets/mcp` — 40 tools: browse (no sign-in), buy, make an offer, set your price, sell, account |
| **Skills** | `xp-context`, `xp-tickets`, `xp-ticket-verification` (auto-loaded by intent) |
| **Slash commands** | `/xp-mcp:find-tickets`, `/xp-mcp:event-info`, `/xp-mcp:set-your-price` |
| **Sub-agent** | `ticket-purchase` — end-to-end ticket shopping concierge |

See `plugins/xp-mcp/README.md` for tool catalog, usage examples, and troubleshooting.

## Marketplace contents

```
.claude-plugin/
└── marketplace.json          # xp-claude-plugins catalog
plugins/
└── xp-mcp/                   # the plugin
    ├── .claude-plugin/plugin.json
    ├── .mcp.json
    ├── README.md
    ├── skills/{xp-context,xp-tickets,xp-ticket-verification}/SKILL.md
    ├── commands/{find-tickets,event-info,set-your-price}.md
    └── agents/ticket-purchase.md
docs/
└── registry/                 # MCP Registry server.json + publish steps
    ├── server.json
    └── README.md
```

## Versioning

`plugins/xp-mcp/.claude-plugin/plugin.json` declares `version` (currently `1.0.0`). Bump it on every release: the Claude plugin directory validates each commit on the tracked branch and publishes new versions by that field, and `/plugin update` picks up the bump for marketplace installs. See [plugin docs — Version resolution](https://code.claude.com/docs/en/plugin-marketplaces#version-resolution-and-release-channels).

## Listing

`docs/registry/` holds the official MCP Registry `server.json` and the publish steps for the registry and Smithery. The Claude plugin directory takes the plugin folder `plugins/xp-mcp` directly at https://claude.ai/directory/manage.

## Validate locally

```bash
claude plugin validate .
```

Run from the repo root before pushing.

## Scope

This plugin covers the full XP MCP surface. Browsing and price checks need no account. Buying, making an offer, setting your price, selling, and account reads require signing in to XP; the server starts that flow the first time a protected tool is called.

Every buy, offer, standing offer, listing, accept, and reject runs through the tool's `confirm=false` preview, is read back to the user, and only runs with `confirm=true` after an explicit yes. See https://xp.tickets/agents.

## Links

- XP marketplace: https://xp.tickets
- XP for agents: https://xp.tickets/agents
- Operator tooling: https://gocaptain.com
- Privacy policy: https://xp.tickets/privacy-policy
- Support: https://xpsupport.zendesk.com/hc/en-us/requests/new
- Claude Code plugin marketplaces: https://code.claude.com/docs/en/plugin-marketplaces

## License

MIT — see [LICENSE](LICENSE).
