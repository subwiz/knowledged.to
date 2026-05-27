---
title: BuddyHQ Desktop App — Architecture Analysis
description: Architecture analysis and comparison of PWA, Electron, Tauri, Tauri+Sidecar, and SwiftUI options for a BuddyHQ desktop app, with Electron recommended for v1
tags:
  - architecture
  - desktop
  - electron
  - tauri
  - appui
  - appwebexpms
date: "2026-04-28T09:51:35.930106Z"
lastmod: "2026-04-28T15:33:39+05:30"
draft: false
---
# BuddyHQ Desktop App — Architecture Analysis

## Context

Codebase analyzed: /Users/subhash/code/bhq (polyrepo-in-a-folder, independent deployable services)

### Key components evaluated

| Component | What it is | Key constraints |
|---|---|---|
| `appui` | React 18 + Vite 6 SPA | FabricJS canvas, Revideo video editor, SSE chat, private packages (@buddyhq/richtext-editor, @buddyhq/video-renderer) |
| `appwebexpms` | Go 1.25 BFF at app.buddyhq.ai | Cookie-based JWT auth, SSE proxy to mwms, 16 downstream service clients |

`appui` tech stack: React 18, TypeScript, Vite 6, Tailwind CSS, Radix UI, shadcn/ui, Zustand 5, TanStack Query 5, React Router 7, FabricJS, Revideo, Framer Motion, react-dnd, react-moveable.

`appwebexpms`: Go, chi router, cookie JWT auth, Valkey (Redis-compat), GCS, 62 API handler files, SSE proxied to mwms with FlushInterval=-1.

One existing native experiment: `knowledged-mac` (SwiftUI macOS utility for the knowledged KB service — not a full app port).

---

## Option 1 — PWA

Add manifest.json + service worker to appui.

- appui reuse: ~97% | appwebexpms reuse: 100%
- Bundle: ~0 MB extra
- Pros: Zero new infra, ships in days, auto-updates, cross-platform
- Cons: Not a real desktop app. No system tray, global shortcuts, native notifications, menubar, OS file pickers. Not App Store distributable. No offline support.
- Verdict: Good for an installable shortcut, not a first-class desktop product.

---

## Option 2 — Electron + appui (cloud BFF) ✅ RECOMMENDED FOR V1

Wrap appui in an Electron shell. BrowserWindow loads the Vite build. Electron uses Chromium — cookies, SSE, REST all work natively.

- appui reuse: ~90% | appwebexpms reuse: 100%
- Bundle: ~150–250 MB

Architecture:
```
[Electron Main Process]          [Renderer = appui]
  - BrowserWindow                  - Same React SPA
  - Native menus, tray             - Talks to app.buddyhq.ai
  - Auto-updater (electron-updater)  - Cookie auth works natively
  - IPC bridge via contextBridge   - SSE works natively
  - Local file I/O via ipcMain     - All private packages included
```

Pros:
- Chromium renderer = zero compatibility risk for FabricJS, Revideo, private packages
- appwebexpms needs zero changes — desktop app is just another browser client
- Native menus, tray, global shortcuts, OS file pickers via IPC
- electron-updater, electron-builder, code signing, notarization all solved
- macOS + Windows + Linux from one codebase
- Cookie-based JWT works via Electron session API
- IPC layer is TypeScript — no new language for Go team

Cons:
- Bundle ~200 MB (Chromium included)
- Memory ~300–600 MB at idle
- Mac App Store distribution is painful (sandboxing, notarization)
- Security: nodeIntegration must be disabled, contextBridge required

appui changes needed:
- Add platform.ts utility detecting window.electron
- Route native file picker vs <input type=file> through IPC
- Deep-link handling via app.setAsDefaultProtocolClient

Minimal structure:
```
appui-desktop/
  electron/
    main.ts       — BrowserWindow, auto-updater, IPC handlers
    preload.ts    — contextBridge exposing safe IPC APIs
    tray.ts       — System tray icon + menu
  src/            — appui/src (pnpm workspace or copy)
  package.json    — electron, electron-builder, electron-updater
```

Time to v1: 2–4 weeks.

---

## Option 3 — Tauri 2.0 + appui (cloud BFF)

Tauri replaces Electron's Chromium with the OS native WebView (WebKit on macOS/iOS, WebView2 on Windows). Rust binary handles the native shell.

- appui reuse: ~85% | appwebexpms reuse: 100%
- Bundle: ~10–20 MB

Pros:
- Bundle ~10–20 MB (no bundled Chromium)
- Memory ~100–200 MB at idle
- Tauri 2.0 targets macOS + Windows + Linux + iOS + Android — mobile path
- Better security model (capabilities-based permissions)
- Mac App Store friendly
- Faster startup

