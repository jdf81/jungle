# Jungle Explorer — PWA

A 2.5D isometric procedurally-generated jungle, playable in any modern browser
and installable as a Progressive Web App on iOS, Android, and desktop.

## Files

| File             | What it is                                                     |
| ---------------- | -------------------------------------------------------------- |
| `index.html`     | The entire game — HTML + CSS + JS in one self-contained file.  |
| `manifest.json`  | PWA manifest (name, icons, theme, display mode).               |
| `sw.js`          | Service worker — caches everything for full offline play.      |
| `icon-192.png`   | App icon (home-screen, splash).                                |
| `icon-512.png`   | App icon (larger sizes, maskable).                             |

## Run it locally (zero install)

Just open `index.html` in a browser. The service worker won't register over
`file://` (browsers block it for security), but the game itself works fine.

For full PWA functionality (offline install, home-screen launch), serve it from
a local HTTP server:

```bash
# Python (any version 3.x)
python3 -m http.server 8000
# then visit http://localhost:8000
```

```bash
# Node
npx serve .
```

## Deploy as an installable PWA

Drop the files on any static host that serves over HTTPS. Free options:

- **GitHub Pages** — push to a repo, enable Pages in settings.
- **Netlify Drop** — drag the folder onto <https://app.netlify.com/drop>.
- **Vercel** — `vercel deploy` from inside the folder.
- **Cloudflare Pages** — drag-and-drop.

Once live, open the URL on your phone. iOS Safari → Share → "Add to Home
Screen". Chrome on Android → menu → "Install app". The app launches
fullscreen, runs offline, and shows on the home screen with its own icon.

## Controls

| Input                 | Action                            |
| --------------------- | --------------------------------- |
| On-screen D-pad       | Move (mobile)                     |
| `W A S D` / arrows    | Move (desktop)                    |
| `R`                   | New random world (desktop)        |
| ⟳ button (top-right)  | New random world (mobile)         |

Movement is iso-aligned: screen-up means walking NW into the world, screen-right
means walking NE, and so on.

## Sharing a world

The current seed lives in the URL hash. To share a specific world, copy the
URL — `…/index.html#1337` will always generate the same map. The reroll button
updates the hash so you can bookmark interesting worlds.

## How it works

- **Terrain**: hash-based value noise + fBm (5 octaves) for elevation, plus a
  second noise channel for moisture. Eight biomes: deep water, water, beach,
  swamp, clearing, jungle, highland, peaks.
- **Generation**: lazy — tiles are only computed when they enter the camera,
  then cached in a `Map` keyed by `(x, y)`.
- **Rendering**: HTML5 canvas, painter's algorithm sorted by `wx + wy`. Each
  tile is a diamond top plus two darker side faces sized to its elevation.
  Decorations (trees, bushes, ruins) draw above the tile top.
- **Camera**: locked to the player; smooth tween of both position and
  elevation when stepping between tiles of different heights.
- **Offline**: service worker uses cache-first, falls back to network for
  anything not yet cached.

## Tweaking it

Most knobs are at the top of the `<script>` block in `index.html`:

- `TILE_W` clamp range in `resize()` — bigger = chunkier tiles.
- `MOVE_SPEED` — walk speed (steps per second when held).
- The threshold ladder inside `World.generate` — shift the `h < …` bounds to
  shrink/grow biomes.
- Add new decorations to the `DECOS` array; biome rules in `generate` decide
  when they spawn.
