---
title: Language Server Protocol (LSP)
description: Overview of LSP, a JSON-RPC protocol connecting editors to language servers for code intelligence.
tags:
  - lsp
  - protocols
  - editors
  - tooling
date: "2026-05-28T15:47:53Z"
lastmod: "2026-05-28T15:48:42Z"
draft: false
---

LSP is an open, JSON-RPC-based protocol — originally created by Microsoft for VS Code — that standardizes how **code editors/IDEs** talk to **language-specific tools** (parsers, type checkers, linters, formatters). Before LSP, every editor had to integrate every language separately (an *M × N* problem). LSP collapses this to *M + N*: each editor implements the protocol once, and each language implements a server once.

## Components

LSP defines three components:

1. **Language Client**
   Lives inside the editor/IDE (e.g., VS Code, Neovim, Emacs, JetBrains via plugin). Responsible for:
   - Spawning/managing the language server process
   - Forwarding user actions (file opened, cursor moved, save, completion request) to the server
   - Rendering server responses (diagnostics, completions, hovers) in the UI

2. **Language Server**
   A standalone process specific to one language (e.g., `gopls` for Go, `rust-analyzer` for Rust, `pyright` for Python, `typescript-language-server` for TS). Responsible for:
   - Parsing the workspace
   - Producing diagnostics, completions, hovers, definitions, references, rename edits, formatting, code actions
   - Maintaining its own in-memory model of the project

3. **The Protocol itself (JSON-RPC 2.0 over stdio / pipes / sockets)**
   Defines the message shapes. Three message kinds:
   - **Requests** — expect a response (client → server or server → client), e.g., `textDocument/completion`, `textDocument/definition`
   - **Responses** — reply to a request, with `result` or `error`
   - **Notifications** — fire-and-forget, e.g., `textDocument/didChange`, `textDocument/publishDiagnostics`

## How they interact

The lifecycle of a session:

```
┌─────────────┐                          ┌──────────────────┐
│   Editor    │                          │ Language Server  │
│  (Client)   │                          │  (e.g. gopls)    │
└──────┬──────┘                          └─────────┬────────┘
       │                                           │
       │ ── initialize (capabilities) ────────────▶│
       │ ◀──── InitializeResult (capabilities) ────│
       │ ── initialized (notification) ───────────▶│
       │                                           │
       │ ── textDocument/didOpen ─────────────────▶│   (server parses file)
       │ ◀── textDocument/publishDiagnostics ──────│   (errors/warnings pushed)
       │                                           │
       │ ── textDocument/completion ──────────────▶│   (user typed `.`)
       │ ◀──────── CompletionList ─────────────────│
       │                                           │
       │ ── textDocument/didChange ───────────────▶│   (incremental edits)
       │ ◀── textDocument/publishDiagnostics ──────│
       │                                           │
       │ ── textDocument/definition ──────────────▶│   (user hit "Go to def")
       │ ◀──────── Location[] ─────────────────────│
       │                                           │
       │ ── shutdown ─────────────────────────────▶│
       │ ── exit (notification) ──────────────────▶│
```

Key interaction patterns:

- **Capability negotiation**: at `initialize`, both sides advertise what they support (e.g., does the server provide rename? does the client support snippet completions?). Neither side assumes — they only use features both agreed on.
- **Document sync**: the client owns the source of truth for unsaved buffer content and streams `didOpen`/`didChange`/`didSave`/`didClose` notifications so the server's view stays consistent with what the user sees.
- **Pull vs push**: most features are **pull** (client requests on demand: hover, completion, definition). **Diagnostics** are traditionally **push** (server publishes whenever it re-analyzes), though LSP 3.17+ also added pull diagnostics.
- **Bidirectional**: the server can also send requests *to* the client (e.g., `workspace/applyEdit` to apply a refactor, `window/showMessage` to surface a notification).

The whole idea: a small, well-defined contract between two cooperating processes, which is why one server (say `rust-analyzer`) now works in dozens of editors with zero per-editor effort.