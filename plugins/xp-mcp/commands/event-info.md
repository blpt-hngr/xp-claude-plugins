---
description: Get full details for an XP event (venue, performers, price range)
argument-hint: <event name or id>
---

The user wants full details on: **$ARGUMENTS**

Use the XP MCP `xp` server.

1. If the argument looks like an `event_id` (numeric or slug-shaped, no spaces, returned previously by `search_events`), call `get_event_details(event_id=<id>)` directly. Otherwise call `search_events` first to resolve the id, then `get_event_details`.
2. Present:
   - Event title + date + venue name + city
   - Performers (with role if multiple)
   - All-in price range in dollars (include the `$` symbol)
   - Image URL if returned
3. If the user asks about the venue specifically (capacity, address, parking), call `get_venue_details(venue_id=<id>)` and surface that. For "where is section X", call `get_seat_map(event_id=<id>)`; it is the layout, not a seat view.

Do not fabricate fields the tool did not return. Voice: warm, plainspoken, fan-first.
