# Anthropic directory checklist — xp-mcp

Portal: https://claude.ai/directory/manage → Submit new. Two separate submissions:

1. **Plugin bundle** — source repo `blpt-hngr/xp-claude-plugins`, plugin path `plugins/xp-mcp`.
2. **Connector** — remote server `https://mcp.xp.tickets/mcp`.

Inspection date 2026-10-19, branch `xp_claude/suspicious-tu-8f505e`. Rules from
https://claude.com/docs/plugins/pre-submission-checklist,
https://claude.com/docs/connectors/building/review-criteria and
https://claude.com/docs/connectors/building/submission.

## Part 1 — Plugin bundle (checked against the repo)

Levels are Anthropic's: **Blocks** stops submission, **Held** means a human reads it, **Warning** is advisory.

| # | Item | Level | Result | Evidence |
|---|---|---|---|---|
| 1 | Folder contains `.claude-plugin/plugin.json` | Blocks | PASS | `plugins/xp-mcp/.claude-plugin/plugin.json` |
| 2 | One plugin per submission; in a marketplace repo submit the plugin folder, not the root | Blocks | PASS | marketplace has one plugin; submit path `plugins/xp-mcp` |
| 3 | All component paths inside the plugin folder | Blocks | PASS | plugin.json lists no component paths; defaults (`skills/`, `commands/`, `agents/`, `.mcp.json`) are all inside |
| 4 | Regular files only: no symlinks, submodules, LFS pointers | Blocks | PASS | `git ls-files -s`: all 10 files mode 100644 |
| 5 | No `.DS_Store`, `Thumbs.db`, `desktop.ini`, `__MACOSX` | Blocks | PASS | none tracked; `.gitignore` covers the first two |
| 6 | Windows/macOS-safe names: no colon, trailing dot/space, device names, case-only duplicates | Stops | PASS | names: `SKILL.md`, `*.md`, `.mcp.json`, `plugin.json` |
| 7 | Path folders use letters, digits, dots, hyphens, underscores | Stops | PASS | `plugins/xp-mcp` |
| 8 | No `.gitattributes` with `export-ignore`, `export-subst`, or `filter` | Stops | PASS | no `.gitattributes` in repo |
| 9 | Size: repo < 50 MiB archived, < 256 MiB unpacked, < 10,000 entries, every file < 5 MiB | Stops | PASS | plugin folder is 37 KB across 10 files |
| 10 | `name` is lowercase letters, digits, hyphens, ≤ 64 chars, starts and ends alphanumeric | Blocks / Warn | PASS | `xp-mcp` |
| 11 | `name` is not a reserved word (`claude`, `anthropic`, `official`, `plugin`, `mcp`, `test`) **as the whole name**, and not only generic words | Blocks / Held | PASS with risk | `xp-mcp` is not wholly reserved. But `mcp` is on the list and `xp` is the only distinctive token, so the "name made only of generic words" hold could fire. See Risks. |
| 12 | `name` / `displayName` / `author.name` not confusable with an existing plugin, publisher, connector, or brand | Blocks / Held | PASS, unverified | "XP" is short and generic; portal search is the only way to know. `displayName` is not set (optional). |
| 13 | Not a fork using the upstream name | Held | PASS | original work |
| 14 | `displayName`, `author.name` single writing system, no look-alike or invisible characters | Blocks | PASS | `author.name: "XP"` |
| 15 | Component keys spelled per manifest reference, none under `experimental` | Blocks | PASS | no component keys set |
| 16 | `description`, `author`, `version` set | Warning | PASS | `version: "1.0.0"` added this commit; `claude plugin validate .` passes clean |
| 17 | README ≥ 40 words in plugin folder, words in code blocks don't count | Blocks | PASS | `plugins/xp-mcp/README.md`: 1,051 words outside code blocks |
| 18 | `LICENSE` file in the plugin folder **or** `license` in plugin.json | Blocks | PASS | `"license": "MIT"` in plugin.json; repo-root `LICENSE` is MIT too (root file alone would not count) |
| 19 | Every non-image/font file < 256 KiB | Held | PASS | largest is `skills/xp-tickets/SKILL.md` at 10 KB |
| 20 | ≤ 512 files | Held | PASS | 10 |
| 21 | Only text, PNG/JPEG/GIF/WebP, fonts. No `.ico`, `.pdf`, `.zip`, executables | Held | PASS | all text |
| 22 | Images only referenced from README via Markdown image syntax | Held | PASS | no images bundled |
| 23 | MCP servers declared with `command`+`args` or `url`, not `.mcpb`/`.dxt` | Held / Blocks | PASS | `.mcp.json` uses `url` |
| 24 | Remote MCP `type` is `http`, `sse`, or `ws`; `url` absolute `https://` | Blocks | PASS | `type: "http"`, `url: "https://mcp.xp.tickets/mcp"` |
| 25 | `.mcp.json` valid JSON matching MCP schema | Blocks | PASS | validated by `claude plugin validate .` |
| 26 | No real credentials; secrets via `userConfig` `sensitive: true` | Blocks | PASS | no headers, no tokens; auth is OAuth at runtime |
| 27 | No reading/sending of user-environment credentials | Held / Blocks | PASS | no hooks, no scripts |
| 28 | Pinned `npx`/`uvx` versions, no registry overrides, no local-server shell launch | Blocks / Held | N/A | no local servers, no launchers |
| 29 | `hooks/hooks.json` valid; not listed in plugin.json `hooks` field | Blocks / Warn | N/A | no hooks |
| 30 | Skills/commands/agents have YAML front matter with string `description` | Blocks / Warn | PASS | all 7 files have `description:` (3 skills, 3 commands, 1 agent) |
| 31 | Exact folder/file names (`skills/`, `SKILL.md`, `commands/`, `agents/`) | Blocks | PASS | |
| 32 | README describes everything the plugin runs, sends, or fetches (post-submit security scan) | Scan | PASS | README names the single remote server, OAuth-on-demand, and the two-phase confirm |
| 33 | Readable source, no minified/packed code | Scan | PASS | Markdown and JSON only |

