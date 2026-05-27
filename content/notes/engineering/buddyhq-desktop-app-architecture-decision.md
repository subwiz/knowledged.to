---
title: BuddyHQ Desktop App — Architecture Decision
description: Architecture decision record for BuddyHQ desktop app, selecting Electron for v1 with cloud BFF, evaluating PWA, Tauri, and SwiftUI alternatives
tags:
  - architecture
  - desktop
  - electron
  - tauri
  - buddyhq
  - appui
  - appwebexpms
date: "2026-04-28T10:19:41.347873Z"
lastmod: "2026-04-28T15:50:30+05:30"
draft: false
---
# BuddyHQ Desktop App — Architecture Decision

Date: 2026-04-28

## Stack Being Wrapped

- appui: React 18 + Vite 6 SPA — FabricJS canvas, Revideo video editor, SSE chat, Zustand, TanStack Query, private packages (@buddyhq/richtext-editor, @buddyhq/video-renderer)
- appwebexpms: Go 1.25 BFF at app.buddyhq.ai — cookie-based JWT auth, SSE proxied to mwms, 16 downstream service clients, 62 handler files

## Options Evaluated

### PWA — appui reuse 97%, appwebexpms 100%
Days to ship. No bundle overhead. But: not a real desktop app, no system tray/global shortcuts/native menus, not App Store distributable.

### Electron + appui (cloud BFF) — RECOMMENDED FOR V1
appui reuse ~90%, appwebexpms 100% unchanged. Bundle ~200 MB. 2–4 weeks to v1.
- Chromium renderer eliminates all WebKit risk for FabricJS and Revideo
- appwebexpms needs zero changes — desktop is just another browser client
- Cookie JWT + SSE work natively in Electron
- IPC layer is TypeScript (no new language for Go team)
- electron-builder, electron-updater, code signing all solved
- Cons: large bundle, memory ~300–600 MB at idle, Mac App Store is painful

Minimal structure: electron/main.ts (BrowserWindow + IPC), electron/preload.ts (contextBridge), appui/src unchanged + platform.ts shim for native vs web APIs.

### Tauri 2.0 + appui (cloud BFF) — V2 if mobile matters
appui reuse ~85%, appwebexpms 100% unchanged. Bundle ~15 MB. 4–8 weeks to v1.
- Much smaller bundle, better security model, Tauri 2.0 supports iOS + Android
- Serious risk: WebKit ≠ Chromium — FabricJS, Revideo, and @buddyhq/video-renderer are untested on WebKit. Budget 2–3 weeks for WebKit fixes.
- SSE buffering issues on WebKit require careful testing
- Cookie SameSite config for app.buddyhq.ai needs Tauri http plugin setup
- Rust IPC (Tauri commands) is a new language for the Go team

### Tauri + Local Go Sidecar — NOT for v1
appwebexpms cannot be reused as-is (depends on Valkey, GCS, 16 K8s microservices). Would need a new appdesktopbff stripping cloud dependencies. Doubles maintenance surface. 4–6 months.

### Native SwiftUI — NOT for v1
Zero appui reuse. Full rewrite: canvas editor, image editor (FabricJS), video timeline (Revideo), SSE chat, rich text. macOS only. 12–24 months.

## Decision

**Electron for v1.** appwebexpms unchanged — desktop app points at https://app.buddyhq.ai.

If mobile (iOS/Android) lands on the 12-month roadmap, switch to Tauri 2.0 from the start and absorb the WebKit debugging cost early.

## appwebexpms Changes for v1
None. If offline support needed later: add SQLite cache in Electron main process (better-sqlite3). If desktop-specific APIs needed: add /desktop/* route namespace to appwebexpms.