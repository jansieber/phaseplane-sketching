# Phase Portrait Sketcher — specification, plan and implementation notes

This file records the requirements as given, the feasibility assessment, the decisions taken, and enough of the design to regenerate the tool from scratch, for example by another AI instance or developer. The current implementation is `index.html` (about 1,700 lines, one self-contained file). User-facing help is in `HELP.md`; `README.md` is a short overview for the GitHub landing page.

---

## 1. Original instructions (verbatim, in order received)

### Round 1 — initial request
> Create a planar phase portrait sketching tool runnable in the browser (based on javascript if possible).
>
> * It should be possible to draw trajectories (curves with arrows), if possible enforcing a no-crossing policy, drawable as Bezier curves.
> * Some standard elements: equilibria of sink, source and saddle type, where the saddle has 4 terminals for the stable and unstable separatrices.
> * periodic orbits (stable and unstable)
> * lines indicating outflowing or inflowing boundaries (to set trapping regions)
> * possibly some magnetism to make it easy to connect a trajectory to a sink/source/saddle or stable/unstable periodic orbit or inflow/outflow curve.
> * export to svg/pdf/png, import/export of native (2d vector graphics ) format
> * toggle to enforce structural stability, when this is off permit other (codimension one) elements such as transversally stable/unstable saddle-nodes, periodic orbits with onesided attraction/repulsion
>
> Make a plan initially to indicate what is easy to provide and what implies impractical programming or computational complexity.

### Round 2 — decisions after the feasibility plan
> The hard parts are not required and not even desirable.
> for pdf: use print-to-pdf
> tikz: yes, this is potentially useful
> crossing: Initially try "refuse". If this turns out impractical, I may have to reconsider.
> If possible, include a zoom feature and a select and scale feature.
> If possible, permit adding text labels.

### Round 3 — feedback after first tests
> * the export menu after dropping down is obscured by the side pane (eg Drawing)
> * the "refuse crossing" interferes with the "duplicate" feature when the trajectory is curved, so set to "off" by default.
> * set the default scale slightly smaller such that arrows are slightly larger.
> * can the "free" end of a trajectory snap to a "free" start of a trajectory to join them?
> * the export and save appear to save to "Downloads", but the "open" button asks for a folder. Could save and export also open a folder choice menu?
> * The periodic orbits are thicker than ordinary trajectories. Can the separatrices also be thicker?
> * A "simplify" button for hand-drawn trajectories that are quite wonky when done with the mouse.

### Round 4
> when the browser window is slightly to small to fit the left or top menu pane, is it possible to double stack them instead of having a scrollbar

### Round 5
> Create a git commit, add a Readme.md with the help and a separate plan.md that shows the instructions and any further summary that enables regeneration by (eg) another AI instance.

### Round 6
> Just tested on ipad (firefox): the "save as" does not open any folder, and the "open" only shows the "files" area, not giving access to the firefox downloads sandbox. Is there anyway I can load a file that I have saved?

Response: on iOS all browsers use WebKit, which has no File System Access API, and Firefox keeps its downloads in its own app area. Changes:
* Saves and exports on iOS offer the share sheet (*Save to Files* lets the user choose a folder) or a plain download.
* A browser-local *Library* of named drawings.
* The `accept` filter was removed from the file input, so iOS does not grey out `.phase.json` files.
* Drag-and-drop opening.

### Round 7
> Ok, also create similar fallbacks for android devices, update plan and readme and commit the updates.

Response: Android browsers have no File System Access API either. Downloads go to the shared Downloads folder, which the system file picker behind *Open…* can browse. The iOS save box is now used on Android too (`MOBILE = IOS || ANDROID`), with Android-specific wording. *Download* is the main route there; *Share…* (Drive, Files, e-mail) is shown only if `navigator.canShare({files})` is true.

### Round 8
> How does "Library" behave if index.html is loaded from different sources into different tabs?

Answer: localStorage is per origin (protocol + host + port), not per path.
* Tabs from the same site share the Library; different sites are isolated.
* `file://` copies are shared in Chrome; Firefox may isolate local files.
* Private windows get a temporary store; different browsers, profiles and devices never share.
* The single autosave slot and the preferences were shared per origin, so two tabs could overwrite each other's autosave.

### Round 9
> Yes, both features are useful. update allf files and git commit

The two features offered were library export/import and per-tab autosave.
* **Library export/import:** the Library modal gained *Export library…* and *Import…*.
* **Per-tab autosave:** see §3.8 and §3.10.

### Round 10
> print as pdf only exports hole pages, making it hard to import them into latex. is there a way to make this optional?

Response:
* A direct vector-PDF writer was added: *PDF file, cropped to drawing*. The page is the drawing's bounding box.
* Print now has two variants: page = drawing (`@page{size:W H; margin:0}`) and full page.

### Round 11
> A bug with the export dialog: if the manu is folded into two rows the export dialog can be on the very left of the screen or window (eg, on ipad). When the dropdown menu opens it is outside of the window and the text is invisible.

Fix: `placeMenu()` measures the opened dropdown and positions it so that it stays inside the window.

### Round 12
> As exporting and importing files is tricky on iOS or Android tablets: is it possible to copy a rectangle to clipboard as an image that can be inserted elsewhere (eg, note-taking applications)? The copy&paste between instances or sketches would be native format, though. Is there a simple user interface that enables both? Offer options first.

Options offered:
* What goes on the clipboard:
  * A: one Copy with both PNG and JSON;
  * B: separate *Copy* and *Copy as image*;
  * C: A plus a setting.
* How the area is chosen:
  * 1: the current selection;
  * 2: a Region tool with a floating bar;
  * 3: a persistent export frame.

Recommendation **B + 1 + 2**, with PNG 3× on white.

### Round 13
> Yes, the recommendation makes sense. Update the file, the plan/readme and commit.

Implemented as B + 1 + 2 (§3.11).

