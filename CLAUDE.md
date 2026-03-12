# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

U-Claw wraps [OpenClaw](https://github.com/openclaw/openclaw) into two distribution forms: a USB portable version (bash/batch scripts + HTML config UI) and an Electron desktop app. Both share the same OpenClaw core dependency and Config.html interface. The repo contains only source code — runtime binaries (Node.js, node_modules, build artifacts) are downloaded at dev time and excluded from git.

## Development Setup

```bash
# Portable version
cd portable && bash setup.sh    # Downloads Node.js v22 + OpenClaw + QQ plugin to app/
bash Mac-Start.command          # Launch (Mac ARM64 only currently)

# Electron desktop app
cd u-claw-app
npm install --registry=https://registry.npmmirror.com
npm start                       # Dev mode (electron . --dev)
npm run build:mac-arm64         # Build Mac ARM64 DMG
npm run build:win               # Build Windows NSIS + portable
```

Testing should be done in a separate clone at `~/uclaw-dev/u-claw/`, not in this dev repo. This repo stays clean (no node_modules, no app/ runtime).

## Architecture

```
portable/           Bash/Batch scripts + HTML pages
                    Expects app/core/ (OpenClaw) and app/runtime/ (Node.js) at runtime
                    Config stored in data/.openclaw/openclaw.json (relative, on USB)

u-claw-app/         Electron app (main.js 419 lines)
                    Bundles Node.js in resources/runtime/node-{platform}-{arch}
                    Config stored in app.getPath('userData')/.openclaw/
                    Spawns openclaw.mjs gateway as child process

website/            Static HTML deployed to u-claw.org via Vercel
                    vercel.json sets outputDirectory: "website"

usb-scripts/        Extract U-Claw.tar.gz + launch (for USB distribution)
```

Both portable and desktop versions auto-find a free port in range 18789–18799 and start the OpenClaw gateway. On first run, they detect whether a model is configured — if not, they open Config.html; otherwise, they open the dashboard.

## Key Technical Details

- **Node.js discovery**: Portable looks at `app/runtime/node-mac-arm64/bin/node`; Electron looks at `resources/runtime/node-{platform}-{arch}` then falls back to system `node`
- **China mirrors**: All downloads use `npmmirror.com` — Node.js binaries from `npmmirror.com/mirrors/node`, npm packages from `registry.npmmirror.com`
- **Environment variables**: `OPENCLAW_HOME`, `OPENCLAW_STATE_DIR`, `OPENCLAW_CONFIG_PATH` control where OpenClaw reads config
- **macOS quarantine**: Mac scripts run `xattr -rd com.apple.quarantine` to remove Gatekeeper blocks
- **Config format**: `{"gateway":{"mode":"local","auth":{"token":"uclaw"}},"agent":{"model":"...","apiKey":"..."}}`
- **Config hot-reload**: OpenClaw watches `openclaw.json` and applies changes without restart

## What NOT to Commit

Never commit runtime dependencies or build artifacts. These are all in .gitignore:
- `portable/app/` and `portable/data/` (runtime + user data)
- `u-claw-app/node_modules/`, `u-claw-app/release/`, `u-claw-app/resources/runtime/`
- `*.dmg`, `*.exe`, `*.blockmap`

Release artifacts go to GitHub Releases, not the repo.

## Branding Rules

- Use only official `openclaw` (not `openclaw-cn` or any community fork)
- All npm installs reference `openclaw@latest` (official package)
- External links point to `u-claw.org` (our site) or `github.com/openclaw/openclaw` (upstream)
- No references to competitor products (Qclaw, AutoClaw) in any tracked files
- Skill marketplace links point to `skillhub.tencent.com` or `github.com/openclaw/clawhub`

## Platform Support Status

- Mac Apple Silicon (ARM64): ✅ Working
- Windows x64: 🚧 In development
- Mac Intel (x64): ❌ Not yet
- Linux: ❌ Not yet
