---
name: xp-ticket-verification
description: Verify a ticket the user is holding — given event title, section, row, and seat numbers, resolve the event on XP, confirm the section exists, and return a normalized JSON record. Load this when the user pastes ticket details (e.g. "I have section 104, row B, seats 4-6 for the Knicks game on Nov 14") and asks to verify, check, or look up the ticket on XP.
---

# Ticket verification

When the user provides event, section, row, and seat details, validate the ticket using MCP tools.

## 1. Resolve event

```
search_events(query=<event_title>, min_similarity=0.45)
```

Select the best match and obtain `event_id`. If no match found:

```json
{ "error": "Event not found" }
```

## 2. Verify section

```
get_ticket_listings(event_id=<event_id>)
```

Match the user's section against the sections available in the ticket listings (`ticket_groups` or equivalent in `structuredContent`).

Rules:
- Ignore case and common prefixes such as "section" or "sec".
- Use the **exact section name returned by the API** in the output.

If no match:

```json
{ "error": "Section not found" }
```

## 3. Determine quantity

Extract ticket quantity from the input.

- If quantity is missing, infer from seat count.
- If seat numbers are given as a range (e.g. `4-6`), expand into individual seats.
- If quantity cannot be determined, ask once before continuing.

## 4. Output

Return ONLY valid JSON. No additional text.

```json
{
  "event_id": "<event_id>",
  "event_title": "<verified_event_title>",
  "section": "<exact_section_name_from_api>",
  "row": "<row_from_input_or_null>",
  "seats": [<seat_numbers>],
  "quantity": <integer>
}
```

## Rules

- Verify event before section.
- Always use MCP tools for validation.
- Never fabricate event, section, row, or seat data.
- Output JSON only.