### Round 14
> One further feature request: to demonstrate separatrix splitting, it is useful to also split trajectories at a node. After selecting a node, one should be able to split the trajectory there.

Implemented as `splitAtNode()`: key `X`, plus a panel button shown when an inner node is selected (see §3.8).

### Round 15
> I plan to push the code to github. For this the long README.md is too long for the landing page intro. So, move README.md to (eg) HELP.md, and provide a short summary in README, with a github compatible link to the github pages link of the html. Is it still ok/suitable to call the main file index.html?.

Changes:
* The full help moved to `HELP.md` (via `git mv`, so its history is kept).
* `README.md` is now a short overview: a link to the GitHub Pages site, a screenshot (`fish_phase.png`), a feature list, links to HELP, PLAN and the examples, and Pages setup instructions.
* `index.html` is kept as the file name: GitHub Pages serves it at the site root.

### Round 16
> When exporting a figure as tikz tex and compiling I get an error ERROR: Dimension too large. […] l.14 ...764) and (-7.577,1.682) .. (-7.207,1.638);

Cause: a numerical weakness of PGF's `decorations.markings`. For some curves, certain mark positions overflow. In the reported case positions 0.5, 0.55 and 0.8 failed, even with a plain dot as the mark, while each Bézier segment alone was fine.

Fix: the TikZ export no longer uses decorations. Arrowheads are written as `\fill` polygons computed by `arrowPoly`, the same shape and position as on screen, so the export needs no TikZ libraries.

### Round 17
> The attached folder contains code and documentation (in PLAN.md) for a browser based phase-plane sketching tool, created by claude. Extensive usage has shown the need for a few additional features:
>
> * periodic orbits (when stable) are hard to distinguish even in moderately complex phase portraits because they have the same thickness as saddle separatrices. Is there an easy interface to assign individual thickness and/or colors
> * when using the pen on an ipad with the bezier tool it is difficult to end the curve
> * some background filling tool to highlight basins of attractions (possibly pure decorations as consistency checks may be hard).
>
> Could these changes be made without breaking importability of existing figures?

Response (details in §3.12):
* **Colours and widths:** optional per-element fields `color` and `width`, edited in a *Colour* / *Line width* section of the side panel. It works on multiple selections, so for example all stable orbits can be made heavy at once.
* **Pen on iPad:** a floating bar (✓ Finish, ↶ Node, ✕), tap the last node again to finish, automatic finish when a node lands on a snap target, larger double-tap tolerance and handle threshold for pen and touch input, and switching tool now finishes the curve instead of discarding it.
* **Background fill** (tool `F`): a new decorative element type `fill`. A tap does a raster flood fill bounded by "basin boundaries" (stable separatrices, unstable and semi-stable orbits, boundaries) or by all curves, and the outline is traced and fitted into Bézier rings with holes. A drag draws a freehand shape instead. Fills are never checked.
* **Compatibility:** all new fields are optional and the format version stays 1. Existing files export byte-identical SVG and TikZ (checked for `fish.phase.json`, all items of `phaseplane-examples.json` and the built-in example). Older copies of the tool ignore the new fields; they draw nothing for a `fill` element.

### Round 18
> Line coloring/thickness works as expected.
> Could the thickness scale be based on a "base thickness", where the other thicknesses are the multiples currently used?
> A line thickness option should also be available for the transversal cross sections.
> The default coloring for for transversals should change from green to another color as dark green and dark/matte red are natural options for stable/unstable periodic orbits (leave default black for for everything except transversals). The dark orange from the palette is suitable.
> The coloring of saddles should only affect the circle in the center, not the arrows.

Changes:
* **Base thickness:** optional document field `lineBase` (default 1.3), set with a slider. Every stroke is the nominal width (the Round 17 value at base 1.3) × `lineBase/1.3`. This covers glyph outlines, spirals, stubs and marks. Per-element widths are stored as multiples `lw` (presets ×0.6, ×1, ×1.5, ×2, ×3, ×4.5, ×6). Defaults are trajectory ×1, separatrix ×1.54, orbit ×1.62, transversal ×1.15. The Round 17 field `width` (absolute px at base 1.3) is still read.
* **Transversals:** they already had *Line width*, but the crossing marks stayed thin. Mark length, stroke and arrowheads now scale with `arrowK`.
* **Default transversal colour** is the palette's dark orange `#c26b00` (also the tool icon and TikZ `ppbnd`). The palette red became a matte dark red `#a8322d`; dark green `#1a7f5a` stays.
* **Saddle colour:** the stubs and stub arrows of saddles and saddle-nodes are always ink. Only the centre glyph takes `color`.
* **Compatibility:** apart from the transversal colour, old files (and Round 17 files with absolute `width`) export identically.

### Round 19
> Is the matte dark red distinguishable from from the dark green for a color blind person? Choose a compromise between "good-looking" (that is, not too bright on the screen) and ok for color blind.

Answer: only barely.
* Method: simulated protanopia, deuteranopia and tritanopia (Machado et al. 2009 matrices, severity 1, in linear RGB), then took the CIEDE2000 difference.
* `#a8322d` vs `#1a7f5a` gave ΔE ≈ 15 (protan) and 13 (deutan): two dull browns of similar lightness, which is weak for thin lines.
* New palette `LINE_COLORS`: dark red `#9b1c1c`, blue `#1f6feb`, teal green `#0b8f87`, orange `#c26b00`, plum `#9c3587`, dark grey `#4b5058`.
  * The red is darker (L* 34) and the green lighter and bluish (L* 53), giving ΔE ≈ 31 / 32 for protan / deutan and 61 normally. The teal still has 4:1 contrast on white.
  * Purple and grey were also changed: the old purple `#8250df` was indistinguishable from blue for deutans (ΔE 2.4), and the old grey from the new teal (4.4).
  * Every pair in the palette now stays ≥ 10.9 under all three deficiencies. The weakest pair is blue vs teal under tritanopia.
* Colours already stored in files are unchanged; they simply show as a custom colour in the panel.

