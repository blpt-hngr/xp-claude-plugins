---
description: Set your price on XP — rest a standing offer that fills when matching tickets appear
argument-hint: <event> at $<price per ticket> [for <qty>] [section <X>]
---

The user wants to set their price for: **$ARGUMENTS**

Use the XP MCP `xp` server. This creates a **standing offer** with `create_standing_bid`. Say "standing offer" to the user; the tool keeps `bid` only because the stored records do.

**It is a binding offer, not an alert.** When a matching fan listing appears at or under their price, XP buys it for them. Tell them that before anything else.

1. Parse the argument for the event(s), a whole-dollar price per ticket, quantity, and any section names. Price is `amount_per_ticket_usd` in whole dollars (`120` = $120.00); XP rejects fractional amounts.
2. Resolve each event with `search_events` (and `get_event_details` if disambiguation is needed) to concrete `event_ids`. The same performer often plays several dates; confirm which ones.
3. Check what's already open with `get_market_read(event_id)` (one event) or `list_open_listings` (several). If a matching listing exists at or near their price, say so: an offer on it (`make_offer_on_listing`) gets an answer from the seller, while a standing offer waits for supply that may never arrive. Both are valid; let them choose, and say which you did.
4. Check `list_my_standing_bids` so you don't stack a duplicate.
5. **Preview.** `create_standing_bid(event_ids, sections, amount_per_ticket_usd, quantity, expires_at?, confirm=false)`. Read back: events and dates, sections, quantity, price per ticket, total commitment, expiry, and that it fills on its own and they'll be charged when it matches.
6. **Confirm.** Only after an explicit yes in this conversation, call again with `confirm=true`. Never skip the preview. Never confirm on an earlier yes.
7. Say what happens next: it rests until it fills or they cancel it (`rescind_standing_bid`, also preview-then-confirm). Fills already taken are completed purchases. Setting a price does not promise tickets at that price.

Sign-in is required; call the tool directly and let the client start OAuth. Do not call `auth_status` first. Voice: warm, plainspoken, fan-first. No auction language.
