---
name: agentic-3d-modeling
description: Use when building a parametric, 3D-printable model from a natural-language or photo spec — enclosures, caps, cradles, stands, brackets — with build123d / OpenCascade, where the result must be numerically verified (fit, clearance, retention, printability) rather than eyeballed. Also when reverse-engineering a part's geometry from photos without calipers, or deciding why an OCC boolean/offset/fillet silently failed. Triggers: build123d, CAD, STEP/STL/DXF export, snap-fit, support-free, FDM overhang, flat-plate / laser-cut / waterjet, lap joint, homography from photo.
---

# Agentic 3D Modeling

## Overview

Produce parametric, 3D-printable models from a spec, **verified by measurement, not by looking**. The core discipline is the verify loop: every iteration renders to PNG **and backs the claim with a number**. A render that "looks solid" proves nothing — a hollow with no occlusion looks identical. Distilled from a series of real builds — caps, covers, cradles, enclosures, and stands (worked examples linked below).

## Step 0 — pick the fabrication form before you model (don't default to a 3D solid)

"3D-printable solid" is **not** the default medium — defaulting there is an RL bias this skill counters (the "Year-1-student" trap: clever solids that don't need to be solid). But **flatten-first is a *check*, not a new dogma** — flat plates are strong in-plane, weak out of it. Triage before you extrude:

- **Profile + uniform thickness?** (bracket, link, arm, gusset) → prototype as a **flat plate** and `export_dxf` alongside the STL — fast to iterate, re-cuttable in sheet stock. **Wrong call when the part sees out-of-plane / torsional load, an eccentric load path, high bearing stress, a precision or high-cycle pivot, or becomes a tall cantilevered spacer-stack** — there a boxed/printed solid or a proper bracket is stiffer and correct.
- **Single-axis pivot?** The real question is **shear mode, not "fork vs flat."** A 2-plate lap is **single shear** (screw bends, holes ovalize under cycling) — fine only for low-load / low-cycle. For loaded joints put the moving link in **double shear**: a center link between two outer plates — which is *exactly a clevis, built flat*. So don't fear the fork *shape*; only skip a bespoke **3D-printed** fork when flat plates give the same double shear cheaper.
- **Bearing + clamp, done right:** a stamped washer is a sacrificial slip surface, **not a bearing**. For cyclic motion use a **shoulder screw + bushing / sleeve / thrust washer**, and let a **spacer or shoulder set the clamp length so pivot friction doesn't depend on nut torque** (the nyloc *retains the stack*, it doesn't set the swing).
- **Reserve full 3D** for what needs it: enclosures, organic shells, true 3D mating to an existing object (a motor boss, a connector).

**Smell test:** "special / hard-to-describe" shape ≈ over-solidified; *nameable* (plate, bar, bracket, link) is usually enough — but **nameable ≠ adequate**, re-check the load cases above.

**DXF reality:** `ExportDXF(unit=Unit.MM); e.add_shape(sketch.sketch); e.write(path)` emits the profile, but the **same DXF is not drop-in across fabs** — laser kerf/HAZ, waterjet taper, router bit-radius/dogbones, acrylic crazing at holes, ply grain, aluminium burrs each need their own allowance. **Don't laser carbon fibre** (toxic fumes, conductive dust, wrecked optics) — CF is routed or waterjet. Printed vertical through-holes dodge the *horizontal*-hole overhang but still print undersized/faceted → **ream pivot holes**; and counterbores / countersinks / hex-pockets are still overhangs.

## Lived-experience reflexes (don't draft in a vacuum)

Failures an agent ships that a human in a workshop wouldn't — embodied knowledge that's in training but not reflexive. These apply when the model **claims an assembly, a load path, motion, or fabrication-readiness** — *not* to an isolated part you're deliberately drawing to a spec (a bracket to a bolt pattern, a spacer, a cover are fine in isolation, as long as the interfaces / datums / loads are stated).

