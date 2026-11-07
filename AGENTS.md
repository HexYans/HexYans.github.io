# HexYans's Blog — Agent & Development Notes

## Purpose

This repository is a private personal blog built on the Homecraft template: a
Minecraft-style 3D site rendered with Three.js and native ES modules, with an
import map in `index.html` (`@world/`, `@config`, `@engine/`). It runs with no
build step in development and is deployed as a static site to GitHub Pages at
**https://blog.hexyans.com**.

## Important: this site customizes the engine

`engine/` is intentionally customized for this site (removed third-party HUD
credits, emptied the built-in default-world list, renamed the sound storage key,
removed the engine update guide, etc.). Keep these local changes. Do **not** run
Homecraft's official updater and do not overwrite `engine/` with an upstream
copy. `engine/manifest.json` is only a fingerprint baseline; `npm run check:site`
warns when `engine/` differs and that is expected here.

Portal friend links come from this site's own `worlds.json`, not from the
upstream public directory.

## Directory structure

```
index.html          Content cards: blog / about / works / contact / mailbox
home.config.js      Settings: name, skin, languages, weather, dock, mailbox, portal
homecraft.json      Public metadata: name, url, logo, intro, lang
worlds.json         Portal friend-link list (see "Portal")
manifest.webmanifest / CNAME / _headers   PWA manifest, custom domain, CORS headers
world/              The 3D world
  layout.js         Data: island bounds, spawn, stations, house, pool, portal, pets
  build.js          build({...}) places blocks/furniture; setup(app) adds behavior
posts/              Blog articles: one HTML file per post + style.css + _template.html
assets/             Icons, skin (me.png), og.jpg, apple-touch-icon.png
engine/             Shared engine — customized, do not overwrite
vendor/three/       three.js
tools/              serve.py (preview), build.sh (build), check-site.mjs / check.mjs
```

## Running, checks, deploy

```sh
python3 tools/serve.py        # local preview; picks a free port and opens the browser
python3 tools/serve.py --lan  # also reachable from a phone on the same Wi-Fi
npm run check:site            # metadata / skin / engine-diff check, no server needed
npm install && npm run check  # headless browser check (desktop + mobile)
bash tools/build.sh           # build minified site into dist/
```

Deployment: pushing to `main` triggers `.github/workflows/pages.yml`, which runs
`tools/build.sh` and publishes `dist/` to GitHub Pages. The build copies only
`index.html, _headers, CNAME, manifest.webmanifest, home.config.js,
homecraft.json, worlds.json, world, posts, assets, engine, vendor`.

- Append `?debug` to the URL for draw calls / triangles / frame time.
- Frame time is machine-specific; check the real experience on a phone.

## What to edit

- To change content or looks, modify `world/`, `index.html`, `home.config.js`,
  `homecraft.json`, `posts/`, `assets/`, `worlds.json`.
- Only touch `engine/` when explicitly asked, and preserve the existing local
  customizations.
- Keep one purpose per change. Write commit messages as `type(scope): summary`.

## Site identity and cards (index.html)

- Site name / logo / intro live in `homecraft.json`; the character skin is
  `config.skin` (`assets/me.png`, a 64×64 PNG, standard 4-pixel arms, ≤18 KB).
- Each `<article class="page" data-station="id">` is a card; `<h2>` is its title.
  Dock order comes from `dock` in `home.config.js` (ids of pages, plus `phone`).
- Bilingual: short labels use `data-en` / `data-zh`, long text uses
  `lang="zh-CN"` / `lang="en"` blocks.
- The `blog` card lists articles; each article is a standalone page under
  `posts/` opening in a new tab. Copy `posts/_template.html` to add one, then add
  a link in the blog card.
