---
title: MCP Interaction Model
description: Deep-dive into the Model Context Protocol interaction model covering components, OAuth 2.1 authorization, initialization, capability discovery, tool calling, sampling, and elicitation.
tags:
  - mcp
  - protocol
  - architecture
  - oauth
date: "2026-05-16T17:21:30.059582Z"
lastmod: "2026-05-16T22:52:19+05:30"
draft: false
---

## Components (official MCP nomenclature)

- **Host** — The user-facing application that embeds the LLM and enforces policy (Claude Desktop, Claude Code, an IDE plugin, etc.). It owns the user, the model, and the trust boundary.
- **Client** — A protocol connector that lives inside the Host. **One Client per Server**, holding a 1:1 stateful session. The Host spawns Clients as needed.
- **Server** — The process that exposes capabilities (tools, resources, prompts) over the MCP protocol. Can be local (stdio transport) or remote (Streamable HTTP transport).
- **Authorization Server (AS)** — For remote Servers: the OAuth 2.1 issuer of access tokens. May be the Server itself or a separate identity provider.
- **Resource Server (RS)** — OAuth role played by the remote MCP Server when it validates bearer tokens on incoming requests.
- **User** — The human who approves connections, consents to tool calls, and answers elicitations.
- **LLM** — Not technically an MCP component, but the reasoning engine the Host drives; never talks to a Server directly.

## Phase 1 — Transport & connection

1. **Host → Client**: Host launches a Client configured for a specific Server (command + args for stdio, or URL for HTTP).
2. **Client ↔ Server**: Transport established.
   - **stdio**: Host spawns the Server as a subprocess; JSON-RPC over stdin/stdout.
   - **Streamable HTTP**: Client opens an HTTPS connection; bidirectional via POST + SSE stream.

## Phase 2 — Authorization (remote Servers only)

MCP uses **OAuth 2.1 + PKCE**, with Resource Indicators (RFC 8707) and Dynamic Client Registration (RFC 7591).

1. **Client → Server**: Initial request without a token.
2. **Server (as RS) → Client**: `401 Unauthorized` with `WWW-Authenticate` pointing at `/.well-known/oauth-protected-resource`.
3. **Client → RS metadata endpoint**: Fetches Protected Resource Metadata, which names the Authorization Server(s).
4. **Client → AS metadata endpoint**: Fetches `/.well-known/oauth-authorization-server` (RFC 8414).
5. **Client → AS**: Dynamic Client Registration (if supported) to obtain a `client_id`.
6. **Client → Host → User**: Host opens browser to AS's `/authorize` with PKCE challenge + `resource` parameter (binds token to this Server).
7. **User ↔ AS**: User authenticates and consents.
8. **AS → Client**: Redirect with authorization code.
9. **Client → AS**: Exchanges code + PKCE verifier at `/token` for an access token (and optional refresh token).
10. **Client → Server**: Retries request with `Authorization: Bearer <token>`. Server validates audience, scopes, expiry.

## Phase 3 — Initialization handshake

1. **Client → Server**: `initialize` request — declares protocol version, Client capabilities (`roots`, `sampling`, `elicitation`), and Client info.
2. **Server → Client**: `initialize` response — agreed protocol version, Server capabilities (`tools`, `resources`, `prompts`, `logging`), Server info.
3. **Client → Server**: `notifications/initialized` — session is now live.

## Phase 4 — Capability discovery

- **Client → Server**: `tools/list`, `resources/list`, `prompts/list`, `resources/templates/list`.
- **Server → Client**: Returns JSON Schemas, URIs, descriptions.
- **Host**: Injects these into the LLM's context as available tools/resources, often filtered by user-granted permissions.

## Phase 5 — Operation (the steady state)

### Tool calling (model-initiated)
1. **LLM → Host**: Emits a tool-use request.
2. **Host → User**: (Policy-dependent) prompts for permission.
3. **Host → Client → Server**: `tools/call` with name + arguments.
4. **Server**: Executes; may consult its own backend / APIs.
5. **Server → Client → Host → LLM**: `CallToolResult` (content blocks: text, image, resource links, or `isError: true`).

### Resource reading (host/app-initiated)
- **Client → Server**: `resources/read` with a URI.
- **Server → Client**: Contents (text or blob).
- Optional **Client → Server** `resources/subscribe`; Server pushes `notifications/resources/updated`.

### Prompts (user-initiated)
- **User → Host**: Picks a prompt (e.g. via slash menu).
- **Client → Server**: `prompts/get` with arguments.
- **Server → Client**: Rendered message list, fed into the LLM.

### Sampling (Server-initiated LLM call)
- **Server → Client**: `sampling/createMessage` — Server asks the Host's LLM to complete something.
- **Host → User**: Confirms (human-in-the-loop required by spec).
- **Host → LLM → Host → Client → Server**: Completion returned.

### Elicitation (Server asks the User for input)
- **Server → Client**: `elicitation/create` with a JSON Schema describing requested fields.
- **Host → User**: Renders a form.
- **User → Host → Client → Server**: Structured response or decline.

### Roots (Client tells Server which filesystem scopes are in-bounds)
- **Client → Server**: `roots/list` on demand; `notifications/roots/list_changed` when they change.

### Notifications (out-of-band, either direction)
- `notifications/tools/list_changed`, `…/resources/list_changed`, `…/prompts/list_changed`
- `notifications/progress` for long-running calls (tied to a `progressToken`)
- `notifications/message` for log lines (`logging/setLevel` controls verbosity)
- `notifications/cancelled` to abort an in-flight request

## Phase 6 — Shutdown

- **Client → Server**: Closes transport (stdio: close stdin → Server exits; HTTP: terminate session, optionally `DELETE` the session ID).

## Trust boundaries

- The **Host** is the only component that talks to both the **User** and the **LLM**. The **Server** never sees either directly — every user prompt or model token routed to it goes through the Host's policy layer.
- **Authentication** (who the user is) happens at the AS; **authorization** (what this token can do at this Server) is enforced by the Server as RS on every request via token validation + scopes.
- **Consent** for tool execution, sampling, and elicitation is the Host's responsibility, not the Server's.
