# Publishing the XP MCP server to registries

Everything here is prepared, nothing is submitted. Three targets:

| Target | What ships | File |
|---|---|---|
| Official MCP Registry (registry.modelcontextprotocol.io) | `server.json`, remote-only | [server.json](server.json) |
| Smithery | the server URL; optional static server card | this file, "Smithery" |
| Anthropic directory (plugin bundle + connector) | the plugin at `plugins/xp-mcp` | this file, "Anthropic directory" |

## Official MCP Registry

Namespace is `tickets.xp/*` (reverse-DNS of `xp.tickets`). Proving it needs either a DNS TXT
record on the apex `xp.tickets` or a file at `https://xp.tickets/.well-known/mcp-registry-auth`.
DNS is written out below; HTTP is one line at the end.

`server.json` is remote-only: no `packages`, one `streamable-http` remote at
`https://mcp.xp.tickets/mcp`. The registry checks you own the namespace, not the remote URL, and
rejects a second server that reuses the same remote URL. `description` is capped at 100
characters (ours is 95). Bump `version` in `server.json` whenever the server version changes.

### 1. Install mcp-publisher

macOS (Homebrew):

```bash
brew install mcp-publisher
```

macOS / Linux (release tarball):

```bash
curl -L "https://github.com/modelcontextprotocol/registry/releases/latest/download/mcp-publisher_$(uname -s | tr '[:upper:]' '[:lower:]')_$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/').tar.gz" | tar xz mcp-publisher && sudo mv mcp-publisher /usr/local/bin/
```

Windows (PowerShell), then move `mcp-publisher.exe` somewhere on `PATH`:

```powershell
$arch = if ([System.Runtime.InteropServices.RuntimeInformation]::ProcessArchitecture -eq "Arm64") { "arm64" } else { "amd64" }; Invoke-WebRequest -Uri "https://github.com/modelcontextprotocol/registry/releases/latest/download/mcp-publisher_windows_$arch.tar.gz" -OutFile "mcp-publisher.tar.gz"; tar xf mcp-publisher.tar.gz mcp-publisher.exe; rm mcp-publisher.tar.gz
```

### 2. Generate the signing key and the TXT record

Needs OpenSSL 3.0+ (macOS ships LibreSSL, which lacks Ed25519: `brew install openssl@3` and call
`/opt/homebrew/opt/openssl@3/bin/openssl`). Keep `key.pem` out of git; it is the publishing
credential for `tickets.xp/*`.

```bash
MY_DOMAIN="xp.tickets"
openssl genpkey -algorithm Ed25519 -out key.pem
PUBLIC_KEY="$(openssl pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)"
echo "${MY_DOMAIN}. IN TXT \"v=MCPv1; k=ed25519; p=${PUBLIC_KEY}\""
```

Add that TXT record on the **apex** `xp.tickets` (host `@`), value `v=MCPv1; k=ed25519; p=<key>`.
Not under a selector such as `_mcp-auth.xp.tickets`; the registry only reads the apex and a
selector record fails with a generic signature error. Confirm propagation:

```bash
dig +short TXT xp.tickets
```

### 3. Log in and publish

From this folder (`docs/registry`):

```bash
PRIVATE_KEY="$(openssl pkey -in key.pem -noout -text | grep -A3 "priv:" | tail -n +2 | tr -d ' :\n')"
mcp-publisher login dns --domain "xp.tickets" --private-key "${PRIVATE_KEY}"
```

```bash
mcp-publisher validate
```

```bash
mcp-publisher publish
```

Expected tail of the output: `✓ Successfully published`. Verify:

```bash
curl "https://registry.modelcontextprotocol.io/v0.1/servers?search=tickets.xp/xp-tickets"
```

`You do not have permission to publish this server` means the login method does not match the
`tickets.xp/` namespace in `server.json`; re-run the `login dns` step.

HTTP auth alternative (same key): serve the literal line `v=MCPv1; k=ed25519; p=<PUBLIC_KEY>` as
`https://xp.tickets/.well-known/mcp-registry-auth`, then
`mcp-publisher login http --domain "xp.tickets" --private-key "${PRIVATE_KEY}"`. Note the
SPA currently answers every unknown path with HTML (see Smithery below), so that file needs a
real route or a static-host rule before this works.

Key rotation: remove the old TXT record first. A stale record is tried first and breaks
verification.

## Smithery

Smithery proxies to our hosted server; nothing is deployed to them. Requirements: streamable
HTTP transport and OAuth. Both hold. Smithery registers its OAuth client via Client ID Metadata
Documents, which `https://mcp.xp.tickets/.well-known/oauth-protected-resource` already
advertises (`client_id_metadata_document_supported: true`).

Install and sign in (Node 20+):

```bash
npm install -g smithery@latest
```

```bash
smithery auth login
```

Publish the URL (namespace must be one the XP Smithery account owns; `smithery namespace list`
shows them):

```bash
smithery mcp publish "https://mcp.xp.tickets/mcp" -n @xp-tickets/xp
```

The web flow at https://smithery.ai/new does the same thing with a form.

### Server card

After publish, Smithery scans the server for tools. Scanning hits the OAuth wall for the 28
protected tools, so Smithery will prompt for a sign-in to finish the scan. If that stalls, the
fallback is a static card at `/.well-known/mcp/server-card.json` **on the MCP host**, i.e.
`https://mcp.xp.tickets/.well-known/mcp/server-card.json`.

Current state (checked 2026-10-19):

| URL | Status |
|---|---|
| `https://mcp.xp.tickets/.well-known/mcp/server-card.json` | 404 text/plain |
| `https://xp.tickets/.well-known/mcp/server-card.json` | 200 text/html (SPA fallback, not JSON) |

The apex one is wrong on two counts (wrong host, HTML body) and Smithery never reads it. The fix
lives in the MCP server, not the website: add a route on `mcp.xp.tickets` returning
`application/json`. Minimal body; `tools`, `resources`, `prompts` are optional and the scan fills
them in once it can authenticate:

```json
{
  "serverInfo": { "name": "XP", "version": "4.0.9" },
  "authentication": { "required": true, "schemes": ["oauth2"] },
  "tools": [],
  "resources": [],
  "prompts": []
}
```

If you want the 12 public tools listed without a scan, paste their `tools/list` entries into
`tools` (name, description, inputSchema). Smithery's scanner identifies as
`SmitheryBot/1.0 (+https://smithery.ai)` from Cloudflare Workers; unauthenticated requests must
get a 401 (not 403) so RFC 9728 discovery works. The server already does this.

## Anthropic directory

Portal: https://claude.ai/directory/manage, "Submit new". Submit twice: once as **Plugin bundle**
pointing at `plugins/xp-mcp` in this repo, once as a **Connector** pointing at
`https://mcp.xp.tickets/mcp`. Requirements: https://claude.com/docs/plugins/pre-submission-checklist
and https://claude.com/docs/connectors/building/review-criteria.
