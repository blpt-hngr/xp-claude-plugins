# xp-claude-plugins

Claude Code plugin marketplace for [XP](https://xp.tickets), the fan-first ticket marketplace.

This repo hosts the **`xp-claude-plugins`** marketplace and the **`xp-mcp`** plugin — XP's public, open-world MCP surface for Claude Code. One command installs the hosted MCP server, ticketing skills, slash commands, and a ticket-purchase concierge agent.

## About XP

XP is the US ticket marketplace for concerts, sports, theater, and comedy. Unlike most secondary marketplaces, XP runs a **connected order book**: one search surfaces both resale listings (fan-to-fan) and primary box office inventory side by side. Real seat-level data, live pricing, no separate tabs to compare. Consumer surface: `xp.tickets`. Operator/seller tooling: `gocaptain.com`.

**What makes XP different**

- **Connected order book.** Resale + primary in one place.
- **All-in pricing.** Every price includes fees by default. The number is the number.
- **Price alerts.** Watch a section, a price ceiling, or a specific event.
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
| **MCP server** | `https://mcp.xp.tickets/mcp` — anonymous, 9 open-world tools: search events, venues, performers; ticket listings; price-alert links |
| **Skills** | `xp-context`, `xp-tickets`, `xp-ticket-verification` (auto-loaded by intent) |
| **Slash commands** | `/xp-mcp:find-tickets`, `/xp-mcp:event-info`, `/xp-mcp:price-alert` |
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
    ├── commands/{find-tickets,event-info,price-alert}.md
    └── agents/ticket-purchase.md
```

## Versioning

The plugin manifest doesn't declare a `version`, so Claude Code uses the git commit SHA. Every push to `main` is a new release — users get it on next `/plugin update` or background auto-update.

To pin an explicit release, add `version` to `plugins/xp-mcp/.claude-plugin/plugin.json` and bump on every change. See [plugin docs — Version resolution](https://code.claude.com/docs/en/plugin-marketplaces#version-resolution-and-release-channels).

## Validate locally

```bash
claude plugin validate .
```

Run from the repo root before pushing.

## Scope

This repo ships **public, open-world** features only. Account-aware features (your tickets, wallet, favorites, alerts) and marketplace mutations (buy/sell offers) require XP OAuth and live in a separate internal plugin (`xp-pro-mcp`) not in this repo.

## Links

- XP marketplace: https://xp.tickets
- Operator tooling: https://gocaptain.com
- Claude Code plugin marketplaces: https://code.claude.com/docs/en/plugin-marketplaces

## License

MIT — see [LICENSE](LICENSE).
