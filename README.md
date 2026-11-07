# Interactive History of Human Progress

An llm generated, interactive, offline map of human progress: 
- Shown as a radial web or a world map, driven by a timeline slider
- 270 milestones across 15 domains (food, health, shelter, energy, science, government, empires and more), from 12,000 BCE to today

![Demo: playing through time, hovering and pinning a milestone, shift+scroll rotating the web, then zooming into the map](demo.gif)

## Running

Simply open `index.html` in a browser. No server, build step or internet connection needed (except for clicking Wikipedia links).

## Using it

- **Slider / Play** — move through time. The slider is non-linear so recent centuries get more room. Colored bands mark eras.
- **Step ◀ ▶** — jump to the next batch of milestones (8, or one era-width), and finally to today. The header lists everything new since the last step.
- **Milestones over time** — bar chart under the slider (open by default), stacked by domain. Click a bar to jump there.
- **Web / Map** — radial tree of domains → groups → milestones, or dots placed by region.
- **Details: Hover / Window** — in Hover mode (default), hovering a node shows a tooltip card and the details window only opens when something is pinned. In Window mode the details window stays open and follows hover.
- **Click** a group to open or close it. **Shift + click** a group to pin its details. **Click** a milestone to pin it: unrelated nodes dim and the details window shows it. **Esc** unpins.
- **Drag** to pan, **scroll** to zoom, **Shift + scroll** to rotate the web. Map icons and labels keep their size as you zoom.
- **Details window** — starts at the left edge; drag its title bar to move it; **–** shrinks it to the title bar.
- **Search** (under the Step / Play buttons), **Nodes: Expand all / Collapse all / Reset view**, **Era: Auto-expand / Hide older / Width**.
- **Domains** — collapsible box at the top left of the graph. Unchecking a domain keeps it collapsed and hides it from the map and chart.

Shapes show the kind of milestone: circle = invention, diamond = discovery, square = institution / idea, triangle = event.

Dashed links on the selected node: amber = built on, blue = led to, green = explains (a discovery that finally explained something people had already used for a long time, e.g. combustion explaining fire).

## Caveats

- Dates for ancient milestones are approximate.
- The map shows regions, not historical borders.
- Wikipedia links go through search, so a slightly off title still lands on results.

## Credits

Land outlines: [Natural Earth](https://www.naturalearthdata.com/) (public domain).

## Files

- `index.html` — the viewer (layout, styles and all logic)
- `data.js` — `REGIONS` and `NODES`, the content
- `world.js` — `LAND`, Natural Earth 1:110m land outlines as one SVG path
- `demo.gif` — the demo above

## Data format

Each entry in `NODES`:

| Field | Meaning |
|---|---|
| `id` | Unique id |
| `parent` | Parent group id (`humanity` for domains) |
| `name` | Display name |
| `year` | Year, negative for BCE. Omit for groups (computed from children) |
| `desc` | Description, shown in tooltips and the panel |
| `from` | Ids of earlier milestones this built on |
| `explains` | Ids of earlier milestones this discovery explained |
| `where` | A key in `REGIONS` (optional; without it the milestone is not on the map) |
| `wiki` | Wikipedia article title |
| `kind` | `invention`, `discovery`, `society` or `event`; inherited from the parent if omitted |

`from` and `explains` must point to milestones with an earlier year. New domains also need a color in `COLORS` in `index.html`; its key order sets the order around the web.

## Re-recording demo.gif

Demo steps (~14 s, 1280×760 window):

1. Start at 12,000 BCE, web view, default zoom.
2. Click **Play**, let it run ~2 s, click **Pause**.
3. Click the slider just past the 1500 tick (lands around 1620).
4. Hover a recent milestone (e.g. Flush toilet) to show its tooltip, then click it to pin the details window.
5. Hold **Shift** and scroll over the web to rotate it.
6. Press **Esc**, switch to **Map**, scroll-zoom into Europe, hover a dot (e.g. Planetary orbits).

Recording notes (last done with Playwright + headless Chromium + ffmpeg):

- Install Playwright outside the repo (`npm i playwright && npx playwright install chromium` in a temp folder).
- Record with `recordVideo` at 1280×760. At 960 wide the header wraps onto extra rows.
- Headless video has no cursor. Inject a small SVG arrow via `addInitScript` that follows `pointermove`, and drive everything with `page.mouse` (`move` with `steps`, `down`/`up`, `wheel`) so it glides between targets.
- Pick hover targets from what's on screen: filter by the `#graph` rect and check `document.elementFromPoint` hits the node.
- The recorder only writes a frame on repaint, so the final hover can go missing. Wiggle the mouse 1 px a few times at the end.
- Convert (~2.2 MB): `ffmpeg -i in.webm -vf "fps=6,scale=800:-1:flags=lanczos,split[a][b];[a]palettegen=max_colors=128:stats_mode=diff[p];[b][p]paletteuse=dither=none:diff_mode=rectangle" demo.gif`. 64 colors washes out the domain colors.
