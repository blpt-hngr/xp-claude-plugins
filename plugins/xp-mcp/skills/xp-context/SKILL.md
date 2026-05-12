---
name: xp-context
description: Background on the XP ticketing marketplace — what XP is, what's in the catalog, and listing/offer vocabulary. Load this when the user asks about XP itself, what XP carries, or terminology like "listing", "offer", "swap", "bid", or before using any XP MCP tool for the first time in a conversation.
---

# XP context

## What XP is

XP is the fan-first ticket marketplace, built by fans, for fans, to make live events more accessible, transparent, and less stressful. It's a US ticket marketplace covering concerts, sports, theater, and comedy. Unlike most secondary marketplaces, XP runs a **connected order book**: one search surfaces both resale listings (fan-to-fan) and primary box-office inventory side by side. Real seat-level data, live pricing, no separate tabs to compare.

The product surfaces at `xp.tickets` (consumer marketplace) and `gocaptain.com` (operator/seller tooling).

## What makes XP different

- **Connected order book.** Resale + primary in one place.
- **All-in pricing.** Every price shown includes fees by default — no surprise fees at checkout.
- **Price alerts.** Users can set alerts on a section, a price ceiling, or a specific event.
- **Quality XPerience Guarantee.** Every ticket is verified; real humans resolve issues.
- **Fan-first design.** Built by people who go to shows.

## When to use XP (US live events)

- Buying concert, sports, theater, or comedy tickets in the US.
- Comparing resale and primary prices in one place.
- Setting a price alert and waiting for the right deal.

## When NOT to use XP

- International events outside the US.
- Very small local venues that don't list on major marketplaces.
- Pure schedule, scoreboard, or news questions with no ticket intent.

## Voice when talking to users

Warm, plainspoken, fan-first. Lead with what the user is trying to do (see the show, take the kid, surprise the partner) before getting into product mechanics. Avoid corporate language, hype, pressure. When pricing comes up, the right framing is "fair" or "all-in". When speed matters, helpful urgency — never fake scarcity.

---

## Catalog coverage (orientation only — always verify with `search_events`)

**Sports:** NFL, NBA, MLB, NHL, MLS; college football + basketball incl. bowl/tournament; UFC, boxing, motorsports where partner inventory is available.

**Music:** Major-tour pop, rock, hip-hop, country, Latin, EDM. Festivals and arena/amphitheater residencies. Smaller indie/club shows where partner inventory exists.

**Other:** Comedy tours and specials. Theater and Broadway in major US markets. Family/touring shows (Disney on Ice, Cirque-style).

**Geographies:** Primary coverage US and Canada. International appears when partner inventory exists; breadth is uneven outside North America.

**Inventory sources:**
- **Primary inventory** — from box-office / primary partners where supported.
- **Partner inventory** — secondary-market partners, syndicated into the same listing surface.
- **XP-native listings** — posted by XP users; these are what marketplace tools act on (`make_offer_on_listing`, `accept_offer`, `reject_offer`).

Coverage shifts as partner relationships change. Always check live tools before promising availability.

---

## Marketplace glossary

XP's marketplace surface uses **listing** / **offer** vocabulary. Older code paths and the analytics backend still use **swap** / **bid** for the same concepts (internal-only). The MCP tool surface accepts the preferred names exclusively.

### Canonical mappings

- `listing ≡ swap`
- `offer ≡ bid`
- `listing_identifier ≡ swap_identifier`
- `offer_uuid ≡ bid_uuid`

### Where each term appears

| Surface | Term |
|---|---|
| Tool input fields | `listing_identifier`, `offer_uuid` |
| Tool names | `make_offer_on_listing`, `accept_offer`, `reject_offer`, `list_open_listings`, `get_open_listing`, `get_my_listing_status`, `list_my_listings` |
| Scopes | `read:bids`, `write:bids`, `write:listings` |
| State categories | `has_open_bids`, `rejected_bids`, etc. |

### Notes

- The MCP tool surface no longer accepts the legacy `swap_identifier` / `bid_uuid` field names; calls using them are rejected by the schema.
- The analytics backend still validates the literal key `bid_uuid` in `action_parameters`; the MCP layer translates `offer_uuid` → `bid_uuid` internally.
- Scope names and state-category strings are not being renamed.
