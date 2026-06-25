# agentic-3d-modeling

A Claude Code skill for building **parametric, 3D-printable models that are numerically
verified, not eyeballed** — with build123d / OpenCascade + trimesh. Covers the verify
loop (render → section → MEASURE), the OCC geometry gotchas that bite silently
(winding-sets-extrude-direction, offset-on-scaled-sphere, blanket-mirror regressions),
snap-fit retention math, FDM support-free printability math, and reverse-engineering a
part's geometry from photos via a credit-card scale ruler + known-hole-pattern homography.

Distilled from a series of real builds — caps, covers, cradles, enclosures, and stands
(worked examples linked below).

## The one load-bearing idea

A 3D render that "looks solid" proves nothing — a hollow with no occlusion looks
identical. Every constraint (fit, clearance, retention, printability) must become a
**falsifiable number** (trimesh proximity, a true 2D section, a volume-drop assertion),
not a vibe. That's the whole discipline.

## Install (Claude Code)

```bash
git clone https://github.com/evnchn-agentic/agentic-3d-modeling.git \
  ~/.claude/skills/agentic-3d-modeling
```

## Worked examples

Each is one model built end-to-end with this discipline:

- [**coin-stand**](https://github.com/evnchn-agentic/coin-stand) — shelf-edge coin-display clips with a Slant3D-style designed snap-off support (a 1.2×0.2 mm bridge line)
- [**connector-cap**](https://github.com/evnchn-agentic/connector-cap) — a commercial robot's connector-panel cover, reverse-engineered from photos because no CAD exists
- [**gimbal-base-cover**](https://github.com/evnchn-agentic/gimbal-base-cover) — a two-part clamshell engineered to print fully support-free
- [**power-board-enclosure**](https://github.com/evnchn-agentic/power-board-enclosure) — a tray-and-lid enclosure built *before the board arrived*, via homography on its mounting holes
