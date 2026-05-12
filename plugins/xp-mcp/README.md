# xp-mcp

XP ticketing on Claude Code — the open-world surface. Public discovery (events, venues, performers, listings, price-alert links) with no sign-in required.

## Install

```
/plugin marketplace add blpt-hngr/xp-plugins
/plugin install xp-mcp@xp-plugins
```

Or via the CLI:

```bash
claude plugin marketplace add blpt-hngr/xp-plugins
claude plugin install xp-mcp@xp-plugins
```

Local testing from a clone of this repo:

```
/plugin marketplace add ./
/plugin install xp-mcp@xp-plugins
```

Reload after install: `/reload-plugins`.

## What you get

| | |
|---|---|
| **MCP server** | `https://mcp.xp.tickets/mcp` (anonymous, 9 open-world tools) |
| **Skills** | `xp-context` (background + glossary), `xp-tickets` (search → details → listings), `xp-ticket-verification` (validate seat data) |
| **Slash commands** | `/xp-mcp:find-tickets`, `/xp-mcp:event-info`, `/xp-mcp:price-alert` |
| **Sub-agent** | `ticket-purchase` — end-to-end ticket shopping concierge |

### Open-world tools

`search_events`, `get_event_details`, `search_performers`, `search_venues`, `get_venue_details`, `get_ticket_listings`, `get_price_alert_link`, `list_price_alert_sections`, `auth_status`.

All return live data. Money fields use integer cents on input; convert to dollars with `$` symbol when presenting.

## How to use

**Just ask** — skills auto-load based on intent:

```
Find me 2 tickets to a Knicks game in November under $200
What's the cheapest seat for Taylor Swift at MSG?
I have section 104, row B, seats 4-6 for Knicks Nov 14, verify it
```

**Slash commands** for explicit invocation:

```
/xp-mcp:find-tickets Knicks vs Celtics
/xp-mcp:event-info Madison Square Garden Dec 15
/xp-mcp:price-alert Taylor Swift under $300
```

**Sub-agent** for end-to-end flows:

```
Use the ticket-purchase agent to find me 2 tickets to a Knicks game in November under $200
```

## What this plugin won't do

- Account or wallet reads, alerts (no OAuth).
- Marketplace mutations (buy/sell offers, accept/reject).

Those live in `xp-pro-mcp`, an internal plugin in a separate marketplace (not in this public repo).

## Troubleshooting

- **Plugin missing after install** → `/reload-plugins` or restart Claude Code.
- **MCP tools not listed** → `/mcp` to confirm `xp` server attached. If absent, check `~/.claude/plugins/cache/xp-plugins/xp-mcp/<sha>/.mcp.json`.
- **Wrong content cached** → `claude plugin uninstall xp-mcp@xp-plugins` then `claude plugin install xp-mcp@xp-plugins`. Local-dir marketplaces don't auto-bump version on file edits.

## See also

- The XP MCP server itself: https://mcp.xp.tickets/mcp
- XP marketplace: https://xp.tickets
- [Claude Code plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
