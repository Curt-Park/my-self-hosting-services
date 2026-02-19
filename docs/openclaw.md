# OpenClaw Setup

[OpenClaw](https://openclaw.ai/) is a personal AI assistant that works across messaging channels (WhatsApp, Telegram, Discord, and more). The stack (`openclaw.yaml`) deploys the gateway service with Traefik integration.

See also: [OpenClaw Docker installation](https://docs.openclaw.ai/install/docker) | [Video guide](https://youtu.be/NhJxxv3f7lI?si=Vr38L2NWy2CPhSMa)

## Prerequisites

- Network stack deployed ([network setup](network.md))
- Create the volume directories:

  ```bash
  mkdir -p ~/docker_volumes/openclaw/{config,workspace}
  ```

## Onboard

### 1. Generate the gateway token

```bash
openssl rand -hex 32
```

Save this value — it will be used as `OPENCLAW_GATEWAY_TOKEN`.

> If you lost the token, you can retrieve it from the config file after onboarding:
>
> ```bash
> cat ~/docker_volumes/openclaw/config/openclaw.json | python3 -c "import sys,json; print(json.load(sys.stdin)['gateway']['auth']['token'])"
> ```

### 2. Run onboarding

Set the config and workspace directories to point to the volume paths used by this project, then run the onboarding wizard:

```bash
export OPENCLAW_CONFIG_DIR=~/docker_volumes/openclaw/config
export OPENCLAW_WORKSPACE_DIR=~/docker_volumes/openclaw/workspace
export OPENCLAW_GATEWAY_TOKEN=<token from step 4>
make openclaw-cli ARGS="onboard --no-install-daemon"
```

When prompted during onboarding:

- Gateway bind: `lan`
- Gateway auth: `token`
- Gateway token: paste the token from step 4
- Tailscale exposure: `Off`
- Install Gateway daemon: `No`
- Enable hooks: `Yes` (accept the defaults)

The wizard also configures your AI provider credentials, which are stored in the config directory (`~/docker_volumes/openclaw/config`).

> **Note:** The health check failure at the end of onboarding is expected — the gateway is not running during onboarding. The dashboard URLs shown are temporary and only apply to the CLI container. After deploying via Portainer, access the Control UI at `https://openclaw.<your-domain>/`.

### 3. Set up providers (optional)

Use `make openclaw-cli` to run CLI commands:

```bash
make openclaw-cli ARGS="--help"            # show available commands
make openclaw-cli ARGS="channels login"    # link WhatsApp Web (QR)
make openclaw-cli ARGS="status"            # show channel health
make openclaw-cli ARGS="doctor"            # health checks + quick fixes
```

See also: [OpenClaw CLI documentation](https://docs.clawd.bot/cli)

### 4. Configure Control UI for reverse proxy access

The gateway requires device pairing for non-local connections by default. Since the gateway runs behind Traefik, the Control UI will not recognize connections as local. Add the following to `~/docker_volumes/openclaw/config/openclaw.json` inside the `gateway` object:

```json
"controlUi": {
  "allowInsecureAuth": true,
  "dangerouslyDisableDeviceAuth": true
}
```

- `allowInsecureAuth` — allows token-only auth when the gateway sees HTTP (Traefik terminates TLS, so the gateway receives plain HTTP)
- `dangerouslyDisableDeviceAuth` — skips device pairing, allowing access with the gateway token alone

This is safe when Authentik handles authentication in front of the gateway.

### 5. Stop the local containers and remove volumes

If any containers were started during the above steps, stop them and remove the Docker volumes before deploying via Portainer. The volumes created by `docker compose` have incorrect mount paths and must be recreated by Portainer:

```bash
docker compose down
docker volume rm openclaw_openclaw-config openclaw_openclaw-workspace
```

## Environment Variables

Configure the following environment variables in Portainer when deploying:

| Variable | Description | How to get |
|---|---|---|
| `HOME` | Home directory path (e.g. `/Users/<username>`) | Required for volume mount paths — other stacks inherit this automatically, but openclaw requires it explicitly |
| `DOMAIN` | Your domain name | Same as other stacks |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway authentication token | Generated in step 4 |

The AI provider credentials (`CLAUDE_AI_SESSION_KEY`, `CLAUDE_WEB_SESSION_KEY`, `CLAUDE_WEB_COOKIE`) are optional environment variable overrides. The onboarding wizard stores credentials in the config directory, so these do not need to be set if onboarding was completed.

## Deploy

Follow the [deploy on Portainer](deploy-on-portainer.md) guide with **Compose path** set to `openclaw.yaml`.

After deploying, the Control UI is accessible at `https://openclaw.<your-domain>/`.

## Authentication

After setting up [Authentik](authentication.md), [register OpenClaw as an app](register-app-authentik.md).

## Volume

OpenClaw data is stored at:

- `$HOME/docker_volumes/openclaw/config` — configuration, credentials, and session data
- `$HOME/docker_volumes/openclaw/workspace` — workspace files

## Docker Limitations

- **Global npm packages are ephemeral** — packages installed with `npm i -g` (e.g. ClawdHub) are lost when the container is recreated. Re-install after redeployment.
- **No native app integration** — macOS app (system notifications), iOS/Android apps (camera/canvas) cannot connect to a containerized gateway.
- **No systemd** — the gateway daemon cannot be managed via systemd. Portainer handles container lifecycle instead.
- **Headless browser only** — no GUI inside the container, so browser-based tools (screenshots, web automation) run in headless mode only.

## Useful Knowledge

### Attaching to the container

To open a shell inside the gateway container:

```bash
docker exec -it openclaw-gateway bash
```

Or via Portainer: **Containers** → **openclaw-gateway** → **Console** → **Connect**.

The following sections assume you are attached to the container.

### Installing ClawdHub

[ClawHub](https://docs.openclaw.ai/tools/clawhub) is the public skill registry for OpenClaw:

```bash
# Attach to openclaw-gateway and run:
npm i -g clawhub undici
```

> **Note:** Global npm packages will be lost when the container is recreated. Re-run after redeployment.

Usage:

```bash
clawhub search "<query>"
clawhub install <skill-slug>
clawhub update --all
```

### Installing Homebrew

The gateway container (Debian-based) does not include Homebrew. To install it:

```bash
# Attach to openclaw-gateway and run:
apt-get update && apt-get install -y build-essential procps curl file git
NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
source ~/.bashrc
```

After installation, `brew` is available in the current and new shell sessions.

> **Note:** Like global npm packages, this will be lost when the container is recreated.