### Round 20
> The basin filling interface behaves counterintuitively (to me). I prefer having the selected color as "currently active for clicks that follow" such that only clicks into regions change to the currently active color. At the moment, clicking on another color in the tool changes the last selected basin. The "delete" should also behave like a "currently active color for following clicks" (aka "no color").
> The option "bounded by" basin boundary is too smart (behaves confusingly when connecting loops are present). Bounded by all curves feels simpler, even though one has to perform several clicks for one basin.
> Commit change to git at the end

Changes:
* The fill tool panel shows only an **active colour** row. It has the seven presets, a colour picker and an eraser swatch *no colour* (`st.fillColor = 'none'`). Choosing a colour only sets `st.fillColor` and never touches existing fills.
* Taps behave as follows:
  * on empty space, a new fill in the active colour;
  * on a fill, recolour it (or nothing if it already has that colour);
  * with *no colour*, remove the fill under the tap. A freehand drag with *no colour* is ignored.
* The fill tool no longer selects the fills it creates, and entering the tool clears the selection. Fills are edited further with the Select tool.
* The "bounded by" option and the *basin boundaries* mode are removed: `fillBarriers()` always uses every trajectory, cycle, boundary and stub, with trajectory ends on a cycle still extended onto it. The `fillBound` preference is gone.

### Round 21
> A further modification of the saddle-node: currently it has the options: transversally stable/unstable. Could another option be "tbd" (possibly different name), which  leaves out the transversal arrows initially? This would permit the drawing of 1d phase portraits (just along horizontal lines). Once, one connects another transversal trajectory, the transversal properties are determined for a 2d phase portrait.
> Commit to git if this is practical and implemented.

Implemented as `trans: 'open'` (panel: "open (1-d, decided later)").
* `eqPorts`: an open saddle-node has the centre stubs `cm` (angle, role *start*) and `cn` (angle + 180°, role *end*). It also has `h1`/`h2` marked `hidden:true` and no node sector `c`. `portRole` gives the hidden ports role `'any'` (single, separatrix), and `compat` accepts `'any'` for either role.
* Hidden ports are skipped for drawing, hit areas, crossing checks, fill barriers and TikZ, but `findSnap` and `resolveTraj` still see them.
* The glyph is filled on the `cn` side (the same as *stable*).
* `settleSN()`, run at the start of `resolveAll()` (so before every validation and commit), converts an open saddle-node as soon as a trajectory is bound to a hidden port: bound by its end → `convertSN(e,'stable')`; by its start → `'unstable'`.
* `convertSN(e,to)` remaps attached bindings:
  * open→stable: `cn→c`;
  * open→unstable: angle += 180°, `cm→c`, `cn→cm`, `h1↔h2`, so that the picture (glyph, centre-line flow, transversal positions) is unchanged;
  * stable→open: `c→cn`;
  * unstable→open: angle −= 180°, `c→cm`, `cm→cn`.
  * Going to *open* is refused while `h1`/`h2` are bound or when more than one trajectory ends in the node sector. stable↔unstable keeps the old behaviour.
* Panel changes of `trans` go through `convertSN`. The last choice is stored as `st.snTrans` (preferences) and used by `mkEq` for new saddle-nodes.
* Compatibility: existing stable and unstable saddle-nodes and all other files export identically (checked against the Round 20 version).

---

## 2. Feasibility assessment (agreed scope)

