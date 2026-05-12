---
description: Set up an XP price alert (browser opt-in) for an event with an optional max-price ceiling
argument-hint: <event> [under $<price>]
---

The user wants a price alert for: **$ARGUMENTS**

Use the XP MCP `xp` server.

1. Parse the argument for an event reference and an optional max-price ceiling (e.g. "under $200", "below 150", "$300 ceiling"). If a ceiling is given, convert dollars to integer cents for `max_price_cents` (multiply by 100).
2. Resolve the event with `search_events` (and `get_event_details` if disambiguation is needed) to get a concrete `event_id`.
3. Optionally call `list_price_alert_sections(event_id=<id>)` if the user mentioned a specific section, and pass matching `section_ids` to the link generator.
4. Call `get_price_alert_link(event_id=<id>, max_price_cents=<cents or omit>, section_ids=<list or omit>)`.
5. Return the URL to the user with a one-line explainer: "Open this link in your browser to confirm the alert — XP will email/push when matching tickets drop."
