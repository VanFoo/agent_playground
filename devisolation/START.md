# Starting the Isolated Dev Container

This brings up two containers:

- **`devcontainer`** — where you work, with Claude Code installed. Has no
  direct path to the internet.
- **`firewall`** — a forward proxy. The *only* way `devcontainer` can reach
  the internet, and only for domains listed in `firewall/whitelist.txt`.

See `PRD.md` in this folder for the full design rationale.

## Prerequisites (Windows + Docker Desktop)

1. **Docker Desktop** installed and running, with the **WSL2 backend**
   enabled (Settings → General → "Use the WSL 2 based engine" — this is the
   default on modern installs).
2. **WSL2** installed if it isn't already (`wsl --install` from an admin
   PowerShell).
3. In Docker Desktop → Settings → Resources → WSL Integration, make sure
   integration is turned on for whichever WSL distro / terminal you'll run
   commands from.
4. **VS Code** with the **Dev Containers** extension — needed for "Reopen
   in Container" (recommended way to start this). If you'd rather drive it
   from a terminal only, see the `devcontainer` CLI option below instead.
5. Make sure the project folder you plan to mount is somewhere Docker
   Desktop can actually share — anything under your Windows user profile
   works out of the box; otherwise add it under Settings → Resources →
   File Sharing.

> **Why not just `docker compose up`?** That starts both containers, but
> `devcontainer` won't have Claude Code or the GitHub CLI installed.
> Those come from devcontainer *features*, which are only applied when
> launched through VS Code's "Reopen in Container" or the `devcontainer`
> CLI — `docker compose` itself doesn't know about them. Plain
> `docker compose up` is fine for testing the firewall on its own, but use
> one of the two methods below for actual development.

## 1. Configure the project mount

```
cd devisolation
cp .env.example .env
```

Edit `.env` and set `HOST_PROJECT_PATH` to the absolute path of the project
you want to work on:

```
HOST_PROJECT_PATH=C:/Users/yourname/code/myproject
```

Use forward slashes, even on Windows. This gets mounted into the
devcontainer at `/workspace`.

## 2. Start it

**Option A — VS Code (recommended)**

1. Open the `devisolation` folder in VS Code.
2. Command Palette → **Dev Containers: Reopen in Container**.
3. VS Code builds and starts both `firewall` and `devcontainer`, applies the
   Claude Code / GitHub CLI features, and opens a terminal inside
   `devcontainer` with `/workspace` as the working directory.

**Option B — terminal only, via the devcontainer CLI**

```
npm install -g @devcontainers/cli   # one-time
devcontainer up --workspace-folder .
devcontainer exec --workspace-folder . bash
```

Both options bring up the firewall sidecar automatically and apply the same
features — pick whichever fits your workflow.

## 3. Log in to Claude Code

First run, inside the devcontainer:

```
claude login
```

The session is stored in a named Docker volume (`claude-config`), not the
container's own filesystem, so it survives a rebuild — you won't need to
log in again next time you change `devcontainer.json` and rebuild.

## 4. Adding a new whitelist domain

Since this starts whitelist-only, expect to hit a blocked domain at some
point. When something fails with a vague network error:

1. Check what got blocked:

   ```
   docker compose logs -f firewall
   ```

   Denied requests show up with the domain that was attempted.

2. Open `firewall/whitelist.txt` and add it on its own line. Prefer the
   exact hostname over a wildcard (e.g. `pypi.org`, not `.org`) — narrower
   entries keep the whitelist meaningful.

3. Reload Squid without restarting the container:

   ```
   docker compose exec firewall squid -k reconfigure
   ```

4. Retry whatever failed inside the devcontainer.

## Stopping

```
docker compose down
```

Add `-v` only if you also want to delete the named volumes (Claude Code
login state, command history) — you normally don't want this, since it
means logging in again next time.
