# Claude Code Devcontainers — iO Digital

Devcontainer configurations for running **Claude Code** in an isolated, reproducible Docker environment. These are flavours of [Claude Code's official devcontainer](https://github.com/anthropics/claude-code), tweaked to meet the demands of iO Digital's organisation.

Two variants are provided:

| Variant | Firewall | Use case |
|---|---|---|
| `HACK DEVCON` | Enabled (allowlist) | Hackathons, restricted environments |
| `HACK DEVCON - NO FIREWALL` | Disabled | Unrestricted development |

---

## What's included

Both variants come pre-installed with:

- **Claude Code** (`@anthropic-ai/claude-code`) — Anthropic's CLI for Claude
- **Beads** (`@beads/bd`) — Git-backed issue tracker for multi-session work
- **Node.js 20** — Runtime and npm
- **Python 3.11** — With `uv`, `ruff`, `mypy`, `poetry`, and common packages pre-cached (`langchain`, `langgraph`, `pydantic`, `pytest`, etc.)
- **git-delta** — Enhanced git diffs
- **Zsh** — With Powerlevel10k theme and fzf integration
- **Tools** — `gh`, `jq`, `nano`, `vim`, `tmux`, `tree`

Persistent Docker volumes keep your shell history and Claude configuration intact across container rebuilds.

---

## Firewall (HACK DEVCON variant)

The `HACK DEVCON` variant runs `init-firewall.sh` on startup. It uses `iptables` + `ipset` to drop all outbound traffic except to an explicit allowlist:

- **GitHub** — All IP ranges from the GitHub meta API (web, API, git)
- **GitLab** — `gitlab.com`
- **npm** — `registry.npmjs.org`
- **PyPI** — `pypi.org`, `files.pythonhosted.org`, `pythonhosted.org`, `pydantic.dev`
- **Anthropic** — `api.anthropic.com`, `statsig.anthropic.com`, `statsig.com`, `sentry.io`
- **VS Code** — `marketplace.visualstudio.com`, `vscode.blob.core.windows.net`, `update.code.visualstudio.com`
- **Atlassian** — `atlassian.net`
- **Host network** — The Docker host subnet (for local services)
- **DNS and SSH** — Always allowed

After applying rules, the script self-verifies: it checks that `example.com` is unreachable and `api.github.com` is reachable.

### Adding custom domains

Create a file at `.devcontainer/firewall-domains.local` (one domain per line, not tracked by git):

```
# My project's API
api-v2.bonzai.iodigital.com
my-api.example.com
internal-service.company.io
```

This file is loaded automatically on container startup.

---

## Getting started

### 1. Copy the devcontainer into your project

Pick the variant you want and copy its `.devcontainer` folder into the root of your project:

```bash
# With firewall
cp -r "HACK DEVCON/.devcontainer" /path/to/your-project/

# Without firewall
cp -r "HACK DEVCON - NO FIREWALL/.devcontainer" /path/to/your-project/
```

### 2. Merge the docker-compose.yml

If your project already has a `docker-compose.yml`, merge the `claude-code-sandbox` service and its volumes into it. If not, copy it directly:

```bash
cp "HACK DEVCON/docker-compose.yml" /path/to/your-project/
```

The `docker-compose.override.yml` is intentionally empty — use it for local overrides that you don't want committed.

### 3. Open in VS Code / Cursor

1. Install the **Dev Containers** extension if not already installed.
2. Open your project in VS Code.
3. Run **"Dev Containers: Reopen in Container"** from the command palette.

The container builds, applies the firewall (if applicable), and opens with Claude Code ready to use.

### Docker Compose (without VS Code)

```bash
cd your-project
TZ=Europe/Amsterdam docker compose --profile development up -d
docker compose exec claude-code-sandbox zsh
```

Then inside the container:

```bash
claude
```

---

## Configuration

### Build arguments

Both `Dockerfile` and `docker-compose.yml` accept these build args:

| Argument | Default | Description |
|---|---|---|
| `TZ` | `Europe/Amsterdam` | Container timezone |
| `CLAUDE_CODE_VERSION` | `latest` | Claude Code npm version |
| `GIT_DELTA_VERSION` | `0.18.2` | git-delta release version |
| `ZSH_IN_DOCKER_VERSION` | `1.2.0` | zsh-in-docker release version |

### VS Code extensions

Pre-configured in `devcontainer.json`:

- `anthropic.claude-code`
- `dbaeumer.vscode-eslint`
- `esbenp.prettier-vscode`
- `eamodio.gitlens`

---

## Requirements

- Docker with `NET_ADMIN` and `NET_RAW` capabilities (required for the firewall variant)
- VS Code with the Dev Containers extension, **or** Docker Compose

> **Note:** The firewall variant requires Linux kernel network capabilities. Docker Desktop on macOS and Windows supports this via the Linux VM.
