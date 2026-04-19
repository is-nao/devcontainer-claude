# devcontainer-claude

A Dev Container setup for running [Claude Code](https://claude.ai/code) with a network firewall that restricts outbound traffic to a curated allowlist.

## Features

- **Claude Code** pre-installed and ready to use
- **iptables firewall** that blocks all outbound traffic except explicitly allowed domains
- **Node.js 24** (Debian Trixie slim) with pnpm
- **zsh** with Oh My Zsh (robbyrussell theme), fzf, autosuggestions, and completions
- **git-delta** for improved diff output
- Persistent shell history and Claude config via Docker volumes
- VS Code extensions for Nuxt / Vue development pre-configured

## Requirements

- [Docker](https://www.docker.com/)
- [VS Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- A valid `GITHUB_PERSONAL_ACCESS_TOKEN` environment variable on the host (used by the firewall init script to fetch GitHub IP ranges)

## Getting Started

1. Clone this repository alongside your project, or copy `.devcontainer/` into your project root.

2. Set your GitHub token on the host:

   ```bash
   export GITHUB_PERSONAL_ACCESS_TOKEN=ghp_xxxxxxxxxxxx
   ```

3. Open the project in VS Code and choose **Reopen in Container**.

4. Authenticate Claude Code inside the container:

   ```bash
   claude
   ```

## Firewall

The firewall (`init-firewall.sh`) is applied on container start via `postStartCommand`. It uses `iptables` + `ipset` to whitelist the following outbound destinations:

| Destination | Purpose |
|---|---|
| GitHub IP ranges (from `/meta` API) | git, gh CLI, GitHub packages |
| `registry.npmjs.org` | npm / pnpm packages |
| `api.anthropic.com` | Claude API |
| `claude.ai` | Claude Code authentication |
| `statsig.anthropic.com`, `statsig.com` | Telemetry |
| `marketplace.visualstudio.com`, `vscode.blob.core.windows.net`, `update.code.visualstudio.com` | VS Code extensions |
| `fonts.googleapis.com`, `fonts.gstatic.com`, `cdn.jsdelivr.net` | Web assets |
| Host network (`/24`) | Local development |

DNS (UDP 53) and SSH (TCP 22) are always allowed. All other outbound traffic is rejected.

To disable the firewall for local development, set `ENABLE_FIREWALL=false` in `compose.yaml`:

```yaml
environment:
  - ENABLE_FIREWALL=false
```

If you don't need the firewall at all, just tell Claude Code — it can remove the firewall script, the `NET_ADMIN`/`NET_RAW` capabilities, and the `postStartCommand` for you.

## Volumes

| Volume | Mount point | Purpose |
|---|---|---|
| `claude-config` | `/home/node/.claude` | Persists Claude Code configuration and auth across rebuilds |
| `bash-history` | `/commandhistory` | Persists shell history across rebuilds |

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `TZ` | `Asia/Tokyo` | Timezone |
| `ENABLE_FIREWALL` | `false` | Set to `true` to enable the iptables firewall |
| `GITHUB_PERSONAL_ACCESS_TOKEN` | *(from host)* | Required when firewall is enabled |
| `NODE_OPTIONS` | `--max-old-space-size=4096` | Node.js memory limit |

> **Note:** After changing any environment variable in `compose.yaml`, rebuild the container (**Dev Containers: Rebuild Container**) for the changes to take effect.

## License

[MIT](LICENSE)
