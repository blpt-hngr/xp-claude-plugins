# Tool annotations audit — mcp.xp.tickets

Source: `tools/list` on `https://mcp.xp.tickets/mcp`, server `XP` v4.0.9, fetched 2026-10-19.
40 tools returned. Re-run with:

```bash
curl -s -X POST https://mcp.xp.tickets/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```

## Verdict

**No tool fails the Anthropic connector annotation rules.** The review criteria
(https://claude.com/docs/connectors/building/review-criteria) require every tool to carry a
`title` plus `readOnlyHint: true` if read-only or `destructiveHint: true` if it modifies or
deletes data, and tool names of 64 characters or fewer.

| Check | Result |
|---|---|
| `title` present | 40 / 40 |
| `readOnlyHint` present | 40 / 40 |
| `destructiveHint` present | 40 / 40 |
| `openWorldHint` present | 40 / 40 |
| `idempotentHint` present | 0 / 40 (not required by Anthropic; see notes) |
| `outputSchema` present | 40 / 40 |
| Longest tool name | 25 chars (`get_my_referral_kickbacks`), limit 64 |
| Read/write split | Clean. No catch-all `api_request`-style tool, no tool mixing safe and unsafe methods |

The things the review will trip on are **policy**, not annotations. See "Review risks" below.

## Per-tool table

Columns: name length, title, `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`, description length (chars).

| Tool | len | title | readOnly | destructive | idempotent | openWorld | desc |
|---|---|---|---|---|---|---|---|
| `search_events` | 13 | Search Events | true | false | missing | true | 361 |
| `get_event_details` | 17 | Get Event Details | true | false | missing | true | 387 |
| `search_performers` | 17 | Search Performers | true | false | missing | true | 457 |
| `get_ticket_listings` | 19 | Get Ticket Listings | true | false | missing | true | 519 |
| `buy_tickets` | 11 | Buy Tickets | false | true | missing | false | 1175 |
| `search_venues` | 13 | Search Venues | true | false | missing | true | 327 |
| `get_venue_details` | 17 | Get Venue Details | true | false | missing | true | 518 |
| `check_listing` | 13 | Check Listing | true | false | missing | true | 3938 |
| `search_market` | 13 | Search the Market | true | false | missing | true | 509 |
| `get_market_read` | 15 | Read the Market | true | false | missing | true | 463 |
| `get_recent_sales` | 16 | Recent Sales | true | false | missing | true | 425 |
| `get_price_history` | 17 | Price History | true | false | missing | true | 592 |
| `get_my_context` | 14 | What XP Knows | true | false | missing | false | 529 |
| `get_seat_map` | 12 | Seat Map | true | false | missing | true | 583 |
| `get_user_account` | 16 | Get User Account | true | false | missing | false | 448 |
| `auth_status` | 11 | Auth Status | true | false | missing | true | 686 |
| `get_my_orders` | 13 | Get My Orders | true | false | missing | false | 622 |
| `get_order_status` | 16 | Get Order Status | true | false | missing | false | 530 |
| `get_my_tickets` | 14 | Get My Tickets | true | false | missing | false | 525 |
| `list_my_favorites` | 17 | List My Favorites | true | false | missing | false | 557 |
| `get_my_wallet` | 13 | Get My Wallet | true | false | missing | false | 395 |
| `get_my_referral_kickbacks` | 25 | Get My Referral Kickbacks | true | false | missing | false | 372 |
| `list_open_listings` | 18 | List Open Listings | true | false | missing | false | 458 |
| `get_open_listing` | 16 | Get Open Listing | true | false | missing | false | 446 |
| `get_my_listing_status` | 21 | Get My Listing Status | true | false | missing | false | 980 |
| `list_my_listings` | 16 | List My Listings | true | false | missing | false | 767 |
| `get_my_bids` | 11 | Get My Bids | true | false | missing | false | 617 |
| `make_offer_on_listing` | 21 | Make Offer on Listing | false | true | missing | false | 717 |
| `accept_offer` | 12 | Accept Offer | false | true | missing | false | 1166 |
| `reject_offer` | 12 | Reject Offer | false | true | missing | false | 699 |
| `cancel_my_offer` | 15 | Cancel My Offer | false | true | missing | false | 819 |
| `list_muted_listings` | 19 | List Muted Listings | true | false | missing | false | 463 |
| `mute_listing` | 12 | Mute Listing | false | false | missing | false | 503 |
| `list_my_standing_bids` | 21 | My Standing Offers | true | false | missing | false | 448 |
| `get_standing_bid` | 16 | Standing Offer Detail | true | false | missing | false | 332 |
| `create_standing_bid` | 19 | Rest a Standing Offer | false | true | missing | false | 1248 |
| `rescind_standing_bid` | 20 | Rescind Standing Offer | false | true | missing | false | 362 |
| `buy_listing_now` | 15 | Buy Listing at the Ask | false | true | missing | false | 714 |
| `create_listing` | 14 | List Tickets for Sale | false | false | missing | false | 876 |
| `cancel_my_listing` | 17 | Take Listing Down | false | true | missing | false | 696 |

Totals: 27 read-only, 13 write. Of the 13 writes, 11 carry `destructiveHint: true`;
`mute_listing` and `create_listing` are `false` on both hints (additive, reversible), which is
correct per the MCP spec and does not break the Anthropic rule (it only demands `destructiveHint:
true` for tools that modify or delete data; a create that doesn't overwrite is fine either way).
If a reviewer asks, flipping `create_listing` to `destructiveHint: true` is the conservative
move: it puts tickets on a public market.

## Review risks (not annotation gaps)

These are the items a human reviewer will raise. Ordered by likelihood of rejection.

1. **Financial-transaction policy.** The connector criteria say: "Connectors that do the following
   aren't accepted: Transfer money, cryptocurrency, or other financial assets." The submission form
   makes you acknowledge this explicitly. Exposure, by tool:
   - `buy_listing_now`: description says it buys "from your XP USDC balance". That is a stablecoin
     transfer from the user's wallet. **Highest risk.**
   - `get_my_wallet`: description says it returns the user's "Privy embedded-wallet address and
     USDC balance". Read-only, but it names a crypto wallet, which tells the reviewer the server is
     wallet-aware.
   - `make_offer_on_listing`, `create_standing_bid`, `accept_offer`: settle in USDC when they fill.
     Same category as `buy_listing_now`, one step removed.
   - `buy_tickets`: default rail is a Stripe Checkout link. The user pays on Stripe's page, not
     through the tool. This is a purchase of goods, which is normal e-commerce, and is the safest
     framing. Keep "card checkout link" wording; drop any mention of alternate rails that settle
     from a balance.
   Mitigations to decide before submitting: (a) hide `get_my_wallet` and `buy_listing_now` from the
   listed connector surface (server-side tag or a separate connector endpoint), (b) reword the
   USDC/wallet language to "XP account balance" so it reads as a stored-value balance rather than a
   crypto asset, or (c) submit as-is and argue it is marketplace checkout. Option (c) is the one the
   policy text does not support.
2. **`check_listing` needs an `X-XP-Correlation-Id` request header.** Its description (3,938
   chars, the longest by far) explains that identity comes from a header, not a parameter. Claude's
   connector client does not set custom headers per call, so this tool cannot return a success with
   valid params in the reviewer's run. The criteria require that "every tool returns a success with
   valid params". Either hide it from the connector listing or give it an explicit `correlation_id`
   parameter.
3. **Deprecated parameters still on schemas.** `get_ticket_listings.min_price_cents` /
   `max_price_cents`, `list_open_listings.min_amount` / `max_amount`, and
   `make_offer_on_listing.amount_cents` are still in `inputSchema` and marked deprecated in their
   descriptions. Not a rejection, but reviewers test with what the schema offers. Removing them
   shrinks the surface and avoids a "sent cents, got dollars" bug report.
4. **`auth_status` is `openWorldHint: true`.** Harmless, but it is the only account tool flagged
   open-world. Probably a copy-paste default; set it `false` for consistency.
5. **`idempotentHint` absent everywhere.** Not required by Anthropic or the official registry.
   Worth setting `true` on the cancel/rescind/reject tools and the two-phase `confirm=false`
   previews if the server ever wants clients to retry safely.
6. **Descriptions over 1,000 chars:** `buy_tickets`, `check_listing`, `accept_offer`,
   `create_standing_bid`. No stated limit, and the content is behavioral guidance rather than
   prompt injection, so it passes the injection scan. Just noting them as the places a reviewer
   reads most closely.

## Prompt-injection scan (manual)

Read all 40 descriptions against the rejected patterns (instructing Claude to call unrelated
external tools, pulling instructions from external sources, hidden or encoded text, overriding
system instructions, promoting products). None present. The descriptions do push the two-phase
`confirm=false` / `confirm=true` pattern and tell Claude to call the requested tool rather than
`auth_status` first. Both are in-scope behavioral guidance for this server's own tools.

`_meta` carries `openai/toolInvocation/invoking` / `invoked` status strings and an empty
`fastmcp.tags` list. Inert for Anthropic's review.
