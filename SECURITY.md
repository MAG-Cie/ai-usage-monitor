# Security

## Reporting a vulnerability

Email **antoine.guittet@mag-cie.com** with a description and reproduction steps.
Please do not open a public issue for security reports. Expect an acknowledgement
within a few business days.

## How the app handles your data

- **Local-first.** Usage data is fetched from the provider APIs you configure and
  kept on your machine. There is no server operated by us — no telemetry, no
  analytics, no crash reporter. See [NETWORK.md](NETWORK.md) for the exhaustive
  list of network destinations.
- **Secrets at rest.** API keys and session cookies are encrypted with the OS
  keychain — Windows DPAPI via Electron `safeStorage`. They never touch disk in
  clear text and are never written to logs. If OS encryption is unavailable, the
  app tells you so in Settings rather than silently storing secrets in clear.
- **Secrets in transit.** Keys are sent only to their own provider's API over
  HTTPS, and the Pro license key only to Lemon Squeezy's license API.
- **Least privilege.** Read-only provider credentials are sufficient (admin/usage
  read scopes). The app never needs write access to your provider accounts.

## Build integrity

- Releases are built by GitHub Actions from a tagged commit and published as
  GitHub releases. `electron-updater` verifies each update against the signed
  metadata (`latest.yml` + blockmap) published alongside the installer.
- The distributed build **excludes** the personal `src/personal/**` sources and
  all test files (see `build.files` in `package.json`).

## Code signing

Early releases are **not** signed with an OV certificate, so Windows SmartScreen
may show a warning on first run ("unknown publisher"). This is expected. Verify
your download against the checksums / VirusTotal link on the release page before
running. Code signing will be added once the product is established.