| Feature | Difficulty | Decision |
|---|---|---|
| Trajectories as piecewise cubic Béziers with editable handles; freehand → Bézier fit; Bézier pen | Easy | Done |
| Arrows on trajectories (by arc length) | Easy | Done |
| Sink / source (node or focus glyph), saddle with 4 terminals | Easy | Done |
| Periodic orbits (stable / unstable / orientation) | Easy | Done |
| Inflow / outflow boundaries, open or closed (trapping region) | Easy | Done |
| Magnetism (ports, orbits, boundaries, other trajectories' free ends) | Easy–moderate | Done |
| Direction rules on attach (into sink, out of source, stable separatrix into saddle, …) | Easy | Done |
| No-crossing **detection** (polyline intersection with bounding-box chunks) | Moderate | Done |
| No-crossing **refusal** (undo an edit that adds a new violation) | Moderate | Done; optional, **off by default** |
| Automatic re-routing to avoid crossings | Hard | Not wanted |
| Boundary crossing direction check | Easy | Done |
| Structural-stability toggle; codim-1 elements (saddle-node, semi-stable orbit, saddle connection / homoclinic loop) | Easy–moderate | Done |
| Index checks (sum = 1 inside an orbit / closed transversal) | Easy | Done (warnings) |
| Export SVG / PNG / TikZ, PDF via print | Easy | Done |
| Global realisability (does a C¹ vector field exist for the picture?) | Research-level | Not wanted |
| Generating and integrating a vector field from the sketch | Hard, fragile | Not wanted |
| Per-element colour and line width | Easy | Done (Round 17) |
| Background fills for basins: raster flood fill bounded by basin boundaries, traced into Bézier rings | Moderate | Done; decorative only, not kept in sync with later edits |
| Non-planar phase spaces (sphere, torus, Poincaré disc) | Out of scope | — |
| Hopf / weak focus | — | Omitted: topologically identical to a focus |

---

## 3. Architecture

* **One file**, `index.html`, with inline CSS and plain JavaScript (`'use strict'`). There are no libraries and no build step. ES modules are avoided because Chrome blocks them on `file://`.
* **Layout:** CSS grid `#app`:
  * areas: `top` (toolbar), `tools` (left tool palette), `stage` (drawing SVG), `side` (properties + consistency panels), `status` (bottom status bar);
  * rows `auto minmax(0,1fr) 26px`; columns `auto 1fr 290px`.
  * The top bar uses `flex-wrap: wrap`, with related controls grouped in `.grp` spans so they wrap together.
  * The tool palette gets class `.two` (a 2-column grid) when its `scrollHeight` exceeds its `clientHeight`. This is checked in `layoutTools()` at startup and on window resize.
  * The Export dropdown uses `position: fixed`, so the side pane cannot cover it. `placeMenu()` positions it after opening, and again on resize:
    * by default its right edge is aligned with the button's right edge;
    * if that would cross the left window edge, its left edge is aligned with the button's left edge instead;
    * finally it is clamped to [4, innerWidth − width − 4] horizontally, and its max-height is limited to the space below the button (scrolls if needed).
* **Main SVG** `#svg` has three layers:
  * `#gridLayer`: screen coordinates, not exported.
  * `#world > #scene`: world coordinates, with a `translate(-x·k, -y·k) scale(k)` transform.
  * `#overlay`: screen coordinates. Holds selection handles, the marquee, magnet rings and violation markers. It is click-through (`pointer-events: none`) except elements with `[data-h]`.
* **Rendering:** each frame the scene is rebuilt from the model as an SVG string (`renderScene(exp)`; `exp=true` gives export-clean markup with no hit areas or highlights). Updates are throttled with `requestAnimationFrame` (`schedule()`). `refresh()` runs `validate()`, then renders scene, overlay, grid, problems panel and status, and the properties panel when `st.panelDirty` is set.
* **Coordinates:** the world uses SVG convention (y down). Angles stored on equilibria are **degrees counter-clockwise as seen on screen**: `eqDir(deg) = (cos, −sin)`. The status bar shows y negated (math convention). TikZ export maps `X = x/50`, `Y = −y/50` (50 px = 1 cm).

### 3.1 Data model (native `*.phase.json`)

```json
{ "format":"phaseplane-sketch", "version":1, "structStable":true, "arrowScale":1.3, "nextId":N,
  "elements":[ ... ] }
```

Path node (shared by trajectory, cycle and boundary). Handles are offsets relative to the node:

```json
{ "x":0, "y":0, "ix":-10, "iy":0, "ox":10, "oy":0, "smooth":true }
```

Element types:

| type | fields |
|---|---|
| `eq` | `id, kind ∈ {sink, source, saddle, sn}, x, y, angle (deg), spread (deg, saddle: angle from unstable to stable dir, default 90), len (separatrix stub length, default 30), spiral ∈ {none, cw, ccw} (sink/source), trans ∈ {stable, unstable, open} (sn; open since Round 21)` |
| `traj` | `id, nodes[], start, end (binding or null), arrows (0–8, default 1), arrowPos (0–1, used when arrows = 1), weight ∈ {auto, normal, thick}` |
| `cycle` | `id, nodes[], closed:true, stability ∈ {stable, unstable, semi-inner, semi-outer}, dir (±1, flow vs parametrisation), arrows (default 2)` |
| `boundary` | `id, nodes[], closed (bool), dir (±1), spacing (crossing-arrow spacing, default 36)` |
| `label` | `id, x, y, text, size (default 16)` |
| `fill` | `id, nodes[] (outer ring), closed:true, holes (optional array of node arrays), color` |

Optional on every element (Round 17): `color` (hex string; missing = default ink `#111`, boundaries `#c26b00` since Round 18; for saddles and saddle-nodes only the centre glyph). Optional on `traj`, `cycle` and `boundary`: `lw`, the width as a multiple of the base (Round 18). The older `width` (absolute px at base 1.3, Round 17) is still read when `lw` is missing, then the legacy trajectory `weight`. Setting a width from the panel writes `lw` and deletes `width` and `weight`.

Optional document field (Round 18): `lineBase`, the base line width in px (default 1.3). All strokes are multiplied by `LB() = lineBase/1.3`.

`semi-inner` = attracting from inside and repelling outside; `semi-outer` is the reverse.

Bindings (trajectory ends):

* `{el, port}` for an equilibrium terminal;
* `{el, t, side}` for a cycle, where `t` is the global path parameter (segment index + local u) and `side` is +1 inside or −1 outside;
* `{el, t}` for a boundary.

### 3.2 Equilibrium ports and roles

`eqPorts(e)` returns `{port, pos, dir}`, where `dir` is the outward unit vector for stub ports and `null` for the centre.

* **Sink:** `c`, role *end*, multiple trajectories allowed.
* **Source:** `c`, role *start*, multiple.
* **Saddle:** `u1` (angle), `u2` (angle+180) with role *start*; `s1` (angle+spread), `s2` (+180) with role *end*. Single use each; all are separatrices.
* **Saddle-node:** `cm` (angle; centre-manifold separatrix on the saddle side), `h1`/`h2` (angle ± 90), and `c` (node sector, on the side −angle).
  * Transversally stable: `cm` = start, `h` = end, `c` = end.
  * Transversally unstable: all roles reversed.
  * `c` accepts trajectories only from the node half-plane (checked using a point about 15 units back along the trajectory).
* **Index:** saddle −1, saddle-node 0, others +1.
* **Cycle role:**
  * stable → end; unstable → start;
  * `semi-inner`: inside → end, outside → start;
  * `semi-outer`: the opposite.

### 3.3 Binding resolution (`resolveAll`, run before every validation)

* **Equilibrium port:** the endpoint node is moved to the port position. For stub ports, the end handle is aligned with `dir` (outward), keeping its length (at least 8).
* **Cycle:** the endpoint is placed at `C(t) + side·2.5·n_in` (just inside or outside the orbit). The end handle is aligned with the flow tangent `dir·C'(t)`, with length clamped to 8–18. Start: out-handle = +T·L; end: in-handle = −T·L. This gives the "spiralling onto the orbit" look.
* **Boundary:** the endpoint is placed at `B(t)`.
* Bindings to deleted elements or missing ports are dropped.

### 3.4 Geometry utilities

* Cubic evaluation, derivative and de Casteljau split.
* `flatten(el)`: polyline with per-point `t` and arc length `s`, 6–80 samples per segment, cached in a WeakMap keyed by a signature of the nodes.
* `atArc` (point and tangent at an arc-length fraction); nearest point on a polyline; ray-casting point-in-polygon.
* **RDP** simplification and arc-length `resample`.
* **`crNodes(pts, closed)`:** chord-scaled Catmull–Rom → Bézier nodes. Interior tangent `m = (next−prev)/(d_in+d_out)`; handles `m·d_out/3` and `−m·d_in/3`; reflected end tangent for open ends.
* **`ellipseNodes`:** 8-node exact Bézier ellipse, handle factor `4/3·tan(π/2n)`.
* **`smoothPoly(pts, passes, closed)`:** repeated [¼ ½ ¼] binomial smoothing with the ends fixed.
* **Intersection:** polylines are split into chunks of 8 segments with bounding boxes; segment–segment tests use half-open parameter ranges. Self-intersection skips adjacent segments and the closing pair.

### 3.5 Turning strokes into curves

* **Trajectory or open boundary:**
  1. Smooth the raw points (`passes` 1/4/10 for low/medium/high).
  2. Run RDP with `eps` 2/4/7 px ÷ zoom.
  3. Drop points closer together than 8/16/28 px.
  4. Run `crNodes`.
* **Closed boundary:** detected when start and end are within 25 px and the stroke is longer than 80 px. The stroke is resampled to n = clamp(perimeter/45, 6, 16) points and fitted with closed `crNodes`. `dir` is chosen so that the crossing arrows point inwards (inflow).
* **Cycle:** same closed fit; a click without a drag gives a default ellipse of 60×42.
* **Pen:** clicks give nodes; a drag sets symmetric handles. Nodes without handles get automatic 1/3-chord handles.
* **Simplify:** target node count m = max(min, min(n−1, round(0.7n))).
  1. Resample to 200 points.
  2. Smooth with ≈ (200/m)²/6 + 2 passes, which makes the smoothing width about one node spacing.
  3. Resample to m points.
  4. Restore the exact endpoints (open curves) and run `crNodes`.

  This reduces gradually (19 → 13 → 9 → 6 → …); plain RDP collapses a wobbly line straight to 2 nodes. Bindings to the curve are re-projected (`rebindTo`).

### 3.6 Magnet (`findSnap(p, tr, which)`)

* Radius: 14 screen px (plus the glyph radius for `c` ports).
* Candidates:
  * equilibrium ports, with a priority bonus of −4;
  * nearest point on cycles and boundaries;
  * joins: for `which = end`, other trajectories with a free start; for `which = start`, those with a free end (bonus −4).
* Candidates are sorted by distance. The first one accepted wins:
  * a join;
  * a target compatible in the current role;
  * a target compatible in the opposite role, provided the other end's binding stays valid → the trajectory is reversed.
* Otherwise the result is `{fail, why}`, shown as a red ring and a status message.
* `compat()` also rejects:
  * a separatrix terminal already used by another trajectory;
  * a saddle-node node sector approached from the wrong side;
  * in structurally stable mode, a saddle connection (both ends on separatrix ports).
* **Join** (`joinTraj`): concatenate the nodes, merge the junction node (in-handle from the first trajectory, out-handle from the second, aligned to be smooth), keep the outer bindings, take the larger arrow count, and delete the second trajectory.

### 3.7 Validation (`validate()` → `{hard[], warn[]}`)

**Curves checked:** trajectories, cycles and boundaries (flattened), plus saddle and saddle-node stubs as straight segments oriented in the flow direction.

**Pair rules:**

| Pair | Rule |
|---|---|
| boundary × boundary | Allowed. |
| boundary × (trajectory or stub) | Direction check: `dot(flow tangent, dir·perp(boundary tangent)) > 0`, else `bnddir`. |
| any other pair (including cycle × boundary) | Crossing forbidden (`cross`). Stubs of the same equilibrium are skipped. |
| self-crossing of a trajectory or cycle | `self`. |

**Ignored intersections:**

* Near a shared attachment point: both curves touch the same equilibrium at the same point and the intersection is within 7 units. This covers several trajectories into one sink, and a trajectory on a terminal against its stub.
* Near a trajectory end bound to the other curve: within 9 units for cycles, 4 for boundaries.

**Other hard problems:**

* `through`: an unattached curve passes within R_EQ + 0.5 of an equilibrium.
* `role`: a binding is incompatible (`compat` ignoring use and structural-stability rules).
* `dup`: two trajectories share a single-use terminal.
* `bnddir`: an end bound to a boundary points against the crossing direction.

**Keys:** `type:minId:maxId`, merged per pair, with up to 12 marker points.

**Warnings:**

* Index sum ≠ 1 inside each cycle and each closed boundary.
* In structurally stable mode: saddle-nodes, semi-stable cycles and saddle connections present.

**Refusal** (`commit(before, label, {force})`):

* The new state is validated. If *Refuse crossings* is on and some hard key is not in `st.hardKeys` (the keys of the last accepted state), the edit is reverted to the `before` snapshot.
* A ghost of the refused state and red flashes are shown for 1.6 s, with the reason in the status bar.
* `force` (used by Duplicate) bypasses refusal.
* Existing problems never block edits; only new ones do.

### 3.8 Interaction

* **Pointer events on the SVG**, with pointer capture.
* **Double-click:** detected manually as two *clicks* (pointer moved < 6 px) within 380 ms at the same spot, because re-rendering the scene between clicks suppresses the native `dblclick`.
* **Drag types:** `pan, move, scale, rotate, eqrot, eqspread, node, hin, hout, marquee, stroke, penhandle`.
  * Move, scale and rotate restore the document from a `structuredClone` of the base and apply an affine map each frame (`affine(el, M)`). This maps nodes and handles, transforms equilibrium angles and spread, flips boundary `dir` and spiral sense when `det < 0`, and leaves glyph sizes unchanged.
  * Scale handles: corners are uniform (`Shift` free, `Alt` about the centre); edges scale one axis; a rotate knob sits above the box.
* **Splitting** (`splitAtNode`, key `X`):
  * Applies to the selected inner node `i` (0 < i < n−1) of a trajectory or open boundary.
  * Element A keeps `nodes[0..i]` and its start binding; a clone B gets a new id, `nodes[i..]` and the end binding.
  * At the cut, A's out-handle and B's in-handle are zeroed and both cut nodes become corners. The trajectories' cut ends are free, and `arrowPos` is reset to 0.5.
  * For a boundary, trajectory bindings with `t ≥ i` move to B with `t − i`.
  * B is inserted after A and selected, with its node 0 selected. The edit is committed with `{force:true}`.
  * Validation ignores intersections within 4 units of coinciding trajectory endpoints, so freshly split pieces do not flag each other.
  * Clicking a node re-renders the panel so that the split button appears.
  * The inverse is the existing join magnet.
* **Undo/redo:** JSON snapshot stacks (up to 300 entries).
* **Autosave, per tab:**
  * Each tab has an id `TAB` stored in `sessionStorage['phaseplane-tab']`, which survives reloads. The slot is `localStorage['phaseplane-autosave:'+TAB]` = `{date, doc}`.
  * On startup, a tab that already has an id restores its own slot. A fresh tab restores the newest slot of any tab, falling back to the legacy key `phaseplane-autosave`, then writes its own slot from then on.
  * Duplicated tabs inherit `sessionStorage`. To detect this, each page posts `{q:TAB, from:INST}` on `BroadcastChannel('phaseplane-tabs')`; a live tab owning that id replies `{taken, to}`, and the newcomer takes a fresh id.
  * Only the 12 newest slots are kept.
* **Preferences:** `localStorage['phaseplane-prefs2']` stores grid, snap, magnet, refuse and smoothing.
* **Keyboard shortcuts** are ignored while focus is in a text, number or select input (checkboxes blur themselves on change so shortcuts keep working).

### 3.9 Rendering constants

| Item | Value |
|---|---|
| Ink | `#111` |
| Boundary colour | `#c26b00` (dark orange; was `#1a7f5a` before Round 18) |
| Selection colour | `#2f81f7` |
| Trajectory width | 1.3 (all widths × `lineBase`/1.3, Round 18) |
| Separatrix width | 2.0 (trajectory touching a separatrix port, or `weight = thick`; stubs always) |
| Cycle width | 2.1 (unstable dashed 6/4) |
| Semi-stable cycle | Extra dashed (3/3) offset curve, 4 units on the repelling side |
| Equilibrium radius | `R_EQ = 4.5` |
| Saddle centre | Dot of radius 2.6 |
| Saddle-node glyph | White disc of radius 5 with a filled half on the node side (stable) or saddle side (unstable) |
| Arrowheads | Notched triangle 8 × 6.2 (cycle 10 × 7.5, stub 7.5 × 6), all × `doc.arrowScale` (default 1.3) |
| Boundary marks | A short line across the curve with a small arrowhead in the crossing direction, every `spacing` units |
| Focus spiral | Radius 17 → 5 (sink) or 5 → 17 (source), 1.3 turns, arrow in the middle |
| Labels | Times serif. Mini-TeX inside `$…$`: italic letters, `_`/`^` with `{…}` groups as sub/superscript tspans (70% size, ±0.32 em), Greek and a few symbols via a lookup table |
| Default zoom | 0.8; Fit caps at 4 |

### 3.10 Export

* **SVG:** `renderScene(true)` placed in an off-screen SVG to measure `getBBox`, plus a 16 px margin and a white background rect.
* **PNG:** the SVG drawn into a canvas at 3×, then `toBlob`.
* **Print:** the export SVG is placed in `#print`; `@media print` hides the app; then `window.print()`. For the "page = drawing" variant, a `<style id=printPageStyle>` sets `@page{size:Wpx Hpx;margin:0}` and fixes the SVG to W×H px. Chrome, Edge and Firefox honour this.
* **Direct PDF** (`makePDF`):
  * It parses the export SVG (`DOMParser`). Supported elements: `path` (M/L/C/Z only), `circle` (4 cubics), `rect`, `text`/`tspan`. The saddle-node half disc is therefore drawn with cubics (`halfDiscD`), not an SVG arc.
  * Coordinates: `cm` = scale 28.3465/50 pt per px (50 px = 1 cm, as in TikZ), with y flipped. MediaBox = export bounding box.
  * Styles: fill/stroke colours, width, round caps and joins, dashes.
  * Text: built-in Type1 fonts Times-Roman, Times-Italic (WinAnsi) and Symbol for Greek and maths symbols (`SYMBOLMAP`). Each text run gets `Tm 1 0 0 -1` to undo the flip. Run widths are measured with canvas `measureText` in Times New Roman so that centred labels stay centred. Baseline = y + 0.33·size (SVG `dominant-baseline: central`).
  * File structure: hand-written objects (catalog, pages, page, uncompressed content stream, 3 fonts, info) and an xref table. Bytes are written via a `Uint8Array`.
  * Checked with qpdf, pdfinfo, pdftoppm, and `\includegraphics` in pdflatex.
* **TikZ:**
  * Styles: `traj`, `sep`, `cyc`, `bnd` (line widths converted at 50 px/cm).
  * Béziers become `.. controls .. and ..`; closed paths end in `.. cycle`.
  * Arrowheads are explicit filled polygons, `\fill (tip) -- (a1) -- (notch) -- (a2) -- cycle;`, from `arrowPoly(p, t, L, W)`. Positions are the same arc-length points as on screen; boundary marks are a line plus a polygon. This avoids `decorations.markings`, which gives "Dimension too large" on some curves, so only `\usepackage{tikz}` is needed.
  * Spirals and semi-stable offsets are polylines (`--`). `plot[smooth]` caused "Dimension too large" with dense points.
  * Labels: `\node[font=\fontsize{..}{..}\selectfont]` with the raw text.
  * Tested with pdflatex.
* **File dialogs:** `showSaveFilePicker` / `showOpenFilePicker` (Chrome and Edge; works on `file://`). The handle is kept, so later Saves overwrite the same file. Other browsers fall back to an `<a download>` link (Downloads folder) and an `<input type=file>`. For PNG, the picker is opened **before** rendering, so the user-activation requirement is still met.
* **iOS and Android** (`IOS`: user agent, or `MacIntel` with touch points > 1; `ANDROID`: `/Android/` in the user agent; `MOBILE = IOS || ANDROID`): when no save picker exists, `writeTarget` opens `#shareModal`. Its hint text `SHARE_HINT` depends on the platform. Its *Share…* button calls `navigator.share({files:[File]})`, a fresh user gesture, so the activation requirement is met even after an asynchronous PNG render. The *Download* button falls back to `<a download>`.
* **Library:** `localStorage['phaseplane-library']` holds `{name: {date, doc: JSON string}}`; managed in `#libModal`.
  * **Export** writes `{format:'phaseplane-library', version:1, exported, items:{…}}` via `pickTarget`/`writeTarget`, so it also goes through the iOS/Android save box. `offerShare` closes other modals first.
  * **Import** (`#libInput`, multiple files) accepts library files or single drawings and validates that each `doc` parses with an `elements` array. Identical entries are skipped; name clashes get ` (2)`, ` (3)`, ….
  * Storage scope: per origin (see Round 8).
* **Drag-and-drop:** a file dropped on `#stage` is loaded with `loadDoc`.

### 3.11a Pen finishing (Round 17)

* `pointerdown` records `st.coarse = pointerType ∈ {pen, touch}`. For coarse input, a double-tap may be 480 ms and 16 px apart (mouse: 380 ms, 6 px), the "click without movement" test allows 12 px, and a drag must exceed 9 px (mouse: 3 px) to pull out pen handles; smaller wobbles leave a corner node.
* `downPen`:
  * a tap within 16 px (mouse 5 px) of the last node finishes the curve, provided it has at least 2 nodes;
  * if `penEndSnap(q)` finds a valid end target, the node is placed on it and `st.pen.finishOnUp` is set; `pointerup` then calls `finishPen(true)`. `penEndSnap` tests the curve so far plus `q` as a trajectory end with `findSnap(…,'end')`, skipping points within `SNAP_PX` of the first node. While the pen is active, `pointermove` shows this end target as a magnet ring.
* `#penBar` is an absolutely positioned div at the top centre of `#stage`, shown while the pen has nodes (`updatePenBar`). Its buttons are ✓ Finish (disabled below 2 nodes), ↶ Node (pop the last node) and ✕ (discard). `Backspace`/`Del` also pop a node while the pen is active.
* `setTool` calls `finishPen(true)`, so tapping another tool or the pen tool again keeps the curve. `Esc` still discards it.

### 3.11 Clipboard

* **Native sketch:**
  * `clipText(els)` serialises `{format:'phaseplane-clip', version:1, arrowScale, elements}`. Trajectory bindings to elements outside the copied set are set to null.
  * The text is also stored in `localStorage['phaseplane-clipboard']`, which serves as an internal clipboard per origin.
* **Copying:**
  * Keyboard: the document `copy`/`cut` events set `text/plain` synchronously. This is reliable and needs no permission.
  * Button: `navigator.clipboard.writeText`. If that fails, the copy lives in the browser only.
* **Pasting:**
  * Keyboard: the document `paste` event reads `text/plain`.
  * Button: `navigator.clipboard.readText()` (Safari shows its "Paste" bubble).
  * Either one falls back to the internal clipboard if the system text is not a sketch.
  * `parseClip` accepts clip payloads and whole `phaseplane-sketch` documents.
  * `pasteClip` gives new ids, remaps bindings inside the set, then positions the paste:
    * if the copied area is visible, it is offset by (20, 20) repeatedly while any pasted element would coincide (same type, same bounding box) with an existing one;
    * otherwise it is moved to the view centre.
  * The pasted elements are selected and committed with `{force:true}` (never refused).
  * Typing in inputs is not intercepted (`isTyping`).
* **Image:**
  * `copyImage(rect?)` renders `exportSVG({only:selectedIds})`, or the whole drawing, or `exportSVG({rect})` for a region (viewBox = rectangle, so content is clipped).
  * The result goes through `pngBlob(3, src)`.
  * `navigator.clipboard.write([new ClipboardItem({'image/png': promise})])`: the promise is passed directly so that Safari keeps the user activation.
  * Fallback: `offerShare` on mobile, PNG download on desktop.
* **Region tool** (`G`):
  * The drag produces `st.region` (world rect), drawn in the overlay.
  * `#regionBar` is an absolutely positioned div in `#stage`, placed below the rectangle's right corner, flipped above it or clamped to stay visible.
  * *Copy sketch* takes the elements whose bounding box lies fully inside the rectangle.
* **UI:** top-bar group Copy / Paste / Copy image; key `I` = copy image. `Ctrl+Shift+C` was avoided because browsers reserve it for developer tools.

### 3.12 Colours, line widths and background fills (Round 17)

**Style fields.**
* Helpers: `colOf(el, def)` validates `el.color` as hex, else returns `def`; `widthOf(el, def)`; `arrowK(w, base) = max(1, √(w/base))` scales arrowheads (and the dashes of unstable orbits, linearly) for lines thicker than the default.
* Base widths: `W_TRAJ` 1.3, `W_SEP` 2.0 (bases trajectory arrows), `W_CYC` 2.1, `W_BND` 1.5.
* The semi-stable shadow offset grows by `(w − 2.1)/2`. Equilibrium colour applies to the glyph, stubs, stub arrows and spiral (a source stays white inside); label colour to the text fill.

**Panel.**
* `styleRows(els)` builds the Colour row: `.sw` swatch buttons with `data-color` (empty = default) plus a `<label class="sw pick">` wrapping `<input type=color data-color-pick>`. Fills use `data-fcolor` and `FILL_COLORS`.
* It also builds the *Line width* `<select data-stylewidth>`: '' = auto/default, `WIDTHS` presets, the current non-preset value, and 'mixed'.
* Mixed selections show "(mixed)" and no active swatch.
* `applyStyle(kind, val)` changes all selected elements of the matching kind and commits once.

**Fill element.**
* `hasNodes(el)` = `isPath(el) || type === 'fill'`. It is used for node editing, bounding boxes, `affine` (which also maps `holes`), simplify and node insertion.
* `isPath` stays restricted to traj, cycle and boundary, so validation, the magnet and bindings never see fills.
* `ORDER.fill = −1` draws fills under everything; among fills, document order decides, adjusted by *Send to back* / *Bring to front*.
* Rendering: `fillD` = outer ring + hole rings, `fill-rule="evenodd"`, no stroke.
* In the editor an unselected fill has `pointer-events="none"`, so marquee drags work on top of fills. A selected fill is hittable, so it can be dragged.
* A click without a drag on empty space selects the topmost fill containing the point (`fillAt`, at marquee `pointerup`).

**Fill tool** (`st.fillColor` stored in preferences; `'none'` = eraser since Round 20; the *basin* mode described below was removed in Round 20).
* A drag of more than 40 px makes a freehand fill (`closedNodes`). A tap calls `bucketFill(p)`:
  1. If `fillAt(p)` hits an existing fill, recolour it.
  2. Raster window = visible area ∩ (union bounding box of non-fill, non-label elements + 24). Scale `s = min(1600/max side, max(2, 1.5·zoom))` canvas px per world unit.
  3. Draw the barrier polylines (`fillBarriers(mode)`) with canvas line width 3 px. Pixels with alpha > 40 are barriers.
     * *basin*: trajectories whose end is a separatrix port with role *end* (`isStableSep`), equilibrium stubs with role *end*, cycles that are not stable, and all boundaries.
     * *all*: every trajectory, cycle, boundary and stub.
     * Trajectory ends bound to a cycle are extended to the cycle point `C(t)`, closing the `CYC_OFF` gap.
  4. 4-connected stack flood fill from the seed (moved to a free pixel within 4 px if it lands on a line).
  5. Grow the region 3 times by one pixel, into barrier pixels only. This brings the colour under the curves and closes the slits left by curves ending inside a region, without leaking across a line.
  6. Trace the region: directed pixel-edge contour, clockwise on screen, chained per vertex with a right-turn preference. The largest positive-area loop is the outer ring; negative loops inside it with area > max(60, (10s)²) px² are holes.
  7. Each loop: `smoothPoly` (3 passes) → `rdpClosed` (ε = max(0.8, s/zoom) px, i.e. about 1 screen px) → world coordinates → closed `crNodes`. Nodes turning by more than 55° become corners (zero handles).
  8. If the region touches the raster border, the status bar reports that the region is not closed.

**Export.**
* SVG and PNG need no change.
* PDF: the path branch emits `f*` / `B*` when `fill-rule="evenodd"`.
* TikZ: colours are collected into `\definecolor{ppcN}{HTML}{RRGGBB}` lines inserted after `ppbnd`. Non-default widths become `line width=…`. Arrowhead polygons get the colour and the `arrowK` size. A fill becomes `\fill[ppcN,even odd rule] ring .. cycle ring .. cycle;`.

---

## 4. Testing approach used

The tool was tested with headless Chromium through Playwright (Python), using real mouse events mapped from world to screen coordinates. The tests covered:

* loading the example: no problems reported;
* a freehand stroke across a separatrix: refused or flagged;
* snapping, with automatic reversal when a stroke starts on a sink;
* saddle connections: refused in structurally stable mode, allowed otherwise; homoclinic loop;
* boundary direction refusal;
* closed boundary created as inflow;
* index warnings;
* pen tool; double-click node insertion; undo/redo;
* approaching a stable orbit from outside (binding `side = −1`);
* re-attaching a dragged end; an end following a moved sink;
* joining two trajectories; simplify progression;
* duplicate with refusal on;
* separatrix width;
* the export menu not being covered (`elementFromPoint`);
* toolbar wrapping and two-column tools at 1000×560 and 820×480;
* SVG/PNG export; TikZ compiled with pdflatex (standalone class).
* Round 17:
  * backward compatibility: old and new `index.html` load `fish.phase.json`, every item of `phaseplane-examples.json` and the built-in example, and produce byte-identical `exportSVG()`, `tikz()` and validation results;
  * colour and width via panel clicks, including multi-selection;
  * basin fills on the example (two basins split by the stable manifold; the outside of the unstable orbit gives a ring with a hole); recolour by tap; selection by click;
  * pen with synthetic `pointerType:'pen'` events: tap-last-node with 9 px jitter, finish on a sink (end bound to the sink), Finish and ↶ Node buttons, tool switch keeps the curve, mouse double-click;
  * TikZ with colours and fills compiled by pdflatex; direct PDF passes `qpdf --check`; rasterised outputs compared visually with the screen.

---

## 5. Known limitations and ideas for later

* Only local consistency is checked. Global realisability (e.g. adjacent anti-parallel trajectories with nothing between them) is not.
* Crossing refusal works per pair of elements: if a pair already crosses, further crossings between the same pair are not refused.
* Double-clicking a curve exactly on one of its handle dots does not insert a node.
* An end attached to a sink, source or node-sector centre has no node handle; use *Detach end*.
* Spirals are decorative and are excluded from crossing checks.
* Bucket fills are traced at screen resolution from the visible part of the drawing: zoom in for accuracy, and pan so that the whole region is visible.
* Possible extensions:
  * axes / nullcline elements;
  * a Poincaré-disc frame;
  * fills that follow later edits of the curves (currently a fill is a static shape; re-tap after editing);
  * an optional per-trajectory "asymptotic spiral" ending at foci.