- **Nothing floats in a claimed assembly.** If a part drives or carries load, name what fixtures it and where the reaction force goes — don't render it secured to nothing. The environment is often free fixturing + a free datum (a bench, a clamp, a table edge/underside) — consider it, but check it's actually stiff/flat/safe enough; it's an example, not a law.
- **Static placement ≠ motion validation.** Putting parts in position is not testing a mechanism. If the design depends on motion, define the moving DOF and **sweep it through its range** (sliders / animation / trace) — a single static pose hides clips, singularities, and binds. (In CAD a "mate" is an assembly *constraint*; many are deliberately static — the point is to *exercise* the DOFs, not that every mate moves.)
- **Separate the schematic from the part.** A layout/kinematic schematic (beams, blocks) legitimately has meaningless overlaps — don't read printability off it. A *manufacturable* model must pass real checks: interference (`signed_distance` / vertex-in-solid → ideally zero), bed-fit, seating, and overhang/support cost. The checks that can be numbers should be; printability isn't one scalar, and support-free is a goal, not a law.
- **Keep the printer fed — with reviewed, low-risk jobs.** If *fabrication* is the bottleneck and a candidate has passed its fit checks and is safe to run unattended, queue the smallest useful plate before downtime rather than leaving the bed idle. Not every model qualifies — don't speculatively print un-reviewed or stale revisions (that's a shoebox of obsolete plastic + overnight-failure risk, not progress). Machine utilisation ≠ engineering progress.

## Toolchain — reuse a venv, don't rebuild (and don't hardcode the path)

build123d 0.10 (Python + OpenCascade via `cadquery-ocp`) gives real fillets, true shell, and STEP (editable B-rep) **and** STL (mesh) from one source; plus `trimesh` (section + proximity), `matplotlib` (Agg, headless), numpy.

