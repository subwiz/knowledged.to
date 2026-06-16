---
title: 'MCP vs Plugins in Claude Code: which to use when'
description: 'MCP vs plugins: layered distinction (external systems vs assistant-extension packaging); both cross-surface across Claude Code, Cowork, and the Claude apps'
tags:
  - claude-code
  - mcp
  - plugins
  - architecture
date: "2026-06-15T10:26:47Z"
lastmod: "2026-06-15T11:08:55Z"
draft: false
---

MCP and plugins are often framed as alternatives, but they live at different layers, and the best setups use both together. Both are also cross-surface — they work across Claude Code, Cowork, and the Claude apps (claude.ai / desktop) — so the surface you're on is *not* the distinguishing axis.

## The core distinction

**MCP (Model Context Protocol)** = how Claude *connects to external systems*. An MCP server is a running process that exposes **tools, resources, and prompts** over a protocol (stdio/SSE/HTTP). It's the integration layer — databases, APIs, SaaS apps, internal services.

**Plugins** = how you *package and distribute extensions to the Claude assistant/harness*. A plugin is an installable bundle that can contain **skills, slash commands, subagents, hooks, and MCP server configs**. (Plugins originated in Claude Code but plugin/skill support now spans Cowork and the Claude apps as well.)

The key thing: **MCP is one of the things a plugin can bundle.** They're not competing — a plugin is the wrapper, MCP is one possible ingredient.

```
Plugin (distribution unit)
 ├── skills/          (workflows, knowledge)
 ├── commands/        (slash commands)
 ├── agents/          (subagents)
 ├── hooks/           (lifecycle automation)
 └── .mcp.json        (MCP server config)  ← MCP lives here
```

## When to use which

| You want to… | Use |
|---|---|
| Connect Claude to an external system (DB, API, Jira, K8s, your backend) | **MCP server** |
| Give Claude real-time data or the ability to take actions in another system | **MCP server** |
| Package a workflow/process/prompt-knowledge for reuse | **Skill** (often inside a plugin) |
| Add a `/command` shortcut | **Plugin** (commands) |
| Distribute a curated bundle of skills + commands + agents + integrations to a team | **Plugin** |
| Automate behavior on lifecycle events (on-save, on-stop, session-start) | **Plugin** (hooks) |

## Rules of thumb

1. **Need to reach an external system (live data or actions)?** → MCP. That's literally what it's for.

2. **Need to share *how to work* — a process, a set of commands, conventions?** → Plugin.

3. **Building something for a team to install in one step?** → Plugin, and put the MCP config *inside* it. Then teammates get the integration + the skills that know how to use it, together.

4. **Just connecting one tool for yourself?** → Bare MCP server config (in `.mcp.json` or settings) is enough; no plugin needed.

## One trap to avoid

Don't reach for MCP just to add *knowledge or instructions* — that's what **skills** are for, and they're cheaper (loaded on demand, no running process). MCP earns its weight when there's an actual external system with live data or side effects. If you're tempted to write an MCP server that just returns static text or guidance, write a skill instead.

**TL;DR:** MCP = connections to the outside world. Plugins = packaging for assistant extensions (which may *include* MCP). Both are cross-surface (Claude Code, Cowork, Claude apps). They compose; pick MCP for integration, plugins for distribution.
