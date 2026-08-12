# Network activity

AI Usage Monitor has **no backend of its own**: no telemetry, no analytics, no
crash reporter, no install counter. Every byte it sends leaves your machine for
one of three purposes, and only when the corresponding feature is configured.

All destinations below are derived directly from the source. If you audit the
build, these are the only hosts it contacts.

## 1. Provider APIs (the ones you configure)

Contacted only for a provider you have connected, to read your own usage/cost:

| Provider | Host(s) | Purpose |
|----------|---------|---------|
| Anthropic | `api.anthropic.com` | Admin usage & cost report (real spend) |
| OpenAI | `api.openai.com` | Organization costs |
| OpenRouter | `openrouter.ai` | Activity / credits |
| Google Gemini | `monitoring.googleapis.com`, `bigquery.googleapis.com`, `www.googleapis.com`, `oauth2.googleapis.com` | Cloud Monitoring (estimated) / BigQuery billing (real); `oauth2.googleapis.com` mints the short-lived service-account token |

The local "Claude Code" source reads JSONL files on disk only — **no network**.

## 2. Updates (GitHub)

`electron-updater` checks the project's GitHub releases for a newer version and,
if you accept, downloads the installer. Hosts: `github.com`, `api.github.com`,
`objects.githubusercontent.com`. No account or token is required for public
releases.

## 3. License (Lemon Squeezy)

When you activate, validate, or deactivate a Pro key, the app calls the public
Lemon Squeezy License API: `api.lemonsqueezy.com`. The **Buy Pro** button opens
your default browser to the checkout page (`magcie.lemonsqueezy.com`) — the app
itself never handles payment data.

## Personal build only (not in the sold app)

The dedicated "perso" build (`aiumPersonal`) additionally reads your personal
claude.ai Max gauges from `claude.ai`, using your browser session cookie. This
source lives under `src/personal/**` and is **excluded from the distributed
build** — the sold app never contacts `claude.ai`.

## Secrets

API keys and session cookies are stored with the OS keychain (Windows DPAPI via
Electron `safeStorage`). They are never written to disk in clear text and are
never logged. See [SECURITY.md](SECURITY.md).
