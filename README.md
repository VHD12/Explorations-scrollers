# Explorations — scrollers

A case-study listing built from ancient hanging scrolls. Each project is a 3D
scroll resting on a carved stool, in an ink-wash scene lit by two paper
lanterns. Click a scroll and it unties, lifts off the stool and unrolls from the
middle towards the top and bottom. The project is painted onto the parchment,
and its details appear beside it.

Prototype study. No build step and no dependencies to install. Three.js and the
fonts load from CDNs.

## Run it

The scroll model can't load from a `file://` page, so serve the folder over http:

```bash
python3 -m http.server 3002
```

Then open:

- `http://localhost:3002/scrolls.html` — the listing
- `http://localhost:3002/scroll-viewer.html` — a plain check of the model on its own

## Files

| Path | What it is |
|---|---|
| `scrolls.html` | The listing: five 3D scrolls, one on the stool at a time |
| `scroll-viewer.html` | Model check: open and close, scrub the clip, frame closed or open, show the content area |
| `scrolls-css.html` | The earlier version, drawn in CSS and SVG with a procedural dragon (no 3D model) |
| `models/scroll.glb` | The web asset: 3.8 MB, about 91k triangles, a 160-bone skinned sheet, one 3.5 s clip |
| `blender/scroll.blend` | The editable Blender source (9.3 MB, textures packed) |
| `images/` | One plate per project |

`blender/scroll.blend` was saved after `models/scroll.glb` was exported. If the
model was changed since, re-export it as GLB to `models/scroll.glb`.

## The model

- Y-up, in metres. The origin is the top roller's axis, and the sheet hangs down −Y.
- The front of the parchment faces +Z. The parchment is 1.0 wide and 2.4 long.
- One clip, `Animation`, runs from 0 to 3.5 s. The cord and seal binding lifts
  away from 0 to 0.5 s, then the sheet unrolls from 0.5 to 3.5 s. Played
  forward it opens; played backward it closes.
- **The rest pose is the scroll fully open.** Pin the clip to t=0 before the
  first render, or the scroll shows open with the binding floating near the
  bottom roller.
- `Parchment` has two materials, so Three.js loads it as a group of two skinned
  meshes. Turn off `frustumCulled` on both, because their bounds come from the
  flat rest pose.
- The clear area for content on `ParchmentFront` (a 2048×4096 texture) is x 276–1772, y 1135–3482.

## How the listing uses it

- **Loaded once, cloned five times** with `SkeletonUtils.clone`. Each clone gets
  its own copy of the materials, tinted per project:

  | Project | Silk (`ParchmentBack`) | Cord (`CordRed`) | Wax | Wood |
  |---|---|---|---|---|
  | LeaseHub | violet | purple | indigo | natural |
  | Youhonk | saffron | navy | rust | honey |
  | BSE Star MF | teal | deep teal | red | grey |
  | Nuvama | crimson | red | oxblood | rose |
  | GoDark | moss | green | dark green | charcoal |

- **One timeline per scroll** (`T` from 0 to 4.2 s). Each frame sets the clip's
  time directly from `T`, so closing is opening played backwards:
  1. untie on the stool;
  2. rise off it;
  3. unroll while the camera stays centred on the ring and the real bottom
     roller, so the sheet opens towards the top and the bottom of the screen
     at once;
  4. the reading panels appear.
- **The parchment is the page.** On opening, the project (glyph, name, plate,
  blurb, year and industry, a red seal, a faint dragon) is painted into a copy
  of the `ParchmentFront` texture, inside the content area. It rolls with the
  paper.
- **Changing scrolls** is a slow, level glide lasting 2.8 s. Both scrolls rise
  a little off the stool, slide across side by side, and the new one is set down.
  New input waits until it lands.
- The stool, floor, lanterns and hills are HTML and SVG, but they move with the
  3D camera, so the stool stays under the scroll and drops away as it rises.

## The loose parts

The hanger cord, the seal's thread and the tassels are simulated in the
browser rather than left in their modelled pose.

- Every cord is a Verlet chain drawn as a red-and-gold twisted pair, using the
  model's own cord materials. The modelled `HangerStrand_*`, `CordTail_*` and
  `CordEnd_*` meshes are hidden.
- **Hanger.** At rest, it falls forward over the top of both rollers and down
  the front, with the ring in front of the roll. Then it sleeps, perfectly
  still. Opening draws the ring up on a soft spring; closing carries it back
  down to the front before letting go.
- **Seal.** It hangs on its thread, turns with it, and sways slowly in a faint
  draught, with its two loose ends trailing. It shrinks away with the binding
  when the scroll opens.
- **Tassels.** Each one has two links, so the swing travels down the strands
  like a wave.
- The physics runs on its own fixed clock of 1/120 s steps. Uneven browser
  frame times otherwise make Verlet cords buzz and never settle.

## Size and performance

| Part of `scroll.glb` | Size |
|---|---|
| Geometry | 2.10 MB |
| Textures (5 JPEGs) | 1.05 MB |
| Skin weights and joints | 0.35 MB |
| Animation | 0.20 MB |
| **Total** | **3.83 MB** (about 1.9 MB with server gzip) |

Once loaded, it runs comfortably: only one or two scrolls are drawn at a time.
The real costs are the download, and GPU memory on phones: the 2048×4096
parchment takes about 43 MB once decoded. Before production, I'd do these:

- meshopt-compress the geometry;
- convert the parchment to KTX2 (UASTC);
- load the model after first paint, with a 2D placeholder;
- pause drawing when off-screen;
- serve it with a fingerprinted file name and long caching.

## Known rough edges

- Text painted on the parchment is small on phones; on desktop the side panels carry the same content.
- "Read the full scroll" is a placeholder link.
- The project images are compressed exports pulled from Figma.