Cons (serious for this stack):
- WebKit on macOS ≠ Chromium. FabricJS, Revideo, @buddyhq/video-renderer are tested against Chrome. Canvas2D, WebGL, OffscreenCanvas differences on WebKit are real. Budget 2–3 weeks of WebKit-specific fixes.
- SSE on WebKit has historically had buffering issues (fixed in recent Safari but Tauri pins specific WebKit versions). useSSEChat hook needs careful testing.
- Cookie-based JWT: Set-Cookie with SameSite requires Tauri http plugin + setCookies configuration for app.buddyhq.ai. Non-trivial.
- Rust learning curve for Go-heavy team (IPC = Tauri commands in Rust).
- @buddyhq/video-renderer may use WebCodecs/Workers in Chromium-specific ways.

Time to v1: 4–8 weeks (+ WebKit debugging).

Best for: macOS-first, bundle size is a marketing concern, mobile (iOS/Android) is on 12-month roadmap.

---

## Option 4 — Tauri 2.0 + appui + Local Go Sidecar

Tauri + a slimmed-down appwebexpms-derived Go binary as a Tauri sidecar. App talks to localhost instead of app.buddyhq.ai.

- appui reuse: ~85% | appwebexpms reuse: ~30–40% (subset of handlers)

Pros:
- True offline support for some features
- Faster response times (localhost)
- Air-gapped environments

Cons:
- appwebexpms cannot be used as-is: depends on Valkey, GCS, 16 downstream microservices in K8s. A stripped appdesktopbff must be created — significant new codebase.
- Cross-compilation toolchain for Go sidecar (GOOS=darwin/windows/linux GOARCH=arm64/amd64)
- Sidecar process lifecycle complexity (crash recovery, port conflicts, startup sequencing)
- Auth model changes: cloud BFF uses HttpOnly cookie JWT; local BFF needs Keychain/Credential Store
- Doubles maintenance surface — every appwebexpms API change must be mirrored

Time to v1: 4–6 months. Not recommended for v1.

---

## Option 5 — Native macOS SwiftUI

Build from scratch in SwiftUI consuming appwebexpms APIs directly. Existing precedent: knowledged-mac.

- appui reuse: 0% | appwebexpms reuse: 100% (as API server, unchanged)

Pros:
- Best native experience (Handoff, Spotlight, Share Sheet, VoiceOver free)
- Bundle ~5–15 MB
- Mac App Store friendly by default

Cons:
- Zero code reuse from appui — canvas, image editor (FabricJS), video timeline (Revideo), SSE chat, rich text editor all must be rebuilt
- macOS only (Windows requires separate stack)
- Team expertise is React + Go, not Swift
- 12–24+ month project for full feature set

---

## Decision Matrix

| | PWA | Electron | Tauri | Tauri+Sidecar | SwiftUI |
|---|---|---|---|---|---|
| appui reuse | ~97% | ~90% | ~85% | ~85% | 0% |
| appwebexpms reuse | 100% | 100% | 100% | 30-40% | 100% (API) |
| Bundle size | ~0 MB | ~200 MB | ~15 MB | ~25 MB | ~10 MB |
| Windows support | ✓ | ✓ | ✓ | ✓ | ✗ |
| FabricJS/Revideo risk | None | None | Medium | Medium | N/A |
| SSE risk | None | None | Low | Low | Medium |
| Cookie auth risk | None | None | Low | High | Low |
| Time to v1 | Days | 2-4 weeks | 4-8 weeks | 4-6 months | 12-24 months |
| Native OS integration | Poor | Good | Good | Good | Excellent |
| Mobile path | ✗ | ✗ | ✓ | ✓ | ✗ |
| Team expertise fit | ✓✓ | ✓✓ | ✓ | ✓ | ✗ |

---

## Recommendation

**Electron for v1. Evaluate Tauri migration for v2 if mobile becomes a priority.**

Rationale:
1. appui IS the product — canvas, video editor, image editor, SSE chat are tested against Chromium. Electron eliminates WebKit risk.
2. appwebexpms needs zero changes — desktop app is another browser client.
3. Time to market: 2–4 weeks vs 4–8 weeks for Tauri.
4. Go team doesn't need to learn Rust — Electron IPC is TypeScript.
5. Bundle size concern is real but accepted by users of Figma, Notion, Slack, Linear, VS Code.

### appwebexpms for v1: No changes needed.

Desktop app points at https://app.buddyhq.ai. If later needed:
- Offline support: Add SQLite cache layer (better-sqlite3) in Electron main process for recent conversations/assets. Cloud BFF still handles writes.
- Desktop-specific APIs: Add /desktop/* route namespace to appwebexpms for features needing desktop context (local file path handling).

### If mobile is on the 12-month roadmap: Use Tauri 2.0 from the start.
Budget 2–3 weeks extra for FabricJS/Revideo WebKit compatibility work. Tauri 2.0's iOS/Android target from the same frontend codebase is a genuine competitive advantage.