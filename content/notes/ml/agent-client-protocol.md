---
title: Agent Client Protocol (ACP)
description: Explains ACP, a JSON-RPC protocol connecting editors to AI coding agents with client-brokered tools.
tags:
  - acp
  - protocol
  - agents
  - jsonrpc
  - zed
date: "2026-05-28T15:43:52Z"
lastmod: "2026-05-28T15:44:33Z"
draft: false
---

ACP is an open, JSON-RPC 2.0–based protocol (originated by Zed Industries) that standardizes the wire between **code editors / IDEs** and **AI coding agents** — conceptually the same idea as LSP, but for agents instead of language servers. It lets any compliant editor host any compliant agent (Claude Code, Gemini CLI, custom agents, etc.) without each pair needing a bespoke integration.

Transport is typically **stdio**: the editor spawns the agent as a child process and they exchange JSON-RPC messages over stdin/stdout.

## The two roles

- **Client** — the editor/host (e.g., Zed, Neovim). Owns the UI, the workspace, the user, and gates side effects (file writes, shell commands, network).
- **Agent** — the AI process. Owns the model loop, planning, and decides *what* to do. It can't touch the workspace directly; it asks the client.

The asymmetry is the whole point: **the agent is sandboxed and the client is the trusted broker** for filesystem, terminal, and user consent.

## Core concepts

| Concept | What it is |
|---|---|
| **Session** | A conversation context with history and a working directory. Identified by a `sessionId`. |
| **Prompt** | A user turn submitted into a session. |
| **Content blocks** | The unit of message content: `text`, `image`, `audio`, `resource`, `resource_link`. Used in both prompts and agent output. |
| **Tool calls** | The agent's intent to perform an action (edit a file, run a command, search, etc.), reported with status and progress. |
| **Permissions** | The client's user-facing approval gate for sensitive tool calls. |
| **Capabilities** | Negotiated at `initialize`: what the client supports (fs, terminal) and what the agent supports (load, image input, etc.). |
| **Plan** | Optional structured task list the agent streams to show its intended steps. |

## The main methods

**Lifecycle & auth**
- `initialize` — protocol version + capability exchange
- `authenticate` — optional, if the agent advertises auth methods

**Session management** (client → agent)
- `session/new` — start a session, passing `cwd` and any MCP servers
- `session/load` — resume a prior session (if agent supports it)
- `session/prompt` — submit a user turn
- `session/cancel` — interrupt the current turn

**Streaming output** (agent → client, notifications)
- `session/update` — chunks of `agent_message`, `agent_thought`, `tool_call`, `tool_call_update`, `plan`, etc.

**Client services the agent calls back into**
- `session/request_permission` — ask the user to approve a tool call
- `fs/read_text_file`, `fs/write_text_file` — workspace I/O via the client
- `terminal/create`, `terminal/output`, `terminal/wait_for_exit`, `terminal/kill`, `terminal/release` — managed shell sessions

## How a turn flows

1. **Spawn & handshake.** Client launches the agent process; both sides exchange `initialize` with versions and capabilities.
2. **Session setup.** Client calls `session/new` (or `session/load`), gets a `sessionId`.
3. **Prompt.** User types; client sends `session/prompt` with content blocks.
4. **Agent reasons and streams.** Agent emits `session/update` notifications — text chunks, thoughts, and `tool_call` entries as it works.
5. **Agent acts via the client.** For each side effect, the agent calls back: `fs/write_text_file`, `terminal/create`, etc. If the action is sensitive, it first calls `session/request_permission`; the client shows a prompt and replies allow/deny.
6. **Tool results.** Agent streams `tool_call_update` notifications with status (`pending` → `in_progress` → `completed`/`failed`) and output, so the editor can render a live tool panel.
7. **Turn ends.** Agent resolves the `session/prompt` request with a stop reason (`end_turn`, `cancelled`, `max_tokens`, etc.). Client returns control to the user.

The key invariant: **the agent never touches the host directly** — every file read, file write, shell command, and user question is a request back into the client, which enforces policy and surfaces UI.
