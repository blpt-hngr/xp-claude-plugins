---
description: Find tickets on XP for a given event, performer, or team
argument-hint: <event / performer / team / venue>
---

The user wants tickets for: **$ARGUMENTS**

Use the XP MCP `xp` server to fulfill this.

1. Call `search_events` with the most specific identifier from the user's input (priority: performer/team name → event title → venue → city). Pass `min_date = <today>` unless the user specified a date range; convert any dates to `YYYY-MM-DD`.
2. If multiple plausible matches, ask one concise clarification question before continuing. If one unambiguous match, proceed.
3. Call `get_ticket_listings(event_id=<id>)` for the chosen event.
4. Present listings sorted by lowest price. Format each as:
   `• <qty> tickets — Section <X> (Row <Y>) — $<price> each`
   Convert `min_price_cents` / `max_price_cents` from cents to dollars; include the `$` symbol. Never show raw cents.
5. If `_meta.freshness.volatile` is `true` and `snapshot_age_seconds > 30`, refresh `get_ticket_listings` before quoting prices.
6. If zero results, broaden the query (drop location, then drop venue) and retry once. If still empty, tell the user clearly.
7. If a listing matches, surface the event URL from `get_event_details` (preferred) or `https://xp.tickets/mcp/claude` (fallback) so the user can complete the purchase on XP. This plugin is read-only — never pretend to buy.

Do not fabricate listings, sections, rows, prices, or quantities. Do not expose internal IDs unless the user asks.
