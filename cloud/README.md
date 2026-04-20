# Edrin Claw — cloud deploy scaffolding

This directory contains everything needed to run our OpenClaw fork in
Coolify at `agents.edrintravel.cloud`. Upstream OpenClaw files at the
repo root are left untouched so `git merge upstream/main` stays trivial.

## Layout

```
cloud/
├── README.md                       ← this file
├── docker-compose.coolify.yml      ← single-gateway compose Coolify deploys
├── Dockerfile                      ← thin wrapper: upstream build + bake config
└── config/                         ← contents land in /home/node/.openclaw/
    ├── agents/                     ← agent definitions committed to git
    │   └── .gitkeep
    └── skills/                     ← custom skills committed to git
        └── .gitkeep
```

## Continuous deployment flow

```
local edit cloud/config/… ─► git push ─► Coolify webhook ─► docker build
                                                              │
                                                              ▼
                                       image with baked-in agents/skills
                                                              │
                                                              ▼
                                           agents.edrintravel.cloud:443
                                             (Traefik → gateway:18789)
```

- **Runtime secrets** (gateway token, provider API keys) live in Coolify env
  vars, never in git.
- **Runtime state** (conversations, memory, the `openclaw.json` file the
  gateway writes back) persists in a Docker volume mounted at
  `/home/node/.openclaw`. First boot seeds that volume from the image layer
  we baked in; subsequent boots keep whatever the gateway wrote.

## Coolify env vars to populate

| Key | Notes |
|-----|-------|
| `OPENCLAW_GATEWAY_TOKEN` | Generate with `openssl rand -hex 32`; clients use this as bearer auth. |
| `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS` | Leave empty unless debugging. |
| `OPENCLAW_TZ` | `America/Santo_Domingo` to match Edrin. |
| `CLAUDE_AI_SESSION_KEY` | Optional — only if using the Claude web session integration. |

## Initial deploy commands (from ops workstation, once)

1. `git remote set-url origin https://github.com/qwiktech-so/edrin-claw.git`
2. `git checkout -b main` (if not already)
3. `git add cloud/ && git commit -m "chore: Coolify cloud deploy scaffolding"`
4. `git push -u origin main`
5. Coolify API call creates the Application (done via script in `~/.claude/secrets/` land, not here).

## Keeping up with upstream

```bash
git remote add upstream https://github.com/openclaw/openclaw.git
git fetch upstream
git merge upstream/main      # resolve conflicts (likely none in cloud/ or Dockerfile)
git push origin main
```
