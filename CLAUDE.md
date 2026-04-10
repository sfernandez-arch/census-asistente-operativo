# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

"Néstor" is an internal operational assistant for Census, an accounting firm in Montevideo, Uruguay. It is a **single-file web application** (`index.html`) with no build system, package manager, or test framework. To develop, open `index.html` directly in a browser or use any static file server.

## Architecture

The entire app lives in `index.html` — inline CSS in `<style>`, HTML markup, and all JavaScript in a single `<script>` block. There are no external JS/CSS files aside from Google Fonts and the SheetJS (XLSX) CDN library.

### Screens and flow

The app has four screens toggled via `showScreen()`:
1. **Login** (`#loginScreen`) — email-based auth against a hardcoded `ALLOWED_USERS` object
2. **Blocked** (`#blockedScreen`) — shown when a user has ≥3 consecutive days without sending a daily summary; requires a 6-digit unlock code
3. **Admin** (`#adminScreen`) — shown to `socio` and `encargado` roles; generates time-limited unlock codes for blocked users
4. **Chat** (`#chatScreen`) — the main interface; sends messages to the Claude API via a Cloudflare Worker proxy

### Backend proxy

All API calls go to `https://census-proxy.sfernandez-8a7.workers.dev` (Cloudflare Worker). The proxy forwards requests to the Anthropic Claude API. The worker code is **not** in this repo.

- Chat: `POST` to `API_URL` with Claude messages format (model, max_tokens, system, messages)
- Summary: `POST` to `SUMMARY_ENDPOINT` (`/send-summary`) with conversation data

### Key mechanisms

- **User roles**: `junior`, `senior`, `encargado`, `socio`, `tercerizado` — each with different permissions and escalation paths
- **Block/unlock system**: Uses `localStorage` to track `lastSummaryDate` per user. Roles `socio` and `tercerizado` are exempt from blocking. Unlock codes are stored in `localStorage` with 5-minute expiry.
- **System prompt**: `buildSystemPrompt()` constructs a detailed system prompt in Spanish with Uruguayan tax/labor regulations, internal procedures, team structure, and deadlines. This is the core domain knowledge of the assistant.
- **File attachments**: Supports images (base64), PDFs (base64), and Excel files (parsed client-side via SheetJS into CSV text)
- **Daily summary**: `sendSummary()` tries the proxy endpoint first; on failure, falls back to `mailto:` link

### State

All state is client-side: `localStorage` for persistence (summary dates, unlock codes), and in-memory variables (`currentUser`, `history`, `attachedFile*`) for session state. There is no database.

## Language

All UI text, system prompts, and domain content are in Spanish (Rioplatense). The assistant persona uses "vos" conjugation.
