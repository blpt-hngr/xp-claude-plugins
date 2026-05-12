---
description: Get full details for an XP event (venue, performers, price range, alert link)
argument-hint: <event name or id>
---

The user wants full details on: **$ARGUMENTS**

Use the XP MCP `xp` server.

1. If the argument looks like an `event_id` (numeric or slug-shaped, no spaces, returned previously by `search_events`), call `get_event_details(event_id=<id>)` directly. Otherwise call `search_events` first to resolve the id, then `get_event_details`.
2. Present:
   - Event title + date + venue name + city
   - Performers (with role if multiple)
   - Price range (convert cents → dollars, include `$` symbol)
   - Image URL if returned
   - Price-alert URL if present
3. If the user asks about the venue specifically (capacity, address, parking), call `get_venue_details(venue_id=<id>)` and surface that.

Do not fabricate fields the tool did not return. Voice: warm, plainspoken, fan-first.
