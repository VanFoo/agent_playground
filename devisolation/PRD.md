# PRD: Isolated Dev Container with Whitelisted Firewall

## Goal

Provide a self-contained, host-isolated development environment under `/devisolation` that runs Claude Code (and arbitrary mounted projects) inside a devcontainer with **no direct internet access**. All outbound traffic must pass through a sidecar firewall/proxy container that enforces a domain whitelist. Start simple: whitelist-only, HTTP/HTTPS only, easy to extend.

This is independent of the repo's existing `.devcontainer/devcontainer.json`, which remains dedicated to GitHub Codespaces and must not be modified or reused.

## Architecture

Two Docker Compose services:

1. **`devcontainer`** — the dev environment. Claude Code installed via devcontainer feature. Sits on an **internal-only** Docker network (`internal: true`) — no route to the internet at all, enforced by Docker's network model, not by in-container firewall rules.
2. **`firewall`** — a Squid forward proxy. Dual-homed: attached to the internal network (reachable by `devcontainer`) *and* a second network with real internet egress. This is the only path out.

Because the internal network has no internet gateway, `devcontainer` cannot bypass the proxy even if misconfigured — isolation is structural, not just policy-based.

## HTTPS handling

Squid uses **SNI peek-and-stare-and-splice**: it reads the plaintext SNI field of the TLS ClientHello to learn the claimed destination domain, checks it against the whitelist, then **stares** at the upstream server's TLS handshake to verify the server's actual certificate (SAN/CN) matches the claimed SNI, and only then splices the connection through untouched. If the domain isn't whitelisted, or the server's certificate doesn't match the claimed SNI, the connection is dropped. No TLS interception/MITM of application data, no CA certificate to generate or trust inside the devcontainer — Squid never decrypts the payload, it only validates the handshake metadata before splicing. Traffic stays end-to-end encrypted between the devcontainer and the real destination.

This is a deliberate requirement, not an optional hardening step: naive "peek-and-splice" (checking SNI only, without verifying the server's certificate matches it) is vulnerable to SNI spoofing — a process inside the devcontainer could claim an allowed SNI (e.g. `api.anthropic.com`) while actually connecting to an arbitrary attacker-controlled IP, and a splice-only proxy would relay the tunnel through unchecked. Validating the server certificate against the claimed SNI before splicing closes that gap.

## Protocol scope

HTTP/HTTPS (ports 80/443) only. No SSH tunneling through the proxy. Git/GitHub access must use HTTPS remotes (matches `gh` CLI's default auth mode), not `git@github.com:...` SSH remotes.

## Whitelist

- Plain-text domain list file, mounted into the `firewall` container as a volume (not baked into the image).
- Adding a domain = edit the file + run `docker compose exec firewall squid -k reconfigure` (no rebuild required).
- Squid logs denied requests (with the attempted domain) so blocked domains are easy to discover and add.

Default whitelist entries:

| Purpose | Domains |
|---|---|
| Claude Code / Anthropic | `api.anthropic.com`, `console.anthropic.com`, `claude.ai`, `statsig.anthropic.com` |
| npm / Node (Claude Code install) | `registry.npmjs.org` |
| GitHub / devcontainer features | `github.com`, `api.github.com`, `raw.githubusercontent.com`, `objects.githubusercontent.com`, `codeload.github.com`, `ghcr.io`, `pkg-containers.githubusercontent.com` |
| OS packages (apt) | `archive.ubuntu.com`, `security.ubuntu.com`, `ports.ubuntu.com` |

Additional registries (PyPI, Docker Hub, private registries, etc.) are left for the user to add later via the same workflow — not included by default, to keep the base language-agnostic.

## VS Code Dev Containers integration

`devisolation/.devcontainer/devcontainer.json` uses `dockerComposeFile` + `service` + `runServices` so VS Code's "Reopen in Container" brings up both `devcontainer` and `firewall` together. The same `docker-compose.yml` also works from a plain terminal via `docker compose up`.

`devcontainer` is configured with `HTTP_PROXY` / `HTTPS_PROXY` / `NO_PROXY` environment variables pointing at the `firewall` service.

## Base image

`mcr.microsoft.com/devcontainers/base:ubuntu` — deliberately language-agnostic (not tied to Node/TypeScript), since arbitrary host projects get mounted in. Features:

- `ghcr.io/anthropics/devcontainer-features/claude-code:latest`
- `ghcr.io/devcontainers/features/github-cli:1`

## Project mount

Host project directory is mounted into the devcontainer via an `.env`-driven variable:

- `.env` (gitignored, user-created from `.env.example`): `HOST_PROJECT_PATH=/path/to/your/project`
- `docker-compose.yml` volume: `${HOST_PROJECT_PATH}:/workspace`

Works identically for manual `docker compose up` and VS Code's "Reopen in Container", since both read Compose's `.env` resolution.

## Credential / state persistence

Claude Code login state (`~/.claude`) and shell history persist across container rebuilds via a **named Docker volume** — not a host bind mount. This avoids re-authenticating on every rebuild while keeping host credentials untouched (mounting the host's real `~/.claude` was considered and explicitly rejected: it would let any process inside the devcontainer read/write live host credentials, defeating the isolation goal).

## Deliverables

All files land under `/devisolation`:

- `docker-compose.yml` — `devcontainer` + `firewall` services, two networks, named volumes
- `.devcontainer/devcontainer.json` — VS Code integration (`dockerComposeFile`, `service`, `runServices`, proxy env vars)
- `Dockerfile` for the firewall container (Squid + SNI peek-and-splice config)
- Squid config + whitelist domain file
- `.env.example`
- `START.md` (or similar) — step-by-step instructions covering:
  - Windows + Docker Desktop prerequisites
  - How to configure `.env` and start the containers
  - How to add a new whitelist domain (edit file → reconfigure command)

## Explicitly out of scope (v1)

- TLS interception / content inspection
- SSH/non-HTTP(S) protocol tunneling through the proxy
- Pre-installed language toolchains beyond what Claude Code itself needs
- Mounting host credentials directly
