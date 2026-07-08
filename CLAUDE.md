# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file, no-build static web page (`index.html`) that picks a random album from the KINK Album Top 1000 (2025) list and lets the user open/play it on Spotify or Apple Music. There is no server, package manager, build step, linter, or test suite — the entire app is one HTML file with inline `<style>` and `<script>`.

Deploy is just: push `index.html` (e.g. to GitHub Pages/Netlify) and open it over HTTPS.

## Running it locally

There's no dev server or tooling. Open `index.html` directly in a browser, or serve the directory with any static file server (e.g. `python3 -m http.server`). Spotify login (PKCE OAuth) requires the page to be served from the exact Redirect URI registered in the Spotify dashboard, so `file://` works for the random-pick feature but not for the Spotify-connected features.

## Code layout (all in `index.html`)

- **Lines ~1-91**: HTML markup and inline CSS. UI text is in Dutch. Key elements: `#titel`/`#artiest` (the drawn album), `#spotify`/`#apple` buttons (open the drawn album on each service), `#mijnKnop` (Spotify-library random pick), `#status`/`#setup` (feedback and first-run instructions).
- **Line 102**: `const ALBUMS` — the full 1000-entry list as `{"a": artist, "t": title}` objects, all on one very long line. When editing entries, use targeted string replacement rather than reading/rewriting the whole line.
- **Lines 94-397**: the script, in two parts:
  - **Random draw + service links** (~119-186): `trekIndex(n)` does unbiased random selection via `crypto.getRandomValues` (rejection sampling to avoid modulo bias) — reuse this instead of `Math.random()` for anything draw-related. `toon()` draws and renders an album. The Spotify/Apple buttons build search/open URLs from the drawn album; the Spotify button additionally tries to auto-play via the Web API if a token is available.
  - **Spotify PKCE integration** (~188-392): OAuth Authorization Code + PKCE flow (`login`, `handleRedirect`, `saveToken`/`getValidToken` with refresh) entirely client-side, tokens kept in `localStorage`. `playAlbum` picks a playback device matching the current device type (`kiesApparaat`) deliberately — it does not send playback to an unrelated device — and falls back to opening the album in the browser/app when there's no usable device, no Premium, or no token.

## Configuration

`CLIENT_ID` (line ~96) is the Spotify app Client ID and is currently blank. Without it, Spotify auto-play/library features are disabled and clicking related buttons shows first-run setup instructions (`toonSetup()`) instead of failing silently. The redirect URI is derived automatically as `location.origin + location.pathname`, so it must be registered exactly (including path) in the Spotify Developer Dashboard.

## Conventions to preserve

- Identifiers, comments, and user-facing copy are in Dutch — match this when adding code or strings.
- No frameworks, bundlers, or external dependencies are used; keep it a dependency-free single file.
- Failure paths degrade to opening a search/album URL in a new tab rather than showing a dead end — follow this pattern for new integrations.
