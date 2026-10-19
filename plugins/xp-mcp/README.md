# xp-mcp

Find tickets, check all-in prices, make an offer, set your price, buy, or sell — on XP, inside Claude Code.

XP (xp.tickets) is a ticket marketplace for concerts, sports, theater, and comedy across the U.S. and Canada. On most resale sites, sellers set a price and buyers take it or leave it. On XP, buyers have a say too: buy at the listed price, make an offer on tickets another fan is selling, or set the price you'd go for and let it fill when tickets match. Every price is all-in. The number you see is the number you pay.

Searching and price checks need no account. Claude asks you to sign in to XP only when you're ready to buy, make an offer, or sell. More for people at https://xp.tickets/agents.

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

Local testing from a clone of this repo:

```
/plugin marketplace add ./
/plugin install xp-mcp@xp-claude-plugins
```

Reload after install: `/reload-plugins`.

## What you get

| | |
|---|---|
| **MCP server** | `https://mcp.xp.tickets/mcp` — 40 tools: browse, buy, offer, set your price, sell, account |
| **Skills** | `xp-context` (background + glossary), `xp-tickets` (search → prices → buy / offer / set your price), `xp-ticket-verification` (validate seat data) |
| **Slash commands** | `/xp-mcp:find-tickets`, `/xp-mcp:event-info`, `/xp-mcp:set-your-price` |
| **Sub-agent** | `ticket-purchase` — end-to-end ticket shopping concierge |

## Ask before you act

Buying, making an offer, setting your price, listing tickets, and accepting or rejecting an offer are all binding. Every one of those tools runs in two phases:

1. `confirm=false` — a preview. Nothing is placed or charged.
2. Claude reads the preview back to you: event, date, section, quantity, all-in total.
3. `confirm=true` — only after you say yes, in this conversation.

Claude never skips the preview and never confirms on an earlier yes.

## Tools

Live from the server (v4.0.9). Money is dollars everywhere: `max_price_usd`, `amount_per_ticket_usd`, `ask_per_ticket_usd`.

### Browse (no sign-in)

| Tool | What it does |
|---|---|
| `search_events` | Find events by performer, team, venue, city, or keyword |
| `search_performers` | Find an artist, team, or comedian |
| `search_venues` | Find a venue by name, city, or area |
| `get_venue_details` | Venue info plus what's coming up there |
| `get_event_details` | Full event card: venue, performers, price preview, images |
| `get_ticket_listings` | Ticket options and all-in prices for one event |
| `search_market` | Events with both the get-in price and whether fans are selling |
| `get_market_read` | Lowest all-in price per area of the building, plus fan listings open to offers |
| `get_recent_sales` | What tickets actually sold for recently |
| `get_price_history` | Daily get-in price over time |
| `get_seat_map` | Venue seating chart (layout, not a seat view) |
| `check_listing` | Poll a listing's status by correlation id (advanced; needs the `X-XP-Correlation-Id` header) |

### Buy

| Tool | What it does |
|---|---|
| `buy_tickets` | Buy listed tickets at the all-in price. Default rail is a Stripe card checkout link; preview, pay, then confirm with `checkout_session_id` |
| `buy_listing_now` | Buy a fan listing outright at the seller's ask, from your XP USDC balance |
| `get_order_status` | Status of one order by `order_uuid` |
| `get_my_orders` | Your recent orders and where each stands |

### Make an offer

| Tool | What it does |
|---|---|
| `make_offer_on_listing` | Offer a whole-dollar price per ticket on a fan listing. The seller can accept, counter, or decline; you're charged only if they accept |
| `cancel_my_offer` | Pull an offer you made |
| `get_my_bids` | Offers you've made, open or completed |
| `list_open_listings` | Fan listings open to offers, filterable by event, performer, amount |
| `get_open_listing` | One listing's full record before you make an offer |
| `mute_listing` | Hide (or unhide) a listing from your own view |
| `list_muted_listings` | What you've hidden |

### Set your price

| Tool | What it does |
|---|---|
| `create_standing_bid` | Rest a **standing offer** at your price across one or more events and sections. It fills itself when a matching fan listing appears — it is a binding offer, not a notification |
| `list_my_standing_bids` | Standing offers you have resting |
| `get_standing_bid` | One standing offer, including any fills |
| `rescind_standing_bid` | Close a standing offer to new fills. Fills already taken are completed purchases |

### Sell

| Tool | What it does |
|---|---|
| `create_listing` | List tickets you hold. Section, row, seats, and ADA / obstructed-view / standing-room answers are read back before it's created. New listings are reviewed before going live |
| `get_my_listing_status` | Open offers, state, and transfer instructions for one of your listings |
| `list_my_listings` | Your listings by lifecycle category |
| `accept_offer` | Accept a buyer's offer. Binding, irreversible, and starts a transfer deadline XP sets |
| `reject_offer` | Decline a buyer's offer |
| `cancel_my_listing` | Take a listing down. Every open offer on it is rejected |

### Account

| Tool | What it does |
|---|---|
| `auth_status` | Am I signed in? Only when asked, or after a sign-in error |
| `get_user_account` | Profile, order history, referral stats |
| `get_my_tickets` | Tickets delivered to your XP account |
| `get_my_wallet` | XP wallet address and USDC balance |
| `list_my_favorites` | Performers you follow |
| `get_my_referral_kickbacks` | Referral totals and rows |
| `get_my_context` | Budget, quantity, and notes you've already told XP |

## How to use

**Just ask** — skills auto-load based on intent:

```
Find me 2 tickets to a Knicks game in November under $200
What's the lowest all-in price for Taylor Swift at MSG?
Offer $80 a seat on those Phish tickets
Set my price at $120 for any of the three Lakers home games in December
Sell my two seats for Friday, section 104 row C
I have section 104, row B, seats 4-6 for Knicks Nov 14, verify it
```

**Slash commands** for explicit invocation:

```
/xp-mcp:find-tickets Knicks vs Celtics
/xp-mcp:event-info Madison Square Garden Dec 15
/xp-mcp:set-your-price Taylor Swift at $300
```

**Sub-agent** for end-to-end flows:

```
Use the ticket-purchase agent to find me 2 tickets to a Knicks game in November under $200
```

## Signing in

Protected tools return a sign-in error the first time; your MCP client then starts XP's OAuth flow. Claude calls the tool you asked for directly rather than checking `auth_status` first.

## Troubleshooting

- **Plugin missing after install** → `/reload-plugins` or restart Claude Code.
- **MCP tools not listed** → `/mcp` to confirm `xp` server attached. If absent, check `~/.claude/plugins/cache/xp-claude-plugins/xp-mcp/<sha>/.mcp.json`.
- **Wrong content cached** → `claude plugin uninstall xp-mcp@xp-claude-plugins` then `claude plugin install xp-mcp@xp-claude-plugins`. Local-dir marketplaces don't auto-bump version on file edits.

## See also

- XP for agents: https://xp.tickets/agents
- The XP MCP server itself: https://mcp.xp.tickets/mcp
- XP marketplace: https://xp.tickets
- [Claude Code plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
