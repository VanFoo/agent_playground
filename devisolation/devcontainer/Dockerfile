FROM mcr.microsoft.com/devcontainers/base:ubuntu

# Intentionally minimal and language-agnostic: Claude Code, the GitHub CLI,
# etc. are layered on top by devcontainer *features* declared in
# ../.devcontainer/devcontainer.json. Those are applied automatically when
# this is started via VS Code's "Reopen in Container" or the `devcontainer`
# CLI (`devcontainer up --workspace-folder .`).
#
# Plain `docker compose up` will start this bare base image WITHOUT those
# tools, since features are a devcontainer-spec concept that docker-compose
# itself doesn't know about. Use one of the two launch methods above to get
# the full environment — see ../START.md.
