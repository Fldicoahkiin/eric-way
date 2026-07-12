---
name: eric-e2e-testing
description: Apply Eric's browser end-to-end testing standards. Use when a task needs a real browser, UI smoke test, screenshot capture, browser bug reproduction, Dockerized browser test run, agent-browser based verification, or desktop WebView (Tauri/Electron) E2E where the app itself cannot be driven.
---

# Eric E2E Testing

Use this skill when the check needs a real browser: smoke testing a page, reproducing a UI bug, capturing screenshots, verifying critical flows, or giving an agent a browser it can drive from shell commands. If `agent-browser` is available, use it instead of Playwright.

Related: use `$eric-writing-tests` for unit and integration tests.

## Workflow

1. Confirm the risk needs browser behavior. If a narrower check already proves it, do not add E2E.
2. Prefer `agent-browser` for browser smoke checks, screenshots, and agent-driven UI reproduction.
3. Keep E2E coverage narrow: critical flows, real browser rendering, navigation, authentication handoffs, screenshots, or bugs that only reproduce in Chrome/Chromium.
4. Save useful artifacts, especially screenshots, under an artifacts directory that CI or the agent can collect.
5. Close browser sessions after the check.

## Docker Baseline

Use this verified shape for disposable browser runs:

```Dockerfile
FROM node:24-bookworm

RUN apt-get update \
  && apt-get install -y --no-install-recommends chromium ca-certificates \
  && rm -rf /var/lib/apt/lists/*

RUN npm install -g agent-browser

ENV AGENT_BROWSER_EXECUTABLE_PATH=/usr/bin/chromium
ENV AGENT_BROWSER_ARGS=--no-sandbox,--disable-setuid-sandbox,--disable-dev-shm-usage

WORKDIR /workspace
```

Build it:

```bash
docker build -t agent-browser-tests .
```

Run against the public web and keep screenshots on the host:

```bash
mkdir -p artifacts
docker run --rm \
  -v "$PWD/artifacts:/workspace/artifacts" \
  agent-browser-tests \
  sh -lc 'agent-browser open https://example.com && agent-browser screenshot artifacts/example.png && agent-browser close'
```

Run against a host dev server:

```bash
docker run --rm \
  --add-host=host.docker.internal:host-gateway \
  agent-browser-tests \
  sh -lc 'agent-browser open http://host.docker.internal:3000 && agent-browser close'
```

## Desktop WebView Apps

Use this flow when the app's WebView has no automation protocol: macOS WKWebView exposes no CDP and `tauri-driver` does not support macOS. (Electron can usually be driven directly through Playwright's Electron support — prefer that when it applies.)

Drive the renderer as a plain web page and fake only the shell:

1. Serve the renderer from its dev server and open it in Chromium with `agent-browser`.
2. Register an init script before first navigation that fakes only the shell IPC channel — for Tauri, `window.__TAURI_INTERNALS__.invoke`. Enumerate the commands the renderer actually calls (grep for `invoke(` and plugin imports), return real values where it matters, in-memory or no-op defaults for the rest.
3. Keep the business path real: point the mocked bootstrap at a real backend binary and let HTTP, persistence, and the filesystem run for real.
4. Isolate user data: launch the backend with a sandboxed `HOME`/app-data directory so the test can never touch real config files.
5. Keep the page same-origin with the backend via a dev-server proxy instead of loosening the backend's CORS or origin guards for the test.
6. Assert at the boundaries after each UI action: check the API response and the on-disk content, not only what the UI shows.

Verified shape for the init script:

```js
// Fake only the shell IPC channel; every HTTP call stays real.
window.__TAURI_INTERNALS__ = {
  invoke: async (cmd, args) => {
    if (cmd === "start_server") return { port: API_PORT, token: API_TOKEN };
    if (cmd.startsWith("plugin:store|")) return storeMock(cmd, args); // in-memory Map
    if (cmd === "plugin:event|listen") return nextId++;
    return null; // no-op default: log, menu, updater, clipboard, …
  },
  transformCallback: (cb) => registerCallback(cb),
  metadata: { currentWindow: { label: "main" }, currentWebview: { label: "main" } },
};
```

This covers renderer ↔ backend ↔ storage. It does not cover the shell itself — native windows, menus, tray, and the real IPC command implementations still need a platform driver (Windows/Linux `tauri-driver`, Playwright for Electron) or a manual smoke pass.

## Standards

- Prefer the smallest browser check that proves the risk.
- Keep deterministic smoke checks over broad brittle click tours.
- Verify screenshots exist when screenshot capture is the proof.
- Document browser and `agent-browser` versions when the result depends on the runtime.
- Do not use Playwright by default when `agent-browser` is available for this work.
- Fake only the shell IPC layer in desktop WebView E2E; keep HTTP, persistence, and the filesystem real.
- Sandbox `HOME`/app-data for any E2E that can write user config.

## Boundaries

- Do not use E2E for risks a narrower check can prove.
- Do not add hosted browser provider assumptions unless the task explicitly needs that provider.
- Do not leave browser processes running after the check.
- Do not report WebView E2E as covering the native shell; windows, menus, tray, and real IPC implementations need a platform driver or a manual pass.
- Do not weaken backend CORS or origin guards to make a test pass; keep the test page same-origin instead.