- `contact` (`ul.contact > li[data-app]`) feeds the phone UI; `mailbox` posts to
  `config.mailbox.email` by default (opens the visitor's mail app). There is no
  in-site mailbox backend in this repo.

## Portal (friend links)

The 3D portal shows destinations listed in `worlds.json`, an array of
`{ name, url, owner, intro, logo, homecraft, lang }`. `home.config.js` points the
portal at it: `portal: { directory: ['worlds.json'] }`. Keep `directory: []` to
show no destinations while keeping the portal itself. Non-Homecraft entries
(`homecraft: false`) open as a plain link.

## Debug APIs (browser console)

| Call | Purpose |
|---|---|
| `__home.stats()` | Draw stats |
| `__home.go('about')` | Navigate to a station |
| `__home.player.teleport(x, z)` | Move to coordinates |
| `__home.setTimeMode('night')` | Switch time of day |
| `__home.setWeatherMode('rain')` | Switch weather |
| `__home.rig.showcase([camX,Y,Z], [lookX,Y,Z])` | Lock camera for screenshots |

## Coordinates

One block = 1 meter; x east, z south, y up; ground is y = 0. `grid.fill` uses
half-open intervals (x0 ≤ x < x1). The character is 1.8 m tall and steps up
~0.6 m; larger height changes need stairs or flying.

## World API

### `world/layout.js`

| Export | Required | Description |
|---|---|---|
| `ISLAND` | ✓ | `{ x0, x1, z0, z1 }`: world bounds; pathfinding grid, map and blocks use it |
| `SPAWN` | ✓ | `{ x, z, heading }`: spawn point; heading 0 faces south, π faces north |
| `STATIONS` | ✓ | Array of stations (see below) |
| `MAILBOX` | | `{ x, z, yaw }`: mailbox prefab |
| `PORTAL` | | `{ x, z, yaw }`: portal position/facing (frameless blue vortex). Omit to hide the portal |
| `PETS` | | `{ cat: {…}, dog: {…} }`: `anchors`, optional `within(x,z)`, `home` |
| `HOUSE` | | `{ x0, x1, z0, z1, height }`: enterable house; walls/roof auto-hide inside |
| `POOL` | | `{ x0, x1, z0, z1, water, bed }`: swimmable pool; also `POOL_LADDER`, `POOL_LAMPS` |
| `floorAt(x, z)` | | Floor height at a position (stairs / upper floors) |
| `INTRO` | | `{ pos, portrait, look }`: opening camera shot |
| `LIGHT_VOLUME`, `SHELTER` | | Bounds for baked lighting and weather occlusion |

Station example: `{ id: 'about', name: '关于我', en: 'About me', at: [1.0, -0.3], look: [1.0, -6] }`.
`at` is where the player stands, `look` the point they face. `page: false` means
no card. `floor` sets the level for upper-floor stations.

### `world/build.js`

```js
export function build({ grid, layer, materials, stage, quality, scene }) {
  return { batches, planes, obstacles, hotspots, bakes, lights, surfaces, forest };
}
export function setup({ stage, world, nav, player, avatar, sound, rig, quality, config }) {
  return { update(dt) { /* per frame */ }, warmup(on) {} };  // optional
}
```

All return fields are optional. `batches` are BoxBatch groups; groups prefixed
`wall` keep glass visible when walls hide, the `roof` group hides indoors.
`bakes` holds baked light (`{ id, position, power, radius, direction }`); at most
2 real-time `lights`. `surfaces` supports `fireplace`, `scoreboard`, `tvScreen`.

### Building

```js
grid.set(x, y, z, 'stone');                       // one block
grid.fill(x0, y0, z0, x1, y1, z1, 'planks');      // region (half-open)
grid.set(x, y, z, null);                          // remove
grid.define('marble', { tex: { all: 'prada_marble' }, merge: true, map: '#3b5a4a' });
```

Built-in blocks: `grass sand dirt stone deepslate gravel concrete planks deck
pavers travertine log leaves hedge glass`.

Furniture uses `BoxBatch` from `@engine/world/blocks.js`:

```js
const B = new BoxBatch(layer);
B.box(cx, y, cz, w, h, d, { tex: 'oak' });            // y is the bottom face
B.box(cx, y, cz, w, h, d, { tex: 'paint', tint: 0x2a2d31 });
B.box(cx, y, cz, w, h, d, { tex: 'glass', glass: true });
B.at(x, 0, z, ry, () => { /* relative, rotated coordinates */ });
```

Textures are procedurally generated 16×16 images (e.g. `grass_top oak wood_panel
stainless marble glass leaves paint`). Terrain: `buildIsland(grid, ISLAND, { … })`.
Trees: `buildForest`. Grass/flowers: `buildTufts`. Signs/paintings:
`canvasTexture(w, h, (ctx,w,h) => …)` + `pixelText(ctx, text, x, y, scale, color)`.

The existing world files (`world/furnish.js`, `paint.js`, `plants.js`,
`house.js`, `mountain.js`, …) are the reference for reusable proportions and
materials — reuse them when adding furniture or decor.

## Performance budget

| Metric | Limit |
|---|---|
| Draw calls at common viewpoints | 60 |
| Triangles | 200,000 |
| Real-time lights | 2 (bake the rest) |
| Shadows | Sun only, static shadow map |
| Textures | Built-in 16×16 / small `canvasTexture`; photos go in cards |
| Post-processing | None |
| Initial load | < 500 KB compressed |
| Per frame | No allocations |

## Visual design

- Two to three primary materials per space (one wood, one metal, one muted accent).
- One main piece per area with breathing room.
- Real-world proportions: 1 block = 1 m; doors 2.2–2.8 m tall, ≥1.2 m wide;
  tabletops 0.75 m; seats 0.45 m; floor-to-ceiling ≥3 m; stair risers ~0.17 m.
- Put names/descriptions in cards. Write lights to `bakes`.