**Plugin bundle: 31 PASS, 2 N/A, 0 FAIL.** One hold risk (item 11).

## Part 2 — Connector (checked against the live server)

| # | Item | Result | Evidence |
|---|---|---|---|
| C1 | Reachable over `https://` | PASS | `https://mcp.xp.tickets/mcp`, 200 on `initialize` |
| C2 | OAuth 2.0 for tools that act on the user's account | PASS | `/.well-known/oauth-protected-resource` and `/.well-known/oauth-authorization-server` present; authorization code + PKCE S256, refresh tokens, DCR at `/oauth/register`, Client ID Metadata Documents supported |
| C3 | Auth prompted on demand per tool (portal lets you flag this) | PASS | 12 tools public, 28 protected; protected tools 401 until signed in |
| C4 | Every tool has `title` + `readOnlyHint`/`destructiveHint` | PASS | 40/40, see [tool-annotations-audit.md](tool-annotations-audit.md) |
| C5 | Tool names ≤ 64 chars | PASS | longest 25 |
| C6 | Read and write split into separate tools, no catch-all `api_request` | PASS | 27 read, 13 write, each fixed-purpose |
| C7 | Custom-query tools name their target API | N/A | none |
| C8 | No prompt-injection patterns in descriptions | PASS | manual read of all 40, see audit |
| C9 | Every tool returns success with valid params | FAIL (1) | `check_listing` needs an `X-XP-Correlation-Id` header the Claude client cannot send. Hide it from the connector or add a parameter. |
| C10 | Does not query Claude memory, chat history, or user files | PASS | |
| C11 | First-party API; server domain matches the service | PASS | `mcp.xp.tickets` under `xp.tickets` |
| C12 | Not in the unsupported list: "Transfer money, cryptocurrency, or other financial assets" | **AT RISK** | see below |
| C13 | Not AI image/video/audio generation | PASS | |
| C14 | Tested as a custom connector and via MCP Inspector, every tool | TODO | Albert confirms in the portal's Test & launch step |

### Listing assets Albert must supply

The portal's Listing step asks for these. None exist in the repo today.

