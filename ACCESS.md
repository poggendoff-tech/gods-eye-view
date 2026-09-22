# Access — God's Eye View on this fork

This checkout is the owner's fork of [bilawalsidhu/gods-eye-view](https://github.com/bilawalsidhu/gods-eye-view) (MIT): [poggendoff-tech/gods-eye-view](https://github.com/poggendoff-tech/gods-eye-view). Owner is Poggendoff (REISMS). Remy is coordinating.

Walkthrough video: [God's Eye View on YouTube](https://www.youtube.com/watch?v=o_FJ1NIH9yw).

Feature work stays on **this fork**. Do not open feature pull requests against upstream `bilawalsidhu/gods-eye-view`.

The app starts **without API keys**. Imagery is Esri World Imagery with keyless terrain. Optional keys are upgrades you paste inside the app. Do not put secrets in git.

## Path 1 — Pinokio (one click)

1. Install or update [Pinokio](https://desktop.pinokio.co/) to **8.2 or later** (Windows, macOS, or Linux).
2. Open [God's Eye View in Pinokio](https://pinokio.co/apps/github-com-bilawalsidhu-gods-eye-view).
3. Click **Install**, then **Start**.

The launcher installs the locked dependencies, picks a free local port, and opens the same app. Add keys later with **POWER UP** inside the app (it writes the ignored `pinokio/ENVIRONMENT` file). Do not type credentials into Pinokio 8.0.40's native Configure panel.

## Path 2 — Terminal steps that worked on this VM

Recorded **2026-09-22** on the cloud VM for this fork. The default `node` on `PATH` was **v22.14.0** (`/exec-daemon/node`), which the setup doctor rejects. The working runtime is **Node v24.15.0** from nvm. Put that binary ahead of `/exec-daemon` or `npm` and `vite` keep using Node 22.

```bash
export NVM_DIR="$HOME/.nvm"
. "$NVM_DIR/nvm.sh"
nvm install 24.15.0
nvm use 24.15.0
export PATH="$HOME/.nvm/versions/node/v24.15.0/bin:$PATH"
hash -r
node -v    # v24.15.0
npm -v     # 11.12.1

cd /workspace
npm ci
npm run doctor
npm run dev
```

`npm run doctor` (keyless, no `.env` present) reported:

- Node 24.15.0: supported LTS
- npm 11.12.1
- dependencies installed
- Map: Esri World Imagery (keyless satellite basemap) with keyless terrain
- Flights: OpenSky keyless anonymous access (rate-limited)
- Voice, vessels, and fires off until their keys are added
- Traffic: built-in simulation
- Missions: Launch Library 2 public access
- Every listed provider shown as not configured (`[--]`)

Vite then printed:

```text
VITE v6.4.3  ready in 228 ms
➜  Local:   http://localhost:4173/
```

Open **http://localhost:4173**. The first-run panel offers Live Contacts, Space Missions, Environmental, or Explore Manually.

The dev server binds to localhost only. On this VM it listened on **IPv6 `::1:4173`**. `curl http://localhost:4173/` worked because `localhost` resolved to `::1` first. `curl http://127.0.0.1:4173/` was refused. Use `http://localhost:4173/` or `http://[::1]:4173/`.

## Smoke test (keyless HTTP)

Run while `npm run dev` was up, **2026-09-22**, no `.env` and no `pinokio/ENVIRONMENT`:

```bash
curl -sS -D - -o /tmp/gev-index.html --max-time 20 http://localhost:4173/
```

| Check | Result |
| --- | --- |
| `GET http://localhost:4173/` | **HTTP/1.1 200 OK**, `Content-Type: text/html`, `Content-Length: 59308`, connected to `::1` |
| HTML title | `God's Eye View` (page heading `GOD'S EYE VIEW`) |
| `GET /src/main.js` | **200** `text/javascript`, 4005 bytes |
| `GET /logo.svg` | **200** `image/svg+xml`, 8434 bytes |
| `GET /@vite/client` | **200** `text/javascript` |
| `GET http://127.0.0.1:4173/` | connection refused (listener is `::1` only) |
| Credential markers in the HTML (`AIza`, `sk-`, `OPENAI_API_KEY`, `CESIUM_ION_TOKEN`, and the other key names) | none found |

That is a keyless document response from the Vite dev server, not a headed-browser click-through of the globe.

## Optional POWER UP keys

Keys are not required to run. When you want one, open the **POWER UP** chip (bottom-right), or `?setup=1` if the chip is hidden. Paste the value, then **SAVE KEYS**. The app writes a local, owner-only, gitignored file (`.env` for a terminal checkout, `pinokio/ENVIRONMENT` under Pinokio) and restarts. Never commit that file. `.env` and `.env.*` are gitignored; `.env.example` is the name-only template.

| Key | What it turns on | Where to get it |
| --- | --- | --- |
| Cesium ion | Google Photorealistic 3D, world terrain, and other ion imagery. Community plan is for eligible personal, non-commercial use and has quotas. Use a public `assets:read` token. | [cesium.com/ion](https://cesium.com/ion) · [pricing and eligibility](https://cesium.com/platform/cesium-ion/pricing/) |
| Google Maps | Direct Google Photorealistic 3D and in-app place search. Metered; restrict the key. | [Google Cloud Console](https://console.cloud.google.com/) · [Map Tiles API](https://developers.google.com/maps/documentation/tile) |
| OpenAI | Voice control and the AI HUD summary. Metered. | [platform.openai.com](https://platform.openai.com) |
| AISStream | Live global ships. | [aisstream.io](https://aisstream.io) |
| NASA FIRMS | Live active fires (trailing 24h). | [FIRMS map key](https://firms.modaps.eosdis.nasa.gov/api/map_key/) |
| TomTom | Live flow speeds and congestion colors on the simulated traffic layer. | [developer.tomtom.com](https://developer.tomtom.com) |

Browser-visible keys are **Google Maps** and **Cesium ion**. Restrict them at the provider. OpenAI, AISStream, FIRMS, and TomTom stay on the server. See [SECURITY.md](SECURITY.md) and the keys section of [README.md](README.md).

## Stop the dev server

In the terminal where `npm run dev` is running, press Ctrl+C. On this VM the process was left in a `tmux` session named `gev-dev`.
