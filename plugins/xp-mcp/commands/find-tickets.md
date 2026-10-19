---
description: Find tickets on XP for a given event, performer, or team
argument-hint: <event / performer / team / venue>
---

The user wants tickets for: **$ARGUMENTS**

Use the XP MCP `xp` server. Follow the `xp-tickets` skill.

1. Call `search_events` with the most specific identifier from the user's input (priority: performer/team name → event title → venue → city). Pass `min_date = <today>` unless the user specified a date range; convert any dates to `YYYY-MM-DD`.
2. If multiple plausible matches, ask one concise clarification question before continuing. If one unambiguous match, proceed.
3. Call `get_ticket_listings(event_id=<id>)` for the chosen event. Pass `quantity` and `max_price_usd` (dollars) if the user gave them.
4. Present listings sorted by lowest all-in price. Format each as:
   `• <qty> tickets — Section <X> (Row <Y>) — $<price> each, all-in`
5. If `_meta.freshness.volatile` is `true` and `snapshot_age_seconds > 30`, refresh `get_ticket_listings` before quoting prices.
6. If zero results or `NO_INVENTORY`, broaden the query (drop location, then drop venue) and retry once. If still empty, say so and suggest nearby dates or setting their price (`/xp-mcp:set-your-price`).
7. If the user wants to buy one: `buy_tickets(event_id, uvid, quantity, confirm=false)`, read back event, date, section, quantity, and all-in total, and only after an explicit yes give them the `payment_link`, then `confirm=true` with `checkout_session_id` once they've paid. To make an offer on a fan listing instead, use `make_offer_on_listing` the same two-phase way with whole-dollar `amount_per_ticket_usd`. Never skip the preview. Never pretend to buy.

Do not fabricate listings, sections, rows, prices, or quantities. Do not expose internal IDs unless the user asks. Sign-in is only needed at step 7; call the tool directly and let the client start OAuth.