**Probe for an existing venv** — the path rotates (a disk-cleanup sweep deleted `~/cap-model/venv` once; don't trust a memorized location):
```bash
for v in ~/oak-d-iot-75-cradle/venv ~/*/venv; do
  "$v/bin/python" -c "import build123d,trimesh" 2>/dev/null && echo "USE $v" && break; done
```
(The seed candidate above is illustrative — the `~/*/venv` glob is what actually finds it; don't treat any single path as fixed.) If none found: build on a **Python 3.12** interpreter (`cadquery-ocp` has **no wheel for bleeding-edge CPython** — no cp314). On this Mac that's `/opt/homebrew/bin/python3.12`; on the Linux homelab nodes resolve `python3.12`/`which python3.12` instead (the homebrew path is Apple-Silicon-only). Never `--break-system-packages`. Helpers pulled on ImportError: `rtree` (proximity), `networkx` (mesh section).

**Exports are FREE FUNCTIONS, not methods:** `export_stl(shape, path, tolerance=.01, angular_tolerance=.1)`, `export_step(shape, path)`. (`shape.export_stl` → AttributeError.) **Units strictly mm**; resolve mixed cm/mm in the spec up front.

## Reach the empirical loop fast — the interpreter is your spatial scratchpad

You have no spatial workspace; holding the finished part in your head as prose is slow **and** error-prone — in-head reach/tolerance derivation is exactly where a dimension gets silently mis-computed (it's how the self-confirming bug below is born). Don't pre-simulate the whole part before writing code. Get the smallest runnable script — one solid, one cut, one `print(solid.bounding_box())` — executing in the first minute, then let measurement, not mental imagery, carry the geometry forward. OCC *can* fail silently (empty booleans, no-op offsets, swallowed fillets); only *running* surfaces it, so a minute of `python build.py` beats ten of in-head deliberation. Leave the printed numbers and asserts in the script — they are the audit trail that makes a long build transparent to the next reader.

## The verify loop (the core — render → section → MEASURE)

Each iteration: render PNG, `Read` it, **and** attach a number.

- **Section, not 3D plot.** matplotlib 3D (`Poly3DCollection`) has no occlusion. Verify with a true **2D cross-section** — B-rep edges, or `trimesh.section(plane_origin, plane_normal)` on the exported mesh — plus numeric probes.
- **`faces().filter_by(Plane.XZ)` misclassifies cut faces on lofted/tilted solids** (empty section) → fall back to `trimesh.section` on the mesh.
- **Make every constraint a falsifiable metric:**
  - *fit/clearance (two bodies):* `trimesh.proximity.signed_distance` / `closest_point`; report min/max/mean/σ; want 0 penetration.
  - *wall thickness (one hollow body):* `signed_distance` against a watertight mesh of itself doesn't directly give it. Two ways: (a) take the `trimesh.section` at the cut and measure outer-edge → inner-edge distance numerically on the 2D section polygon; or (b) sample points on the outer surface (`mesh.sample(n)` filtered to outward-normal faces) and ray-cast inward (`mesh.ray.intersects_location`) — the first hit distance is the local wall. Report min/mean to confirm the intended thickness.
  - *retention:* the cradle math below.
  - *printability:* the overhang math below.
- **Assert the volume drop after every boolean cut** — the single highest-value check (catches wrong-direction extrudes, missed booleans).
- **A check fed the value it's testing is not a test (self-confirming verification).** Probe the *as-built* solid, never the nominal parameter or an unconfirmed assumption you fed in: `bb = solid.bounding_box(); assert abs(bb.max.X - R_OUT) < tol` (`tol` ~1e-6 on a B-rep; looser on a mesh), read contact/reach off the mesh extent or `trimesh.section`, not off the constant. Burned once — a rounded nose `Circle(FW/2)` *centred* at the tip radius put real material `FW/2` beyond it; the verify checked the tip radius and certified a part whose true reach ran 8 mm into the keep-out it was meant to clear. Same trap for guessed interfaces (a bolt PCD, an across-flats): state the assumption to the operator up front — a part that "verifies" against its own guess proves nothing.
- Render gravity-aligned views (y=0 side cut + ⟂-axis cross-section, gravity up), not raw point clouds.
- After any parameter change, **re-grep artifacts** for hardcoded old values (plot titles, comments) — a stale caption on correct geometry is a misleading deliverable.

## Geometry gotchas (each hit and solved)

- **Winding sets extrude direction.** `extrude(sketch, +h)` follows the face normal; a **clockwise**-wound `Polygon` normals **−Z** → extrudes *downward*, and a CW cut profile silently **misses** the boolean (symptom: volume barely drops). `Rectangle`/`Trapezoid` helpers are CCW (safe); hand-built `Polygon([...])` is the trap — list points **CCW**. Assert the volume drop.
- **OCC 3D offset dies on a `scale()`d sphere** (`Standard_Failure: BRep_API: command not done`, degenerate pole/seam). Build domes as a **loft of stacked ellipse sections** instead — also yields a vertical equator tangent that blends into a straight wall.
- **Uniform-wall open cap:** `offset(solid,+t) − solid`, then **`split` flat at the rim plane** → sharp planar edges → fillet **inner only**. Avoid `Kind.ARC`/`Kind.INTERSECTION` on curved offsets when an outer edge must stay sharp.
- **Fillet radius ≥ wall thickness consumes the whole wall** — cap it (e.g. 0.5 mm on a 1 mm wall) and flag the judgment call.
- **Don't blanket-mirror features.** A Y-mirror of an X-symmetric profile looks like a 180° rotation — flips chamfer/keying. Mirror only features referenced to the *moved* edge; edge-referenced features ("5 mm from wide edge") are flip-invariant and must NOT be mirrored (mirroring = regression).
- **Vertical truncation cut:** to amputate a region of a tilted part cleanly, subtract a world-z half-space box, not the irregular feature boundary.

## Anchor first — solve load-bearing geometry before details

For "what angle does it rest at": find the natural tilt by **minimizing CG height vs lean angle** — rotate the mesh through θ, compute `cg_z − min_vertex_z` (CG height after dropping to ground), take the local min. Don't hand-derive contacts. **Then ENFORCE that pose** by building the seat at it — the part fixes the angle, robust to unknown CG/weight (weight only affects the stability margin).

## Cradle retention math (snap-capture a cylinder)

A trough retains a cylinder only if **opening width < cylinder diameter**:
`opening = 2·(r + clearance)·cos(wrap/2 − 90)` must be `< 2r`.
- **`clearance`** = the print gap you cut the trough oversize by, in mm, **positive** (typical FDM 0.2–0.4 mm so the rod actually inserts; 0 only for a theoretical check). A bigger clearance *widens* the opening → harder to retain, so size wrap against the clearance you'll actually print.
- **Angle convention: degrees.** The `90` is degrees and `wrap` is in degrees. In code, `math.cos` takes **radians** — convert: `math.cos(math.radians(wrap/2 - 90))`. A literal `math.cos(wrap/2 - 90)` is silently wrong.
- Needs **wrap > 180°** by a real margin. 190° ≈ 99.6% of dia = **no retention** (lifts straight out). **~220° ≈ 1 mm constriction, grips ~67% of height** = real snap-fit (press in with slight wall flex).
- "Grip >50% of height" = lips past the equator; height gripped `= r·(1 + sin(wrap/2 − 90))`.

## Printability metric (FDM, support-free)

Per face: if it faces **down** (`normal_z < 0`) AND is above the bed, surface incline from horizontal `= arccos(|normal_z|)`; **needs support if < ~45°**. **CRITICAL: exclude bed-contact faces (`centroid_z < ~0.6 mm`)** or the flat bottom false-flags as a 0°-overhang ceiling (it's the first layer). Design tricks: print top-plate-down for caps; a **0.2 mm membrane** closing a counterbore floor lets the printer *bridge* a flat instead of an overhang ledge (drill through after).

## Reverse-engineer geometry from photos (no calipers)

- **Credit card = free scale ruler.** ISO/IEC 7810 ID-1 = 85.60×53.98 mm; threshold its color + `scipy.ndimage.label` for px→mm (check X-vs-Y anisotropy = perspective).
- **Known hole pattern = free homography anchor.** A Pi-stackable board uses **58×49 mm, Ø2.7 (M2.5), 3.5 mm inset**; those 4 holes give an exact 4-point DLT homography (numpy) → rectify to mm, read outline + connector X-positions. **Classical CV (scipy blobs + DLT) beats YOLO** here — deterministic, sub-mm, no training set.
- **What a photo CANNOT give:** connector **heights** above the PCB → look up by part type (KF301 ~10, DC barrel ~11, JST XH ~6 / PH ~5, Type-C ~3.2, 2.54 header ~8.5 mm) or caliper on arrival. This is why a **SAFE variant** (walls open to rim, height-tolerant) is worth carrying alongside the tight BEAUTIFUL one.
- **1:1 PDF for physical overlay:** `figsize=(w_mm/25.4, h_mm/25.4)`, `ax=fig.add_axes([0,0,1,1])`, data limits = part extent, `aspect equal`, **NO `bbox_inches='tight'`** (it rescales!). Add a 10 mm verify-bar; user prints at **100% (no fit-to-page)**, overlays the real part, reports per-axis residuals (screen overlay gives **per-axis**, not uniform, scale error — correct each axis independently; handedness/flip is the #1 confusion).

## Workflow patterns

- **Parametric, one source.** Knobs as constants at the top; a dimensional variant is a one-line change or a `VARIANTS` list, **not a forked file**.
- **Emit BOTH variants, never overwrite one file** — `VARIANTS=[("cap_dev",False),("cap_sleek",True)]` writes both `.step/.stl`. Overwriting a single `cap.stl` lost a good cap once.
- Driving Bambu Studio to slice: see memory `bambu-studio-computer-use` (native file dialogs are invisible to screenshots → `open -a BambuStudio file.stl`).

## Common mistakes

| Mistake | Fix |
|---|---|
| Defaulting to a 3D solid without checking if it's a plate problem | Step 0 triage; flat plate for in-plane loads, **double-shear + bushing** for loaded pivots (not a single-shear washer-lap), `export_dxf` for sheet |
| "It renders solid" as proof | 3D plots have no occlusion — section + a number |
| A metric computed from the input parameter / a guessed dimension | Self-confirming, not falsifying — probe the as-built solid (`assert bounding_box().max ≈ expected`); state guessed interfaces to the operator |
| Hardcoding a venv path | Probe; the path rotates (cap-model/venv was deleted) |
| `shape.export_stl(...)` | Exports are free functions: `export_stl(shape, path)` |
| CW-wound `Polygon` cut silently misses | List points CCW; assert volume drop per cut |
| `offset()` on a `scale()`d sphere crashes | Loft stacked ellipse sections for domes |
| Blanket-mirroring all features | Mirror only moved-edge-referenced ones |
| Flat print-bed face flagged as overhang | Exclude `centroid_z < ~0.6 mm` from the metric |
| Trusting a photo for connector heights | Photo gives X/Y only; heights from part-type lookup → SAFE variant |

## Worked examples

Models built end-to-end with this discipline: [coin-stand](https://github.com/evnchn-agentic/coin-stand) (snap-off-support coin clips), [connector-cap](https://github.com/evnchn-agentic/connector-cap) (photo-reverse-engineered, no CAD), [gimbal-base-cover](https://github.com/evnchn-agentic/gimbal-base-cover) (support-free clamshell), [power-board-enclosure](https://github.com/evnchn-agentic/power-board-enclosure) (homography recon before the board arrived).