| Asset | Constraint | Status / suggestion |
|---|---|---|
| Server name | ≤ 100 chars | "XP Tickets" |
| One-liner | ≤ 200 chars | reuse `server.json` description (90 chars) |
| Description | ≤ 2,000 chars | lift from `plugins/xp-mcp/README.md` intro + "Ask before you act" |
| Categories | 1 to 5 | Shopping / Entertainment / Marketplaces, whatever the portal offers |
| **Documentation URL** | public, by publish date; help-center article is enough | `https://xp.tickets/agents` (live, 200) or the plugin README on GitHub |
| **Privacy policy URL** | HTTPS; must cover collection, usage/storage, third-party sharing, retention, contact. "Missing or incomplete privacy policies result in immediate rejection." | `https://xp.tickets/privacy-policy` (returns 200, confirmed 2026-10-19). Confirm the page actually covers retention and third-party sharing, including Stripe and Privy. |
| **Support contact** | email or URL | `https://xpsupport.zendesk.com/hc/en-us/requests/new` (XP support request form; returns 200 to browsers, 403 to bare curl) |
| **Icon** | portal upload; no size stated in docs, assume square PNG ≥ 512 px | needs an XP mark; none in repo |
| URL slug | permanent once published | `xp-tickets` |
| Company name + website, review contact | | XP, `https://xp.tickets`, Albert |
| Use cases | primary uses, what users need first, read/write/both | "find tickets, compare all-in prices, make an offer, set your price, buy, sell"; needs an XP account only for buy/offer/sell; both |
| Data handling | own API vs proxied; health data; sponsored content | own API; no; no |
| Allowed link URIs (optional) | HTTPS origins the org owns | `https://xp.tickets`, `https://checkout.stripe.com` if `buy_tickets` returns a Stripe link Claude should open |
| **Reviewer test account** | required, "fully populated": orders, tickets held, a live listing, an open offer, a standing offer, balance | needs a seeded XP account plus written sign-in steps. Must be able to complete the OAuth flow from Claude. |
| Seven policy acknowledgments | directory guidelines, first-party API, financial transactions, AI media, prompt injection, conversation data, public docs | all required; the financial one is the honest blocker (below) |

### Risks

**R1. Money-transfer policy (C12).** The criteria say connectors that "transfer money, cryptocurrency,
or other financial assets" aren't accepted, and the Compliance step makes you acknowledge it.
Two tools say it out loud:

- `buy_listing_now` — "from your XP USDC balance". A stablecoin debit from the user's wallet.
- `get_my_wallet` — "Privy embedded-wallet address and USDC balance". Read-only, but it names a
  crypto wallet.

`make_offer_on_listing`, `create_standing_bid` and `accept_offer` settle the same way when they
fill. `buy_tickets` on its default Stripe Checkout rail is a normal goods purchase on Stripe's
page, which is the clean story.

Options, pick one before submitting:

1. Exclude `get_my_wallet` and `buy_listing_now` from the connector (server-side tag, or a second
   endpoint such as `/mcp/connector` with the trimmed list) and reword the three offer tools to
   "charged to your XP account balance". Lowest rejection risk. The plugin bundle can keep the
   full 40 since it is just `.mcp.json` pointing at the server.
2. Submit all 40 and answer the financial-transactions acknowledgment truthfully. Expect a reject
   or a Verified-review escalation to `mcp-review@anthropic.com`.
3. Keep the wallet tools but relabel USDC as "account balance" everywhere. Reads as stored
   value, not crypto, but it is the same wire transfer and a reviewer who signs in will see Privy.

**R2. Plugin name hold (item 11).** `xp-mcp` contains the reserved word `mcp`. Not a block, but the
"only generic words" hold is plausible. Cheapest fix if held: rename to `xp-tickets` (plugin
folder, `plugin.json.name`, `marketplace.json.plugins[].name`, slash commands become
`/xp-tickets:find-tickets`). Decide now or wait for the portal verdict; renaming after users
install is worse.

**R3. `check_listing` (C9).** Will not succeed from Claude. Hide or parameterise before the
reviewer runs every tool.

**R4. Privacy policy coverage.** `https://xp.tickets/privacy-policy` exists; whether it names Stripe,
Privy, retention periods and a contact is not checked here. Immediate-rejection item, so read it.

**R5. Deprecated `_cents` params** still on three schemas. Not a rejection; a reviewer may send
them. Drop them server-side when convenient.

## Submission order

1. Resolve R1 (decide the tool surface) and R3 on the server; redeploy.
2. Confirm R4 and gather the asset table above.
3. Submit the **plugin bundle** first: it has no asset dependencies and passes today.
4. Submit the **connector** once assets and the test account exist.
